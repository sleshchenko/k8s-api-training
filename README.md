# About

This is a guide-line which has exercises to create basic application that needs almost all basic K8s/OS objects: Deployment, Service, Route, RBAC, PVC.

You're supposed to craft objects manually using skeletons from the corresponding folder in repo.

All basic objects diagram can be found at https://docs.google.com/drawings/d/1cXmbsN05FmS12Hd83RvOGGnSSYqBQDPNpo0qXgpcnV8/edit?usp=sharing

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
```yaml
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

Usually, user needs to modify their shell with ~/.bashrc, but since container does not have storage yet, it'll be erased after container restart. So, this step about adding PVC into container.

### Execution

Create PVC, like `home-storage` and mount it to existing deployment, into terminal container to `/home/user`.

### Verification step

Open a terminal and execute:
```
echo "export TEST_ENV=blah-blah" >> /home/user/.bashrc
```
Restart a deployment (scale to 0, scale to 1), wait a new pod is ready and test

```
source /home/user/.bashrc && echo $TEST_ENV
```
output should be blah-blah

# Step 6: Add an configmap/secret

If you execute `oc whoami` you'll see that it's SA token that is used but as user, I want my identity to be used inside my terminal. So, add a secret with your token and mount it inside container as `/var/run/secrets/kubernetes.io/serviceaccount/token`

### Execution

Go to OpenShift console and get your token from there (right top corner, click your username, copy login command). Extract only value of token, like sha256~.....

### Execution

Create a secret with token item and copied token as value.
Since value should be base64 encoded, it's easier to use OpenShift Console to create it.

Then mount it into terminal container into /tmp/token

### Verification step
execute
```bash
oc login --token=$(cat /tmp/token/token) https://${KUBERNETES_SERVICE_HOST}:${KUBERNETES_SERVICE_PORT}
```
and then

```bash
oc whoami
```
output should be your username.

Congrats!!! Your set up manually an application for yourself that provides you terminal ready to work with your K8s cluster. It's not really production ready, because of authentication leaks, but still =)
