---
title: SDK y API de REST para Azure Communication Services
titleSuffix: An Azure Communication Services concept document
description: Obtenga más información acerca de los SDK de Azure Communication Services y las API de REST.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 03/25/2021
ms.topic: conceptual
ms.service: azure-communication-services
ms.openlocfilehash: d71720b446baa8e2e5e15c407a69d9ff884004b2
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307697"
---
# <a name="sdks-and-rest-apis"></a>SDK y API REST

Las funcionalidades de Azure Communication Services están organizadas conceptualmente en seis áreas. La mayoría de las áreas tienen SDK de código abierto totalmente programados con las API de REST publicadas que puede usar directamente a través de Internet. El SDK de llamada usa interfaces de red propietarias y tiene actualmente formato de código cerrado. En el [repositorio de GitHub de Azure Communication Services](https://github.com/Azure/communication) hay publicados ejemplos y detalles técnicos adicionales de los SDK.

## <a name="rest-apis"></a>API de REST
Las API de Communication Services están documentadas junto con otras API REST de Azure en [docs.microsoft.com](/rest/api/azure/). Esta documentación le indicará cómo estructurar los mensajes HTTP y ofrece instrucciones para el uso de Postman. Esta documentación también se ofrece en formato Swagger en [GitHub](https://github.com/Azure/azure-rest-api-specs).


## <a name="sdks"></a>SDK

| Ensamblado | Espacios de nombres| Protocolos | Funcionalidades |
|------------------------|-------------------------------------|---------------------------------|--------------------------------------------------------------------------------------------|
| Azure Resource Manager | Azure.ResourceManager.Communication | [REST](https://docs.microsoft.com/rest/api/communication/communicationservice)| Aprovisiona y administra recursos de Communication Services.|
| Comunes | Azure.Communication.Common| REST | Proporciona tipos base para otros SDK |
| Identidad | Azure.Communication.Identity| [REST](https://docs.microsoft.com/rest/api/communication/communicationidentity)| Administración de usuarios y tokens de acceso|
| Números de teléfono _(beta)_| Azure.Communication.PhoneNumbers| [REST](https://docs.microsoft.com/rest/api/communication/phonenumberadministration)| Adquisición y administración de números de teléfono |
| Chat | Azure.Communication.Chat| [REST](https://docs.microsoft.com/rest/api/communication/) con señalización propietaria | Incorpora chat basado en texto en tiempo real a las aplicaciones. |
| SMS| Azure.Communication.SMS | [REST](https://docs.microsoft.com/rest/api/communication/sms)| Envía y recibe mensajes SMS.|
| Llamar| Azure.Communication.Calling | Transporte propietario | Permite usar la voz, el vídeo, el uso compartido de pantalla y otras capacidades de comunicación de datos en tiempo real. |

Tenga en cuenta que los SDK de Azure Resource Manager, de las identidades y de los SMS se centran en la integración del servicio y, en muchos casos, surgen problemas de seguridad si se integran estas funciones en aplicaciones de usuario final. Los SDK comunes y de chat son adecuados para las aplicaciones cliente y de servicio. Los SDK de llamada están diseñados para las aplicaciones cliente. Un SDK centrado en escenarios de servicio está en fase de desarrollo.


### <a name="languages-and-publishing-locations"></a>Idiomas y ubicaciones de publicación

A continuación se detallan las ubicaciones de publicación para los paquetes de SDK individuales.

| Área           | JavaScript | .NET | Python | Java SE | iOS | Android | Otros                          |
| -------------- | ---------- | ---- | ------ | ---- | -------------- | -------------- | ------------------------------ |
| Azure Resource Manager | -         | [NuGet](https://www.nuget.org/packages/Azure.ResourceManager.Communication)    |   [PyPi](https://pypi.org/project/azure-mgmt-communication/)    |  -  | -              | -  | [Go a través de GitHub](https://github.com/Azure/azure-sdk-for-go/releases/tag/v46.3.0) |
| Comunes         | [npm](https://www.npmjs.com/package/@azure/communication-common)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Common/)    | N/D      | [Maven](https://search.maven.org/search?q=a:azure-communication-common)   | [GitHub](https://github.com/Azure/azure-sdk-for-ios/releases)            | [Maven](https://search.maven.org/artifact/com.azure.android/azure-communication-common)             | -                              |
| Identidad | [npm](https://www.npmjs.com/package/@azure/communication-identity)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Identity)    | [PyPi](https://pypi.org/project/azure-communication-identity/)      | [Maven](https://search.maven.org/search?q=a:azure-communication-identity)   | -              | -              | -                            |
| Números de teléfono | [npm](https://www.npmjs.com/package/@azure/communication-phone-numbers)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.PhoneNumbers)    | [PyPi](https://pypi.org/project/azure-communication-phonenumbers/)      | [Maven](https://search.maven.org/search?q=a:azure-communication-phonenumbers)   | -              | -              | -                            |
| Chat           | [npm](https://www.npmjs.com/package/@azure/communication-chat)        | [NuGet](https://www.nuget.org/packages/Azure.Communication.Chat)     | [PyPi](https://pypi.org/project/azure-communication-chat/)     | [Maven](https://search.maven.org/search?q=a:azure-communication-chat)   | [GitHub](https://github.com/Azure/azure-sdk-for-ios/releases)  | [Maven](https://search.maven.org/search?q=a:azure-communication-chat)   | -                              |
| SMS            | [npm](https://www.npmjs.com/package/@azure/communication-sms)         | [NuGet](https://www.nuget.org/packages/Azure.Communication.Sms)    | [PyPi](https://pypi.org/project/azure-communication-sms/)       | [Maven](https://search.maven.org/artifact/com.azure/azure-communication-sms)   | -              | -              | -                              |
| Llamar        | [npm](https://www.npmjs.com/package/@azure/communication-calling)         | -      | -      | -     | [GitHub](https://github.com/Azure/Communication/releases)     | [Maven](https://search.maven.org/artifact/com.azure.android/azure-communication-calling/)            | -                              |
| Documentación de referencia     | [docs](https://azure.github.io/azure-sdk-for-js/communication.html)         | [docs](https://azure.github.io/azure-sdk-for-net/communication.html)      | -      | [docs](http://azure.github.io/azure-sdk-for-java/communication.html)     | [docs](/objectivec/communication-services/calling/)      | [docs](/java/api/com.azure.android.communication.calling)            | -                              |


## <a name="rest-api-throttles"></a>Limitaciones de la API de REST
Ciertas API de REST y los métodos de SDK correspondientes tienen límites que se deben tener en cuenta. Si se superan estos límites, se desencadenará una respuesta de error `429 - Too Many Requests`. Estos límites se pueden aumentar a través de [una solicitud al Soporte técnico de Azure](https://docs.microsoft.com/azure/azure-portal/supportability/how-to-create-azure-support-request).

| API                                                                                                                          | Limitación            |
|------------------------------------------------------------------------------------------------------------------------------|---------------------|
| [Todas las API de plan de número de teléfono de búsqueda](https://docs.microsoft.com/rest/api/communication/phonenumberadministration)         | 4 solicitudes/día      |
| [Plan de número de teléfono de compra](https://docs.microsoft.com/rest/api/communication/phonenumberadministration/purchasesearch) | 1 compra al mes  |
| [Envío de SMS](https://docs.microsoft.com/rest/api/communication/sms/send)                                                       | 200 solicitudes por minuto |


## <a name="sdk-platform-support-details"></a>Detalles de compatibilidad de la plataforma de SDK

### <a name="ios-and-android"></a>iOS y Android 

- Los SDK de iOS de Communication Services tienen como destino iOS versión 13+ y Xcode 11+.
- Los SDK de Java para Android se dirigen al nivel de API de Android 21+ y Android Studio 4.0+.

### <a name="net"></a>.NET 

A excepción de las llamadas, los paquetes de Communication Services tienen como destino .NET Standard 2.0, que es compatible con las plataformas que se enumeran a continuación.

Compatibilidad mediante .NET Framework 4.6.1
- Windows 10, 8.1, 8 y 7
- Windows Server 2012 R2, 2012 y 2008 R2 SP1

Compatibilidad mediante .NET Core 2.0:
- Windows 10 (1607+), 7 SP1+, 8.1
- Windows Server 2008 R2 SP1 y versiones posteriores
- Max OS X 10.12+
- Varias versiones o distribuciones de Linux
- UWP 10.0.16299 (RS3) septiembre 2017
- Unity 2018.1
- Mono 5.4
- Xamarin iOS 10.14
- Xamarin Mac 3.8

## <a name="api-stability-expectations"></a>Expectativas de estabilidad de API

> [!IMPORTANT]
> En esta sección se proporcionan instrucciones sobre las API DE REST y los SDK marcados como **estables**. Las API marcadas como versión preliminar o beta se pueden cambiar o dejar de usar **sin previo aviso**.

En el futuro, es posible que retiremos las versiones de los SDK de Communication Services y que introduzcamos cambios importantes en nuestras API de REST y SDK publicados. Azure Communication Services *generalmente* sigue dos directivas de compatibilidad para retirar versiones de servicio:

- Se le notificará con una antelación de al menos tres años cuando sea necesario cambiar el código debido a un cambio en la interfaz de Communication Services. Todas las API de REST documentadas y las API de los SDK generalmente disfrutan de un período de al menos tres años de advertencia antes de que se retiren las interfaces.
- Se le notificará al menos un año antes de que tenga que actualizar los ensamblados de los SDK a la versión secundaria más reciente. Estas actualizaciones necesarias no deben requerir ningún cambio en el código porque están en la misma versión principal. Esto es especialmente pertinente en el caso de las bibliotecas de llamadas y chat que tienen componentes en tiempo real que requieren con frecuencia actualizaciones de rendimiento y seguridad. Le recomendamos encarecidamente que mantenga actualizadas sus SDK de Communication Services.

### <a name="api-and-sdk-decommissioning-examples"></a>Ejemplos de retirada de API y SDK

**Ha integrado la versión 24 de la API REST de SMS en la aplicación. Publicaciones de Azure Communication Services v25.**

Recibirá una advertencia tres años antes de que estas API dejen de funcionar y sea obligatorio actualizarlas a la v25. Es posible que esta actualización requiera un cambio de código.

**Ha integrado la versión v2.02 de los SDK de llamadas en la aplicación. Publicaciones de Azure Communication Services v2.05.**

Es posible que se le pida que actualice a la versión v2.05 de los SDK de llamadas en un plazo de 12 meses a partir de la publicación de la v2.05. Debe ser un reemplazo sencillo del artefacto sin necesidad de un cambio de código porque la v2.05 está en la versión principal v2 y no tiene cambios importantes.

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, consulte la siguiente información general de los SDK:

- [Información general del SDK de llamada](../concepts/voice-video-calling/calling-sdk-features.md)
- [Información general del SDK de chat](../concepts/chat/sdk-features.md)
- [Información general del SDK de SMS](../concepts/telephony-sms/sdk-features.md)

Para empezar a trabajar con Azure Communication Services:

- [Cree recursos de Azure Communication Services](../quickstarts/create-communication-resource.md).
- Genere [tokens de acceso de usuario](../quickstarts/access-tokens.md).
