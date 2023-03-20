# k8s-pet-clinic-splunk-otel


prereq 

- Kuberenetes cluster.
- PORT 8080 open on the machine

## Deploy the non-instrumented version of the pet clinic

```
kubectl apply -f namespace.yaml
```

```
kubectl apply -f spring-petclinic.yaml
```

```
kubectl apply -f spring-service.yaml
```
- Validate that you can see the pods running 

```
kubectl -n petclinic get pods 
```

- delete the petclinic 

```
kubectl -n petclinic delete -f spring-petclinic.yaml
```

## Deploy the instrumented version of the pet clinic

- deploy the splunk otel collector 
```
helm repo add splunk-otel-collector-chart https://signalfx.github.io/splunk-otel-collector-chart && helm repo update
```

```
helm install splunk-otel-collector \
--set="splunkObservability.realm=$REALM" \
--set="splunkObservability.accessToken=$ACCESS_TOKEN" \
--set="clusterName=$(hostname)-k3s-cluster" \
--set="splunkObservability.logsEnabled=true" \
splunk-otel-collector-chart/splunk-otel-collector \
--namespace splunk \
--create-namespace
```

- deploy the instrumented version of the pet clinic

```
kubectl apply -f instru-spring-petclinic.yaml
```

-----
To get access to the petclinic UI and generate load

```
kubectl -n petclinic port-forward --address 0.0.0.0 svc/petclinic 8080
```

## Deploy the auto-config instrumented version of the pet clinic

- deploy the auto-instrumented version of the pet clinic

```
kubectl apply -f auto-instru-spring-petclinic.yaml
```

ultimately we are  using the default image with additionnal annotations 
```
  template:
    metadata:
      annotations:
        otel.splunk.com/inject-java: "true"
```
