This scenario is using an IPI install on AWS to setup a private OCP cluster in an existing VPC.
I have used the eu-west-2 region (London) to organise the AWS setup.

As part of the setup I created the following:
- one VPC
- 3 private subnets
- 3 public subnets
- 1 Elastic IP address
- one NAT Gateway
- 2 route tables associated with the VPC
- one Internet Gateway


I then generated a install-config.yaml file that can be used as the basis for the OCP cluster creation.

The OCP documentation for this is available https://docs.openshift.com/container-platform/4.8/installing/installing_aws/installing-aws-private.html#installation-custom-aws-vpc-requirements_installing-aws-private

However I struggle with the actual implementation of it (the AWS setup essentially).

Here are the various screenshots and a diagram of the setup.

- figure 1 - Overall AWS network diagram
![alt text](https://github.com/[username]/[reponame]/blob/[branch]/image.jpg?raw=true)
- figure 2 - VPC details
- figure 3 - subnets details
- figure 4 - route tables
- figure 5 - internet gateway
- figure 6 - elastic IPs
- figure 7 - NAT Gateways

