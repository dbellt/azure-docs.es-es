---
author: zhangmanling
ms.service: azure-cdn
ms.topic: include
ms.date: 11/21/2018
ms.author: mazha
ms.openlocfilehash: 4967991b0edaa854acbf6b308596859d662311fe
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95993369"
---
## <a name="prerequisites"></a>Requisitos previos
Antes de escribir el código de administración de la red CDN, tiene que realizar algunos preparativos para habilitar el código para que interactúe con Azure Resource Manager. Para hacer esta preparación, deberá:

* Crear un grupo de recursos que contenga el perfil de CDN creado en este tutorial.
* Configurar Azure Active Directory para que proporcione autenticación para la aplicación.
* Aplicar permisos al grupo de recursos para que solo los usuarios autorizados del inquilino de Azure AD puedan interactuar con el perfil de CDN.

### <a name="creating-the-resource-group"></a>Creación del grupo de recursos
1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Haga clic en **Crear un recurso**.
3. Busque **Grupo de recursos** y, en el panel Grupo de recursos, haga clic en **Crear**.

    ![Crear un grupo de recursos](./media/cdn-app-dev-prep/cdn-new-rg-1-include.png)
3. Llame al grupo de recursos *CdnConsoleTutorial*.  Seleccione la suscripción y elija una ubicación cercana.  Si lo desea, puede hacer clic en la casilla **Anclar al panel** para anclar el grupo de recursos al panel en el portal.  El anclaje hará que sea más fácil encontrarlo luego.  Cuando termine las selecciones, haga clic en **Crear**.

    ![Captura de pantalla del cuadro de diálogo Grupo de recursos.](./media/cdn-app-dev-prep/cdn-new-rg-2-include.png)
4. Una vez creado el grupo de recursos, si no lo ancló al panel, para buscarlo, haga clic en **Examinar** y en **Grupos de recursos**.  Para abrirlo, haga clic en el grupo de recursos.  Tome nota del valor en **Id. de suscripción**. Lo necesitará más adelante.

    ![Captura de pantalla de la sección del tutorial de la consola de CDN.](./media/cdn-app-dev-prep/cdn-subscription-id-include.png)

### <a name="creating-the-azure-ad-application-and-applying-permissions"></a>Creación de la aplicación de Azure AD y aplicación de los permisos
Existen dos enfoques para la autenticación de la aplicación con Azure Active Directory: usuarios individuales o una entidad de servicio. Una entidad de servicio se parece a una cuenta de servicio en Windows.  En lugar de conceder permisos a usuarios concretos para que interactúen con los perfiles de CDN, los permisos se conceden a la entidad de servicio.  Las entidades de servicio se suelen utilizar para procesos automatizados no interactivos.  Aunque en este tutorial escribimos una aplicación de consola interactiva, nos centraremos en el enfoque de entidad de servicio.

Para crear una entidad de servicio, se siguen varios pasos, incluida la creación de una aplicación de Azure Active Directory.  Para crearlo, [complete este tutorial](../articles/active-directory/develop/howto-create-service-principal-portal.md).

> [!IMPORTANT]
> Asegúrese de seguir los pasos descritos en el [tutorial vinculado](../articles/active-directory/develop/howto-create-service-principal-portal.md).  Es *importante* que lo complete exactamente como se describe.  No se olvide de tomar nota del **identificador de inquilino**, el **nombre de dominio del inquilino** (por lo general, un dominio *.onmicrosoft.com*, a menos que haya especificado uno personalizado), el **identificador de cliente** y la **clave de autenticación de cliente**, ya que se necesitará esta información más adelante.  Tenga cuidado de proteger el **identificador de cliente** y la **clave de autenticación de cliente**, ya que cualquiera puede usar estas credenciales para ejecutar operaciones como entidad de servicio.
>
> Cuando llegue al paso denominado Configuración de aplicación multiinquilino, seleccione **No**.
>
> Cuando llegue al paso [Asignación de aplicación a un rol](../articles/active-directory/develop/howto-create-service-principal-portal.md#assign-a-role-to-the-application), use el grupo de recursos que creó antes (*CdnConsoleTutorial*), pero en lugar del rol **Lector**, asigne el rol **Colaborador de perfil de CDN**.  Después de asignar la aplicación al rol **CDN Profile Contributor** (Colaborador de perfil de CDN) en el grupo de recursos, vuelva a este tutorial. 
>
>

Una vez creada la entidad de servicio y asignado el rol **CDN Profile Contributor** (Colaborador de perfil de CDN), la hoja **Usuarios** del grupo de recursos debería parecerse a la imagen siguiente.

![Hoja Usuarios](./media/cdn-app-dev-prep/cdn-service-principal-include.png)

### <a name="interactive-user-authentication"></a>Autenticación interactiva de usuarios
Si, en lugar de una entidad de servicio, prefiere la autenticación interactiva de usuario individual, el proceso se parece al usado con una entidad de servicio.  De hecho, debe seguir el mismo procedimiento, con algunos cambios menores.

> [!IMPORTANT]
> Siga estos pasos únicamente si opta por usar la autenticación de usuario individual en lugar de una entidad de servicio.
>
>

1. Al crear la aplicación, en lugar de **Aplicación web**, elija **Aplicación nativa**.

    ![Aplicación nativa](./media/cdn-app-dev-prep/cdn-native-application-include.png)
2. En la página siguiente, se le pedirá un **URI de redirección**.  No se validará el URI, pero debe recordarlo. Lo necesitará más adelante.
3. No hay necesidad de crear una **clave de autenticación de cliente**.
4. En lugar de asignar una entidad de servicio al rol **CDN Profile Contributor** (Colaborador de perfil de CDN), vamos a asignar usuarios individuales o grupos.  En este ejemplo, puede ver que hemos asignado *CDN Demo User* (Usuario de demostración de CDN) al rol **CDN Profile Contributor** (Colaborador de perfil de CDN).  

    ![Acceso de usuario individual](./media/cdn-app-dev-prep/cdn-aad-user-include.png)
