---
title: Validación de XML con esquemas
description: Incorporación de esquemas para validar documentos XML en Azure Logic Apps con Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/27/2021
ms.openlocfilehash: 39fc84e9a83f7417836c895481da27a0d00cc71f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108137191"
---
# <a name="validate-xml-with-schemas-in-azure-logic-apps-with-enterprise-integration-pack"></a>Validación XML con esquemas en Azure Logic Apps con Enterprise Integration Pack

Para comprobar que los documentos usan XML válidos y cuentan con los datos esperados en el formato predefinido para escenarios de integración empresarial en Azure Logic Apps, la aplicación lógica puede usar esquemas. Un esquema también puede validar los mensajes que las aplicaciones lógicas intercambia en escenarios negocio a negocio (B2B).

Para conocer los límites relacionados con las cuentas de integración y artefactos como esquemas, consulte [Información de límites y configuración para Azure Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene una, <a href="https://azure.microsoft.com/free/" target="_blank">regístrese para obtener una cuenta de Azure gratuita</a>.

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) donde almacena los esquemas y otros artefactos para soluciones negocio a negocio (B2B) y de integración empresarial. 

  Si el esquema es de [2 MB o menos](#smaller-schema), puede agregarlo a la cuenta de integración directamente desde Azure Portal. Sin embargo, si el esquema tiene más de 2 MB pero no más que el [límite de tamaño del esquema](../logic-apps/logic-apps-limits-and-config.md#artifact-capacity-limits), puede cargarlo en una cuenta de almacenamiento de Azure. 
  Para agregar ese esquema a la cuenta de integración, puede vincular a la cuenta de almacenamiento desde la cuenta de integración. 
  Para esta tarea, necesita estos elementos: 

  * La [cuenta de almacenamiento de Azure](../storage/common/storage-account-overview.md) donde crea un contenedor de blobs para el esquema. Aprenda a [crear una cuenta de almacenamiento](../storage/common/storage-account-create.md). 

  * El contenedor de blobs para almacenar el esquema. Aprenda a [crear un contenedor de blobs](../storage/blobs/storage-quickstart-blobs-portal.md). 
  Necesitará el URI de contenido del contenedor más adelante, cuando agregue el esquema a la cuenta de integración.

  * El [Explorador de Azure Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md), que puede usar para administrar las cuentas de almacenamiento y los contenedores de blobs. 
  Para usar el Explorador de Storage, elija una de estas opciones:
  
    * En Azure Portal, busque y seleccione la cuenta de almacenamiento. 
    En el menú de la cuenta de almacenamiento, seleccione **Explorador de Storage**.

    * Para la versión de escritorio, [descargue e instale el Explorador de Azure Storage](https://www.storageexplorer.com/). 
    Luego, para conectarlo con su cuenta de almacenamiento, siga los pasos que aparecen en [Introducción al Explorador de Storage](../vs-azure-tools-storage-manage-with-storage-explorer.md). 
    Para más información, vea esta [Inicio rápido: Cree un blob en el almacenamiento de objetos con el Explorador de Azure Storage](../storage/blobs/storage-quickstart-blobs-storage-explorer.md).

No necesita una aplicación lógica al crear y agregar esquemas. Sin embargo, al usar un esquema, la aplicación lógica se debe vincular a una cuenta de integración donde se almacena ese esquema. Aprenda a [vincular aplicaciones lógicas a cuentas de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md#link-account). Si aún no tiene una aplicación lógica, obtenga información sobre [cómo crear aplicaciones lógicas](../logic-apps/quickstart-create-first-logic-app-workflow.md).

## <a name="add-schemas"></a>Incorporación de esquemas

1. Inicie sesión en <a href="https://portal.azure.com" target="_blank">Azure Portal</a> con sus credenciales de su cuenta de Azure.

1. Para buscar y abrir la cuenta de integración, en el menú principal de Azure, seleccione **Todos los servicios**. En el cuadro de búsqueda, escriba "cuenta de integración". Seleccione **Cuentas de integración**.

   ![Búsqueda de la cuenta de integración](./media/logic-apps-enterprise-integration-schemas/find-integration-account.png)

1. Seleccione la cuenta de integración en la que quiere agregar el esquema, por ejemplo:

   ![Seleccionar cuenta de integración](./media/logic-apps-enterprise-integration-schemas/select-integration-account.png)

1. En la página **Información general** de la cuenta de integración, en **Componentes**, seleccione el icono **Esquemas**.

   ![Selección de "Esquemas"](./media/logic-apps-enterprise-integration-schemas/select-schemas.png)

1. Una vez que se abra la página **Esquemas**, seleccione **Agregar**.

   ![Elección de "Agregar"](./media/logic-apps-enterprise-integration-schemas/add-schema.png)

Según el tamaño de archivo del esquema (.xsd), siga los pasos para cargar un esquema que tengas [hasta 2 MB](#smaller-schema) o [más de 2 MB pero menos de 8 MB](#larger-schema).

<a name="smaller-schema"></a>

### <a name="add-schemas-up-to-2-mb"></a>Incorporación de esquemas de hasta 2 MB

1. En **Agregar esquema**, escriba un nombre para el esquema. 
   Mantenga seleccionado **Archivo pequeño**. Junto a la casilla **Esquema**, elija el icono de carpeta. Busque y seleccione el esquema que está cargando, por ejemplo:

   ![Carga de esquema más pequeño](./media/logic-apps-enterprise-integration-schemas/upload-smaller-schema-file.png)

1. Cuando esté listo, elija **Aceptar**.

   Una vez que el esquema termine de cargarse, aparecerá en la lista de **esquemas**.

<a name="larger-schema"></a>

### <a name="add-schemas-more-than-2-mb"></a>Incorporación de esquemas de más de 2 MB

Para agregar esquemas de mayor tamaño, puede cargar el esquema en un contenedor de blobs de Azure de la cuenta de almacenamiento de Azure. Los pasos que debe seguir para agregar esquemas varían en función de si el contenedor de blobs tiene acceso de lectura público. En primer lugar, revise si el contenedor de blobs tiene o no acceso de lectura público con estos pasos: [Establecimiento del nivel de acceso público para un contenedor de blobs](../vs-azure-tools-storage-explorer-blobs.md#set-the-public-access-level-for-a-blob-container)

#### <a name="check-container-access-level"></a>Comprobación del nivel de acceso de un contenedor

1. Abra el Explorador de Azure Storage. En la ventana del Explorador, expanda la suscripción de Azure si todavía no está expandida.

1. Expanda **Cuentas de almacenamiento** > {*su-cuenta-de-almacenamiento*} > **Contenedores de blobs**. Seleccione el contenedor de blobs.

1. En el menú contextual del contenedor de blobs, seleccione **Establecer nivel de acceso público**.

   * Si el contenedor de blobs al menos tiene acceso público, elija **Cancelar** y siga estos pasos más adelante en esta página: [Carga en contenedores con acceso público](#public-access)

     ![Acceso público](media/logic-apps-enterprise-integration-schemas/azure-blob-container-public-access.png)

   * Si el contenedor de blobs no tiene acceso público, elija **Cancelar** y siga estos pasos más adelante en esta página: [Carga en contenedores sin acceso público](#public-access)

     ![Sin acceso público](media/logic-apps-enterprise-integration-schemas/azure-blob-container-no-public-access.png)

<a name="public-access"></a>

#### <a name="upload-to-containers-with-public-access"></a>Carga en contenedores con acceso público

1. Cargue el esquema en la cuenta de almacenamiento. 
   En la ventana de la derecha, elija **Cargar**.

1. Cuando termine de cargar, seleccione el esquema cargado. En la barra de herramientas, elija **Copiar dirección URL** para copiar la dirección URL del esquema.

1. Vuelva a Azure Portal donde está abierto el panel **Agregar esquema**. 
   Escriba un nombre para el esquema. 
   Elija **Large file (larger than 2 MB)** (Archivo de gran tamaño [más de 2 MB]). 

   Ahora aparece la casilla **URI de contenido** en lugar de la casilla **Esquema**.

1. En la casilla **URI de contenido**, pegue la dirección URL del esquema. 
   Termine de agregar el esquema.

Una vez que el esquema termine de cargarse, aparecerá en la lista de **esquemas**. En la página **Información general** de la cuenta de integración, en **Componentes**, el icono **Esquemas** ahora muestra la cantidad de esquemas cargados.

<a name="no-public-access"></a>

#### <a name="upload-to-containers-without-public-access"></a>Carga en contenedores sin acceso público

1. Cargue el esquema en la cuenta de almacenamiento. 
   En la ventana de la derecha, elija **Cargar**.

1. Cuando termine de cargar, genere una firma de acceso compartido (SAS) para el esquema. 
   En el menú contextual del esquema, seleccione **Obtener firma de acceso compartido**.

1. En el panel **Firma de acceso compartido**, seleccione **Generate container-level shared access signature URI** > **Create** (Generar URI de firma de acceso compartido de nivel de contenedor > Crear). 
   Una vez que se genere la dirección URL de SAS, junto a la casilla **Dirección URL**, elija **Copiar**.

1. Vuelva a Azure Portal donde está abierto el panel **Agregar esquema**. Elija **Archivo grande**.

   Ahora aparece la casilla **URI de contenido** en lugar de la casilla **Esquema**.

1. En el cuadro **URI de contenido**, pegue el URI de SAS que generó previamente. Termine de agregar el esquema.

Una vez que el esquema termine de cargarse, aparecerá en la lista de **esquemas**. En la página **Información general** de la cuenta de integración, en **Componentes**, el icono **Esquemas** ahora muestra la cantidad de esquemas cargados.

## <a name="edit-schemas"></a>Editar esquemas

Para actualizar un esquema existente, debe cargar un nuevo archivo de esquema que tiene todos los cambios que desea. Sin embargo, primero puede descargar el esquema existente para editarla.

1. En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, busque y abra la cuenta de integración, si todavía no está abierto.

1. En el menú principal de Azure, seleccione **Todos los servicios**. 
   En el cuadro de búsqueda, escriba "cuenta de integración". 
   Seleccione **Cuentas de integración**.

1. Seleccione la cuenta de integración en la que quiere cargar el esquema.

1. En la página **Información general** de la cuenta de integración, en **Componentes**, seleccione el icono **Esquemas**.

1. Una vez que se abre la página **Esquemas**, seleccione el esquema. 
   Para descargar y editar primero el esquema, elija **Descargar** y guarde el esquema.

1. Cuando esté listo para cargar el esquema actualizado, en la página **Esquemas**, seleccione el esquema que quiere actualizar y elija **Actualizar**.

1. Busque y seleccione el esquema actualizado que quiere cargar. 
   Una vez que el esquema termine de cargarse, aparecerá en la lista de **esquemas**.

## <a name="delete-schemas"></a>Eliminar esquemas

1. En <a href="https://portal.azure.com" target="_blank">Azure Portal</a>, busque y abra la cuenta de integración, si todavía no está abierto.

1. En el menú principal de Azure, seleccione **Todos los servicios**. 
   En el cuadro de búsqueda, escriba "cuenta de integración". 
   Seleccione **Cuentas de integración**.

1. Seleccione la cuenta de integración de la que quiere eliminar el esquema.

1. En la página **Información general** de la cuenta de integración, en **Componentes**, seleccione el icono **Esquemas**.

1. Una vez que se abre la página **Esquemas**, seleccione el esquema y elija **Eliminar**.

1. Para confirmar que desea eliminar el esquema, elija **Sí**.

## <a name="limitations"></a>Limitaciones

Normalmente, cuando se usa una cuenta de integración con el flujo de trabajo y se quiere validar código XML, se agrega o se carga el esquema en esa cuenta. Si hace referencia a un esquema que no está en la cuenta de integración, o lo importa, es posible que reciba el siguiente error al usar el elemento `xsd:redefine`:

`An error occurred while processing the XML schemas: ''SchemaLocation' must successfully resolve if <redefine> contains any child other than <annotation>.'.`

Para resolver este error, debe usar el elemento `xsd:import` o `xsd:include`, en lugar de `xsd:redefine`, o bien usar un URI.

## <a name="next-steps"></a>Pasos siguientes

* [Más información acerca de Enterprise Integration Pack](logic-apps-enterprise-integration-overview.md)
* [Más información sobre las asignaciones](../logic-apps/logic-apps-enterprise-integration-maps.md)
* [Más información sobre las transformaciones](../logic-apps/logic-apps-enterprise-integration-transform.md)
