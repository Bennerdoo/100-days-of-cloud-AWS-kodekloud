# Question

The Nautilus DevOps team is currently working on setting up a simple application on the AWS cloud. They aim to establish an Application Load Balancer (ALB) in front of an EC2 instance where an Nginx server is currently running. While the Nginx server currently serves a sample page, the team plans to deploy the actual application later.

1. Set up an Application Load Balancer named `xfusion-alb`.
2. Create a target group named `xfusion-tg`.
3. Create a security group named `xfusion-sg` to open port `80` for the public.
4. Attach this security group to the ALB.
5. The ALB should route traffic on port `80` to port `80` of the `xfusion-ec2` instance.
6. Make appropriate changes in the default security group attached to the EC2 instance if necessary.

# Step-by-Step Solution

### 1. Set variables

Run these on the AWS client/control host:

#### *AWS CLI*

```Bash
REGION=$(aws configure get region)
VPC_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query 'Reservations[0].Instances[0].VpcId' \
  --output text)

INSTANCE_ID=$(aws ec2 describe-instances \
  --filters "Name=tag:Name,Values=xfusion-ec2" \
  --query 'Reservations[0].Instances[0].InstanceId' \
  --output text)

echo "Region: $REGION"
echo "VPC: $VPC_ID"
echo "Instance: $INSTANCE_ID"
```

Verify that the instance was found:

```Bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PrivateIpAddress,VpcId]' \
  --output table
```

### 2. Get two subnets for the ALB

An Application Load Balancer should normally be associated with subnets in at least two Availability Zones.
```bash
SUBNETS=$(aws ec2 describe-subnets \
  --filters "Name=vpc-id,Values=$VPC_ID" \
  --query 'Subnets | sort_by(@,&AvailabilityZone)[:2].SubnetId' \
  --output text)

echo "$SUBNETS"
```

You should get two subnet IDs.

If the environment has specifically provided subnets, use those instead.

### 3. Create xfusion-sg

Create the security group in the same VPC:
```bash
SG_ID=$(aws ec2 create-security-group \
  --group-name xfusion-sg \
  --description "Security group for xfusion ALB" \
  --vpc-id "$VPC_ID" \
  --query 'GroupId' \
  --output text)

echo "$SG_ID"
```
Open HTTP port 80 to the public:
```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$SG_ID" \
  --ip-permissions \
    IpProtocol=tcp,FromPort=80,ToPort=80,IpRanges='[{CidrIp=0.0.0.0/0,Description="Public HTTP access"}]'
```
**Verify:**
```bash
aws ec2 describe-security-groups \
  --group-ids "$SG_ID" \
  --query 'SecurityGroups[0].[GroupName,GroupId,IpPermissions]' \
  --output json
  ```
### 4. Create the target group

Create `xfusion-tg` for HTTP traffic on port 80:
```bash
TG_ARN=$(aws elbv2 create-target-group \
  --name xfusion-tg \
  --protocol HTTP \
  --port 80 \
  --vpc-id "$VPC_ID" \
  --target-type instance \
  --health-check-protocol HTTP \
  --health-check-port traffic-port \
  --health-check-path / \
  --query 'TargetGroups[0].TargetGroupArn' \
  --output text)

echo "$TG_ARN"
```
This tells the ALB:
```

ALB :80  --->  xfusion-tg :80  --->  xfusion-ec2 :80
```
### 5. Register xfusion-ec2 with the target group
```bash
aws elbv2 register-targets \
  --target-group-arn "$TG_ARN" \
  --targets Id="$INSTANCE_ID",Port=80
```
**Check the registration:**
```bash
aws elbv2 describe-target-health \
  --target-group-arn "$TG_ARN"
```
Initially, the state may be initial. After the health check succeeds, it should become:
```
healthy
```
### 6. Create the Application Load Balancer

Create an internet-facing ALB named xfusion-alb:
```bash
ALB_ARN=$(aws elbv2 create-load-balancer \
  --name xfusion-alb \
  --subnets $SUBNETS \
  --security-groups "$SG_ID" \
  --scheme internet-facing \
  --type application \
  --ip-address-type ipv4 \
  --query 'LoadBalancers[0].LoadBalancerArn' \
  --output text)

echo "$ALB_ARN"
```
**Get its DNS name:**
```bash
aws elbv2 describe-load-balancers \
  --load-balancer-arns "$ALB_ARN" \
  --query 'LoadBalancers[0].[LoadBalancerName,DNSName,State.Code]' \
  --output table
```

### 7. Create the port 80 listener

Create a listener that accepts public HTTP traffic on port 80 and forwards it to xfusion-tg:
```bash
LISTENER_ARN=$(aws elbv2 create-listener \
  --load-balancer-arn "$ALB_ARN" \
  --protocol HTTP \
  --port 80 \
  --default-actions Type=forward,TargetGroupArn="$TG_ARN" \
  --query 'Listeners[0].ListenerArn' \
  --output text)

echo "$LISTENER_ARN"
```
**The resulting architecture is:**

                    Internet
                       |
                       | HTTP :80
                       v
              +-------------------+
              |   xfusion-alb     |
              | Application LB    |
              +-------------------+
                       |
                       | HTTP :80
                       v
              +-------------------+
              |   xfusion-tg      |
              | Target Group      |
              +-------------------+
                       |
                       | HTTP :80
                       v
              +-------------------+
              |   xfusion-ec2     |
              |      Nginx        |
              |       :80         |
              +-------------------+

### 8. Allow the ALB to reach the EC2 instance

This is the important part of the security-group configuration.

The ALB accepts:
```
Internet ---> xfusion-sg :80
```
But the EC2 instance must also allow:
```
xfusion-sg ---> EC2 :80
```
First find the security groups attached to the EC2 instance:
```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].SecurityGroups[*].[GroupId,GroupName]' \
  --output table
```
If the instance's security group is, for example, sg-xxxxxxxx, add an inbound rule allowing HTTP traffic from xfusion-sg:
```bash
EC2_SG_ID=$(aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].SecurityGroups[0].GroupId' \
  --output text)
```
```bash
aws ec2 authorize-security-group-ingress \
  --group-id "$EC2_SG_ID" \
  --ip-permissions \
    IpProtocol=tcp,FromPort=80,ToPort=80,UserIdGroupPairs="[{GroupId=$SG_ID,Description='HTTP from xfusion ALB'}]"
```

This is preferable to opening port 80 on the EC2 instance to 0.0.0.0/0.

**Important**

You do not need to remove the existing default security-group rules unless the task specifically requires it.

The desired security model is:

Public Internet
      |
      | TCP 80
      v
+----------------+
|  xfusion-sg    |
|     ALB        |
+----------------+
      |
      | TCP 80
      | source = xfusion-sg
      v
+----------------+
| EC2 SG         |
| xfusion-ec2    |
+----------------+
      |
      v
    Nginx :80

### 9. Verify the ALB

Check the load balancer:
```bash
aws elbv2 describe-load-balancers \
  --names xfusion-alb \
  --query 'LoadBalancers[0].[LoadBalancerName,DNSName,Scheme,State.Code]' \
  --output table
```
Check the listener:
```bash
aws elbv2 describe-listeners \
  --load-balancer-arn "$ALB_ARN" \
  --query 'Listeners[*].[Protocol,Port,DefaultActions[0].Type,DefaultActions[0].ForwardConfig.TargetGroups[0].TargetGroupArn]' \
  --output table
```
Check the target:
```bash
aws elbv2 describe-target-health \
  --target-group-arn "$TG_ARN" \
  --query 'TargetHealthDescriptions[*].[Target.Id,Target.Port,TargetHealth.State,TargetHealth.Reason]' \
  --output table
```
You want to see:
```
Target.Id       Target.Port    State
--------------  ------------   -------
i-xxxxxxxxxxx   80             healthy
```
### 10. Test through the ALB

Retrieve the DNS name:
```bash
ALB_DNS=$(aws elbv2 describe-load-balancers \
  --names xfusion-alb \
  --query 'LoadBalancers[0].DNSName' \
  --output text)

echo "$ALB_DNS"
```
Then test:
```bash
curl http://$ALB_DNS
```

You should receive the sample Nginx page.

You can also test headers:

```bash
curl -I http://$ALB_DNS
```

A successful response should look approximately like:

```
HTTP/1.1 200 OK
Server: nginx/...
Content-Type: text/html
```

### Final validation checklist

| Requirement | Configuration |
|-------------|---------------|
| ALB name    | xfusion-alb   |
| ALB type    | Application Load Balancer |
| ALB scheme  | Internet-facing |
| ALB listener| HTTP :80 |
| ALB security group | xfusion-sg |
| Public access	| 0.0.0.0/0 → TCP 80 |
| Target group	| xfusion-tg |
| Target type	| Instance |
| Target	| xfusion-ec2 |
| Target port	| 80 |
| Health check	| HTTP / |
| EC2 inbound	| TCP 80 from xfusion-sg |
| Backend	| Nginx on xfusion-ec2:80 |

### One common mistake to avoid

Do not configure the EC2 security group simply as:

TCP 80 from 0.0.0.0/0

if you can avoid it. The cleaner configuration is:

xfusion-sg → EC2 security group → TCP 80

That means the instance receives HTTP traffic from the ALB rather than directly exposing its HTTP port to the entire Internet.