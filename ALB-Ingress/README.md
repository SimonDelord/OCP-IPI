This is the set of files and commands used to deploy an Application Load-balancer on AWS to front-end OpenShift.  
The typical use case for this is for customers who need to deploy an AWS WAF as part of their solution.  

I have relied on the [following article](https://github.com/rh-mobb/documentation/tree/main/docs/aws/waf).  
However, I had to modify several components to make it work for myself, as I found some of the instructions a little bit vague (for me).  

First step was to create a standard OCP cluster on AWS using the IPI installer.  
An example of install-config.yaml file is located in this GitHub repo.  
To install run the following command (please note that OCP4.9 didn't work as Kubernetes 1.22 deprecated some of the CRDs used by this ALB Ingress Controller and I did the deployment on a 4.7.8 OCP cluster).  

To create the IPI cluster, download the Openshift installer and type. 
```
openshift-install create cluster --dir=<DIR where the install-config.yaml is> --log-level=info
``` 
After 20-30 minutes an OCP cluster should be available with a kubeadmin password generated (available under the /auth/kubeadmin file).
  
For this exercise, as explained in the link above, you need:
- Helm3 CLI
- oc/kubectl CLI
- AWS CLI. 
  
I typically use a jumphost with all the software above in one AWS region.  
  
## 1st Step: you need to create an AWS Policy and Service Account.
```
curl -so iam-policy.json https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/${ALB_VERSION}/docs/install/iam_policy.json  
POLICY_ARN=$(aws iam create-policy --policy-name "AWSLoadBalancerControllerIAMPolicy" --policy-document file://iam-policy.json --query Policy.Arn --output text)  
echo $POLICY_ARN
```
## 2nd Step: Create a Service account
```
aws iam create-user --user-name aws-lb-controller --query User.Arn --output text
```
## 3rd Step: Attach the Policy to the User
```
aws iam attach-user-policy --user-name aws-lb-controller --policy-arn ${POLICY_ARN}
```
## 4th: Create an Access and Secret Key for the User
```
aws iam create-access-key --user-name aws-lb-controller
```
These keys will be used as part of the helm values.yaml file to configure the ServiceAccount installing the controller onto OCP.  
  
## 5th: Create a namespace or project to deploy the ALB-Ingress-Controller
```
oc new-project aws-load-balancer-controller (in my case)
```
## 6th: Apply the CRDs required for supporting this ALB Controller.
```
kubectl apply -f https://raw.githubusercontent.com/aws/eks-charts/master/stable/aws-load-balancer-controller/crds/crds.yaml
```
As mentioned those CRDs are not supported on K1.22 as it deprecated the support for networking.k8s.io/v1beta1, but the current load balancer controller still uses v1beta1.  
There is an outstanding feature request to support v1 (not sure about the ETA on this one).  
One more reason when building your own K8 environment to be careful.
  
## 7th Step: Add the Helm Repo and Install the Controller
For this step you need Helm3 installed.  
```
helm repo add eks https://aws.github.io/eks-charts 
helm install -n aws-load-balancer-controller aws-load-balancer-controller eks/aws-load-balancer-controller -f values.yaml
```

This is really what took me most of the time to sort out.  
For the values.yaml I used the base file [here](https://github.com/aws/eks-charts/blob/master/stable/aws-load-balancer-controller/values.yaml).  
I then modified accordingly based on the values I saw from the [article at the top](https://github.com/rh-mobb/documentation/blob/main/docs/aws/waf/alb.md) for Step 10.  

If everything works fine, you should get 2 PoDs deployed in the namespace you created in Step 5.

# Test that this works

You can then deploy an Application that uses a specific Ingress that will trigger the creation of the ALB.  
You will also need to add a DNS entry that points to the ALB created by the application.  

I have reused the application defined in the [article at the top](https://github.com/rh-mobb/documentation/blob/main/docs/aws/waf/alb.md).
  
## App deployment

```
oc new-project demo   
oc new-app https://github.com/sclorg/django-ex.git
kubectl -n demo patch service django-ex -p '{"spec":{"type":"NodePort"}}'
```

You then create an Ingress that will trigger the creation of an ALB in AWS.  
The file I used is called alb-ingress.yaml. 
I had to modify some of the parameters in the Top Link with the Ingress.  
Typically the alb.ingress.kubernetes.io/subnets (e.g all the public Subnet IDs for my OCP cluster), the path (from /* to /) and the relevant host "django-demo.apps.ocpaws.melbourneopenshift.com".

You need to make sure that there is a DNS entry matching this host.  
I have attached here a screenshot of my route53 setup, where the following DNS entry (* .apps.ocpaws.melbourneopenshift.com) points to the ALB that was created by the deployment of the alb-ingress.yaml file.

For the alb-ingress.yaml file, you will see that the host is defined as. 
```
spec:
  rules:
    - host: "django-demo.apps.ocpaws.melbourneopenshift.com"
      http:
        paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: django-ex
                port:
                  number: 8080
```

  
