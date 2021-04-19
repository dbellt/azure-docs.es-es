---
title: Atribución del uso de Azure por parte de los clientes
description: Conozca en líneas generales el seguimiento del uso por parte de los clientes de las aplicaciones de Azure del marketplace comercial y de otra IP implementable desarrollada por los asociados.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: cpercy737
ms.author: camper
ms.date: 04/12/2021
ms.custom: devx-track-terraform
ms.openlocfilehash: 7f09e744dfdd7be73c62c70bcfa91804f14aaa68
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107313681"
---
# <a name="azure-customer-usage-attribution"></a>Atribución del uso de Azure por parte de los clientes

La atribución del uso por los clientes asocia el uso de los recursos de Azure en las suscripciones de cliente creadas al implementar la IP como asociado. La formación de estas asociaciones en sistemas internos de Microsoft aporta mayor visibilidad de la superficie de Azure que ejecuta el software. En las [ofertas de aplicación de Azure del marketplace comercial](#commercial-marketplace-azure-apps), esta funcionalidad de seguimiento le ayuda a mantenerse en consonancia con los equipos de ventas de Microsoft y obtener créditos por los programas de asociados de Microsoft. La atribución de uso por parte de los clientes no es aplicable a las [ofertas de máquinas virtuales de Azure en el marketplace comercial](./azure-vm-create.md). No es necesario que un anunciante del marketplace haga nada con las ofertas de máquinas virtuales para asegurarse de que se realiza un seguimiento de su consumo de Azure en las suscripciones de los clientes finales.

La atribución de uso del cliente admite tres opciones de implementación:

1. Plantillas de Azure Resource Manager (los fundamentos comunes de las aplicaciones de Azure, que también se conocen como "plantillas de solución" o "aplicaciones administradas" en el marketplace comercial): los asociados crean plantillas de Resource Manager para definir la infraestructura y la configuración de sus soluciones de Azure. Una plantilla de Resource Manager permite a los clientes implementar los recursos de la solución en un estado coherente y repetible.
1. API de Azure Resource Manager: los asociados pueden llamar a las API de Azure Resource Manager para implementar una plantilla de Azure Resource Manager o para aprovisionar directamente los servicios de Azure.
1. Terraform: los asociados pueden usar Terraform para implementar una plantilla de Resource Manager o implementar servicios de Azure directamente.

Existen casos de uso secundarios de la atribución del uso por parte de los clientes fuera del marketplace comercial que se describen [más adelante en este artículo](#other-use-cases).

>[!IMPORTANT]
>- La atribución del uso por parte de los clientes no tiene como fin realizar un seguimiento del trabajo de los integradores de sistemas, los proveedores de servicios administrados o las herramientas diseñadas para implementar y administrar recursos de Azure.
>- Va dirigida a nuevas implementaciones y no admite el seguimiento de recursos que ya se hayan implementado.
>- No todos los servicios de Azure son compatibles con la atribución de uso del cliente. Azure Kubernetes Services (AKS), Virtual Machine Scale Sets y Azure Batch tienen problemas conocidos que hacen que no se notifique todo el uso.

## <a name="commercial-marketplace-azure-apps"></a>Aplicaciones de Azure del marketplace comercial

El seguimiento del uso de Azure de las aplicaciones de Azure publicadas en el marketplace comercial es, en gran medida, automático. Al cargar una plantilla de Resource Manager como parte de la [configuración técnica del plan de la aplicación de Azure del marketplace](./create-new-azure-apps-offer-solution.md#define-the-technical-configuration), el Centro de partners agregará un identificador de seguimiento legible por Azure Resource Manager.

Si usa las API de Azure Resource Manager, deberá agregar el identificador de seguimiento [según las instrucciones que se indican a continuación](#use-resource-manager-apis) para pasarlo a Azure Resource Manager a medida que el código implementa los recursos. Este identificador es visible en el Centro de partners en la página de configuración técnica del plan. 

> [!NOTE]
> En el caso de las aplicaciones de Azure existentes, comenzó una migración puntual en marzo de 2021 para actualizar los identificadores de seguimiento en la configuración técnica de cada plan. El uso de implementaciones anteriores de esas ofertas quedará rastreado en los sistemas de Microsoft.
>
>Al actualizar las ofertas, ya no es necesario agregar el tipo de recurso **Microsoft.Resources/deployments** en el archivo de plantilla principal.

## <a name="other-use-cases"></a>Otros casos de uso 

Puede usar la atribución del uso por parte de los clientes para realizar un seguimiento del uso de Azure de las soluciones que no están disponibles en el marketplace comercial. Estas soluciones suelen residir en el repositorio de inicios rápidos o en los repositorios de GitHub privados, o pueden provenir de las interacciones de clientes 1:1 que crean una IP duradera (como una aplicación implementable y escalable).

Son necesarios varios pasos manuales:

1. Cree uno o varios GUID para usarlos como identificadores de seguimiento.
1. Registre esos GUID en el Centro de partners.
1. Agregue los GUID registrados a la aplicación de Azure o a las cadenas del agente de usuario.

### <a name="create-guids"></a>Creación de los identificadores únicos globales

A diferencia de los identificadores de seguimiento que el Centro de partners crea en su nombre para las aplicaciones de Azure del marketplace comercial, otros usos de CUA requieren la creación de un GUI como identificador de seguimiento. Un GUID es un identificador de referencia único que tiene 32 dígitos hexadecimales. Para crear GUID de seguimiento, debe usar un generador de GUID, por ejemplo, PowerShell:

```powershell
[guid]::NewGuid()
```

Debe crear un GUID único para cada producto y canal de distribución. Puede optar por usar un solo GUID en los múltiples canales de distribución del producto si no quiere que los informes se dividan. Los informes se generan mediante un identificador y un GUID de Microsoft Partner Network.

### <a name="register-guids"></a>Registro de GUID

Los GUID deben registrarse en el Centro de partners para que se puedan asociar a usted como asociado:

1. Inicie sesión en el [Centro de partners](https://partner.microsoft.com/dashboard).

1. Regístrese como [publicador comercial de Marketplace](https://aka.ms/JoinMarketplace).

1. Seleccione **Configuración** (icono de engranaje) en la esquina superior derecha y, luego, **Configuración de cuenta**.

1. En **Perfil de la organización** > **Identificadores** > **Add Tracking GUID** (Agregar GUID de seguimiento).

1. En el cuadro **GUID**, escriba su identificador único global de seguimiento. Escriba solo el GUID, sin el prefijo `pid-`. En el cuadro **Descripción**, escriba el nombre o la descripción de la solución.

1. Para registrar varios identificadores únicos globales, vuelva a seleccionar **Add Tracking GUID** (Agregar GUID de seguimiento). Aparecen más cuadros en la página.

1. Seleccione **Guardar**.

### <a name="add-a-guid-to-a-resource-manager-template"></a>Adición de un GUID a una plantilla de Resource Manager

Para agregar el GUID registrado a una plantilla de Resouce Manager, realice una única modificación en el archivo de plantilla principal:

1. Abra la plantilla de Resource Manager.

1. Agregue un nuevo recurso de tipo [Microsoft.Resources/deployments](/azure/templates/microsoft.resources/deployments) en el archivo de plantilla principal. El recurso solo debe estar en los archivos **mainTemplate.json** o **azuredeploy.json**, no en ninguna de las plantillas vinculadas o anidadas.

1. Como nombre del recurso, escriba el valor de GUID después del prefijo `pid-`. Por ejemplo, si el GUID es eb7927c8-dd66-43e1-b0cf-c346a422063, el nombre del recurso será **pid-eb7927c8-dd66-43e1-b0cf-c346a422063**. Ejemplo:
 
```json
{ // add this resource to the resources section in the mainTemplate.json
    "apiVersion": "2020-06-01",
    "name": "pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX", // use your generated GUID here
    "type": "Microsoft.Resources/deployments",
    "properties": {
        "mode": "Incremental",
        "template": {
            "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
            "contentVersion": "1.0.0.0",
            "resources": []
        }
    }
} // remove all comments from the file when complete
```
4. Comprobación de errores de la plantilla.

1. Vuelva a publicar la plantilla en los repositorios adecuados.

1. [Compruebe que el GUID es correcto en la implementación de plantillas](#verify-deployments-tracked-with-a-guid).

> [!TIP]
> Para más información sobre la creación y publicación de plantillas de Resource Manager, consulte [Creación e implementación de la primera plantilla de Resouce Manager](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md).

### <a name="verify-deployments-tracked-with-a-guid"></a>Comprobación de las implementaciones rastreadas con un GUID

Tras modificar la plantilla y realizar una implementación de prueba, use el siguiente script de PowerShell para recuperar los recursos que implementó y etiquetó.

Puede usar dicho script para comprobar que el GUID se ha agregado correctamente a la plantilla de Resource Manager. El script no se aplica a la implementación de la API de Resource Manager ni Terraform.

Inicie sesión en Azure. Antes de ejecutar el script, seleccione la suscripción con la implementación que quiere comprobar. Ejecute el script en el contexto de la suscripción de la implementación.

El **GUID** (llamado a continuación "deploymentName") y el nombre de **resourceGroup** de la implementación son parámetros necesarios.

[El script original](https://gist.github.com/bmoore-msft/ae6b8226311014d6e7177c5127c7eba1) se puede obtener en GitHub.

```powershell
Param(
    [string][Parameter(Mandatory=$true)]$deploymentName, # the full name of the deployment, e.g. pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
    [string][Parameter(Mandatory=$true)]$resourceGroupName
)

# Get the correlationId of the named deployment
$correlationId = (Get-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -Name "$deploymentName").correlationId

# Find all deployments with that correlationId
$deployments = Get-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName | Where-Object{$_.correlationId -eq $correlationId}

# Find all deploymentOperations in all deployments with that correlationId as PowerShell doesn't surface outputResources on the deployment or correlationId on the deploymentOperation

foreach ($deployment in $deployments){
    # Get deploymentOperations by deploymentName
    # then the resourceIds for each resource
    ($deployment | Get-AzResourceGroupDeploymentOperation | Where-Object{$_.targetResource -notlike "*Microsoft.Resources/deployments*"}).TargetResource
}
```

### <a name="notify-your-customers"></a>Notificación a los clientes

Los asociados deben informar a sus clientes de las implementaciones que usan atribución de uso del cliente. Microsoft notifica al asociado el uso de Azure vinculado a estas implementaciones. Los ejemplos siguientes incluyen contenido que puede usar para notificar a los clientes estas implementaciones. En los ejemplos, reemplace \<PARTNER> por el nombre de la empresa. Los asociados deben asegurarse de que la notificación esté en consonancia con sus directivas de privacidad y recopilación de datos, incluidas las opciones para que los clientes se puedan excluir del seguimiento.

#### <a name="notification-for-resource-manager-template-deployments"></a>Notificación de implementaciones de plantillas de Resource Manager

Al implementar esta plantilla, Microsoft puede identificar la instalación del software de \<PARTNER> con los recursos de Azure implementados. Microsoft puede correlacionar estos recursos usados para admitir el software. Microsoft recopila esta información para proporcionar las mejores experiencias con sus productos y conseguir que sus negocios funcionen. Los datos se recopilan y regulan en función de las directivas de privacidad de Microsoft, que se encuentran en [https://www.microsoft.com/trustcenter](https://www.microsoft.com/trustcenter).

#### <a name="notification-for-sdk-or-api-deployments"></a>Notificación para las implementaciones de SDK o API

Al implementar el software de \<PARTNER>, Microsoft puede identificar la instalación de dicho software con los recursos de Azure implementados. Microsoft puede correlacionar estos recursos usados para admitir el software. Microsoft recopila esta información para proporcionar las mejores experiencias con sus productos y conseguir que sus negocios funcionen. Los datos se recopilan y regulan en función de las directivas de privacidad de Microsoft, que se encuentran en [https://www.microsoft.com/trustcenter](https://www.microsoft.com/trustcenter).

## <a name="use-resource-manager-apis"></a>Uso de las API de Resource Manager

En algunos casos, es posible realizar llamadas directamente a las API REST de Resource Manager para implementar los servicios de Azure. [Azure admite varios SDK](../index.yml?pivot=sdkstools) para habilitar estas llamadas. Puede usar uno de los SDK o llamar a las API REST directamente para implementar los recursos.

Para habilitar la atribución del uso por parte de los clientes, al diseñar las llamadas API, incluya el identificador de seguimiento en el encabezado del agente de usuario de la solicitud. Aplique formato a la cadena con el prefijo `pid-`. Ejemplos:

```xml
//Commercial Marketplace Azure app
pid-contoso-myoffer-partnercenter //copy the tracking ID exactly as it appears in Partner Center

//Other use cases
pid-b6addd8f-5ff4-4fc0-a2b5-0ec7861106c4 //enter your GUID after "pid-"
```
> [!IMPORTANT]
> Si va a usar las API de Resource Manager con una aplicación de Azure del marketplace comercial, utilice el identificador de seguimiento proporcionado en el Centro de partners. No use un GUID.

Los diversos SDK interactúan de forma diferente con las API de Resource Manager y requerirán algunas diferencias en el código. Los ejemplos siguientes presentan el enfoque de las soluciones no relacionadas con el marketplace comercial mediante un GUID y abarcan varios de los SDK de Azure conocidos.

#### <a name="example-python-sdk"></a>Ejemplo: SDK de Python

Para Python, use el atributo **config**. El atributo solo se puede agregar a un UserAgent. Ejemplo:

```python
client = azure.mgmt.servicebus.ServiceBusManagementClient(**parameters)
client.config.add_user_agent("pid-b6addd8f-5ff4-4fc0-a2b5-0ec7861106c4")
```
> [!IMPORTANT]
> Agregue el atributo a cada cliente. No hay ninguna configuración estática global. Puede etiquetar una fábrica de clientes para asegurarse de que todos los clientes realizan el seguimiento. Para más información, consulte este [ejemplo de fábrica de cliente en GitHub](https://github.com/Azure/azure-cli/blob/7402fb2c20be2cdbcaa7bdb2eeb72b7461fbcc30/src/azure-cli-core/azure/cli/core/commands/client_factory.py#L70-L79).

#### <a name="example-net-sdk"></a>Ejemplo: SDK de .NET

Para .NET, asegúrese de establecer el agente de usuario. Use la biblioteca [Microsoft.Azure.Management.Fluent](/dotnet/api/microsoft.azure.management.fluent) para establecer el agente de usuario con el código siguiente (ejemplo en C#):

```csharp
var azure = Microsoft.Azure.Management.Fluent.Azure
    .Configure()
    // Add your pid in the user agent header
    .WithUserAgent("pid-XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX", String.Empty) 
    .Authenticate(/* Credentials created via Microsoft.Azure.Management.ResourceManager.Fluent.SdkContext.AzureCredentialsFactory */)
    .WithSubscription("<subscription ID>");
```

#### <a name="example-azure-powershell"></a>Ejemplo: Azure PowerShell

Si implementa recursos a través de Azure PowerShell, use el siguiente método para anexar el GUID:

```powershell
[Microsoft.Azure.Common.Authentication.AzureSession]::ClientFactory.AddUserAgent("pid-eb7927c8-dd66-43e1-b0cf-c346a422063")
```

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

#### <a name="example-azure-cli"></a>Ejemplo: CLI de Azure

Cuando use la CLI de Azure para anexar un GUID, establezca la variable de entorno **AZURE_HTTP_USER_AGENT** dentro del ámbito de un script. Pero también se puede establecer globalmente para el ámbito de una shell:

```powershell
export AZURE_HTTP_USER_AGENT='pid-eb7927c8-dd66-43e1-b0cf-c346a422063'
```

Para más información, consulte [Azure SDK para Go](/azure/developer/go/).

## <a name="use-terraform"></a>Uso de Terraform

La compatibilidad con Terraform está disponible a partir de la versión 1.21.0 del proveedor de Azure: [https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md#1210-january-11-2019](https://github.com/terraform-providers/terraform-provider-azurerm/blob/master/CHANGELOG.md#1210-january-11-2019). Esto se aplica a todos los asociados que implementen su solución mediante Terraform y a todos los recursos que implemente y mida el proveedor de Azure (versión 1.21.0 o posteriores).

El proveedor de Azure para Terraform ha agregado un nuevo campo opcional denominado [*partner_id*](https://www.terraform.io/docs/providers/azurerm/#partner_id) que es donde se especifica el GUID de seguimiento que se usa con la solución. El valor de este campo también puede proceder de la variable de entorno *ARM_PARTNER_ID*.

```
provider "azurerm" {
          subscription_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          client_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
          ……
          # new stuff for ISV attribution
          partner_id = "xxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"}
```
Establezca el valor de *partner_id* en un GUID registrado. NO preceda el GUID con "pid-", simplemente establézcalo en el GUID real.

> [!IMPORTANT]
> Si usa Terraform con una aplicación de Azure del marketplace comercial, utilice el identificador de seguimiento entero proporcionado en el Centro de partners. No use un GUID.

## <a name="get-support"></a>Obtención de soporte técnico

Descubra las opciones de soporte técnico en el Marketplace comercial en [Soporte técnico para el programa Marketplace comercial en el Centro de partners](support.md).

### <a name="how-to-submit-a-technical-consultation-request"></a>Envío de una solicitud de consulta técnica

1. Visite [Servicios técnicos para socios](https://aka.ms/TechnicalJourney).
1. Seleccione **Cloud infrastructure and management** (Infraestructura en la nube y administración) para ver el recorrido técnico.
1. Seleccione **Deployment Services** > **Submit a request** (Servicios de implementación > Enviar una solicitud).
1. Inicie sesión con su MSA (cuenta MPN) o con su AAD (cuenta del Panel del asociado).
1. Complete o revise la información de contacto en el formulario que se abre. Los detalles de la consulta se pueden rellenar previamente o puede que tenga opciones desplegables.
1. Escriba un título y una descripción detallada del problema.
1. Seleccione **Submit** (Enviar).

Vea instrucciones paso a paso con capturas de pantallas en [Uso de los servicios de preventas e implementación técnicos](https://aka.ms/TechConsultInstructions).

Un asesor técnico para los asociados de Microsoft le contactará para programar una llamada y determinar sus necesidades.

## <a name="report"></a>Informe
Los informes de uso de Azure de los que se realiza un seguimiento a través de la atribución del uso del cliente no están disponibles actualmente para asociados de ISV. La adición de informes al programa de Marketplace comercial en el Centro de partners para cubrir la atribución del uso del cliente está prevista para la segunda mitad de 2021.

## <a name="faq"></a>Preguntas más frecuentes

#### <a name="after-a-tracking-id-is-added-can-it-be-changed"></a>Una vez que se ha agregado un identificador de seguimiento, ¿se puede cambiar?

Los identificadores de seguimiento de las aplicaciones de Azure del marketplace comercial se administran automáticamente en el Centro de partners. Sin embargo, un cliente puede descargar una plantilla y cambiar o quitar el identificador de seguimiento. Los asociados deben describir de forma anticipada el rol del identificador de seguimiento a sus clientes para impedir que se elimine o modifique. Cambiar el identificador de seguimiento afecta solo a las nuevas implementaciones y recursos, no a los ya existentes.

#### <a name="can-i-track-templates-deployed-from-a-non-microsoft-repository-like-github"></a>¿Puedo realizar un seguimiento de las plantillas implementadas desde un repositorio que no sea de Microsoft, como GitHub?

Sí. Si el identificador de seguimiento está presente cuando se implemente la plantilla, se realiza el seguimiento del uso. Para mantener la asociación entre usted como editor y la plantilla implementada desde un repositorio que no es de Microsoft, descargue primero una copia de la plantilla publicada (que contendrá el identificador de seguimiento) de la descripción del marketplace comercial de la oferta en Azure Portal. Publique esa versión en GitHub o en otro repositorio que no sea de Microsoft.

Si la plantilla no aparece en el marketplace comercial e incluye un GUID registrado, asegúrese de que el GUID esté presente en la versión que publique en GitHub o en otro repositorio que no sea de Microsoft.

#### <a name="does-the-customer-receive-reporting-as-well"></a>¿El cliente recibe también un informe?

No. Los clientes pueden realizar el seguimiento del uso tanto de recursos individuales como de grupos de recursos dentro de Azure Portal. Los clientes no ven el uso desglosado por identificador de seguimiento de CUA.

#### <a name="is-customer-usage-attribution-similar-to-the-digital-partner-of-record-dpor-or-partner-admin-link-pal"></a>¿La atribución del uso por parte de los clientes es similar al asociado de registro digital (DPOR) o al vínculo de administrador de asociado (PAL)?

La atribución del uso por parte de los clientes es un mecanismo para asociar el uso de Azure con la IP repetible e implementable de un asociado, que forma la asociación en el momento de la implementación. DPOR y PAL están pensados para vincular un asociado de consultoría (integrador de sistemas) o de administración (proveedor de servicios administrados) con la superficie de Azure pertinente del cliente mientras el asociado está involucrado con él.