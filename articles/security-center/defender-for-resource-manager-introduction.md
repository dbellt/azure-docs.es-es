---
title: 'Azure Defender para Resource Manager: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para Resource Manager
author: memildin
ms.author: memildin
ms.date: 12/07/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 71209d9e20ec0f0cbc263e84d04bf8eba6e98b99
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108279071"
---
# <a name="introduction-to-azure-defender-for-resource-manager"></a>Introducción a Azure Defender para Resource Manager

[Azure Resource Manager](../azure-resource-manager/management/overview.md) es el servicio de implementación y administración para Azure. Proporciona una capa de administración que le permite crear, actualizar y eliminar recursos de la cuenta de Azure. Se usan las características de administración, como el control de acceso, la auditoría y las etiquetas, para proteger y organizar los recursos después de la implementación.

La capa de administración de la nube es un servicio crucial conectado a todos los recursos en la nube. Ese el motivo por el que es también es un objetivo potencial para los atacantes. Por lo tanto, se recomienda que los equipos de operaciones de seguridad supervisen estrechamente la capa de administración de recursos. 

Azure Defender para Resource Manager supervisa automáticamente las operaciones de administración de recursos de cualquier organización, independientemente de que se realicen a través de Azure Portal, las API REST de Azure, la CLI de Azure u otros clientes de programación de Azure. Azure Defender ejecuta análisis de seguridad avanzados para detectar amenazas y avisarle de cualquier actividad sospechosa.

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Versión preliminar<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)] |
|Precios:|**Azure Defender para Resource Manager** se factura como se muestra en [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-resource-manager"></a>¿Cuáles son las ventajas de Azure Defender para Resource Manager?

Azure Defender para Resource Manager sirve de protección para los siguientes problemas:

- **Operaciones de administración de recursos sospechosas**, como las operaciones desde direcciones IP malintencionadas, la deshabilitación de antimalware y scripts sospechosos que se ejecutan en las extensiones de máquina virtual
- **Uso de kits de herramientas de explotación** como Microburst o PowerZure
- **Movimiento lateral** del nivel de administración de Azure al plano de datos de los recursos de Azure

:::image type="content" source="media/defender-for-resource-manager-introduction/consistent-management-layer-with-defender.png" alt-text="Diagrama con información general sobre Azure Resource Manager":::

En la [página de referencia de alertas](alerts-reference.md#alerts-resourcemanager) se encuentra una lista completa de las alertas que proporciona Azure Defender para Resource Manager.


 ## <a name="how-to-investigate-alerts-from-azure-defender-for-resource-manager"></a>Investigación de alertas de Azure Defender para Resource Manager

Las alertas de seguridad de Azure Defender para Resource Manager se basan en las amenazas detectadas por la supervisión de operaciones de Azure Resource Manager. Azure Defender usa orígenes de registros internos de Azure Resource Manager, así como el registro de actividad de Azure, un registro de plataforma en Azure que proporciona información detallada sobre los eventos de nivel de suscripción.

Obtenga más información sobre el [registros de actividad de Azure](../azure-monitor/essentials/activity-log.md).

Para investigar las alertas de Azure Defender para Resource Manager:

1. Abra el registro de actividad de Azure.

    :::image type="content" source="media/defender-for-resource-manager-introduction/opening-azure-activity-log.png" alt-text="Cómo abrir el registro de actividad de Azure":::

1. Filtre los eventos para:
    - La suscripción mencionada en la alerta
    - El período de tiempo de la actividad detectada
    - La cuenta de usuario relacionada (si procede)

1. Busque actividades sospechosas.

> [!TIP]
> Para disfrutar de una experiencia de investigación mejor y más rica, transmita los registros de actividad de Azure a Azure Sentinel como se describe en [Conexión de datos del registro de actividad de Azure](../sentinel/connect-azure-activity.md).



## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para Resource Manager. Para obtener material relacionado, vea el siguiente artículo: 

- Las alertas de seguridad las puede haber generado Security Center, o bien las puede haber recibido Security Center de distintos productos de seguridad. Para exportar todas estas alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md).

- > [!div class="nextstepaction"]
    > [Habilitación de Azure Defender](enable-azure-defender.md)