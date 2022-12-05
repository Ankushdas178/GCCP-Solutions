# Set Up and Configure a Cloud Environment in Google Cloud: Challenge Lab Solution :

## Run the following code lines/blocks one-by-one in the Cloud Shell:

- Firstly Let's Create Environment Variables:

```yaml
export USER_NAME2=
```

## TASK 1 

```yaml
gcloud compute networks create griffin-dev-vpc --subnet-mode custom
```
```yaml
gcloud compute networks subnets create griffin-dev-wp --network=griffin-dev-vpc --region us-east1 --range=192.168.16.0/20
```
```yaml
gcloud compute networks subnets create griffin-dev-mgmt --network=griffin-dev-vpc --region us-east1 --range=192.168.32.0/20
```

## TASK 2 
```yaml
gsutil cp -r gs://cloud-training/gsp321/dm .
```
```yaml
cd dm
```
```yaml
sed -i s/SET_REGION/us-east1/g prod-network.yaml
```
```yaml
gcloud deployment-manager deployments create prod-network \
    --config=prod-network.yaml
```
```yaml
cd ..
```

## TASK 3 
```yaml
gcloud compute instances create bastion --network-interface=network=griffin-dev-vpc,subnet=griffin-dev-mgmt  --network-interface=network=griffin-prod-vpc,subnet=griffin-prod-mgmt --tags=ssh --zone=us-east1-b
```
```yaml
gcloud compute firewall-rules create fw-ssh-dev --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-dev-vpc
```
```yaml
gcloud compute firewall-rules create fw-ssh-prod --source-ranges=0.0.0.0/0 --target-tags ssh --allow=tcp:22 --network=griffin-prod-vpc
```

## TASK 4 
```yaml
gcloud sql instances create griffin-dev-db --root-password password --region=us-east1 --database-version=MYSQL_5_7
```
```yaml
gcloud sql connect griffin-dev-db
```
If asked for password, type `password`
```yaml
CREATE DATABASE wordpress;
GRANT ALL PRIVILEGES ON wordpress.* TO "wp_user"@"%" IDENTIFIED BY "stormwind_rules";
FLUSH PRIVILEGES;
```
```yaml
exit
```

## TASK 5 
```yaml
gcloud container clusters create griffin-dev \
  --network griffin-dev-vpc \
  --subnetwork griffin-dev-wp \
  --machine-type n1-standard-4 \
  --num-nodes 2  \
  --zone us-east1-b
```
```yaml
gcloud container clusters get-credentials griffin-dev --zone us-east1-b
```
```yaml
cd ~/
```
```yaml
gsutil cp -r gs://cloud-training/gsp321/wp-k8s .
```

## TASK 6 
```yaml
sed -i "s/username_goes_here/wp_user/g" wp-k8s/wp-env.yaml
sed -i "s/username_goes_here/stormwind_rules/g" wp-k8s/wp-env.yaml
```
```yaml
cd wp-k8s
```
```yaml
kubectl create -f wp-env.yaml
```
```yaml
gcloud iam service-accounts keys create key.json \
    --iam-account=cloud-sql-proxy@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com
kubectl create secret generic cloudsql-instance-credentials \
    --from-file key.json
```

## TASK 7 
```yaml
sed -i "s/YOUR_SQL_INSTANCE/griffin-dev-db/g" wp-deployment.yaml
```
```yaml
kubectl create -f wp-deployment.yaml
kubectl create -f wp-service.yaml
```
```yaml
kubectl get svc -w
```
After  running the above command, you'll get the external IP. Now copy it. It will be used in the next task.

**NOTE:** If the `kubectl get svc -w` is showing error, then run `kubectl get svc`

## TASK 8 

- Search `Monitoring` and click on `Monitoring`
- Click `Uptime Checks` on the left pane
- `+Create Uptime Check`
- Set title as `Wordpress Uptime`
- Copy and paste the external IP of 'Wordpress'
- Click `Create`

## TASK 9 
```yaml
gcloud projects add-iam-policy-binding $DEVSHELL_PROJECT_ID --member=user:$USER_NAME2 --role=roles/editor
```
