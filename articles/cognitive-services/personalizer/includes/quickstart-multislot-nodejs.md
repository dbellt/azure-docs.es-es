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
ms.openlocfilehash: 551b6901b60e55e9496cf5c4483aa3a05dfd72a7
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110382338"
---
[Documentación de referencia](/javascript/api/@azure/cognitiveservices-personalizer/) | [Personalización conceptual de varias ranuras](..\concept-multi-slot-personalization.md) | [Ejemplos](https://aka.ms/personalizer/ms-nodejs)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services)
* La versión actual de [Node.js](https://nodejs.org) y NPM.
* Cuando tenga la suscripción de Azure, <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesPersonalizer"  title="Creación de un recurso de Personalizer"  target="_blank">cree un recurso de Personalizer</a> en Azure Portal para obtener la clave y el punto de conexión. Una vez que se implemente, haga clic en **Ir al recurso**.
    * Necesitará la clave y el punto de conexión del recurso que cree para conectar la aplicación a API Personalizer. Más adelante en este inicio rápido, debe pegar la clave y el punto de conexión en el código que se incluye a continuación.
    * Puede usar el plan de tarifa gratis (`F0`) para probar el servicio y actualizarlo más adelante a un plan de pago para producción.

## <a name="setting-up"></a>Instalación

[!INCLUDE [Upgrade Personalizer instance to Multi-Slot](upgrade-personalizer-multi-slot.md)]

[!INCLUDE [Change model frequency](change-model-frequency.md)]

### <a name="create-a-new-nodejs-application"></a>Creación de una aplicación Node.js

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él.

```console
mkdir myapp && cd myapp
```

Ejecute el comando `npm init -y` para crear un archivo `package.json`.

```console
npm init -y
```

Cree una aplicación de Node.js en el editor o IDE preferidos denominada `sample.js` y cree variables para el punto de conexión y la clave de suscripción del recurso. 

[!INCLUDE [Personalizer find resource info](find-azure-resource-info.md)]

```javascript
const axios = require('axios');
const { v4: uuidv4 } = require('uuid');
const readline = require('readline-sync');
// The endpoint specific to your personalization service instance; 
// e.g. https://<your-resource-name>.cognitiveservices.azure.com
const PersonalizationBaseUrl = 'https://<REPLACE-WITH-YOUR-PERSONALIZER-ENDPOINT>.cognitiveservices.azure.com';
// The key specific to your personalization service instance; e.g. "0123456789abcdef0123456789ABCDEF"
const ResourceKey = '<REPLACE-WITH-YOUR-PERSONALIZER-KEY>';
```

### <a name="install-the-npm-packages-for-quickstart"></a>Instalación de los paquetes NPM para realizar un inicio rápido

```console
npm install @azure/ms-rest-azure-js @azure/ms-rest-js readline-sync uuid axios --save
```

## <a name="object-model"></a>Modelo de objetos

Con el fin de solicitar el mejor elemento de contenido para cada ranura, cree una solicitud [rankRequest] y, a continuación, envíe una solicitud POST al punto de conexión [multislot/rank] ( https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api-v1-1-preview-1/operations/Rank). A continuación, la respuesta se analiza en otra respuesta [rankResponse].

Para enviar una puntuación de recompensa a Personalizer, cree una llamada [reward] y, a continuación, envíe una solicitud POST a [multislot/events/{eventId}/reward](https://westus2.dev.cognitive.microsoft.com/docs/services/personalizer-api-v1-1-preview-1/operations/Events_Reward).

La determinación de la puntuación de recompensa en este inicio rápido no es importante. En un sistema de producción, la determinación de lo que afecta a la [puntuación de recompensa](../concept-rewards.md) y cuánto le afecta, puede ser un proceso complejo que decida cambiar con el tiempo. Esta debe ser una de las decisiones de diseño principales en la arquitectura de Personalizer.

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas enviando solicitudes HTTP para NodeJS:

* [Creación de direcciones URL base](#create-base-urls)
* [API Rank](#request-the-best-action)
* [API Reward](#send-a-reward)


## <a name="create-base-urls"></a>Creación de direcciones URL base

En esta sección, hará dos cosas:
* Crear las direcciones URL Rank y Reward.
* Crear los encabezados de las solicitudes Rank o Reward.

Cree las direcciones URL Rank o Reward usando la dirección URL base y los encabezados de solicitud usando la clave de recurso.

```javascript
const MultiSlotRankUrl = PersonalizationBaseUrl.concat('personalizer/v1.1-preview.1/multislot/rank');
const MultiSlotRewardUrlBase = PersonalizationBaseUrl.concat('personalizer/v1.1-preview.1/multislot/events/');
const Headers = {
    'ocp-apim-subscription-key': ResourceKey,
    'Content-Type': 'application/json'
};
```

## <a name="get-content-choices-represented-as-actions"></a>Obtención de opciones de contenido representadas como acciones

Las acciones representan las opciones de contenido entre las que quiere que Personalizer seleccione el mejor elemento de contenido. Agregue los siguientes métodos al script para representar el conjunto de acciones y sus características. 

```javascript
function getActions() {
    return [
        {
            'id': 'Red-Polo-Shirt-432',
            'features': [
                {
                    'onSale': 'true',
                    'price': 20,
                    'category': 'Clothing'
                }
            ]
        },
        {
            'id': 'Tennis-Racket-133',
            'features': [
                {
                    'onSale': 'false',
                    'price': 70,
                    'category': 'Sports'
                }
            ]
        },
        {
            'id': '31-Inch-Monitor-771',
            'features': [
                {
                    'onSale': 'true',
                    'price': 200,
                    'category': 'Electronics'
                }
            ]
        },
        {
            'id': 'XBox-Series X-117',
            'features': [
                {
                    'onSale': 'false',
                    'price': 499,
                    'category': 'Electronics'
                }
            ]
        }
    ];
}
```

## <a name="get-user-preferences-for-context"></a>Obtención de las preferencias del usuario para el contexto

Agregue los siguientes métodos al script para obtener la entrada de un usuario desde la línea de comandos con relación a la hora del día y el tipo de dispositivo en el que se encuentra el usuario. Se usarán como características de contexto.

```javascript
function getContextFeatures() {
    const timeOfDayFeatures = ['morning', 'afternoon', 'evening', 'night'];
    const deviceFeatures = ['mobile', 'tablet', 'desktop'];

    let answer = readline.question('\nWhat time of day is it (enter number)? 1. morning 2. afternoon 3. evening 4. night\n');
    let selection = parseInt(answer);
    const timeOfDay = selection >= 1 && selection <= 4 ? timeOfDayFeatures[selection - 1] : timeOfDayFeatures[0];

    answer = readline.question('\nWhat type of device is the user on (enter number)? 1. mobile 2. tablet 3. desktop\n');
    selection = parseInt(answer);
    const device = selection >= 1 && selection <= 3 ? deviceFeatures[selection - 1] : deviceFeatures[0];

    console.log('Selected features:\n');
    console.log('Time of day: ' + timeOfDay + '\n');
    console.log('Device: ' + device + '\n');

    return [
        {
            'time': timeOfDay
        },
        {
            'device': device
        }
    ];
}
```

## <a name="get-slots"></a>Obtención de ranuras

Las ranuras componen la página con la que va a interactuar el usuario. Personalizer decidirá qué acción mostrar en cada una de las ranuras definidas. Es posible excluir acciones de ranuras específicas, que se muestran como `ExcludeActions`. `BaselineAction` es la acción predeterminada para la ranura que se habría mostrado si no se hubiera usado Personalizer.


Esta guía de inicio rápido tiene características de ranura sencillas. En sistemas de producción, determinar y [evaluar](../concept-feature-evaluation.md) [características](../concepts-features.md) puede ser una tarea compleja.

```javascript
function getSlots() {
    return [
        {
            'id': 'BigHeroPosition',
            'features': [
                {
                    'size': 'large',
                    'position': 'left',
                }
            ],
            'excludedActions': ['31-Inch-Monitor-771'],
            'baselineAction': 'Red-Polo-Shirt-432'
        },
        {
            'id': 'SmallSidebar',
            'features': [
                {
                    'size': 'small',
                    'position': 'right',
                }
            ],
            'excludedActions': ['Tennis-Racket-133'],
            'baselineAction': 'XBox-Series X-117'
        }
    ];
}
```

## <a name="make-http-requests"></a>Realización de solicitudes HTTP

Envíe solicitudes POST al punto de conexión de Personalizer para realizar llamadas Rank y Reward de varias ranuras.

```javascript
async function sendMultiSlotRank(rankRequest) {
    try {
        let response = await axios.post(MultiSlotRankUrl, rankRequest, { headers: Headers })
        return response.data;
    }
    catch (err) {
        console.log(err);
    }
}
```

```javascript
async function sendMultiSlotReward(rewardRequest, eventId) {
    try {
        let rewardUrl = MultiSlotRewardUrlBase.concat(eventId, '/reward');
        let response = await axios.post(rewardUrl, rewardRequest, { headers: Headers })
    }
    catch (err) {
        console.log(err);
    }
}
```

## <a name="get-feedback-for-personalizer-decisions"></a>Obtención de comentarios sobre decisiones de Personalizer


Agregue el siguiente método al script. Indicará si Personalizer ha tomado una decisión correcta con relación a cada ranura mediante un símbolo de la línea de comandos.

```javascript
function getRewardForSlot() {
    let answer = readline.question('\nIs this correct? (y/n)\n').toUpperCase();
    if (answer === 'Y') {
        console.log('\nGreat! The application will send Personalizer a reward of 1 so it learns from this choice of action for this slot.\n');
        return 1;
    }
    else if (answer === 'N') {
        console.log('\nYou didn\'t like the recommended item.The application will send Personalizer a reward of 0 for this choice of action for this slot.\n');
        return 0;
    }
    console.log('\nEntered choice is invalid. Service assumes that you didn\'t like the recommended item.\n');
    return 0;
}
```

## <a name="create-the-learning-loop"></a>Creación del bucle de aprendizaje

El bucle de aprendizaje de Personalizer es un ciclo de llamadas [Rank](#request-the-best-action) y [Reward](#send-a-reward). En este inicio rápido, cada llamada Rank para personalizar el contenido, va seguida de una llamada Reward para indicar a Personalizer cómo es de eficaz el funcionamiento del servicio.

El siguiente código pasa por un ciclo en el que se pregunta al usuario sus preferencias en la línea de comandos, se envía esa información a Personalizer para que seleccione la mejor acción para cada ranura, se presenta la selección al cliente para que elija entre las opciones de la lista y, después, se envía una puntuación de recompensa a Personalizer en la que se señala el grado de acierto que ha tenido el servicio en su selección.

```javascript
runLoop = true;

(async () => {
    do {

        let multiSlotRankRequest = {};

        // Generate an ID to associate with the request.
        multiSlotRankRequest.eventId = uuidv4();

        // Get context information from the user.
        multiSlotRankRequest.contextFeatures = getContextFeatures();

        // Get the actions list to choose from personalization with their features.
        multiSlotRankRequest.actions = getActions();

        // Get the list of slots for which Personalizer will pick the best action.
        multiSlotRankRequest.slots = getSlots();

        multiSlotRankRequest.deferActivation = false;

        //Rank the actions for each slot
        try {
            var multiSlotRankResponse = await sendMultiSlotRank(multiSlotRankRequest);
        }
        catch (err) {
            console.log(err);
        }

        let multiSlotrewards = {};
        multiSlotrewards.reward = [];

        for (i = 0; i < multiSlotRankResponse.slots.length; i++) {
            console.log('\nPersonalizer service decided you should display: '.concat(multiSlotRankResponse.slots[i].rewardActionId, ' in slot ', multiSlotRankResponse.slots[i].id, '\n'));

            let slotReward = {};
            slotReward.slotId = multiSlotRankResponse.slots[i].id;
            // User agrees or disagrees with Personalizer decision for slot
            slotReward.value = getRewardForSlot();
            multiSlotrewards.reward.push(slotReward);
        }

        // Send the rewards for the event
        try {
            await sendMultiSlotReward(multiSlotrewards, multiSlotRankResponse.eventId);
        }
        catch (err) {
            console.log(err);
        }

        let answer = readline.question('\nPress q to break, any other key to continue:\n').toUpperCase();
        if (answer === 'Q') {
            runLoop = false;
        }

    } while (runLoop);
})()
```

Observe detalladamente las llamadas de clasificación y recompensa en las secciones siguientes.

Agregue los siguientes métodos, que [obtienen las opciones de contenido seleccionadas](#get-content-choices-represented-as-actions), [obtienen las preferencias del usuario con respecto al contexto](#get-user-preferences-for-context), [obtienen las ranuras](#get-slots), [realizan solicitudes HTTP](#make-http-requests) y [obtienen la recompensa de cada ranura](#get-feedback-for-personalizer-decisions) antes de ejecutar el archivo de código:

* getActions
* getContextFeatures
* getSlots
* sendRank
* sendReward
* getRewardForSlot

## <a name="request-the-best-action"></a>Solicitud de la mejor acción

Para completar la solicitud Rank, el programa solicita las preferencias del usuario para crear opciones de contenido. El cuerpo de la solicitud contiene las características de contexto, las acciones y sus características, las ranuras y sus características, y un id. de evento único, con el fin de recibir la respuesta. El método `sendMultiSlotRank` necesita el objeto rankRequest para enviar la solicitud Rank de varias ranuras.

Esta guía de inicio rápido tiene características de contexto sencillas de hora del día y dispositivo del usuario. En los sistemas de producción, la determinación y [evaluación](../concept-feature-evaluation.md) de [acciones y características](../concepts-features.md) no es una cuestión trivial.

```javascript
let multiSlotRankRequest = {};

// Generate an ID to associate with the request.
multiSlotRankRequest.eventId = uuidv4();

// Get context information from the user.
multiSlotRankRequest.contextFeatures = getContextFeatures();

// Get the actions list to choose from personalization with their features.
multiSlotRankRequest.actions = getActions();

// Get the list of slots for which Personalizer will pick the best action.
multiSlotRankRequest.slots = getSlots();

multiSlotRankRequest.deferActivation = false;

//Rank the actions for each slot
try {
    var multiSlotRankResponse = await sendMultiSlotRank(multiSlotRankRequest);
}
catch (err) {
    console.log(err);
}
```

## <a name="send-a-reward"></a>Envío de una recompensa

Para obtener la puntuación de recompensa que se enviará en la solicitud Reward, el programa obtiene la selección del usuario mediante la línea de comandos, asigna un valor numérico a la selección y, después, envía el id. de evento único y la puntuación de recompensa para cada ranura como valor numérico al método `sendMultiSlotReward`. No es necesario definir una recompensa para cada ranura.

En este inicio rápido se asigna un número simple como puntuación de recompensa: 0 o 1. En sistemas de producción, determinar cuándo y qué enviar a la llamada [Reward](../concept-rewards.md) puede ser más complicado, en función de las necesidades específicas.

```javascript
let multiSlotrewards = {};
multiSlotrewards.reward = [];

for (i = 0; i < multiSlotRankResponse.slots.length; i++) {
    console.log('\nPersonalizer service decided you should display: '.concat(multiSlotRankResponse.slots[i].rewardActionId, ' in slot ', multiSlotRankResponse.slots[i].id, '\n'));

    let slotReward = {};
    slotReward.slotId = multiSlotRankResponse.slots[i].id;
    // User agrees or disagrees with Personalizer decision for slot
    slotReward.value = getRewardForSlot();
    multiSlotrewards.reward.push(slotReward);
}

// Send the rewards for the event
try {
    await sendMultiSlotReward(multiSlotrewards, multiSlotRankResponse.eventId);
}
catch (err) {
    console.log(err);
}
```

## <a name="run-the-program"></a>Ejecución del programa

Ejecute la aplicación con el comando de Node.js desde el directorio de aplicación.

```console
node sample.js
```

![El programa del inicio rápido realiza dos preguntas para recopilar las preferencias del usuario, conocidas como características, y, después, proporciona la acción principal.](../media/csharp-quickstart-commandline-feedback-loop/multislot-quickstart-program-feedback-loop-example-1.png)


El [código fuente de este inicio rápido](https://aka.ms/personalizer/ms-nodejs) está disponible.
