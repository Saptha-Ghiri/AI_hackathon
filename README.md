# AI-Powered Data Ingestion Pipeline

An n8n workflow that automates data extraction from multiple sources (MySQL, AWS S3), merges and deduplicates data, and uses AI to generate Snowflake SQL commands for data ingestion.

## Overview

This workflow consists of two main sections:

1. **Source Extraction** - Extracts data from MySQL database and AWS S3, merges and deduplicates
2. **AI Data Ingestion** - Uses AI to generate and execute Snowflake SQL commands

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SOURCE EXTRACTION                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   [Chat Trigger] → [Respond to Chat] → [Edit Fields1]                       │
│                                              │                               │
│                                              ↓                               │
│                              [Check Ingestion Control Table]                 │
│                                     (Snowflake15)                            │
│                                              │                               │
│                                              ↓                               │
│                                           [If1]                              │
│                          ┌───────────────────┴───────────────────┐          │
│                          │                                       │          │
│                   [Table Exists]                        [Table Not Exists]  │
│                          │                                       │          │
│                          │                          [Create Table]          │
│                          │                          (Snowflake13)           │
│                          │                                       │          │
│                          │                          [Insert Init Record]    │
│                          │                          (Snowflake14)           │
│                          │                                       │          │
│                          └───────────────────┬───────────────────┘          │
│                                              ↓                               │
│                              [Get Last Processed ID]                        │
│                                     (Snowflake11)                           │
│                                              │                               │
│                          ┌───────────────────┴───────────────────┐          │
│                          │                                       │          │
│                   [MySQL Query]                           [S3 Download]     │
│              (Execute a SQL query1)                    (Download a file1)   │
│                          │                                       │          │
│                   [Convert to CSV]                      [Parse S3 CSV]      │
│              (Code in JavaScript3)                  (Code in JavaScript2)   │
│                          │                                       │          │
│                          └───────────────────┬───────────────────┘          │
│                                              ↓                               │
│                                          [Merge]                            │
│                                              │                               │
│                                              ↓                               │
│                               [Deduplicate & Sort]                          │
│                              (Code in JavaScript4)                          │
│                                              │                               │
│                                              ↓                               │
│                                    [Upload to S3]                           │
│                                    (Upload a file2)                         │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────────┐
│                           AI DATA INGESTION                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   [Upload a file2] → [AI Agent] → [Code: Replace Placeholders]              │
│                           │                    │                             │
│                           │                    ↓                             │
│                    [Azure OpenAI]      [Execute Snowflake SQL]              │
│                    [Snowflake Tool]           (Snowflake9)                  │
│                                                │                             │
│                                                ↓                             │
│                                        [Get Max ID]                         │
│                                        (Snowflake)                          │
│                                                │                             │
│                                                ↓                             │
│                                    [Update Control Table]                   │
│                                        (Snowflake12)                        │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Node Descriptions

### Trigger Nodes

| Node | Type | Description |
|------|------|-------------|
| **When chat message received** | Chat Trigger | Initiates the workflow via chat interface with webhook |
| **Respond to Chat** | Chat Response | Displays welcome message to user |

### Configuration Nodes

| Node | Type | Description |
|------|------|-------------|
| **Edit Fields1** | Set | Configures all workflow parameters (schema, table names, AWS credentials, etc.) |

### Data Source Nodes

| Node | Type | Description |
|------|------|-------------|
| **Execute a SQL query1** | MySQL | Fetches incremental data from MySQL where ID > last_processed_id |
| **Download a file1** | AWS S3 | Downloads existing CSV batch file from S3 bucket |

### Snowflake Nodes

| Node | Purpose |
|------|---------|
| **Snowflake15** | Checks if ingestion control table exists in schema |
| **Snowflake13** | Creates ingestion control table if it doesn't exist |
| **Snowflake14** | Inserts initial record into control table with status 'INIT' |
| **Snowflake11** | Gets last processed ID from control table |
| **Snowflake9** | Executes AI-generated SQL queries for data loading |
| **Snowflake** | Gets max ID from source table after ingestion |
| **Snowflake12** | Updates last processed ID in control table |
| **Snowflake10** | (Available) Queries sales_data table |
| **Snowflake Tool** | AI tool for checking table existence |

### AI Nodes

| Node | Model | Purpose |
|------|-------|---------|
| **AI Agent** | Azure OpenAI (GPT-5-mini) | Generates Snowflake SQL commands based on user input |
| **Azure OpenAI Chat Model1** | GPT-5-mini | Language model for AI Agent |
| **Snowflake Tool** | - | Allows AI Agent to verify table existence in Snowflake |

### Code Nodes

| Node | Purpose |
|------|---------|
| **Code in JavaScript** | Parses AI output and replaces placeholders with actual AWS credentials |
| **Code in JavaScript2** | Converts S3 binary CSV to text format |
| **Code in JavaScript3** | Converts MySQL query results to CSV with proper column ordering |
| **Code in JavaScript4** | Merges S3 and MySQL data, deduplicates, sorts by ID, outputs binary CSV |

### Utility Nodes

| Node | Type | Purpose |
|------|------|---------|
| **If1** | Conditional | Routes flow based on whether ingestion control table exists |
| **Merge** | Merge | Combines MySQL and S3 data streams by position |
| **Upload a file2** | AWS S3 | Uploads merged/deduplicated CSV to S3 external stage |

---

## Workflow Flow

### Complete Data Pipeline

1. **User Interaction**
   - User sends chat message describing the pipeline requirements
   - Bot responds with welcome message asking for pipeline details

2. **Configuration**
   - **Edit Fields1** sets all parameters: schema, table names, bucket, AWS credentials, file names

3. **Ingestion Control Check**
   - **Snowflake15** checks if `INGESTION_CONTROL` table exists
   - **If1** routes based on result:
     - **True**: Proceed to get last processed ID
     - **False**: Create table and insert initial record first

4. **Data Extraction (Parallel)**
   - **MySQL Path**: Query records where ID > last_processed_id → Convert to CSV
   - **S3 Path**: Download existing batch file → Parse CSV from binary

5. **Data Transformation**
   - **Merge**: Combines both data streams
   - **Code in JavaScript4**: Deduplicates rows, sorts by ID, creates binary CSV

6. **Data Loading**
   - **Upload a file2**: Uploads merged CSV to S3 external stage
   - **AI Agent**: Generates Snowflake SQL for loading data from S3 stage
   - **Code in JavaScript**: Replaces placeholders with actual credentials
   - **Snowflake9**: Executes the generated SQL commands

7. **Control Update**
   - **Snowflake**: Gets max ID from loaded data
   - **Snowflake12**: Updates `last_processed_id` in control table

---

## Configuration Parameters

The **Edit Fields1** node configures these values:

| Parameter | Value | Description |
|-----------|-------|-------------|
| `schema` | TEST | Snowflake schema name |
| `ingestion_control_table` | INGESTION_CONTROL | Table tracking processed records |
| `table_name` | sales_data | Target table name |
| `source_table` | sales_data | Source table for queries |
| `bucket` | hackathon-bucket-for-ingestion-tool | S3 bucket name |
| `file_name` | sales_data.csv | Output CSV file name |
| `aws_key` | (configured) | AWS access key ID |
| `aws_secret` | (configured) | AWS secret access key |
| `user_input` | (from chat) | User's pipeline description |

---

## Data Schema

### Ingestion Control Table

```sql
CREATE TABLE INGESTION_CONTROL (
    source_name        VARCHAR(100) PRIMARY KEY,
    last_processed_id  BIGINT,
    last_processed_ts  TIMESTAMP,
    status             VARCHAR(20),
    updated_at         TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Sales Data Columns

| Column | Description |
|--------|-------------|
| ID | Primary identifier |
| YR | Year |
| PD | Period |
| QTR | Quarter |
| CATG_CD | Category code |
| PLAN_TO_NM | Plan name |
| SALES_ORG | Sales organization |
| SALES_RPTG_CHNL_NM | Sales reporting channel name |
| SALES_RPTG_SUBCHNL_NM | Sales reporting sub-channel name |
| SALES_RPTG_RETL_ENV_NM | Sales reporting retail environment name |
| VOL_NKG_QTY | Volume quantity |
| GSV | Gross sales value |
| NSV | Net sales value |
| COGS_DIR_CHRG_VAL | Cost of goods sold direct charge value |
| GROSS_PRFT_VAL | Gross profit value |
| NSV_USD | Net sales value in USD |
| GROSS_PRFT_VAL_USD | Gross profit value in USD |
| USD_RATE | USD exchange rate |
| SALES_REF_KEY | Sales reference key |

---

## AI Agent Configuration

### System Prompt
The AI Agent is configured as a Snowflake SQL generator with these rules:
- Generate ONLY Snowflake SQL
- No stored procedures (no $$, BEGIN, END, RETURN)
- Each SQL statement must be executable independently
- Return ONLY a valid JSON array of SQL strings
- No explanations or markdown
- Use placeholders for sensitive values:
  - `<bucket>` - S3 bucket name
  - `<awsKey>` - AWS access key
  - `<awsSecret>` - AWS secret key
  - `<fileName>` - CSV file name

### Tools Available
- **Snowflake Tool**: Checks if target table exists in the schema

---

## Requirements

- n8n instance (self-hosted or cloud)
- Snowflake account with appropriate permissions
- MySQL database with source data
- AWS S3 bucket for file staging
- Azure OpenAI API access (GPT-5-mini model deployment)

---

## Usage

1. Import the `workflow.json` into n8n
2. Configure all credentials (see CREDENTIALS.md)
3. Update **Edit Fields1** with your specific values
4. Activate the workflow
5. Send a chat message describing your pipeline requirements
6. The AI will generate and execute the appropriate Snowflake SQL

### Example Chat Input
```
Load sales data from S3 external stage into the sales_data table.
Create the table if it doesn't exist with appropriate columns for sales metrics.
```

---

## Google Looker Studio Integration

After the complete execution of the n8n workflow, the data is available in Snowflake and can be visualized using Google Looker Studio.

### Dashboard Connection

The Snowflake database is connected to Google Looker Studio for real-time data visualization. Once the workflow completes:

1. **Data Flow**: n8n workflow → Snowflake → Google Looker Studio
2. **Refresh**: Simply refresh the Looker Studio dashboard to see the newly populated data
3. **Real-time Updates**: Each workflow execution updates the Snowflake tables, which are reflected in the dashboard upon refresh

### Viewing Updated Data

1. Open your Google Looker Studio dashboard
2. Click the **Refresh** button (or press F5)
3. The dashboard will display the latest data from Snowflake

```
┌─────────────────┐      ┌─────────────────┐      ┌─────────────────────┐
│   n8n Workflow  │  →   │    Snowflake    │  →   │  Google Looker      │
│   (Ingestion)   │      │   (Data Store)  │      │  Studio (Dashboard) │
└─────────────────┘      └─────────────────┘      └─────────────────────┘
```

---

## File Structure

```
AI_hackathon/
├── workflow.json      # n8n workflow definition
├── README.md          # This documentation
└── CREDENTIALS.md     # Credentials configuration guide
```

---
