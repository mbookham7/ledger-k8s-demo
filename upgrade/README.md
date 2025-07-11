# Perform a Rolling Upgrade

This is done by updating the image tag in the statefulSet manifest.

```
      containers:
      - name: cockroachdb
        image: cockroachdb/cockroach:v23.2.1
        imagePullPolicy: IfNotPresent
```

Each manifest needs to be applied in turn, a region at a time to ensure there is no disruption to service.


Apply region one and observe the changes in the CockroachDB UI
```
kubectl patch statefulset cockroachdb -n $loc1 \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"cockroachdb/cockroach:v24.1.19"}]' \
  --context $clus1
```

Wait for the rollout to complete before moving on to next region.
```
kubectl patch statefulset cockroachdb -n $loc2 \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"cockroachdb/cockroach:v24.1.19"}]' \
  --context $clus2
```

Again wait for the rollout to finish and then finish with the final region
```
kubectl patch statefulset cockroachdb -n $loc3 \
  --type='json' \
  -p='[{"op": "replace", "path": "/spec/template/spec/containers/0/image", "value":"cockroachdb/cockroach:v24.1.19"}]' \
  --context $clus3
```