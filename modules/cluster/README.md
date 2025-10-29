# Check Point CloudGuard Network Security Cluster Terraform module for AWS

Terraform module which deploys a Check Point CloudGuard Network Security Cluster into an existing VPC on AWS.

These types of Terraform resources are supported:
* [AWS Instance](https://www.terraform.io/docs/providers/aws/r/instance.html)
* [Security Group](https://www.terraform.io/docs/providers/aws/r/security_group.html)
* [Network interface](https://www.terraform.io/docs/providers/aws/r/network_interface.html)
* [EIP](https://www.terraform.io/docs/providers/aws/r/eip.html) - conditional creation
* [Route](https://www.terraform.io/docs/providers/aws/r/route.html) - conditional creation
* [IAM Role](https://www.terraform.io/docs/providers/aws/r/iam_role.html) - conditional creation

See the [Deploying a Check Point Cluster in AWS (Amazon Web Services)](https://supportcenter.checkpoint.com/supportcenter/portal?eventSubmit_doGoviewsolutiondetails=&solutionid=sk104418) for additional information

This solution uses the following modules:
- amis
- cluster_iam_role

## Usage
Follow best practices for using CGNS modules on [the root page](https://registry.terraform.io/modules/lyqwaterway/cloudguard-network-security-cn/aws/latest#:~:text=Best%20Practices%20for%20Using%20Our%20Modules).

**Example:**
```
provider "aws" {}

module "example_module" {

    source  = "lyqwaterway/cloudguard-network-security-cn/aws//modules/cluster"
    version = "1.0.2"

    // --- VPC Network Configuration ---
    vpc_id = "vpc-12345678"
    public_subnet_id = "subnet-123456"
    private_subnet_id = "subnet-345678"
    private_route_table = "rtb-12345678"

    // --- EC2 Instance Configuration ---
    gateway_name = "Check-Point-Cluster-tf"
    gateway_instance_type = "c5.xlarge"
    key_name = "publickey"
    allocate_and_associate_eip = true
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
  - To create an Elastic IP and associate it to each cluster member instance:
  ```
  allocate_and_associate_eip = "true"
  ```
  - To create IAM Role:
  ```
  predefined_role = ""
  ```
  - To create route from '0.0.0.0/0' to the Active Cluster member instance, please provide route table:
  ```
  private_route_table = "rtb-12345678"
  ```

  ### Optional re-deploy of cluster member:
  In case of re-deploying one cluster member, make sure that it's in STANDBY state, and the second member is the ACTIVE one.
  Follow the commands below in order to re-deploy (replace MEMBER with a or b):
  - terraform taint aws_instance.member-MEMBER-instance
  - terraform plan (review the changes)
  - terraform apply
  - In Smart Console: reset SIC with the re-deployed member and install policy

## Inputs
# Parameters Description

| Name | Description | Type | Allowed values |
|------|-------------|------|----------------|
| **vpc_id** | The VPC ID in which to deploy | string | Required |
| **public_subnet_id** | The public subnet of the cluster. The cluster's public IPs will be generated from this subnet | string | Required |
| **private_subnet_id** | The private subnet of the cluster. The cluster's private IPs will be generated from this subnet | string | Required |
| **private_route_table** | (Optional) Set 0.0.0.0/0 route to the Active Cluster member instance in this route table (e.g. rtb-12a34567). Route table cannot have an existing 0.0.0.0/0 route | string | **Default:** "" |
| **gateway_name** | (Optional) The name tag of the Security Gateway instances | string | **Default:** "Check-Point-Cluster-tf" |
| **gateway_instance_type** | The instance type of the Security Gateways | string | - c4.large <br/> - c4.xlarge <br/> - c5.large <br/> - c5.xlarge <br/> - c5.2xlarge <br/> - c5.4xlarge <br/> - c5.9xlarge <br/> - c5.12xlarge <br/> - c5.18xlarge <br/> - c5.24xlarge <br/> - c5d.large <br/> - c5d.xlarge <br/> - c5d.2xlarge <br/> - c5d.4xlarge <br/> - c5d.9xlarge <br/> - c5d.12xlarge <br/> - c5d.18xlarge <br/> - c5d.24xlarge <br/> - m5.large <br/> - m5.xlarge <br/> - m5.2xlarge <br/> - m5.4xlarge <br/> - m5.8xlarge <br/> - m5.12xlarge <br/> - m5.16xlarge <br/> - m5.24xlarge <br/> - m6i.large <br/> - m6i.xlarge <br/> - m6i.2xlarge <br/> - m6i.4xlarge <br/> - m6i.8xlarge <br/> - m6i.12xlarge <br/> - m6i.16xlarge <br/> - m6i.24xlarge <br/> - m6i.32xlarge <br/> - c6i.large <br/> - c6i.xlarge <br/> - c6i.2xlarge <br/> - c6i.4xlarge <br/> - c6i.8xlarge <br/> - c6i.12xlarge <br/> - c6i.16xlarge <br/> - c6i.24xlarge <br/> - c6i.32xlarge <br/>  - r5.large <br/> - r5.xlarge <br/> - r5.2xlarge <br/> - r5.4xlarge <br/> - r5.8xlarge <br/> - r5.12xlarge <br/> - r5.16xlarge <br/> - r5.24xlarge <br/> - r5a.large <br/> - r5a.xlarge <br/> - r5a.2xlarge <br/> - r5a.4xlarge <br/> - r5a.8xlarge <br/> - r5a.12xlarge <br/> - r5a.16xlarge <br/> - r5a.24xlarge <br/> - r6i.large <br/> - r6i.xlarge <br/> - r6i.2xlarge <br/> - r6i.4xlarge <br/> - r6i.8xlarge <br/> - r6i.12xlarge <br/> - r6i.16xlarge <br/> - r6i.24xlarge <br/> - r6i.32xlarge <br/> **Default:** "c5.xlarge" |
| **key_name** | The EC2 Key Pair name to allow SSH access to the instance | string | Required |
| **allocate_and_associate_eip** | If true, an Elastic IP will be allocated and associated with each cluster member, in addition to the shared cluster Elastic IP | bool | true;<br/>false.<br/>**Default:** true |
| **volume_size** | Root volume size (GB) - minimum 100 | number | **Default:** 100 |
| **volume_encryption** | KMS or CMK key identifier. Use key ID, alias, or ARN. Key alias should be prefixed with 'alias/' | string | **Default:** "alias/aws/ebs" |
| **enable_instance_connect** | Enable AWS Instance Connect. Supported regions are listed [here](https://aws.amazon.com/about-aws/whats-new/2019/06/introducing-amazon-ec2-instance-connect/) | bool | true;<br/>false.<br/>**Default:** false |
| **disable_instance_termination** | Prevents an instance from accidental termination. Note: Setting this true may prevent `terraform destroy` from working properly | bool | true;<br/>false.<br/>**Default:** false |
| **metadata_imdsv2_required** | Set true to deploy the instance with metadata v2 token required | bool | true;<br/>false.<br/>**Default:** true<br/>Required |
| **instance_tags** | (Optional) A map of tags as key=value pairs. Tags will be added to the Gateway EC2 instances | map(string) | **Default:** {} |
| **predefined_role** | (Optional) A predefined IAM role to attach to the cluster profile | string | **Default:** "" |
| **gateway_version** | Gateway version and license | string | "R81.10-BYOL";<br/>"R81.20-BYOL";<br/>"R82-BYOL".<br/>**Default:** "R81.20-BYOL" |
| **admin_shell** | Set the admin shell to enable advanced command line configuration | string | "/etc/cli.sh";<br/>"/bin/bash";<br/>"/bin/csh".<br/>**Default:** "/etc/cli.sh" |
| **gateway_SICKey** | The Secure Internal Communication key for trusted connection between Check Point components. Must be at least 8 alphanumeric characters | string | **Default:** "12345678" |
| **gateway_password_hash** | (Optional) Admin user's password hash (use `openssl passwd -6 PASSWORD` to generate) | string | **Default:** "" |
| **memberAToken** | (Recommended) Token for quick connection to Smart-1 Cloud. Instructions in SK180501 | string | **Default:** "" |
| **memberBToken** | (Recommended) Token for quick connection to Smart-1 Cloud. Instructions in SK180501 | string | **Default:** "" |
| **resources_tag_name** | (Optional) Name tag prefix for resources | string | **Default:** "" |
| **gateway_hostname** | (Optional) Hostname will append 'member-a' or 'member-b' accordingly. Cannot contain reserved words (see SK40179 for details) | string | **Default:** "" |
| **allow_upload_download** | Automatically download Blade Contracts and other important data to enhance product experience | bool | true;<br/>false.<br/>**Default:** true |
| **enable_cloudwatch** | Report Check Point-specific CloudWatch metrics | bool | true;<br/>false.<br/>**Default:** false |
| **gateway_bootstrap_script** | (Optional) Semicolon (;) separated commands to run on the initial boot | string | **Default:** "" |
| **primary_ntp** | (Optional) IPv4 address of the primary Network Time Protocol server | string | **Default:** "169.254.169.123" |
| **secondary_ntp** | (Optional) IPv4 address of the secondary Network Time Protocol server | string | **Default:** "0.pool.ntp.org" |
| **gateway_maintenance_mode_password_hash** | (Optional) Maintenance-mode password hash (generate using `grub2-mkpasswd-pbkdf2` on Linux) | string | **Default:** "" |

## Outputs
To display the outputs defined by the module, create an `outputs.tf` file with the following structure:
```
output "instance_public_ip" {
  value = module.{module_name}.instance_public_ip
}
```
| Name               | Description                                         |
|--------------------|-----------------------------------------------------|
| ami_id             | The ami id of the deployed Security Cluster members |
| cluster_public_ip  | The public address of the cluster                   |
| member_a_public_ip | The public address of member A                      |
| member_b_public_ip | The public address of member B                      |
| member_a_ssh       | SSH command to member A                             |
| member_b_ssh       | SSH command to member B                             |
| member_a_url       | URL to the member A portal                          |
| member_b_url       | URL to the member B portal                          |
