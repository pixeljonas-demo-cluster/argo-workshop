# Hands-On ArgoCD Workshop

## Cluster Prerquisites

Install the following Operator to your cluter to get started:

- CodeReady Workspaces (for participant environments)
- OpenShift GitOps / ArgoCD (to have participants provision their own ArgoCD Instances)

Create a group for all participants and allow them to create ArgoCD Instances (You can find a Kustomize file at [cluster-preperation/rolebindings](./cluster-preperation/rolebindings)).

## Workshop Agenda

- Provision your own Workspace
- Deploy an ArgoCD Instance
- Deploy an App via Helm

# Provision your own Workspace

- Fork this repository
- Change the repo in the devfile
  - `https://github.com/<your-username>/argo-workshop.git`
- Login to CodeReady Workspaces
  - username: `userx` (e.g. `user30`)
  - password: `r3dh4t1!`
- Hover above your workspace (`userx-workspace`) and click on `Edit Workspace`
  - ![Edit Workspace](/images/codeready_workspaces_config.png "CodeReady Workspaces Config")
- Copy [the devfile](./workshop-devfile.yaml) to your CodeReady Workspace instance
- restart the workspace and click on "**Yes, I trust**"
- Start a new Terminal in the `openshift-tools`
- Login to the cluster using your username
  - `oc login https://$KUBERNETES_SERVICE_HOST:$KUBERNETES_SERVICE_PORT --insecure-skip-tls-verify=true -p r3dh4t1! -u userX`

## Making sure we can commit to your fork

- Create a new SSH Key using
  `ssh-keygen -t ed25519 -C "userx@argo-workshop" -f /home/jboss/config/id_ed25519`
- Start the SSH-Agent `eval "$(ssh-agent -s)"`
- Add the created SSH-Key to the SSH-Agent `ssh-add /home/jboss/config/id_ed25519`
- Copy the public SSH Key `cat /home/jboss/config/id_ed25519.pub`
- Add the key to your repo as "Deploy Key"
  - click on `Settings` -> `Security/Deploy keys` -> `Add deploy key`
  - Enter a title -> Paste the key -> click on `Allow write access`
  - click on `Add key`
- Change the URL for Origin from HTTP to SSH (so that we can use this key to push)
  - `git remote set-url origin git@github.com:<your-username>/argo-workshop.git`
- Make a small change and see if you have push-access to your repo

## Deploying your ArgoCD Instance

- change the name of your ArgoCD Instance
- create a namespace for your `ArgoCD` Instance
  - `oc new-project userX-argo`
- create your ArgoCD Instance
  - `oc apply -f argocd.yaml -n userX-argo`
- open your ArgoCD Instance
  - `oc get route -n userX-argo`
  - copy the Hostname
  - `Login via OpenShift`
  - Username: `userX` | password: `r3dh4t1!`

## Deploy an App via Helm

- Create a namespace for this App
  - `oc new-project userX-sharry`
- Tell the ArgoCD Operator, that this namespace is to be managed by your ArgoCD Instance (this is usually done by an administrator)
  - `oc label namespace userX-sharry argocd.argoproj.io/managed-by=userX-argo`

- go to [manifests/apps/sharry](./manifests/apps/sharry/)
- take a look at [sharry-app.yaml](./manifests/apps/sharry/base/sharry-app.yaml).
  - Change the `.metadata.namespace` to your users namespace e.g. `userX-argo`
  - Change the `.spec.destination.namespace` to your users namespace e.g. `userX-sharry`
  - Change the `.spec.source.repoURL` to your repository e.g. `https://github.com/<your-username>/argo-workshop.git`

The initial App will deploy multiple other apps (one for PostgreSQL and one for Sharry). This is called the `app-of-apps` pattern.

- Take a look at [sharry-chart.yaml](./manifests/apps/sharry/base/apps/sharry-chart.yaml) and [postgresql-chart.yaml](./manifests/apps/sharry/base/apps/postgresql-chart.yaml).
  - Change the namespace for these both and the destination to your users namespaces
    - `metadata.namespace: userX-argo`
    - `spec.destination.namespace: userX-sharry`

- run `oc apply --dry-run=client -o yaml -k ./manifests/apps/sharry/base` and verify, that your initial app has the correct namespaces
- run `oc apply --dry-run=client -o yaml -k ./manifests/apps/sharry/base/apps` and verify that the other apps have the correct namespace as well

- !Push these changes to your Repository!
  - `git add .`
  - `git commit -m "change namespaces to my user"`
  - `git push`

- Apply the first App to the cluster
  - `oc apply -k ./manifests/apps/sharry/base -n userX-argo`
- Look at your ArgoCD Instance and manual sync everything.
- Expose your app
  - `oc create route edge --service=sharry --insecure-policy='Redirect'`
  - `oc get route -n userX-sharry`

If we got more time:
Turn On "Auto-Sync" manually -> Route will disappear because it is not in GIT
Add the route to your git - see how it is created
