---
layout: post
comments: true
title:  'Using Terraform on GCP, AWS and Physical Server'
layout: post
author: malike
categories: [devops,sre]
tags: [terraform]
image:
  path: /posts/dilbert_system_admin.png
  width: 800
  height: 500
  alt: automate.
---

### [Terrafrom](https://terraform.io/)

_"Terraform is an open-source infrastructure as code software tool created by HashiCorp. It enables users to define and provision a datacenter infrastructure using a high-level configuration language known as Hashicorp Configuration Language (HCL), or optionally JSON. Terraform supports a number of cloud infrastructure providers such as Amazon Web Services, IBM Cloud (formerly Bluemix), Google Cloud Platform, Linode, Microsoft Azure, Oracle Cloud Infrastructure, or VMware vSphere as well as OpenStack."_

For me that's the one thing that sets Terraform apart from the others. It supports other cloud infra and doesn't take a lot of effort to switch config from one to the other. Some people argue that to be cloud agnostic is a myth and just a marketing term, which may be partly true, but there many advantages to not tightly coupling everything in your infrastructure to one cloud service provider. For example AWS CloudFormation is cool but restricts you to just AWS. Terraform however supports many providers and allows you to provision resources on different servers and if abstracted properly you can use same terraform files to different cloud providers with minimal changes.

### Terraform 101

To help us appreciate this let's create an nginx server on 3 types of servers, linux box, GCP and AWS.
Before that a short Terraform 101 for more details you can read more [here](https://www.terraform.io/docs/).

*1. Variables :* Input variables are used to define values that we can use to configure your infrastructure. There are different formats:

***i. string***

Strings mark a single value per structure and are commonly used to simplify and make complicated values more user-friendly. Below is an example of a string variable definition.

```hcl
variable "username" {
  default = "malike_st"
}
```

and by using the template below we can substitute values for our variables in the `*.tf` files.

```hcl
username = "${var.username}"
```

***ii. list***

List is a collection type that allows us to use multiple values for a variable and can easily be read by their indexes.

```hcl
variable "servers" {
    default = ["server", "server1", "server2"]
}
```

and using indexes we can pick the value we want.

```hcl
server = "${var.servers[0]}"
```

***iii. map***

Maps represent another type of collection defined by key value pairs.

```hcl
variable "region_zone" {
    type = "map"
    default = {
        "us-central1"  = "us-central1-b"
        "us-west2" = "us-west2-a"
    }
}
```

We can access any value by it matching key.

```hcl
deployment_zone = "${var.region_zone["us-central1"]}"
```

***iv. boolean***

The last type of variable types is boolean. Simple `true` or `false` values we can use in our infrastructure set up

```hcl
variable "set_password" {
    default = false
}
```

Variables can be set in a separate file, `tfvars`  and used during deployment. This variable file will have passwords and other sensitive stuff we wouldn't want to share with the world. It can be kept outside the terraform files so it's not mistakenly versioned. By passing our variable file  `terraform apply -var-file=../../aws_vars.tfvars` we can set the variables to use for a deployment.

*2. Output Variables :* Output variables helps us get useful information about your infrastructure. For example to return the public ip of an EC2 instance after deployment. Since most of the computation is done during deployment, Output variables help us peek into the deployment as it happens.

```hcl
output "aws_instance_public_dns" {
    value = "${aws_instance.nginx.public_dns}"
}
```

*3. Providers :*  A provider on Terraform provides an abstraction through which Terraform can work with the underlying resources of the IaaS (e.g. AWS, GCP, Microsoft Azure, Physical servers, Vagraant), PaaS (e.g. Heroku, CloudFoundry).

A list of providers supported by Terraform can be found [here](https://www.terraform.io/docs/providers/) but that's not an exhaustive list.

###### <center>An AWS provider </center> 

```hcl
provider "aws" {
  access_key = "${var.aws_access_key}"
  secret_key = "${var.aws_secret_key}"
  region     = "us-east-1"
}
```

###### <center>A GCP provider </center>

```hcl
provider "google" {
    credentials = "${file("${var.access_file}")}"
    project     = "${var.project_id}"
    region      = "us-west2"
}
```

*Resources :* A resource is basically any object under a provider (IaaS, PaaS) that we can can set up with a set of parameters. For example an EC2 instance is a type of resource on AWS and compute a resource on GCP.

###### <center>An AWS resource </center>

```hcl
resource "aws_instance" "nginx1" {
  ami           = "ami-c58c1dd3"
  instance_type = "t2.micro"
  subnet_id     = "${aws_subnet.subnet1.id}"
  vpc_security_group_ids = ["${aws_security_group.nginx-sg.id}"]
  key_name        = "${var.key_name}"
}
```

###### <center>A GCP resource </center>

```hcl
resource "google_compute_instance" "nginx" {
    name         = "nginx-vm-${random_id.instance_id.hex}"
    machine_type = "f1-micro"
    zone         = "us-west2-a"
}
```

#### Simple Terraform Script To NGINX server

Putting everything together [here's](https://github.com/malike/terraform-poc) a simple terraform set up to install NGINX server on local server, AWS and GCP with security groups/ firewall restrictions. You can see the similarities between all three scripts and the main difference being in the ***provider*** and ***resources*** because obviously you cant use AWS resources on GCP and vice-versa but because of terraform's HCL it's easier (imo) to use than the JSON or YAML.

For example this is the same resource declared in HCL (Terraform) and YML (CloudFormation)

###### <center>HCL : Terrafrom </center>  
```hcl
 resource "aws_instance" "nginx" {
    ami           = "ami-c58c1dd3"
    instance_type = "t2.micro"
    subnet_id     = "${aws_subnet.subnet1.id}"
 }
```

###### <center>YML : CloudFormation </center> 

```yml
 Resources:
   NGINXEC2Instance:
     Type: "AWS::EC2::Instance"
     Properties:
        InstanceType: "t2.micro"
        ImageId: "ami-c58c1dd3"
        SubnetId: !RefNginxSubnet1
```

But why would one want to set up their environments on both AWS GCP or locally?, I don't have all the answers but it's quite easy to see different concerns by different teams for example one company will love to set up their environments locally and and not strictly restrict it to the cloud.

One other thing to like about Terraform is it keeps states in a file `terraform.tfstate`, where it keeps details about the terraform version and other meta data to improve performance of infrastructure and keep your infrastructure _sane_.

Terraform validates your configuration with `terraform plan`, where it will give you a summary of everything it will be doing before it actually does it with `terraform apply`.

Terraform is really cool and it's opensource and I personally love it for being cloud agnostic and HCL, if you find it interesting as well and want to learn more [https://learn.hashicorp.com/terraform/](https://learn.hashicorp.com/terraform/).

<br>
<br>
**REFERENCES**

[https://www.terraform.io/docs/](https://www.terraform.io/docs/)