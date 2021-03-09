## Kong & Kong ingress controller implementation AWS-EKS


### Running with Postgres: 
Note: Postgres 10+

Configuring Postgres:

note: postgres user with admin grant  

#### Prepare posgres for kong migration 

1. postgres=> CREATE USER kong;
2. postgres=> GRANT kong TO postgres;
3. postgres=> ALTER USER kong WITH password 'kong';
4. postgres=> CREATE DATABASE kong OWNER kong;

#### Prepare Kong:
1. Create kong NameSpace/Run kong db migration
   1. kubectl create -f namespace.yaml
   2. We assume that are using RDS service to connect postgres. You need to indicate the proper endpoint y yoour RDS service.  
      . If you are not using RDS juts need remove in kong_migration_postgres.yaml file 
        the postgres host and set up it with your current host. You have different options to do this
        set up our services with your new host address/create a headless service if your plan is to include 
        your dababase under your cluster/orchestrator etc. 
      . kubectl create -f postgres-service.yaml

  3. kubectl create -f kong_migration_postgres.yaml
      . wait until migration get done

  4. Deploy kong ingress controller
      . kubectl create -f kong_all_in_one_aws_postgres.yaml
  5. Deploy ingress
      . kubectl create -f kong_ingress.yaml

 ref: </br>
 AWS RDS: &nbsp;https://docs.aws.amazon.com/rds/index.html </br>
 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;https://aws.amazon.com/getting-started/tutorials/create-connect-postgresql-db/</br>
 POSTGRES: &nbsp;https://www.postgresql.org/docs/current/  
