# Provision ROSA STS Cluster using Red Hat RHCS Terraform provider

This guide shows how to create a Private Link STS ROSA cluster (Public coming soon!), the required operator IAM roles and the oidc provider using Red Hat [RHCS Terraform Provider](https://github.com/terraform-redhat/terraform-provider-rhcs). This guide also provides examples of creating other necessary components like AWS VPC, Azure App Registration for Azure AD IDP provider and Azure AD IDP for ROSA Cluster. These additional component creations can be enabled using terraform variables. The goal of this guide is to show how to create a ROSA STS cluster and how to add additional terraform modules to extend cluster provisioning using terraform automation. 

> This guide extends the official RHCS ROSA Cluster TF privisioning example. Detail info can be found [here](https://github.com/terraform-redhat/terraform-provider-rhcs/tree/main/examples/create_rosa_sts_cluster/classic_sts/cluster)

## Prerequisites

* Install [Terraform](https://www.terraform.io/downloads.html)
* OCM authentication [token](https://console.redhat.com/openshift/token)
* Install and configure `aws` cli

### Environment Setup

Variables can be passed to terraform using either Environment variables or using the terraform.tfvars file, which is placed in the directory where the terraform command executes. 

Following example shows how to configure common terraform environment variables.
   ```bash
   export TF_VAR_url="https://api.openshift.com"
   export TF_LOG="DEBUG"
   export TF_LOG_PATH="logs/terraform.log"
   export RHCS_TOKEN="OCM TOKEN Value"
   ```

### Usage
Following examples show how to create the terraform.tfvars file for Public and Private link ROSA STS clusters.

## Quick getting started

A quick setup without using modules to build a ROSA STS cluster can be found in the ./rosa_sts_managed_oidc_no_modules directory. This can be helpful if you need to copy working terraform code into a protected environment without needing to copy the modules directory. This has limited customisation but is a good starting point for new users.

## Example of a ROSA public cluster with a managed OIDC provider

Change to the ./rosa_sts_managed_oidc_no_modules directory and run:

```bash
terraform apply -var cluster_name=my-rosa -var account_role_prefix=my-rosa -var operator_role_prefix=my-rosa -var aws_region=eu-west-1 -var availability_zones='["eu-west-1a","eu-west-1b","eu-west-1c"]' -var multi_az=true -var create_vpc=true -var enable_private_link=false
```

## Customisable ROSA STS Cluster with managed OIDC provider

You can use the example in the ./rosa_sts_managed_oidc directory to make a ROSA STS cluster with a Managed OIDC provider

## Customisable ROSA STS Cluster with unmanaged OIDC provider

You can use the example in the ./rosa_sts_unmanaged_oidc directory to make a ROSA STS cluster with a self managed (unmanaged) OIDC provider

### Sample terraform.tfvars file for the Public ROSA STS Cluster
```
token="OCM TOKEN Value"
url="https://api.openshift.com"
# Module selection
create_vpc=false
create_aad_app=false
create_idp_aad=false
create_account_roles=false
#ROSA Cluster Info
account_role_prefix="ManagedOpenShift"
operator_role_prefix="mobbtf"
cluster_name="mobbtf-01"
multi_az=true
#AWS Info
aws_region="us-east-2"
availability_zones = ["us-east-2a","us-east-2b","us-east-2c"]
#Private Link Cluster
enable_private_link=false
additional_tags={
     Terraform = "true"
     TFEnvironment = "dev"
     TFOwner = "mobb@redhat.com"
     ROSAClusterName="mobbtf-01"
   }
#Azure AD app reg Info
aad_tenant_id="AZURE_Tenant_id"
```

## Sample terraform.tfvars file for the Private Link ROSA STS Cluster
```
token="OCM TOKEN Value"
url="https://api.openshift.com"
# Module selection
create_vpc=true
create_aad_app=false
create_idp_aad=false
create_account_roles=false
#ROSA Cluster Info
account_role_prefix="ManagedOpenShift"
operator_role_prefix="mobbtf"
cluster_name="mobbtf-01"
multi_az=true
#AWS Info
aws_region="us-east-2"
availability_zones = ["us-east-2a","us-east-2b","us-east-2c"]
#Private Link Cluster Info. Enable create_vpc to create new AWS VPC
enable_private_link=true
vpc_cidr_block="10.66.0.0/16"
private_subnet_cidrs=["10.66.1.0/24", "10.66.2.0/24", "10.66.3.0/24"]
public_subnet_cidrs=["10.66.101.0/24", "10.66.102.0/24", "10.66.103.0/24"]
additional_tags={
     Terraform = "true"
     TFEnvironment = "dev"
     TFOwner = "mobb@redhat.com"
     ROSAClusterName="mobbtf-01"
   }
#Azure AD app reg Info
aad_tenant_id="AZURE_Tenant_id"
```

## Deploy

1. Download this repo

    ```bash
    git clone https://github.com/rh-mobb/terraform_ocm_rosa_sts.git
    ```

1. Change to the directory you wish to use to deploy your cluster

1. Create terraform.tfvars as above, then run

    ```bash
    terraform init
    terraform plan
    terraform apply
    ```

## Cleanup

1. To destroy resources

    ```bash
    terraform destroy
    ```

## FAQ

1. Do I want managed or unmanaged OIDC providers?

The managed OIDC provider creates the underlying resources in a Red Hat account, which you can consume. An unmanaged OIDC provider creates the underlying resources in your own account. If you are unsure, starting with Managed is probably easier.

2. I'm getting this error, what do I do?

```
│ Error: either a token, an user name and password or a client identifier and secret are necessary, but none has been provided
```

Please export your RHCS_TOKEN as an environment variable, you can get this from https://console.redhat.com/openshift/token/rosa
