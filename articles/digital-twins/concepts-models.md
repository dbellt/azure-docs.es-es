---
title: Modelos de DTDL
titleSuffix: Azure Digital Twins
description: Descubra cómo Azure Digital Twins usa modelos personalizados para describir las entidades del entorno.
author: baanders
ms.author: baanders
ms.date: 3/12/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 713a829ee8c7a3d036bc82f6f509e5c79dfb71aa
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205780"
---
# <a name="understand-twin-models-in-azure-digital-twins"></a>Descripción de los modelos gemelos de Azure Digital Twins

Una característica clave de Azure Digital Twins es la capacidad de definir su propio vocabulario y crear el grafo de gemelos en los términos de la empresa definidos de manera automática. Esta funcionalidad la ofrecen los **modelos** proporcionados por el usuario. Puede considerar los modelos como los nombres de una descripción de su mundo. 

Un modelo es similar a una **clase** en un lenguaje de programación orientado a objetos, el que define una forma de datos para un concepto determinado en el entorno de trabajo real. Los modelos tienen nombres (como *Sala* o *SensorDeTemperatura*) y contienen elementos como propiedades, telemetría/eventos y comandos que describen lo que puede hacer este tipo de entidad en el entorno. Más adelante usará estos modelos para crear [gemelos digitales](concepts-twins-graph.md) que representen entidades específicas que cumplan con esta descripción de tipo.

Los modelos de Azure Digital Twins se representan mediante el **lenguaje de definición de gemelos digitales (DTDL)** , que se basa en JSON-LD.  

## <a name="digital-twin-definition-language-dtdl-for-models"></a>Lenguaje de definición de gemelos digitales (DTDL) para modelos

Los modelos de Azure Digital Twins se definen con el lenguaje de definición de gemelos digitales (DTDL). 

Puede ver las especificaciones del lenguaje completas de DTDL en GitHub: [Lenguaje de definición de gemelos digitales (DTDL), versión 2](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

DTDL se basa en JSON-LD y es independiente del lenguaje de programación. DTDL no es exclusivo de Azure Digital Twins, sino que también se usa para presentar datos de dispositivo en otros servicios de IoT, como [IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md). Azure Digital Twins usa la **versión 2** de DTDL (el uso de DTDL versión 1 con Azure Digital Twins ahora está en desuso). 

En el resto de este artículo se resume el uso del lenguaje en Azure Digital Twins.

### <a name="azure-digital-twins-dtdl-implementation-specifics"></a>Detalles específicos de la implementación de DTDL de Azure Digital Twins

No todos los servicios que usan DTDL implementan las mismas características exactas de DTDL. Por ejemplo, IoT Plug and Play no usa las características de DTDL que son para los grafos, mientras que Azure Digital Twins no implementa actualmente comandos de DTDL. 

Para que un modelo de DTDL sea compatible con Azure Digital Twins, tiene que cumplir estos requisitos:

* Todos los elementos de DTDL de nivel superior de un modelo deben ser de tipo *interfaz*. Esto se debe a que las API del modelo de Azure Digital Twins pueden recibir objetos JSON que representan una interfaz o una matriz de interfaces. Como resultado, no se permite ningún otro tipo de elemento de DTDL en el nivel superior.
* DTDL para Azure Digital Twins no debe definir ningún *comando*.
* Azure Digital Twins solo permite un único nivel de anidamiento de componente. Esto significa que una interfaz que se usa como componente no puede tener ningún componente. 
* Las interfaces no pueden ser definidas insertadas en línea dentro de otras interfaces de DTDL, sino que deben definirse como entidades independientes de nivel superior con sus propios identificadores. A continuación, cuando otra interfaz quiere incluir esa interfaz como componente o a través de la herencia, puede hacer referencia a su identificador.

Azure Digital Twins tampoco observa el atributo `writable` en las propiedades o relaciones. Aunque esto puede establecerse según las especificaciones de DTDL, el valor no lo usa Azure Digital Twins. En su lugar, siempre se tratan como grabables por parte de los clientes externos que tienen permisos de escritura generales en el servicio Azure Digital Twins.

## <a name="elements-of-a-model"></a>Elementos de un modelo

Dentro de una definición de modelo, el elemento de código de nivel superior es una **interfaz**. Encapsula todo el modelo y el resto del modelo se define dentro de la interfaz. 

Una interfaz de modelo de DTDL puede contener cero, uno o varios de los campos siguientes:
* **Propiedad**: las propiedades son campos de datos que representan el estado de una entidad (como las propiedades de muchos lenguajes de programación orientados a objetos). Las propiedades tienen almacenamiento de seguridad y se pueden leer en cualquier momento.
* **Telemetría**: los campos de telemetría representan medidas o eventos y a menudo se usan para describir las lecturas de los sensores del dispositivo. A diferencia de las propiedades, la telemetría no se almacena en un gemelo digital; es una serie de eventos de datos con límites temporales, que deben administrarse a medida que se producen. Para obtener más información sobre las diferencias entre propiedades y telemetría, consulte la sección [Propiedades frente a telemetría](#properties-vs-telemetry) a continuación.
* **Componente**: los componentes permiten compilar la interfaz de modelo como un ensamblado de otras interfaces, si lo desea. Un ejemplo de componente es una interfaz *cámaraFrontal* (y otra interfaz de componente *cámaraPosterior*) que se usa para definir un modelo para un *teléfono*. Primero debe definir una interfaz para *cámaraFrontal* como si fuera su propio modelo y, luego, puede hacer referencia a ella al definir el *Teléfono*.

    Use un componente para describir algo que es una parte integral de la solución, pero no necesita una identidad independiente y no es necesario crearla, eliminarla ni reorganizarla en el grafo de gemelos de forma independiente. Si quiere que las entidades tengan existencias independientes en el grafo de gemelos, represéntelas como gemelos digitales independientes de distintos modelos, conectadas por *relaciones* (consulte la viñeta siguiente).
    
    >[!TIP] 
    >Los componentes también se pueden usar para la organización, a fin de agrupar conjuntos de propiedades relacionadas dentro de una interfaz de modelo. En esta situación, puede considerar cada componente como un espacio de nombres o "carpeta" dentro de la interfaz.
* **Relación**: las relaciones permiten representar cómo un gemelo digital puede estar implicado con otros gemelos digitales. Las relaciones pueden representar distintos significados semánticos, como *contains* ("floor contains room"), *cools* ("hvac cools room"), *isBilledTo* ("compressor is billed to user"), etc. Las relaciones permiten que la solución proporcione un grafo de las entidades interrelacionadas. Las relaciones también pueden tener [propiedades](#properties-of-relationships) propias.

> [!NOTE]
> La [especificación de DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) también define **comandos**, que son métodos que se pueden ejecutar en un gemelo digital (como un comando de restablecimiento o un comando para encender o apagar un ventilador). Sin embargo, *en este momento no se admiten comandos en Azure Digital Twins*.

### <a name="properties-vs-telemetry"></a>Propiedades frente a telemetría

A continuación se ofrecen algunas guías adicionales para distinguir entre los campos de **propiedad** y de **telemetría** de DTDL en Azure Digital Twins.

La diferencia entre las propiedades y la telemetría de los modelos de Azure Digital Twins es la siguiente:
* Se espera que las **propiedades** tengan almacenamiento de seguridad. Esto significa que puede leer una propiedad en cualquier momento y recuperar su valor. Si la propiedad es grabable, también puede almacenar un valor en la propiedad.  
* La **telemetría** es más similar a un flujo de eventos; es un conjunto de mensajes de datos que tienen una duración breve. Si no configura la escucha del evento y las acciones que deben realizarse cuando tiene lugar, no hay ningún seguimiento del evento en un momento posterior. No puede volver y leerlo más tarde. 
  - En términos de C#, la telemetría es como un evento de C#. 
  - En términos de IoT, la telemetría suele ser una medida única que envía un dispositivo.

La **telemetría** se usa a menudo con dispositivos de IoT, ya que muchos dispositivos no son capaces de almacenar los valores de medida que generan, o no les interesa hacerlo. Simplemente las envían como un flujo de eventos de "telemetría". En este caso, no se puede consultar al dispositivo en cualquier momento el valor más reciente del campo de telemetría. En su lugar, deberá escuchar los mensajes desde el dispositivo y tomar medidas a medida que lleguen los mensajes. 

Como consecuencia, al diseñar un modelo en Azure Digital Twins, es probable que use **propiedades** en la mayoría de los casos para modelar los gemelos. Esto le permite tener el almacenamiento de seguridad y la capacidad de leer y consultar los campos de datos.

La telemetría y las propiedades a menudo funcionan en conjunto para controlar la entrada de datos desde dispositivos. Como toda la entrada a Azure Digital Twins se realiza a través de [API](how-to-use-apis-sdks.md), normalmente usará la función de entrada para leer los eventos de telemetría o propiedad de los dispositivos, y establecer una propiedad en Azure Digital Twins en respuesta. 

También puede publicar un evento de telemetría desde la API de Azure Digital Twins. Como con otros tipos de telemetría, es un evento de corta duración que requiere un agente de escucha para el control.

#### <a name="properties-of-relationships"></a>Propiedades de las relaciones

DTDL también permite que las **relaciones** tengan propiedades propias. Al definir una relación dentro de un modelo de DTDL, la relación puede tener su propio campo `properties` donde puede definir propiedades personalizadas para describir el estado específico de la relación.

## <a name="model-inheritance"></a>Herencia de modelo

En algunas ocasiones, puede que quiera especializar aún más un modelo. Por ejemplo, podría resultar útil tener un modelo genérico *Sala* y las variantes especializadas *SalaDeConferencias* y *Gimnasio*. Para expresar la especialización, DTDL admite la herencia: las interfaces pueden heredar de una o varias interfaces. 

En el ejemplo siguiente se recrea el modelo *Planeta* del ejemplo de DTDL anterior como subtipo de un modelo *CuerpoCelestial* más grande. Primero se define el modelo "primario" y, a continuación, el modelo "secundario" se basa en él mediante el campo `extends`.

:::code language="json" source="~/digital-twins-docs-samples/models/CelestialBody-Planet-Crater.json":::

En este ejemplo, *CuerpoCelestial* aporte un nombre, una masa y una temperatura a *Planeta*. La sección `extends` es un nombre de interfaz o una matriz de nombres de interfaz (lo que permite que la interfaz de extensión herede de varios modelos primarios si lo desea).

Una vez que se aplica la herencia, la interfaz de extensión expone todas las propiedades de toda la cadena de herencia.

La interfaz de extensión no puede cambiar ninguna de las definiciones de las interfaces primarias; solo puede agregar a ellas. Tampoco puede volver a definir una funcionalidad que ya esté definida en ninguna de sus interfaces primarias (incluso si las funcionalidades están definidas para ser iguales). Por ejemplo, si una interfaz primaria define una propiedad `double` *masa*, la interfaz de extensión no puede contener una declaración de *masa*, incluso si también es `double`.

## <a name="model-code"></a>Código del modelo

Los modelos de tipo gemelo se pueden escribir en cualquier editor de texto. El lenguaje DTDL sigue la sintaxis JSON, por lo que debe almacenar los modelos con la extensión *.json*. El uso de la extensión JSON permitirá que muchos editores de texto de programación proporcionen comprobación de sintaxis básica y resaltado para los documentos de DTDL. También hay una [extensión de DTDL](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl) disponible para [Visual Studio Code](https://code.visualstudio.com/).

### <a name="possible-schemas"></a>Esquemas posibles

Según DTDL, el esquema de los atributos *Propiedad* y *Telemetría* puede ser de tipos primitivos estándar, `integer`, `double`, `string` y `Boolean`, y otros tipos como `DateTime` y `Duration`. 

Además de los tipos primitivos, los campos *Propiedad* y *Telemetría* pueden tener estos tipos complejos:
* `Object`
* `Map`
* `Enum`

Los campos *Telemetría* también admiten `Array`.

### <a name="example-model"></a>Ejemplo del modelo

Esta sección contiene un ejemplo de un modelo típico, escrito como una interfaz DTDL. El modelo describe los **planetas**, cada uno con un nombre, una masa y una temperatura.
 
Tenga en cuenta que los planetas también pueden interactuar con **lunas**, que son sus satélites, y pueden contener **cráteres**. En el ejemplo siguiente, el modelo `Planet` expresa las conexiones con estas otras entidades haciendo referencia a dos modelos externos: `Moon` y `Crater`. Estos modelos también se definen en el código de ejemplo siguiente, pero se mantienen muy simples para no restar valor al ejemplo principal `Planet`.

:::code language="json" source="~/digital-twins-docs-samples/models/Planet-Crater-Moon.json":::

Los campos del modelo son los siguientes:

| Campo | Descripción |
| --- | --- |
| `@id` | Identificador del modelo. Debe tener el formato `dtmi:<domain>:<unique model identifier>;<model version number>`. |
| `@type` | Identifica el tipo de información que se describe. En el caso de una interfaz, el tipo es *Interfaz*. |
| `@context` | Establece el [contexto](https://niem.github.io/json/reference/json-ld/context/) del documento JSON. Los modelos deben usar `dtmi:dtdl:context;2`. |
| `displayName` | [opcional] Permite asignar un nombre descriptivo al modelo, si lo desea. |
| `contents` | Todos los datos restantes de la interfaz se colocan aquí, como una matriz de definiciones de atributo. Cada atributo debe proporcionar un elemento `@type` (*Propiedad*, *Telemetría*, *Comando*, *Relación* o *Componente*) para identificar el tipo de información de interfaz que describe y, luego, un conjunto de propiedades que definen el atributo real (por ejemplo, `name` y `schema` para definir una *Propiedad*). |

> [!NOTE]
> Tenga en cuenta que la interfaz de componente (em este ejemplo, *Cráter*) se define en la misma matriz que la interfaz que la usa (*Planeta*). Los componentes se deben definir de esta manera en las llamadas API para encontrar la interfaz.

## <a name="best-practices-for-designing-models"></a>Procedimientos recomendados para el diseño de modelos

Al diseñar modelos para que reflejen las entidades de su entorno, puede resultar útil realizar una búsqueda anticipada y considerar las implicaciones de [consulta](concepts-query-language.md) del diseño. Si lo desea, puede diseñar las propiedades de forma que se evite que grandes conjuntos de resultados atraviesen un grafo. También puede modelar relaciones que se responderán en una consulta individual como relaciones de nivel individual.

### <a name="validating-models"></a>Validación de modelos

[!INCLUDE [Azure Digital Twins: validate models info](../../includes/digital-twins-validate.md)]

## <a name="tools-for-models"></a>Herramientas para los modelos 

Hay varios ejemplos disponibles para que sea aún más fácil usar modelos y ontologías. Se encuentran en este repositorio: [Herramientas del Lenguaje de definición de Digital Twins (DTDL)](https://github.com/Azure/opendigitaltwins-tools).

En esta sección se describe con más detalle el conjunto de ejemplos actual.

### <a name="model-uploader"></a>Usuario de carga del modelo 

_**Para cargar modelos en Azure Digital Twins**_

Una vez que haya terminado de crear, extender o seleccionar los modelos, puede cargarlos en la instancia de Azure Digital Twins para que estén disponibles para su uso en la solución. Esto se realiza mediante las [API de Azure Digital Twins](how-to-use-apis-sdks.md), tal y como se describe en [Procedimiento: Administración de modelos DTDL](how-to-manage-model.md#upload-models).

Sin embargo, si tiene muchos modelos para cargar, o si estos tienen muchas interdependencias que harían que la ordenación de las cargas individuales fuera complicada, puede usar este ejemplo para cargar muchos modelos a la vez: [Usuario de carga de modelos de Azure Digital Twins](https://github.com/Azure/opendigitaltwins-building-tools/tree/master/ModelUploader). Siga las instrucciones que se proporcionan con el ejemplo para configurar y usar este proyecto para cargar modelos en su propia instancia.

### <a name="model-visualizer"></a>Visualizador de modelos 

_**Para visualizar modelos**_

Una vez que cargue los modelos en la instancia de Azure Digital Twins, podrá verlos en dicha instancia, incluidas todas las relaciones de herencia y modelo, mediante el [visualizador de modelos de Azure Digital Twins](https://github.com/Azure/opendigitaltwins-building-tools/tree/master/AdtModelVisualizer). Este ejemplo se encuentra actualmente en un estado de borrador. Se recomienda que la comunidad de desarrollo de gemelos digitales extienda y contribuya con este ejemplo. 

## <a name="next-steps"></a>Pasos siguientes

* Obtenga información sobre cómo crear modelos basados en ontologías estándar del sector: [Conceptos: ¿Qué es una ontología?](concepts-ontologies.md) 

* Analice en profundidad la administración de modelos con operaciones de API: [Procedimiento: Administración de modelos DTDL](how-to-manage-model.md)

* Obtenga información sobre cómo se usan los modelos para crear gemelos digitales: [Conceptos: Gemelos digitales y grafo de gemelos](concepts-twins-graph.md)

