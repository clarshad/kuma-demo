# README
* The following steps are inspired by the [OPA Envoy Tutorial](https://www.openpolicyagent.org/docs/latest/envoy-authorization/), yet based on the [Kuma Kubernetes Quickstart](https://kuma.io/docs/0.7.1/quickstart/kubernetes/)

Note: there are several versions of the core app YAML that differ around policy maangement
1. kuma-demo-ns.yaml: original kuma-demo YAML without OPA
1. kuma-demo-app-with-opa.yaml: using OPA with ConfigMaps (explained in detail below)
1. playground-kuma-demo-app.yaml: using OPA with Playground
1. das-kuma-demo-app.yaml: using OPA with DAS

## Steps

### 1. Start Minikube

### 2. Download Kuma and Install the Kuma Control Plane
* https://kuma.io/docs/0.7.1/installation/kubernetes/

### 3. Create the `kuma-demo` namespace
```
kubectl apply -f kuma-demo-ns.yaml
```

### 4. Create the OPA policy
```
kubectl create secret generic opa-policy --from-file policy.rego -n kuma-demo
```

### 5. Deploy the Kuma quickstart demo app
The quickstart demo app YAML has been updated to:
* Update the frontend service to type `NodePort`
* Add OPA sidecar to the frontend (i.e. app) deployment
* Add OPA sidecar to the backend-v0 deployment
```
kubectl apply -f kuma-demo-app-with-opa.yaml

# wait for pods to start
kubectl get pods -w -n kuma-demo

# test out the app - we haven't yet applied the HTTP Filter for OPA - so everything should work fine

# get the frontend service URL and open in browser
minikube service list -n kuma-demo

# port-forward so you can use the browser to
kubectl port-forward svc/frontend -n kuma-demo 8080:8080
```

### 6. Apply the ProxyTemplate with the HTTP Filter config
* The current `policy.rego` is very simple - it allows all GETs
```
# start logging in the namespace to watch activity in a second terminal
kubectl tail -n kuma-demo

# apply proxy template
kubectl apply -f proxy-template.yaml

# Refresh the page in the browser and you will see successful connectivity to OPA and allow decisions in the logs for both the frontend and backend pods
```

