

# Task 1. Create the configuration files

-  **Run in Cloud Shell**
```yaml
touch main.tf
```
```yaml
touch variables.tf
```
```yaml
mkdir modules
```
```yaml
cd modules
```
```yaml
mkdir instances
```
```yaml
cd instances
```
```yaml
touch instances.tf
```
```yaml
touch outputs.tf
```
```yaml
touch variables.tf
```
```yaml
cd ..
```
```yaml
mkdir storage
```
```yaml
cd storage
```
```yaml
touch storage.tf
```
```yaml
touch outputs.tf
```
```yaml
touch variables.tf
```
```yaml
cd
```

- **Add the following to the each `variables.tf` file, and fill in the GCP Project ID:**

```yaml
variable "region" {

 default = "us-central1"

}

variable "zone" {

 default = "us-central1-a"

}

variable "project_id" {

 default = "<REPLACE PROJECT ID>"

}
```

- **Add the following to the `main.tf` file :**

```yaml
terraform {

  required_providers {

    google = {

      source = "hashicorp/google"

      version = "3.55.0"

    }

  }

}

provider "google" {

  project     = var.project_id

  region      = var.region

  zone        = var.zone

}

module "instances" {

  source     = "./modules/instances"

}
```

- **Run `terraform init` in Cloud Shell in the root directory to initialize terraform.**

# Task 2. Import infrastructure

- **Next, navigate to modules/instances/instances.tf. Copy the following configuration into the file:**
```yaml

resource "google_compute_instance" "tf-instance-1" {

  name         = "tf-instance-1"

  machine_type = "n1-standard-1"

  zone         = var.zone

  boot_disk {

    initialize_params {

      image = "debian-cloud/debian-10"

    }

  }

  network_interface {

 network = "default"

  }

  metadata_startup_script = <<-EOT

        #!/bin/bash

    EOT

  allow_stopping_for_update = true

}

resource "google_compute_instance" "tf-instance-2" {

  name         = "tf-instance-2"

  machine_type = "n1-standard-1"

  zone         = var.zone

  boot_disk {

    initialize_params {

      image = "debian-cloud/debian-10"

    }

  }

  network_interface {

 network = "default"

  }

  metadata_startup_script = <<-EOT

        #!/bin/bash

    EOT

  allow_stopping_for_update = true

}
```

- **Navigate to Compute Engine > VM Instances. Click on tf-instance-1. Copy the Instance ID**

- **Navigate to Compute Engine > VM Instances. Click on tf-instance-2. Copy the Instance ID**

- **Run in cloud shell**

```yaml
terraform import module.instances.google_compute_instance.tf-instance-1 <Instance ID - 1>
```
```yaml
terraform import module.instances.google_compute_instance.tf-instance-2 <Instance ID - 2>
```
```yaml
terraform plan
```
```yaml
terraform apply
```

# Task 3. Configure a remote backend

- **Add the following code to the `modules/storage/storage.tf` file**
```yaml
resource "google_storage_bucket" "storage-bucket" {

  name          = "<YOUR-BUCKET>"

  location      = "US"

  force_destroy = true

  uniform_bucket_level_access = true

}
```

- **Next, add the following to the main.tf file:**

```yaml

module "storage" {

  source     = "./modules/storage"

}
```

- **Run in cloud shell :**

```yaml
terraform init
```
```yaml
terraform apply
```
- **Next, update the `main.tf`**
```yaml

terraform {

  backend "gcs" {

    bucket  = "<REPLACE YOUR BUCKET>"

 prefix  = "terraform/state"

  }

  required_providers {

    google = {

      source = "hashicorp/google"

      version = "3.55.0"

    }

  }

}
```

```yaml
terraform init
```

# Task 4. Modify and update infrastructure

- **Navigate to `modules/instances/instances.tf` modify `tf-instance-1` and `tf-instance-2` by changing `machine_type` from `"n1-standard-1"` to `"n1-standard-2"` **

- **add tf-instance-3**

```yaml
resource "google_compute_instance" "<Instance Name>" {

  name         = "<Instance Name>"

  machine_type = "n1-standard-2"

  zone         = var.zone

  boot_disk {

    initialize_params {

      image = "debian-cloud/debian-10"

    }

  }

  network_interface {

 network = "default"

  }

  metadata_startup_script = <<-EOT

        #!/bin/bash

    EOT

  allow_stopping_for_update = true

}
```

- **Run in cloud shell :**
```yaml
terraform init
```
```yaml
terraform apply
```
# Task 5. Taint and destroy resources
```yaml
terraform taint module.instances.google_compute_instance.Instance_name
```
```yaml
terraform plan
```
```yaml
terraform apply
```
- **Navigate to `modules/instances/instances.tf` remove `tf-instance-3`**
```yaml
terraform apply
```
# Task 6. Use a module from the Registry

- **In the Terraform Registry, browse to the Network Module.**

- **Copy and paste the following into the `main.tf`**
```yaml
module "vpc" {

    source  = "terraform-google-modules/network/google"

    version = "~> 3.4.0"

    project_id   = "PROJECT_ID"

    network_name = "VPC_NAME"

    routing_mode = "GLOBAL"

    subnets = [

        {

            subnet_name           = "subnet-01"

            subnet_ip             = "10.10.10.0/24"

            subnet_region         = "us-central1"

        },

        {

            subnet_name           = "subnet-02"

            subnet_ip             = "10.10.20.0/24"

            subnet_region         = "us-central1"

            subnet_private_access = "true"

            subnet_flow_logs      = "true"

            description           = "This subnet has a description"

        },

    ]

}
```

- **Run in cloud shell :**
```yaml
terraform init
```
```yaml
terraform apply
```
- **Next, navigate to the `instances.tf` file and update the configuration resources to connect `tf-instance-1` to `subnet-01` and `tf-instance-2` to `subnet-02`**
```yaml
resource "google_compute_instance" "tf-instance-1"{

  name         = "tf-instance-1"

  machine_type = "n1-standard-2"

  zone         = var.zone

  boot_disk {

    initialize_params {

      image = "debian-cloud/debian-10"

    }

  }

  network_interface {

    network = "VPC_NAME"

     subnetwork = "subnet-01"

  }

  metadata_startup_script = <<-EOT

        #!/bin/bash

    EOT

  allow_stopping_for_update = true

}

resource "google_compute_instance" "tf-instance-2"{

  name         = "tf-instance-2"

  machine_type = "n1-standard-2"

  zone         = var.zone

  boot_disk {

    initialize_params {

      image = "debian-cloud/debian-10"

    }

  }

  network_interface {

    network = "VPC_NAME"

     subnetwork = "subnet-02"

  }

  metadata_startup_script = <<-EOT

        #!/bin/bash

    EOT

  allow_stopping_for_update = true

}

module "vpc" {

    source  = "terraform-google-modules/network/google"

    version = "~> 3.4.0"

    project_id   = "PROJECT_ID"

    network_name = "VPC_NAME"

    routing_mode = "GLOBAL"

    subnets = [

        {

            subnet_name           = "subnet-01"

            subnet_ip             = "10.10.10.0/24"

            subnet_region         = "us-central1"

        },

        {

            subnet_name           = "subnet-02"

            subnet_ip             = "10.10.20.0/24"

            subnet_region         = "us-central1"

            subnet_private_access = "true"

            subnet_flow_logs      = "true"

            description           = "This subnet has a description"

        },

    ]

}
```

- **run in cloud shell :**
```yaml
terraform init
```
```yaml
terraform apply
```
# Task 6 : Configure a firewall

- **Add the following resource to the main.tf file and fill in the GCP Project ID:**
```yaml
resource "google_compute_firewall" "tf-firewall"{

  name    = "tf-firewall"

 network = "projects/<PROJECT_ID>/global/networks/VPC_Name"

  allow {

    protocol = "tcp"

    ports    = ["80"]

  }

  source_tags = ["web"]

  source_ranges = ["0.0.0.0/0"]

}
```

- **run in cloud shell :**
```yaml
terraform init
```
```yaml
terraform apply
```
