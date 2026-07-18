# 🛡️ Azure GRC Auditor

An automated compliance scanning engine that audits Azure environments against logging and monitoring controls required by major security frameworks — SOC 2, NIST 800-53, and ISO 27001.

Security teams and auditors typically need to manually verify that Activity Logs (the record of who did what across a subscription) are actually being exported to a durable destination — a step that's easy to overlook and tedious to check by hand. This tool queries Azure Resource Manager directly to confirm Diagnostic Settings are configured correctly, flagging any gaps in audit trail coverage before an auditor finds them first.

---

## 🔒 Security

- **Secret Management:** No API keys, secrets, or passwords are hardcoded or stored. Authentication relies entirely on short-lived OAuth tokens.
- **Least Privilege:** The production deployment restricts the Managed Identity strictly to the `Reader` RBAC role — the tool can read configurations but never modify them.
- **Secure Communication:** All communication with Azure is enforced over HTTPS (TLS 1.2+).
- **Audit Trail:** The application itself produces timestamped, non-repudiable logs as part of its output.

---

## ✨ Features

- **Automated Compliance Verification:** Checks Azure subscriptions against SOC 2 (CC7.2) and NIST 800-53 (AU-2/AU-12) logging controls.
- **Direct API Integration:** Queries Azure REST APIs directly rather than relying on SDK wrappers, ensuring accurate, real-time diagnostic settings data.
- **Dual Execution Modes:** Run locally for an on-demand check, or deploy to Azure Automation for continuous, scheduled auditing.
- **Agentless Authentication:** Uses `DefaultAzureCredential` to authenticate via local Azure CLI login during development, and Managed Identity in production.
- **Multi-Format Reporting:** Generates JSON reports for data pipelines and CSV summaries for auditors.
- **Infrastructure as Code:** Includes Terraform modules for automated deployment to Azure.

---

## 🏗️ System Architecture

- **Compute Layer:** Runs as a scheduled Python 3 Runbook inside an Azure Automation Account.
- **Identity Layer:** Secured by a System-Assigned Managed Identity scoped to the `Reader` role only.
- **API Integration Layer:** Communicates over TLS 1.2+ directly with the Azure Resource Manager REST API (`management.azure.com`).
- **Output Layer:** Writes JSON and CSV reports to standard output or an integrated logging sink (e.g., Azure Monitor).

```text
+---------------------+        OAuth 2.0         +-------------------------+
|  Azure Automation   | -----------------------> |    Azure Entra ID       |
|    (Python Runbook) |                          |   (Managed Identity)    |
+---------------------+                          +-------------------------+
          |                                                   |
          | REST API GET (TLS)                                | Validate Token
          v                                                   v
+--------------------------------------------------------------------------+
|                        Azure Resource Manager (ARM)                      |
|                      (Diagnostic Settings Evaluation)                    |
+--------------------------------------------------------------------------+
```

---

## 🚀 How It Works

1. **Authentication:** The script authenticates via the Azure Identity library — using the local CLI login during development, or its Managed Identity when running in the cloud.
2. **API Query:** It sends a request to the Azure REST API to retrieve `microsoft.insights/diagnosticSettings` for the target subscription.
3. **Data Parsing:** The response is parsed to determine where Administrative and Security logs are currently being exported (e.g., Log Analytics Workspace, Storage Account).
4. **Compliance Evaluation:** These settings are checked against the tool's GRC baseline. Missing exports or unsecured storage destinations are flagged.
5. **Report Generation:** A detailed JSON file and a summary CSV file are generated for handoff to auditors or ingestion into a dashboard.

---

## 🧠 Key Design Decisions

- **Raw REST APIs over the Python SDK:** Azure's Python SDK (`azure-mgmt-monitor`) has changed diagnostic settings methods across versions. Calling the ARM REST API directly via `requests` avoids that instability.
- **Azure Automation over Azure Functions:** Automation Accounts natively support long-running, scheduled Python scripts without HTTP-trigger boilerplate, keeping the deployment simple and reducing the attack surface.

---

## 📈 Scalability

- **Stateless Execution:** The script holds no state between runs, allowing it to run concurrently across many subscriptions.
- **Horizontal Scaling:** The Terraform module can be adapted to deploy the runbook across multiple tenants.
- **Low Overhead:** Direct REST API calls avoid the memory overhead of loading full Azure SDK client libraries.

---

## 🚀 Getting Started

### Project Structure

```text
Logging_Monitoring/
├── azure_monitor_validator.py   # Core compliance scanning engine
├── terraform/                   # IaC deployment configurations
│   └── main.tf                  # Automation Account & Managed Identity setup
├── README.md                    # Project overview and setup guide
└── Azure_GRC_Auditor_Guide.md   # Deep-dive user and architectural guide
```

### Prerequisites
- Python 3.10+
- Azure CLI (`az`)
- Terraform (optional, for cloud deployment)

### Installation
1. Clone the repository:
```bash
   git clone <YOUR_REPO_URL>
   cd Logging_Monitoring
```
2. Create and activate a virtual environment:
```bash
   python3 -m venv venv
   source venv/bin/activate
```
3. Install dependencies:
```bash
   pip install azure-identity requests
```
4. Authenticate with Azure:
```bash
   az login
```

### Usage

**Run a local, on-demand scan:**
```bash
python azure_monitor_validator.py --subscription-id <YOUR_SUBSCRIPTION_ID>
```
This outputs a JSON report and a CSV summary directly to the working directory.

**Deploy the continuous, automated version to Azure:**
```bash
cd terraform
terraform init
terraform plan
terraform apply
```

---

## 🔮 Future Improvements

- **Multi-Subscription Support:** Query Azure Management Groups to audit across many subscriptions at once.
- **Auto-Remediation:** An optional "fix" mode using an elevated Managed Identity to automatically correct missing diagnostic settings.
- **Dashboard Integration:** Pipe JSON output directly into Azure Sentinel or Power BI for visual tracking.

---

## 🎓 Skills Demonstrated

- Secure software development
- Cloud architecture (Azure)
- Cybersecurity best practices and GRC
- Identity & Access Management (OAuth / Managed Identities)
- Infrastructure as Code (Terraform)
- API integration and REST
- Python systems engineering
