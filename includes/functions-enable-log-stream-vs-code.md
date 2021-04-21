---
author: ggailey777
ms.service: azure-functions
ms.topic: include
ms.date: 06/25/2019
ms.author: glenga
ms.openlocfilehash: 437b4ab62cc8c4903af88ca2f9632e89b953c798
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96010491"
---
Para activar los registros de streaming de su aplicación de funciones en Azure:

1. Seleccione F1 para abrir la paleta de comandos y, a continuación, busque y ejecute el comando **Azure Functions: Start Streaming Logs** (Azure Functions: Iniciar transmisión de registros).

1. Seleccione su aplicación de funciones en Azure y seleccione **Yes** (Sí) para habilitar el registro de aplicaciones de la aplicación de funciones.

1. Desencadene las funciones en Azure. Observará que los datos de registro se muestran en la ventana de salida de Visual Studio Code.

1. Cuando haya terminado, recuerde ejecutar el comando **Azure Functions: Stop Streaming Logs** (Azure Functions: Detener transmisión de registros) para deshabilitar el registro de la aplicación de funciones.