## Project run up by me:

### First

Create the cluster and the node group

- eksctl create cluster -f cluster.yml && aws eks update-kubeconfig --name tincito-udacity

### Second

Deploy the Database with helm:

- helm repo add tinProject https://charts.bitnami.com/bitnami && helm install postgress-db tinProject/postgresql

We can check the status of the deploy with..

- kubectl get svc && kubectl get pods

The deployment is ready when the kubectl get pods prints that the postgress pod id ready

WE have to get the credentials to interact with Postgres

- export POSTGRES_PASSWORD=$(kubectl get secret --namespace default postgress-db-postgresql -o jsonpath="{.data.postgres-password}" | base64 -d) && echo $POSTGRES_PASSWORD
  TGut82TbxR

Check the DB is running with

- kubectl port-forward --namespace default svc/postgress-db-postgresql 5432:5432 & PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432
- PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432

Seed the db with the following:

- PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < ./db/1_create_tables.sql && PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < ./db/2_seed_users.sql && PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d postgres -p 5432 < ./db/3_seed_tokens.sql

### Third create the ECR and the codebuild project with the aws console

### fourth lets make the build:

last lastly.. lets deploy the build in my cluster

Check CloudWatchAgentServerPolicy in the node role
run this unnecesary commands to make flluentbit to send the logs to the cloudwatch
ClusterName=tincito-udacity
RegionName=us-east-1
FluentBitHttpPort='2020'
FluentBitReadFromHead='Off'
[[${FluentBitReadFromHead} = 'On']] && FluentBitReadFromTail='Off'|| FluentBitReadFromTail='On'
[[-z ${FluentBitHttpPort}]] && FluentBitHttpServer='Off' || FluentBitHttpServer='On'
curl https://raw.githubusercontent.com/aws-samples/amazon-cloudwatch-container-insights/latest/k8s-deployment-manifest-templates/deployment-mode/daemonset/container-insights-monitoring/quickstart/cwagent-fluent-bit-quickstart.yaml | sed 's/{{cluster_name}}/'${ClusterName}'/;s/{{region_name}}/'${RegionName}'/;s/{{http_server_toggle}}/"'${FluentBitHttpServer}'"/;s/{{http_server_port}}/"'${FluentBitHttpPort}'"/;s/{{read_from_head}}/"'${FluentBitReadFromHead}'"/;s/{{read_from_tail}}/"'${FluentBitReadFromTail}'"/' | kubectl apply -f -

kubectl apply -f secret.yml && kubectl apply -f configMap.yml && kubectl apply -f analytics.svc.yml
