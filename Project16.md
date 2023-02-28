## AUTOMATE INFRASTRUCTURE WITH IAC USING TERRAFORM PART 1

In previous implementations, we have built AWS infrastructure for 2 websites manually, it is time to automate the process using Terraform.
The same setup will be built with the power of Infrastructure as Code (IaC).

The architectural design is presented below;

![Arch](https://user-images.githubusercontent.com/114196715/221730434-a0091eb1-4a5f-4057-be8c-02a4194444d3.png)

Prerequisites before you begin writing Terraform code;

- Create an IAM user, name it terraform (ensure that the user has only programatic access to your AWS account) and grant this user AdministratorAccess permissions.

- Copy the secret access key and access key ID. Save them in a notepad temporarily.


- Install Terraform;

```
sudo apt install software-properties-common
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform -y

```

- Configure programmatic access from your workstation to connect to AWS using the access keys copied above and a Python SDK (boto3). You must have Python 3.6 or higher on your workstation.

## STEPS TO INSTALL BOTO3

First install python 3.7 and above as use with python 3.6 is deprecated.

```
sudo apt install software-properties-common
sudo apt-add-repository universe
sudo apt update
sudo apt install python3-pip
pip3 install boto3

```
Check the version of Boto3 installed ; `pip show boto3`

![boto3 installed](https://user-images.githubusercontent.com/114196715/221730881-4aeae960-163f-400f-9fa4-67130204c776.png)

## CONFIGURATION 

Before using Boto3, you need to set up authentication credentials for your AWS account using either the IAM Console or the AWS CLI. We shall install AWS CLI on our terraform instance; ` sudo apt install awscli `

Pass your authentication credentials to AWS CLI; ` aws configure `

![AWS cli configure](https://user-images.githubusercontent.com/114196715/221731028-be500b20-d322-491d-850a-daf33c2de94c.png)

Specify the access key and secret access key, you may also add a default region to the AWS configuration file.

- Create an S3 bucket to store Terraform state file.

When you have configured authentication and installed boto3, make sure you can programmatically access your AWS account by running following commands in >python:

```
import boto3
s3 = boto3.resource('s3')
for bucket in s3.buckets.all():
    print(bucket.name)

```
You should see your previously created S3 bucket name.

![print bucket name](https://user-images.githubusercontent.com/114196715/221731169-78e2ca1c-8a80-4e3f-beb7-8b92fe798d8b.png)

## VPC | SUBNETS | SECURITY GROUPS

* Open your Visual Studio Code and Create a folder called "PBL"

* Create a file in the folder, name it "main.tf"

![tree PBL](https://user-images.githubusercontent.com/114196715/221731304-62825cf1-b038-4b58-af20-6ad35fa035af.png)

## Provider and VPC resource section

* Add AWS as a provider, and a resource to create a VPC in the main.tf file.

* Provider block informs Terraform that we intend to build infrastructure within AWS while the resource block will create a VPC.

```
provider "aws" {
  region = "us-east-1"
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = "172.16.0.0/16"
  enable_dns_support             = "true"
  enable_dns_hostnames           = "true"
  enable_classiclink             = "false"
  enable_classiclink_dns_support = "false"
}

```

* Download necessary plugins for Terraform to work. These plugins are used by providers and provisioners. At this stage, we only have provider in our main.tf file. So, Terraform will just download plugin for AWS provider. 
We shall accomplish this with the ` terraform init ` command.

![terraf init](https://user-images.githubusercontent.com/114196715/221731405-74648e0a-739f-4a36-a3ce-89c5bb052fe3.png)

*OBSERVATIONS*: a new directory has been created: ".terraform\....", This is where Terraform keeps plugins. Generally, it is safe to delete this folder. It just means that one must execute terraform init again, to download them.

![dot terraaf](https://user-images.githubusercontent.com/114196715/221731514-61382d66-43a5-4bb4-aeda-b9c738ab8518.png)

* Create the only resource we just defined; "aws_vpc". We can check what terraform tends to create with ` terraform plan `, after whih we run ` terraform apply ` to create the resource.

![vpc terraf](https://user-images.githubusercontent.com/114196715/221732560-5662f1c8-b3d2-4351-ad73-2f00ec59732a.png)

*OBSERVATIONS*:

- A new file is created "terraform.tfstate". This is how Terraform keeps itself up to date with the exact state of the infrastructure. It reads this file to know what already exists, what should be added, or destroyed based on the entire terraform code that is being developed.

- Another file also gets created during planning and apply. But this file gets deleted immediately, "terraform.tfstate.lock.info". This is what Terraform uses to track, who is running its code against the infrastructure at any point in time. This is very important for teams working on the same Terraform repository at the same time. The lock prevents a user from executing Terraform configuration against the same infrastructure when another user is doing the same – it allows to avoid duplicates and conflicts. It is a json format file that stores information about a user: user’s ID, what operation he/she is doing, timestamp, and location of the state file.

## SUBNETS RESOURCE SECTION

According to our architectural design, we require 6 subnets: 2 public, 2 private for webservers and 2 private for data layer,

* To create the first 2 public subnets, add the below configuration to the main.tf file:

```
# Create public subnets1
    resource "aws_subnet" "public1" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.0.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1a"

}

# Create public subnet2
    resource "aws_subnet" "public2" {
    vpc_id                     = aws_vpc.main.id
    cidr_block                 = "172.16.1.0/24"
    map_public_ip_on_launch    = true
    availability_zone          = "us-east-1b"
}

```

* Run `terraform plan` and `terraform apply`.

![pub sub created](https://user-images.githubusercontent.com/114196715/221731734-ba3d92ee-b5c9-4f87-8c03-81ee6cc754a8.png)

![aws pub sub created](https://user-images.githubusercontent.com/114196715/221731837-6199f36b-e029-45d0-979b-7500da09adaa.png)

*OBSERVATIONS*:

- Hard coded values: Both the availability_zone and cidr_block arguments are hard coded. We should always endeavour to make our work dynamic.

- Multiple Resource Blocks: we have declared multiple resource blocks for each subnet in the code. This is bad coding practice. We need to create a single resource block that can dynamically create resources without specifying multiple blocks. Imagine if we wanted to create 10 subnets, our code would look very clumsy. So, we need to optimize this by introducing a count argument.

Thus, we shall refactor our code.

* Destroy the earlier created infrastructure with `terraform destroy` and press "yes" to confirm the action or ` terraform destroy --auto-approve ` to proceed without confirmation.


## FIXING THE PROBLEMS BY CODE REFACTORING

1. Fixing Hard Coded Values: We will introduce variables, and remove hard coding.

- Starting with the provider block, declare a variable named "region", give it a default value, and update the provider section by referring to the declared variable.

```
variable "REGION" {
        default = "us-east-1"
    }

    provider "aws" {
        region = var.REGION
    }

```

- Do the same to cidr value in the vpc block, and all the other arguments.

```
variable "region" {
        default = "us-east-1"
    }

    variable "vpc_cidr" {
        default = "172.16.0.0/16"
    }

    variable "enable_dns_support" {
        default = "true"
    }

    variable "enable_dns_hostnames" {
        default ="true" 
    }

    variable "enable_classiclink" {
        default = "false"
    }

    variable "enable_classiclink_dns_support" {
        default = "false"
    }

    provider "aws" {
    region = var.REGION
    }

    # Create VPC
    resource "aws_vpc" "main" {
    cidr_block                     = var.vpc_cidr
    enable_dns_support             = var.enable_dns_support 
    enable_dns_hostnames           = var.enable_dns_support
    enable_classiclink             = var.enable_classiclink
    enable_classiclink_dns_support = var.enable_classiclink

    }

```

2. Fixing multiple resource blocks

Terraform has a functionality that allows us to pull data which exposes information to us. Using Terraform’s Data Sources, we can fetch information from outside of terraform, in this case, AWS.

To fetch Availability zones from AWS, we replace the hard coded value in the subnet’s availability_zone section.

```
# Get list of availability zones
        data "aws_availability_zones" "available" {
        state = "available"
        }

```

To make use of this new data resource, we will need to introduce a count argument in the subnet block: 

```
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = "172.16.1.0/24"
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]
	  tags = {
    		Name = "Public subnet ${count.index}"
 	 }
    }

```

The count tells us that we need 2 subnets. Therefore, Terraform will invoke a loop to create 2 subnets. The data resource will return a list object that contains a list of AZ which in our case; us-east-1, is 6.

Running terraform with the snippet above may be successful when terraform runs the loop the first time but will encounter an error the second and subsequent times as the cidr block was hard-coded. No two VPC will have the same cidr block. Hence, the need for cidrsubnet function.

```
 # Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = 2
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }

```

Finally, to remove the hardcoded count value, we can introuduce length() function, which basically determines the length of a given list, map, or string.

Update the snippet to contain the length function;

```
# Create public subnet1
    resource "aws_subnet" "public" { 
        count                   = length(data.aws_availability_zones.available.names)
        vpc_id                  = aws_vpc.main.id
        cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
        map_public_ip_on_launch = true
        availability_zone       = data.aws_availability_zones.available.names[count.index]

    }

```

With the above, the length function returns a value of 6 considering the available AZs in our chosen region, US-EAST-1. Our required number of subnets is 2. Hence, declare a variable;

```
variable "preferred_number_of_public_subnets" {
  default = 2
}

```
stating the desired number of subnets required and a conditional statement as shown below;

```
# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}

```

- The first part var.preferred_number_of_public_subnets == null checks if the value of the variable is set to null or has some value defined.

- The second part ? and length(data.aws_availability_zones.available.names) means, if the first part is true, then use this. In other words, if preferred number of public subnets is null (Or not known) then set the value to the data returned by lenght function.

- The third part ; var.preferred_number_of_public_subnets means, if the first condition is false, i.e preferred number of public subnets is not null then set the value to whatever is definied in var.preferred_number_of_public_subnets.

The entire configuration should now look like the below;

```
# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = 2
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]

}

```

## Introducing variables.tf & terraform.tfvars

Instead of having a long list of variable in our main.tf file, we could revamp our code to make it more readable and reusable by creating a variable.tf file wherein our variables shall be defined as well as a terraform.tfvars to contain our default values. The whole snippet is presented below;

MAIN.TF FILE

```
# Get list of availability zones
data "aws_availability_zones" "available" {
state = "available"
}

provider "aws" {
  region = var.region
}

# Create VPC
resource "aws_vpc" "main" {
  cidr_block                     = var.vpc_cidr
  enable_dns_support             = var.enable_dns_support 
  enable_dns_hostnames           = var.enable_dns_support
  enable_classiclink             = var.enable_classiclink
  enable_classiclink_dns_support = var.enable_classiclink

}

# Create public subnets
resource "aws_subnet" "public" {
  count  = var.preferred_number_of_public_subnets == null ? length(data.aws_availability_zones.available.names) : var.preferred_number_of_public_subnets   
  vpc_id = aws_vpc.main.id
  cidr_block              = cidrsubnet(var.vpc_cidr, 4 , count.index)
  map_public_ip_on_launch = true
  availability_zone       = data.aws_availability_zones.available.names[count.index]
}

```

VARIABLES.TF

```
variable "region" {
      default = "eu-central-1"
}

variable "vpc_cidr" {
    default = "172.16.0.0/16"
}

variable "enable_dns_support" {
    default = "true"
}

variable "enable_dns_hostnames" {
    default ="true" 
}

variable "enable_classiclink" {
    default = "false"
}

variable "enable_classiclink_dns_support" {
    default = "false"
}

  variable "preferred_number_of_public_subnets" {
      default = null
}

```

TERRAFORM.TFVARS

```
region = "eu-central-1"

vpc_cidr = "172.16.0.0/16" 

enable_dns_support = "true" 

enable_dns_hostnames = "true"  

enable_classiclink = "false" 

enable_classiclink_dns_support = "false" 

preferred_number_of_public_subnets = 2

```

![pub 0 and 1](https://user-images.githubusercontent.com/114196715/221732285-3415f78e-4765-4c7f-991f-5e4564d1a87f.png)

ADDITONAL TASK.

When the preferred number of subnet was defined in the variable.tf file as null, given the condition passed ,six subnets were created as the length function returned the value "6" .

![6 subnet provisoned](https://user-images.githubusercontent.com/114196715/221732409-d0e621dc-df33-422a-bdc8-93fd44a63238.png)
