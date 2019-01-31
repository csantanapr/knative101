## Install Istio, Knative, and Kaniko Build Template

Knative is currently built on top of both Kubernetes and Istio. You will need to install Istio to install Knative. It's not required for this lab, but you can learn more about Istio by completing the [Istio 101 lab](https://github.com/IBM/istio101/tree/master/workshop).

### Install Istio

1. A Custom Resource Definition enables you to create custom resources, extensions to the Kubernetes API on your Kubernetes cluster. Istio needs these CRDs to be created before we can install.  Install Istio’s CRDs via kubectl apply, and wait a few seconds for the CRDs to be committed in the kube-apiserver.

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/istio-crds.yaml
	```

2. Install Istio:

	```
	kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/istio.yaml
	```
3. Label the default namespace with `istio-injection=enabled`:

	```
	kubectl label namespace default istio-injection=enabled
	```

4. Monitor the Istio components until all of the components show a `STATUS` of
    `Running` or `Completed`:

    ```
    kubectl get pods --namespace istio-system --watch
    ```

### Install Knative

After installing Istio, you can install Knative. For this lab, we will install all available Knative components.

1. Install Knative Serving:

    ```
    kubectl apply --filename https://github.com/knative/serving/releases/download/v0.3.0/serving.yaml
    ```

    If you get an error that starts with `unable to recognize` run the command a second time as a workaround.
    FIXME: Remove workaround when fixed upstream

2. Install Knative Monitoring (Optional):

    There is a problem with the monitoring.yaml file see issue [knative/serving#3031](https://github.com/knative/serving/pull/3031/files).
    You need to download the file and edit it as a workaround.
    ```
    curl -OL https://github.com/knative/serving/releases/download/v0.3.0/monitoring.yaml
    ```
    Now edit and remove the extra slash at the end of `prometheus/` in lines:
    ```
    mountPath: /etc/prometheus
    ```
    and
    ```
    mountPath: /prometheus
    ```
    FIXME: Remove workaround when fixed upstream

    Then deploy using `kubectl`
    ```
    kubectl apply --filename monitoring.yaml
    ```

3. Install Knative building:

    ```
    kubectl apply --filename https://github.com/knative/build/releases/download/v0.3.0/release.yaml
    ```

4. Monitor the Knative components until all of the components show a `STATUS` of `Running` or `Completed`:

	Commands:
    ```
    kubectl get pods --namespace knative-serving
    kubectl get pods --namespace knative-monitoring
    kubectl get pods --namespace knative-build
    ```
    Example Ouput:

    ```
    NAME                          READY   STATUS    RESTARTS   AGE
    activator-5c4755585c-26bck    2/2     Running   0          35m
    autoscaler-78cd88f869-lhncn   2/2     Running   0          35m
    controller-8d5b85958-7qbr4    1/1     Running   0          35m
    webhook-7585d7488c-fdp2b      1/1     Running   0          35m
    ```

    ```
    NAME                                  READY   STATUS    RESTARTS   AGE
    elasticsearch-logging-0               1/1     Running   0          28m
    elasticsearch-logging-1               1/1     Running   0          27m
    grafana-6df4646f87-g7x9j              1/1     Running   0          27m
    kibana-logging-5d8bdbdbf7-8tb52       1/1     Running   0          28m
    kube-state-metrics-7466cdbd57-tfzx7   4/4     Running   0          27m
    node-exporter-7bnhx                   2/2     Running   0          27m
    node-exporter-tzfcl                   2/2     Running   0          27m
    node-exporter-wzn8h                   2/2     Running   0          27m
    prometheus-system-0                   1/1     Running   0          6m
    prometheus-system-1                   1/1     Running   0          6m9s
    ```

    ```
    NAME                                READY   STATUS    RESTARTS   AGE
    build-controller-864c5d4c55-kkdmb   1/1     Running   0          88s
    build-webhook-7ccd8fb76-65bm2       1/1     Running   0          88s
    ```

### Install Kaniko Build Template

As a part of this lab, we will use the [kaniko](https://github.com/GoogleContainerTools/kaniko) build template for building source into a container image from a Dockerfile. Kaniko doesn't depend on a Docker engine and instead executes each command within the Dockerfile completely in userspace. This enables building container images in environments that can't easily or securely run a Docker engine, such as Kubernetes.


1. Install the Knative build template for kaniko.

    ```
    kubectl apply --filename https://raw.githubusercontent.com/knative/build-templates/master/kaniko/kaniko.yaml
    ```

2. Use kubectl to confirm you installed the kaniko build template, as well as to see some more details about it.  You'll see that this build template accepts parameters of `IMAGE` and `DOCKERFILE`.  `IMAGE` is the name of the image you will push to the container registry, and `DOCKERFILE` is the path to the Dockerfile that will be built.

	Command:
	```
	kubectl get buildtemplate kaniko -o yaml
	```

	Example Output:
	```yaml
    spec:
    generation: 1
    parameters:
    - description: The name of the image to push
        name: IMAGE
    - default: /workspace/Dockerfile
        description: Path to the Dockerfile to build.
        name: DOCKERFILE
    steps:
    - args:
        - --dockerfile=${DOCKERFILE}
        - --destination=${IMAGE}
        env:
        - name: DOCKER_CONFIG
        value: /builder/home/.docker
        image: gcr.io/kaniko-project/executor
        name: build-and-push
    ```


Continue on to [exercise 2](../exercise-2/README.md).
