## Kong ingress controller implementation minikube-aws-eks

This configuration is using a different rbac refering ingress controller ver 0.5.</br>
**We are not taking into account the last implementations basically
because in those implementations, everything related to the administration API is not well documented**.</br>
*I have appreciated that access to api admin at last ingress controller's versions are very tricky.*  
They have change the point of view in relationship between Kong-Cassandra so now they are promoting dbless design and a marriadge with some db rds implementation.

### Running dbless:

### Running with Cassandra 
Note: Cassandra 3.11</br> 
cassandra.yaml:</br>

**Authentication** 
* PasswordAuthenticator
* usr: cassandra 
* password: cassandra      

#### Steps for running Kong with cassandra ###

1. Create cassandra resources([cassandra headless services](https://github.com/ldipotetjob/kong/blob/master/k8s/cassandra_service.yaml)/[cassandra db](https://github.com/ldipotetjob/kong/blob/master/k8s/cassandra_statefulset_minikube.yaml))
2. Prepare cassandra. You can find script below. ref. https://cassandra.apache.org/doc/latest/operating/security.html#authentication
3. Create migration [job resource](https://github.com/ldipotetjob/kong/blob/master/k8s/kong_migration_cassandra.yaml)
4. Create [kong resources](https://github.com/ldipotetjob/kong/blob/master/k8s/kong_all_in_one_aws.yaml).

ref. Prepare Cassandra(https://cassandra.apache.org/doc/latest/operating/security.html#authentication) 

```shell
cqlsh -u cassandra -p cassandra
ALTER KEYSPACE system_auth WITH replication = {'class': 'NetworkTopologyStrategy', 'datacenter1': 2};

# create a new superuser
CREATE ROLE dba WITH SUPERUSER = true AND LOGIN = true AND PASSWORD = 'super';

ALTER ROLE cassandra WITH SUPERUSER = false AND LOGIN = false;
```


**Some pending task:**</br> 
 With the current rbac: </br>
 Kong:1.5.0 => upper versions 
 Kong ingress ingress controller: 0.5.0 => upper versions 

ref.
[Ingress Controller On EKS ] https://github.com/Kong/kubernetes-ingress-controller/blob/master/docs/deployment/eks.md
[Ingress Controller On Minikube] 
