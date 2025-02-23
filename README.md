# terraform-google-compute-engine-instance

Create virtual machines on Google Cloud. With DNS A record for easy access. Published in the [Terraform registry](https://registry.terraform.io/modules/Eimert/compute-engine-instance/).

## Usage example:

Create VMs with public IP addresses and DNS A record alias. Machine type can be changed without destroying the boot disk.

1. Create a new directory for this terraform configuration
2. Create a main.tf, for example:
```ruby
# Configure the Google Cloud provider
provider "google" {
  credentials = "${file("king-of-my-google-cloud-castle.json")}"
  project     = "smashing-dash-1992"
}

module "google-dns-managed-zone" {
  source          = "github.com/Eimert/terraform-google-dns-managed-zone"

  # descriptive name for dns zone
  dns_name        = "cloud-zone"
  # requires last dot. Ex.: prod.example.com.
  dns_zone        = "cloud.eimertvink.nl."
}

module "vm1" {
  source          = "github.com/Eimert/terraform-google-compute-engine-instance"
  amount          = 1
  region          = "europe-west4"
  zone            = "europe-west4-c"
  # hostname format: name_prefix-amount
  name_prefix     = "vm"
  machine_type    = "n1-standard-2"
  disk_type       = "pd-ssd"
  disk_size       = "15"
  disk_image      = "centos-cloud/centos-7"

  dns_name        = "${module.google-dns-managed-zone.dns_name}"
  dns_zone        = "${module.google-dns-managed-zone.dns_zone}"
  dns_record_name = "tower-dev"

  user_data       = "firestone-lab"
  username        = "eimert"
  public_key_path = "~/.ssh/id_rsa.pub"
}

# module "gc2" {}
```
3. ```terraform init```
4. ```terraform plan``` Boom! Credentials file missing.
5. Add your google cloud credentials in a .json file. [Getting started guide](https://www.terraform.io/docs/providers/google/getting_started.html#adding-credentials)

> Keep the Google Cloud credentials in a safe place. Don't push them to Git.

6. Adapt the Terraform variables in `main.tf` to match your Google cloud project name, and VM requirements. All optional parameters can be found in [variables.tf](./variables.tf).
5. Let terraform fire up the VM's:
```bash
terraform apply
```
6. Wait a few ~~minutes~~ seconds.
7. Connect using SSH (private key auth): `ssh -i <private key> <username>@<ip from output>`. Or: `ssh eimert@ansible-dev.cloud.eimertvink.nl`.
8. Break down the resources:
```bash
terraform destroy
```

## machine_type
Overview of choices for variable machine_type.
```
f1-micro
g1-small

n1-standard-1
n1-standard-2
n1-standard-4
n1-standard-8

n1-highmem-2
n1-highmem-4
n1-highmem-8

n1-highcpu-2
n1-highcpu-4
n1-highcpu-8
```

