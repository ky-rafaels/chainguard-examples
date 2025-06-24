# Chainguard Helm Charts Catalog

## Installing ArgoCD and Keycloak

```bash
helm repo add argocd https://argoproj.github.io/argo-helm
helm upgrade --install argocd argocd/argo-cd \
    --namespace argocd \
    --create-namespace \
    --values argo/argo-values.yaml

# Install keycloak iamguarded chart
chainctl auth login && chainctl auth configure-docker --pull-token --save
helm upgrade --install keycloak oci://cgr.dev/ky-rafaels.example.com/iamguarded-charts/keycloak -n keycloak --create-namespace --set global.org=ky-rafaels.example.com
```

### *Optional* Create pull token on kind nodes for cgr registry

Your pull token should already be created if you followed the steps above to deploy keycloak. Run script to add the pull token to each of the nodes

```bash
./scripts/kind-chainguard-pull-token.sh
```

## Register ArgoCD with Keycloak

```bash
cat <<EOF | kubectl create -n default -f -

```

## Create a Keycloak Assumable Identity


## Deploy a chart

Charts have been packaged as ArgoCD applications easing in the deployment. To deploy a chart:
```bash
kubectl apply -f apps/<app-name>.yaml
```