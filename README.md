# About

This is a guide-line which has exercises to create basic application that needs almost all basic K8s/OS objects: Deployment, Service, Route, RBAC, PVC.

All basic objects diagram can be found at https://docs.google.com/drawings/d/1lcqlBNV4M0OV5w0Ovlq7AsqmxxWJplISiyx1-_TO-x8/edit?usp=sharing

# Step 1: Create a deployment

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

# Step 2: Expose https endpoint

Cloud Shell listen to 4444, so it's needed to expose service and route for it.

# Step 3: Add ServiceAccount

TODO

# Step 4: Add a sidecar for terminal target
image:
```yaml
    image: quay.io/wto/web-terminal-tooling:1.2
```
entrypoint:
```yaml
    command: ["tail"]
    args:
    - "-f"
    - "/dev/null"
```

# Step 5: Add an storage
TODO

# Step 6: Add an configmap/secret
TODO

# Events
