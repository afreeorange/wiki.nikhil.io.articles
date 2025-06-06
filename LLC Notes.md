[TOC]

## Project Gld2-Zluhs

The idea was easy-peasy.

```
┌──────────────┐            ┌──────────────┐             ┌──────────────┐
│   Route53    │───────────▶│  CloudFront  │──────┬─────▶│      S3      │         /
└──────────────┘            └──────────────┘      │      └──────────────┘
                                                  │
                                                  │
                                                  │      ┌──────────────┐
                                                  └─────▶│    Lambda    │    /api, /rpc
                                                         └──────────────┘
```

### API Gateway

I did not know that you did _not_ need an API Gateway wrapper. APIG is convoluted as fuck. Also, the default timeout for any Lambda or BE it proxies is 30s/60s default/max. You can ask AWS to raise this limit. It's a PITA for simple use-cases and I wondered if I could just invoke the Lambda directly...

### You can in CloudFront

You assign a Lambda a Function URI, lop off the `https://` part, and can _paste that into the "Origin" field_ even if it doesn't show up in a list of targets (like S3, API-G)!

The issue is that, while you can set a timeout of 15 minutes for a Lambda, a CF origin has a max timeout of 29 seconds. Could be a problem with naive, big uploads as in my case. You can open an issue with AWS over this and raise the limit. I think you can go as high as 10 minutes. I requested 5 minutes.

### RDS

For a Lambda integration, you will need Security Group:Lambda -> Security Group:RDS set up. RDS integrations are very common and you can specify them in the Lamdba config itself (online or in Terraform).

### Lambda

I needed to connect to an external API (on the same domain). You'll see `UND_ERR_CONNECT_TIMEOUT` when your Lambda is unable to connect externally. This is [not a problem with `undici`](https://github.com/nodejs/undici/issues/1531).

👉 You can _only_ use _private subnets via NAT Gateways_ to connect _externally_! Public Subnets that use Internet Gateways (IGs) won't cut it!

## Other

### Public Bucket Policy

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::some-bucket/*"
    }
  ]
}
```
