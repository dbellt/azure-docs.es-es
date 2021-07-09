---
title: Cómo usar puntos de conexión privados con el servicio Voz
titleSuffix: Azure Cognitive Services
description: Aprenda a usar el servicio Voz con los puntos de conexión privados que proporciona Azure Private Link
services: cognitive-services
author: alexeyo26
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/07/2021
ms.author: alexeyo
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d5de7ed4536ce7c83de4cc1e9a2d886015188e20
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110695353"
---
# <a name="use-speech-service-through-a-private-endpoint"></a>Uso del servicio Voz mediante un punto de conexión privado

[Azure Private Link](../../private-link/private-link-overview.md) le permite conectarse a los servicios de Azure mediante un [punto de conexión privado](../../private-link/private-endpoint-overview.md). Un punto de conexión privado es una dirección IP privada a la que solo se puede acceder desde una [red virtual](../../virtual-network/virtual-networks-overview.md) y una subred específicas.

En este artículo se explica cómo configurar y usar Private Link y los puntos de conexión privados con los servicios de voz de Azure Cognitive Services.
También se describe cómo quitar los puntos de conexión privados más adelante, pero seguir usando el recurso de voz.

> [!NOTE]
> Antes de continuar, vea [cómo usar redes virtuales con Cognitive Services](../cognitive-services-virtual-networks.md).



Para configurar un recurso de Voz en escenarios de punto de conexión privado, debe realizar las tareas siguientes:
1. [Creación de un nombre de dominio personalizado](#create-a-custom-domain-name)
1. [Activación de puntos de conexión privados](#turn-on-private-endpoints)
1. [Ajustar aplicaciones y soluciones existentes](#adjust-an-application-to-use-a-speech-resource-with-a-private-endpoint)

[!INCLUDE [](includes/speech-vnet-service-enpoints-private-endpoints.md)]

En este artículo se describe el uso de los puntos de conexión privados con el servicio de Voz. El uso de los puntos de conexión de servicio de red virtual se describe [aquí](speech-service-vnet-service-endpoint.md).


## <a name="create-a-custom-domain-name"></a>Creación de un nombre de dominio personalizado

Los puntos de conexión privados requieren un [nombre de subdominio personalizado para Azure Cognitive Services](../cognitive-services-custom-subdomains.md). Use las siguientes instrucciones para crear uno para el recurso de voz.

> [!WARNING]
> Un recurso de Voz con un nombre de dominio personalizado habilitado usa un método diferente para interactuar con el servicio de Voz. Puede que deba ajustar el código de la aplicación en estos dos escenarios: con un [punto de conexión privado](#adjust-an-application-to-use-a-speech-resource-with-a-private-endpoint) y [*sin*](#adjust-an-application-to-use-a-speech-resource-without-private-endpoints) un punto de conexión privado.
>
> Cuando se habilita un nombre de dominio personalizado, la operación [no es reversible](../cognitive-services-custom-subdomains.md#can-i-change-a-custom-domain-name). La única manera de volver a establecer el [nombre regional](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) es crear un recurso de voz nuevo.
>
> Si el recurso de voz tiene muchos modelos y proyectos personalizados asociados creados mediante [Speech Studio](https://speech.microsoft.com/), se recomienda encarecidamente probar la configuración en un recurso de prueba antes de modificar el usado en producción.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

Para crear un nombre de dominio personalizado mediante Azure Portal, siga estos pasos:

1. Vaya a [Azure Portal](https://portal.azure.com/) e inicie sesión con su cuenta de Azure.
1. Seleccione el recurso de voz necesario.
1. En el grupo **Administración de recursos** del panel izquierdo, seleccione **Redes**.
1. En la pestaña **Firewalls y redes virtuales**, seleccione **Generate Custom Domain Name** (Generar nombre de dominio personalizado). En el lado derecho aparece un panel con instrucciones para crear un subdominio personalizado único para el recurso.
1. En el panel **Generate Custom Domain Name** (Generar nombre de dominio personalizado), escriba un nombre de dominio personalizado. El dominio personalizado completo tiene el siguiente aspecto: `https://{your custom name}.cognitiveservices.azure.com`. 
    
    Recuerde que después de crear un nombre de dominio personalizado, _no_ se puede cambiar.
    
    Después de haber escrito su nombre de dominio personalizado, seleccione **Guardar**.
1. Una vez finalizada la operación, en el grupo **Administración de recursos**, seleccione **Keys and Endpoint** (Claves y punto de conexión). Confirme que el nuevo nombre del punto de conexión del recurso empieza de esta manera: `https://{your custom name}.cognitiveservices.azure.com`.

# <a name="powershell"></a>[PowerShell](#tab/powershell)

Para crear un nombre de dominio personalizado con PowerShell, confirme que el equipo tiene la versión 7.x o posterior de PowerShell con la versión 5.1.0 o posterior del módulo de Azure PowerShell. Para ver las versiones de estas herramientas, siga estos pasos:

1. En una ventana de PowerShell, escriba:

    `$PSVersionTable`

    Confirme que el valor de `PSVersion` es 7.x o posterior. Para actualizar PowerShell, siga las instrucciones de [Instalación de distintas versiones de PowerShell](/powershell/scripting/install/installing-powershell).

1. En una ventana de PowerShell, escriba:

    `Get-Module -ListAvailable Az`

    Si no aparece nada, o si esa versión del módulo de Azure PowerShell es anterior a 5.1.0, siga las instrucciones de [Instalación del módulo de Azure PowerShell](/powershell/azure/install-Az-ps) para actualizarla.

Antes de continuar, ejecute `Connect-AzAccount` para crear una conexión con Azure.

## <a name="verify-that-a-custom-domain-name-is-available"></a>Comprobación de que un nombre de dominio personalizado está disponible

Compruebe si el dominio personalizado que quiere usar está disponible. En el código siguiente se usa la operación [Check Domain Availability](/rest/api/cognitiveservices/accountmanagement/checkdomainavailability/checkdomainavailability) (Comprobar disponibilidad del dominio) de la API REST de Cognitive Services para confirmar que el dominio está disponible.

> [!TIP]
> El siguiente código *no* funcionará en Azure Cloud Shell.

```azurepowershell
$subId = "Your Azure subscription Id"
$subdomainName = "custom domain name"

# Select the Azure subscription that contains the Speech resource.
# You can skip this step if your Azure account has only one active subscription.
Set-AzContext -SubscriptionId $subId

# Prepare the OAuth token to use in the request to the Cognitive Services REST API.
$Context = Get-AzContext
$AccessToken = (Get-AzAccessToken -TenantId $Context.Tenant.Id).Token
$token = ConvertTo-SecureString -String $AccessToken -AsPlainText -Force

# Prepare and send the request to the Cognitive Services REST API.
$uri = "https://management.azure.com/subscriptions/" + $subId + `
    "/providers/Microsoft.CognitiveServices/checkDomainAvailability?api-version=2017-04-18"
$body = @{
subdomainName = $subdomainName
type = "Microsoft.CognitiveServices/accounts"
}
$jsonBody = $body | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri $uri -ContentType "application/json" -Authentication Bearer `
    -Token $token -Body $jsonBody | Format-List
```
Si el nombre que quiere usar está disponible, recibirá una respuesta similar a la siguiente:
```azurepowershell
isSubdomainAvailable : True
reason               :
type                 :
subdomainName        : my-custom-name
```
Si el nombre ya está en uso, obtendrá la siguiente respuesta:
```azurepowershell
isSubdomainAvailable : False
reason               : Sub domain name 'my-custom-name' is already used. Please pick a different name.
type                 :
subdomainName        : my-custom-name
```
## <a name="create-your-custom-domain-name"></a>Creación del nombre de dominio personalizado

Para activar un nombre de dominio personalizado para el recurso de voz seleccionado, use el cmdlet [Set-AzCognitiveServicesAccount](/powershell/module/az.cognitiveservices/set-azcognitiveservicesaccount).

> [!WARNING]
> Una vez que el código siguiente se ejecute correctamente, se creará un nombre de dominio personalizado para el recurso de voz. Recuerde que este nombre *no se puede* cambiar.

```azurepowershell
$resourceGroup = "Resource group name where Speech resource is located"
$speechResourceName = "Your Speech resource name"
$subdomainName = "custom domain name"

# Select the Azure subscription that contains the Speech resource.
# You can skip this step if your Azure account has only one active subscription.
$subId = "Your Azure subscription Id"
Set-AzContext -SubscriptionId $subId

# Set the custom domain name to the selected resource.
# WARNING: THIS CANNOT BE CHANGED OR UNDONE!
Set-AzCognitiveServicesAccount -ResourceGroupName $resourceGroup `
    -Name $speechResourceName -CustomSubdomainName $subdomainName
```

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

En esta sección necesitará la versión más reciente de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="verify-that-the-custom-domain-name-is-available"></a>Comprobación de que el nombre de dominio personalizado está disponible

Compruebe si el dominio personalizado que quiere utilizar está libre. Use el método [Check Domain Availability](/rest/api/cognitiveservices/accountmanagement/checkdomainavailability/checkdomainavailability) (Comprobar disponibilidad del dominio) de la API REST de Cognitive Services.

Copie el bloque de código siguiente, inserte el nombre de dominio personalizado que prefiera y guárdelo en el archivo `subdomain.json`.

```json
{
    "subdomainName": "custom domain name",
    "type": "Microsoft.CognitiveServices/accounts"
}
```

Copie el archivo en la carpeta actual o cárguelo en Azure Cloud Shell y ejecute el siguiente comando. Reemplace `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` con la identificación de su suscripción de Azure.

```azurecli-interactive
az rest --method post --url "https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.CognitiveServices/checkDomainAvailability?api-version=2017-04-18" --body @subdomain.json
```
Si el nombre que quiere usar está disponible, recibirá una respuesta similar a la siguiente:
```azurecli
{
  "isSubdomainAvailable": true,
  "reason": null,
  "subdomainName": "my-custom-name",
  "type": null
}
```

Si el nombre ya está en uso, obtendrá la siguiente respuesta:
```azurecli
{
  "isSubdomainAvailable": false,
  "reason": "Sub domain name 'my-custom-name' is already used. Please pick a different name.",
  "subdomainName": "my-custom-name",
  "type": null
}
```
## <a name="turn-on-a-custom-domain-name"></a>Activación de un nombre de dominio personalizado

Para usar un nombre de dominio personalizado para el recurso de voz seleccionado, use el comando [az cognitiveservices account update](/cli/azure/cognitiveservices/account#az_cognitiveservices_account_update).

Seleccione la suscripción de Azure que contiene el recurso de voz. Si la cuenta de Azure tiene solo una suscripción activa, puede omitir este paso. Reemplace `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` con la identificación de su suscripción de Azure.
```azurecli-interactive
az account set --subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```
Establezca el nombre de dominio personalizado en el recurso seleccionado. Reemplace los valores de ejemplo de los parámetros por los reales y ejecute el siguiente comando.

> [!WARNING]
> Después de ejecutar correctamente el siguiente comando, se creará un nombre de dominio personalizado para el recurso de voz. Recuerde que este nombre *no se puede* cambiar.

```azurecli
az cognitiveservices account update --name my-speech-resource-name --resource-group my-resource-group-name --custom-domain my-custom-name
```

***

## <a name="turn-on-private-endpoints"></a>Activación de puntos de conexión privados

Se recomienda usar para los puntos de conexión privados la [zona DNS privada](../../dns/private-dns-overview.md) asociada a la red virtual con las actualizaciones necesarias. Puede crear una zona DNS privada durante el proceso de aprovisionamiento. Si va a usar su propio servidor DNS, puede que también deba cambiar la configuración de DNS.

*Antes* de aprovisionar los puntos de conexión privados para un recurso de voz de producción, decida una estrategia de DNS. Además, pruebe los cambios de DNS, especialmente si usa su propio servidor DNS.

Para crear puntos de conexión privados, consulte uno de los artículos siguientes.
En estos artículos se emplea una aplicación web como recurso de ejemplo para que esté disponible con puntos de conexión privados.

- [Creación de un punto de conexión privado mediante Azure Portal](../../private-link/create-private-endpoint-portal.md)
- [Creación de un punto de conexión privado mediante Azure PowerShell](../../private-link/create-private-endpoint-powershell.md)
- [Cree un punto de conexión privado mediante la CLI de Azure](../../private-link/create-private-endpoint-cli.md)

Use estos parámetros en lugar de los parámetros del artículo que eligió:

| Configuración             | Value                                    |
|---------------------|------------------------------------------|
| Tipo de recurso       | **Microsoft.CognitiveServices/accounts** |
| Recurso            | **\<your-speech-resource-name>**         |
| Recurso secundario de destino | **account**                              |

**DNS de puntos de conexión privados:** Revise los principios generales de [DNS para puntos de conexión privados en los recursos de Cognitive Services](../cognitive-services-virtual-networks.md#dns-changes-for-private-endpoints). Después, realice las comprobaciones que se describen en las secciones siguientes para confirmar que la configuración de DNS funciona correctamente.

### <a name="resolve-dns-from-the-virtual-network"></a>Resolución de DNS en la red virtual

Esta comprobación es *obligatoria*.

Para probar la entrada de DNS personalizada desde la red virtual, siga estos pasos:

1. Inicie sesión en una máquina virtual que esté ubicada en la red virtual a la que haya asociado el punto de conexión privado. 
1. Abra un símbolo del sistema de Windows o un shell de Bash, ejecute `nslookup` y confirme que se resuelve correctamente el nombre de dominio personalizado del recurso.

   ```dos
   C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
   Server:  UnKnown
   Address:  168.63.129.16

   Non-authoritative answer:
   Name:    my-private-link-speech.privatelink.cognitiveservices.azure.com
   Address:  172.28.0.10
   Aliases:  my-private-link-speech.cognitiveservices.azure.com
   ```

1. Confirme que la dirección IP coincide con la dirección IP del punto de conexión privado.

### <a name="resolve-dns-from-other-networks"></a>Resolución de DNS desde otras redes

Realice esta comprobación solo si ha activado la opción **Todas las redes** o la opción de acceso **Redes y puntos de conexión privados seleccionados** en la sección **Redes** del recurso. 

Si tiene previsto acceder al recurso con solo un punto de conexión privado, puede omitir esta sección.

1. Inicie sesión en un equipo conectado a una red a la que se permita el acceso al recurso.
2. Abra un símbolo del sistema de Windows o el shell de Bash, ejecute `nslookup` y confirme que se resuelve correctamente el nombre de dominio personalizado del recurso.

   ```dos
   C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
   Server:  UnKnown
   Address:  fe80::1

   Non-authoritative answer:
   Name:    vnetproxyv1-weu-prod.westeurope.cloudapp.azure.com
   Address:  13.69.67.71
   Aliases:  my-private-link-speech.cognitiveservices.azure.com
             my-private-link-speech.privatelink.cognitiveservices.azure.com
             westeurope.prod.vnet.cog.trafficmanager.net
   ```

> [!NOTE]
> La dirección IP resuelta apunta a un punto de conexión proxy de red virtual que envía el tráfico de red al punto de conexión privado del recurso de Cognitive Services. El comportamiento será diferente en el caso de un recurso con un nombre de dominio personalizado pero *sin* puntos de conexión privados. Vea [esta sección](#dns-configuration) para obtener más información.

## <a name="adjust-an-application-to-use-a-speech-resource-with-a-private-endpoint"></a>Ajuste de una aplicación para usar un recurso de voz con un punto de conexión privado

Un recurso de voz con un nombre de dominio personalizado interactúa con los servicios de voz de una manera diferente. Esta afirmación se aplica a los recursos de voz con dominios personalizados habilitados tanto si tienen puntos de conexión privados como si no. La información de esta sección se aplica a ambos escenarios.

Siga las instrucciones de esta sección para ajustar las aplicaciones y las soluciones existentes para usar un recurso de voz con un nombre de dominio personalizado y un punto de conexión privado activado.

Un recurso de voz con un nombre de dominio personalizado y un punto de conexión privado activado usa un método diferente para interactuar con los servicios de voz. En esta sección se explica cómo usar este tipo de recurso con las API REST de los servicios de voz y el [SDK de voz](speech-sdk.md).

> [!NOTE]
> Un recurso de voz sin puntos de conexión privados, que usa un nombre de dominio personalizado también tiene una forma especial de interactuar con los servicios de voz.
> Esta forma difiere del escenario de un recurso de voz que usa un punto de conexión privado. Es importante tener esto en cuenta porque puede decidir quitar los puntos de conexión privados más adelante.
> Consulte _Ajuste de una aplicación para usar un recurso de voz sin puntos de conexión privados_ más adelante en este artículo.

### <a name="speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-rest-apis"></a>Recurso de voz con un nombre de dominio personalizado y un punto de conexión privado: Uso con las API REST

En esta sección, se usará `my-private-link-speech.cognitiveservices.azure.com` como nombre DNS del recurso de voz de ejemplo (dominio personalizado).

El servicio de Voz tiene varias API REST para la [conversión de voz en texto](rest-speech-to-text.md) y la [conversión texto en voz](rest-text-to-speech.md). Tenga en cuenta la siguiente información para el escenario con puntos de conexión privados habilitados.

Speech-to-Text tiene dos API REST. Cada API sirve para un propósito diferente, usa puntos de conexión distintos y requiere un enfoque diferente cuando se emplea en el escenario con puntos de conexión privados habilitados.

Las API de REST de conversión de voz en texto son:
- [Speech-to-Text REST API v3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30), que se usa con [transcripciones por lotes](batch-transcription.md) y [Custom Speech](custom-speech-overview.md). La versión 3.0 es la [sucesora de la versión 2.0](./migrate-v2-to-v3.md).
- [Speech-to-Text REST API para audios de corta duración](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio), que se usa con transcripciones en línea. 

El uso de Speech-to-Text REST API para audios de corta duración y Text-to-Speech REST API en el escenario de puntos de conexión privados es el mismo. Es equivalente al [caso del SDK de voz](#speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk) que se describe más adelante en este artículo. 

Speech-to-Text REST API v3.0 emplea un conjunto diferente de puntos de conexión y, por lo tanto, requiere un enfoque diferente en el escenario con puntos de conexión privados habilitados.

En las siguientes subsecciones se describen ambos casos.

#### <a name="speech-to-text-rest-api-v30"></a>API de REST de conversión de voz en texto v3.0

Normalmente, los recursos de voz usan los [puntos de conexión regionales de Cognitive Services](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) para comunicarse con [Speech-to-Text REST API v3.0](rest-speech-to-text.md#speech-to-text-rest-api-v30). Estos recursos tienen el siguiente formato de nomenclatura: <p/>`{region}.api.cognitive.microsoft.com`.

Esta es la dirección URL de una solicitud de ejemplo:

```http
https://westeurope.api.cognitive.microsoft.com/speechtotext/v3.0/transcriptions
```

> [!NOTE]
> Consulte [este artículo](sovereign-clouds.md) para puntos de conexión de Azure Government y Azure China.

Después de activar un dominio personalizado para un recurso de voz (lo cual es necesario para los puntos de conexión privados), dicho recurso usará el siguiente patrón de nombres DNS para el punto de conexión básico de la API REST: <p/>`{your custom name}.cognitiveservices.azure.com`

Esto significa que, en nuestro ejemplo, el nombre del punto de conexión de la API REST será: <p/>`my-private-link-speech.cognitiveservices.azure.com`

Asimismo, la dirección URL de la solicitud de ejemplo se debe convertir en:
```http
https://my-private-link-speech.cognitiveservices.azure.com/speechtotext/v3.0/transcriptions
```
Esta dirección URL debe ser accesible desde la red virtual que tenga el punto de conexión privado asociado (siempre que la [resolución de DNS sea correcta](#resolve-dns-from-the-virtual-network)).

Después de activar un nombre de dominio personalizado para un recurso de voz, normalmente se reemplaza el nombre de host que aparece en todas las direcciones URL de la solicitud por el nuevo nombre de host del dominio personalizado. Todas las demás partes de la solicitud (como la ruta de acceso `/speechtotext/v3.0/transcriptions` en el ejemplo anterior) permanecen igual.

> [!TIP]
> Algunos clientes desarrollan aplicaciones que usan la parte de la región del nombre DNS del punto de conexión regional (por ejemplo, para enviar la solicitud al recurso de voz implementado en una región de Azure determinada).
>
> Un dominio personalizado de un recurso de voz *no* contiene información sobre la región en la que se implementa el recurso. Por lo tanto, la lógica de aplicación descrita anteriormente *no* funcionará y deberá modificarla.

#### <a name="speech-to-text-rest-api-for-short-audio-and-text-to-speech-rest-api"></a>API de REST de conversión de voz en texto para audios breves y API de REST de conversión de texto a voz

[Speech-to-Text REST API para audios de corta duración](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) y [Text-to-Speech REST API](rest-text-to-speech.md) emplean dos tipos de puntos de conexión:
- [Puntos de conexión regionales de Cognitive Services](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) para comunicarse con la API de REST de Cognitive Services y así obtener un token de autorización.
- Puntos de conexión especiales para el resto de operaciones.

> [!NOTE]
> Consulte [este artículo](sovereign-clouds.md) para puntos de conexión de Azure Government y Azure China.

La descripción detallada de estos puntos de conexión especiales, y cómo se debe transformar su dirección URL en el caso de un recurso de voz con un punto de conexión privado habilitado, se proporciona en [esta subsección](#construct-endpoint-url) sobre el uso con el SDK de voz. El mismo principio descrito para el SDK se aplica a Speech-to-Text REST API para audios de corta duración y Text-to-Speech REST API.

Familiarícese con el material de la subsección mencionado en el párrafo anterior y consulte el ejemplo siguiente. En el ejemplo se describe Text-to-Speech REST API. El uso de Speech-to-Text REST API para audios de corta duración es totalmente equivalente.

> [!NOTE]
> Cuando se use Speech-to-Text REST API para audios de corta duración y Text-to-Speech REST API en escenarios con puntos de conexión privados, emplee una clave de suscripción que se pase a través del encabezado `Ocp-Apim-Subscription-Key`. (Más información sobre [Speech-to-text REST API para audios de corta duración](rest-speech-to-text.md#request-headers) y [Text-to-speech REST API](rest-text-to-speech.md#request-headers))
>
> Usar un token de autorización y pasarlo al punto de conexión especial a través del encabezado `Authorization` funcionará *solo* si ha activado la opción de acceso **Todas las redes** en la sección **Redes** del recurso de voz. En los demás casos, al intentar obtener un token de autorización recibirá el error `Forbidden` o `BadRequest`.

**Ejemplo de uso de Text-to-Speech API REST**

Usaremos Oeste de Europa como región de Azure de ejemplo y `my-private-link-speech.cognitiveservices.azure.com` como nombre DNS del recurso de voz de ejemplo (dominio personalizado). El nombre de dominio personalizado `my-private-link-speech.cognitiveservices.azure.com` en nuestro ejemplo pertenece al recurso de voz creado en la región Oeste de Europa.

Para obtener la lista de las voces admitidas en la región, realice la siguiente solicitud:

```http
https://westeurope.tts.speech.microsoft.com/cognitiveservices/voices/list
```
Consulte más información en la [documentación de Text-to-Speech REST API](rest-text-to-speech.md).

En los recursos de voz con puntos de conexión privados habilitados, es necesario modificar la dirección URL del punto de conexión para realizar la misma operación. La misma solicitud tendrá el siguiente aspecto:

```http
https://my-private-link-speech.cognitiveservices.azure.com/tts/cognitiveservices/voices/list
```
Puede ver una explicación detallada en la subsección [Construcción de la dirección URL del punto de conexión](#construct-endpoint-url) del SDK de voz.

### <a name="speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk"></a>Recurso de voz con un nombre de dominio personalizado y un punto de conexión privado: Uso con el SDK de voz

El uso del SDK de voz con un nombre de dominio personalizado y de los recursos de voz con un punto de conexión privado habilitado requiere que revise y, probablemente, cambie el código de la aplicación.

En esta sección, se usará `my-private-link-speech.cognitiveservices.azure.com` como nombre DNS del recurso de voz de ejemplo (dominio personalizado).

#### <a name="construct-endpoint-url"></a>Construcción de la dirección URL del punto de conexión

Normalmente, en los escenarios del SDK (así como en los de Speech-to-Text REST API y Text-to-Speech REST API) los recursos de voz usan los puntos de conexión regionales dedicados en las distintas ofertas de servicio. El formato del nombre DNS para estos puntos de conexión es:

`{region}.{speech service offering}.speech.microsoft.com`

Un nombre DNS de ejemplo es:

`westeurope.stt.speech.microsoft.com`

Todos los valores posibles de la región (primer elemento del nombre DNS) se enumeran en [Regiones admitidas del servicio de voz](regions.md) (en [este artículo](sovereign-clouds.md) encontrará más información sobre los puntos de conexión de Azure Government y Azure China). La siguiente tabla presenta los posibles valores del servicio de Voz (segundo elemento del nombre DNS):

| Valor del nombre DNS | Oferta del servicio de voz                                    |
|----------------|-------------------------------------------------------------|
| `commands`     | [Comandos personalizados](custom-commands.md)                       |
| `convai`       | [Transcripción de conversaciones](conversation-transcription.md) |
| `s2s`          | [Traducción de voz](speech-translation.md)                 |
| `stt`          | [Voz a texto](speech-to-text.md)                         |
| `tts`          | [Texto a voz](text-to-speech.md)                         |
| `voice`        | [Voz personalizada](how-to-custom-voice.md)                      |

Por lo tanto, el ejemplo anterior (`westeurope.stt.speech.microsoft.com`) significa un punto de conexión de conversión de voz en texto en Oeste de Europa.

Los puntos de conexión privados habilitados se comunican con el servicio de Voz a través de un proxy especial. Por eso, *debe cambiar las direcciones URL de la conexión del punto de conexión*. 

Una dirección URL estándar de punto de conexión tiene el siguiente aspecto: <p/>`{region}.{speech service offering}.speech.microsoft.com/{URL path}`

Una dirección URL de punto de conexión privado es similar a: <p/>`{your custom name}.cognitiveservices.azure.com/{speech service offering}/{URL path}`

**Ejemplo 1.** Una aplicación se comunica mediante la siguiente dirección URL (reconocimiento de voz mediante el modelo base para inglés de EE. UU. en el Oeste de Europa):

```
wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

Para usarla en el escenario del punto de conexión privado habilitado cuando el nombre de dominio personalizado del recurso de voz es `my-private-link-speech.cognitiveservices.azure.com`, debe modificar la dirección URL de la siguiente manera:

```
wss://my-private-link-speech.cognitiveservices.azure.com/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US
```

Observe esta información:

- El nombre de host `westeurope.stt.speech.microsoft.com` se reemplaza por el nombre de host del dominio personalizado `my-private-link-speech.cognitiveservices.azure.com`.
- El segundo elemento del nombre DNS original (`stt`) se convierte en el primer elemento de la ruta acceso de la dirección URL y precede a la ruta de acceso original. Por lo tanto, la dirección URL original `/speech/recognition/conversation/cognitiveservices/v1?language=en-US` se convierte en `/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US`.

**Ejemplo 2.** Una aplicación usa la siguiente dirección URL para sintetizar voz en la región Oeste de Europa mediante un modelo de voz personalizado:
```http
https://westeurope.voice.speech.microsoft.com/cognitiveservices/v1?deploymentId=974481cc-b769-4b29-af70-2fb557b897c4
```

La siguiente dirección URL equivalente usa un punto de conexión privado activado en el que el nombre de dominio personalizado del recurso de voz es `my-private-link-speech.cognitiveservices.azure.com`:

```http
https://my-private-link-speech.cognitiveservices.azure.com/voice/cognitiveservices/v1?deploymentId=974481cc-b769-4b29-af70-2fb557b897c4
```

Se aplica el mismo principio que en el ejemplo 1, pero el elemento clave esta vez es `voice`.

#### <a name="modifying-applications"></a>Modificación de aplicaciones

Para modificar el código, siga estos pasos:

1. Determine la dirección URL del punto de conexión de la aplicación:

   - [Active el registro de la aplicación](how-to-use-logging.md) y ejecútelo para registrar la actividad.
   - En el archivo de registro, busque `SPEECH-ConnectionUrl`. En las líneas coincidentes, el parámetro `value` contiene la dirección URL completa que la aplicación usó para comunicarse con los servicios de voz.

   Ejemplo:

   ```
   (114917): 41ms SPX_DBG_TRACE_VERBOSE:  property_bag_impl.cpp:138 ISpxPropertyBagImpl::LogPropertyAndValue: this=0x0000028FE4809D78; name='SPEECH-ConnectionUrl'; value='wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?traffictype=spx&language=en-US'
   ```

   Por lo tanto, la dirección URL que utiliza la aplicación en este ejemplo es:

   ```
   wss://westeurope.stt.speech.microsoft.com/speech/recognition/conversation/cognitiveservices/v1?language=en-US
   ```

2. Cree una instancia de `SpeechConfig` mediante una dirección URL de punto de conexión completa:

   1. Modifique el punto de conexión que acaba de determinar, como se describe en la sección anterior [Construcción de la dirección URL del punto de conexión](#construct-endpoint-url).

   1. Modifique el modo de crear la instancia de `SpeechConfig`. Lo más probable es que la aplicación esté usando algo parecido a esto:
      ```csharp
      var config = SpeechConfig.FromSubscription(subscriptionKey, azureRegion);
      ```
      Esto no funcionará en el recurso de voz con un punto de conexión privado habilitado, debido a los cambios en el nombre de host y la dirección URL descritos en las secciones anteriores. Si intenta ejecutar la aplicación existente sin ninguna modificación mediante la clave de un recurso con un punto de conexión privado habilitado, obtendrá un error de autenticación (401).

      Para que funcione, modifique la manera de crear una instancia de la clase `SpeechConfig` y use la inicialización "desde el punto de conexión"/"con el punto de conexión". Supongamos que tenemos definidas las dos variables siguientes:
      - `subscriptionKey` contiene la clave del recurso de voz con el punto de conexión privado habilitado.
      - `endPoint` contiene la dirección URL completa *modificada* del punto de conexión (para lo que se ha usado el tipo requerido por el lenguaje de programación correspondiente). En nuestro ejemplo, esta variable debe contener:
        ```
        wss://my-private-link-speech.cognitiveservices.azure.com/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US
        ```

      Cree una instancia `SpeechConfig`.
      ```csharp
      var config = SpeechConfig.FromEndpoint(endPoint, subscriptionKey);
      ```
      ```cpp
      auto config = SpeechConfig::FromEndpoint(endPoint, subscriptionKey);
      ```
      ```java
      SpeechConfig config = SpeechConfig.fromEndpoint(endPoint, subscriptionKey);
      ```
      ```python
      import azure.cognitiveservices.speech as speechsdk
      speech_config = speechsdk.SpeechConfig(endpoint=endPoint, subscription=subscriptionKey)
      ```
      ```objectivec
      SPXSpeechConfiguration *speechConfig = [[SPXSpeechConfiguration alloc] initWithEndpoint:endPoint subscription:subscriptionKey];
      ```

> [!TIP]
> Los parámetros de consulta especificados en el URI del punto de conexión no cambian, aunque se hayan establecido con otras API. Por ejemplo, si el idioma de reconocimiento se define en el URI con el parámetro de consulta `language=en-US` y también se establece en `ru-RU` a través de la propiedad correspondiente, se usa la configuración de idioma del URI. El idioma válido es entonces `en-US`.
>
> Los parámetros establecidos en el URI del punto de conexión siempre tienen prioridad. Otras API solo pueden invalidar parámetros que no se especifiquen en el URI del punto de conexión.

Después de realizar esta modificación, la aplicación debería funcionar con los recursos de voz con el punto de conexión privado habilitado. Estamos trabajando para proporcionarle una compatibilidad más fluida en los escenarios de punto de conexión privado.

## <a name="adjust-an-application-to-use-a-speech-resource-without-private-endpoints"></a>Ajuste de una aplicación para usar un recurso de voz sin un punto de conexión privado

En este artículo, hemos señalado varias veces que la habilitación de un dominio personalizado para un recurso de voz es *irreversible*. Un recurso así usará otra forma diferente de comunicarse con el servicios de Voz, en comparación con los que usan los [nombres de puntos de conexión regionales](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints).

En esta sección se explica cómo usar un recurso de voz con un nombre de dominio personalizado activado, pero *sin* ningún punto de conexión privado, con las API REST de servicios de voz y el [SDK de voz](speech-sdk.md). Este recurso puede ser uno que se haya usado una vez en un escenario de punto de conexión privado, pero del cual se hayan eliminado los puntos de conexión privados.

### <a name="dns-configuration"></a>Configuración de DNS

Recuerde que el nombre DNS del dominio personalizado del recurso de voz con un punto de conexión privado habilitado se [resuelve desde redes públicas](#resolve-dns-from-other-networks). En este caso, la dirección IP resuelta apunta a un punto de conexión proxy de una red virtual. Dicho punto de conexión se usa para distribuir el tráfico de red al recurso de Cognitive Services con un punto de conexión privado habilitado.

Sin embargo, cuando se quitan *todos* los puntos de conexión privados del recurso (o justo después de habilitar el nombre de dominio personalizado), se reaprovisiona el registro CNAME del recurso de voz. Este recurso apunta ahora a la dirección IP del [punto de conexión regional de Cognitive Services](../cognitive-services-custom-subdomains.md#is-there-a-list-of-regional-endpoints) correspondiente.

Así pues, el resultado del comando `nslookup` debe tener el siguiente aspecto:
```dos
C:\>nslookup my-private-link-speech.cognitiveservices.azure.com
Server:  UnKnown
Address:  fe80::1

Non-authoritative answer:
Name:    apimgmthskquihpkz6d90kmhvnabrx3ms3pdubscpdfk1tsx3a.cloudapp.net
Address:  13.93.122.1
Aliases:  my-private-link-speech.cognitiveservices.azure.com
          westeurope.api.cognitive.microsoft.com
          cognitiveweprod.trafficmanager.net
          cognitiveweprod.azure-api.net
          apimgmttmdjylckcx6clmh2isu2wr38uqzm63s8n4ub2y3e6xs.trafficmanager.net
          cognitiveweprod-westeurope-01.regional.azure-api.net
```
Compárelo con el resultado de [esta sección](#resolve-dns-from-other-networks).

### <a name="speech-resource-with-a-custom-domain-name-and-without-private-endpoints-usage-with-the-rest-apis"></a>Recurso de voz con un nombre de dominio personalizado sin puntos de conexión privados: Uso con las API REST

#### <a name="speech-to-text-rest-api-v30"></a>API de REST de conversión de voz en texto v3.0

El uso de Speech-to-Text REST API v3.0 es completamente equivalente al caso de los [recursos de voz con puntos de conexión privados habilitados](#speech-to-text-rest-api-v30).

#### <a name="speech-to-text-rest-api-for-short-audio-and-text-to-speech-rest-api"></a>API de REST de conversión de voz en texto para audios breves y API de REST de conversión de texto a voz

En este caso, el uso de Speech-to-Text REST API para audios de corta duración y el uso de Text-to-Speech REST API no presentan diferencias con respecto al caso general; sin embargo, hay una excepción. (Consulte la nota siguiente). Ambas API deben usarse tal y como se describe en la documentación de [Speech-to-Text REST API para audios de corta duración](rest-speech-to-text.md#speech-to-text-rest-api-for-short-audio) y [Text-to-Speech REST API](rest-text-to-speech.md).

> [!NOTE]
> Si va a usar Speech-to-Text REST API para audios de corta duración y Text-to-Speech REST API en escenarios de dominios personalizados, utilice una clave de suscripción pasada a través del encabezado `Ocp-Apim-Subscription-Key`. (Más información sobre [Speech-to-text REST API para audios de corta duración](rest-speech-to-text.md#request-headers) y [Text-to-speech REST API](rest-text-to-speech.md#request-headers))
>
> Usar un token de autorización y pasarlo al punto de conexión especial a través del encabezado `Authorization` funcionará *solo* si ha activado la opción de acceso **Todas las redes** en la sección **Redes** del recurso de voz. En los demás casos, al intentar obtener un token de autorización recibirá el error `Forbidden` o `BadRequest`.

### <a name="speech-resource-with-a-custom-domain-name-and-without-private-endpoints-usage-with-the-speech-sdk"></a>Recurso de voz con un nombre de dominio personalizado sin puntos de conexión privados: Uso con el SDK de voz

El uso del SDK de voz con recursos de voz con dominios personalizados habilitados *sin* puntos de conexión privados es equivalente al caso general, tal y como se describe en la [documentación del SDK de voz](speech-sdk.md).

En caso de que haya modificado el código para usarlo con [recursos de voz con puntos de conexión privados habilitados](#speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk), tenga en cuenta lo siguiente.

En la sección sobre los [recursos de voz con puntos de conexión privados habilitados](#speech-resource-with-a-custom-domain-name-and-a-private-endpoint-usage-with-the-speech-sdk), hemos explicado cómo determinar la dirección URL del punto de conexión, cómo modificarla y hacer que funcione a través de la inicialización "desde el punto de conexión"/"con el punto de conexión" de la instancia de clase `SpeechConfig`.

Sin embargo, si intenta ejecutar la misma aplicación después de quitar todos los puntos de conexión privados (lo que deja tiempo para que se reaprovisione el registro DNS correspondiente), obtendrá un error de servicio interno (404). La razón es que el [registro DNS](#dns-configuration) apunta ahora al punto de conexión regional de Cognitive Services en lugar de al proxy de red virtual y ahí no se encuentran las rutas de acceso a la dirección URL, como `/stt/speech/recognition/conversation/cognitiveservices/v1?language=en-US`.

Debe revertir la aplicación a la creación de instancias estándar de `SpeechConfig` en el estilo del código siguiente:

```csharp
var config = SpeechConfig.FromSubscription(subscriptionKey, azureRegion);
```

[!INCLUDE [](includes/speech-vnet-service-enpoints-private-endpoints-simultaneously.md)]

## <a name="pricing"></a>Precios

Para más información sobre los precios, consulte [Precios de Azure Private Link](https://azure.microsoft.com/pricing/details/private-link).

## <a name="learn-more"></a>Más información

* [Uso del servicio de Voz a través de un punto de conexión de Virtual Network](speech-service-vnet-service-endpoint.md)
* [Azure Private Link](../../private-link/private-link-overview.md)
* [Punto de conexión del servicio de una red virtual de Azure](../../virtual-network/virtual-network-service-endpoints-overview.md)
* [Acerca del SDK de Voz](speech-sdk.md)
* [Speech-to-text REST API](rest-speech-to-text.md)
* [Text-to-speech REST API](rest-text-to-speech.md)
