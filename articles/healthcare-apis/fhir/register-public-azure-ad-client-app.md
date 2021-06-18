---
title: 'Registro de una aplicación cliente pública en Azure AD: Azure API for FHIR'
description: En este artículo se explica cómo registrar una aplicación cliente pública en Azure Active Directory, como preparación para la implementación de la API de FHIR en Azure.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: cavoeg
ms.openlocfilehash: 559803a45fc42129d55739ec1d1feba20de41536
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112299148"
---
# <a name="register-a-public-client-application-in-azure-active-directory"></a>Registro de una aplicación cliente pública en Azure Active Directory

En este artículo aprenderá a registrar una aplicación pública en Azure Active Directory.  

Los registros de aplicaciones cliente son representaciones de Azure Active Directory de aplicaciones que pueden autenticar y solicitar permisos de API en nombre de un usuario. Los clientes públicos son aplicaciones, como las aplicaciones móviles y las aplicaciones JavaScript de una sola página, que no pueden mantener la confidencialidad de los secretos. El procedimiento es similar a [registrar un cliente confidencial](register-confidential-azure-ad-client-app.md), pero dado que no se puede confiar en los clientes públicos para almacenar un secreto de aplicación, no es necesario agregar uno.

En este inicio rápido se proporciona información general sobre cómo [registrar una aplicación mediante la plataforma de identidad de Microsoft](../../active-directory/develop/quickstart-register-app.md).

## <a name="app-registrations-in-azure-portal"></a>Registros de aplicaciones en Azure Portal

1. En el panel de navegación izquierdo de [Azure Portal](https://portal.azure.com), haga clic en **Azure Active Directory**.

2. En la hoja **Azure Active Directory**, haga clic en **Registros de aplicaciones**:

    ![Azure Portal. Nuevo registro de aplicaciones.](media/how-to-aad/portal-aad-new-app-registration.png)

3. Haga clic en **Nuevo registro**.

## <a name="application-registration-overview"></a>Introducción al registro de aplicaciones

1. Asigne un nombre para mostrar a la aplicación.

2. Proporcione una dirección URL de respuesta. La dirección URL de respuesta es donde los códigos de autenticación se devolverán a la aplicación cliente. Puede agregar más direcciones URL de respuesta y editar las existentes más adelante.

    ![Azure Portal. Nuevo registro de aplicaciones pública.](media/how-to-aad/portal-aad-register-new-app-registration-PUB-CLIENT-NAME.png)


Para configurar la aplicación de [escritorio](../../active-directory/develop/scenario-desktop-app-registration.md), de [móvil](../../active-directory/develop/scenario-mobile-app-registration.md) o de [página única](../../active-directory/develop/scenario-spa-app-registration.md) como aplicación pública:

1. En [Azure Portal](https://portal.azure.com), en **Registros de aplicaciones**, seleccione la aplicación y, a continuación, seleccione **Autenticación**.

2. Seleccione **Configuración avanzada** > **Tipo de cliente predeterminado**. Para **Treat application as a public client** (Tratar la aplicación como un cliente público), seleccione **Sí**.

3. En el caso de una aplicación de página única, seleccione **Tokens de acceso** y **Tokens de id.** para habilitar el flujo implícito.

   - Si los usuarios pueden iniciar sesión en la aplicación, seleccione **Tokens de id.**
   - Si la aplicación también necesita llamar a una API web protegida, seleccione **Tokens de acceso**.

## <a name="api-permissions"></a>Permisos de API

De forma similar a la [aplicación cliente confidencial](register-confidential-azure-ad-client-app.md), deberá seleccionar los permisos de API que esta aplicación debería poder solicitar en nombre de los usuarios:

1. Abra **Permisos de API**.

    Si usa Azure API for FHIR, agregará un permiso a las API de salud de Azure mediante la búsqueda de dichas API en **API usadas en mi organización**. Solo las encontrará si ya tiene la [implementación de Azure API for FHIR](fhir-paas-powershell-quickstart.md).

    
    Si hace referencia a una aplicación de recursos diferente, seleccione el [registro de la aplicación de recursos de la API de FHIR](register-resource-azure-ad-client-app.md) que creó anteriormente en **Mis API**:

    ![Azure Portal. Nuevos permisos de API pública: valor predeterminado de Azure API for FHIR](media/public-client-app/api-permissions.png)


2. Seleccione los ámbitos que le gustaría que la aplicación pudiera solicitar: ![Azure Portal. Agregar permisos](media/public-client-app/app-permissions.png)

## <a name="validate-fhir-server-authority"></a>Validación de la autoridad del servidor de FHIR
Si la aplicación que registró en este artículo y el servidor de FHIR se encuentran en el mismo inquilino de Azure AD, puede continuar con los pasos siguientes.

Si configura la aplicación cliente en un inquilino de Azure AD diferente del servidor de FHIR, tendrá que actualizar la **autoridad**. En Azure API for FHIR, debe establecer la autoridad en Configuración -- > Autenticación. Establezca su autoridad en **https://login.microsoftonline.com/\<TENANT-ID>** .

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido cómo registrar una aplicación cliente pública en Azure Active Directory. A continuación, pruebe el acceso al servidor de FHIR mediante Postman.
 
>[!div class="nextstepaction"]
>[Acceso a Azure API for FHIR con Postman](access-fhir-postman-tutorial.md)