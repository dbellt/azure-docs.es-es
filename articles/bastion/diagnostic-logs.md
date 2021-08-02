---
title: Habilitación y uso de registros de recursos de Azure Bastion
description: Aprenda a habilitar y usar registros de diagnóstico de Azure Bastion.
services: bastion
author: charwen
ms.service: bastion
ms.topic: how-to
ms.date: 02/03/2020
ms.author: charwen
ms.openlocfilehash: 58ad863c1133ee7be984fe723b567ce43a8ef6ad
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110534829"
---
# <a name="enable-and-work-with-bastion-resource-logs"></a>Habilitación y uso de registros de recursos de Bastion

A medida que los usuarios se conectan a las cargas de trabajo mediante Azure Bastion, Bastion puede registrar los diagnósticos de las sesiones remotas. Después, puede usar los diagnósticos para ver qué usuarios se han conectado a las distintas cargas de trabajo, en qué momento, desde dónde y otros datos de registro relevantes. Para poder usar los diagnósticos, debe habilitar los registros de diagnóstico en Azure Bastion. Este artículo le ayuda a habilitar los registros de diagnóstico y a ver los registros.

## <a name="enable-the-resource-log"></a><a name="enable"></a>Habilitación del registro de recursos

1. En [Azure Portal](https://portal.azure.com), navegue hasta el recurso Azure Bastion y seleccione **Configuración de diagnóstico** en la página de Azure Bastion.

   ![Captura de pantalla que muestra la página "Diagnostics settings" (Configuración de diagnóstico).](./media/diagnostic-logs/1diagnostics-settings.png)
2. Seleccione **Configuración de diagnóstico** y seleccione **+ Agregar configuración de diagnóstico** para agregar un destino para los registros.

   ![Captura de pantalla que muestra la página "Diagnostics settings" (Configuración de diagnóstico) con el botón "Add diagnostic setting" (Agregar configuración de diagnóstico) seleccionado.](./media/diagnostic-logs/2add-diagnostic-setting.png)
3. En la página **Configuración de diagnóstico**, seleccione el tipo de cuenta de almacenamiento que se va a usar para almacenar los registros de diagnóstico.

   ![Captura de pantalla de la página "Diagnostics settings" (Configuración de diagnóstico) con la sección para seleccionar una ubicación de almacenamiento resaltada.](./media/diagnostic-logs/3add-storage-account.png)
4. Una vez completada, la configuración tendrá un aspecto similar al de este ejemplo:

   ![Configuración de ejemplo](./media/diagnostic-logs/4example-settings.png)

## <a name="view-diagnostics-log"></a><a name="view"></a>Ver registro de diagnóstico

Para acceder a los registros de diagnóstico, puede usar directamente la cuenta de almacenamiento que especificó al habilitar la configuración de diagnóstico.

1. Navegue hasta el recurso de la cuenta de almacenamiento y, a continuación, vaya a **Contenedores**. Verá el blob **insights-logs-bastionauditlogs** creado en el contenedor de blobs de la cuenta de almacenamiento.

   ![Configuración de diagnóstico](./media/diagnostic-logs/1-navigate-to-logs.png)
2. A medida que se adentre en el contenedor, verá las distintas carpetas del blog. Estas carpetas indican la jerarquía de recursos de su recurso de Azure Bastion.

   ![Agregar configuración de diagnóstico](./media/diagnostic-logs/2-resource-h.png)
3. Vaya a la jerarquía completa del recurso de Azure Bastion que contiene los registros de diagnóstico a los que quiere acceder o que quiere ver. Las letras "y=", "m=", "d=", "h=" y "m=" indican el año, el mes, el día, la hora y los minutos, respectivamente, de los registros de recursos.

   ![Seleccionar la ubicación de almacenamiento](./media/diagnostic-logs/3-resource-location.png)
4. Busque el archivo JSON creado por Azure Bastion que contiene los datos del registro de diagnóstico del período de tiempo al que ha navegado.

5. Descargue el archivo JSON del contenedor de blobs de almacenamiento. A continuación se muestra una entrada de ejemplo del archivo JSON como referencia:

   ```json
   { 
   "time":"2019-10-03T16:03:34.776Z",
   "resourceId":"/SUBSCRIPTIONS/<subscripionID>/RESOURCEGROUPS/MYBASTION/PROVIDERS/MICROSOFT.NETWORK/BASTIONHOSTS/MYBASTION-BASTION",
   "operationName":"Microsoft.Network/BastionHost/connect",
   "category":"BastionAuditLogs",
   "level":"Informational",
   "location":"eastus",
   "properties":{ 
      "userName":"<username>",
      "userAgent":"Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.90 Safari/537.36",
      "clientIpAddress":"131.107.159.86",
      "clientPort":24039,
      "protocol":"ssh",
      "targetResourceId":"/SUBSCRIPTIONS/<subscripionID>/RESOURCEGROUPS/MYBASTION/PROVIDERS/MICROSOFT.COMPUTE/VIRTUALMACHINES/LINUX-KEY",
      "subscriptionId":"<subscripionID>",
      "message":"Successfully Connected.",
      "resourceType":"VM",
      "targetVMIPAddress":"172.16.1.5",
      "tunnelId":"<tunnelID>"
   },
   "FluentdIngestTimestamp":"2019-10-03T16:03:34.0000000Z",
   "Region":"eastus",
   "CustomerSubscriptionId":"<subscripionID>"
   }
   ```

## <a name="next-steps"></a>Pasos siguientes

Lea el artículo sobre [preguntas frecuentes de Bastion](bastion-faq.md).
