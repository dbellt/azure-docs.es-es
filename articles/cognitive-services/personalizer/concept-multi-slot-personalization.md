---
title: Personalización de varias ranuras
description: Obtenga información sobre dónde y cuándo usar la personalización de una sola ranura y de varias ranuras con las API Rank y Reward de Personalizer.
services: cognitive-services
ms.author: edjez
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: quickstart
ms.date: 05/24/2021
ms.openlocfilehash: 042bce09afeabb10b20d8dfcb575040ccf5e1c03
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382384"
---
# <a name="multi-slot-personalization-preview"></a>Personalización de varias ranuras (versión preliminar)

Con Personalización de varias ranuras (versión preliminar) puede mostrar contenido en diseños web, carruseles y listas donde se muestre más de una acción (por ejemplo, un producto o un fragmento de contenido) a los usuarios. Con las API de varias ranuras de Personalizer puede conseguir que los modelos de IA de Personalizer aprendan qué contextos y productos de usuario desencadenan determinados comportamientos; para ello, tienen en cuenta la selección de ubicación en la interfaz de usuario y aprenden de ella. Por ejemplo, Personalizer puede aprender que determinados productos o contenido desencadenan más clics como barra lateral o pie de página que como resaltado principal en una página.

En este artículo, obtendrá información sobre por qué la personalización de varias ranuras mejora los resultados, cómo habilitarla y cuándo usarla. En este artículo se da por supuesto que conoce API de Personalizer como `Rank` y `Reward` y que ha adquirido los conceptos necesarios para usarlas en la aplicación. Si no conoce Personalizer ni sabe cómo funciona, consulte lo siguiente antes de continuar:

* [¿Qué es Personalizer?](what-is-personalizer.md) 
* [Funcionamiento de Personalizer](how-personalizer-works.md) 

> [!IMPORTANT] 
>  La personalización de varias ranuras está en versión preliminar pública. Las características, los enfoques y los procesos cambiarán en función de los comentarios de los usuarios. Al habilitar la versión preliminar de varias ranuras, se deshabilitará de forma permanente otra funcionalidad de Personalizer de su bucle. La personalización de varias ranuras no se puede desactivar una vez habilitada para un bucle de Personalizer. Lea este documento y tenga en cuenta las consecuencias antes de configurar un bucle de Personalizer para la personalización de varias ranuras.

<!-- 
We encourage you to provide feedback on multi-slot personalization APIs via ...
-->

## <a name="when-to-use-multi-slot-personalization"></a>Cuándo usar la personalización de varias ranuras

Siempre que muestre productos o contenido a usuarios, es posible que desee mostrar más de un elemento a sus clientes. Por ejemplo:

* **Diseños de sitios web para páginas principales:** muchos iconos y áreas de página están destinados a resaltar contenido en cuadros, banners y barras laterales de diferentes formas y tamaños. La personalización de varias ranuras aprenderá cómo afectan las características de este diseño a las opciones elegidas por los clientes y a sus acciones.
* **Carruseles:** los carruseles de contenido que cambia dinámicamente necesitan varios elementos para el ciclo. La personalización de varias ranuras puede aprender cómo afectan la secuencia e incluso la duración de la visualización a los clics y a la interacción.
* **Productos o contenido relacionados y referencias insertadas:** es habitual captar la atención de los usuarios insertando o intercalando referencias a contenido y productos adicionales en banners, barras laterales, viñetas y cuadros a pie de página. La personalización de varias ranuras puede ayudarle a asignar sus referencias allí donde es más probable que generen un mayor uso.
* **Resultados o listas de búsqueda:** si su aplicación tiene una funcionalidad de búsqueda, donde proporciona resultados como listas o iconos, puede usar la personalización de varias ranuras para elegir qué elementos resaltar en la parte superior teniendo en cuenta más metadatos que clasificadores tradicionales.
* **Canales dinámicos y listas de reproducción:** la personalización de varias ranuras puede contribuir a determinar una secuencia corta para una lista de vídeos o canciones que se reproducirán a continuación en un canal dinámico.

Con la personalización de varias ranuras puede declarar las "ranuras" de la interfaz de usuario para las que deben elegirse acciones. También puede proporcionar más información sobre las ranuras, que Personalizer puede usar para mejorar la ubicación del producto; por ejemplo, si un cuadro es demasiado grande o pequeño, si se debe mostrar una descripción o solo una característica, o bien si debe encontrarse en un pie de página o en una barra lateral.


## <a name="how-to-use-multi-slot-personalization"></a>Cómo usar la personalización de varias ranuras 

1. [Habilitación de la personalización de varias ranuras](#enable-multi-slot-personalization)
1. [Creación de un objeto JSON para una solicitud Rank](#create-json-object-for-a-rank-request)
1. [Llamada a la API Rank que define ranuras y acciones de línea base](#use-the-response-of-the-rank-api)
1. [Llamada a API Reward](#call-the-reward-api)


### <a name="enable-multi-slot-personalization"></a>Habilitación de la personalización de varias ranuras 

Consulte a continuación [Diferencias entre la personalización de una sola ranura y de varias ranuras](#differences-between-single-slot-and-multi-slot-personalization) para saber y decidir si la personalización de varias ranuras resulta útil en su caso. *La personalización de varias ranuras es una característica en versión preliminar:* le recomendamos crear un bucle de Personalizer si desea probar API de personalización de varias ranuras, ya que, una vez habilitada, la acción es reversible y tendrá efectos en un bucle de Personalizer que se esté ejecutando en producción.

Cuando haya decidido convertir un bucle en personalización de varias ranuras, siga estos pasos una vez para este bucle de Personalizer:

[!INCLUDE [Upgrade Personalizer instance to Multi-Slot](./includes/upgrade-personalizer-multi-slot.md)]

### <a name="create-json-object-for-a-rank-request"></a>Creación de un objeto JSON para una solicitud Rank
Para usar la personalización de varias ranuras, hace falta una API que difiere ligeramente de la personalización de una sola ranura.

Las ranuras a las que es posible asignar acciones se declaran en cada solicitud de llamada Rank, en el objeto "slots":

* **Matriz de ranuras:** debe declarar una matriz de ranuras. Las ranuras *se ordenan:* la posición de cada ranura en la matriz es importante. Se recomienda ordenar las definiciones de ranura en función del número de recompensas, clics o conversiones que cada ranura suela obtener, empezando por la que más obtenga. Por ejemplo, coloque como ranura 1 un gran cuadro prominente de página principal para un sitio web, en lugar de un pie de página pequeño.  Dado que todo lo demás es igual, Personalizer asignará las acciones con más posibilidades de obtener recompensas al inicio de la secuencia.
* **Id. de ranura:** debe proporcionar un valor "slotId" a cada ranura, una cadena única para todas las demás ranuras de esta llamada Rank.
* **Características de ranura:** debe proporcionar más metadatos que la describan y la distingan aún más de otras ranuras. Es lo que se denomina *características*. Al determinar las características de ranura, debe seguir las mismas directrices recomendadas para las características de contexto y acciones. (Consulte [Características de contexto y acciones]()).  Las características típicas de ranura contribuyen a identificar el tamaño, la posición o las características visuales de un elemento de la interfaz de usuario. Por ejemplo, `position: "top-left"`, `size: "big"`, `animated: "no"`, `sidebar: "true"` o `sequence: "1"`.
* **Acciones de línea de base:** debe especificar el id. de acción de línea de base para cada ranura, es decir, el id. de la acción que se mostraría en la ranura si Personalizer no existiera. Esta acción es necesaria para entrenar a Personalizer en el [modo de aprendiz]() y para tener un número significativo al realizar [evaluaciones sin conexión]().
* **Tener suficientes acciones:** asegúrese de llamar a Rank con más acciones que ranuras, de modo que Personalizer pueda asignar al menos una acción a cada ranura. Personalizer no repetirá las recomendaciones de acción entre ranuras: la respuesta Rank asignará cada acción como máximo a una ranura. 

No pasa nada si agrega o quita ranuras con el tiempo, agrega y cambia sus características, o bien vuelve a ordenar la matriz: Personalizer se adaptará y seguirá entrenándose en función de la nueva información.

A continuación figura un `slots` de muestra con características de ejemplo. Aunque la mayor parte del objeto `slots` es estable (ya que las IU suelen cambiar lentamente), la mayoría no cambia a menudo. No obstante, asegúrese de asignar los ids. de acción de línea de base adecuados a cada llamada Rank. 


```json
"slots": [ 
    { 
      "id": "BigHighlight", 
      "features": [ 
            { 
              "size": "Large", 
              "position": "Left-Middle" 
            }
        ],
        "baselineAction": "BlackBoot_4656" 
    }, 

    { 
      "id": "Sidebar1", 
      "features": [ 
            { 
              "size": "Small", 
              "position": "Right-Top" 
            } 
        ],
        "baselineAction": "TrekkingShoe_1122"  
    }  
  ]

```


### <a name="use-the-response-of-the-rank-api"></a>Uso de la respuesta de la API Rank

Una respuesta Rank de varias ranuras a partir de la solicitud anterior puede tener un aspecto parecido al siguiente: 

```json
{ 
  "slots": [ 
        { 
          "id": "BigHighlight", 
          "rewardActionId": "WhiteSneaker_8181" 
        }, 
        { 
          "id": "SideBar1", 
          "rewardActionId": "BlackBoot_4656" 
        } 
    ], 
  "eventId": "123456D0-BFEE-4598-8196-C57383D38E10" 
} 
```

Tome el valor "rewardActionId" de cada ranura y úselo para representar la interfaz de usuario como corresponda.

### <a name="call-the-reward-api"></a>Llamada a la API Reward

Personalizer aprende a elegir acciones que maximicen la recompensa obtenida. Su aplicación observará el comportamiento del usuario y calculará una "puntuación de recompensa" para Personalizer en función de la reacción observada. Por ejemplo, si el usuario ha hecho clic en la acción en el objeto `"slotId": "SideBar1",`, enviará un "1" a Personalizer a modo de refuerzo positivo de las opciones de la acción.

La API Reward especifica el valor de "eventId" correspondiente a la recompensa en la dirección URL:

```http
https://{endpoint}/personalizer/v1.0/events/{eventId}/reward
``` 

Por ejemplo, la recompensa del evento anterior con id. 123456D0-BFEE-4598-8196-C57383D38E10/reward se enviará a `https://{endpoint}/personalizer/v1.0/events/123456D0-BFEE-4598-8196-C57383D38E10/reward/reward`:

```json
{ 
  "reward": [ 
    { 
      "slotId": "BigHighlight", 
      "value": 0.2 
    }, 
    { 
      "slotId": "SideBar1", 
     "value": 1.0 
    }, 
  ] 
} 
```

No es necesario que proporcione todas las puntuaciones de recompensa en una sola llamada de la API Reward. Puede llamar a la API Reward varias veces, cada una con los valores correspondientes de "eventId" y "slotIds". Si no se recibe ninguna puntuación de recompensa con relación a una ranura en un evento, Personalizer asignará implícitamente la recompensa predeterminada configurada para el bucle (normalmente 0).


## <a name="differences-between-single-slot-and-multi-slot-personalization"></a>Diferencias entre la personalización de una sola ranura y de varias ranuras

Hay diferencias en el modo de usar las API Rank y Reward con personalización de una sola ranura y de varias:

|  Descripción  | Personalización de una sola ranura   | Personalización de varias ranuras    |  
|---------------|-------------------------------|-------------------------------|
| Elementos de la solicitud de llamada a la API Rank | Se envían un objeto Context y una lista de acciones.  | Se envían un objeto Context, una lista de acciones y una lista ordenada de ranuras.     | 
| Solicitud de Rank que especifica la línea de base | Personalizer tomará la primera acción de la lista como la acción de línea de base (el elemento que su aplicación habría elegido si Personalizer no existiera).|Usted debe especificar el id. de acción de línea de base que se habría usado en cada ranura.|
| Respuesta de la llamada a la API Rank | Su aplicación resalta la acción indicada en el campo "rewardActionId".     | La respuesta incluye un valor de "rewardActionId" diferente para cada ranura que se haya especificado en la solicitud. La aplicación mostrará esas acciones "rewardActionId" en cada ranura. | 
| Llamada a la API Reward | Se llama a la API Reward con una puntuación de recompensa, que se calcula a partir del modo en que los usuarios han interactuado con la acción "rewardActionId" para este valor de "eventId" en particular. Por ejemplo, si el usuario ha hecho clic en ella, se enviará una recompensa de 1.     | Especifique la recompensa para cada ranura, dado el grado en que la acción con "rewardActionId" haya provocado el comportamiento deseado del usuario. Esta información se puede enviar en una o varias llamadas a la API Reward con el mismo valor de "eventId".   | 


### <a name="impact-of-enabling-multi-slot-for-a-personalizer-loop"></a>Consecuencias de habilitar la personalización de varias ranuras para un bucle de Personalizer

Asimismo, al habilitar la personalización de varias ranuras, tenga en cuenta lo siguiente:

|  Descripción  | Personalización de una sola ranura   | Personalización de varias ranuras    | 
|---------------|-------------------------------|-------------------------------|
| Eventos inactivos y activación    | Al llamar a la API de activación, Personalizer activará el evento, tras lo cual esperará una puntuación de recompensa o asignará la recompensa predeterminada configurada si se supera el tiempo de espera de la recompensa. | Personalizer activa y espera recompensas para todas las ranuras que se hayan especificado en el valor "eventId". |     
| Modo de aprendiz | La API Rank de Personalizer siempre devuelve la acción de línea de base y entrena modelos internos imitando dicha acción.| La API Rank de Personalizer devuelve la acción de línea de base para cada ranura especificada en el campo "baselineAction". Personalizer entrenará modelos internos imitando dicha acción. |
| Velocidad de aprendizaje | Solo aprende de la acción resaltada. | Puede aprender de las interacciones con cualquier ranura. Esto suele implicar más comportamientos de usuario que pueden producir recompensas, lo que agilizaría el aprendizaje de Personalizer. |  
| Evaluaciones sin conexión |Compara el rendimiento de Personalizer con las configuraciones de aprendizaje de línea de base y optimizada, en función de la acción que las habría elegido.| (Limitación de la versión preliminar) Solo evalúa el rendimiento de la primera ranura de la matriz. En el caso de evaluaciones más precisas, se recomienda asegurarse de que la ranura con más recompensas sea la primera de la matriz. |
| Optimización automática (versión preliminar) | Su bucle de Personalizer puede realizar periódicamente evaluaciones sin conexión en segundo plano y optimizar la configuración de aprendizaje sin intervención administrativa. | (Limitación de la versión preliminar) La optimización automática está deshabilitada para los bucles de Personalizer que tienen habilitadas API de varias ranuras. |

## <a name="next-steps"></a>Pasos siguientes
* [Cómo usar la personalización de varias ranuras](how-to-multi-slot.md)
* [Ejemplo: ejecución de un bucle de personalización de varias ranuras en Jupyter Notebook](https://github.com/Azure-Samples/cognitive-services-quickstart-code/tree/master/python/Personalizer/azure-notebook)