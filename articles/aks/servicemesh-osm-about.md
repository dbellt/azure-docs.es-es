---
title: Open Service Mesh (versión preliminar)
description: Open Service Mesh (OSM) en Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 3/12/2021
ms.custom: mvc, devx-track-azurecli
ms.author: pgibson
zone_pivot_groups: client-operating-system
ms.openlocfilehash: 606151084d399f605f12012ab5b3e323c705199d
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110468130"
---
# <a name="open-service-mesh-aks-add-on-preview"></a>Complemento Open Service Mesh (versión preliminar) de AKS

## <a name="overview"></a>Información general

[Open Service Mesh (OSM)](https://docs.openservicemesh.io/) es una malla de servicio nativa ligera, extensible y en la nube que permite a los usuarios administrar de forma uniforme características de observación para entornos de microservicios muy dinámicos, así como protegerlas y disfrutar de ellas de inmediato.

OSM ejecuta un plano de control basado en Envoy en Kubernetes, se puede configurar con API de [SMI](https://smi-spec.io/) y funciona insertando un proxy de Envoy como un contenedor sidecar junto a cada instancia de la aplicación. El proxy de Envoy contiene y ejecuta reglas en torno a las directivas de control de acceso, implementa la configuración de enrutamiento y captura las métricas. El plano de control configura continuamente los servidores proxy para asegurarse de que las directivas y las reglas de enrutamiento están actualizadas y garantiza que los servidores proxy sean correctos.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="capabilities-and-features"></a>Características y funcionalidades

OSM proporciona el siguiente conjunto de funcionalidades y características para proporcionar una malla de servicio nativa en la nube para los clústeres de Azure Kubernetes Service (AKS):

- Protección de la comunicación de servicio a servicio habilitando mTLS

- Incorporación de aplicaciones a la malla con facilidad habilitando la inserción automática del sidecar del proxy de Envoy

- Configuraciones sencillas y transparentes para el desplazamiento del tráfico en las implementaciones

- Capacidad de definir y ejecutar directivas de control de acceso detalladas de los servicios

- Observabilidad e información detallada sobre las métricas de la aplicación para los servicios de depuración y supervisión

- Integración con servicios o soluciones externos de administración de certificados con una interfaz acoplable

## <a name="scenarios"></a>Escenarios

OSM puede ayudar a las implementaciones de AKS con los siguientes escenarios:

- Provisión de comunicaciones cifradas entre los puntos de conexión de servicio implementados en el clúster

- Autorización del tráfico HTTP/HTTPS y TCP en la malla

- Configuración de controles de tráfico ponderado entre dos o más servicios para implementaciones A/B o de valor controlado

- Recopilación y visualización de KPI desde el tráfico de la aplicación

## <a name="prerequisites"></a>Requisitos previos

- CLI de Azure, versión 2.20.0 o posterior
- Extensión `aks-preview`, versión 0.5.5 o posterior
- Versión v.0.8.0 o posterior de OSM

## <a name="osm-service-quotas-and-limits-preview"></a>Límites y cuotas de servicio de OSM (versión preliminar)

Las limitaciones de la versión preliminar de OSM respecto a las cuotas y los límites de servicio se pueden encontrar en la [página de cuotas y límites regionales](./quotas-skus-regions.md) de AKS.

::: zone pivot="client-operating-system-linux"

[!INCLUDE [Linux - download and install client binary](includes/servicemesh/osm/install-osm-binary-linux.md)]

::: zone-end

::: zone pivot="client-operating-system-macos"

[!INCLUDE [macOS - download and install client binary](includes/servicemesh/osm/install-osm-binary-macos.md)]

::: zone-end

::: zone pivot="client-operating-system-windows"

[!INCLUDE [Windows - download and install client binary](includes/servicemesh/osm/install-osm-binary-windows.md)]

::: zone-end

> [!WARNING]
> No intente instalar OSM desde el archivo binario mediante `osm install`. Esto producirá una instalación de OSM que no está integrada como un complemento para AKS.

### <a name="register-the-aks-openservicemesh-preview-feature"></a>Registro de la característica en vista previa (GB) `AKS-OpenServiceMesh`

Para crear un clúster de AKS que use el complemento Open Service Mesh, debe habilitar la marca de características `AKS-OpenServiceMesh` en su suscripción.

Registre la marca de la característica `AKS-OpenServiceMesh` con el comando [az feature register][az-feature-register], como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "AKS-OpenServiceMesh"
```

Tarda unos minutos en que el estado muestre _Registrado_. Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/AKS-OpenServiceMesh')].{Name:name,State:properties.state}"
```

Cuando haya terminado, actualice el registro del proveedor de recursos _Microsoft.ContainerService_ con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="install-open-service-mesh-osm-azure-kubernetes-service-aks-add-on-for-a-new-aks-cluster"></a>Instalación del complemento Open Service Mesh (OSM) de Azure Kubernetes Service (AKS) para un nuevo clúster de AKS

Para un nuevo escenario de implementación de clústeres de AKS, comenzará con una nueva implementación de un clúster de AKS habilitando el complemento OSM en la operación de creación del clúster.

### <a name="create-a-resource-group"></a>Crear un grupo de recursos

En Azure, puede asignar recursos relacionados a un grupo de recursos. Para crear un grupo de recursos, use [az group create](/cli/azure/group#az_group_create). En el ejemplo siguiente se crea un grupo de recursos denominado _myOsmAksGroup_ en la ubicación _eastus2_:

```azurecli-interactive
az group create --name <myosmaksgroup> --location <eastus2>
```

### <a name="deploy-an-aks-cluster-with-the-osm-add-on-enabled"></a>Implementación de un clúster de AKS con el complemento OSM habilitado

Ahora implementará un nuevo clúster de AKS con el complemento OSM habilitado.

> [!NOTE]
> Tenga en cuenta que el siguiente comando de implementación de AKS utiliza discos de sistema operativo efímeros. Puede encontrar más información aquí sobre los [discos de sistema operativo efímeros para AKS](./cluster-configuration.md#ephemeral-os)

```azurecli-interactive
az aks create -n osm-addon-cluster -g <myosmaksgroup> --kubernetes-version 1.19.6 --node-osdisk-type Ephemeral --node-osdisk-size 30 --network-plugin azure --enable-managed-identity -a open-service-mesh
```

#### <a name="get-aks-cluster-access-credentials"></a>Obtención de credenciales de acceso de clúster de AKS

Obtenga las credenciales de acceso para el nuevo clúster de Kubernetes administrado.

```azurecli-interactive
az aks get-credentials -n <myosmakscluster> -g <myosmaksgroup>
```

## <a name="enable-open-service-mesh-osm-azure-kubernetes-service-aks-add-on-for-an-existing-aks-cluster"></a>Habilitación del complemento Open Service Mesh (OSM) de Azure Kubernetes Service (AKS) para un clúster de AKS existente

En un escenario de clúster de AKS existente, habilitará el complemento OSM en un clúster de AKS existente que ya se ha implementado.

### <a name="enable-the-osm-add-on-to-existing-aks-cluster"></a>Habilitación del complemento OSM en el clúster de AKS existente

Para habilitar el complemento OSM de AKS, deberá ejecutar el comando `az aks enable-addons --addons` pasando el parámetro `open-service-mesh`.

```azurecli-interactive
az aks enable-addons --addons open-service-mesh -g <resource group name> -n <AKS cluster name>
```

Debería ver una salida similar a la salida que se muestra a continuación para confirmar que se ha instalado el complemento OSM de AKS.

```json
{- Finished ..
  "aadProfile": null,
  "addonProfiles": {
    "KubeDashboard": {
      "config": null,
      "enabled": false,
      "identity": null
    },
    "openServiceMesh": {
      "config": {},
      "enabled": true,
      "identity": {
...
```

## <a name="validate-the-aks-osm-add-on-installation"></a>Validación de la instalación del complemento OSM de AKS

Hay varios comandos que se ejecutan para comprobar que todos los componentes del complemento OSM de AKS están habilitados y en ejecución:

En primer lugar, se pueden consultar los perfiles de complemento del clúster para comprobar el estado habilitado de los complementos instalados. El siguiente comando debe devolver "true".

```azurecli-interactive
az aks list -g <resource group name> -o json | jq -r '.[].addonProfiles.openServiceMesh.enabled'
```

Los siguientes comandos `kubectl` informarán del estado del controlador de OSM.

```azurecli-interactive
kubectl get deployments -n kube-system --selector app=osm-controller
kubectl get pods -n kube-system --selector app=osm-controller
kubectl get services -n kube-system --selector app=osm-controller
```

## <a name="accessing-the-aks-osm-add-on"></a>Acceso al complemento OSM de AKS

Actualmente, puede acceder a la configuración del controlador de OSM y ajustarla a través de configmap. Para ver las opciones de configuración del controlador de OSM, consulte el archivo configmap osm-config mediante `kubectl` para ver sus opciones de configuración.

```azurecli-interactive
kubectl get configmap -n kube-system osm-config -o json | jq '.data'
```

La salida del configmap de OSM debe ser similar a la siguiente:

```json
{
  "egress": "true",
  "enable_debug_server": "true",
  "enable_privileged_init_container": "false",
  "envoy_log_level": "error",
  "outbound_ip_range_exclusion_list": "169.254.169.254/32,168.63.129.16/32,<YOUR_API_SERVER_PUBLIC_IP>/32",
  "permissive_traffic_policy_mode": "true",
  "prometheus_scraping": "false",
  "service_cert_validity_duration": "24h",
  "use_https_ingress": "false"
}
```

Observe que **permissive_traffic_policy_mode** está configurado en **true**. El modo de la directiva de tráfico permisivo en OSM es un modo en el que se omite la aplicación de directivas de tráfico de [SMI](https://smi-spec.io/). En este modo, OSM detecta automáticamente los servicios que forman parte de la malla de servicio y programa reglas de la directiva de tráfico en cada sidecar del proxy de Envoy para comunicarse con estos servicios.

> [!WARNING]
> Antes de continuar, compruebe que el modo de la directiva de tráfico permisivo está establecida en true; si no es así, cámbiela a **true** mediante el comando siguiente.

```OSM Permissive Mode to True
kubectl patch ConfigMap -n kube-system osm-config --type merge --patch '{"data":{"permissive_traffic_policy_mode":"true"}}'
```

## <a name="deploy-a-new-application-to-be-managed-by-the-open-service-mesh-osm-azure-kubernetes-service-aks-add-on"></a>Implementación de una nueva aplicación para que la administre el complemento Open Service Mesh (OSM) de Azure Kubernetes Service (AKS)

### <a name="before-you-begin"></a>Antes de empezar

En los pasos que se detallan en este tutorial se supone que ha creado un clúster de AKS (Kubernetes `1.19+` y versiones posteriores, con el RBAC de Kubernetes habilitado), ha establecido una conexión `kubectl` con el clúster (si necesita ayuda con cualquiera de estos elementos, consulte el [inicio rápido de AKS](./kubernetes-walkthrough.md)) y ha instalado el complemento OSM de AKS.

Debe tener instalados los siguientes recursos:

- CLI de Azure, versión 2.20.0 o posterior
- Extensión `aks-preview`, versión 0.5.5 o posterior
- Versión v.0.8.0 o posterior de OSM
- apt-get install jq

### <a name="create-namespaces-for-the-application"></a>Creación de espacios de nombres para la aplicación

En este tutorial, vamos a usar la aplicación de librería de OSM que tiene los siguientes servicios de Kubernetes:

- bookbuyer
- bookthief
- bookstore
- bookwarehouse

Cree espacios de nombres para cada uno de estos componentes de la aplicación.

```azurecli-interactive
for i in bookstore bookbuyer bookthief bookwarehouse; do kubectl create ns $i; done
```

Debería ver la siguiente salida:

```Output
namespace/bookstore created
namespace/bookbuyer created
namespace/bookthief created
namespace/bookwarehouse created
```

### <a name="onboard-the-namespaces-to-be-managed-by-osm"></a>Incorporación de los espacios de nombres que se van a administrar mediante OSM

La adición de los espacios de nombres a la malla de OSM permitirá que el controlador de OSM inserte automáticamente los contenedores del proxy de sidecar de Envoy en la aplicación. Ejecute el siguiente comando para incorporar los espacios de nombres de la aplicación de librería de OSM.

```azurecli-interactive
osm namespace add bookstore bookbuyer bookthief bookwarehouse
```

Debería ver la siguiente salida:

```Output
Namespace [bookstore] successfully added to mesh [osm]
Namespace [bookbuyer] successfully added to mesh [osm]
Namespace [bookthief] successfully added to mesh [osm]
Namespace [bookwarehouse] successfully added to mesh [osm]
```

### <a name="deploy-the-bookstore-application-to-the-aks-cluster"></a>Implementación de la aplicación Bookstore en el clúster de AKS

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookbuyer.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookthief.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookstore.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookwarehouse.yaml
```

A continuación se resumen todas las salidas de la implementación.

```Output
serviceaccount/bookbuyer created
service/bookbuyer created
deployment.apps/bookbuyer created

serviceaccount/bookthief created
service/bookthief created
deployment.apps/bookthief created

service/bookstore created
serviceaccount/bookstore created
deployment.apps/bookstore created

serviceaccount/bookwarehouse created
service/bookwarehouse created
deployment.apps/bookwarehouse created
```

### <a name="checkpoint-what-got-installed"></a>Punto de control: ¿qué se ha instalado?

La aplicación Bookstore de ejemplo es una aplicación de varios niveles que consta de cuatro servicios, que son bookbuyer, bookthief, bookstore y bookwarehouse. El servicio bookbuyer y bookthief se comunican con el servicio bookstore para recuperar libros del servicio bookstore. El servicio bookstore recupera los libros del servicio bookwarehouse para abastecer a bookbuyer y bookthief. Se trata de una aplicación sencilla de varios niveles que funciona bien para mostrar cómo se puede usar una malla de servicio para proteger y autorizar las comunicaciones entre los servicios de aplicaciones. A medida que sigamos con el tutorial, se habilitarán y deshabilitarán las directivas de Service Mesh Interface (SMI) para permitir y no permitir que los servicios se comuniquen a través de OSM. A continuación se muestra un diagrama de arquitectura de lo que se ha instalado para la aplicación bookstore.

![Arquitectura de la aplicación bookbuyer de OSM](./media/aks-osm-addon/osm-bookstore-app-arch.png)

### <a name="verify-the-bookstore-application-running-inside-the-aks-cluster"></a>Comprobación de que la aplicación Bookstore se ejecuta en el clúster de AKS

Ahora, hemos implementado la aplicación bookstore de varios contenedores, pero solo se puede acceder a ella desde el clúster de AKS. Los tutoriales posteriores le ayudarán a exponer la aplicación fuera del clúster a través de un controlador de entrada. Por ahora, vamos a utilizar el reenvío de puertos para acceder a la aplicación bookbuyer dentro del clúster de AKS para comprobar que compra libros del servicio bookstore.

Para comprobar que la aplicación se ejecuta dentro del clúster, utilizaremos un reenvío de puerto para ver la interfaz de usuario de los componentes bookbuyer y bookthief.

En primer lugar, vamos a obtener el nombre del pod de bookbuyer.

```azurecli-interactive
kubectl get pod -n bookbuyer
```

Debería ver una salida similar a la siguiente. El pod de bookbuyer tendrá un nombre único anexado.

```Output
NAME                         READY   STATUS    RESTARTS   AGE
bookbuyer-7676c7fcfb-mtnrz   2/2     Running   0          7m8s
```

Una vez que tenemos el nombre del pod, podemos usar el comando de reenvío de puerto para configurar un túnel desde nuestro sistema local a la aplicación dentro del clúster de AKS. Ejecute el siguiente comando para configurar el reenvío de puertos para el puerto del sistema local 8080. Vuelva a usar el nombre del pod de bookbuyer especificado.

> [!NOTE]
> Para todos los comandos de reenvío de puertos es mejor utilizar un terminal adicional para que pueda seguir trabajando en este tutorial y no desconectar el túnel. También es mejor establecer el túnel de reenvío de puertos fuera de Azure Cloud Shell.

```Bash
kubectl port-forward bookbuyer-7676c7fcfb-mtnrz -n bookbuyer 8080:14001
```

Debería mostrarse una salida similar a esta.

```Output
Forwarding from 127.0.0.1:8080 -> 14001
Forwarding from [::1]:8080 -> 14001
```

Mientras la sesión de reenvío de puertos está en marcha, navegue a la siguiente URL desde un navegador `http://localhost:8080`. Ahora debería poder ver la interfaz de usuario de la aplicación bookbuyer en el explorador, similar a la imagen siguiente.

![Imagen de la interfaz de usuario de la aplicación bookbuyer de OSM](./media/aks-osm-addon/osm-bookbuyer-service-ui.png)

También observará que el número total de libros comprados continúa aumentando al servicio bookstore v1. Todavía no se ha implementado el servicio bookstore v2. Implementaremos el servicio bookstore v2 cuando se demuestren las directivas de división del tráfico de SMI.

También puede comprobar lo mismo para el servicio bookthief.

```azurecli-interactive
kubectl get pod -n bookthief
```

Debería ver una salida similar a la siguiente. El pod de bookthief tendrá un nombre único anexado.

```Output
NAME                         READY   STATUS    RESTARTS   AGE
bookthief-59549fb69c-cr8vl   2/2     Running   0          15m54s
```

Reenvío de puertos al pod de bookthief.

```Bash
kubectl port-forward bookthief-59549fb69c-cr8vl -n bookthief 8080:14001
```

Abra un explorador y vaya a la dirección URL siguiente `http://localhost:8080`. Debería ver que bookthief está robando libros del servicio bookstore. Más adelante, implementaremos una directiva de tráfico para detener a bookthief.

![Imagen de la interfaz de usuario de la aplicación bookthief de OSM](./media/aks-osm-addon/osm-bookthief-service-ui.png)

### <a name="disable-osm-permissive-traffic-mode-for-the-mesh"></a>Deshabilitación del modo de tráfico permisivo de OSM para la malla

Como se mencionó anteriormente al ver la configuración del clúster de OSM, la configuración de OSM tiene habilitada de manera predeterminada la directiva del modo de tráfico permisivo. En este modo, la aplicación de la directiva del trafico se omite y OSM detecta automáticamente los servicios que forman parte de la malla de servicio y programa reglas de la directiva de tráfico en cada sidecar del proxy de Envoy para comunicarse con estos servicios.

Ahora deshabilitaremos la directiva del modo de tráfico permisivo y OSM necesitará directivas de [SMI](https://smi-spec.io/) explícitas implementadas en el clúster para permitir las comunicaciones en la malla de cada servicio. Para deshabilitar el modo de tráfico permisivo, ejecute el siguiente comando para actualizar la propiedad configmap cambiando el valor de `true` a `false`.

```azurecli-interactive
kubectl patch ConfigMap -n kube-system osm-config --type merge --patch '{"data":{"permissive_traffic_policy_mode":"false"}}'
```

Debería ver una salida similar a la siguiente. El pod de bookthief tendrá un nombre único anexado.

```Output
configmap/osm-config patched
```

Para comprobar que se ha deshabilitado el modo de tráfico permisivo, vuelva a enviar el puerto al pod de bookbuyer o bookthief para ver su interfaz de usuario en el explorador y ver si los libros comprados o los libros robados ya no se incrementan. Asegúrese de actualizar el explorador. Si se ha detenido el incremento, la directiva se aplicó correctamente. Ha conseguido impedir que bookthief robe libros, pero ni el elemento bookbuyer puede comprar en el servicio bookstore ni este servicio bookstore puede recuperar libros del almacén bookwarehouse. A continuación, implementaremos directivas de [SMI](https://smi-spec.io/) para permitir que solo se comuniquen los servicios de la malla que desee.

### <a name="apply-service-mesh-interface-smi-traffic-access-policies"></a>Aplicación de directivas de acceso de tráfico de Service Mesh Interface (SMI)

Ahora que hemos deshabilitado todas las comunicaciones de la malla, permitiremos que nuestro servicio bookbuyer se comunique con nuestro servicio bookstore para comprar libros, y que nuestro servicio bookstore se comunique con nuestro servicio bookwarehouse para recuperar los libros que se van a vender.

Implemente las siguientes directivas de [SMI](https://smi-spec.io/).

```azurecli-interactive
kubectl apply -f - <<EOF
---
apiVersion: access.smi-spec.io/v1alpha3
kind: TrafficTarget
metadata:
  name: bookbuyer-access-bookstore
  namespace: bookstore
spec:
  destination:
    kind: ServiceAccount
    name: bookstore
    namespace: bookstore
  rules:
  - kind: HTTPRouteGroup
    name: bookstore-service-routes
    matches:
    - buy-a-book
    - books-bought
  sources:
  - kind: ServiceAccount
    name: bookbuyer
    namespace: bookbuyer
---
apiVersion: specs.smi-spec.io/v1alpha4
kind: HTTPRouteGroup
metadata:
  name: bookstore-service-routes
  namespace: bookstore
spec:
  matches:
  - name: books-bought
    pathRegex: /books-bought
    methods:
    - GET
    headers:
    - "user-agent": ".*-http-client/*.*"
    - "client-app": "bookbuyer"
  - name: buy-a-book
    pathRegex: ".*a-book.*new"
    methods:
    - GET
  - name: update-books-bought
    pathRegex: /update-books-bought
    methods:
    - POST
---
kind: TrafficTarget
apiVersion: access.smi-spec.io/v1alpha3
metadata:
  name: bookstore-access-bookwarehouse
  namespace: bookwarehouse
spec:
  destination:
    kind: ServiceAccount
    name: bookwarehouse
    namespace: bookwarehouse
  rules:
  - kind: HTTPRouteGroup
    name: bookwarehouse-service-routes
    matches:
    - restock-books
  sources:
  - kind: ServiceAccount
    name: bookstore
    namespace: bookstore
  - kind: ServiceAccount
    name: bookstore-v2
    namespace: bookstore
---
apiVersion: specs.smi-spec.io/v1alpha4
kind: HTTPRouteGroup
metadata:
  name: bookwarehouse-service-routes
  namespace: bookwarehouse
spec:
  matches:
    - name: restock-books
      methods:
      - POST
      headers:
      - host: bookwarehouse.bookwarehouse
EOF
```

Debería ver una salida similar a la siguiente.

```Output
traffictarget.access.smi-spec.io/bookbuyer-access-bookstore-v1 created
httproutegroup.specs.smi-spec.io/bookstore-service-routes created
traffictarget.access.smi-spec.io/bookstore-access-bookwarehouse created
httproutegroup.specs.smi-spec.io/bookwarehouse-service-routes created
```

Ahora puede configurar una sesión de reenvío de puertos en los pods de bookbuyer o bookstore y ver que las métricas de libros comprados y libros vendidos vuelven a incrementarse. También puede hacer lo mismo con el pod de bookthief para comprobar que aún no puede robar libros.

### <a name="apply-service-mesh-interface-smi-traffic-split-policies"></a>Aplicación de directivas de división de tráfico de Service Mesh Interface (SMI)

Para nuestra demostración final, crearemos una directiva de división de tráfico de [SMI](https://smi-spec.io/) para configurar la ponderación de las comunicaciones de un servicio a varios servicios como back-end. La función de división del tráfico permite desplazar progresivamente las conexiones de un servicio a otro ponderando el tráfico en una escala de 0 a 100.

El gráfico siguiente es un diagrama de la directiva de división del tráfico de [SMI](https://smi-spec.io/) que se va a implementar. Implementaremos una versión 2 adicional de bookstore y, a continuación, dividiremos el tráfico entrante de bookbuyer, ponderando el 25 % del tráfico al servicio bookstore v1 y el 75 % al servicio bookstore v2.

![Diagrama de división del tráfico de bookbuyer de OSM](./media/aks-osm-addon/osm-bookbuyer-traffic-split-diagram.png)

Implemente el servicio bookstore V2.

```azurecli-interactive
kubectl apply -f - <<EOF
---
apiVersion: v1
kind: Service
metadata:
  name: bookstore-v2
  namespace: bookstore
  labels:
    app: bookstore-v2
spec:
  ports:
  - port: 14001
    name: bookstore-port
  selector:
    app: bookstore-v2
---
# Deploy bookstore-v2 Service Account
apiVersion: v1
kind: ServiceAccount
metadata:
  name: bookstore-v2
  namespace: bookstore
---
# Deploy bookstore-v2 Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: bookstore-v2
  namespace: bookstore
spec:
  replicas: 1
  selector:
    matchLabels:
      app: bookstore-v2
  template:
    metadata:
      labels:
        app: bookstore-v2
    spec:
      serviceAccountName: bookstore-v2
      containers:
        - name: bookstore
          image: openservicemesh/bookstore:v0.8.0
          imagePullPolicy: Always
          ports:
            - containerPort: 14001
              name: web
          command: ["/bookstore"]
          args: ["--path", "./", "--port", "14001"]
          env:
            - name: BOOKWAREHOUSE_NAMESPACE
              value: bookwarehouse
            - name: IDENTITY
              value: bookstore-v2
---
kind: TrafficTarget
apiVersion: access.smi-spec.io/v1alpha3
metadata:
  name: bookbuyer-access-bookstore-v2
  namespace: bookstore
spec:
  destination:
    kind: ServiceAccount
    name: bookstore-v2
    namespace: bookstore
  rules:
  - kind: HTTPRouteGroup
    name: bookstore-service-routes
    matches:
    - buy-a-book
    - books-bought
  sources:
  - kind: ServiceAccount
    name: bookbuyer
    namespace: bookbuyer
EOF
```

Debería ver la siguiente salida.

```Output
service/bookstore-v2 configured
serviceaccount/bookstore-v2 created
deployment.apps/bookstore-v2 created
traffictarget.access.smi-spec.io/bookstore-v2 created
```

Ahora implemente la directiva de división del tráfico para dividir el tráfico de bookbuyer entre los dos servicios bookstore v1 y v2.

```azurecli-interactive
kubectl apply -f - <<EOF
apiVersion: split.smi-spec.io/v1alpha2
kind: TrafficSplit
metadata:
  name: bookstore-split
  namespace: bookstore
spec:
  service: bookstore.bookstore
  backends:
  - service: bookstore
    weight: 25
  - service: bookstore-v2
    weight: 75
EOF
```

Debería ver la siguiente salida.

```Output
trafficsplit.split.smi-spec.io/bookstore-split created
```

Configure un túnel de reenvío de puertos al pod de bookbuyer, y ahora debería ver los libros que se compran en el servicio bookstore v2. Si sigue observando el incremento de las compras, debería notar un incremento más rápido de las compras que se realizan a través del servicio bookstore v2.

![IU de libros comprados de bookbuyer en OSM](./media/aks-osm-addon/osm-bookbuyer-traffic-split-ui.png)

## <a name="manage-existing-deployed-applications-to-be-managed-by-the-open-service-mesh-osm-azure-kubernetes-service-aks-add-on"></a>Administración de aplicaciones implementadas existentes para que las administre el complemento Open Service Mesh (OSM) de Azure Kubernetes Service (AKS)

### <a name="before-you-begin"></a>Antes de empezar

En los pasos que se detallan en este tutorial se supone que ya ha habilitado el complemento OSM de AKS para el clúster de AKS. Si no es así, revise la sección [Habilitación del complemento Open Service Mesh (OSM) de Azure Kubernetes Service (AKS) para un clúster de AKS existente](#enable-open-service-mesh-osm-azure-kubernetes-service-aks-add-on-for-an-existing-aks-cluster) antes de continuar. Además, el clúster de AKS debe ser de la versión Kubernetes `1.19+` o posterior, tener habilitado RBAC de Kubernetes y haber establecido una conexión `kubectl` con el clúster (si necesita ayuda con cualquiera de estos elementos, consulte el [inicio rápido de AKS](./kubernetes-walkthrough.md)), además de haber instalado el complemento OSM de AKS.

Debe tener instalados los siguientes recursos:

- CLI de Azure, versión 2.20.0 o posterior
- Extensión `aks-preview`, versión 0.5.5 o posterior
- Versión v.0.8.0 o posterior de OSM
- apt-get install jq

### <a name="verify-the-open-service-mesh-osm-permissive-traffic-mode-policy"></a>Comprobación de la directiva de modo de tráfico permisivo de Open Service Mesh (OSM)

El modo de la directiva de tráfico permisivo de OSM es un modo en el que se omite la aplicación de directivas de tráfico de [SMI](https://smi-spec.io/). En este modo, OSM detecta automáticamente los servicios que forman parte de la malla de servicio y programa reglas de la directiva de tráfico en cada sidecar del proxy de Envoy para comunicarse con estos servicios.

Para comprobar el modo de tráfico permisivo actual de OSM para el clúster, ejecute el siguiente comando:

```azurecli-interactive
kubectl get configmap -n kube-system osm-config -o json | jq '.data'
```

La salida del configmap de OSM debe ser similar a la siguiente:

```Output
{
  "egress": "true",
  "enable_debug_server": "true",
  "envoy_log_level": "error",
  "permissive_traffic_policy_mode": "true",
  "prometheus_scraping": "false",
  "service_cert_validity_duration": "24h",
  "use_https_ingress": "false"
}
```

Si **permissive_traffic_policy_mode** está configurado en **true**, puede incorporar los espacios de nombres de forma segura sin que se produzca ninguna interrupción en las comunicaciones entre servicios. Si **permissive_traffic_policy_mode** está configurado en **false**, deberá asegurarse de que tiene los manifiestos de directiva de acceso de tráfico de [SMI](https://smi-spec.io/) correctos y de que tiene una cuenta de servicio que representa cada servicio implementado en el espacio de nombres. Siga las instrucciones de [Incorporación de aplicaciones implementadas existentes con la directiva de tráfico permisivo de Open Service Mesh (OSM) configurada como False](#onboard-existing-deployed-applications-with-open-service-mesh-osm-permissive-traffic-policy-configured-as-false).

### <a name="onboard-existing-deployed-applications-with-open-service-mesh-osm-permissive-traffic-policy-configured-as-true"></a>Incorporación de aplicaciones implementadas existentes con la directiva de tráfico permisivo de Open Service Mesh (OSM) configurada como True

Lo primero que haremos es agregar los espacios de nombres de la aplicación implementada a OSM para administrarlos.

```azurecli-interactive
osm namespace add bookstore
```

Debería ver la siguiente salida:

```Output
Namespace [bookstore] successfully added to mesh [osm]
```

A continuación, echaremos un vistazo a la implementación del pod actual en el espacio de nombres. Ejecute el siguiente comando para ver los pods en el espacio de nombres designado.

```azurecli-interactive
kubectl get pod -n bookbuyer
```

Debería ver una salida similar a la siguiente:

```Output
NAME                         READY   STATUS    RESTARTS   AGE
bookbuyer-78666dcff8-wh6wl   1/1     Running   0          43s
```

Observe que la columna **READY** muestra **1/1**, lo que significa que la aplicación del pod solo tiene un contenedor. A continuación, será necesario reiniciar las implementaciones de la aplicación para que OSM pueda insertar el contenedor del proxy sidecar de Envoy con el pod de la aplicación. Vamos a obtener una lista de las implementaciones del espacio de nombres.

```azurecli-interactive
kubectl get deployment -n bookbuyer
```

Debería ver la siguiente salida:

```Output
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
bookbuyer   1/1     1            1           23h
```

Ahora se reiniciará la implementación para insertar el contenedor sidecar del proxy de Envoy con el pod de la aplicación. Ejecute el siguiente comando:

```azurecli-interactive
kubectl rollout restart deployment bookbuyer -n bookbuyer
```

Debería ver la siguiente salida:

```Output
deployment.apps/bookbuyer restarted
```

Si echamos un vistazo a los pods en el espacio de nombres de nuevo:

```azurecli-interactive
kubectl get pod -n bookbuyer
```

Ahora observará que la columna **READY** muestra **2/2** contenedores listos para su pod. El segundo contenedor es el proxy de sidecar de Envoy.

```Output
NAME                         READY   STATUS    RESTARTS   AGE
bookbuyer-84446dd5bd-j4tlr   2/2     Running   0          3m30s
```

Podemos inspeccionar más el pod para ver el proxy de Envoy mediante la ejecución del comando de descripción para ver la configuración.

```azurecli-interactive
kubectl describe pod bookbuyer-84446dd5bd-j4tlr -n bookbuyer
```

```Output
Containers:
  bookbuyer:
    Container ID:  containerd://b7503b866f915711002292ea53970bd994e788e33fb718f1c4f8f12cd4a88198
    Image:         openservicemesh/bookbuyer:v0.8.0
    Image ID:      docker.io/openservicemesh/bookbuyer@sha256:813874bd2dc9c5a259b9657995348cf0822b905e29c4e86f21fdefa0ef21dcee
    Port:          <none>
    Host Port:     <none>
    Command:
      /bookbuyer
    State:          Running
      Started:      Tue, 23 Mar 2021 10:52:53 -0400
    Ready:          True
    Restart Count:  0
    Environment:
      BOOKSTORE_NAMESPACE:  bookstore
      BOOKSTORE_SVC:        bookstore
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from bookbuyer-token-zft2r (ro)
  envoy:
    Container ID:  containerd://f5f1cb5db8d5304e23cc984eb08146ea162a3e82d4262c4472c28d5579c25e10
    Image:         envoyproxy/envoy-alpine:v1.17.1
    Image ID:      docker.io/envoyproxy/envoy-alpine@sha256:511e76b9b73fccd98af2fbfb75c34833343d1999469229fdfb191abd2bbe3dfb
    Ports:         15000/TCP, 15003/TCP, 15010/TCP
    Host Ports:    0/TCP, 0/TCP, 0/TCP
```

Compruebe que la aplicación sigue siendo funcional después de la inserción del proxy de sidecar de Envoy.

### <a name="onboard-existing-deployed-applications-with-open-service-mesh-osm-permissive-traffic-policy-configured-as-false"></a>Incorporación de aplicaciones implementadas existentes con la directiva de tráfico permisivo de Open Service Mesh (OSM) configurada como False

Cuando la configuración de OSM para la directiva de tráfico permisivo está establecida en `false`, OSM requerirá directivas de acceso de tráfico de [SMI](https://smi-spec.io/) explícitas implementadas para que la comunicación entre servicios se produzca en el clúster. Actualmente, OSM también usa cuentas de servicio de Kubernetes como parte de la autorización de comunicaciones entre servicios. Para asegurarse de que las aplicaciones implementadas existentes se comunicarán cuando se administren mediante la malla de OSM, será necesario comprobar la existencia de una cuenta de servicio para usarla, actualizar la implementación de la aplicación con la información de la cuenta de servicio y aplicar las directivas de acceso de tráfico de [SMI](https://smi-spec.io/).

#### <a name="verify-kubernetes-service-accounts"></a>Comprobación de cuentas de servicio de Kubernetes

Compruebe si tiene una cuenta de servicio de Kubernetes en el espacio de nombres en el que está implementada la aplicación.

```azurecli-interactive
kubectl get serviceaccounts -n bookbuyer
```

En el siguiente, hay una cuenta de servicio denominada `bookbuyer` en el espacio de nombres de bookbuyer.

```Output
NAME        SECRETS   AGE
bookbuyer   1         25h
default     1         25h
```

Si no tiene una cuenta de servicio que no sea la cuenta predeterminada, deberá crear una para la aplicación. Use el comando siguiente como ejemplo para crear una cuenta de servicio en el espacio de nombres implementado de la aplicación.

```azurecli-interactive
kubectl create serviceaccount myserviceaccount -n bookbuyer
```

```Output
serviceaccount/myserviceaccount created
```

#### <a name="view-your-applications-current-deployment-specification"></a>Ver la especificación de la implementación actual de la aplicación

Si tuviera que crear una cuenta de servicio desde la sección anterior, lo más probable es que la implementación de la aplicación no esté configurada con un valor `serviceAccountName` específico en las especificaciones de implementación. Podemos ver la especificación de la implementación de la aplicación con los siguientes comandos:

```azurecli-interactive
kubectl get deployment -n bookbuyer
```

En la salida se mostrará una lista de las implementaciones.

```Output
NAME        READY   UP-TO-DATE   AVAILABLE   AGE
bookbuyer   1/1     1            1           25h
```

Ahora describiremos la implementación como una comprobación para ver si hay una cuenta de servicio enumerada en la sección de plantilla de pod.

```azurecli-interactive
kubectl describe deployment bookbuyer -n bookbuyer
```

En esta implementación en particular, puede ver que hay una cuenta de servicio asociada a la implementación enumerada en la sección de plantilla de pod. Esta implementación usa la cuenta de servicio bookbuyer. Si no ve la propiedad **Service Account:** , la implementación no está configurada para usar una cuenta de servicio.

```Output
Pod Template:
  Labels:           app=bookbuyer
                    version=v1
  Annotations:      kubectl.kubernetes.io/restartedAt: 2021-03-23T10:52:49-04:00
  Service Account:  bookbuyer
  Containers:
   bookbuyer:
    Image:      openservicemesh/bookbuyer:v0.8.0

```

Hay varias técnicas para actualizar la implementación con el fin de agregar una cuenta de servicio de kubernetes. Revise la documentación de Kubernetes sobre cómo [actualizar una implementación](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#updating-a-deployment) insertada o [configurar cuentas de servicio para pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/). Una vez que haya actualizado las especificaciones de implementación con la cuenta de servicio, vuelva a implementar (kubectl apply -f your-deployment.yaml) su implementación en el clúster.

#### <a name="deploy-the-necessary-service-mesh-interface-smi-policies"></a>Implementación de las directivas de Service Mesh Interface (SMI) necesarias

El último paso para permitir que el tráfico autorizado fluya en la malla es implementar las directivas de acceso de tráfico de [SMI](https://smi-spec.io/) necesarias para la aplicación. La cantidad de configuración que se puede lograr con las directivas de acceso de tráfico de [SMI](https://smi-spec.io/) queda fuera del ámbito de este tutorial, pero se detallan algunos de los componentes comunes de la especificación y se muestra cómo configurar una directiva sencilla de TrafficTarget y HTTPRouteGroup para habilitar la comunicación entre servicios para la aplicación.

La especificación [SMI](https://smi-spec.io/) [**Traffic Access Control**](https://github.com/servicemeshinterface/smi-spec/blob/main/apis/traffic-access/v1alpha3/traffic-access.md#traffic-access-control) permite a los usuarios definir la directiva de control de acceso para sus aplicaciones. Nos centraremos en los recursos de API de **TrafficTarget** y **HTTPRoutGroup**.

El recurso TrafficTarget consta de tres valores de configuración principales: destino, reglas y orígenes. A continuación, se muestra un ejemplo de TrafficTarget.

```TrafficTarget Example spec
apiVersion: access.smi-spec.io/v1alpha3
kind: TrafficTarget
metadata:
  name: bookbuyer-access-bookstore-v1
  namespace: bookstore
spec:
  destination:
    kind: ServiceAccount
    name: bookstore
    namespace: bookstore
  rules:
  - kind: HTTPRouteGroup
    name: bookstore-service-routes
    matches:
    - buy-a-book
    - books-bought
  sources:
  - kind: ServiceAccount
    name: bookbuyer
    namespace: bookbuyer
```

En la especificación de TrafficTarget anterior, `destination` denota la cuenta de servicio que está configurada para el servicio de origen de destino. Recuerde que la cuenta de servicio que se agregó a la implementación anterior se usará para autorizar el acceso a la implementación a la que está asociada. La sección `rules`, en este ejemplo concreto, define el tipo de tráfico HTTP que se permite a través de la conexión. Puede configurar patrones regex precisos para que los encabezados HTTP sean específicos respecto al tráfico que se permite a través de HTTP. La sección `sources` es el servicio que origina las comunicaciones. Esta especificación lee que el elemento bookbuyer necesita comunicarse con bookstore.

El recurso HTTPRouteGroup se compone de una coincidencia o una matriz de ellas de la información del encabezado HTTP y es un requisito de la especificación TrafficTarget. En el ejemplo siguiente, puede ver que HTTPRouteGroup está autorizando tres acciones HTTP, dos GET y una POST.

```HTTPRouteGroup Example Spec
apiVersion: specs.smi-spec.io/v1alpha4
kind: HTTPRouteGroup
metadata:
  name: bookstore-service-routes
  namespace: bookstore
spec:
  matches:
  - name: books-bought
    pathRegex: /books-bought
    methods:
    - GET
    headers:
    - "user-agent": ".*-http-client/*.*"
    - "client-app": "bookbuyer"
  - name: buy-a-book
    pathRegex: ".*a-book.*new"
    methods:
    - GET
  - name: update-books-bought
    pathRegex: /update-books-bought
    methods:
    - POST
```

Si no está familiarizado con el tipo de tráfico HTTP que la aplicación de front-end realiza a otros niveles de la aplicación, ya que la especificación de TrafficTarget requiere una regla, puede crear el equivalente de una regla de permitir todo con la siguiente especificación para HTTPRouteGroup.

```HTTPRouteGroup Allow All Example
apiVersion: specs.smi-spec.io/v1alpha4
kind: HTTPRouteGroup
metadata:
  name: allow-all
  namespace: yournamespace
spec:
  matches:
  - name: allow-all
    pathRegex: '.*'
    methods: ["GET","PUT","POST","DELETE","PATCH"]
```

Una vez configurada la especificación de TrafficTarget y HTTPRouteGroup, puede combinarla como un YAML e implementarlo. A continuación se muestra la configuración del ejemplo de bookstore.

```Bookstore Example TrafficTarget and HTTPRouteGroup configuration
kubectl apply -f - <<EOF
---
apiVersion: access.smi-spec.io/v1alpha3
kind: TrafficTarget
metadata:
  name: bookbuyer-access-bookstore-v1
  namespace: bookstore
spec:
  destination:
    kind: ServiceAccount
    name: bookstore
    namespace: bookstore
  rules:
  - kind: HTTPRouteGroup
    name: bookstore-service-routes
    matches:
    - buy-a-book
    - books-bought
  sources:
  - kind: ServiceAccount
    name: bookbuyer
    namespace: bookbuyer
---
apiVersion: specs.smi-spec.io/v1alpha4
kind: HTTPRouteGroup
metadata:
  name: bookstore-service-routes
  namespace: bookstore
spec:
  matches:
  - name: books-bought
    pathRegex: /books-bought
    methods:
    - GET
    headers:
    - "user-agent": ".*-http-client/*.*"
    - "client-app": "bookbuyer"
  - name: buy-a-book
    pathRegex: ".*a-book.*new"
    methods:
    - GET
  - name: update-books-bought
    pathRegex: /update-books-bought
    methods:
    - POST
EOF
```

Visite el sitio de [SMI](https://smi-spec.io/) para obtener información más detallada sobre la especificación.

### <a name="manage-the-applications-namespace-with-osm"></a>Administración del espacio de nombres de la aplicación con OSM

A continuación, configuraremos OSM para administrar el espacio de nombres y reiniciaremos las implementaciones para obtener el proxy de sidecar de Envoy insertado con la aplicación.

Ejecute el siguiente comando para configurar el espacio de nombres `azure-vote` que se va a administrar en mi OSM.

```azurecli-interactive
osm namespace add azure-vote
```

```Output
Namespace [azure-vote] successfully added to mesh [osm]
```

A continuación, reinicie las implementaciones `azure-vote-front` y `azure-vote-back` con los siguientes comandos.

```azurecli-interactive
kubectl rollout restart deployment azure-vote-front -n azure-vote
kubectl rollout restart deployment azure-vote-back -n azure-vote
```

```Output
deployment.apps/azure-vote-front restarted
deployment.apps/azure-vote-back restarted
```

Si vemos los pods del espacio de nombres `azure-vote`, veremos la fase **READY** de `azure-vote-front` y `azure-vote-back` como 2/2, lo que significa que se ha insertado el proxy de sidecar de Envoy junto a la aplicación.

## <a name="tutorial-deploy-an-application-managed-by-open-service-mesh-osm-with-nginx-ingress"></a>Tutorial: Implementación de una aplicación administrada por Open Service Mesh (OSM) con entrada de NGINX

Open Service Mesh (OSM) es una malla de servicio nativa ligera, extensible y en la nube que permite a los usuarios administrar de forma uniforme características de observación para entornos de microservicios muy dinámicos, así como protegerlas y disfrutar de ellas de inmediato.

En este tutorial va a:

> [!div class="checklist"]
>
> - Ver la configuración actual del clúster de OSM
> - Crear los espacios de nombres de OSM para administrar las aplicaciones implementadas en los espacios de nombres
> - Incorporar los espacios de nombres que se van a administrar mediante OSM
> - Implementación de la aplicación de ejemplo
> - Comprobar que la aplicación se ejecuta en el clúster de AKS
> - Crear un controlador de entrada de NGINX usado para aplicación
> - Exponer un servicio a través de la entrada de Azure Application Gateway a Internet

### <a name="before-you-begin"></a>Antes de empezar

En los pasos que se detallan en este artículo se supone que ha creado un clúster de AKS (Kubernetes `1.19+` y versiones posteriores, con el RBAC de Kubernetes habilitado), ha establecido una conexión `kubectl` con el clúster (si necesita ayuda con cualquiera de estos elementos, consulte el [inicio rápido de AKS](./kubernetes-walkthrough.md)) y ha instalado el complemento OSM de AKS.

Debe tener instalados los siguientes recursos:

- CLI de Azure, versión 2.20.0 o posterior
- Extensión `aks-preview`, versión 0.5.5 o posterior
- Versión v.0.8.0 o posterior de OSM
- apt-get install jq

### <a name="view-and-verify-the-current-osm-cluster-configuration"></a>Visualización y comprobación de la configuración actual del clúster de OSM

Una vez que el complemento OSM para AKS se ha habilitado en el clúster de AKS, puede ver los parámetros de configuración actuales en el archivo ConfigMap osm-config de Kubernetes. Ejecute el siguiente comando para ver las propiedades de ConfigMap:

```azurecli-interactive
kubectl get configmap -n kube-system osm-config -o json | jq '.data'
```

La salida muestra la configuración actual de OSM para el clúster.

```json
{
  "egress": "true",
  "enable_debug_server": "true",
  "enable_privileged_init_container": "false",
  "envoy_log_level": "error",
  "outbound_ip_range_exclusion_list": "169.254.169.254,168.63.129.16,20.193.57.43",
  "permissive_traffic_policy_mode": "false",
  "prometheus_scraping": "false",
  "service_cert_validity_duration": "24h",
  "use_https_ingress": "false"
}
```

Observe que **permissive_traffic_policy_mode** está configurado en **true**. El modo de la directiva de tráfico permisivo en OSM es un modo en el que se omite la aplicación de directivas de tráfico de [SMI](https://smi-spec.io/). En este modo, OSM detecta automáticamente los servicios que forman parte de la malla de servicio y programa reglas de la directiva de tráfico en cada sidecar del proxy de Envoy para comunicarse con estos servicios.

### <a name="create-namespaces-for-the-application"></a>Creación de espacios de nombres para la aplicación

En este tutorial, vamos a usar la aplicación bookstore de OSM que tiene los siguientes componentes:

- bookbuyer
- bookthief
- bookstore
- bookwarehouse

Cree espacios de nombres para cada uno de estos componentes de la aplicación.

```azurecli-interactive
for i in bookstore bookbuyer bookthief bookwarehouse; do kubectl create ns $i; done
```

Debería ver la siguiente salida:

```Output
namespace/bookstore created
namespace/bookbuyer created
namespace/bookthief created
namespace/bookwarehouse created
```

### <a name="onboard-the-namespaces-to-be-managed-by-osm"></a>Incorporación de los espacios de nombres que se van a administrar mediante OSM

La adición de los espacios de nombres a la malla de OSM permitirá que el controlador de OSM inserte automáticamente los contenedores del proxy de sidecar de Envoy en la aplicación. Ejecute el siguiente comando para incorporar los espacios de nombres de la aplicación de librería de OSM.

```azurecli-interactive
osm namespace add bookstore bookbuyer bookthief bookwarehouse
```

Debería ver la siguiente salida:

```Output
Namespace [bookstore] successfully added to mesh [osm]
Namespace [bookbuyer] successfully added to mesh [osm]
Namespace [bookthief] successfully added to mesh [osm]
Namespace [bookwarehouse] successfully added to mesh [osm]
```

### <a name="deploy-the-bookstore-application-to-the-aks-cluster"></a>Implementación de la aplicación Bookstore en el clúster de AKS

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookbuyer.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookthief.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookstore.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookwarehouse.yaml
```

A continuación se resumen todas las salidas de la implementación.

```Output
serviceaccount/bookbuyer created
service/bookbuyer created
deployment.apps/bookbuyer created

serviceaccount/bookthief created
service/bookthief created
deployment.apps/bookthief created

service/bookstore created
serviceaccount/bookstore created
deployment.apps/bookstore created

serviceaccount/bookwarehouse created
service/bookwarehouse created
deployment.apps/bookwarehouse created
```

### <a name="update-the-bookbuyer-service"></a>Actualización del servicio bookbuyer

Actualice el servicio bookbuyer a la configuración de puerto de entrada correcta con el siguiente manifiesto de servicio.

```azurecli-interactive
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: bookbuyer
  namespace: bookbuyer
  labels:
    app: bookbuyer
spec:
  ports:
  - port: 14001
    name: inbound-port
  selector:
    app: bookbuyer
EOF
```

### <a name="verify-the-bookstore-application-running-inside-the-aks-cluster"></a>Comprobación de que la aplicación Bookstore se ejecuta en el clúster de AKS

Ahora, hemos implementado la aplicación bookstore de varios contenedores, pero solo se puede acceder a ella desde el clúster de AKS. Más adelante se agregará el controlador de entrada de Azure Application Gateway para exponer la aplicación fuera del clúster de AKS. Para comprobar que la aplicación se ejecuta dentro del clúster, utilizaremos un reenvío de puertos para ver la interfaz de usuario del componente bookbuyer.

En primer lugar, vamos a obtener el nombre del pod de bookbuyer.

```azurecli-interactive
kubectl get pod -n bookbuyer
```

Debería ver una salida similar a la siguiente. El pod de bookbuyer tendrá un nombre único anexado.

```Output
NAME                         READY   STATUS    RESTARTS   AGE
bookbuyer-7676c7fcfb-mtnrz   2/2     Running   0          7m8s
```

Una vez que tenemos el nombre del pod, podemos usar el comando de reenvío de puerto para configurar un túnel desde nuestro sistema local a la aplicación dentro del clúster de AKS. Ejecute el siguiente comando para configurar el reenvío de puertos para el puerto del sistema local 8080. Vuelva a usar el nombre del pod de bookbuyer especificado.

```azurecli-interactive
kubectl port-forward bookbuyer-7676c7fcfb-mtnrz -n bookbuyer 8080:14001
```

Debería mostrarse una salida similar a esta.

```Output
Forwarding from 127.0.0.1:8080 -> 14001
Forwarding from [::1]:8080 -> 14001
```

Mientras la sesión de reenvío de puertos está en marcha, navegue a la siguiente URL desde un navegador `http://localhost:8080`. Ahora debería poder ver la interfaz de usuario de la aplicación bookbuyer en el explorador, similar a la imagen siguiente.

![Imagen de la IU de la aplicación bookbuyer para NGINX de OSM](./media/aks-osm-addon/osm-agic-bookbuyer-img.png)

### <a name="create-an-nginx-ingress-controller-in-azure-kubernetes-service-aks"></a>Creación de un controlador de entrada de NGINX en Azure Kubernetes Service (AKS)

Un controlador de entrada es un software que proporciona el proxy inverso, el enrutamiento del tráfico configurable y la terminación de TLS para los servicios de Kubernetes. Los recursos de entrada de Kubernetes se usan para configurar las reglas de entrada y las rutas de los distintos servicios de Kubernetes. Mediante reglas de entrada y un controlador de entrada, se puede usar una sola dirección IP para enrutar el tráfico a varios servicios en un clúster de Kubernetes.

Usaremos el controlador de entrada para exponer la aplicación administrada por OSM a Internet. Para crear el controlador de entrada, use Helm para instalar nginx-ingress. Para obtener redundancia adicional, se implementan dos réplicas de los controladores de entrada NGINX con el parámetro `--set controller.replicaCount`. Para sacar el máximo provecho de las réplicas en ejecución del controlador de entrada, asegúrese de que hay más de un nodo en el clúster de AKS.

El controlador de entrada también debe programarse en un nodo de Linux. Los nodos de Windows Server no deben ejecutar el controlador de entrada. Un selector de nodos se especifica mediante el parámetro `--set nodeSelector` para indicar al programador de Kubernetes que ejecute el controlador de entrada NGINX en un nodo basado en Linux.

> [!TIP]
> En el siguiente ejemplo se crea un espacio de nombres de Kubernetes para los recursos de entrada denominado _ingress-basic_. Especifique un espacio de nombres para su propio entorno según sea necesario.

```azurecli-interactive
# Create a namespace for your ingress resources
kubectl create namespace ingress-basic

# Add the ingress-nginx repository
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx

# Update the helm repo(s)
helm repo update

# Use Helm to deploy an NGINX ingress controller in the ingress-basic namespace
helm install nginx-ingress ingress-nginx/ingress-nginx \
    --namespace ingress-basic \
    --set controller.replicaCount=1 \
    --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set defaultBackend.nodeSelector."beta\.kubernetes\.io/os"=linux \
    --set controller.admissionWebhooks.patch.nodeSelector."beta\.kubernetes\.io/os"=linux
```

Cuando se crea el servicio del equilibrador de carga de Kubernetes para el controlador de entrada NGINX, se asigna la dirección IP pública dinámica, como se muestra en la salida del ejemplo siguiente:

```Output
$ kubectl --namespace ingress-basic get services -o wide -w nginx-ingress-ingress-nginx-controller

NAME                                     TYPE           CLUSTER-IP    EXTERNAL-IP     PORT(S)                      AGE   SELECTOR
nginx-ingress-ingress-nginx-controller   LoadBalancer   10.0.74.133   EXTERNAL_IP     80:32486/TCP,443:30953/TCP   44s   app.kubernetes.io/component=controller,app.kubernetes.io/instance=nginx-ingress,app.kubernetes.io/name=ingress-nginx
```

No se han creado reglas de entrada aún, por lo que aparece la página 404 predeterminada del controlador de entrada NGINX si navega a la dirección IP interna. Las reglas de entrada se configuran en los pasos siguientes.

### <a name="expose-the-bookbuyer-service-to-the-internet"></a>Exposición del servicio bookbuyer a Internet

```azurecli-interactive
kubectl apply -f - <<EOF
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: bookbuyer-ingress
  namespace: bookbuyer
  annotations:
    kubernetes.io/ingress.class: nginx

spec:

  rules:
    - host: bookbuyer.contoso.com
      http:
        paths:
        - path: /
          backend:
            serviceName: bookbuyer
            servicePort: 14001

  backend:
    serviceName: bookbuyer
    servicePort: 14001
EOF
```

Debería ver la siguiente salida:

```Output
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions/bookbuyer-ingress created
```

### <a name="view-the-nginx-logs"></a>Visualización de los registros de NGINX

```azurecli-interactive
POD=$(kubectl get pods -n ingress-basic | grep 'nginx-ingress' | awk '{print $1}')

kubectl logs $POD -n ingress-basic -f
```

La salida muestra el estado del controlador de entrada de NGINX cuando se ha aplicado correctamente la regla de entrada:

```Output
I0321 <date>       6 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-basic", Name:"nginx-ingress-ingress-nginx-controller-54cf6c8bf4-jdvrw", UID:"3ebbe5e5-50ef-481d-954d-4b82a499ebe1", APIVersion:"v1", ResourceVersion:"3272", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
I0321 <date>        6 event.go:282] Event(v1.ObjectReference{Kind:"Ingress", Namespace:"bookbuyer", Name:"bookbuyer-ingress", UID:"e1018efc-8116-493c-9999-294b4566819e", APIVersion:"networking.k8s.io/v1beta1", ResourceVersion:"5460", FieldPath:""}): type: 'Normal' reason: 'Sync' Scheduled for sync
I0321 <date>        6 controller.go:146] "Configuration changes detected, backend reload required"
I0321 <date>        6 controller.go:163] "Backend successfully reloaded"
I0321 <date>        6 event.go:282] Event(v1.ObjectReference{Kind:"Pod", Namespace:"ingress-basic", Name:"nginx-ingress-ingress-nginx-controller-54cf6c8bf4-jdvrw", UID:"3ebbe5e5-50ef-481d-954d-4b82a499ebe1", APIVersion:"v1", ResourceVersion:"3272", FieldPath:""}): type: 'Normal' reason: 'RELOAD' NGINX reload triggered due to a change in configuration
```

### <a name="view-the-nginx-services-and-bookbuyer-service-externally"></a>Visualización de los servicios de NGINX y el servicio bookbuyer externamente

```azurecli-interactive
kubectl get services -n ingress-basic
```

```Output
NAME                                               TYPE           CLUSTER-IP    EXTERNAL-IP   PORT(S)                      AGE
nginx-ingress-ingress-nginx-controller             LoadBalancer   10.0.100.23   20.193.1.74   80:31742/TCP,443:32683/TCP   4m15s
nginx-ingress-ingress-nginx-controller-admission   ClusterIP      10.0.163.98   <none>        443/TCP                      4m15s
```

Dado que el nombre de host en el manifiesto de entrada es un pseudonombre que se usa para las pruebas, el nombre DNS no estará disponible en Internet. También podemos usar el programa de curl y después el encabezado de nombre de host para la dirección IP pública de NGINX y recibir un código 200 que nos conecta correctamente con el servicio bookbuyer.

```azurecli-interactive
curl -H 'Host: bookbuyer.contoso.com' http://EXTERNAL-IP/
```

Debería ver la siguiente salida:

```Output
<!doctype html>
<html itemscope="" itemtype="http://schema.org/WebPage" lang="en">
  <head>
      <meta content="Bookbuyer" name="description">
      <meta content="text/html; charset=UTF-8" http-equiv="Content-Type">
      <title>Bookbuyer</title>
      <style>
        #navbar {
            width: 100%;
            height: 50px;
            display: table;
            border-spacing: 0;
            white-space: nowrap;
            line-height: normal;
            background-color: #0078D4;
            background-position: left top;
            background-repeat-x: repeat;
            background-image: none;
            color: white;
            font: 2.2em "Fira Sans", sans-serif;
        }
        #main {
            padding: 10pt 10pt 10pt 10pt;
            font: 1.8em "Fira Sans", sans-serif;
        }
        li {
            padding: 10pt 10pt 10pt 10pt;
            font: 1.2em "Consolas", sans-serif;
        }
      </style>
      <script>
        setTimeout(function(){window.location.reload(1);}, 1500);
      </script>
  </head>
  <body bgcolor="#fff">
    <div id="navbar">
      &#128214; Bookbuyer
    </div>
    <div id="main">
      <ul>
        <li>Total books bought: <strong>1833</strong>
          <ul>
            <li>from bookstore V1: <strong>277</strong>
            <li>from bookstore V2: <strong>1556</strong>
          </ul>
        </li>
      </ul>
    </div>

    <br/><br/><br/><br/>
    <br/><br/><br/><br/>
    <br/><br/><br/><br/>

    Current Time: <strong>Fri, 26 Mar 2021 15:02:53 UTC</strong>
  </body>
</html>
```

## <a name="tutorial-deploy-an-application-managed-by-open-service-mesh-osm-using-azure-application-gateway-ingress-aks-add-on"></a>Tutorial: Implementación de una aplicación administrada por Open Service Mesh (OSM) mediante el complemento de AKS de entrada de Azure Application Gateway

Open Service Mesh (OSM) es una malla de servicio nativa ligera, extensible y en la nube que permite a los usuarios administrar de forma uniforme características de observación para entornos de microservicios muy dinámicos, así como protegerlas y disfrutar de ellas de inmediato.

En este tutorial va a:

> [!div class="checklist"]
>
> - Ver la configuración actual del clúster de OSM
> - Crear los espacios de nombres de OSM para administrar las aplicaciones implementadas en los espacios de nombres
> - Incorporar los espacios de nombres que se van a administrar mediante OSM
> - Implementación de la aplicación de ejemplo
> - Comprobar que la aplicación se ejecuta en el clúster de AKS
> - Crear una instancia de Azure Application Gateway que se usará como controlador de entrada para la aplicación
> - Exponer un servicio a través de la entrada de Azure Application Gateway a Internet

### <a name="before-you-begin"></a>Antes de empezar

En los pasos que se detallan en este artículo se supone que ha creado un clúster de AKS (Kubernetes `1.19+` y versiones posteriores, con el RBAC de Kubernetes habilitado), ha establecido una conexión `kubectl` con el clúster (si necesita ayuda con cualquiera de estos elementos, consulte el [inicio rápido de AKS](./kubernetes-walkthrough.md)), ha instalado el complemento OSM de AKS y va a crear una nueva instancia de Azure Application Gateway para la entrada.

Debe tener instalados los siguientes recursos:

- CLI de Azure, versión 2.20.0 o posterior
- Extensión `aks-preview`, versión 0.5.5 o posterior
- Clúster de AKS, versión 1.19, que use redes de Azure CNI (conectadas a una red virtual de Azure)
- Versión v.0.8.0 o posterior de OSM
- apt-get install jq

### <a name="view-and-verify-the-current-osm-cluster-configuration"></a>Visualización y comprobación de la configuración actual del clúster de OSM

Una vez que el complemento OSM para AKS se ha habilitado en el clúster de AKS, puede ver los parámetros de configuración actuales en el archivo ConfigMap osm-config de Kubernetes. Ejecute el siguiente comando para ver las propiedades de ConfigMap:

```azurecli-interactive
kubectl get configmap -n kube-system osm-config -o json | jq '.data'
```

La salida muestra la configuración actual de OSM para el clúster.

```json
{
  "egress": "true",
  "enable_debug_server": "true",
  "enable_privileged_init_container": "false",
  "envoy_log_level": "error",
  "outbound_ip_range_exclusion_list": "169.254.169.254,168.63.129.16,20.193.57.43",
  "permissive_traffic_policy_mode": "false",
  "prometheus_scraping": "false",
  "service_cert_validity_duration": "24h",
  "use_https_ingress": "false"
}
```

Observe que **permissive_traffic_policy_mode** está configurado en **true**. El modo de la directiva de tráfico permisivo en OSM es un modo en el que se omite la aplicación de directivas de tráfico de [SMI](https://smi-spec.io/). En este modo, OSM detecta automáticamente los servicios que forman parte de la malla de servicio y programa reglas de la directiva de tráfico en cada sidecar del proxy de Envoy para comunicarse con estos servicios.

### <a name="create-namespaces-for-the-application"></a>Creación de espacios de nombres para la aplicación

En este tutorial, vamos a usar la aplicación bookstore de OSM que tiene los siguientes componentes:

- bookbuyer
- bookthief
- bookstore
- bookwarehouse

Cree espacios de nombres para cada uno de estos componentes de la aplicación.

```azurecli-interactive
for i in bookstore bookbuyer bookthief bookwarehouse; do kubectl create ns $i; done
```

Debería ver la siguiente salida:

```Output
namespace/bookstore created
namespace/bookbuyer created
namespace/bookthief created
namespace/bookwarehouse created
```

### <a name="onboard-the-namespaces-to-be-managed-by-osm"></a>Incorporación de los espacios de nombres que se van a administrar mediante OSM

La adición de los espacios de nombres a la malla de OSM permitirá que el controlador de OSM inserte automáticamente los contenedores del proxy de sidecar de Envoy en la aplicación. Ejecute el siguiente comando para incorporar los espacios de nombres de la aplicación de librería de OSM.

```azurecli-interactive
osm namespace add bookstore bookbuyer bookthief bookwarehouse
```

Debería ver la siguiente salida:

```Output
Namespace [bookstore] successfully added to mesh [osm]
Namespace [bookbuyer] successfully added to mesh [osm]
Namespace [bookthief] successfully added to mesh [osm]
Namespace [bookwarehouse] successfully added to mesh [osm]
```

### <a name="deploy-the-bookstore-application-to-the-aks-cluster"></a>Implementación de la aplicación Bookstore en el clúster de AKS

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookbuyer.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookthief.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookstore.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v0.8/docs/example/manifests/apps/bookwarehouse.yaml
```

A continuación se resumen todas las salidas de la implementación.

```Output
serviceaccount/bookbuyer created
service/bookbuyer created
deployment.apps/bookbuyer created

serviceaccount/bookthief created
service/bookthief created
deployment.apps/bookthief created

service/bookstore created
serviceaccount/bookstore created
deployment.apps/bookstore created

serviceaccount/bookwarehouse created
service/bookwarehouse created
deployment.apps/bookwarehouse created
```

### <a name="update-the-bookbuyer-service"></a>Actualización del servicio bookbuyer

Actualice el servicio bookbuyer a la configuración de puerto de entrada correcta con el siguiente manifiesto de servicio.

```azurecli-interactive
kubectl apply -f - <<EOF
apiVersion: v1
kind: Service
metadata:
  name: bookbuyer
  namespace: bookbuyer
  labels:
    app: bookbuyer
spec:
  ports:
  - port: 14001
    name: inbound-port
  selector:
    app: bookbuyer
EOF
```

### <a name="verify-the-bookstore-application-running-inside-the-aks-cluster"></a>Comprobación de que la aplicación Bookstore se ejecuta en el clúster de AKS

Ahora, hemos implementado la aplicación bookstore de varios contenedores, pero solo se puede acceder a ella desde el clúster de AKS. Más adelante se agregará el controlador de entrada de Azure Application Gateway para exponer la aplicación fuera del clúster de AKS. Para comprobar que la aplicación se ejecuta dentro del clúster, utilizaremos un reenvío de puertos para ver la interfaz de usuario del componente bookbuyer.

En primer lugar, vamos a obtener el nombre del pod de bookbuyer.

```azurecli-interactive
kubectl get pod -n bookbuyer
```

Debería ver una salida similar a la siguiente. El pod de bookbuyer tendrá un nombre único anexado.

```Output
NAME                         READY   STATUS    RESTARTS   AGE
bookbuyer-7676c7fcfb-mtnrz   2/2     Running   0          7m8s
```

Una vez que tenemos el nombre del pod, podemos usar el comando de reenvío de puerto para configurar un túnel desde nuestro sistema local a la aplicación dentro del clúster de AKS. Ejecute el siguiente comando para configurar el reenvío de puertos para el puerto del sistema local 8080. Vuelva a usar el nombre del pod de bookbuyer específico.

```azurecli-interactive
kubectl port-forward bookbuyer-7676c7fcfb-mtnrz -n bookbuyer 8080:14001
```

Debería mostrarse una salida similar a esta.

```Output
Forwarding from 127.0.0.1:8080 -> 14001
Forwarding from [::1]:8080 -> 14001
```

Mientras la sesión de reenvío de puertos está en marcha, navegue a la siguiente URL desde un navegador `http://localhost:8080`. Ahora debería poder ver la interfaz de usuario de la aplicación bookbuyer en el explorador, similar a la imagen siguiente.

![Imagen de interfaz de usuario de la aplicación bookbuyer de OSM para App Gateway](./media/aks-osm-addon/osm-agic-bookbuyer-img.png)

### <a name="create-an-azure-application-gateway-to-expose-the-bookbuyer-application-outside-the-aks-cluster"></a>Creación de una instancia de Azure Application Gateway para exponer la aplicación bookbuyer fuera del clúster de AKS

> [!NOTE]
> Las instrucciones siguientes crearán una nueva instancia de Azure Application Gateway que se usará para la entrada. Si ya tiene una instancia de Azure Application Gateway que desea usar, vaya a la sección para habilitar el complemento del controlador de entrada de Application Gateway.

#### <a name="deploy-a-new-application-gateway"></a>Implementación de una nueva instancia de Application Gateway

> [!NOTE]
> Nos estamos refiriendo a la documentación existente para habilitar el complemento de Controlador de entrada de Application Gateway para un clúster de AKS existente. Se han realizado algunas modificaciones para adaptarse a los materiales de OSM. [Aquí](../application-gateway/tutorial-ingress-controller-add-on-existing.md)encontrará documentación más detallada sobre el tema.

Ahora implementará una nueva instancia de Application Gateway para simular una instancia de Application Gateway existente que quiera usar para equilibrar la carga del tráfico en el clúster de AKS, _myCluster_. El nombre de la instancia de Application Gateway será _myApplicationGateway_, pero tendrá que crear primero un recurso de dirección IP pública, denominado _myPublicIp_, y una nueva red virtual denominada _myVnet_ con el espacio de direcciones 11.0.0.0/8 y una subred con el espacio de direcciones 11.1.0.0/16 llamada _mySubnet_ e implementar la instancia de Application Gateway en _mySubnet_ con _myPublicIp_.

Cuando se usa un clúster de AKS y una instancia de Application Gateway en redes virtuales independientes, los espacios de direcciones de las dos redes virtuales no deben superponerse. El espacio de direcciones predeterminado que implementa un clúster de AKS es 10.0.0.0/8, por lo que establecemos el prefijo de dirección de red virtual de la instancia de Application Gateway en 11.0.0.0/8.

```azurecli-interactive
az group create --name myResourceGroup --location eastus2
az network public-ip create -n myPublicIp -g MyResourceGroup --allocation-method Static --sku Standard
az network vnet create -n myVnet -g myResourceGroup --address-prefix 11.0.0.0/8 --subnet-name mySubnet --subnet-prefix 11.1.0.0/16
az network application-gateway create -n myApplicationGateway -l eastus2 -g myResourceGroup --sku Standard_v2 --public-ip-address myPublicIp --vnet-name myVnet --subnet mySubnet
```

> [!NOTE]
> El complemento del controlador de entrada de Application Gateway (AGIC) admite **solo** las SKU de Application Gateway v2 (estándar y WAF) y **no** las SKU de Application Gateway v1.

#### <a name="enable-the-agic-add-on-for-an-existing-aks-cluster-through-azure-cli"></a>Habilitación del complemento AGIC para un clúster de AKS existente a través de la CLI de Azure

Si quiere seguir usando la CLI de Azure, puede seguir habilitando el complemento AGIC en el clúster de AKS que creó, _myCluster_, y especificar el complemento AGIC para usar la instancia de Application Gateway existente que creó, _myApplicationGateway_.

```azurecli-interactive
appgwId=$(az network application-gateway show -n myApplicationGateway -g myResourceGroup -o tsv --query "id")
az aks enable-addons -n myCluster -g myResourceGroup -a ingress-appgw --appgw-id $appgwId
```

Puede comprobar que el complemento de AKS de Azure Application Gateway se ha habilitado mediante el siguiente comando.

```azurecli-interactive
az aks list -g osm-aks-rg -o json | jq -r .[].addonProfiles.ingressApplicationGateway.enabled
```

Este comando debería mostrar la salida como `true`.

#### <a name="peer-the-two-virtual-networks-together"></a>Emparejamiento de dos redes virtuales juntas

Dado que hemos implementado el clúster de AKS en su propia red virtual y la instancia de Application Gateway en otra red virtual, deberá emparejar las dos redes virtuales juntas para que el tráfico fluya de la instancia de Application Gateway a los pods del clúster. Emparejar las dos redes virtuales requiere ejecutar el comando de la CLI de Azure dos veces independientes para asegurarse de que la conexión sea bidireccional. El primer comando creará una conexión de emparejamiento desde la red virtual de Application Gateway a la red virtual de AKS; el segundo comando creará una conexión de emparejamiento en la otra dirección.

```azurecli-interactive
nodeResourceGroup=$(az aks show -n myCluster -g myResourceGroup -o tsv --query "nodeResourceGroup")
aksVnetName=$(az network vnet list -g $nodeResourceGroup -o tsv --query "[0].name")

aksVnetId=$(az network vnet show -n $aksVnetName -g $nodeResourceGroup -o tsv --query "id")
az network vnet peering create -n AppGWtoAKSVnetPeering -g myResourceGroup --vnet-name myVnet --remote-vnet $aksVnetId --allow-vnet-access

appGWVnetId=$(az network vnet show -n myVnet -g myResourceGroup -o tsv --query "id")
az network vnet peering create -n AKStoAppGWVnetPeering -g $nodeResourceGroup --vnet-name $aksVnetName --remote-vnet $appGWVnetId --allow-vnet-access
```

### <a name="expose-the-bookbuyer-service-to-the-internet"></a>Exposición del servicio bookbuyer a Internet

Aplique el siguiente manifiesto de entrada al clúster de AKS para exponer el servicio bookbuyer a Internet a través de Azure Application Gateway.

```azurecli-interactive
kubectl apply -f - <<EOF
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: bookbuyer-ingress
  namespace: bookbuyer
  annotations:
    kubernetes.io/ingress.class: azure/application-gateway

spec:

  rules:
    - host: bookbuyer.contoso.com
      http:
        paths:
        - path: /
          backend:
            serviceName: bookbuyer
            servicePort: 14001

  backend:
    serviceName: bookbuyer
    servicePort: 14001
EOF
```

Debería ver la siguiente salida.

```Output
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.extensions/bookbuyer-ingress created
```

Dado que el nombre de host en el manifiesto de entrada es un pseudonombre que se usa para las pruebas, el nombre DNS no estará disponible en Internet. También podemos usar el programa de curl y después el encabezado de nombre de host para la dirección IP pública de Azure Application Gateway y recibir un código 200 que nos conecta correctamente con el servicio bookbuyer.

```azurecli-interactive
appGWPIP=$(az network public-ip show -g MyResourceGroup -n myPublicIp -o tsv --query "ipAddress")
curl -H 'Host: bookbuyer.contoso.com' http://$appGWPIP/
```

Debería ver la siguiente salida.

```Output
<!doctype html>
<html itemscope="" itemtype="http://schema.org/WebPage" lang="en">
  <head>
      <meta content="Bookbuyer" name="description">
      <meta content="text/html; charset=UTF-8" http-equiv="Content-Type">
      <title>Bookbuyer</title>
      <style>
        #navbar {
            width: 100%;
            height: 50px;
            display: table;
            border-spacing: 0;
            white-space: nowrap;
            line-height: normal;
            background-color: #0078D4;
            background-position: left top;
            background-repeat-x: repeat;
            background-image: none;
            color: white;
            font: 2.2em "Fira Sans", sans-serif;
        }
        #main {
            padding: 10pt 10pt 10pt 10pt;
            font: 1.8em "Fira Sans", sans-serif;
        }
        li {
            padding: 10pt 10pt 10pt 10pt;
            font: 1.2em "Consolas", sans-serif;
        }
      </style>
      <script>
        setTimeout(function(){window.location.reload(1);}, 1500);
      </script>
  </head>
  <body bgcolor="#fff">
    <div id="navbar">
      &#128214; Bookbuyer
    </div>
    <div id="main">
      <ul>
        <li>Total books bought: <strong>5969</strong>
          <ul>
            <li>from bookstore V1: <strong>277</strong>
            <li>from bookstore V2: <strong>5692</strong>
          </ul>
        </li>
      </ul>
    </div>

    <br/><br/><br/><br/>
    <br/><br/><br/><br/>
    <br/><br/><br/><br/>

    Current Time: <strong>Fri, 26 Mar 2021 16:34:30 UTC</strong>
  </body>
</html>
```

### <a name="troubleshooting"></a>Solución de problemas

- [Documentación de solución de problemas de AGIC](../application-gateway/ingress-controller-troubleshoot.md)
- [Hay otras herramientas de solución de problemas disponibles en el repositorio de GitHub de AGIC](https://github.com/Azure/application-gateway-kubernetes-ingress/blob/master/docs/troubleshootings/troubleshooting-installing-a-simple-application.md).

## <a name="open-service-mesh-osm-monitoring-and-observability-using-azure-monitor-and-applications-insights"></a>Supervisión y observabilidad de Open Service Mesh (OSM) mediante Azure Monitor y Application Insights

Azure Monitor y Azure Application Insights ayudan a maximizar la disponibilidad y el rendimiento de las aplicaciones y los servicios con una completa solución que permite recopilar datos de telemetría tanto en la nube como en entornos locales, así como analizarlos y actuar sobre ellos.

El complemento OSM de AKS tendrá integraciones profundas en ambos servicios de Azure y proporcionará una experiencia de Azure aparentemente ininterrumpida para ver los KPI críticos proporcionados por las métricas de OSM y responder a ellos. Para obtener más información sobre cómo habilitar y configurar estos servicios para el complemento OSM de AKS, visite la página [Azure Monitor para OSM](https://aka.ms/azmon/osmpreview) para obtener más información.

## <a name="tutorial-manually-deploy-prometheus-grafana-and-jaeger-to-view-open-service-mesh-osm-metrics-for-observability"></a>Tutorial: Implementación manual de Prometheus, Grafana y Jaeger para ver las métricas de Open Service Mesh (OSM) con fines de observabilidad

> [!WARNING]
> La instalación de Prometheus, Grafana y Jaeger se proporciona como guía general para mostrar cómo se pueden usar estas herramientas para ver los datos de métricas de OSM. La guía de instalación no se utiliza para una configuración de producción. Consulte la documentación de cada herramienta sobre la mejor manera de adaptarse a sus necesidades de instalación. Lo más notable es la falta de almacenamiento persistente, lo que significa que todos los datos se pierden una vez que se cierra un pod de Prometheus Grafana o Jaeger.

Open Service Mesh (OSM) genera métricas detalladas relacionadas con todo el tráfico dentro de la malla. Estas métricas proporcionan información sobre el comportamiento de las aplicaciones de la malla, ayudando a los usuarios a solucionar, mantener y analizar sus aplicaciones.

Hoy, OSM recopila las métricas directamente de los servidores proxy de sidecar (Envoy). OSM proporciona métricas enriquecidas para el tráfico entrante y saliente de todos los servicios de la malla. Con estas métricas, el usuario puede obtener información sobre el volumen total de tráfico, los errores dentro del tráfico y el tiempo de respuesta de las solicitudes.

OSM usa Prometheus para recopilar y almacenar estadísticas y métricas de tráfico coherentes para todas las aplicaciones que se ejecutan en la malla. Prometheus es un kit de herramientas de supervisión y alertas de código abierto, que se usa normalmente en entornos de Kubernetes y de malla de servicio (pero no exclusivamente).

Cada aplicación que forma parte de la malla se ejecuta en un pod que contiene un sidecar de Envoy que expone las métricas (métricas de proxy) en el formato de Prometheus. Además, cada pod que forma parte de la malla tiene anotaciones de Prometheus, lo que permite que el servidor de Prometheus desbloquee la aplicación de forma dinámica. Este mecanismo habilita automáticamente la extracción de métricas cada vez que se agrega un nuevo espacio de nombres/pod/servicio a la malla.

Las métricas de OSM se pueden ver con Grafana, que es un software de análisis y visualización de código abierto. Permite consultar, visualizar, enviar alertas y explorar las métricas.

En este tutorial va a:

> [!div class="checklist"]
>
> - Crear e implementar una instancia de Prometheus
> - Configurar OSM para permitir la extracción de Prometheus
> - Actualizar el archivo Configmap de Prometheus
> - Crear e implementar una instancia de Grafana
> - Configurar Grafana con el origen de datos de Prometheus
> - Importar el panel de OSM para Grafana
> - Crear e implementar una instancia de Jaeger
> - Configurar el seguimiento de Jaeger para OSM

### <a name="deploy-and-configure-a-prometheus-instance-for-osm"></a>Implementación y configuración de una instancia de Prometheus para OSM

Usaremos Helm para implementar la instancia de Prometheus. Ejecute los siguientes comandos para instalar Prometheus a través de Helm:

```azurecli-interactive
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install stable prometheus-community/prometheus
```

Debería ver una salida similar a la siguiente si la instalación se realizó correctamente. Anote el puerto del servidor de Prometheus y el nombre DNS del clúster. Esta información se usará más adelante para configurar Prometheus como un origen de datos para Grafana.

```Output
NAME: stable
LAST DEPLOYED: Fri Mar 26 13:34:51 2021
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The Prometheus server can be accessed via port 80 on the following DNS name from within your cluster:
stable-prometheus-server.default.svc.cluster.local


Get the Prometheus server URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9090


The Prometheus alertmanager can be accessed via port 80 on the following DNS name from within your cluster:
stable-prometheus-alertmanager.default.svc.cluster.local


Get the Alertmanager URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=alertmanager" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9093
#################################################################################
######   WARNING: Pod Security Policy has been moved to a global property.  #####
######            use .Values.podSecurityPolicy.enabled with pod-based      #####
######            annotations                                               #####
######            (e.g. .Values.nodeExporter.podSecurityPolicy.annotations) #####
#################################################################################


The Prometheus PushGateway can be accessed via port 9091 on the following DNS name from within your cluster:
stable-prometheus-pushgateway.default.svc.cluster.local


Get the PushGateway URL by running these commands in the same shell:
  export POD_NAME=$(kubectl get pods --namespace default -l "app=prometheus,component=pushgateway" -o jsonpath="{.items[0].metadata.name}")
  kubectl --namespace default port-forward $POD_NAME 9091

For more information on running Prometheus, visit:
https://prometheus.io/
```

#### <a name="configure-osm-to-allow-prometheus-scraping"></a>Configuración de OSM para permitir la extracción de Prometheus

Para asegurarse de que los componentes de OSM estén configurados para las extracciones de Prometheus, queremos comprobar la configuración de **prometheus_scraping** ubicada en el archivo de configuración de osm-config. Vea la configuración con el siguiente comando:

```azurecli-interactive
kubectl get configmap -n kube-system osm-config -o json | jq '.data.prometheus_scraping'
```

La salida del comando anterior debe devolver `true` si OSM está configurado para la extracción de Prometheus. Si el valor devuelto es `false`, es necesario actualizar la configuración para que sea `true`. Ejecute el siguiente comando para **activar** la extracción de Prometheus en OMS:

```azurecli-interactive
kubectl patch ConfigMap -n kube-system osm-config --type merge --patch '{"data":{"prometheus_scraping":"true"}}'
```

Debería ver la siguiente salida.

```Output
configmap/osm-config patched
```

#### <a name="update-the-prometheus-configmap"></a>Actualización del archivo Configmap de Prometheus

La instalación predeterminada de Prometheus contendrá dos configmaps de Kubernetes. Puede ver la lista de configmaps de Prometheus con el siguiente comando.

```azurecli-interactive
kubectl get configmap | grep prometheus
```

```Output
stable-prometheus-alertmanager   1      4h34m
stable-prometheus-server         5      4h34m
```

Tendremos que reemplazar la configuración de prometheus.yml que se encuentra en el configmap **stable-prometheus-server** por la siguiente configuración de OSM. Hay varias técnicas de edición de archivos para realizar esta tarea. Una manera sencilla y segura es exportar el configmap, crear una copia de seguridad y, a continuación, editarlo con un editor como Visual Studio Code.

> [!NOTE]
> Si no tiene Visual Studio Code instalado, puede descargarlo e instalarlo [aquí](https://code.visualstudio.com/Download).

Vamos a exportar primero el configmap **stable-prometheus-server** y luego realizar una copia de seguridad.

```azurecli-interactive
kubectl get configmap stable-prometheus-server -o yaml > cm-stable-prometheus-server.yml
cp cm-stable-prometheus-server.yml cm-stable-prometheus-server.yml.copy
```

A continuación, vamos a abrir el archivo con Visual Studio Code para editarlo.

```azurecli-interactive
code cm-stable-prometheus-server.yml
```

Una vez que tenga el configmap abierto en el editor de Visual Studio Code, reemplace el archivo prometheus.yml por la configuración OSM que aparece a continuación y guarde el archivo.

> [!WARNING]
> Es muy importante asegurarse de que mantiene la estructura de la sangría del archivo YAML. Cualquier cambio en la estructura del archivo YAML podría provocar que el configmap no se pudiera volver a aplicar.

```OSM Prometheus Configmap Configuration
prometheus.yml: |
    global:
      scrape_interval: 10s
      scrape_timeout: 10s
      evaluation_interval: 1m

    scrape_configs:
      - job_name: 'kubernetes-apiservers'
        kubernetes_sd_configs:
        - role: endpoints
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          # TODO need to remove this when the CA and SAN match
          insecure_skip_verify: true
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        metric_relabel_configs:
        - source_labels: [__name__]
          regex: '(apiserver_watch_events_total|apiserver_admission_webhook_rejection_count)'
          action: keep
        relabel_configs:
        - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
          action: keep
          regex: default;kubernetes;https

      - job_name: 'kubernetes-nodes'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics

      - job_name: 'kubernetes-pods'
        kubernetes_sd_configs:
        - role: pod
        metric_relabel_configs:
        - source_labels: [__name__]
          regex: '(envoy_server_live|envoy_cluster_upstream_rq_xx|envoy_cluster_upstream_cx_active|envoy_cluster_upstream_cx_tx_bytes_total|envoy_cluster_upstream_cx_rx_bytes_total|envoy_cluster_upstream_cx_destroy_remote_with_active_rq|envoy_cluster_upstream_cx_connect_timeout|envoy_cluster_upstream_cx_destroy_local_with_active_rq|envoy_cluster_upstream_rq_pending_failure_eject|envoy_cluster_upstream_rq_pending_overflow|envoy_cluster_upstream_rq_timeout|envoy_cluster_upstream_rq_rx_reset|^osm.*)'
          action: keep
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        - source_labels: [__meta_kubernetes_namespace]
          action: replace
          target_label: source_namespace
        - source_labels: [__meta_kubernetes_pod_name]
          action: replace
          target_label: source_pod_name
        - regex: '(__meta_kubernetes_pod_label_app)'
          action: labelmap
          replacement: source_service
        - regex: '(__meta_kubernetes_pod_label_osm_envoy_uid|__meta_kubernetes_pod_label_pod_template_hash|__meta_kubernetes_pod_label_version)'
          action: drop
        # for non-ReplicaSets (DaemonSet, StatefulSet)
        # __meta_kubernetes_pod_controller_kind=DaemonSet
        # __meta_kubernetes_pod_controller_name=foo
        # =>
        # workload_kind=DaemonSet
        # workload_name=foo
        - source_labels: [__meta_kubernetes_pod_controller_kind]
          action: replace
          target_label: source_workload_kind
        - source_labels: [__meta_kubernetes_pod_controller_name]
          action: replace
          target_label: source_workload_name
        # for ReplicaSets
        # __meta_kubernetes_pod_controller_kind=ReplicaSet
        # __meta_kubernetes_pod_controller_name=foo-bar-123
        # =>
        # workload_kind=Deployment
        # workload_name=foo-bar
        # deplyment=foo
        - source_labels: [__meta_kubernetes_pod_controller_kind]
          action: replace
          regex: ^ReplicaSet$
          target_label: source_workload_kind
          replacement: Deployment
        - source_labels:
          - __meta_kubernetes_pod_controller_kind
          - __meta_kubernetes_pod_controller_name
          action: replace
          regex: ^ReplicaSet;(.*)-[^-]+$
          target_label: source_workload_name

      - job_name: 'smi-metrics'
        kubernetes_sd_configs:
        - role: pod
        relabel_configs:
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
          action: keep
          regex: true
        - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
          action: replace
          target_label: __metrics_path__
          regex: (.+)
        - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
          action: replace
          regex: ([^:]+)(?::\d+)?;(\d+)
          replacement: $1:$2
          target_label: __address__
        metric_relabel_configs:
        - source_labels: [__name__]
          regex: 'envoy_.*osm_request_(total|duration_ms_(bucket|count|sum))'
          action: keep
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_(\d{3})_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: response_code
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_(.*)_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: source_namespace
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_(.*)_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: source_kind
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_.*_source_name_(.*)_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: source_name
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_(.*)_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: source_pod
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_(.*)_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: destination_namespace
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_(.*)_destination_name_.*_destination_pod_.*_osm_request_total
          target_label: destination_kind
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_(.*)_destination_pod_.*_osm_request_total
          target_label: destination_name
        - source_labels: [__name__]
          action: replace
          regex: envoy_response_code_\d{3}_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_(.*)_osm_request_total
          target_label: destination_pod
        - source_labels: [__name__]
          action: replace
          regex: .*(osm_request_total)
          target_label: __name__

        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_(.*)_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: source_namespace
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_(.*)_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: source_kind
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_.*_source_name_(.*)_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: source_name
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_(.*)_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: source_pod
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_(.*)_destination_kind_.*_destination_name_.*_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: destination_namespace
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_(.*)_destination_name_.*_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: destination_kind
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_(.*)_destination_pod_.*_osm_request_duration_ms_(bucket|sum|count)
          target_label: destination_name
        - source_labels: [__name__]
          action: replace
          regex: envoy_source_namespace_.*_source_kind_.*_source_name_.*_source_pod_.*_destination_namespace_.*_destination_kind_.*_destination_name_.*_destination_pod_(.*)_osm_request_duration_ms_(bucket|sum|count)
          target_label: destination_pod
        - source_labels: [__name__]
          action: replace
          regex: .*(osm_request_duration_ms_(bucket|sum|count))
          target_label: __name__

      - job_name: 'kubernetes-cadvisor'
        scheme: https
        tls_config:
          ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        kubernetes_sd_configs:
        - role: node
        metric_relabel_configs:
        - source_labels: [__name__]
          regex: '(container_cpu_usage_seconds_total|container_memory_rss)'
          action: keep
        relabel_configs:
        - action: labelmap
          regex: __meta_kubernetes_node_label_(.+)
        - target_label: __address__
          replacement: kubernetes.default.svc:443
        - source_labels: [__meta_kubernetes_node_name]
          regex: (.+)
          target_label: __metrics_path__
          replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
```

Aplique el archivo YAML de configmap actualizado con el siguiente comando.

```azurecli-interactive
kubectl apply -f cm-stable-prometheus-server.yml
```

```Output
configmap/stable-prometheus-server configured
```

> [!NOTE]
> Es posible que reciba un mensaje que indica que falta una anotación kubernetes necesaria. Esto puede pasarse por alto por ahora.

#### <a name="verify-prometheus-is-configured-to-scrape-the-osm-mesh-and-api-endpoints"></a>Comprobación de que Prometheus está configurado para extraer los puntos de conexión de la API y la malla de OSM

Para comprobar que Prometheus está correctamente configurado para extraer los puntos de conexión de la API y la malla de OSM, haremos un reenvío de puertos al pod de Prometheus y veremos la configuración de destino. Ejecute los comandos siguientes:

```azurecli-interactive
PROM_POD_NAME=$(kubectl get pods -l "app=prometheus,component=server" -o jsonpath="{.items[0].metadata.name}")
kubectl --namespace <promNamespace> port-forward $PROM_POD_NAME 9090
```

Abra un explorador en `http://localhost:9090/targets`.

Si se desplaza hacia abajo, debería ver que el estado de los puntos de conexión de la métrica de SMI es **UP**, así como otras métricas de OSM definidas como se muestra a continuación.

![Imagen de la IU de métricas de destino de Prometheus de OSM](./media/aks-osm-addon/osm-prometheus-smi-metrics-target-scrape.png)

### <a name="deploy-and-configure-a-grafana-instance-for-osm&quot;></a>Implementación y configuración de una instancia de Grafana para OSM

Usaremos Helm para implementar la instancia de Grafana. Ejecute los siguientes comandos para instalar Grafana a través de Helm:

```
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
helm install osm-grafana grafana/grafana
```

A continuación, recuperaremos la contraseña predeterminada de Grafana para iniciar sesión en el sitio de Grafana.

```azurecli-interactive
kubectl get secret --namespace default osm-grafana -o jsonpath=&quot;{.data.admin-password}&quot; | base64 --decode ; echo
```

Anote la contraseña de Grafana.

A continuación, recuperaremos el pod de Grafana para hacer un reenvío de puertos al panel de Grafana para iniciar sesión.

```azurecli-interactive
GRAF_POD_NAME=$(kubectl get pods -l &quot;app.kubernetes.io/name=grafana&quot; -o jsonpath=&quot;{.items[0].metadata.name}")
kubectl port-forward $GRAF_POD_NAME 3000
```

Abra un explorador en `http://localhost:3000`.

En la pantalla de inicio de sesión que se muestra a continuación, escriba **admin** como nombre de usuario y use la contraseña de Grafana capturada anteriormente.

![Imagen de IU de la página de inicio de sesión de Grafana de OSM](./media/aks-osm-addon/osm-grafana-ui-login.png)

#### <a name="configure-the-grafana-prometheus-data-source"></a>Configuración del origen de datos de Prometheus en Grafana

Una vez que haya iniciado sesión correctamente en Grafana, el siguiente paso es agregar Prometheus como orígenes de datos para Grafana. Para ello, vaya al icono de configuración en el menú de la izquierda y seleccione Orígenes de datos, como se muestra a continuación.

![Imagen de la interfaz de usuario de la página de orígenes de datos de Grafana de OSM](./media/aks-osm-addon/osm-grafana-ui-datasources.png)

Haga clic en el botón **Agregar origen de datos** y seleccione Prometheus en las bases de datos de serie temporal.

![Imagen de la interfaz de usuario de la página de selección de orígenes de datos de Grafana de OSM](./media/aks-osm-addon/osm-grafana-ui-datasources-select-prometheus.png)

En la página para **configurar el origen de datos de Prometheus**, escriba el FQDN del clúster de Kubernetes para el servicio Prometheus para la configuración de la dirección URL de HTTP. El FQDN predeterminado debe ser `stable-prometheus-server.default.svc.cluster.local`. Una vez que haya escrito ese punto de conexión de servicio Prometheus, desplácese hasta la parte inferior de la página y seleccione **Guardar y probar**. Debe recibir una casilla verde que indica que el origen de datos funciona.

#### <a name="importing-osm-dashboards"></a>Importación de paneles de OSM

Los paneles de OSM están disponibles a través de:

- [Nuestro repositorio](https://github.com/grafana/grafana), y son importables como blobs de JSON a través del portal de administración web;
- o [en línea en Grafana.com](https://grafana.com/grafana/dashboards/14145).

Para importar un panel, busque el signo `+` en el menú de la izquierda y seleccione `import`.
Puede importar directamente el panel por su identificador en `Grafana.com`. Por ejemplo, el panel `OSM Mesh Details` usa el identificador `14145`, puede usar el identificador directamente en el formulario y seleccionar `import`:

![Imagen de la IU de importar página del panel de Grafana de OSM](./media/aks-osm-addon/osm-grafana-dashboard-import.png)

En cuanto seleccione importar, se le llevará automáticamente al panel importado.

![Imagen de IU de la página de detalles de malla del panel de Grafana de OSM](./media/aks-osm-addon/osm-grafana-mesh-dashboard-details.png)

### <a name="deploy-and-configure-a-jaeger-operator-on-kubernetes-for-osm"></a>Implementación y configuración de un operador de Jaeger en Kubernetes para OSM

[Jaeger](https://www.jaegertracing.io/) es un sistema de seguimiento de código abierto que se usa para supervisar sistemas distribuidos y solucionar sus problemas. Se puede implementar con OSM como una nueva instancia o puede traer su propia instancia. Las instrucciones siguientes implementan una nueva instancia de Jaeger en el espacio de nombres `jaeger` del clúster de AKS.

#### <a name="deploy-jaeger-to-the-aks-cluster"></a>Implementación de Jaeger en el clúster de AKS

Aplique el siguiente manifiesto para instalar Jaeger:

```azurecli-interactive
kubectl apply -f - <<EOF
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger
  namespace: jaeger
  labels:
    app: jaeger
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jaeger
  template:
    metadata:
      labels:
        app: jaeger
    spec:
      containers:
        - name: jaeger
          image: jaegertracing/all-in-one
          args:
          - --collector.zipkin.host-port=9411
          imagePullPolicy: IfNotPresent
          ports:
          - containerPort: 9411
          resources:
            limits:
              cpu: 500m
              memory: 512M
            requests:
              cpu: 100m
              memory: 256M
---
kind: Service
apiVersion: v1
metadata:
  name: jaeger
  namespace: jaeger
  labels:
    app: jaeger
spec:
  selector:
    app: jaeger
  ports:
  - protocol: TCP
    # Service port and target port are the same
    port: 9411
  type: ClusterIP
EOF
```

```Output
deployment.apps/jaeger created
service/jaeger created
```

#### <a name="enable-tracing-for-the-osm-add-on"></a>Habilitación del seguimiento para el complemento OSM

A continuación, deberá habilitar el seguimiento para el complemento OSM.

> [!NOTE]
> Las propiedades de seguimiento no son visibles en el configmap osm-config en este momento. Esto se hará visible en una nueva versión del complemento OSM de AKS.

Ejecute el comando siguiente para habilitar el seguimiento para el complemento OSM:

```azurecli-interactive
kubectl patch configmap osm-config -n kube-system -p '{"data":{"tracing_enable":"true", "tracing_address":"jaeger.jaeger.svc.cluster.local", "tracing_port":"9411", "tracing_endpoint":"/api/v2/spans"}}' --type=merge
```

```Output
configmap/osm-config patched
```

#### <a name="view-the-jaeger-ui-with-port-forwarding"></a>Visualización de la interfaz de usuario de Jaeger con el reenvío de puertos

La interfaz de usuario de Jaeger se está ejecutando en el puerto 16686. Para ver la interfaz de usuario web, puede usar el reenvío de puertos de kubectl:

```azurecli-interactive
JAEGER_POD=$(kubectl get pods -n jaeger --no-headers  --selector app=jaeger | awk 'NR==1{print $1}')
kubectl port-forward -n jaeger $JAEGER_POD  16686:16686
http://localhost:16686/
```

En el explorador, debería ver un desplegable de servicio, que le permite seleccionar entre las distintas aplicaciones implementadas por el servicio bookstore de demostración. Seleccione un servicio para ver todos sus intervalos. Por ejemplo, si selecciona bookbuyer con una retrospectiva de una hora, puede ver sus interacciones con bookstore-v1 y bookstore-v2 ordenadas por hora.

![Imagen de IU de página de seguimiento de Jaeger de OSM](./media/aks-osm-addon/osm-jaeger-trace-view-ui.png)

Seleccione cualquier elemento para verlo con más detalle. Seleccione varios elementos para comparar los seguimientos. Por ejemplo, puede comparar las interacciones de bookbuyer con bookstore y bookstore-v2 en un momento determinado en el tiempo.

También puede seleccionar la pestaña de arquitectura del sistema para ver un gráfico de cómo las distintas aplicaciones han estado interactuando o comunicándose. Esto proporciona una idea de cómo fluye el tráfico entre las aplicaciones.

![Imagen de IU de arquitectura de sistema de Jaeger de OSM](./media/aks-osm-addon/osm-jaeger-sys-arc-view-ui.png)

## <a name="open-service-mesh-osm-aks-add-on-troubleshooting-guides"></a>Guías de solución de problemas del complemento Open Service Mesh (OSM) de AKS

Al implementar el complemento OSM de AKS, ocasionalmente puede experimentar algún problema. Las guías siguientes le ayudarán a solucionar errores y resolver problemas comunes.

### <a name="verifying-and-troubleshooting-osm-components"></a>Comprobación y solución de problemas de componentes de OSM

#### <a name="check-osm-controller-deployment"></a>Comprobación de la implementación del controlador de OSM

```azurecli-interactive
kubectl get deployment -n kube-system --selector app=osm-controller
```

Un controlador de OSM correcto tendría el siguiente aspecto:

```Output
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
osm-controller   1/1     1            1           59m
```

#### <a name="check-the-osm-controller-pod"></a>Comprobación del pod del controlador de OSM

```azurecli-interactive
kubectl get pods -n kube-system --selector app=osm-controller
```

Un pod de OSM correcto tendría el siguiente aspecto:

```Output
NAME                            READY   STATUS    RESTARTS   AGE
osm-controller-b5bd66db-wglzl   0/1     Evicted   0          61m
osm-controller-b5bd66db-wvl9w   1/1     Running   0          31m
```

A pesar de que un controlador fue expulsado en algún momento, tenemos otro con el estado READY 1/1 y en ejecución con 0 reinicios. Si el valor de la columna READY es distinto de 1/1, la malla del servicio estaría en un estado interrumpido.
La columna READY con 0/1 indica que el contenedor del plano de control se está bloqueando: necesitamos obtener registros. Consulte la sección de obtención de registros del controlador de OSM del centro de soporte técnico de Azure a continuación. Una columna READY con un número mayor que 1 después de / indicaría que hay sidecars instalados. Lo más probable es que el controlador de OSM no funcione con los sidecars conectados a él.

> [!NOTE]
> A partir de la versión v0.8.2, el controlador de OSM no está en modo de alta disponibilidad y se ejecutará en una implementación con un número de réplicas de 1(pod individual). El pod tiene sondeos de estado y el kubelet lo reiniciará si es necesario.

#### <a name="check-osm-controller-service"></a>Comprobación del servicio de controlador de OSM

```azurecli-interactive
kubectl get service -n kube-system osm-controller
```

Un servicio de controlador de OSM correcto tendría el siguiente aspecto:

```Output
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
osm-controller   ClusterIP   10.0.31.254   <none>        15128/TCP,9092/TCP   67m
```

> [!NOTE]
> CLUSTER-IP sería diferente. Los valores NAME y PORT(S) del servicio deben ser iguales al ejemplo anterior.

#### <a name="check-osm-controller-endpoints"></a>Comprobación de los puntos de conexión del controlador de OSM

```azurecli-interactive
kubectl get endpoints -n kube-system osm-controller
```

Un punto de conexión del controlador de OSM correcto tendría un aspecto similar al siguiente:

```Output
NAME             ENDPOINTS                              AGE
osm-controller   10.240.1.115:9092,10.240.1.115:15128   69m
```

#### <a name="check-osm-injector-deployment"></a>Comprobación de la implementación del inyector de OSM

```azurecli-interactive
kubectl get pod -n kube-system --selector app=osm-injector
```

Una implementación correcta del inyector de OSM tendría el siguiente aspecto:

```Output
NAME                            READY   STATUS    RESTARTS   AGE
osm-injector-5986c57765-vlsdk   1/1     Running   0          73m
```

#### <a name="check-osm-injector-pod"></a>Comprobación del pod del inyector de OSM

```azurecli-interactive
kubectl get pod -n kube-system --selector app=osm-injector
```

Un pod correcto del inyector de OSM tendría el siguiente aspecto:

```Output
NAME                            READY   STATUS    RESTARTS   AGE
osm-injector-5986c57765-vlsdk   1/1     Running   0          73m
```

#### <a name="check-osm-injector-service"></a>Comprobación del servicio del inyector de OSM

```azurecli-interactive
kubectl get service -n kube-system osm-injector
```

Un servicio correcto del inyector de OSM tendría el siguiente aspecto:

```Output
NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
osm-injector   ClusterIP   10.0.39.54   <none>        9090/TCP   75m
```

#### <a name="check-osm-endpoints"></a>Comprobación de puntos de conexión de OSM

```azurecli-interactive
kubectl get endpoints -n kube-system osm-injector
```

Un punto de conexión de OSM correcto tendría el siguiente aspecto:

```Output
NAME           ENDPOINTS           AGE
osm-injector   10.240.1.172:9090   75m
```

#### <a name="check-validating-and-mutating-webhooks"></a>Comprobación de los webhooks de validación y mutación

```azurecli-interactive
kubectl get ValidatingWebhookConfiguration --selector app=osm-controller
```

Un webhook de validación de OSM correcto tendría el siguiente aspecto:

```Output
NAME              WEBHOOKS   AGE
aks-osm-webhook-osm   1      81m
```

```azurecli-interactive
kubectl get MutatingWebhookConfiguration --selector app=osm-injector
```

Un webhook de mutación de OSM correcto tendría el siguiente aspecto:

```Output
NAME              WEBHOOKS   AGE
aks-osm-webhook-osm   1      102m
```

#### <a name="check-for-the-service-and-the-ca-bundle-of-the-validating-webhook"></a>Comprobación del servicio y el paquete de CA del webhook de validación

```azurecli-interactive
kubectl get ValidatingWebhookConfiguration aks-osm-webhook-osm -o json | jq '.webhooks[0].clientConfig.service'
```

Una configuración de webhook de validación bien configurada tendría un aspecto similar al siguiente:

```json
{
  "name": "osm-config-validator",
  "namespace": "kube-system",
  "path": "/validate-webhook",
  "port": 9093
}
```

#### <a name="check-for-the-service-and-the-ca-bundle-of-the-mutating-webhook"></a>Comprobación del servicio y el paquete de CA del webhook de mutación

```azurecli-interactive
kubectl get MutatingWebhookConfiguration aks-osm-webhook-osm -o json | jq '.webhooks[0].clientConfig.service'
```

Una configuración de webhook de mutación bien configurada tendría un aspecto similar al siguiente:

```json
{
  "name": "osm-injector",
  "namespace": "kube-system",
  "path": "/mutate-pod-creation",
  "port": 9090
}
```

#### <a name="check-whether-osm-controller-has-given-the-validating-or-mutating-webhook-a-ca-bundle"></a>Comprobación de si el controlador de OSM ha dado al webhook de validación (o de mutación) un paquete de CA

> [!NOTE]
> A partir de v0.8.2, es importante saber que el RP de AKS instala el webhook de validación y el reconciliador de AKS garantiza que existe, pero el controlador de OSM es el que rellena el paquete de CA.

```azurecli-interactive
kubectl get ValidatingWebhookConfiguration aks-osm-webhook-osm -o json | jq -r '.webhooks[0].clientConfig.caBundle' | wc -c
```

```azurecli-interactive
kubectl get MutatingWebhookConfiguration aks-osm-webhook-osm -o json | jq -r '.webhooks[0].clientConfig.caBundle' | wc -c
```

```Example Output
1845
```

Este número indica el número de bytes o el tamaño del paquete de CA. Si está vacío o es 0 o algún número por debajo de 1000, el paquete de CA no está correctamente aprovisionado. Sin un paquete de CA correcto, se producirá un error en el webhook de validación y se prohibirá que el usuario realice cambios en el ConfigMap osm-config en el espacio de nombres kube-system.

Un ejemplo de error en el que el paquete de CA es incorrecto:

- Intento de cambiar el ConfigMap osm-config:

```azurecli-interactive
kubectl patch ConfigMap osm-config -n kube-system --type merge --patch '{"data":{"config_resync_interval":"2m"}}'
```

- Error:

```
Error from server (InternalError): Internal error occurred: failed calling webhook "osm-config-webhook.k8s.io": Post https://osm-config-validator.kube-system.svc:9093/validate-webhook?timeout=30s: x509: certificate signed by unknown authority
```

Solución alternativa para cuando la configuración del webhook de **validación** tenga un certificado incorrecto:

- Opción 1 - Reiniciar el controlador de OSM: se reiniciará el controlador de OSM. En el inicio, sobrescribirá el paquete de CA de los webhooks de mutación y validación.

```azurecli-interactive
kubectl rollout restart deployment -n kube-system osm-controller
```

- Opción 2 - Eliminar el webhook de validación: al quitar el webhook de validación, las mutaciones del ConfigMap `osm-config` ya no se validan. Cualquier revisión se realizará. En algún momento, el reconciliador de AKS garantiza que el webhook de validación existe y volverá a crearlo. Es posible que sea necesario reiniciar el controlador de OSM para volver a escribir rápidamente el paquete de CA.

```azurecli-interactive
kubectl delete ValidatingWebhookConfiguration aks-osm-webhook-osm
```

- Opción 3 - Eliminación y revisión: el siguiente comando eliminará el webhook de validación, lo que nos permitirá agregar valores, e intentará aplicar una revisión de inmediato. Lo más probable es que el reconciliador de AKS no tenga tiempo suficiente para reconciliar y restaurar el webhook de validación, lo que nos permite aplicar un cambio como último recurso:

```azurecli-interactive
kubectl delete ValidatingWebhookConfiguration aks-osm-webhook-osm; kubectl patch ConfigMap osm-config -n kube-system --type merge --patch '{"data":{"config_resync_interval":"15s"}}'
```

#### <a name="check-the-osm-config-configmap"></a>Comprobacón del **ConfigMap** `osm-config`

> [!NOTE]
> El controlador de OSM no requiere que el ConfigMap `osm-config` esté presente en el espacio de nombres kube-system. El controlador tiene valores predeterminados razonables para la configuración y puede funcionar sin él.

Comprobación de la existencia:

```azurecli-interactive
kubectl get ConfigMap -n kube-system osm-config
```

Comprobación del contenido del ConfigMap osm-config

```azurecli-interactive
kubectl get ConfigMap -n kube-system osm-config -o json | jq '.data'
```

```json
{
  "egress": "true",
  "enable_debug_server": "true",
  "enable_privileged_init_container": "false",
  "envoy_log_level": "error",
  "outbound_ip_range_exclusion_list": "169.254.169.254,168.63.129.16,20.193.20.233",
  "permissive_traffic_policy_mode": "true",
  "prometheus_scraping": "false",
  "service_cert_validity_duration": "24h",
  "use_https_ingress": "false"
}
```

Valores de ConfigMap `osm-config`:

| Clave                              | Tipo   | Valores permitidos                                          | Valor predeterminado                          | Función                                                                                                                                                                                                                                |
| -------------------------------- | ------ | ------------------------------------------------------- | -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| egress                           | bool   | true, false                                             | `"false"`                              | Habilita la salida en la malla.                                                                                                                                                                                                             |
| enable_debug_server              | bool   | true, false                                             | `"true"`                               | Habilita un punto de conexión de depuración en el pod del controlador de OSM para mostrar información relacionada con la malla, como conexiones proxy, certificados y directivas de SMI.                                                                                    |
| enable_privileged_init_container | bool   | true, false                                             | `"false"`                              | Habilita los contenedores de inicialización con privilegios para pods en la malla. Cuando es false, los contenedores de inicialización solo tienen NET_ADMIN.                                                                                                                                   |
| envoy_log_level                  | string | trace, debug, info, warning, warn, error, critical, off | `"error"`                              | Establece el nivel de detalle de registro del sidecar del proxy de Envoy, solo es aplicable a los pods recién creados que se unen a la malla. Para actualizar el nivel de registro de los pods existentes, reinicie la implementación con `kubectl rollout restart`.                            |
| outbound_ip_range_exclusion_list | string | lista separada por comas de intervalos IP con la forma a.b.c.d/x | `-`                                    | Lista global de intervalos de direcciones IP que se van a excluir de la interceptación de tráfico saliente por el proxy de sidecar.                                                                                                                                    |
| permissive_traffic_policy_mode   | bool   | true, false                                             | `"false"`                              | Si se establece en `true`, se habilita el modo de permitir todo en la malla; es decir, no se aplica la directiva de tráfico en la malla. Si se establece en `false`, se habilita la directiva de tráfico de denegar todo en la malla; es decir, `SMI Traffic Target` es necesario para que los servicios se comuniquen. |
| prometheus_scraping              | bool   | true, false                                             | `"true"`                               | Habilita la extracción de métricas de Prometheus en servidores proxy de sidecar.                                                                                                                                                                                 |
| service_cert_validity_duration   | string | 24 horas, 1h30m (cualquier duración)                          | `"24h"`                                | Establece la duración de la validez del certificado de servicio, representada como una secuencia de números decimales con una fracción opcional y un sufijo de unidad.                                                                                             |
| tracing_enable                   | bool   | true, false                                             | `"false"`                              | Habilita el seguimiento de Jaeger para la malla.                                                                                                                                                                                                    |
| tracing_address                  | string | jaeger.mesh-namespace.svc.cluster.local                 | `jaeger.kube-system.svc.cluster.local` | Dirección de la implementación de Jaeger, si está habilitado el seguimiento.                                                                                                                                                                                |
| tracing_endpoint                 | string | /api/v2/spans                                           | /api/v2/spans                          | Punto de conexión para los datos de seguimiento, si está habilitado el seguimiento.                                                                                                                                                                                          |
| tracing_port                     | int    | cualquier valor entero distinto de cero                              | `"9411"`                               | Puerto en el que está habilitado el seguimiento.                                                                                                                                                                                                       |
| use_https_ingress                | bool   | true, false                                             | `"false"`                              | Habilita la entrada HTTPS en la malla.                                                                                                                                                                                                      |
| config_resync_interval           | string | por debajo de 1 minuto se deshabilita                            | 0 (deshabilitado)                           | Cuando se proporciona un valor superior a 1m (60s), el controlador de OSM enviará toda la configuración disponible a cada envío Envoy en el intervalo especificado.                                                                                                    |

#### <a name="check-namespaces"></a>Comprobación de espacios de nombres

> [!NOTE]
> El espacio de nombres espacio de nombres kube-system nunca participará en una malla de servicio y nunca se etiquetará ni anotará con la clave o los valores siguientes.

Usamos el comando `osm namespace add` para unir espacios de nombres a una malla de servicio determinada.
Cuando un espacio de nombres de K8S forma parte de la malla (o para que forme parte de la malla) debe cumplirse lo siguiente:

Vea las anotaciones con

```azurecli-interactive
kubectl get namespace bookbuyer -o json | jq '.metadata.annotations'
```

Las siguientes anotaciones deben estar presentes:

```Output
{
  "openservicemesh.io/sidecar-injection": "enabled"
}
```

Vea las etiquetas con

```azurecli-interactive
kubectl get namespace bookbuyer -o json | jq '.metadata.labels'
```

La siguiente etiqueta debe estar presente:

```Output
{
  "openservicemesh.io/monitored-by": "osm"
}
```

Si un espacio de nombres no está anotado con `"openservicemesh.io/sidecar-injection": "enabled"` o no está etiquetado con `"openservicemesh.io/monitored-by": "osm"`, el inyector de agregará sidecars de Envoy.

> Nota: Después de que se llame a `osm namespace add`, solo se insertarán los pods **nuevos** con un sidecar de Envoy. Los pods existentes deben reiniciarse con `kubectl rollout restart deployment ...`.

#### <a name="verify-the-smi-crds"></a>Compruebe las CRD de SMI:

Compruebe si el clúster tiene las CRD necesarias:

```azurecli-interactive
kubectl get crds
```

Debemos tener lo siguiente instalado en el clúster:

- httproutegroups.specs.smi-spec.io
- tcproutes.specs.smi-spec.io
- trafficsplits.split.smi-spec.io
- traffictargets.access.smi-spec.io
- udproutes.specs.smi-spec.io

Obtenga las versiones de las CRD instaladas con este comando:

```azurecli-interactive
for x in $(kubectl get crds --no-headers | awk '{print $1}' | grep 'smi-spec.io'); do
    kubectl get crd $x -o json | jq -r '(.metadata.name, "----" , .spec.versions[].name, "\n")'
done
```

Resultado esperado:

```Output
httproutegroups.specs.smi-spec.io
----
v1alpha4
v1alpha3
v1alpha2
v1alpha1


tcproutes.specs.smi-spec.io
----
v1alpha4
v1alpha3
v1alpha2
v1alpha1


trafficsplits.split.smi-spec.io
----
v1alpha2


traffictargets.access.smi-spec.io
----
v1alpha3
v1alpha2
v1alpha1


udproutes.specs.smi-spec.io
----
v1alpha4
v1alpha3
v1alpha2
v1alpha1
```

El controlador de OSM v0.8.2 requiere las siguientes versiones:

- traffictargets.access.smi-spec.io - [v1alpha3](https://github.com/servicemeshinterface/smi-spec/blob/v0.6.0/apis/traffic-access/v1alpha3/traffic-access.md)
- httproutegroups.specs.smi-spec.io - [v1alpha4](https://github.com/servicemeshinterface/smi-spec/blob/v0.6.0/apis/traffic-specs/v1alpha4/traffic-specs.md#httproutegroup)
- tcproutes.specs.smi-spec.io - [v1alpha4](https://github.com/servicemeshinterface/smi-spec/blob/v0.6.0/apis/traffic-specs/v1alpha4/traffic-specs.md#tcproute)
- udproutes.specs.smi-spec.io - No compatible
- trafficsplits.split.smi-spec.io - [v1alpha2](https://github.com/servicemeshinterface/smi-spec/blob/v0.6.0/apis/traffic-split/v1alpha2/traffic-split.md)
- \*.metrics.smi-spec.io - [v1alpha1](https://github.com/servicemeshinterface/smi-spec/blob/v0.6.0/apis/traffic-metrics/v1alpha1/traffic-metrics.md)

Si faltan CRD, use los siguientes comandos para instalarlas en el clúster:

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/v0.8.2/charts/osm/crds/access.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/v0.8.2/charts/osm/crds/specs.yaml
```

```azurecli-interactive
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/v0.8.2/charts/osm/crds/split.yaml
```

## <a name="disable-open-service-mesh-osm-add-on-for-your-aks-cluster"></a>Deshabilitación del complemento Open Service Mesh (OSM) para el clúster de AKS

Para deshabilitar el complemento de OSM, ejecute el comando siguiente:

```azurecli-interactive
az aks disable-addons -n <AKS-cluster-name> -g <AKS-resource-group-name> -a open-service-mesh
```

<!-- LINKS - internal -->

[kubernetes-service]: concepts-network.md#services
[az-feature-register]: /cli/azure/feature?view=azure-cli-latest&preserve-view=true#az_feature_register
[az-feature-list]: /cli/azure/feature?view=azure-cli-latest&preserve-view=true#az_feature_list
[az-provider-register]: /cli/azure/provider?view=azure-cli-latest&preserve-view=true#az_provider_register