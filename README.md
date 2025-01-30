
# Kyverno
![image](https://github.com/user-attachments/assets/5b9aefb4-ae67-4db9-a800-8dfeb7ddc8e9)

Kyverno (Greek for “govern”) is a cloud native policy engine. It allows platform engineers to automate security, compliance, and best practices validation and deliver secure self-service to application teams. The Kyverno project provides a comprehensive set of tools to manage the complete Policy-as-Code (PaC) lifecycle for Kubernetes and other cloud native environments. It is a policy engine for Kubernetes that can be used to define, validate, and enforce a wide range of security and operational best practices within Kubernetes resources. It ensures compliance with security policies, operational practices, and Kubernetes best practices, especially in complex, large-scale environments where manual enforcement would be impractical.


## Key Features:

- **Policy Enforcement**: Kyverno allows users to create policies to enforce security best practices, such as ensuring that only approved images are used in container deployments or that certain labels are applied to resources.

- **Easy Policy Definition**: Policies in Kyverno are written in YAML, making them easy to define and integrate into existing Kubernetes workflows.

- **Mutating and Validating Policies**: Kyverno can enforce policies by both mutating (modifying resources) and validating (ensuring resources meet specific criteria) Kubernetes objects.

- **Admission Controllers**: Kyverno interacts with the Kubernetes admission controllers to enforce policies before resources are admitted to the cluster.

- **Policy Auditing**: Kyverno supports auditing functionality, allowing users to check resources against policies without automatically enforcing them, which helps in assessing the current state of a cluster.

- **Native Kubernetes Integration**: Kyverno is natively integrated with Kubernetes, utilizing the Kubernetes API server to operate, making it a seamless addition to a Kubernetes-based environment.


## Kyverno Architecture:

![image](https://github.com/user-attachments/assets/f88d8f60-3878-4e17-9feb-062a648999ba)

## Kyverno Workflow

**API Request**: An API request enters the Kubernetes system.

**Authentication & Authorization**: The request is authenticated and authorized.

**Mutating Admission**: Kyverno can mutate the resource at this stage based on predefined policies.

**Schema Validation**: The resource is validated against the Kubernetes schema.

**Validating Admission**: Kyverno can validate the resource to ensure it meets policy requirements.

**Admission Review**: An admission controller reviews the resource, applying any final policies before the resource is admitted to the cluster. Kyverno effectively validate, mutate, and generate resources, ensuring that all admitted resources comply with organizational policies.
## Install Kyverno:

Kyverno provides multiple methods for installation: Helm and YAML manifest. When installing in a production environment, Helm is the recommended and most flexible method as it offers convenient configuration options to satisfy a wide range of customizations. Regardless of the method, Kyverno must always be installed in a dedicated Namespace; it must not be co-located with other applications in existing Namespaces including system namespaces such as kube-system. The Kyverno Namespace should also not be used for deployment of other, unrelated applications and services. Below is the compatibility matrix and steps to install Keyverno in kubernetes cluster.

## Compatibility Matrix:

| Kyverno Version             | Kubernetes Min                    | Kubernetes Max                    |
|-----------------------------|-----------------------------------|----------------------------------|
| 1.11.x             		  | 1.25       						  | 1.28							 |
| 1.12.x                	  | 1.26                   			  | 1.29                 			 |
| 1.13.x     				  | 1.28      						  | 1.31     						 |

•	Helm client installed - [Install Here](https://helm.sh/docs/intro/install/)

•	Kubectl installed – [Install Here](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html#kubectl-install-update)

•	helm repo add kyverno https://kyverno.github.io/kyverno/ 

•	helm repo update

•	helm install kyverno kyverno/kyverno -n kyverno --create-namespace

•	helm install kyverno-policies kyverno/kyverno-policies -n kyverno (optional)

•	kubectl get clusterpolicy


## Use Cases for Kyverno:

 ### 1. Validation:
Validation is Kyverno’s most straightforward use case. It involves categorizing resources into two outcomes: compliant or non-compliant. Compliant resources proceed to deployment, while non-compliant resources are blocked.

### 2. Mutation
Mutation allows Kyverno to dynamically alter resources before they are admitted to the cluster. This feature is particularly useful for automatically adding or modifying labels, annotations, or other configurations.

### 3. Generation
The generation feature in Kyverno is powerful, allowing users to create new Kubernetes resources dynamically. This can be particularly useful for tasks like cloning secrets across namespaces.
## Steps to Perform POC
### 1 Validation: Enforcing Compliance
Validation is one of Kyverno’s core features, allowing you to enforce policies that define which resources are allowed in your cluster. It operates as a binary decision mechanism either a resource complies and is allowed, or it doesn’t and is rejected. validation policies can help enforce Kubernetes Pod Security Standards, ensuring compliance and security.

**Validation Policy**:

validation_policy.yaml
   ```bash
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-labels
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-team
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "label 'team' is required"
      pattern:
        metadata:
          labels:
            team: "?*"
   ```

**Apply the Policy**:
   ```bash
   kubectl apply -f validation_policy.yaml
   ```

**Check Available Cluster Policies**:
   ```bash
   kubectl get clusterpolicy
   ```
   You should see the require-labels policy listed along with others if you installed additional Kyverno policies.

**Create a Test Namespace**:
   ```bash
   kubectl create ns kyverno-policy-tests
   ```
Test the policy try deploying a pod without the required team label to see the validation in action

no_label_deploy.yaml
 ```bash
# create an nginx deployment with no labels
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: kyverno-policy-tests
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
          name: nginx
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: "0.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
   ```

```bash
   kubectl apply -f no_label_deploy.yaml
   ```
You will see an error message indicating that the deployment was blocked because it lacked the required team label.

Fix the deployment add the required label and try again.

proper_label_deploy.yaml

```bash
# create an nginx deployment with no labels
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: kyverno-policy-tests
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        team: "platform_engineering" # notice the right label here has team
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
          name: nginx
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: "0.5"
            memory: "256Mi"
          requests:
            cpu: "0.1"
            memory: "128Mi"
   ```
Deploy it

```bash
   kubectl apply -f proper_label_deploy.yaml
   ```
This time the deployment will succeed.

Check the Policy Report
```bash
   kubectl get policyreport
   kubectl describe policyreport <report-name>
   ```
You will see a report indicating that the policy was enforced, and the Pod passed the validation.

### 2 Mutation: Dynamic Resource Modification
Mutation allows Kyverno to dynamically modify resources before being admitted to the cluster. This is useful for scenarios where you want to ensure specific configurations, such as adding labels or annotations. Kyverno annotations can be used to provide descriptive metadata for policies, enhancing their effectiveness.

**Mutation Policy**:

mutation_policy.yaml
```bash
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-labels
spec:
  rules:
  - name: add-team
    match:
      any:
      - resources:
          kinds:
          - Pod
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            +(team): bravo
   ```

Apply the Mutation Policy
```bash
kubectl apply -f mutation_policy.yaml
   ```
Deploy a Pod Without a Team Label

redis.yaml

```bash
apiVersion: v1
kind: Pod
metadata:
  name: redis-pod
spec:
  containers:
  - name: redis
    image: redis:latest
    ports:
    - containerPort: 6379
   ```

Apply it
```bash
kubectl apply -f redis.yaml
   ```
Check the Pod for the Mutated Label
```bash
kubectl get pod redis-pod --show-labels
   ```
The output will show that Kyverno has added the team=bravo label.

Test with an Existing Label: Deploy another Pod with an existing team label to see that Kyverno does not override it

```bash
kubectl run newredis --image redis -l team=alpha
   ```
Check the labels

```bash
kubectl get pod newredis --show-labels
   ```
The team=alpha label should remain unchanged, demonstrating that Kyverno only adds the label if it’s missing

### 3.Generation: Automating Resource Creation
Generation is one of Kyverno’s most powerful features, enabling the creation of new resources automatically. This is particularly useful for tasks like synchronizing secrets across namespaces. Consider a scenario where you want to ensure that a Docker registry secret is available in every namespace. Kyverno can automate this process by generating the secret in new namespaces based on a template from an existing namespace.

registry_secret_policy.yaml

```bash
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: sync-secrets
spec:
  rules:
  - name: sync-image-pull-secret
    match:
      any:
      - resources:
          kinds:
          - Namespace
    generate:
      apiVersion: v1
      kind: Secret
      name: regcred
      namespace: "{{request.object.metadata.name}}"
      synchronize: true
      clone:
        namespace: kyverno-policy-tests
        name: regcred
   ```

Create a Docker Registry Secret

registry_secret.yaml

```bash
apiVersion: v1
kind: Secret
metadata:
  name: regcred
  namespace: kyverno-policy-tests
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJteWludGVybmFscmVnLmNvcnAuY29tIjp7InVzZXJuYW1lIjoiam9obi5kb2UiLCJwYXNzd29yZCI6IlBhc3N3MHJkMTIzISIsImVtYWlsIjoiam9obi5kb2VAY29ycC5jb20iLCJhdXRoIjoiYW05b2JpNWtiMlU2VUdGemN6QjNjbVF4TWpNaCJ9fX0=
   ```

Apply the Generation Policy
```bash
kubectl apply -f registry_secret_policy.yaml
   ```

Create a New Namespace
```bash
kubectl create ns mytestns
   ```

Check the Secret in the New Namespace
```bash
kubectl -n mytestns get secret
   ```

You will see the regcred secret, which Kyverno automatically generated based on the source secret in the kyverno-policy-tests namespace. Kyverno will also keep this secret synchronized across all namespaces, ensuring consistency. Kyverno can be used to generate resources across various Kubernetes environments, ensuring consistency and security.

### Cleanup: Removing Kyverno Resources

**Clean-Up Commands**

Delete the Test Namespaces
```bash
kubectl delete ns kyverno-policy-tests
kubectl delete ns mytestns
   ```

Delete the Policies
```bash
kubectl delete -f registry_secret_policy.yaml 
kubectl delete -f mutation_policy.yaml 
kubectl delete -f validation_policy.yaml
   ```












## Kyverno-Pricing:

| Plan			              | Price		                      | Features	                     |
|-----------------------------|-----------------------------------|----------------------------------|
| Open-Source           	  | $0       						  | All Product Features			 |
|							  |									  |	Community support				 |
| Enterprise               	  | Custom                   		  | Everything in Open-Source plus   |
|      				  		  |       						  	  | 24×7 Enterprise Support    		 |
|							  |									  | Prioritized feature requests	 |
|							  |									  | Curated Policy Sets				 |
|							  |									  | CLI for Git Repo Scanning		 |
|							  |									  | Adapters for Enterprise Integrations|
|							  |									  | Training and services available	 |

## Additional Resources:

Kyverno Documentation – [Click Here](https://kyverno.io/docs/)

Helm Chart for Kyverno – [Click Here](https://kyverno.github.io/kyverno/)
