## Kong & Kong ingress controller implementation AWS-EKS


### Running with Postgres: 
Note: Postgres 10+

Configuring Postgres:

note: postgres user with admin grant  

1. Prepare posgres for kong migration 

postgres=> CREATE USER kong;
postgres=> GRANT kong TO postgres;
postgres=> ALTER USER kong WITH password 'kong';
postgres=> CREATE DATABASE kong OWNER kong;

2. Prepare Kong .....

	1. Create kong NameSpace/Run kong db migration
       1. kubectl create -f namespace.yaml
       2. We assume that are using RDS service to connect postgres. You need to indicate 
          the proper endpoint y yoour RDS service.  
          . kubectl create -f postgres-service.yaml
          2.1 If you are not using RDS juts need remove in kong_migration_postgres.yaml file 
              the postgres host and set up it with your current host. You have different options to do this
              set up our services with your new host address/create a headless service if your plan is to include 
              your dababase under your cluster/orchestrator etc. 

       3. kubectl create -f kong_migration_postgres.yaml
          3.1 wait until migration get done

    2. Deploy kong ingress controller
       1. kubectl create -f kong_all_in_one_aws_postgres.yaml
    3. Deploy ingress

       1. kubectl create -f kong_ingress.yaml

 ref: 
 
 AWS RDS:
 POSTGRES:  