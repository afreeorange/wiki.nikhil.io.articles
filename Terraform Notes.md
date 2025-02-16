I use Terraform v1.10 and the AWS Provider v5 to manage a few websites et al. Some notes.

### Miscellaneous

Plan ahead and _do things incrementally_. Don't "Big Bang" and attempt to change large portions of the state of the world at once. Terraform tries its best to help but it's not a freaking genie.

CloudFront Updates: These take a _loooong_ time (sometimes >15 mins.) Be patient and dont <kbd>Ctrl</kbd>+<kbd>c</kbd>!

Claude.ai is better than most at migrating from old to new things.

### Route53 and Nameservers

Route53 is two things: a Domain Registrar and a DNS Host. _The Nameservers must match_! Else you'll get the DNS resolution errors one would reasonably expect with this kind of misconfiguration.

Get what the "DNS Host" part of Route53 thinks they are with this

    aws route53 get-hosted-zone --id ZUYTRTR6MIDU2W

And then copy those over to the "Domain Registrar" part (Domains &rarr; Registered Domains.)

### UI and UX

The official Terraform plugins are great.

There's really no UI but there's [Terraform Enterprise](https://app.terraform.io) which is more than sufficient for personal use and [Terrakube](https://terrakube.org/) as an open-source alternative (it's a pretty involved installation, even with Docker).

### Modules

* Wondered what the "name" should be when defining a module resource. Looks like the world uses `this`:

    module.cloudfront_example_com.aws_cloudfront_distribution.this

* Outputs are like `module.<MODULE NAME>.<OUTPUT NAME>`
* When redefining resources with reusable modules, had to

```bash
# Remove the entry in state
terraform state rm aws_cloudfront_distribution.example_com

# Adopt the resource with its module name. NOTE that CloudFront imports
# don't take the full ARN, just the distribution ID!
terraform import module.cloudfront_example_com.aws_cloudfront_distribution.this <ARN>
```

### Some Commands

```bash
# Validate your configuration
terraform validate

# See all warnings. You don't get to see all of them unless you use the `-json`
# flag for some mystifying reason. So pipe to `jq` and enjoy the list.
terraform validate -json | jq '.diagnostics[] | {detail: .detail, filename: .range.filename, start_line: .range.start.line}'

# Import an existing resource
terraform import module.bucket_example_com.aws_s3_bucket.this docker.nikhil.io
```

### ACM and Route53 &rarr; Same CloudFront Distribution

Added two new domains which, for now, I wanted to point at the same CloudFront distribution. This was a fucking nightmare because I tried to do too much at once. Needed a brand new certificate that included the domains and their wildcarded subdomains (`*.blah.com`.) Terraform recommends `DNS` as the verification method but this was _way_ more irksome and frustrating than I thought it would be. Just used `EMAIL` validation and checked my inbox. Some notes

* You cannot manually create a cert in the console and import it via `terraform import`. See [this issue](https://github.com/terraform-providers/terraform-provider-aws/issues/3560).
* You cannot modify [the `validation_emails` property](https://www.terraform.io/docs/providers/aws/r/acm_certificate.html#validation_emails).

Here's what an attempted config looked like. Assume we're dealing with `example.com` and `example.net`.

```terraform
resource "aws_route53_record" "example_com_cert" {
  name    = aws_acm_certificate.example_net_bundle.domain_validation_options.0.resource_record_name
  type    = aws_acm_certificate.example_net_bundle.domain_validation_options.0.resource_record_type
  zone_id = aws_route53_zone.example_com.zone_id
  ttl     = 60
  records = [
    aws_acm_certificate.example_net_bundle.domain_validation_options.0.resource_record_value
  ]
}
```

### New Domains

The _giant_ PITAs are:

- You absolutely need to make sure that the nameservers _assigned to you by Route53_ (in the "Hosted Zones" section) are the ones you use at the Registrar! So when Terraform creates an HZ, you will see `NS` records created/assigned to you. Do not fuck with them.
- You will need ACM to get new certificates for the CloudFront distribution
- You will need SES to work to approve certificate requests sent to `webmaster@newdomain.tld`

There's a dependency chain here that you could make better with the scripts.

### Resources and References

* Use [GraphViz](https://dreampuf.github.io/GraphvizOnline) online to see pretty dependency graphs
* [How to rename things](https://www.terraform.io/docs/commands/state/mv.html).
