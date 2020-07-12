## Kong ingress controller implementation minikube-aws-eks

This configuration is using a different rbac refering ingress controller ver 0.5. **We are not taking into account the last implementations basically
because in those implementations, everything related to the administration API is not well documented**.</br>
Something our team has appreciated is that access to api admin at last ingress controller's versions are very tricky.  
They have change the point of view in relationship between Kong-Cassandra so now they are promoting dbless design and a marriadge with some db rds implementation.
Some pending task is with our rbac using Kong:1.5.0 increase ingress controller at upper version.








ref.
[Ingress Controller On EKS ] https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/deployment/eks.md
[Ingress Controller On Minikube] 
