# TERRAFORM



## What's Terraform

* A tool for managing infrastructure
* A product from Hashicorp
* Written in Go
* Single binary
* Current version 0.12.19 (08/01/2020)
* Actively developed
* +20.000 stars on Github


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

* Required config ([docs](https://www.terraform.io/docs/providers/aws/index.html))
* Access and secret key from **~/.aws/credentials**
* Assume role or Profile
* Fix version ([repo tags](https://github.com/terraform-providers/terraform-provider-aws/releases))

```
provider "aws" {
  version = "2.44.0"
  region  = "eu-west-1"
}
```


## Provider vSphere

* Required parameters ([docs](https://www.terraform.io/docs/providers/vsphere/index.html))
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
  vpc_id     = aws_vpc.main.id
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
    security_groups = [aws_security_group.admin.id]
  }
}

resource "aws_security_group" "admin" {
  ingress {
    ...
    security_groups = [aws_security_group.default.id]
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

  depends_on = [aws_s3_bucket.example]
}
```



## Interpolation

* Attributes of resources
```
aws_vpc.main.id
```
* Variables
```
var.foo
```
* Functions
```
jsonencode(value)
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
  value = aws_eip.ip.public_ip
}
```



## Using modules

* Local source or git repo (use tags)
* Customized vars
* Loaded by **terraform get**

```
module "vpc" {
  source = "./modules/vpc"

  name = "my_vpc"
}
```



## Generalized Type System


### 0.11 Types

* Strings
* Lists of strings (not as module param)
* Maps of strings (not as module param)


### Primitive types

* String
* Number
  * Whole numbers like 15
  * fractional values such as 6.283185
* Bool
  * either true or false
  * can be used in conditional logic


### Conversion of Primitive Types

* **true** converts to **"true"**, and vice-versa
* **false** converts to **"false"**, and vice-versa
* **15** converts to **"15"**, and vice-versa 


### Collection Types

* List
  * sequence of values with number index
* Map
  * collection of values each identified by string label
* Set
  * unique values without any label or ordering


### Type Constraints

```
variable "network" {
  type = string
  default = "Lan_VirtualMachine"
}

variable "netmask" {
  type = number
  default = 16
}

variable "nameserver" {
  type    = list(string)
  default = ["128.1.233.215", "128.1.233.216"]
}
```


### Rich Types

* Modules input/output:
  * Complex types (map, list, etc...)
  * Whole resource (all attributes)



## Iteration constructs


### For each

```
resource "aws_subnet" "subnet" {
  for_each = {
    subnet_a = "192.168.0.0/24"
    subnet_b = "192.168.1.0/24"
  }

  vpc_id     = aws_vpc.main.id
  cidr_block = each.value

  tags = {
    Name = each.key
  }
}

```


### Dynamic Nested Blocks 

```
variable "ingress_ports" {
  type        = list(number)
  default     = [8200, 8201]
}

resource "aws_security_group" "vault" {
  name        = "vault"

  dynamic "ingress" {
    iterator = port
    for_each = var.ingress_ports
    content {
      from_port   = port.value
      to_port     = port.value
    }
  }
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
  id = var.vpc_id
}

resource "aws_subnet" "example" {
  vpc_id = data.aws_vpc.selected.id
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
  subnet_id = data.terraform_remote_state.vpc.subnet_id
}
```



## Atlantis

https://www.runatlantis.io/