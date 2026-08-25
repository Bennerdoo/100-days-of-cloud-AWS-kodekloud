# Question

The Nautilus DevOps team needs to set up a new EC2 instance that can be accessed securely from their landing host (aws-client). The instance should be of type `t2.micro` and named `xfusion-ec2`. A new SSH key with name `id_rsa` should be created on the `aws-client` host under the `/root/.ssh/` folder, if it doesn't already exist. This key should then be added to the `root` user's authorised keys on the EC2 instance, allowing passwordless SSH access from the `aws-client` host.

# Step-by-Step Solution

### Step 1: Generate RSA Key Pair

Generate the key on aws-client if it doesn't already exist:

```bash
ssh-keygen -t rsa -b 2048 -f /root/.ssh/id_rsa -N ""
```

### Step 2: Copy Public Key

Output and copy the public key content:

```bash
cat /root/.ssh/id_rsa.pub
```

### Step 3: Log In as ec2-user and Switch to Root

SSH into the instance using default credentials and switch to a root shell:

```bash
ssh ec2-user@<EC2-PUBLIC-IP>
sudo su -
```

### Step 4: Replace /root/.ssh/authorized_keys

Ensure the directory exists, then overwrite the file (using `>`) to remove Amazon Linux's default forced-command restriction on root:

```bash
mkdir -p /root/.ssh
chmod 700 /root/.ssh
echo "<PASTE_YOUR_PUBLIC_KEY_HERE>" > /root/.ssh/authorized_keys
chmod 600 /root/.ssh/authorized_keys
```

### Step 5: Enable Root SSH Login & Restart Service

Ensure SSH allows root public-key authentication, then restart sshd:

```bash
sed -i 's/^#*PermitRootLogin.*/PermitRootLogin prohibit-password/' /etc/ssh/sshd_config
systemctl restart sshd
```

### Verification

From aws-client, run:

```bash
ssh root@<EC2-PUBLIC-IP>
```

You will directly enter the root shell without passwords or redirection prompts.