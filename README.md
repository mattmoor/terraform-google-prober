# Cloud Run-based custom probers.

This repository contains a terraform module and Go library for deploying probers
that perform custom probing logic to Google Cloud.  The module packages a custom
Go prober as a container image, deploys it to Cloud Run, and then configures an
Uptime Check to periodically hit the Cloud Run URL.

## Defining a custom prober

With the little Go library provided here, a probe can be defined with as little
code as:

```go
import (
	"context"
	"log"

	"github.com/chainguard-dev/terraform-google-prober/pkg/prober"
)

func main() {
	prober.Go(context.Background(), prober.Func(func(ctx context.Context) error {
		log.Print("Got a probe!")
		return nil
	}))
}
```

> See our [basic example](./examples/basic/).

## Deploying a custom prober

With the terraform module provided here, a probe can be deployed with a little
configuration as:

```terraform
module "prober" {
  source  = "chainguard-dev/prober/google"
  version = "v0.1.2"

  name       = "basic-example"
  project_id = var.project_id

  importpath  = "github.com/chainguard-dev/terraform-google-prober/examples/basic"
  working_dir = path.module
}
```

> See our [basic example](./examples/basic/).

## Passing additional configuration

You can pass additional configuration to your custom probes via environment
variables passed to the prober application.  These can be specified in the
prober module:

```terraform
  env = {
    "FOO" : "bar"
  }
```

> See our [complex example](./examples/complex/).

## Multi-regional probers

By default, the probers run as a single-homed Cloud Run application, which is
great for development, and virtually free, but to take advantage of the
geographic distribution of GCP Uptime Checks, we need to deploy Cloud Run
applications to multiple regions behind a Google Cloud Load Balancer
(expensive!) with a TLS-terminated domain.

This can be done by specifying the following additional configuration:
```terraform

  # Deploy to three regions behind GCLB with a Google-managed
  # TLS certificate under the provided domain.
  locations = [
    "us-east1",
    "us-central1",
    "us-west1",
  ]

  # The domain under which we will provision hostnames
  domain   = var.domain

  # The Google Cloud DNS Zone to use for directing prober hostnames to the GCLB
  # IP address.
  dns_zone = google_dns_managed_zone.prober_zone.name
```

> See our [complex example](./examples/complex/).


<!-- BEGIN_TF_DOCS -->
## Requirements

No requirements.

## Providers

| Name | Version |
|------|---------|
| <a name="provider_google"></a> [google](#provider\_google) | n/a |
| <a name="provider_random"></a> [random](#provider\_random) | n/a |

## Modules

| Name | Source | Version |
|------|--------|---------|
| <a name="module_gclb"></a> [gclb](#module\_gclb) | chainguard-dev/common/infra//modules/serverless-gclb | 0.3.2 |
| <a name="module_this"></a> [this](#module\_this) | chainguard-dev/common/infra//modules/regional-go-service | 0.3.2 |

## Resources

| Name | Type |
|------|------|
| [google_monitoring_alert_policy.uptime_alert](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/monitoring_alert_policy) | resource |
| [google_monitoring_uptime_check_config.global_uptime_check](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/monitoring_uptime_check_config) | resource |
| [google_monitoring_uptime_check_config.regional_uptime_check](https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/monitoring_uptime_check_config) | resource |
| [random_password.secret](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/password) | resource |
| [google_cloud_run_v2_service.this](https://registry.terraform.io/providers/hashicorp/google/latest/docs/data-sources/cloud_run_v2_service) | data source |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| <a name="input_alert_description"></a> [alert\_description](#input\_alert\_description) | Alert documentation. Use this to link to playbooks or give additional context. | `string` | `"An uptime check has failed."` | no |
| <a name="input_cpu"></a> [cpu](#input\_cpu) | The CPU limit for the prober. | `string` | `"1000m"` | no |
| <a name="input_dns_zone"></a> [dns\_zone](#input\_dns\_zone) | The managed DNS zone in which to create prober record sets (required for multiple locations). | `string` | `""` | no |
| <a name="input_domain"></a> [domain](#input\_domain) | The domain of the environment to probe (required for multiple locations). | `string` | `""` | no |
| <a name="input_egress"></a> [egress](#input\_egress) | The level of egress the prober requires. | `string` | `"ALL_TRAFFIC"` | no |
| <a name="input_enable_alert"></a> [enable\_alert](#input\_enable\_alert) | If true, alert on failures. Outputs will return the alert ID for notification and dashboards. | `bool` | `false` | no |
| <a name="input_env"></a> [env](#input\_env) | A map of custom environment variables (e.g. key=value) | `map` | `{}` | no |
| <a name="input_importpath"></a> [importpath](#input\_importpath) | The import path that contains the prober application. | `string` | n/a | yes |
| <a name="input_memory"></a> [memory](#input\_memory) | The memory limit for the prober. | `string` | `"512Mi"` | no |
| <a name="input_name"></a> [name](#input\_name) | Name to prefix to created resources. | `string` | n/a | yes |
| <a name="input_notification_channels"></a> [notification\_channels](#input\_notification\_channels) | A list of notification channels to send alerts to. | `list(string)` | `[]` | no |
| <a name="input_period"></a> [period](#input\_period) | The period for the prober in seconds. | `string` | `"300s"` | no |
| <a name="input_project_id"></a> [project\_id](#input\_project\_id) | The project that will host the prober. | `string` | n/a | yes |
| <a name="input_regions"></a> [regions](#input\_regions) | A map from region names to a network and subnetwork.  A prober service will be created in each region. | <pre>map(object({<br>    network = string<br>    subnet  = string<br>  }))</pre> | n/a | yes |
| <a name="input_service_account"></a> [service\_account](#input\_service\_account) | The email address of the service account to run the service as. | `string` | n/a | yes |
| <a name="input_timeout"></a> [timeout](#input\_timeout) | The timeout for the prober in seconds. | `string` | `"60s"` | no |
| <a name="input_working_dir"></a> [working\_dir](#input\_working\_dir) | The working directory that contains the importpath. | `string` | n/a | yes |

## Outputs

| Name | Description |
|------|-------------|
| <a name="output_alert_id"></a> [alert\_id](#output\_alert\_id) | n/a |
| <a name="output_uptime_check"></a> [uptime\_check](#output\_uptime\_check) | n/a |
| <a name="output_uptime_check_name"></a> [uptime\_check\_name](#output\_uptime\_check\_name) | n/a |
<!-- END_TF_DOCS -->
