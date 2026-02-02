---

# EC2 Memory & Disk Monitoring with CloudWatch

Monitor **EC2 Memory and Disk utilization** using CloudWatch and SSM.

---

## Steps

### 1️⃣ Create IAM Role

* Name: `EC2-CloudWatch-Role`
* Policies: `CloudWatchFullAccess`, `AmazonSSMFullAccess`

### 2️⃣ Create SSM Parameter

* Name: `/alarm/AWS-CWAgentLinConfig`
* Type: String
* Value:

```json
{
  "metrics": {
    "append_dimensions": { "InstanceId": "${aws:InstanceId}" },
    "metrics_collected": {
      "mem": { "measurement": ["mem_used_percent"], "metrics_collection_interval": 60 },
      "disk": { "measurement": ["disk_used_percent"], "metrics_collection_interval": 60 }
    }
  }
}
```

### 3️⃣ Launch EC2 & Install CloudWatch Agent

* Attach IAM role.
* Add **Userdata** script:

```bash
#!/bin/bash
wget https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip
unzip AmazonCloudWatchAgent.zip
sudo ./install.sh
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c ssm:/alarm/AWS-CWAgentLinConfig -s
```

### 4️⃣ Verify Agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```

### 5️⃣ View Metrics

* Go to **CloudWatch → Metrics → CWAgent**
* Monitor `mem_used_percent` and `disk_used_percent`.

---

✅ Done! Your EC2 instance now reports **Memory & Disk usage to CloudWatch**.

---
