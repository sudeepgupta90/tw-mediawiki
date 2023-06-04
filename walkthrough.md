# Walkthtough

## Problem

- Setup [MediaWiki](https://www.mediawiki.org/wiki/MediaWiki) application on Kubernetes &#9745; 
- Use some Configuration Management tool (suggested in description was Terraform) &#9745;

### Additional Objective
The above automation should support CI/CD practices of chosen deployment style like Rolling Update or BlueGreen Deployment. (Optional) &#9745;

## Solution

### Artefacts

- Helm chart was found which is maintained by Bitnami [here](https://github.com/bitnami/charts/tree/main/bitnami/mediawiki)
- Application [Docker container](https://github.com/bitnami/containers/blob/main/bitnami/mediawiki) is also maintained by Bitnami, as Bitnami (Provider) requires corresponding Docker containers for every chart it provides

### Infrastructure Components

- Kubernetes: [minikube](https://minikube.sigs.k8s.io/) was used to emulate a K8s cluster on local
- ArgoCD: [ArgoCD](https://argo-cd.readthedocs.io/en/stable/) was used for Continuous Deployment and to achieve the [Additional Objective](#additional-objective)
- ArgoCD Autopilot: [ArgoCD Autopilot](https://argocd-autopilot.readthedocs.io/en/stable/) to bootstraps the ArgoCD deployment

### Repository Walkthrough

The below code block lays out the repo structure:

```
.
├── README.md
├── apps
│   └── README.md
├── bootstrap   # deploys the ArgoCD core
│   ├── argo-cd
│   │   └── kustomization.yaml
│   ├── argo-cd.yaml
│   ├── cluster-resources
│   │   ├── in-cluster
│   │   │   ├── README.md
│   │   │   └── argocd-ns.yaml
│   │   └── in-cluster.json
│   ├── cluster-resources.yaml
│   └── root.yaml
├── helm-charts # colocate the charts in the same repo for easy reference
│   └── mediawiki
│       ├── Chart.lock
│       ├── Chart.yaml
│       ├── charts
│       │   └── mediawiki-15.1.2.tgz # commit the chart to prevent pulling everytime
│       └── values.yaml
├── projects
│   ├── README.md
│   └── thoughtworks.yaml   # project config for *thoughtworks* project for a specific cluster
├── thoughtworks    # this app deploys in a specific cluster
│   ├── Chart.yaml
│   ├── templates
│   │   └── mediawiki.yaml
│   └── values.yaml
└── walkthrough.md

```

### ArgoCD Setup for Multiple Deploys or Blue/Green deploys

**HOW?**

This [project](/projects/thoughtworks.yaml) named as `thoughtworks` deploys to a specific cluster which can be assumed as `dev`
Likewise, we can create another project, which can be named as `thoughtworks-prod` and can be used to deploy another copy of the same application

Typically, the infrastructure for Blue/Green deployment is divided into as *dev* and *prod*. The `.kubecontext` can be saved for each cluster, and be added as an application secret. This destination can eventually be linked to the project as already explained above.

If you notice the [thoughtworks app](/thoughtworks/) we have implemented the Argo *App of Apps* pattern. This can hence be used to deploy and logic group several applications together.

**App of Apps**
There can be multiple ways this can actually be interpreted/implemented. In our case, we implemented this as a dev cluster application in the organisation thoughtworks

Typically, applications can be organised/scoped to as
- application
- and, destination 

being the most common and easy to understandably demarcate/organise

### Setup instructions

- Deploy minikube as ```minikube start```
- Open another terminal, and start a Load Balancer ```minikube tunnel```
- Setup ArgoCD using the AutoPilot and it's prequisites as [here](https://argocd-autopilot.readthedocs.io/en/stable/Getting-Started/)
- Create
    - [Application Project](/projects/thoughtworks.yaml) which deploys the apps to dev cluster
    - [App of Apps](/thoughtworks/) which groups deploys all apps from one place to a destination as speficied by project config
    - Colocate chart in the repo as [helm-charts/mediawiki](/helm-charts/mediawiki/)
    - [Application Manifest](/thoughtworks/templates/mediawiki.yaml) which is scoped to a project config and deploys accordingly

### Screenshots

#### mediawiki application homepage

![mediawiki_homepage](/.media/mediawiki_homepage.png)

#### ArgoCD App Homepage

![argo_app_homepage](/.media/argo_app_homepage.png)

#### Argo App of Apps

![app_of_apps](/.media/app_of_apps.png)

#### Argo MediaWiki App 

![app](/.media/app.png)
