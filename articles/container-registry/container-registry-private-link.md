---
title: Configuración de un punto de conexión privado con Private Link
description: Configure un punto de conexión privado en un registro de contenedor y habilite un vínculo privado en una red virtual local. El acceso de vínculo privado es una característica del nivel de servicio Premium.
ms.topic: article
ms.date: 03/31/2021
ms.openlocfilehash: d3c7c573b0ffc08a85f5cbe5cc62d3f7c052f0af
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107781440"
---
# <a name="connect-privately-to-an-azure-container-registry-using-azure-private-link"></a>Conexión privada a un registro de contenedor de Azure mediante Azure Private Link


Limite el acceso a un registro mediante la asignación de direcciones IP privadas de red virtual a los puntos de conexión del registro y el uso de [Azure Private Link](../private-link/private-link-overview.md). El tráfico de red entre los clientes de la red virtual y los puntos de conexión privados del registro atraviesa la red virtual y un vínculo privado de la red troncal de Microsoft, lo que elimina la exposición a la red pública de Internet. Azure Private Link también permite acceder al registro privado desde el entorno local utilizando el emparejamiento privado de [Azure ExpressRoute](../expressroute/expressroute-introduction.MD) o una [puerta de enlace de VPN](../vpn-gateway/vpn-gateway-about-vpngateways.md).

Puede [establecer la configuración DNS](../private-link/private-endpoint-overview.md#dns-configuration) de los puntos de conexión privados del registro, de modo que la configuración se resuelva en la dirección IP privada asignada del registro. Con la configuración DNS, los clientes y servicios de la red pueden seguir accediendo al registro en el nombre de dominio completo de este, como *myregistry.azurecr.io*. 

Esta característica está disponible en el nivel de servicio de un registro de contenedor **Premium**. Actualmente, se puede configurar un máximo de 10 puntos de conexión privados para un registro. Para obtener información sobre los límites y niveles de servicio de registro, consulte [Niveles de Azure Container Registry](container-registry-skus.md).

[!INCLUDE [container-registry-scanning-limitation](../../includes/container-registry-scanning-limitation.md)]

## <a name="prerequisites"></a>Requisitos previos

* Para usar los pasos de la CLI de Azure de este artículo, se recomienda la versión 2.6.0 o posterior de la CLI de Azure. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure][azure-cli]. O bien ejecute en [Azure Cloud Shell](../cloud-shell/quickstart.md).
* Si aún no tiene un registro de contenedor, créelo (se requiere el nivel Premium) e [importe](container-registry-import-images.md) una imagen pública de ejemplo, como `mcr.microsoft.com/hello-world` desde el Registro de contenedor de Microsoft. Por ejemplo, use [Azure Portal][quickstart-portal] o la [CLI de Azure][quickstart-cli] para crear un registro.
* Si quiere configurar el acceso al Registro mediante un vínculo privado en otra suscripción de Azure, tiene que registrar el proveedor de recursos para Azure Container Registry en esa suscripción. Por ejemplo:

  ```azurecli
  az account set --subscription <Name or ID of subscription of private link>

  az provider register --namespace Microsoft.ContainerRegistry
  ``` 

En los ejemplos de la CLI de Azure de este artículo se usan las siguientes variables de entorno. Sustituya los valores adecuados para el entorno. Todos los ejemplos tienen el formato del shell de Bash:

```bash
REGISTRY_NAME=<container-registry-name>
REGISTRY_LOCATION=<container-registry-location> # Azure region such as westeurope where registry created
RESOURCE_GROUP=<resource-group-name>
VM_NAME=<virtual-machine-name>
```

[!INCLUDE [Set up Docker-enabled VM](../../includes/container-registry-docker-vm-setup.md)]

## <a name="set-up-private-link---cli"></a>Configuración de vínculo privado: CLI

### <a name="get-network-and-subnet-names"></a>Obtención de nombres de red y subred

Si aún no los tiene, necesita los nombres de una red virtual y una subred para configurar un vínculo privado. En este ejemplo se usa la misma subred para la máquina virtual y el punto de conexión privado del registro. Pero en muchos escenarios se configuraría el punto de conexión en una subred independiente. 

Al crear una máquina virtual, Azure crea de manera predeterminada crea una red virtual en el mismo grupo de recursos. El nombre de dicha red virtual se basa en el nombre de la máquina virtual. Por ejemplo, si asigna a la máquina virtual el nombre *myDockerVM*, el nombre predeterminado de la red virtual es *myDockerVMVNET*, con una subred llamada *myDockerVMSubnet*. Establezca estos valores en variables de entorno al ejecutar el comando [az network vnet list][az-network-vnet-list]:

```azurecli
NETWORK_NAME=$(az network vnet list \
  --resource-group $RESOURCE_GROUP \
  --query '[].{Name: name}' --output tsv)

SUBNET_NAME=$(az network vnet list \
  --resource-group $RESOURCE_GROUP \
  --query '[].{Subnet: subnets[0].name}' --output tsv)

echo NETWORK_NAME=$NETWORK_NAME
echo SUBNET_NAME=$SUBNET_NAME
```

### <a name="disable-network-policies-in-subnet"></a>Deshabilitación de directivas de red en subred

[Deshabilite las directivas de red](../private-link/disable-private-endpoint-network-policy.md), como los grupos de seguridad de red de la subred, para el punto de conexión privado. Actualice la configuración de subred con [az network vnet subnet update][az-network-vnet-subnet-update]:

```azurecli
az network vnet subnet update \
 --name $SUBNET_NAME \
 --vnet-name $NETWORK_NAME \
 --resource-group $RESOURCE_GROUP \
 --disable-private-endpoint-network-policies
```

### <a name="configure-the-private-dns-zone"></a>Configuración de la zona DNS privada

Cree una [zona DNS privada de Azure](../dns/private-dns-privatednszone.md) para el dominio del registro de contenedor de Azure privado. En pasos posteriores se crean registros DNS para el dominio del registro dentro de esta zona DNS. Para más información, consulte [Opciones de configuración de DNS](#dns-configuration-options), más adelante en este artículo.

Para usar una zona privada con el fin de invalidar la resolución DNS predeterminada del registro de contenedor de Azure, la zona debe tener el nombre **privatelink.azurecr.io**. Ejecute el comando siguiente [az network private-dns zone create][az-network-private-dns-zone-create] para crear la zona privada:

```azurecli
az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name "privatelink.azurecr.io"
```

### <a name="create-an-association-link"></a>Creación de un vínculo de asociación

Ejecute [az network private-dns link vnet create][az-network-private-dns-link-vnet-create] para asociar la zona privada con la red virtual. En este ejemplo se crea un vínculo denominado *myDNSLink*.

```azurecli
az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name "privatelink.azurecr.io" \
  --name MyDNSLink \
  --virtual-network $NETWORK_NAME \
  --registration-enabled false
```

### <a name="create-a-private-registry-endpoint"></a>Creación de un punto de conexión de registro privado

En esta sección se crea el punto de conexión privado del registro en la red virtual. En primer lugar, obtenga el identificador de recurso del registro:

```azurecli
REGISTRY_ID=$(az acr show --name $REGISTRY_NAME \
  --query 'id' --output tsv)
```

Ejecute el comando [az network private-endpoint create][az-network-private-endpoint-create] para crear el punto de conexión privado del registro.

En el ejemplo siguiente se crea el punto de conexión *myPrivateEndpoint* y la conexión de servicio *myConnection*. Para especificar un recurso de registro de contenedor para el punto de conexión, pase `--group-ids registry`:

```azurecli
az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group $RESOURCE_GROUP \
    --vnet-name $NETWORK_NAME \
    --subnet $SUBNET_NAME \
    --private-connection-resource-id $REGISTRY_ID \
    --group-ids registry \
    --connection-name myConnection
```

### <a name="get-endpoint-ip-configuration"></a>Obtención de la configuración de IP del punto de conexión

Para configurar los registros DNS, obtenga la configuración de IP del punto de conexión privado. En este ejemplo, hay dos direcciones IP privadas del registro de contenedor asociadas a la interfaz de red del punto de conexión privado: una del propio registro y otra del punto de conexión de datos del registro. 

En primer lugar, ejecute [az network private-endpoint show][az-network-private-endpoint-show] para consultar el identificador de la interfaz de red al punto de conexión privado:

```azurecli
NETWORK_INTERFACE_ID=$(az network private-endpoint show \
  --name myPrivateEndpoint \
  --resource-group $RESOURCE_GROUP \
  --query 'networkInterfaces[0].id' \
  --output tsv)
```

Los siguientes comandos [az network nic show][az-network-nic-show] obtienen las direcciones IP privadas del registro de contenedor y el punto de conexión de datos del registro:

```azurecli
REGISTRY_PRIVATE_IP=$(az network nic show \
  --ids $NETWORK_INTERFACE_ID \
  --query "ipConfigurations[?privateLinkConnectionProperties.requiredMemberName=='registry'].privateIpAddress" \
  --output tsv)

DATA_ENDPOINT_PRIVATE_IP=$(az network nic show \
  --ids $NETWORK_INTERFACE_ID \
  --query "ipConfigurations[?privateLinkConnectionProperties.requiredMemberName=='registry_data_$REGISTRY_LOCATION'].privateIpAddress" \
  --output tsv)

# An FQDN is associated with each IP address in the IP configurations

REGISTRY_FQDN=$(az network nic show \
  --ids $NETWORK_INTERFACE_ID \
  --query "ipConfigurations[?privateLinkConnectionProperties.requiredMemberName=='registry'].privateLinkConnectionProperties.fqdns" \
  --output tsv)

DATA_ENDPOINT_FQDN=$(az network nic show \
  --ids $NETWORK_INTERFACE_ID \
  --query "ipConfigurations[?privateLinkConnectionProperties.requiredMemberName=='registry_data_$REGISTRY_LOCATION'].privateLinkConnectionProperties.fqdns" \
  --output tsv)
```

> [!NOTE]
> Si el registro es [con replicación geográfica](container-registry-geo-replication.md), consulte el punto de conexión de datos adicional para cada réplica del registro.

### <a name="create-dns-records-in-the-private-zone"></a>Creación de registros DNS en la zona privada

Los siguientes comandos crean registros DNS en la zona privada para el punto de conexión del registro y su punto de conexión de datos. Por ejemplo, si tiene un registro denominado *myregistry* en la región *westeurope*, los nombres de punto de conexión son `myregistry.azurecr.io` y `myregistry.westeurope.data.azurecr.io`. 

> [!NOTE]
> Si el registro es [con replicación geográfica](container-registry-geo-replication.md), cree registros DNS adicionales para la dirección IP del punto de conexión de datos de la réplica.

En primer lugar, ejecute [az network private-dns record-set a create][az-network-private-dns-record-set-a-create] para crear conjuntos de registros A vacíos para el punto de conexión del registro y el punto de conexión de datos:

```azurecli
az network private-dns record-set a create \
  --name $REGISTRY_NAME \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP

# Specify registry region in data endpoint name
az network private-dns record-set a create \
  --name ${REGISTRY_NAME}.${REGISTRY_LOCATION}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP
```

Ejecute el comando [az network private-dns record-set a add-record][az-network-private-dns-record-set-a-add-record] para crear registros A para el punto de conexión del registro y el punto de conexión de datos:

```azurecli
az network private-dns record-set a add-record \
  --record-set-name $REGISTRY_NAME \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP \
  --ipv4-address $REGISTRY_PRIVATE_IP

# Specify registry region in data endpoint name
az network private-dns record-set a add-record \
  --record-set-name ${REGISTRY_NAME}.${REGISTRY_LOCATION}.data \
  --zone-name privatelink.azurecr.io \
  --resource-group $RESOURCE_GROUP \
  --ipv4-address $DATA_ENDPOINT_PRIVATE_IP
```

El vínculo privado ya está configurado y listo para su uso.

## <a name="set-up-private-link---portal"></a>Configuración de vínculo privado: portal

Configure un vínculo privado al crear un registro o agregue un vínculo privado a un registro existente. En los pasos siguientes se da por hecho que ya tiene una red virtual y una subred configuradas con una máquina virtual para pruebas. También puede [crear una nueva red virtual y una subred](../virtual-network/quick-create-portal.md).

### <a name="create-a-private-endpoint---new-registry"></a>Creación de un punto de conexión privado: nuevo registro

1. Al crear un registro en el portal, en la pestaña de **Basics**, en **SKU**, seleccione **Premium**.
1. Seleccione la pestaña **Redes**.
1. En **Conectividad de red**, seleccione **Punto de conexión privado** >  **+ Agregar**.
1. Escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | Suscripción | Seleccione su suscripción. |
    | Resource group | Escriba el nombre de un grupo existente o cree uno nuevo.|
    | Nombre | Escriba un nombre único. |
    | Subrecurso |Seleccione **registro**.|
    | **Redes** | |
    | Virtual network| Seleccione la red virtual en la que está implementada la máquina virtual, como *myDockerVMVNET*. |
    | Subnet | Seleccione una subred, como *myDockerVMSubnet*, en la que está implementada la máquina virtual. |
    |**Integración de DNS privado**||
    |Integración con una zona DNS privada |Seleccione **Sí**. |
    |Zona DNS privada |Seleccione *(Nuevo) privatelink.azurecr.io*. |
    |||
1. Acepte los valores predeterminados en los demás valores y seleccione **Revisar y crear**.

  ![Crear registro con punto de conexión privado](./media/container-registry-private-link/private-link-create-portal.png)

### <a name="create-a-private-endpoint---existing-registry"></a>Creación de un punto de conexión privado: registro existente

1. En el portal, vaya al registro de contenedor.
1. En **Configuración**, seleccione **Redes**.
1. En la pestaña **Puntos de conexión privados**, seleccione **+ Punto de conexión privado**.
1. En la pestaña **Datos básicos**, escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    | **Detalles del proyecto** | |
    | Suscripción | Seleccione su suscripción. |
    | Resource group | Escriba el nombre de un grupo existente o cree uno nuevo.|
    | **Detalles de instancia** |  |
    | Nombre | Escriba un nombre. |
    |Region|Seleccione una región.|
    |||
5. Seleccione **Siguiente: Resource** (Siguiente: Recurso).
6. Escriba o seleccione la siguiente información:

    | Configuración | Value |
    | ------- | ----- |
    |Método de conexión  | Seleccione **Conectarse a un recurso de Azure en mi directorio**.|
    | Suscripción| Seleccione su suscripción. |
    | Tipo de recurso | Seleccione **Microsoft.ContainerRegistry/registries**. |
    | Resource |Seleccione el nombre del registro.|
    |Subrecurso de destino |Seleccione **registro**.|
    |||
7. Seleccione **Siguiente: Configuration** (Siguiente: Configuración).
8. Escriba o seleccione la información:

    | Configuración | Value |
    | ------- | ----- |
    |**Redes**| |
    | Virtual network| Seleccione la red virtual en la que está implementada la máquina virtual, como *myDockerVMVNET*. |
    | Subnet | Seleccione una subred, como *myDockerVMSubnet*, en la que está implementada la máquina virtual. |
    |**Integración de DNS privado**||
    |Integración con una zona DNS privada |Seleccione **Sí**. |
    |Zona DNS privada |Seleccione *(Nuevo) privatelink.azurecr.io*. |
    |||

1. Seleccione **Revisar + crear**. Se le remitirá a la página **Revisar y crear**, donde Azure validará la configuración. 
2. Cuando reciba el mensaje **Validación superada**, seleccione **Crear**.

Una vez creado el punto de conexión privado, la configuración DNS de la zona privada aparece en la página **Información general** del punto de conexión:

1. En el portal, vaya al registro de contenedor y seleccione **Configuración > Redes**.
1. En la pestaña **Puntos de conexión privados**, seleccione el punto de conexión privado que creó.
1. En la página **información general**, revise la configuración del vínculo y la configuración de DNS personalizada.

  ![Configuración DNS del punto de conexión](./media/container-registry-private-link/private-endpoint-overview.png)

El vínculo privado ya está configurado y listo para su uso.

## <a name="disable-public-access"></a>Deshabilitación del acceso público

Para muchos escenarios, deshabilite el acceso al registro desde redes públicas. Esta configuración impide que los clientes que están fuera de la red virtual alcancen los puntos de conexión del registro. 

### <a name="disable-public-access---cli"></a>Deshabilitación del acceso público: CLI

Para deshabilitar el acceso público mediante la CLI de Azure, ejecute [az acr update][az-acr-update] y establezca `--public-network-enabled` en `false`. 

> [!NOTE]
> El argumento `public-network-enabled` requiere una versión 2.6.0 o posterior de la CLI de Azure. 

```azurecli
az acr update --name $REGISTRY_NAME --public-network-enabled false
```


### <a name="disable-public-access---portal"></a>Deshabilitación del acceso público: portal

1. En el portal, vaya al registro de contenedor y seleccione **Configuración > Redes**.
1. En la pestaña **Acceso público**, en **Permitir el acceso de red público**, seleccione **Deshabilitado**. Después, seleccione **Guardar**.

## <a name="validate-private-link-connection"></a>Validación de una conexión de vínculo privado

Debe validar que los recursos de la subred del punto de conexión privado se conectan al registro mediante una dirección IP privada y que tienen la integración correcta de la zona DNS privada.

Para validar la conexión de vínculo privado, use SSH en la máquina virtual configurada en la red virtual.

Ejecute una utilidad, como `nslookup` o `dig`, para buscar la dirección IP del registro a través del vínculo privado. Por ejemplo:

```bash
dig $REGISTRY_NAME.azurecr.io
```

La salida del ejemplo muestra la dirección IP del registro en el espacio de direcciones de la subred:

```console
[...]
; <<>> DiG 9.11.3-1ubuntu1.13-Ubuntu <<>> myregistry.azurecr.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 52155
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 65494
;; QUESTION SECTION:
;myregistry.azurecr.io.         IN      A

;; ANSWER SECTION:
myregistry.azurecr.io.  1783    IN      CNAME   myregistry.privatelink.azurecr.io.
myregistry.privatelink.azurecr.io. 10 IN A      10.0.0.7

[...]
```

Compare este resultado con la dirección IP pública de la salida `dig` para el mismo registro a través de un punto de conexión público:

```console
[...]
;; ANSWER SECTION:
myregistry.azurecr.io.  2881    IN  CNAME   myregistry.privatelink.azurecr.io.
myregistry.privatelink.azurecr.io. 2881 IN CNAME xxxx.xx.azcr.io.
xxxx.xx.azcr.io.    300 IN  CNAME   xxxx-xxx-reg.trafficmanager.net.
xxxx-xxx-reg.trafficmanager.net. 300 IN CNAME   xxxx.westeurope.cloudapp.azure.com.
xxxx.westeurope.cloudapp.azure.com. 10  IN A 20.45.122.144

[...]
```

### <a name="registry-operations-over-private-link"></a>Operaciones del registro a través de un vínculo privado

Además, compruebe que puede realizar operaciones del registro desde la máquina virtual de la subred. Cree una conexión SSH con la máquina virtual y ejecute [az acr login][az-acr-login] para iniciar sesión en el registro. En función de la configuración de la máquina virtual, es posible que tenga que agregar el prefijo `sudo` a los siguientes comandos.

```bash
az acr login --name $REGISTRY_NAME
```

Realice operaciones del registro como `docker pull` para extraer una imagen de ejemplo del registro. Sustituya `hello-world:v1` por una imagen y la etiqueta pertinente para el registro, y anteponga el nombre del servidor de inicio de sesión del registro (todo en minúsculas):

```bash
docker pull myregistry.azurecr.io/hello-world:v1
``` 

Docker extrae correctamente la imagen a la máquina virtual.

## <a name="manage-private-endpoint-connections"></a>Administración de conexiones de punto de conexión privado

Administre las conexiones del punto de conexión privado del registro mediante Azure Portal o los comandos del grupo de comandos [az acr private-endpoint-connection][az-acr-private-endpoint-connection]. Las operaciones incluyen aprobar, eliminar, enumerar, rechazar o mostrar detalles de las conexiones del punto de conexión privado del registro.

Por ejemplo, para enumerar las conexiones del punto de conexión privado de un registro, ejecute el comando [az acr private-endpoint-connection list][az-acr-private-endpoint-connection-list]. Por ejemplo:

```azurecli
az acr private-endpoint-connection list \
  --registry-name $REGISTRY_NAME 
```

Al configurar una conexión de punto de conexión privado mediante los pasos de este artículo, el registro acepta automáticamente las conexiones de los clientes y servicios que tienen permisos RBAC de Azure en el registro. Puede configurar el punto de conexión de modo que exija la aprobación manual de conexiones. Para obtener información sobre cómo aprobar y rechazar conexiones de punto de conexión privado, vea [Administrar una conexión de punto de conexión privado](../private-link/manage-private-endpoint.md).

> [!IMPORTANT]
> Actualmente, si elimina un punto de conexión privado de un registro, es posible que también tenga que eliminar el vínculo de la red virtual a la zona privada. Si no se elimina el vínculo, es posible que se produzca un error similar a `unresolvable host`.

## <a name="dns-configuration-options"></a>Opciones de configuración de DNS

El punto de conexión privado de este ejemplo se integra con una zona DNS privada asociada a una red virtual básica. Esta configuración usa el servicio DNS proporcionado por Azure para resolver directamente el FQDN público del registro en sus direcciones IP privadas de la red virtual. 

El vínculo privado admite escenarios de configuración de DNS adicionales que usan la zona privada, incluso con soluciones DNS personalizadas. Por ejemplo, podría tener una solución DNS personalizada implementada en la red virtual o en el entorno local en una red que se conecte a la red virtual mediante una puerta de enlace de VPN o Azure ExpressRoute. 

Para resolver el FQDN público del registro en la dirección IP privada en estos escenarios, debe configurar un reenviador de nivel de servidor para el servicio Azure DNS (168.63.129.16). Las opciones de configuración y los pasos exactos dependen de las redes y DNS existentes. Para obtener ejemplos, vea [Configuración de DNS para puntos de conexión privados de Azure](../private-link/private-endpoint-dns.md).

> [!IMPORTANT]
> Si para conseguir una alta disponibilidad ha creado puntos de conexión privados en varias regiones, se recomienda usar un grupo de recursos independiente en cada región y colocar en él la red virtual y la zona DNS privada asociada. Esta configuración también evita la resolución de DNS imprevisible debido al uso compartido de la misma zona DNS privada.

### <a name="manually-configure-dns-records"></a>Configuración manual de los registros DNS

En algunos escenarios, es posible que deba configurar manualmente los registros DNS de una zona privada en lugar de usar la zona privada proporcionada por Azure. Asegúrese de crear registros para cada uno de los siguientes puntos de conexión: el punto de conexión del registro, el punto de conexión de datos del registro y el punto de conexión de datos de cualquier réplica regional adicional. Si no se configuran todos los registros, es posible que no se pueda acceder al registro.

> [!IMPORTANT]
> Si posteriormente agrega una nueva réplica, debe agregar manualmente un nuevo registro DNS para el punto de conexión de datos de esa región. Por ejemplo, si crea una réplica de *myregistry* en la ubicación northeurope, agregue un registro para `myregistry.northeurope.data.azurecr.io`.

Los FQDN y las direcciones IP privadas que necesita para crear registros DNS están asociados a la interfaz de red del punto de conexión privado. Puede obtener esta información mediante la CLI de Azure o desde el portal:

* En la CLI de Azure, ejecute el comando [az network nic show][az-network-nic-show]. Para ver comandos de ejemplo, consulte [Obtención de la configuración de IP del punto de conexión](#get-endpoint-ip-configuration), anteriormente en este artículo.

* En el portal, vaya al punto de conexión privado y seleccione **Configuración de DNS**.

Después de crear registros DNS, asegúrese de que los FQDN del registro se resuelven correctamente en sus respectivas direcciones IP privadas.

## <a name="clean-up-resources"></a>Limpieza de recursos

Si ha creado todos los recursos de Azure en el mismo grupo de recursos y ya no los necesita, puede eliminarlos con el comando [az group delete](/cli/azure/group):

```azurecli
az group delete --name $RESOURCE_GROUP
```

Para limpiar los recursos en el portal, vaya al grupo de recursos. Una vez que se ha cargado el grupo de recursos, haga clic en **Eliminar grupo de recursos** para quitar el grupo de recursos y los recursos almacenados en él.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información sobre Private Link, consulte la documentación de [Azure Private Link](../private-link/private-link-overview.md).

* Si necesita configurar reglas de acceso a un registro desde detrás de un firewall cliente, consulte [Configuración de reglas para acceder a un registro de contenedor de Azure desde detrás de un firewall](container-registry-firewall-access-rules.md).

* [Solución de problemas de conectividad de puntos de conexión privados de Azure](../private-link/troubleshoot-private-endpoint-connectivity.md)

<!-- LINKS - external -->
[docker-linux]: https://docs.docker.com/engine/installation/#supported-platforms
[docker-login]: https://docs.docker.com/engine/reference/commandline/login/
[docker-mac]: https://docs.docker.com/docker-for-mac/
[docker-push]: https://docs.docker.com/engine/reference/commandline/push/
[docker-tag]: https://docs.docker.com/engine/reference/commandline/tag/
[docker-windows]: https://docs.docker.com/docker-for-windows/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-show]: /cli/azure/acr#az_acr_show
[az-acr-repository-show]: /cli/azure/acr/repository#az_acr_repository_show
[az-acr-repository-list]: /cli/azure/acr/repository#az_acr_repository_list
[az-acr-login]: /cli/azure/acr#az_acr_login
[az-acr-private-endpoint-connection]: /cli/azure/acr/private-endpoint-connection
[az-acr-private-endpoint-connection-list]: /cli/azure/acr/private-endpoint-connection#az_acr_private-endpoint-connection-list
[az-acr-private-endpoint-connection-approve]: /cli/azure/acr/private-endpoint-connection#az_acr_private_endpoint_connection_approve
[az-acr-update]: /cli/azure/acr#az_acr_update
[az-group-create]: /cli/azure/group
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
[az-vm-create]: /cli/azure/vm#az_vm_create
[az-network-vnet-subnet-show]: /cli/azure/network/vnet/subnet/#az_network_vnet_subnet_show
[az-network-vnet-subnet-update]: /cli/azure/network/vnet/subnet/#az_network_vnet_subnet_update
[az-network-vnet-list]: /cli/azure/network/vnet/#az_network_vnet_list
[az-network-private-endpoint-create]: /cli/azure/network/private-endpoint#az_network_private_endpoint_create
[az-network-private-endpoint-show]: /cli/azure/network/private-endpoint#az_network_private_endpoint_show
[az-network-private-dns-zone-create]: /cli/azure/network/private-dns/zone#az_network_private_dns_zone_create
[az-network-private-dns-link-vnet-create]: /cli/azure/network/private-dns/link/vnet#az_network_private_dns_link_vnet_create
[az-network-private-dns-record-set-a-create]: /cli/azure/network/private-dns/record-set/a#az_network_private_dns_record_set_a_create
[az-network-private-dns-record-set-a-add-record]: /cli/azure/network/private-dns/record-set/a#az_network_private_dns_record_set_a_add_record
[az-network-nic-show]: /cli/azure/network/nic#az_network_nic_show
[quickstart-portal]: container-registry-get-started-portal.md
[quickstart-cli]: container-registry-get-started-azure-cli.md
[azure-portal]: https://portal.azure.com
