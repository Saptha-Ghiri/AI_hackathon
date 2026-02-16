# Credentials Configuration Guide

This document lists all credentials required for the n8n workflow nodes.

---

## Required Credentials Overview

| Credential Type | Credential Name | Credential ID | Used By |
|-----------------|-----------------|---------------|---------|
| Snowflake | Snowflake account | HTeej5ZUAp23QMLD | 8 nodes |
| MySQL | MySQL account | 667tlu1AYvBIPY0y | 1 node |
| AWS | AWS (IAM) account | cj1zXCTQCe1AEZ7v | 2 nodes |
| Azure OpenAI | Azure Open AI account | yqJFYU4DJLECmZNj | 1 node |

---

## 1. Snowflake Credentials

**Credential ID:** `HTeej5ZUAp23QMLD`
**Credential Name:** `Snowflake account`

### Nodes Using This Credential

| Node Name | Purpose |
|-----------|---------|
| Snowflake9 | Execute AI-generated SQL queries |
| Snowflake10 | Query sales_data table |
| Snowflake11 | Get last processed ID |
| Snowflake12 | Update control table |
| Snowflake13 | Create ingestion control table |
| Snowflake14 | Insert initial control record |
| Snowflake15 | Check if control table exists |
| Snowflake | Get max ID from source table |
| Snowflake Tool | AI tool for table existence check |

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| Account | Snowflake account identifier | `xy12345.us-east-1` |
| Username | Snowflake username | `your_username` |
| Password | Snowflake password | `your_password` |
| Database | Default database | `YOUR_DATABASE` |
| Schema | Default schema | `TEST` |
| Warehouse | Compute warehouse | `COMPUTE_WH` |
| Role | User role (optional) | `ACCOUNTADMIN` |

### Setup Instructions

1. Go to n8n **Credentials** section
2. Click **Add Credential** → Select **Snowflake**
3. Enter your Snowflake account details:
   - Account: Your Snowflake account locator (e.g., `abc12345.us-east-1`)
   - Username: Your Snowflake username
   - Password: Your Snowflake password
   - Database: The database to connect to
   - Schema: Default schema (TEST)
   - Warehouse: Compute warehouse name
4. Test the connection
5. Save the credential

---

## 2. MySQL Credentials

**Credential ID:** `667tlu1AYvBIPY0y`
**Credential Name:** `MySQL account`

### Nodes Using This Credential

| Node Name | Purpose |
|-----------|---------|
| Execute a SQL query1 | Fetch incremental data from MySQL |

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| Host | MySQL server hostname | `localhost` or `mysql.example.com` |
| Port | MySQL port | `3306` |
| Database | Database name | `your_database` |
| User | MySQL username | `your_username` |
| Password | MySQL password | `your_password` |
| SSL | Enable SSL connection | `false` |

### Setup Instructions

1. Go to n8n **Credentials** section
2. Click **Add Credential** → Select **MySQL**
3. Enter your MySQL connection details
4. Test the connection
5. Save the credential

---

## 3. AWS Credentials

**Credential ID:** `cj1zXCTQCe1AEZ7v`
**Credential Name:** `AWS (IAM) account`

### Nodes Using This Credential

| Node Name | Purpose |
|-----------|---------|
| Download a file1 | Download CSV from S3 bucket |
| Upload a file2 | Upload merged CSV to S3 bucket |

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| Region | AWS region | `us-east-1` |
| Access Key ID | IAM access key | `AKIA...` |
| Secret Access Key | IAM secret key | `wJal...` |

### IAM Permissions Required

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:GetObject",
                "s3:PutObject",
                "s3:ListBucket"
            ],
            "Resource": [
                "arn:aws:s3:::hackathon-bucket-for-ingestion-tool",
                "arn:aws:s3:::hackathon-bucket-for-ingestion-tool/*"
            ]
        }
    ]
}
```

### Setup Instructions

1. Go to AWS Console → IAM → Users
2. Create or select a user with programmatic access
3. Attach the S3 policy above (or AmazonS3FullAccess for testing)
4. Copy the Access Key ID and Secret Access Key
5. In n8n, go to **Credentials** section
6. Click **Add Credential** → Select **AWS**
7. Enter your IAM credentials
8. Test the connection
9. Save the credential

---

## 4. Azure OpenAI Credentials

**Credential ID:** `yqJFYU4DJLECmZNj`
**Credential Name:** `Azure Open AI account`

### Nodes Using This Credential

| Node Name | Purpose |
|-----------|---------|
| Azure OpenAI Chat Model1 | Language model for AI Agent |

### Required Fields

| Field | Description | Example |
|-------|-------------|---------|
| API Key | Azure OpenAI API key | `abc123...` |
| Resource Name | Azure resource name | `your-resource-name` |
| API Version | API version | `2024-02-15-preview` |

### Model Configuration

The workflow uses `gpt-5-mini` as the model deployment name. Ensure this matches your Azure OpenAI deployment.

### Setup Instructions

1. Go to Azure Portal → Azure OpenAI resource
2. Navigate to **Keys and Endpoint**
3. Copy the API key (Key 1 or Key 2)
4. Note the endpoint URL to extract the resource name
5. In n8n, go to **Credentials** section
6. Click **Add Credential** → Select **Azure OpenAI API**
7. Enter:
   - API Key: Your Azure OpenAI key
   - Resource Name: From your endpoint URL
   - API Version: `2024-02-15-preview` (or latest)
8. Test the connection
9. Save the credential

---

## Workflow Configuration Parameters

In addition to credentials, the **Edit Fields1** node contains these configurable values:

| Parameter | Current Value | Description |
|-----------|---------------|-------------|
| `schema` | `TEST` | Snowflake schema |
| `ingestion_control_table` | `INGESTION_CONTROL` | Control table name |
| `table_name` | `sales_data` | Target table name |
| `source_table` | `sales_data` | Source table for queries |
| `bucket` | `hackathon-bucket-for-ingestion-tool` | S3 bucket name |
| `file_name` | `sales_data.csv` | Output CSV file name |
| `aws_key` | `AKIAW42RQYAAMWQJZG2K` | AWS access key (replace with yours) |
| `aws_secret` | `Zssa9C/777CIacG2LAUYB+egMivm6TFxkW0NLVsi` | AWS secret (replace with yours) |
| `user_input` | `{{ $json.chatInput }}` | User's chat input |

---

## S3 Bucket Configuration

| Setting | Value |
|---------|-------|
| Bucket Name | `hackathon-bucket-for-ingestion-tool` |
| Input File | `sales_data_batch_1.csv` |
| Output File | `sales_data.csv` (configurable via file_name) |

---

## Security Recommendations

1. **Never commit credentials** to version control
2. Use **environment variables** in production:
   ```
   AWS_ACCESS_KEY_ID=your_key
   AWS_SECRET_ACCESS_KEY=your_secret
   ```
3. **Rotate credentials** regularly (every 90 days recommended)
4. Apply **least privilege** principle for IAM roles
5. Enable **SSL/TLS** for database connections
6. Use **Snowflake key-pair authentication** for enhanced security
7. Store sensitive values in n8n's **credential store**, not in workflow JSON
8. Consider using **AWS Secrets Manager** or **Azure Key Vault** for credential management

---

## Troubleshooting

### Snowflake Connection Issues
- Verify account identifier format (account.region)
- Check warehouse is running and accessible
- Ensure role has required permissions

### MySQL Connection Issues
- Verify host is accessible from n8n server
- Check firewall allows port 3306
- Verify user has SELECT permissions on source table

### AWS S3 Issues
- Verify bucket exists and is in correct region
- Check IAM user has s3:GetObject and s3:PutObject permissions
- Ensure bucket policy allows access

### Azure OpenAI Issues
- Verify deployment name matches (gpt-5-mini)
- Check API key is valid and not expired
- Ensure model deployment is active

---

## Credential Setup Checklist

- [ ] Snowflake account configured and tested
- [ ] MySQL account configured and tested
- [ ] AWS IAM credentials configured with S3 permissions
- [ ] Azure OpenAI API configured with GPT model deployment
- [ ] Edit Fields1 parameters updated with your values
- [ ] S3 bucket created and accessible
- [ ] Test workflow execution successful
