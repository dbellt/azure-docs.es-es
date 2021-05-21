---
title: 'Tutorial: Administración del acceso a los recursos en la administración de derechos de Azure AD'
description: Tutorial paso a paso sobre cómo crear el primer paquete de acceso mediante Azure Portal en la administración de derechos de Azure Active Directory.
services: active-directory
documentationCenter: ''
author: ajburnle
manager: daveba
editor: markwahl-msft
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: tutorial
ms.subservice: compliance
ms.date: 09/30/2020
ms.author: ajburnle
ms.reviewer: markwahl-msft
ms.collection: M365-identity-device-management
ms.openlocfilehash: 607a68870d3124eadba63b7a04b73ec7705a97a0
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109713979"
---
# <a name="tutorial-manage-access-to-resources-in-azure-ad-entitlement-management"></a>Tutorial: Administración del acceso a los recursos en la administración de derechos de Azure AD

Administrar el acceso a todos lol recursos que necesitan los empleados, como grupos, aplicaciones y sitios, es una función importante en las organizaciones. Querrá conceder a los empleados el nivel de acceso correcto que necesitan para ser productivos y eliminar su acceso cuando ya no se precise.

En este tutorial, trabajará para Woodgrove Bank como administrador de TI. Le han pedido que cree un paquete de recursos para una campaña de marketing que los usuarios internos puedan usar para realizar solicitudes de autoservicio. Las solicitudes no requerirán aprobación y el acceso del usuario expirará al cabo de 30 días. En este tutorial, los recursos de la campaña de marketing son simplemente la pertenencia a un único grupo, pero podrían ser una colección de grupos, aplicaciones o sitios de SharePoint Online.

![Diagrama que muestra la información general del escenario.](./media/entitlement-management-access-package-first/elm-scenario-overview.png)

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear un paquete de acceso con un grupo como recurso
> * Permitir que un usuario del directorio solicite acceso
> * Demostrar cómo un usuario interno puede solicitar el paquete de acceso

Para consultar una demostración detallada del proceso de implementación de la administración de derechos de Azure Active Directory, incluida la creación del primer paquete de acceso, vea el siguiente vídeo:

>[!VIDEO https://www.youtube.com/embed/zaaKvaaYwI4]

## <a name="prerequisites"></a>Prerrequisitos

Para usar la administración de derechos de Azure AD, debe tener una de las licencias siguientes:

- Azure AD Premium P2
- Licencia de Enterprise Mobility + Security (EMS) E5

Para obtener más información, consulte [Requisitos de licencia](entitlement-management-overview.md#license-requirements).

## <a name="step-1-set-up-users-and-group"></a>Paso 1: Configuración de usuarios y grupos

Un directorio de recurso tiene uno o más recursos para compartir. En este paso, creará un grupo denominado **Recursos de marketing** en el directorio de Woodgrove Bank que es el recurso de destino de la administración de derechos. También configurará un solicitante interno.

**Rol necesario:** administrador global o administrador de usuarios.

![Creación de usuarios y grupos](./media/entitlement-management-access-package-first/elm-users-groups.png)

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como administrador global o administrador de usuarios.  

1. En el panel de navegación izquierdo, haga clic en **Azure Active Directory**.

1. Cree o configure los siguientes dos usuarios. Puede usar estos nombres u otros diferentes. **Admin1** puede ser el usuario con el que inició sesión.

    | Nombre | Rol del directorio |
    | --- | --- |
    | **Admin1** | Administrador global<br/>O bien<br/>Administrador de usuarios |
    | **Solicitante1** | Usuario |

1. Cree un grupo de seguridad de Azure AD llamado **Recursos de marketing** con un tipo de pertenencia de **Asignado**.

    Este grupo será el recurso de destino para la administración de derechos. El grupo debe estar vacío para comenzar.

## <a name="step-2-create-an-access-package"></a>Paso 2: Creación de un paquete de acceso

Un *paquete de acceso* es un conjunto de recursos que un equipo o proyecto necesita y se rige por directivas. Los paquetes de acceso se definen en contenedores llamados *catálogos*. En este paso, creará un paquete de acceso **Campaña de marketing** en el catálogo **General**.

**Rol necesario:** administrador global, administrador de Identity Governance, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

![Creación de un paquete de acceso](./media/entitlement-management-access-package-first/elm-access-package.png)

1. En la barra de navegación izquierda de Azure Portal, haga clic en **Azure Active Directory**.

2. En el menú izquierdo, haga clic en **Gobernanza de identidades**.

3. En el menú izquierdo, haga clic en **Paquetes de acceso**.  Si ve **Acceso denegado**, asegúrese de que haya una licencia Azure AD Premium P2 en el directorio.

4. Haga clic en **New access package** (Nuevo paquete acceso).

    ![Administración de derechos en Azure Portal](./media/entitlement-management-shared/access-packages-list.png)

5. En la pestaña **Básico**, escriba el nombre **Campaña de marketing** y la descripción **Acceso a los recursos de la campaña**.

6. Deje la lista desplegable **Catálogo** establecida en **General**.

    ![New access package (Nuevo paquete de acceso): pestaña Datos básicos](./media/entitlement-management-access-package-first/basics.png)

7. Haga clic en **Siguiente** para abrir la pestaña **Resource roles** (Roles de recursos).

    En esta pestaña se seleccionan los recursos y el rol de recurso que se incluirán en el paquete de acceso.

8. Haga clic en **Grupos y equipos**.

9. En el panel Seleccionar grupos, busque y seleccione el grupo **Recursos de marketing** que creó anteriormente.

     De forma predeterminada, verá grupos dentro del catálogo general. Al seleccionar un grupo fuera del catálogo general, que puede ver si activa la casilla **Ver todo**, se agregará al catálogo general.

    ![Captura de pantalla que muestra la pestaña "Nuevo paquete de acceso: roles de recursos" y la ventana "Seleccionar grupos".](./media/entitlement-management-access-package-first/resource-roles-select-groups.png)

10. Haga clic en **Seleccionar** para agregar el grupo a la lista.

11. En la lista desplegable **Rol**, seleccione **Miembro**.

    ![New access package (Nuevo paquete de acceso): pestaña Resource roles (Roles de recursos)](./media/entitlement-management-access-package-first/resource-roles.png)

    >[!IMPORTANT]
    >Los grupos a los que se pueden asignar roles que se agregan a un paquete de acceso se indicarán mediante el subtipo **Assignable to roles** (asignable a roles). Consulte [Creación de un grupo al que se pueden asignar roles](../roles/groups-create-eligible.md) en Azure Active Directory para obtener más detalles sobre los grupos que se pueden asignar a roles de Azure AD. Tenga en cuenta que una vez que un grupo al que se pueden asignar roles está presente en un catálogo de paquetes de acceso, los usuarios administrativos que pueden encargarse de la administración de derechos, como administradores globales, administradores de usuarios y propietarios de catálogos, podrán controlar los paquetes de acceso en el catálogo. De este modo, tendrán la posibilidad de decidir a quién agregar a esos grupos. Si no ve un grupo al que se pueden asignar roles que quiera agregar, o no puede agregarlo, asegúrese de que tiene los roles de administración de derechos y de Azure AD que se requieren para realizar esta operación. Quizá necesite pedirle a alguien con los roles necesarios que agregue el recurso al catálogo. Para obtener más información, vea [Roles necesarios para agregar recursos a un catálogo](entitlement-management-delegate.md#required-roles-to-add-resources-to-a-catalog).

    >[!NOTE]
    > Si usa [grupos dinámicos](../enterprise-users/groups-create-rule.md) no verá ningún otro rol disponible además del propietario; es así por diseño.
    > ![Información general de los escenarios](./media/entitlement-management-access-package-first/dynamic-group-warning.png)
    


12. Haga clic en **Siguiente** para abrir la pestaña **Solicitudes**.

    En esta pestaña creará una directiva de solicitud. Una *directiva* define las reglas o barreras para acceder a un paquete de acceso. Creará una directiva que permite que un usuario específico del directorio del recurso solicite este paquete de acceso.

13. En la sección **Usuarios que pueden solicitar acceso**, haga clic en **Para los usuarios de su directorio** y, a continuación, haga clic en **Usuarios y grupos específicos**.

    ![Nuevo paquete de acceso: pestaña Solicitudes](./media/entitlement-management-access-package-first/requests.png)

14. Haga clic en **Agregar usuarios y grupos**.

15. En el panel Seleccionar usuarios y grupos, seleccione el usuario **Solicitante1** que creó anteriormente.

    ![Nuevo paquete de acceso: pestaña Solicitudes - Seleccionar usuarios y grupos](./media/entitlement-management-access-package-first/requests-select-users-groups.png)

16. Haga clic en **Seleccionar**.

17. Desplácese hacia abajo hasta las secciones **Aprobación** y **Enable requests** (Habilitar solicitudes).

18. Deje **Requerir aprobación** establecido en **No**.

19. En **Enable requests** (Habilitar solicitudes), haga clic en **Sí** para permitir que este paquete de acceso se solicite en cuanto se cree.

    ![Nuevo paquete de acceso: pestaña Solicitudes - Aprobación y Enable requests (Habilitar solicitudes)](./media/entitlement-management-access-package-first/requests-approval-enable.png)

20. Haga clic en **Siguiente** para abrir la pestaña **Ciclo de vida**.

21. En la sección **Expiración**, establezca **Las asignaciones de paquetes de acceso expiran** en **Número de días**.

22. Establezca **Las asignaciones expiran después de** en **30** días.

    ![Nuevo paquete de acceso: pestaña Ciclo de vida](./media/entitlement-management-access-package-first/lifecycle.png)

23. Haga clic en **Siguiente** para abrir la pestaña **Revisar y crear**.

    ![New access package (Nuevo paquete de acceso): pestaña Revisar y crear](./media/entitlement-management-access-package-first/review-create.png)

    Transcurridos unos instantes, verá una notificación que dice que el paquete de acceso se ha creado correctamente.

24. En el menú izquierdo del paquete de acceso Campaña de marketing, haga clic en **Información general**.

25. Copie el **vínculo del portal Mi acceso**.

    Este vínculo lo usará en el paso siguiente.

    ![Información general del paquete acceso: vínculo del portal Mi acceso](./media/entitlement-management-shared/my-access-portal-link.png)

## <a name="step-3-request-access"></a>Paso 3: Solicitar acceso

En este paso, realizará los pasos como **solicitante interno** y solicitará acceso al paquete acceso. Los solicitantes envían sus solicitudes mediante un sitio conocido como el portal Mi acceso. El portal Mi acceso permite a los solicitantes enviar solicitudes de paquetes de acceso, ver los paquetes de acceso a los que ya tienen acceso y ver sus historiales de solicitudes.

**Rol necesario:** solicitante interno

1. Cierre sesión en Azure Portal.

1. En una nueva ventana del explorador, vaya al vínculo del portal Mi acceso que copió en el paso anterior.

1. Inicie sesión en el portal Mi acceso como **Solicitante1**.

    Debería ver el paquete de acceso **Campaña de marketing**.

1. Si es necesario, en la columna **Descripción**, haga clic en la flecha para ver detalles sobre el paquete de acceso.

    ![Portal Mi acceso: paquetes de acceso](./media/entitlement-management-shared/my-access-access-packages.png)

1. Haga clic en la marca de verificación para seleccionar el paquete.

1. Haga clic en **Solicitar acceso** para abrir el panel Solicitar acceso.

    ![Portal Mi acceso: botón Solicitar acceso](./media/entitlement-management-access-package-first/my-access-request-access-button.png)

1. En el cuadro **Justificación comercial**, escriba la justificación **Estoy trabajando en la nueva campaña de marketing**.

    ![Portal Mi acceso: solicitud de acceso](./media/entitlement-management-shared/my-access-request-access.png)

1. Haga clic en **Enviar**.

1. En el menú izquierdo, haga clic en **Historial de solicitudes** para comprobar que la solicitud se ha enviado.

## <a name="step-4-validate-that-access-has-been-assigned"></a>Paso 4: Validación de que se ha asignado el acceso

En este paso, confirmará que se asignó el paquete de acceso al **solicitante interno** y que este es ahora miembro del grupo **Recursos de marketing**.

**Rol necesario:** Administrador global, administrador de usuarios, propietario del catálogo o administrador de paquetes de acceso.

1. Cierre sesión en el portal Mi acceso.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) como **Admin1**.

1. Haga clic en **Azure Active Directory** y, luego, haga clic en **Gobernanza de identidades**.

1. En el menú izquierdo, haga clic en **Paquetes de acceso**.

1. Busque y haga clic en el paquete de acceso **Campaña de marketing**.

1. En el menú izquierdo, haga clic en **Solicitudes**.

    Verá Solicitante1 y la directiva inicial con el estado **Entregado**.

1. Haga clic en la solicitud para ver los detalles.

    ![Paquete de acceso: detalles de la solicitud](./media/entitlement-management-access-package-first/request-details.png)

1. En el panel de navegación izquierdo, haga clic en **Azure Active Directory**.

1. Haga clic en **Grupos** y abra el grupo **Recursos de marketing**.

1. Haga clic en **Miembros**.

    Verá que **Solicitante1** aparece como miembro.

    ![Miembros de Recursos de marketing](./media/entitlement-management-access-package-first/group-members.png)

## <a name="step-5-clean-up-resources"></a>Paso 5: Limpieza de recursos

En este paso, se quitarán los cambios realizados y se eliminará el paquete de acceso **Campaña de marketing**.

**Rol necesario:**  administrador global o administrador de usuarios.

1. En Azure Portal, haga clic en **Azure Active Directory** y, luego, en **Gobernanza de identidades**.

1. Abra el paquete de acceso **Campaña de marketing**.

1. Haga clic en **Asignaciones**.

1. En **Solicitante1**, haga clic en el botón de puntos suspensivos ( **...** ) y, luego, en **Quitar acceso**. En el mensaje que aparece, haga clic en **Sí**.

    Tras unos momentos, el estado cambiará de Entregado a Expirado.

1. Haga clic en **Roles de recursos**.

1. En **Recursos de marketing**, haga clic en el botón de puntos suspensivos ( **...** ) y, luego, en **Eliminar rol de recurso**. En el mensaje que aparece, haga clic en **Sí**.

1. Abra la lista de paquetes de acceso.

1. En **Campaña de marketing**, haga clic en el botón de puntos suspensivos ( **...** ) y, a continuación, haga clic en **Eliminar**. En el mensaje que aparece, haga clic en **Sí**.

1. En Azure Active Directory, elimine los usuarios creados como **Solicitante1** y **Admin1**.

1. Elimine el grupo **Recursos de marketing**.

## <a name="next-steps"></a>Pasos siguientes

Avance al siguiente artículo para conocer los pasos del escenario común de administración de derechos.
> [!div class="nextstepaction"]
> [Escenarios comunes](entitlement-management-scenarios.md)
