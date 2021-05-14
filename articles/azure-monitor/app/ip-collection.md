---
title: Colección de direcciones IP de Azure Application Insights | Microsoft Docs
description: Información acerca de cómo controlar las direcciones IP y la ubicación geográfica con Azure Application Insights
ms.topic: conceptual
ms.date: 09/23/2020
ms.custom: devx-track-js, devx-track-azurepowershell
ms.openlocfilehash: d5bacde864d42dfefab9c4b0d5dc90072081a25f
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108321364"
---
# <a name="geolocation-and-ip-address-handling"></a>Administración de la ubicación geográfica y la dirección IP

En este artículo se explica cómo funcionan la búsqueda de geolocalización y la administración de direcciones IP en Application Insights, además de cómo modificar el comportamiento predeterminado.

## <a name="default-behavior"></a>Comportamiento predeterminado

De forma predeterminada, las direcciones IP se recopilan temporalmente, pero no se almacenan en Application Insights. El proceso básico es el siguiente:

Cuando se envía telemetría a Azure, la dirección IP se usa para realizar una búsqueda de geolocalización mediante [GeoLite2 de MaxMind](https://dev.maxmind.com/geoip/geoip2/geolite2/). Los resultados de esta búsqueda se usan para rellenar los campos `client_City`, `client_StateOrProvince` y `client_CountryOrRegion`. Luego, la dirección se descarta y se escribe `0.0.0.0` en el campo `client_IP`.

* Telemetría del explorador: recopilamos temporalmente la dirección IP del remitente. La dirección IP se calcula mediante el punto de conexión de ingesta.
* Telemetría del servidor: el módulo de telemetría de Application Insights recopila temporalmente la dirección IP del cliente. La dirección IP no se recopila localmente cuando se establece el encabezado `X-Forwarded-For`.

Este comportamiento es así de forma predeterminada, ya que le ayudará a evitar la recopilación innecesaria de datos personales. Siempre que sea posible, se recomienda evitar la recopilación de datos personales. 

## <a name="overriding-default-behavior"></a>Invalidar el comportamiento predeterminado

Aunque el valor predeterminado es no recopilar direcciones IP, todavía ofrecemos la flexibilidad para invalidar este comportamiento. Sin embargo, se recomienda comprobar que la recopilación no infrinja ningún requisito de cumplimiento o normativa local. 

Para obtener más información sobre la administración de datos personales en Application Insights, consulte la [guía de datos personales](../logs/personal-data-mgmt.md).

## <a name="storing-ip-address-data"></a>Almacenamiento de datos de direcciones IP

Para habilitar la recopilación y el almacenamiento de direcciones IP, la propiedad `DisableIpMasking` del componente de Application Insights debe establecerse en `true`. Esta propiedad se puede establecer mediante las plantillas de Azure Resource Manager o llamando a la API REST. 

### <a name="azure-resource-manager-template"></a>Plantilla de Azure Resource Manager

```json
{
       "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/microsoft.insights/components/<resource-name>",
       "name": "<resource-name>",
       "type": "microsoft.insights/components",
       "location": "westcentralus",
       "tags": {
              
       },
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "Flow_Type": "Redfield",
              "Request_Source": "IbizaAIExtension",
              // ...
              "DisableIpMasking": true
       }
}
```

### <a name="portal"></a>Portal 

Si solo necesita modificar el comportamiento de un recurso de Application Insights, use Azure Portal. 

1. Vaya al recurso de Application Insights > **Automation** > **Exportar plantilla**. 

2. Seleccione **Implementar**.

    ![Botón con la palabra "Implementar" resaltada en rojo](media/ip-collection/deploy.png)

3. Seleccione **Editar plantilla**.

    ![Botón con la palabra "Editar" resaltada en rojo](media/ip-collection/edit-template.png)

4. Realice los siguientes cambios en el archivo JSON del recurso y, luego, seleccione **Guardar**:

    ![La captura de pantalla agrega una coma después de "IbizaAIExtension" y agrega una nueva línea a continuación con "DisableIpMasking": true.](media/ip-collection/save.png)

    > [!WARNING]
    > Si experimenta un error que dice: **_El grupo de recursos está en una ubicación que no es compatible con uno o varios recursos de la plantilla. Elija otro grupo de recursos._** Seleccione temporalmente otro grupo de recursos en la lista desplegable y, a continuación, vuelva a seleccionar el grupo de recursos original para resolver el error.

5. Seleccione **Acepto** > **Comprar**. 

    ![Casilla con las palabras "Acepto los términos y condiciones indicados anteriormente" resaltadas en rojo encima de un botón con la palabra "Compra" resaltada en rojo.](media/ip-collection/purchase.png)

    En este caso, no se compra nada nuevo. Solo se va a actualizar la configuración del recurso de Application Insights existente.

6. Una vez finalizada la implementación, se registrarán nuevos datos de telemetría.

    Si selecciona y edita de nuevo la plantilla, solo verá la plantilla predeterminada sin la propiedad recién agregada. Si no ve los datos de dirección IP y quiere confirmar que `"DisableIpMasking": true` está establecido, ejecute el siguiente comando de PowerShell: 
    
    ```powershell
    # Replace `Fabrikam-dev` with the appropriate resource and resource group name.
    # If you aren't using the cloud shell you will need to connect to your Azure account
    # Connect-AzAccount 
    $AppInsights = Get-AzResource -Name 'Fabrikam-dev' -ResourceType 'microsoft.insights/components' -ResourceGroupName 'Fabrikam-dev'
    $AppInsights.Properties
    ```
    
    Como resultado, se devolverá una lista de propiedades. Una de las propiedades debe ser `DisableIpMasking: true`. Si ejecuta el comando de PowerShell antes de implementar la nueva propiedad con Azure Resource Manager, la propiedad no existirá.

### <a name="rest-api"></a>API de REST

La carga útil de la [API de Rest](/rest/api/azure/) para realizar las mismas modificaciones es la siguiente:

```
PATCH https://management.azure.com/subscriptions/<sub-id>/resourceGroups/<rg-name>/providers/microsoft.insights/components/<resource-name>?api-version=2018-05-01-preview HTTP/1.1
Host: management.azure.com
Authorization: AUTH_TOKEN
Content-Type: application/json
Content-Length: 54

{
       "location": "<resource location>",
       "kind": "web",
       "properties": {
              "Application_Type": "web",
              "DisableIpMasking": true
       }
}
```

## <a name="telemetry-initializer"></a>Inicializador de telemetría

Si necesita una alternativa más flexible que `DisableIpMasking`, puede usar un [inicializador de telemetría](./api-filtering-sampling.md#addmodify-properties-itelemetryinitializer) para copiar todo o parte de la dirección IP en un campo personalizado. 

# <a name="net"></a>[.NET](#tab/net)

### <a name="aspnet--aspnet-core"></a>ASP.NET/ASP.NET Core

```csharp
using Microsoft.ApplicationInsights.Channel;
using Microsoft.ApplicationInsights.DataContracts;
using Microsoft.ApplicationInsights.Extensibility;

namespace MyWebApp
{
    public class CloneIPAddress : ITelemetryInitializer
    {
        public void Initialize(ITelemetry telemetry)
        {
            ISupportProperties propTelemetry = telemetry as ISupportProperties;

            if (propTelemetry !=null && !propTelemetry.Properties.ContainsKey("client-ip"))
            {
                string clientIPValue = telemetry.Context.Location.Ip;
                propTelemetry.Properties.Add("client-ip", clientIPValue);
            }
        }
    } 
}
```

> [!NOTE]
> Si no puede acceder a `ISupportProperties`, asegúrese de que ejecuta la versión estable más reciente del SDK de Application Insights. `ISupportProperties` están pensadas para valores de cardinalidad alta, mientras que `GlobalProperties` son más adecuadas para valores de cardinalidad baja, como el nombre de la región, el nombre del entorno, etc. 

### <a name="enable-telemetry-initializer-for-aspnet"></a>Habilitar el inicializador de telemetría para .ASP.NET

```csharp
using Microsoft.ApplicationInsights.Extensibility;


namespace MyWebApp
{
    public class MvcApplication : System.Web.HttpApplication
    {
        protected void Application_Start()
        {
              //Enable your telemetry initializer:
              TelemetryConfiguration.Active.TelemetryInitializers.Add(new CloneIPAddress());
        }
    }
}

```

### <a name="enable-telemetry-initializer-for-aspnet-core"></a>Habilitar el inicializador de telemetría para ASP.NET Core

Puede crear su inicializador de telemetría de la misma manera en ASP.NET Core y en ASP.NET, pero para habilitar el inicializador, use el siguiente ejemplo como referencia:

```csharp
 using Microsoft.ApplicationInsights.Extensibility;
 using CustomInitializer.Telemetry;
 public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton<ITelemetryInitializer, CloneIPAddress>();
}
```
# <a name="nodejs"></a>[Node.js](#tab/nodejs)

### <a name="nodejs"></a>Node.js

```javascript
appInsights.defaultClient.addTelemetryProcessor((envelope) => {
    const baseData = envelope.data.baseData;
    if (appInsights.Contracts.domainSupportsProperties(baseData)) {
        const ipAddress = envelope.tags[appInsights.defaultClient.context.keys.locationIp];
        if (ipAddress) {
            baseData.properties["client-ip"] = ipAddress;
        }
    }
});
```
# <a name="client-side-javascript"></a>[JavaScript del lado cliente](#tab/javascript)

### <a name="client-side-javascript"></a>JavaScript del lado cliente

A diferencia de los SDK del lado servidor, el SDK de JavaScript del lado cliente no calcula la dirección IP. De forma predeterminada, el cálculo de la dirección IP para la telemetría del lado cliente tiene lugar en el punto de conexión de ingesta de Azure. 

Si quiere calcular la dirección IP directamente en el lado cliente, necesitará agregar su propia lógica personalizada y usar el resultado para establecer la etiqueta `ai.location.ip`. Cuando se establece `ai.location.ip`, el punto de conexión de ingesta no realiza el cálculo de la dirección IP, y la dirección IP proporcionada se usa para la búsqueda de geolocalización. En este caso, la dirección IP seguirá estableciéndose en cero de forma predeterminada. 

Para conservar la dirección IP completa calculada a partir de su lógica personalizada, puede usar un inicializador de telemetría que copie los datos de la dirección IP que proporcionó en `ai.location.ip` en un campo personalizado aparte. Sin embargo, de nuevo, a diferencia de los SDK del lado servidor, sin depender de bibliotecas de terceros o de su propia lógica de recopilación de IP del lado cliente, el SDK del lado cliente no calculará la dirección.    


```javascript
appInsights.addTelemetryInitializer((item) => {
    const ipAddress = item.tags && item.tags["ai.location.ip"];
    if (ipAddress) {
        item.baseData.properties = {
            ...item.baseData.properties,
            "client-ip": ipAddress
        };
    }
});

```  

Si los datos del lado cliente atraviesan un proxy antes de reenviarse al punto de conexión de ingesta, el cálculo de la dirección IP podría mostrar la dirección IP del proxy y no la del cliente. 

---

### <a name="view-the-results-of-your-telemetry-initializer"></a>Ver los resultados del inicializador de telemetría

Si envía nuevo tráfico al sitio y espera unos minutos, puede ejecutar una consulta para confirmar que la recopilación funciona:

```kusto
requests
| where timestamp > ago(1h) 
| project appName, operation_Name, url, resultCode, client_IP, customDimensions.["client-ip"]
```

Las direcciones IP recién recopiladas aparecerán en la columna `customDimensions_client-ip`. La columna `client-ip` predeterminada seguirá teniendo los cuatro octetos establecidos en cero. 

Si se prueba desde localhost y el valor de `customDimensions_client-ip` es `::1`, este valor es el comportamiento esperado. `::1` representa la dirección de bucle invertido en IPv6. Es equivalente a `127.0.0.1` en IPv4.

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre la [recopilación de datos personales](../logs/personal-data-mgmt.md) en Application Insights.

* Obtenga más información sobre cómo funciona la [recopilación de direcciones IP](https://apmtips.com/posts/2016-07-05-client-ip-address/) en Application Insights. (Este artículo es una entrada de blog externa antigua que escribió uno de nuestros ingenieros. Es anterior al comportamiento predeterminado actual donde la dirección IP se registra como `0.0.0.0`, pero se profundiza en la mecánica del elemento `ClientIpHeaderTelemetryInitializer` integrado).
