# Terraform 0.12



## Features

* First-class expression syntax
* Generalized type system
* Iteration constructs
* Template syntax


## Requirements

* Terraform >=0.12 binary
* Updated providers



##  First-class expressions


### Variable

```
# TF 0.11

data "itop_organization" "org" {
  name = "${var.organization}"
}
```

```
# TF 0.12

data "itop_organization" "org" {
  name = var.organization
}
```


### Ternary operator

```
# TF 0.11

resource "itop_virtual_machine" "vm" {
  backup_id = "${var.backup ? data.itop_backup.rubrik.id : "0"}"
}
```

```
# TF 0.12

resource "itop_virtual_machine" "vm" {
  backup_id = var.backup ? data.itop_backup.rubrik.id : "0"
}
```


### Array

```
# TF 0.11

resource "disk" "test" {
  thin_provisioned = "${data.vm.template.disks.0.thin_provisioned}"
}
```

```
# TF 0.12

resource "disk" "test" {
  thin_provisioned = data.vm.template.disks[0].thin_provisioned
}
```


### Function

```
# TF 0.11

datacenter = "${substr(var.name, 1, 1)}"
```

```
# TF 0.12

datacenter = substr(var.name, 1, 1)
```


### Format

```
# TF 0.11

interfaces {
  dns  = "${var.name}.${var.domain}"
}
```

```
# TF 0.12

interfaces {
  dns  = format("%s.%s", var.name, var.domain)
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



## Template syntax


### Loop

```
locals {
  lb_config = <<EOT
%{ for instance in opc_compute_instance.example ~}
server ${instance.label} ${instance.ip_address}:8080
%{ endfor }
EOT
}
```
```
server example0 192.168.2.12:8080
server example1 192.168.2.65:8080
server example2 192.168.2.23:8080
```


## Condition

```
output "just_mary" {
  value = <<EOT
%{ for name in var.names ~}
%{ if name == "Mary" }${name}%{ endif ~}
%{ endfor ~}
EOT
}
```