# About

This is a guide-line which has exercises to create basic application that needs almost all basic K8s/OS objects: Deployment, Service, Route, RBAC, PVC.

All basic objects diagram can be found at https://docs.google.com/drawings/d/1lcqlBNV4M0OV5w0Ovlq7AsqmxxWJplISiyx1-_TO-x8/edit?usp=sharing

# Step 1: Create a deployment

### Execution

There is an container which provides an ability to get Web Terminal inside sidecar.
So, input:
```yaml
image: quay.io/eclipse/che-machine-exec:nightly
```

required env var:
```yaml
env:
- name: POD_SELECTOR
  value: app=cloud-shell
```

exposed port:
```
      - containerPort: 4444
        name: cloud-shell
        protocol: TCP
```

### Verification step

Make sure deployment created the desired pod and container is successfully started, check container logs.

# Step 2: Expose https endpoint

### Execution

Cloud Shell listen to 4444, so it's needed to expose service and route for it.

### Verification step

Make sure you're able to cloud shell app with host from the created Route.
But the error will be displayed since application does not have needed permissions on K8s cluster.

# Step 3: Add ServiceAccount

### Execution

Our application needs an ability to list pods and create exec into them.
So, it's needed to create ServiceAccount, Role and RoleBinding that would grant application the following permissions:
```yaml
rules:
- apiGroups:
  - ""
  resources:
    - pods/exec
  verbs:
    - create
- apiGroups:
    - ''
  resources:
    - pods
  verbs:
    - list
```
Note: Don't forget to add serviceAccountName into existing deployment.

### Verification step

Open the route host and check that now error is different:
```
Unable to connect to json-rpc channel. Cause: Error: failed to initialize terminal in any of {cloud-shell-584f496f9-rwf8d\cloud-shell} -- errors:
- cloud-shell: command terminated with exit code 1
```
It's caused by fact that no target container exist for terminal.

# Step 4: Add a sidecar for terminal target

### Execution

To get the shell ready-to-use, we need to add one more container into existing deployment
image:
```yaml
    image: quay.io/wto/web-terminal-tooling:1.2
```

Since by default, this container main process ends with 0, we need to override entrypoint entrypoint:
```yaml
    command: ["tail"]
    args:
    - "-f"
    - "/dev/null"
```

### Verification step

Open the route URL and make sure that terminal is initialized.

# Step 5: Add an storage
TODO

# Step 6: Add an configmap/secret
TODO

# Events
