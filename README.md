## 🧪 AWS VPC Endpoint Lab:
## Objective:

Enable EC2 instances in a private subnet to securely and privately access AWS services using:

Gateway Endpoint for Amazon S3

Interface Endpoint for AWS Secrets Manager

## 📐 Architecture Overview
vbnet
Copy
Edit
                   ┌───────────────────────────────┐
                   │           VPC                 │
                   │         10.0.0.0/16           │
                   └───────┬──────────┬────────────┘
                           │          │
        ┌──────────────────┘          └──────────────────┐
        │                                            │
┌──────────────────┐                    ┌────────────────────┐
│ Public Subnet    │                    │ Private Subnet     │
│ 10.0.1.0/24      │                    │ 10.0.2.0/24        │
│ (EC2‑A :Bastion) │                    │ (EC2‑B, no public IP)│
└──────────────────┘                    └─────────┬──────────┘
                                                  │
                                 ┌────────────────────────────────┐
                                 │ Gateway Endpoint (S3)         │
                                 └────────────────────────────────┘
                                 ┌──────────────────────────────────┐
                                 │ Interface Endpoint (SecretsMgr) │
                                 └──────────────────────────────────┘

## 🧱 Part 1 — VPC & Subnet Setup:
VPC

Name: VPC-Endpoint-Lab

CIDR: 10.0.0.0/16

Subnets

Subnet	CIDR	Type
PublicSubnet	10.0.1.0/24	Public
PrivateSubnet	10.0.2.0/24	Private

Internet Gateway

Name: IGW-Lab

Attach to VPC-Endpoint-Lab

Route Tables

Public Route Table:

Route: 0.0.0.0/0 → IGW-Lab

Associated Subnet: PublicSubnet

Private Route Table:

No 0.0.0.0/0 route

Associated Subnet: PrivateSubnet

## 🛠️ Part 2 — EC2 Instances Setup & Verify Connectivity
EC2-A (Bastion Host)

Subnet: PublicSubnet

AMI: Amazon Linux 2

Public IP: Enabled

SG: Allow inbound SSH (port 22) from your IP

EC2-B (Private Instance)

Subnet: PrivateSubnet

AMI: Amazon Linux 2

No public IP

SG:

SSH from EC2-A

HTTPS (443) inbound

SSH Flow

# From local → Bastion (EC2-A):
```
ssh -i key.pem ec2-user@<EC2-A-Public-IP>
```

# From Bastion → Private EC2-B:
```
ssh -i key.pem ec2-user@<EC2-B-Private-IP>
```
☁️ Part 3 — Gateway Endpoint for S3
Create S3 Bucket

```
aws s3 mb s3://vpc-endpoint-lab-<your-name>
Test Access from EC2-B (should fail pre‑endpoint)
```

```
aws s3 ls
Create Gateway Endpoint

Console: VPC → Endpoints → Create Endpoint

Name: S3GatewayEndpoint

Service: com.amazonaws.<region>.s3

Type: Gateway

Select: VPC-Endpoint-Lab, Private Route Table

Test again from EC2-B
```

```
echo "test" > test.txt
aws s3 cp test.txt s3://vpc-endpoint-lab-<your-name>/
```
✅ Should succeed via gateway endpoint, without internet access.

## 🔐 Part 4 — Interface Endpoint for Secrets Manager:

Create a Test Secret

Console: AWS Secrets Manager → Store new secret

Key/Value: username: admin

Name: MyTestSecret

Create Interface Endpoint

Console: VPC → Endpoints → Create Endpoint

Name: SecretsManagerEndpoint

Service: com.amazonaws.<region>.secretsmanager

Type: Interface

Subnet: PrivateSubnet

Enable Private DNS

SG: Allow HTTPS (port 443) from EC2-B

Test from EC2-B

```
aws secretsmanager get-secret-value --secret-id MyTestSecret
```
✅ Should succeed via interface endpoint, using private DNS/IP.

## Screenshot Images:


## 🧹 Cleanup (Optional):
Terminate EC2-A, EC2-B

Delete S3GatewayEndpoint, SecretsManagerEndpoint

Delete S3 bucket: aws s3 rb s3://vpc-endpoint-lab-<your-name> --force

Delete Secrets Manager secret

Delete VPC VPC‑Endpoint‑Lab (subnets, IGW, etc.)

## 📋 Summary Table:

| **Component**       | **Endpoint Type** | **Traffic Pattern**    | **Notes**                                                                            |
| ------------------- | ----------------- | ---------------------- | ------------------------------------------------------------------------------------ |
| Amazon S3           | Gateway           | Routed via route table | Uses AWS-managed prefix list; no cost; supports only IPv4 ([docs.aws.amazon.com][1]) |
| AWS Secrets Manager | Interface         | ENI + Private DNS      | Each endpoint creates ENIs in specified subnets; resolves via private DNS name       |

[1]: https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html?utm_source=chatgpt.com "Gateway endpoints for Amazon S3 - Amazon Virtual Private Cloud"

