---
 type: post
 title: Multi-vCluster Deployment Made Easy with vCluster.Pro and Argo CD
 slug: multi-vcluster-deployment-made-easy-with-vcluster-pro-and-argo-cd
 authors:
   - kurt-madel
 tags:
   - vcluster
   - cost-optimization
   - enterprise
   - use-cases
 metaTitle: Multi-vCluster Deployment Made Easy with vCluster.Pro and Argo CD
 metaDescription: "vCluster for Kubernetes Demo Environments: ."
 comments: true
 ---

vCluster.Pro has long supported the automatic deployment of Helm charts and Kubernetes manifests to newly created vClusters instances by adding [vCluster.Pro Apps](https://www.vcluster.com/pro/docs/apps/what-are-apps) to a [vCluster.Pro vCluster template](https://www.vcluster.com/pro/docs/virtual-clusters/templates), and then using that vCluster template to create vCluster instances. This is a quick and easy way to automatically deploy the same application stack and Kubernetes resources to every vCluster created from that vCluster template - no Argo CD required. 

However, if Argo CD is your preferred way to manage the deployment of your Kubernetes applications and resources, then Loft's previously available vCluster.Pro integrations for Argo CD made it much easier to use vClusters with Argo CD. Basically, [that vCluster.Pro Argo CD integration](https://www.vcluster.com/pro/docs/virtual-clusters/argocd#enable-argo-cd-integration) provided automatic syncing of vClusters, created with vCluster.Pro, into Argo CD as an available cluster for application destinations. First, you connected an Argo CD instance to a vCluster.Pro project, then created a vCluster and flipped the 'Add to ArgoCD' toggle, and vCluster.Pro would auto-generate an Argo CD cluster `Secret` for that vCluster, in the `Namespace` where Argo CD is running, making that vCluster instantly available as cluster for application deployments. Here is an example of such a synced `Secret`:
```
apiVersion: v1
kind: Secret
metadata:
  name: loft-api-framework-vcluster-api-framework-qa
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
    loft.sh/vcluster-instance-name: api-framework-qa
    loft.sh/vcluster-instance-namespace: loft-p-api-framework
  annotations:
    co-managed-by: loft.sh
    managed-by: argocd.argoproj.io
data:
  ...
```
And to streamline the vCluster Argo CD integration even further, you could also configure a vCluster template to automatically add a vCluster instance created from that template to Argo CD - no toggle needed. This integration is great for use cases like CI automation, where you want an ephemeral Kubernetes environment to test and/or preview deployments with Argo CD.

But some of our customers have been using [Argo CD ApplicationSets](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/) with the [Cluster Generator](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Cluster/) and [label selectors](https://argo-cd.readthedocs.io/en/stable/operator-manual/applicationset/Generators-Cluster/#label-selector) to deploy applications to specific sets of vClusters based on `labels` applied to the Argo CD cluster `Secret`. However, there was no vCluster.Pro integrated way to automatically inject additional vCluster `labels` into the Argo CD cluster `Secret` to be used with the Cluster Generator label selector. So, our customers had to either manually update the Argo CD cluster `Secret` or find some other way to automate vCluster integration with Argo CD. But now, starting with version v3.3.2, vCluster.Pro has updated its Argo CD integration and now adds vCluster `metadata.labels` to the Argo CD cluster `Secret`. Here is an example `VirtualClusterInstance` manifest (a [CRD provided by vCluster.Pro](https://www.vcluster.com/pro/docs/api/resources/virtualclusterinstance/) for managing vClusters as plain Kubernetes manifests) with some custom `metadata.labels` and the resulting vCluster.Pro generated Argo CD cluster `Secret`:
```
apiVersion: management.loft.sh/v1
kind: VirtualClusterInstance
metadata:
  name: api-framework-qa
  namespace: loft-p-api-framework
  labels:
    env: qa
    loft.sh/import-argocd: 'true'
    team: api-framework
spec:
  owner:
    user: anna-smith
  templateRef:
    name: vcluster-pro-template
    version: 1.0.x
  clusterRef:
    cluster: loft-cluster
    namespace: loft-api-framework-v-api-framework-qa
    virtualCluster: api-framework-qa
  parameters: |
    k8sVersion: v1.27.8
    env: 'qa'
```
```
apiVersion: v1
kind: Secret
metadata:
  name: loft-api-framework-vcluster-api-framework-qa
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
    env: qa
    loft.sh/vcluster-instance-name: api-framework-qa
    loft.sh/vcluster-instance-namespace: loft-p-api-framework
    team: api-framework
  annotations:
    co-managed-by: loft.sh
    managed-by: argocd.argoproj.io
data:
  ...
```
The `env` and `team` vCluster `labels` were automatically added to the auto-generated and synced Argo CD cluster `Secret`. A couple other things to point out in regards to the above `VirtualClusterInstance` example is the `templateRef` and the `parameters`. The enhanced integration also includes `template.metadata.labels` that are part of vCluster templates, and vCluster template `parameters` can be used to populate the values of those `labels`. Here is an example vCluster.Pro `VirtualClusterTemplate` that leverages template `parameters` to set the values of the template `instanceTemplate.metadata.labels` and the resulting vCluster.Pro generated Argo CD cluster `Secret`:
```
apiVersion: management.loft.sh/v1
kind: VirtualClusterTemplate
metadata:
  name: vcluster-pro-template
spec:
  displayName: Virtual Cluster Pro Template
  description: This virtual cluster template deploys a vCluster.Pro virtual cluster
  template:
    metadata:
      labels:
        loft.sh/import-argocd: 'true'
    instanceTemplate:
      metadata:
        labels:
          env: '{{ .Values.env }}'
          team: '{{ .Values.loft.project }}'
    pro:
      enabled: true
    helmRelease:
      chart:
        version: v0.18.1
      values: >-
        # Use an embedded managed etcd server instead of using the k3s default SQLite backend
        embeddedEtcd:
          enabled: true
        coredns:
          integrated: true
        # Checkout https://vcluster.com/pro/docs/ for more config options
  parameters:
    - variable: env
      label: Deployment Environment
      description: >-
        Environment for deployments for this vCluster used as cluster label
        for Argo CD ApplicationSet Cluster Generator
      options:
        - dev
        - qa
        - prod
      defaultValue: dev
```
```
apiVersion: v1
kind: Secret
metadata:
  name: loft-api-framework-vcluster-api-framework-qa
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: cluster
    env: qa
    loft.sh/vcluster-instance-name: api-framework-qa
    loft.sh/vcluster-instance-namespace: loft-p-api-framework
    team: api-framework
  annotations:
    co-managed-by: loft.sh
    managed-by: argocd.argoproj.io
data:
  ...
```
Note that the value for the `env` label is populated with `'{{ .Values.env }}'` that will be the value of the `- variable: env` parameter. Also, note that the value of the `team` label is populated with `'{{ .Values.loft.project }}'`, the `Values.loft` parameters are provided automatically by the vCluster.Pro controller with a full list of those parameters available [here](https://www.vcluster.com/pro/docs/apps/parameters#vclusterpro-parameter-values).

Now, with the `labels` being added to the Argo CD cluster `Secret`, we are able to create an Argo CD `ApplicationSet` that utilized the Cluster generator with label selectors. Here is a simple example:
```
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: REPO_NAME-env-config
  namespace: argocd
spec:
  generators:
    - clusters:
        selector:
          matchLabels:
            env: "dev"
    - clusters:
        selector:
          matchLabels:
            env: "qa"
    - clusters:
        selector:
          matchLabels:
            env: "prod"
  template:
    metadata:
      # {{name}} is the name of the kubernetes cluster as selected by the spec above
      name: REPO_NAME-{{name}}
    spec:
      destination:
        # {{server}} is the url of the cluster
        server: '{{server}}'
        # {{metadata.labels.env}} is the value of the env label that is being used to select kubernetes clusters 
        # and used as sub-folder in the target git repository
        namespace: hello-world-app-{{metadata.labels.env}}
      info:
        - name: GitHub Repo
          value: https://github.com/loft-demos/REPO_NAME/
      project: default
      source:
        path: k8s-manifests/{{metadata.labels.env}}/
        repoURL: https://github.com/loft-demos/REPO_NAME.git
        targetRevision: main
      syncPolicy:
        automated:
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```
Note that we are not only using the `env` label value to select different clusters, we are also use the same `env' label value to create a dynamic `path` for the generated `Application` `spec.source.path` - `path: k8s-manifests/{{metadata.labels.env}}/`.

So, with this enhanced vCluster.Pro integration for Argo CD, using Argo CD to deploy the same Kubernetes workloads to multiple vClusters is easier than ever!