CloudFormation to Terraform migration example
===============

In this project we explore all the steps required in order to move from using
CloudFormation as the default IaC to Terraform. This is especially tricky when
there are stacks dependent on the current one. For drifts, also those undetected
by CloudFormation, this does not guarantee no changes, as if we want to get rid
of the stack, we need to set `DeletionPolicy: Retain` on each resource.

Follow the steps described on my blog for more info:

- [https://pabis.eu/blog/2025-06-26-From-CloudFormation-to-Terraform-with-Terraformer.html](https://pabis.eu/blog/2025-06-26-From-CloudFormation-to-Terraform-with-Terraformer.html)
- [https://dev.to/...](https://dev.to/...)
