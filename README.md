# argocd + gloo edge + flagger canary demo

## Prerequisites
- Gloo Edge installed on a Kubernetes cluster
- Flagger installed on a Kubernetes cluster

## Usage
If the prerequisites above are met, you can just deploy the base overlay argo app-of-app. This will deploy the podinfo application v6.0.0
```
kubectl apply -f https://raw.githubusercontent.com/ably77/canary-flagger-demo/main/base-overlay-aoa.yaml
```

Check the test namespace
```
% k get pods -n test
NAME                                  READY   STATUS    RESTARTS   AGE
flagger-loadtester-6cb5cdcd75-c5lm9   1/1     Running   0          3h49m
podinfo-primary-7db76b46bb-ls5xk      1/1     Running   0          119s
podinfo-primary-7db76b46bb-t2lqk      1/1     Running   0          119s
```

## Progressive Delivery
In this example we are going to promote to podinfo v6.0.1 which is part of the dev overlay
```
kubectl apply -f https://raw.githubusercontent.com/ably77/canary-flagger-demo/main/dev-overlay-aoa.yaml
```

## Watch flagger logs
```
kubectl -n gloo-system logs deployment/flagger -f | jq .msg
```

## Successful output
Below is the output of a successful progressive delivery. Note that at the beginning of the test, a few 500 errors were injected by hitting the `/status/500` endpoint in order to visualize the halting process. 
```
"New revision detected! Scaling up podinfo.test"
"canary deployment podinfo.test not ready: waiting for rollout to finish: 0 of 2 (readyThreshold 100%) updated replicas are available"
"canary deployment podinfo.test not ready: waiting for rollout to finish: 0 of 2 (readyThreshold 100%) updated replicas are available"
"Starting canary analysis for podinfo.test"
"Pre-rollout check acceptance-test passed"
"Advance podinfo.test canary weight 5"
"Halt advancement no values found for gloo metric request-success-rate probably podinfo.test is not receiving traffic: running query failed: no values found"
"Halt podinfo.test advancement podinfo-prometheus 8.57 > 1"
"Halt podinfo.test advancement podinfo-prometheus 3.61 > 1"
"Halt podinfo.test advancement podinfo-prometheus 2.28 > 1"
"Halt podinfo.test advancement podinfo-prometheus 1.68 > 1"
"Halt podinfo.test advancement podinfo-prometheus 1.14 > 1"
"Advance podinfo.test canary weight 10"
"Advance podinfo.test canary weight 15"
"Advance podinfo.test canary weight 20"
"Advance podinfo.test canary weight 25"
"Advance podinfo.test canary weight 30"
"Advance podinfo.test canary weight 35"
"Advance podinfo.test canary weight 40"
"Advance podinfo.test canary weight 45"
"Advance podinfo.test canary weight 50"
"Copying podinfo.test template spec to podinfo-primary.test"
"podinfo-primary.test not ready: waiting for rollout to finish: 1 old replicas are pending termination"
"HorizontalPodAutoscaler v2 podinfo-primary.test updated"
"Routing all traffic to primary"
"Promotion completed! Scaling down podinfo.test"
```