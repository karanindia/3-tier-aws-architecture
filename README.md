# Traditional 3-Tier Architecture in AWS

### Overview
This solution presents a traditional 3-tier architecture in AWS, which is consisted of presentation, application and data tiers. The customer query comes through an external ALB, then autoscaling set of Nginx web servers serve it on the presentation level. The presentation tier instances talk to the application tier via the internal ALB, which has an autoscaling group of application instances behind itself. Each application tier instance has the app which runs in a docker container. Only application tier instances can communicate to the RDS database on the data tier. Communications on each level are secured with security groups so every layer can only connect to the corresponding endpoints only required for its functioning.       

### How to Deploy

- Create a SSL certificate in AWS Certificate Manager and add its ARN to "ssl_certificate_arn" variable

- Create a S3 bucket, which will be used to store the Terraform state file. Please use a different name for the S3 bucket from the one in the following example:
```
$ aws s3 mb s3://3-tier-aws-terraform-state
```
- Clone the git repository:
```
$ git clone https://github.com/asabitov/3-tier-aws-architecture.git
```
- Prepare for deployment: go to the "terraform" directory and set variables in "terraform.tfvar" file:
```
$ cd 3-tier-aws-architecture/terraform
$ cp terraform.tfvar.template terraform.tfvar
$ vi terraform.tfvar
```
- Please note that you would have to have a real SSL certificate installed in the "AWS Certificate Manager", while deploying the solution, otherwise you will see an error. If you don't have a SSL certificate, please amend the corresponding section for the resource "aws_lb_listener" in "presentation_tier.tf". 
- Deploy:
``` 
$ terraform init -backend-config="bucket=3-tier-aws-terraform-state"
$ terraform apply -var-file="terraform.tfvars"
```

### How to Test
- Run "curl" on your host or use a web browser to go to the external ALB (the external ALB output name is "presentation_alb"). Please note that you would have to use a real SSL certificate while deploying the solution, otherwise you will see an error.
- SSH to the bastion (the bastion IP address is displayed in output as "bastion_ip_address"), then SSH to a presentation tier instance and run "curl" to query the internal load-balancer (the internal ALB output name is "application_alb") 
- SSH to the bastion (the bastion IP address is displayed in output as "bastion_ip_address"), then SSH to an application tier instance and run "telnet" to the RDS instance on port 5432 to connect to the database (the RDS instance name output name is "database_endpoint") 
     
