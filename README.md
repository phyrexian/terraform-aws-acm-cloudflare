This is a simple terraform .12 module that simplifies the creation of an ACM certificate with a validation rule on a Cloudflare account.

To use this module;  you'll need the following:

* An AWS account
  * an IAM user with [rights to create ACM resources](https://docs.aws.amazon.com/acm/latest/userguide/authen-apipermissions.html)
* A Cloudflare account
  * a Cloudflare API key with rights to create and update records

The following example manifest can be used to get started;  keep in mind that you may need to create ACM certificates in the `US-EAST-1` region to be supported for use by ELB / Cloudfront / etc.

You'll need to:

* Make sure that you can use the [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-configure.html)
* Fill in the Cloudflare provider values are updated
* If the zone already exists;  you'll need to `terraform import cloudflare_zone.primary_domain [zone_id]`

```
provider "aws" {
  region = "us-east-1"
}

resource "cloudflare_zone" "primary_domain" {
    zone = "domain.tld"
}

provider "cloudflare" {
  email = "" # OR USE ENV named $CLOUDFLARE_EMAIL
  token = "" # OR USE ENV named $CLOUDFLARE_API_KEY
}

module "site_certificates" {
  source      = "github.com/phyrexian/terraform-aws-acm-cloudflare"
  domain_name = "some-name.domain.tld"
  zone_id     = cloudflare_zone.primary_domain.id
}
```

This will:

* Create the Cloudflare zone (unless it was already imported)
* Create an ACM certificate request
* collect + create the "validation" in Cloudflare
* create a waiter that checks the DNS zone(s) for the valication status

From there;  the certificate(s) ARN's (`mod.site_certificates.this_acm_certificate_arn`) will become available to reference for the setup of any other resources that require them. 
