# EC2 


> **Tasks**:

> - [Task 1: Provider](#task1)
> - [Task 2: Backend](#task2)
> - [Task 3: Data sources](#task3)
> - [Task 4: Variables](#task4)
> - [Task 5: Security Groups](#task5)
> - [Task 6: Key Pair](#task6)
> - [Task 7: EC2](#task7)

### </a> <a name="task1"></a>Getting set up

Start by exporting the right profile and region to start working with terraform:
```sh
export AWS_PROFILE="cloudreach-trainer-lab"
export AWS_DEFAULT_REGION="us-east-1"
```

Go to GitHub to create a new repository for the EC2 project.

Clone your EC2 repository to start working on the terraform code:

```sh
git clone github.com/terraform/new-EC2-project-folder
cd new-ec2-project-folder
```

### <a name="task2"></a> Provider

Start by setting up your provider to define which cloud plugins our project will require to deploy the resources.

Create a new file called `provider.tf`

```javascript
provider "aws" {
  region = "us-east-1"
}
```

### <a name="task3"></a> Backend

Create a `backend.tf` file to select your bucket, key and lock table for your project. 

```javascript
terraform {
  backend "s3" {
    bucket = "talent-academy-account_id-tfstates-pascal"
    key    = "projects/ec2/terraform.tfstates"
    dynamodb_table = "terraform-lock"
  }
}
```

Do not forget to update the `key` value to keep it separate from other projects.

### <a name="task4"></a> Data Sources

Create a `main.tf` file to collect information about existing resources in AWS and be able to use them within the terraform code.

E.G. Collecting the AMI id to be used for the instance creations.

```javascript
data "aws_ami" "aws_basic_linux" {
  owners      = [var.aws_owner_id]
  most_recent = true
  filter {
    name   = "name"
    values = [var.aws_ami_name]
  }
}
```

Use this example to create data sources for the following resources:
* VPC
* Public Subnet

### <a name="task5"></a> Variables

All variables needs to be defined first before being assigned a value. It's best practice to create a file called `variables.tf` which will define all the variables.

E.G of `variables.tf` :

```javascript
# DEFINING ALL VARIABLES
variable "aws_owner_id" {
  description = "Contains the Owner ID of the ami for amazon linux"
  type        = string
}

variable "aws_ami_name" {
  description = "Name of the ami I want for my project"
  type        = string
}
```

And use the `terraform.tfvars` to assign values to your variables:
```javascript
# ASSIGNING VALUE TO VARIABLES
aws_owner_id       = "137112412989"
aws_ami_name       = "amzn2-ami-kernel-5.10-hvm*"
```

### <a name="task6"></a> Security Groups

Create a new file called `security_groups.tf` that creates connectivity rules to allow inbound and outbound access to the EC2 instance.

E.G. for SSH access ingress and internet access in egress.

```javascript
resource "aws_security_group" "my_public_app_sg" {
  name        = "my_public_app_sg"
  description = "Allow access to this server"
  vpc_id      = data.aws_vpc.main_vpc.id

  # INBOUND CONNECTIONS
  ingress {
    description = "Allow SSH into the EC2"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  # OUTBOUT CONNECTIONS
  egress {
    description = "Allow access to the world"
    from_port   = 0
    to_port     = 0
    protocol    = "-1" # TCP + UDP
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

### </a> <a name="task7"></a> Key Pair

Make sure you have a key pair created in AWS console. The private key needs to exists in your local machines at the default ssh keys location. 
E.G. key location and changing permission
```sh
cat ~/.ssh/aws_keypair
chmod 600 ~/.ssh/aws_keypair
```

### </a> <a name="task8"></a> EC2 instances

Create the EC2 instance using all the other data sources and resources:

```javascript
resource "aws_instance" "my_public_server" {
  ami                    = data.aws_ami.aws_basic_linux.id
  instance_type          = var.ec2_type
  subnet_id              = data.aws_subnet.public.id
  vpc_security_group_ids = [aws_security_group.my_public_app_sg.id]
  key_name               = var.my_keypair

  tags = {
      Name = "public_server"
  }
}
```
