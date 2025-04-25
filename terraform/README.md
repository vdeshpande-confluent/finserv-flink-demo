![image](img/confluent-logo-300-2.png)

# Workshop Deployment via terraform

IMPORTANT TO KNOW FOR THE WORKSHOP:
We run in AWS, Azure and Google Cloud. Currently we do support [24 Regions](https://docs.confluent.io/cloud/current/flink/overview.html#af-long-is-everywhere).
Please be aware that the cluster and the Flink Pool need to be in the same Cloud-Provider-Region.

This is the deployment of confluent cloud infrastructure resources to run the Flink SQL Hands-on Workshop.
We will deploy with terraform:
 - Environment:
     - Name: flink_hands-on+UUID
     - with enabled Schema Registry (essentials) in AWS region (eu-central-1)
 - Confluent Cloud Basic Cloud: cc_handson_cluster
    - in AWS in region (eu-central-1)
 - Connectors:
    - Datagen for shoe_products
    - Datagen for shoe_customers 
    - Datagen for show_orders
 - Service Accounts
    - app_manager-XXXX with Role EnvironmentAdmin
    - sr-XXXX with Role EnvironmentAdmin
    - clients-XXXX with Role CloudClusterAdmin
    - connectors-XXXX
 
![image](img/terraform_deployment_1.png)

# Pre-requisites
- User account on [Confluent Cloud](https://www.confluent.io/confluent-cloud/tryfree)
- Local install of [Terraform](https://www.terraform.io) (details below)
- Local install of [jq](https://jqlang.github.io/jq/download) (details below)
- Local install Confluent CLI, [install the cli](https://docs.confluent.io/confluent-cli/current/install.html) 
- Create API Key in Confluent Cloud via CLI:
    ```bash
    confluent login
    confluent api-key create --resource cloud --description "API for terraform"
    # It may take a couple of minutes for the API key to be ready.
    # Save the API key and secret. The secret is not retrievable later.
    #+------------+------------------------------------------------------------------+
    #| API Key    | <your generated key>                                             |
    #| API Secret | <your generated secret>                                          |
    #+------------+------------------------------------------------------------------+
    ``````
 - Or visit the [Cloud API Key](https://confluent.cloud/settings/api-keys) page to create a Cloud API Key for your user, if you don't have any yet.

# Installation (only need to do that once)

## Install Terraform on MacOS
```
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
brew update
brew upgrade hashicorp/tap/terraform
```
If you are running Windows, please use this [guide](https://learn.microsoft.com/en-us/azure/developer/terraform/get-started-windows-bash?tabs=bash) <br>
If you are running on Ubuntu (or WSL2 with Ubuntu), please use this [guide](https://computingforgeeks.com/how-to-install-terraform-on-ubuntu/)

This tutorial was tested with Terraform v1.6.4 and confluent terraform provider 1.55.0 . To upgrade terraform on MacOS use
```bash
brew upgrade terraform
```
Or download new version from [website](https://www.terraform.io/downloads.html)

## Install jq
```
brew install jq
```
If you are running Windows, download from [here](https://jqlang.github.io/jq/download/) <br>
If you are running on Ubuntu (or WSL2 with Ubuntu), please follow the instructions [here](https://lindevs.com/install-jq-on-ubuntu)

## Install Confluent Cli
Please install the Confluent CLI, with these [instructions](https://docs.confluent.io/confluent-cli/current/install.html) 

```sh
# For Mac
brew install confluentinc/tap/cli
```

This tutorial was developed with Confluent CLI v3.41.0 and run on lates: v4.9.8

# Provision services for the demo

Clone the repo on your desktop.
```bash
cd $HOME # or where-ever directory you want to use
git clone https://github.com/griga23/shoe-store.git
cd shoe-store
```

## Create Confluent Cloud Resource management API Keys

Go to the Confluent Console via the url [https://confluent.cloud/settings/api-keys](https://confluent.cloud/settings/api-keys) then, use the "+ Add API key" button on the right side of the page. Select "My account", >> "Next" then select the "Cloud resource management" tile, complete and download the API key. It will be use to get Terraform plan and provision resources.   

## Set environment variables
- Add your API key to the Terraform variables by creating a tfvars file

```bash
cat > $PWD/terraform/terraform.tfvars <<EOF
confluent_cloud_api_key = "{Confluent Cloud API Key}"
confluent_cloud_api_secret = "{Confluent Cloud API Key Secret}"
cc_cloud_provider = "{the_selected_cloud_provider}"
cc_cloud_region= "{the_selected_region_of_the_confluent_cloud}"
EOF
```

### Optional: Prefix your resources

In some cases you may want to prefix your resources, to do so use:

```bash
cat >> $PWD/terraform/terraform.tfvars <<EOF
use_prefix = "{choose your prefix}"
EOF
```

## Deploy via terraform
run the following commands:
```Bash
cd ./terraform
terraform init
terraform plan
terraform apply
# Apply shows you what will be provision, enter yes to continue provisioning 
terraform output -json
# for sensitive data
terraform output -raw SRSecret
terraform output -raw AppManagerSecret
terraform output -raw ClientSecret
```

Please check whether the terraform execution went without errors.

You can copy the login instruction also from the UI.

To continue with the UI:
 - Access Confluent Cloud WebUI: https://confluent.cloud/login
 - Access your Environment: `flink_handson_terraform-XXXXXXXX`
 - Start with [lab1](../lab1.md)

You deployed:

![image](img/terraform_deployment_1.png)

You are ready to [start with LAB1](../lab1.md)

# Destroy the hands.on infrastructure
```bash
terraform destroy
```
There could be a conflict destroying everything with our Tags. In this case destroy again via terraform.
```bash
#╷
#│ Error: error deleting Tag "<tagID>/PII": 409 Conflict
#│ 
#│ 
#╵
#╷
#│ Error: error deleting Tag "<tagID>/Public": 409 Conflict
#│ 
#│ 
#╵
# destroy again
terraform destroy
``` 
