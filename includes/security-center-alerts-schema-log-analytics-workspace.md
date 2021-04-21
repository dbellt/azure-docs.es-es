---
title: archivo de inclusión
description: archivo de inclusión
services: data-factory
author: memildin
ms.service: data-factory
ms.topic: include
ms.date: 03/17/2020
ms.author: memildin
ms.custom: include file
ms.openlocfilehash: 7d81799f7fbdb2b41db421daa1a85ec8cde511eb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "79541388"
---
### <a name="the-data-model-of-the-schema"></a>El modelo de datos del esquema

|Campo|Descripción|
|----|----|
|**AlertName**|Nombre para mostrar de la alerta|
|**AlertType**|Identificador único de la alerta|
|**ConfidenceLevel**|(Opcional) El nivel de confianza de esta alerta (alta/baja)|
|**ConfidenceScore**|(Opcional) Indicador de confianza numérico de la alerta de seguridad|
|**Descripción**|Texto de descripción para la alerta|
|**DisplayName**|Nombre para mostrar de la alerta|
|**EndTime**|Hora de finalización del impacto de la alerta (es la hora del último evento que contribuyó a la alerta)|
|**Entidades**|Lista de entidades relacionadas con la alerta. Esta lista puede contener una combinación de entidades de diversos tipos|
|**ExtendedLinks**|(Opcional) Un contenedor para todos los vínculos relacionados con la alerta. Este contenedor puede contener una combinación de vínculos para diversos tipos|
|**ExtendedProperties**|Un contenedor de campos adicionales que son pertinentes para la alerta|
|**IsIncident**|Determina si la alerta es un incidente o una alerta normal. Un incidente es una alerta de seguridad que agrega varias alertas a un incidente de seguridad.|
|**ProcessingEndTime**|Marca de tiempo UTC en la que se creó la alerta|
|**ProductComponentName**|(Opcional) Nombre de un componente dentro del producto que generó la alerta|
|**ProductName**|Constante ("Azure Security Center")|
|**ProviderName**|unused|
|**RemediationSteps**|Elementos de acción manual que se deben llevar a cabo para corregir la amenaza de seguridad|
|**ResourceId**|Identificador completo del recurso afectado|
|**Gravedad**|La gravedad de alerta (alta/media/baja/informativa)|
|**SourceComputerId**|Un GUID único para el servidor afectado (si la alerta se genera en el servidor)|
|**SourceSystem**|unused|
|**StartTime**|Hora de inicio del impacto de la alerta (es la hora del primer evento que contribuyó a la alerta)|
|**SystemAlertId**|Identificador único de esta instancia de alerta de seguridad|
|**TenantId**|El identificador del inquilino principal de Azure Active Directory de la suscripción en la que reside el recurso examinado|
|**TimeGenerated**|Marca de tiempo UTC en la que tuvo lugar la evaluación (hora del examen de Security Center) (idéntica a DiscoveredTimeUTC)|
|**Tipo**|Constante ("SecurityAlert")|
|**VendorName**|Nombre del proveedor que proporcionó la alerta (p. ej., "Microsoft").|
|**VendorOriginalId**|unused|
|**WorkspaceResourceGroup**|En caso de que la alerta se genere en una instancia de máquina virtual, de Server, del conjunto de escalado de máquinas virtuales o de App Service que informe a un área de trabajo, contiene ese nombre del grupo de recursos del área de trabajo|
|**WorkspaceSubscriptionId**|En caso de que la alerta se genere en una instancia de máquina virtual, de Server, del conjunto de escalado de máquinas virtuales o de App Service que informe a un área de trabajo, contiene ese subscriptionId del área de trabajo|
|||
