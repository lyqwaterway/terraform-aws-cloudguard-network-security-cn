# Check Point CloudGuard Network Security Cross AZ Cluster Master Terraform module for AWS

Terraform module which deploys a Check Point CloudGuard Network Security Cross AZ Cluster into into a new VPC.

These types of Terraform resources are supported:
* [AWS Instance](https://www.terraform.io/docs/providers/aws/r/instance.html)
* [VPC](https://www.terraform.io/docs/providers/aws/r/vpc.html)
* [Security Group](https://www.terraform.io/docs/providers/aws/r/security_group.html)
* [Network interface](https://www.terraform.io/docs/providers/aws/r/network_interface.html)
* [Route](https://www.terraform.io/docs/providers/aws/r/route.html)
* [EIP](https://www.terraform.io/docs/providers/aws/r/eip.html)
* [IAM Role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) - conditional creation

See the [Deploying a Check Point Cluster in AWS (Amazon Web Services)](https://sc1.checkpoint.com/documents/IaaS/WebAdminGuides/EN/CP_CloudGuard_for_AWS_Cross_AZ_Cluster/Default.htm) for additional information

This solution uses the following modules:
- cross_az_cluster
- amis
- vpc
- cluster_iam_role

## Usage
Follow best practices for using CGNS modules on [the root page](https://registry.terraform.io/modules/lyqwaterway/cloudguard-network-security-cn/aws/latest#:~:text=Best%20Practices%20for%20Using%20Our%20Modules).

**Instead of the standard terraform apply command, use the following:**
```
  terraform apply -target=module.{module_name}.aws_route_table.private_subnet_rtb -auto-approve && terraform apply 
  ```

**Example:**
```
provider "aws" {}

module "example_module" {

    source  = "lyqwaterway/cloudguard-network-security-cn/aws//modules/cross_az_cluster_master"
    version = "1.0.2"

    // --- VPC Network Configuration ---
    vpc_cidr = "10.0.0.0/16"
    public_subnets_map = {
      "cn-northwest-1a" = 1
      "cn-northwest-1b" = 2
    }
    private_subnets_map = {
      "cn-northwest-1a" = 3
      "cn-northwest-1b" = 4
    }
    subnets_bit_length = 8


    
    // --- EC2 Instance Configuration ---
    gateway_name = "Check-Point-Cluster-tf"
    gateway_instance_type = "c5.xlarge"
    key_name = "publickey"
    volume_size = 100
    volume_encryption = "alias/aws/ebs"
    enable_instance_connect = false
    disable_instance_termination = false
    instance_tags = {
      key1 = "value1"
      key2 = "value2"
    }
    predefined_role = ""

    // --- Check Point Settings ---
    gateway_version = "R81.20-BYOL"
    admin_shell = "/etc/cli.sh"
    gateway_SICKey = "12345678"
    gateway_password_hash = ""
    gateway_maintenance_mode_password_hash = "" # For R81.10 and below the gateway_password_hash is used also as maintenance-mode password.

    // --- Quick connect to Smart-1 Cloud (Recommended) ---
    memberAToken = ""
    memberBToken = ""
  
    // --- Advanced Settings ---
    resources_tag_name = "tag-name"
    gateway_hostname = "gw-hostname"
    allow_upload_download = true
    enable_cloudwatch = false
    gateway_bootstrap_script = "echo 'this is bootstrap script' > /home/admin/bootstrap.txt"
    primary_ntp = ""
    secondary_ntp = ""
}
  ```

- Conditional creation
  - To create IAM Role:
  ```
  predefined_role = ""
  ```

  ### Optional re-deploy of cluster member:
  In case of re-deploying one cluster member, make sure that it's in STANDBY state, and the second member is the ACTIVE one.
  Follow the commands below in order to re-deploy (replace MEMBER with a or b):
  - terraform taint module.launch_cluster_into_vpc.aws_instance.member-MEMBER-instance
  - terraform plan (review the changes)
  - terraform apply
  - In Smart Console: reset SIC with the re-deployed member and install policy

## Inputs

| Name                                   | Description                                                                                                                                                 | Type         | Allowed Values                                                                                         |
|----------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------|-------------------------------------------------------------------------------------------------------|
| vpc_cidr                               | The CIDR block of the VPC                                                                                                                                  | string       |                                                                                                    |
| public_subnets_map                     | Map of {availability-zone = subnet-suffix-number}. Minimum 2 pairs. Example: {"cn-northwest-1a" = 1, "cn-northwest-1b" = 2}                                           | map          |                                                                                                    |
| private_subnets_map                    | Map of {availability-zone = subnet-suffix-number}. Minimum 2 pairs. Example: {"cn-northwest-1a" = 3, "cn-northwest-1b" = 4}                                           | map          |                                                                                                    |
| subnets_bit_length                     | Number of additional bits to extend the VPC CIDR. For example, /16 CIDR with 4-bit length results in /20 subnets.                                           | number       |                                                                                                    |
| gateway_name                           | (Optional) The name tag of the Security Gateway instances                                                                                                  | string       | **Default:** Check-Point-Cluster-tf                                                          |
| gateway_instance_type                  | The instance type of the Security Gateways                                                                                                                 | string       | - c4.large <br/> - c4.xlarge <br/> - c5.large <br/> - c5.xlarge <br/> - c5.2xlarge <br/> - c5.4xlarge <br/> - c5.9xlarge <br/> - c5.12xlarge <br/> - c5.18xlarge <br/> - c5.24xlarge <br/> - c5d.large <br/> - c5d.xlarge <br/> - c5d.2xlarge <br/> - c5d.4xlarge <br/> - c5d.9xlarge <br/> - c5d.12xlarge <br/> - c5d.18xlarge <br/> - c5d.24xlarge <br/> - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.8xlarge <br/> - m5.12xlarge <br/> - m5.16xlarge <br/> - m5.24xlarge <br/> - m6i.large <br/> - m6i.xlarge <br/> - m6i.2xlarge <br/> - m6i.4xlarge <br/> - m6i.8xlarge <br/> - m6i.12xlarge <br/> - m6i.16xlarge <br/> - m6i.24xlarge <br/> - m6i.32xlarge <br/> - c6i.large <br/> - c6i.xlarge <br/> - c6i.2xlarge <br/> - c6i.4xlarge <br/> - c6i.8xlarge <br/> - c6i.12xlarge <br/> - c6i.16xlarge <br/> - c6i.24xlarge <br/> - c6i.32xlarge <br/>  - r5.large <br/> - r5.xlarge <br/> - r5.2xlarge <br/> - r5.4xlarge <br/> - r5.8xlarge <br/> - r5.12xlarge <br/> - r5.16xlarge <br/> - r5.24xlarge <br/> - r5a.large <br/> - r5a.xlarge <br/> - r5a.2xlarge <br/> - r5a.4xlarge <br/> - r5a.8xlarge <br/> - r5a.12xlarge <br/> - r5a.16xlarge <br/> - r5a.24xlarge <br/> - r6i.large <br/> - r6i.xlarge <br/> - r6i.2xlarge <br/> - r6i.4xlarge <br/> - r6i.8xlarge <br/> - r6i.12xlarge <br/> - r6i.16xlarge <br/> - r6i.24xlarge <br/> - r6i.32xlarge <br/>**Default:** c5.xlarge                                    |
| key_name                               | The EC2 Key Pair name to allow SSH access to the instance                                                                                                  | string       |                                                                                                    |
| volume_size                            | Root volume size (GB) - minimum 100                                                                                                                       | number       | **Default:** 100                                                                             |
| volume_type                            | General Purpose SSD Volume Type                                                                                                                           | string       | - gp3<br>- gp2<br>**Default:** gp3                                                                  |
| volume_encryption                      | KMS or CMK key identifier. Use key ID, alias, or ARN. Prefix key alias with 'alias/' (e.g., alias/aws/ebs).                                                | string       | **Default:** alias/aws/ebs                                                                   |
| enable_instance_connect                | Enable AWS Instance Connect. [More info](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/)                       | bool         | true/false<br>**Default:** false                                                                    |
| disable_instance_termination           | Prevents accidental termination. Note: Enabling this will prevent proper `terraform destroy`.                                                              | bool         | true/false<br>**Default:** false                                                                    |
| metadata_imdsv2_required               | Set true to deploy the instance with metadata v2 token required                                                                                          | bool         | true/false<br>**Default:** true                                                                     |
| instance_tags                          | (Optional) A map of tags as key-value pairs. Applied to Gateway EC2 Instances                                                                              | map(string)  |                                                                                                    |
| predefined_role                        | (Optional) A predefined IAM role to attach to the cluster profile                                                                                          | string       |                                                                                                    |
| gateway_version                        | Gateway version and license                                                                                                                                | string       | - R81.20-BYOL<br>- R82-BYOL<br>**Default:** R81.20-BYOL                                              |
| admin_shell                            | Set the admin shell to enable advanced command line configuration                                                                                          | string       | - /etc/cli.sh<br>- /bin/bash<br>- /bin/csh<br>**Default:** /etc/cli.sh                               |
| gateway_SICKey                         | The Secure Internal Communication key for trusted connection between Check Point components. Minimum 8 alphanumeric characters.                           | string       |                                                                        |
| gateway_password_hash                  | (Optional) Admin user's password hash.                                                                                                                    | string       |                                                                                                    |
| memberAToken                           | (Recommended) Token for quick connection to Smart-1 Cloud. Follow instructions in SK180501.                                                               | string       |                                                                                                    |
| memberBToken                           | (Recommended) Token for quick connection to Smart-1 Cloud. Follow instructions in SK180501.                                                               | string       |                                                                                                    |
| resources_tag_name                     | (Optional) Name tag prefix of the resources                                                                                                               | string       |                                                                                                    |
| gateway_hostname                       | (Optional) Host name will be appended with member-a/b. Must not contain reserved words. Refer to SK40179.                                                 | string       |                                                                                                    |
| allow_upload_download                  | Automatically download Blade Contracts and other important data.                                                                                          | bool         | true/false<br>**Default:** true                                                                     |
| enable_cloudwatch                      | Report Check Point specific CloudWatch metrics                                                                                                            | bool         | true/false<br>**Default:** false                                                                    |
| gateway_bootstrap_script               | (Optional) Semicolon (;) separated commands to run on the initial boot                                                                                    | string       |                                                                                                    |
| primary_ntp                            | (Optional) IPv4 address of the primary Network Time Protocol server                                                                                       | string       | **Default:** 169.254.169.123                                                                |
| secondary_ntp                          | (Optional) IPv4 address of the secondary Network Time Protocol server                                                                                     | string       | **Default:** 0.pool.ntp.org                                                                 |
| gateway_maintenance_mode_password_hash | (Optional) Password hash for maintenance mode. Follow Check Point's recommendations.                                                                      | string       |                                                                                                    |
 security_rules | List of security rules for ingress and egress.                                                         | list(object({<br/>    direction   = string    <br/>from_port   = any    <br/>to_port     = any <br/>protocol    = any <br/>cidr_blocks = list(any)<br/>}))         | **Default:** []|



## Outputs
To display the outputs defined by the module, create an `outputs.tf` file with the following structure:
```
output "instance_public_ip" {
  value = module.{module_name}.instance_public_ip
}
```
| Name               | Description                                                  |
|--------------------|--------------------------------------------------------------|
| ami_id             | The ami id of the deployed Security Cross AZ Cluster members |
| cluster_public_ip  | The public address of the cluster                            |
| member_a_public_ip | The public address of member A                               |
| member_b_public_ip | The public address of member B                               |
| member_a_ssh       | SSH command to member A                                      |
| member_b_ssh       | SSH command to member B                                      |
| member_a_url       | URL to the member A portal                                   |
| member_b_url       | URL to the member B portal                                   |
