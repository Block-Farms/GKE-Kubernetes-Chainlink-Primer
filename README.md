# GKE-Kubernetes-Chainlink-Primer
GCP/GKE K8S Guide to deploy and monitor a Chainlink Cluster

#### Prerequisites
1. GCP Account
2. Project Created
3. PostgreSQL DB Server created, with username and password and database name setup
4. Websocket RPC Endpoint either cloud provider such as Infura, or self hosted (ie: ip:8546)

#### Login to google cloud sdk
```
gcloud auth login
```

#### Set the project:
```
gcloud config set project kubernetes-chainlink
```

#### Create GKE Cluster: 4core CPU & 8GB RAM, 10 GB SSD:
```
gcloud beta container --project "kubernetes-chainlink" clusters create "cluster-1" --zone "us-central1-c" --no-enable-basic-auth --cluster-version "1.22.8-gke.202" --release-channel "regular" --machine-type "e2-custom-4-8192" --image-type "COS_CONTAINERD" --disk-type "pd-ssd" --disk-size "10" --metadata disable-legacy-endpoints=true --scopes "https://www.googleapis.com/auth/devstorage.read_only","https://www.googleapis.com/auth/logging.write","https://www.googleapis.com/auth/monitoring","https://www.googleapis.com/auth/servicecontrol","https://www.googleapis.com/auth/service.management.readonly","https://www.googleapis.com/auth/trace.append" --max-pods-per-node "110" --num-nodes "1" --logging=SYSTEM,WORKLOAD --monitoring=SYSTEM --enable-ip-alias --network "projects/kubernetes-bf-chainlink/global/networks/default" --subnetwork "projects/kubernetes-bf-chainlink/regions/us-central1/subnetworks/default" --no-enable-intra-node-visibility --default-max-pods-per-node "110" --no-enable-master-authorized-networks --addons HorizontalPodAutoscaling,HttpLoadBalancing,GcePersistentDiskCsiDriver --enable-autoupgrade --enable-autorepair --max-surge-upgrade 1 --max-unavailable-upgrade 0 --enable-shielded-nodes --tags "chainlink-node" --node-locations "us-central1-c"
```

#### Once the cluster is created, you should see:
```
NAME       LOCATION       MASTER_VERSION  MASTER_IP     MACHINE_TYPE      NODE_VERSION    NUM_NODES  STATUS
cluster-1  us-central1-c  1.22.8-gke.202  35.225.229.3  e2-custom-4-8192  1.22.8-gke.202  1          RUNNING
```

#### Login to the K8S cluster:
```
gcloud container clusters get-credentials cluster-1 --zone us-central1-c --project kubernetes-chainlink
```

#### Create Chainlink Namespace to make efficient usage of the K8S physical resources:
```
kubectl create namespace chainlink
```

#### Confirm the namespace was created:
```
kubectl get namespaces --show-labels
```

#### Set the active namespace:
```
kubectl config set-context --current --namespace=chainlink
```

#### Upload `.api` which contains the email+password to access the GUI, and `.password` which contains the keystore password:
```
kubectl create secret generic api-env --from-file=".api"
kubectl create secret generic api-env --from-file=".password"
```

#### Upload HTTPS for the Chainlink Operator GUI:
```
kubectl create secret generic crt-env --from-file="server.crt"
kubectl create secret generic key-env --from-file="server.key"
```

#### Confirm the secrets are uploaded correctly:
```
kubectl get secret --namespace chainlink
```

#### Deploy the RPC Failover Pod and start its service:
```
kubectl apply -f "rpc-failover-deploy.yaml"
```

#### Upload the Chainlink `.env` file which will be referenced by the Chainlink Pod:
```
kubectl apply -f "node-env.yaml"
```

#### Deploy the Chainlink Pod and start the Chainlink service:
```
kubectl apply -f "deploy.yaml"
```

#### Verify the status of the Chainlink Pod Creation:
```
kubectl get pod -n chainlink
```
#### You should see the following:
```
NAME                            READY   STATUS    RESTARTS   AGE
chainlink-7c77f9cf-5xqvh        1/1     Running   0          74s
rpc-failover-7b78bfb988-xwt6t   1/1     Running   0          80s
```

#### Verify the Chainlink node logs are clean:
```
kubectl logs chainlink-7c77f9cf-5xqvh
```

#### Access the Chainlink Operator GUI via:
```
kubectl port-forward service/chainlink 6689
```
#### Enter your email and password found within `.api`:
```
<your gui email>
<your gui password>
```
