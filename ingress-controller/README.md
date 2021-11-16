# Ingress Controller Sharding

This is the setup for adding an external load-balancer to a private Cluster (so external people can access apps).

I used both:
* the OpenShift documentation - which I found not super detailed (as I couldn't find an explicit mention 
of matching labels between an IngressController and the associated routes) 
* the following blog written by Roberto Carratalá [Blog](https://rcarrata.com/openshift/ocp4_route_sharding/) which is a great tutorial
  
Now the process for setting up an external access to a private OCP cluster is the following:

* create an IngressController (ingress-controller.yaml file). There are 3 important components to this IngressController:
    * endpointPublishingStrategy: to LoadBalancerService + External (this is to automate the creation of the AWS ELB)
    * routeSelector: a way of selecting which routes/namespaces are going to get routed through this IngressController. In this example
      I have used the following "label sharded", this label is reused in the route.yaml files for the apps.
    * domain: you must have a "routable" domain from a DNS perspective. In this example I have used apps.simon-public.melbourneopenshift.com.
      
* deploy the ingress controller, it will then create:
    * 2 extra PoDs in the openshift-ingress namespace
    * one extra ingress-controller in the openshift-ingress-operator namespace (use the oc get ingresscontroller -n openshift-ingress-operator)
    * an external load-balancer on AWS targetting ports 443 and 80 towards the ingressController (similar to the Default Ingress Controller)
      
* associate the domain configured on the IngressController to a proper DNS entry and have this entry point to the Load-balancer. In this example,
  I have used route53. I did this by creating under the AWS console a new record under my specific hosted zone (melbourneopenshift.com).
    * record name: the name you are going to use for all your apps (*.apps.simon-public.melbourneopenshift.com* for my example)
    * record type: A - routes traffic
    * turn on Alias and then select Alias to Application and Classic Load-balancer or Network Load-balancer depending on the LB you are using,
    then select the AWS region and the associated load-balancer (which was created when you deployed the ingress controller in the previous step).

You can then start deploying apps after this step. I have attached two hello-world apps that I have used to check that I could access both apps
externally (E.g directly from my laptop).

The final step is typically to remove those "routes" from the Default Ingress-controller otherwise those routes will be reachable by both Ingress-controllers.
I didn't play with the certificate but you can also provide a certificate to the Ingress-controller (similar to the steps for the Default Ingress). 

