---
title: Visualización y configuración de la telemetría de protección contra DDoS para Azure DDoS Protection Standard
description: Aprenda a ver y configurar la telemetría de protección contra DDoS para Azure DDoS Protection Standard.
services: ddos-protection
documentationcenter: na
author: aletheatoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 12/28/2020
ms.author: yitoh
ms.openlocfilehash: 0a04c6c58f8bfa5370a6529b81a5a85090413a2a
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107540"
---
# <a name="view-and-configure-ddos-protection-telemetry"></a>Visualización y configuración de DDoS Protection

La versión estándar de Azure DDoS Protection proporciona información detallada y visualización de ataques con DDoS Attack Analytics. Los clientes que protegen sus redes virtuales frente a ataques DDoS disponen de visibilidad detallada sobre el tráfico de ataques y las acciones llevadas a cabo para mitigarlos mediante informes de mitigación de ataques y registros de flujo de mitigación. La telemetría enriquecida se expone a través de Azure Monitor, incluidas las métricas detalladas mientras dure un ataque DDoS. Las alertas se pueden configurar para cualquiera de las métricas de Azure Monitor que expone DDoS Protection. El registro se puede integrar con [Azure Sentinel](../sentinel/connect-azure-ddos-protection.md), Splunk (Azure Event Hubs), OMS Log Analytics y Azure Storage para realizar análisis avanzados con la interfaz de Azure Monitor Diagnostics.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Ver la telemetría de protección contra DDoS
> * Visualización de las directivas de mitigación de DDoS
> * Validar y probar la telemetría de protección contra DDoS

### <a name="metrics"></a>Métricas

> [!NOTE]
> Aunque se muestran varias opciones de **agregación** en Azure Portal, solo se admiten para cada métrica los tipos de agregación que se enumeran en la tabla siguiente. Lamentamos esta confusión; estamos trabajando para resolverlo.

Las siguientes [métricas](../azure-monitor/essentials/metrics-supported.md#microsoftnetworkpublicipaddresses) están disponibles para Azure DDoS Protection Standard. Estas métricas también se pueden exportar a través de la configuración de diagnóstico (consulte [Visualización y configuración del registro de diagnóstico de DDoS](diagnostic-logging.md)).


| Métrica | Nombre de métrica para mostrar | Unidad | Tipo de agregación | Descripción |
| --- | --- | --- | --- | --- |
| BytesDroppedDDoS | Bytes de entrada de DDoS descartados | BytesPerSecond | Máxima | Bytes de entrada de DDoS descartados| 
| BytesForwardedDDoS | Bytes de entrada de DDoS reenviados | BytesPerSecond | Máxima | Bytes de entrada de DDoS reenviados |
| BytesInDDoS | Bytes de entrada de DDoS | BytesPerSecond | Máxima | Bytes de entrada de DDoS |
| DDoSTriggerSYNPackets | Paquetes entrantes SYN para desencadenar la mitigación de DDoS | CountPerSecond | Máxima | Paquetes entrantes SYN para desencadenar la mitigación de DDoS |
| DDoSTriggerTCPPackets | Paquetes entrantes TCP para desencadenar la mitigación de DDoS | CountPerSecond | Máxima | Paquetes entrantes TCP para desencadenar la mitigación de DDoS |
| DDoSTriggerUDPPackets | Paquetes entrantes UDP para desencadenar la mitigación de DDoS | CountPerSecond | Máxima | Paquetes entrantes UDP para desencadenar la mitigación de DDoS |
| IfUnderDDoSAttack | Bajo ataque de DDoS o no | Count | Máxima | Bajo ataque de DDoS o no |
| PacketsDroppedDDoS | Paquetes entrantes de DDoS descartados | CountPerSecond | Máxima | Paquetes entrantes de DDoS descartados |
| PacketsForwardedDDoS | Paquetes entrantes de DDoS reenviados | CountPerSecond | Máxima | Paquetes entrantes de DDoS reenviados |
| PacketsInDDoS | Paquetes entrantes de DDoS | CountPerSecond | Máxima | Paquetes entrantes de DDoS |
| TCPBytesDroppedDDoS | Bytes de entrada TCP de DDoS descartados | BytesPerSecond | Máxima | Bytes de entrada TCP de DDoS descartados |
| TCPBytesForwardedDDoS | Bytes de entrada TCP de DDoS reenviados | BytesPerSecond | Máxima | Bytes de entrada TCP de DDoS reenviados |
| TCPBytesInDDoS | Bytes de entrada TCP de DDoS | BytesPerSecond | Máxima | Bytes de entrada TCP de DDoS |
| TCPPacketsDroppedDDoS | Paquetes TCP entrantes de DDoS descartados | CountPerSecond | Máxima | Paquetes TCP entrantes de DDoS descartados |
| TCPPacketsForwardedDDoS | Paquetes TCP entrantes de DDoS reenviados | CountPerSecond | Máxima | Paquetes TCP entrantes de DDoS reenviados |
| TCPPacketsInDDoS | Paquetes TCP entrantes de DDoS | CountPerSecond | Máxima | Paquetes TCP entrantes de DDoS |
| UDPBytesDroppedDDoS | Bytes de entrada UDP de DDoS descartados | BytesPerSecond | Máxima | Bytes de entrada UDP de DDoS descartados |
| UDPBytesDroppedDDoS | Bytes de entrada UDP de DDoS reenviados | BytesPerSecond | Máxima | Bytes de entrada UDP de DDoS reenviados |
| UDPBytesInDDoS | Bytes de entrada UDP de DDoS | BytesPerSecond | Máxima | Bytes de entrada UDP de DDoS |
| UDPPacketsDroppedDDoS | Paquetes UDP entrantes de DDoS descartados | CountPerSecond | Máxima | Paquetes UDP entrantes de DDoS descartados |
| UDPPacketsForwardedDDoS | Paquetes UDP entrantes de DDoS reenviados | CountPerSecond | Máxima | Paquetes UDP entrantes de DDoS reenviados |
| UDPPacketsInDDoS | Paquetes UDP entrantes de DDoS | CountPerSecond | Máxima | Paquetes UDP entrantes de DDoS |

## <a name="prerequisites"></a>Prerrequisitos

- Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.
- Antes realizar los pasos de este tutorial, primero debe crear un [plan de protección de Azure DDoS Standard](manage-ddos-protection.md) y DDoS Protection Standard debe estar habilitado en una red virtual.
- DDoS supervisa las direcciones IP públicas asignadas a los recursos dentro de una red virtual. Si no tiene ningún recurso con direcciones IP públicas en la red virtual, primero debe crear un recurso con una dirección IP pública. Puede supervisar la dirección IP pública de todos los recursos implementados a través de Resource Manager (no clásico) que aparecen en [Red virtual para servicios de Azure](../virtual-network/virtual-network-for-azure-services.md#services-that-can-be-deployed-into-a-virtual-network) (incluidas las instancias de Azure Load Balancer donde las máquinas virtuales de back-end se encuentran en la red virtual), excepto para entornos de Azure App Service. Para continuar con este tutorial, puede crear rápidamente una máquina virtual [Windows](../virtual-machines/windows/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json) o [Linux](../virtual-machines/linux/quick-create-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json).  

## <a name="view-ddos-protection-telemetry"></a>Ver la telemetría de protección contra DDoS

La telemetría para un ataque se proporciona a través de Azure Monitor en tiempo real. La telemetría solo está disponible cuando una dirección IP pública se ha solucionado. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com/) y vaya al plan de DDoS Protection.
2. En **Supervisión**, seleccione **Métricas**.
3. Seleccione **Ámbito**. Seleccione la **suscripción** que contiene la dirección IP pública que quiere registrar, elija **Dirección IP pública** en **Tipo de recurso**, seleccione la dirección IP pública específica para la que quiere registrar las métricas y, a continuación, elija **Aplicar**.
4. En **Agregación**, seleccione el tipo **Máximo**.

Los nombres de las métricas presentan distintos tipos de paquetes y bytes frente a paquetes, con una construcción básica de nombres de etiqueta en cada métrica, como se muestra aquí:

- **Nombre de etiqueta de eliminado** (por ejemplo, **Paquetes de entrada quitados por DDoS**): El número de paquetes que el sistema de protección contra DDoS elimina o limpia.
- **Nombre de etiqueta de reenviado** (por ejemplo, **Paquetes de entrada reenviados por DDoS**): el número de paquetes que reenvía el sistema DDoS a la dirección IP virtual de destino (tráfico que no se filtró).
- **Sin nombre de etiqueta** (por ejemplo **DDoS de paquetes de entrada**): el número total de paquetes que han llegado al sistema de limpieza (representa la suma de los paquetes eliminados y los reenviados).

## <a name="view-ddos-mitigation-policies"></a>Visualización de las directivas de mitigación de DDoS

DDoS Protection Standard aplica tres directivas de mitigación de ajuste automático (TCP SYN, TCP y UDP) a cada dirección IP pública del recurso protegido, en la red virtual que tiene habilitado DDoS. Puede ver los umbrales de directiva si selecciona las métricas **Paquetes entrantes TCP para desencadenar la mitigación de DDoS** y **Paquetes entrantes UDP para desencadenar la mitigación de DDoS** con el tipo de **agregación** como "Máximo", tal como se muestra en la siguiente imagen:

![Vista de directivas de mitigación](./media/manage-ddos-protection/view-mitigation-policies.png)

Los umbrales de las directivas se configuran automáticamente con nuestro sistema de generación de perfiles de tráfico de red basado en aprendizaje automático de Azure. Solo cuando se supera el umbral de la directiva, la mitigación de DDoS se produce para la dirección IP que está siendo atacada.

## <a name="validate-and-test"></a>Validación y prueba

Para simular un ataque DDoS para validar la telemetría de la protección contra DDoS, consulte [Validación de la detección de DDoS](test-through-simulations.md).

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

- Configuración de alertas para métricas de protección contra DDoS
- Ver la telemetría de protección contra DDoS
- Visualización de las directivas de mitigación de DDoS
- Validar y probar la telemetría de protección contra DDoS

Para obtener información sobre cómo configurar los informes de mitigación de ataques y los registros de flujo, continúe con el siguiente tutorial.

> [!div class="nextstepaction"]
> [Visualización y configuración del registro de diagnóstico de DDoS](diagnostic-logging.md)