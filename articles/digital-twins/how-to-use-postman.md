---
title: Realización de solicitudes con Postman
titleSuffix: Azure Digital Twins
description: Aprenda a configurar y usar Postman para probar las API de Azure Digital Twins.
ms.author: baanders
author: baanders
ms.service: digital-twins
services: digital-twins
ms.topic: how-to
ms.date: 11/10/2020
ms.openlocfilehash: a528e224511fda363afb80a7749a018e07b5fa26
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588010"
---
# <a name="how-to-use-postman-to-send-requests-to-the-azure-digital-twins-apis"></a>Uso de Postman para enviar solicitudes a las API de Azure Digital Twins

[Postman](https://www.getpostman.com/) es una herramienta de pruebas REST que proporciona funcionalidades clave de solicitud HTTP en una interfaz gráfica de usuario basada en escritorio y complementos. Puede usarla para elaborar solicitudes HTTP y enviarlas a las [API REST de Azure Digital Twins](how-to-use-apis-sdks.md).

En este artículo se describe cómo configurar el [cliente REST de Postman](https://www.getpostman.com/) para interactuar con las API de Azure Digital Twins. Para ello, es preciso seguir estos pasos:

1. Use la CLI de Azure para [**obtener un token de portador**](#get-bearer-token) que usará para realizar solicitudes de API en Postman.
1. Configure una [**colección de Postman**](#about-postman-collections) y el cliente de REST de Postman para que use el token de portador para la autenticación. Al configurar la colección, puede elegir cualquiera de estas opciones:
    1. [**Importar**](#import-collection-of-azure-digital-twins-apis) una colección precompilada de solicitudes de API de Azure Digital Twins.
    1. [**Crear**](#create-your-own-collection) su propia colección desde cero.
1. [**Agregar solicitudes**](#add-an-individual-request) a la colección configurada y enviarlas a las API de Azure Digital Twins.

Azure Digital Twins tiene dos conjuntos de API con los que puede trabajar: el **plano de datos** y el **plano de control**. Para obtener más información sobre la diferencia entre estos conjuntos de API, consulte [*Cómo usar las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md). Este artículo contiene información de ambos conjuntos de API.

## <a name="prerequisites"></a>Prerrequisitos

Para continuar con el uso de Postman para acceder a las API de Azure Digital Twins, es preciso configurar una instancia de Azure Digital Twins y descargar Postman. El resto de esta sección le guía a través de estos pasos.

### <a name="set-up-azure-digital-twins-instance"></a>Configuración de la instancia de Azure Digital Twins

[!INCLUDE [digital-twins-prereq-instance.md](../../includes/digital-twins-prereq-instance.md)]

### <a name="download-postman"></a>Descarga de Postman

A continuación, descargue la versión de escritorio del cliente de Postman. Vaya a [*www.getpostman.com/apps*](https://www.getpostman.com/apps) y siga las indicaciones para descargar la aplicación.

## <a name="get-bearer-token"></a>Obtención de un token de portador

Ahora que ha configurado Postman y su instancia de Azure Digital Twins, deberá obtener un token de portador que las solicitudes de Postman puedan usar para la autorización en las API de Azure Digital Twins.

Este token se puede obtener de varias formas. En este artículo se usa la [CLI de Azure](/cli/azure/install-azure-cli) para iniciar sesión en su cuenta de Azure y obtener así un token.

Si tiene una CLI de Azure [instalada localmente](/cli/azure/install-azure-cli), puede iniciar un símbolo del sistema en la máquina para ejecutar los siguientes comandos.
De lo contrario, puede abrir una ventana de [Azure Cloud Shell](https://shell.azure.com) en el explorador y ejecutar los comandos en ella.

1. En primer lugar, asegúrese de que ha iniciado sesión en Azure con las credenciales apropiadas. Para ello, ejecute este comando:

    ```azurecli-interactive
    az login
    ```

2. Luego, use el comando [az account get-access-token](/cli/azure/account#az_account_get_access_token) para obtener un token de portador con acceso al servicio Azure Digital Twins. En este comando, pasará el id. de recurso del punto de conexión del servicio Azure Digital Twins, con el fin de obtener un token de acceso que pueda acceder a los recursos de Azure Digital Twins. 

    El contexto necesario para el token depende del conjunto de API que esté usando, por lo que debe usar las pestañas siguientes para seleccionar entre las API del [plano de datos](how-to-use-apis-sdks.md#overview-data-plane-apis) y del [plano de control](how-to-use-apis-sdks.md#overview-control-plane-apis).

    # <a name="data-plane"></a>[Plano de datos](#tab/data-plane)
    
    Para obtener un token y usarlo con las API del **plano de datos**, use el siguiente valor estático para el contexto del token: `0b07f429-9f4b-4714-9392-cc5e8e80c8b0`. Este es el id. de recurso del punto de conexión del servicio de Azure Digital Twins.
    
    ```azurecli-interactive
    az account get-access-token --resource 0b07f429-9f4b-4714-9392-cc5e8e80c8b0
    ```
    
    # <a name="control-plane"></a>[Plano de control](#tab/control-plane)
    
    Para obtener un token y usarlo con las API del **plano de control**, use el siguiente valor para el contexto del token: `https://management.azure.com/`.
    
    ```azurecli-interactive
    az account get-access-token --resource https://management.azure.com/
    ```
    ---

    >[!NOTE]
    > Si necesita acceder a la instancia de Azure Digital Twins mediante una entidad de servicio o una cuenta de usuario que pertenezca a un inquilino distinto de Azure Active Directory de la instancia, necesitará solicitar un **token** del inquilino "principal" de la instancia de Azure Digital Twins. Para obtener más información sobre este proceso, consulte [*Procedimiento: Escritura de código de autenticación de aplicación.* ](how-to-authenticate-client.md#authenticate-across-tenants)

3. Copie el valor de `accessToken` en el resultado y guárdelo para usarlo en la siguiente sección. Este es el **valor del token** que proporcionará a Postman para autenticar las solicitudes.

    :::image type="content" source="media/how-to-use-postman/console-access-token.png" alt-text="Captura de pantalla de la consola que muestra el resultado del comando az account get-access-token. El campo accessToken y su valor de muestra están resaltados.":::

>[!TIP]
>Este token es válido durante un mínimo de 5 minutos y un máximo de 60. Si se agota el tiempo asignado al token actual, puede repetir los pasos de esta sección para obtener uno nuevo.

A continuación, configurará Postman para usar este token y realizar solicitudes de API a Azure Digital Twins.

## <a name="about-postman-collections"></a>Acerca de las colecciones de Postman

En Postman, las solicitudes se guardan en **colecciones** (grupos de solicitudes). Al crear una colección para agrupar las solicitudes, puede aplicar una configuración común a muchas solicitudes a la vez. Esto puede simplificar considerablemente la autorización si planea crear más de una solicitud con las API de Azure Digital Twins, ya que solo tiene que configurar estos detalles una vez para toda la colección.

Al trabajar con Azure Digital Twins, puede empezar importando una [colección precompilada de todas las solicitudes de Azure Digital Twins](#import-collection-of-azure-digital-twins-apis). Puede que quiera hacer esto si está explorando las API y quiere configurar rápidamente un proyecto con ejemplos de solicitudes.

Como alternativa, también puede empezar desde cero [creando su propia colección vacía](#create-your-own-collection) y rellenándola con solicitudes individuales que solo llamen a las API que necesite. 

Las siguientes secciones describen ambos procesos. El resto del artículo se lleva a cabo en la aplicación Postman local, por lo que debe abrirla en el equipo.

## <a name="import-collection-of-azure-digital-twins-apis"></a>Importación de la colección de API de Azure Digital Twins

Una forma rápida de comenzar a usar Azure Digital Twins en Postman es importar una colección de solicitudes predefinidas para las API de Azure Digital Twins.

### <a name="download-the-collection-file"></a>Descarga del archivo de recopilación

El primer paso para importar el conjunto de API es descargar una colección. Elija la pestaña siguiente para elegir el plano de datos o el plano de control y así ver las opciones de la colección precompilada.

# <a name="data-plane"></a>[Plano de datos](#tab/data-plane)

Actualmente hay dos colecciones del plano de datos de Azure Digital Twins disponibles para elegir:
* [**Colección de Postman de Azure Digital Twins**](https://github.com/microsoft/azure-digital-twins-postman-samples): esta colección proporciona una experiencia sencilla de introducción a Azure Digital Twins en Postman. Las solicitudes incluyen datos de ejemplo, por lo que puede ejecutarlas con las modificaciones mínimas necesarias. Elija esta colección si quiere un conjunto digerible de solicitudes de API clave que contengan información de ejemplo.
    - Para buscar la colección, vaya al vínculo del repositorio y abra el archivo denominado *postman_collection.jsen*.
* **[Swagger en el plano de datos de Azure Digital Twins](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/data-plane/Microsoft.DigitalTwins)** : este repositorio contiene el archivo de Swagger completo para el conjunto de API de Azure Digital Twins, que se puede descargar e importar a Postman como una colección. Esto proporcionará un conjunto completo de todas las solicitudes de API, pero con cuerpos de datos vacíos en lugar de datos de ejemplo. Elija esta colección si quiere tener acceso a cada llamada API y rellenar todos los datos por sí mismo.
    - Para buscar la colección, vaya al vínculo del repositorio y elija la carpeta de la versión más reciente de la especificación. Desde aquí, abra el archivo llamado *digitaltwins.json*.

# <a name="control-plane"></a>[Plano de control](#tab/control-plane)

La colección disponible actualmente para el plano de control es la instancia de [**Swagger del plano de control de Azure Digital Twins**](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/digitaltwins/data-plane/Microsoft.DigitalTwins). Este repositorio contiene el archivo de Swagger completo para el conjunto de API de Azure Digital Twins, que se puede descargar e importar a Postman como una colección. Esto le proporcionará un conjunto completo de todas las solicitudes de API.

Para buscar la colección, vaya al vínculo del repositorio y elija la carpeta de la versión más reciente de la especificación. Desde aquí, abra el archivo llamado *digitaltwins.json*.

---

Aquí se muestra cómo descargar la colección elegida en la máquina para poder importarla en Postman.
1. Use los vínculos anteriores para abrir en el explorador el archivo de colección en GitHub.
1. Seleccione el botón **Sin formato** para abrir el texto sin formato del archivo.
    :::image type="content" source="media/how-to-use-postman/swagger-raw.png" alt-text="Captura de pantalla del archivo digitaltwins.json del plano de datos en GitHub. El botón Sin formato está resaltado." lightbox="media/how-to-use-postman/swagger-raw.png":::
1. Copie el texto de la ventana y péguelo en un nuevo archivo en la máquina.
1. Guarde el archivo con una extensión *.json* (el nombre de archivo puede ser el que quiera, siempre que pueda recordarlo para encontrar el archivo más adelante).

### <a name="import-the-collection"></a>Importación de la colección

A continuación, importe la colección en Postman.

1. En la ventana principal de Postman, seleccione el botón **Importar**.
    :::image type="content" source="media/how-to-use-postman/postman-import-collection.png" alt-text="Captura de pantalla de una ventana de Postman recién abierta. El botón &quot;Importar&quot; está resaltado." lightbox="media/how-to-use-postman/postman-import-collection.png":::

1. En la ventana importar que aparece a continuación, seleccione **Cargar archivos** y vaya al archivo de colección que creó anteriormente en la máquina. Seleccione Abrir.
1. Seleccione el botón **Importar** para confirmar la acción.

    :::image type="content" source="media/how-to-use-postman/postman-import-collection-2.png" alt-text="Captura de pantalla de la ventana &quot;Importar&quot; de Postman, que muestra el archivo que se va a importar como una colección y el botón Importar.":::

La colección recién importada se puede ver desde la vista principal de Postman, en la pestaña Colecciones.

:::image type="content" source="media/how-to-use-postman/postman-post-collection-imported.png" alt-text="Captura de pantalla de la ventana principal de Postman. La colección recién importada está resaltada en la pestaña &quot;Colecciones&quot;." lightbox="media/how-to-use-postman/postman-post-collection-imported.png":::

A continuación, vaya a la sección siguiente para agregar un token de portador a la colección para la autorización y conectarlo a la instancia de Azure Digital Twins.

### <a name="configure-authorization"></a>Configuración de la autorización

A continuación, edite la colección que ha creado para configurar algunos detalles de acceso. Resalte la colección que ha creado y seleccione el icono **Ver más acciones** para desplegar un menú. Seleccione **Editar**.

:::image type="content" source="media/how-to-use-postman/postman-edit-collection.png" alt-text="Captura de pantalla Postman. El icono &quot;Ver más acciones&quot; de la colección importada está resaltado, y la opción &quot;Editar&quot; está resaltada en el botón de opciones." lightbox="media/how-to-use-postman/postman-edit-collection.png":::

Siga estos pasos para agregar un token de portador a la colección para la autorización. Ahí es donde usará el **valor del token** que apuntó en la sección [Obtención de un token de portador](#get-bearer-token) para usarlo en todas las solicitudes de API de la colección.

1. En el cuadro de diálogo Editar de la colección, asegúrese de que se encuentra en la pestaña **Autorización**. 

    :::image type="content" source="media/how-to-use-postman/postman-authorization-imported.png" alt-text="Captura de pantalla del cuadro de diálogo de edición de la colección importada en Postman, que muestra la pestaña &quot;Autorización&quot;." lightbox="media/how-to-use-postman/postman-authorization-imported.png":::

1. Establezca el tipo en **OAuth 2.0** y pegue el token de acceso en el cuadro Token de acceso y seleccione **Guardar**.

    :::image type="content" source="media/how-to-use-postman/postman-paste-token-imported.png" alt-text=" Captura de pantalla del cuadro de diálogo de edición de Postman para la colección importada, en la pestaña &quot;Autorización&quot;. El tipo es &quot;OAuth 2.0&quot; y el cuadro del token de acceso está resaltado." lightbox="media/how-to-use-postman/postman-paste-token-imported.png":::

### <a name="additional-configuration"></a>Configuración adicional

# <a name="data-plane"></a>[Plano de datos](#tab/data-plane)

Si va a realizar una colección de [plano de datos](how-to-use-apis-sdks.md#overview-data-plane-apis), puede facilitar la conexión de la colección a los recursos de Azure Digital Twins si establece algunas **variables** que se proporcionan con las colecciones. Cuando muchas solicitudes de una colección necesitan el mismo valor (como el nombre de host de la instancia de Azure Digital Twins), puede almacenar el valor en una variable que se aplique a todas las solicitudes de la colección. Las dos colecciones descargables de Azure Digital Twins ofrecen variables creadas previamente que puede establecer en el nivel de colección.

1. Asimismo, en el cuadro de diálogo Editar de la colección, desplácese a la pestaña **Variables**.

1. Use el **nombre de host** de la instancia de la sección de [*Requisitos previos*](#prerequisites) para establecer el campo CURRENT VALUE de la variable correspondiente. Seleccione **Guardar**.

    :::image type="content" source="media/how-to-use-postman/postman-variables-imported.png" alt-text="Captura de pantalla del cuadro de diálogo de edición de la colección importada en Postman, que muestra la pestaña &quot;Variables&quot;. El campo &quot;CURRENT VALUE&quot; está resaltado." lightbox="media/how-to-use-postman/postman-variables-imported.png":::

1. Si la colección tiene variables adicionales, rellene y guarde también esos valores.

Cuando haya terminado con los pasos anteriores, habrá terminado de configurar la colección. Si lo desea, puede cerrar la pestaña de edición de la colección.

# <a name="control-plane"></a>[Plano de control](#tab/control-plane)

Si va a realizar una colección de [planos de control](how-to-use-apis-sdks.md#overview-control-plane-apis), ya ha hecho todo lo que necesita para configurar la colección. Si quiere, puede cerrar la pestaña de edición de la colección y pasar a la sección siguiente.

--- 

### <a name="explore-requests"></a>Exploración de solicitudes

A continuación, explore las solicitudes de la colección de API de Azure Digital Twins. Puede expandir la colección para ver las solicitudes creadas previamente (ordenadas por categoría de operación). 

Las distintas solicitudes requieren información diferente sobre su instancia y sus datos. Para ver toda la información necesaria para elaborar una solicitud determinada, consulte los detalles de la solicitud en la [documentación de referencia de la API de REST de Azure Digital Twins](/rest/api/azure-digitaltwins/).

Puede editar los detalles de una solicitud en la colección de Postman mediante estos pasos:

1. Selecciónela en la lista para extraer sus detalles editables. 

1. Rellene los valores de las variables que aparecen en la pestaña **Parámetros** en **Variables de la ruta de acceso**.

    :::image type="content" source="media/how-to-use-postman/postman-request-details-imported.png" alt-text="Captura de pantalla de Postman. La colección se amplía para mostrar una solicitud. La sección &quot;Variables de ruta&quot; está resaltada en los detalles de la solicitud." lightbox="media/how-to-use-postman/postman-request-details-imported.png":::

1. Proporcione los detalles de los **encabezados** o del **cuerpo** necesarios en las pestañas correspondientes.

Una vez que se proporcionen todos los detalles necesarios, puede ejecutar la solicitud con el botón **Enviar**.

También puede agregar sus propias solicitudes a la colección mediante el proceso que se describe en la sección [*Adición de una solicitud individual*](#add-an-individual-request) que se indica a continuación.

## <a name="create-your-own-collection"></a>Creación de sus propias colecciones

En lugar de importar la colección existente de todas las API de Azure Digital Twins, también puede crear su propia colección desde el principio. A continuación, puede rellenarla con solicitudes individuales mediante la [documentación de referencia de la API de REST de Azure Digital Twins](/rest/api/azure-digitaltwins/).

### <a name="create-a-postman-collection"></a>Creación de la colección de Postman

1. Para crear una colección, seleccione el botón **Nueva** en la ventana principal de Postman.

    :::image type="content" source="media/how-to-use-postman/postman-new.png" alt-text="Captura de pantalla de la ventana principal de Postman. El botón &quot;Siguiente&quot; está resaltado." lightbox="media/how-to-use-postman/postman-new.png":::

    Elija un tipo de **colección**.

    :::image type="content" source="media/how-to-use-postman/postman-new-collection-2.png" alt-text="Captura de pantalla del cuadro de diálogo &quot;Crear nueva&quot; en Postman. La opción &quot;Colección&quot; está resaltada.":::

1. Se abrirá una pestaña para rellenar los detalles de la nueva colección. Seleccione el icono de edición situado junto al nombre predeterminado de la colección (**Nueva colección**) para reemplazarlo por un nombre de su elección. 

    :::image type="content" source="media/how-to-use-postman/postman-new-collection-3.png" alt-text=" Captura de pantalla del cuadro de diálogo de edición de la nueva colección en Postman. El icono Editar junto al nombre &quot;Nueva colección&quot; está resaltado." lightbox="media/how-to-use-postman/postman-new-collection-3.png":::

Luego, vaya a la sección siguiente para agregar un token de portador a la colección para la autorización.

### <a name="configure-authorization"></a>Configuración de la autorización

Siga estos pasos para agregar un token de portador a la colección para la autorización. Ahí es donde usará el **valor del token** que apuntó en la sección [Obtención de un token de portador](#get-bearer-token) para usarlo en todas las solicitudes de API de la colección.

1. Todavía en el cuadro de diálogo Editar de la nueva colección, vaya a la pestaña **Autorización**.

    :::image type="content" source="media/how-to-use-postman/postman-authorization-custom.png" alt-text="Captura de pantalla del cuadro de diálogo de edición de la nueva colección en Postman, que muestra la pestaña &quot;Autorización&quot;." lightbox="media/how-to-use-postman/postman-authorization-custom.png":::

1. Establezca el tipo en **OAuth 2.0** y pegue el token de acceso en el cuadro Token de acceso y seleccione **Guardar**.

    :::image type="content" source="media/how-to-use-postman/postman-paste-token-custom.png" alt-text="Captura de pantalla del cuadro de diálogo de edición de Postman para la nueva colección, en la pestaña &quot;Autorización&quot;. El tipo es &quot;OAuth 2.0&quot; y el cuadro del token de acceso está resaltado." lightbox="media/how-to-use-postman/postman-paste-token-custom.png":::

Cuando haya terminado con los pasos anteriores, habrá terminado de configurar la colección. Si lo desea, puede cerrar la pestaña de edición de la colección nueva.

La nueva colección se puede ver desde la vista principal de Postman, en la pestaña Colecciones.

:::image type="content" source="media/how-to-use-postman/postman-post-collection-custom.png" alt-text="Captura de pantalla de la ventana principal de Postman. La colección personalizada recién creada está resaltada en la pestaña &quot;Colecciones&quot;colección." lightbox="media/how-to-use-postman/postman-post-collection-custom.png":::

## <a name="add-an-individual-request"></a>Adición de una solicitud individual

Ahora que la colección está configurada, puede agregar sus propias solicitudes a las API de Azure Digital Twins.

1. Para crear una solicitud, pulse el botón **Nueva**.

    :::image type="content" source="media/how-to-use-postman/postman-new.png" alt-text="Captura de pantalla de la ventana principal de Postman. El botón &quot;Siguiente&quot; está resaltado." lightbox="media/how-to-use-postman/postman-new.png":::

    Elija un tipo de **Solicitud**.

    :::image type="content" source="media/how-to-use-postman/postman-new-request-2.png" alt-text="Captura de pantalla del cuadro de diálogo &quot;Crear nueva&quot; en Postman. La opción &quot;Solicitud&quot; está resaltada.":::

1. Esta acción abre la ventana GUARDAR SOLICITUD, donde puede especificar el nombre de la solicitud, darle una descripción opcional y elegir la colección de la que forma parte. Rellene los detalles y guarde la solicitud en la colección que creó anteriormente.

    :::row:::
        :::column:::
            :::image type="content" source="media/how-to-use-postman/postman-save-request.png" alt-text="Captura de pantalla de la ventana &quot;Guardar solicitud&quot; en Postman que muestra los campos descritos. El botón &quot;Save to Azure Digital Twins collection&quot; (Guardar en colección de Azure Digital Twins) está resaltado.":::
        :::column-end:::
        :::column:::
        :::column-end:::
    :::row-end:::

Ahora puede ver la solicitud en la colección y seleccionarla para extraer sus detalles editables.

:::image type="content" source="media/how-to-use-postman/postman-request-details-custom.png" alt-text="Captura de pantalla de Postman. La colección Azure Digital Twins se amplía para mostrar los detalles de la solicitud." lightbox="media/how-to-use-postman/postman-request-details-custom.png":::

### <a name="set-request-details"></a>Establecimiento de los detalles de la solicitud

Para realizar una solicitud de Postman en una de las API de Azure Digital Twins, necesitará la dirección URL de la API e información sobre los detalles que requiere. Esta información se puede encontrar en la [documentación de referencia de la API REST de Azure Digital Twins](/rest/api/azure-digitaltwins/).

Para continuar con una consulta de ejemplo, en este artículo se usará Query API (y su [documentación de referencia](/rest/api/digital-twins/dataplane/query/querytwins)) para consultar todos los gemelos digitales de una instancia.

1. Obtenga la dirección URL y el tipo de la solicitud en la documentación de referencia. En el caso de Query API, actualmente es *POST `https://digitaltwins-hostname/query?api-version=2020-10-31`* .
1. En Postman, establezca el tipo de la solicitud y escriba la dirección URL de la solicitud, rellenando los marcadores de posición en la dirección URL según sea necesario. Aquí es donde usará el **nombre de host** de la instancia de la sección [*Requisitos previos*](#prerequisites).
    
   :::image type="content" source="media/how-to-use-postman/postman-request-url.png" alt-text="Captura de pantalla de los detalles de la nueva solicitud en Postman. La dirección URL de la documentación de referencia se ha rellenado en el cuadro de la dirección URL de la solicitud." lightbox="media/how-to-use-postman/postman-request-url.png":::
    
1. Compruebe que los parámetros que se muestran para la solicitud en la pestaña **Params** (Parámetros) coinciden con los descritos en la documentación de referencia. En el caso de esta solicitud en Postman, el parámetro `api-version` se rellenó automáticamente al escribir la dirección URL de la solicitud en el paso anterior. Para Query API, este es el único parámetro necesario, por lo que este paso ha terminado.
1. En la pestaña **Authorization** (Autorización), en Type (Tipo) seleccione **Inherit auth from parent** (Heredar autorización de primario). Esto indica que esta solicitud usará la autorización que configuró anteriormente para toda la colección.
1. Compruebe que los encabezados que se muestran para la solicitud en la pestaña **Headers** (Encabezados) coinciden con los descritos en la documentación de referencia. Para esta solicitud, se han rellenado automáticamente varios encabezados. Para Query API, no se requiere ninguna de las opciones de encabezado, por lo que este paso ha finalizado.
1. Compruebe que el cuerpo que se muestra para la solicitud en la pestaña **Body** (Cuerpo) coincide con las necesidades descritas en la documentación de referencia. Para Query API, se requiere un cuerpo JSON que proporcione el texto de la consulta. Este es un cuerpo de ejemplo para esta solicitud que consulta todos los gemelos digitales de la instancia:

   :::image type="content" source="media/how-to-use-postman/postman-request-body.png" alt-text="Captura de pantalla de los detalles de la nueva solicitud en Postman, en la pestaña Cuerpo. Contiene un cuerpo JSON sin formato con una consulta de &quot;SELECT * FROM DIGITALTWINS&quot;." lightbox="media/how-to-use-postman/postman-request-body.png":::

   Para más información sobre cómo crear consultas de Azure Digital Twins, consulte [*Procedimiento: Consulta del grafo de gemelos*](how-to-query-graph.md).

1. Consulte en la documentación de referencia otros campos que pueden ser necesarios para su tipo de solicitud. En el caso de Query API, ahora se han cumplido todos los requisitos en la solicitud de Postman, por lo que este paso ha finalizado.
1. Use el botón **Send** (Enviar) para enviar la solicitud completada.
   :::image type="content" source="media/how-to-use-postman/postman-request-send.png" alt-text="Captura de pantalla de Postman que muestra los detalles de la nueva solicitud. El botón Enviar está resaltado." lightbox="media/how-to-use-postman/postman-request-send.png":::

Después de enviar la solicitud, los detalles de la respuesta aparecerán en la ventana de Postman debajo de la solicitud. Puede ver el código de estado de la respuesta y cualquier texto del cuerpo.

:::image type="content" source="media/how-to-use-postman/postman-request-response.png" alt-text="Captura de pantalla de la solicitud enviada en Postman. Debajo de los detalles de la solicitud, se muestra la respuesta. El estado es &quot;200 OK&quot; y el cuerpo muestra los resultados de la consulta." lightbox="media/how-to-use-postman/postman-request-response.png":::

También puede comparar la respuesta con los datos de respuesta esperados que se proporcionan en la documentación de referencia para comprobar el resultado u obtener más información sobre los errores que surjan.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre las API de Digital Twins, lea [*Procedimiento: Uso de las API y los SDK de Azure Digital Twins*](how-to-use-apis-sdks.md) o vea la [documentación de referencia de las API REST](/rest/api/azure-digitaltwins/).