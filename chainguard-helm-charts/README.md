# Chainguard Helm Charts Catalog

## Installing ArgoCD

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

## Create a plugin using custom-assembly

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
```

## Create an assumable identity representing the argocd repo server

```bash
chainctl iam identities create argocd-plugin \
    --issuer-keys="$(kubectl get --raw /openid/v1/jwks)" \
    --identity-issuer=https://kubernetes.default.svc.cluster.local \
    --subject=system:serviceaccount:argocd:argocd-repo-server \
    --role=registry.pull
```

## Deploy a chart

Charts have been packaged as ArgoCD applications easing in the deployment. To deploy a chart:
```bash
kubectl apply -f apps/<app-name>.yaml
```