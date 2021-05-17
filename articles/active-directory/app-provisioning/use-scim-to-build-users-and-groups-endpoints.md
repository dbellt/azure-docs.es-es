---
title: Creación de un punto de conexión de SCIM para el aprovisionamiento de usuarios en aplicaciones desde Azure Active Directory
description: Aprenda a desarrollar un punto de conexión de SCIM, integrar la API de SCIM con Azure Active Directory, y aprovisionar automáticamente usuarios y grupos en las aplicaciones en la nube.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: conceptual
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: ac876ff4370c20ca0b4db59575a3d8ab49b7b0a3
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784852"
---
# <a name="tutorial-develop-a-sample-scim-endpoint-in-azure-active-directory"></a>Tutorial: Desarrollo de un punto de conexión SCIM de ejemplo en Azure Active Directory

Nadie quiere crear un punto de conexión desde cero, por lo que hemos creado [código de referencia](https://aka.ms/scimreferencecode) para que pueda empezar a trabajar con [System for Cross-domain Identity Management (SCIM)](https://aka.ms/scimoverview). En apenas 5 minutos, podrá poner en marcha el punto de conexión de SCIM sin código.

En este tutorial se describe cómo implementar el código de referencia de SCIM en Azure y probarlo con Postman o mediante la integración con el cliente de SCIM de Azure Active Directory (Azure AD). Este tutorial va dirigido a desarrolladores que buscan empezar a trabajar con SCIM o a usuarios interesados en probar un punto de conexión de SICM.

En este tutorial, aprenderá a:

> [!div class="checklist"]
>
> * Implementar el punto de conexión de SCIM en Azure
> * Probar el punto de conexión de SCIM

## <a name="deploy-your-scim-endpoint-in-azure"></a>Implementación de un punto de conexión de SCIM en Azure

Los pasos que se proporcionan aquí permiten implementar el punto de conexión de SCIM en un servicio con [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/) y [Azure App Service](../../app-service/index.yml). El código de referencia de SCIM también se puede ejecutar localmente, hospedar en un servidor local o implementar en otro servicio externo.

1. Vaya al [código de referencia](https://github.com/AzureAD/SCIMReferenceCode) de GitHub y seleccione **Clonar o descargar**.

1. Elija **Open in Desktop** (Abrir en el escritorio), o bien, copie el vínculo, abra Visual Studio y seleccione **Clonar o extraer código del repositorio** para escribir el vínculo copiado y crear una copia local.

1. En Visual Studio, asegúrese de iniciar sesión en la cuenta que tiene acceso a los recursos de hospedaje.

1. En el Explorador de soluciones, abra *Microsoft.SCIM.sln* y haga clic con el botón derecho en el archivo *Microsoft.SCIM.WebHostSample*. Seleccione **Publicar**.

    ![Captura de pantalla que muestra el archivo de ejemplo.](media/use-scim-to-build-users-and-groups-endpoints/cloud-publish.png)

    > [!NOTE]
    > Para ejecutar esta solución de forma local, haga doble clic en el proyecto y seleccione **IIS Express** para iniciar el proyecto como una página web con una dirección URL de host local.

1. Seleccione **Crear perfil** y asegúrese de que **App Service** y **Crear nuevo** estén seleccionados.

    ![Captura de pantalla que muestra la ventana Publicar.](media/use-scim-to-build-users-and-groups-endpoints/cloud-publish-2.png)

1. Recorra las opciones del cuadro de diálogo y cambie el nombre de la aplicación por un nombre de su elección. Este nombre se usa tanto en la dirección URL de la aplicación como en el punto de conexión de SCIM.

    ![Captura de pantalla que muestra la creación de un servicio de aplicaciones.](media/use-scim-to-build-users-and-groups-endpoints/cloud-publish-3.png)

1. Seleccione el grupo de recursos que quiera usar y elija **Publicar**.

    ![Captura de pantalla que muestra la publicación de un nuevo servicio de aplicaciones.](media/use-scim-to-build-users-and-groups-endpoints/cloud-publish-4.png)

1. Vaya a la aplicación en **Azure App Service** > **Configuración** y seleccione **Nueva configuración de la aplicación** para agregar la opción *Token__TokenIssuer* con el valor `https://sts.windows.net/<tenant_id>/`. Reemplace `<tenant_id>` por el identificador de inquilino de Azure AD. Si quiere probar el punto de conexión de SCIM mediante [Postman](https://github.com/AzureAD/SCIMReferenceCode/wiki/Test-Your-SCIM-Endpoint), agregue una opción *ASPNETCORE_ENVIRONMENT* con el valor `Development`.

   ![Captura de pantalla que muestra la ventana Configuración de la aplicación.](media/use-scim-to-build-users-and-groups-endpoints/app-service-settings.png)

   Al probar el punto de conexión con una aplicación empresarial en [Azure Portal](use-scim-to-provision-users-and-groups.md#integrate-your-scim-endpoint-with-the-aad-scim-client), tiene dos opciones: Puede mantener el entorno en `Development` y proporcionar el token de prueba desde el punto de conexión `/scim/token` o puede cambiar el entorno a `Production` y dejar el campo de token vacío.

Eso es todo. El punto de conexión de SCIM ahora está publicado y le permite usar la dirección URL de Azure App Service para probarlo.

## <a name="test-your-scim-endpoint"></a>Prueba de un punto de conexión de SCIM

Las solicitudes a un punto de conexión de SCIM requieren autorización. El estándar SCIM tiene varias opciones para la autenticación y la autorización, como cookies, autenticación básica, autenticación de cliente TLS o cualquiera de los métodos enumerados en la especificación [RFC 7644](https://tools.ietf.org/html/rfc7644#section-2).

Evite métodos poco seguros, como nombre de usuario y contraseña, y use mejor los seguros, como OAuth. Azure AD admite tokens de portador de larga duración (para aplicaciones de la galería y ajenas a la galería) y la concesión de autorización OAuth (para aplicaciones publicadas de la galería).

> [!NOTE]
> Los métodos de autorización proporcionados en el repositorio son solo con fines de prueba. Al realizar la integración con Azure AD, puede revisar la guía de autorización. Consulte [Planeamiento del aprovisionamiento de un punto de conexión de SCIM](use-scim-to-provision-users-and-groups.md).

El entorno de desarrollo habilita características poco seguras para producción, como el código de referencia para controlar el comportamiento de la validación del token de seguridad. El código de validación del token emplea un token de seguridad autofirmado, y la clave de firma se almacena en el archivo de configuración. Consulte el parámetro **Token:IssuerSigningKey** del archivo *appsettings.Development.jsen*.

```json
"Token": {
    "TokenAudience": "Microsoft.Security.Bearer",
    "TokenIssuer": "Microsoft.Security.Bearer",
    "IssuerSigningKey": "A1B2C3D4E5F6A1B2C3D4E5F6",
    "TokenLifetimeInMins": "120"
}
```

> [!NOTE]
> Cuando se envía una solicitud **GET** al punto de conexión `/scim/token`, se emite un token con la clave configurada. Ese token se puede usar posteriormente como token de portador para la autorización.

El código de validación de tokens predeterminado está configurado para usar un token de Azure AD y requiere que el inquilino emisor se configure mediante el parámetro **Token:TokenIssuer** del archivo *appsettings.json*.

``` json
"Token": {
    "TokenAudience": "8adf8e6e-67b2-4cf2-a259-e3dc5476c621",
    "TokenIssuer": "https://sts.windows.net/<tenant_id>/"
}
```

### <a name="use-postman-to-test-endpoints"></a>Uso de Postman para probar los puntos de conexión

Después de implementar el punto de conexión de SCIM, puede realizar una prueba para asegurarse de que es compatible con la especificación RFC de SCIM. En este ejemplo se proporciona un conjunto de pruebas en Postman para validar operaciones CRUD (crear, leer, actualizar y eliminar) en usuarios y grupos, el filtrado, las actualizaciones de la pertenencia a grupos y la deshabilitación de usuarios.

Los puntos de conexión están en el directorio `{host}/scim/` y puede usar solicitudes HTTP estándar para interactuar con ellos. Para modificar la ruta de `/scim/`, consulte *ControllerConstant.cs* en **AzureADProvisioningSCIMreference** > **ScimReferenceApi** > **Controllers**.

> [!NOTE]
> Solo puede utilizar puntos de conexión HTTP en pruebas locales. El servicio de aprovisionamiento de Azure AD requiere que el punto de conexión admita HTTPS.

1. Descargue [Postman](https://www.getpostman.com/downloads/) e inicie la aplicación.
1. Copie el vínculo y péguelo en Postman para importar la colección de pruebas: `https://aka.ms/ProvisioningPostman`.

    ![Captura de pantalla que muestra la importación de la colección de prueba en Postman.](media/use-scim-to-build-users-and-groups-endpoints/postman-collection.png)

1. Cree un entorno de prueba que tenga estas variables:

   |Entorno|Variable|Value|
   |-|-|-|
   |Ejecución del proyecto de manera local con IIS Express|||
   ||**Servidor**|`localhost`|
   ||**Puerto**|`:44359` *(no olvide los **`:`** )*|
   ||**Api**|`scim`|
   |Ejecución del proyecto de manera local con Kestrel|||
   ||**Servidor**|`localhost`|
   ||**Puerto**|`:5001` *(no olvide los **`:`** )*|
   ||**Api**|`scim`|
   |Hospedaje del punto de conexión en Azure|||
   ||**Servidor**|*(escriba la dirección URL de SCIM)*|
   ||**Puerto**|*(deje en blanco)*|
   ||**Api**|`scim`|

1. Use **Get Key** (Obtener clave) desde la colección de Postman para enviar una solicitud **GET** al punto de conexión del token y recuperar un token de seguridad que se almacenará en la variable **token** de las solicitudes subsiguientes.

   ![Captura de pantalla que muestra la carpeta Get Key (Obtener clave) de Postman.](media/use-scim-to-build-users-and-groups-endpoints/postman-get-key.png)

   > [!NOTE]
   > Para proteger un punto de conexión de SCIM, necesita un token de seguridad antes de conectarse. En el tutorial se usa el punto de conexión `{host}/scim/token` para generar un token autofirmado.

Eso es todo. Ahora puede ejecutar la colección de **Postman** para probar la funcionalidad del punto de conexión de SCIM.

## <a name="next-steps"></a>Pasos siguientes

Para desarrollar un punto de conexión de usuario y grupo compatible con SCIM con interoperabilidad para un cliente, consulte la [implementación de cliente de SCIM](http://www.simplecloud.info/#Implementations2).

> [!div class="nextstepaction"]
> [Tutorial: Desarrollo y planeación del aprovisionamiento de un punto de conexión de SCIM](use-scim-to-provision-users-and-groups.md)
> [Tutorial: Configuración del aprovisionamiento en una aplicación de la galería](configure-automatic-user-provisioning-portal.md)