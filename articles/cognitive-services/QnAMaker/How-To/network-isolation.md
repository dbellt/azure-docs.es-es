---
title: Aislamiento de red
description: Los usuarios pueden restringir el acceso público a los recursos de QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: fa24347c8fcc0550dc6dc86c96624d1b1f6dcf25
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110376329"
---
# <a name="recommended-settings-for-network-isolation"></a>Configuración recomendada para el aislamiento de red

Debe seguir los pasos siguientes para restringir el acceso público a los recursos de QnA Maker. Proteja un recurso de Cognitive Services del acceso público mediante la [configuración de la red virtual](../../cognitive-services-virtual-networks.md?tabs=portal).

## <a name="restrict-access-to-app-service-qna-runtime"></a>Restricción del acceso a App Service (entorno de ejecución de QnA)

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

Puede agregar direcciones IP a la lista de permitidos de App Service para restringir el acceso o configurar el entorno de App Service para hospedar App Service en QnA Maker.

#### <a name="add-ips-to-app-service-allow-list"></a>Adición de direcciones IP a la lista de permitidos de App Service

1. Permita el tráfico solo desde direcciones IP de Cognitive Services. Estas direcciones ya están incluidas en la etiqueta de servicio `CognitiveServicesManagement`. Esto es necesario para la creación de API (crear o actualizar KB) para invocar el servicio de aplicaciones y actualizar el servicio Azure Search en consecuencia. Consulte [más información sobre las etiquetas de servicio](../../../virtual-network/service-tags-overview.md).
2. Asegúrese de que también permite otros puntos de entrada como Azure Bot Service, el portal de QnA Maker, etc., para el acceso de la API "GenerateAnswer" de predicción.
3. Siga estos pasos para agregar los intervalos de direcciones IP a una lista de permitidos:

   1. Descargue los [intervalos IP de todas las etiquetas de servicio](https://www.microsoft.com/download/details.aspx?id=56519).
   2. Seleccione las direcciones IP de "CognitiveServicesManagement".
   3. Vaya a la sección de redes del recurso App Service y haga clic en la opción "Configure Access Restriction" (Configurar restricción de acceso) para agregar las direcciones IP a una lista de permitidos.

También tenemos un script automatizado para hacer lo mismo para App Service. El [script de PowerShell para configurar una lista de permitidos](https://github.com/pchoudhari/QnAMakerBackupRestore/blob/master/AddRestrictedIPAzureAppService.ps1) se encuentra en GitHub. Los parámetros del script que hay que especificar son el identificador de la suscripción, el grupo de recursos y el nombre real de App Service. Al ejecutar el script se agregarán automáticamente las direcciones IP a la lista de permitidos de App Service.

#### <a name="configure-app-service-environment-to-host-qna-maker-app-service"></a>Configuración de App Service Environment para hospedar una instancia de App Service de QnA Maker
    
App Service Environment se puede usar para hospedar la instancia de App Service para QnA Maker. Siga estos pasos:

1. Cree una instancia de App Service Environment y márquela como "externa". Para obtener instrucciones, siga este [tutorial](../../../app-service/environment/create-external-ase.md).
2.  Cree una instancia de App Service en App Service Environment.
    1. Compruebe la configuración de App Service y agregue "PrimaryEndpointKey" como configuración de la aplicación. El valor de "PrimaryEndpointKey" debe establecerse en "\<app-name\>-PrimaryEndpointKey". El nombre de la aplicación se define en la dirección URL de App Service. Por ejemplo, si la dirección URL de App Service es "misitioweb.myase.p.azurewebsite.net", el nombre de la aplicación será "misitioweb". En este caso, el valor de "PrimaryEndpointKey" se debe establecer en "misitioweb-PrimaryEndpointKey".
    2. Cree un servicio Azure Search.
    3. Asegúrese de que Azure Search y la configuración de la aplicación estén configuradas correctamente. 
          Siga este [tutorial](../reference-app-service.md?tabs=v1#app-service).
3.  Actualización del grupo de seguridad de red asociado con App Service Environment
    1. Actualice las reglas de seguridad de entrada creadas previamente según sus requisitos.
    2. Agregue una nueva regla de seguridad de entrada con el origen "etiqueta de servicio" y la etiqueta de servicio de origen como "CognitiveServicesManagement".
       
    ![excepciones de puertos de entrada](../media/inbound-ports.png)

4.  Cree una instancia de Cognitive Services de QnA Maker (Microsoft.CognitiveServices/accounts) mediante Azure Resource Manager, donde el punto de conexión de QnA Maker debe establecerse en el punto de conexión de App Service creado anteriormente (https:// misitioweb.myase.p.azurewebsite.net).
    
# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

App Service no se implementa con la característica de respuesta a preguntas personalizada.

---

## <a name="restrict-access-to-cognitive-search-resource"></a>Restricción del acceso a los recursos de Cognitive Search

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

La instancia de Cognitive Search se puede aislar a través de un punto de conexión privado después de haber creado los recursos de QnA Maker. Las conexiones de punto de conexión privado requieren una red virtual a través de la que se pueda tener acceso a la instancia del servicio Search. 

Si la instancia de App Service en QnA Maker está restringida con App Service Environment, use la misma red virtual para crear una conexión de punto de conexión privado a la instancia de Cognitive Search. Cree una nueva entrada DNS en la red virtual para asignar el punto de conexión de Cognitive Search a la dirección IP del punto de conexión privado de Cognitive Search. 

Si no se usa App Service Environment para App Service en QnAMaker, cree primero un nuevo recurso de red virtual y, a continuación, cree la conexión de punto de conexión privado a la instancia de Cognitive Search. En este caso, App Service en QnA Maker debe [integrarse con la red virtual](../../../app-service/web-sites-integrate-with-vnet.md) para conectarse a la instancia de Cognitive Search. 

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

[Cree puntos de conexión privados](../reference-private-endpoint.md) al recurso de Azure Search.

---

