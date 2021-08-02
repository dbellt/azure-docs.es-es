---
title: Reglas de red de salida necesarias para Azure Managed Instance for Apache Cassandra
description: Obtenga información sobre cuáles son las reglas de red de salida y los FQDN necesarios para Azure Managed Instance for Apache Cassandra.
author: christopheranderson
ms.service: managed-instance-apache-cassandra
ms.topic: how-to
ms.date: 05/21/2021
ms.author: chrande
ms.openlocfilehash: 8c66657832000d1d0f9e1e9d842f16ce149e314a
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111955969"
---
# <a name="required-outbound-network-rules"></a>Reglas de red de salida necesarias

> [!IMPORTANT]
> Azure Managed Instance for Apache Cassandra se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

El servicio Azure Managed Instance for Apache Cassandra requiere ciertas reglas de red para administrar correctamente el servicio. Asegúrese de que tiene expuestas las reglas adecuadas para mantener el servicio seguro y evitar problemas operativos.

## <a name="azure-global-required-network-rules"></a>Reglas de red obligatorias globales de Azure

Las reglas de red obligatorias y las dependencias de dirección IP son las siguientes:

| Punto de conexión de destino                                                             | Protocolo | Port    | Uso  |
|----------------------------------------------------------------------------------|----------|---------|------|
|snovap`<region>`.blob.core.windows.net:443</br> Or</br> [ServiceTag:](../virtual-network/service-tags-overview.md#available-service-tags) Azure Storage | HTTPS | 443 | Necesario para proteger la comunicación entre los nodos y Azure Storage, para la comunicación y configuración del plano de control.|
|*.blob.core.windows.net:443</br> Or</br> [ServiceTag:](../virtual-network/service-tags-overview.md#available-service-tags) Azure Storage | HTTPS | 443 | Necesario para proteger la comunicación entre los nodos y Azure Storage, para almacenar copias de seguridad. *Se está revisando la característica de copia de seguridad y el nombre del almacenamiento seguirá un patrón cuando tenga disponibilidad general.*|
|vmc-p-`<region>`.vault.azure.net:443</br> Or</br> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): Azure KeyVault | HTTPS | 443 | Necesario para proteger la comunicación entre los nodos y Azure Key Vault. Para proteger la comunicación dentro del clúster se usan certificados y claves.|
|management.azure.com:443</br> Or</br> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): Azure Virtual Machine Scale Sets/Azure Management API | HTTPS | 443 | Necesario para recopilar información sobre los nodos de Cassandra y administrarlos (por ejemplo, reiniciar).|
|*.servicebus.windows.net:443</br> Or</br> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): Azure EventHub | HTTPS | 443 | Necesario para reenviar registros a Azure.|
|jarvis-west.dc.ad.msft.net:443</br> Or</br> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): Azure Monitor | HTTPS | 443 | Necesario para reenviar métricas a Azure. |
|login.microsoftonline.com:443</br> Or</br> [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): Azure AD | HTTPS | 443 | Obligatorio para autenticación de Azure Active Directory.|
| packages.microsoft.com | HTTPS | 443 | Necesario para las actualizaciones de las definiciones y firmas del analizador de seguridad de Azure. |

## <a name="managed-instance-for-apache-cassandra-internal-port-usage"></a>Uso de los puertos internos de Managed Instance for Apache Cassandra

Los puertos siguientes solo son accesibles dentro de la red virtual (o redes virtuales emparejadas o ExpressRoute). Las instancias de Managed Instance for Apache Cassandra no tienen una dirección IP pública y no deben ser accesibles en Internet.

| Port | Uso |
| ---- | --- |
| 8443 | Interno |
| 9443 | Interno |
| 7001 | Gossip: se usa para que los nodos de Cassandra se comuniquen entre sí. |
| 9042 | Cassandra: se usa para que los clientes se conecten a Cassandra. |
| 7199 | Interno |

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido acerca de las reglas de red para administrar correctamente el servicio. Más información sobre Azure Managed Instance for Apache Cassandra en los siguientes artículos:

* [¿Qué es Azure Managed Instance for Apache Cassandra? (versión preliminar)](introduction.md)
* [Administración de recursos de Azure Managed Instance for Apache Cassandra mediante la CLI de Azure](manage-resources-cli.md)