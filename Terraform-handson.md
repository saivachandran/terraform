#install Terraform
===================

wget https://releases.hashicorp.com/terraform/1.0.9/terraform_1.0.9_linux_amd64.zip


unzip terraform_1.0.9_linux_amd64.zip

mv ~/terraform /usr/local/bin/


terraform --help

------------------------------------------------------------------

## Deploy docker conatiner using terraform

create diectory

$ mkdir learn-terraform-docker-container

$ cd learn-terraform-docker-container

$ vim main.tf

terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 2.13.0"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx:latest"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.latest
  name  = "tutorial"
  ports {
    internal = 80
    external = 8000
  }
}
.

$ terraform init
Provision the NGINX server container with apply. When Terraform asks you to confirm type yes and press ENTER.

$ terraform apply


curl localhost 8000

-------------------------------------------------------------------------------

Aws-instance-spinup-terraform
=============================

# create directory

$ mkdir learn-terraform-aws-instance

# change directory

$ cd learn-terraform-aws-instance

# create main.tf file

$ touch main.tf

# add below content

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.27"
    }
  }

  required_version = ">= 0.14.9"
}

provider "aws" {
  profile = "default"
  region  = "us-west-2"
}

resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"

  tags = {
    Name = "ExampleAppServerInstance"
  }
}


# Initialize the directory

$ terraform init




# Format and validate the configuration

$ terraform fmt

# validate terraform file

$ terraform validate

# Create infrastructure

$ terraform apply

# Inspect state

$ terraform show

-----------------------------------------------------------------------------------

# launch aws instance using access key
--------------------------------------

touch instance.tf past the following content

provider "aws" {

  access_key = "AKIAYF4M6DKY2IEGQ2EY"
  secret_key = "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"
  region     = "us-east-1"

}

resource "aws_instance" "demo" {

  ami            = "ami-09e67e426f25ce0d7"
  instance_type  = "t2.micro"
  key_name       = "SAIRSA"
  
}


touch versions.tf and past following content

terraform {

  required_version = ">= 0.12"
}



# initalize the terraform directory

$ terraform init

# plan the infra

$ terraform plan -out -out.terraform

# apply changes 

$ terraform apply -out.terraform

# view state of the terraform

$ terraform show

-------------------------------------------------------------------------------


## variable in terraform

$  touch instance.tf paste the following content

resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
}



$ touch provider.tf paste the following content

provider "aws" {
  access_key = var.AWS_ACCESS_KEY
  secret_key = var.AWS_SECRET_KEY
  region     = var.AWS_REGION
}


$ touch vars.tf paste the following content

variable "AWS_ACCESS_KEY" {
}

variable "AWS_SECRET_KEY" {
}

variable "AWS_REGION" {
  default = "us-east-1"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-09e67e426f25ce0d7"
    us-east-2 = "ami-09e67e426f25ce0d7"
    us-west-1 = "ami-09e67e426f25ce0d7"
  }
}



$ touch versions.tf  paste the following

terraform {
  required_version = ">= 0.12"
}


# configure credential file

$ touch  terraform.tfvars paste following and also add .gitignorefile

AWS_ACCESS_KEY = "AKIAYF4M6DKY2IEGQ2EY"

AWS_SECRET_KEY = "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"


# inialize the directory

$ terraform init

# plan the terraform

$ terraform plan -out out.terraform

# apply the changes in the terraform

$ terraform apply out.terraform

type : yes


# view terraform state

$ terraform show


# destroy the terraform infrastructure

$ terraform destroy

------------------------------------------------------------------------------------

# terraform software provision
-----------------------------


touch instance.tf paste the following content

resource "aws_key_pair" "mykey" {
  key_name   = "mykey"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}

resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  key_name      = aws_key_pair.mykey.key_name

  provisioner "file" {
    source      = "script.sh"
    destination = "/tmp/script.sh"
  }
  provisioner "remote-exec" {
    inline = [
      "chmod +x /tmp/script.sh",
      "sudo sed -i -e 's/\r$//' /tmp/script.sh",  # Remove the spurious CR characters.
      "sudo /tmp/script.sh",
    ]
  }
  connection {
    host        = coalesce(self.public_ip, self.private_ip)
    type        = "ssh"
    user        = var.INSTANCE_USERNAME
    private_key = file(var.PATH_TO_PRIVATE_KEY)
  }
}


# touch provider.tf paste the following 

provider "aws" {
  access_key = var.AWS_ACCESS_KEY
  secret_key = var.AWS_SECRET_KEY
  region     = var.AWS_REGION
}




# touch vars.tf paste the following


variable "AWS_ACCESS_KEY" {
}

variable "AWS_SECRET_KEY" {
}

variable "AWS_REGION" {
  default = "us-east-1"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-09e67e426f25ce0d7"
    us-west-2 = "ami-03d5c68bab01f3496"
    eu-west-1 = "ami-0d382e80be7ffdae5"
  }
}

variable "PATH_TO_PRIVATE_KEY" {
  default = "mykey"
}

variable "PATH_TO_PUBLIC_KEY" {
  default = "mykey.pub"
}

variable "INSTANCE_USERNAME" {
  default = "ubuntu"
}


# touch versions.tf paste the following 

terraform {
  required_version = ">= 0.12"
}



# touch script.sh paste the following

#!/bin/bash

# sleep until instance is ready
until [[ -f /var/lib/cloud/instance/boot-finished ]]; do
  sleep 1
done

# install nginx
apt-get update
apt-get -y install nginx

# make sure nginx is started
service nginx start


# touch terraform.tfvars file paste the following 

AWS_ACCESS_KEY = "AKIAYF4M6DKY2IEGQ2EY"


AWS_SECRET_KEY = "8fC6nwU71J5mPy6Fu0iCnPCe3m6uR+7SIBdv8QKv"


# Genetate private key into cureent directory

$ ssh-keygen -f mykey


# initialize the terraform directoey

$ terraform init

# plan terraform infra

$ terraform plan -out out.terraform

# apply the terraform infra

$ terraform apply out.terraform

# view the terraform state

$ terraform show

# destory the infra

$ terraform destroy


------------------------------------------------------------------------------------

## get aws instance ip address
------------------------------

# touch instance.tf paste following content

resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"
  provisioner "local-exec" {
    command = "echo ${aws_instance.example.private_ip} >> private_ips.txt"
  }
}

output "ip" {
  value = aws_instance.example.public_ip


# touch provider.tf paste the following content

provider "aws" {
  access_key = var.AWS_ACCESS_KEY
  secret_key = var.AWS_SECRET_KEY
  region     = var.AWS_REGION
}


# touch vars.tf paste the following content

variable "AWS_ACCESS_KEY" {
}

variable "AWS_SECRET_KEY" {
}

variable "AWS_REGION" {
  default = "us-east-1"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-09e67e426f25ce0d7"
    us-west-1 = "ami-0d382e80be7ffdae5"
    us-west-2 = "ami-03d5c68bab01f3496"

  }
}



}


# trerraform.tfvars paste the following content

AWS_ACCESS_KEY = "AKIAYF4M6DKY375OVIVX"

AWS_SECRET_KEY = "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"



# touch versions.tf paste the following content

terraform {
  required_version = ">= 0.12"
}



# initialize the terraform directoey

$ terraform init

# plan terraform infra

$ terraform plan -out out.terraform

# apply the terraform infra

$ terraform apply out.terraform

# view the terraform state

$ terraform show

# destory the infra

$ terraform destroy

-------------------------------------------------------------------------------------

## data sources in terraform
----------------------------


# touch provider.tf paste the following content

provider "aws" {
  region = var.AWS_REGION
}


# touch securitygroup.tf paste the following content

data "aws_ip_ranges" "european_ec2" {
  regions  = ["eu-west-1", "eu-central-1"]
  services = ["ec2"]
}

resource "aws_security_group" "from_europe" {
  name = "from_europe"

  ingress {
    from_port   = "443"
    to_port     = "443"
    protocol    = "tcp"
    cidr_blocks = slice(data.aws_ip_ranges.european_ec2.cidr_blocks, 0, 50)
  }
  tags = {
    CreateDate = data.aws_ip_ranges.european_ec2.create_date
    SyncToken  = data.aws_ip_ranges.european_ec2.sync_token
  }
}


# touch vars.tf paste the following content

variable "AWS_ACCESS_KEY" {
}

variable "AWS_SECRET_KEY" {
}

variable "AWS_REGION" {
  default = "eu-west-1"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-09e67e426f25ce0d7"
    us-west-1 = "ami-0943382e114f188e8"
    eu-west-1 = "ami-844e0bf7"
  }
}


# touch versions.tf paste the following content

terraform {
  required_version = ">= 0.12"
}



# trerraform.tfvars paste the following content

AWS_ACCESS_KEY = "AKIAYF4M6DKYVKRCF3W2"

AWS_SECRET_KEY = "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"



# initialize the terraform directoey

$ terraform init

# plan terraform infra

$ terraform plan -out out.terraform

# apply the terraform infra

$ terraform apply out.terraform

# view the terraform state

$ terraform show

# destory the infra

$ terraform destroy

---------------------------------------------------------------------------------

# create aws vpc using terraform
--------------------------------

# configure aws cli for credential

touch vpc.tf paste the following content

# Internet VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = "true"
  enable_dns_hostnames = "true"
  enable_classiclink   = "false"
  tags = {
    Name = "main"
  }
}

# Subnets
resource "aws_subnet" "main-public-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-public-1"
  }
}

resource "aws_subnet" "main-public-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-public-2"
  }
}

resource "aws_subnet" "main-public-3" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.3.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1c"

  tags = {
    Name = "main-public-3"
  }
}

resource "aws_subnet" "main-private-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.4.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-private-1"
  }
}

resource "aws_subnet" "main-private-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.5.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-private-2"
  }
}

resource "aws_subnet" "main-private-3" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.6.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1c"

  tags = {
    Name = "main-private-3"
  }
}

# Internet GW
resource "aws_internet_gateway" "main-gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main"
  }
}

# route tables
resource "aws_route_table" "main-public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main-gw.id
  }

  tags = {
    Name = "main-public-1"
  }
}

# route associations public
resource "aws_route_table_association" "main-public-1-a" {
  subnet_id      = aws_subnet.main-public-1.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-2-a" {
  subnet_id      = aws_subnet.main-public-2.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-3-a" {
  subnet_id      = aws_subnet.main-public-3.id
  route_table_id = aws_route_table.main-public.id
}



# touch nat.tf paste the following content

# nat gw
resource "aws_eip" "nat" {
  vpc = true
}

resource "aws_nat_gateway" "nat-gw" {
  allocation_id = aws_eip.nat.id
  subnet_id     = aws_subnet.main-public-1.id
  depends_on    = [aws_internet_gateway.main-gw]
}

# VPC setup for NAT
resource "aws_route_table" "main-private" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block     = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.nat-gw.id
  }

  tags = {
    Name = "main-private-1"
  }
}

# route associations private
resource "aws_route_table_association" "main-private-1-a" {
  subnet_id      = aws_subnet.main-private-1.id
  route_table_id = aws_route_table.main-private.id
}

resource "aws_route_table_association" "main-private-2-a" {
  subnet_id      = aws_subnet.main-private-2.id
  route_table_id = aws_route_table.main-private.id
}

resource "aws_route_table_association" "main-private-3-a" {
  subnet_id      = aws_subnet.main-private-3.id
  route_table_id = aws_route_table.main-private.id
}



# touch vars.tf paste the following content

variable "AWS_REGION" {
  default = "eu-west-1"
}


# touch provider.tf paste the following content

provider "aws" {
  region = var.AWS_REGION
}


# touch versions.tf paste the following content


terraform {
  required_version = ">= 0.12"
}


# intialize the directory

$ terraform init

# plan the infra

$ terraform plan

# apply the infra

$ terraform apply

# view the infra

$ terraform show

# remove the infra

$ terraform destroy

---------------------------------------------------------------------------------------

# create ec2 instance using created vpc
---------------------------------------


# touch vpc.tf paste the following content

# Internet VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = "true"
  enable_dns_hostnames = "true"
  enable_classiclink   = "false"
  tags = {
    Name = "main"
  }
}

# Subnets
resource "aws_subnet" "main-public-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-public-1"
  }
}

resource "aws_subnet" "main-public-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-public-2"
  }
}

resource "aws_subnet" "main-public-3" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.3.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1c"

  tags = {
    Name = "main-public-3"
  }
}

resource "aws_subnet" "main-private-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.4.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-private-1"
  }
}

resource "aws_subnet" "main-private-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.5.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-private-2"
  }
}

resource "aws_subnet" "main-private-3" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.6.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1c"

  tags = {
    Name = "main-private-3"
  }
}

# Internet GW
resource "aws_internet_gateway" "main-gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main"
  }
}

# route tables
resource "aws_route_table" "main-public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main-gw.id
  }

  tags = {
    Name = "main-public-1"
  }
}

# route associations public
resource "aws_route_table_association" "main-public-1-a" {
  subnet_id      = aws_subnet.main-public-1.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-2-a" {
  subnet_id      = aws_subnet.main-public-2.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-3-a" {
  subnet_id      = aws_subnet.main-public-3.id
  route_table_id = aws_route_table.main-public.id
}



# touch securitygroup.tf paste the following content

resource "aws_security_group" "allow-ssh" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-ssh"
  description = "security group that allows ssh and all egress traffic"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "allow-ssh"
  }
}


# touch the provider.tf paste the following content

provider "aws" {
  region = var.AWS_REGION
}

# touch key.tf paste the following content

resource "aws_key_pair" "mykeypair" {
  key_name   = "mykeypair"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}



# generate mykey 

$ ssk-keygen -f mykey


# touch vars.tf and paste the following content

variable "AWS_REGION" {
  default = "eu-west-1"
}

variable "PATH_TO_PRIVATE_KEY" {
  default = "mykey"
}

variable "PATH_TO_PUBLIC_KEY" {
  default = "mykey.pub"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-13be557e"
    us-west-2 = "ami-06b94666"
    eu-west-1 = "ami-0a8e758f5e873d1c1"
  }
}


# touch versions.tf paste the following content


terraform {
  required_version = ">= 0.12"
}


# copy previous state file and backup file to current directory

$ cp ../demo07/terraform.tfstate .

$ cp ../demo07/terraform.tfstate.backup .



# intialize the directory

$ terraform init

# plan the infra

$ terraform plan

# apply the infra

$ terraform apply

# view the infra

$ terraform show

# remove the infra

$ terraform destroy


# using pem ssh ec2 instance check vpc ip

--------------------------------------------------------------------------------------------------

# create EBS volume using terraform
--------------------------------------

# touch instance.tf and paste the following content

resource "aws_instance" "example" {
  ami           = var.AMIS[var.AWS_REGION]
  instance_type = "t2.micro"

  # the VPC subnet
  subnet_id = aws_subnet.main-public-1.id

  # the security group
  vpc_security_group_ids = [aws_security_group.allow-ssh.id]

  # the public SSH key
  key_name = aws_key_pair.mykeypair.key_name
}

resource "aws_ebs_volume" "ebs-volume-1" {
  availability_zone = "eu-west-1a"
  size              = 20
  type              = "gp2"
  tags = {
    Name = "extra volume data"
  }
}

resource "aws_volume_attachment" "ebs-volume-1-attachment" {
  device_name = "/dev/xvdh"
  volume_id   = aws_ebs_volume.ebs-volume-1.id
  instance_id = aws_instance.example.id
}


# touch key.tf paste the following content

resource "aws_key_pair" "mykeypair" {
  key_name   = "mykeypair"
  public_key = file(var.PATH_TO_PUBLIC_KEY)
}




# touch provider.tf and paste the following content

provider "aws" {
  region = var.AWS_REGION
}



# touch securitygroup.tf and paste the following content

resource "aws_security_group" "allow-ssh" {
  vpc_id      = aws_vpc.main.id
  name        = "allow-ssh"
  description = "security group that allows ssh and all egress traffic"
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
  tags = {
    Name = "allow-ssh"
  }
}



# touch vars.tf and paste the following content

variable "AWS_REGION" {
  default = "eu-west-1"
}

variable "PATH_TO_PRIVATE_KEY" {
  default = "mykey"
}

variable "PATH_TO_PUBLIC_KEY" {
  default = "mykey.pub"
}

variable "AMIS" {
  type = map(string)
  default = {
    us-east-1 = "ami-13be557e"
    us-west-2 = "ami-06b94666"
    eu-west-1 = "ami-844e0bf7"
  }
}



# touch vpc.tf and paste the following the content

# Internet VPC
resource "aws_vpc" "main" {
  cidr_block           = "10.0.0.0/16"
  instance_tenancy     = "default"
  enable_dns_support   = "true"
  enable_dns_hostnames = "true"
  enable_classiclink   = "false"
  tags = {
    Name = "main"
  }
}

# Subnets
resource "aws_subnet" "main-public-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.1.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-public-1"
  }
}

resource "aws_subnet" "main-public-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.2.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-public-2"
  }
}

resource "aws_subnet" "main-public-3" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.3.0/24"
  map_public_ip_on_launch = "true"
  availability_zone       = "eu-west-1c"

  tags = {
    Name = "main-public-3"
  }
}

resource "aws_subnet" "main-private-1" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.4.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1a"

  tags = {
    Name = "main-private-1"
  }
}

resource "aws_subnet" "main-private-2" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.5.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1b"

  tags = {
    Name = "main-private-2"
  }
}

resource "aws_subnet" "main-private-3" {
  vpc_id                  = aws_vpc.main.id
  cidr_block              = "10.0.6.0/24"
  map_public_ip_on_launch = "false"
  availability_zone       = "eu-west-1c"

  tags = {
    Name = "main-private-3"
  }
}

# Internet GW
resource "aws_internet_gateway" "main-gw" {
  vpc_id = aws_vpc.main.id

  tags = {
    Name = "main"
  }
}

# route tables
resource "aws_route_table" "main-public" {
  vpc_id = aws_vpc.main.id
  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.main-gw.id
  }

  tags = {
    Name = "main-public-1"
  }
}

# route associations public
resource "aws_route_table_association" "main-public-1-a" {
  subnet_id      = aws_subnet.main-public-1.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-2-a" {
  subnet_id      = aws_subnet.main-public-2.id
  route_table_id = aws_route_table.main-public.id
}

resource "aws_route_table_association" "main-public-3-a" {
  subnet_id      = aws_subnet.main-public-3.id
  route_table_id = aws_route_table.main-public.id
}



# touch versions.tf and paste the foloowing content


terraform {
  required_version = ">= 0.12"
}


# create keyfile to access instance

$ ssh-keygen -f mykey



# intialize the directory

$ terraform init

# plan the infra

$ terraform plan

# apply the infra

$ terraform apply

# view the infra

$ terraform show

# remove the infra

$ terraform destroy

----------------------------------------------------------------------------------------------





