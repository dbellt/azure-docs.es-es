---
title: Restricción del tráfico de salida en un clúster de Red Hat OpenShift (ARO) en Azure
description: Obtenga información sobre qué puertos y direcciones son necesarios para controlar el tráfico de salida de Red Hat OpenShift (ARO) en Azure
author: sakthi-vetrivel
ms.author: jzim
ms.service: azure-redhat-openshift
ms.topic: article
ms.date: 04/09/2021
ms.openlocfilehash: 24c4686306aff9d84fe7bf74ddfdccff987244d9
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107368564"
---
# <a name="control-egress-traffic-for-your-azure-red-hat-openshift-aro-cluster-preview"></a>Control del tráfico de salida del clúster de Red Hat OpenShift (ARO) en Azure (versión preliminar)

En este artículo se proporcionan los detalles necesarios que permiten proteger el tráfico de salida desde el clúster de Red Hat OpenShift (ARO) en Azure. Contiene los requisitos del clúster para una implementación básica de ARO y los requisitos adicionales para los componentes opcionales de Red Hat y de terceros. Al final, se proporciona un [ejemplo](#private-aro-cluster-setup) de cómo configurar estos requisitos con Azure Firewall. Tenga en cuenta que puede aplicar esta información a Azure Firewall o a cualquier dispositivo o método de restricción de salida.

## <a name="before-you-begin"></a>Antes de comenzar

En este artículo se asume que va a crear un clúster. Si necesita un clúster de ARO básico, consulte el tutorial de [inicio rápido de ARO](https://docs.microsoft.com/azure/openshift/tutorial-create-cluster).

> [!IMPORTANT]
> Las características en vista previa de ARO están disponibles como opción de participación y autoservicio. Las versiones preliminares se proporcionan "tal cual" y "como están disponibles", y están excluidas de los Acuerdos de nivel de servicio y la garantía limitada. Las versiones preliminares de ARO reciben cobertura parcial del soporte al cliente en la medida de lo posible.

## <a name="minimum-required-fqdn--application-rules"></a>Reglas de aplicación o instancias de FQDN obligatorias mínimas

Esta lista se basa en la lista de instancias de FQDN que se encuentra en la documentación de OpenShift, aquí: https://docs.openshift.com/container-platform/4.6/installing/install_config/configuring-firewall.html

Se requieren las siguientes reglas de aplicación / FQDN:

| FQDN de destino | Port | Uso |
| ----------- | ----------- | ------------- |
| **`quay.io`** | **HTTPS:443** | Obligatorio para la instalación; lo usa el clúster. El clúster lo usa para descargar las imágenes del contenedor de plataformas. |
| **`registry.redhat.io`** | **HTTPS:443** | Obligatorio para los complementos principales. El clúster lo usa para descargar los componentes principales, como las herramientas de desarrollo, complementos basados en operadores e imágenes de contenedor proporcionadas por Red Hat.
| **`mirror.openshift.com`** | **HTTPS:443** | Necesario en el entorno de VDI o en el portátil para acceder a las imágenes y al contenido de la instalación reflejados. En el clúster, se requiere para descargar firmas de versión de la plataforma a fin de saber qué imágenes deben extraerse de quay.io. |
| **`api.openshift.com`** | **HTTPS:443** | Requerido por el clúster para comprobar si hay actualizaciones disponibles antes de descargar las firmas de imagen. |
| **`arosvc.azurecr.io`** | **HTTPS:443** | Registro privado interno para operadores de ARO.  Es necesario si no permite los puntos de conexión de servicio Microsoft.ContainerRegistry en las subredes. |
| **`management.azure.com`** | **HTTPS:443** | El clúster lo usa para acceder a las API de Azure. |
| **`login.microsoftonline.com`** | **HTTPS:443** | El clúster lo usa para la autenticación en Azure. |
| **`gcs.prod.monitoring.core.windows.net`** | **HTTPS:443** | Se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente. |
| **`*.blob.core.windows.net`** | **HTTPS:443** | Se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente. |
| **`*.servicebus.windows.net`** | **HTTPS:443** | Se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente. |
| **`*.table.core.windows.net`** | **HTTPS:443** | Se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente. |

> [!NOTE] 
> Para muchos clientes que exponen *.blob, *.table y otros espacios de direcciones grandes, se crea un posible problema de filtración de datos. Puede que quiera usar el [firewall de salida de OpenShift](https://docs.openshift.com/container-platform/4.6/networking/openshift_sdn/configuring-egress-firewall.html) para proteger las aplicaciones implementadas en el clúster de forma que no lleguen a estos destinos y usar Azure Private Link para necesidades específicas de la aplicación.

---

## <a name="complete-list-of-required-and-optional-fqdns"></a>Lista completa de FQDN obligatorios y opcionales

### <a name="first-group-installing-and-downloading-packages-and-tools"></a>PRIMER GRUPO: INSTALACIÓN Y DESCARGA DE PAQUETES Y HERRAMIENTAS

- **`quay.io`** : obligatorio para la instalación; lo usa el clúster. El clúster lo usa para descargar las imágenes del contenedor de plataformas.
- **`registry.redhat.io`** : obligatorio para los complementos principales. El clúster lo usa para descargar componentes principales, como las herramientas de desarrollo, los complementos basados en operadores o imágenes de contenedor proporcionadas por Red Hat, como nuestro middleware, la Imagen Base Universal (UBI), entre otros.
- **`sso.redhat.com`** : necesario en el entorno de VDI o en el portátil para conectarse a cloud.redhat.com. Este es el sitio donde podemos descargar el secreto de extracción y usar algunas de las soluciones SaaS que se ofrecen en Red Hat para facilitar la supervisión de las suscripciones, el inventario de clústeres o los informes de contracargo, entre otros.
- **`openshift.org`** : necesario en el entorno de VDI o en el portátil para conectarse y descargar imágenes de RH CoreOS, pero en Azure se seleccionan desde Marketplace, no es necesario descargar imágenes del sistema operativo.

---

### <a name="second-group-telemetry"></a>SEGUNDO GRUPO: TELEMETRÍA

Puede rechazarse toda esta sección, pero antes de saber cómo, compruebe qué es: https://docs.openshift.com/container-platform/4.6/support/remote_health_monitoring/about-remote-health-monitoring.html
- **`cert-api.access.redhat.com`** : se usa en el entorno de VDI o en el portátil.
- **`api.access.redhat.com`** : se usa en el entorno de VDI o en el portátil.
- **`infogw.api.openshift.com`** : se usa en el entorno de VDI o en el portátil.
- **`https://cloud.redhat.com/api/ingress`** : se usa en el clúster para el operador de información que se integra con Red Hat Insights de SaaS.
En OpenShift Container Platform, los clientes pueden rechazar la notificación de información de mantenimiento y uso. Sin embargo, los clústeres conectados permiten que Red Hat reaccione de forma más rápida a los problemas y que admita mejor a nuestros clientes, además de comprender mejor cómo actualiza el producto los clústeres. Compruebe los detalles aquí: https://docs.openshift.com/container-platform/4.6/support/remote_health_monitoring/opting-out-of-remote-health-reporting.html.

---

### <a name="third-group-cloud-apis"></a>TERCER GRUPO: API DE NUBE

- **`management.azure.com`** : el clúster lo usa para acceder a las API de Azure.

---

### <a name="fourth-group-other-openshift-requirements"></a>CUARTO GRUPO: OTROS REQUISITOS DE OPENSHIFT

- **`mirror.openshift.com`** : necesario en el entorno de VDI o en el portátil para acceder a las imágenes y al contenido de la instalación reflejados; en el clúster se requiere para descargar firmas de versión de la plataforma, que el clúster usa para saber qué imágenes deben extraerse de quay.io.
- **`storage.googleapis.com/openshift-release`** : sitio alternativo para descargar firmas de versión de la plataforma que el clúster usa para saber qué imágenes deben extraerse de quay.io.
- **`*.apps.<cluster_name>.<base_domain>`** (O DIRECCIÓN URL DE ARO EQUIVALENTE): cuando se agregan dominios a la lista de permitidos, se usa en la red corporativa para acceder a las aplicaciones implementadas en OpenShift o a la consola de OpenShift.
- **`api.openshift.com`** : requerido por el clúster para comprobar si hay actualizaciones disponibles antes de descargar las firmas de imagen.
- **`registry.access.redhat.com`** : se requiere acceso al Registro en el entorno de VDI o en el portátil para descargar imágenes de desarrollo cuando se use la herramienta de la CLI de ODO. (Esta es una herramienta de la CLI alternativa para los desarrolladores que no están familiarizados con Kubernetes). https://docs.openshift.com/container-platform/4.6/cli_reference/developer_cli_odo/understanding-odo.html

---

### <a name="fifth-group-microsoft--red-hat-aro-monitoring-service"></a>QUINTO GRUPO: SERVICIO DE SUPERVISIÓN DE MICROSOFT Y ARO DE RED HAT

- **`login.microsoftonline.com`** : el clúster lo usa para la autenticación en Azure.
- **`gcs.prod.monitoring.core.windows.net`** : se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente.
- **`*.blob.core.windows.net`** : se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente.
- **`*.servicebus.windows.net`** : se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente.
- **`*.table.core.windows.net`** : se usa para la supervisión de Microsoft Geneva, para que el equipo de ARO pueda supervisar los clústeres del cliente.

## <a name="aro-integrations"></a>Integraciones de ARO

### <a name="azure-monitor-for-containers"></a>Azure Monitor para contenedores

Hay dos opciones para proporcionar a los contenedores acceso a Azure Monitor; puede permitir [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) de Azure Monitor, **o bien** proporcionar acceso a las reglas de aplicación y FQDN necesarias.  Estas son las [instrucciones](https://docs.microsoft.com/azure/azure-monitor/containers/container-insights-azure-redhat4-setup) sobre cómo agregar Azure Monitor al clúster de ARO existente.

#### <a name="required-network-rules"></a>Reglas de red obligatorias

Se requieren las siguientes reglas de aplicación / FQDN:

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
| [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags) -  **`AzureMonitor:443`**  | TCP           | 443      | Este punto de conexión se usa para enviar datos de métricas y registros a Azure Monitor y Log Analytics. |

#### <a name="required-fqdn--application-rules"></a>Reglas de aplicación o FQDN obligatorias

Los clústeres de ARO que tienen habilitado Azure Monitor para contenedores necesitan las reglas de aplicación/FQDN siguientes:

| FQDN                                    | Port      | Uso      |
|-----------------------------------------|-----------|----------|
| **`dc.services.visualstudio.com`** | **`HTTPS:443`**    | Este punto de conexión se usa para la telemetría de supervisión y las métricas mediante Azure Monitor. |
| **`*.ods.opinsights.azure.com`**    | **`HTTPS:443`**    | Azure Monitor usa este punto de conexión para la ingesta de datos de análisis de registros. |
| **`*.oms.opinsights.azure.com`** | **`HTTPS:443`** | omsagent usa este punto de conexión para autenticar el servicio de análisis de registros. |
| **`*.monitoring.azure.com`** | **`HTTPS:443`** | Este punto de conexión se usa para enviar datos de métricas a Azure Monitor. |


## <a name="private-aro-cluster-setup"></a>Configuración de un clúster privado de ARO
El objetivo es proteger el clúster de ARO mediante el enrutamiento del tráfico de salida a través de una instancia de Azure Firewall.
### <a name="before"></a>Antes:
![Antes](media/concepts-networking/aro-private.jpg)
### <a name="after"></a>Después:
![Después](media/concepts-networking/aro-fw.jpg)

## <a name="create-a-private-aro-cluster"></a>Creación de un clúster privado de ARO

### <a name="set-up-vars-for-your-environment"></a>Configuración de variables para el entorno
```bash

CLUSTER=aro-cluster # Name of your created cluster
RESOURCEGROUP=aro-rg # The name of your resource group where you created the ARO cluster
AROVNET=aro-vnet # The name of your vnet from your created ARO cluster
JUMPSUBNET=jump-subnet
LOCATION=eastus # The location where ARO cluster is deployed

```

### <a name="create-a-resource-group"></a>Crear un grupo de recursos
```bash
az group create -g "$RESOURCEGROUP" -l $LOCATION
```

### <a name="create-the-virtual-network"></a>Crear la red virtual
```bash
az network vnet create \
  -g $RESOURCEGROUP \
  -n $AROVNET \
  --address-prefixes 10.0.0.0/8
```

### <a name="add-two-empty-subnets-to-your-virtual-network"></a>Adición de dos subredes vacías a la red virtual
```bash
  az network vnet subnet create \
    -g "$RESOURCEGROUP" \
    --vnet-name $AROVNET \
    -n "$CLUSTER-master" \
    --address-prefixes 10.10.1.0/24 \
    --service-endpoints Microsoft.ContainerRegistry

  az network vnet subnet create \
    -g $RESOURCEGROUP \
    --vnet-name $AROVNET \
    -n "$CLUSTER-worker" \
    --address-prefixes 10.20.1.0/24 \
    --service-endpoints Microsoft.ContainerRegistry
```

### <a name="disable-network-policies-for-private-link-service-on-your-virtual-network-and-subnets-this-is-a-requirement-for-the-aro-service-to-access-and-manage-the-cluster"></a>Deshabilite las directivas de red para el servicio Private Link en la red virtual y las subredes. Este es un requisito para que el servicio ARO acceda al clúster y lo administre.
```bash
az network vnet subnet update \
  -g "$RESOURCEGROUP" \
  --vnet-name $AROVNET \
  -n "$CLUSTER-master" \
  --disable-private-link-service-network-policies true
```
### <a name="create-a-firewall-subnet"></a>Creación de una subred de firewall
```bash
az network vnet subnet create \
    -g "$RESOURCEGROUP" \
    --vnet-name $AROVNET \
    -n "AzureFirewallSubnet" \
    --address-prefixes 10.100.1.0/26
```

## <a name="create-a-jump-host-vm"></a>Creación de una máquina virtual de host de salto
### <a name="create-a-jump-subnet"></a>Creación de una subred de salto
```bash
  az network vnet subnet create \
    -g "$RESOURCEGROUP" \
    --vnet-name $AROVNET \
    -n $JUMPSUBNET \
    --address-prefixes 10.30.1.0/24 \
    --service-endpoints Microsoft.ContainerRegistry
```
### <a name="create-a-jump-host-vm"></a>Creación de una máquina virtual de host de salto
```bash
VMUSERNAME=aroadmin

az vm create --name ubuntu-jump \
             --resource-group $RESOURCEGROUP \
             --ssh-key-values ~/.ssh/id_rsa.pub \
             --admin-username $VMUSERNAME \
             --image UbuntuLTS \
             --subnet $JUMPSUBNET \
             --public-ip-address jumphost-ip \
             --vnet-name $AROVNET 
```

## <a name="create-an-azure-red-hat-openshift-cluster"></a>Creación de un clúster de Red Hat OpenShift en Azure
### <a name="get-a-red-hat-pull-secret-optional"></a>Obtención de un secreto de extracción de Red Hat (opcional)

Los secretos de extracción de Red Hat permiten al clúster acceder a los registros de contenedor de Red Hat, junto con otro contenido. Este paso es opcional pero recomendable.

1. **[Vaya al portal del administrador de clústeres de Red Hat OpenShift](https://cloud.redhat.com/openshift/install/azure/aro-provisioned) e inicie sesión.**

   Tendrá que iniciar sesión en su cuenta de Red Hat, o bien crear una cuenta de Red Hat con su correo electrónico empresarial y aceptar los términos y condiciones.

2. **Haga clic en Download pull secret** (Descargar secreto de incorporación de cambios).

Mantenga el archivo `pull-secret.txt` guardado en un lugar seguro, ya que se usará cada vez que se cree un clúster.

Al ejecutar el comando `az aro create`, puede hacer referencia al secreto de incorporación de cambios mediante el parámetro `--pull-secret @pull-secret.txt`. Ejecute `az aro create` desde el directorio donde haya almacenado el archivo `pull-secret.txt`. De lo contrario, reemplace `@pull-secret.txt` por `@<path-to-my-pull-secret-file`.

Si va a copiar el secreto de incorporación de cambios o hacer referencia a él en otros scripts, debe tener el formato de una cadena JSON válida.

```bash
az aro create \
  -g "$RESOURCEGROUP" \
  -n "$CLUSTER" \
  --vnet $AROVNET \
  --master-subnet "$CLUSTER-master" \
  --worker-subnet "$CLUSTER-worker" \
  --apiserver-visibility Private \
  --ingress-visibility Private \
  --pull-secret @pull-secret.txt
```

## <a name="create-an-azure-firewall"></a>Creación de una instancia de Azure Firewall

### <a name="create-a-public-ip-address"></a>Creación de una dirección IP pública
```bash
az network public-ip create -g $RESOURCEGROUP -n fw-ip --sku "Standard" --location $LOCATION
```
### <a name="update-install-azure-firewall-extension"></a>Actualización de la instalación de la extensión de Azure Firewall
```bash
az extension add -n azure-firewall
az extension update -n azure-firewall
```

### <a name="create-azure-firewall-and-configure-ip-config"></a>Creación de una instancia de Azure Firewall y configuración de IP
```bash
az network firewall create -g $RESOURCEGROUP -n aro-private -l $LOCATION
az network firewall ip-config create -g $RESOURCEGROUP -f aro-private -n fw-config --public-ip-address fw-ip --vnet-name $AROVNET

```

### <a name="capture-azure-firewall-ips-for-a-later-use"></a>Captura de direcciones IP de Azure Firewall para uso posterior
```bash
FWPUBLIC_IP=$(az network public-ip show -g $RESOURCEGROUP -n fw-ip --query "ipAddress" -o tsv)
FWPRIVATE_IP=$(az network firewall show -g $RESOURCEGROUP -n aro-private --query "ipConfigurations[0].privateIpAddress" -o tsv)

echo $FWPUBLIC_IP
echo $FWPRIVATE_IP
```

### <a name="create-a-udr-and-routing-table-for-azure-firewall"></a>Creación de una tabla UDR y de enrutamiento para Azure Firewall
```bash
az network route-table create -g $RESOURCEGROUP --name aro-udr

az network route-table route create -g $RESOURCEGROUP --name aro-udr --route-table-name aro-udr --address-prefix 0.0.0.0/0 --next-hop-type VirtualAppliance --next-hop-ip-address $FWPRIVATE_IP
```

### <a name="add-application-rules-for-azure-firewall"></a>Adición de reglas de aplicación para Azure Firewall
Regla para que OpenShift funcione de acuerdo con esta [lista](https://docs.openshift.com/container-platform/4.3/installing/install_config/configuring-firewall.html#configuring-firewall_configuring-firewall):
```bash
az network firewall application-rule create -g $RESOURCEGROUP -f aro-private \
 --collection-name 'ARO' \
 --action allow \
 --priority 100 \
 -n 'required' \
 --source-addresses '*' \
 --protocols 'http=80' 'https=443' \
 --target-fqdns 'registry.redhat.io' '*.quay.io' 'sso.redhat.com' 'management.azure.com' 'mirror.openshift.com' 'api.openshift.com' 'quay.io' '*.blob.core.windows.net' 'gcs.prod.monitoring.core.windows.net' 'registry.access.redhat.com' 'login.microsoftonline.com' '*.servicebus.windows.net' '*.table.core.windows.net' 'grafana.com'
```
Reglas opcionales para imágenes de Docker:
```bash
az network firewall application-rule create -g $RESOURCEGROUP -f aro-private \
 --collection-name 'Docker' \
 --action allow \
 --priority 200 \
 -n 'docker' \
 --source-addresses '*' \
 --protocols 'http=80' 'https=443' \
 --target-fqdns '*cloudflare.docker.com' '*registry-1.docker.io' 'apt.dockerproject.org' 'auth.docker.io'
```

### <a name="associate-aro-subnets-to-fw"></a>Asociación de subredes de ARO a FW
```bash
az network vnet subnet update -g $RESOURCEGROUP --vnet-name $AROVNET --name "$CLUSTER-master" --route-table aro-udr
az network vnet subnet update -g $RESOURCEGROUP --vnet-name $AROVNET --name "$CLUSTER-worker" --route-table aro-udr
```

## <a name="test-the-configuration-from-the-jumpbox"></a>Comprobación de la configuración desde Jumpbox
Estos pasos solo funcionan si ha agregado reglas para las imágenes de Docker. 
### <a name="configure-the-jumpbox"></a>Configuración de jumpbox
Inicie sesión en una máquina virtual de jumpbox e instale las utilidades `azure-cli`, `oc-cli` y `jq`. Para la instalación de openshift-cli, consulte el portal del cliente de Red Hat.
```bash
#Install Azure-cli
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
#Install jq
sudo apt install jq -y
```
### <a name="log-into-the-aro-cluster"></a>Inicio de sesión en el clúster de ARO
Enumere las credenciales del clúster:
```bash

# Login to Azure
az login
# Set Vars in Jumpbox
CLUSTER=aro-cluster # Name of your created cluster
RESOURCEGROUP=aro-rg # The name of your resource group where you created the ARO cluster

#Get the cluster credentials
ARO_PASSWORD=$(az aro list-credentials -n $CLUSTER -g $RESOURCEGROUP -o json | jq -r '.kubeadminPassword')
ARO_USERNAME=$(az aro list-credentials -n $CLUSTER -g $RESOURCEGROUP -o json | jq -r '.kubeadminUsername')
```
Obtenga un punto de conexión del servidor de API:
```bash
ARO_URL=$(az aro show -n $CLUSTER -g $RESOURCEGROUP -o json | jq -r '.apiserverProfile.url')
```

### <a name="download-the-oc-cli-to-the-jumpbox"></a>Descarga de la CLI de oc en jumpbox
```bash
cd ~
wget https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/openshift-client-linux.tar.gz

mkdir openshift
tar -zxvf openshift-client-linux.tar.gz -C openshift
echo 'export PATH=$PATH:~/openshift' >> ~/.bashrc && source ~/.bashrc
```

Inicie sesión con `oc login`:
```bash
oc login $ARO_URL -u $ARO_USERNAME -p $ARO_PASSWORD
```

### <a name="run-centos-to-test-outside-connectivity"></a>Ejecución de CentOS para probar la conectividad externa
Cree un pod
```bash
cat <<EOF | oc apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: centos
spec:
  containers:
  - name: centos
    image: centos
    ports:
    - containerPort: 80
    command:
    - sleep
    - "3600"
EOF
```
Una vez que el pod se esté ejecutando, aplique el comando exec en este y pruebe la conectividad externa.

```bash
oc exec -it centos -- /bin/bash
curl microsoft.com
```

## <a name="access-the-web-console-of-the-private-cluster"></a>Acceso a la consola web del clúster privado

### <a name="set-up-ssh-forwards-commands"></a>Configuración de comandos de reenvío ssh

```bash
sudo ssh -i $SSH_PATH -L 443:$CONSOLE_URL:443 aroadmin@$JUMPHOST

example:
sudo ssh -i /Users/jimzim/.ssh/id_rsa -L 443:console-openshift-console.apps.d5xm5iut.eastus.aroapp.io:443 aroadmin@104.211.18.56
```

### <a name="modify-the-etc-hosts-file-on-your-local-machine"></a>Modificación del archivo de hosts en la máquina local
```bash
##
# Host Database
#
127.0.0.1 console-openshift-console.apps.d5xm5iut.eastus.aroapp.io
127.0.0.1 oauth-openshift.apps.d5xm5iut.eastus.aroapp.io
```

### <a name="use-sshuttle-as-another-option"></a>Uso de sshuttle como opción adicional

[SSHuttle](https://github.com/sshuttle/sshuttle)


## <a name="clean-up-resources"></a>Limpieza de recursos

```bash

# Clean up the ARO cluster, vnet, firewall and jumpbox

# Remove udr from master and worker subnets first or will get error when deleting ARO cluster
az network vnet subnet update --vnet-name $AROVNET -n aro-cluster-master -g $RESOURCEGROUP --route-table aro-udr --remove routeTable
az network vnet subnet update --vnet-name $AROVNET -n aro-cluster-worker -g $RESOURCEGROUP --route-table aro-udr --remove routeTable

# Remove ARO Cluster
az aro delete -n $CLUSTER -g $RESOURCEGROUP

# Remove the resource group that contains the firewall, jumpbox and vnet
az group delete -n $RESOURCEGROUP
```