---
title: 'Registro de una aplicación de servicio en Azure AD: Azure API for FHIR'
description: Obtenga información sobre cómo registrar una aplicación cliente de servicio en Azure Active Directory.
services: healthcare-apis
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: conceptual
ms.date: 02/07/2019
ms.author: cavoeg
ms.openlocfilehash: 12df23908d1d5aab18598fcefc574422e233bf19
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112299040"
---
# <a name="register-a-service-client-application-in-azure-active-directory"></a>Registro de una aplicación cliente de servicio en Azure Active Directory

En este artículo aprenderá a registrar una aplicación cliente de servicio en Azure Active Directory. Los registros de aplicaciones cliente son representaciones de Azure Active Directory de aplicaciones que se pueden utilizar para autenticar y obtener tokens. Un cliente de servicio está destinado a ser utilizado por una aplicación para obtener un token de acceso sin la autenticación interactiva de un usuario. Tendrá ciertos permisos de aplicación y utilizará un secreto de aplicación (contraseña) al obtener los tokens de acceso.

Siga estos pasos para crear un cliente de servicio nuevo.

## <a name="app-registrations-in-azure-portal"></a>Registros de aplicaciones en Azure Portal

1. En [Azure Portal](https://portal.azure.com), diríjase a **Azure Active Directory**.

2. Seleccione **App registrations** (Registros de aplicaciones).

    ![Azure Portal. Nuevo registro de aplicaciones.](media/how-to-aad/portal-aad-new-app-registration.png)

3. Seleccione **Nuevo registro**.

4. Asigne al cliente de servicio un nombre para mostrar. Las aplicaciones cliente de servicio no suelen usar una dirección URL de respuesta.

    :::image type="content" source="media/service-client-app/service-client-registration.png" alt-text="Azure Portal. Nuevo registro de aplicaciones de cliente de servicio ":::.

5. Seleccione **Registrar**.

## <a name="api-permissions"></a>Permisos de API

Ahora que ha registrado la aplicación, deberá seleccionar los permisos de API que esta aplicación debería poder solicitar en nombre de los usuarios:

1. Seleccione **Permisos de API**.
1. Seleccione **Agregar un permiso**.

    Si usa Azure API for FHIR, agregará un permiso a las **Azure Healthcare API** mediante la búsqueda de dichas API en **API usadas en mi organización**. 

    Si hace referencia a una aplicación de recursos diferente, seleccione el [registro de la aplicación de recursos de la API de FHIR](register-resource-azure-ad-client-app.md) que creó anteriormente en **Mis API**.

    :::image type="content" source="media/service-client-app/service-client-org-api.png" alt-text="Cliente confidencial. API de mi organización" lightbox="media/service-client-app/service-client-org-api-expanded.png":::.

1. Seleccione los ámbitos (permisos) que la aplicación confidencial debería poder solicitar en nombre del usuario:

    :::image type="content" source="media/service-client-app/service-client-add-permission.png" alt-text="Cliente del servicio. Permisos delegados":::.

1. Dé su consentimiento a la aplicación. Si no tiene los permisos necesarios, consulte con el administrador de Azure Active Directory:

    :::image type="content" source="media/service-client-app/service-client-grant-permission.png" alt-text="Cliente del servicio. Otorgar consentimiento":::

## <a name="application-secret"></a>Secreto de aplicación

El cliente de servicio necesita un secreto (contraseña) para obtener un token.

1. Seleccione **Certificados y secretos**.
2. Seleccione **Nuevo secreto de cliente**.

    ![Azure Portal. Secreto de cliente de servicio](media/how-to-aad/portal-aad-register-new-app-registration-SERVICE-CLIENT-SECRET.png)

3. Proporcione una descripción y la duración del secreto (1 año, 2 años o nunca).

4. Una vez generado el secreto, solo se mostrará una vez en el portal. Tome nota del mismo y guárdelo de forma segura.

## <a name="next-steps"></a>Pasos siguientes

En este artículo ha aprendido cómo registrar una aplicación cliente de servicio en Azure Active Directory. A continuación, pruebe el acceso al servidor de FHIR mediante Postman.
 
>[!div class="nextstepaction"]
>[Acceso a Azure API for FHIR con Postman](access-fhir-postman-tutorial.md)
