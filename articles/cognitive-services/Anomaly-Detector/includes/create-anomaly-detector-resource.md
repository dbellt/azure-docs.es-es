---
title: Compatibilidad con los contenedores
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/10/2020
ms.author: mbullwin
ms.openlocfilehash: feb79d047a6c3b25176a13dcc3c3afd53a51459e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "102445376"
---
## <a name="create-an-anomaly-detector-resource"></a>Creación de un recurso de Anomaly Detector

1. Inicie sesión en <a href="https://portal.azure.com" target="_blank">Azure Portal</a>.
1. Haga clic en <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesAnomalyDetector" target="_blank">Creación de un recurso de Anomaly Detector</a>.
1. Establezca todas las opciones de configuración necesarias:

    |Configuración|Value|
    |--|--|
    |Nombre|Nombre que quiera (2-64 caracteres).|
    |Subscription|Seleccione una suscripción adecuada.|
    |Location|Seleccione cualquier ubicación disponible cercana.|
    |Nivel de precios|`F0`: 10 llamadas por segundo, 20 000 transacciones al mes. <br> O:<br> `S0`: 80 llamadas por segundo.|
    |Grupo de recursos|Seleccione un grupo de recursos disponible.|

1. Haga clic en **Crear** y espere a que el recurso se cree. Una vez creado, vaya a la página de recursos.
1. Recopile el elemento `endpoint` configurado y una clave de API:

    |Pestaña de claves y punto de conexión en el portal|Configuración|Value|
    |--|--|--|
    |**Información general**|Punto de conexión|Copie el punto de conexión. Tiene un aspecto similar a ` https://<your-resource-name>.cognitiveservices.azure.com/`.|
    |**Claves**|Clave de API|Copie una de las dos claves. Es una cadena de 32 caracteres alfanuméricos sin espacios ni guiones, `xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`.|



