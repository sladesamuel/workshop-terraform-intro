# Workshop: Terraform Intro

This workshop is intended to introduce the concept of Infrastructure as Code, using Terraform to produce a repeatable approach to building out AWS resources.

A completed example is provided under the [completed-example/](completed-example/) folder.

## Prerequisites

The following needs to be setup before this workshop can be attempted.

1. [AWS CLI v2](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
1. [Terraform CLI](https://learn.hashicorp.com/tutorials/terraform/install-cli?in=terraform/aws-get-started)
1. A text/code editor ([Visual Studio Code](https://code.visualstudio.com/) is recommended, but any other familiar text editors can also be used, such as Notepad, Notepad++, Sublime Text, Vim, Nano, etc)

As part of the AWS CLI setup, you will also need to ensure you have an AWS account and have generated an [Access Key and Secret Key](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-prereqs.html#getting-started-prereqs-keys) pair.

> Note: A little more advanced, but an alternative to directly configuring your AWS CLI instance to access a single AWS account is to use [AWS Vault](https://github.com/99designs/aws-vault) to manage access to multiple AWS accounts.

## Instructions

Once all prerequisites are setup, follow along with these instructions to walk you through the workshop.

### Step 1: Configure Terraform

To begin with, create a file called `main.tf`. Terraform supports breaking out resources into separate files (as well as layers and modules), and correctly works out the dependencies by stiching all files back together internally. For now, though, we will put all of our Terraform code (known as HCL, or Hashicorp Configuration Language) in this one file.

We first need to tell Terraform what provider(s) we want to use. In our case, we are going to use AWS, as that is where we want to build our infrastructure. So, start by entering the following into the `main.tf` file.

```diff
+ terraform {
+   required_providers {
+     aws = {
+       source  = "hashicorp/aws"
+       version = "~> 3.27"
+     }
+   }
+
+   required_version = ">= 0.14.9"
+ }
```

To quickly sum up each part here:

- The `terraform` block contains Terraform settings.
- A provider is a plugin that Terraform uses to create and manage resources. In this case, we have specified `aws` as our provider.
  - The `aws` provider plugin is provided by "hashicorp/aws", as defined in our `source` property
  - The `aws` provider plugin is pinned to version 3.27
- Our required Terraform version needs to be at least 0.14.9

> Further reading on providers can be found [here](https://www.terraform.io/language/providers/requirements).

### Step 2: Specify the provider

Now that we've configured Terraform, we need to add our provider, so, add the following into the same file.

```diff
# terraform {
+   required_providers {
+     aws = {
+       source  = "hashicorp/aws"
+       version = "~> 3.27"
+     }
+   }
+
#   required_version = ">= 0.14.9"
# }
+
+ provider "aws" {
+   profile = "default"
+   region  = "eu-west-1"
+ }
```

You can see here that we've added a Terraform setting to load in our AWS provider plugin and pin its version to 3.27. We've then defined some settings for the AWS provider in the `provider` block. This defines the profile and region we are using.

> Note: The `profile` argument here is simply set to `default`. If you have configured AWS credentials locally with a different profile name, you may need to update this value. However, this should work when using AWS Vault. You can read more about authentication [here](https://registry.terraform.io/providers/hashicorp/aws/latest/docs#authentication).

### Step 3: Create the VPC

Finally, we can start creating our resources. For this workshop, we are only going to create a VPC resource with a predefined configuration. We will look later how to pass variables to allow reuse and customization of our Terraform scripts.

```diff
...
+
+ resource "aws_vpc" "our_vpc" {
+   cidr_block = "10.0.0.0/16"
+
+   tags = {
+     Name = "our-vpc"
+   }
+ }
```

This is known in HCL as a [Resource Block](https://www.terraform.io/language/resources/syntax). The `aws_vpc` defines what the resource is we are creating. In this case, we are using a VPC as defined in the AWS provider plugin. We then name it `our_vpc`, but we could name it anything we like. This is just a reference for use within our HCL scripts. We have set some properties here, namely the CIDR Block we wish to use, and also we have given it a name tag so that we can easily identify it within our AWS Console later.

### Step 4: Let's run it!

#### Initialization

Before we can deploy our changes to our AWS account, we need to first initialize our local Terraform state. Terraform uses state to identify changes that need to be made, and any dependencies, so that it can produce a plan to be executed efficiently. By default, Terraform uses a local state (i.e. state stored on the local device), but it can be configured to store its state remotely. This is recommended, especially when working as part of a team.

> State configuration is beyond the scope of this workshop, but you can read more about using a remote state [here](https://www.terraform.io/language/state/remote).

To initialize our local state, and download/install any plugins we might have configured, run the following in a terminal from the folder containing the Terraform scripts.

```shell
$ terraform init
```

#### The plan

Now that we have installed our AWS provider plugin and initialized our local state, we can ask Terraform to produce a plan of what changes it should apply to our AWS account. To do this, we can run the following command.

```shell
$ terraform plan
```

This plan will show what is going to be created, changed, or deleted. In our case, everything is new, so the plan should show that our VPC is going to be created in our AWS account.

> Note that this plan has not been executed within our AWS account - it is just to show what will be done when we choose to apply the plan.

#### Application

The final step is to run the plan. To apply changes, run the following command.

```shell
$ terraform apply
```

Notice here that we are prompted to enter `yes` to confirm the application and we should see the same output above as we did when running the plan. This is because Terraform's `apply` command produces a plan first and then asks for confirmation before making any changes to our AWS account. If you enter anything other than `yes` here, then the job will be aborted and no changes will be made. This can be skipped for automation purposes, but for now, it's wise to keep.

This command producing a plan also means we don't need to run the plan step from before if we intend on applying anyway. A plan can also be stored in a local file, and then applied directly without producing the plan again. You can read more about Terraform Plans [here](https://www.terraform.io/cli/commands/plan). For now, just type `yes` and hit enter to apply the changes to our AWS account.

And that's it! We should now be able to open up our AWS Console in the browser and see our newly created VPC [here](https://eu-west-2.console.aws.amazon.com/vpc/home?region=eu-west-2#vpcs:).

### Step 5: Variables and updates

So far, we have walked through a very simply setup. So, let's take it a little further to demonstrate a more common use case.

Within your `main.tf` file, make the following changes.

```diff
+ variable "vpc_name" {
+   description = "The name of our VPC"
+   value       = string
+   default     = "our-vpc"
+ }

# resource "aws_vpc" "our_vpc" {
#   cidr_block = "10.0.0.0/16"
#
#   tags = {
!     Name = var.vpc_name
#   }
# }
```

We have now added an optional variable to our script. This variable will allow us to change the name of our VPC, but it defaults to the value we had before. Now we can run our apply command again, but this time we will specify the name for the VPC.

```shell
TF_VAR_vpc_name=this-is-my-vpc terraform apply
```

Again, a plan will be produced, which should show the suggested change. Since we are only updating a tag, the resource does not need to be destroyed and recreated in AWS. This is good as no interruption would occur here if this VPC was actually in use in a production scenario.

You will notice that to define the variable value, we must pass it through as a variable in the terminal in the format `TF_VAR_{variable_name}`, matching the case (usually `lower_case_with_an_underscore`). If we had omitted this, the value would take the default of `our-vpc`. If a variable has no default and needs to be set when applying the change, then the `default` property would not be set in the `variable` block.

> You can read more about variables in Terraform [here](https://www.terraform.io/language/values/variables).

### Step 6: Cleanup

That's all for this workshop. Before we finish, let's make sure we cleanup what we've done, so that we do not incur any unwanted costs within our AWS account. To do this, we simply need to run the [destroy](https://www.terraform.io/cli/commands/destroy) command.

```shell
$ terraform destroy
```

Again, this will present you with a plan of what is going to be removed and will also ask for confirmation before proceeding. Type `yes` and hit enter to confirm the cleanup.

## Additional Reading

It is recommended you do some additional reading to better understand the concepts and tools referenced in this workshop.

- [AWS CLI](https://aws.amazon.com/cli/)
- [Terraform](https://www.terraform.io/)
- [Infrastructure as Code: What Is It? Why Is It Important?](https://www.hashicorp.com/resources/what-is-infrastructure-as-code)
- [Introduction to Infrastructure as Code with Terraform](https://learn.hashicorp.com/tutorials/terraform/infrastructure-as-code?in=terraform/aws-get-started)
