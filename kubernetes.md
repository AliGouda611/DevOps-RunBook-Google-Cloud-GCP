## 2. Kubernetes Cluster

In this section, we'll walk through the process of setting up a Kubernetes cluster.

### 2.1 Setting Up Kubernetes Cluster

### 2.1.1 Pre-requisites

- Ensure you have the necessary permissions on the GCP project.
- Make sure you have the `gcloud` and `kubectl` command-line tools installed on your machine.

### 2.1.2 Cluster Overview 

- **Cluster Name**: `backend-cluster`
- **Region**: `europe-west2`
- **Project ID**: `sapient-mariner-398411`

The `backend-cluster` consists of a single node, which in turn hosts four pods, each corresponding to a different microservice. Each pod was named after its respective microservice to maintain a straightforward naming convention.

### 2.1.3 Steps

1. **Login to Google Cloud**:
    
    Use the following command to authenticate to your GCP account:

    ```bash
    gcloud auth login
    ```

2. **Set GCP Project**:
    
    Set your working project to `sapient-mariner-398411` using the following command:

    ```bash
    gcloud config set project sapient-mariner-398411
    ```

3. **Create the Kubernetes Cluster**:
    
    Create the `backend-cluster` in the `europe-west2` region using the following command:

    ```bash
    gcloud container clusters create backend-cluster --region=europe-west2
    ```

4. **Get Credentials for the Cluster**:
    
    After the cluster is created, retrieve the credentials to interact with the cluster using `kubectl`:

    ```bash
    gcloud container clusters get-credentials backend-cluster --region=europe-west2
    ```

5. **Verify Cluster Creation**:
    
    Use the following command to view the details of your clusters:

    ```bash
    kubectl get nodes
    ```

6. **Create the Pods**:

    The deployment of microservices was orchestrated through deployment files named following the pattern `<microservice-name>-deployment.yaml` (e.g., `backend-hotels-deployment.yaml`). These files outline the configurations necessary for each microservice, including but not limited to:

    - Docker image to be utilized.
    - Desired number of pod replicas.
    - Required environment variables and how they are sourced from Kubernetes Secrets.

    This is an example with explanation of the deployment file structure and components.

    ```yaml
    # This is the deployment configuration file for the 'backend' microservice.

    # Specifies the API version and the type of the resource.
    apiVersion: apps/v1
    kind: Deployment

    # Provides metadata about the deployment.
    metadata:
    name: backend  # Name of this deployment.

    # Describes the desired state of the deployment.
    spec:
    replicas: 1  # Specifies the number of pod replicas.

    # Helps the deployment identify which pods to manage based on labels.
    selector:
        matchLabels:
        app: backend  # Manages pods with the label 'app: backend'.

    # Defines the specifications for the pods.
    template:
        metadata:
        labels:
            app: backend  # Labels for this pod.

        # Describes the container specifications for the pod.
        spec:
        containers:
            - name: backend  # Name of the container.
            image: gcr.io/sapient-mariner-398411/backend  # Docker image to be used.
            ports:
                - containerPort: 3000  # Exposes port 3000 of the container.

            # Defines the environment variables for the container using Kubernetes secrets.
            env:
                - name: DB_USER  # Environment variable name.
                valueFrom:
                    secretKeyRef:
                    name: backend-secret  # Name of the Kubernetes secret.
                    key: DB_USER  # Key of the secret to use.

                - name: DB_PASSWORD
                valueFrom:
                    secretKeyRef:
                    name: backend-secret
                    key: DB_PASSWORD

                - name: DB_HOST
                valueFrom:
                    secretKeyRef:
                    name: backend-secret
                    key: DB_HOST

                - name: DB_PORT
                valueFrom:
                    secretKeyRef:
                    name: backend-secret
                    key: DB_PORT

                - name: DB_DATABASE
                valueFrom:
                    secretKeyRef:
                    name: backend-secret
                    key: DB_DATABASE
    ```


    Each microservice was deployed to the `backend-cluster` using the `kubectl apply` command, specifying the respective deployment file:

    ```bash
    kubectl apply -f <microservice-name>-deployment.yaml
    ```

    This command was executed for each microservice, triggering the creation and management of the respective pods within the `backend-cluster`.

    For example,

    ```bash
    kubectl apply -f backend-transportation-deployment.yaml
    ```

7. **Verify Pod Creation**:
    
    Verify the pods are running using the following command:

    ```bash
    kubectl get pods
    ```

---

### 2.2 Managing Secret Keys in Kubernetes


Managing sensitive information such as database credentials, API keys, etc., securely is crucial in a Kubernetes environment. Kubernetes `Secret` object is used to store and manage such sensitive information. 

### 2.2.1 Creating Secrets
Below is the structure and explanation of the `secrets.yaml` file used for managing the database credentials for our microservices.

```yaml
# This is the secret configuration file for the 'backend' microservice.

# Specifies the API version and the kind of resource.
apiVersion: v1
kind: Secret

# Provides metadata about the secret.
metadata:
  name: backend-secret  # Name of this secret.

# Specifies the type of the secret. 
# 'Opaque' is a generic type used for user-defined data.
type: Opaque

# Holds the actual secret data.
# Each key is a configuration name,
# and its value is the base64 encoding of the actual secret value.
data:
  DB_USER: db_user     # Base64 encoded username for the database.
  DB_PASSWORD: db_password   # Base64 encoded password for the database.
```

To encode your secrets in base64, you can use the following command in your terminal:

```bash
echo -n 'secret' | base64
```

The `secrets.yaml` file should be applied to the Kubernetes cluster using the command:

```bash
kubectl apply -f secrets.yaml
```

### 2.2.2 Referencing Secrets

```yaml
env:
- name: DB_USER  # Environment variable name.
    valueFrom:
    secretKeyRef:
        name: backend-secret  # Name of the Kubernetes secret.
        key: DB_USER  # Key of the secret to use.
```

Each `env` entry specifies an environment variable that should be provided to the container.

The `valueFrom` field specifies that the value should be taken from a Secret.

The `secretKeyRef` field references the Secret and the specific key within that Secret to use as the value for the environment variable.

---

### 2.3 Configuring Nginx Ingress

### 2.3.1 Kubernetes Services

Before diving into the configuration of the Nginx Ingress, it's essential to understand the concept of a `Service` in Kubernetes. A `Service` in Kubernetes is an abstraction which defines a logical set of Pods and a policy by which to access them. The set of Pods targeted by a Service is usually determined by a selector. Let's take a look at an example of a Service definition that targets a Pod with the `app: backend` label:

```yaml
# The Service manifest for the backend service
apiVersion: v1  # The version of the API to use
kind: Service  # What kind of object this is
metadata:
  name: backend-service  # The name of the service
spec:
  selector:
    app: backend  # This service will route traffic to pods with the label app: backend
  ports:
  - protocol: TCP  # The protocol for connections to the service ports
    port: 80  # The port that the service listens on
    targetPort: 3000  # The port on the pod that the service forwards requests to
  type: ClusterIP  # The type of service, in this case accessible within the cluster
```

In the above configuration, `backend-service` is a Service that routes traffic to Pods labeled `app: backend`. The Service forwards incoming requests on its port 80 to port 3000 on the Pods.

Now, onto configuring the Nginx Ingress. Ingress in Kubernetes is a collection of routing rules that govern how external HTTP/S traffic is processed and directed to the various Services within the cluster. In our setup, we utilize the Nginx Ingress Controller to handle these operations. Below is the structure and explanation of the `nginx-ingress.yaml` file used to configure the Ingress rules for our microservices.

### 2.3.2 Ingress Configuration

```yaml
# Specifies the API version and the kind of resource.
apiVersion: networking.k8s.io/v1
kind: Ingress

# Provides metadata about the ingress.
metadata:
  name: backend-ingress  # Name of this ingress.
  annotations:
    nginx.ingress.kubernetes.io/ssl-redirect: "false"  # Disables SSL redirect.
    nginx.ingress.kubernetes.io/use-regex: "true"  # Enables the use of regex in paths.
    nginx.ingress.kubernetes.io/rewrite-target: /$2  # Rewrites the URL path based on the regex captured value.

# Specifies the ingress class and the rules for routing.
spec:
  ingressClassName: nginx  # Specifies the ingress class to use.
  rules:
  - http:
      paths:
      # Defines a path for routing to the 'backend-service'.
      - path: /api(/|$)(.*)  # Matches '/api' and anything that follows.
        pathType: Prefix  # Specifies the match as a prefix.
        backend:
          service:
            name: backend-service  # Name of the backend service.
            port:
              number: 80  # Port number of the backend service.

      # Similar paths are defined for other backend services.
      - path: /api-hotels(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: backend-hotels-service
            port:
              number: 80

      - path: /api-payment(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: backend-payment-service
            port:
              number: 80

      - path: /api-transportation(/|$)(.*)
        pathType: Prefix
        backend:
          service:
            name: backend-transportation-service
            port:
              number: 80

      # A catch-all path routing to the 'backend-service'.
      - path: /(.*)
        pathType: Prefix
        backend:
          service:
            name: backend-service
            port:
              number: 80
```

Apply this configuration to the Kubernetes cluster using the following command:

```bash
kubectl apply -f nginx-ingress.yaml
```

In this configuration, HTTP requests are routed to different backend services based on the URL path. The use of regex in the path along with the `rewrite-target` annotation ensures that the path in the request URL is correctly rewritten to match the expectations of the backend services.

--- 

### 2.4 Configuring Network Policy

Network Policies are crucial for controlling the flow of traffic between pods/containers in a Kubernetes cluster. In this section, we will discuss two types of network policies: one for managing public traffic and the other for internal communication between pods.

### 2.4.1 Public Traffic Policy

The following network policy named `allow-public-traffic-backend` is designed to accept connections from the ingress controller to the `backend` pod:

```yaml
# This file is saved as allow-public-traffic-backend.yaml
apiVersion: networking.k8s.io/v1  # API version
kind: NetworkPolicy  # Kind of Kubernetes resource
metadata:
  name: allow-public-traffic-backend  # Name of this network policy
spec:
  podSelector:
    matchLabels:
      app: backend  # Selects the backend pod
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: "ingress-nginx"  # Allows traffic from the ingress-nginx namespace
    ports:
      - protocol: TCP  # Protocol type
        port: 3000  # Port number
```

Apply this network policy using the command:

```bash
kubectl apply -f allow-public-traffic-backend.yaml
```

In this configuration, the `allow-public-traffic-backend` NetworkPolicy permits traffic to the `backend` pod on TCP port 3000 from any pod in the `ingress-nginx` namespace.

### 2.4.2 Internal Communication Policy

The `backend-policy` network policy outlined below is aimed at controlling the communication between the `backend` pod and other internal pods (`backend-hotels`, `backend-payment`):

```yaml
# This file is saved as backend-policy.yaml
apiVersion: networking.k8s.io/v1  # API version
kind: NetworkPolicy  # Kind of Kubernetes resource
metadata:
  name: backend-policy  # Name of this network policy
spec:
  podSelector:
    matchLabels:
      app: backend  # Selects the backend pod
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: backend-hotels  # Allows traffic from backend-hotels pod
  - from:
    - podSelector:
        matchLabels:
          app: backend-payment  # Allows traffic from backend-payment pod
```

Apply this network policy using the command:

```bash
kubectl apply -f backend-policy.yaml
```

In this setup, the `backend-policy` NetworkPolicy allows traffic to the `backend` pod from the `backend-hotels` and `backend-payment` pods, thus ensuring controlled communication between these specific internal microservices.

All the configurations described above should be replicated and appropriately modified for other microservices as well to ensure a standardized setup across the cluster. This includes setting up corresponding network policies, and deployment configurations for `backend-transportation`, `backend-hotels`, and `backend-payment` microservices. 
