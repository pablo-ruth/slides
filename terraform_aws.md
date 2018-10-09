# TERRAFORM



## What's Terraform

* A tool for managing infrastructure
* A product from Hashicorp
* Written in Go
* Single binary
* Current version 0.11.8 (16/08/2018)
* Actively developed
* +13.000 stars on Github


## Why Terraform

* Manage infrastructure lifecycle
* Cloud providers polyglot
* IaC vs Configuration Management
* Infrastructure as Code
  * Collaborative (versioning)
  * Automation friendly



## Simple workflow

* Write (describe infra)
* Plan (review changes)
* Apply (idempotent changes)



## Files

```
$ tree example-module/
.
+-- main.tf
+-- vars.tf
+-- outputs.tf
+
+-- modules/
|   +-- nestedA/
|   |   +-- vars.tf
|   |   +-- main.tf
|   |   +-- outputs.tf
```



## Providers

* Core
  * reading and interpolating config files 
  * state management
  * communication with plugins
* Provider
  * logical abstraction of an upstream API
  * specific by service (AWS, vSphere, ...)


## Provider AWS

* Access and secret key from **~/.aws/credentials**
* Assume role or Profile
* Fix version (best practice)

```
provider "aws" {
  version = "~> 1.0.0"
  region     = "eu-west-1"
}
```


## Provider vSphere

* Required parameters ([docs](https://www.terraform.io/docs/providers/vsphere/index.html  ))
* Fix version ([repo tags](https://github.com/terraform-providers/terraform-provider-vsphere))

```
provider "vsphere" {
  version = "v1.8.1"
  
  vsphere_server = "https://vsphere.local"
  user = "vsphereuser"
  password = "vspherepass"
}
```



## Init

* Run in a new or cloned terraform repo
* Initialize backend (state storage)
* Download plugins (providers)

```
$ terraform init
```



## Define resource

* Resource type
* Internal name

```
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "a" {
  vpc_id     = "${aws_vpc.main.id}"
  cidr_block = "10.0.1.0/24"
}
```



## Plan

* Refresh remote state
* Generate dependency graph
* Display plan summary

```
$ terraform plan
```



## Apply

* Create/update/destroy resources
* Parallelize or serialize changes
* Update state

```
$ terraform apply
```



## Destroy

* Destroy all terraform managed resources
* Can be targeted to resource 

```
$ terraform destroy -target=aws_subnet.a
```



## Dependency


## Implicit

* Dependency by interpolation
* Be careful of cyclic dependencies

```
resource "aws_security_group" "default" {
  ingress {
    ...
    security_groups = ["${aws_security_group.admin.id}"]
  }
}

resource "aws_security_group" "admin" {
  ingress {
    ...
    security_groups = ["${aws_security_group.default.id}"]
  }
}
```


## Explicit

* Using *depends_on* argument

```
resource "aws_s3_bucket" "example" {
  bucket = "terraform_getting_started_guide"
}

resource "aws_instance" "example" {
  ami           = "ami-2757f631"
  instance_type = "t2.micro"

  depends_on = ["aws_s3_bucket.example"]
}
```



## Interpolation

* Attributes of resources
```
${aws_vpc.main.id}
```
* Variables
```
${var.foo}
```
* Functions
```
${jsonencode(value)}
```



## Input variables

* Must be defined AND assigned
* Defined in *vars.tf* file
```
variable "myip" {}
```
* Assigned in *terraform.tfvars*
```
myip = "192.168.0.10"
```



## Output variables

* Needed to expose module vars
* Defined in *outputs.tf*
```
output "ip" {
  value = "${aws_eip.ip.public_ip}"
}
```



## Using modules

* Local source or git repo
* Customized vars
* Loaded by **terraform get**

```
module "vpc" {
  source = "./modules/vpc"

  name = "my_vpc"
}
```



## Remote state

* State is local by default
* Remote backend for shared state
* State locking to avoid inconsistency


## S3 / Dynamodb state

* S3 allow backup and versionning
* DynamoDB for state locking

```
terraform {
  backend "s3" {
    bucket         = "terraform-bucket"
    key            = "myproject/terraform.tfstate"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "terraform-table"
  }
}
```



## Data sources


## Existing resources

```
data "aws_vpc" "selected" {
  id = "${var.vpc_id}"
}

resource "aws_subnet" "example" {
  vpc_id            = "${data.aws_vpc.selected.id}"
  ...
}
```


## Remote state

```
data "terraform_remote_state" "vpc" {
  backend = "atlas"

  config {
    name = "hashicorp/vpc-prod"
  }
}

resource "aws_instance" "foo" {
  subnet_id = "${data.terraform_remote_state.vpc.subnet_id}"
}
```



## Atlantis

https://www.runatlantis.io/