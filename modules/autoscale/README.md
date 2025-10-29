# Check Point CloudGuard Network Auto Scaling Terraform module for AWS

Terraform module which deploys an Auto Scaling Group of Check Point Security Gateways into an existing VPC.

These types of Terraform resources are supported:
* [Launch template](https://www.terraform.io/docs/providers/aws/r/launch_template.html)
* [Auto Scaling Group](https://www.terraform.io/docs/providers/aws/r/autoscaling_group.html)
* [Security group](https://www.terraform.io/docs/providers/aws/r/security_group.html)
* [CloudWatch Metric Alarm](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/cloudwatch_metric_alarm)
* [IAM Role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) - conditional creation
* [Proxy Elastic Load Balancer](https://www.terraform.io/docs/providers/aws/r/elb.html) - conditional creation


See the [CloudGuard Auto Scaling for AWS](https://sc1.checkpoint.com/documents/IaaS/WebAdminGuides/EN/CloudGuard_Network_for_AWS_AutoScaling_DeploymentGuide/Default.htm) for additional information

This solution uses the following modules:
- amis

 
## Usage
Follow best practices for using CGNS modules on [the root page](https://registry.terraform.io/modules/lyqwaterway/cloudguard-network-security-cn/aws/latest#:~:text=Best%20Practices%20for%20Using%20Our%20Modules).

**Example:**
```
provider "aws" {}

module "example_module" {

    source  = "lyqwaterway/cloudguard-network-security-cn/aws//modules/autoscale"
    version = "1.0.2"

    // --- Environment ---
    prefix = "env1"
    asg_name = "autoscaling_group"

    // --- VPC Network Configuration ---
    vpc_id = "vpc-12345678"
    subnet_ids = ["subnet-abc123", "subnet-def456"]

    // --- Automatic Provisioning with Security Management Server Settings ---
    gateways_provision_address_type = "private"
    management_server = "mgmt_env1"
    configuration_template = "tmpl_env1"

    // --- EC2 Instances Configuration ---
    gateway_name = "asg_gateway"
    gateway_instance_type = "c5.xlarge"
    key_name = "publickey"
    instances_tags = {
      key1 = "value1"
      key2 = "value2"
    }

    // --- Auto Scaling Configuration ---
    minimum_group_size = 2
    maximum_group_size = 10
    target_groups = ["arn:aws-cn:tg1/abc123", "arn:aws:tg2/def456"]

    // --- Check Point Settings ---
    gateway_version = "R81.20-BYOL"
    admin_shell = "/etc/cli.sh"
    gateway_password_hash = ""
    gateway_maintenance_mode_password_hash = "" # For R81.10 and below the gateway_password_hash is used also as maintenance-mode password.
    gateway_SICKey = "12345678"
    enable_instance_connect = false
    allow_upload_download = true
    enable_cloudwatch = false
    gateway_bootstrap_script = "echo 'this is bootstrap script' > /home/admin/bootstrap.txt"

    // --- Outbound Proxy Configuration (optional) ---
    proxy_elb_type = "internet-facing"
    proxy_elb_clients = "0.0.0.0/0"
    proxy_elb_port = 8080
}
  ```

- Conditional creation
  - To enable cloudwatch for ASG:
  ```
  enable_cloudwatch = true
  ```
  Note: enabling cloudwatch will automatically create IAM role with cloudwatch:PutMetricData permission
  - To create an ASG configuration without a proxy ELB:
  ```
  proxy_elb_type= "none"
  ```

    
    

## Inputs

| Name                                   | Description                                                                                                                                                                           | Type         | Allowed Values                                                                                                                      |
|----------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------------------------------------|
| prefix                                 | (Optional) Instances name prefix                                                                                                                                                      | string       |                                                                                                                                |
| asg_name                               | Autoscaling Group name                                                                                                                                                                | string       |                                                                                                                                |
| vpc_id                                 | The VPC ID in which to deploy                                                                                                                                                         | string       |                                                                                                                                |
| subnet_ids                             | List of public subnet IDs to launch resources into. Recommended at least 2                                                                                                            | list(string) |                                                                                                                                |
| gateways_provision_address_type        | Determines if the gateways are provisioned using their private or public address                                                                                                      | string       | - private<br>- public<br>**Default:** private                                                                                     |
| management_server                      | The name that represents the Security Management Server in the CME configuration                                                                                                      | string       |                                                                                                                                |
| configuration_template                 | Name of the provisioning template in the CME configuration                                                                                                                            | string       |                                                                                                                                |
| gateway_name                           | The name tag of the Security Gateways instances                                                                                                                                       | string       | **Default:** Check-Point-ASG-gateway-tf                                                                                   |
| gateway_instance_type                  | The instance type of the Security Gateways                                                                                                                                            | string       | - c4.large <br/> - c4.xlarge <br/> - c5.large <br/> - c5.xlarge <br/> - c5.2xlarge <br/> - c5.4xlarge <br/> - c5.9xlarge <br/> - c5.12xlarge <br/> - c5.18xlarge <br/> - c5.24xlarge <br/> - c5d.large <br/> - c5d.xlarge <br/> - c5d.2xlarge <br/> - c5d.4xlarge <br/> - c5d.9xlarge <br/> - c5d.12xlarge <br/> - c5d.18xlarge <br/> - c5d.24xlarge <br/> - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.8xlarge <br/> - m5.12xlarge <br/> - m5.16xlarge <br/> - m5.24xlarge <br/> - m6i.large <br/> - m6i.xlarge <br/> - m6i.2xlarge <br/> - m6i.4xlarge <br/> - m6i.8xlarge <br/> - m6i.12xlarge <br/> - m6i.16xlarge <br/> - m6i.24xlarge <br/> - m6i.32xlarge <br/> - c6i.large <br/> - c6i.xlarge <br/> - c6i.2xlarge <br/> - c6i.4xlarge <br/> - c6i.8xlarge <br/> - c6i.12xlarge <br/> - c6i.16xlarge <br/> - c6i.24xlarge <br/> - c6i.32xlarge <br/>  - r5.large <br/> - r5.xlarge <br/> - r5.2xlarge <br/> - r5.4xlarge <br/> - r5.8xlarge <br/> - r5.12xlarge <br/> - r5.16xlarge <br/> - r5.24xlarge <br/> - r5a.large <br/> - r5a.xlarge <br/> - r5a.2xlarge <br/> - r5a.4xlarge <br/> - r5a.8xlarge <br/> - r5a.12xlarge <br/> - r5a.16xlarge <br/> - r5a.24xlarge <br/> - r6i.large <br/> - r6i.xlarge <br/> - r6i.2xlarge <br/> - r6i.4xlarge <br/> - r6i.8xlarge <br/> - r6i.12xlarge <br/> - r6i.16xlarge <br/> - r6i.24xlarge <br/> - r6i.32xlarge <br/> **Default:** c5.xlarge                                                                |
| key_name                               | The EC2 Key Pair name to allow SSH access to the instances                                                                                                                            | string       |                                                                                                                                |
| volume_size                            | Root volume size (GB) - minimum 100                                                                                                                                                   | number       | **Default:** 100                                                                                                          |
| enable_volume_encryption               | Encrypt Environment instances volume with default AWS KMS key                                                                                                                         | bool         | true/false<br>**Default:** true                                                                                                  |
| instances_tags                         | (Optional) A map of tags as key-value pairs. All tags will be added to all AutoScaling Group instances                                                                                 | map(string)  |                                                                                                                                |
| metadata_imdsv2_required               | Set to true to deploy the instance with metadata v2 token required                                                                                                                    | bool         | true/false<br>**Default:** true                                                                                                  |
| minimum_group_size                     | The minimum number of instances in the Auto Scaling group                                                                                                                             | number       | **Default:** 2                                                                                                            |
| maximum_group_size                     | The maximum number of instances in the Auto Scaling group                                                                                                                             | number       | **Default:** 10                                                                                                           |
| target_groups                          | (Optional) List of Target Group ARNs to associate with the Auto Scaling group                                                                                                         | list(string) |                                                                                                                                |
| gateway_version                        | Gateway version and license                                                                                                                                                           | string       | - R81.20-BYOL<br>- R81.20-PAYG-NGTP<br>- R82-BYOL<br>**Default:** R81.20-BYOL                                                    |
| admin_shell                            | Set the admin shell to enable advanced command-line configuration                                                                                                                     | string       | - /etc/cli.sh<br>- /bin/bash<br>- /bin/csh<br>**Default:** /etc/cli.sh                                                           |
| gateway_password_hash                  | (Optional) Admin user's password hash (use command `grub2-mkpasswd-pbkdf2` on Linux to generate a hash)                                                                                | string       |                                                                                                                                |
| gateway_SICKey                         | Secure Internal Communication key for trusted connection between Check Point components (at least 8 alphanumeric characters)                                                          | string       |                                                                                                     |
| enable_instance_connect                | Enable SSH connection over AWS web console. Supported regions can be found [here](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/)         | bool         | true/false<br>**Default:** false                                                                                                 |
| allow_upload_download                  | Automatically download Blade Contracts and other important data to improve the product experience                                                                                     | bool         | true/false<br>**Default:** true                                                                                                  |
| enable_cloudwatch                      | Report Check Point specific CloudWatch metrics                                                                                                                                       | bool         | true/false<br>**Default:** false                                                                                                |
| gateway_bootstrap_script               | (Optional) Semicolon-separated commands to run on the initial boot                                                                                                                    | string       |                                                                                                                                |
| proxy_elb_type                         | Type of ELB to create as an HTTP/HTTPS outbound proxy                                                                                                                                 | string       | - none<br>- internal<br>- internet-facing<br>**Default:** none                                                                   |
| proxy_elb_port                         | The TCP port on which the proxy will be listening                                                                                                                                     | number       | **Default:** 8080                                                                                                         |
| proxy_elb_clients                      | The CIDR range of the clients of the proxy                                                                                                                                            | string       | **Default:** 0.0.0.0/0                                                                                                    |
| gateway_maintenance_mode_password_hash | (Optional) Admin user's password and maintenance-mode password for recovery purposes. Use `grub2-mkpasswd-pbkdf2` on Linux to generate a password hash                                | string       |                                                                                                                                |
 security_rules | List of security rules for ingress and egress.                                                         | list(object({<br/>    direction   = string    <br/>from_port   = any    <br/>to_port     = any <br/>protocol    = any <br/>cidr_blocks = list(any)<br/>}))         | **Default:** []|



## Outputs
To display the outputs defined by the module, create an `outputs.tf` file with the following structure:
```
output "instance_public_ip" {
  value = module.{module_name}.instance_public_ip
}
```
| Name                                           | Description                                                       |
|------------------------------------------------|-------------------------------------------------------------------|
| autoscale_autoscaling_group_name               | The name of the deployed AutoScaling Group                        |
| autoscale_autoscaling_group_arn                | The ARN for the deployed AutoScaling Group                        |
| autoscale_autoscaling_group_availability_zones | The AZs on which the Autoscaling Group is configured              |
| autoscale_autoscaling_group_desired_capacity   | The deployed AutoScaling Group's desired capacity of instances    |
| autoscale_autoscaling_group_min_size           | The deployed AutoScaling Group's minimum number of instances      |
| autoscale_autoscaling_group_max_size           | The deployed AutoScaling Group's maximum number  of instances     |
| autoscale_autoscaling_group_load_balancers     | The deployed AutoScaling Group's configured load balancers        |
| autoscale_autoscaling_group_target_group_arns  | The deployed AutoScaling Group's configured target groups         |
| autoscale_autoscaling_group_subnets            | The subnets on which the deployed AutoScaling Group is configured |
| autoscale_launch_template_id                   | The id of the Launch Template                                     |
| autoscale_autoscale_security_group_id          | The deployed AutoScaling Group's security group id                |
| autoscale_iam_role_name                        | The deployed AutoScaling Group's IAM role name (if created)       |
