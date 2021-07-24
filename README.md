## Managing nodes
`kubectl get nodes`  

```
NAME               STATUS    ROLES                  AGE   VERSION
ip-172-31-19-204   Ready     <none>                 27d   v1.21.2
ip-172-31-22-126   Ready     <none>                 27d   v1.21.2
ip-172-31-28-65    Ready     control-plane,master   27d   v1.21.2
```  

`kubectl get pods -o wide`  

```
root@ip-172-31-28-65:~# kubectl get pods -o wide
NAME                              READY   STATUS             RESTARTS   AGE     IP                NODE               NOMINATED NODE   READINESS GATES
web-7b7d55d9cd-2c6bs              1/1     Running            0          64s     192.168.92.244    ip-172-31-19-204   <none>           <none>
web-7b7d55d9cd-ccrgg              1/1     Running            0          64s     192.168.155.186   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-d79x5              1/1     Running            0          64s     192.168.155.140   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-dbpss              1/1     Running            0          64s     192.168.155.143   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-dkmkw              1/1     Running            0          64s     192.168.155.187   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-g6tmr              1/1     Running            0          64s     192.168.155.139   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-hkm56              1/1     Running            0          64s     192.168.92.251    ip-172-31-19-204   <none>           <none>
web-7b7d55d9cd-hsqd4              1/1     Running            0          64s     192.168.155.129   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-kd8sn              1/1     Running            0          64s     192.168.92.242    ip-172-31-19-204   <none>           <none>
web-7b7d55d9cd-vwfj6              1/1     Running            0          64s     192.168.92.241    ip-172-31-19-204   <none>           <none>
```  

to cordon node:

`kubectl cordon ip-172-31-19-204`  

```
NAME               STATUS                     ROLES                  AGE   VERSION
ip-172-31-19-204   Ready,SchedulingDisabled   <none>                 27d   v1.21.2
ip-172-31-22-126   Ready                      <none>                 27d   v1.21.2
ip-172-31-28-65    Ready                      control-plane,master   27d   v1.21.2
```  

to drain node:

`kubectl drain ip-172-31-19-204`  

and then check the pods:

`kubectl get pods -o wide`  

```
NAME                              READY   STATUS             RESTARTS   AGE     IP                NODE               NOMINATED NODE   READINESS GATES
web-7b7d55d9cd-ccrgg              1/1     Running            0          2m52s   192.168.155.186   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-d79x5              1/1     Running            0          2m52s   192.168.155.140   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-dbpss              1/1     Running            0          2m52s   192.168.155.143   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-dkmkw              1/1     Running            0          2m52s   192.168.155.187   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-fhz8x              1/1     Running            0          24s     192.168.155.135   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-g6tmr              1/1     Running            0          2m52s   192.168.155.139   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-hsqd4              1/1     Running            0          2m52s   192.168.155.129   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-qfvkb              1/1     Running            0          24s     192.168.155.141   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-r8fw6              1/1     Running            0          24s     192.168.155.132   ip-172-31-22-126   <none>           <none>
web-7b7d55d9cd-sch44              1/1     Running            0          24s     192.168.155.137   ip-172-31-22-126   <none>           <none>
```  

web pods are evicted from Node1 and replaced in Node2

## Autoscaling
`kubectl apply -f web-autoscaling.yml`  

to check:

`kubectl get hpa-service`  

to generate loads:

`kubectl run -i --tty load-generator --rm --image busybox --restart Never -- /bin/sh -c "while sleep 0.01; do wget -q -O- http://php-apache:7000; done"`  

then do:

`watch kubectl get hpa-service`  

## ClusterRoles and ClusterRoleBindings
`kubectl apply -f cluster-role.yml`  
`kubectl apply -f cluster-role-binding.yml`  

to verify:

`kubectl auth can-i create pods --as cluster-admin --namespace kube-system`  
`kubectl auth can-i create node --as cluster-admin`  
`kubectl auth can-i delete node --as cluster-admin`

## Roles and Bindings
`kubectl create serviceaccount simplilearn`  

to verify:

`kubectl get role`  
`kubectl get rolebinding`  
`kubectl auth can-i get pods --as amit`
`kubectl auth can-i create pods --as amit`
`kubectl auth can-i delete pods --as amit`

## initContainer
`kubectl apply -f pod-initContainer.yml`  

expect from describe command to see 2 images -- alpine and nginx used:

```
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  34s   default-scheduler  Successfully assigned default/pod-init to ip-172-31-22-126
  Normal  Pulling    34s   kubelet            Pulling image "alpine"
  Normal  Pulled     32s   kubelet            Successfully pulled image "alpine" in 987.178154ms
  Normal  Created    32s   kubelet            Created container r-n-g
  Normal  Started    32s   kubelet            Started container r-n-g
  Normal  Pulling    31s   kubelet            Pulling image "nginx"
  Normal  Pulled     30s   kubelet            Successfully pulled image "nginx" in 1.004103224s
  Normal  Created    30s   kubelet            Created container nginx
  Normal  Started    30s   kubelet            Started container nginx
```

run curl ip-address and expect some random number

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
`kubectl apply -f mysql-with-secrets.yml`  
`kubectl describe pod mysql-pod-with-secret`

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
`kubectl create configmap nginx-cm --from-file html/index.html`  
`kubectl describe cm nginx-cm`  
`kubectl apply -f pod-with-cm.yml`  
`kubectl describe pod nginx-with-cm`

expect the following from describe command:

```yaml
Containers:
  nginx-with-cm:
    Mounts:
      /usr/share/nginx/html/index.html from html (rw,path="index.html")
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-ln254 (ro)
```

## Monitoring using Metrics Server
`kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`  
`kubectl edit -n kube-system deployment metrics-server`

add a line under "Args"
`- --kubelet-insecure-tls`

`kubectl top pod`  
`kubectl top node`

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

## Run pod with multi containers
`kubectl apply -f pod-with-multi-containers.yml`  
`kubectl describe pod pod-with-logs`  
`kubectl logs pod-with-logs -c nginx`  
`kubectl logs pod-with-logs -c nginx`

should see logs from both containers

