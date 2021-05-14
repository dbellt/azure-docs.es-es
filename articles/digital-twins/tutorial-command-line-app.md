---
title: 'Tutorial: Creación de un grafo en Azure Digital Twins (aplicación cliente)'
titleSuffix: Azure Digital Twins
description: Tutorial para crear un escenario de Azure Digital Twins mediante una aplicación de línea de comandos de ejemplo
author: baanders
ms.author: baanders
ms.date: 5/8/2020
ms.topic: tutorial
ms.service: digital-twins
ms.openlocfilehash: 419e609c4b78007f215d67ab4a69671bc9cbb198
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205636"
---
# <a name="tutorial-create-an-azure-digital-twins-graph-using-a-sample-client-app"></a>Tutorial: Creación de un grafo de Azure Digital Twins mediante una aplicación cliente de ejemplo

[!INCLUDE [digital-twins-tutorial-selector.md](../../includes/digital-twins-tutorial-selector.md)]

En este tutorial creará un grafo en Azure Digital Twins con modelos, gemelos y relaciones. La herramienta de este tutorial es una **aplicación cliente de línea de comandos de ejemplo** para interactuar con una instancia de Azure Digital Twins. La aplicación cliente es similar a la escrita en el [Tutorial: Programación de una aplicación cliente](tutorial-code.md).

Puede usar este ejemplo para realizar acciones básicas de Azure Digital Twins, como cargar modelos, crear y modificar gemelos y crear relaciones. No obstante, si lo prefiere, también puede examinar el [código del ejemplo](https://github.com/Azure-Samples/digital-twins-samples/tree/master/) para aprender sobre las API de Azure Digital Twins y practicar la implementación de sus propios comandos mediante la modificación del proyecto de ejemplo.

En este tutorial:
> [!div class="checklist"]
> * Modelará un entorno.
> * Creación de gemelos digitales
> * Agregará relaciones para formar un grafo.
> * Consultará el grafo para responder preguntas.

[!INCLUDE [Azure Digital Twins tutorial: sample prerequisites](../../includes/digital-twins-tutorial-sample-prereqs.md)]

[!INCLUDE [Azure Digital Twins tutorial: configure the sample project](../../includes/digital-twins-tutorial-sample-configure.md)]

### <a name="run-the-sample-project"></a>Ejecución del proyecto de ejemplo

Ahora que la aplicación y la autenticación están configuradas, ejecute el proyecto con este botón de la barra de herramientas:

:::image type="content" source="media/tutorial-command-line/app/start-button-sample.png" alt-text="Captura de pantalla del botón de inicio de Visual Studio (proyecto SampleClientApp)." lightbox="media/tutorial-command-line/app/start-button-sample.png":::

Se abre una ventana de la consola, se lleva a cabo la autenticación y se espera un comando. 
* La autenticación se realiza a través del explorador: el explorador web predeterminado se abrirá con un aviso de autenticación. Use este aviso para iniciar sesión con sus credenciales de Azure. Luego, puede cerrar la pestaña o la ventana del explorador.

Esta es una captura de pantalla de la apariencia de la consola del proyecto:

:::image type="content" source="media/tutorial-command-line/app/command-line-app.png" alt-text="Captura de pantalla del mensaje de bienvenida de la aplicación de línea de comandos." lightbox="media/tutorial-command-line/app/command-line-app.png":::

> [!TIP]
> Si quiere una lista de todos los comandos posibles que puede usar con este proyecto, escriba `help` en la consola del proyecto y presione Entrar.

Mantenga la consola del proyecto en ejecución durante el resto de los pasos de este tutorial.

## <a name="model-a-physical-environment-with-dtdl"></a>Modelado de un entorno físico con DTDL

Ahora que la instancia de Azure Digital Twins y la aplicación de ejemplo están configuradas, puede empezar a crear un grafo de un escenario. 

El primer paso para crear una solución de Azure Digital Twins es definir los [modelos](concepts-models.md) de gemelos del entorno. 

Los modelos son parecidos a las clases de los lenguajes de programación orientados a objetos en el sentido de que proporcionan plantillas definidas por el usuario para [gemelos digitales](concepts-twins-graph.md), las cuales se siguen. Más tarde también se crean instancias de estas plantillas. Se escriben en un lenguaje tipo JSON llamado **lenguaje de definición de gemelos digitales (DTDL)** y pueden definir las *propiedades*, la *telemetría*, las *relaciones* y los *componentes* de un gemelo.

> [!NOTE]
> DTDL también permite la definición de *comandos* en digital twins. Sin embargo, en este momento no se admiten comandos en el servicio Azure Digital Twins.

En la ventana de Visual Studio donde está abierto el proyecto _**AdtE2ESample**_, use el panel *Explorador de soluciones* para ir a la carpeta *AdtSampleApp\SampleClientApp\Models*. Esta carpeta contiene modelos de ejemplo.

Seleccione *Room.json* para abrirlo en la ventana de edición y cámbielo de la siguiente manera:

[!INCLUDE [digital-twins-tutorial-model-create.md](../../includes/digital-twins-tutorial-model-create.md)]

### <a name="upload-models-to-azure-digital-twins"></a>Carga de modelos en Azure Digital Twins

Después de diseñar los modelos, debe cargarlos en la instancia de Azure Digital Twins. De esta forma se configura la instancia del servicio Azure Digital Twins con su propio vocabulario de dominio personalizado. Cuando haya cargado los modelos, puede crear instancias gemelas que los usen.

1. En la ventana de la consola del proyecto, ejecute el siguiente comando para cargar el modelo *Room* actualizado, así como un modelo *Floor* que también usará en la sección siguiente para crear distintos tipos de gemelos.

    ```cmd/sh
    CreateModels Room Floor
    ```
    
    La salida debe indicar que los modelos se crearon correctamente.

1. Para comprobar que los modelos se han creado, ejecute el comando `GetModels true`. Se consultará la instancia de Azure Digital Twins para conocer todos los modelos que se han cargado y se imprimirá toda su información. Busque el modelo *Room* editado en los resultados:

    :::image type="content" source="media/tutorial-command-line/app/output-get-models.png" alt-text="Captura de pantalla del resultado de GetModels, que muestra el modelo de Room actualizado." lightbox="media/tutorial-command-line/app/output-get-models.png":::

### <a name="errors"></a>Errors

La aplicación de ejemplo también controla los errores del servicio. 

Vuelva a ejecutar el comando `CreateModels` para intentar volver a cargar uno de los mismos modelos que acaba de cargar, por segunda vez:

```cmd/sh
CreateModels Room
```

Dado que los modelos no se pueden sobrescribir, esto le devolverá un error de servicio.
Para información detallada sobre cómo eliminar modelos existentes, consulte [Procedimientos: Administración de modelos DTDL](how-to-manage-model.md).
```cmd/sh
Response 409: Service request failed.
Status: 409 (Conflict)

Content:
{"error":{"code":"ModelAlreadyExists","message":"Could not add model dtmi:example:Room;2 as it already exists. Use Model_List API to view models that already exist. See the Swagger example.(http://aka.ms/ModelListSwSmpl)"}}

Headers:
Strict-Transport-Security: REDACTED
Date: Wed, 20 May 2020 00:53:49 GMT
Content-Length: 223
Content-Type: application/json; charset=utf-8
```

## <a name="create-digital-twins"></a>Creación de gemelos digitales

Ahora que algunos modelos se han cargado en la instancia de Azure Digital Twins, puede crear [gemelos digitales](concepts-twins-graph.md) basados en las definiciones de modelo. Los gemelos digitales representan las entidades del entorno empresarial, como los sensores de una granja, las salas de un edificio o las luces de un coche. 

Para crear un gemelo digital, se usa el comando `CreateDigitalTwin`. Se debe hacer referencia al modelo en el que se basa el gemelo y, opcionalmente, puede definir los valores iniciales de las propiedades del modelo. En esta fase, no es necesario pasar ninguna información de relación.

1. Ejecute este código en la consola del proyecto en ejecución para crear varios gemelos, según el modelo *Room* que ha actualizado anteriormente y otro modelo, *Floor*. Recuerde que *Room* tiene tres propiedades, por lo que puede proporcionar argumentos con los valores iniciales de estas. (La inicialización de los valores de propiedad es opcional en general, pero son necesarios para este tutorial).

    ```cmd/sh
    CreateDigitalTwin dtmi:example:Room;2 room0 RoomName string Room0 Temperature double 70 HumidityLevel double 30
    CreateDigitalTwin dtmi:example:Room;2 room1 RoomName string Room1 Temperature double 80 HumidityLevel double 60
    CreateDigitalTwin dtmi:example:Floor;1 floor0
    CreateDigitalTwin dtmi:example:Floor;1 floor1
    ```

    La salida de estos comandos debe indicar que los gemelos se crearon correctamente. 
    
    :::image type="content" source="media/tutorial-command-line/app/output-create-digital-twin.png" alt-text="Captura de pantalla que muestra un extracto del resultado de los comandos de CreateDigitalTwin, que incluye floor0, floor1, room0 y room1." lightbox="media/tutorial-command-line/app/output-create-digital-twin.png":::

1. Para comprobar que se han creado los gemelos, ejecute el comando `Query`. Este comando consulta la instancia de Azure Digital Twins para conocer todos los gemelos digitales que contiene. Busque los gemelos *room0*, *room1*, *floor0* y *floor1* en los resultados.

### <a name="modify-a-digital-twin"></a>Modificación de un gemelo digital

También puede modificar las propiedades de un gemelo que haya creado. 

> [!NOTE]
> La API REST subyacente usa el formato de [revisión de JSON](http://jsonpatch.com/) para definir las actualizaciones en un gemelo. La aplicación de línea de comandos también usa este formato para ofrecer una experiencia auténtica con lo que esperan las API subyacentes.

1. Ejecute este comando para cambiar la propiedad RoomName de *room0* de *Room0* a *PresidentialSuite*:
    
    ```cmd/sh
    UpdateDigitalTwin room0 add /RoomName string PresidentialSuite
    ```
    
    La salida indicará que el gemelo se actualizó correctamente.

1. Para comprobar que la actualización se ha realizado correctamente, ejecute este comando para ver la información de *room0*:

    ```cmd/sh
    GetDigitalTwin room0
    ```
    
    La salida reflejará el nombre actualizado.


## <a name="create-a-graph-by-adding-relationships"></a>Creación de un gráfico mediante la adición de relaciones

A continuación, puede crear algunas **relaciones** entre estos gemelos para conectarlos en un [gráfico de gemelos](concepts-twins-graph.md). Los gráficos de gemelos se utilizan para representar un entorno completo. 

Los tipos de relaciones que puede crear de un gemelo a otro se definen dentro de los [modelos](#model-a-physical-environment-with-dtdl) que cargó anteriormente. La [definición de modelo de Floor](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) especifica que los pisos pueden tener un tipo de relación denominado *contains*. Esto permite crear una relación de tipo *contains* a partir de cada gemelo de *Floor* en la planta correspondiente que lo contiene.

Para agregar una relación, use el comando `CreateRelationship`. Especifique el gemelo del que procede la relación, el tipo de relación y el gemelo con el que conecta la relación. Por último, asígnele un identificador único a la relación.

1. Ejecute el código siguiente para agregar una relación "contiene" desde cada uno de los gemelos *Floor* que creó anteriormente hasta un gemelo *Room* correspondiente. Las relaciones se denominan *relationship0* y *relationship1*.

    ```cmd/sh
    CreateRelationship floor0 contains room0 relationship0
    CreateRelationship floor1 contains room1 relationship1
    ```

    >[!TIP]
    >La relación *contains* en el modelo de  [Floor](https://github.com/azure-Samples/digital-twins-samples/blob/master/AdtSampleApp/SampleClientApp/Models/Floor.json) también se ha definido con dos propiedades de cadena, `ownershipUser` y `ownershipDepartment`, por lo que también puede proporcionar argumentos con los valores iniciales para estas al crear las relaciones.
    > Esta es una versión alternativa del comando anterior para crear *relationship0* que también especifica valores iniciales para estas propiedades:
    > ```cmd/sh
    > CreateRelationship floor0 contains room0 relationship0 ownershipUser string MyUser ownershipDepartment string myDepartment
    > ``` 
    
    La salida de estos comandos confirma que las relaciones se crearon correctamente:
    
    :::image type="content" source="media/tutorial-command-line/app/output-create-relationship.png" alt-text="Captura de pantalla de un extracto del resultado de los comandos de CreateRelationship, que incluye relationship0 y relationship1." lightbox="media/tutorial-command-line/app/output-create-relationship.png":::

1. Puede comprobar las relaciones con cualquiera de los siguientes comandos, que consultan las relaciones en la instancia de Azure Digital Twins.
    * Para ver todas las relaciones que proceden de cada planta (vista de las relaciones desde un lado):
        ```cmd/sh
        GetRelationships floor0
        GetRelationships floor1
        ```
    * Para ver todas las relaciones que llegan a cada habitación (vista de la relación desde el "otro" lado):
        ```cmd/sh
        GetIncomingRelationships room0
        GetIncomingRelationships room1
        ```
    * Para buscar estas relaciones individualmente, por identificador:
        ```cmd/sh
        GetRelationship floor0 relationship0
        GetRelationship floor1 relationship1
        ```

Los gemelos y las relaciones que ha configurado en este tutorial forman el siguiente gráfico conceptual:

:::image type="content" source="media/tutorial-command-line/app/sample-graph.png" alt-text="Un diagrama que muestra un grafo conceptual. floor0 conectado a través de relationship0 con room0 y floor1, a través de relationship1 con room1." border="false" lightbox="media/tutorial-command-line/app/sample-graph.png":::

## <a name="query-the-twin-graph-to-answer-environment-questions"></a>Consulta del gráfico de gemelos para responder a las preguntas del entorno

Una de las principales características de Azure Digital Twins es la posibilidad de [consultar](concepts-query-language.md) el gráfico de gemelos de forma fácil y eficaz para responder a las preguntas sobre el entorno. 

Ejecute los siguientes comandos en la consola del proyecto en ejecución para responder a algunas preguntas sobre el entorno de ejemplo.

1. **¿Cuáles son las entidades de mi entorno que se representan en Azure Digital Twins?** (consulta de todo)

    ```cmd/sh
    Query
    ```

    Esto le permite evaluar su entorno de un vistazo y asegurarse de que todo se representa tal y como desea en Azure Digital Twins. El resultado es una salida que contiene cada gemelo digital con sus detalles. Aquí se muestra un extracto:

    :::image type="content" source="media/tutorial-command-line/app/output-query-all.png" alt-text="Captura de pantalla que muestra un resultado parcial de la consulta gemela, que incluye room0 y floor1.":::

    >[!NOTE]
    >En el proyecto de ejemplo, el comando `Query` sin argumentos adicionales equivale a `Query SELECT * FROM DIGITALTWINS`. Para consultar todos los gemelos de la instancia mediante [API de consulta](/rest/api/digital-twins/dataplane/query) o [comandos de la CLI](how-to-use-cli.md), use el formato de consulta más largo (completo).

1. **¿Cuáles son todas las salas de mi entorno?** (consulta por modelo)

    ```cmd/sh
    Query SELECT * FROM DIGITALTWINS T WHERE IS_OF_MODEL(T, 'dtmi:example:Room;2')
    ```

    Puede restringir la consulta a los gemelos de un tipo determinado para tener información más específica sobre lo que se representa. El resultado muestra *room0* y *room1*, pero **no** *floor0* o *floor1* (dado que son plantas, no salas).
    
    :::image type="content" source="media/tutorial-command-line/app/output-query-model.png" alt-text="Captura de pantalla del resultado de la consulta del modelo, que muestra solo room0 y room1.":::

1. **¿Cuáles son todas las salas de *floor0*?** (consulta por relación)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0'
    ```

    Puede realizar consultas en función de las relaciones del gráfico para tener información sobre cómo se conectan los gemelos o para restringir la consulta a un área determinada. Solo *room0* está en *floor0*, por lo que es la única sala del resultado.

    :::image type="content" source="media/tutorial-command-line/app/output-query-relationship.png" alt-text="Captura de pantalla del resultado de la consulta de relación, que muestra room0.":::

1. **¿Cuáles son todos los gemelos de mi entorno con una temperatura superior a 75?** (consulta por propiedad)

    ```cmd/sh
    Query SELECT * FROM DigitalTwins T WHERE T.Temperature > 75
    ```

    Puede consultar el gráfico en función de las propiedades para responder a diversas preguntas, como buscar valores atípicos en el entorno que puedan necesitar atención. También se admiten otros operadores de comparación ( *<* , *>* , *=* o *!=* ). *room1* se muestra aquí en los resultados porque tiene una temperatura de 80.

    :::image type="content" source="media/tutorial-command-line/app/output-query-property.png" alt-text="Captura de pantalla del resultado de la consulta de propiedad, que muestra solo room1.":::

1. **¿Cuáles son todas las salas de *floor0* con una temperatura superior a 75?** (consulta compuesta)

    ```cmd/sh
    Query SELECT room FROM DIGITALTWINS floor JOIN room RELATED floor.contains where floor.$dtId = 'floor0' AND IS_OF_MODEL(room, 'dtmi:example:Room;2') AND room.Temperature > 75
    ```

    También puede combinar las consultas anteriores como lo haría en SQL, mediante operadores combinados como `AND`, `OR``NOT`. Esta consulta usa `AND` para que la consulta anterior sobre las temperaturas gemelas sea más específica. El resultado ahora solo incluye las salas con temperaturas superiores a 75 que se encuentran en *floor0*, que en este caso no es ninguna de ellas. El conjunto de resultados está vacío.

    :::image type="content" source="media/tutorial-command-line/app/output-query-compound.png" alt-text="Captura de pantalla del resultado de la consulta compuesta, que no muestra ningún resultado." lightbox="media/tutorial-command-line/app/output-query-compound.png":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Después de completar este tutorial, puede elegir los recursos que desea quitar en función de lo que desee hacer a continuación.

* **Si tiene previsto continuar con el siguiente tutorial**, puede mantener los recursos configurados aquí para seguir usando esta instancia de Azure Digital Twins y la aplicación de ejemplo configurada para el siguiente tutorial.

* **Si desea seguir usando la instancia de Azure Digital Twins, pero borra todos sus modelos, gemelos y relaciones**, puede usar los comandos `DeleteAllTwins` y `DeleteAllModels` de la aplicación de ejemplo para borrar los gemelos y los modelos de la instancia, respectivamente.

[!INCLUDE [digital-twins-cleanup-basic.md](../../includes/digital-twins-cleanup-basic.md)]

También puede que desee eliminar la carpeta del proyecto de la máquina local.

## <a name="next-steps"></a>Pasos siguientes 

En este tutorial ha empezado a usar Azure Digital Twins mediante la creación de un grafo en la instancia con una aplicación cliente de ejemplo. Ha creado modelos, gemelos digitales y relaciones para formar un grafo. También ha ejecutado algunas consultas en el grafo para hacerse una idea de los tipos de preguntas que Azure Digital Twins puede responder sobre un entorno.

Continúe con el siguiente tutorial para combinar Azure Digital Twins con otros servicios de Azure con el fin de completar un escenario integral basado en datos:
> [!div class="nextstepaction"]
> [Tutorial: Conexión de una solución de un extremo a otro](tutorial-end-to-end.md)