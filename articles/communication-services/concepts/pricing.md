---
title: Escenarios de precios para llamadas (voz o vídeo) y chat
titleSuffix: An Azure Communication Services concept document
description: Obtenga información sobre el modelo de precios de Communication Services.
author: nmurav
manager: nmurav
services: azure-communication-services
ms.author: nmurav
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 038b4df78ca7f10b0ec0e9dfe224f6aca2430e82
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111986307"
---
# <a name="pricing-scenarios"></a>Escenarios de precios

Los precios de Azure Communication Services se suelen basar en un modelo de pago por uso. Los precios de los siguientes ejemplos solo tienen fines ilustrativos y puede que no reflejen los precios más recientes de Azure.

## <a name="voicevideo-calling-and-screen-sharing"></a>Llamadas de voz o vídeo y uso compartido de pantalla

Azure Communication Services permite agregar llamadas de voz o vídeo a las aplicaciones, así como uso compartido de pantalla. Puede insertar la experiencia en sus aplicaciones mediante los SDK para JavaScript, Objective-C (Apple), Java (Android) o .NET. Consulte nuestra [lista completa de SDK disponibles](./sdk-options.md).

### <a name="pricing"></a>Precios

Los servicios de llamadas y uso compartido de pantalla se cobran por minuto a cada participante con un costo de 0,004 USD por participante por minuto para las llamadas de grupo. Para más información sobre los distintos flujos de llamada posibles, consulte [esta página](./call-flows.md).

Cada participante de la llamada se contará en la facturación por cada minuto que esté conectado a la llamada. Este principio se aplica independientemente de si el usuario está realizando llamadas por vídeo o voz, o si está compartiendo su pantalla.

### <a name="pricing-example-group-audiovideo-call-using-js-and-ios-sdks"></a>Ejemplo de precios: llamada grupal de audio o vídeo mediante los SDK para JS e iOS

Alice realizó una llamada grupal con sus compañeros de trabajo, Bob y Charlie. Alice y Bob han utilizado los SDK para JS y Charlie, los SDK para iOS.

- La llamada duró un total de 60 minutos.
- Alice y Bob participaron durante la llamada completa. Alicia activó su vídeo durante cinco minutos y compartió su pantalla durante 23 minutos. Bob mantuvo encendido el vídeo durante la llamada completa (60 minutos) y compartió su pantalla durante 12 minutos.
- Charlie abandonó la llamada después de 43 minutos. Charlie usó audio y vídeo durante el tiempo que participó (43 minutos).

**Cálculos de costos**

- 2 participantes x 60 minutos x 0,004 USD por participante por minuto = 0,48 USD [el vídeo y el audio se cobran a la misma tarifa]
- 1 participante x 43 minutos x 0,004 USD por participante por minuto = 0,172 USD [el vídeo y el audio se cobran a la misma tarifa]

**Costo total de la llamada grupal**: 0,48 USD + 0,172 USD = 0,652 USD


### <a name="pricing-example-outbound-call-from-app-using-js-sdk-to-a-pstn-number"></a>Ejemplo de precios: llamada saliente de la aplicación que usa el SDK para JS a un número de RTC

Alice hace una llamada RTC desde una aplicación a Bob en su número de teléfono de EE. UU. que comienza por `+1-425`.

- Alice usó el SDK para JS para compilar la aplicación.
- La llamada dura un total de 10 minutos.

**Cálculos de costos**

- Un participante en el tramo VoIP (Alice) de la aplicación a los servidores de Communication Services x 10 minutos x 0,004 USD por tramo de participante por minuto = 0,04 USD
- Un participante en el tramo saliente PSTN (Bob) de los servidores de Communication Services a un número de teléfono de EE. UU. x 10 minutos x 0,013 USD por tramo de participante por minuto = 0,13 USD.

Nota: las tarifas mixtas de Estados Unidos a `+1-425` es 0,013 USD. Consulte el vínculo siguiente para más detalles: https://github.com/Azure/Communication/blob/master/pricing/communication-services-pstn-rates.csv)

**Costo total de la llamada**: 0,04 USD + 0,13 USD = 0,17 USD


### <a name="pricing-example-group-audio-call-using-js-sdk-and-1-pstn-leg"></a>Ejemplo de precios: llamada de audio de grupo mediante el SDK para JS y un tramo de PSTN

Alice y Bob se encuentran en una llamada de VOIP. Bob escaló la llamada a Charlie en el número PSTN de Charlie, un número de teléfono de EE. UU. que comienza por `+1-425`.

- Alice usó el SDK para JS para compilar la aplicación. Habló durante 10 minutos antes de llamar a Charlie en el número PSTN.
- Cuando Bob escaló la llamada a Charlie en su número PSTN, los tres hablaron durante otros 10 minutos.

**Cálculos de costos**

- Dos participantes en el tramo VoIP (Alice y Bob) de la aplicación a los servidores de Communication Services x 20 minutos x 0,004 USD por tramo de participante por minuto = 0,16 USD
- Un participante en el tramo saliente PSTN (Charlie) de los servidores de Communication Services a un número de teléfono de EE. UU. x 10 minutos x 0,013 USD por tramo de participante por minuto = 0,13 USD.

Nota: las tarifas mixtas de Estados Unidos a `+1-425` es 0,013 USD. Consulte el vínculo siguiente para más detalles: https://github.com/Azure/Communication/blob/master/pricing/communication-services-pstn-rates.csv)

**Costo total de la llamada de escalado + VoIP**: 0,16 USD + 0,13 USD = 0,29 USD


### <a name="pricing-example-a-user-of-the-communication-services-javascript-sdk-joins-a-scheduled-microsoft-teams-meeting"></a>Ejemplo de precios: un usuario del SDK para JS de Communication Services participa en una reunión programada de Microsoft Teams

Alice es médico y tiene una cita con uno de sus pacientes, Bob. Alice se conectará a la visita desde la aplicación de escritorio Teams. Bob recibirá un vínculo para conectarse mediante el sitio web del proveedor de atención sanitaria, que usa el SDK para JavaScript de Communication Services para conectarse a la cita. Bob usará su teléfono móvil para entrar en la reunión mediante un explorador web (iPhone con Safari). El chat estará disponible durante la visita virtual.

- La llamada dura un total de 30 minutos.
- Cuando Bob se une a la reunión, accede a la sala de reuniones de Teams según la directiva de Teams. Al cabo de un minuto, Alice lo admite en la reunión.
- Cuando Bob es admitido a la reunión, Alice y Bob participan en toda la llamada. Alice activa su vídeo 5 minutos después de que se inicie la llamada y comparte la pantalla durante 13 minutos. Bob tiene su vídeo activado durante toda la llamada.
- Alice envía cinco mensajes y Bob responde con tres mensajes.


**Cálculos de costos**

- 1 participante (Bob) conectado a la sala de reuniones de Teams x 1 minuto x 0,004 USD por participante por minuto (cargos por tarifa regular de la sala de reuniones) = 0,004 USD
- 1 participante (Bob) x 29 minutos x 0,004 USD por participante y minuto = 0,116 USD [el vídeo y el audio se cobran a la misma tarifa]
- 1 participante (Alice) x 30 minutos x 0,000 USD por participante y minuto = 0,0 USD*.
- 1 participante (Bob) x 3 mensajes de chat x 0,0008 USD = 0,0024 USD.
- 1 participante (Alice) x 5 mensajes de chat x 0,000 USD = 0,0* USD.

\* La participación de Alice está incluida en su licencia de Teams. Para su comodidad, en la factura de Azure se mostrarán los minutos y mensajes de chat que los usuarios de Teams tenían con los usuarios de Communication Services, pero los minutos y mensajes que se originan en el cliente de Teams no costarán nada.

**Costo total de la visita**:
- Conexión del usuario mediante el SDK para JavaScript de Communication Services: 0,004 USD + 0,116 USD = 0,0024 USD
- Conexión del usuario en la aplicación de escritorio Teams: 0 USD (incluido en la licencia de Teams)


## <a name="chat"></a>Chat

Con Communication Services puede mejorar su aplicación, puede permitirle enviar y recibir mensajes de chat entre dos o más usuarios. Hay SDK de chat disponibles para JavaScript, .NET, Python y Java. Consulte [esta página para obtener información sobre los SDK](./sdk-options.md).

### <a name="price"></a>Price

Se le cobrarán 0,0008 USD por cada mensaje de chat enviado.

### <a name="pricing-example-chat-between-two-users"></a>Ejemplo de precios: Chat entre dos usuarios

Geeta inicia una conversación de chat con Emily para compartir una actualización y le envía cinco mensajes. El chat dura 10 minutos. Geeta y Emily envían otros 15 mensajes cada una.

**Cálculos de costos**
- Número de mensajes enviados (5 + 15 + 15) x 0,0008 USD = 0,028 USD

### <a name="pricing-example-group-chat-with-multiple-users"></a>Ejemplo de precios: Chat grupal con varios usuarios

Charlie inicia una conversación de chat con sus amigos Casey y Jasmine para planear unas vacaciones. Conversan por unos momentos, en los que Charlie, Casey y Jasmine envían 20, 30 y 18 mensajes, respectivamente. Saben que quizá a su amiga Rose también le interese unirse al viaje, de modo que la agregan a la conversación de chat y comparten todo el historial de mensajes con ella.

Rose ve los mensajes y comienza a responder. Mientras tanto, Casey recibe una llamada y decide ponerse al día con la conversación más adelante. Charlie, Jasmine y Rose deciden las fechas del viaje y envían otros 30, 25 y 35 mensajes, respectivamente.

**Cálculos de costos**

- Número de mensajes enviados (20 + 30 + 18 + 30 + 25 + 35) x 0,0008 USD = 0,1264 USD


## <a name="telephony-and-sms"></a>Telefonía y SMS

## <a name="price"></a>Price

Los precios de los servicios de telefonía se calculan por minuto, mientras que el precio de los SMS se calcula por mensaje. Los precios vienen determinados por el tipo y la ubicación del número que usa, así como el destino de las llamadas y los mensajes SMS.

### <a name="telephone-number-leasing"></a>Alquiler del número de teléfono

Las tarifas de alquiler de números de teléfono se cobran por adelantado y, después, mes a mes:

|Tipo de número   |Precio mensual   |
|--------------|-----------|
|Local (Estados Unidos)     |1 USD/mes        |
|Gratuito (Estados Unidos) |2 USD/mes |


### <a name="telephone-calling"></a>Llamada telefónica

Las llamadas telefónicas tradicionales (llamadas que se producen a través de la red telefónica conmutada pública) están disponibles con los precios de pago por uso para los números de teléfono de Estados Unidos. El precio es un cargo por minuto según el tipo de número utilizado y el destino de la llamada. En la tabla siguiente se incluyen los detalles de precios de los destinos de llamada más populares. Consulte la [lista de precios detallada](https://github.com/Azure/Communication/blob/master/pricing/communication-services-pstn-rates.csv) para obtener una lista completa de destinos.


#### <a name="united-states-calling-prices"></a>Precios de llamadas en Estados Unidos

Los precios siguientes incluyen los impuestos y cuotas de comunicaciones necesarios:

|Tipo de número   |Para realizar llamadas   |Para recibir llamadas|
|--------------|-----------|------------|
|Local     |A partir de 0,013 USD/minuto       |0,0085 USD/minuto        |
|Llamada gratuita |0,013 USD/minuto   |0,0220 USD/minuto |

#### <a name="other-calling-destinations"></a>Otros destinos de llamada

Los precios siguientes incluyen los impuestos y cuotas de comunicaciones necesarios:

|Realizar llamadas a   |Precio por minuto|
|-----------|------------|
|Canadá     |A partir de 0,013 USD/minuto   |
|Reino Unido     |A partir de 0,015 USD/minuto   |
|Alemania     |A partir de 0,015 USD/minuto   |
|Francia     |A partir de 0,016 USD/minuto   |


### <a name="sms"></a>SMS

SMS ofrece precios de pago por uso. El precio es un cargo por mensaje en función del destino del mensaje. Los mensajes se pueden enviar mediante números de teléfono gratuitos a números de teléfono ubicados en Estados Unidos. Tenga en cuenta que los números de teléfono locales (geográficos) no se pueden usar para enviar mensajes SMS.

Los precios siguientes incluyen los impuestos y cuotas de comunicaciones necesarios:

|País   |Envío de mensajes|Recepción de mensajes|
|-----------|------------|------------|
|EE. UU. (gratuito)    |0,0075 USD/mensaje   | 0,0075 USD/mensaje |
