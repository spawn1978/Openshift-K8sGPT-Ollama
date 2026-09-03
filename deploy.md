
Gemini
Nuevo chat
Buscar chats
Imágenes
Biblioteca
Nuevo cuaderno
ciberdelitos
logica simbolica
Todos los cuadernos
Listar Modelos de Ollama en K8sGPT
Relación entre Arte Pasta y Arte de Mafia
Desactivar voz del cursor en Mac
IA Local para CRC OpenShift
Comparativa de Workstations de Alto Rendimiento
Precios MacBook Pro Max en EE. UU.
Fecha de lanzamiento OpenShift 4.22
Plan de actualización OpenShift 4.18
Consulta sobre documentación de Red Hat
Mantenimiento de tabla log en Quay
Peugeot 208 vs VW Nivus: Comparativa de Largo
Tipos y Características de Pisos de Madera
Barras de sonido para TV
Solicitud de contexto para tarea
Bot de Binance para Ganancia Neta
Servicios de la empresa +Activos
Precios de Disney+ en Argentina
Ideas de regalos originales
Usos alternativos para Garmin nüvi 40
Mejores escritorios regulables en Argentina
Asientos de la Nissan X-Trail Advance en Argentina
Ford F-100 y motores BMW
Materiales para impresoras 3D
Guía para empacar vajilla
Mejores hornos eléctricos empotrables Argentina
Costo de pintura en CABA
Dónde vive Leonel Toti Ríos
Perfil de Marcelo Amiano
Bicicleta Financiera con Dólar Digital
Guía para Preparar y Aplicar Pastina
Cuentas en Euros en Argentina
Desayunos Saludables en Belgrano
Controlar Android desde iPhone
Código Postal Av Juan B Justo 3641
Depreciación SUV 0km en Argentina
Aclaración Sobre Trámites Obligatorios
EZVIZ DL05 vs DL06: Diferencias Clave
Mejores Cerraduras Inteligentes en Argentina
Franquicia Vehicular Discapacidad Ford Nissan
Programar Envío de WhatsApp en iPhone
Conversación con Gemini
necesito listar los modelos disponibles de ollama con el comando k8sgpt

El comando k8sgpt no cuenta con un subcomando directo para consultar el catálogo de modelos locales de Ollama. Esto se debe a que K8sGPT actúa como un cliente que interactúa con la interfaz API OpenAI-compatible de Ollama o LocalAI.  
GitLab
+ 1

Para listar los modelos y verificar la integración, se utiliza una combinación de los siguientes métodos según el contexto:

1. Listar los modelos locales desde la CLI de Ollama
Dado que los modelos se descargan y ejecutan en la instancia de Ollama, el comando directo para ver qué modelos tienes disponibles en tu sistema es:

Bash
ollama list
(O de forma equivalente: ollama ls)  
Rost Glukhov

2. Consultar los modelos expuestos por Ollama vía API (curl)
Si K8sGPT se conecta a una instancia remota o dentro del clúster mediante la API de Ollama, puedes listar los modelos expuestos haciendo una consulta HTTP:

Bash
curl http://localhost:11434/v1/models
(Reemplaza localhost:11434 por la IP o servicio Kubernetes donde esté corriendo tu pod de Ollama).

3. Verificar la configuración activa en K8sGPT  
K8sGPT
Para confirmar qué backend y qué modelo tiene configurado actualmente k8sgpt para analizar tu clúster:

Bash
k8sgpt auth list
Configurar un nuevo modelo de Ollama en K8sGPT
Si descargas un nuevo modelo (por ejemplo, llama3 o mistral) con ollama pull <modelo> y deseas vincularlo a K8sGPT, ejecuta:

Bash
k8sgpt auth add --backend openai --model mistral --baseurl http://localhost:11434/v1
el ejecutar k8sgpt analyze --explain, me devuelve el error : Error: failed while calling AI provider ollama: 404 Not Found: model 'llama3.2' not found

Ese error indica que Ollama no tiene descargado el modelo llama3.2 en la instancia a la que K8sGPT está intentando conectarse, por lo que la API devuelve un código HTTP 404.

Para solucionarlo, debes descargar el modelo en Ollama o reconfigurar K8sGPT con un modelo que ya tengas instalado.

Solución 1: Descargar el modelo en Ollama
Ejecuta el comando de descarga directamente en el entorno donde corre Ollama:

Bash
ollama pull llama3.2
Si Ollama corre dentro de Kubernetes:
Debes ejecutar el comando dentro del pod de Ollama:

Bash
kubectl exec -it deployment/ollama -n <namespace> -- ollama pull llama3.2
Solución 2: Cambiar el modelo en K8sGPT
Si prefieres utilizar otro modelo que ya tengas descargado (por ejemplo, llama3 o mistral), actualiza la configuración de K8sGPT:

Bash
k8sgpt auth add --backend openai --model <nombre-del-modelo> --baseurl http://localhost:11434/v1
Verificación
Una vez descargado el modelo o actualizada la autenticación, vuelve a lanzar el análisis:

Bash
k8sgpt analyze --explain
dentro de CRC de puede desplegar algun widget o recurso similar para interactuar con K8sGPT + Ollama?

Sí, dentro de Red Hat OpenShift Local (CRC) puedes desplegar e integrar interfaces gráficas (widgets, dashboards o consolas) para interactuar con K8sGPT y Ollama.

Existen principalmente tres alternativas según la experiencia visual que busques:

1. Extensión para la Consola Web de OpenShift (Console Dynamic Plugin)
OpenShift permite registrar Dynamic Plugins en su consola web estándar.

Uso: Puedes usar o crear un plugin nativo para la Consola Web de OpenShift. Este desplegará una pestaña o widget en la interfaz de OpenShift donde se incrusta el estado del clúster detectado por el k8sgpt-operator.  
NPM

Cómo funciona: Despliegas el k8sgpt-operator en CRC, el cual genera Custom Resources (K8sGPT). El plugin de la consola consulta las métricas y los Custom Resources para mostrar los errores analizados por Ollama en tiempo real en un panel visual dentro de OpenShift.  
Spectro Cloud

2. Dashboard Web / UI para Ollama + K8sGPT (Open WebUI o Chatbot local)
Si lo que deseas es un entorno de chat visual e interactivo donde puedas consultar en lenguaje natural sobre el estado de tu clúster CRC:

Uso: Despliegas en CRC una interfaz web como Open WebUI o Chatbot UI.  
Medium

Configuración:  
K8sGPT Documentation

Conectas la Web UI al servicio local de Ollama dentro del clúster.

Mediante una arquitectura RAG o integrando un MCP Server (Model Context Protocol) de K8sGPT, la interfaz web no solo te permite chatear con Ollama, sino que le da contexto en tiempo real sobre las alertas e inspecciones de K8sGPT.

3. Kubernetes Dashboard Estándar o Lens
Si usas un panel general como Kubernetes Dashboard o clientes de escritorio conectados a CRC (como Lens o Headlamp):

Integración: Puedes instalar los plugins o extensions de K8sGPT que incorporan botones de "Explain with K8sGPT" directamente en la lista de Pods o eventos con errores. Al hacer clic, llaman a Ollama dentro de la misma interfaz.  
K8sGPT Documentation

Ejemplo rápido de despliegue en CRC (Operador K8sGPT)
Para habilitar la recolección automática de datos en CRC y exponer los resultados a un widget/UI:

Bash
# 1. Instalar el Operador de K8sGPT en CRC
helm repo add k8sgpt https://charts.k8sgpt.ai/
helm repo update
helm install k8sgpt-operator k8sgpt/k8sgpt-operator -n k8sgpt-operator --create-namespace

# 2. Crear el recurso K8sGPT apuntando al servicio de Ollama
oc apply -f - <<EOF
apiVersion: core.k8sgpt.ai/v1alpha1
kind: K8sGPT
metadata:
  name: k8sgpt-ollama
  namespace: k8sgpt-operator
spec:
  ai:
    enabled: true
    model: llama3.2
    backend: openai
    baseUrl: http://ollama.ollama.svc.cluster.local:11434/v1
  noUnkown: true
  repository: ghcr.io/k8sgpt-ai/k8sgpt
EOF
el widget de lightspeed se puede desplegar en crc sin licencia?

No, Red Hat Lightspeed (anteriormente conocido como Ansible Lightspeed / OpenShift Lightspeed) requiere suscripciones o derechos comerciales específicos para su uso en producción y acceso a las funcionalidades y modelos respaldados por Red Hat/IBM.

Sin embargo, en OpenShift Local (CRC) existen matices sobre cómo puedes probarlo o utilizar alternativas sin coste:

1. Developer License / Developer Sandbox (Para pruebas en CRC)
Suscripción de desarrollador gratuita: Red Hat ofrece el programa Red Hat Developer Program (sin coste), el cual te permite descargar CRC e incluye licencias de desarrollo individuales para probar tecnologías de Red Hat.

OpenShift Lightspeed Service Preview / Developer Trial: En entornos de prueba, Red Hat suele habilitar versiones preliminares (Tech Preview o Developer Preview) del operador de OpenShift Lightspeed a las que puedes acceder vinculando tu cuenta gratuita de Red Hat Developer.

Limitación: Aunque no pagues una licencia comercial, requieres un Red Hat Subscription Account válido y estar conectado a los registros y servicios en la nube de Red Hat.

2. Uso con modelos de terceros (Bring Your Own Model / BYOM)
OpenShift Lightspeed está diseñado conceptualmente como un asistente de IA integrado en la consola web de OpenShift.

Si se despliega el OpenShift Lightspeed Operator mediante la consola de OperatorHub en CRC, la integración con LLMs gestionados (como IBM granite o WatsonX) exige credenciales o suscripciones a esos servicios de API.

Actualmente no se permite un despliegue 100% "offline y sin registro" si se busca utilizar las funciones oficiales completas del ecosistema Red Hat Lightspeed sin una cuenta asociada.

3. Alternativa 100% libre / sin licencia en CRC: K8sGPT + Open WebUI / Console Plugin
Si buscas la experiencia de un widget o chat interactivo de IA en CRC sin requerir ninguna licencia ni cuenta de Red Hat:

Desplegar el operador de K8sGPT apuntando a un pod local de Ollama.

Utilizar un Console Dynamic Plugin independiente o desplegar Open WebUI expuesto como una Ruta (Route) en OpenShift.

Esto te brinda una experiencia similar a Lightspeed (asistente de IA analizando los recursos y logs de tu clúster) utilizando modelos totalmente de código abierto (como Llama 3, Mistral o Granite en Ollama) de forma 100% local en tu MacBook/CRC.

necesito el paso a paso detallado de la opecion 3 , alternativa 100% libre

Aquí tienes la guía paso a paso detallada para desplegar Ollama, K8sGPT y una Interfaz Gráfica (Open WebUI) 100% gratuita, local y sin necesidad de licencias en tu entorno de OpenShift Local (CRC).

1
Preparar el proyecto en OpenShift (CRC)
Crear el espacio de trabajo aislado
Abre tu terminal, asegúrate de estar autenticado como administrador en tu clúster de CRC (oc login) y crea un proyecto dedicado para la pila de IA:

Bash
oc new-project ai-assistant
2
Desplegar Ollama en CRC
Ejecutar el servidor LLM local con persistencia
Crea el despliegue de Ollama junto con un almacenamiento persistente (PersistentVolumeClaim) para no perder los modelos al reiniciar los pods.

Aplica el siguiente archivo de manifiestos:

Bash
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
3
Descargar el modelo de IA dentro del Pod
Cargar Llama 3.2 o el modelo de tu elección
Espera a que el pod de Ollama esté en estado Running y ejecuta el comando de descarga dentro del pod:

Bash
# Verificar que el pod esté activo
oc get pods -l app=ollama -n ai-assistant

# Descargar el modelo llama3.2
oc exec -it deployment/ollama -n ai-assistant -- ollama pull llama3.2
4
Instalar y configurar el K8sGPT Operator
Conectar el motor de diagnóstico con Ollama
Instala el operador de K8sGPT utilizando Helm y configura el Custom Resource (K8sGPT) para apuntar al servicio local de Ollama:

Bash
# 1. Agregar el repositorio Helm de K8sGPT
helm repo add k8sgpt https://charts.k8sgpt.ai/
helm repo update

# 2. Instalar el operador
helm install k8sgpt-operator k8sgpt/k8sgpt-operator -n ai-assistant

# 3. Crear la configuración de integración K8sGPT -> Ollama
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
5
Desplegar Open WebUI y exponer la Ruta de OpenShift
Crear la interfaz gráfica interactiva
Despliega la interfaz web para interactuar visualmente con Ollama y expón una Route nativa de OpenShift para acceder desde tu navegador.

Debido a que OpenShift impone restricciones de seguridad de pods (SCC / Security Context Constraints), asignamos los permisos necesarios para el contenedor de Open WebUI:

Bash
# Permitir permisos de usuario al ServiceAccount por defecto
oc adm policy add-scc-to-user anyuid -z default -n ai-assistant

# Desplegar Open WebUI y crear la Route
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
Verificación y Uso
Obtener la URL de la interfaz:
Obtén la dirección pública expuesta por OpenShift ejecutando:

Bash
oc get route open-webui-route -n ai-assistant -o jsonpath='{.spec.host}'
Acceder desde el navegador: Copia el enlace obtenido e ingresa en tu navegador web.

Pinchado de cuenta inicial: Crea una cuenta de administrador local en el primer inicio de Open WebUI.

Selección del modelo: En la barra superior de la interfaz, verás el modelo llama3.2 disponible automáticamente desde Ollama.

Diagnosticar el Clúster con K8sGPT desde la CLI
Si deseas ejecutar un diagnóstico manual rápido del clúster CRC directamente desde la línea de comandos conectada al Pod de K8sGPT:

Bash
# Obtener diagnósticos generados por el operador
oc get result -n ai-assistant -o yaml
bueno, todos los pasos detallados los quiero en un archivo del tipo markdown.md

Paso 2: Despliegue de Ollama con Almacenamiento Persistente
Despliega el servidor de LLM Ollama junto con un PersistentVolumeClaim (PVC) de 15 GiB para evitar la pérdida de los modelos descargados al reiniciar los pods.

Aplica el siguiente archivo de manifiestos:

Bash
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
Paso 3: Descargar el Modelo de IA en Ollama
Una vez que el Pod de Ollama esté en estado Running, descarga el modelo de lenguaje (por ejemplo, llama3.2) ejecutando el comando directamente dentro del contenedor:

Bash
# 1. Verificar estado del pod
oc get pods -l app=ollama -n ai-assistant

# 2. Descargar el modelo llama3.2 dentro del pod
oc exec -it deployment/ollama -n ai-assistant -- ollama pull llama3.2
Paso 4: Instalar y Configurar K8sGPT Operator
Conecta el motor de análisis de Kubernetes (K8sGPT) con el servicio local de Ollama.

4.1. Instalación mediante Helm
Bash
# Agregar e importar repositorio Helm
helm repo add k8sgpt [https://charts.k8sgpt.ai/](https://charts.k8sgpt.ai/)
helm repo update

# Instalar el operador en el namespace ai-assistant
helm install k8sgpt-operator k8sgpt/k8sgpt-operator -n ai-assistant
4.2. Configuración del recurso personalizado K8sGPT
Bash
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
    baseUrl: [http://ollama-service.ai-assistant.svc.cluster.local:11434/v1](http://ollama-service.ai-assistant.svc.cluster.local:11434/v1)
  noUnkown: true
  repository: ghcr.io/k8sgpt-ai/k8sgpt
EOF
Paso 5: Desplegar Open WebUI y Exponer la Ruta de OpenShift
Despliega la interfaz gráfica interactiva Open WebUI ajustando los permisos de Security Context Constraints (SCC) requeridos por OpenShift.

Bash
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
          value: [http://ollama-service.ai-assistant.svc.cluster.local:11434](http://ollama-service.ai-assistant.svc.cluster.local:11434)
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
Paso 6: Acceso y Verificación
6.1. Acceder a Open WebUI
Obten el hostname público asignado por la Route de OpenShift:

Bash
oc get route open-webui-route -n ai-assistant -o jsonpath='{.spec.host}'
Abre la URL devuelta en tu navegador preferido.

Registra una cuenta de administrador local en el primer acceso.

En el menú desplegable superior, verifica que el modelo llama3.2 esté seleccionado.

6.2. Consultar Diagnósticos de K8sGPT desde OpenShift
Para listar los hallazgos y análisis generados automáticamente por K8sGPT Operator:

Bash
oc get result -n ai-assistant -o yaml
"""

file_name = "despliegue_ia_local_crc.md"
with open(file_name, "w", encoding="utf-8") as f:
f.write(markdown_content)

print(f"File saved successfully as {file_name}")


```text?code_stdout&code_event_index=1
File saved successfully as despliegue_ia_local_crc.md

Your Markdown file is ready

Ícono de MD
despliegue_ia_local_crc
 MD 
He generado el archivo despliegue_ia_local_crc.md con la guía detallada paso a paso. El documento incluye todos los manifiestos YAML, comandos de OpenShift/Helm y las explicaciones estructuradas para que puedas ejecutar el despliegue de forma directa.




Gemini es una IA y puede cometer errores.

Analizando
Finalizaó tu suscripción a Google AI Pro
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
