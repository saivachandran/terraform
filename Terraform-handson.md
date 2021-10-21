#install Terraform
===================

wget https://releases.hashicorp.com/terraform/1.0.9/terraform_1.0.9_linux_amd64.zip


unzip terraform_1.0.9_linux_amd64.zip

mv ~/terraform /usr/local/bin/


terraform --help



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

----------------------------------------


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

------------------------------------------------

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











