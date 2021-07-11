## Persistance Volume
`kubectl apply -f persistance-volume-exercise.yml` 
`kubectl get pv` 
`kubectl describe pv pv-exercise` 

expect from describe command:

``` yaml
Claim:           default/pvc-exercise
Reclaim Policy:  Retain
Access Modes:    RWO
VolumeMode:      Filesystem
Capacity:        2Gi
```

## MySQL with Secrets
- `kubectl apply -f mysql-with-secrets.yml`
- `kubectl describe pod mysql-pod-with-secret`

expect from describe command:

```yaml
Containers:
  mysql:
    Environment:
      MYSQL_ROOT_PASSWORD:  <set to the key 'ROOT_PASSWORD' in secret 'simplilearn-secret'>  Optional: false
      MYSQL_DATABASE:       <set to the key 'DATABASE' in secret 'simplilearn-secret'>       Optional: false
      MYSQL_USER:           <set to the key 'USER' in secret 'simplilearn-secret'>           Optional: false
      MYSQL_PASSWORD:       <set to the key 'PASSWORD' in secret 'simplilearn-secret'>       Optional: false
```

`kubectl exec mysql-pod-with-secret -- printenv`

## Nginx using ConfigMap
- `kubectl create configmap nginx-cm --from-file html/index.html`
- `kubectl describe cm nginx-cm`
- `kubectl apply -f pod-with-cm.yml`
- `kubectl describe pod nginx-with-cm`

expect the following from describe command:

```yaml
Containers:
  nginx-with-cm:
    Mounts:
      /usr/share/nginx/html/index.html from html (rw,path="index.html")
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ln254 (ro)
```

## Monitoring using Metrics Server
- `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
- `kubectl edit -n kube-system deployment metrics-server`

add a line under "Args"
`- --kubelet-insecure-tls`

- `kubectl top pod`
- `kubectl top node`

## Run pod with custom environment

add `env` under `spec.containers` data

```yaml
spec:
  containers:
    - env:
        - name: author
          value: azrad
        - name: job
          value: devops
```
