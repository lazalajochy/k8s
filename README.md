# k8s


# Guía Completa de Kubernetes (K8s)

## Tabla de Contenidos
1. [Arquitectura de Kubernetes](#arquitectura)
2. [YAML y Manifiestos](#yaml-manifiestos)
3. [Pods](#pods)
4. [Namespaces](#namespaces)
5. [API Versioning](#api-versioning)
6. [Resource Monitoring](#resource-monitoring)
7. [Requests y Limits](#requests-limits)
8. [Probes (Sondas)](#probes)
9. [ConfigMaps](#configmaps)
10. [Volume Mounting](#volume-mounting)
11. [Secrets](#secrets)
12. [Viewing Logs](#logs)
13. [Labels](#labels)
14. [Deployments](#deployments)
15. [Storage](#storage)
16. [Networking](#networking)
17. [Network Policies](#network-policies)
18. [ClusterIP Services](#clusterip)
19. [NodePort Services](#nodeport)
20. [LoadBalancer Services](#loadbalancer)

---

## <a name="arquitectura"></a>1. Arquitectura de Kubernetes
<img width="1344" height="768" alt="Gemini_Generated_Image_vgiexpvgiexpvgie" src="https://github.com/user-attachments/assets/b34d7056-34fb-484b-a56b-d6722289d093" />

### Descripción
Kubernetes es un sistema de orquestación de contenedores con arquitectura maestro-worker.

### Componentes principales

#### Control Plane (Nodo Maestro)
- **API Server**: Punto de entrada para todas las operaciones REST
- **etcd**: Base de datos distribuida que almacena el estado del cluster
- **Scheduler**: Asigna pods a nodos
- **Controller Manager**: Ejecuta los controladores del cluster
- **Cloud Controller Manager**: Interactúa con proveedores de nube

#### Worker Nodes
- **Kubelet**: Agente que se ejecuta en cada nodo
- **Kube-proxy**: Gestiona reglas de red en los nodos
- **Container Runtime**: Docker, containerd, CRI-O

### Comandos básicos
```bash
# Ver información del cluster
kubectl cluster-info

# Ver nodos del cluster
kubectl get nodes

# Ver componentes del sistema
kubectl get componentstatuses

# Descripción detallada de un nodo
kubectl describe node <node-name>
```

---

## <a name="yaml-manifiestos"></a>2. YAML y Manifiestos

### Descripción
Los manifiestos son archivos YAML que definen recursos de Kubernetes de forma declarativa.

### Estructura básica
```yaml
apiVersion: v1              # Versión de la API
kind: Pod                   # Tipo de recurso
metadata:                   # Metadatos
  name: mi-pod
  labels:
    app: web
spec:                       # Especificación del recurso
  containers:
  - name: nginx
    image: nginx:1.14.2
```

### Comandos
```bash
# Aplicar un manifest
kubectl apply -f manifest.yaml

# Crear recurso desde manifest
kubectl create -f manifest.yaml

# Eliminar recurso desde manifest
kubectl delete -f manifest.yaml

# Validar manifest sin aplicarlo
kubectl apply -f manifest.yaml --dry-run=client

# Ver manifest de un recurso existente
kubectl get pod mi-pod -o yaml
```

---

## <a name="pods"></a>3. Making Pods

### Descripción
Los Pods son la unidad más pequeña de despliegue en Kubernetes. Pueden contener uno o más contenedores.

### Ejemplo de manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
    tier: frontend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
  - name: sidecar
    image: busybox
    command: ['sh', '-c', 'while true; do echo Hello; sleep 10; done']
```

### Comandos
```bash
# Crear pod imperativo
kubectl run nginx --image=nginx

# Crear pod desde manifest
kubectl apply -f pod.yaml

# Listar pods
kubectl get pods
kubectl get pods -o wide

# Ver detalles de un pod
kubectl describe pod nginx-pod

# Eliminar pod
kubectl delete pod nginx-pod

# Ejecutar comando en pod
kubectl exec nginx-pod -- ls /

# Acceder a shell interactivo
kubectl exec -it nginx-pod -- /bin/bash

# Port forwarding
kubectl port-forward nginx-pod 8080:80
```

---

## <a name="namespaces"></a>4. Namespaces

### Descripción
Los namespaces proporcionan aislamiento lógico de recursos dentro del cluster.

### Ejemplo de manifest
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: desarrollo
  labels:
    env: dev
```

### Pod en namespace específico
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
  namespace: desarrollo
spec:
  containers:
  - name: app
    image: myapp:1.0
```

### Comandos
```bash
# Listar namespaces
kubectl get namespaces
kubectl get ns

# Crear namespace
kubectl create namespace desarrollo

# Crear namespace desde manifest
kubectl apply -f namespace.yaml

# Eliminar namespace
kubectl delete namespace desarrollo

# Listar recursos en namespace específico
kubectl get pods -n desarrollo
kubectl get all -n desarrollo

# Establecer namespace por defecto
kubectl config set-context --current --namespace=desarrollo

# Ver namespace actual
kubectl config view --minify | grep namespace:
```

---

## <a name="api-versioning"></a>5. API Versioning

### Descripción
Kubernetes usa versiones de API para evolucionar sus recursos sin romper compatibilidad.

### Niveles de estabilidad
- **Alpha** (v1alpha1): Experimental, puede cambiar
- **Beta** (v1beta1): Pre-release, más estable
- **Stable** (v1, apps/v1): Producción

### Ejemplos por versión
```yaml
# Core API (v1)
apiVersion: v1
kind: Pod

# Apps API
apiVersion: apps/v1
kind: Deployment

# Batch API
apiVersion: batch/v1
kind: Job

# Networking API
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy

# Storage API
apiVersion: storage.k8s.io/v1
kind: StorageClass
```

### Comandos
```bash
# Ver versiones de API disponibles
kubectl api-versions

# Ver recursos disponibles
kubectl api-resources

# Ver recursos de una API específica
kubectl api-resources --api-group=apps

# Explicar schema de un recurso
kubectl explain pod
kubectl explain deployment.spec

# Ver versión preferida de un recurso
kubectl api-resources | grep deployment
```

---

## <a name="resource-monitoring"></a>6. Resource Monitoring

### Descripción
Monitoreo del uso de recursos (CPU, memoria) en el cluster.

### Instalación de Metrics Server
```bash
# Instalar metrics server
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Comandos
```bash
# Ver uso de recursos de nodos
kubectl top nodes

# Ver uso de recursos de pods
kubectl top pods
kubectl top pods -n kube-system

# Ver uso de un pod específico
kubectl top pod nginx-pod

# Ver uso con ordenamiento
kubectl top pods --sort-by=cpu
kubectl top pods --sort-by=memory

# Ver uso de contenedores dentro de pods
kubectl top pods --containers
```

---

## <a name="requests-limits"></a>7. Requests y Limits

### Descripción
- **Requests**: Recursos mínimos garantizados
- **Limits**: Recursos máximos que puede usar

### Ejemplo de manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: resource-pod
spec:
  containers:
  - name: app
    image: nginx
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### Ejemplo con Deployment
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: nginx:1.21
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "200m"
```

### Comandos
```bash
# Ver recursos de un pod
kubectl describe pod resource-pod | grep -A 5 "Limits\|Requests"

# Crear LimitRange para namespace
kubectl create -f limitrange.yaml

# Ver LimitRanges
kubectl get limitrange
kubectl describe limitrange
```

### LimitRange example
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-cpu-limit
  namespace: desarrollo
spec:
  limits:
  - max:
      memory: "1Gi"
      cpu: "1"
    min:
      memory: "64Mi"
      cpu: "100m"
    default:
      memory: "512Mi"
      cpu: "500m"
    defaultRequest:
      memory: "256Mi"
      cpu: "250m"
    type: Container
```

---

## <a name="probes"></a>8. Probes (Sondas)

### Descripción
Las probes verifican el estado de salud de los contenedores.

### Tipos de Probes
- **livenessProbe**: Verifica si el contenedor está vivo
- **readinessProbe**: Verifica si el contenedor está listo para recibir tráfico
- **startupProbe**: Verifica si la aplicación ha iniciado

### Ejemplo completo
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: probe-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    ports:
    - containerPort: 8080
    
    # Liveness Probe - reinicia si falla
    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 30
      periodSeconds: 10
      timeoutSeconds: 5
      failureThreshold: 3
    
    # Readiness Probe - quita del servicio si falla
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
      successThreshold: 1
    
    # Startup Probe - para apps de inicio lento
    startupProbe:
      httpGet:
        path: /startup
        port: 8080
      initialDelaySeconds: 0
      periodSeconds: 10
      failureThreshold: 30
```

### Tipos de verificación
```yaml
# HTTP GET
livenessProbe:
  httpGet:
    path: /health
    port: 8080
    httpHeaders:
    - name: Custom-Header
      value: Awesome

# TCP Socket
livenessProbe:
  tcpSocket:
    port: 8080
  initialDelaySeconds: 15

# Command Exec
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
```

### Comandos
```bash
# Ver estado de probes
kubectl describe pod probe-pod | grep -A 10 "Liveness\|Readiness"

# Ver eventos relacionados con probes
kubectl get events --field-selector involvedObject.name=probe-pod

# Ver logs cuando falla una probe
kubectl logs probe-pod
```

---

## <a name="configmaps"></a>9. ConfigMaps

### Descripción
ConfigMaps permiten separar la configuración del código de la aplicación.

### Ejemplo de manifest
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
  namespace: default
data:
  # Propiedades simples
  database_url: "postgres://db:5432/mydb"
  log_level: "info"
  
  # Archivo de configuración
  app.properties: |
    server.port=8080
    server.host=0.0.0.0
    cache.enabled=true
  
  config.json: |
    {
      "api": {
        "timeout": 30,
        "retries": 3
      }
    }
```

### Uso en Pod - Variables de entorno
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Variable individual
    - name: DATABASE_URL
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: database_url
    
    # Todas las claves como variables
    envFrom:
    - configMapRef:
        name: app-config
```

### Uso en Pod - Volumen
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: config-volume-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
  volumes:
  - name: config-volume
    configMap:
      name: app-config
```

### Comandos
```bash
# Crear ConfigMap desde literal
kubectl create configmap app-config --from-literal=key1=value1 --from-literal=key2=value2

# Crear desde archivo
kubectl create configmap app-config --from-file=config.properties

# Crear desde directorio
kubectl create configmap app-config --from-file=config-dir/

# Crear desde manifest
kubectl apply -f configmap.yaml

# Listar ConfigMaps
kubectl get configmaps
kubectl get cm

# Ver contenido
kubectl describe configmap app-config
kubectl get configmap app-config -o yaml

# Editar ConfigMap
kubectl edit configmap app-config

# Eliminar ConfigMap
kubectl delete configmap app-config
```

---

## <a name="volume-mounting"></a>10. Volume Mounting

### Descripción
Los volúmenes permiten persistir datos más allá del ciclo de vida de los contenedores.

### Tipos comunes de volúmenes

#### emptyDir - Temporal
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: emptydir-pod
spec:
  containers:
  - name: writer
    image: busybox
    command: ['sh', '-c', 'echo "Hello" > /data/hello.txt; sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  - name: reader
    image: busybox
    command: ['sh', '-c', 'cat /data/hello.txt; sleep 3600']
    volumeMounts:
    - name: shared-data
      mountPath: /data
  volumes:
  - name: shared-data
    emptyDir: {}
```

#### hostPath - Nodo local
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: hostpath-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: host-volume
      mountPath: /usr/share/nginx/html
  volumes:
  - name: host-volume
    hostPath:
      path: /data/web
      type: DirectoryOrCreate
```

#### PersistentVolume
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: standard
---
apiVersion: v1
kind: Pod
metadata:
  name: pvc-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: persistent-storage
      mountPath: /data
  volumes:
  - name: persistent-storage
    persistentVolumeClaim:
      claimName: my-pvc
```

### Comandos
```bash
# Ver volúmenes persistentes
kubectl get pv
kubectl get persistentvolumes

# Ver reclamaciones de volúmenes
kubectl get pvc
kubectl get persistentvolumeclaims

# Describir PVC
kubectl describe pvc my-pvc

# Ver volúmenes en uso por pod
kubectl describe pod pvc-pod | grep -A 5 "Volumes:"

# Eliminar PVC
kubectl delete pvc my-pvc
```

---

## <a name="secrets"></a>11. Secrets

### Descripción
Los Secrets almacenan información sensible como contraseñas, tokens y claves.

### Tipos de Secrets
- **Opaque**: Datos arbitrarios (por defecto)
- **kubernetes.io/service-account-token**: Token de service account
- **kubernetes.io/dockerconfigjson**: Credenciales de Docker registry
- **kubernetes.io/tls**: Certificados TLS

### Ejemplo de manifest
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
data:
  # Valores en base64
  username: YWRtaW4=
  password: cGFzc3dvcmQxMjM=
stringData:
  # Valores en texto plano (se codifican automáticamente)
  database: mydb
```

### Secret para Docker Registry
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: registry-secret
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: eyJhdXRocyI6eyJodHRwczovL2luZGV4LmRvY2tlci5pby92MS8iOnsidXNlcm5hbWUiOiJ1c2VyIiwicGFzc3dvcmQiOiJwYXNzIiwiYXV0aCI6ImRYTmxjanB3WVhOeiJ9fX0=
```

### Secret TLS
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: tls-secret
type: kubernetes.io/tls
data:
  tls.crt: LS0tLS1CRUdJTi...
  tls.key: LS0tLS1CRUdJTi...
```

### Uso en Pod - Variables de entorno
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-pod
spec:
  containers:
  - name: app
    image: myapp:1.0
    env:
    # Variable individual
    - name: DB_USERNAME
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: username
    - name: DB_PASSWORD
      valueFrom:
        secretKeyRef:
          name: db-secret
          key: password
    
    # Todas las claves como variables
    envFrom:
    - secretRef:
        name: db-secret
```

### Uso en Pod - Volumen
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secret-volume-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: /etc/secrets
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: db-secret
```

### Comandos
```bash
# Crear Secret desde literal
kubectl create secret generic db-secret --from-literal=username=admin --from-literal=password=pass123

# Crear Secret desde archivo
kubectl create secret generic ssh-secret --from-file=ssh-privatekey=~/.ssh/id_rsa

# Crear Docker registry secret
kubectl create secret docker-registry registry-secret \
  --docker-server=docker.io \
  --docker-username=user \
  --docker-password=pass \
  --docker-email=user@example.com

# Crear TLS secret
kubectl create secret tls tls-secret --cert=path/to/cert.crt --key=path/to/cert.key

# Listar secrets
kubectl get secrets

# Ver secret (datos codificados)
kubectl get secret db-secret -o yaml

# Decodificar secret
kubectl get secret db-secret -o jsonpath='{.data.password}' | base64 --decode

# Describir secret (no muestra valores)
kubectl describe secret db-secret

# Editar secret
kubectl edit secret db-secret

# Eliminar secret
kubectl delete secret db-secret
```

---

## <a name="logs"></a>12. Viewing Logs

### Descripción
Visualización de logs de contenedores para debugging y monitoreo.

### Comandos básicos
```bash
# Ver logs de un pod
kubectl logs nginx-pod

# Logs en tiempo real (follow)
kubectl logs -f nginx-pod

# Logs de contenedor específico en pod multi-contenedor
kubectl logs nginx-pod -c sidecar

# Últimas N líneas
kubectl logs nginx-pod --tail=50

# Logs desde hace X tiempo
kubectl logs nginx-pod --since=1h
kubectl logs nginx-pod --since=10m

# Logs anteriores (de contenedor crasheado)
kubectl logs nginx-pod --previous
kubectl logs nginx-pod -p

# Logs con timestamps
kubectl logs nginx-pod --timestamps

# Logs de todos los contenedores del pod
kubectl logs nginx-pod --all-containers=true

# Logs de múltiples pods por label
kubectl logs -l app=nginx

# Logs de deployment
kubectl logs deployment/web-deployment

# Combinar opciones
kubectl logs -f nginx-pod --tail=20 --timestamps
```

### Comandos avanzados
```bash
# Ver logs de pods en namespace específico
kubectl logs nginx-pod -n desarrollo

# Exportar logs a archivo
kubectl logs nginx-pod > pod-logs.txt

# Ver logs con grep
kubectl logs nginx-pod | grep ERROR

# Logs de múltiples pods simultáneamente (requiere stern)
# Instalar: https://github.com/stern/stern
stern nginx

# Ver logs de jobs
kubectl logs job/my-job

# Ver logs de cronjob
kubectl logs -l job-name=my-cronjob-xxxxx
```

---

## <a name="labels"></a>13. Labels

### Descripción
Los labels son pares clave-valor que se adjuntan a objetos para identificarlos y organizarlos.

### Ejemplo en manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: labeled-pod
  labels:
    app: web
    tier: frontend
    environment: production
    version: "1.0"
    team: backend
spec:
  containers:
  - name: nginx
    image: nginx:1.21
```

### Deployment con labels
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web-deployment
  labels:
    app: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
        tier: frontend
        version: "2.0"
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
```

### Comandos
```bash
# Ver labels de pods
kubectl get pods --show-labels

# Filtrar por label (equality-based)
kubectl get pods -l app=web
kubectl get pods -l tier=frontend
kubectl get pods -l environment=production

# Múltiples labels (AND)
kubectl get pods -l app=web,tier=frontend

# Filtrar por label (set-based)
kubectl get pods -l 'environment in (production, staging)'
kubectl get pods -l 'tier notin (backend)'
kubectl get pods -l 'app'  # Tiene el label app
kubectl get pods -l '!app'  # No tiene el label app

# Ver labels en formato columna
kubectl get pods -L app,tier,version

# Añadir label a pod existente
kubectl label pod nginx-pod env=dev

# Modificar label existente
kubectl label pod nginx-pod env=prod --overwrite

# Eliminar label
kubectl label pod nginx-pod env-

# Añadir label a múltiples recursos
kubectl label pods --all status=active

# Ver recursos por label en todos los namespaces
kubectl get pods -l app=web --all-namespaces

# Describir pod y ver labels
kubectl describe pod nginx-pod | grep Labels
```

### Annotations (similar a labels pero para metadata)
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: annotated-pod
  annotations:
    description: "Pod for web application"
    owner: "team-backend@example.com"
    build-date: "2025-01-01"
spec:
  containers:
  - name: nginx
    image: nginx
```

```bash
# Ver annotations
kubectl describe pod annotated-pod | grep Annotations

# Añadir annotation
kubectl annotate pod nginx-pod description="My web server"
```

---

## <a name="deployments"></a>14. Deployments

### Descripción
Los Deployments gestionan el despliegue y escalado de aplicaciones sin estado, proporcionando actualizaciones declarativas para Pods y ReplicaSets.

### Ejemplo completo
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        image: nginx:1.21
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "250m"
          limits:
            memory: "128Mi"
            cpu: "500m"
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 1
```

### Estrategias de actualización

#### Rolling Update (por defecto)
```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1        # Pods adicionales durante update
      maxUnavailable: 1  # Pods que pueden estar down
```

#### Recreate
```yaml
spec:
  strategy:
    type: Recreate  # Elimina todos los pods antes de crear nuevos
```

### Comandos de creación
```bash
# Crear deployment imperativo
kubectl create deployment nginx --image=nginx:1.21

# Con réplicas
kubectl create deployment nginx --image=nginx:1.21 --replicas=3

# Desde manifest
kubectl apply -f deployment.yaml

# Crear y exponer puerto
kubectl create deployment nginx --image=nginx --port=80
```

### Comandos de gestión
```bash
# Listar deployments
kubectl get deployments
kubectl get deploy

# Ver detalles
kubectl describe deployment nginx-deployment

# Ver estado del rollout
kubectl rollout status deployment/nginx-deployment

# Ver historial de rollouts
kubectl rollout history deployment/nginx-deployment

# Ver revisión específica
kubectl rollout history deployment/nginx-deployment --revision=2

# Escalar deployment
kubectl scale deployment nginx-deployment --replicas=5

# Autoescalar
kubectl autoscale deployment nginx-deployment --min=2 --max=10 --cpu-percent=80

# Actualizar imagen
kubectl set image deployment/nginx-deployment nginx=nginx:1.22

# Editar deployment
kubectl edit deployment nginx-deployment

# Pausar rollout
kubectl rollout pause deployment/nginx-deployment

# Reanudar rollout
kubectl rollout resume deployment/nginx-deployment

# Rollback a versión anterior
kubectl rollout undo deployment/nginx-deployment

# Rollback a revisión específica
kubectl rollout undo deployment/nginx-deployment --to-revision=2

# Eliminar deployment
kubectl delete deployment nginx-deployment
```

### Comandos de debugging
```bash
# Ver ReplicaSets del deployment
kubectl get rs -l app=nginx

# Ver eventos del deployment
kubectl get events --field-selector involvedObject.name=nginx-deployment

# Ver logs de todos los pods del deployment
kubectl logs -l app=nginx --all-containers=true

# Verificar pods del deployment
kubectl get pods -l app=nginx
```

---

## <a name="storage"></a>15. Storage

### Descripción
Kubernetes proporciona varios tipos de almacenamiento para persistir datos.

### PersistentVolume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-local
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node01
```

### PersistentVolumeClaim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-web
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: local-storage
```

### StorageClass
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iopsPerGB: "10"
  fsType: ext4
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### Pod usando PVC
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: storage-pod
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: web-storage
      mountPath: /usr/share/nginx/html
  volumes:
  - name: web-storage
    persistentVolumeClaim:
      claimName: pvc-web
```

### StatefulSet con Storage
```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: database
spec:
  serviceName: "database"
  replicas: 3
  selector:
    matchLabels:
      app: database
  template:
    metadata:
      labels:
        app: database
    spec:
      containers:
      - name: postgres
        image: postgres:13
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "fast-storage"
      resources:
        requests:
          storage: 10Gi
```

### Comandos
```bash
# Listar PersistentVolumes
kubectl get pv

# Listar PersistentVolumeClaims
kubectl get pvc

# Describir PV
kubectl describe pv pv-local

# Describir PVC
kubectl describe pvc pvc-web

# Ver StorageClasses
kubectl get storageclass
kubectl get sc

# Ver detalles de StorageClass
kubectl describe sc fast-storage

# Crear PVC desde manifest
kubectl apply -f pvc.yaml

# Elim
