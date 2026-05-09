# cn-helloworld

Smallest possible cluster smoke-test for [amun-kubernetes](https://github.com/GonzaloAlvarez/amun-kubernetes):
an echo server with 2 replicas, a 1Gi Longhorn PVC, and an Ingress.
Reachable as **http://hello.k8s.lan** once the cluster + DNS are up.

## Deploy

Standalone:

```sh
kubectl apply -k k8s/
```

Or list it in [`amun-kubernetes/deployment/deployment.yml`](https://github.com/GonzaloAlvarez/amun-kubernetes/blob/main/deployment/deployment.yml)
(it's there by default) and run that:

```sh
cd /path/to/amun-kubernetes
kubectl apply -k deployment/
```

## Health check

```sh
kubectl get pods -l app=helloworld -o wide        # 2 replicas, on different nodes
kubectl get pvc helloworld-data                   # Bound, longhorn StorageClass
kubectl get ingress helloworld
curl -sS http://hello.k8s.lan/ | jq .             # echoes the request

# storage round-trip (writes to the Longhorn PVC)
kubectl exec deploy/helloworld -- sh -c 'date >> /data/log; cat /data/log'
```

## Two-failure resilience drill

This is the headline check of the cluster's promise: kill any 2 storage nodes
and writes still succeed.

```sh
ssh rpid3.lan sudo systemctl stop k3s-agent
ssh rpid4.lan sudo systemctl stop k3s-agent
sleep 60

kubectl exec -it deploy/helloworld -- sh -c 'echo "still alive: $(date)" >> /data/log'
kubectl exec -it deploy/helloworld -- cat /data/log    # write succeeded

# restore
ssh rpid3.lan sudo systemctl start k3s-agent
ssh rpid4.lan sudo systemctl start k3s-agent
```

## Remove

```sh
kubectl delete -k k8s/
```

## License

GNU GPL v3. Copyright (c) 2026 Gonzalo Alvarez.
