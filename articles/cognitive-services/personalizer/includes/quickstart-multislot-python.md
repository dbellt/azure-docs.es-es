---
title: archivo de inclusión
description: Archivo de inclusión
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: personalizer
ms.topic: include
ms.custom: cog-serv-seo-aug-2020
ms.date: 03/23/2021
ms.openlocfilehash: e772182cfd1ba656c730f423a7b4b8f0a5d709a7
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382333"
---
[Personalización conceptual de varias ranuras](..\concept-multi-slot-personalization.md) | [Ejemplos](https://aka.ms/personalizer/ms-python)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* [Python 3.x](https://www.python.org/)
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer"  title="Creación de un recurso de Personalizer"  target="_blank">cree un recurso de Personalizer</a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a API Personalizer. Más adelante en este inicio rápido, debe pegar la clave y el punto de conexión en el código que se incluye a continuación.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

[!INCLUDE [Upgrade Personalizer instance to Multi-Slot](upgrade-personalizer-multi-slot.md)]

[!INCLUDE [Change model frequency](change-model-frequency.md)]

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Cree un archivo de Python y variables para el punto de conexión y la clave de suscripción del recurso.

[!INCLUDE [Personalizer find resource info](find-azure-resource-info.md)]

```python
import datetime, json, os, time, uuid, requests

# The endpoint specific to your personalization service instance; 
# e.g. https://<your-resource-name>.cognitiveservices.azure.com
PERSONALIZATION_BASE_URL = "https://<REPLACE-WITH-YOUR-PERSONALIZER-ENDPOINT>.cognitiveservices.azure.com"
# The key specific to your personalization service instance; e.g. "0123456789abcdef0123456789ABCDEF"
RESOURCE_KEY = "<REPLACE-WITH-YOUR-PERSONALIZER-KEY>"
```

## <a name="object-model"></a>Modelo de objetos

Con el fin de solicitar el mejor elemento de contenido para cada ranura, cree una solicitud [rank_request] y, a continuación, envíe una solicitud POST al punto de conexión [multislot/rank] ( https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api-v1-1-preview-1/operations/Rank). A continuación, la respuesta se analiza en otra respuesta [rank_response].

Para enviar una puntuación de recompensa a Personalizer, cree una llamada [reward] y, a continuación, envíe una solicitud POST a [multislot/events/{eventId}/reward](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api-v1-1-preview-1/operations/Events_Reward).

La determinación de la puntuación de recompensa en este inicio rápido no es importante. En un sistema de producción, la determinación de lo que afecta a la [puntuación de recompensa](../concept-rewards.md) y cuánto le afecta, puede ser un proceso complejo que decida cambiar con el tiempo. Esta debe ser una de las decisiones de diseño principales en la arquitectura de Personalizer.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas enviando solicitudes HTTP para Python:

* [Creación de direcciones URL base](#create-base-urls)
* [API Rank](#request-the-best-action)
* [API Reward](#send-a-reward)

## <a name="create-base-urls"></a>Creación de direcciones URL base

En esta sección, hará dos cosas:
* Crear las direcciones URL Rank y Reward.
* Crear los encabezados de las solicitudes Rank o Reward.

Cree las direcciones URL Rank o Reward usando la dirección URL base y los encabezados de solicitud usando la clave de recurso.

```python
MULTI_SLOT_RANK_URL = '{0}personalizer/v1.1-preview.1/multislot/rank'.format(PERSONALIZATION_BASE_URL)
MULTI_SLOT_REWARD_URL_BASE = '{0}personalizer/v1.1-preview.1/multislot/events/'.format(PERSONALIZATION_BASE_URL)
HEADERS = {
    'ocp-apim-subscription-key.': RESOURCE_KEY,
    'Content-Type': 'application/json'
}
```

## <a name="get-content-choices-represented-as-actions"></a>Obtención de opciones de contenido representadas como acciones

Las acciones representan las opciones de contenido entre las que quiere que Personalizer seleccione el mejor elemento de contenido. Agregue los siguientes métodos al script para representar el conjunto de acciones y sus características. 

```python
def get_actions():
    return [
        {
            "id": "Red-Polo-Shirt-432",
            "features": [
                {
                    "onSale": "true",
                    "price": 20,
                    "category": "Clothing"
                }
            ]
        },
        {
            "id": "Tennis-Racket-133",
            "features": [
                {
                    "onSale": "false",
                    "price": 70,
                    "category": "Sports"
                }
            ]
        },
        {
            "id": "31-Inch-Monitor-771",
            "features": [
                {
                    "onSale": "true",
                    "price": 200,
                    "category": "Electronics"
                }
            ]
        },
        {
            "id": "XBox-Series X-117",
            "features": [
                {
                    "onSale": "false",
                    "price": 499,
                    "category": "Electronics"
                }
            ]
        }
    ]

```

## <a name="get-user-preferences-for-context"></a>Obtención de las preferencias del usuario para el contexto

Agregue los siguientes métodos al script para obtener la entrada de un usuario desde la línea de comandos con relación a la hora del día y el tipo de dispositivo en el que se encuentra el usuario. Se usarán como características de contexto.

```python
def get_context_features():
    time_features = ["morning", "afternoon", "evening", "night"]
    time_pref = input("What time of day is it (enter number)? 1. morning 2. afternoon 3. evening 4. night\n")
    try:
        parsed_time = int(time_pref)
        if(parsed_time <=0 or parsed_time > len(time_features)):
            raise IndexError
        time_of_day = time_features[parsed_time-1]
    except (ValueError, IndexError):
        print("Entered value is invalid. Setting feature value to", time_features[0] + ".")
        time_of_day = time_features[0]

    device_features = ['mobile', 'tablet', 'desktop']
    device_pref = input("What type of device is the user on (enter number)? 1. mobile 2. tablet 3. desktop\n")
    try:
        parsed_device = int(device_pref)
        if(parsed_device <=0 or parsed_device > len(device_features)):
            raise IndexError
        device = device_features[parsed_device-1]
    except (ValueError, IndexError):
        print("Entered value is invalid. Setting feature value to", device_features[0]+ ".")
        device = device_features[0]

    return [
        {'time': time_of_day},
        {'device': device}
        ]
```

## <a name="get-slots"></a>Obtención de ranuras

Las ranuras componen la página con la que va a interactuar el usuario. Personalizer decidirá qué acción mostrar en cada una de las ranuras definidas. Es posible excluir acciones de ranuras específicas, que se muestran como `ExcludeActions`. `BaselineAction` es la acción predeterminada para la ranura que se habría mostrado si no se hubiera usado Personalizer.

Esta guía de inicio rápido tiene características de ranura sencillas. En sistemas de producción, determinar y [evaluar](../concept-feature-evaluation.md) [características](../concepts-features.md) puede ser una tarea compleja.

```python
def get_slots():
    return [
        {
            "id": "BigHeroPosition",
            "features": [
                {
                    "size": "large",
                    "position": "left",
                }
            ],
            "excludedActions": ["31-Inch-Monitor-771"],
            "baselineAction": "Red-Polo-Shirt-432"
        },
        {
            "id": "SmallSidebar",
            "features": [
                {
                    "size": "small",
                    "position": "right",
                }
            ],
            "excludedActions": ["Tennis-Racket-133"],
            "baselineAction": "XBox-Series X-117"
        }
    ]
```

## <a name="make-http-requests"></a>Realización de solicitudes HTTP

Envíe solicitudes POST al punto de conexión de Personalizer para realizar llamadas Rank y Reward de varias ranuras.

```python
def send_multi_slot_rank(rank_request):
    multi_slot_response = requests.post(MULTI_SLOT_RANK_URL, data=json.dumps(rank_request), headers=HEADERS )
    return json.loads(multi_slot_response.text)
```

```python
def send_multi_slot_reward(reward_request, event_id):
    reward_url = '{0}{1}/reward'.format(MULTI_SLOT_REWARD_URL_BASE, event_id)
    requests.post(reward_url, data=json.dumps(reward_request), headers=HEADERS)
```

## <a name="get-feedback-for-personalizer-decisions"></a>Obtención de comentarios sobre decisiones de Personalizer

Agregue el siguiente método al script. Indicará si Personalizer ha tomado una decisión correcta con relación a cada ranura mediante un símbolo de la línea de comandos.

```python
def get_reward_for_slot():
    answer = input('\nIs this correct? (y/n)\n').upper()
    if (answer == 'Y'):
        print('\nGreat! The application will send Personalizer a reward of 1 so it learns from this choice of action for this slot.\n')
        return 1
    elif (answer == 'N'):
        print('\nYou didn\'t like the recommended item.The application will send Personalizer a reward of 0 for this choice of action for this slot.\n')
        return 0
    print('\nEntered choice is invalid. Service assumes that you didn\'t like the recommended item.\n')
    return 0
```

## <a name="create-the-learning-loop"></a>Creación del bucle de aprendizaje

El bucle de aprendizaje de Personalizer es un ciclo de llamadas [Rank](#request-the-best-action) y [Reward](#send-a-reward). En este inicio rápido, cada llamada Rank para personalizar el contenido, va seguida de una llamada Reward para indicar a Personalizer cómo es de eficaz el funcionamiento del servicio.

El siguiente código pasa por un ciclo en el que se pregunta al usuario sus preferencias en la línea de comandos, se envía esa información a Personalizer para que seleccione la mejor acción para cada ranura, se presenta la selección al cliente para que elija entre las opciones de la lista y, después, se envía una puntuación de recompensa a Personalizer en la que se señala el grado de acierto que ha tenido el servicio en su selección.

```python
run_loop = True

while run_loop:

    eventId = str(uuid.uuid4())
    context = get_context_features()
    actions = get_actions()
    slots = get_slots()

    rank_request = {
        "eventId": eventId,
        "contextFeatures": context,
        "actions": actions,
        "slots": slots,
        "deferActivation": False
      }

    #Rank the actions for each slot
    multi_slot_rank_response = send_multi_slot_rank(rank_request)
    multi_slot_rewards = {"reward": []}

    for i in range(len(multi_slot_rank_response['slots'])):
        print('\nPersonalizer service decided you should display: {0} in slot {1}\n'.format(multi_slot_rank_response['slots'][i]['rewardActionId'], multi_slot_rank_response['slots'][i]['id']))

        slot_reward = {'slotId': multi_slot_rank_response['slots'][i]['id']}
        # User agrees or disagrees with Personalizer decision for slot
        slot_reward['value'] = get_reward_for_slot()
        multi_slot_rewards['reward'].append(slot_reward)

    # Send the rewards for the event
    send_multi_slot_reward(multi_slot_rewards, multi_slot_rank_response['eventId'])

    answer = input('\nPress q to break, any other key to continue:\n').upper()
    if (answer == 'Q'):
        run_loop = False
```

Observe detalladamente las llamadas de clasificación y recompensa en las secciones siguientes.

Agregue los siguientes métodos, que [obtienen las opciones de contenido seleccionadas](#get-content-choices-represented-as-actions), [obtienen las preferencias del usuario con respecto al contexto](#get-user-preferences-for-context), [obtienen las ranuras](#get-slots), [realizan solicitudes HTTP](#make-http-requests) y [obtienen la recompensa de cada ranura](#get-feedback-for-personalizer-decisions) antes de ejecutar el archivo de código:

* get_actions
* get_context_features
* get_slots
* send_rank
* send_reward
* get_reward_for_dsot

## <a name="request-the-best-action"></a>Solicitud de la mejor acción

Para completar la solicitud Rank, el programa solicita las preferencias del usuario para crear opciones de contenido. El cuerpo de la solicitud contiene las características de contexto, las acciones y sus características, las ranuras y sus características, y un id. de evento único, con el fin de recibir la respuesta. El método `send_multi_slot_rank` necesita el objeto rank_request para enviar la solicitud Rank de varias ranuras.

Esta guía de inicio rápido tiene características de contexto sencillas de hora del día y dispositivo del usuario. En los sistemas de producción, la determinación y [evaluación](../concept-feature-evaluation.md) de [acciones y características](../concepts-features.md) no es una cuestión trivial.

```python
eventId = str(uuid.uuid4())
context = get_context_features()
actions = get_actions()
slots = get_slots()

rank_request = {
    "eventId": eventId,
    "contextFeatures": context,
    "actions": actions,
    "slots": slots,
    "deferActivation": False
    }

#Rank the actions for each slot
multi_slot_rank_response = send_multi_slot_rank(rank_request)
```

## <a name="send-a-reward"></a>Envío de una recompensa

Para obtener la puntuación de recompensa que se enviará en la solicitud Reward, el programa obtiene la selección del usuario mediante la línea de comandos, asigna un valor numérico a la selección y, después, envía el id. de evento único y la puntuación de recompensa para cada ranura como valor numérico al método `send_multi_slot_reward`. No es necesario definir una recompensa para cada ranura.

En este inicio rápido se asigna un número simple como puntuación de recompensa: 0 o 1. En sistemas de producción, determinar cuándo y qué enviar a la llamada [Reward](../concept-rewards.md) puede ser más complicado, en función de las necesidades específicas.

```python
multi_slot_rewards = {"reward": []}

for i in range(len(multi_slot_rank_response['slots'])):
    print('\nPersonalizer service decided you should display: {0} in slot {1}\n'.format(multi_slot_rank_response['slots'][i]['rewardActionId'], multi_slot_rank_response['slots'][i]['id']))

    slot_reward = {'slotId': multi_slot_rank_response['slots'][i]['id']}
    # User agrees or disagrees with Personalizer decision for slot
    slot_reward['value'] = get_reward_for_slot()
    multi_slot_rewards['reward'].append(slot_reward)

# Send the rewards for the event
send_multi_slot_reward(multi_slot_rewards, multi_slot_rank_response['eventId'])
```

## <a name="run-the-program"></a>Ejecución del programa

Ejecute la aplicación con el comando de Python desde el directorio de aplicación.

```console
python sample.py
```

![El programa del inicio rápido realiza dos preguntas para recopilar las preferencias del usuario, conocidas como características, y, después, proporciona la acción principal.](../media/csharp-quickstart-commandline-feedback-loop/multislot-quickstart-program-feedback-loop-example-1.png)


El [código fuente de este inicio rápido](https://aka.ms/personalizer/ms-python) está disponible.
