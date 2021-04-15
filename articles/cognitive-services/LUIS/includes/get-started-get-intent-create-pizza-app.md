---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.date: 12/29/2020
ms.subservice: language-understanding
ms.topic: include
ms.custom: include file
ms.author: aahi
ms.reviewer: roy-har
ms.openlocfilehash: 7aa2fba6ef551a745ccaf5b00f36021b9d8680ce
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "97820789"
---
1. Seleccione [pizza-app-for-luis-v6.json](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/luis/apps/pizza-app-for-luis-v6.json) para abrir la página de GitHub para el archivo `pizza-app-for-luis.json`.
1. Pulse de forma prolongada o haga clic con el botón derecho en el botón **Sin formato** y seleccione **Guardar vínculo como** para guardar el archivo `pizza-app-for-luis.json` en el equipo.
1. Inicie sesión en el [portal de LUIS](https://www.luis.ai).
1. Seleccione [Mis aplicaciones](https://www.luis.ai/applications).
1. En la página **Mis aplicaciones**, seleccione **+ Nueva aplicación para la conversación**.
1. Seleccione **Importar como JSON**.
1. En el cuadro de diálogo **Importar nueva aplicación**, seleccione el botón **Elegir archivo**.
1. Seleccione el archivo de `pizza-app-for-luis.json` que descargó y, a continuación, seleccione **Abrir**.
1. En el cuadro de diálogo **Importar nueva aplicación** en el campo **Nombre**, escriba un nombre para la aplicación de pizza y, a continuación, seleccione el botón **Listo**.

Se importará la aplicación.

Si ve el cuadro de diálogo **Cómo crear una aplicación eficaz de LUIS**, ciérrelo.

## <a name="train-and-publish-the-pizza-app"></a>Entrene y publique la aplicación de pizza

Debería consultar la página **Intenciones** con una lista de las intenciones de la aplicación de pizza.

[!INCLUDE [How to train](howto-train.md)]

[!INCLUDE [How to publish](howto-publish.md)]

La aplicación de pizza ya está lista para usarse.

## <a name="record-the-app-id-prediction-key-and-prediction-endpoint-of-your-pizza-app"></a>Registre el identificador de la aplicación, la clave de predicción y el punto de conexión de predicción de la aplicación de pizza

Para usar la nueva aplicación de pizza, necesitará el identificador de la aplicación, la clave de predicción y el punto de conexión de predicción de la aplicación de pizza.

Para establecer estos valores:

1. En la página **Intenciones**, seleccione **ADMINISTRAR**.
1. En la página **Configuración de la aplicación**, registre el **Id. de aplicación**.
1. Seleccione **Recursos de Azure**.
1. En la página **Recursos de Azure**, registre la **Clave principal**. Este valor es su clave de predicción.
1. Registre la **Dirección URL del punto de conexión**. Este valor es su punto de conexión de predicción.
