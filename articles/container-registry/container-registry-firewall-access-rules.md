---
title: Reglas de acceso del firewall
description: Configure reglas para acceder a un registro de contenedor de Azure desde detrás de un firewall. Para ello, permita el acceso a la API REST y los nombres de dominio del punto de conexión de datos o los intervalos de direcciones IP específicos del servicio.
ms.topic: article
ms.date: 05/18/2020
ms.openlocfilehash: 6aea4415468eb21e8d010b74597fc68e4ebf573f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783942"
---
# <a name="configure-rules-to-access-an-azure-container-registry-behind-a-firewall"></a>Configuración de reglas para acceder a un registro de contenedor de Azure desde detrás de un firewall

En este artículo se explica cómo configurar reglas en el firewall para permitir el acceso a un registro de contenedor de Azure. Por ejemplo, es posible que un dispositivo Azure IoT Edge que esté detrás de un firewall o de un servidor proxy necesite acceder a un registro de contenedor para extraer una imagen del contenedor. O bien, que un servidor bloqueado de una red local necesite tener acceso para enviar una imagen.

Si, por el contrario, desea configurar el acceso de red entrante a un registro de contenedor solo dentro de una red virtual de Azure, consulte [Configurar Azure Private Link para un registro de contenedor de Azure](container-registry-private-link.md).

## <a name="about-registry-endpoints"></a>Acerca de los puntos de conexión del registro

Para extraer imágenes u otros artefactos de un registro de contenedor de Azure (o para insertarlas en él), un cliente (por ejemplo, un demonio de Docker) debe interactuar a través de HTTPS con dos puntos de conexión distintos. En el caso de los clientes que acceden a un registro desde detrás de un firewall, debe configurar las reglas de acceso para ambos puntos de conexión. Ambos puntos de conexión se alcanzan a través del puerto 443.

* **Punto de conexión de la API de REST del registro**: las operaciones de autenticación y administración del registro se administran a través del punto de conexión público de la API de REST del registro. Este punto de conexión es el nombre del servidor de inicio de sesión del registro. Ejemplo: `myregistry.azurecr.io`

* **Punto de conexión de almacenamiento (datos)** : Azure [asigna almacenamiento de blob](container-registry-storage.md) en cuentas de Azure Storage en nombre de cada registro para administrar los datos de imágenes de contenedor y otros artefactos. Cuando un cliente accede a capas de imagen en un registro de contenedor de Azure, realiza solicitudes mediante un punto de conexión de la cuenta de almacenamiento proporcionado por el registro.

Si el registro está [replicado geográficamente](container-registry-geo-replication.md), es posible que el cliente tenga que interactuar con los puntos de conexión de datos en una región específica o en varias regiones replicadas.

## <a name="allow-access-to-rest-and-data-endpoints"></a>Permitir el acceso a los puntos de conexión de REST y de datos

* **Punto de conexión de REST**: permite el acceso al nombre del servidor de inicio de sesión del registro completo, `<registry-name>.azurecr.io`, o un rango de direcciones IP asociado
* **Punto de conexión de almacenamiento (datos)** : permite el acceso a todas las cuentas de almacenamiento blob de Azure usando el carácter comodín `*.blob.core.windows.net`, o un rango de direcciones IP asociado.
> [!NOTE]
> Azure Container Registry introduce [puntos de conexión de datos dedicados](#enable-dedicated-data-endpoints), lo que le permite ajustar el ámbito de las reglas de Firewall de cliente para el almacenamiento del registro. Opcionalmente, puede habilitar puntos de conexión de datos en todas las regiones donde se encuentra o replica el registro, con el formato `<registry-name>.<region>.data.azurecr.io`.

## <a name="allow-access-by-ip-address-range"></a>Permitir el acceso por intervalo de direcciones IP

Si su organización tiene directivas para permitir el acceso solo a direcciones o intervalos de direcciones IP específicos, descargue [Azure IP Ranges and Service Tags – Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519) (Intervalos de direcciones IP y etiquetas de servicio de Azure: nube pública).

Para buscar los intervalos IP del punto de conexión de REST de ACR, busque **AzureContainerRegistry** en el archivo JSON.

> [!IMPORTANT]
> Los intervalos de direcciones IP de los servicios de Azure pueden cambiar. Estos cambios se publican semanalmente. Descargue el archivo JSON con regularidad y realice las modificaciones necesarias en las reglas de acceso. Si su escenario implica configurar reglas de grupo de seguridad de red en una red virtual de Azure o usa Azure Firewall, use la [etiqueta de servicio](#allow-access-by-service-tag) **AzureContainerRegistry** en su lugar.
>

### <a name="rest-ip-addresses-for-all-regions"></a>Direcciones IP de REST para todas las regiones

```json
{
  "name": "AzureContainerRegistry",
  "id": "AzureContainerRegistry",
  "properties": {
    "changeNumber": 10,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.66.140.72/29",
    [...]
```

### <a name="rest-ip-addresses-for-a-specific-region"></a>Direcciones IP de REST para una región específica

Busque la región específica, como **AzureContainerRegistry.AustraliaEast**.

```json
{
  "name": "AzureContainerRegistry.AustraliaEast",
  "id": "AzureContainerRegistry.AustraliaEast",
  "properties": {
    "changeNumber": 1,
    "region": "australiaeast",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.70.72.136/29",
    [...]
```

### <a name="storage-ip-addresses-for-all-regions"></a>Direcciones IP de almacenamiento para todas las regiones

```json
{
  "name": "Storage",
  "id": "Storage",
  "properties": {
    "changeNumber": 19,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "13.65.107.32/28",
    [...]
```

### <a name="storage-ip-addresses-for-specific-regions"></a>Direcciones IP de almacenamiento para regiones específicas

Busque la región específica, como **Storage.AustraliaCentral**.

```json
{
  "name": "Storage.AustraliaCentral",
  "id": "Storage.AustraliaCentral",
  "properties": {
    "changeNumber": 1,
    "region": "australiacentral",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "52.239.216.0/23"
    [...]
```

## <a name="allow-access-by-service-tag"></a>Permitir el acceso por etiqueta de servicio

En una red virtual de Azure, use las reglas de seguridad de red para filtrar el tráfico de un recurso (por ejemplo, una máquina virtual) a un registro de contenedor. Para simplificar la creación de las reglas de red de Azure, use la [etiqueta de servicio](../virtual-network/network-security-groups-overview.md#service-tags) **AzureContainerRegistry**. Una etiqueta de servicio constituye un grupo de prefijos de dirección IP para tener acceso a un servicio de Azure de manera global o por región de Azure. La etiqueta se actualiza automáticamente cuando las direcciones cambian. 

Por ejemplo, cree una regla de grupo de seguridad de red saliente con el destino **AzureContainerRegistry** para permitir el tráfico a un registro de contenedor de Azure. Para permitir el acceso a la etiqueta de servicio únicamente en una región específica, especifique la región con el siguiente formato: **AzureContainerRegistry**.[*nombre de la región*].

## <a name="enable-dedicated-data-endpoints"></a>Habilitar puntos de conexión de datos dedicados 

> [!WARNING]
> Si ha configurado previamente el acceso de firewall de cliente a los puntos de conexión de `*.blob.core.windows.net` existentes, el cambio a puntos de conexión de datos dedicados afectará la conectividad del cliente, lo que producirá errores de extracción. Para asegurarse de que los clientes tienen un acceso coherente, agregue las nuevas reglas de punto de conexión de datos a las reglas de firewall de cliente. Una vez completados, habilite los puntos de conexión de datos dedicados para los registros mediante el CLI de Azure u otras herramientas.

Los puntos de conexión de datos dedicados son una característica opcional del nivel de servicio del registro de contenedor **Premium**. Para obtener información sobre los límites y los niveles de servicio de los registros, consulte [Niveles de servicio de Azure Container Registry](container-registry-skus.md). 

Puede habilitar puntos de conexión de datos dedicados mediante el Azure Portal o el CLI de Azure. Los puntos de conexión de datos siguen un patrón regional, `<registry-name>.<region>.data.azurecr.io`. En un registro con replicación geográfica, la habilitación de puntos de conexión de datos permite puntos de conexión en todas las regiones de réplica.

### <a name="portal"></a>Portal

Para habilitar los puntos de conexión de datos mediante el portal:

1. Vaya al registro de contenedor.
1. Seleccione **Redes** > **de acceso público**.
1. Active la casilla **Habilitar punto de conexión de datos dedicado**.
1. Seleccione **Guardar**.

Los puntos de conexión de datos o puntos de conexión aparecen en el portal.

:::image type="content" source="media/container-registry-firewall-access-rules/dedicated-data-endpoints-portal.png" alt-text="Puntos de conexión de datos dedicados en el portal":::

### <a name="azure-cli"></a>Azure CLI

Para habilitar los puntos de conexión de datos mediante el CLI de Azure, use CLI de Azure versión 2.4.0 o posterior. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

El siguiente comando [az acr update][az-acr-update] permite puntos de conexión de datos dedicados en un registro *myregistry*. 

```azurecli
az acr update --name myregistry --data-endpoint-enabled
```

Para ver los puntos de conexión de datos, use el comando [az acr show-endpoints][az-acr-show-endpoints]:

```azurecli
az acr show-endpoints --name myregistry
```

La salida con fines de demostración muestra dos puntos de conexión regionales

```
{
    "loginServer": "myregistry.azurecr.io",
    "dataEndpoints": [
        {
            "region": "eastus",
            "endpoint": "myregistry.eastus.data.azurecr.io",
        },
        {
            "region": "westus",
            "endpoint": "myregistry.westus.data.azurecr.io",
        }
    ]
}
```

Después de configurar los puntos de conexión de datos dedicados para el registro, puede habilitar las reglas de acceso de firewall de cliente para los puntos de conexión de datos. Habilite las reglas de acceso de punto de conexión de datos para todas las regiones del registro necesarias.

## <a name="configure-client-firewall-rules-for-mcr"></a>Configuración de las reglas de firewall de cliente para MCR

Si necesita obtener acceso a Microsoft Container Registry (MCR) desde detrás de un firewall, consulte las instrucciones para configurar las [reglas de firewall de cliente de MCR](https://github.com/microsoft/containerregistry/blob/master/client-firewall-rules.md). MCR es el registro principal para todas las imágenes de Docker publicadas por Microsoft, como las imágenes de Windows Server.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre los [Procedimientos recomendados de seguridad de la red de Azure](../security/fundamentals/network-best-practices.md).

* Obtenga más información sobre los [grupos de seguridad](../virtual-network/network-security-groups-overview.md) en una red virtual de Azure.

* Más información sobre la configuración de [Private Link](container-registry-private-link.md) para un registro de contenedor

* Más información sobre [puntos de conexión de datos dedicados](https://azure.microsoft.com/blog/azure-container-registry-mitigating-data-exfiltration-with-dedicated-data-endpoints/) para Azure Container Registry



<!-- IMAGES -->

<!-- LINKS - External -->

<!-- LINKS - Internal -->

[az-acr-update]: /cli/azure/acr#az_acr_update
[az-acr-show-endpoints]: /cli/azure/acr#az_acr_show_endpoints