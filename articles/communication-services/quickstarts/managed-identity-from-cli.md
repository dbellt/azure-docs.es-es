---
title: Creación de una aplicación de identidad administrada de Azure Active Directory mediante la CLI de Azure
titleSuffix: An Azure Communication Services quickstart
description: Las identidades administradas le permiten autorizar a Azure Communication Services el acceso desde aplicaciones que se ejecutan en máquinas virtuales de Azure, aplicaciones de funciones y otros recursos. Esta guía de inicio rápido se centra en la administración de identidades mediante la CLI de Azure.
services: azure-communication-services
author: jbeauregardb
ms.service: azure-communication-services
ms.topic: how-to
ms.date: 03/10/2021
ms.author: jbeauregardb
ms.reviewer: mikben
ms.openlocfilehash: 2ef5a3b162d62fa79ed01a156345070ee12b4862
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105110685"
---
# <a name="authorize-access-with-managed-identity-to-your-communication-resource-in-your-development-environment"></a>Autorice el acceso con identidad administrada al recurso de comunicación del entorno de desarrollo.

El SDK de identidades de Azure proporciona compatibilidad con la autenticación de token de Azure Active Directory para Azure SDK. Las versiones más recientes del SDK de Azure Communication Services para .NET, Java, Python y JavaScript se integran con la biblioteca de identidades de Azure para proporcionar un medio sencillo y seguro de obtener un token de OAuth 2.0 para la autorización de las solicitudes de Azure Communication Services.

Una ventaja del SDK de Azure Identity es que permite usar el mismo código para autenticarse en varios servicios tanto si la aplicación se ejecuta en el entorno de desarrollo como en Azure. El SDK de Azure Identity autentica una entidad de seguridad. Cuando el código se ejecuta en Azure, la entidad de seguridad es una identidad administrada para los recursos de Azure. En el entorno de desarrollo, la identidad administrada no existe, por lo que el SDK autentica tanto el usuario como una aplicación registrada con fines de prueba.

## <a name="prerequisites"></a>Prerrequisitos

 - CLI de Azure. [Guía de instalación](/cli/azure/install-azure-cli)
 - Una cuenta de Azure con una suscripción activa. [Cree su cuenta de forma gratuita](https://azure.microsoft.com/free).

## <a name="setting-up"></a>Instalación

Las identidades administradas deben estar habilitadas en los recursos de Azure que va a autorizar. Para aprender a habilitar las identidades administradas para los recursos de Azure, consulte uno de estos artículos:

- [Azure Portal](../../active-directory/managed-identities-azure-resources/qs-configure-portal-windows-vm.md)
- [Azure PowerShell](../../active-directory/managed-identities-azure-resources/qs-configure-powershell-windows-vm.md)
- [CLI de Azure](../../active-directory/managed-identities-azure-resources/qs-configure-cli-windows-vm.md)
- [Plantilla de Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-template-windows-vm.md)
- [SDK de Azure Resource Manager](../../active-directory/managed-identities-azure-resources/qs-configure-sdk-windows-vm.md)
- [Servicios de aplicaciones](../../app-service/overview-managed-identity.md)

## <a name="authenticate-a-registered-application-in-the-development-environment"></a>Autenticación de una aplicación registrada en el entorno de desarrollo

Si el entorno de desarrollo no admite el inicio de sesión único ni mediante un explorador web, puede usar una aplicación registrada para autenticarse desde el entorno de desarrollo.

### <a name="creating-an-azure-active-directory-registered-application"></a>Creación de una aplicación registrada de Azure Active Directory

Para crear una aplicación registrada mediante la CLI de Azure, debe haber iniciado sesión en la cuenta de Azure en la que desea que se realicen las operaciones. Para ello, puede usar el comando `az login` y escribir sus credenciales en el explorador. Una vez que haya iniciado sesión en su cuenta de Azure desde la CLI, podemos llamar al comando `az ad sp create-for-rbac` para crear la aplicación registrada.

En los ejemplos siguientes se usa la CLI de Azure para crear una nueva aplicación registrada

```azurecli
az ad sp create-for-rbac --name <application-name> 
```

El comando `az ad sp create-for-rbac` devuelve una lista de propiedades de entidad de servicio en formato JSON. Copie estos valores para poder usarlos con el fin de crear las variables de entorno necesarias en el paso siguiente.

```json
{
    "appId": "generated-app-ID",
    "displayName": "service-principal-name",
    "name": "http://service-principal-uri",
    "password": "generated-password",
    "tenant": "tenant-ID"
}
```
> [!IMPORTANT]
> La propagación de las asignaciones de roles de Azure pueden tardar unos minutos.

#### <a name="set-environment-variables"></a>Establecimiento de variables de entorno

El SDK de Azure Identity lee valores de tres variables del entorno de ejecución para autenticar la entidad de servicio. En la tabla siguiente se describe el valor que se debe establecer para cada variable de entorno.

|Variable de entorno|Value
|-|-
|`AZURE_CLIENT_ID`|Valor `appId` del JSON generado 
|`AZURE_TENANT_ID`|Valor `tenant` del JSON generado
|`AZURE_CLIENT_SECRET`|Valor `password` del JSON generado

> [!IMPORTANT]
> Después de establecer las variables de entorno, cierre y vuelva a abrir la ventana de la consola. Si usa Visual Studio u otro entorno de desarrollo, es posible que tenga que reiniciarlo para que registre las nuevas variables de entorno.


## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Información sobre la autenticación](../concepts/authentication.md)

Puede que también le interese:

- [Más información sobre la biblioteca de identidades de Azure](/dotnet/api/overview/azure/identity-readme)