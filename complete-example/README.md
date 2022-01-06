# Workshop: Terraform Intro (complete example)

This is the complete example of the workshop. You can use this as a reference for your own attempt, or as a working sample to refer back to later.

> Note: Please refer to the [prerequisites](../README.md#Prerequisites) section of the parent [read me](../README.md) before trying to run this sample.

## Running the sample

This sample will create a VPC resource within the targeted AWS account. To run the sample, you must first ensure the layer has been initialized.

```shell
$ terraform init
```

Once initialized, you can apply the layer to AWS.

```shell
$ terraform apply
```

> You will need to enter "yes" when prompted to apply the generated plan to the AWS account.

When you are finished, make sure you remove all AWS resources you have created so that you do not incur any unwanted costs.

```shell
$ terraform destroy
```

> You will need to enter "yes" when prompted to remove the resources from the AWS account.
