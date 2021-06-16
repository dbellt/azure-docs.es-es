---
title: Integrar API Management con Service Fabric en Azure
description: Aprenda a usar rápidamente Azure API Management y a enrutar el tráfico a un servicio de back-end de Service Fabric.
ms.topic: conceptual
ms.date: 07/10/2019
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 6d72db8e1baf1109c222badb83c473c4defa6038
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110665093"
---
# <a name="integrate-api-management-with-service-fabric-in-azure"></a>Integrar API Management con Service Fabric en Azure

La implementación de Azure API Management con Service Fabric es un escenario avanzado.  API Management es útil cuando es necesario publicar API con un completo conjunto de reglas de enrutamiento para los servicios de Service Fabric de back-end. Las aplicaciones en la nube normalmente necesitan una puerta de enlace front-end para proporcionar un único punto de entrada para usuarios, dispositivos u otras aplicaciones. En Service Fabric, una puerta de enlace puede ser cualquier servicio sin estado diseñado para la entrada de tráfico, como una aplicación ASP.NET Core, Event Hubs, IoT Hub o Azure API Management.

En este artículo se muestra cómo configurar [Azure API Management](../api-management/api-management-key-concepts.md) con Service Fabric para enrutar el tráfico a un servicio de back-end de Service Fabric.  Cuando haya terminado, habrá implementado API Management en una red virtual y configurado una operación de API para enviar tráfico a servicios sin estado de back-end. Para más información sobre escenarios de Azure API Management con Service Fabric, consulte el artículo de [introducción](service-fabric-api-management-overview.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="availability"></a>Disponibilidad

> [!IMPORTANT]
> Esta característica ya está disponible en los niveles **Premium** y **Developer** de API Management debido a la compatibilidad requerida con redes virtuales.

## <a name="prerequisites"></a>Prerequisites

Antes de empezar:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Instale [Azure Powershell](/powershell/azure/install-az-ps) o la [CLI de Azure](/cli/azure/install-azure-cli).
* Cree un [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) en un grupo de seguridad de red.
* Si implementa un clúster de Windows, configure un entorno de desarrollo de Windows. Instale [Visual Studio 2019](https://www.visualstudio.com) y las cargas de trabajo de **desarrollo Azure**, **desarrollo web y ASP.NET** y **desarrollo a través de plataformas .NET Core**.  Después, configure un [entorno de desarrollo .NET](service-fabric-get-started.md).

## <a name="network-topology"></a>Topología de red

Ahora que tiene un [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md) seguro en Azure, implemente API Management en la red virtual (VNET), en la subred y en el grupo de seguridad de red diseñados para API Management. En este artículo la plantilla de Resource Manager para API Management está preconfigurada para usar los nombres de la red virtual, la subred y el grupo de seguridad de red que configuró en el [tutorial del clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md). Asimismo, este artículo implementa la siguiente topología en Azure, donde API Management y Service Fabric están en subredes de la misma red virtual:

 ![Leyenda de la imagen][sf-apim-topology-overview]

## <a name="sign-in-to-azure-and-select-your-subscription"></a>Inicio de sesión en Azure y selección de la suscripción

Inicie sesión en su cuenta de Azure y seleccione su suscripción antes de ejecutar comandos de Azure.

```powershell
Connect-AzAccount
Get-AzSubscription
Set-AzContext -SubscriptionId <guid>
```

```azurecli
az login
az account set --subscription <guid>
```

## <a name="deploy-a-service-fabric-back-end-service"></a>Implementación de un servicio de back-end de Service Fabric

Antes de configurar API Management para enrutar el tráfico a un servicio back-end de Service Fabric, primero necesita un servicio en ejecución que acepte solicitudes.  

Cree un servicio básico y confiable de ASP.NET Core sin estado, con la plantilla de proyecto de Web API predeterminada. Así se crea un punto de conexión HTTP para el servicio que se expone en Azure API Management.

Inicie Visual Studio como administrador y cree un servicio ASP.NET Core:

 1. En Visual Studio, seleccione Archivo -> Nuevo proyecto.
 2. En Nube, seleccione la plantilla de aplicación de Service Fabric y asígnele el nombre **"ApiApplication"** .
 3. Seleccione la plantilla de servicio de ASP.NET Core sin estado y denomine al proyecto **"WebApiService"** .
 4. Seleccione la plantilla de proyecto de ASP.NET Core 2.1 de API web.
 5. Una vez creado el proyecto, abra `PackageRoot\ServiceManifest.xml` y quite los atributos `Port` de la configuración del recurso de punto de conexión:

    ```xml
    <Resources>
      <Endpoints>
        <Endpoint Protocol="http" Name="ServiceEndpoint" Type="Input" />
      </Endpoints>
    </Resources>
    ```

    Al quitar el puerto, Service Fabric puede especificar un puerto dinámicamente del intervalo de puertos de la aplicación, abierta con el grupo de seguridad de red en la plantilla del clúster de Resource Manager, que permite que el tráfico fluya a él desde API Management.

 6. Presione F5 en Visual Studio para comprobar que la API web está disponible de manera local.

    Abra Service Fabric Explorer y vaya a una instancia específica del servicio ASP.NET Core para ver la dirección base donde escucha el servicio. Agregue `/api/values` a la dirección base y ábralo en un explorador. Esto invoca el método Get en ValuesController en la plantilla de Web API. Devuelve la respuesta predeterminada de la plantilla, una matriz JSON con dos cadenas:

    ```json
    ["value1", "value2"]`
    ```

    Este es el punto de conexión que debe exponer a través de API Management en Azure.

 7. Por último, implemente la aplicación en el clúster en Azure. In Visual Studio, haga clic con el botón derecho en el proyecto de la aplicación y seleccione **Publicar**. Proporcione el punto de conexión del clúster (por ejemplo, `mycluster.southcentralus.cloudapp.azure.com:19000`) para implementar la aplicación en el clúster de Service Fabric en Azure.

En el clúster de Service Fabric en Azure ahora se ejecutará un servicio sin estado ASP.NET Core denominado `fabric:/ApiApplication/WebApiService`.

## <a name="download-and-understand-the-resource-manager-templates"></a>Descarga e información de las plantillas de Resource Manager

Descargue y guarde las plantillas de Resource Manager y el archivo de parámetros siguientes:

* [network-apim.json][network-arm]
* [network-apim.parameters.json][network-parameters-arm]
* [apim.json][apim-arm]
* [apim.parameters.json][apim-parameters-arm]

La plantilla *network-apim.json* implementa un nuevo grupo de seguridad de red y una nueva subred en la red virtual donde se implementa el clúster de Service Fabric.

En las siguientes secciones se describen los recursos que se definen mediante la plantilla *apim.json*. Para más información, siga los vínculos a la documentación de referencia de plantilla dentro de cada sección. Los parámetros configurables que se definen en el archivo de parámetros *apim.parameters.json* se establecen más adelante en este artículo.

### <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

[Microsoft.ApiManagement/service](/azure/templates/microsoft.apimanagement/service) describe la instancia de servicio de API Management: nombre, SKU o nivel, ubicación del grupo de recursos, información del editor y red virtual.

### <a name="microsoftapimanagementservicecertificates"></a>Microsoft.ApiManagement/service/certificates

[Microsoft.ApiManagement/service/certificates](/azure/templates/microsoft.apimanagement/service/certificates) configura la seguridad de API Management. API Management debe autenticarse con el clúster de Service Fabric para la detección de servicios mediante un certificado de cliente con acceso al clúster. En este artículo se usa el mismo certificado que se especificó anteriormente al crear el [clúster de Windows](service-fabric-tutorial-create-vnet-and-windows-cluster.md#createvaultandcert_anchor) que, de forma predeterminada, se puede usar para tener acceso al clúster.

En este artículo se usa el mismo certificado para la autenticación de cliente y la seguridad de nodo a nodo del clúster. Puede utilizar otro certificado de cliente si tiene uno configurado para acceder al clúster de Service Fabric. Rellene los campos **name**, **password** y **data** (cadena codificada en base-64) del archivo de clave privada (.pfx) del certificado del clúster que especificó al crear el clúster de Service Fabric.

### <a name="microsoftapimanagementservicebackends"></a>Microsoft.ApiManagement/service/backends

[Microsoft.ApiManagement/service/backends](/azure/templates/microsoft.apimanagement/service/backends) describe el servicio  back-end al que se reenvía el tráfico.

En los servidores back-end de Service Fabric, su clúster sirve de back-end, en lugar de usar un servicio de Service Fabric específico. Esto permite que una única directiva enrute a más de un servicio del clúster. Aquí el campo **url** es el nombre completo de un servicio del clúster al que se enrutan todas las solicitudes de forma predeterminada si no se especifica ningún nombre de servicio en una directiva de back-end. Puede usar un nombre de servicio falso, como "fabric:/fake/service" si no desea un servicio de reserva. **resourceId** especifica el punto de conexión de administración del clúster.  **clientCertificateThumbprint** y **serverCertificateThumbprints** identifican los certificados usados para autenticarse con el clúster.

### <a name="microsoftapimanagementserviceproducts"></a>Microsoft.ApiManagement/service/products

[Microsoft.ApiManagement/service/products](/azure/templates/microsoft.apimanagement/service/products) crea un producto. En Azure API Management, un producto contiene una o varias API, así como una cuota de uso y los términos de uso. Una vez publicado un producto, los desarrolladores pueden suscribirse al producto y comenzar a usar las API del producto.

Rellene el campo **displayName** con un valor descriptivo y el campo **description** del producto. En este artículo, se necesita una suscripción, pero no es necesario que la apruebe un administrador.  El valor de **state** de este producto es "publicado" y es visible para los suscriptores.

### <a name="microsoftapimanagementserviceapis"></a>Microsoft.ApiManagement/service/apis

[Microsoft.ApiManagement/service/apis](/azure/templates/microsoft.apimanagement/service/apis) crea una API. En API Management, una API representa un conjunto de operaciones que puede invocarse por las aplicaciones cliente. Una vez agregadas las operaciones, la API se agrega a un producto y se puede publicar. Una vez publicada una API, pueden usarla y suscribirse a ella los desarrolladores.

* **displayName** puede ser cualquier nombre para la API. En este artículo usará "Service Fabric App".
* **name** proporciona un nombre único y descriptivo para la API, como "aplicación de service fabric". Se muestra en el portal para desarrolladores y en el portal del publicador.
* **serviceUrl** hace referencia al servicio HTTP que implementa la API. API Management envía las solicitudes a esta dirección. En servidores back-end de Service Fabric, este valor de dirección URL no se usa. Puede escribir aquí cualquier valor. En este artículo, por ejemplo "http:\//servicefabric".
* **path** se anexa a la dirección URL base del servicio API Management. La dirección URL base es común para todas las API hospedadas en una instancia del servicio API Management. API Management distingue las API por su sufijo, por lo que el sufijo debe ser único para cada API de un publicador determinado.
* El campo **protocols** determina los protocolos que se pueden usar para acceder a la API. En este artículo, se muestran **http** y **https**.
* **path** es un sufijo de la API. En este artículo deberá usar "myapp".

### <a name="microsoftapimanagementserviceapisoperations"></a>Microsoft.ApiManagement/service/apis/operations

[Microsoft.ApiManagement/service/apis/operations](/azure/templates/microsoft.apimanagement/service/apis/operations): antes de usar una API de API Management, se deben agregar operaciones a la API.  Los clientes externos usan una operación para comunicarse con el servicio sin estado de ASP.NET Core que se ejecuta en el clúster de Service Fabric.

Para agregar una operación de API de front-end, rellene los valores:

* **displayName** y **description** describen la operación. En este artículo deberá usar "Values".
* **method** especifica el verbo HTTP.  En este artículo deberá especificar **GET**.
* **urlTemplate** se anexa a la dirección URL base de la API e identifica una única operación HTTP.  En este artículo deberá usar `/api/values` si agregó el servicio back-end de .NET, o `getMessage` si agregó el servicio back-end de Java.  De forma predeterminada, la ruta de acceso URL que se especifica aquí se envía al servicio de back-end de Service Fabric. Si usa aquí la misma ruta de acceso URL que usa el servicio, como "/api/values", la operación funciona sin más modificaciones. También puede especificar aquí una ruta de acceso URL distinta de la que usa el servicio de back-end de Service Fabric, en cuyo caso también debe especificar después una ruta de acceso de reescritura en la directiva de la operación.

### <a name="microsoftapimanagementserviceapispolicies"></a>Microsoft.ApiManagement/service/apis/policies

[Microsoft.ApiManagement/service/apis/policies](/azure/templates/microsoft.apimanagement/service/apis/policies) crea una directiva de back-end, que vincula todos los elementos. Es donde se configura el servicio de back-end de Service Fabric al que se enrutan las solicitudes. Puede aplicar esta directiva a cualquier operación de API.  Para más información, consulte [Introducción a las directivas](../api-management/api-management-howto-policies.md).

La [configuración de back-end de Service Fabric](../api-management/api-management-transformation-policies.md#SetBackendService) proporciona los controles de enrutamiento de solicitudes siguientes:

* Selección de instancias de servicio mediante la especificación de un nombre de instancia de servicio de Service Fabric, ya sea codificado de forma rígida (por ejemplo, `"fabric:/myapp/myservice"`) o generado a partir de la solicitud HTTP (por ejemplo, `"fabric:/myapp/users/" + context.Request.MatchedParameters["name"]`).
* Resolución de la partición mediante la generación de una clave de partición a partir de cualquier esquema de partición de Service Fabric.
* Selección de réplicas para los servicios con estado.
* Condiciones de reintento de resolución que permiten especificar las condiciones para volver a resolver una ubicación de servicio y volver a enviar una solicitud.

**policyContent** es el contenido XML con escape JSON de la directiva.  En este artículo deberá crear una directiva de back-end que enrute las solicitudes directamente al servicio sin estado .NET o Java implementado anteriormente. Agregue una directiva `set-backend-service` bajo las directivas de entrada.  Reemplace el valor de *sf-service-instance-name* por `fabric:/ApiApplication/WebApiService`, si anteriormente implementó el servicio back-end de .NET, o por `fabric:/EchoServerApplication/EchoServerService`, si implementó el servicio Java.  *backend-id* hace referencia a un recurso de back-end, en este caso el recurso `Microsoft.ApiManagement/service/backends` definido en la plantilla *apim.json*. *backend-id* también puede hacer referencia a otro recurso de back-end creado mediante las API de API Management. En este artículo deberá establecer *backend-id* en el valor del parámetro *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Para obtener un conjunto completo de los atributos de directiva de back-end de Service Fabric, consulte la [documentación de back-end de API Management](../api-management/api-management-transformation-policies.md#SetBackendService)

## <a name="set-parameters-and-deploy-api-management"></a>Definición de los parámetros e implementación de API Management

Rellene los siguientes parámetros vacíos en *apim.parameters.json* para la implementación.

|Parámetro|Value|
|---|---|
|apimInstanceName|sf-apim|
|apimPublisherEmail|myemail@contosos.com|
|apimSku|Desarrollador|
|serviceFabricCertificateName|sfclustertutorialgroup320171031144217|
|certificatePassword|q6D7nN%6ck@6|
|serviceFabricCertificateThumbprint|C4C1E541AD512B8065280292A8BA6079C3F26F10 |
|serviceFabricCertificate|&lt;cadena codificada en base 64&gt;|
|url_path|/api/values|
|clusterHttpManagementEndpoint|`https://mysfcluster.southcentralus.cloudapp.azure.com:19080`|
|inbound_policy|&lt;Cadena XML&gt;|

*certificatePassword* y *serviceFabricCertificateThumbprint* deben coincidir con el certificado usado para configurar el clúster.

*serviceFabricCertificate* es el certificado como una cadena codificada en base 64, que se puede generar mediante el siguiente script:

```powershell
$bytes = [System.IO.File]::ReadAllBytes("C:\mycertificates\sfclustertutorialgroup220171109113527.pfx");
$b64 = [System.Convert]::ToBase64String($bytes);
[System.Io.File]::WriteAllText("C:\mycertificates\sfclustertutorialgroup220171109113527.txt", $b64);
```

En *inbound_policy*, reemplace el valor de *sf-service-instance-name* por `fabric:/ApiApplication/WebApiService` si implementó antes el servicio de back-end de .NET, o por `fabric:/EchoServerApplication/EchoServerService` si implementó el servicio Java. *backend-id* hace referencia a un recurso de back-end, en este caso el recurso `Microsoft.ApiManagement/service/backends` definido en la plantilla *apim.json*. *backend-id* también puede hacer referencia a otro recurso de back-end creado mediante las API de API Management. En este artículo deberá establecer *backend-id* en el valor del parámetro *service_fabric_backend_name*.

```xml
<policies>
  <inbound>
    <base/>
    <set-backend-service
        backend-id="servicefabric"
        sf-service-instance-name="service-name"
        sf-resolve-condition="@(context.LastError?.Reason == "BackendConnectionFailure")" />
  </inbound>
  <backend>
    <base/>
  </backend>
  <outbound>
    <base/>
  </outbound>
</policies>
```

Use el siguiente script para implementar la plantilla de Resource Manager y los archivos de parámetros para API Management:

```powershell
$groupname = "sfclustertutorialgroup"
$clusterloc="southcentralus"
$templatepath="C:\clustertemplates"

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\network-apim.json" -TemplateParameterFile "$templatepath\network-apim.parameters.json" -Verbose

New-AzResourceGroupDeployment -ResourceGroupName $groupname -TemplateFile "$templatepath\apim.json" -TemplateParameterFile "$templatepath\apim.parameters.json" -Verbose
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az deployment group create --name ApiMgmtNetworkDeployment --resource-group $ResourceGroupName --template-file network-apim.json --parameters @network-apim.parameters.json

az deployment group create --name ApiMgmtDeployment --resource-group $ResourceGroupName --template-file apim.json --parameters @apim.parameters.json
```

## <a name="test-it"></a>Pruébelo

Ahora puede intentar enviar una solicitud al servicio back-end de Service Fabric a través de API Management directamente desde [Azure Portal](https://portal.azure.com).

 1. En el servicio API Management, seleccione **API**.
 2. En la API de **Service Fabric App** que creó en los pasos anteriores, seleccione la pestaña **Prueba** y, a continuación, la operación **Values**.
 3. Haga clic en el botón **Enviar** para enviar una solicitud de prueba al servicio de back-end.  La respuesta HTTP debe ser similar a la siguiente:

    ```http
    HTTP/1.1 200 OK

    Transfer-Encoding: chunked

    Content-Type: application/json; charset=utf-8

    Vary: Origin

    Ocp-Apim-Trace-Location: https://apimgmtstodhwklpry2xgkdj.blob.core.windows.net/apiinspectorcontainer/PWSQOq_FCDjGcaI1rdMn8w2-2?sv=2015-07-08&sr=b&sig=MhQhzk%2FEKzE5odlLXRjyVsgzltWGF8OkNzAKaf0B1P0%3D&se=2018-01-28T01%3A04%3A44Z&sp=r&traceId=9f8f1892121e445ea1ae4d2bc8449ce4

    Date: Sat, 27 Jan 2018 01:04:44 GMT


    ["value1", "value2"]
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Un clúster está formado por muchos otros recursos de Azure, además del propio recurso del clúster. La manera más sencilla de eliminar el clúster y todos los recursos que consume es eliminar el grupo de recursos.

Inicie sesión en Azure y seleccione el identificador de suscripción con el que quiere quitar el clúster.  Para encontrar el identificador de suscripción, inicie sesión en [Azure Portal](https://portal.azure.com). Elimine el grupo de recursos y todos los recursos del clúster mediante el cmdlet [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup).

```powershell
$ResourceGroupName = "sfclustertutorialgroup"
Remove-AzResourceGroup -Name $ResourceGroupName -Force
```

```azurecli
ResourceGroupName="sfclustertutorialgroup"
az group delete --name $ResourceGroupName
```

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de cómo usar [API Management](../api-management/import-and-publish.md).

También puede usar [Azure Portal](../api-management/how-to-configure-service-fabric-backend.md) para crear y administrar back-ends de Service Fabric para API Management.

[azure-powershell]: /powershell/azure/

[apim-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.json
[apim-parameters-arm]:https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/apim.parameters.json

[network-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.json
[network-parameters-arm]: https://github.com/Azure/service-fabric-scripts-and-templates/blob/master/templates/service-integration/network-apim.parameters.json

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png

<!-- pics -->
[sf-apim-topology-overview]: ./media/service-fabric-tutorial-deploy-api-management/sf-apim-topology-overview.png
