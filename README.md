# Grey Matter Operator

Grey Matter Operator is a Kubernetes operator that watches for greymatter.io/v1.Mesh CR (Custom Resource) objects in a Kubernetes cluster and installs the core Grey Matter services into the same namespace where the Mesh CR exists. Grey Matter Operator also spawns a process that will call the Control API to configure routing for each installed core service.

## Quickstart

This assumes you have at least Go 1.15, K3d, and kubectl installed.

1. Download Go dependencies: `go mod vendor`
2. Build the Docker image: `make docker-build`
3. Make a K3d cluster: `make k3d`
4. Import the Docker image into K3d: `make k3d-import`

Prior to deploying, store your Grey Matter LDAP credentials in the environment variables `NEXUS_USER` and `NEXUS_PASSWORD`. These will be used to create a secret for pulling Docker images.

To deploy:

1. Deploy to the K3d cluster: `make deploy`.
2. Create a Mesh CR: `make sample`
3. Check the Mesh CR: `kubectl get mesh sample-mesh -o yaml`

The mesh will then be available via `localhost:30000`.

To clean up:

1. `make remove-sample`
2. `make undeploy`
3. `make destroy`

## Development

### Scaffolding

Grey Matter Operator makes use of a tool called [controller-gen](https://github.com/kubernetes-sigs/controller-tools) for scaffolding the Mesh CRD (Custom Resource Definition) in code as well as the code that registers it. It is used in this project via two commands:
- `make generate`: Generates code used to implement a Kubernetes object. This enables Mesh CR objects to be managed entities in the Kubernetes API.
- `make manifests`: Generates configuration used to deploy Grey Matter Operator and the Mesh CRD into a Kubernetes cluster.
  
*NOTE: As this project matures, `make manifests` should be replaced by a versioned CLI that programmatically applies all of the configurations to the Kubernetes cluster.*

### Important Files

- [api/v1/mesh_types.go](api/v1/mesh_types.go): Used to generate the Mesh CRD. Every time this is updated, run `make generate` and `make manifests`.
- [config/crd/bases/greymatter.io_meshes.yaml](config/crd/bases/greymatter.io_meshes.yaml): The Mesh CRD generated by `make manifests`.
- [controllers/mesh_controller.go](controllers/mesh_controller.go): Logic for what to do when a Mesh CR is created/updated (i.e. deploy and configure the mesh).
- [controllers/reconciler.go](controllers/reconciler.go): A custom interface with methods for reconciling Kubernetes resources based on configuration passed in from a Mesh CR. See the `reconcilers` directory for how this interface is implemented for each Kubernetes resource.
- [controllers/gmcore/gmcore.go](controllers/gmcore/gmcore.go): This package defines configuration for each version of Grey Matter. At runtime, configurations will be applied starting with `base.go` and then continuing with either `version_1_3.go` or `version_1_6_beta.go` as patches. `base.go` contains configuration that should apply to all versions, while the others allow for specifying version-specific configurations.
- [controllers/meshobjects/objects.go](controllers/meshobjects/objects.go): This package provides a client with methods for scaffolding mesh objects (i.e. Cluster, Listener, Domain, Proxy, etc.). It is based loosely on this [gm-catalog internal package](https://github.com/greymatter-io/gm-catalog/tree/main/internal/meshobjects).

## Resources

- [Kubebuilder](https://book.kubebuilder.io/introduction.html)
- [Operator Framework: Go Operator Tutorial](https://sdk.operatorframework.io/docs/building-operators/golang/tutorial/)
- [Operator SDK Installation](https://sdk.operatorframework.io/docs/building-operators/golang/installation/)
- [Operator Manager Overview](https://book.kubebuilder.io/cronjob-tutorial/empty-main.html)
- [Istio's operator spec](https://github.com/istio/api/blob/master/operator/v1alpha1/operator.pb.go#L97)
