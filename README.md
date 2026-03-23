<div align="center">

<br/>

```
 █████╗ ██╗    ██╗███████╗    ██╗███╗   ██╗███████╗██████╗  █████╗
██╔══██╗██║    ██║██╔════╝    ██║████╗  ██║██╔════╝██╔══██╗██╔══██╗
███████║██║ █╗ ██║███████╗    ██║██╔██╗ ██║█████╗  ██████╔╝███████║
██╔══██║██║███╗██║╚════██║    ██║██║╚██╗██║██╔══╝  ██╔══██╗██╔══██║
██║  ██║╚███╔███╔╝███████║    ██║██║ ╚████║██║     ██║  ██║██║  ██║
╚═╝  ╚═╝ ╚══╝╚══╝ ╚══════╝    ╚═╝╚═╝  ╚═══╝╚═╝     ╚═╝  ╚═╝╚═╝  ╚═╝

 ██████╗ ██████╗ ███╗   ██╗████████╗██████╗  ██████╗ ██╗
██╔════╝██╔═══██╗████╗  ██║╚══██╔══╝██╔══██╗██╔═══██╗██║
██║     ██║   ██║██╔██╗ ██║   ██║   ██████╔╝██║   ██║██║
██║     ██║   ██║██║╚██╗██║   ██║   ██╔══██╗██║   ██║██║
     ╚██████╗╚██████╔╝██║ ╚████║   ██║   ██║  ██║╚██████╔╝███████╗
      ╚═════╝ ╚═════╝ ╚═╝  ╚═══╝   ╚═╝   ╚═╝  ╚═╝ ╚═════╝ ╚══════╝
```

### **Serverless EC2 Lifecycle Monitoring, Automation & Alerting on AWS**

<br/>

![Python](https://img.shields.io/badge/Python-3.12-3776AB?style=for-the-badge&logo=python&logoColor=white)
![AWS Lambda](https://img.shields.io/badge/AWS_Lambda-Serverless-FF9900?style=for-the-badge&logo=awslambda&logoColor=white)
![Amazon EC2](https://img.shields.io/badge/Amazon_EC2-Compute-FF9900?style=for-the-badge&logo=amazonec2&logoColor=white)
![EventBridge](https://img.shields.io/badge/EventBridge-Event_Driven-FF4F8B?style=for-the-badge&logo=amazonaws&logoColor=white)
![SNS](https://img.shields.io/badge/SNS-Notifications-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
![S3](https://img.shields.io/badge/Amazon_S3-Persistent_Logs-569A31?style=for-the-badge&logo=amazons3&logoColor=white)
![Boto3](https://img.shields.io/badge/Boto3-AWS_SDK-232F3E?style=for-the-badge&logo=amazonaws&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

<br/>

> *"Infrastructure that talks back — real-time visibility, zero servers, zero guesswork."*

<br/>

</div>

---

## 📌 Overview

**AWS InfraControl CLI Tool** is a fully serverless cloud infrastructure management and observability system built on AWS. It captures every EC2 instance lifecycle event — start, stop, reboot, terminate — and responds in real time: firing email alerts via **Amazon SNS**, persisting structured event logs to **Amazon S3**, and exposing a set of **Python CLI scripts** powered by **Boto3** for direct instance control from the terminal.

The system is event-driven at its core. There is no polling loop, no cron job, and no always-on server. Instead, **Amazon EventBridge** listens passively for EC2 state-change notifications from the AWS event bus and triggers a **Lambda function** the instant something happens — making the entire monitoring pipeline reactive, cost-free at idle, and infinitely scalable.

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                          AWS Cloud Environment                          │
│                                                                         │
│   ┌──────────────┐     State Change      ┌──────────────────────────┐  │
│   │              │  ──────────────────►  │    Amazon EventBridge    │  │
│   │  Amazon EC2  │                       │  (EC2 State-Change Rule) │  │
│   │  Instances   │                       └──────────┬───────────────┘  │
│   └──────────────┘                                  │                  │
│                                                     │  Trigger         │
│                                                     ▼                  │
│                                        ┌────────────────────────┐      │
│                                        │      AWS Lambda        │      │
│                                        │   lambda_function.py   │      │
│                                        │     Python 3.12        │      │
│                                        └─────────┬──────────────┘      │
│                                                  │                     │
│                          ┌───────────────────────┴──────────────────┐  │
│                          │                                          │  │
│                          ▼                                          ▼  │
│              ┌─────────────────────┐                ┌─────────────────┐│
│              │    Amazon SNS       │                │   Amazon S3     ││
│              │  (Email Alerts to   │                │  (JSON Event    ││
│              │   Subscribers)      │                │   Audit Logs)   ││
│              └─────────────────────┘                └─────────────────┘│
│                                                                         │
│  ┌─────────────────────────────────────────────────────────────────┐   │
│  │               Local CLI Scripts  ·  Python + Boto3              │   │
│  │  auto_terminate.py │ terminate_instance.py │ recent_instance.py │   │
│  └─────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

### Event Flow

| Step | What Happens |
|---|---|
| **1** | EC2 instance transitions state (e.g., `pending → running`, `running → stopping`) |
| **2** | AWS emits an **EC2 Instance State-change Notification** to the default EventBridge bus |
| **3** | EventBridge matches the event against the configured rule and **invokes Lambda** asynchronously |
| **4** | `lambda_function.py` parses the payload, publishes to **SNS** (email alert fired) |
| **5** | Lambda also serializes the event as JSON and writes it to **S3** with a timestamped key |
| **6** | Independently, CLI scripts can terminate EC2 instances directly via Boto3 |

---

## 🔬 Component Deep-Dive

### `lambda_function.py` — The Serverless Event Handler

The core of the system. Deployed to AWS Lambda on Python 3.12 runtime, it acts as the central hub for all EC2 event processing:

- Receives the raw **EventBridge event payload** via the `event` argument
- Extracts `instance-id`, `state`, region, and timestamp from the `detail` block
- Constructs a human-readable alert and publishes it to an SNS Topic via `boto3.client('sns').publish()`
- Serializes the full event detail to JSON and writes it to an S3 bucket via `boto3.client('s3').put_object()` with a timestamp-based object key for uniqueness and chronological ordering in the log store

```python
# Illustrative flow — lambda_function.py
import boto3, json, datetime

def lambda_handler(event, context):
    instance_id = event['detail']['instance-id']
    state       = event['detail']['state']

    # Publish SNS alert
    boto3.client('sns').publish(
        TopicArn = SNS_TOPIC_ARN,
        Subject  = f"[InfraControl] EC2 State Change: {instance_id}",
        Message  = f"Instance {instance_id} transitioned to: {state}"
    )

    # Persist structured log to S3
    key = f"ec2-events/{datetime.datetime.utcnow().isoformat()}-{instance_id}.json"
    boto3.client('s3').put_object(
        Bucket = S3_BUCKET,
        Key    = key,
        Body   = json.dumps(event['detail'], indent=2)
    )

    return {"statusCode": 200}
```

The Lambda execution role follows the **principle of least privilege** — scoped only to `sns:Publish`, `s3:PutObject`, and CloudWatch Logs for observability.

---

### `auto_terminate.py` — Countdown-Based Termination

Implements **graceful, time-delayed termination** for EC2 instances. Accepts a target instance ID and countdown duration (in seconds), prints elapsed time to the terminal, then terminates the instance via `boto3.client('ec2').terminate_instances()` after the timer expires.

Designed for ephemeral workloads, automated lab teardowns, or any workflow where a predictable shutdown window is required — without needing to manually track the clock.

---

### `terminate_instance.py` — Targeted Direct Termination

A clean, focused script for **terminating a specific EC2 instance by ID**. Accepts the instance ID as a CLI argument or interactive prompt, optionally includes a confirmation step to prevent accidental termination of critical instances, then fires `terminate_instances()` immediately.

---

### `recent_instance.py` — Most-Recently-Launched Termination

The most intelligent CLI script in the toolkit. Operates without needing any instance ID input:

1. Calls `ec2.describe_instances()` with a `running` state filter
2. Sorts all returned instances by `LaunchTime` in descending order
3. Selects and terminates the **most recently launched** one automatically

Ideal for CI/CD pipelines, test environments, and any workflow where instances are spun up frequently and cleanup always targets the freshest one.

---

## ✨ Features

| Feature | Detail |
|---|---|
| ⚡ **Event-Driven Monitoring** | Zero-polling — EventBridge reacts to EC2 state changes the moment they occur |
| 📧 **Real-Time Email Alerts** | SNS publishes instant email notifications to all subscribers on any state change |
| 🗃️ **Structured S3 Audit Log** | Every event persisted as a timestamped JSON object — queryable, archivable, auditable |
| 🔧 **Python CLI Automation** | Three Boto3-powered scripts for direct EC2 lifecycle control from the terminal |
| ⏱️ **Countdown Termination** | Time-delayed auto-termination for ephemeral and test workloads |
| 🆕 **Smart Latest-Instance Targeting** | Automatically identify and terminate the most recently launched EC2 instance |
| 🔒 **IAM Least-Privilege Design** | Lambda role scoped to only the permissions it actually needs |
| 💸 **Zero Idle Cost** | Fully serverless — Lambda and EventBridge only incur cost on invocation |

---

## 🧰 Tech Stack

| Component | Technology |
|---|---|
| Language | Python 3.12 |
| AWS SDK | Boto3 |
| Serverless Compute | AWS Lambda |
| Event Router | Amazon EventBridge |
| Notification Service | Amazon SNS (Simple Notification Service) |
| Log Storage | Amazon S3 |
| Managed Compute | Amazon EC2 |
| Access Control | AWS IAM (least-privilege execution role) |
| CLI Interface | AWS CLI + custom Python scripts |

---

## 🛠️ Setup & Deployment

### Prerequisites

- AWS account with permissions to create Lambda, EventBridge rules, SNS topics, S3 buckets, and IAM roles
- **AWS CLI** installed and configured
- **Python 3.12** with `boto3` installed
- SNS topic created with your email address subscribed and confirmed
- S3 bucket created for log storage

### Step 1 — Configure AWS CLI

```bash
aws configure
# Provide: Access Key ID, Secret Access Key, Default Region, Output Format
```

### Step 2 — Install Boto3

```bash
pip install boto3
```

### Step 3 — Deploy the Lambda Function

```bash
# Package the handler
zip lambda_function.zip lambda_function.py

# Create the Lambda function
aws lambda create-function \
  --function-name EC2InfraControlMonitor \
  --runtime python3.12 \
  --role arn:aws:iam::<ACCOUNT_ID>:role/<LAMBDA_EXECUTION_ROLE> \
  --handler lambda_function.lambda_handler \
  --zip-file fileb://lambda_function.zip

# Set environment variables
aws lambda update-function-configuration \
  --function-name EC2InfraControlMonitor \
  --environment Variables="{SNS_TOPIC_ARN=<YOUR_SNS_ARN>,S3_BUCKET=<YOUR_BUCKET>}"
```

### Step 4 — Create EventBridge Rule

```bash
# Create the rule targeting EC2 state changes
aws events put-rule \
  --name "EC2StateChangeRule" \
  --event-pattern '{"source":["aws.ec2"],"detail-type":["EC2 Instance State-change Notification"]}' \
  --state ENABLED

# Add Lambda as the target
aws events put-targets \
  --rule EC2StateChangeRule \
  --targets "Id=1,Arn=arn:aws:lambda:<REGION>:<ACCOUNT_ID>:function:EC2InfraControlMonitor"

# Grant EventBridge permission to invoke Lambda
aws lambda add-permission \
  --function-name EC2InfraControlMonitor \
  --statement-id EventBridgeInvoke \
  --action lambda:InvokeFunction \
  --principal events.amazonaws.com \
  --source-arn arn:aws:events:<REGION>:<ACCOUNT_ID>:rule/EC2StateChangeRule
```

### Step 5 — Run CLI Scripts

```bash
# Countdown-based auto termination
python auto_terminate.py

# Terminate a specific EC2 instance by ID
python terminate_instance.py

# Terminate the most recently launched EC2 instance
python recent_instance.py
```

---

## 🔐 IAM Policy Reference

### Lambda Execution Role (minimum required permissions)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["sns:Publish"],
      "Resource": "arn:aws:sns:<REGION>:<ACCOUNT_ID>:<TOPIC_NAME>"
    },
    {
      "Effect": "Allow",
      "Action": ["s3:PutObject"],
      "Resource": "arn:aws:s3:::<BUCKET_NAME>/*"
    },
    {
      "Effect": "Allow",
      "Action": [
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "arn:aws:logs:*:*:*"
    }
  ]
}
```

### CLI Scripts (IAM user/role)

```json
{
  "Effect": "Allow",
  "Action": [
    "ec2:DescribeInstances",
    "ec2:TerminateInstances"
  ],
  "Resource": "*"
}
```

---

## 📁 Repository Structure

```
AWS-InfraControl-CLI-Tool/
│
├── lambda_function.py                    # Lambda handler — SNS alert + S3 event log writer
├── auto_terminate.py                     # CLI: countdown-based EC2 termination
├── terminate_instance.py                 # CLI: direct termination by instance ID
├── recent_instance.py                    # CLI: terminate most recently launched EC2
│
├── CloudProjectSynopsisPart1.docx        # Academic synopsis — Part 1
├── CloudProjectSynopsisPart2.docx        # Academic synopsis — Part 2
├── ProjectReportAWSInfraControlCLITool.docx  # Full project report
│
├── LICENSE                               # MIT License
└── README.md                             # This file
```

---

## 🔮 Potential Extensions

- **CloudWatch Dashboard** — Visualize EC2 state transition frequency, Lambda error rates, and invocation latency
- **Multi-Region Monitoring** — Extend EventBridge rules across regions using cross-region event buses
- **Slack / Teams Alerts** — Add a webhook Lambda subscriber to SNS for rich channel notifications
- **Auto-Remediation** — Detect unexpected `stopped` states and automatically restart the instance
- **DynamoDB Event Store** — Replace S3 JSON blobs with DynamoDB for queryable, indexed event history
- **Terraform / AWS CDK** — Infrastructure-as-code for fully reproducible, version-controlled deployments
- **Cost Anomaly Alerts** — Integrate AWS Cost Explorer API to trigger alerts when EC2 spend spikes

---

## 👤 Author

<div align="center">

**Nitanshu Tak**

*CS Engineering Student @ UPES · Backend Developer · Cloud & DevOps Builder*

[![GitHub](https://img.shields.io/badge/GitHub-Nitanshu715-181717?style=for-the-badge&logo=github)](https://github.com/Nitanshu715)
[![LinkedIn](https://img.shields.io/badge/LinkedIn-Nitanshu_Tak-0077B5?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/nitanshu-tak-89a1ba289/)
[![Email](https://img.shields.io/badge/Email-nitanshutak070105%40gmail.com-D14836?style=for-the-badge&logo=gmail)](mailto:nitanshutak070105@gmail.com)

</div>

---

## 📄 License

This project is licensed under the **MIT License** — see the [`LICENSE`](./LICENSE) file for details.

---

<div align="center">

*If this helped you understand AWS serverless event-driven architecture, drop a ⭐ — it means a lot!*

</div>
