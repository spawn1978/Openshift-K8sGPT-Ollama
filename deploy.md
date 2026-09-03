# Guía Paso a Paso: Despliegue de IA 100% Local y Libre en OpenShift Local (CRC)
## Integración de Ollama, K8sGPT y Open WebUI

Esta guía detalla el procedimiento completo para desplegar una solución completa de inteligencia artificial y diagnóstico de clústeres en **Red Hat OpenShift Local (CRC)** sin depender de licencias ni suscripciones de pago.

---

## Prerrequisitos

- **Red Hat OpenShift Local (CRC)** instalado y funcionando.
- CLI de OpenShift (`oc`) instalada y con sesión iniciada como usuario con privilegios de administrador (`oc login -u kubeadmin ...`).
- CLI de **Helm** (`helm`) instalada en el sistema local.

---

## Paso 1: Preparar el Proyecto en OpenShift

Crea un proyecto dedicado llamado `ai-assistant` para aislar los recursos de la pila de IA:

```bash
oc new-project ai-assistant
```

---

## Paso 2: Despliegue de Ollama con Almacenamiento Persistente

Despliega el servidor de LLM Ollama junto con un `PersistentVolumeClaim` (PVC) de 15 GiB para evitar la pérdida de los modelos descargados al reiniciar los pods.

Aplica el siguiente archivo de manifiestos:

```bash
oc apply -f - <<EOF
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ollama-storage
  namespace: ai-assistant
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 15Gi
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ollama
  namespace: ai-assistant
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ollama
  template:
    metadata:
      labels:
        app: ollama
    spec:
      containers:
      - name: ollama
        image: ollama/ollama:latest
        ports:
        - containerPort: 11434
        volumeMounts:
        - name: ollama-data
          mountPath: /root/.ollama
      volumes:
      - name: ollama-data
        persistentVolumeClaim:
          claimName: ollama-storage
---
apiVersion: v1
kind: Service
metadata:
  name: ollama-service
  namespace: ai-assistant
spec:
  ports:
  - port: 11434
    targetPort: 11434
  selector:
    app: ollama
EOF
```

---

## Paso 3: Descargar el Modelo de IA en Ollama

Una vez que el Pod de Ollama esté en estado `Running`, descarga el modelo de lenguaje (por ejemplo, `llama3.2`) ejecutando el comando directamente dentro del contenedor:

```bash
# 1. Verificar estado del pod
oc get pods -l app=ollama -n ai-assistant

# 2. Descargar el modelo llama3.2 dentro del pod
oc exec -it deployment/ollama -n ai-assistant -- ollama pull llama3.2
```

---

## Paso 4: Instalar y Configurar K8sGPT Operator

Conecta el motor de análisis de Kubernetes (K8sGPT) con el servicio local de Ollama.

### 4.1. Instalación mediante Helm

```bash
# Agregar e importar repositorio Helm
helm repo add k8sgpt https://charts.k8sgpt.ai/
helm repo update

# Instalar el operador en el namespace ai-assistant
helm install k8sgpt-operator k8sgpt/k8sgpt-operator -n ai-assistant
```

### 4.2. Configuración del recurso personalizado `K8sGPT`

```bash
oc apply -f - <<EOF
apiVersion: core.k8sgpt.ai/v1alpha1
kind: K8sGPT
metadata:
  name: k8sgpt-ollama
  namespace: ai-assistant
spec:
  ai:
    enabled: true
    model: llama3.2
    backend: openai
    baseUrl: http://ollama-service.ai-assistant.svc.cluster.local:11434/v1
  noUnkown: true
  repository: ghcr.io/k8sgpt-ai/k8sgpt
EOF
```

---

## Paso 5: Desplegar Open WebUI y Exponer la Ruta de OpenShift

Despliega la interfaz gráfica interactiva **Open WebUI** ajustando los permisos de Security Context Constraints (SCC) requeridos por OpenShift.

```bash
# 1. Conceder permisos al ServiceAccount predeterminado para correr el contenedor
oc adm policy add-scc-to-user anyuid -z default -n ai-assistant

# 2. Desplegar Open WebUI, Service y Route
oc apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: open-webui
  namespace: ai-assistant
spec:
  replicas: 1
  selector:
    matchLabels:
      app: open-webui
  template:
    metadata:
      labels:
        app: open-webui
    spec:
      containers:
      - name: open-webui
        image: ghcr.io/open-webui/open-webui:main
        env:
        - name: OLLAMA_BASE_URL
          value: http://ollama-service.ai-assistant.svc.cluster.local:11434
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: open-webui-service
  namespace: ai-assistant
spec:
  ports:
  - port: 8080
    targetPort: 8080
  selector:
    app: open-webui
---
apiVersion: route.openshift.io/v1
kind: Route
metadata:
  name: open-webui-route
  namespace: ai-assistant
spec:
  to:
    kind: Service
    name: open-webui-service
  port:
    targetPort: 8080
EOF
```

---

## Paso 6: Acceso y Verificación

### 6.1. Acceder a Open WebUI

1. Obten el hostname público asignado por la Route de OpenShift:
   ```bash
   oc get route open-webui-route -n ai-assistant -o jsonpath='{.spec.host}'
   ```
2. Abre la URL devuelta en tu navegador preferido.
3. Registra una cuenta de administrador local en el primer acceso.
4. En el menú desplegable superior, verifica que el modelo `llama3.2` esté seleccionado.

### 6.2. Consultar Diagnósticos de K8sGPT desde OpenShift

Para listar los hallazgos y análisis generados automáticamente por K8sGPT Operator:

```bash
oc get result -n ai-assistant -o yaml
```