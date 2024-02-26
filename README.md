## Project run up by me:

### First

Create the cluster and the node group

- eksctl create cluster -f cluster.yml
- aws eks update-kubeconfig --name tin-udacity

### Second

Deploy the Database with helm:

- helm repo add tinProject https://charts.bitnami.com/bitnami
- helm install postgress-db tinProject/postgresql

We can check the status of the deploy with..

- kubectl get svc
- kubectl get pods

The deployment is ready when the kubectl get pods prints that the postgress pod id ready

WE have to get the credentials to interact with Postgres

- export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgress-db-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d)
- echo $POSTGRES_PASSWORD
  MSAI8KOuHJ

Check the DB is running with

- kubectl port-forward --namespace default svc/postgress-db-postgresql 5432:5432
- PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

Seed the db with the following:

- PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < ./db/1_create_tables.sql
- PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < ./db/2_seed_users.sql
- PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < ./db/3_seed_tokens.sql

### Trhid

We need to create the ECR repository for our docker image

- aws ecr create-repository --repository-name tin-udacity-repo --region us-east-1

Lets create the role to use codebuild:

- aws iam create-role \
   --role-name codebuild-role \
   --assume-role-policy-document '{
  "Version": "2012-10-17",
  "Statement": [
  {
  "Effect": "Allow",
  "Principal": {
  "Service": "codebuild.amazonaws.com"
  },
  "Action": "sts:AssumeRole"
  }
  ]
  }'

Attach policies

- aws iam attach-role-policy --role-name codebuild-role --policy-arn arn:aws:iam::aws:policy/AmazonEC2ContainerRegistryFullAccess
- aws iam attach-role-policy --role-name codebuild-role --policy-arn arn:aws:iam::aws:policy/AWSCodeBuildDeveloperAccess
- aws iam get-role --role-name codebuild-role | jq -r '.Arn'

Lastly lets make the build:

- aws codebuild create-project --cli-input-json file://awsCodeBuild.json

- aws codebuild start-build \
   --project-name tin-analytics

FORGET THIS, go to the aws console and create the project manually.
