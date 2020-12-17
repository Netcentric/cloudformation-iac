# AWS CloudFormation IaC Templates

These CloudFormation Templates are designed to simplify the creation of AWS resources and create complex environments with low effort.

# Table of Content
- [AWS CloudFormation IaC Templates](#aws-cloudformation-iac-templates)
- [Table of Content](#table-of-content)
- [Feature summary](#feature-summary)
- [Getting started](#getting-started)
  - [Requirements](#requirements)
  - [Installation](#installation)
- [Quickstart](#quickstart)
  - [Example project](#example-project)
- [About](#about)
  - [Development guidelines](#development-guidelines)
  - [Authors / Contributors](#authors--contributors)

# Feature summary

- ALB setup with granular settings
- Automated EBS backups
- Automated uptime (start/stop instances using schedules, reducing costs)
- Certificate Management
- Configurable resource naming
- Custom tagging (e.g. to feed config management tools or firewalls)
- EC2 Instance creation with custom EBS settings
- Route53 DNS management, locally and remotely (in a different account)
- Support for fully encrypted setups (full HTTPS + EBS encryption)
- Support for provisioning with Ansible
- Support for provisioning with Puppet (AWS Opsworks)
- Support for various setups (from 1:1:0 up to 1:3:3)
- VPC creation (with all its sub components)
- VPC peering
- VPN creation (AWS side) using Transit Gateways
- VPN creation (AWS side) using VPN Gateways

Checkout our [insight talk](https://netcentric.biz/insights/) on the Netcentric website for more information.

# Getting started

## Requirements

In order to use this framework, you will need:

- The AWS CLI
- AWS CLI access (Access Key and Secret Access Key)
- Amazon EC2 key pair to access the EC2 instances
- An AWS S3 bucket to upload the CloudFormation Templates to
- Optional: AWS console management access (if you don't want to use the AWS CLI to create/manage required resources)

**AWS CLI**

Depending on the operating system / platform you are using, there are different ways to install the AWS CLI.
* Windows: [Installing, updating, and uninstalling the AWS CLI version 2 on Windows](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html)
* MacOS: Other than described in the official documentation by Amazon, we recommend to install the AWS CLI using [Homebrew](https://brew.sh/)
    ```
    $ brew install awscli
    ```
* Linux: [Installing, updating, and uninstalling the AWS CLI version 2 on Linux](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
* Docker: [Using the official AWS CLI version 2 Docker image](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-docker.html)

**AWS CLI access (Access and Secret Access Key)**

The Access and the Secret Key are personalized and need to be generated. For further information, please consult the AWS documentation on [Understanding and getting your AWS credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys).

**Amazon EC2 key pair**

A key pair, consisting of a private key and a public key, can be generated using the AWS CLI. The value of `KeyMaterial` will be your private key.
```
$ aws ec2 create-key-pair --profile <your-profile> --key-name <your-project>
```
For more information, please consult the AWS documentation on [Amazon EC2 key pairs and Linux instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

**Amazon S3 bucket**

CloudFormation requires the templates to be stored in an Amazon S3 bucket. A bucket can be easily created using the AWS CLI.
```
$ aws s3 mb s3://<your-project>-templates --profile <your-profile>
```

## Installation

Due to CloudFormation lacking any mechanism for dependencies, we recommend to use this project as a git submodule.
```
$ git submodule add https://github.com/Netcentric/cloudformation-iac cloudformation-iac
```

For detailed information on how to work with Git submodules have a look at [this blog entry](https://subfictional.com/fun-with-git-submodules/)

**AWS CLI Named Profiles**

To ensure that changes are only applied to a selected account and region, we recommend using the AWS CLI with [named profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
```
$ aws configure --profile <your-profile>
```

Finally test your setup, you should get a print out of all CloudFormation Stacks in the region you chose
```
$ aws cloudformation --profile <your-profile> list-stacks
```

# Quickstart

1. **(OPTIONAL)** Add the repository as a submodule to your project
    ```
    $ git submodule add https://github.com/Netcentric/cloudformation-iac
    ```

2. Generate a key pair (the value of `KeyMaterial` will be your private key to access all EC2 instances)
    ```
    $ aws ec2 create-key-pair --profile <your-profile> --key-name <your-project>
    ```

3. Create a S3 bucket
   ```
   $ aws s3 mb s3://<your-project>-templates --profile <your-profile>
   ```

4. Create a copy of the example configuration `example.yaml` and change it as needed. When done, package and upload it
    ```
    $ aws cloudformation package --profile <your-profile> --template-file <your-template>.yaml --s3-bucket <your-project>-templates --output-template-file <your-template>.packaged.yaml
    ```

5. Finally create a stack, using the packaged template
    ```
    $ aws cloudformation deploy --profile <your-profile> --template-file <your-template>.packaged.yaml --stack-name <your-project>
    ```

    If your stack is creating IAM resources at any point (e.g. if you use OpsWorks Puppet Enterprise) you'll also need to specify the necessary capabilities
    ```
    $ aws cloudformation deploy --profile <your-profile> --template-file <your-template>.packaged.yaml --stack-name <your-project> --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM
    ```

## Example project

The repository contains a file `example.yaml` as an example for a minimal AEM project.

**What it does**

In short, the `example.yaml` creates exemplary infrastructure for a minimal AEM project:

* 1 Author host (`m5a.xlarge`) with an encrypted 500 GB volume mounted at `/opt`
* 2 Publisher hosts (`m5a.large`) with an encrypted 500 GB volume mounted at `/opt`

In addition to these hosts it creates all infrastructure necessary for a solid setup:

* Network (VPC) "Nonprod" with public and private subnets, using two Availability Zones
* VPN between an imaginary corporate network and the "Nonprod" VPC (requires additional configuration by the companies IT)
* Load Balancers with SSL termination for both, Author and Publisher
* DNS entries for all hosts and the LBs (in the format `*.dev1.example.com`)
* SSL Certificates for the LBs
* Automatic daily snapshots of all non-root EBS Volumes
* Automatic limitation of the uptime of all instances to MON-FRI 4am to 8pm

# About

## Development guidelines

1. Before adding new logic, check in the code if a similar logic already exists
2. When adding new logic, do it bottom up. For example when adding something to the template `resources/ec2/instance.yaml`, add the new logic there and then recursively add the required parameters to all templates will end up using it, until you map them in the `example.yaml` file.
3. As a piece of advice, try not to hard code!
4. Enjoy!

## Authors / Contributors

* **[Michael Strache](https://github.com/Jarodiv)** - *Initial work*
* **[Valentin Savenko](https://github.com/valentinsavenko)** - *Initial work*
* **Adrian Andrei**
* **George Stefanov**
* **[Marco Godhino](https://github.com/aureliomarcoag)**