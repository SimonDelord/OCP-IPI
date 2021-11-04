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
![alt text](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/AWS%20Overview%20Before.png)
- figure 2 - VPC details
![alt text](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/VPC.png)
- figure 3 - subnets details
![alt text](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/Subnets.png)
- figure 4 - route tables
![Overview](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/Route%20Tables.png)
![Default Route Table](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/Default%20Route%20Table.png)
![Private Route Table](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/Private%20Route%20Table.png)
- figure 5 - internet gateway
![alt text](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/Internet%20Gateway.png)
- figure 6 - elastic IPs
![alt text](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/Elastic%20IP.png)
- figure 7 - NAT Gateways
![alt text](https://github.com/SimonDelord/OCP-IPI/blob/main/Private-install/Images/NAT%20Gateway.png)

