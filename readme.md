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

## 5. Specifying a config cluster

The config cluster is a GKE cluster you choose to be the central point of control for Ingress across the member clusters. Unlike GKE Ingress, the Anthos Ingress controller does not live in a single cluster but is a Google-managed service that watches resources in the config cluster. This GKE cluster is used as a multi-cluster API server to store resources such as MultiClusterIngress and MultiClusterService. Any member cluster can become a config cluster, but there can only be one config cluster at a time.

For more information about config clusters, see [Config cluster design](https://cloud.google.com/kubernetes-engine/docs/concepts/ingress-for-anthos#config_cluster_design).

If the config cluster is down or inaccessible, then MultiClusterIngress and MultiClusterService objects cannot update across the member clusters. Load balancers and traffic can continue to function independently of the config cluster in the case of an outage.

Enabling Ingress for Anthos and selecting the config cluster occurs in the same step. The GKE cluster you choose as the config cluster must already be registered as a member to Hub.

Identify the URI of the cluster you want to specify as the config cluster:

```sh
gcloud container hub memberships list
```

The output is similar to this:

```none
NAME                                  EXTERNAL_ID
gke-us                                0375c958-38af-11ea-abe9-42010a800191
gke-eu                                d3278b78-38ad-11ea-a846-42010a840114
```

Enable Ingress for Anthos and select gke-us as the config cluster:

```sh
gcloud alpha container hub ingress enable \
  --config-membership=projects/project_id/locations/global/memberships/gke-us
```

The output is similar to this:

```none
Waiting for Feature to be created...done.
```

Note that this process can take a few minutes while the controller is bootstrapping. If successful, the output is similar to this:

```none
Waiting for Feature to be created...done.
Waiting for controller to start...done.
```

If unsuccessful, the command will timeout like below:

```none
Waiting for controller to start...failed.
ERROR: (gcloud.alpha.container.hub.ingress.enable) Controller did not start in 2 minutes. Please use the `describe` command to check Feature state for debugging information.
```

If no failure occurred in the previous step, you may proceed with next steps. If a failure occurred in the previous step, then check the feature state. It should indicate what exactly went wrong:

```sh
gcloud alpha container hub ingress describe
```

An example failure state is below:

```none
featureState:
  detailsByMembership:
    projects/393818921412/locations/global/memberships/0375c958-38af-11ea-abe9-42010a800191:
      code: FAILED,
      description: "... is not a VPC-native cluster..."
lifecycleState: ENABLED
multiclusteringressFeatureSpec:
  configMembership: projects/project_id/locations/global/memberships/0375c958-38af-11ea-abe9-42010a800191
name: projects/project_id/locations/global/features/multiclusteringress
updateTime: '2020-01-22T19:16:51.172840703Z'
```

To find out about more such error messages, see [Troubleshooting and operations](https://cloud.google.com/kubernetes-engine/docs/how-to/troubleshooting-and-ops).

## 6. Deploy zoneprinter app

+ First of all, get your cluster credentials through the GCP UI (Kubernetes Engine > Clusters > Connect). Your connection string should look like this : `gcloud container clusters get-credentials gke-us --zone us-central1-a --project project_id`

+ Follow instructions [here](https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-for-anthos)
