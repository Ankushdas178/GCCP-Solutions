<h1>Create and Manage Cloud Resources: Challenge Lab</h1> 

> Lab Link : https://google.qwiklabs.com/focuses/10258?parent=catalog
<h3>Solution:</h3>
Change the names/numbers in the square brackets [ ] according to the credentials given to you on the left side of the lab.
<hr>

### Task 1: Create a project jumphost instance

* 1. Login to Google Cloud Platform using the given credentials
* 2. Go to **Navigation Menu (☰)** from the top-left corner > Select **'Compute Engine'** > Select **'VM Instances'**
* 3. 1. Change the instance name to the given name (Check for the Instance Name on Lab , e.g. nucleus-jumphost-626)
     2. Scroll down to **Machine Configuration** > Select **N1** from 'Series' Dropdown > Just Below, Select **f1-micro** from 'Machine Type' Dropdown  
     3. Scroll down to the end and select **CREATE**

### Task 2: Create a Kubernetes service cluster

* Run the following code **lines one-by-one** in the **Cloud Shell**:

```yaml
gcloud config set compute/zone us-east1-b
```
```yaml
gcloud container clusters create [ANY NAME]
```
```yaml
gcloud container clusters get-credentials [SAME NAME AS ABOVE ONE]
```
```yaml
kubectl create deployment hello-server --image=gcr.io/google-samples/hello-app:2.0
```
```yaml
kubectl expose deployment hello-server --type=LoadBalancer --port [APP PORT NUMBER]                    
```
### Task 3: Setup an HTTP load balancer

* Run the following code **blocks one-by-one** in the **Cloud Shell**:

```yaml
gcloud compute instance-templates create web-server-template \
          --metadata-from-file startup-script=startup.sh \
          --network nucleus-vpc \
          --machine-type g1-small \
          --region us-east1
```
```yaml
gcloud compute instance-groups managed create web-server-group \
          --base-instance-name web-server \
          --size 2 \
          --template web-server-template \
          --region us-east1
 ```        
 ```yaml         
gcloud compute firewall-rules create [REPLACE YOUR FIREWALL NAME WHICH IS MENTIONED ON LEFT SIDE] \
          --allow tcp:80 \
          --network nucleus-vpc
gcloud compute http-health-checks create http-basic-check
gcloud compute instance-groups managed \
          set-named-ports web-server-group \
          --named-ports http:80 \
          --region us-east1
 ```         
 ```yaml        
gcloud compute backend-services create web-server-backend \
          --protocol HTTP \
          --http-health-checks http-basic-check \
          --global
gcloud compute backend-services add-backend web-server-backend \
          --instance-group web-server-group \
          --instance-group-region us-east1 \
          --global
```          
```yaml
gcloud compute url-maps create web-server-map \
          --default-service web-server-backend
gcloud compute target-http-proxies create http-lb-proxy \
          --url-map web-server-map
```
```yaml       
gcloud compute forwarding-rules create http-content-rule \
        --global \
        --target-http-proxy http-lb-proxy \
        --ports 80
```
```yaml
gcloud compute forwarding-rules list
```
