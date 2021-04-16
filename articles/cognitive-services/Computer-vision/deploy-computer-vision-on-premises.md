---
title: Uso de un contenedor de Computer Vision con Kubernetes y Helm
titleSuffix: Azure Cognitive Services
description: Aprenda a implementar el contenedor de Computer Vision con Kubernetes y Helm.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 01/27/2020
ms.author: aahi
ms.openlocfilehash: 36091c62814cffd78c5f8132e01820070968af52
ms.sourcegitcommit: b8995b7dafe6ee4b8c3c2b0c759b874dff74d96f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106284953"
---
# <a name="use-computer-vision-container-with-kubernetes-and-helm"></a>Uso de un contenedor de Computer Vision con Kubernetes y Helm

Una opción para administrar los contenedores de Computer Vision es usar Kubernetes y Helm. Vamos a crear un paquete de Kubernetes usando Kubernetes y Helm para definir una imagen de contenedor de Computer Vision. Este paquete se va a implementar en un clúster de Kubernetes en el entorno local. Por último, exploraremos cómo probar los servicios implementados. Para más información sobre la ejecución de contenedores de Docker sin orquestación de Kubernetes, consulte [Instalación y ejecución de contenedores de Computer Vision](computer-vision-how-to-install-containers.md).

## <a name="prerequisites"></a>Requisitos previos

Estos son los requisitos previos para poder usar contenedores de Computer Vision en el entorno local:

| Obligatorio | Propósito |
|----------|---------|
| Cuenta de Azure | Si no tiene una suscripción a Azure, cree una [cuenta gratuita][free-azure-account] antes de empezar. |
| CLI de Kubernetes | La [CLI de Kubernetes][kubernetes-cli] es necesaria para administrar las credenciales compartidas desde el registro de contenedor. Kubernetes también se necesita antes que Helm, que es el administrador de paquetes de Kubernetes. |
| CLI de Helm | Instale la [CLI de Helm][helm-install], que se usa para instalar un gráfico de Helm (definición de paquete de contenedor). |
| Recurso de Computer Vision |Para poder usar el contenedor, debe tener:<br><br>Un recurso de **Computer Vision** de Azure y la clave de API asociada con el URI del punto de conexión. Ambos valores están disponibles en las páginas de introducción y claves del recurso y son necesarios para iniciar el contenedor.<br><br>**{API_KEY}** : una de las dos claves de recurso disponibles en la página **Claves**<br><br>**{ENDPOINT_URI}** : el punto de conexión tal como se proporciona en la página de **Información general**.|

[!INCLUDE [Gathering required parameters](../containers/includes/container-gathering-required-parameters.md)]

### <a name="the-host-computer"></a>El equipo host

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>Recomendaciones y requisitos del contenedor

[!INCLUDE [Container requirements and recommendations](includes/container-requirements-and-recommendations.md)]

## <a name="connect-to-the-kubernetes-cluster"></a>Conectar al clúster de Kubernetes

Se espera que el equipo host tenga un clúster de Kubernetes disponible. Vea este tutorial sobre la [implementación de un clúster de Kubernetes](../../aks/tutorial-kubernetes-deploy-cluster.md) para lograr un reconocimiento conceptual de cómo implementar un clúster de Kubernetes en un equipo host. Encontrará más información sobre las implementaciones en la [documentación de Kubernetes](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

## <a name="configure-helm-chart-values-for-deployment"></a>Configuración de los valores del gráfico de Helm para la implementación

Empiece por crear una carpeta llamada *read*. Luego pegue el siguiente contenido de YAML en un nuevo archivo denominado `chart.yaml`:

```yaml
apiVersion: v2
name: read
version: 1.0.0
description: A Helm chart to deploy the Read OCR container to a Kubernetes cluster
dependencies:
- name: rabbitmq
  condition: read.image.args.rabbitmq.enabled
  version: ^6.12.0
  repository: https://kubernetes-charts.storage.googleapis.com/
- name: redis
  condition: read.image.args.redis.enabled
  version: ^6.0.0
  repository: https://kubernetes-charts.storage.googleapis.com/
```

Para configurar los valores predeterminados del gráfico de Helm, copie y pegue el contenido de YAML en un archivo denominado `values.yaml`. Reemplace los comentarios `# {ENDPOINT_URI}` y `# {API_KEY}` por sus propios valores. Configure resultExpirationPeriod, Redis y RabbitMQ si es necesario.

```yaml
# These settings are deployment specific and users can provide customizations
read:
  enabled: true
  image:
    name: cognitive-services-read
    registry:  mcr.microsoft.com/
    repository: azure-cognitive-services/vision/read
    tag: 3.2-preview.1
    args:
      eula: accept
      billing: # {ENDPOINT_URI}
      apikey: # {API_KEY}
      
      # Result expiration period setting. Specify when the system should clean up recognition results.
      # For example, resultExpirationPeriod=1, the system will clear the recognition result 1hr after the process.
      # resultExpirationPeriod=0, the system will clear the recognition result after result retrieval.
      resultExpirationPeriod: 1
      
      # Redis storage, if configured, will be used by read OCR container to store result records.
      # A cache is required if multiple read OCR containers are placed behind load balancer.
      redis:
        enabled: false # {true/false}
        password: password

      # RabbitMQ is used for dispatching tasks. This can be useful when multiple read OCR containers are
      # placed behind load balancer.
      rabbitmq:
        enabled: false # {true/false}
        rabbitmq:
          username: user
          password: password
```

> [!IMPORTANT]
> - Si no se proporcionan los valores `billing` y `apikey`, los servicios expiran pasados 15 minutos. Del mismo modo, se produce un error en la comprobación porque los servicios no están disponibles.
> 
> - Si implementa varios contenedores OCR de lectura detrás de un equilibrador de carga en, por ejemplo, Docker Compose o Kubernetes, debe tener una caché externa. Dado que el contenedor de procesamiento y el contenedor de la solicitud GET podrían no ser los mismos, una caché externa almacena los resultados y los comparte entre contenedores. Para más información sobre la configuración de caché, consulte [Configuración de contenedores de Docker de Computer Vision](./computer-vision-resource-container-config.md).
>

Cree una carpeta *plantillas* en el directorio *lectura*. Copie y pegue el siguiente YAML en un archivo denominado `deployment.yaml`. El archivo `deployment.yaml` servirá como una plantilla de Helm.

> Las plantillas generan archivos de manifiesto, que son descripciones de recursos con formato YAML que Kubernetes puede entender. [-Guía de plantilla de gráfico de Helm][chart-template-guide]

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: read
  labels:
    app: read-deployment
spec:
  selector:
    matchLabels:
      app: read-app
  template:
    metadata:
      labels:
        app: read-app       
    spec:
      containers:
      - name: {{.Values.read.image.name}}
        image: {{.Values.read.image.registry}}{{.Values.read.image.repository}}
        ports:
        - containerPort: 5000
        env:
        - name: EULA
          value: {{.Values.read.image.args.eula}}
        - name: billing
          value: {{.Values.read.image.args.billing}}
        - name: apikey
          value: {{.Values.read.image.args.apikey}}
        args:        
        - ReadEngineConfig:ResultExpirationPeriod={{ .Values.read.image.args.resultExpirationPeriod }}
        {{- if .Values.read.image.args.rabbitmq.enabled }}
        - Queue:RabbitMQ:HostName={{ include "rabbitmq.hostname" . }}
        - Queue:RabbitMQ:Username={{ .Values.read.image.args.rabbitmq.rabbitmq.username }}
        - Queue:RabbitMQ:Password={{ .Values.read.image.args.rabbitmq.rabbitmq.password }}
        {{- end }}      
        {{- if .Values.read.image.args.redis.enabled }}
        - Cache:Redis:Configuration={{ include "redis.connStr" . }}
        {{- end }}
      imagePullSecrets:
      - name: {{.Values.read.image.pullSecret}}      
--- 
apiVersion: v1
kind: Service
metadata:
  name: read-service
spec:
  type: LoadBalancer
  ports:
  - port: 5000
  selector:
    app: read-app
```

En la misma carpeta *templates*, copie y pegue las siguientes funciones auxiliares en `helpers.tpl`. `helpers.tpl` define funciones útiles para ayudar a generar la plantilla de Helm.

> [!NOTE]
> Este artículo contiene referencias al término esclavo, un término que Microsoft ya no usa. Cuando se quite el término del software, se quitará también del artículo.

```yaml
{{- define "rabbitmq.hostname" -}}
{{- printf "%s-rabbitmq" .Release.Name -}}
{{- end -}}

{{- define "redis.connStr" -}}
{{- $hostMain := printf "%s-redis-master:6379" .Release.Name }}
{{- $hostReplica := printf "%s-redis-slave:6379" .Release.Name -}}
{{- $passWord := printf "password=%s" .Values.read.image.args.redis.password -}}
{{- $connTail := "ssl=False,abortConnect=False" -}}
{{- printf "%s,%s,%s,%s" $hostMain $hostReplica $passWord $connTail -}}
{{- end -}}
```
La plantilla especifica un servicio de equilibrador de carga y la implementación del contenedor o la imagen para lectura.

### <a name="the-kubernetes-package-helm-chart"></a>Paquete de Kubernetes (gráfico de Helm)

El *gráfico de Helm* contiene la configuración de las imágenes de Docker que se van a extraer del registro de contenedor `mcr.microsoft.com`.

> Un [gráfico de Helm][helm-charts] es una colección de archivos que describen un conjunto relacionado de recursos de Kubernetes. Un solo gráfico se podría usar para implementar algo sencillo, como un pod almacenado en memoria, o complejo, como una pila de aplicación web completa con servidores HTTP, bases de datos, memorias caché, etc.

Los *gráficos de Helm* proporcionados extraen las imágenes de Docker del servicio Computer Vision y el servicio correspondiente del registro de contenedor `mcr.microsoft.com`.

## <a name="install-the-helm-chart-on-the-kubernetes-cluster"></a>Instalación del gráfico de Helm en el clúster de Kubernetes

Para instalar el *gráfico de Helm*, es necesario ejecutar el comando [`helm install`][helm-install-cmd]. Asegúrese de ejecutar el comando de instalación desde el directorio situado encima de la carpeta `read`.

```console
helm install read ./read
```

Este es el resultado de ejemplo que se puede ver tras una ejecución de instalación correcta:

```console
NAME: read
LAST DEPLOYED: Thu Sep 04 13:24:06 2019
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Pod(related)
NAME                    READY  STATUS             RESTARTS  AGE
read-57cb76bcf7-45sdh   0/1    ContainerCreating  0         0s

==> v1/Service
NAME     TYPE          CLUSTER-IP    EXTERNAL-IP  PORT(S)         AGE
read     LoadBalancer  10.110.44.86  localhost    5000:31301/TCP  0s

==> v1beta1/Deployment
NAME    READY  UP-TO-DATE  AVAILABLE  AGE
read    0/1    1           0          0s
```

La implementación de Kubernetes puede tardar varios minutos en completarse. Para confirmar que los pods y los servicios se han implementado correctamente y están disponibles, ejecute el siguiente comando:

```console
kubectl get all
```

Debería ver algo parecido al resultado siguiente:

```console
kubectl get all
NAME                        READY   STATUS    RESTARTS   AGE
pod/read-57cb76bcf7-45sdh   1/1     Running   0          17s

NAME                   TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes     ClusterIP      10.96.0.1      <none>        443/TCP          45h
service/read           LoadBalancer   10.110.44.86   localhost     5000:31301/TCP   17s

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/read   1/1     1            1           17s

NAME                              DESIRED   CURRENT   READY   AGE
replicaset.apps/read-57cb76bcf7   1         1         1       17s
```

## <a name="deploy-multiple-v3-containers-on-the-kubernetes-cluster"></a>Implementación de varios contenedores de la versión 3 en el clúster de Kubernetes

A partir de la versión 3 del contenedor, puede utilizar los contenedores en paralelo tanto a nivel de tarea como de página.

Por naturaleza, cada contenedor de la versión 3 tiene un distribuidor y un trabajo de reconocimiento. El primero es responsable de dividir una tarea de varias páginas en varias subtareas de una sola página. El trabajo de reconocimiento está optimizado para reconocer documentos de una sola página. Para lograr el paralelismo en el nivel de página, implemente varios contenedores de la versión 3 detrás de un equilibrador de carga y deje que los contenedores compartan una cola y un almacenamiento universales. 

> [!NOTE] 
> Actualmente solo se admiten Azure Storage y Azure Queue. 

El contenedor que recibe la solicitud puede dividir la tarea en subtareas de una sola página y agregarlas a la cola universal. Cualquier trabajo de reconocimiento de un contenedor menos ocupado puede consumir subtareas de una sola página de la cola, realizar el reconocimiento y cargar el resultado en el almacenamiento. El rendimiento se puede mejorar hasta `n` veces, en función del número de contenedores que se implementen.

El contenedor v3 expone la API de sondeo de ejecución en la ruta de acceso `/ContainerLiveness`. Use el ejemplo de implementación siguiente para configurar un sondeo de ejecución para Kubernetes. 

Copie y pegue el siguiente YAML en un archivo denominado `deployment.yaml`. Reemplace los comentarios `# {ENDPOINT_URI}` y `# {API_KEY}` por sus propios valores. Reemplace el comentario `# {AZURE_STORAGE_CONNECTION_STRING}` por su cadena de conexión de Azure Storage. Configure el número de `replicas` que desee, que se establece en `3` en el ejemplo siguiente.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: read
  labels:
    app: read-deployment
spec:
  selector:
    matchLabels:
      app: read-app
  replicas: # {NUMBER_OF_READ_CONTAINERS}
  template:
    metadata:
      labels:
        app: read-app
    spec:
      containers:
      - name: cognitive-services-read
        image: mcr.microsoft.com/azure-cognitive-services/vision/read
        ports:
        - containerPort: 5000
        env:
        - name: EULA
          value: accept
        - name: billing
          value: # {ENDPOINT_URI}
        - name: apikey
          value: # {API_KEY}
        - name: Storage__ObjectStore__AzureBlob__ConnectionString
          value: # {AZURE_STORAGE_CONNECTION_STRING}
        - name: Queue__Azure__ConnectionString
          value: # {AZURE_STORAGE_CONNECTION_STRING}
        livenessProbe:
          httpGet:
            path: /ContainerLiveness
            port: 5000
          initialDelaySeconds: 60
          periodSeconds: 60
          timeoutSeconds: 20
--- 
apiVersion: v1
kind: Service
metadata:
  name: azure-cognitive-service-read
spec:
  type: LoadBalancer
  ports:
  - port: 5000
    targetPort: 5000
  selector:
    app: read-app
```

Ejecute el siguiente comando. 

```console
kubectl apply -f deployment.yaml
```

A continuación hay un resultado de ejemplo que se puede ver tras la ejecución de una implementación correcta:

```console
deployment.apps/read created
service/azure-cognitive-service-read created
```

La implementación de Kubernetes puede tardar varios minutos en completarse. Para confirmar que tanto los pods como los servicios se han implementado correctamente y están disponibles, ejecute el siguiente comando:

```console
kubectl get all
```

Debería ver una salida de la consola similar a esta:

```console
kubectl get all
NAME                       READY   STATUS    RESTARTS   AGE
pod/read-6cbbb6678-58s9t   1/1     Running   0          3s
pod/read-6cbbb6678-kz7v4   1/1     Running   0          3s
pod/read-6cbbb6678-s2pct   1/1     Running   0          3s

NAME                                   TYPE           CLUSTER-IP   EXTERNAL-IP    PORT(S)          AGE
service/azure-cognitive-service-read   LoadBalancer   10.0.134.0   <none>         5000:30846/TCP   17h
service/kubernetes                     ClusterIP      10.0.0.1     <none>         443/TCP          78d

NAME                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/read   3/3     3            3           3s

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/read-6cbbb6678   3         3         3       3s
```

<!--  ## Validate container is running -->

[!INCLUDE [Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre la instalación de aplicaciones con Helm en Azure Kubernetes Service (AKS), [visite esta página][installing-helm-apps-in-aks].

> [!div class="nextstepaction"]
> [Contenedores de Cognitive Services][cog-svcs-containers]

<!-- LINKS - external -->
[free-azure-account]: https://azure.microsoft.com/free
[git-download]: https://git-scm.com/downloads
[azure-cli]: /cli/azure/install-azure-cli
[docker-engine]: https://www.docker.com/products/docker-engine
[kubernetes-cli]: https://kubernetes.io/docs/tasks/tools/install-kubectl
[helm-install]: https://helm.sh/docs/using_helm/#installing-helm
[helm-install-cmd]: https://helm.sh/docs/intro/using_helm/#helm-install-installing-a-package
[helm-charts]: https://helm.sh/docs/topics/charts/
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[helm-test]: https://helm.sh/docs/helm/#helm-test
[chart-template-guide]: https://helm.sh/docs/chart_template_guide

<!-- LINKS - internal -->
[vision-container-host-computer]: computer-vision-how-to-install-containers.md#the-host-computer
[installing-helm-apps-in-aks]: ../../aks/kubernetes-helm.md
[cog-svcs-containers]: ../cognitive-services-container-support.md
