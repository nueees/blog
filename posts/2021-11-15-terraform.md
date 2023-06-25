---
aliases:
- /aws/terraform/2021/11/15/terraform
categories:
- aws
- terraform
date: '2021-11-15'
description: section1
layout: post
title: Terraform
toc: true

---

Terraform with AWS

---

# 1. DevOps

<br><br>
---
# 2. Terraform

## 2.1. 구성요소
- resource : 실제로 생성할 인프라 자원  
ex) aws_security_group, aws_lb, aws_instance  

- provider : Terraform으로 정의할 Infrastructure Provider  
[각종 providers](https://www.terraform.io/docs/providers/index.html)  

- output : 인프라를 provisioning 한 후에 생성된 자원을 스크립트로 추출  
추출한 script는 이후에 remote state에서 활용 가능  

- backend : terraform의 상태를 저장하는 공간(default local)  
현재 배포된 최신 상태를 외부에 저장하기 때문에 다른 사람과의 협업이 가능 (S3...등)  

- module : 공통적으로 활용할 수 있는 인프라 코드를 한 곳으로 모아서 정의한 함수  
변수만 바꿔서 동일한 리소스를 손쉽게 생성 가능  

- remote state : remote state를 사용하면 VPC, IAM 등과 같은 공용 서비스를 다른 서비스에서 참조 가능, tfstate파일(최신 테라폼 상태정보)이 저장되어 있는 backend 정보를 명시하면, terraform이 해당 backend에서 output 정보들을 가져옴  


## 2.2. 작동 원리

### Form  
- Local 코드 : 현재 개발자가 작성/수정하고 있는 코드  
- AWS 실제 인프라 : 실제로 AWS에 배포되어 있는 인프라  
- Backend에 저장된 상태 : 가장 최근에 배포한 테라폼 코드(tfstate파일) 형상  

- Terraform init
지정한 backend에 상태 저장을 위한 .tfstate 파일을 생성되고 가장 마지막에 적용한 테라폼 내역이 저장  
init 작업을 완료하면, local에는 .tfstate에 정의된 내용을 담은 .terraform 파일이 생성  
기존에 다른 개발자가 이미 .tfstate에 인프라를 정의해 놓은 것이 있다면, 다른 개발자는 init작업을 통해서 local에 sync를 맞출 수 O  

- Terraform plan
정의한 코드가 어떤 인프라를 만들게 되는지 미리 예측 결과를 보여줍니다. 단, plan을 한 내용에 에러가 없다고 하더라도, 실제 적용되었을 때는 에러가 발생 가  
Plan 명령어는 어떠한 형상에도 변화 X  


- Terraform apply
실제로 인프라를 배포하기 위한 명령어입니다. apply를 완료하면, AWS 상에 실제로 해당 인프라가 생성되고 작업 결과가 backend의 .tfstate 파일에 저장  
해당 결과는 local의 .terraform 파일에도 저장  


- Terraform import
AWS 인프라에 배포된 리소스를 terraform state로 옮겨주는 작업  
이는 local의 .terraform에 해당 리소스의 상태 정보를 저장해주는 역할 (절대 코드를 생성해주지 X)  
Apply 전까지는 backend에 저장되지 X  
Import 이후에 plan을 하면 로컬에 해당 코드가 없기 때문에 리소스가 삭제 또는 변경된다는 결과를 display. 이 결과를 바탕으로 코드를 작성 O  





## 2.3. 구동

### provider 생성  
```
] cat providers.tf 
provider "aws" { 
	region = "eu-central-1" 
}
```

```
] cat providers.tf # gcp용
provider "google" {}
```

### init  
```
] terraform init
Initializing the backend...
Terraform has been successfully initialized!
```

### resource 생성  
```
] cat s3.tf
resource "aws_s3_bucket" "test" {
	bucket = "terraform220222"
}
```

```
] cat main.tf # gcp용
variable "instance_name" {}
variable "instance_zone" {}
variable "instance_type" {
  default = "n1-standard-1"
  }
variable "instance_network" {}

resource google_compute_instance "vm_instance" {
  name = "${var.instance_name}"
  #RESOURCE properties go here
  zone         = "${var.instance_zone}"
  machine_type = "${var.instance_type}"
  boot_disk {
    initialize_params {
      image = "debian-cloud/debian-9"
      }
  }
  network_interface {
    network = "${var.instance_network}"
    access_config {
      # Allocate a one-to-one NAT IP to the instance
    }
  }
}
```

### plan  
```
] terraform plan
Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.test will be created
  + resource "aws_s3_bucket" "test" {
      + acceleration_status                  = (known after apply)
      + acl                                  = (known after apply)
      + arn                                  = (known after apply)
      + bucket                               = "terraform220222"
      + bucket_domain_name                   = (known after apply)
      + bucket_regional_domain_name          = (known after apply)
      + cors_rule                            = (known after apply)
      + force_destroy                        = false
      + grant                                = (known after apply)
      + hosted_zone_id                       = (known after apply)
      + id                                   = (known after apply)
      + lifecycle_rule                       = (known after apply)
      + logging                              = (known after apply)
      + policy                               = (known after apply)
      + region                               = (known after apply)
      + replication_configuration            = (known after apply)
      + request_payer                        = (known after apply)
      + server_side_encryption_configuration = (known after apply)
      + tags_all                             = (known after apply)
      + versioning                           = (known after apply)
      + website                              = (known after apply)
      + website_domain                       = (known after apply)
      + website_endpoint                     = (known after apply)

      + object_lock_configuration {
          + object_lock_enabled = (known after apply)
          + rule                = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```


```
] terraform plan # gcp용
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_compute_firewall.mynetwork-allow-http-ssh-rdp-icmp will be created
  + resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
      + creation_timestamp = (known after apply)
      + destination_ranges = (known after apply)
      + direction          = (known after apply)
      + enable_logging     = (known after apply)
      + id                 = (known after apply)
      + name               = "mynetwork-allow-http-ssh-rdp-icmp"
      + network            = (known after apply)
      + priority           = 1000
      + project            = (known after apply)
      + self_link          = (known after apply)
      + source_ranges      = [
          + "0.0.0.0/0",
        ]

      + allow {
          + ports    = [
              + "22",
              + "80",
              + "3389",
            ]
          + protocol = "tcp"
        }
      + allow {
          + ports    = []
          + protocol = "icmp"
        }
    }

  # google_compute_network.mynetwork will be created
  + resource "google_compute_network" "mynetwork" {
      + auto_create_subnetworks         = true
      + delete_default_routes_on_create = false
      + gateway_ipv4                    = (known after apply)
      + id                              = (known after apply)
      + mtu                             = (known after apply)
      + name                            = "mynetwork"
      + project                         = (known after apply)
      + routing_mode                    = (known after apply)
      + self_link                       = (known after apply)
    }

  # module.mynet-eu-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "n1-standard-1"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "mynet-eu-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "europe-west1-d"

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "debian-cloud/debian-9"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart   = (known after apply)
          + min_node_cpus       = (known after apply)
          + on_host_maintenance = (known after apply)
          + preemptible         = (known after apply)
          + provisioning_model  = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

  # module.mynet-us-vm.google_compute_instance.vm_instance will be created
  + resource "google_compute_instance" "vm_instance" {
      + can_ip_forward       = false
      + cpu_platform         = (known after apply)
      + current_status       = (known after apply)
      + deletion_protection  = false
      + guest_accelerator    = (known after apply)
      + id                   = (known after apply)
      + instance_id          = (known after apply)
      + label_fingerprint    = (known after apply)
      + machine_type         = "n1-standard-1"
      + metadata_fingerprint = (known after apply)
      + min_cpu_platform     = (known after apply)
      + name                 = "mynet-us-vm"
      + project              = (known after apply)
      + self_link            = (known after apply)
      + tags_fingerprint     = (known after apply)
      + zone                 = "us-central1-a"

      + boot_disk {
          + auto_delete                = true
          + device_name                = (known after apply)
          + disk_encryption_key_sha256 = (known after apply)
          + kms_key_self_link          = (known after apply)
          + mode                       = "READ_WRITE"
          + source                     = (known after apply)

          + initialize_params {
              + image  = "debian-cloud/debian-9"
              + labels = (known after apply)
              + size   = (known after apply)
              + type   = (known after apply)
            }
        }

      + confidential_instance_config {
          + enable_confidential_compute = (known after apply)
        }

      + network_interface {
          + ipv6_access_type   = (known after apply)
          + name               = (known after apply)
          + network            = (known after apply)
          + network_ip         = (known after apply)
          + stack_type         = (known after apply)
          + subnetwork         = (known after apply)
          + subnetwork_project = (known after apply)

          + access_config {
              + nat_ip       = (known after apply)
              + network_tier = (known after apply)
            }
        }

      + reservation_affinity {
          + type = (known after apply)

          + specific_reservation {
              + key    = (known after apply)
              + values = (known after apply)
            }
        }

      + scheduling {
          + automatic_restart   = (known after apply)
          + min_node_cpus       = (known after apply)
          + on_host_maintenance = (known after apply)
          + preemptible         = (known after apply)
          + provisioning_model  = (known after apply)

          + node_affinities {
              + key      = (known after apply)
              + operator = (known after apply)
              + values   = (known after apply)
            }
        }
    }

Plan: 4 to add, 0 to change, 0 to destroy.
```


### apply  
```
] terraform apply

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_s3_bucket.test will be created
  + resource "aws_s3_bucket" "test" {
      + acceleration_status                  = (known after apply)
      + acl                                  = (known after apply)
      + arn                                  = (known after apply)
      + bucket                               = "terraform220222"
      + bucket_domain_name                   = (known after apply)
      + bucket_regional_domain_name          = (known after apply)
      + cors_rule                            = (known after apply)
      + force_destroy                        = false
      + grant                                = (known after apply)
      + hosted_zone_id                       = (known after apply)
      + id                                   = (known after apply)
      + lifecycle_rule                       = (known after apply)
      + logging                              = (known after apply)
      + policy                               = (known after apply)
      + region                               = (known after apply)
      + replication_configuration            = (known after apply)
      + request_payer                        = (known after apply)
      + server_side_encryption_configuration = (known after apply)
      + tags_all                             = (known after apply)
      + versioning                           = (known after apply)
      + website                              = (known after apply)
      + website_domain                       = (known after apply)
      + website_endpoint                     = (known after apply)

      + object_lock_configuration {
          + object_lock_enabled = (known after apply)
          + rule                = (known after apply)
        }
    }

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.
Enter a value: yes
aws_s3_bucket.test: Creating...
aws_s3_bucket.test: Creation complete after 3s [id=terraform220222]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

terraform.tfstate 파일이 생성된 거 확인 가능  

```
] aws s3 ls  # s3 생성된 거 확인
2022-02-22 16:22:24 terraform220222
```

### import 준비  

이전에 먼저 만든 s3 리소스 tf 파일(s3.tf) 날리고 시작
```
] rm s3.tf
] terraform plan
aws_s3_bucket.test: Refreshing state... [id=terraform220222]

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the
last "terraform apply":

  # aws_s3_bucket.test has changed
  ~ resource "aws_s3_bucket" "test" {
        id                                   = "terraform220222"
      + tags                                 = {}
        # (18 unchanged attributes hidden)
    }


Unless you have made equivalent changes to your configuration, or ignored the
relevant attributes using ignore_changes, the following plan may include
actions to undo or respond to these changes.

───────────────────────────────────────────────────────────────────────────────

Terraform used the selected providers to generate the following execution plan.
Resource actions are indicated with the following symbols:
  - destroy # 삭제된거 확인

Terraform will perform the following actions:

  # aws_s3_bucket.test will be destroyed # s3 삭제할거라 함
  # (because aws_s3_bucket.test is not in configuration)
  - resource "aws_s3_bucket" "test" {
      - acl                                  = "private" -> null
      - arn                                  = "arn:aws:s3:::terraform220222" -> null
      - bucket                               = "terraform220222" -> null
      - bucket_domain_name                   = "terraform220222.s3.amazonaws.com" -> null
      - bucket_regional_domain_name          = "terraform220222.s3.eu-central-1.amazonaws.com" -> null
      - cors_rule                            = [] -> null
      - force_destroy                        = false -> null
      - grant                                = [] -> null
      - hosted_zone_id                       = "Z21DNDUVLTQW6Q" -> null
      - id                                   = "terraform220222" -> null
      - lifecycle_rule                       = [] -> null
      - logging                              = [] -> null
      - region                               = "eu-central-1" -> null
      - replication_configuration            = [] -> null
      - request_payer                        = "BucketOwner" -> null
      - server_side_encryption_configuration = [] -> null
      - tags                                 = {} -> null
      - tags_all                             = {} -> null
      - versioning                           = [
          - {
              - enabled    = false
              - mfa_delete = false
            },
        ] -> null
      - website                              = [] -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.
```
state상으로는 s3가 남아있으나 실제는 destroy되었기 때문에,  
Objects have changed라는 메시지와 함께 삭제할거라 함  


state파일도 지우고 terraform init 상태로 만들어서 plan 확인
```
] rm -f terraform.tfstate
] terraform init
] terraform plan
No changes. Your infrastructure matches the configuration.
Terraform has compared your real infrastructure against your configuration and
found no differences, so no changes are needed.
```
아무것도 안만들어진 상태로 나옴

여기서 부터 기존 리소스 import 시작

### 실제 import
```
] terraform import aws_s3_bucket.test terraform220222 # 코드 공식문서 참조
Error: resource address "aws_s3_bucket.test" does not exist in the configuration.

Before importing this resource, please create its configuration in the root module. For example:
resource "aws_s3_bucket" "test" {
  # (resource arguments)
}
```

위에 에러 가이드 대로, resource 파일(s3.tf) 재생성 후 import  
```
] terraform import aws_s3_bucket.test terraform220222 # 다시 import
aws_s3_bucket.test: Importing from ID "terraform220222"...
aws_s3_bucket.test: Import prepared!
  Prepared aws_s3_bucket for import
aws_s3_bucket.test: Refreshing state... [id=terraform220222]

Import successful!

The resources that were imported are shown above. These resources are now in
your Terraform state and will henceforth be managed by Terraform.
```

```
]  terraform plan
aws_s3_bucket.test: Refreshing state... [id=terraform220222]
No changes. Your infrastructure matches the configuration.

] terraform apply
aws_s3_bucket.test: Refreshing state... [id=terraform220222]
No changes. Your infrastructure matches the configuration.
Terraform has compared your real infrastructure against your configuration and
found no differences, so no changes are needed.
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

] terraform state list
aws_s3_bucket.test
```


<br><br>
---
# 3. VPC resource 생성

## 3.1. VPC configure

```
] cat vpc_test.tf
provider "aws" {
  region  = "ap-northeast-2"
}

resource "aws_vpc" "main" {
  cidr_block       = "10.0.0.0/16"

  tags = {
    Name = "terraform220222"
  }
}
}
```


## 3.2. Subnet configure
```
] cat subnet.tf
provider "aws" {
  region  = "ap-northeast-2"
}

resource "aws_vpc" "main" {
  cidr_block       = "10.0.0.0/16"

  tags = {
    Name = "terraform220222"
  }
}

resource "aws_subnet" "first_subnet" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.1.0/24"

  availability_zone = "ap-northeast-2a"

  tags = {
    Name = "220222subnet-1"
  }
}


resource "aws_subnet" "second_subnet" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.2.0/24"

  availability_zone = "ap-northeast-2b"

  tags = {
    Name = "220222subnet-2"
  }
}
```


## 3.3. Internet Gateway configure
```
] cat internet_gateway.tf
resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main"
  }
}
```



## 3.4. Route Table configure
```
] cat route_table.tf
resource "aws_route_table" "route_table" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main"
  }
}

resource "aws_route_table_association" "route_table_association_1" {
  subnet_id      = aws_subnet.first_subnet.id
  route_table_id = aws_route_table.route_table.id
}

resource "aws_route_table_association" "route_table_association_2" {
  subnet_id      = aws_subnet.second_subnet.id
  route_table_id = aws_route_table.route_table.id
}
```



## 3.5. Private Subnet and NAT configure
```
] cat private_subnet_nat.tf
resource "aws_subnet" "first_private_subnet" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.3.0/24"

  availability_zone = "ap-northeast-2a"

  tags = {
    Name = "220222subnet-private-1"
  }
}

resource "aws_subnet" "second_private_subnet" {
  vpc_id     = aws_vpc.main.id
  cidr_block = "10.0.4.0/24"

  availability_zone = "ap-northeast-2b"

  tags = {
    Name = "220222subnet-private-2"
  }
}
```

### 3.6. AWS Elastic IP도 함께 생성  
Nat Gateway는 Public 서브넷에 위치하지만 private 서브넷과 연결  

```
resource "aws_eip" "nat_1" {
  vpc   = true

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_eip" "nat_2" {
  vpc   = true

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_nat_gateway" "nat_gateway_1" {
  allocation_id = aws_eip.nat_1.id

  # Private subnet이 아니라 public subnet을 연결
  subnet_id = aws_subnet.first_subnet.id

  tags = {
    Name = "NAT-GW-1"
  }
}

resource "aws_nat_gateway" "nat_gateway_2" {
  allocation_id = aws_eip.nat_2.id

  subnet_id = aws_subnet.second_subnet.id

  tags = {
    Name = "NAT-GW-2"
  }
}
```

### 3.7. Private 서브넷에도 Route Table을 생성하여 연결  
```
resource "aws_route_table" "route_table_private_1" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-private-1"
  }
}

resource "aws_route_table" "route_table_private_2" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main-private-2"
  }
}

resource "aws_route_table_association" "route_table_association_private_1" {
  subnet_id      = aws_subnet.first_private_subnet.id
  route_table_id = aws_route_table.route_table_private_1.id
}

resource "aws_route_table_association" "route_table_association_private_2" {
  subnet_id      = aws_subnet.second_private_subnet.id
  route_table_id = aws_route_table.route_table_private_2.id
}

resource "aws_route" "private_nat_1" {
  route_table_id              = aws_route_table.route_table_private_1.id
  destination_cidr_block      = "0.0.0.0/0"
  nat_gateway_id              = aws_nat_gateway.nat_gateway_1.id
}

resource "aws_route" "private_nat_2" {
  route_table_id              = aws_route_table.route_table_private_2.id
  destination_cidr_block      = "0.0.0.0/0"
  nat_gateway_id              = aws_nat_gateway.nat_gateway_2.id
}
``` 


```
] cat mynetwork.tf # gcp용
# Create the mynetwork network
resource "google_compute_network" "mynetwork" {
  name = "mynetwork"
  #RESOURCE properties go here
  auto_create_subnetworks = "true"
}
# Add a firewall rule to allow HTTP, SSH, RDP and ICMP traffic on mynetwork
resource "google_compute_firewall" "mynetwork-allow-http-ssh-rdp-icmp" {
  name = "mynetwork-allow-http-ssh-rdp-icmp"
  #RESOURCE properties go here
  network = google_compute_network.mynetwork.self_link
  allow {
    protocol = "tcp"
    ports    = ["22", "80", "3389"]
  }
  allow {
    protocol = "icmp"
  }
  source_ranges = ["0.0.0.0/0"]
}
# Create the mynet-us-vm instance
module "mynet-us-vm" {
  source           = "./instance"
  instance_name    = "mynet-us-vm"
  instance_zone    = "us-central1-a"
  instance_network = google_compute_network.mynetwork.self_link
}
# Create the mynet-eu-vm" instance
module "mynet-eu-vm" {
  source           = "./instance"
  instance_name    = "mynet-eu-vm"
  instance_zone    = "europe-west1-d"
  instance_network = google_compute_network.mynetwork.self_link
}
```


<br><br>
---
# 4. S3 resource 생성

## 4.1. S3 configure


<br><br>
---
# 5. Terraform Advancement


<br><br>
---
