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


