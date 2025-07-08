# Chainguard Helm Charts Catalog

## Helm basics with Chainguard iamguarded charts

```bash
# First login with chainctl, helm will use your local docker credentials to authenticate to the OCI repo
chainctl auth login && chainctl auth configure-docker
# check if you can view the chart and its values
helm show values oci://cgr.dev/ky-rafaels.example.com/iamguarded-charts/keycloak
# OR 
helm show all oci://cgr.dev/ky-rafaels.example.com/iamguarded-charts/keycloak
# Then install the chart 
helm upgrade --install keycloak \
-n keycloak \
--create-namespace \
oci://cgr.dev/ky-rafaels.example.com/iamguarded-charts/keycloak
```

## Using ArgoCD

```bash
helm repo add argocd https://argoproj.github.io/argo-helm
helm upgrade --install argocd argocd/argo-cd \
    --namespace argocd \
    --create-namespace \
    --values argo/argo-values.yaml
```

<!-- # Install keycloak iamguarded chart
chainctl auth login && chainctl auth configure-docker --pull-token --save
helm upgrade --install keycloak oci://cgr.dev/ky-rafaels.example.com/iamguarded-charts/keycloak -n keycloak --create-namespace --set global.org=ky-rafaels.example.com -->


### *Optional* Create pull token on kind nodes for cgr registry

Your pull token should already be created if you followed the steps above to deploy keycloak. Run script to add the pull token to each of the nodes

```bash
./scripts/kind-chainguard-pull-token.sh
```

## Create an assumable identity representing the argocd repo server

```bash
chainctl iam identities create argocd-repo-server \
    --issuer-keys="$(kubectl get --raw /openid/v1/jwks)" \
    --identity-issuer=https://kubernetes.default.svc.cluster.local \
    --subject=system:serviceaccount:argocd:argocd-repo-server \
    --parent=ky-rafaels.example.com \
    --role=registry.pull
```

## Create a secret for ArgoCD Repo Server

```bash
cat << EOF >> cgr-helm-secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: cgr-oci-repo
  namespace: argocd
  labels:
    argocd.argoproj.io/secret-type: repository
stringData:
  type: helm
  name: cgr-oci-repo
  url: cgr.dev/ky-rafaels.example.com/iamguarded-charts
  enableOCI: "true"
  ForceHttpBasicAuth: "true"
  username: argocd-repo-server 
  password: <insert-token-here> # <- how can we pass token from volume?
EOF

kubectl apply -f cgr-helm-secret.yaml
```

## Get a token and test AuthN

```bash
TOKEN=$(kubectl create token argocd-repo-server -n argocd --audience https://issuer.enforce.dev)
helm 
```

<!-- ## Create a plugin using custom-assembly

First, ensure that you have the packages necessary for the argocd-plugin available in your private apk repo as well as the chainguard-base image. 

```bash
cat << EOF >> argocd-plugin.yaml
contents:
  packages:
    - jq
    - yq 
    - helm 
    - bash-binsh
EOF
```

Then generate a package file and create the image we will use as our argocd plugin. We will use this plugin help the argocd repo server to authenticate with the chainguard registry to read charts.

```bash
chainctl image repo build apply -f custom-assembly/argo-plugin-apks.yaml --parent ky-rafaels.example.com --repo custom-base
``` -->

## Deploy an app from a chart

Charts have been packaged as ArgoCD applications easing in the deployment. To deploy a chart:
```bash
kubectl apply -f apps/<app-name>.yaml
```