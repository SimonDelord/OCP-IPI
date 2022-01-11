This is the set of files and commands used to deploy an Application Load-balancer on AWS to front-end OpenShift.
The typical use case for this is for customers who need to deploy an AWS WAF as part of their solution.

I have relied on the following article - https://github.com/rh-mobb/documentation/tree/main/docs/aws/waf
However, I had to modify several components to make it work for myself, as I found some of the instructions a little bit vague (for me).

First step was to create a standard OCP cluster on AWS using the IPI installer. An example of install-config.yaml file is located in this GitHub repo.
To install run the following command (please note that OCP4.9 didn't work as Kubernetes 1.22 deprecated some of the CRDs used by this ALB Ingress Controller and I did the deployment on a 4.7.8 OCP cluster).
