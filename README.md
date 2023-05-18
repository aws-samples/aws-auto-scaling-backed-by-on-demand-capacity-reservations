# ODCR Backed ASG

## Overview

The main purpose of this AWS CloudFormation template is to spin up the solution used for describing how to work with AWS EC2 Capacity Reservations in the blog post referenced at the bottom of this page, but it is also intended as an example of how to reserve capacity for your Auto Scaling groups in AWS using Infrastructure as Code (IaC).

This solution creates a new VPC with 3 public subnets across 3 Availability Zones (AZs) in a selected region, with a specified number of On Demand Capacity Reservations (ODCRs) in each AZ. On top of this sits an Auto Scaling group across all AZs, backed by the ODCRs via a Capacity Management Resource Group.

## How to Use

For a detailed description of how use CloudFormation with the template provided in this repository, please refer to the AWS [CloudFormation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html).

If you "just want to get it to work", here are some stripped down steps:

1. Download the template from this repository:
   - If you have cloned the repo, then you will already have it locally, and there is nothing more to do here.
   - If you have not cloned the repo, then the simplest way is by right-clicking the file and "Save link as" (or somthing similar, depending on the Browser that you use).
2. Decide on an AWS account in which to run the template:
   - This template will be creating (among other things) a VPC and some EC2 instances. If your account already has other things running in it, make sure that you are not close to its account limits for these or the CloudFormation stack creation will fail.
3. Locate CloudFormation in the AWS Console
   ![Finding CloudFormation in the AWS Console](/images/aws-console-cloudformation.png)
4. Click the "Create Stack" button.
5. Reference the template that you have donwloaded, and click "Next":
   ![Filling in the Create Stack information](/images/cloudformation-create-stack.png)
6. The next page holds the stack name (mandatory, it can be anything you want it to be as long as if follows the specified naming convention, like "My-ODCR-Stack"), and parameters that you can change if you want to. The defaults work well with the accompanying Blog post. Note that if you change the EC2InstanceType parameter from t2.micro, you may go outside of the AWS Free Tier, which will result in charges for running this solution until you tear it down!
7. The next page holds a number of advanced settings that are not required for our simple demo case, so just click "Next". If you are planning to run this template in a production setting, it is _strongly_ recommended that you read up on these settings in the AWS [CloudFormation documentation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/cfn-console-create-stack.html) before proceding.
8. Next comes a summary of all settings. Read through to ensure that you have not accidentally changed something that you do not want changing. After this step, CloudFormation _will_ create what you have asked for, so if you have picked an expensive instance and changed the nr of instances to 42, then this _will_ significantly impact your AWS bill. Once you are happy, click "Submit".
9. Now wait for a few minutes for all resources to be spun up. When the Status turns to "CREATE_COMPLETE", you're done!
   ![Stack Creation Complete](/images/cloudformation-create-stack-complete.png)

## Architecture

![Architecture Diagram](/images/odcr-backed-architecture.png)

The architecture used here is a minimal one required to demonstrate the use of Capacity Reservation for an Autoscaling Group in AWS. Before it is ready to be used in a live scenario, a number of things would need to be added, such as for example:

- DR capability
- Monitoring
- Security / IAM
- Load Balancing
- Something actually **running** on the Instances
- etc

All values used for things like AWS Instance Types are available in parameters, but have been set to "sensible" defaults based on being used together with the Blog Post.

### VPC

Nothing special here. VPC with a CIDR range large enough to fit out EC2 Instances.

### Internet Gateway

Not explicitly needed to demonstrate the Capacity Reservation, but we've got it here anyway so that the Subnets can have public access.

### Subnets

One in each Availability Zone. In the accompanying Blog post (see "Related Documentation"), we then remove one of them to simulate an AZ becoming unavailable.

### Route Table

### Route & Route Table Association

Without these our VPC would not be connected to the Internet Gateway

### Capacity Reservations

This is the whole point of the solution! We set up X Capacity Reservations (ODCRs) in each AZ.

### Resource Group

The Capacity Reservations (across AZs) go into this Resource Group, where they can later be referenced / used as a combined entity.

### Security Group

To be used in the Launch Template for the EC2 Instances.

### Launch Template

Definition of how to Launch new EC2 Instances, to be used with the Auto Scaling Group. This is where we tie in with the Resource Group, which in turn pulls in the Capacity Reservations when the ASG scales in/out.

### Auto Scaling Group

The brains of the whole solution. The ASG is the component that we interact with in the Blog Post, to demonstrate how the Capacity Reservations are used when it scales in/out.

## Known Limitations

In its current form, this template will only work in Regions that have at least 3 Availability Zones.

## Related Documentation

- [AWS Compute Blog Post - Reserving EC2 Capacity across Availability Zones by utilizing On Demand Capacity Reservations (ODCRs)](https://aws.amazon.com/blogs/compute/reserving-ec2-capacity-across-availability-zones-by-utilizing-on-demand-capacity-reservations-odcrs/)

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
