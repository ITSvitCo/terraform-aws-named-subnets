# terraform-aws-named-subnets [![Codefresh Build Status](https://g.codefresh.io/api/badges/pipeline/cloudposse/terraform-modules%2Fterraform-aws-named-subnets?type=cf-1)](https://g.codefresh.io/public/accounts/cloudposse/pipelines/5d2f63aa58ad25c1f8ee8e94) [![Latest Release](https://img.shields.io/github/release/cloudposse/terraform-aws-named-subnets.svg)](https://github.com/cloudposse/terraform-aws-named-subnets/releases/latest) [![Slack Community](https://slack.cloudposse.com/badge.svg)](https://slack.cloudposse.com)


Terraform module for named [`subnets`](http://docs.aws.amazon.com/AmazonVPC/latest/UserGuide/VPC_Subnets.html) provisioning.


## Usage


**IMPORTANT:** The `master` branch is used in `source` just as an example. In your code, do not pin to `master` because there may be breaking changes between releases.
Instead pin to the release tag (e.g. `?ref=tags/x.y.z`) of one of our [latest releases](https://github.com/cloudposse/terraform-aws-named-subnets/releases).



Simple example, with private and public subnets in one Availability Zone:

```hcl
module "vpc" {
  source     = "git::https://github.com/cloudposse/terraform-aws-vpc.git?ref=master"
  namespace  = "eg"
  name       = "vpc"
  stage      = "dev"
  cidr_block = var.cidr_block
}

locals {
  public_cidr_block  = cidrsubnet(module.vpc.vpc_cidr_block, 1, 0)
  private_cidr_block = cidrsubnet(module.vpc.vpc_cidr_block, 1, 1)
}

module "public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["web1", "web2", "web3"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.public_cidr_block
  type              = "public"
  igw_id            = module.vpc.igw_id
  availability_zone = "us-east-1a"
}

module "private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "database"
  subnet_names      = ["kafka", "cassandra", "zookeeper"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.private_cidr_block
  type              = "private"
  availability_zone = "us-east-1a"
  ngw_id            = module.public_subnets.ngw_id
}
```

Simple example, with `ENI` as default route gateway for private subnets

```hcl
resource "aws_network_interface" "default" {
  subnet_id         = module.us_east_1b_public_subnets.subnet_ids[0]
  source_dest_check = false
  tags              = module.network_interface_label.id
}

module "us_east_1b_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1b"
  eni_id            = aws_network_interface.default.id
  attributes        = ["us-east-1b"]
}
```

Full example, with private and public subnets in two Availability Zones for High Availability:

```hcl
module "vpc" {
  source     = "git::https://github.com/cloudposse/terraform-aws-vpc.git?ref=master"
  namespace  = "eg"
  name       = "vpc"
  stage      = "dev"
  cidr_block = var.cidr_block
}

locals {
  us_east_1a_public_cidr_block  = cidrsubnet(module.vpc.vpc_cidr_block, 2, 0)
  us_east_1a_private_cidr_block = cidrsubnet(module.vpc.vpc_cidr_block, 2, 1)
  us_east_1b_public_cidr_block  = cidrsubnet(module.vpc.vpc_cidr_block, 2, 2)
  us_east_1b_private_cidr_block = cidrsubnet(module.vpc.vpc_cidr_block, 2, 3)
}

module "us_east_1a_public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["apples", "oranges", "grapes"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1a_public_cidr_block
  type              = "public"
  igw_id            = module.vpc.igw_id
  availability_zone = "us-east-1a"
  attributes        = ["us-east-1a"]
}

module "us_east_1a_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1a_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1a"
  ngw_id            = module.us_east_1a_public_subnets.ngw_id
  attributes        = ["us-east-1a"]
}

module "us_east_1b_public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["apples", "oranges", "grapes"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_public_cidr_block
  type              = "public"
  igw_id            = module.vpc.igw_id
  availability_zone = "us-east-1b"
  attributes        = ["us-east-1b"]
}

module "us_east_1b_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1b"
  ngw_id            = module.us_east_1b_public_subnets.ngw_id
  attributes        = ["us-east-1b"]
}

resource "aws_network_interface" "default" {
  subnet_id         = module.us_east_1b_public_subnets.subnet_ids[0]
  source_dest_check = false
  tags              = module.network_interface_label.id
}

module "us_east_1b_private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["charlie", "echo", "bravo"]
  vpc_id            = module.vpc.vpc_id
  cidr_block        = local.us_east_1b_private_cidr_block
  type              = "private"
  availability_zone = "us-east-1b"
  eni_id            = aws_network_interface.default.id
  attributes        = ["us-east-1b"]
}
```

## Caveat
You must use only one type of device for a default route gateway per route table. `ENI` or `NGW`

Given the following configuration (see the Simple example above)

```hcl
locals {
  public_cidr_block  = cidrsubnet(var.vpc_cidr, 1, 0)
  private_cidr_block = cidrsubnet(var.vpc_cidr, 1, 1)
}

module "public_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "app"
  subnet_names      = ["web1", "web2", "web3"]
  vpc_id            = var.vpc_id
  cidr_block        = local.public_cidr_block
  type              = "public"
  availability_zone = "us-east-1a"
  igw_id            = var.igw_id
}

module "private_subnets" {
  source            = "git::https://github.com/cloudposse/terraform-aws-named-subnets.git?ref=master"
  namespace         = "eg"
  stage             = "dev"
  name              = "database"
  subnet_names      = ["kafka", "cassandra", "zookeeper"]
  vpc_id            = var.vpc_id
  cidr_block        = local.private_cidr_block
  type              = "private"
  availability_zone = "us-east-1a"
  ngw_id            = module.public_subnets.ngw_id
}

output "private_named_subnet_ids" {
  value = module.private_subnets.named_subnet_ids
}

output "public_named_subnet_ids" {
  value = module.public_subnets.named_subnet_ids
}
```

the output Maps of subnet names to subnet IDs look like these

```hcl
public_named_subnet_ids = {
  web1 = subnet-ea58d78e
  web2 = subnet-556ee131
  web3 = subnet-6f54db0b
}
private_named_subnet_ids = {
  cassandra = subnet-376de253
  kafka = subnet-9e53dcfa
  zookeeper = subnet-a86fe0cc
}
```

and the created subnet IDs could be found by the subnet names using `map["key"]` or [`lookup(map, key, [default])`](https://www.terraform.io/docs/configuration/interpolation.html#lookup-map-key-default-),

for example:

`public_named_subnet_ids["web1"]`

`lookup(private_named_subnet_ids, "kafka")`






## Makefile Targets
```
Available targets:

  help                                Help screen
  help/all                            Display help for all targets
  help/short                          This help short screen
  lint                                Lint terraform code

```
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| attributes | Additional attributes (e.g. `policy` or `role`) | list(string) | `<list>` | no |
| availability_zone | Availability Zone | string | - | yes |
| cidr_block | Base CIDR block which will be divided into subnet CIDR blocks (e.g. `10.0.0.0/16`) | string | - | yes |
| delimiter | Delimiter to be used between `name`, `namespace`, `stage`, `attributes` | string | `-` | no |
| enabled | Set to false to prevent the module from creating any resources | bool | `true` | no |
| eni_id | An ID of a network interface which is used as a default route in private route tables (_e.g._ `eni-9c26a123`) | string | `` | no |
| igw_id | Internet Gateway ID which will be used as a default route in public route tables (e.g. `igw-9c26a123`). Conflicts with `ngw_id` | string | `` | no |
| max_subnets | Maximum number of subnets which can be created. This variable is being used for CIDR blocks calculation. Defaults to length of `subnet_names` argument | number | `16` | no |
| name | Application or solution name | string | - | yes |
| namespace | Namespace (e.g. `eg` or `cp`) | string | `` | no |
| nat_enabled | Enable/disable NAT Gateway | bool | `true` | no |
| ngw_id | NAT Gateway ID which will be used as a default route in private route tables (e.g. `igw-9c26a123`). Conflicts with `igw_id` | string | `` | no |
| private_network_acl_egress | Private network egress ACL rules | object | `<list>` | no |
| private_network_acl_id | Network ACL ID that will be added to the subnets. If empty, a new ACL will be created | string | `` | no |
| private_network_acl_ingress | Private network ingress ACL rules | object | `<list>` | no |
| public_network_acl_egress | Public network egress ACL rules | object | `<list>` | no |
| public_network_acl_id | Network ACL ID that will be added to the subnets. If empty, a new ACL will be created | string | `` | no |
| public_network_acl_ingress | Public network ingress ACL rules | object | `<list>` | no |
| stage | Stage (e.g. `prod`, `dev`, `staging`) | string | `` | no |
| subnet_names | List of subnet names (e.g. `['apples', 'oranges', 'grapes']`) | list(string) | - | yes |
| tags | Additional tags (e.g. map(`BusinessUnit`,`XYZ`) | map(string) | `<map>` | no |
| type | Type of subnets (`private` or `public`) | string | `private` | no |
| vpc_id | VPC ID | string | - | yes |

## Outputs

| Name | Description |
|------|-------------|
| named_subnet_ids | Map of subnet names to subnet IDs |
| ngw_id | NAT Gateway ID |
| ngw_private_ip | Private IP address of the NAT Gateway |
| ngw_public_ip | Public IP address of the NAT Gateway |
| route_table_ids | Route table IDs |
| subnet_ids | Subnet IDs |




