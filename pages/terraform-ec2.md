## Setting up a simple webpage on EC2 with a custom URL using Terraform

In a previous [blog](webpage-custom-url.md), we created a basic static site on EC2 using mainly AWS CLI commands.  In this blog we will see how we can automate much of this process using [Terraform](https://www.terraform.io/).  The final code for this blog can be found [here](https://github.com/imrankhan17/terraform-aws).

Let's start by creating a new directory for this project and a few configuration files.
```shell script
mkdir terraform-aws
cd terraform-aws
touch main.tf variables.tf outputs.tf
```

Within `variables.tf` let's define some variables providing types, descriptions and defaults:
```hcl
variable "region" {
  type    = string
  default = "eu-west-2"
}

variable "public_key" {
  type        = string
  description = "File path of public key."
  default     = "~/.ssh/id_rsa.pub"
}

variable "private_key" {
  type        = string
  description = "File path of private key."
  default     = "~/.ssh/id_rsa"
}

variable "domain" {
  type        = string
  description = "Name of domain."
}
```

`region` is the AWS region our instance will be hosted in.  `public_key` and `private_key` should be file paths to the relevant keys in your `~/.ssh/` directory.  `domain` should be a domain name which you own e.g. `example.com`.

Within `main.tf` let's define the Terraform provider as:
```hcl
provider "aws" {
  region = var.region
}
```

Once we have these two files, we can run `terraform init` to initialise the working directory and download the Terraform AWS provider plugin.  You should see a message saying it has been successful.

Back to the `main.tf` file, we can add a security group resource with the necessary ports to allow SSH and web hosting access.
```hcl
resource "aws_security_group" "ssh_web" {
  name = "tf_sg"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

We will also need a key pair to authenticate SSH access.  This is created using the public key referenced in `variables.tf`.
```hcl
resource "aws_key_pair" "key" {
  key_name   = "tf_key"
  public_key = file(var.public_key)
}
```

We will now add the resource defining the EC2 instance.  This first uses a data source to search for the most recent Ubuntu 18.04 AMI.  Within the `aws_instance` resource, we define a `connection` block with the relevant SSH log-in details.  This allows the `remote-exec` provisioner to access the instance and run the three lines of bash commands.  Note: we are not using Docker in this example and instead installing Nginx directly within the instance.

```hcl
data "aws_ami" "ubuntu" {
  owners      = ["099720109477"]
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-bionic-18.04-amd64-server-*"]
  }
}

resource "aws_instance" "web" {
  ami             = data.aws_ami.ubuntu.id
  instance_type   = "t2.micro"
  key_name        = aws_key_pair.key.key_name
  security_groups = [aws_security_group.ssh_web.name]

  connection {
    type        = "ssh"
    user        = "ubuntu"
    host        = self.public_ip
    private_key = file(var.private_key)
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt update -y",
      "sudo apt install nginx -y",
      "sudo bash -c 'echo Welcome to ${var.domain} > /var/www/html/index.html'"
    ]
  }
}
```

From there on, we can define the Route 53 hosted zone and records.
```hcl
resource "aws_route53_zone" "primary" {
  name = var.domain
}

resource "aws_route53_record" "www" {
  zone_id = aws_route53_zone.primary.zone_id
  name    = "www.${aws_route53_zone.primary.name}"
  type    = "A"
  ttl     = 300
  records = [aws_instance.web.public_ip]
}

resource "aws_route53_record" "blank" {
  zone_id = aws_route53_zone.primary.zone_id
  name    = aws_route53_zone.primary.name
  type    = "A"
  ttl     = 300
  records = [aws_instance.web.public_ip]
}
```

In `outputs.tf` we can insert variables to be printed out at the end of the process.
```hcl
output "ip" {
  value       = aws_instance.web.public_dns
  description = "The URL of the server instance."
}

output "nameservers" {
  value       = aws_route53_zone.primary.name_servers
  description = "List of nameservers to be used by the domain name provider e.g. GoDaddy."
}
```

We are now ready to create our infrastructure.  Let's run `terraform plan -var "domain=example.com"` to view the execution plan.  If you are happy, you can run `terraform apply -var "domain=example.com"`.  If you navigate to the public DNS URL outputted at the end of the process, you should see the message: `Welcome to example.com`.

The four nameservers, the other output, are needed to update the nameservers in the domain name provider's settings.  After anything from 30 minutes to 48 hours, you should see the same message when navigating to `example.com`.

---
[Home](../index.md)
