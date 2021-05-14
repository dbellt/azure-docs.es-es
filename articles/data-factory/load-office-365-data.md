---
title: Carga de datos de Office 365 mediante Azure Data Factory
description: Uso de Azure Data Factory para copiar datos de Office 365
author: jianleishen
ms.service: data-factory
ms.topic: conceptual
ms.date: 02/18/2021
ms.author: jianleishen
ms.openlocfilehash: eae3a83a5354c61db27f38b7b73a396923427073
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109485628"
---
# <a name="load-data-from-office-365-by-using-azure-data-factory"></a>Carga de datos de Office 365 mediante Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo se muestra cómo usar Data Factory para _cargar datos de Office 365 en Azure Blob Storage_. Puede seguir pasos similares para copiar datos en Azure Data Lake Gen1 o Gen2. Consulte el [artículo del conector de Office 365](connector-office-365.md) sobre la copia de datos de Office 365 en general.

## <a name="create-a-data-factory"></a>Crear una factoría de datos

1. En el menú de la izquierda, seleccione **Crear un recurso** > **Analytics** > **Data Factory**: 
   
   ![Selección de la factoría de datos en el panel Nuevo](./media/quickstart-create-data-factory-portal/new-azure-data-factory-menu.png)

2. En la página **Nueva factoría de datos**, proporcione los valores de los campos que se muestran en la imagen siguiente:
      
   ![Página New data factory (Nueva factoría de datos)](./media/load-office-365-data/new-azure-data-factory.png)
 
    * **Name**: escriba un nombre único global para la factoría de datos de Azure. Si recibe el error "El nombre *LoadFromOffice365Demo* de factoría de datos no está disponible", escriba un nombre diferente para la factoría de datos. Por ejemplo, podría usar el nombre _**su nombre**_ **LoadFromOffice365Demo**. Intente crear de nuevo la factoría de datos. Para conocer las reglas de nomenclatura de los artefactos de Data Factory, consulte [Data Factory: reglas de nomenclatura](naming-rules.md).
    * **Suscripción**: seleccione la suscripción de Azure donde desea crear la factoría de datos. 
    * **Grupo de recursos**: seleccione un grupo de recursos existente en la lista desplegable o seleccione la opción **Crear nuevo** y escriba el nombre de un grupo de recursos. Para obtener más información sobre los grupos de recursos, consulte [Uso de grupos de recursos para administrar los recursos de Azure](../azure-resource-manager/management/overview.md).  
    * **Versión**: seleccione **V2**.
    * **Ubicación**: Seleccione la ubicación de la factoría de datos. Solo las ubicaciones admitidas se muestran en la lista desplegable. Los almacenes de datos que las factorías de datos usan pueden estar en otras ubicaciones y regiones. Estos almacenes de datos incluyen Azure Data Lake Store, Azure Storage, Azure SQL Database, etc.

3. Seleccione **Crear**.
4. Una vez completada la creación, vaya a la factoría de datos. Verá la página principal de **Factoría de datos**, tal y como se muestra en la siguiente imagen:
   
   ![Página principal Factoría de datos](./media/load-office-365-data/data-factory-home-page.png)

5. Seleccione el icono **Author & Monitor** (Creación y supervisión) para iniciar la aplicación de integración de datos en una pestaña independiente.

## <a name="create-a-pipeline"></a>Crear una canalización

1. En la página de introducción seleccione **Create pipeline** (Crear canalización).
 
    ![Creación de una canalización](./media/load-office-365-data/create-pipeline-entry.png)

2. En la pestaña **General** de la canalización, escriba "CopyPipeline" como **nombre** de la canalización.

3. En el cuadro de herramientas Activities (Actividades) > categoría Move & Transform (Mover y transformar), arrastre la actividad **Copy** (Copia) desde el cuadro de herramientas hasta la superficie del diseñador de canalizaciones. Especifique "CopyFromOffice365ToBlob" como nombre de la actividad.

### <a name="configure-source"></a>Configuración del origen

1. Vaya a la canalización > **pestaña Source** (Origen) y haga clic en **+ New** (+Nuevo) para crear un conjunto de datos de origen. 

2. En la ventana del nuevo conjunto de datos, seleccione **Office 365** y, a continuación, seleccione **Continue** (Continuar).
 
3. Ahora se encuentra en la pestaña de configuración de la actividad de copia. Haga clic en el botón **Edit** (Editar) situado junto al conjunto de datos de Office 365 para continuar con la configuración de los datos.

    ![Configuración de Office 365: conjunto de datos general](./media/load-office-365-data/transition-to-edit-dataset.png)
 
4. Verá que se abre una nueva pestaña para el conjunto de datos de Office 365. En la pestaña **General** de la parte inferior de la ventana Propiedades, escriba "SourceOffice365Dataset" como nombre.
 
5. Vaya a la pestaña **Connection** (Conexión) de la ventana de propiedades. Junto al cuadro de texto del servicio vinculado, haga clic en **+ New** (+Nuevo).

6. En la ventana del nuevo servicio vinculado, escriba "Office365LinkedService" como nombre, escriba el identificador y la clave de la entidad de servicio y, luego, seleccione **Create** (Crear) para implementar el servicio vinculado.

    ![Nuevo servicio vinculado de Office 365](./media/load-office-365-data/new-office-365-linked-service.png)
 
7. Una vez que se ha creado el servicio vinculado, volverá a la configuración del conjunto de datos. Junto a **Table** (Tabla), elija la flecha hacia abajo para expandir la lista de conjuntos de datos de Office 365 disponibles y seleccione "BasicDataSet_v0.Message_v0" en la lista desplegable:

    ![Configuración de Office 365: tabla de conjunto de datos](./media/load-office-365-data/edit-dataset.png)

8. Ahora, vuelva a la pestaña **pipeline** > **Source** (Origen de canalización) para continuar con la configuración de las propiedades adicionales de la extracción de datos de Office 365.  El ámbito de usuario y el filtro de ámbito de usuario son predicados opcionales que puede definir para restringir los datos que desea extraer de Office 365. Consulte la sección [Propiedades del conjunto de datos de Office 365](./connector-office-365.md#dataset-properties) sobre cómo configurar estas opciones.

9. Se le pedirá que elija uno de los filtros de fecha y que proporcione los valores de hora de inicio y de finalización.

10. Haga clic en la pestaña **Import Schema** (Importar esquema) para importar el esquema del conjunto de datos de los mensajes.

    ![Configuración de Office 365: esquema de conjunto de datos](./media/load-office-365-data/edit-source-properties.png)

### <a name="configure-sink"></a>Configuración del receptor

1. Vaya a la canalización > **pestaña Sink** (Receptor) y seleccione **+ New** (+Nuevo) para crear un conjunto de datos del receptor.
 
2. En la ventana del nuevo conjunto de datos, observe que solo se selecciona el destino admitido al copiar desde Office 365. Seleccione **Azure Blob Storage**, seleccione el formato binario y, después, **Continue** (Continuar).  En este tutorial, copiará datos de Office 365 en una instancia de Azure Blob Storage.

3. Haga clic en el botón **Edit** (Editar) situado junto al conjunto de datos de Azure Blob Storage para continuar con la configuración de los datos.

4. En la pestaña **General** de la ventana de propiedades, escriba "OutputBlobDataset" como nombre.

5. Vaya a la pestaña **Connection** (Conexión) de la ventana de propiedades. Junto al cuadro de texto de servicio vinculado, seleccione **+ New** (+Nuevo).

6. En la ventana del nuevo servicio vinculado, escriba "AzureStorageLinkedService" como nombre, seleccione la entidad de servicio en la lista desplegable de métodos de autenticación, rellene el punto de conexión de servicio, el identificador de la entidad de servicio del inquilino y la clave de la entidad de servicio y, luego, seleccione Save (Guardar) para implementar el servicio vinculado.  Consulte [aquí](connector-azure-blob-storage.md#service-principal-authentication) acerca de cómo configurar la autenticación de la entidad de servicio para Azure Blob Storage.

    ![Nuevo servicio vinculado de blob](./media/load-office-365-data/configure-blob-linked-service.png)


## <a name="validate-the-pipeline"></a>Comprobación de la canalización

Para validar la canalización, seleccione **Validar** en la barra de herramientas.

También puede ver el código JSON asociado a la canalización. Para ello, haga clic en Code (Código) en la parte superior derecha.

## <a name="publish-the-pipeline"></a>Publicación de la canalización

En la barra de herramientas superior, seleccione **Publish All** (Publicar todo). Esta acción publica las entidades (conjuntos de datos y canalizaciones) que creó para Data Factory.

![Publicación de cambios](./media/load-office-365-data/publish-changes.png) 

## <a name="trigger-the-pipeline-manually"></a>Desencadenamiento manual de la canalización

Seleccione **Add Trigger** (Agregar desencadenador) en la barra de herramientas y, después, seleccione **Trigger Now** (Desencadenar ahora). En la página de ejecución de la canalización, seleccione **Finish** (Finalizar). 

## <a name="monitor-the-pipeline"></a>Supervisar la canalización

Vaya a la pestaña **Monitor** (Supervisar) de la izquierda. Verá una ejecución de canalización que se desencadena de forma manual. Puede usar los vínculos de la columna **Actions** (Acciones) para ver los detalles de la actividad y volver a ejecutar la canalización.

![Supervisión de la canalización](./media/load-office-365-data/pipeline-status.png) 

Para ver las ejecuciones de actividad asociadas con la ejecución de canalización, seleccione el vínculo **View Activity Runs** (Ver ejecuciones de actividad) en la columna de acciones. En este ejemplo, solo hay una actividad, así que solo verá una entrada en la lista. Para ver más información sobre la operación de copia, seleccione el vínculo **Details** (Detalles) (icono de gafas) en la columna de acciones.

![Supervisión de la actividad](./media/load-office-365-data/activity-status.png) 

Si es la primera vez que solicita datos para este contexto (una combinación de los datos de tabla a los que se accede, la cuenta de destino en la que se cargan los datos y la identidad de usuario que realiza la solicitud de acceso a los datos), verá el estado de la actividad de copia como **In Progress** (En curso), y solo al hacer clic en el vínculo de detalles en Actions (Acciones), verá el estado como "**RequesetingConsent**".  Un miembro del grupo de aprobadores de acceso a datos debe aprobar la solicitud en Privileged Access Management antes de pasar a la extracción de datos.

_Estado como solicitando consentimiento:_ 
![Detalles de la ejecución de la actividad: solicitar consentimiento](./media/load-office-365-data/activity-details-request-consent.png) 

_Estado como extrayendo datos:_

![Detalles de ejecución de la actividad: extraer datos](./media/load-office-365-data/activity-details-extract-data.png) 

Una vez que se dé el consentimiento, la extracción de los datos continuará y, al cabo de un tiempo, la ejecución de la canalización se mostrará como completada.

![Supervisión de la canalización: correcta](./media/load-office-365-data/pipeline-monitoring-succeeded.png) 

Ahora, vaya a la instancia de Azure Blob Storage de destino y compruebe que Office 365 se ha extraído en formato binario.

## <a name="next-steps"></a>Pasos siguientes

Continúe al artículo siguiente para obtener información sobre la compatibilidad de Azure Synapse Analytics: 

> [!div class="nextstepaction"]
>[Conector de Office 365](connector-office-365.md)