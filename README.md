# AWS Automated Access Review

AI-powered cloud GRC automation using Amazon Bedrock (Claude 3 Haiku)

---

## About this project

This project demonstrates how Governance, Risk, and Compliance (GRC) engineering can be automated and scaled across AWS environments.

The AWS Access Review tool uses Amazon Bedrock’s Claude 3 Haiku model to translate complex technical findings into governance-aligned narratives making it easier for security, compliance, and leadership teams to understand cloud risks and make informed decisions.

It bridges the gap between security operations and compliance management by producing:

- Structured evidence of control performance
- AI-generated summaries mapped to frameworks like NIST CSF, SOC 2, and ISO 27001
- Executive-ready risk narratives for audit reporting and board communication

---

## Overview

| Area                      | Description                                                          |
| ------------------------- | -------------------------------------------------------------------- |
| **Purpose**               | Automate AWS access reviews for GRC and compliance readiness         |
| **Focus**                 | Translate security findings into governance-aligned insights         |
| **AI Engine**             | Amazon Bedrock: Claude 3 Haiku (Messages API)                        |
| **Key Outcome**           | CSV of technical evidence + AI-written compliance narrative          |
| **Compliance Frameworks** | NIST CSF, SOC 2, ISO 27001, PCI DSS                                  |
| **Tech Stack**            | Python • AWS Lambda • Bedrock • S3 • SES • CloudTrail • Security Hub |

---

## Logic Flowchart

![flowchart](./assets/automated-access-review.png)

---

## Setup, deployment and report output

Follow the [Quick Start Guide](https://github.com/ajy0127/aws_automated_access_review#quick-start-guide) to set up your Python virtual environment, configure your AWS credentials and run the deployment script.

#### Examples of output after deploying script:

<details> <summary> <strong>Sample AI-generated output (sent via email)</strong> (click to expand)</summary>

![report](./assets/ai-report.png)

</details>

<details> <summary> <strong>Sample CSV output (also sent via email)</strong> (click to expand)</summary>

![CSVreport](./assets/csv-report.png)

</details>

---

## How it works

1. You deploy the project using the CloudFormation template.
2. AWS creates all the needed infrastructure for you.
3. The Lambda function runs on a schedule.
4. It collects data, generates a report, uses AI to summarize, uploads to S3 and emails the result.
5. The report is emailed to stakeholders via SES.

All of this is fully automated and there's no need to log into the console once it's deployed.

---

## The core building blocks 

1. **AWS CloudFormation**: sets up the infrastructure (IaC).
2. **AWS Lambda**: runs the logic (serverless code execution).
3. **Amazon Bedrock**: generates human-readable AI summaries.

#### 1. AWS CloudFormation (IaC)

Instead of manually creating AWS resources, CloudFormation defines everything in the YAML template named `access-review-real.yaml`. The template is found [here](https://github.com/ajy0127/aws_automated_access_review/tree/main/templates).

CloudFormation automatically sets up the:
- Lambda function
- IAM roles & permissions
- S3 bucket (for CSV reports)
- SES configuration (for emails)
- CloudWatch scheduled trigger (runs every 30 days)

Benefit of CloudFormation: repeatable deployments with no manual setup.

#### 2. AWS Lambda (how the code runs)

The [`index.py`](https://github.com/ajy0127/aws_automated_access_review/tree/main/src/lambda) file is where the Lambda function logic lives. This file contains the entry-point function: `def handler(event, context):`

Every time the Lambda runs, it:
1. collects security data from IAM, CloudTrail, and Security Hub.
2. generates a CSV report.
3. uploads report to S3.
4. calls Amazon Bedrock for AI summary.
5. sends email via SES.

If you're new to Lambda, read `index.py` from top to bottom, then follow how it imports and calls "**helper**" [`/modules`](https://github.com/ajy0127/aws_automated_access_review/tree/main/src/lambda/modules). Each module handles one specific task.

#### 3. Amazon Bedrock (AI-generated summary)

The AI summary process is handled by the [`bedrock_integration.py`](https://github.com/ajy0127/aws_automated_access_review/tree/main/src/lambda) module and connects to Claude 3 Haiku and guides the model to think like a compliance analyst.

Core Functions:
`prepare_prompt()`: Converts technical findings into GRC-ready evidence summaries
`invoke_claude_model()`: Sends structured prompt to Amazon Bedrock
`extract_narrative_claude()`: Extracts AI-written compliance narrative
`generate_fallback_narrative()`: Provides a compliant fallback when AI is unavailable

This makes reports useful for non-technical readers or executives who need high-level insights, not raw CSVs.

---

## Key challenges and fixes

These are the real-world issues I faced and how I solved them:

#### 1. Bash scripts don’t work in PowerShell.
Bash commands like `./scripts/deploy.sh` won’t work in PowerShell and will throw an error.
> **Fix**: I installed [`Git Bash`](https://git-scm.com/download/win) on Windows and ran code in the Git Bash terminal instead of using PowerShell.

#### 2. Bedrock model mismatch.
- The original code used an older model ID (claude-v2) which failed silently, and the email output was the generic fallback narrative outlined in the [code](https://github.com/ajy0127/aws_automated_access_review/blob/main/src/lambda/bedrock_integration.py).
- **Fix**: I updated the model ID in `bedrock_integration.py` to Claude 3 Haiku model.

#### 3. Lambda logging can be misleading sometimes.
- Lambda logs initially showed “IAM Access Analyzer findings collected” even when no findings were returned due to missing analyzer configuration. This required careful reading of both CloudWatch logs and the generated report to discover the inconsistency.
- **Fix**: I manually created an external access analyzer.

#### 4. CloudTrail returned incomplete findings.
- Only write operations were tracked which led to incomplete findings from the CloudTrail module.
- **Fix**: I reconfigured CloudTrail to log all management events for more complete analysis.

With these fixes, my deployment succeeded: reports generated, uploaded to S3, summarized by Bedrock, and emailed via SES.

---

## Compliance Context

| Framework | Relevant Domain                                  |
| --------- | ------------------------------------------------ |
| NIST CSF  | PR.AC (Access Control), DE.CM (Monitoring)       |
| SOC 2     | Security & Confidentiality                       |
| ISO 27001 | A.9 (Access Control), A.12 (Operations Security) |

---

## Governance & Security

- Evidence stored in S3 with encryption at rest
- Reports aligned to NIST CSF and SOC 2 control families
- IAM permissions follow the principle of least privilege
- AI output audited and traceable for assurance purposes

---

## Skills Demonstrated

- GRC Engineering: Automated control validation and compliance reporting
- Framework Mapping: NIST CSF, SOC 2, ISO 27001 alignment
- AI-Augmented Governance: Use of Bedrock to interpret compliance evidence
- Automation & Evidence Management: Continuous compliance via AWS services
- Executive Communication: Clear risk summaries for non-technical stakeholders

---

## Resources

Project repo: [Automated Access Review](https://github.com/ajy0127/aws_automated_access_review).

---
