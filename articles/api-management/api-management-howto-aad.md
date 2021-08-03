---
title: Autorización de cuentas de desarrollador con Azure Active Directory
titleSuffix: Azure API Management
description: Aprenda a autorizar a los usuarios mediante Azure Active Directory en API Management.
services: api-management
documentationcenter: API Management
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/22/2021
ms.author: apimpm
ms.openlocfilehash: 37a0ac51f5cbc7d3eaa98027b5e8568dfcf208ee
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111747714"
---
# <a name="authorize-developer-accounts-by-using-azure-active-directory-in-azure-api-management"></a>Autorización de las cuentas de desarrollador mediante Azure Active Directory en Azure API Management

En este artículo se muestra cómo habilitar el acceso al portal para desarrolladores para usuarios de Azure Active Directory. También se muestra cómo administrar grupos de usuarios de Azure AD mediante la adición de grupos externos que contienen a los usuarios.

## <a name="prerequisites"></a>Requisitos previos

- Complete el siguiente inicio rápido: [Creación de una instancia de Azure API Management](get-started-create-service-instance.md).
- Importe y publique una instancia de Azure API Management. Para más información, consulte [Importación y publicación](import-and-publish.md).

[!INCLUDE [premium-dev-standard.md](../../includes/api-management-availability-premium-dev-standard.md)]

## <a name="authorize-developer-accounts-by-using-azure-ad"></a>Autorización de las cuentas de desarrollador con Azure AD

1. Inicie sesión en [Azure Portal](https://portal.azure.com). 
2. Seleccionar ![Icono de flecha.](./media/api-management-howto-aad/arrow.png).
3. En el cuadro de búsqueda, escriba **api**.
4. Seleccione **Servicios API Management**.
5. Seleccione su instancia de servicio API Management.
6. En **Portal para desarrolladores**, seleccione **Identidades**.
7. Haga clic en **+ Agregar** en la parte superior.

    Aparece el panel **Agregar proveedor de identidades** a la derecha.
8. En **Tipo de proveedor**, seleccione **Azure Active Directory**.

    En el panel aparecen controles que le permiten escribir otra información necesaria. Los controles incluyen **Id. de cliente** y **Secreto de cliente**. (Más adelante en el artículo obtendrá información sobre estos controles).
9. Anote el contenido de **Dirección URL de redireccionamiento**.
    

    :::image type="content" source="media/api-management-howto-aad/api-management-with-aad001.png" alt-text="Agregar un proveedor de identidades en Azure Portal":::
    > [!NOTE]
    > Hay dos URL de redireccionamiento:<br/>
    > **URL de redireccionamiento** : apunta al portal para desarrolladores más reciente de la API Management.<br/>
    > **URL de redireccionamiento (portal en desuso)** : apunta al portal para desarrolladores en desuso de API Management.
    >
    > Se recomienda usar la URL de redireccionamiento del portal para desarrolladores más reciente.
   
10. En el explorador, abra una pestaña diferente. 
11. Navegue hasta [Azure Portal: Registros de aplicaciones](https://go.microsoft.com/fwlink/?linkid=2083908) para registrar una aplicación en Active Directory.
12. En **Administrar**, seleccione **Registros de aplicaciones**.
13. Seleccione **Nuevo registro**. En la página **Registrar una aplicación**, establezca los valores de la manera siguiente:
    
    * Establezca **Nombre** en un nombre con sentido; por ejemplo, *portal-desarrollador*
    * Establezca **Tipos de cuenta admitidos** en **Solo las cuentas de este directorio organizativo**. 
    * Establezca **URI de redirección** en el valor obtenido en el paso 9. 
    * Elija **Registro**. 

14.  Una vez registrada la aplicación, copie el **Id. de aplicación (cliente)** de la página **Información general**. 
15. Vuelva a la instancia de API Management. En la ventana **Agregar proveedor de identidades**, pegue el valor de **Id. de aplicación (cliente)** en el cuadro **Id. de cliente**.
16. Vuelva a la configuración de Azure AD, seleccione **Certificados y secretos** en **Administrar**. Seleccione el botón **Nuevo secreto de cliente**. Escriba un valor en **Descripción**, seleccione cualquier opción para **Expira** y elija **Agregar**. Copie el valor del secreto del cliente antes de salir de la página. Esta información la necesitará en el siguiente paso. 
17. En **Administrar**, seleccione **Autenticación** y luego **Tokens de id.** en **Concesión implícita**
18. Vuelva a la instancia de API Management, pegue el secreto en el cuadro **Secreto de cliente**.

    > [!IMPORTANT]
    > Asegúrese de actualizar el **secreto de cliente** antes de que expire la clave. 
    >  
    >

19. La ventana **Add identity provider** (Agregar proveedor de identidades) también contiene el cuadro de texto **Allowed Tenants** (Inquilinos permitidos). Ahí, especifique los dominios de las instancias de Azure AD a las que quiere conceder acceso a las API de la instancia de servicio API Management. Puede separar varios dominios mediante nuevas líneas, espacios o comas.

    > [!NOTE]
    > Puede especificar varios dominios en la sección **Allowed Tenants** (Inquilinos permitidos). Para que un usuario pueda iniciar sesión desde otro dominio distinto al dominio original donde se registró la aplicación, un administrador global de ese otro dominio debe conceder antes a la aplicación permiso de acceso a los datos del directorio. Para conceder permiso, el administrador global debe: a. Ir a `https://<URL of your developer portal>/aadadminconsent` (por ejemplo, https://contoso.portal.azure-api.net/aadadminconsent) ).
    > b. Escribir el nombre de dominio del inquilino de Azure AD al que desea dar acceso.
    > c. Seleccione **Submit** (Enviar). 

20.  Después de especificar la configuración deseada, seleccione **Agregar**.

Después de guardar los cambios, los usuarios de la instancia de Azure AD especificada pueden iniciar sesión en el portal para desarrolladores siguiendo los pasos de [Inicio de sesión en el portal para desarrolladores con una cuenta de Azure AD](#log_in_to_dev_portal).

## <a name="add-an-external-azure-ad-group"></a>Incorporación de un grupo externo de Azure AD

Después de permitir el acceso para los usuarios en un inquilino de Azure AD, se pueden agregar grupos de Azure AD en API Management. Como resultado, se puede controlar la visibilidad del producto mediante el uso de grupos de Azure AD.

Para agregar un grupo de Azure AD externo a APIM, primero se debe completar la sección anterior. De manera predeterminada, la aplicación que registró tiene acceso a Microsoft Graph API con el permiso delegado `User.Read` requerido. También debe conceder a la aplicación acceso a Microsoft Graph API y Graph API de Azure Active Directory con el permiso de aplicación `Directory.Read.All`. Para ello, siga estos pasos: 

1. Vuelva al Registro de aplicaciones que se ha creado en la sección anterior.
2. Seleccione **Permisos de API** y, luego, **Agregar un permiso**. 
3. En el panel **Solicitud de permisos de API**, seleccione la pestaña **API de Microsoft** y, a continuación, seleccione el icono **Microsoft Graph**. Seleccione **Permisos de aplicación** y busque **Directorio**. Seleccione el permiso **Directory.Read.All** y, luego, **Agregar permisos** en la parte inferior del panel.
4. Seleccione **Agregar un permiso**. 
5. En el panel **Solicitud de permisos de API**, seleccione la pestaña **API de Microsoft** y, a continuación, seleccione el icono **Azure Active Directory Graph**. Seleccione **Permisos de aplicación** y busque **Directorio**. Seleccione el permiso **Directory.Read.All** y, luego, **Agregar permisos**. 
6. Seleccione **Conceder consentimiento del administrador para {tenantname}** con el fin de conceder acceso a todos los usuarios de este directorio. 

Ahora los grupos externos de Azure AD se pueden agregan desde la pestaña **Grupos** de la instancia de API Management.

1. Seleccione la pestaña **Grupos** .
2. Seleccione el botón **Add AAD group** (Agregar grupo de AAD).

   ![Botón "Add A A D group" (Agregar grupo de AAD)](./media/api-management-howto-aad/api-management-with-aad008.png)
3. Seleccione el grupo que quiere agregar.
4. Presione el botón **Select** (Seleccionar).

Después de agregar un grupo externo de Azure AD, puede revisar y configurar sus propiedades. Seleccione el nombre del grupo en la pestaña **Groups** (Grupos). Desde aquí, puede editar la información de **nombre** y **descripción** del grupo.
 
Los usuarios de la instancia de Azure AD configurada ya pueden iniciar sesión en el portal para desarrolladores. Pueden ver los grupos para los que tengan visibilidad y suscribirse a ellos.

## <a name="developer-portal---add-azure-ad-account-authentication"></a><a id="log_in_to_dev_portal"></a> Portal para desarrolladores: incorporación de la autenticación de la cuenta de Azure AD

En el portal para desarrolladores, se puede iniciar sesión con AAD utilizando el widget **Sign-in button OAuth** (Botón de inicio de sesión: OAuth). El widget ya está incluido en la página de inicio de sesión del contenido predeterminado del portal para desarrolladores.

Aunque se creará automáticamente una nueva cuenta cada vez que un nuevo usuario inicie sesión con AAD, puede considerar la idea de agregar el mismo widget a la página de registro.

El widget **Sign-up form: OAuth** (Formulario de inicio de sesión: OAuth) representa un formulario que se utiliza para registrarse con OAuth.

> [!IMPORTANT]
> Debe [volver a publicar el portal](api-management-howto-developer-portal-customize.md#publish) para que los cambios de AAD surtan efecto.

## <a name="legacy-developer-portal---how-to-sign-in-with-azure-ad"></a>Portal para desarrolladores heredado: registro con Azure AD

[!INCLUDE [api-management-portal-legacy.md](../../includes/api-management-portal-legacy.md)]

Para iniciar sesión en el portal para desarrolladores mediante una cuenta de Azure AD que ha configurado en las secciones anteriores:

1. Abra una nueva ventana del explorador mediante la dirección URL de inicio de sesión de la configuración de aplicación de Active Directory y seleccione **Azure Active Directory**.

   ![Página de inicio de sesión][api-management-dev-portal-signin]

1. Escriba las credenciales de uno de los usuarios de Azure AD y seleccione **Iniciar sesión**.

   ![Inicio de sesión con el nombre de usuario y la contraseña][api-management-aad-signin]

1. En caso de requerirse más información, puede que se le solicite un formulario de registro. Complete el formulario de registro y seleccione **Suscribirse**.

   ![Botón Suscribirse del formulario de registro][api-management-complete-registration]

Ahora ya inició sesión en el portal para desarrolladores de la instancia del servicio API Management.

![Portal para desarrolladores una vez completado el registro][api-management-registration-complete]

[api-management-dev-portal-signin]: ./media/api-management-howto-aad/api-management-dev-portal-signin.png
[api-management-aad-signin]: ./media/api-management-howto-aad/api-management-aad-signin.png
[api-management-complete-registration]: ./media/api-management-howto-aad/api-management-complete-registration.png
[api-management-registration-complete]: ./media/api-management-howto-aad/api-management-registration-complete.png

[How to add operations to an API]: ./mock-api-responses.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[Get started with Azure API Management]: get-started-create-service-instance.md
[API Management policy reference]: ./api-management-policies.md
[Caching policies]: ./api-management-policies.md#caching-policies
[Create an API Management service instance]: get-started-create-service-instance.md

[https://oauth.net/2/]: https://oauth.net/2/
[WebApp-GraphAPI-DotNet]: https://github.com/AzureADSamples/WebApp-GraphAPI-DotNet

[Prerequisites]: #prerequisites
[Configure an OAuth 2.0 authorization server in API Management]: #step1
[Configure an API to use OAuth 2.0 user authorization]: #step2
[Test the OAuth 2.0 user authorization in the Developer Portal]: #step3
[Next steps]: #next-steps

[Sign in to the developer portal by using an Azure AD account]: #Sign-in-to-the-developer-portal-by-using-an-Azure-AD-account
