## Kong ingress controller implementation minikube-aws-eks

This configuration is using a different rbac refering ingress controller ver 0.5.</br>
**We are not taking into account the last implementations basically
because in those implementations, everything related to the administration API is not well documented**.</br>
*I have appreciated that access to api admin at last ingress controller's versions are very tricky.*  
They have change the point of view in relationship between Kong-Cassandra so now they are promoting dbless design and a marriadge with some db rds implementation.

**Some pending task:**</br> 
 With the current rbac: </br>
 Kong:1.5.0 => upper versions 
 Kong ingress ingress controller: 0.5.0 => upper versions 

ref.
[Ingress Controller On EKS ] https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/deployment/eks.md
[Ingress Controller On Minikube] 
