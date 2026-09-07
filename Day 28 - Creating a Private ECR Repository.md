# Question

The Nautilus DevOps team has been tasked with setting up a containerized application. They need to create a private Amazon Elastic Container Registry (ECR) repository to store their Docker images. Once the repository is created, they will build a Docker image from a Dockerfile located on the aws-client host and push this image to the ECR repository. This process is essential for maintaining and deploying containerized applications in a streamlined manner.

- Create a private ECR repository named `datacenter-ecr`. 
- There is a Dockerfile under `/root/pyapp` directory on aws-client host, build a docker image using this Dockerfile and push the same to the newly created ECR repo, the image tag must be `latest`.

# Step by Step Solution

### Step 1: Create the private ECR repository
```bash
aws ecr create-repository --repository-name datacenter-ecr
```

ECR repositories are private by default, so no extra flag is needed. Note the repositoryUri from the output (e.g. 338912023448.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr) — you'll need it in later steps.

### Step 2: Authenticate Docker to ECR
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 338912023448.dkr.ecr.us-east-1.amazonaws.com
```

Replace the account ID/region with your own values. This fetches a temporary token and logs Docker into your ECR registry. You should see Login Succeeded.

### Step 3: (If needed) Fix cgroup driver compatibility issue

On some hosts, building will fail at any RUN step with:

OCI runtime create failed: ... setting cgroup config for procHooks process caused: can't load program: operation not permitted

This happens when Docker is on cgroup v2 with the systemd cgroup driver, which has known compatibility problems with certain runc/containerd versions. Fix it by switching to the cgroupfs driver:

```bash
cat <<EOF | sudo tee /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=cgroupfs"]
}
EOF
```

```bash
sudo systemctl restart docker
```

Verify with `docker info 2>&1 | grep -i "cgroup driver"` — it should now show cgroupfs. (Skip this step entirely if your build works fine without it.)

### Step 4: Build the Docker image
```bash
cd /root/pyapp
docker build -t 338912023448.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest .
```

Tagging the image with the full ECR repository URI plus :latest at build time means no separate docker tag step is needed later.

### Step 5: Push the image to ECR
```bash
docker push 338912023448.dkr.ecr.us-east-1.amazonaws.com/datacenter-ecr:latest
```

This uploads the image to the datacenter-ecr repository under the latest tag.

### Step 6 (optional): Verify the push
```bash
aws ecr list-images --repository-name datacenter-ecr
```

You should see the latest tag listed, confirming the image is in the repository.

Summary of commands (happy path, assuming no cgroup issue):

```bash
aws ecr create-repository --repository-name datacenter-ecr
aws ecr get-login-password --region <region> | docker login --username AWS --password-stdin <account-id>.dkr.ecr.<region>.amazonaws.com
cd /root/pyapp
docker build -t <account-id>.dkr.ecr.<region>.amazonaws.com/datacenter-ecr:latest .
docker push <account-id>.dkr.ecr.<region>.amazonaws.com/datacenter-ecr:latest
```