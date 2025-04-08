# Amazon Route 53: DNS Management with AWS CLI

This project demonstrates how to configure and manage DNS records using **Amazon Route 53** via the **AWS CLI**. It covers the core features, advantages, and practical examples such as setting up a hosted zone, adding A and CNAME records, and implementing failover routing.

---

## 🚀 What is Amazon Route 53?

Amazon Route 53 is a **highly available and scalable Domain Name System (DNS) web service** by AWS. It is used for:

- Registering domains
- Managing DNS records
- Routing internet traffic based on health checks, latency, geography, etc.
- Monitoring the health of resources

---

## ✅ Why Route 53?

- **Highly Available & Scalable**: Built on AWS's global infrastructure.
- **Flexible Routing**: Supports simple, weighted, latency-based, geo, and failover routing policies.
- **Health Checks & Failover**: Automatically reroutes traffic if endpoints fail.
- **Integration with AWS services**: EC2, ELB, S3, CloudFront, etc.
- **DNS + Domain Management**: All in one place.

---

## 🛠️ Prerequisites

- AWS CLI installed and configured:
  ```bash
  aws configure
  ```
- A registered domain name (e.g., `example.com`)
- Sufficient AWS permissions to manage Route 53

---

## 📦 1. Create a Hosted Zone

```bash
aws route53 create-hosted-zone \
  --name example.com \
  --caller-reference $(date +%s) \
  --hosted-zone-config Comment="Public hosted zone",PrivateZone=false
```

Save the `HostedZoneId` from the response. It will be used in the next steps.

---

## 🔧 2. Create an A Record

Create a file named `record-set.json`:

```json
{
  "Comment": "Creating A record for example.com",
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "example.com.",
        "Type": "A",
        "TTL": 300,
        "ResourceRecords": [
          { "Value": "192.0.2.123" }
        ]
      }
    }
  ]
}
```

Then apply it:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id ZONE_ID_HERE \
  --change-batch file://record-set.json
```

---

## 🌐 3. Create a CNAME Record for `www`

Modify `record-set.json`:

```json
{
  "Comment": "Creating CNAME for www.example.com",
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "www.example.com.",
        "Type": "CNAME",
        "TTL": 300,
        "ResourceRecords": [
          { "Value": "example.com." }
        ]
      }
    }
  ]
}
```

And apply it the same way.

---

## ⚠️ 4. Failover Routing Example

### Step 1: Create a health check

```bash
aws route53 create-health-check \
  --caller-reference $(date +%s) \
  --health-check-config '{
    "IPAddress": "192.0.2.1",
    "Port": 80,
    "Type": "HTTP",
    "ResourcePath": "/",
    "RequestInterval": 30,
    "FailureThreshold": 3
  }'
```

Copy the `HealthCheckId` from the response.

### Step 2: Create primary and secondary records

```json
{
  "Comment": "Failover routing for example.com",
  "Changes": [
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "example.com.",
        "Type": "A",
        "SetIdentifier": "primary",
        "Failover": "PRIMARY",
        "TTL": 30,
        "HealthCheckId": "your-health-check-id",
        "ResourceRecords": [
          { "Value": "192.0.2.1" }
        ]
      }
    },
    {
      "Action": "CREATE",
      "ResourceRecordSet": {
        "Name": "example.com.",
        "Type": "A",
        "SetIdentifier": "secondary",
        "Failover": "SECONDARY",
        "TTL": 30,
        "ResourceRecords": [
          { "Value": "192.0.2.2" }
        ]
      }
    }
  ]
}
```

Apply the changes:

```bash
aws route53 change-resource-record-sets \
  --hosted-zone-id ZONE_ID_HERE \
  --change-batch file://failover-records.json
```

---

## 📘 Learn More

- [Amazon Route 53 Developer Guide](https://docs.aws.amazon.com/route53/)
- [AWS CLI Reference for Route 53](https://docs.aws.amazon.com/cli/latest/reference/route53/)
- [Route 53 Pricing](https://aws.amazon.com/route53/pricing/)

---

## 📢 Author

This project is a demonstration of my understanding of Amazon Route 53 DNS service and its real-world applications using AWS CLI.  
Feel free to fork, use, or contribute.