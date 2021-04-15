---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 04/15/2019
ms.author: alkohli
ms.openlocfilehash: 9ac7966538102273b91d6b7f15b90e18ceedd421
ms.sourcegitcommit: 867cb1b7a1f3a1f0b427282c648d411d0ca4f81f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/19/2021
ms.locfileid: "83779930"
---
También puede ver las métricas para supervisar el rendimiento del dispositivo y, en algunos casos, para solucionar problemas del dispositivo.

Para crear un gráfico para las métricas del dispositivo seleccionadas, siga estos pasos en Azure Portal.

1. Para el recurso en Azure Portal, vaya a **Supervisión > Métricas** y seleccione **Agregar métrica**.

    ![Adición de métrica](media/data-box-edge-gateway-view-metrics/view-metrics-1.png)

2. El nombre del recurso se rellena automáticamente.  

    ![Recurso actual](media/data-box-edge-gateway-view-metrics/view-metrics-2.png)

    Para especificar otro recurso, selecciónelo. En la hoja **Seleccionar un recurso**, seleccione la suscripción, el grupo de recursos, el tipo de recurso y el recurso específico para el que quiera mostrar las métricas y seleccione **Aplicar**.

    ![Selección de otro recurso](media/data-box-edge-gateway-view-metrics/view-metrics-3.png)

3. En la lista desplegable, seleccione una métrica para supervisar el dispositivo. Para obtener una lista completa de estas métricas, consulte [Métricas sobre el dispositivo](#metrics-on-your-device).

4. Cuando se selecciona una métrica en la lista desplegable, también se pueden definir agregaciones. La agregación hace referencia al valor real que se agrega durante un intervalo de tiempo especificado. Los valores agregados pueden ser un valor medio, un valor mín. o un valor máx. Seleccione el valor que quiera para la agregación: medio, mín. o máx.

    ![Visualización del gráfico](media/data-box-edge-gateway-view-metrics/view-metrics-4.png)

5. Si la métrica que seleccionó tiene varias instancias, tendrá disponible la opción de división. Seleccione **Aplicar división** y, después, seleccione el valor por el que quiere ver el desglose.

    ![Aplicación de la división](media/data-box-edge-gateway-view-metrics/view-metrics-5.png)

6. Si solo quiere ver el desglose para algunas instancias, puede filtrar los datos. Por ejemplo, en este caso, si solo quiere ver el rendimiento de la red para las dos interfaces de red conectadas en el dispositivo, se podrían filtrar esas interfaces. Seleccione **Agregar filtro** y especifique el nombre de la interfaz de red para el filtrado.

    ![Adición de filtro](media/data-box-edge-gateway-view-metrics/view-metrics-6.png)

7. También puede anclar el gráfico al panel para facilitar el acceso.

    ![Anclar al panel](media/data-box-edge-gateway-view-metrics/view-metrics-7.png)

8. Para exportar los datos del gráfico a una hoja de cálculo de Excel u obtener un vínculo al gráfico para poder compartirlo, seleccione la opción de recurso compartido en la barra de comandos.

    ![Exportar datos](media/data-box-edge-gateway-view-metrics/view-metrics-8.png)
