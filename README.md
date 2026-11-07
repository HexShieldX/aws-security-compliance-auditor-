# AWS Security & Compliance Auditor

**A Python-based, multi-region and multi-account AWS security auditing tool** that identifies common cloud security misconfigurations, evaluates security posture, maps findings to compliance frameworks, and generates detailed HTML, JSON, and PDF evidence reports.

The tool audits five major AWS security areas:

- **IAM:** Users, access keys, password policies, and MFA configurations.
- **Amazon S3:** Bucket access controls, public exposure, encryption, and security configurations.
- **Amazon EC2:** Security groups and potentially dangerous network access rules.
- **AWS CloudTrail:** Logging configuration and audit trail coverage.
- **Amazon GuardDuty:** Threat detection service configuration and availability.

It uses a deterministic weighted risk-scoring model to prioritize findings and maps security checks to **ISO 27001, SOC 2, and NIST Cybersecurity Framework (CSF)** controls.

---

## Key Features

### 1. Multi-Region AWS Scanning
- Automatically discovers all AWS regions enabled for the account.
- Scans region-specific services across discovered regions.
- Supports manually specifying regions.
- Avoids limiting security assessments to a single AWS region.

### 2. Multi-Account Scanning
The desktop GUI supports scanning multiple AWS accounts using locally configured AWS profiles.

- Automatically discovers profiles from `~/.aws/config` and `~/.aws/credentials`.
- Provides Select All, Clear, and Refresh Profiles options.
- Scans selected accounts sequentially using separate `boto3.Session` instances.
- Generates individual reports for each account.
- Creates combined cross-account reports when multiple profiles are selected.

### 3. Complete API Pagination
All listable AWS API operations use Boto3 paginators where supported.

This prevents incomplete assessments caused by AWS APIs returning only the first page of results.

**Regression tested:** Includes a test with 150 mocked IAM users to verify that pagination works correctly.

### 4. Compliance Framework Mapping
Security findings are mapped to relevant controls from:

- ISO/IEC 27001
- SOC 2
- NIST Cybersecurity Framework (CSF)

This helps organizations understand how technical security weaknesses relate to broader compliance requirements.

### 5. Deterministic Risk Scoring
Uses a weighted scoring methodology to prioritize security findings.

The scoring system is deterministic, meaning the same findings produce the same risk score. It is not an AI-based risk prediction engine.

### 6. Detailed Evidence Reports
Generates reports in three formats:

| Format | Purpose |
|---|---|
| HTML | Human-readable security assessment |
| JSON | Structured evidence for automation and integrations |
| PDF | Shareable security assessment report |

Reports include findings, severity levels, risk scores, compliance mappings, and AWS API errors.

### 7. AWS Error Tracking
AWS API failures are captured separately in `ScanResult.errors`.

Instead of silently hiding failures, the auditor displays them in a dedicated report section so users can distinguish between security findings and incomplete scan coverage.

### 8. Security-Focused Design
- Uses a dedicated least-privilege IAM policy.
- Performs read-only AWS operations.
- Escapes dynamic HTML content to prevent HTML injection.
- Supports CI/CD integration through severity-based exit codes.

---

## Project Improvements

The following improvements address major limitations in the earlier implementation.

| Previous limitation | Improvement |
|---|---|
| Scanned only `us-east-1` | Automatic multi-region discovery |
| API pagination was missing | Paginators for listable operations |
| Separate mock and live scanners | Unified scanning architecture |
| Inconsistent report formats | Shared data model and report generators |
| No compliance mappings in live scans | ISO 27001, SOC 2, and NIST CSF mappings |
| Misleading AI-powered branding | Honest deterministic risk scoring |
| Dynamic HTML values were unescaped | HTML escaping for dynamic content |
| AWS errors were hidden in logs | Dedicated error collection and reporting |
| No automated tests | 12 tests using Moto |
| No CI/CD integration | `--fail-on` severity threshold |
| Administrator-level permissions could be used | Dedicated least-privilege audit policy |

---

## Installation

**Requirements:** Python and AWS credentials for live scanning.

Clone the repository:

```bash
git clone <repository-url>
cd aws-security-compliance-auditor
```

Install dependencies:

```bash
pip install -r requirements.txt
```

---

## Usage

### 1. Demo Mode (No AWS Credentials Required)

Run the auditor using mock data:

```bash
python -m aws_auditor.cli --mock
```

This is useful for demonstrations, testing, and understanding the reporting functionality without connecting to an AWS account.

### 2. Live AWS Security Scan

Run a live assessment using your default AWS credential chain:

```bash
python -m aws_auditor.cli
```

The auditor automatically discovers enabled regions and scans the supported AWS services.

### 3. Scan Specific Regions

```bash
python -m aws_auditor.cli --regions us-east-1,eu-west-1
```

### 4. CI/CD Security Gate

Fail a pipeline when critical security findings are detected:

```bash
python -m aws_auditor.cli --fail-on CRITICAL
```

The command exits with a non-zero status when the configured severity threshold is met.

### 5. Generate JSON Evidence

```bash
python -m aws_auditor.cli --format json --output evidence.json
```

JSON output can be consumed by external security tools, dashboards, and automation pipelines.

---

## Desktop GUI: Multi-Account Scanning

Launch the graphical interface:

```bash
python -m aws_auditor.gui
```

Alternatively:

```bash
python run_gui.py
```

The GUI automatically detects AWS profiles configured on the machine.

**How multi-account scanning works:**

1. Open the desktop application.
2. Select one or more AWS profiles.
3. Start Live Scan.
4. The auditor scans each selected account sequentially.
5. The dashboard updates as each account completes.
6. Individual and combined reports are generated.

**Report structure:**

```text
reports/
├── account_profile_1/
│   ├── aws_audit_report.html
│   ├── aws_audit_report.json
│   └── aws_audit_report.pdf
│
├── account_profile_2/
│   ├── aws_audit_report.html
│   ├── aws_audit_report.json
│   └── aws_audit_report.pdf
│
└── combined_summary/
    ├── combined_summary.html
    ├── combined_summary.json
    └── combined_summary.pdf
```

When no profiles are selected, the GUI uses the default AWS credential chain and generates reports in the working directory.

**Implementation details:**
- Profile discovery uses Python's standard-library `configparser`.
- Each selected profile gets its own `boto3.Session(profile_name=...)`.
- Multi-account aggregation is implemented in `aws_auditor/multi_account.py`.
- Existing report writers are reused without modification.

---

## AWS Permissions

The auditor requires AWS read-only permissions to inspect the supported services.

A dedicated least-privilege policy is provided:

```text
policies/read_only_audit_policy.json
```

Attach this policy to the IAM user or role used for the security assessment.

The policy is designed around read-only operations such as:

- `Get*`
- `List*`
- `Describe*`

The auditor does not modify AWS resources or configurations.

**Security recommendation:** Avoid using `AdministratorAccess` for security auditing. Use the provided policy and review its permissions before deployment.

---

## Testing

Run the automated test suite:

```bash
pytest tests/ -v
```

The project includes 12 automated tests using Moto to simulate AWS services.

Testing covers:

- AWS service scanning.
- IAM security checks.
- S3 security configurations.
- EC2 security group assessments.
- CloudTrail and GuardDuty checks.
- API pagination.
- HTML output escaping.
- Risk scoring and report generation.

A dedicated pagination regression test verifies that all 150 mocked IAM users are processed rather than just the first API response page.

---

## Project Structure

```text
aws-security-compliance-auditor/
│
├── aws_auditor/
│   ├── cli.py
│   ├── gui.py
│   ├── aws_profiles.py
│   ├── multi_account.py
│   ├── scoring.py
│   ├── compliance.py
│   ├── report_html.py
│   ├── report_json.py
│   └── gui_pdf.py
│
├── policies/
│   └── read_only_audit_policy.json
│
├── tests/
│
├── reports/
│
├── requirements.txt
├── run_gui.py
└── README.md
```

*This structure highlights the main components; additional scanner modules and project files may also be present.*

---

## Limitations and Scope

This project is designed as a practical AWS security assessment and prioritization tool. It is not intended to replace enterprise-grade Cloud Security Posture Management (CSPM) platforms or formal compliance audits.

### Current limitations

**1. S3 Regional Handling**

The S3 client is initialized once because S3 has a global service endpoint model. AWS generally redirects bucket metadata requests appropriately, but some requests involving opt-in regions that are not enabled for the account may fail.

These failures are recorded in the report's `errors` section rather than silently ignored. Automatic retry using an alternative regional endpoint is not implemented.

**2. Limited Security Check Coverage**

The auditor currently implements approximately 20 curated security checks.

It focuses on common, high-impact misconfigurations rather than attempting to cover every possible AWS security control.

Tools such as Prowler, ScoutSuite, and AWS Security Hub provide broader CSPM capabilities.

**3. No Historical Tracking**

Each scan produces a point-in-time security assessment.

There is currently no built-in database or historical tracking system for comparing security posture across multiple scans.

Users can store JSON reports in an external database to implement historical analysis.

**4. Not a Certified Compliance Audit**

Compliance mappings indicate relationships between technical findings and selected framework controls.

They do not establish that an organization is compliant or certified against ISO 27001, SOC 2, or NIST CSF.

The generated reports are intended to support security reviews and evidence collection, not replace formal audits.

---

## Future Enhancements

Potential future improvements include:

- Historical security posture tracking.
- Risk trend visualization.
- Additional AWS security checks.
- Integration with AWS Security Hub.
- Automated remediation recommendations.
- Scheduled security assessments.
- Exportable compliance evidence.
- Integration with SIEM and security monitoring platforms.

---

## Technologies Used

| Technology | Purpose |
|---|---|
| Python | Core implementation |
| Boto3 | AWS API integration |
| AWS IAM | Authentication and permissions |
| Moto | AWS service mocking and testing |
| Pytest | Automated testing |
| HTML/CSS | Security report generation |
| JSON | Structured evidence output |
| PDF | Shareable audit reports |

---

## Disclaimer

This project is intended for educational, research, and authorized security assessment purposes.

Only scan AWS accounts for which you have explicit authorization.

The tool performs read-only assessments and does not automatically remediate security findings.

**This is a security prioritization and evidence-generation tool, not a certified auditing solution or a guarantee of AWS security or regulatory compliance.**

---

## Author

Developed as a cybersecurity project focused on AWS security auditing, cloud misconfiguration detection, and compliance evidence generation.
