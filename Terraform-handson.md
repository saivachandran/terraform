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





