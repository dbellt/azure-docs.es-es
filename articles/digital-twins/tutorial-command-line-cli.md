---
title: 'Tutorial: Creación de un grafo en Azure Digital Twins (CLI)'
titleSuffix: Azure Digital Twins
description: Tutorial para crear un escenario de Azure Digital Twins mediante la CLI de Azure
author: baanders
ms.author: baanders
ms.date: 2/26/2021
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: 578befe3e26ebb42fa2172976e07d0a5836e3743
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107166"
---
# <a name="tutorial-create-an-azure-digital-twins-graph-using-the-azure-cli"></a>Tutorial: Creación de un grafo de Azure Digital Twins mediante la CLI de Azure

[!INCLUDE [digital-twins-tutorial-selector.md](../../includes/digital-twins-tutorial-selector.md)]

En este tutorial creará un grafo en Azure digital gemelos con modelos, gemelos y relaciones. La herramienta para este tutorial es el [conjunto de comandos de Azure Digital Twins para la **CLI de Azure**](how-to-use-cli.md). 

Puede usar estos comandos de la CLI para realizar acciones básicas de Azure Digital Twins, como cargar modelos, crear y modificar gemelos, y crear relaciones. También puede consultar la [documentación de referencia del conjunto de comandos *az dt*](/cli/azure/dt) para ver el conjunto completo de comandos de la CLI.

En este tutorial:
> [!div class="checklist"]
> * Modelará un entorno.
> * Creación de gemelos digitales
> * Agregará relaciones para formar un grafo.
> * Consultará el grafo para responder preguntas.

## <a name="prerequisites"></a>Requisitos previos

Para realizar los pasos de este tutorial, debe completar primero los siguientes requisitos previos.

Si no tiene una suscripción a Azure, **cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)** antes de empezar.

### <a name="download-the-sample-models"></a>Descarga de los modelos de ejemplo

En el tutorial se usan dos modelos escritos previamente que forman parte del [proyecto de ejemplo integral](/samples/azure-samples/digital-twins-samples/digital-twins-samples/) de C# para Azure Digital Twins. Los archivos de modelo se encuentran aquí: 
* [*Room.json*](https://github.com/Azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Room.json)
* [*Floor.json*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json)

Para obtener los archivos en la máquina, use los vínculos de navegación anteriores y copie los cuerpos de archivo en archivos locales de la máquina con los mismos nombres (*Room.json* y *Floor.json*).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="set-up-cloud-shell-session"></a>Configuración de una sesión de Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

### <a name="prepare-an-azure-digital-twins-instance"></a>Preparación de una instancia de Azure Digital Twins

Para trabajar con Azure Digital Twins en este artículo, antes es preciso **configurar una instancia de Azure Digital Twins** y los permisos necesarios para usarla. Si ya tiene una instancia de Azure Digital Twins configurada del trabajo anterior, puede usarla.

En caso contrario, siga las instrucciones de [*Procedimiento de configuración de una instancia y de autenticación*](how-to-set-up-instance-cli.md). Las instrucciones también contienen pasos para comprobar que ha completado cada paso correctamente y está listo para pasar a usar la nueva instancia.

Después de configurar la instancia de Azure Digital Twins, tome nota de los siguientes valores que necesitará para conectarse a la instancia más adelante:
* El **_nombre de host_** de la instancia.
* La **suscripción de Azure** que usó para crear la instancia. 

Puede obtener estos dos valores para su instancia de la salida del siguiente comando de la CLI de Azure: 

```azurecli-interactive
az dt show -n <ADT_instance_name>
```

:::image type="content" source="media/tutorial-command-line/cli/instance-details.png" alt-text="Captura de pantalla de la ventana del explorador de Cloud Shell que muestra la salida del comando az dt show. El campo hostName y el identificador de suscripción (parte del campo id) están resaltados.":::

## <a name="model-a-physical-environment-with-dtdl"></a>Modelado de un entorno físico con DTDL

Ahora que la CLI y la instancia de Azure Digital Twins están configuradas, puede empezar a crear un grafo para un escenario. 

El primer paso para crear una solución de Azure Digital Twins es definir los [**modelos**](concepts-models.md) de gemelos del entorno. 

Los modelos son parecidos a las clases de los lenguajes de programación orientados a objetos en el sentido de que proporcionan plantillas definidas por el usuario para [gemelos digitales](concepts-twins-graph.md), las cuales se siguen. Más tarde también se crean instancias de estas plantillas. Se escriben en un lenguaje tipo JSON llamado **lenguaje de definición de gemelos digitales (DTDL)** y pueden definir las *propiedades*, la *telemetría*, las *relaciones* y los *componentes* de un gemelo.

> [!NOTE]
> DTDL también permite la definición de *comandos* en digital twins. Sin embargo, en este momento no se admiten comandos en el servicio Azure Digital Twins.

En la máquina, vaya al archivo *Room.json* el archivo que creó en la sección [Requisitos previos](#prerequisites). Ábralo en un editor de código y cámbielo de las siguientes maneras:

[!INCLUDE [digital-twins-tutorial-model-create.md](../../includes/digital-twins-tutorial-model-create.md)]

### <a name="upload-models-to-azure-digital-twins"></a>Carga de modelos en Azure Digital Twins

Después de diseñar los modelos, debe cargarlos en la instancia de Azure Digital Twins. De esta forma se configura la instancia del servicio Azure Digital Twins con su propio vocabulario de dominio personalizado. Cuando haya cargado los modelos, puede crear instancias gemelas que los usen.

1. Para agregar modelos mediante Cloud Shell, deberá cargar los archivos de modelo en el almacenamiento de Cloud Shell para que estén disponibles al ejecutar el comando de Cloud Shell que los usa. Para ello, seleccione el icono de carga/descarga de archivos y elija "Upload" (Cargar).

    :::image type="content" source="media/how-to-set-up-instance/cloud-shell/cloud-shell-upload.png" alt-text="Captura de pantalla de la ventana del explorador de Cloud Shell que muestra la selección del icono Upload (Cargar).":::
    
    Vaya al archivo *Room.json* en la máquina y pulse "Open" (Abrir). A continuación, repita este paso para *Floor.json*.

1. A continuación, use el comando [**az dt model create**](/cli/azure/dt/model#az_dt_model_create) como se muestra a continuación para cargar el modelo *Room* actualizado en la instancia de Azure Digital Twins. El segundo comando carga otro modelo, *Floor*, que también usará en la sección siguiente para crear distintos tipos de gemelos.

    ```azurecli-interactive
    az dt model create -n <ADT_instance_name> --models Room.json
    az dt model create -n <ADT_instance_name> --models Floor.json
    ```
    
    La salida de cada comando mostrará información sobre el modelo cargado correctamente.

    >[!TIP]
    >También puede cargar todos los modelos dentro de un directorio al mismo tiempo mediante la opción `--from-directory` para el comando model create. Para más información, consulte [Parámetros opcionales para *az dt model create*](/cli/azure/dt/model#az_dt_model_create-optional-parameters).

1. Compruebe que los modelos se crearon con el comando [**az dt model list**](/cli/azure/dt/model#az_dt_model_list), como se muestra a continuación. Así se imprimirá una lista de todos los modelos que se han cargado en la instancia de Azure Digital Twins con toda su información. 

    ```azurecli-interactive
    az dt model list -n <ADT_instance_name> --definition
    ```
    
    Busque el modelo *Room* editado en los resultados:
    
    :::image type="content" source="media/tutorial-command-line/cli/output-get-models.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado del comando model list, que incluye el modelo Room actualizado." lightbox="media/tutorial-command-line/cli/output-get-models.png":::

### <a name="errors"></a>Errors

La CLI también controla los errores del servicio. 

Vuelva a ejecutar el comando `az dt model create` para intentar volver a cargar uno de los mismos modelos que acaba de cargar, por segunda vez:

```azurecli-interactive
az dt model create -n <ADT_instance_name> --models Room.json
```

Dado que los modelos no se pueden sobrescribir, esto devolverá un código de error `ModelIdAlreadyExists`.

## <a name="create-digital-twins"></a>Creación de gemelos digitales

Ahora que algunos modelos se han cargado en la instancia de Azure Digital Twins, puede crear [**gemelos digitales**](concepts-twins-graph.md) basados en las definiciones de modelo. Los gemelos digitales representan las entidades del entorno empresarial, como los sensores de una granja, las salas de un edificio o las luces de un coche. 

Para crear un gemelo digital, se usa el comando [**az dt twin create**](/cli/azure/dt/twin#az_dt_twin_create). Se debe hacer referencia al modelo en el que se basa el gemelo y, opcionalmente, puede definir los valores iniciales de las propiedades del modelo. En esta fase, no es necesario pasar ninguna información de relación.

1. Ejecute este código en Cloud Shell para crear varios gemelos, según el modelo *Room* que ha actualizado anteriormente y otro modelo, *Floor*. Recuerde que *Room* tiene tres propiedades, por lo que puede proporcionar argumentos con los valores iniciales de estas. (La inicialización de los valores de propiedad es opcional en general, pero son necesarios para este tutorial).

    ```azurecli-interactive
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room0 --properties '{"RoomName":"Room0", "Temperature":70, "HumidityLevel":30}'
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room1 --properties '{"RoomName":"Room1", "Temperature":"80", "HumidityLevel":"60"}'
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Floor;1" --twin-id floor0
    az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Floor;1" --twin-id floor1
    ```

    >[!NOTE]
    > Si usa Cloud Shell en el entorno de PowerShell, quizá deba usar los caracteres de comillas como escape para que el valor JSON `--properties` se analice correctamente. Con esta edición, los comandos para crear gemelos de Room tienen el siguiente aspecto:
    >
    > ```azurecli-interactive
    > az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room0 --properties '{\"RoomName\":\"Room0\", \"Temperature\":70, \"HumidityLevel\":30}'
    > az dt twin create -n <ADT_instance_name> --dtmi "dtmi:example:Room;2" --twin-id room1 --properties '{\"RoomName\":\"Room1\", \"Temperature\":80, \"HumidityLevel\":60}'
    > ```
    > Esto se refleja en la siguiente captura de pantalla.
    
    La salida de cada comando mostrará información sobre el gemelo creado correctamente (incluidas las propiedades de los gemelos de Room que se inicializaron con ellos).

1. Puede comprobar la creación de los gemelos con el comando [**az dt twin query**](/cli/azure/dt/twin#az_dt_twin_query), como se muestra a continuación. La consulta mostrada busca todos los gemelos digitales en la instancia de Azure Digital Twins.
    
    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DIGITALTWINS"
    ```
    
    Busque los gemelos *room0*, *room1*, *floor0* y *floor1* en los resultados. Este es un extracto que muestra parte del resultado de la consulta.
    
    :::image type="content" source="media/tutorial-command-line/cli/output-query-all.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado parcial de una consulta de gemelos, que incluye room0 y room1." lightbox="media/tutorial-command-line/cli/output-query-all.png":::

### <a name="modify-a-digital-twin"></a>Modificación de un gemelo digital

También puede modificar las propiedades de un gemelo que haya creado. 

1. Pruebe a ejecutar el comando [**az dt twin update**](/cli/azure/dt/twin#az_dt_twin_update) para cambiar la propiedad RoomName de *room0* de *Room0* a *PresidentialSuite*:

    ```azurecli-interactive
    az dt twin update -n <ADT_instance_name> --twin-id room0 --json-patch '{"op":"add", "path":"/RoomName", "value": "PresidentialSuite"}'
    ```
    
    >[!NOTE]
    > Si usa Cloud Shell en el entorno de PowerShell, quizá deba usar los caracteres de comillas como escape para que el valor JSON `--json-patch` se analice correctamente. Con esta edición, el comando para actualizar el gemelo tiene el siguiente aspecto:
    >
    > ```azurecli-interactive
    > az dt twin update -n <ADT_instance_name> --twin-id room0 --json-patch '{\"op\":\"add\", \"path\":\"/RoomName\", \"value\": \"PresidentialSuite\"}'
    > ```
    > Esto se refleja en la siguiente captura de pantalla.
    
    La salida de este comando mostrará la información actual del gemelo y el nuevo valor de `RoomName` debería verse en el resultado.

    :::image type="content" source="media/tutorial-command-line/cli/output-update-twin.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado del comando update, que incluye una propiedad RoomName como PresidentialSuite." lightbox="media/tutorial-command-line/cli/output-update-twin.png":::

1. Para comprobar que la actualización se ha realizado correctamente, ejecute el comando [**az dt twin show**](/cli/azure/dt/twin#az_dt_twin_show) para ver la información de *room0*:

    ```azurecli-interactive
    az dt twin show -n <ADT_instance_name> --twin-id room0
    ```
    
    La salida reflejará el nombre actualizado.

## <a name="create-a-graph-by-adding-relationships"></a>Creación de un gráfico mediante la adición de relaciones

A continuación, puede crear algunas **relaciones** entre estos gemelos para conectarlos en un [**gráfico de gemelos**](concepts-twins-graph.md). Los gráficos de gemelos se utilizan para representar un entorno completo. 

Los tipos de relaciones que puede crear de un gemelo a otro se definen dentro de los [modelos](#model-a-physical-environment-with-dtdl) que cargó anteriormente. La [definición del modelo para *Floor*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) especifica que las plantas pueden tener un tipo de relación denominado *contains*. Esto permite crear una relación de tipo *contains* a partir de cada gemelo de *Floor* en la planta correspondiente que lo contiene.

Para agregar una relación, use el comando [**az dt twin relationship create**](/cli/azure/dt/twin/relationship#az_dt_twin_relationship_create). Especifique el gemelo del que procede la relación, el tipo de relación y el gemelo con el que conecta la relación. Por último, asígnele un identificador único a la relación. Si se definió una relación para que tenga propiedades, también puede inicializar las propiedades de la relación en este comando.

1. Ejecute el código siguiente para agregar una relación de tipo *contains* a partir de los gemelos de *Floor* que creó anteriormente en el gemelo de *Room* correspondiente. Las relaciones se denominan *relationship0* y *relationship1*.

    ```azurecli-interactive
    az dt twin relationship create -n <ADT_instance_name> --relationship-id relationship0 --relationship contains --twin-id floor0 --target room0
    az dt twin relationship create -n <ADT_instance_name> --relationship-id relationship1 --relationship contains --twin-id floor1 --target room1
    ```
    
    >[!TIP]
    >La relación *contains* en el modelo de [ *Floor*](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) también se ha definido con dos propiedades, `ownershipUser` y `ownershipDepartment`, por lo que también puede proporcionar argumentos con los valores iniciales para estas al crear las relaciones.
    > Para crear una relación con estas propiedades inicializadas, agregue la opción `--properties` a cualquiera de los comandos anteriores, como se indica a continuación:
    > ```azurecli-interactive
    > ... --properties '{"ownershipUser":"MyUser", "ownershipDepartment":"MyDepartment"}'
    > ``` 
    > 
    > Si usa Cloud Shell en el entorno de PowerShell, quizá deba usar los caracteres de comillas como escape para que el valor JSON `--properties` se analice correctamente.
    
    La salida de cada comando mostrará información sobre la relación creada correctamente.

1. Puede comprobar las relaciones con cualquiera de los siguientes comandos, que consultan las relaciones en la instancia de Azure Digital Twins.
    * Para ver todas las relaciones que proceden de cada planta (vista de las relaciones desde un lado):
        ```azurecli-interactive
        az dt twin relationship list -n <ADT_instance_name> --twin-id floor0
        az dt twin relationship list -n <ADT_instance_name> --twin-id floor1
        ```
    * Para ver todas las relaciones que llegan a cada habitación (vista de la relación desde el "otro" lado):
        ```azurecli-interactive
        az dt twin relationship list -n <ADT_instance_name> --twin-id room0 --incoming
        az dt twin relationship list -n <ADT_instance_name> --twin-id room1 --incoming
        ```
    * Para buscar estas relaciones individualmente, por identificador:
        ```azurecli-interactive
        az dt twin relationship show -n <ADT_instance_name> --twin-id floor0 --relationship-id relationship0
        az dt twin relationship show -n <ADT_instance_name> --twin-id floor1 --relationship-id relationship1
        ```

Los gemelos y las relaciones que ha configurado en este tutorial forman el siguiente gráfico conceptual:

:::image type="content" source="media/tutorial-command-line/app/sample-graph.png" alt-text="Un diagrama que muestra un grafo conceptual. floor0 conectado a través de relationship0 con room0 y floor1, a través de relationship1 con room1." border="false" lightbox="media/tutorial-command-line/app/sample-graph.png":::

## <a name="query-the-twin-graph-to-answer-environment-questions"></a>Consulta del gráfico de gemelos para responder a las preguntas del entorno

Una de las principales características de Azure Digital Twins es la posibilidad de [consultar](concepts-query-language.md) el gráfico de gemelos de forma fácil y eficaz para responder a las preguntas sobre el entorno. En la CLI de Azure, esto se hace con el comando [**az dt twin query**](/cli/azure/dt/twin#az_dt_twin_query).

Ejecute las siguientes consultas en Cloud Shell para responder a algunas preguntas sobre el entorno de ejemplo.

1. **¿Cuáles son las entidades de mi entorno que se representan en Azure Digital Twins?** (consulta de todo)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DIGITALTWINS"
    ```

    Esto le permite evaluar su entorno de un vistazo y asegurarse de que todo se representa tal y como desea en Azure Digital Twins. El resultado es una salida que contiene cada gemelo digital con sus detalles. Aquí se muestra un extracto:

    :::image type="content" source="media/tutorial-command-line/cli/output-query-all.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado parcial de una consulta de gemelos, que incluye room0 y room1." lightbox="media/tutorial-command-line/cli/output-query-all.png":::

    >[!TIP]
    >Puede que reconozca que es el mismo comando que usó en la sección [*Creación de gemelos digitales*](#create-digital-twins) anterior para buscar todos los gemelos digitales de Azure en la instancia.

1. **¿Cuáles son todas las salas de mi entorno?** (consulta por modelo)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DIGITALTWINS T WHERE IS_OF_MODEL(T, 'dtmi:example:Room;2')"
    ```

    Puede restringir la consulta a los gemelos de un tipo determinado para tener información más específica sobre lo que se representa. El resultado muestra *room0* y *room1*, pero **no** *floor0* o *floor1* (dado que son plantas, no salas).
    
    :::image type="content" source="media/tutorial-command-line/cli/output-query-model.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado de la consulta del modelo, que solo incluye room0 y room1." lightbox="media/tutorial-command-line/cli/output-query-model.png":::

1. **¿Cuáles son todas las salas de *floor0*?** (consulta por relación)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.`$dtId = 'floor0'"
    ```

    Puede realizar consultas en función de las relaciones del gráfico para tener información sobre cómo se conectan los gemelos o para restringir la consulta a un área determinada. Solo *room0* está en *floor0*, por lo que es la única sala del resultado.

    :::image type="content" source="media/tutorial-command-line/cli/output-query-relationship.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado de la consulta de relación, que incluye room0." lightbox="media/tutorial-command-line/cli/output-query-relationship.png":::

    > [!NOTE]
    > Observe que se consulta un identificador del gemelo (como *floor0* en la consulta anterior) mediante el campo de metadatos `$dtId`. 
    >
    >Cuando se usa Cloud Shell para ejecutar una consulta con campos de metadatos como este que comienzan por `$`, se debe eludir `$` con un acento grave para que Cloud Shell sepa que no es una variable y se debe consumir como literal en el texto de la consulta. Esto se refleja en la captura de pantalla anterior.

1. **¿Cuáles son todos los gemelos de mi entorno con una temperatura superior a 75?** (consulta por propiedad)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT * FROM DigitalTwins T WHERE T.Temperature > 75"
    ```

    Puede consultar el gráfico en función de las propiedades para responder a diversas preguntas, como buscar valores atípicos en el entorno que puedan necesitar atención. También se admiten otros operadores de comparación ( *<* , *>* , *=* o *!=* ). *room1* se muestra aquí en los resultados porque tiene una temperatura de 80.

    :::image type="content" source="media/tutorial-command-line/cli/output-query-property.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado de la consulta de propiedad, que incluye solo room1." lightbox="media/tutorial-command-line/cli/output-query-property.png":::

1. **¿Cuáles son todas las salas de *floor0* con una temperatura superior a 75?** (consulta compuesta)

    ```azurecli-interactive
    az dt twin query -n <ADT_instance_name> -q "SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.`$dtId = 'floor0' AND IS_OF_MODEL(room, 'dtmi:example:Room;2') AND room.Temperature > 75"
    ```

    También puede combinar las consultas anteriores como lo haría en SQL, mediante operadores combinados como `AND`, `OR``NOT`. Esta consulta usa `AND` para que la consulta anterior sobre las temperaturas gemelas sea más específica. El resultado ahora solo incluye las salas con temperaturas superiores a 75 que se encuentran en *floor0*, que en este caso no es ninguna de ellas. El conjunto de resultados está vacío.

    :::image type="content" source="media/tutorial-command-line/cli/output-query-compound.png" alt-text="Captura de pantalla de Cloud Shell que muestra el resultado de la consulta compuesta, que no incluye ningún elemento." lightbox="media/tutorial-command-line/cli/output-query-compound.png":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar este tutorial, puede elegir los recursos que desea quitar en función de lo que desee hacer a continuación.

* **Si va a continuar con el siguiente tutorial**, puede conservar los recursos que configure aquí y volver a usar la instancia de Azure Digital Twins sin borrar nada.

* **Si desea seguir usando la instancia de Azure Digital Twins, pero borrar todos sus modelos, gemelos y relaciones**, puede usar los comandos [**az dt twin rellationship delete**](/cli/azure/dt/twin/relationship#az_dt_twin_relationship_delete), [**az dt twin delete**](/cli/azure/dt/twin#az_dt_twin_delete) y [**az ddt model delete**](/cli/azure/dt/model#az_dt_model_delete) para borrar las relaciones, los gemelos y los modelos de la instancia, respectivamente.

[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

También puede eliminar los archivos de modelo creados en la máquina local.

## <a name="next-steps"></a>Pasos siguientes 

En este tutorial ha empezado a usar Azure Digital Twins mediante la creación de un grafo en la instancia con la CLI de Azure. Ha creado modelos, gemelos digitales y relaciones para formar un grafo. También ha ejecutado algunas consultas en el grafo para hacerse una idea de los tipos de preguntas que Azure Digital Twins puede responder sobre un entorno.

Continúe con el siguiente tutorial para combinar Azure Digital Twins con otros servicios de Azure con el fin de completar un escenario integral basado en datos:
> [!div class="nextstepaction"]
> [*Tutorial: Conexión de una solución de un extremo a otro*](tutorial-end-to-end.md).