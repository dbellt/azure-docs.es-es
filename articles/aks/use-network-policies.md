---
title: Protección del tráfico de pod con la directiva de red
titleSuffix: Azure Kubernetes Service
description: Obtenga información sobre cómo proteger el tráfico que fluye dentro y fuera de los pods mediante directivas de red de Kubernetes en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 03/16/2021
ms.openlocfilehash: b05c4add0a62f07b187376d670f23179ba97f3a8
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107767446"
---
# <a name="secure-traffic-between-pods-using-network-policies-in-azure-kubernetes-service-aks"></a>Protección del tráfico entre pods mediante directivas de red en Azure Kubernetes Service (AKS)

Al ejecutar aplicaciones modernas basadas en microservicios en Kubernetes, con frecuenta querrá controlar qué componentes pueden comunicarse entre sí. El principio de privilegios mínimos debe aplicarse a la manera en que el tráfico puede fluir entre pods de un clúster de Azure Kubernetes Service (AKS). Supongamos que quiere bloquear el tráfico directamente a las aplicaciones de back-end. La característica *Directiva de red* de Kubernetes permite definir las reglas de tráfico de entrada y salida entre pods de un clúster.

En este artículo se muestra cómo instalar el motor de directiva de red y cómo crear directivas de red de Kubernetes para controlar el flujo de tráfico entre pods en AKS. La directiva de red solo se debe usar para los pods y los nodos basados en Linux de AKS.

## <a name="before-you-begin"></a>Antes de empezar

Es preciso que esté instalada y configurada la versión 2.0.61 de la CLI de Azure u otra versión posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][install-azure-cli].

> [!TIP]
> Si usó la característica de directiva de red en la versión preliminar, se recomienda que [cree un clúster](#create-an-aks-cluster-and-enable-network-policy).
> 
> Si quiere seguir usando los clústeres de prueba existentes que usaban la directiva de red en la versión preliminar, actualice el clúster a una nueva versión de Kubernetes para la versión de disponibilidad general más reciente y, luego, implemente el siguiente manifiesto de YAML para corregir el bloqueo del servidor de métricas y del panel de Kubernetes. Esta corrección solo es necesaria para los clústeres que usaban el motor de directiva de red de Calico.
>
> Como procedimiento recomendado de seguridad, [revise el contenido de este manifiesto de YAML][calico-aks-cleanup] para entender qué se implementa en el clúster de AKS.
>
> `kubectl delete -f https://raw.githubusercontent.com/Azure/aks-engine/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml`

## <a name="overview-of-network-policy"></a>Información general sobre la directiva de red

De forma predeterminada, todos los pods de un clúster de AKS pueden enviar y recibir tráfico sin limitaciones. Para mejorar la seguridad, puede definir reglas que controlen el flujo de tráfico. A menudo, las aplicaciones de back-end solo se exponen a los servicios front-end necesarios, por ejemplo. Además, los componentes de base de datos solo son accesibles para las capas de aplicación que se conectan a ellos.

La directiva de red es una especificación de Kubernetes que define las directivas de acceso para la comunicación entre pods. Con las directivas de red, puede definir un conjunto ordenado de reglas para enviar y recibir tráfico y aplicarlas a una colección de pods que coincidan con uno o varios selectores de etiqueta.

Estas reglas de las directivas de red se definen como manifiestos de YAML. Las directivas de red se pueden incluir como parte de un manifiesto más amplio que también crea una implementación o un servicio.

### <a name="network-policy-options-in-aks"></a>Opciones de directivas de red en AKS

Azure ofrece dos maneras de implementar la directiva de red. Debe elegir una opción de directiva de red al crear un clúster de AKS. La opción de directiva no se puede cambiar una vez que se ha creado el clúster:

* La implementación propia de Azure, denominada *directivas de red de Azure*.
* *Directivas de red de Calico*, una solución de seguridad de red y redes de código abierto fundada por [Tigera][tigera].

Ambas implementaciones usan *IPTables* de Linux para aplicar las directivas especificadas. Las directivas se traducen en conjuntos de pares de IP permitidas y no permitidas. Después, estos pares se programan como reglas de filtro de IPTable.

### <a name="differences-between-azure-and-calico-policies-and-their-capabilities"></a>Diferencias entre las directivas de Azure y Calico y sus funcionalidades

| Capacidad                               | Azure                      | Calico                      |
|------------------------------------------|----------------------------|-----------------------------|
| Plataformas compatibles                      | Linux                      | Linux, Windows Server 2019 (versión preliminar)  |
| Opciones de redes admitidas             | CNI de Azure                  | Azure CNI (Windows Server 2019 y Linux) y kubenet (Linux)  |
| Compatibilidad con la especificación de Kubernetes | Se admiten todos los tipos de directiva. |  Se admiten todos los tipos de directiva. |
| Características adicionales                      | None                       | Modelo de directiva extendida que consta de una directiva de red global, un conjunto de red global y un punto de conexión de host. Para más información sobre el uso de la CLI de `calicoctl` para administrar estas características extendidas, consulte la [referencia de usuario de calicoctl][calicoctl]. |
| Soporte técnico                                  | Compatible con el soporte técnico de Azure y el equipo de ingeniería | Soporte técnico de la comunidad de Calico. Para más información sobre el soporte técnico de pago adicional, consulte las [opciones de soporte técnico del proyecto Calico][calico-support]. |
| Registro                                  | Las reglas agregadas o eliminadas en IPTables se registran en cada host en */var/log/azure-npm.log* | Para más información, consulte los [registros de los componentes de Calico][calico-logs]. |

## <a name="create-an-aks-cluster-and-enable-network-policy"></a>Creación de un clúster de AKS y habilitación de la directiva de red

Para ver las directivas de red en acción, vamos a crear y luego expandir una directiva que define el flujo de tráfico:

* Deniegue todo el tráfico al pod.
* Permita el tráfico en función de las etiquetas de pod.
* Permita el tráfico según el espacio de nombres.

En primer lugar, crearemos un clúster de AKS que admite la directiva de red.

> [!IMPORTANT]
>
> La característica de directiva de red solo se puede habilitar cuando se crea el clúster. No se puede habilitar la directiva de red en un clúster de AKS existente.

Para usar la directiva de red de Azure, debe usar el [complemento CNI de Azure][azure-cni] y definir su propia red virtual y subredes. Para más información sobre cómo planear los rangos de subred requeridos, consulte la sección sobre cómo [configurar redes avanzadas][use-advanced-networking]. La directiva de red de Calico se puede usar con este mismo complemento CNI de Azure o con el complemento CNI de Kubenet.

En el ejemplo siguiente se invoca el script:

* Crea una red virtual y una subred.
* Crea una entidad de servicio de Azure Active Directory (Azure AD) para usarse con el clúster de AKS.
* Asigna permisos de *colaborador* para la entidad de servicio de clúster de AKS en la red virtual.
* Crea un clúster de AKS en la red virtual definida y habilita la directiva de red.
    * Se usa la opción de directiva de _Red de Azure_. Para usar Calico como opción de directiva de red, use el parámetro `--network-policy calico`. Nota: Calico se puede usar con `--network-plugin azure` o `--network-plugin kubenet`.

Recuerde que en lugar de una entidad de servicio, puede usar una identidad administrada para los permisos. Para más información, consulte [Uso de identidades administradas](use-managed-identity.md).

Proporcione su propia variable segura *SP_PASSWORD*. Puede reemplazar las variables *RESOURCE_GROUP_NAME* y *CLUSTER_NAME*:

```azurecli-interactive
RESOURCE_GROUP_NAME=myResourceGroup-NP
CLUSTER_NAME=myAKSCluster
LOCATION=canadaeast

# Create a resource group
az group create --name $RESOURCE_GROUP_NAME --location $LOCATION

# Create a virtual network and subnet
az network vnet create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name myVnet \
    --address-prefixes 10.0.0.0/8 \
    --subnet-name myAKSSubnet \
    --subnet-prefix 10.240.0.0/16

# Create a service principal and read in the application ID
SP=$(az ad sp create-for-rbac --output json)
SP_ID=$(echo $SP | jq -r .appId)
SP_PASSWORD=$(echo $SP | jq -r .password)

# Wait 15 seconds to make sure that service principal has propagated
echo "Waiting for service principal to propagate..."
sleep 15

# Get the virtual network resource ID
VNET_ID=$(az network vnet show --resource-group $RESOURCE_GROUP_NAME --name myVnet --query id -o tsv)

# Assign the service principal Contributor permissions to the virtual network resource
az role assignment create --assignee $SP_ID --scope $VNET_ID --role Contributor

# Get the virtual network subnet resource ID
SUBNET_ID=$(az network vnet subnet show --resource-group $RESOURCE_GROUP_NAME --vnet-name myVnet --name myAKSSubnet --query id -o tsv)
```

### <a name="create-an-aks-cluster-for-azure-network-policies"></a>Creación de un clúster de AKS para las directivas de red de Azure

Cree el clúster de AKS y especifique la red virtual, la información de la entidad de servicio y *azure* como complemento de red y directiva de red.

```azurecli
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --network-plugin azure \
    --network-policy azure
```

La operación de creación del clúster tarda unos minutos. Cuando el clúster esté listo, configure `kubectl` para conectarse a su clúster de Kubernetes con el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas:

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

### <a name="create-an-aks-cluster-for-calico-network-policies"></a>Creación de un clúster de AKS para las directivas de red de Calico

Cree el clúster de AKS y especifique la red virtual, la información de la entidad de servicio y *azure* como complemento de red y *calico* como directiva de red. Al usar *calico* como directiva de red, se habilitan las redes de Calico en los grupos de nodos de Windows y Linux.

Si planea agregar grupos de nodos de Windows al clúster, incluya los parámetros `windows-admin-username` y `windows-admin-password`, para que se cumplan los [requisitos de contraseña de Windows Server][windows-server-password]. Para usar Calico con grupos de nodos de Windows, también debe registrar `Microsoft.ContainerService/EnableAKSWindowsCalico`.

Registro de `EnableAKSWindowsCalico` la marca de característica con el comando de [característica de registro az][az-feature-register], tal como se muestra en el siguiente ejemplo:

```azurecli-interactive
az feature register --namespace "Microsoft.ContainerService" --name "EnableAKSWindowsCalico"
```

 Puede comprobar el estado de registro con el comando [az feature list][az-feature-list]:

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/EnableAKSWindowsCalico')].{Name:name,State:properties.state}"
```

Cuando todo esté listo, actualice el registro del proveedor de recursos *Microsoft.ContainerService* con el comando [az provider register][az-provider-register]:

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

> [!IMPORTANT]
> En estos momentos, el uso de directivas de red de Calico con nodos de Windows está disponible en los clústeres nuevos que usan la versión 1.20 o posterior de Kubernetes con Calico 3.17.2 y requiere el uso de redes de Azure CNI. Los nodos de Windows en los clústeres de AKS con Calico habilitado también tienen habilitado [Direct Server Return (DSR)][dsr] de manera predeterminada.
>
> En el caso de los clústeres que solo tienen grupos de nodos de Linux que ejecutan Kubernetes 1.20 con versiones anteriores de Calico, la versión de Calico se actualizará automáticamente a la versión 3.17.2.

Las directivas de red de Calico con nodos de Windows se encuentran actualmente en versión preliminar.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

Cree un nombre de usuario para usarlo como credenciales de administrador para los contenedores de Windows Server en el clúster. Los comandos siguientes le solicitan un nombre de usuario y lo establecen en WINDOWS_USERNAME para su uso en un comando posterior (recuerde que los comandos de este artículo se incluyen en un shell de BASH).

```azurecli-interactive
echo "Please enter the username to use as administrator credentials for Windows Server containers on your cluster: " && read WINDOWS_USERNAME
```

```azurecli
az aks create \
    --resource-group $RESOURCE_GROUP_NAME \
    --name $CLUSTER_NAME \
    --node-count 1 \
    --generate-ssh-keys \
    --service-cidr 10.0.0.0/16 \
    --dns-service-ip 10.0.0.10 \
    --docker-bridge-address 172.17.0.1/16 \
    --vnet-subnet-id $SUBNET_ID \
    --service-principal $SP_ID \
    --client-secret $SP_PASSWORD \
    --windows-admin-username $WINDOWS_USERNAME \
    --vm-set-type VirtualMachineScaleSets \
    --kubernetes-version 1.20.2 \
    --network-plugin azure \
    --network-policy calico
```

La operación de creación del clúster tarda unos minutos. De manera predeterminada, el clúster se crea solo con un grupo de nodos de Linux. Si desea usar grupos de nodos de Windows, puede agregarlos. Por ejemplo:

```azurecli
az aks nodepool add \
    --resource-group $RESOURCE_GROUP_NAME \
    --cluster-name $CLUSTER_NAME \
    --os-type Windows \
    --name npwin \
    --node-count 1
```

Cuando el clúster esté listo, configure `kubectl` para conectarse a su clúster de Kubernetes con el comando [az aks get-credentials][az-aks-get-credentials]. Con este comando se descargan las credenciales y se configura la CLI de Kubernetes para usarlas:

```azurecli-interactive
az aks get-credentials --resource-group $RESOURCE_GROUP_NAME --name $CLUSTER_NAME
```

## <a name="deny-all-inbound-traffic-to-a-pod"></a>Denegación de todo el tráfico entrante a un pod

Antes de definir reglas para permitir el tráfico de red específico, primero cree una directiva de red para denegar todo el tráfico. Esta directiva proporciona un punto inicial para empezar a crear una lista de permitidos solo para el tráfico deseado. También puede ver claramente que el tráfico se descarta cuando se aplica la directiva de red.

En lo que respecta al entorno de aplicación de ejemplo y a las reglas de tráfico, vamos a crear un espacio de nombres denominado *development* para ejecutar los pods de ejemplo:

```console
kubectl create namespace development
kubectl label namespace/development purpose=development
```

Cree un pod de back-end de ejemplo que ejecute NGINX. Este pod de back-end puede usarse para simular una aplicación web de back-end de ejemplo. Cree este pod en el espacio de nombres *development* y abra el puerto *80* para atender el tráfico web. Etiquete el pod con *app=webapp,role=backend* para que podamos utilizarlo como destino con una directiva de red en la sección siguiente:

```console
kubectl run backend --image=mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine --labels app=webapp,role=backend --namespace development --expose --port 80
```

Cree otro pod y asocie una sesión de terminal para probar que puede llegar correctamente a la página web de NGINX predeterminada:

```console
kubectl run --rm -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 network-policy --namespace development
```

Cuando se encuentre en el símbolo del sistema del shell, use `wget` para confirmar que puede acceder a la página web de NGINX predeterminada:

```console
wget -qO- http://backend
```

La siguiente salida de ejemplo muestra que se devuelve la página web de NGINX predeterminada:

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Salga de la sesión de terminal asociada. El pod de prueba se elimina automáticamente.

```console
exit
```

### <a name="create-and-apply-a-network-policy"></a>Creación y aplicación de una directiva de red

Ahora que ha confirmado que usar la página web de NGINX básica en el pod de back-end de ejemplo, cree una directiva de red para denegar todo el tráfico. Cree un archivo denominado `backend-policy.yaml` y pegue el siguiente manifiesto de YAML. Este manifiesto usa un elemento *podSelector* para asociar la directiva a los pods que tienen la etiqueta *app:webapp,role:backend*, como el pod de NGINX de ejemplo. No hay reglas definidas en *ingress*, por lo que se deniega todo el tráfico entrante en el pod:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress: []
```

Vaya a [https://shell.azure.com](https://shell.azure.com) para abrir Azure Cloud Shell en el explorador.

Aplique la directiva de red mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-network-policy"></a>Prueba de la directiva de red

Comprobemos que puede usar la página web de NGINX en el pod de back-end de nuevo. Cree otro pod de prueba y asocie una sesión de terminal:

```console
kubectl run --rm -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 network-policy --namespace development
```

Cuando se encuentre en el símbolo del sistema del shell, use `wget` para ver si puede acceder a la página web de NGINX predeterminada. Esta vez, establezca un valor de tiempo de expiración de *2* segundos. Ahora, la directiva de red bloquea todo el tráfico entrante, por lo que no se puede cargar la página, tal como se muestra en el ejemplo siguiente:

```console
wget -qO- --timeout=2 http://backend
```

```output
wget: download timed out
```

Salga de la sesión de terminal asociada. El pod de prueba se elimina automáticamente.

```console
exit
```

## <a name="allow-inbound-traffic-based-on-a-pod-label"></a>Tráfico entrante permitido en función de una etiqueta de pod

En la sección anterior, se programó un pod de NGINX de back-end y se creó una directiva de red para denegar todo el tráfico. Vamos a crear un pod de front-end y actualizar la directiva de red para permitir el tráfico desde los pods de front-end.

Actualice la directiva de red para permitir el tráfico desde los pods con las etiquetas *app:webapp,role:frontend* y en cualquier espacio de nombres. Edite el anterior archivo *backend-policy.yaml* y agregue unas reglas de entrada *matchLabels* para que el manifiesto sea similar al ejemplo siguiente:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

> [!NOTE]
> Esta directiva de red usa un elemento *namespaceSelector* y un elemento *podSelector* para la regla de entrada. La sintaxis YAML es importante para que las reglas de entrada sean aditivas. En este ejemplo, ambos elementos deben coincidir para que se aplique la regla de entrada. Puede que las versiones de Kubernetes anteriores a *1.12* no interpreten correctamente estos elementos y no restrinjan el tráfico de red según lo esperado. Para más información sobre este comportamiento, consulte [Behavior of to and from selectors][policy-rules] (Comportamiento de los selectores to y from).

Aplique la política de red actualizada mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f backend-policy.yaml
```

Programe un pod que se etiquete como *app=webapp,role=frontend* y asocie una sesión de terminal:

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace development
```

Cuando se encuentre en el símbolo del sistema del shell, use `wget` para ver si puede acceder a la página web de NGINX predeterminada:

```console
wget -qO- http://backend
```

Como la regla de entrada permite el tráfico con los pods que tengan las etiquetas *app: webapp,role: frontend*, se permite el tráfico desde el pod de front-end. La siguiente salida de ejemplo muestra que se devuelve la página web de NGINX predeterminada:

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Salga de la sesión de terminal asociada. El pod se elimina automáticamente.

```console
exit
```

### <a name="test-a-pod-without-a-matching-label"></a>Prueba de un pod sin una etiqueta coincidente

La directiva de red permite el tráfico desde los pods con la etiqueta *app: webapp,role: frontend*, pero debe denegar todo el tráfico. Vamos a realizar una prueba para ver si otro pod sin esas etiquetas puede acceder al pod de NGINX de back-end. Cree otro pod de prueba y asocie una sesión de terminal:

```console
kubectl run --rm -it --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 network-policy --namespace development
```

Cuando se encuentre en el símbolo del sistema del shell, use `wget` para ver si puede acceder a la página web de NGINX predeterminada. La directiva de red bloquea el tráfico entrante, por lo que no se puede cargar la página, tal como se muestra en el ejemplo siguiente:

```console
wget -qO- --timeout=2 http://backend
```

```output
wget: download timed out
```

Salga de la sesión de terminal asociada. El pod de prueba se elimina automáticamente.

```console
exit
```

## <a name="allow-traffic-only-from-within-a-defined-namespace"></a>Tráfico permitido solo desde dentro de un espacio de nombres definido

En los ejemplos anteriores, creó una directiva de red que denegaba todo el tráfico y, después, actualizó la directiva para permitir el tráfico desde los pods con una etiqueta específica. Otra de las cosas que se suele necesitar es limitar el tráfico a solo dentro de un espacio de nombres especificado. Si los ejemplos anteriores eran para el tráfico en un espacio de nombres *development*, cree una directiva de red que impida que el tráfico de otro espacio de nombres, como *production*, llegue a los pods.

En primer lugar, cree un espacio de nombres para simular un espacio de nombres production:

```console
kubectl create namespace production
kubectl label namespace/production purpose=production
```

Programe un pod de prueba en el espacio de nombres *production* que está etiquetado como *app=webapp,role=frontend*. Asocie una sesión de terminal:

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace production
```

Cuando se encuentre en el símbolo del sistema del shell, use `wget` para confirmar que puede acceder a la página web de NGINX predeterminada:

```console
wget -qO- http://backend.development
```

Como las etiquetas del pod coinciden con lo que actualmente está permitido en la directiva de red, se permite el tráfico. La directiva de red no analiza los espacios de nombres, solo las etiquetas de pod. La siguiente salida de ejemplo muestra que se devuelve la página web de NGINX predeterminada:

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Salga de la sesión de terminal asociada. El pod de prueba se elimina automáticamente.

```console
exit
```

### <a name="update-the-network-policy"></a>Actualización de la directiva de red

Vamos a actualizar la sección *namespaceSelector* de la regla de entrada para permitir únicamente el tráfico desde el espacio de nombres *development*. Edite el archivo de manifiesto *backend-policy.yaml*, como se muestra en el ejemplo siguiente:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-policy
  namespace: development
spec:
  podSelector:
    matchLabels:
      app: webapp
      role: backend
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          purpose: development
      podSelector:
        matchLabels:
          app: webapp
          role: frontend
```

En ejemplos más complejos, podría definir varias reglas de entrada, como un elemento *namespaceSelector* y, después, un elemento *podSelector*.

Aplique la política de red actualizada mediante el comando [kubectl apply][kubectl-apply] y especifique el nombre del manifiesto de YAML:

```console
kubectl apply -f backend-policy.yaml
```

### <a name="test-the-updated-network-policy"></a>Prueba de la directiva de red actualizada

Programe otro pod en el espacio de nombres *production* y asocie una sesión de terminal:

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace production
```

Una vez en el símbolo del sistema del shell, use `wget` para ver que ahora la directiva de red deniega el tráfico:

```console
wget -qO- --timeout=2 http://backend.development
```

```output
wget: download timed out
```

Salga del pod de prueba:

```console
exit
```

Con el tráfico denegado desde el espacio de nombres *production*, programe de nuevo un pod de prueba en el espacio de nombres *development* y asocie una sesión de terminal:

```console
kubectl run --rm -it frontend --image=mcr.microsoft.com/aks/fundamental/base-ubuntu:v0.0.11 --labels app=webapp,role=frontend --namespace development
```

Una vez en el símbolo del sistema del shell, use `wget` para ver que la directiva de red permite el tráfico:

```console
wget -qO- http://backend
```

El tráfico se permite porque el pod está programado en el espacio de nombres que coincide con lo permitido en la directiva de red. En la siguiente salida de ejemplo se muestra que se devuelve la página web de NGINX predeterminada:

```output
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
[...]
```

Salga de la sesión de terminal asociada. El pod de prueba se elimina automáticamente.

```console
exit
```

## <a name="clean-up-resources"></a>Limpieza de recursos

En este artículo, hemos creado dos espacios de nombres y hemos aplicado una directiva de red. Para limpiar estos recursos, use el comando [kubectl delete][kubectl-delete] y especifique los nombres de recurso:

```console
kubectl delete namespace production
kubectl delete namespace development
```

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre los recursos red, consulte [Conceptos de redes de aplicaciones en Azure Kubernetes Service (AKS)][concepts-network].

Para más información sobre las directivas, consulte las [directivas de red de Kubernetes][kubernetes-network-policies].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubernetes-network-policies]: https://kubernetes.io/docs/concepts/services-networking/network-policies/
[azure-cni]: https://github.com/Azure/azure-container-networking/blob/master/docs/cni.md
[policy-rules]: https://kubernetes.io/docs/concepts/services-networking/network-policies/#behavior-of-to-and-from-selectors
[aks-github]: https://github.com/azure/aks/issues
[tigera]: https://www.tigera.io/
[calicoctl]: https://docs.projectcalico.org/reference/calicoctl/
[calico-support]: https://www.tigera.io/tigera-products/calico/
[calico-logs]: https://docs.projectcalico.org/maintenance/troubleshoot/component-logs
[calico-aks-cleanup]: https://github.com/Azure/aks-engine/blob/master/docs/topics/calico-3.3.1-cleanup-after-upgrade.yaml

<!-- LINKS - internal -->
[install-azure-cli]: /cli/azure/install-azure-cli
[use-advanced-networking]: configure-azure-cni.md
[az-aks-get-credentials]: /cli/azure/aks#az_aks_get_credentials
[concepts-network]: concepts-network.md
[az-feature-register]: /cli/azure/feature#az_feature_register
[az-feature-list]: /cli/azure/feature#az_feature_list
[az-provider-register]: /cli/azure/provider#az_provider_register
[windows-server-password]: /windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference
[az-extension-add]: /cli/azure/extension#az_extension_add
[az-extension-update]: /cli/azure/extension#az_extension_update
[dsr]: ../load-balancer/load-balancer-multivip-overview.md#rule-type-2-backend-port-reuse-by-using-floating-ip