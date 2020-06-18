# Contribute to Keptn Sandbox

In Keptn Sandbox we keep a collection of tools, services and integrations that extend Keptn. 
If you want to contribute, please read the following guidelines.


## How to contribute new content?

1. Make sure your repository has the following files

    - README.md 
      - with description of project
      - how to use it
      - how to install/uninstall
      - compatibility matrix for Keptn version compatibility
    - LICENSE
    - CODEOWNERS (with your git account)
    
1. Create an issue in this repo to inform us about your repo
1. The team will reach out to you to start the transfer process
1. We will transfer the project from your account to the keptn-sandbox organization. All issues and history will be kept.
1. You will be added as maintainer to the repo with full write access.
1. We will support you in setting up the Travis build for your repo.
1. Present your service in a Keptn Community meeting to have your service listed on [Keptn.sh](https://keptn.sh) 

## How to contribute to existing repositories?

For existing repositories in the Keptn Sandbox Organization the respective code owners (see CODEOWNERS file of each repo) are mainly responsible. To contribute, please open a PR in the repo and ask the code owners for approval.

## How to be promoted to Keptn Contrib organization?

The [Keptn Contrib](https://github.com/keptn-contrib) holds projects that have been promoted from Keptn Sandbox due to their adoption and high quality standards.
To suggest your contribution to the Keptn Contrib organization, it has to fulfull all criteria listed above, plus the following:
1. Automated builds
1. Automated tests that are validated for each build
1. At least one release of the service compatible with the latest Keptn version
1. At least one sponsor of the Keptn core team for your contribution

## RBAC Guidelines
* Use the Service Account `keptn-default` of the keptn installation namespace wherever possible. This account has no k8s permissions assigned.
* Never apply additional Roles or Rolebindings on the Service Account `keptn-default`
* If your service needs permissions on k8s, create a new service account with the name `keptn-<your-service-name>` in the keptn installation namespace
* Apply Roles, ClusterRoles, Rolebindings and ClusterRoleBindings with a minimum set of useful permissions on that service account
* Prefer namespaced Roles and Rolebindings over ClusterRoles and ClusterRoleBindings

### Example 1: Keptn Service does not require any permissions:
Let's imagine the Keptn service (which corresponds to a Kubernetes deployment and service) named `keptn-sample-service` requires
no permissions (i.e. does not have to access the Kubernetes API at all).
Therefore, the Service Account `keptn-default` can be used in the Deployment.

**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: keptn-sample-service
  name: keptn-sample-service
  namespace: keptn
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keptn-sample-service
  template:
    metadata:
      labels:
        app: keptn-sample-service
    spec:
      containers:
      - image: keptn/keptn-sample-service
        name: keptn-sample-service
  serviceAccountName: keptn-default
```

### Example 2: Keptn Service requries read access for a secret
Let's imagine the Keptn service (which corresponds to a Kubernetes deployment and service) named `keptn-sample-service` requires read-access for a secret called `keptn-sample-secret` in the keptn namespace.
Therefore, please create the following Service Account, Role and RoleBinding. Finally, use this Service Account in the Deployment.

**Service Account:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: keptn-sample-service
  namespace: keptn
  labels:
    app: keptn
``` 
**Role:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: keptn-sample-service-read-secret
  namespace: keptn
  labels:
    app: keptn
rules:
  - apiGroups:
      - ""
    resources:
      - secrets
    verbs:
      - get
    resourceNames:
      - "keptn-sample-secret"
```
**RoleBinding:**
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: keptn-sample-service-read-secret
  namespace: keptn
  labels:
    app: keptn
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: keptn-sample-service-read-secret
subjects:
  - kind: ServiceAccount
    name: keptn-sample-service
    namespace: keptn
```
**Deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: keptn-sample-service
  name: keptn-sample-service
  namespace: keptn
spec:
  replicas: 1
  selector:
    matchLabels:
      app: keptn-sample-service
  template:
    metadata:
      labels:
        app: keptn-sample-service
    spec:
      containers:
      - image: keptn/keptn-sample-service
        name: keptn-sample-service
  serviceAccountName: keptn-sample-service
```
**Validation:**
To check if your RBAC rules are working as intended, use 
```kubectl auth can-i get secret/keptn-sample-secret --as system:serviceaccount:keptn:keptn-sample-service```
