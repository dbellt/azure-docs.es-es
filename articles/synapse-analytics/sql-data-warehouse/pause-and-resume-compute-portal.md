---
title: 'Inicio rápido: Pausa y reanudación del proceso en un grupo de SQL dedicado mediante Azure Portal'
description: Use Azure Portal para poner en pausa el proceso en el grupo de SQL dedicado para ahorrar costos. Reanude el proceso cuando esté listo para usar el almacenamiento de datos.
services: synapse-analytics
author: julieMSFT
ms.author: jrasnick
manager: craigg
ms.reviewer: igorstan
ms.date: 11/23/2020
ms.topic: quickstart
ms.service: synapse-analytics
ms.subservice: sql-dw
ms.custom:
- seo-lt-2019
- azure-synapse
- mode-portal
ms.openlocfilehash: 27587f436013bcaccbacd668ce62f1d939f89e48
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107566824"
---
# <a name="quickstart-pause-and-resume-compute-in-dedicated-sql-pool-via-the-azure-portal"></a>Inicio rápido: Pausa y reanudación del proceso en un grupo de SQL dedicado mediante Azure Portal

Puede usar Azure Portal para poner en pausa y reanudar los recursos de proceso del grupo de SQL dedicado. Si no tiene una suscripción a Azure, cree una cuenta [gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="before-you-begin"></a>Antes de empezar

Use [Creación y conexión en el portal](../quickstart-create-sql-pool-portal.md) para crear un grupo de SQL dedicado llamado **mySampleDataWarehouse**. 

## <a name="pause-compute"></a>Pausa del proceso

Para reducir costos, puede pausar y reanudar los recursos de proceso a petición. Por ejemplo, si no va a usar la base de datos durante la noche y los fines de semana, puede pausarla durante esas horas y reanudarla durante el día.
 
>[!NOTE]
>No se le cobrará por recursos de proceso mientras la base de datos se encuentre en pausa. Sin embargo, se le seguirá cobrando por el almacenamiento. 

Para pausar un grupo de SQL dedicado, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Vaya a la página **Dedicated SQL pool** (Grupo de SQL dedicado) para abrir el grupo de SQL. 
3. Observe que el valor del campo **Status** (Estado) es **Online** (En línea).

    ![Proceso en línea](././media/pause-and-resume-compute-portal/compute-online.png)

4. Para pausar el grupo de SQL dedicado, haga clic en el botón **Pausar**. 
5. Aparece una pregunta de confirmación en la que tiene que indicar si quiera continuar. Haga clic en **Sí**.
6. Espere un momento y, a continuación, observe que el **Estado** sea **Pausando**.

    ![Captura de pantalla que muestra Azure Portal para un almacenamiento de datos de ejemplo con un valor de estado Pausando.](./media/pause-and-resume-compute-portal/pausing.png)

7. Una vez completada la operación de pausa, el estado es **En pausa** y el botón de opción es **Reanudar**.
8. Ahora, los recursos de proceso del grupo de SQL dedicado están sin conexión. No se le cobrará el proceso hasta que reanude el servicio.

    ![Proceso sin conexión](././media/pause-and-resume-compute-portal/compute-offline.png)


## <a name="resume-compute"></a>Reanudación del proceso

Para reanudar un grupo de SQL dedicado, siga estos pasos:

1. Vaya a la página **Dedicated SQL pool** (Grupo de SQL dedicado) para abrir el grupo de SQL.
3. En la página **mySampleDataWarehouse**, observe que el **Estado** sea **En pausa**.

    ![Proceso sin conexión](././media/pause-and-resume-compute-portal/compute-offline.png)

1. Para reanudar el grupo de SQL, haga clic en **Reanudar**. 
1. Aparece una pregunta de confirmación en la que tiene que indicar si lo quiere iniciar. Haga clic en **Sí**.
1. Observe que el **Estado** sea **Reanudando**.

    ![Captura de pantalla que muestra Azure Portal para un almacenamiento de datos de ejemplo con el botón Iniciar seleccionado y un valor de estado Reanudando.](./media/pause-and-resume-compute-portal/resuming.png)

1. Cuando el grupo de SQL vuelva a estar en línea, el estado será **En línea** y el botón de opción será **Pausar**.
1. Los recursos de proceso para el grupo de SQL ahora están en línea y se puede usar el servicio. Los cargos del proceso se han reanudado.

    ![Proceso en línea](././media/pause-and-resume-compute-portal/compute-online.png)

## <a name="clean-up-resources"></a>Limpieza de recursos

Se le cobra por las unidades de almacenamiento de datos y los datos almacenados en el grupo de SQL dedicado. Estos recursos de proceso y de almacenamiento se facturan por separado. 

- Si quiere conservar los datos de almacenamiento, pause el proceso.
- Si quiere eliminar cobros futuros, puede eliminar el grupo de SQL dedicado. 

Siga estos pasos para limpiar los recursos según estime oportuno.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y seleccione el grupo de SQL dedicado.

    ![Limpieza de recursos](./media/pause-and-resume-compute-portal/clean-up-resources.png)

1. Para pausar el proceso, haga clic en el botón **Pausar**. 

1. Para quitar el grupo de SQL dedicado para que no le cobren por proceso o almacenamiento, haga clic en **Eliminar**.



## <a name="next-steps"></a>Pasos siguientes

Ya ha pausado y reanudado el proceso para el grupo de SQL dedicado. Continúe con el siguiente artículo para más información sobre cómo [cargar datos en un grupo de SQL dedicado](./load-data-from-azure-blob-storage-using-copy.md). Para más información acerca de la administración de funcionalidades de proceso, consulte el artículo sobre [introducción a la administración de proceso](sql-data-warehouse-manage-compute-overview.md).
