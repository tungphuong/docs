# Setup Consul 

## Locally

You can run Consul locally using Docker:

```
docker run -d --name=dev-consul -e CONSUL_BIND_INTERFACE=eth0 consul
```

You can then interact with the server using `localhost:8500`.

## Kubernetes

The easiest way to install Consul on Kubernetes is by using the [Helm chart](https://github.com/helm/charts/tree/master/stable/consul):

```
helm install consul stable/consul
```

This will install Consul into the `default` namespace.
To interact with Consul, find the service with: `kubectl get svc consul`.

For example, if installing using the example above, the Consul host address would be:

`consul.default.svc.cluster.local:8500`

## Create a Dapr component

The next step is to create a Dapr component for Consul.

Create the following YAML file named `consul.yaml`:

```
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <name>
spec:
  type: state.consul
  metadata:
  - name: datacenter
    value: <REPLACE-WITH-DATA-CENTER> # Required. Example: dc1
  - name: httpAddr
    value: <REPLACE-WITH-CONSUL-HTTP-ADDRESS> # Required. Example: "consul.default.svc.cluster.local:8500"
  - name: aclToken
    value: <REPLACE-WITH-ACL-TOKEN> # Optional. default: ""
  - name: scheme
    value: <REPLACE-WITH-SCHEME> # Optional. default: "http"
  - name: keyPrefixPath
    value: <REPLACE-WITH-TABLE> # Optional. default: ""
```

The above example uses secrets as plain strings. It is recommended to use a secret store for the secrets as described [here](../../concepts/secrets/README.md)

The following example uses the Kubernetes secret store to retrieve the acl token:

```
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <name>
spec:
  type: state.consul
  metadata:
  - name: datacenter
    value: <REPLACE-WITH-DATACENTER>
  - name: httpAddr
    value: <REPLACE-WITH-HTTP-ADDRESS>
  - name: aclToken
    secretKeyRef:
      name: <KUBERNETES-SECRET-NAME>
      key: <KUBERNETES-SECRET-KEY>
  ...
```

## Apply the configuration

### In Kubernetes

To apply the Consul state store to Kubernetes, use the `kubectl` CLI:

```
kubectl apply -f consul.yaml
```

### Running locally

The Dapr CLI will automatically create a directory named `components` in your current working directory with a Redis component.
To use Consul, replace the redis.yaml file with the consul.yaml above.
