---
title: Conceptos de integración de telefonía RTC para Azure Communication Services
description: Aprenda a integrar las funcionalidades de llamadas RTC a una aplicación de Azure Communication Services.
author: boris-bazilevskiy
manager: nmurav
services: azure-communication-services
ms.author: bobazile
ms.date: 03/10/2021
ms.topic: overview
ms.service: azure-communication-services
ms.openlocfilehash: 5fba53125e22eed6797db7ada6b99e679e136ea8
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110092411"
---
# <a name="telephony-concepts"></a>Conceptos de telefonía

[!INCLUDE [Regional Availability Notice](../../includes/regional-availability-include.md)]

Los SDK de llamadas de Azure Communication Services se pueden usar para agregar telefonía y RTC a las aplicaciones. En esta página se resumen las funcionalidades y los conceptos clave de la telefonía. Para más información sobre los lenguajes y funcionalidades específicos del SDK, consulte la [biblioteca de llamadas](../../quickstarts/voice-video-calling/calling-client-samples.md).

## <a name="overview-of-telephony"></a>Introducción a la telefonía
Siempre que los usuarios interactúan con un número de teléfono tradicional, las llamadas se facilitan mediante llamadas de voz de RTC (red telefónica conmutada). Para hacer y recibir llamadas RTC, tiene que agregar funcionalidades de telefonía a su recurso de Azure Communication Services. En este caso, las señales y los elementos multimedia usan una combinación de tecnologías basadas en IP y en RTC para conectar a los usuarios. Communication Services proporciona dos maneras discretas de acceder a la red RTC: llamadas de voz (RTC) y enrutamiento directo de Azure.

### <a name="voice-calling-pstn"></a>Llamadas de voz (RTC)

Una manera sencilla de agregar conectividad RTC a una aplicación o servicio, en este caso, Microsoft es su proveedor de telecomunicaciones. Puede comprar números directamente a Microsoft. Azure Cloud Calling es una solución de telefonía todo en la nube para Communication Services. Es la opción más sencilla para conectar ACS a la red telefónica conmutada (RTC) pública para habilitar las llamadas a teléfonos fijos y móviles de todo el mundo. Con esta opción, Microsoft actúa como operador de RTC, como se muestra en el diagrama siguiente:

![Diagrama de llamadas de voz (RTC).](../media/telephony-concept/azure-calling-diagram.png)

Si responde afirmativamente a las siguientes afirmaciones, Llamadas de voz (RTC) es la solución más adecuada para usted:
- Llamadas de voz (RTC) está disponible en su región.
- No es preciso que siga utilizando el operador de RTC actual.
- Quiere usar el acceso administrado por Microsoft a la RTC.

Con esta opción:
- Obtiene los números directamente de Microsoft y puede llamar a teléfonos de todo el mundo.
- No se requiere la implementación ni el mantenimiento de una implementación local, ya que Llamadas de voz (RTC) funciona fuera de Azure Communication Services.
- Nota: Si es necesario, puede conectar un controlador de límites de sesión (SBC) compatible mediante el enrutamiento directo de Azure para lograr interoperabilidad con las PBX de terceros, los dispositivos analógicos y otros equipos de telefonía de terceros compatibles con el SBC.

Esta opción requiere una conexión ininterrumpida a Azure Communication Services.  

En el caso de las llamadas a la nube, las llamadas salientes se facturan según tarifas por minuto en función del país de destino. Consulte la [lista de tarifas actuales para las llamadas RTC](https://github.com/Azure/Communication/blob/master/pricing/communication-services-pstn-rates.csv).

### <a name="azure-direct-routing"></a>Enrutamiento directo de Azure

[!INCLUDE [Private Preview Notice](../../includes/private-preview-include.md)]

Con esta opción, puede conectar tanto la telefonía local heredada como el operador que prefiera a Azure Communication Services. Proporciona funcionalidades de llamada RTC a las aplicaciones de ACS aunque Llamadas de voz (RTC) no esté disponible en su país o región. 

![Diagrama del enrutamiento directo de Azure.](../media/telephony-concept/sip-interface-diagram.png)

Si su respuesta es afirmativa a cualquiera de las siguientes preguntas, el enrutamiento directo de Azure es la solución adecuada para usted:

- Quiere usar ACS con las funcionalidades de llamada RTC.
- Necesita seguir usando el operador de RTC actual.
- Desea mezclar el enrutamiento, es decir, que algunas llamadas vayan a través de Llamadas de voz (RTC) y otras por el operador de telefonía.
- Necesita interoperar con PBX de terceros o con equipos como buscapersonas, dispositivos analógicos, etc.

Con esta opción:

- Puede conectar su propio SBC compatible a Azure Communication Services sin necesidad de software local adicional.
- Puede usar literalmente todos los operadores de telefonía con ACS.
- Si lo desea, puede optar por configurar y administrar esta opción, o bien puede encargarse de ello su operador o asociado (pregunte si ambos ofrecen esta opción)
- Puede configurar la interoperabilidad entre el equipo de telefonía, como un PBX de terceros y dispositivos analógicos, y ACS.

Esta opción requiere los siguiente elementos:

- Conexión ininterrumpida a Azure.
- Implementación y mantenimiento de un SBC compatible.
- Un contrato con un operador de terceros (salvo que se implemente como una opción para proporcionar conexión con una PBX de terceros, dispositivos analógicos u otro equipo de telefonía para los usuarios que se encuentran en Communication Services).

## <a name="next-steps"></a>Pasos siguientes

### <a name="conceptual-documentation"></a>Documentación conceptual

- [Tipos de número de teléfono en Azure Communication Services](./plan-solution.md)
- [Planeamiento del enrutamiento directo de Azure](./sip-interface-infrastructure.md)
- [Precios](../pricing.md)

### <a name="quickstarts"></a>Guías de inicio rápido

- [Obtención de un número de teléfono](../../quickstarts/telephony-sms/get-phone-number.md)
- [Llamada a teléfono](../../quickstarts/voice-video-calling/pstn-call.md)
