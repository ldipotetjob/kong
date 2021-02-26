## Kong 

Some point of views 

It’s rather cumbersome to use NodePort for Services that are in production..
As you are using non-standard ports, you often need to set-up an external load balancer that listens to the standard 
ports and redirects the traffic to the <NodeIp>:<NodePort>.
