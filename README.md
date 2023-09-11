# scripts
# Hands-on with AWS and Terraform

# Terraform Configuration File (main.tf)

```jsx
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region  = "us-east-1"
}

resource "aws_s3_bucket" "s3_bucket" {
  bucket = "tcb-app-qa-jr"
}

resource "aws_s3_bucket_public_access_block" "s3_block" {
  bucket = aws_s3_bucket.s3_bucket.id

  block_public_acls       = true
  block_public_policy     = true
  ignore_public_acls      = true
  restrict_public_buckets = true
}
```

# Steps

- Create folder cloud-warmup
- Create folder live2-terraform-aws
- Open the cloud-warmup folder on VS Code
- Create the file [main.tf](http://main.tf/)
- Create the configuration to deploy an S3 bucket

```jsx
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }

  required_version = ">= 1.2.0"
}

provider "aws" {
  region  = "us-east-1"
}

resource "aws_s3_bucket" "s3_bucket" {
  bucket = "tcb-app-qa-jr"
}
```

- Login to AWS, open the Cloud Shell
- Install Terraform
https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started
- Upload the [main.tf](http://main.tf/) file to the Cloud Shell
- Init, plan and apply to create the S3

```jsx
terraform init
terraform plan
terraform apply
```

- Add the configuration to prevent public access on S3

```jsx
resource "aws_s3_bucket_public_access_block" "s3_block" {
bucket = aws_s3_bucket.s3_bucket.id

block_public_acls       = true
block_public_policy     = true
ignore_public_acls      = true
restrict_public_buckets = true
}
```

# Troubleshooting:

If the Cloud Shell AWS CLI token session expires, run any AWS CLI to refresh it. Example:
aws s3 ls s3://tcb-app-qa-jr



# Hands-on with Docker and Cloud

# Steps

- Download app.zip: https://www.dropbox.com/s/vqe1cwk1o8fb4h2/app-en.zip?dl=0
- Create Dockerfile

```docker
FROM python:3

COPY ./requirements.txt /app/requirements.txt

WORKDIR /app

RUN pip install --no-cache-dir -r requirements.txt

COPY . /app

ENTRYPOINT [ "python" ]

CMD [ "app.py" ]
```

- Zip and Upload app.zip to Cloud Shell
- Unzip app.zip
- Build Docker image

```bash
docker build -t app:1.0 .
docker image ls
```

- Test image locally on Cloud Shell

```bash
docker container run --name app -p 5000:5000 app:1.0
docker container ls 
docker container ls --all
docker container start app
docker container stop app
```

- Tag the image

```bash
docker tag app:1.0 us.gcr.io/<PROJECT_ID>/app
```

- Push the image to Container Registry

```bash
docker push us.gcr.io/<PROJECT_ID>/app
```

- Deploy the image as container in Cloud Run


# Hands-on with Kubernetes on Cloud

# Steps

- Download and upload the tcb-vote.zip file to Cloud Shell:
https://www.dropbox.com/s/oxsnuzydxubwyhb/tcb-vote-en.zip?dl=0
- Unzip tcb-vote.zip
mkdir kube
mv tcb-vote*.zip kube
cd kube
unzip tcb-vote*.zip
- Set project

```
gcloud config set project <PROJECT_ID>

```

- Build and push the image to Container Registry

```
gcloud services enable cloudbuild.googleapis.com --project <PROJECT_ID>
gcloud builds submit --tag gcr.io/<PROJECT_ID>/tcb-vote-front

```

- Create and Connect to GKE
- Deploy the app to GKE

```
kubectl apply -f tcb-vote-plus-redis-v2.yaml

```

- Enable autoscaling - horizontal pod autoscaling

```
kubectl autoscale deployment tcb-vote-front --cpu-percent=50 --min=1 --max=10
kubectl get hpa
kubectl get pods
kubectl get deployment tcb-vote-front

```

- Simulate user access (increase the load)

```
kubectl run -i --tty load-generator --rm --image=busybox:1.28 --restart=Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- <http://tcb-vote-front>; done"

```

- Watch autoscale

```
kubectl get hpa tcb-vote-front --watch
kubectl get deployment tcb-vote-front

```

- Stop load
CTRL +C on cloud shell tab running the load inscrease
- Watch scale down


# Hands-on with Terraform and Google Cloud

# Steps

- Create the terraform config files
[variables.tf](http://variables.tf/):

```
variable "gcp_project_id" {
  default = "dotted-rookery-321921"
}

variable "gcp_region" {
  default     = "us-east4"
}

```

[main.tf](http://main.tf/):

```
terraform {
  required_providers {
    google = {
      source = "hashicorp/google"
      version = "~> 4.37.0"
    }
  }
}

provider "google" {
    project = var.gcp_project_id
}

resource "google_container_cluster" "primary" {
  name               = "kubernetes-cluster-001"
  location           = var.gcp_region
  initial_node_count = 1
  enable_autopilot = true
  ip_allocation_policy {
  }
}

resource "google_sql_database_instance" "instance" {
  name             = "my-database-instancexxxx"
  region           = var.gcp_region
  database_version = "MYSQL_8_0"
  settings {
    tier = "db-f1-micro"
  }

  deletion_protection  = "true"
}

resource "google_sql_database" "database" {
  name     = "my-database"
  instance = google_sql_database_instance.instance.name
}
```

- Upload them to Cloud Shell
- Run terraform commands to start provisioning

```
terraform init
terraform plan
terraform apply

```

- Run terraform command to destroy it



