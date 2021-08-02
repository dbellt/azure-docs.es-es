---
title: Requisitos del sistema de Microsoft Azure Stack Edge Mini R | Microsoft Docs
description: Obtenga información sobre los requisitos de software y de red de Azure Stack Edge Mini R
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 02/05/2021
ms.author: alkohli
ms.openlocfilehash: 31183be893a9216f33adc4f5e25a55cfb3545db8
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110099575"
---
# <a name="azure-stack-edge-mini-r-system-requirements"></a>Requisitos del sistema de Azure Stack Edge Mini R

En este artículo se describen los requisitos importantes del sistema para la solución Microsoft Azure Stack Edge Mini R y para los clientes que se conectarán a Azure Stack Edge Mini R. Le recomendamos que revise atentamente la información antes de implementar Azure Stack Edge Mini R. Puede volver a consultar esta información según sea necesario durante la implementación y la operación subsiguiente.

Los requisitos del sistema de Azure Stack Edge Mini R son los siguientes:

- **Requisitos de software para hosts**: describe las plataformas compatibles, los exploradores de la interfaz de usuario de configuración local, los clientes SMB y los requisitos adicionales de los clientes que acceden al dispositivo.
- **Requisitos de red para el dispositivo**: proporciona información acerca de los requisitos de red para el funcionamiento del dispositivo físico.

## <a name="supported-os-for-clients-connected-to-device"></a>Sistema operativo compatible con los clientes conectados al dispositivo

[!INCLUDE [Supported OS for clients connected to device](../../includes/azure-stack-edge-gateway-supported-client-os.md)]

## <a name="supported-protocols-for-clients-accessing-device"></a>Protocolos compatibles con los clientes que acceden al dispositivo

[!INCLUDE [Supported protocols for clients accessing device](../../includes/azure-stack-edge-gateway-supported-client-protocols.md)]

## <a name="supported-storage-accounts"></a>Cuentas de almacenamiento admitidas

[!INCLUDE [Supported storage accounts](../../includes/azure-stack-edge-gateway-supported-storage-accounts.md)]

## <a name="supported-edge-storage-accounts"></a>Cuentas admitidas de almacenamiento de Edge

Se admiten las siguientes cuentas de almacenamiento de Edge con la interfaz de REST del dispositivo. Las cuentas de almacenamiento de Edge se crean en el dispositivo. Para obtener más información, consulte [Cuentas de almacenamiento de Edge](azure-stack-edge-gpu-manage-storage-accounts.md#about-edge-storage-accounts).

|Tipo  |Cuenta de almacenamiento  |Comentarios  |
|---------|---------|---------|
|Estándar     |GPv1: Blob en bloques         |         |


* Actualmente no se admiten blobs en páginas ni archivos de Azure Files.

## <a name="supported-local-azure-resource-manager-storage-accounts"></a>Cuentas locales de almacenamiento de Azure Resource Manager admitidas

Estas cuentas de almacenamiento se generan mediante las API locales del dispositivo cuando está conectado a una instancia local de Azure Resource Manager. Se admiten las siguientes cuentas de almacenamiento:

|Tipo  |Cuenta de almacenamiento  |Comentarios  |
|---------|---------|---------|
|Estándar     |GPv1: blob en bloques, blob en páginas         | El tipo de SKU es Standard_LRS.        |
|Premium   |GPv1: blob en bloques, blob en páginas         |El tipo de SKU es Premium_LRS.         |


## <a name="supported-storage-types"></a>Tipos de almacenamiento compatibles

[!INCLUDE [Supported storage types](../../includes/azure-stack-edge-gateway-supported-storage-types.md)]


## <a name="supported-browsers-for-local-web-ui"></a>Exploradores compatibles con la interfaz de usuario web local

[!INCLUDE [Supported browsers for local web UI](../../includes/azure-stack-edge-gateway-supported-browsers.md)]

## <a name="networking-port-requirements"></a>Requisitos de los puertos de redes

### <a name="port-requirements-for-azure-stack-edge-mini-r"></a>Requisitos de puertos de Azure Stack Edge Mini R

La siguiente tabla enumera los puertos que deben abrirse en el firewall para permitir el tráfico de administración, de la nube o de SMB. En esta tabla, *dentro* o *entrante* hace referencia a la dirección desde la que el cliente entrante solicita acceso al dispositivo. Por su parte, *fuera* o *saliente* hacen referencia a la dirección en la que el dispositivo Azure Stack Edge Mini R envía datos al exterior, fuera de los límites de la implementación; por ejemplo, datos que salen a Internet.

[!INCLUDE [Port configuration for device](../../includes/azure-stack-edge-gateway-port-config.md)]

### <a name="port-requirements-for-iot-edge"></a>Requisitos de puertos para IoT Edge

Azure IoT Edge permite la comunicación saliente desde un dispositivo Edge local a la nube de Azure mediante protocolos de IoT Hub compatibles. La comunicación entrante solo es necesaria para escenarios específicos donde Azure IoT Hub necesita insertar mensajes en el dispositivo de Azure IoT Edge (por ejemplo, mensajes de la nube al dispositivo).

Utilice la siguiente tabla para la configuración de los puertos de los servidores que hospedan Azure IoT Edge en tiempo de ejecución:

| N.º de puerto | Dentro o fuera | Ámbito de puerto | Obligatorio | Guía |
|----------|-----------|------------|----------|----------|
| TCP 443 (HTTPS)| Fuera       | WAN        | Sí      | Abierto para comunicación de salida en el aprovisionamiento de IoT Edge. Esta configuración es necesaria cuando se usen scripts manuales o Azure IoT Device Provisioning Service (DPS).|

Para obtener información completa, vaya a [Reglas de configuración de puertos y firewall para la implementación de IoT Edge](../iot-edge/troubleshoot.md).

## <a name="url-patterns-for-firewall-rules"></a>Patrones de URL para reglas de firewall

Con frecuencia, los administradores de red pueden configurar reglas avanzadas de firewall de acuerdo con los patrones de URL para filtrar el tráfico saliente y entrante. El dispositivo Azure Stack Edge Mini R y el servicio dependen de otras aplicaciones de Microsoft, como Azure Service Bus, Azure Active Directory Access Control, cuentas de almacenamiento y servidores de Microsoft Update. Es posible usar los patrones de URL asociados a estas aplicaciones para configurar las reglas de firewall. Es importante entender que los patrones de URL asociados a estas aplicaciones pueden cambiar. Estos cambios requieren que el administrador de red supervise y actualice las reglas de firewall de Azure Stack Edge Mini R cuando sea necesario.

Se recomienda configurar las reglas de firewall para el tráfico de salida en función de las direcciones IP fijas de Azure Stack Edge Mini R, que pueden establecerse libremente en la mayoría de los casos. Sin embargo, puede utilizar la información siguiente con el objetivo de establecer las reglas avanzadas de firewall que se necesitan para crear entornos seguros.

> [!NOTE]
> - Las direcciones IP del dispositivo (origen) siempre se deben establecer en todas las interfaces de red habilitadas para la nube.
> - Las IP de destino, por su parte, se deben establecer en los [intervalos de direcciones IP del centro de datos de Azure](https://www.microsoft.com/download/confirmation.aspx?id=41653).

### <a name="url-patterns-for-gateway-feature"></a>Patrones de dirección URL para la característica de puerta de enlace

[!INCLUDE [URL patterns for firewall](../../includes/azure-stack-edge-gateway-url-patterns-firewall.md)]

### <a name="url-patterns-for-compute-feature"></a>Patrones de dirección URL para la característica de proceso

| Patrón de URL                      | Componente o funcionalidad                     |   
|----------------------------------|---------------------------------------------|
| https:\//mcr.microsoft.com<br></br>https://\*.cdn.mscr.io | Registro de contenedor de Microsoft (obligatorio)               |
| https://\*.azurecr.io                     | Registros de contenedores personales y de terceros (opcional) | 
| https://\*.azure-devices.net              | Acceso de IoT Hub (obligatorio)                             | 
| https://\*.docker.com              | StorageClass (obligatorio)                             |

### <a name="url-patterns-for-gateway-for-azure-government"></a>Patrones de dirección URL para la puerta de enlace de Azure Government

[!INCLUDE [Azure Government URL patterns for firewall](../../includes/azure-stack-edge-gateway-gov-url-patterns-firewall.md)]

### <a name="url-patterns-for-compute-for-azure-government"></a>Patrones de dirección URL para el proceso de Azure Government

| Patrón de URL                      | Componente o funcionalidad                     |  
|----------------------------------|---------------------------------------------|
| https:\//mcr.microsoft.com<br></br>https://\*.cdn.mscr.com | Registro de contenedor de Microsoft (obligatorio)               |
| https://\*.azure-devices.us              | Acceso de IoT Hub (obligatorio)           |
| https://\*.azurecr.us                    | Registros de contenedores personales y de terceros (opcional) | 

## <a name="internet-bandwidth"></a>Ancho de banda de Internet

[!INCLUDE [Internet bandwidth](../../includes/azure-stack-edge-gateway-internet-bandwidth.md)]

## <a name="compute-sizing-considerations"></a>Consideraciones de tamaño de proceso

Haga uso de su experiencia al desarrollar y probar la solución para asegurarse de que hay suficiente capacidad en el dispositivo Azure Stack Edge Mini R y que se obtiene un rendimiento óptimo del dispositivo.

Debe considerar los siguientes factores:

- **Detalles específicos del contenedor** - Piense en lo siguiente.

    - ¿Cuál es la superficie del contenedor? ¿Cuánta memoria, almacenamiento y CPU consume el contenedor?
    - ¿Cuántos contenedores hay en la carga de trabajo? Podría tener muchos contenedores ligeros en vez unos pocos que consuman muchos recursos.
    - ¿Cuáles son los recursos asignados a estos contenedores en comparación con los recursos que están consumiendo (superficie)?
    - ¿Cuántas capas comparten los contenedores? Las imágenes de contenedor son una agrupación de archivos organizados en una pila de capas. Para la imagen de contenedor, determine el número de capas y sus respectivos tamaños para calcular el consumo de recursos.
    - ¿Hay contenedores sin usar? Un contenedor detenido todavía ocupa espacio en disco.
    - ¿En qué idioma están escritos los contenedores?
- **Tamaño de los datos procesados**: ¿cuántos datos procesarán sus contenedores? ¿Estos datos consumirán espacio en disco o los datos se procesarán en la memoria?
- **Rendimiento esperado**: ¿cuáles son las características del rendimiento deseado de la solución? 

Para comprender y ajustar el rendimiento de la solución, puede usar:

- Las métricas de proceso disponibles en Azure Portal. Vaya al recurso de Azure Stack Edge y después a **Supervisión > Métricas**. Examine **Proceso perimetral > Uso de la memoria** y **Proceso perimetral > Porcentaje de CPU** para entender los recursos disponibles y cómo se consumen los recursos.
- Los comandos de supervisión disponibles a través de la interfaz de PowerShell del dispositivo, como:

    - Estadísticas de `dkr` para obtener una transmisión en vivo de las estadísticas del uso de recursos de los contenedores. El comando es compatible con las métricas de CPU, uso de memoria, límite de memoria y E/S de red.
    - `dkr system df` para obtener información relacionada con la cantidad de espacio en disco utilizado. 
    - `dkr image [prune]` para limpiar las imágenes no utilizadas y liberar espacio.
    - `dkr ps --size` para ver el tamaño aproximado de un contenedor en ejecución. 

    Para obtener más información sobre los comandos disponibles, vaya a [Depuración de problemas de Kubernetes](azure-stack-edge-gpu-connect-powershell-interface.md#debug-kubernetes-issues-related-to-iot-edge).

Por último, no olvide validar la solución en el conjunto de datos y cuantificar el rendimiento de Azure Stack Edge Mini R antes de realizar la implementación en producción.

## <a name="next-step"></a>Paso siguiente

- [Implementación de Azure Stack Edge Mini R](azure-stack-edge-placeholder.md)
