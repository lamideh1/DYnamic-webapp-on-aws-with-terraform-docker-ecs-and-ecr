# DYnamic-webapp-on-aws-with-terraform-docker-ecs-and-ecr
Here is a full A–Z step-by-step guide for your mini project: **Hosting a Dynamic Web App on AWS using Terraform Modules, Docker, Amazon ECR, and ECS**.

---

## **A–Z Project Guide**

### **A. Prerequisites**

* AWS CLI installed and configured (`aws configure`)
* Docker installed and running
* Terraform installed
* An AWS IAM user with permissions for ECS, ECR, EC2, IAM, and VPC
* Basic knowledge of terminal commands and Git

---

### **B. Project Structure**

```
terraform-ecs-webapp/
│
├── modules/
│   ├── ecr/
│   │   └── main.tf
│   └── ecs/
│       └── main.tf
│
├── main.tf
├── variables.tf
├── outputs.tf
├── backend.tf (optional if using S3 for state)
├── docker/
│   ├── Dockerfile
│   └── app/ (your app source files go here)
```

---

### **C. Step-by-Step Instructions**

#### **1. Dockerize Your App**

* Create a simple dynamic web app (e.g., Node.js, Flask)
* Place the app inside the `docker/app/` folder
* Create a `Dockerfile` in `docker/`:

```dockerfile
# Example for Node.js
FROM node:18
WORKDIR /app
COPY app/ .
RUN npm install
EXPOSE 3000
CMD ["npm", "start"]
```

* Test locally:

```bash
cd docker
docker build -t webapp .
docker run -p 3000:3000 webapp
```

---

#### **2. Terraform Module: Amazon ECR**

* `modules/ecr/main.tf`:

```hcl
resource "aws_ecr_repository" "webapp" {
  name = var.repository_name
}

output "repository_url" {
  value = aws_ecr_repository.webapp.repository_url
}
```

* `modules/ecr/variables.tf`:

```hcl
variable "repository_name" {}
```

---

#### **3. Terraform Module: Amazon ECS**

* `modules/ecs/main.tf` (simplified):

```hcl
resource "aws_ecs_cluster" "webapp_cluster" {
  name = "webapp-cluster"
}

resource "aws_ecs_task_definition" "webapp" {
  family                   = "webapp-task"
  network_mode            = "awsvpc"
  requires_compatibilities = ["FARGATE"]
  cpu                     = "256"
  memory                  = "512"
  execution_role_arn      = var.execution_role_arn
  container_definitions   = jsonencode([
    {
      name  = "webapp",
      image = var.image_url,
      portMappings = [{
        containerPort = 3000,
        protocol      = "tcp"
      }]
    }
  ])
}

resource "aws_ecs_service" "webapp_service" {
  name            = "webapp-service"
  cluster         = aws_ecs_cluster.webapp_cluster.id
  task_definition = aws_ecs_task_definition.webapp.arn
  launch_type     = "FARGATE"
  desired_count   = 1
  network_configuration {
    subnets         = var.subnets
    assign_public_ip = true
    security_groups = [var.security_group]
  }
}
```

* `modules/ecs/variables.tf`:

```hcl
variable "image_url" {}
variable "execution_role_arn" {}
variable "subnets" { type = list(string) }
variable "security_group" {}
```

---

#### **4. Main Terraform Configuration**

* `main.tf`:

```hcl
provider "aws" {
  region = "us-east-1"
}

module "ecr" {
  source           = "./modules/ecr"
  repository_name  = "webapp-repo"
}

module "ecs" {
  source            = "./modules/ecs"
  image_url         = "${module.ecr.repository_url}:latest"
  execution_role_arn = aws_iam_role.ecs_task_execution_role.arn
  subnets           = [aws_subnet.public1.id]
  security_group    = aws_security_group.webapp_sg.id
}
```

* Include IAM role, VPC, subnets, and SG resources in `main.tf` or separate files.

---

#### **5. Build & Push Docker Image to ECR**

```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <your-ecr-url>

docker build -t webapp docker/
docker tag webapp:latest <your-ecr-url>:latest
docker push <your-ecr-url>:latest
```

---

#### **6. Deploy Infrastructure**

```bash
terraform init
terraform apply
```

---

#### **7. Access the App**

* Get the DNS or IP from the ECS Load Balancer or ECS service output
* Visit it in your browser to confirm the app works

---

#### **8. Clean Up**

```bash
terraform destroy
```

---

#### **9. Documentation (Include in your Report)**

* Screenshots of:

  * Docker build & run
  * Terraform init, apply
  * AWS Console: ECR, ECS, EC2, and Load Balancer
* Observations:

  * Issues faced during Docker build, ECR authentication, Terraform deployment
  * Any workaround or adjustments made

---

Would you like me to provide the actual Terraform and Docker code templates in files for you?
