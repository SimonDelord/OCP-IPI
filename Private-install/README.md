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

However I struggled with the actual implementation of it (the AWS setup essentially).

These are the steps that need to be taken:
 - associate the internet gateway with the main Route Table in the VPC (rtb-00c310509fc11aafc) and add this internet gateway to a default route. Add the public subnets of the VPC to this route table (subnet association)
 - create the NAT Gateway (you could have up to 3 - e.g one per AZ) into one of the public subnets of the VPC (subnet-02861a246d9c51974 / vpc-ipi-public-a in this example) and associate the elastic IP address. In this example the NAT GW is 	nat-07b4ea1d8abb57d52
 - create a route table for the private subnets of the VPC (rtb-0bea709b735dd6377 in this example), add a default route to this route table and point it to the NAT Gateway created in the previous step (nat-07b4ea1d8abb57d52), associate the 3 private subnets to this route table. You could probably do 3 different private route tables, each one of them associated with a single private-subnet in the VPC and have a dedicated NAT Gateway in the AZ (e.g if you created 3 NAT GWs in the previous step).
 - The installer should now be able to be used as per the install-config.yaml file here

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

