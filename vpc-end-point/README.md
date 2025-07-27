## 🧪 AWS VPC Endpoint Lab:
## Objective:

Enable EC2 instances in a private subnet to securely and privately access AWS services using:

Gateway Endpoint for Amazon S3

Interface Endpoint for AWS Secrets Manager

## 📐 Architecture Overview:

<img width="1135" height="680" alt="Image" src="https://github.com/user-attachments/assets/c76d177d-86ec-490f-9576-dfc76b4f7189" />

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

## Before we have to copy the pem file to private server you have to give their permissions to pem file by using command called:
```
sudo chmod 400 pem.file
```

## We need to copy the file to the server securely by using command called:
```
scp - i pem.file pem.file ubuntu@10.0.0.21:~
```

## Screenshot Images:

## Cloudformation:

<img width="1893" height="922" alt="Image" src="https://github.com/user-attachments/assets/bbcba8d2-7556-44a0-bb65-7c061577c51e" />

## VPC Endpoint:

<img width="1912" height="902" alt="Image" src="https://github.com/user-attachments/assets/2f10e92b-ca58-4365-85e1-2b289ae1ef25" />

## VPC Endpoint-pub:

<img width="1917" height="916" alt="Image" src="https://github.com/user-attachments/assets/7e16b300-5a0f-4377-b75e-508618db7a02" />

## VPC Endpoint-pvt:

<img width="1908" height="913" alt="Image" src="https://github.com/user-attachments/assets/51928071-4e11-49ff-85bf-f4154eefbb45" />

## VPC Endpoint-pub-rt:

<img width="1913" height="922" alt="Image" src="https://github.com/user-attachments/assets/148d1234-1780-44db-a892-1cfd1c79af21" />

## VPC Endpoint-pvt-rt:

<img width="1918" height="917" alt="Image" src="https://github.com/user-attachments/assets/1e6da0f2-1d69-46c4-b55e-c0b8cc957fd2" />

## VPC Endpoint-igw:

<img width="1912" height="912" alt="Image" src="https://github.com/user-attachments/assets/8cc00088-f9ff-4b60-9be7-c11d8b67ce06" />

## VPC Endpoints:

<img width="1917" height="917" alt="Image" src="https://github.com/user-attachments/assets/136f7ed1-ade2-427a-ab2b-d1093f925576" />

## VPC Endpoint-route-tables:

<img width="1912" height="917" alt="Image" src="https://github.com/user-attachments/assets/498d8656-559b-42a0-aea5-8bdc19ada7aa" />

## EC2A-Pub:

<img width="1900" height="916" alt="Image" src="https://github.com/user-attachments/assets/58b5960d-ccdd-4eda-bf19-c2335354e1c8" />

## EC2B-Pvt:

<img width="1892" height="927" alt="Image" src="https://github.com/user-attachments/assets/eec593ec-b960-4c6a-bc47-36663fa1089c" />

## EC2B-Pvt-IAM:

<img width="1893" height="926" alt="Image" src="https://github.com/user-attachments/assets/694d9545-7173-4fda-8695-723c7ab60601" />

## Copy the pem file:

<img width="1102" height="380" alt="Image" src="https://github.com/user-attachments/assets/04a36387-9826-4bda-b50d-c1c6b297754a" />

## copied the pem file:

<img width="735" height="328" alt="Image" src="https://github.com/user-attachments/assets/cefaf368-1359-4a08-a160-4c367f22a8db" />

## EC2A-Pub S3:

<img width="832" height="255" alt="Image" src="https://github.com/user-attachments/assets/1cc35644-1424-465e-97ce-0da99a16a193" />

## pem file chmod permissions:

<img width="1161" height="795" alt="Image" src="https://github.com/user-attachments/assets/b901a2a8-418a-48c0-96d9-7147ac2c9785" />

## EC2B-Pvt-S3:

<img width="788" height="261" alt="Image" src="https://github.com/user-attachments/assets/9ea55988-e571-4ab8-bdf0-d4aff7b001e0" />

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


## Author:

**Uday Sairam Kommineni**

**AWS Cloud Practioner**

Mail-id: saikommineni5@gmail.com

Linkedin: https://www.linkedin.com/in/uday-sai-ram-kommineni-uday-sai-ram/
