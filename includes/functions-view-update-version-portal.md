---
title: archivo de inclusión
description: archivo de inclusión
services: functions
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 11/26/2018
ms.author: glenga
ms.custom: include file
ms.openlocfilehash: aa9a715fdafc143a116458691965087b016dec1f
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/19/2021
ms.locfileid: "83343361"
---
Utilice el siguiente procedimiento para ver y actualizar la versión del runtime que utiliza una aplicación de función.

1. En [Azure Portal](https://portal.azure.com), vaya a la aplicación de función.

1. En las **opciones de configuración** haga clic en **Configuración**. En la pestaña **Configuración del entorno de ejecución de Function**, busque la **versión del entorno de ejecución**. Anote la versión específica del entorno de ejecución. En el ejemplo siguiente, la versión se establece en `~3`.

    :::image type="content" source="./media/functions-view-update-version-portal/functions-view-runtime-version.png" alt-text="Consulte la versión del entorno de ejecución." border="true":::

1. Para anclar la aplicación de función a la versión 1.x del entorno de ejecución, elija **~1** en **Versión en tiempo de ejecución**. Este modificador está deshabilitado cuando tiene funciones en la aplicación.

1. Cuando cambie la versión del runtime, vuelva a la pestaña **Información general** y elija **Reiniciar** para reiniciar la aplicación.  La aplicación de función se reinicia y se ejecuta en la versión 1.x del entorno de ejecución y se usan las plantillas de la versión 1.x al crear las funciones.

    :::image type="content" source="./media/functions-view-update-version-portal/functions-restart-function-app.png" alt-text="Reinicie la aplicación de función." border="true":::
