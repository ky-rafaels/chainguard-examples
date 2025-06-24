# Keycloak Deployment - Iamguarded

## *Optional* Exposing Keycloak from Local Ingress - Kind

```bash
kubectl apply -f https://kind.sigs.k8s.io/examples/ingress/deploy-ingress-nginx.yaml

# Save ExternalIP of Ingress 
INGRESS=$(kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}')

cat <<EOF | sudo tee -a /etc/hosts
${KEYCLOAK_IP} keycloak.example.com
EOF
```