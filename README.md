
# Guía Técnica de Instalación e Integración de IA en Red Hat OpenShift: Entorno Local (CRC) vs. Entorno Empresarial (OpenShift Lightspeed)

## 1. Introducción y Arquitectura

El propósito de este documento es proporcionar un procedimiento detallado, paso a paso, para la implementación de capacidades de Inteligencia Artificial (IA) generativa y diagnóstico asistido sobre clústeres de **Red Hat OpenShift**. 

En la práctica operativa, existen dos escenarios principales de despliegue:

1. **Entorno Local / Sandbox (CodeReady Containers - CRC en macOS Apple Silicon):**
   - **Enfoque:** Uso de **K8sGPT** combinado con **Ollama** ejecutándose de manera nativa sobre el Host (macOS).
   - **Arquitectura:** Aprovecha la GPU unificada del SoC Apple Silicon (M1/M2/M3/M4) mediante Metal. Al mantener el modelo LLM fuera de la máquina virtual (VM) de CRC, se minimiza la sobrecarga de memoria en el nodo virtual de OpenShift.
   - **Ventajas:** 100% gratuito, privado (*air-gapped*), rápido y sin impacto directo en la asignación de RAM del clúster.

2. **Entorno Empresarial / Producción (OpenShift Container Platform con Suscripción):**
   - **Enfoque:** Despliegue de **Red Hat OpenShift Lightspeed (OLS)** mediante el operador nativo.
   - **Arquitectura:** Operador desplegado en el namespace `openshift-lightspeed`, integrado con la consola web de OpenShift y conectado a proveedores LLM empresariales (p. ej., IBM watsonx.ai, RHEL AI, Azure OpenAI, AWS Bedrock).
   - **Ventajas:** Asistencia contextual integrada en la interfaz gráfica, soporte técnico oficial de Red Hat y capacidades de RAG sobre la documentación oficial.

---

## 2. Matriz Comparativa Técnica

| Criterio / Característica | **Instalación Local Gratuita (K8sGPT + Ollama)** | **Instalación Oficial Red Hat (OpenShift Lightspeed)** |
| :--- | :--- | :--- |
| **Suscripción / Costo** | **100% Gratuito y Open Source.** No requiere créditos, licencias ni registros. | **Requiere Suscripción de OpenShift** + suscripción/API Keys del proveedor LLM (watsonx, RHEL AI, OpenAI, Bedrock). |
| **Arquitectura de Ejecución** | El LLM corre **fuera del clúster**, directamente sobre macOS/Metal (ARM) liberando RAM de la VM de CRC. | Corre como un **Operator dentro del clúster** (`openshift-lightspeed` namespace) y se conecta a un backend LLM remoto u On-Prem. |
| **Modelos Soportados** | Cualquiera compatible con Ollama (`llama3.2`, `deepseek-r1:8b`, `mistral`, `qwen2.5`) en formato cuantizado local. | Proveedores oficialmente homologados: IBM watsonx.ai, RHEL AI, OpenShift AI, OpenAI, Azure OpenAI, AWS Bedrock. |
| **Interfaz de Usuario** | **CLI (Terminal)** mediante `k8sgpt analyze` u `ollama run`. (Opción de añadir Open WebUI manualmente). | **Consola Web Nativa de OpenShift** (bot flotante integrado directamente en la UI del administrador/desarrollador). |
| **Capacidades Principales** | Diagnóstico proactivo, triage de errores (`CrashLoopBackOff`, eventos) y generación de YAMLs vía prompt. | Asistencia contextualizada sobre la plataforma, resolución de dudas operativas, soporte integrado y análisis RAG de docs de Red Hat. |
| **Consumo de Recursos** | **Muy Bajo en el clúster.** Ollama usa la GPU unificada del chip Apple Silicon de tu Mac. | **Medio / Alto en el clúster.** El Operator, API Server y Vector DB (RAG) consumen memoria/CPU asignada al clúster. |
| **Privacidad de Datos** | **Total (Air-gapped local).** Todo el tráfico y los logs se procesan en tu propia máquina sin salir a Internet. | Depende del proveedor LLM; requiere conectividad hacia servicios Cloud AI o infraestructuras dedicadas como RHEL AI / OpenShift AI. |
| **Despliegue / Mantenimiento** | Vía `brew` y CLI en macOS (se configura en 5 minutos). | Instalación vía **OperatorHub / Software Catalog** usando OLM (`OLSConfig` Custom Resource). |

---

## 3. Opción A: Instalación Local en CRC (Procesador ARM - macOS)


Esta sección cubre el proceso completo para configurar un motor de IA local y conectarlo con tu clúster CRC sin utilizar créditos ni suscripciones.

### Prerrequisitos
- macOS corriendo sobre procesador Apple Silicon (M1/M2/M3/M4).
- S.O Tahoe 26.6.2
- 32G RAM
- 1T SSD
- OpenShift CRC instalado e iniciado (`crc start`).
- CLI de OpenShift (`oc`) configurado en el `PATH`.
- Homebrew instalado (`/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"`).

---

### Paso 1: Instalación y Configuración del motor Ollama (Host macOS)

1. Instala Ollama utilizando Homebrew:
   ```bash
   brew install ollama
   ```

2. Configura el servicio de Ollama para que se ejecute en segundo plano y escuche peticiones locales:
   ```bash
   brew services start ollama
   ```

3. Descarga e inicializa un modelo optimizado para tareas operativas y de diagnóstico (ej. `llama3.2` o `deepseek-r1:8b`):
   ```bash
   # Modelo Llama 3.2 (ligero y eficiente)
   ollama pull llama3.2

   # Opcional: Modelo DeepSeek R1 8B para razonamiento técnico profundo
   ollama pull deepseek-r1:8b
   ```

4. Verifica que Ollama esté respondiendo correctamente en el puerto local por defecto (`11434`):
   ```bash
   curl http://localhost:11434/api/version
   ```

---

### Paso 2: Instalación de K8sGPT CLI

1. Agrega el tap oficial de K8sGPT e instala el cliente vía Homebrew:
   ```bash
   brew tap k8sgpt-ai/k8sgpt
   brew install k8sgpt
   ```

2. Valida la instalación verificando la versión:
   ```bash
   k8sgpt version
   ```

---

### Paso 3: Vinculación entre K8sGPT y Ollama

1. Registra Ollama como proveedor de backend en K8sGPT definiendo la URL base y el modelo objetivo:
   ```bash
   k8sgpt auth add      --backend ollama      --model llama3.2      --baseurl http://localhost:11434
   ```

2. Establece Ollama como el backend por defecto en K8sGPT:
   ```bash
   k8sgpt auth default --backend ollama
   ```

---

### Paso 4: Diagnóstico y Operación sobre el Clúster CRC

1. Autentícate en tu clúster CRC local utilizando las credenciales de administrador:
   ```bash
   oc login -u kubeadmin -p <tu-password-crc> https://api.crc.testing:6443
   ```

2. Ejecute un análisis integral del clúster con explicación impulsada por el LLM local:
   ```bash
   k8sgpt analyze --explain
   ```

3. Para filtrar el diagnóstico a un namespace específico (ejemplo: `openshift-ingress` o tu proyecto de trabajo):
   ```bash
   k8sgpt analyze --namespace default --explain
   ```

4. Para obtener la salida en formato JSON y automatizar tuberías de scripts:
   ```bash
   k8sgpt analyze --explain --output json
   ```

---

## 4. Opción B: Instalación Oficial Red Hat OpenShift Lightspeed (Suscripción)

Esta sección describe el procedimiento oficial de Red Hat para desplegar **OpenShift Lightspeed** en un clúster empresarial mediante la consola web o la CLI de OpenShift (`oc`).

### Prerrequisitos
- Clúster de OpenShift Container Platform 4.14 o superior con suscripción activa.
- Permisos de administrador de clúster (`cluster-admin`).
- Proveedor de LLM empresarial habilitado y clave API (API Key) o punto de enlace (Endpoint) configurado (ej. IBM watsonx.ai, Azure OpenAI, RHEL AI).

---

### Paso 1: Instalación del Operador mediante OLM (CLI)

1. Crea el namespace dedicado para OpenShift Lightspeed:
   ```yaml
   # 01-namespace.yaml
   apiVersion: v1
   kind: Namespace
   metadata:
     name: openshift-lightspeed
   ```
   Aplica el archivo:
   ```bash
   oc apply -f 01-namespace.yaml
   ```

2. Crea el `OperatorGroup` correspondiente:
   ```yaml
   # 02-operatorgroup.yaml
   apiVersion: operators.coreos.com/v1
   kind: OperatorGroup
   metadata:
     name: lightspeed-operator-group
     namespace: openshift-lightspeed
   spec:
     targetNamespaces:
     - openshift-lightspeed
   ```
   Aplica el archivo:
   ```bash
   oc apply -f 02-operatorgroup.yaml
   ```

3. Crea la Suscripción (`Subscription`) al operador desde CatalogHub:
   ```yaml
   # 03-subscription.yaml
   apiVersion: operators.coreos.com/v1alpha1
   kind: Subscription
   metadata:
     name: lightspeed-operator
     namespace: openshift-lightspeed
   spec:
     channel: stable
     name: lightspeed-operator
     source: redhat-operators
     sourceNamespace: openshift-marketplace
   ```
   Aplica el archivo:
   ```bash
   oc apply -f 03-subscription.yaml
   ```

4. Espera a que el operador pase al estado `Succeeded`:
   ```bash
   oc get csv -n openshift-lightspeed
   ```

---

### Paso 2: Configuración del Secreto del API Key del Proveedor LLM

Crea un `Secret` en el namespace `openshift-lightspeed` que contenga las credenciales de acceso a tu proveedor LLM (ejemplo con IBM watsonx.ai u OpenAI):

```bash
oc create secret generic watsonx-api-key   --from-literal=apm-key='<TU_API_KEY_AQUI>'   -n openshift-lightspeed
```

---

### Paso 3: Despliegue de la Configuración Custom Resource (`OLSConfig`)

Crea la instancia de configuración principal (`OLSConfig`) para definir el proveedor, modelo y secret de autenticación:

```yaml
# 04-olsconfig.yaml
apiVersion: ols.openshift.io/v1alpha1
kind: OLSConfig
metadata:
  name: cluster
spec:
  llm:
    providers:
    - name: watsonx
      type: watsonx
      credentialsSecretRef:
        name: watsonx-api-key
      watsonx:
        projectId: "<TU_PROJECT_ID_WATSONX>"
        url: "https://us-south.ml.cloud.ibm.com"
  ols:
    defaultModel: "ibm/granite-13b-chat-v2"
    defaultProvider: watsonx
    logLevel: INFO
    conversationExpiration: 24h
```

Aplica el manifiesto de configuración:
```bash
oc apply -f 04-olsconfig.yaml
```

---

### Paso 4: Verificación y Acceso a la Interfaz

1. Verifica que los Pods del servicio de Lightspeed estén en estado `Running`:
   ```bash
   oc get pods -n openshift-lightspeed
   ```

2. Recarga la **Consola Web de OpenShift**.
3. Verás un nuevo icono del widget de **OpenShift Lightspeed** (bot de chat) en la barra superior o en el panel lateral inferior de la interfaz gráfica.
4. Interactúa directamente realizando preguntas contextuales sobre el clúster, resolviendo errores de deployments o solicitando guías de configuración.

---

## 5. Resumen de Recomendaciones de Uso

1. **Para CRC en MacBook ARM:** Utiliza la **Opción A (K8sGPT + Ollama)**. Evitarás agotar la memoria RAM de tu máquina virtual y obtendrás tiempos de respuesta ultra rápidos utilizando los núcleos de la GPU de Apple Silicon.
2. **Para Entornos Empresariales Multinodo:** Implementa la **Opción B (OpenShift Lightspeed)** para ofrecer una herramienta nativa, asistida y centralizada a todo el equipo de operaciones e ingeniería DevOps sobre la consola de administración.