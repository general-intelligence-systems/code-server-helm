# code-server Helm Chart

A Helm chart for deploying [coder/code-server](https://github.com/coder/code-server) on Kubernetes.

This chart is automatically synced from the upstream [coder/code-server](https://github.com/coder/code-server/tree/main/ci/helm-chart) Helm chart on a weekly basis.

## Usage

```bash
helm repo add code-server https://general-intelligence-systems.github.io/code-server-helm
helm repo update
helm install code-server code-server/code-server
```

## Configuration

| Parameter | Description | Default |
|---|---|---|
| `replicaCount` | Number of replicas | `1` |
| `image.repository` | Container image repository | `codercom/code-server` |
| `image.tag` | Container image tag | `4.109.5` |
| `image.pullPolicy` | Image pull policy | `Always` |
| `imagePullSecrets` | Image pull secrets | `[]` |
| `nameOverride` | Override chart name | `""` |
| `fullnameOverride` | Override full release name | `""` |
| `hostnameOverride` | Override pod hostname | `""` |
| `existingSecret` | Existing secret for password (key: `password`) | `""` |
| `password` | code-server password (auto-generated if unset) | `""` |
| `serviceAccount.create` | Create a service account | `true` |
| `serviceAccount.annotations` | Service account annotations | `{}` |
| `serviceAccount.name` | Service account name | `""` |
| `annotations` | Deployment annotations | `{}` |
| `podAnnotations` | Pod annotations | `{}` |
| `podSecurityContext` | Pod security context | `{}` |
| `priorityClassName` | Pod priority class | `""` |
| `service.type` | Service type | `ClusterIP` |
| `service.port` | Service port | `8080` |
| `ingress.enabled` | Enable ingress | `false` |
| `ingress.ingressClassName` | Ingress class name | `""` |
| `ingress.annotations` | Ingress annotations | `{}` |
| `ingress.hosts` | Ingress hosts | `[]` |
| `ingress.tls` | Ingress TLS configuration | `[]` |
| `extraArgs` | Additional code-server CLI arguments | `[]` |
| `extraVars` | Additional environment variables | `[]` |
| `volumePermissions.enabled` | Init container to fix volume permissions | `true` |
| `volumePermissions.securityContext.runAsUser` | Volume permissions init container user | `0` |
| `securityContext.enabled` | Enable pod security context | `true` |
| `securityContext.fsGroup` | Pod filesystem group | `1000` |
| `securityContext.runAsUser` | Pod run-as user | `1000` |
| `resources` | CPU/memory resource requests/limits | `{}` |
| `livenessProbe.enabled` | Enable liveness probe | `true` |
| `readinessProbe.enabled` | Enable readiness probe | `true` |
| `nodeSelector` | Node selector | `{}` |
| `tolerations` | Tolerations | `[]` |
| `affinity` | Affinity rules | `{}` |
| `persistence.enabled` | Enable persistent storage | `true` |
| `persistence.storageClass` | PVC storage class | `""` |
| `persistence.accessMode` | PVC access mode | `ReadWriteOnce` |
| `persistence.size` | PVC size | `10Gi` |
| `persistence.annotations` | PVC annotations | `{}` |
| `persistence.existingClaim` | Use an existing PVC | `""` |
| `persistence.hostPath` | Use a host path instead of PVC | `""` |
| `lifecycle.enabled` | Enable lifecycle hooks | `false` |
| `lifecycle.postStart` | Post-start lifecycle hook | `{}` |
| `lifecycle.preStop` | Pre-stop lifecycle hook | `{}` |
| `extraContainers` | Additional sidecar containers (templated string) | `""` |
| `extraInitContainers` | Additional init containers (templated string) | `""` |
| `extraSecretMounts` | Additional secret volume mounts | `[]` |
| `extraVolumeMounts` | Additional volume mounts | `[]` |
| `extraConfigmapMounts` | Additional configmap volume mounts | `[]` |
| `extraPorts` | Additional container/service ports | `[]` |

## Examples

### Ingress with TLS

```yaml
ingress:
  enabled: true
  ingressClassName: nginx
  annotations:
    kubernetes.io/tls-acme: "true"
  hosts:
    - host: code.example.com
      paths:
        - /
  tls:
    - secretName: code-server-tls
      hosts:
        - code.example.com
```

### Docker-in-Docker sidecar

```yaml
extraVars:
  - name: DOCKER_HOST
    value: "tcp://localhost:2376"

extraContainers: |
  - name: docker-dind
    image: docker:28.3.2-dind
    imagePullPolicy: IfNotPresent
    securityContext:
      privileged: true
    env:
      - name: DOCKER_TLS_CERTDIR
        value: ""
    command:
      - dockerd
      - --host=unix:///var/run/docker.sock
      - --host=tcp://0.0.0.0:2376
```

### Pre-install extensions

```yaml
extraInitContainers: |
  - name: install-extensions
    image: codercom/code-server:4.109.5
    imagePullPolicy: IfNotPresent
    command:
      - sh
      - -c
      - |
        code-server --install-extension ms-python.python
        code-server --install-extension golang.Go
    volumeMounts:
      - name: data
        mountPath: /home/coder
```

### Existing secret for password

```yaml
existingSecret: my-code-server-secret
```

The secret must contain a `password` key:

```bash
kubectl create secret generic my-code-server-secret --from-literal=password=mysecurepassword
```

## Upstream Sync

This chart is automatically synced from the upstream [coder/code-server](https://github.com/coder/code-server) repository weekly. Local overrides in `overrides/code-server/` are applied on top of the upstream chart after each sync.
