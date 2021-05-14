---
title: Creación de una instancia de App Service Environment v1 con ILB
description: Cree un entorno de App Service con un equilibrador de carga interno (ASE con ILB). Este documento solo se proporciona para los clientes que usan App Service Environment v1 heredado.
author: stefsch
ms.assetid: 091decb6-b0de-42a1-9f2f-c18d9b2e67df
ms.topic: article
ms.date: 07/11/2017
ms.author: stefsch
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: a6cc1cae640b97ecb3d95ee1e4f8ec34750e32d2
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833024"
---
# <a name="how-to-create-an-ilb-ase-using-azure-resource-manager-templates"></a>Creación de un ASE de un ILB mediante las plantillas de Azure Resource Manager

> [!NOTE] 
> Este artículo trata sobre App Service Environment v1. Hay una versión más reciente que resulta más fácil de usar y se ejecuta en una infraestructura más eficaz. Para aprender más sobre la nueva versión, consulte [Introducción a App Service Environment](intro.md).
>

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="overview"></a>Información general
Los entornos de App Service pueden crearse con una dirección de red virtual interna, en lugar de una VIP pública.  Esta dirección interna la proporciona un componente de Azure denominado equilibrador de carga interno (ILB).  Un ASE del ILB se puede crear con el Portal de Azure.  También se puede crear de forma automática por medio de las plantillas de Azure Resource Manager.  Este artículo le guía por los pasos y la sintaxis necesarios para crear un ASE de ILB con plantillas de Azure Resource Manager.

Hay tres pasos implicados en la automatización de la creación de un ASE de ILB:

1. En primer lugar, el ASE base se crea en una red virtual con una dirección de equilibrador de carga interno, en lugar de una VIP pública.  Como parte de este paso, se asigna un nombre de dominio raíz al ASE de ILB.
2. Una vez creado el ASE de ILB, se carga un certificado TLS/SSL.  
3. El certificado TLS/SSL cargado se asigna explícitamente al ASE con ILB como certificado TLS/SSL "predeterminado".  Este certificado TLS/SSL se usará para el tráfico TLS a las aplicaciones del ASE de ILB cuando las aplicaciones se direccionan mediante el dominio raíz común asignado al ASE (por ejemplo, `https://someapp.mycustomrootcomain.com`).

## <a name="creating-the-base-ilb-ase"></a>Creación del ASE de ILB base
Puede encontrar un ejemplo de plantilla de Azure Resource Manager y su archivo de parámetros asociado en GitHub, [aquí][quickstartilbasecreate].

La mayoría de los parámetros del archivo *azuredeploy.parameters.json* son comunes para la creación de los dos ASE de ILB, así como de los ASE enlazados a una VIP pública.  La lista siguiente llama a los parámetros especiales o únicos al crear un ASE de ILB:

* *internalLoadBalancingMode*:  en la mayoría de los casos se establece en 3, lo que significa que tanto el tráfico HTTP/HTTPS de los puertos 80 y 443 como los puertos de los canales de control o de datos a los que escucha el servicio FTP en el ASE estarán enlazados a una dirección de red virtual interna asignada al ILB.  Si esta propiedad se establece en 2, solo los puertos relacionados con el servicio FTP (los canales de control y de datos) estarán enlazados a una dirección de ILB, mientras que el tráfico HTTP/HTTPS permanecerá en la VIP pública.
* *dnsSuffix*:  este parámetro define el dominio raíz predeterminado que se asignará al ASE.  En la variación pública de Azure App Service, el dominio raíz predeterminado de todas las aplicaciones web es *azurewebsites.net*.  Sin embargo, dado que un ASE de ILB está dentro de la red virtual de un cliente, no tiene sentido utilizar el dominio raíz predeterminado del servicio público.  En su lugar, un ASE de ILB debe tener un dominio raíz predeterminado que tenga sentido usar en la red virtual interna de una compañía.  Por ejemplo, una empresa hipotética, Contoso Corporation, puede usar el dominio raíz predeterminado *interno contoso.com* para aquellas aplicaciones que se pretende que solo se puedan resolver en la red virtual de Contoso, y a las que solo se pueda acceder desde ella. 
* *ipSslAddressCount*:  el valor predeterminado de este parámetro se establece en 0 automáticamente y se puede encontrar en el archivo *azuredeploy.json*, ya que los ASE de ILB solo tienen una dirección de ILB individual.  No hay direcciones IP-SSL explícitas para un ASE de ILB y, por consiguiente, el grupo de direcciones IP SSL de un ASE de ILB debe establecerse en cero, ya que, de lo contrario, se producirá un error de aprovisionamiento. 

Una vez que se haya rellenado el archivo *azuredeploy.parameters.json* de un ASE de ILB, este se puede crear mediante el siguiente fragmento de código de PowerShell.  Cambie las rutas de acceso de archivo para que coincidan con los archivos de plantilla de Azure Resource Manager del equipo.  Recuerde también especificar sus propios valores para el nombre de implementación y el nombre del grupo de recursos de Azure Resource Manager.

```azurepowershell-interactive
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

Después que se envíe la plantilla de Azure Resource Manager el ASE de ILB tardará unas horas en crearse.  Una vez completada la creación, el ASE de ILB se mostrará en el portal, en la lista de entornos de App Service de la suscripción que desencadenó la implementación.

## <a name="uploading-and-configuring-the-default-tlsssl-certificate"></a>Carga y configuración del certificado TLS/SSL "predeterminado"
Una vez que se crea el ASE de ILB, es preciso asociarle un certificado TLS/SSL "predeterminado" que se use para establecer conexiones TLS/SSL con las aplicaciones.  Continuando con el ejemplo hipotético de Contoso Corporation, si el sufijo DNS predeterminado del ASE es *internal-contoso.com*, una conexión a *`https://some-random-app.internal-contoso.com`* requiere un certificado TLS/SSL que sea válido para * *.internal-contoso.com*. 

Hay varias maneras de obtener un certificado TLS/SSL válido, entre las que se incluyen las CA internas, la adquisición de un certificado de un emisor externo y el uso de un certificado autofirmado.  Independientemente del origen del certificado TLS/SSL, es preciso configurar correctamente los siguientes atributos del certificado:

* *Firmante*:  este atributo se debe establecer en * *.su-dominio-raíz.com*
* *Nombre alternativo del firmante*:  este atributo debe incluir tanto * *.su-dominio-raíz.com* como * *.scm.su-dominio-raíz.com*.  El motivo de la segunda entrada es que las conexiones TLS con el sitio de SCM/Kudu asociadas a cada aplicación se realizarán mediante una dirección, cuyo formato será *your-app-name.scm.your-root-domain-here.com*.

Con un certificado TLS/SSL válido, se necesitan dos pasos preparatorios adicionales.  El certificado TLS/SSL se debe guardar en formato .pfx, o bien convertirse a dicho formato.  Recuerde que es preciso que el archivo .pfx incluya todos los certificados raíz e intermedios, y también debe protegerse con una contraseña.

A continuación, el archivo .pfx resultante debe convertirse en una cadena base64, ya que el certificado TLS/SSL se cargará mediante una plantilla de Azure Resource Manager.  Dado que las plantillas de Azure Resource Manager son archivos de texto, el archivo .pfx debe convertirse en una cadena base64 para que puede incluirse como parámetro de la plantilla.

En el fragmento de código de PowerShell siguiente se muestra un ejemplo de cómo generar un certificado autofirmado, exportarlo como un archivo .pfx, convertir el archivo .pfx en una cadena codificada en base64 y, después, guardarla en otro archivo.  El código de PowerShell para la codificación en base64 se ha adaptado del [blog PowerShell Scripts][examplebase64encoding].

```azurepowershell-interactive
$certificate = New-SelfSignedCertificate -certstorelocation cert:\localmachine\my -dnsname "*.internal-contoso.com","*.scm.internal-contoso.com"

$certThumbprint = "cert:\localMachine\my\" + $certificate.Thumbprint
$password = ConvertTo-SecureString -String "CHANGETHISPASSWORD" -Force -AsPlainText

$fileName = "exportedcert.pfx"
Export-PfxCertificate -cert $certThumbprint -FilePath $fileName -Password $password     

$fileContentBytes = get-content -encoding byte $fileName
$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$fileContentEncoded | set-content ($fileName + ".b64")
```

Una vez que el certificado TLS/SSL se ha generado y convertido correctamente en una cadena con codificación en base64, se puede usar la plantilla de Azure Resource Manager de ejemplo en GitHub para [configurar el certificado TLS/SSL predeterminado][configuringDefaultSSLCertificate].

Los parámetros del archivo *azuredeploy.parameters.json* se enumeran a continuación:

* *appServiceEnvironmentName*:  el nombre del ASE de ILB que se configura.
* *existingAseLocation*:  cadena de texto que contiene la región de Azure en que se implementó el ASE de ILB.  Por ejemplo:  "Centro-sur de EE. UU."
* *pfxBlobString*:  la representación de la cadena con codificación Base64 del archivo pfx.  Mediante el fragmento de código que se ha mostrado anteriormente, se copiaría la cadena de "exportedcert.pfx.b64" y se pegaría como el valor del atributo *pfxBlobString* .
* *password*:  la contraseña que se usa para proteger el archivo pfx.
* *certificateThumbprint*:  la huella digital del certificado.  Si este valor se recupera de PowerShell (por ejemplo, *$certificate.Thumbprint* en el fragmento de código anterior), se puede usar tal cual.  Sin embargo, si copia el valor del cuadro de diálogo del certificado de Windows, no olvide eliminar los espacios superfluos.  El elemento *certificateThumbprint* debe ser similar a:  AF3143EB61D43F6727842115BB7F17BBCECAECAE
* *certificateName*:  es el identificador de cadena fácil de usar que elija y que se usa para identificar el certificado.  El nombre se utiliza como parte del identificador único de Azure Resource Manager para la entidad *Microsoft.Web/certificates* que representa el certificado TLS/SSL.  El nombre **debe** terminar con el sufijo siguiente: \__yourASENameHere_InternalLoadBalancingASE.  El portal utiliza este sufijo como un indicador de que el certificado se usa para asegurar un ASE habilitado para ILB.

A continuación se muestra un ejemplo abreviado de *azuredeploy.parameters.json* :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "appServiceEnvironmentName": {
            "value": "yourASENameHere"
        },
        "existingAseLocation": {
            "value": "East US 2"
        },
        "pfxBlobString": {
            "value": "MIIKcAIBAz...snip...snip...pkCAgfQ"
        },
        "password": {
            "value": "PASSWORDGOESHERE"
        },
        "certificateThumbprint": {
            "value": "AF3143EB61D43F6727842115BB7F17BBCECAECAE"
        },
        "certificateName": {
            "value": "DefaultCertificateFor_yourASENameHere_InternalLoadBalancingASE"
        }
    }
}
```

Una vez que se ha rellenado el archivo *azuredeploy.parameters.json*, se puede configurar el certificado TLS/SSL predeterminado mediante el fragmento de código de PowerShell siguiente.  Cambie las rutas de acceso de archivo para que coincidan con los archivos de plantilla de Azure Resource Manager del equipo.  Recuerde también especificar sus propios valores para el nombre de implementación y el nombre del grupo de recursos de Azure Resource Manager.

```azurepowershell-interactive
$templatePath="PATH\azuredeploy.json"
$parameterPath="PATH\azuredeploy.parameters.json"

New-AzResourceGroupDeployment -Name "CHANGEME" -ResourceGroupName "YOUR-RG-NAME-HERE" -TemplateFile $templatePath -TemplateParameterFile $parameterPath
```

Una vez que se envía la plantilla de Azure Resource Manager, se tarda unos 40 minutos en aplicar el cambio por cada front-end de ASE.  Por ejemplo, con un ASE de un tamaño predeterminado que usa dos front-ends, la plantilla tardará aproximadamente una hora y veinte minutos en completarse.  Mientras la plantilla ejecute el ASE no se podrá escalar.  

Una vez que se completa la plantilla, se puede acceder a las aplicaciones del ASE de ILB a través de HTTPS y las conexiones se protegerán mediante el certificado TLS/SSL predeterminado.  El certificado TLS/SSL predeterminado se usará cuando las direcciones de las aplicaciones del ASE de ILB sean una combinación del nombre de la aplicación y el nombre de host predeterminado.  Por ejemplo, *`https://mycustomapp.internal-contoso.com`* usaría el certificado TLS/SSL predeterminado para * *.internal-contoso.com*.

Sin embargo, al igual que las aplicaciones que se ejecutan en el servicio multiinquilino público, los desarrolladores también pueden configurar nombres de host personalizados para las aplicaciones individuales y luego configurar enlaces de certificados TLS/SSL SNI únicos para aplicaciones individuales.  

## <a name="getting-started"></a>Introducción
Para empezar a trabajar con los entornos de App Service, consulte [Introducción al entorno de App Service](app-service-app-service-environment-intro.md)

[!INCLUDE [app-service-web-try-app-service](../../../includes/app-service-web-try-app-service.md)]

<!-- LINKS -->
[quickstartilbasecreate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-create/
[examplebase64encoding]: https://powershellscripts.blogspot.com/2007/02/base64-encode-file.html 
[configuringDefaultSSLCertificate]: https://azure.microsoft.com/documentation/templates/201-web-app-ase-ilb-configure-default-ssl/ 

