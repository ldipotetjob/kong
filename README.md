## Kong 

Some point of vie


It’s rather cumbersome to use NodePortfor Servicesthat are in production.
As you are using non-standard ports, you often need to set-up an external load balancer that listens to the standard 
ports and redirects the traffic to the <NodeIp>:<NodePort>.
