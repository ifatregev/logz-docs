---
layout: article
title: Terraform Logz.io Provider
permalink: /integrations/terraform/
image: https://dytvr9ot2sszz.cloudfront.net/logz-docs/social-assets/docs-social.jpg
description: Configure Terraform with Logz.io API
flags:
  logzio-plan: pro
open-source:
  - title: Logz.io-Terraform-Provider
    github-repo: logzio_terraform_provider
tags:
  - integrations
contributors:
  - yyyogev
  - mirii1994
  - shalper
---

The Terraform Logz.io provider offers a great way to build integrations using Logz.io APIs.

Terraform is an infrastructure orchestrator written in Hashicorp Language (HCL). It is a popular Infrastructure-as-Code (IaC) tool that does away with manual configuration processes. You can take advantage of the Terraform Logz.io provider to really streamline the process of integrating observability into your dev workflows.

This guide assumes working knowledge of HashiCorp Terraform. If you're new to Terraform, we've got a great [introduction](https://logz.io/blog/terraform-vs-ansible-vs-puppet/) if you're in for one. We also recommend the official [Terraform guides and tutorials](https://www.terraform.io/guides/index.html).

For additional examples, other than those provided here, see our [GitHub project](https://github.com/logzio/logzio_terraform_provider/tree/master/examples).

### Capabilities

You can use the Terraform Logz.io Provider to manage users and log accounts in Logz.io, create and update log-based alerts and notification channels, and more.

The following Logz.io API endpoints are supported by this provider:

* [User management](https://docs.logz.io/api/#tag/Manage-users)
* [Notification channels](https://docs.logz.io/api/#tag/Manage-notification-endpoints)
* [Sub accounts](https://docs.logz.io/api/#tag/Manage-time-based-log-accounts)
* [Logs-based alerts v2](https://docs.logz.io/api/#tag/Alerts)
* [Log shipping tokens](https://docs.logz.io/api/#tag/Manage-log-shipping-tokens)
* [Drop filters](https://docs.logz.io/api/#tag/Drop-filters)
* [Archive logs](https://docs.logz.io/api/#tag/Archive-logs)
* [Restore logs](https://docs.logz.io/api/#tag/Restore-logs)
* [Authentication groups](https://docs.logz.io/api/#tag/Authentication-groups)
* [OpenSearch Dashboards objects](https://docs.logz.io/api/#tag/Import-or-export-Kibana-objects)
* [S3 Fetcher](https://docs.logz.io/api/#tag/Connect-to-S3-Buckets)

#### Working with Terraform

<div class="tasklist">

**Before you begin, you'll need**:

* [Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli)
* [Logz.io API token](/)

##### Get the Terraform Logz.io Provider

To install this provider, copy and paste this code into your Terraform configuration:

```hcl
terraform {
  required_providers {
    logzio = {
      source = "logzio/logzio"
    }
  }
}
```

This will install the latest Logz.io provider.
If you wish to use a specific version of the provider, add under `source` the field `version` and specify your preferred version.


##### Configuring the provider

The provider accepts the following arguments:

* **api_token** - (Required) The API token is used for authentication. [Learn more](/user-guide/tokens/api-tokens.html).

* **region** - (Defaults to null) The 2-letter region code identifies where your Logz.io account is hosted.
Defaults to null for accounts hosted in the US East - Northern Virginia region. [Learn more](https://docs.logz.io/user-guide/accounts/account-region.html)

###### Example

You can pass the variables in a bash command for the arguments:

```bash
provider "logzio" {
  api_token = var.api_token
  region= var.your_api_region
}
```
</div>


### Example - Create a new alert and a new Slack notification endpoint

Here's a great example demonstrating how easy it is to get up and running quickly with the Terraform Logz.io Provider.

This example adds a new Slack notification channel and creates a new alert in OpenSearch Dashboards that will send notifications to the newly-created Slack channel.

The alert in this example will trigger whenever Logz.io records 10 loglevel:ERROR messages in 10 minutes.

```
terraform {
  required_providers {
    logzio = {
      source = "logzio/logzio"
    }
  }
}

provider "logzio" {
  api_token = "8387abb8-4831-53af-91de-5cd3784d9774"
  region= "au"
}

resource "logzio_endpoint" "my_endpoint" {
  title = "my_endpoint"
  description = "my slack endpoint"
  endpoint_type = "slack"
  slack {
    url = "${var.slack_url}"
  }
}

resource "logzio_alert" "my_alert" {
  depends_on = [logzio_endpoint.my_endpoint]
  title = "my_other_title"
  query_string = "loglevel:ERROR"
  operation = "GREATER_THAN"
  notification_emails = []
  search_timeframe_minutes = 10
  value_aggregation_type = "NONE"
  alert_notification_endpoints = ["${logzio_endpoint.my_endpoint.id}"]
  suppress_notifications_minutes = 30
  severity_threshold_tiers {
      severity = "HIGH"
      threshold = 10
  }
}
```

### Example - Create user

This example will create a user in your Logz.io account.

```
terraform {
  required_providers {
    logzio = {
      source = "logzio/logzio"
    }
  }
}

variable "api_token" {
  type = "string"
  description = "Your logzio API token"
}

variable "account_id" {
  description = "The account ID where the new user will be created"
}

provider "logzio" {
  api_token = var.api_token
  region = var.region
}

resource "logzio_user" "my_user" {
  username = "user_name@fun.io"
  fullname = "John Doe"
  roles = [ 2 ]
  account_id = var.account_id
}
```

Run the above plan using the following bash script:

```
export TF_LOG=DEBUG
terraform init
TF_VAR_api_token=${LOGZIO_API_TOKEN} TF_VAR_region=${LOGZIO_REGION} terraform plan -out terraform.plan
terraform apply terraform.plan
```

Before you run the script, update the arguments to match your details.

### Import sub-accounts as resources 

You can import multiple sub-accounts as follows:

```
terraform import logzio_subaccount.logzio_sa_<ACCOUNT-NAME> <ACCOUNT-ID>
```
