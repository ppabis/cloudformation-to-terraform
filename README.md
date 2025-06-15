CloudFormation to Terraform migration example
===============

In this project we explore all the steps required in order to move from using
CloudFormation as the default IaC to Terraform. This is especially tricky when
there are stacks dependent on the current one. For drifts, also those undetected
by CloudFormation, this does not guarantee no changes, as if we want to get rid
of the stack, we need to set `DeletionPolicy: Retain` on each resource.

Example Steps
------------
0. Apply all the CloudFormation resources. Deploy `vpc.yaml` first and then
   create `ec2.yaml` that should use outputs of `vpc.yaml`.
   ```bash
   aws cloudformation deploy --template-file cloudformation/vpc.yaml --stack-name example-vpc --region eu-central-1
   aws cloudformation deploy --template-file cloudformation/ec2.yaml --stack-name example-ec2 --parameter-overrides VpcStackName=example-vpc --region eu-central-1
   ```
1. Using the same CloudFormation template, update the stack but mark each
   resource with `DeletionPolicy: Retain` and `UpdateReplacePolicy: Retain`.
   Drifted changes are not guaranteed to be retained during this update. Use the
   same commands for deployment as above.
2. Despite that you can't remove the resources yet. The outputs of VPC stack
   have dependents. You can start with the EC2 one but let's assume the more
   difficult case of migrating VPC first. This step involves freezing the output
   values. Fortunately, CloudFormation only checks for string equality when
   refusing to update outputs that are used by some downstream stack. Get the
   values for output using the following command:
   ```bash
   aws cloudformation describe-stacks --stack-name example-vpc --region eu-central-1 --query 'Stacks[0].Outputs' --output table
   aws cloudformation deploy --template-file cloudformation/vpc.yaml --stack-name example-vpc --region eu-central-1
   ```
3. Get a copy of all resource ARNs for reference later. You can use them to
   filter resources to use with Terraformer.
   ```bash
   aws cloudformation describe-stack-resources --stack-name example-vpc --region eu-central-1 --query 'StackResources[*].[LogicalResourceId,PhysicalResourceId,StackId]' --output table > example-vpc.txt
   ```
4. Now you can safely remove all the resources in the next deployment. Because
   CloudFormation stack requires at least one resource, I suggest placing
   `AWS::IoT::Thing` resource with an arbitrary and unique name.
   ```yaml
    Resources:
      Placeholder:
        Type: AWS::IoT::Thing
        Properties:
          ThingName: !Sub ${AWS::StackName}-Placeholder
   ```
5. Check which resources are available to import with Terraformer:
   `terraformer import aws list`
6. 