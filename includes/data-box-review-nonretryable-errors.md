---
author: v-dalc
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 05/07/2021
ms.author: alkohli
ms.openlocfilehash: 6ccf37d96da5b52111fd08a51de4949185891193
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109736199"
---
Para revisar los errores que no se pueden reintentar y continuar con la orden, haga lo siguiente:

1. Abra la orden en Azure Portal.  

   Si alguno de los errores que no se puede reintentar impide que se carguen archivos, verá la siguiente notificación. El estado actual de la orden será **Data copy completed with errors. Device pending data erasure** (Copia de datos completada con errores. Eliminación de datos pendientes del dispositivo).

   ![Notificación de errores de copia durante una carga en Azure Portal](media/data-box-review-nonretryable-errors/copy-errors-in-upload-01.png)

   Anote el valor de **COPY LOG PATH** en **DATA COPY DETAILS**. Revisará los errores en el registro de copia de datos.

2. Seleccione **Confirm device erasure** (Confirmar borrado del dispositivo) para abrir un panel de revisión.

   ![Realice la revisión y vaya al panel para ver si hay errores de carga en Azure Portal](media/data-box-review-nonretryable-errors/copy-errors-in-upload-02.png)

3. Revise los errores del registro de copia de datos mediante la ruta de acceso del registro de copia que anotó anteriormente. Si es necesario, puede seleccionar **Cerrar** para volver a mostrar la ruta de acceso. 

   Deberá corregir los problemas de configuración antes de probar otra carga a través de una transferencia de red o una nuevo orden de importación. <!--For guidance, see [Review copy errors in uploads from Azure Data Box and Azure Data Box Heavy devices](../articles/databox/data-box-troubleshoot-data-upload.md). - To make the Include, I needed to move this reference out of the main procedure.-->

4. Después de revisar los errores, seleccione la casilla para confirmar que está listo para continuar con el borrado de datos. Luego, seleccione **Continuar**.

   ![Confirme que está listo para continuar con la eliminación de datos.](media/data-box-review-nonretryable-errors/copy-errors-in-upload-03.png)

   Una vez que los datos se borran del dispositivo de forma segura, el estado de la orden se actualiza a **Copy completed with errors (Copia completada con errores)** .

   ![Visualización del estado de una orden de importación de Data Box que se completó con errores](media/data-box-review-nonretryable-errors/copy-errors-in-upload-04.png)

   Si no realiza ninguna acción, la orden se completa automáticamente después de 14 días.

