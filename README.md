# TerraformSetupInUbuntuWSL
Install and setup terraform in Ubuntu WSL

Ensure that your system is up to date and you have installed the gnupg, software-properties-common, and curl packages installed. You will use these packages to verify HashiCorp's GPG signature and install HashiCorp's Debian package repository.
```
$ sudo apt-get update && sudo apt-get install -y gnupg software-properties-common
```

Install the HashiCorp GPG key.
```
$ wget -O- https://apt.releases.hashicorp.com/gpg | \
gpg --dearmor | \
sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg > /dev/null
```

Verify the key's fingerprint.
```
$ gpg --no-default-keyring \
--keyring /usr/share/keyrings/hashicorp-archive-keyring.gpg \
--fingerprint
```
The gpg command will report the key fingerprint:
```
/usr/share/keyrings/hashicorp-archive-keyring.gpg
-------------------------------------------------
pub   rsa4096 XXXX-XX-XX [SC]
AAAA AAAA AAAA AAAA
uid           [ unknown] HashiCorp Security (HashiCorp Package Signing) <security+packaging@hashicorp.com>
sub   rsa4096 XXXX-XX-XX [E]
```

Add the official HashiCorp repository to your system. The lsb_release -cs command finds the distribution release codename for your current system, such as buster, groovy, or sid.
```
$ echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(grep -oP '(?<=UBUNTU_CODENAME=).*' /etc/os-release || lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
```
Download the package information from HashiCorp.
```
$ sudo apt update
```
Install Terraform from the new repository.
```
$ sudo apt-get install terraform
```

Verify the installation
Verify that the installation worked by opening a new terminal session and listing Terraform's available subcommands.
```
$ terraform -help
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.
##...
```
Add any subcommand to terraform -help to learn more about what it does and available options.
```
$ terraform -help plan
```

install the autocomplete package.
```
$ terraform -install-autocomplete
```

Create a directory named learn-terraform-docker-container.
```
$ mkdir learn-terraform-docker-container
```
This working directory houses the configuration files that you write to describe the infrastructure you want Terraform to create and manage. When you initialize and apply the configuration here, Terraform uses this directory to store required plugins, modules (pre-written configurations), and information about the real infrastructure it created.

Navigate into the working directory.
```
$ cd learn-terraform-docker-container
```
In the working directory, create a file called main.tf and paste the following Terraform configuration into it.
```
terraform {
  required_providers {
    docker = {
      source  = "kreuzwerker/docker"
      version = "~> 3.0.1"
    }
  }
}

provider "docker" {}

resource "docker_image" "nginx" {
  name         = "nginx"
  keep_locally = false
}

resource "docker_container" "nginx" {
  image = docker_image.nginx.image_id
  name  = "tutorial"

  ports {
    internal = 80
    external = 8000
  }
}
```

Initialize the project, which downloads a plugin called a provider that lets Terraform interact with Docker.
```
$ terraform init
```
Provision the NGINX server container with apply. When Terraform asks you to confirm type yes and press ENTER.
```
$ terraform apply
```
Verify the existence of the NGINX container by visiting localhost:8000 in your web browser or running docker ps to see the container.
```
$ docker ps
CONTAINER ID        IMAGE                     COMMAND                  CREATED             STATUS              PORTS                    NAMES
425d5ee58619        e791337790a6              "nginx -g 'daemon of…"   20 seconds ago      Up 19 seconds       0.0.0.0:8000->80/tcp     tutorial
```
To stop the container, run terraform destroy.
```
$ terraform destroy
```
You've now provisioned and destroyed an NGINX webserver with Terraform.


# Reference
https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli
