# Check Point CloudGuard Network Quick Start Auto Scaling Terraform module for AWS

Terraform module which deploys a Check Point CloudGuard Network Security Gateway Auto Scaling Group, an external ALB/NLB, and optionally a Security Management Server and a web server Auto Scaling Group.

These types of Terraform resources are supported:
* [AWS Instance](https://www.terraform.io/docs/providers/aws/r/instance.html)
* [Security Group](https://www.terraform.io/docs/providers/aws/r/security_group.html)
* [Load Balancer](https://www.terraform.io/docs/providers/aws/r/lb.html)
* [Load Balancer Target Group](https://www.terraform.io/docs/providers/aws/r/lb_target_group.html)
* [Launch template](https://www.terraform.io/docs/providers/aws/r/launch_template.html)
* [Auto Scaling Group](https://www.terraform.io/docs/providers/aws/r/autoscaling_group.html)
* [IAM Role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) - conditional creation

See the [Check Point CloudGuard Auto Scaling on AWS](https://aws.amazon.com/quickstart/architecture/check-point-cloudguard/) for additional information

This solution uses the following modules:
- autoscale
- custom_autoscale
- management
- cme_iam_role


## Usage
Follow best practices for using CGNS modules on [the root page](https://registry.terraform.io/modules/lyqwaterway/cloudguard-network-security-cn/aws/latest#:~:text=Best%20Practices%20for%20Using%20Our%20Modules).


**Example:**
```
provider "aws" {}

module "example_module" {

    source  = "lyqwaterway/cloudguard-network-security-cn/aws//examples/qs_autoscale"
    version = "1.0.0"

    //PLEASE refer to README.md for accepted values FOR THE VARIABLES BELOW

    // --- Environment ---
    prefix = "TF"
    asg_name = "asg-qs"

    // --- General Settings ---
    vpc_id = "vpc-12345678"
    key_name = "publickey"
    enable_volume_encryption = true
    enable_instance_connect = false
    disable_instance_termination = false
    allow_upload_download = true
    provision_tag = "quickstart"
    load_balancers_type = "Application Load Balancer"
    load_balancer_protocol = "HTTP"
    certificate = ""
    service_port = "80"
    admin_shell = "/etc/cli.sh"

    // --- Check Point CloudGuard Network Security Gateways Auto Scaling Group Configuration ---
    gateways_subnets = ["subnet-123b5678", "subnet-123a4567"]
    gateway_instance_type = "c5.xlarge"
    gateways_min_group_size = 2
    gateways_max_group_size = 8
    gateway_version = "R81.20-BYOL"
    gateway_password_hash = ""
    gateway_maintenance_mode_password_hash = "" # For R81.10 the gateway_password_hash is used also as maintenance-mode password.
    gateway_SICKey = "12345678"
    enable_cloudwatch = true

    // --- Check Point CloudGuard Network Security Management Server Configuration ---
    management_deploy = true
    management_instance_type = "m5.xlarge"
    management_version = "R81.20-BYOL"
    management_password_hash = ""
    management_maintenance_mode_password_hash = "" # For R81.10 the management_password_hash is used also as maintenance-mode password.
    gateways_policy = "Standard"
    gateways_blades = true
    admin_cidr = "0.0.0.0/0"
    gateways_addresses = "0.0.0.0/0"

    // --- Web Servers Auto Scaling Group Configuration ---
    servers_deploy = false
    servers_subnets = ["subnet-1234abcd", "subnet-56789def"]
    servers_instance_type = "t3.micro"
    server_ami = "ami-12345678"
  }
  ```

- Conditional creation
  - To enable cloudwatch for ASG:
  ```
  enable_cloudwatch = true
  ```
  Note: enabling cloudwatch will automatically create IAM role with cloudwatch:PutMetricData permission
  - To deploy Security Management Server:
  ```
  management_deploy = true
  ```
  - To deploy web servers:
  ```
  servers_deploy = true
  ```
  - To create an ASG configuration without a proxy ELB:
  ```
  proxy_elb_type= "none"
  ```


## Inputs

| Name                                   | Description                                                                                                                                                                          | Type         | Allowed Values                                                                                                                             |
|----------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------------|
| prefix                                 | (Optional) Instances name prefix                                                                                                                                                     | string       |                                                                                                                                        |
| asg_name                               | Autoscaling Group name                                                                                                                                                                | string       |                                                                                                                                        |
| vpc_id                                 | Select an existing VPC                                                                                                                                                               | string       |                                                                                                                                        |
| key_name                               | The EC2 Key Pair name to allow SSH access to the instances                                                                                                                           | string       |                                                                                                                                        |
| enable_volume_encryption               | Encrypt Environment instances volume with default AWS KMS key                                                                                                                        | bool         | true/false<br>**Default:** true                                                                                                          |
| enable_instance_connect                | Enable SSH connection over AWS web console. Supporting regions can be found [here](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/)      | bool         | true/false<br>**Default:** false                                                                                                         |
| disable_instance_termination           | Prevents an instance from accidental termination. Note: Once this attribute is true terraform destroy won't work properly                                                            | bool         | true/false<br>**Default:** false                                                                                                         |
| metadata_imdsv2_required               | Set true to deploy the instance with metadata v2 token required                                                                                                                      | bool         | true/false<br>**Default:** true                                                                                                          |
| allow_upload_download                  | Automatically download Blade Contracts and other important data. Improve product experience by sending data to Check Point                                                           | bool         | true/false<br>**Default:** true                                                                                                          |
| provision_tag                          | The tag used by the Security Management Server to automatically provision the Security Gateways                                                                                       | string       | <br>**Default:** quickstart                                                                                                           |
| load_balancers_type                    | Use Network Load Balancer if you wish to preserve the source IP address or Application Load Balancer for content-based routing                                                        | string       | - Network Load Balancer<br>- Application Load Balancer<br>**Default:** Network Load Balancer                                            |
| load_balancer_protocol                 | The protocol to use on the Load Balancer                                                                                                                                            | string       | Network Load Balancer: TCP, TLS, UDP, TCP_UDP<br>Application Load Balancer: HTTP, HTTPS<br>**Default:** TCP                              |
| certificate                            | Amazon Resource Name (ARN) of an HTTPS Certificate, ignored if the selected protocol is HTTP                                                                                        | string       |                                                                                                                                        |
| service_port                           | The external Load Balancer listens to this port. Leave blank for default: 80 (HTTP) or 443 (HTTPS)                                                                                  | string       |                                                                                                                                        |
| admin_shell                            | Set the admin shell to enable advanced command line configuration                                                                                                                   | string       | - /etc/cli.sh<br>- /bin/bash<br>- /bin/csh<br>**Default:** /etc/cli.sh                                                                  |
| gateways_subnets                       | Select at least 2 public subnets in the VPC. If deploying a Security Management Server, it will be in the first subnet                                                              | list(string) |                                                                                                                                        |
| gateway_instance_type                  | The instance type of the Security Gateways                                                                                                                                           | string       | - c4.large <br/> - c4.xlarge <br/> - c5.large <br/> - c5.xlarge <br/> - c5.2xlarge <br/> - c5.4xlarge <br/> - c5.9xlarge <br/> - c5.12xlarge <br/> - c5.18xlarge <br/> - c5.24xlarge <br/> - c5d.large <br/> - c5d.xlarge <br/> - c5d.2xlarge <br/> - c5d.4xlarge <br/> - c5d.9xlarge <br/> - c5d.12xlarge <br/> - c5d.18xlarge <br/> - c5d.24xlarge <br/> - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.8xlarge <br/> - m5.12xlarge <br/> - m5.16xlarge <br/> - m5.24xlarge <br/> - m6i.large <br/> - m6i.xlarge <br/> - m6i.2xlarge <br/> - m6i.4xlarge <br/> - m6i.8xlarge <br/> - m6i.12xlarge <br/> - m6i.16xlarge <br/> - m6i.24xlarge <br/> - m6i.32xlarge <br/> - c6i.large <br/> - c6i.xlarge <br/> - c6i.2xlarge <br/> - c6i.4xlarge <br/> - c6i.8xlarge <br/> - c6i.12xlarge <br/> - c6i.16xlarge <br/> - c6i.24xlarge <br/> - c6i.32xlarge <br/>  - r5.large <br/> - r5.xlarge <br/> - r5.2xlarge <br/> - r5.4xlarge <br/> - r5.8xlarge <br/> - r5.12xlarge <br/> - r5.16xlarge <br/> - r5.24xlarge <br/> - r5a.large <br/> - r5a.xlarge <br/> - r5a.2xlarge <br/> - r5a.4xlarge <br/> - r5a.8xlarge <br/> - r5a.12xlarge <br/> - r5a.16xlarge <br/> - r5a.24xlarge <br/> - r6i.large <br/> - r6i.xlarge <br/> - r6i.2xlarge <br/> - r6i.4xlarge <br/> - r6i.8xlarge <br/> - r6i.12xlarge <br/> - r6i.16xlarge <br/> - r6i.24xlarge <br/> - r6i.32xlarge <br/>**Default:** c5.xlarge                                                                       |
| gateways_min_group_size                | The minimal number of Security Gateways                                                                                                                                              | number       | <br>**Default:** 2                                                                                                                   |
| gateways_max_group_size                | The maximal number of Security Gateways                                                                                                                                              | number       | <br>**Default:** 10                                                                                                                  |
| gateway_version                        | Gateway version and license                                                                                                                                                          | string       | - R81.20-BYOL<br>- R81.20-PAYG-NGTP<br>- R82-BYOL<br>**Default:** R81.20-BYOL                                                           |
| gateway_password_hash                  | (Optional) Admin user's password hash (use `openssl passwd -6 PASSWORD` to get the hash)                                                                                             | string       |                                                                                                                                        |
| gateway_SIC_Key                        | Secure Internal Communication key for trusted connection between Check Point components                                                                                              | string       | <br>**Default:** 12345678                                                                                                            |
| enable_cloudwatch                      | Report Check Point specific CloudWatch metrics                                                                                                                                      | bool         | true/false<br>**Default:** false                                                                                                        |
| management_deploy                      | Select 'false' to use an existing Security Management Server or deploy one later                                                                                                     | bool         | true/false<br>**Default:** true                                                                                                         |
| management_instance_type               | The EC2 instance type of the Security Management Server                                                                                                                              | string       | - c5.large<br>- c5.xlarge<br>- m6a.large<br>**Default:** m5.xlarge                                                                       |
| management_version                     | The license to install on the Security Management Server                                                                                                                             | string       | - R81.10-BYOL<br>- R81.20-BYOL<br>**Default:** R81.20-BYOL                                                                              |
| gateways_policy                        | The name of the Security Policy package for gateways in the Auto Scaling group                                                                                                       | string       | <br>**Default:** Standard                                                                                                            |
| gateways_blades                        | Turn on the Intrusion Prevention System, Application Control, Anti-Virus, and Anti-Bot Blades                                                                                       | bool         | true/false<br>**Default:** true                                                                                                         |
| admin_cidr                             | (CIDR) Allow web, SSH, and graphical clients only from this network to communicate with the Management Server                                                                        | string       | valid CIDR                                                                                                                              |
| gateway_addresses                      | (CIDR) Allow gateways only from this network to communicate with the Management Server                                                                                               | string       | valid CIDR                                                                                                                              |
| servers_deploy                         | Select 'true' to deploy web servers and an internal Application Load Balancer. 'False' ignores related parameters                                                                    | bool         | true/false<br>**Default:** false                                                                                                        |
| servers_subnets                        | Provide at least 2 private subnet IDs in the chosen VPC                                                                                                                             | list(string) |                                                                                                                                        |
| servers_instance_type                  | The EC2 instance type for the web servers                                                                                                                                            | string       | - t3.nano <br/> - t3.micro <br/> - t3.small <br/> - t3.medium <br/> - t3.large <br/> - t3.xlarge <br/> - t3.2xlarge**Default:** t3.micro                                                                          |
| server_ami                             | Amazon Machine Image ID of a preconfigured web server                                                                                                                                | string       |                                                                                                                                        |
| gateway_maintenance_mode_password_hash | Check Point recommends setting Admin user's password and maintenance-mode password for recovery purposes                                                                             | string       |                                                                                                                                        |
 security_rules | List of security rules for ingress and egress.                                                         | list(object({<br/>    direction   = string    <br/>from_port   = any    <br/>to_port     = any <br/>protocol    = any <br/>cidr_blocks = list(any)<br/>}))         | **Default:** []|

## Outputs
To display the outputs defined by the module, create an `outputs.tf` file with the following structure:
```
output "instance_public_ip" {
  value = module.{module_name}.instance_public_ip
}
```

| Name                             | Description                                                                                                                                                                                                                                                 |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| management_name                  | The deployed Security Management AWS instance name                                                                                                                                                                                                          |
| internal_port                    | The internal Load Balancer should listen to this port                                                                                                                                                                                                       |
| load_balancer_url                | The URL of the external Load Balancer                                                                                                                                                                                                                       |
| external_load_balancer_arn       | The external Load Balancer ARN                                                                                                                                                                                                                              |
| internal_load_balancer_arn       | The internal Load Balancer ARN                                                                                                                                                                                                                              |
| external_LB_target_group_arn     | The external Load Balancer Target Group ARN                                                                                                                                                                                                                 |
| internal_LB_target_group_arn     | The internal Load Balancer Target Group ARN                                                                                                                                                                                                                 |
| autoscale_autoscaling_group_name | The name of the deployed AutoScaling Group                                                                                                                                                                                                                  |
| autoscale_autoscaling_group_arn  | The ARN for the deployed AutoScaling Group                                                                                                                                                                                                                  |
| autoscale_security_group_id      | The deployed AutoScaling Group's security group id                                                                                                                                                                                                          |
| autoscale_iam_role_name          | The deployed AutoScaling Group's IAM role name (if created)                                                                                                                                                                                                 |
| configuration_template           | The name that represents the configuration template. Configurations required to automatically provision the Gateways in the Auto Scaling Group, such as what Security Policy to install and which Blades to enable, will be placed under this template name |
| controller_name                  | The name that represents the controller. Configurations required to connect to your AWS environment, such as credentials and regions, will be placed under this controller name                                                                             |
