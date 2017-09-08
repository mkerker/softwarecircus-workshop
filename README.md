# Buzz-word compliant CI/CD: building and deploying in the cloud
## bol.com Software Circus workshop

During this workshop we'll install a modern CI/CD pipeline in the cloud. To do this we'll be using the Google Cloud platform to run a GKE cluster, upon which we will install Gitlab CI and Spinnaker. We'll user the Google Container Registry and Google Container Builder during the process as well.

### Prerequisites
If you want to follow along as we go, it's best to have the following things ready:

- A GCP account to create your cloud resources (if you don't have one already, simply sign up [here](https://cloud.google.com/free/) for $300 worth of free credits, which is more than enough to complete the workshop)
- A GCP project (call it "circus" for simplicity's sake ;)
- [Google Cloud SDK](https://cloud.google.com/sdk/) (for manipulating cloud resources)

### Getting our cloud resources setup
We're going to need 2 things to be set up in GCP: a GKE cluster to run Gitlab CI and Spinnaker, and a VM to run Halyard (the Spinnaker installer). Both of them can be set up using `gcloud` commands:

- For the GKE cluster run
`gcloud beta container --project "<project-name>" clusters create "sc" --zone "europe-west1-d" --cluster-version "1.7.4" --num-nodes "3"`

- For the Halyard vm run
`gcloud compute --project "<project-name>" instances create "sch" --zone "europe-west1-d" --image "ubuntu-1404-trusty-v20170831" --image-project "ubuntu-os-cloud"`

### Installing Spinnaker
- Connect to the command line of your Halyard VM and run:

```curl -O https://raw.githubusercontent.com/spinnaker/halyard/master/install/stable/InstallHalyard.sh
sudo bash InstallHalyard.sh
```
- Setup the GCP storage for Spinnaker using this guide: https://www.spinnaker.io/setup/storage/gcs/
- Setup the Kubernetes Cloud provider for Spinnaker using this guide: https://www.spinnaker.io/setup/providers/kubernetes/
- Finally, set the Spinnaker version and install to the cluster:
```hal config version edit --version $VERSION
hal deploy apply
```

### Installing Gitlab CI
Once the GKE cluster is up and running we can install Gitlab CI on it. To do this we're going to use the kubectl tool to apply a Kubernetes manifest. 

Run the following commands in order:

```kubectl create -f gitlab-ns.yml
kubectl create -f gitlab/redis-svc.yml
kubectl create -f gitlab/redis-deployment.yml
kubectl create -f gitlab/postgresql-svc.yml
kubectl create -f gitlab/postgresql-deployment.yml
```

Now wait for all pods to become ready. Then run:
```kubectl create -f gitlab/gitlab-svc.yml
kubectl create -f gitlab/gitlab-deployment.yml
```

Now gitlab should be up and running at the IP address of your service.
