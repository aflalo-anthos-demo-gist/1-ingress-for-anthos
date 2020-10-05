# Ingress for Anthos demo

We want here to assess multiple things through this demo :

+ How to easily create clusters through GKE
+ Register those clusters on Anthos Hub
+ Demonstrate the interest of using Anthos on GCP through Ingress for Anthos

What we want to achieve :

![Schema](https://cloud.google.com/kubernetes-engine/images/anthos-ingress-traffic-flow.svg)

For more details, please read [Ingress for Anthos Documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-for-anthos)

## 1. Enable required APIs

Enable the Hub API in your project: `gcloud services enable gkehub.googleapis.com`

Enable the Anthos API in your project: `gcloud services enable anthos.googleapis.com`

Enable the Ingress for Anthos API in your project:`gcloud services enable multiclusteringress.googleapis.com`

## 2. Create the gke-eu cluster

`gcloud container clusters create gke-eu --zone europe-west1-c --release-channel stable --enable-ip-alias`

## 3. Create the gke-us cluster

`gcloud container clusters create gke-us --zone us-central1-a --release-channel stable --enable-ip-alias`

## 4. Register clusters

Be sure to follow prerequisites for registering a cluster [here](https://cloud.google.com/anthos/multicluster-management/connect/prerequisites)

Get your clusters uri : `gcloud container clusters list --uri`

Then register gke-eu cluster :

```sh
gcloud container hub memberships register gke-eu \
    --project=project-id \
    --gke-uri=uri \
    --service-account-key-file=service-account-key-path
```

Then gke-us cluster :

```sh
gcloud container hub memberships register gke-us \
    --project=project-id \
    --gke-uri=uri \
    --service-account-key-file=service-account-key-path
```

Verify cluster registration by running : `gcloud container hub memberships list`

The output should look similar to this:

```none
NAME    EXTERNAL_ID
gke-us  2903db0e-0061-4354-8736-a481b6840f72
gke-eu  02c9bdd4-2d74-4454-8c00-2bccb8b07660
```

## 5. Deploy zoneprinter app

+ First of all, get your cluster credentials through the GCP UI (Kubernetes Engine > Clusters > Connect). Your connection string should look like this : `gcloud container clusters get-credentials gke-us --zone us-central1-a --project project_id`

+ Follow instructions [here](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-for-anthos)
