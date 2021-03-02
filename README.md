# TerraGoat - Vulnerable Terraform Infrastructure

[![Maintained by Bridgecrew.io](https://img.shields.io/badge/maintained%20by-bridgecrew.io-blueviolet)](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat)
[![Infrastructure Tests](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/general)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=INFRASTRUCTURE+SECURITY)
[![CIS Azure](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/cis_azure)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=CIS+AZURE+V1.1)
[![CIS GCP](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/cis_gcp)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=CIS+GCP+V1.1)
[![CIS AWS](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/cis_aws)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=CIS+AWS+V1.2)
[![PCI](https://www.bridgecrew.cloud/badges/github/bridgecrewio/terragoat/pci)](https://www.bridgecrew.cloud/link/badge?vcs=github&fullRepo=bridgecrewio%2Fterragoat&benchmark=PCI-DSS+V3.2)
![Terraform Version](https://img.shields.io/badge/tf-%3E%3D0.12.0-blue.svg) 
[![slack-community](https://slack.bridgecrew.io/badge.svg)](https://slack.bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat)


TerraGoat is Bridgecrew's "Vulnerable by Design" Terraform repository.
![Terragoat](terragoat-logo.png)

TerraGoat is Bridgecrew's "Vulnerable by Design" Terraform repository.
TerraGoat is a learning and training project that demonstrates how common configuration errors can find their way into production cloud environments.

## Table of Contents

* [Introduction](#introduction)
* [Getting Started](#getting-started)
  * [AWS](#aws-setup)
  * [Azure](#azure-setup)
  * [GCP](#gcp-setup)
* [Contributing](#contributing)
* [Support](#support)

## Introduction

TerraGoat was built to enable DevSecOps design and implement a sustainable misconfiguration prevention strategy. It can be used to test a policy-as-code framework like [Bridgecrew](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat) & [Checkov](https://github.com/bridgecrewio/checkov/), inline-linters, pre-commit hooks or other code scanning methods.

TerraGoat follows the tradition of existing *Goat projects that provide a baseline training ground to practice implementing secure development best practices for cloud infrastructure.

## Important notes

* **Where to get help:** the [Bridgecrew Community Slack](https://slack.bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat)

Before you proceed please take a not of these warning:
> :warning: TerraGoat creates intentionally vulnerable AWS resources into your account. **DO NOT deploy TerraGoat in a production environment or alongside any sensitive AWS resources.**

## Requirements

* Terraform 0.12
* aws cli
* azure cli

To prevent vulnerable infrastructure from arriving to production see: [Bridgecrew](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat) & [checkov](https://github.com/bridgecrewio/checkov/), the open source static analysis tool for infrastructure as code.

## Getting started

### AWS Setup

#### Installation (AWS)

You can deploy multiple TerraGoat stacks in a single AWS account using the parameter `TF_VAR_environment`.

#### Create an S3 Bucket backend to keep Terraform state

```bash
export TERRAGOAT_STATE_BUCKET="mydevsecops-bucket"
export TF_VAR_company_name=acme
export TF_VAR_environment=mydevsecops
export TF_VAR_region="us-west-2"

aws s3api create-bucket --bucket $TERRAGOAT_STATE_BUCKET \
    --region $TF_VAR_region --create-bucket-configuration LocationConstraint=$TF_VAR_region

# Enable versioning
aws s3api put-bucket-versioning --bucket $TERRAGOAT_STATE_BUCKET --versioning-configuration Status=Enabled

# Enable encryption
aws s3api put-bucket-encryption --bucket $TERRAGOAT_STATE_BUCKET --server-side-encryption-configuration '{
  "Rules": [
    {
      "ApplyServerSideEncryptionByDefault": {
        "SSEAlgorithm": "aws:kms"
      }
    }
  ]
}'
```

#### Apply TerraGoat (AWS)

```bash
cd terraform/aws/
terraform init \
-backend-config="bucket=$TERRAGOAT_STATE_BUCKET" \
-backend-config="key=$TF_VAR_company_name-$TF_VAR_environment.tfstate" \
-backend-config="region=$TF_VAR_region"

terraform apply
```

#### Remove TerraGoat (AWS)

```bash
terraform destroy
```

#### Creating multiple TerraGoat AWS stacks

```bash
cd terraform/aws/
export TERRAGOAT_ENV=$TF_VAR_environment
export TERRAGOAT_STACKS_NUM=5
for i in $(seq 1 $TERRAGOAT_STACKS_NUM)
do
    export TF_VAR_environment=$TERRAGOAT_ENV$i
    terraform init \
    -backend-config="bucket=$TERRAGOAT_STATE_BUCKET" \
    -backend-config="key=$TF_VAR_company_name-$TF_VAR_environment.tfstate" \
    -backend-config="region=$TF_VAR_region"

    terraform apply -auto-approve
done
```

#### Deleting multiple TerraGoat stacks (AWS)

```bash
cd terraform/aws/
export TF_VAR_environment = $TERRAGOAT_ENV
for i in $(seq 1 $TERRAGOAT_STACKS_NUM)
do
    export TF_VAR_environment=$TERRAGOAT_ENV$i
    terraform init \
    -backend-config="bucket=$TERRAGOAT_STATE_BUCKET" \
    -backend-config="key=$TF_VAR_company_name-$TF_VAR_environment.tfstate" \
    -backend-config="region=$TF_VAR_region"

    terraform destroy -auto-approve
done
```

### Azure Setup

#### Installation (Azure)

You can deploy multiple TerraGoat stacks in a single Azure subscription using the parameter `TF_VAR_environment`.

#### Create an Azure Storage Account backend to keep Terraform state

```bash
export TERRAGOAT_RESOURCE_GROUP="TerraGoatRG"
export TERRAGOAT_STATE_STORAGE_ACCOUNT="mydevsecopssa"
export TERRAGOAT_STATE_CONTAINER="mydevsecops"
export TF_VAR_environment="dev"
export TF_VAR_region="westus"

# Create resource group
az group create --location $TF_VAR_region --name $TERRAGOAT_RESOURCE_GROUP

# Create storage account
az storage account create --name $TERRAGOAT_STATE_STORAGE_ACCOUNT --resource-group $TERRAGOAT_RESOURCE_GROUP --location $TF_VAR_region --sku Standard_LRS --kind StorageV2 --https-only true --encryption-services blob

# Get storage account key
ACCOUNT_KEY=$(az storage account keys list --resource-group $TERRAGOAT_RESOURCE_GROUP --account-name $TERRAGOAT_STATE_STORAGE_ACCOUNT --query [0].value -o tsv)

# Create blob container
az storage container create --name $TERRAGOAT_STATE_CONTAINER --account-name $TERRAGOAT_STATE_STORAGE_ACCOUNT --account-key $ACCOUNT_KEY
```

#### Apply TerraGoat (Azure)

```bash
cd terraform/azure/
terraform init -reconfigure -backend-config="resource_group_name=$TERRAGOAT_RESOURCE_GROUP" \
    -backend-config "storage_account_name=$TERRAGOAT_STATE_STORAGE_ACCOUNT" \
    -backend-config="container_name=$TERRAGOAT_STATE_CONTAINER" \
    -backend-config "key=$TF_VAR_environment.terraform.tfstate"

terraform apply
```

#### Remove TerraGoat (Azure)

```bash
terraform destroy
```

### GCP Setup

#### Installation (GCP)

You can deploy multiple TerraGoat stacks in a single GCP project using the parameter `TF_VAR_environment`.

#### Create a GCS backend to keep Terraform state

To use terraform, a Service Account and matching set of credentials are required.
If they do not exist, they must be manually created for the relevant project.
To create the Service Account:
1. Sign into your GCP project, go to `IAM` > `Service Accounts`.
2. Click the `CREATE SERVICE ACCOUNT`.
3. Give a name to your service account (for example - `terragoat`) and click `CREATE`.
4. Grant the Service Account the `Project` > `Editor` role and click `CONTINUE`.
5. Click `DONE`.

To create the credentials:
1. Sign into your GCP project, go to `IAM` > `Service Accounts` and click on the relevant Service Account.
2. Click `ADD KEY` > `Create new key` > `JSON` and click `CREATE`. This will create a `.json` file and download it to your computer.

We recommend saving the key with a nicer name than the auto-generated one (i.e. `terragoat_credentials.json`), and storing the resulting JSON file inside `terraform/gcp` directory of terragoat.
Once the credentials are set up, create the BE configuration as follows:

```bash
export TF_VAR_environment="dev"
export TF_TERRAGOAT_STATE_BUCKET=remote-state-bucket-terragoat
export TF_VAR_credentials_path=<PATH_TO_CREDNETIALS_FILE> # example: export TF_VAR_credentials_path=terragoat_credentials.json
export TF_VAR_project=<YOUR_PROJECT_NAME_HERE>

# Create storage bucket
gsutil mb gs://${TF_TERRAGOAT_STATE_BUCKET}
```

#### Apply TerraGoat (GCP)

```bash
cd terraform/gcp/
terraform init -reconfigure -backend-config="bucket=$TF_TERRAGOAT_STATE_BUCKET" \
    -backend-config "credentials=$TF_VAR_credentials_path" \
    -backend-config "prefix=terragoat/${TF_VAR_environment}"

terraform apply
```

#### Remove TerraGoat (GCP)

```bash
terraform destroy
```

## Bridgecrew's IaC herd of goats

* [CfnGoat](https://github.com/bridgecrewio/cfngoat) - Vulnerable by design Cloudformation template
* [TerraGoat](https://github.com/bridgecrewio/terragoat) - Vulnerable by design Terraform stack
* [CDKGoat](https://github.com/bridgecrewio/cdkgoat) - Vulnerable by design CDK application

## Contributing

Contribution is welcomed!

We would love to hear about more ideas on how to find vulnerable infrastructure-as-code design patterns.

## Support

[Bridgecrew](https://bridgecrew.io/?utm_source=github&utm_medium=organic_oss&utm_campaign=terragoat) builds and maintains TerraGoat to encourage the adoption of policy-as-code.

If you need direct support you can contact us at [info@bridgecrew.io](mailto:info@bridgecrew.io).

## Existing vulnerabilities (Auto-Generated)
|     | check_id     | file                      | resource                                             | check_name                                                                                                                                                                                               | guideline                                                                                    |
|-----|--------------|---------------------------|------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------|
|   0 | CKV_AZURE_37 | /azure/logging.tf         | azurerm_monitor_log_profile.logging_profile          | Ensure that Activity Log Retention is set 365 days or greater                                                                                                                                            | https://docs.bridgecrew.io/docs/set-activity-log-retention-to-365-days-or-greater            |
|   1 | CKV_AZURE_38 | /azure/logging.tf         | azurerm_monitor_log_profile.logging_profile          | Ensure audit profile captures all the activities                                                                                                                                                         | https://docs.bridgecrew.io/docs/ensure-audit-profile-captures-all-activities                 |
|   2 | CKV_AZURE_16 | /azure/app_service.tf     | azurerm_app_service.app-service1                     | Ensure that Register with Azure Active Directory is enabled on App Service                                                                                                                               | https://docs.bridgecrew.io/docs/bc_azr_iam_1                                                 |
|   3 | CKV_AZURE_18 | /azure/app_service.tf     | azurerm_app_service.app-service1                     | Ensure that 'HTTP Version' is the latest if used to run the web app                                                                                                                                      | https://docs.bridgecrew.io/docs/bc_azr_networking_8                                          |
|   4 | CKV_AZURE_13 | /azure/app_service.tf     | azurerm_app_service.app-service1                     | Ensure App Service Authentication is set on Azure App Service                                                                                                                                            | https://docs.bridgecrew.io/docs/bc_azr_general_2                                             |
|   5 | CKV_AZURE_17 | /azure/app_service.tf     | azurerm_app_service.app-service1                     | Ensure the web app has 'Client Certificates (Incoming client certificates)' set                                                                                                                          | https://docs.bridgecrew.io/docs/bc_azr_networking_7                                          |
|   6 | CKV_AZURE_14 | /azure/app_service.tf     | azurerm_app_service.app-service1                     | Ensure web app redirects all HTTP traffic to HTTPS in Azure App Service                                                                                                                                  | https://docs.bridgecrew.io/docs/bc_azr_networking_5                                          |
|   7 | CKV_AZURE_15 | /azure/app_service.tf     | azurerm_app_service.app-service1                     | Ensure web app is using the latest version of TLS encryption                                                                                                                                             | https://docs.bridgecrew.io/docs/bc_azr_networking_6                                          |
|   8 | CKV_AZURE_16 | /azure/app_service.tf     | azurerm_app_service.app-service2                     | Ensure that Register with Azure Active Directory is enabled on App Service                                                                                                                               | https://docs.bridgecrew.io/docs/bc_azr_iam_1                                                 |
|   9 | CKV_AZURE_18 | /azure/app_service.tf     | azurerm_app_service.app-service2                     | Ensure that 'HTTP Version' is the latest if used to run the web app                                                                                                                                      | https://docs.bridgecrew.io/docs/bc_azr_networking_8                                          |
|  10 | CKV_AZURE_13 | /azure/app_service.tf     | azurerm_app_service.app-service2                     | Ensure App Service Authentication is set on Azure App Service                                                                                                                                            | https://docs.bridgecrew.io/docs/bc_azr_general_2                                             |
|  11 | CKV_AZURE_17 | /azure/app_service.tf     | azurerm_app_service.app-service2                     | Ensure the web app has 'Client Certificates (Incoming client certificates)' set                                                                                                                          | https://docs.bridgecrew.io/docs/bc_azr_networking_7                                          |
|  12 | CKV_AZURE_23 | /azure/sql.tf             | azurerm_sql_server.example                           | Ensure that 'Auditing' is set to 'On' for SQL servers                                                                                                                                                    | https://docs.bridgecrew.io/docs/bc_azr_logging_2                                             |
|  13 | CKV_AZURE_24 | /azure/sql.tf             | azurerm_sql_server.example                           | Ensure that 'Auditing' Retention is 'greater than 90 days' for SQL servers                                                                                                                               | https://docs.bridgecrew.io/docs/bc_azr_logging_3                                             |
|  14 | CKV_AZURE_25 | /azure/sql.tf             | azurerm_mssql_server_security_alert_policy.example   | Ensure that 'Threat Detection types' is set to 'All'                                                                                                                                                     | https://docs.bridgecrew.io/docs/bc_azr_general_6                                             |
|  15 | CKV_AZURE_26 | /azure/sql.tf             | azurerm_mssql_server_security_alert_policy.example   | Ensure that 'Send Alerts To' is enabled for MSSQL servers                                                                                                                                                | https://docs.bridgecrew.io/docs/bc_azr_general_7                                             |
|  16 | CKV_AZURE_27 | /azure/sql.tf             | azurerm_mssql_server_security_alert_policy.example   | Ensure that 'Email service and co-administrators' is 'Enabled' for MSSQL servers                                                                                                                         | https://docs.bridgecrew.io/docs/bc_azr_general_8                                             |
|  17 | CKV_AZURE_28 | /azure/sql.tf             | azurerm_mysql_server.example                         | Ensure 'Enforce SSL connection' is set to 'ENABLED' for MySQL Database Server                                                                                                                            | https://docs.bridgecrew.io/docs/bc_azr_networking_9                                          |
|  18 | CKV_AZURE_53 | /azure/sql.tf             | azurerm_mysql_server.example                         | Ensure 'public network access enabled' is set to 'False' for mySQL servers                                                                                                                               |                                                                                              |
|  19 | CKV_AZURE_54 | /azure/sql.tf             | azurerm_mysql_server.example                         | Ensure MySQL is using the latest version of TLS encryption                                                                                                                                               |                                                                                              |
|  20 | CKV_AZURE_29 | /azure/sql.tf             | azurerm_postgresql_server.example                    | Ensure 'Enforce SSL connection' is set to 'ENABLED' for PostgreSQL Database Server                                                                                                                       | https://docs.bridgecrew.io/docs/bc_azr_networking_10                                         |
|  21 | CKV_AZURE_32 | /azure/sql.tf             | azurerm_postgresql_configuration.thrtottling_config  | Ensure server parameter 'connection_throttling' is set to 'ON' for PostgreSQL Database Server                                                                                                            | https://docs.bridgecrew.io/docs/bc_azr_networking_13                                         |
|  22 | CKV_AZURE_30 | /azure/sql.tf             | azurerm_postgresql_configuration.example             | Ensure server parameter 'log_checkpoints' is set to 'ON' for PostgreSQL Database Server                                                                                                                  | https://docs.bridgecrew.io/docs/bc_azr_networking_11                                         |
|  23 | CKV_AZURE_1  | /azure/instance.tf        | azurerm_linux_virtual_machine.linux_machine          | Ensure Azure Instance does not use basic authentication(Use SSH Key Instead)                                                                                                                             | https://docs.bridgecrew.io/docs/bc_azr_networking_1                                          |
|  24 | CKV_AZURE_39 | /azure/roles.tf           | azurerm_role_definition.example                      | Ensure that no custom subscription owner roles are created                                                                                                                                               | https://docs.bridgecrew.io/docs/do-not-create-custom-subscription-owner-roles                |
|  25 | CKV_AZURE_2  | /azure/storage.tf         | azurerm_managed_disk.example                         | Ensure Azure managed disk have encryption enabled                                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_azr_general_1                                             |
|  26 | CKV_AZURE_51 | /azure/storage.tf         | azurerm_managed_disk.example                         | Ensure Disks are encrypted at rest                                                                                                                                                                       | https://docs.bridgecrew.io/docs/bc_azr_general_15                                            |
|  27 | CKV_AZURE_3  | /azure/storage.tf         | azurerm_storage_account.example                      | Ensure that 'Secure transfer required' is set to 'Enabled'                                                                                                                                               | https://docs.bridgecrew.io/docs/ensure-secure-transfer-required-is-enabled                   |
|  28 | CKV_AZURE_35 | /azure/storage.tf         | azurerm_storage_account.example                      | Ensure default network access rule for Storage Accounts is set to deny                                                                                                                                   | https://docs.bridgecrew.io/docs/set-default-network-access-rule-for-storage-accounts-to-deny |
|  29 | CKV_AZURE_33 | /azure/storage.tf         | azurerm_storage_account.example                      | Ensure Storage logging is enabled for Queue service for read, write and delete requests                                                                                                                  | https://docs.bridgecrew.io/docs/enable-requests-on-storage-logging-for-queue-service         |
|  30 | CKV_AZURE_44 | /azure/storage.tf         | azurerm_storage_account.example                      | Ensure Storage Account is using the latest version of TLS encryption                                                                                                                                     | https://docs.bridgecrew.io/docs/bc_azr_storage_2                                             |
|  31 | CKV_AZURE_43 | /azure/storage.tf         | azurerm_storage_account.example                      | Ensure the Storage Account naming rules                                                                                                                                                                  |                                                                                              |
|  32 | CKV_AZURE_36 | /azure/storage.tf         | azurerm_storage_account_network_rules.test           | Ensure 'Trusted Microsoft Services' is enabled for Storage Account access                                                                                                                                | https://docs.bridgecrew.io/docs/enable-trusted-microsoft-services-for-storage-account-access |
|  33 | CKV_AZURE_10 | /azure/networking.tf      | azurerm_network_security_group.bad_sg                | Ensure that SSH access is restricted from the internet                                                                                                                                                   | https://docs.bridgecrew.io/docs/bc_azr_networking_3                                          |
|  34 | CKV_AZURE_9  | /azure/networking.tf      | azurerm_network_security_group.bad_sg                | Ensure that RDP access is restricted from the internet                                                                                                                                                   | https://docs.bridgecrew.io/docs/bc_azr_networking_2                                          |
|  35 | CKV_AZURE_12 | /azure/networking.tf      | azurerm_network_watcher_flow_log.flow_log            | Ensure that Network Security Group Flow Log retention period is 'greater than 90 days'                                                                                                                   | https://docs.bridgecrew.io/docs/bc_azr_logging_1                                             |
|  36 | CKV_AZURE_6  | /azure/aks.tf             | azurerm_kubernetes_cluster.k8s_cluster               | Ensure AKS has an API Server Authorized IP Ranges enabled                                                                                                                                                | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_3                                          |
|  37 | CKV_AZURE_5  | /azure/aks.tf             | azurerm_kubernetes_cluster.k8s_cluster               | Ensure RBAC is enabled on AKS clusters                                                                                                                                                                   | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_2                                          |
|  38 | CKV_AZURE_4  | /azure/aks.tf             | azurerm_kubernetes_cluster.k8s_cluster               | Ensure AKS logging to Azure Monitoring is Configured                                                                                                                                                     | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_1                                          |
|  39 | CKV_AZURE_8  | /azure/aks.tf             | azurerm_kubernetes_cluster.k8s_cluster               | Ensure Kube Dashboard is disabled                                                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_5                                          |
|  40 | CKV_AZURE_7  | /azure/aks.tf             | azurerm_kubernetes_cluster.k8s_cluster               | Ensure AKS cluster has Network Policy configured                                                                                                                                                         | https://docs.bridgecrew.io/docs/bc_azr_kubernetes_4                                          |
|  41 | CKV_AZURE_19 | /azure/security_center.tf | azurerm_security_center_subscription_pricing.pricing | Ensure that standard pricing tier is selected                                                                                                                                                            | https://docs.bridgecrew.io/docs/ensure-standard-pricing-tier-is-selected                     |
|  42 | CKV_AZURE_20 | /azure/security_center.tf | azurerm_security_center_contact.contact              | Ensure that security contact 'Phone number' is set                                                                                                                                                       | https://docs.bridgecrew.io/docs/bc_azr_general_3                                             |
|  43 | CKV_AZURE_22 | /azure/security_center.tf | azurerm_security_center_contact.contact              | Ensure that 'Send email notification for high severity alerts' is set to 'On'                                                                                                                            | https://docs.bridgecrew.io/docs/bc_azr_general_5                                             |
|  44 | CKV_AZURE_21 | /azure/security_center.tf | azurerm_security_center_contact.contact              | Ensure that 'Send email notification for high severity alerts' is set to 'On'                                                                                                                            | https://docs.bridgecrew.io/docs/bc_azr_general_4                                             |
|  45 | CKV_AZURE_42 | /azure/key_vault.tf       | azurerm_key_vault.example                            | Ensure the key vault is recoverable                                                                                                                                                                      | https://docs.bridgecrew.io/docs/ensure-the-key-vault-is-recoverable                          |
|  46 | CKV_AZURE_40 | /azure/key_vault.tf       | azurerm_key_vault_key.generated                      | Ensure that the expiration date is set on all keys                                                                                                                                                       | https://docs.bridgecrew.io/docs/set-an-expiration-date-on-all-keys                           |
|  47 | CKV_AZURE_41 | /azure/key_vault.tf       | azurerm_key_vault_secret.secret                      | Ensure that the expiration date is set on all secrets                                                                                                                                                    | https://docs.bridgecrew.io/docs/set-an-expiration-date-on-all-secrets                        |
|  48 | CKV_GCP_26   | /gcp/networks.tf          | google_compute_subnetwork.public-subnetwork          | Ensure that VPC Flow Logs is enabled for every subnet in a VPC Network                                                                                                                                   | https://docs.bridgecrew.io/docs/bc_gcp_logging_1                                             |
|  49 | CKV_GCP_3    | /gcp/networks.tf          | google_compute_firewall.allow_all                    | Ensure Google compute firewall ingress does not allow unrestricted rdp access                                                                                                                            | https://docs.bridgecrew.io/docs/bc_gcp_networking_2                                          |
|  50 | CKV_GCP_2    | /gcp/networks.tf          | google_compute_firewall.allow_all                    | Ensure Google compute firewall ingress does not allow unrestricted ssh access                                                                                                                            | https://docs.bridgecrew.io/docs/bc_gcp_networking_1                                          |
|  51 | CKV_GCP_62   | /gcp/gcs.tf               | google_storage_bucket.terragoat_website              | Bucket should log access                                                                                                                                                                                 | https://docs.bridgecrew.io/docs/bc_gcp_logging_2                                             |
|  52 | CKV_GCP_29   | /gcp/gcs.tf               | google_storage_bucket.terragoat_website              | Ensure that Cloud Storage buckets have uniform bucket-level access enabled                                                                                                                               | https://docs.bridgecrew.io/docs/bc_gcp_gcs_2                                                 |
|  53 | CKV_GCP_28   | /gcp/gcs.tf               | google_storage_bucket_iam_binding.allow_public_read  | Ensure that Cloud Storage bucket is not anonymously or publicly accessible                                                                                                                               | https://docs.bridgecrew.io/docs/bc_gcp_public_1                                              |
|  54 | CKV_GCP_14   | /gcp/big_data.tf          | google_sql_database_instance.master_instance         | Ensure all Cloud SQL database instance have backup configuration enabled                                                                                                                                 | https://docs.bridgecrew.io/docs/bc_gcp_general_2                                             |
|  55 | CKV_GCP_11   | /gcp/big_data.tf          | google_sql_database_instance.master_instance         | Ensure that Cloud SQL database Instances are not open to the world                                                                                                                                       | https://docs.bridgecrew.io/docs/bc_gcp_networking_4                                          |
|  56 | CKV_GCP_6    | /gcp/big_data.tf          | google_sql_database_instance.master_instance         | Ensure all Cloud SQL database instance requires all incoming connections to use SSL                                                                                                                      | https://docs.bridgecrew.io/docs/bc_gcp_general_1                                             |
|  57 | CKV_GCP_15   | /gcp/big_data.tf          | google_bigquery_dataset.dataset                      | Ensure that BigQuery datasets are not anonymously or publicly accessible                                                                                                                                 | https://docs.bridgecrew.io/docs/bc_gcp_general_3                                             |
|  58 | CKV_GCP_12   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Network Policy is enabled on Kubernetes Engine Clusters                                                                                                                                           | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_7                                          |
|  59 | CKV_GCP_13   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure a client certificate is used by clients to authenticate to Kubernetes Engine Clusters                                                                                                             | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_8                                          |
|  60 | CKV_GCP_7    | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Legacy Authorization is set to Disabled on Kubernetes Engine Clusters                                                                                                                             | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_2                                          |
|  61 | CKV_GCP_24   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure PodSecurityPolicy controller is enabled on the Kubernetes Engine Clusters                                                                                                                         | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_9                                          |
|  62 | CKV_GCP_1    | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Stackdriver Logging is set to Enabled on Kubernetes Engine Clusters                                                                                                                               | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_1                                          |
|  63 | CKV_GCP_23   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Kubernetes Cluster is created with Alias IP ranges enabled                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_15                                         |
|  64 | CKV_GCP_19   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure GKE basic auth is disabled                                                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_11                                         |
|  65 | CKV_GCP_18   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure GKE Control Plane is not public                                                                                                                                                                   | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_10                                         |
|  66 | CKV_GCP_25   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Kubernetes Cluster is created with Private cluster enabled                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_6                                          |
|  67 | CKV_GCP_8    | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Stackdriver Monitoring is set to Enabled on Kubernetes Engine Clusters                                                                                                                            | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_3                                          |
|  68 | CKV_GCP_21   | /gcp/gke.tf               | google_container_cluster.workload_cluster            | Ensure Kubernetes Clusters are configured with Labels                                                                                                                                                    | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_13                                         |
|  69 | CKV_GCP_9    | /gcp/gke.tf               | google_container_node_pool.custom_node_pool          | Ensure 'Automatic node repair' is enabled for Kubernetes Clusters                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_4                                          |
|  70 | CKV_GCP_22   | /gcp/gke.tf               | google_container_node_pool.custom_node_pool          | Ensure Container-Optimized OS (cos) is used for Kubernetes Engine Clusters Node image                                                                                                                    | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_14                                         |
|  71 | CKV_GCP_10   | /gcp/gke.tf               | google_container_node_pool.custom_node_pool          | Ensure 'Automatic node upgrade' is enabled for Kubernetes Clusters                                                                                                                                       | https://docs.bridgecrew.io/docs/bc_gcp_kubernetes_5                                          |
|  72 | CKV_GCP_32   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure 'Block Project-wide SSH keys' is enabled for VM instances                                                                                                                                         | https://docs.bridgecrew.io/docs/bc_gcp_networking_8                                          |
|  73 | CKV_GCP_36   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure that IP forwarding is not enabled on Instances                                                                                                                                                    | https://docs.bridgecrew.io/docs/bc_gcp_networking_12                                         |
|  74 | CKV_GCP_38   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure VM disks for critical VMs are encrypted with Customer Supplied Encryption Keys (CSEK)                                                                                                             | https://docs.bridgecrew.io/docs/encrypt-boot-disks-for-instances-with-cseks                  |
|  75 | CKV_GCP_30   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure that instances are not configured to use the default service account                                                                                                                              | https://docs.bridgecrew.io/docs/bc_gcp_iam_1                                                 |
|  76 | CKV_GCP_35   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure 'Enable connecting to serial ports' is not enabled for VM Instance                                                                                                                                | https://docs.bridgecrew.io/docs/bc_gcp_networking_11                                         |
|  77 | CKV_GCP_39   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure Compute instances are launched with Shielded VM enabled                                                                                                                                           | https://docs.bridgecrew.io/docs/bc_gcp_general_y                                             |
|  78 | CKV_GCP_34   | /gcp/instances.tf         | google_compute_instance.server                       | Ensure that no instance in the project overrides the project setting for enabling OSLogin(OSLogin needs to be enabled in project metadata for all instances)                                             | https://docs.bridgecrew.io/docs/bc_gcp_networking_10                                         |
|  79 | CKV_GCP_37   | /gcp/instances.tf         | google_compute_disk.unencrypted_disk                 | Ensure VM disks for critical VMs are encrypted with Customer Supplied Encryption Keys (CSEK)                                                                                                             | https://docs.bridgecrew.io/docs/bc_gcp_general_x                                             |
|  80 | CKV_AWS_7    | /aws/kms.tf               | aws_kms_key.logs_key                                 | Ensure rotation for customer created CMKs is enabled                                                                                                                                                     | https://docs.bridgecrew.io/docs/logging_8                                                    |
|  81 | CKV_AWS_50   | /aws/lambda.tf            | aws_lambda_function.analysis_lambda                  | X-ray tracing is enabled for Lambda                                                                                                                                                                      | https://docs.bridgecrew.io/docs/bc_aws_serverless_4                                          |
|  82 | CKV_AWS_45   | /aws/lambda.tf            | aws_lambda_function.analysis_lambda                  | Ensure no hard-coded secrets exist in lambda environment                                                                                                                                                 | https://docs.bridgecrew.io/docs/bc_aws_secrets_3                                             |
|  83 | CKV_AWS_40   | /aws/iam.tf               | aws_iam_user_policy.userpolicy                       | Ensure IAM policies are attached only to groups or roles (Reducing access management complexity may in-turn reduce opportunity for a principal to inadvertently receive or retain excessive privileges.) | https://docs.bridgecrew.io/docs/iam_16-iam-policy-privileges-1                               |
|  84 | CKV_AWS_44   | /aws/neptune.tf           | aws_neptune_cluster.default                          | Ensure Neptune storage is securely encrypted                                                                                                                                                             | https://docs.bridgecrew.io/docs/general_18                                                   |
|  85 | CKV_AWS_101  | /aws/neptune.tf           | aws_neptune_cluster.default                          | Ensure Neptune logging is enabled                                                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_aws_logging_24                                            |
|  86 | CKV_AWS_38   | /aws/eks.tf               | aws_eks_cluster.eks_cluster                          | Ensure Amazon EKS public endpoint not accessible to 0.0.0.0/0                                                                                                                                            | https://docs.bridgecrew.io/docs/bc_aws_kubernetes_1                                          |
|  87 | CKV_AWS_37   | /aws/eks.tf               | aws_eks_cluster.eks_cluster                          | Ensure Amazon EKS control plane logging enabled for all log types                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_aws_kubernetes_4                                          |
|  88 | CKV_AWS_58   | /aws/eks.tf               | aws_eks_cluster.eks_cluster                          | Ensure EKS Cluster has Secrets Encryption Enabled                                                                                                                                                        | https://docs.bridgecrew.io/docs/bc_aws_kubernetes_3                                          |
|  89 | CKV_AWS_39   | /aws/eks.tf               | aws_eks_cluster.eks_cluster                          | Ensure Amazon EKS public endpoint disabled                                                                                                                                                               | https://docs.bridgecrew.io/docs/bc_aws_kubernetes_2                                          |
|  90 | CKV_AWS_92   | /aws/elb.tf               | aws_elb.weblb                                        | Ensure the ELB has access logging enabled                                                                                                                                                                | https://docs.bridgecrew.io/docs/bc_aws_logging_23                                            |
|  91 | CKV_AWS_5    | /aws/es.tf                | aws_elasticsearch_domain.monitoring-framework        | Ensure all data stored in the Elasticsearch is securely encrypted at rest                                                                                                                                | https://docs.bridgecrew.io/docs/elasticsearch_3-enable-encryptionatrest                      |
|  92 | CKV_AWS_84   | /aws/es.tf                | aws_elasticsearch_domain.monitoring-framework        | Ensure Elasticsearch Domain Logging is enabled                                                                                                                                                           | https://docs.bridgecrew.io/docs/elasticsearch_7                                              |
|  93 | CKV_AWS_83   | /aws/es.tf                | aws_elasticsearch_domain.monitoring-framework        | Ensure Elasticsearch Domain enforces HTTPS                                                                                                                                                               | https://docs.bridgecrew.io/docs/elasticsearch_6                                              |
|  94 | CKV_AWS_46   | /aws/ec2.tf               | aws_instance.web_host                                | Ensure no hard-coded secrets exist in EC2 user data                                                                                                                                                      | https://docs.bridgecrew.io/docs/bc_aws_secrets_1                                             |
|  95 | CKV_AWS_8    | /aws/ec2.tf               | aws_instance.web_host                                | Ensure all data stored in the Launch configuration EBS is securely encrypted                                                                                                                             | https://docs.bridgecrew.io/docs/general_13                                                   |
|  96 | CKV_AWS_79   | /aws/ec2.tf               | aws_instance.web_host                                | Ensure Instance Metadata Service Version 1 is not enabled                                                                                                                                                | https://docs.bridgecrew.io/docs/bc_aws_general_31                                            |
|  97 | CKV_AWS_3    | /aws/ec2.tf               | aws_ebs_volume.web_host_storage                      | Ensure all data stored in the EBS is securely encrypted                                                                                                                                                  | https://docs.bridgecrew.io/docs/general_3-encrypt-eps-volume                                 |
|  98 | CKV_AWS_24   | /aws/ec2.tf               | aws_security_group.web-node                          | Ensure no security groups allow ingress from 0.0.0.0:0 to port 22                                                                                                                                        | https://docs.bridgecrew.io/docs/networking_1-port-security                                   |
|  99 | CKV_AWS_52   | /aws/ec2.tf               | aws_s3_bucket.flowbucket                             | Ensure S3 bucket has MFA delete enabled                                                                                                                                                                  |                                                                                              |
| 100 | CKV_AWS_18   | /aws/ec2.tf               | aws_s3_bucket.flowbucket                             | Ensure the S3 bucket has access logging enabled                                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_13-enable-logging                                         |
| 101 | CKV_AWS_21   | /aws/ec2.tf               | aws_s3_bucket.flowbucket                             | Ensure all data stored in the S3 bucket have versioning enabled                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_16-enable-versioning                                      |
| 102 | CKV_AWS_19   | /aws/ec2.tf               | aws_s3_bucket.flowbucket                             | Ensure all data stored in the S3 bucket is securely encrypted at rest                                                                                                                                    | https://docs.bridgecrew.io/docs/s3_14-data-encrypted-at-rest                                 |
| 103 | CKV_AWS_16   | /aws/db-app.tf            | aws_db_instance.default                              | Ensure all data stored in the RDS is securely encrypted at rest                                                                                                                                          | https://docs.bridgecrew.io/docs/general_4                                                    |
| 104 | CKV_AWS_17   | /aws/db-app.tf            | aws_db_instance.default                              | Ensure all data stored in the RDS bucket is not public accessible                                                                                                                                        | https://docs.bridgecrew.io/docs/public_2                                                     |
| 105 | CKV_AWS_8    | /aws/db-app.tf            | aws_instance.db_app                                  | Ensure all data stored in the Launch configuration EBS is securely encrypted                                                                                                                             | https://docs.bridgecrew.io/docs/general_13                                                   |
| 106 | CKV_AWS_79   | /aws/db-app.tf            | aws_instance.db_app                                  | Ensure Instance Metadata Service Version 1 is not enabled                                                                                                                                                | https://docs.bridgecrew.io/docs/bc_aws_general_31                                            |
| 107 | CKV_AWS_41   | /aws/providers.tf         | aws.plain_text_access_keys_provider                  | Ensure no hard coded AWS access key and secret key exists in provider                                                                                                                                    | https://docs.bridgecrew.io/docs/bc_aws_secrets_5                                             |
| 108 | CKV_AWS_52   | /aws/s3.tf                | aws_s3_bucket.data                                   | Ensure S3 bucket has MFA delete enabled                                                                                                                                                                  |                                                                                              |
| 109 | CKV_AWS_20   | /aws/s3.tf                | aws_s3_bucket.data                                   | S3 Bucket has an ACL defined which allows public READ access.                                                                                                                                            | https://docs.bridgecrew.io/docs/s3_1-acl-read-permissions-everyone                           |
| 110 | CKV_AWS_18   | /aws/s3.tf                | aws_s3_bucket.data                                   | Ensure the S3 bucket has access logging enabled                                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_13-enable-logging                                         |
| 111 | CKV_AWS_21   | /aws/s3.tf                | aws_s3_bucket.data                                   | Ensure all data stored in the S3 bucket have versioning enabled                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_16-enable-versioning                                      |
| 112 | CKV_AWS_19   | /aws/s3.tf                | aws_s3_bucket.data                                   | Ensure all data stored in the S3 bucket is securely encrypted at rest                                                                                                                                    | https://docs.bridgecrew.io/docs/s3_14-data-encrypted-at-rest                                 |
| 113 | CKV_AWS_52   | /aws/s3.tf                | aws_s3_bucket.financials                             | Ensure S3 bucket has MFA delete enabled                                                                                                                                                                  |                                                                                              |
| 114 | CKV_AWS_18   | /aws/s3.tf                | aws_s3_bucket.financials                             | Ensure the S3 bucket has access logging enabled                                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_13-enable-logging                                         |
| 115 | CKV_AWS_21   | /aws/s3.tf                | aws_s3_bucket.financials                             | Ensure all data stored in the S3 bucket have versioning enabled                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_16-enable-versioning                                      |
| 116 | CKV_AWS_19   | /aws/s3.tf                | aws_s3_bucket.financials                             | Ensure all data stored in the S3 bucket is securely encrypted at rest                                                                                                                                    | https://docs.bridgecrew.io/docs/s3_14-data-encrypted-at-rest                                 |
| 117 | CKV_AWS_52   | /aws/s3.tf                | aws_s3_bucket.operations                             | Ensure S3 bucket has MFA delete enabled                                                                                                                                                                  |                                                                                              |
| 118 | CKV_AWS_18   | /aws/s3.tf                | aws_s3_bucket.operations                             | Ensure the S3 bucket has access logging enabled                                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_13-enable-logging                                         |
| 119 | CKV_AWS_19   | /aws/s3.tf                | aws_s3_bucket.operations                             | Ensure all data stored in the S3 bucket is securely encrypted at rest                                                                                                                                    | https://docs.bridgecrew.io/docs/s3_14-data-encrypted-at-rest                                 |
| 120 | CKV_AWS_52   | /aws/s3.tf                | aws_s3_bucket.data_science                           | Ensure S3 bucket has MFA delete enabled                                                                                                                                                                  |                                                                                              |
| 121 | CKV_AWS_19   | /aws/s3.tf                | aws_s3_bucket.data_science                           | Ensure all data stored in the S3 bucket is securely encrypted at rest                                                                                                                                    | https://docs.bridgecrew.io/docs/s3_14-data-encrypted-at-rest                                 |
| 122 | CKV_AWS_52   | /aws/s3.tf                | aws_s3_bucket.logs                                   | Ensure S3 bucket has MFA delete enabled                                                                                                                                                                  |                                                                                              |
| 123 | CKV_AWS_18   | /aws/s3.tf                | aws_s3_bucket.logs                                   | Ensure the S3 bucket has access logging enabled                                                                                                                                                          | https://docs.bridgecrew.io/docs/s3_13-enable-logging                                         |
| 124 | CKV_AWS_51   | /aws/ecr.tf               | aws_ecr_repository.repository                        | Ensure ECR Image Tags are immutable                                                                                                                                                                      | https://docs.bridgecrew.io/docs/bc_aws_general_24                                            |
| 125 | CKV_AWS_33   | /aws/ecr.tf               | aws_ecr_repository.repository                        | Ensure ECR image scanning on push is enabled                                                                                                                                                             | https://docs.bridgecrew.io/docs/general_8                                                    |


---


