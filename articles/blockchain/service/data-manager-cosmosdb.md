---
title: 'Uso de Blockchain Data Manager para actualizar Azure Cosmos DB: Azure Blockchain Service'
description: Uso de Blockchain Data Manager para Azure Blockchain Service para enviar datos de la cadena de bloques a Azure Cosmos DB
ms.date: 03/08/2020
ms.topic: tutorial
ms.reviewer: chroyal
ms.openlocfilehash: b054688fb524d82e758f2ac1a9ff643293bc3b29
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "110465755"
---
# <a name="tutorial-use-blockchain-data-manager-to-send-data-to-azure-cosmos-db"></a>Tutorial: Uso de Blockchain Data Manager para enviar datos a Azure Cosmos DB

En este tutorial, usará Blockchain Data Manager  para que Azure Blockchain Service registre datos de transacciones de cadenas de bloques en Azure Cosmos DB.

[!INCLUDE [Retirement note](./includes/retirement.md)]

Blockchain Data Manager captura, transforma y entrega datos de libro de contabilidad de cadena de bloques a los temas de Azure Event Grid. En Azure Event Grid, se usa un conector de aplicación lógica para crear documentos en una base de datos de Azure Cosmos DB. Una vez finalizado el tutorial, puede explorar los datos de transacciones de cadena de bloques en Azure Cosmos DB Data Explorer.

[![La captura de pantalla muestra los detalles de la transacción de cadena de bloques.](./media/data-manager-cosmosdb/raw-msg.png)](./media/data-manager-cosmosdb/raw-msg.png#lightbox)

En este tutorial, hizo lo siguiente:

> [!div class="checklist"]
> * Crear una instancia de Blockchain Data Manager
> * Agregar una aplicación de cadena de bloques para descodificar propiedades y eventos de transacciones
> * Crear una cuenta de Azure Cosmos DB y una base de datos para almacenar datos de transacciones
> * Crear una aplicación lógica de Azure para conectar un tema de Azure Event Grid con Azure Cosmos DB
> * Enviar una transacción a un libro de contabilidad de cadena de bloques
> * Ver los datos de transacción descodificados en Azure Cosmos DB

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prerrequisitos

* Realizar el tutorial [Quickstart: Creación de un miembro de cadena de bloques mediante Azure Portal](create-member.md) o [Inicio rápido: Creación de un miembro de cadena de bloques de Azure Blockchain Service mediante la CLI de Azure](create-member-cli.md)
* Realizar el tutorial [Quickstart: Uso de Visual Studio Code para conectarse a una red del consorcio de Azure Blockchain Service](connect-vscode.md). El inicio rápido le guía por la instalación de [Azure Blockchain Development Kit para Ethereum](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) y la configuración del entorno de desarrollo de la cadena de bloques.
* Complete el [Tutorial: Uso de Visual Studio Code para crear, compilar e implementar contratos inteligentes](send-transaction.md). El tutorial le guía a lo largo de la creación de un contrato inteligente de ejemplo.
* Crear un [tema de Event Grid](../../event-grid/custom-event-quickstart-portal.md#create-a-custom-topic)
* Más información sobre [Controladores de eventos en Azure Event Grid](../../event-grid/event-handlers.md)

## <a name="create-instance"></a>Crear instancia

Una instancia de Blockchain Data Manager conecta y supervisa un nodo de transacción de Azure Blockchain Service. Una instancia captura todos los datos de transacciones y bloques sin procesar del nodo de transacción. Una conexión saliente envía datos de la cadena de bloques a Azure Event Grid. Configura una conexión saliente única cuando crea la instancia.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
1. Vaya al miembro de Azure Blockchain Service que creó en el requisito previo [Inicio rápido: Creación de un miembro de cadena de bloques mediante Azure Portal](create-member.md). Seleccione **Blockchain Data Manager**.
1. Seleccione **Agregar**.

    ![Agregar Blockchain Data Manager](./media/data-manager-cosmosdb/add-instance.png)

    Escriba la siguiente información:

    Configuración | Ejemplo | Descripción
    --------|---------|------------
    Nombre | mywatcher | Escriba un nombre único para una instancia conectada de Blockchain Data Manager.
    Nodo de transacción | myblockchainmember | Elija el nodo de transacción predeterminado del miembro de Azure Blockchain Service que creó en el requisito previo.
    Nombre de conexión | cosmosdb | Escriba un nombre único de la conexión saliente donde se envían los datos de la transacción de la cadena de bloques.
    Punto de conexión de Event Grid | myTopic | Elija un tema de Event Grid que haya creado en el requisito previo. Nota: La instancia de Blockchain Data Manager y el tema de Event Grid deben estar en la misma suscripción.

1. Seleccione **Aceptar**.

    Se tarda menos de un minuto crear una instancia de Blockchain Data Manager. Una vez implementada la instancia, se inicia automáticamente. Una instancia de Blockchain Data Manager en ejecución captura los eventos de la cadena de bloques del nodo de transacción y envía los datos a Event Grid.

## <a name="add-application"></a>Agregar aplicación

Agregue la aplicación de cadena de bloques **helloblockchain** para que Blockchain Data Manager descodifique el estado de la propiedad y el evento. Blockchain Data Manager requiere la ABI del contrato inteligente y un archivo de código de bytes para agregar la aplicación.

### <a name="get-contract-abi-and-bytecode"></a>Obtener la ABI del contrato y el código de bytes

La ABI del contrato define las interfaces del contrato inteligente. Describe cómo interactuar con el contrato inteligente. Puede usar la [extensión del kit de desarrollo para Ethereum de Azure Blockchain](https://marketplace.visualstudio.com/items?itemName=AzBlockchain.azure-blockchain) para copiar la ABI del contrato en el portapapeles.

1. En el panel del explorador de Visual Studio Code, expanda la carpeta **build/contracts** del proyecto Solidity de **helloblockchain** que creó en el requisito previo [Tutorial: Uso de Visual Studio Code para crear, compilar e implementar contratos inteligentes](send-transaction.md)
1. Haga clic con el botón derecho en el archivo JSON de metadatos del contrato. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Seleccione **Copy contract ABI** (Copiar ABI del contrato).

    ![Panel de Visual Studio Code con la selección Copy contract ABI (Copiar ABI del contrato)](./media/data-manager-cosmosdb/abi-devkit.png)

    La ABI del contrato se copia en el portapapeles.

1. Guarde la matriz **abi** como archivo JSON. Por ejemplo, *abi.json*. Lo usará en un paso posterior.

Blockchain Data Manager requiere el código de bytes implementado para el contrato inteligente. El código de bytes implementado es diferente del código de bytes del contrato inteligente. Puede usar la extensión del kit de desarrollo de Azure Blockchain para copiar el código de bytes en el portapapeles.

1. En el panel del explorador de Visual Studio Code, expanda la carpeta **build/contracts** del proyecto de Solidity.
1. Haga clic con el botón derecho en el archivo JSON de metadatos del contrato. El nombre de archivo es el nombre del contrato inteligente seguido de la extensión **.json**.
1. Seleccione **Copiar código de bytes de la transacción**.

    ![Panel de Visual Studio Code con la selección Copiar código de bytes de la transacción](./media/data-manager-cosmosdb/bytecode-devkit.png)

    El código de bytes se copia en el portapapeles.

1. Guarde el valor del **código de bytes** como archivo JSON. Por ejemplo, *bytecode.json*. Lo usará en un paso posterior.

El ejemplo siguiente muestra archivos *abi.json* y *bytecode.json* abiertos en el editor de VS Code. Sus archivos deben ser similares.

![Ejemplo de archivos abi.json y bytecode.json](./media/data-manager-cosmosdb/contract-files.png)

### <a name="create-contract-abi-and-bytecode-url"></a>Obtener la ABI del contrato y la dirección URL del código de bytes

Blockchain Data Manager requiere que una dirección URL pueda acceder a la ABI del contrato y a los archivos del código de bytes al agregar una aplicación. Puede usar una cuenta de Azure Storage para proporcionar una dirección URL de acceso privado.

#### <a name="create-storage-account"></a>Crear cuenta de almacenamiento

[!INCLUDE [storage-create-account-portal-include](../../../includes/storage-create-account-portal-include.md)]

#### <a name="upload-contract-files"></a>Cargar archivos de contrato

1. Cree un nuevo contenedor para la cuenta de almacenamiento. Seleccione **Contenedores > Contenedor**.

    ![Crear un contenedor de cuenta de almacenamiento](./media/data-manager-cosmosdb/create-container.png)

    | Configuración | Descripción |
    |---------|-------------|
    | Nombre  | Nombre del contenedor. Por ejemplo, *smartcontract* |
    | Nivel de acceso público | Elija *Privado (sin acceso anónimo)* |

1. Seleccione **Aceptar** para crear el contenedor.
1. Seleccione el contenedor y, a continuación, seleccione **Cargar**.
1. Elija los dos archivos JSON creados en la sección [Obtener la ABI del contrato y el código de bytes](#get-contract-abi-and-bytecode).

    ![Carga de blob](./media/data-manager-cosmosdb/upload-blobs.png)

    Seleccione **Cargar**.

#### <a name="generate-url"></a>Generar dirección URL

Para cada blob, genere una firma de acceso compartido.

1. Seleccione el blob JSON de ABI.
1. Seleccione **Generar SAS**
1. Establezca la expiración de la firma de acceso deseada y seleccione **Generar el token de SAS de blob y la dirección URL**.

    ![Generación de un token de SAS](./media/data-manager-cosmosdb/generate-sas.png)

1. Copie la **dirección URL de SAS de blob** y guárdela en la siguiente sección.
1. Repita los pasos para [generar una dirección URL](#generate-url) para el blob JSON del código de bytes.

### <a name="add-helloblockchain-application-to-instance"></a>Agregar la aplicación helloblockchain a una instancia

1. Seleccione la instancia de Blockchain Data Manager en la lista de instancias.
1. Seleccione **Blockchain applications** (aplicaciones de cadena de bloques).
1. Seleccione **Agregar**.

    ![Agregar una aplicación de cadena de bloques](./media/data-manager-cosmosdb/add-application.png)

    Escriba el nombre de la aplicación de cadena de bloques y las direcciones URL de la ABI del contrato inteligente y los códigos de bytes.

    Configuración | Descripción
    --------|------------
    Nombre | Escriba un nombre único para la aplicación de cadena de bloques de la que se va a realizar el seguimiento.
    ABI del contrato | Ruta de acceso URL al archivo ABI del contrato. Para más información, consulte [Crear una dirección URL de la ABI del contrato y el código de bytes](#create-contract-abi-and-bytecode-url).
    Código de bytes del contrato | Ruta de acceso URL al archivo de código de bytes. Para más información, consulte [Crear una dirección URL de la ABI del contrato y el código de bytes](#create-contract-abi-and-bytecode-url).

1. Seleccione **Aceptar**.

    Una vez creada la aplicación, esta aparece en la lista de aplicaciones de cadena de bloques.

    ![Lista de aplicaciones de cadena de bloques](./media/data-manager-cosmosdb/artifact-list.png)

Puede eliminar la cuenta de Azure Storage o usarla para configurar más aplicaciones de cadena de bloques. Si quiere eliminar la cuenta de Azure Storage, puede eliminar el grupo de recursos. Al eliminar el grupo de recursos, también se elimina la cuenta de almacenamiento asociada y cualquier otro recurso que esté asociado a dicho grupo.

## <a name="create-azure-cosmos-db"></a>Creación de una base de datos de Azure Cosmos DB

[!INCLUDE [cosmos-db-create-storage-account](../../../includes/cosmos-db-create-dbaccount.md)]

### <a name="add-a-database-and-container"></a>Adición de una base de datos y un contenedor

Para crear una base de datos y un contenedor, puede usar Data Explorer en Azure Portal.

1. Seleccione **Data Explorer** en el panel de navegación izquierdo de la página de la cuenta de Azure Cosmos DB y, después, seleccione **Nuevo contenedor**.
1. En el panel **Agregar contenedor**, especifique la configuración del nuevo contenedor.

    ![Adición de la configuración del contenedor](./media/data-manager-cosmosdb/add-container.png)

    | Configuración | Descripción
    |---------|-------------|
    | Identificador de base de datos | Escriba **blockchain-data** como nombre de la nueva base de datos. |
    | Throughput | Deje el rendimiento en **400** unidades de solicitud por segundo (RU/s). Si quiere reducir la latencia, puede escalar verticalmente el rendimiento más adelante.|
    | Id. de contenedor | Escriba **Messages** como nombre del nuevo contenedor. |
    | Clave de partición | Use **/MessageType** como clave de partición. |

1. Seleccione **Aceptar**. Data Explorer muestra la nueva base de datos y el contenedor que ha creado.

## <a name="create-logic-app"></a>Creación de la aplicación lógica

Azure Logic Apps le ayuda a programar y automatizar procesos empresariales y flujos de trabajo cuando necesita integrar sistemas y servicios. Puede usar una aplicación lógica para conectar Event Grid a Azure Cosmos DB.

1. En [Azure Portal](https://portal.azure.com), seleccione **Crear un recurso** > **Integración** > **Logic App**.
1. Proporcione detalles sobre dónde va a crear la aplicación lógica. Seleccione **Crear** cuando haya terminado.

    Para más información sobre la creación de aplicaciones lógicas, consulte [Creación de flujos de trabajo automatizados con Azure Logic Apps](../../logic-apps/quickstart-create-first-logic-app-workflow.md).

1. Después de que Azure implemente la aplicación, seleccione el recurso de la aplicación lógica.
1. En el Diseñador de aplicaciones lógicas, en **Plantillas**, seleccione **Aplicación lógica en blanco**.

### <a name="add-event-grid-trigger"></a>Incorporación del desencadenador de Event Grid

Cada aplicación lógica debe comenzar con un desencadenador, que se activa cuando sucede un evento específico o cuando se cumple una condición determinada. Cada vez que el desencadenador se activa, el motor de Logic Apps crea una instancia de aplicación lógica que inicia y ejecuta el flujo de trabajo. Use un desencadenador de Azure Event Grid para enviar datos de transacciones de cadena de bloques de Event Grid a Cosmos DB.

1. En el diseñador de Logic Apps, busque y seleccione el conector de **Azure Event Grid**.
1. En la pestaña **Triggers** (Desencadenadores), seleccione **Cuando se produce un evento de recursos**.
1. Cree una conexión de API con su tema de Event Grid.

    ![Configuración de un desencadenador de Event Grid](./media/data-manager-cosmosdb/event-grid-trigger.png)

    | Configuración | Descripción
    |---------|-------------|
    | Subscription | Elija una suscripción que contenga el tema de Event Grid. |
    | Tipo de recurso | Elija **Microsoft.EventGrid.Topics**. |
    | Nombre de recurso | Elija el nombre del tema de Event Grid en que Blockchain Data Manager envía mensajes de datos de transacciones. |

### <a name="add-cosmos-db-action"></a>Agregar acción de Cosmos DB

Agregue una acción para crear un documento en Cosmos DB para cada transacción. Use el tipo de mensaje de transacción como clave de partición para clasificar los mensajes.

1. Seleccione **Nuevo paso**.
1. En **Elegir una acción**, busque **Azure Cosmos DB**.
1. Elija **Azure Cosmos DB > Acciones > Crear o actualizar documento**.
1. Cree una conexión de API con la base de datos de Cosmos DB.

    ![Configuración de conexión de Cosmos DB](./media/data-manager-cosmosdb/cosmosdb-connection.png)

    | Configuración | Descripción
    |---------|-------------|
    | Nombre de la conexión | Elija una suscripción que contenga el tema de Event Grid. |
    | Cuenta de DocumentDB | Elija la cuenta de DocumentDB que creó en la sección [Crear cuenta de Azure Cosmos DB](#create-azure-cosmos-db). |

1. Escriba el **Id. de base de datos** y el **Id. de colección** para la instancia de Azure Cosmos DB que creó anteriormente en la sección [Adición de una base de datos y un contenedor](#add-a-database-and-container).

1. Seleccione la opción **Documento**. En el menú desplegable *Agregar contenido dinámico*, seleccione **Expresión** y copie y pegue la siguiente expresión:

    ```
    addProperty(triggerBody()?['data'], 'id', utcNow())
    ```

    La expresión obtiene la parte de datos del mensaje y establece el identificador en un valor de marca de tiempo.

1. Seleccione **Agregar nuevo parámetro** y elija **Valor de la clave de partición**.
1. Defina el **Valor de la clave de partición** en `"@{triggerBody()['data']['MessageType']}"`. El valor debe ir entre comillas dobles.

    ![Configuración del Diseñador de Logic Apps con Cosmos DB](./media/data-manager-cosmosdb/create-action.png)

    El valor establece la clave de partición en el tipo de mensaje de transacción.

1. Seleccione **Guardar**.

La aplicación lógica supervisa el tema de Event Grid. Cuando se envía un nuevo mensaje de transacción desde Blockchain Data Manager, la aplicación lógica crea un documento en Cosmos DB.

## <a name="send-a-transaction"></a>Enviar una transacción

A continuación, envíe una transacción al libro de contabilidad de cadena de bloques para probar lo que ha creado. Use la función **SendRequest** del contrato de **HelloBlockchain** que ha creado en la sección de requisitos previos del [Tutorial: Uso de Visual Studio Code para crear, compilar e implementar contratos inteligentes](send-transaction.md)

1. Use la página de interacción del contrato inteligente del kit de desarrollo de Azure Blockchain para llamar a la función **SendRequest**. Haga clic con el botón derecho en **HelloBlockchain.sol** y elija **Show Smart Contract Interaction Page** (Mostrar página de interacción del contrato inteligente) en el menú.

    ![Selección de Show Smart Contract Interaction Page (Mostrar página de interacción del contrato inteligente) en el menú](./media/data-manager-cosmosdb/contract-interaction.png)

1. Elija la acción de contrato **SendRequest** y escriba **Hello, Blockchain!** para el parámetro **requestMessage**. Seleccione **Execute** (Ejecutar) para llamar a la función **SendRequest** mediante una transacción.

    ![Ejecutar la acción SendRequest](./media/data-manager-cosmosdb/sendrequest-action.png)

La función SendRequest establece los campos **RequestMessage** y **State**. El estado actual de **RequestMessage** es el argumento que pasó, **Hello, blockchain**. El valor del campo **State** permanece en **Request**.

## <a name="view-transaction-data"></a>Ver datos de transacción

Ahora que ha conectado Blockchain Data Manager con Azure Cosmos DB, puede ver los mensajes de transacciones de cadena de bloques en Cosmos DB Data Explorer.

1. Vaya a la vista de Cosmos DB Data Explorer. Por ejemplo, **cosmosdb-blockchain > Data Explorer > blockchain-data > Mensajes > Elementos**.

    ![Explorador de datos de Cosmos DB](./media/data-manager-cosmosdb/data-explorer.png)

    Data Explorer muestra los mensajes de datos de cadena de bloques que se crearon en la base de datos de Cosmos DB.

1. Para navegar por los mensajes, seleccione el identificador del elemento y busque el mensaje con el hash de transacción coincidente.

    [![La captura de pantalla muestra los detalles de la transacción de cadena de bloques de un elemento seleccionado.](./media/data-manager-cosmosdb/raw-msg.png)](./media/data-manager-cosmosdb/raw-msg.png#lightbox)

    El mensaje de transacción sin procesar contiene detalles sobre la transacción. Sin embargo, la información de la propiedad está cifrada.

    Como ha agregado el contrato inteligente de HelloBlockchain a la instancia de Blockchain Data Manager, también se envía un tipo de mensaje **ContractProperties** que contiene información de la propiedad descodificada.

1. Busque el mensaje **ContractProperties** de la transacción. Debería ser el siguiente mensaje de la lista.

    [![Detalle de la transacción de la cadena de bloques](./media/data-manager-cosmosdb/properties-msg.png)](./media/data-manager-cosmosdb/properties-msg.png#lightbox)

    La matriz **DecodedProperties** contiene las propiedades de la transacción.

Felicidades. Ha creado correctamente un explorador de mensajes de transacción con Blockchain Data Manager y Azure Cosmos DB.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no los necesite, puede eliminar los recursos y los grupos de recursos que usó para este tutorial. Para eliminar un grupo de recursos:

1. En Azure Portal, vaya a **Grupo de recursos** en el panel de navegación izquierdo y seleccione el grupo de recursos que desea eliminar.
1. Seleccione **Eliminar grupo de recursos**. Compruebe la eliminación escribiendo el nombre del grupo de recursos y seleccionando **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre la integración con los libros de contabilidad de cadena de bloques.

> [!div class="nextstepaction"]
> [Uso del conector Ethereum Blockchain con Azure Logic Apps](ethereum-logic-app.md)
