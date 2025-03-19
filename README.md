# telecom-data-pipeline-aws
A serverless ETL pipeline for ingesting and processing telecom customer subscriptions using AWS Glue, S3, and RDS

# AWS Glue Incremental ETL Pipeline

This repository showcases a **simple AWS Glue ETL** process that ingests daily-partitioned data from an upstream source in **Amazon S3**, transforms it incrementally, and loads the cleaned data into an **Amazon RDS** instance. It leverages **AWS Glue Crawlers** to track evolving schemas, **Amazon CloudWatch** for automated triggers, and **IAM + VPC** configurations for secure operations.

---

## Table of Contents
1. [Overview](#overview)  
2. [Architecture Diagram](#architecture-diagram)  
3. [Key AWS Components](#key-aws-components)  
4. [Setup Instructions](#setup-instructions)  
5. [Script Explanation](#script-explanation)  
6. [Incremental Loading & Job Bookmarking](#incremental-loading--job-bookmarking)  
7. [Security Considerations](#security-considerations)  
8. [Further Enhancements](#further-enhancements)  
9. [License](#license)

---

## Overview
- **Purpose**: Demonstrate an **incremental ETL pipeline** using AWS Glue to process new daily data only—reducing compute overhead and processing times.  
- **Core Idea**: An upstream team drops date-partitioned files into S3. AWS Glue automatically detects the new partitions, triggers a job via CloudWatch, transforms the data (filtering, schema mapping, PII redaction, ID generation), then writes everything into an RDS database.  
- **Who’s This For?** Anyone new to AWS Glue wanting a simple template for incremental ETL jobs.

---

## Architecture Diagram

    Upstream Source
         |
         v (daily .csv/.parquet)
  +------------------+
  |   S3 Landing     | (partitioned by date)
  +------------------+
         |
(Crawler #1)        +------------------------+
         v          |     CloudWatch Event   |
  +------------------+------------------------+
  |  AWS Glue Job: Incremental Transformations
  |  - Filter data
  |  - Map schema
  |  - Generate IDs
  |  - Redact PII
  +------------------+
         |
   (Transformed Data)
         |
       RDS Table
         ^
    (Crawler #2)




1. **S3 Landing Zone**: Receives fresh, date-partitioned data from the upstream team.  
2. **Crawler #1**: Monitors the S3 bucket to update the Glue Data Catalog with any new partitions or schema changes.  
3. **CloudWatch Event**: When new data arrives, it triggers the Glue ETL job automatically.  
4. **AWS Glue Job**: Reads the new data, applies transformations (filter, schema mapping, ID generation, PII redaction), then writes the result to RDS.  
5. **Crawler #2**: Monitors the RDS database to keep its schema updated in the Glue Data Catalog.

---

## Key AWS Components
- **Amazon S3**: Stores incoming data, partitioned by date.  
- **AWS Glue**: Orchestrates the ETL (crawlers and job).  
- **AWS Glue Crawlers**: Dynamically discover schemas in both S3 and RDS.  
- **Amazon CloudWatch**: Triggers the Glue job upon new file uploads to S3.  
- **Amazon RDS**: Destination for cleaned, transformed data.  
- **IAM & VPC**: Ensures the job has proper network access and security privileges.

---

## Setup Instructions

1. **Create/Identify S3 Bucket**  
   - Store incoming data in a partitioned format, e.g., `s3://my-bucket/dt=YYYY-MM-DD/`.

2. **Glue Crawlers**  
   - **Crawler #1** (for S3): Points to your S3 partitions.  
   - **Crawler #2** (for RDS): Monitors your database’s schema and tables.

3. **IAM Roles**  
   - Create or configure an **IAM role** for Glue that allows reading from S3, writing to RDS, and updating the Glue Data Catalog.  
   - Apply the **least-privilege** principle.

4. **CloudWatch Rule**  
   - Set up a rule to detect new object creation in your S3 bucket.  
   - The target of this rule = your AWS Glue job.

5. **Glue Job**  
   - Either create a new job in AWS Glue Studio (script or visual mode) or upload your custom PySpark script.  
   - Ensure your job can access your RDS instance (use a **Glue Connection** if needed, configured with the right **VPC** and subnets).

6. **Run or Schedule**  
   - You can manually run the job or rely on CloudWatch events to trigger it as soon as new data arrives in the S3 bucket.

---

## Script Explanation
Inside your PySpark script (e.g., [**etl_script.py**](./etl_script.py)):
- **Read** from S3 using a `create_dynamic_frame.from_options()` or `.from_catalog()` call.  
- **Apply** transformations (filtering, mapping, ID generation, PII redaction, etc.).  
- **Write** to RDS via `write_dynamic_frame.from_jdbc_conf()`, specifying the table name and connection.

> **Note**: Pasting a custom script into the Glue console doesn’t always produce a full drag-and-drop visual workflow in Glue Studio.

---

## Incremental Loading & Job Bookmarking
- **Date Partitioning**: By organizing data in `dt=YYYY-MM-DD` folders, you only process new files each day.  
- **Glue Job Bookmarking** (optional): When enabled, Glue can remember which partitions or files it has already processed, making incremental loads even smoother.

---

## Security Considerations
- **IAM Policies**: Confirm your Glue job’s role can read from the specific S3 paths and write to the RDS table.  
- **VPC Configuration**: If your RDS is in a private subnet, attach the Glue job to the same VPC and security group.  
- **Data Encryption**: Consider server-side encryption on S3 and SSL connections for RDS.

---

## Further Enhancements
- **AWS Lake Formation**: For advanced governance and fine-grained data access controls.  
- **Pushdown Predicates**: Filter data at the source to optimize performance.  
- **Dev Endpoints**: For interactive PySpark debugging and testing.  
- **Error Handling**: Set up robust logging and recovery in case of partial loads or schema mismatches.

**Got Questions or Suggestions?**  
Open an issue or start a discussion if you have any ideas, concerns, or feedback. Let’s keep learning together!


