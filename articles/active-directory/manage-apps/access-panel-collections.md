---
title: Creación de colecciones para los portales Mis aplicaciones en Azure Active Directory | Microsoft Docs
description: Use colecciones de Mis aplicaciones para personalizar las páginas de Mis aplicaciones con el fin de que los usuarios finales reciban una experiencia más sencilla. Organice las aplicaciones en grupos con pestañas independientes.
services: active-directory
documentationcenter: ''
author: iantheninja
manager: CelesteDG
ms.assetid: ''
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 02/10/2020
ms.author: iangithinji
ms.reviewer: kasimpso
ms.collection: M365-identity-device-management
ms.openlocfilehash: cc79e8026cb91b8ef3eac2addbb097b9db83afa7
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107377690"
---
# <a name="create-collections-on-the-my-apps-portal"></a>Creación de colecciones en el portal Mis aplicaciones

Los usuarios pueden usar el portal Mis aplicaciones para ver e iniciar las aplicaciones basadas en la nube a las que tienen acceso. De forma predeterminada, todas las aplicaciones a las que puede tener acceso un usuario se muestran juntas en una sola página. Para organizar mejor esta página para los usuarios, si tiene una licencia de Azure AD Premium P1 o P2, puede configurar colecciones. Con una colección, puede agrupar las aplicaciones que están relacionadas (por ejemplo, por rol de trabajo, tarea o proyecto) y mostrarlas en una pestaña independiente. Una colección aplica básicamente un filtro a las aplicaciones a las que un usuario ya puede acceder, por lo que el usuario solo ve las aplicaciones del área de trabajo que se le han asignado.

> [!NOTE]
> En este artículo se explica cómo un administrador puede habilitar y crear colecciones. Para información sobre cómo los usuarios finales pueden usar el portal Mis aplicaciones y las colecciones, consulte [Acceso y uso de colecciones](../user-help/my-applications-portal-workspaces.md).

## <a name="enable-the-latest-my-apps-features"></a>Habilitación de las características más recientes de Mis aplicaciones

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como administrador global o coadministrador.

2. Vaya a **Azure Active Directory** > **Configuración de usuario**.

3. En **Versiones preliminares de características de usuarios**, seleccione **Administrar la configuración de la versión preliminar de características del usuario**.

4. En **Los usuarios pueden utilizar las características en versión preliminar de Mis aplicaciones**, elija una de las siguientes opciones:
   * **Seleccionado**: habilita las características para un grupo específico. Use la opción **Seleccionar un grupo** para seleccionar el grupo para el que quiere habilitar las características.  
   * **Todos**: habilita las características para todos los usuarios.

> [!NOTE]
> Para abrir el portal de Mis aplicaciones, los usuarios pueden usar el vínculo `https://myapps.microsoft.com` o el vínculo personalizado de la organización, como `https://myapps.microsoft.com/contoso.com`. Después de habilitar la nueva experiencia de Mis aplicaciones, se mostrará un banner similar a **Una experiencia actualizada de Mis aplicaciones está disponible** en la parte superior de la página Mis aplicaciones y los usuarios podrán seleccionar **Probarlo** para ver la nueva experiencia. Para dejar de usar la nueva experiencia, los usuarios pueden seleccionar **Sí** en el banner similar a **Salir de la nueva experiencia** de la parte superior de la página.

## <a name="create-a-collection"></a>Creación de una colección

Para crear una colección, debe tener una licencia de Azure AD Premium P1 o P2.

1. Abra [**Azure Portal**](https://portal.azure.com/) e inicie sesión como administrador con una licencia de Azure AD Premium P1 o P2.

2. Vaya a **Azure Active Directory** > **Aplicaciones empresariales**.

3. En **Administrar**, seleccione **Colecciones**.

4. Seleccione **Nueva colección**. En la página **Nueva colección**, escriba un **nombre** para la colección (se recomienda no usar "colección" en el nombre). Escriba una **descripción**.

   ![Página Nueva colección](media/acces-panel-collections/new-collection.png)

5. Seleccione la pestaña **Aplicaciones** . Seleccione **+ Agregar aplicación** y, luego, en la página **Agregar aplicaciones**, seleccione todas las aplicaciones que quiere agregar a la colección o use el cuadro **Buscar** para buscar aplicaciones.

   ![Agregar una aplicación a la colección](media/acces-panel-collections/add-applications.png)

6. Cuando haya terminado de agregar aplicaciones, seleccione **Agregar**. Aparece la lista de aplicaciones seleccionadas. Puede usar las flechas arriba y abajo para cambiar el orden de las aplicaciones de la lista. Para bajar una aplicación o eliminarla de la colección, seleccione el menú **Más** ( **...** ).

7. Seleccione la pestaña **Propietarios** . Seleccione **+ Agregar usuarios y grupos** y, luego, en la página **Agregar usuarios y grupos**, seleccione los usuarios o grupos a los que quiere asignar la propiedad. Cuando haya terminado de seleccionar usuarios y grupos, elija **Seleccionar**.

9. Seleccione la pestaña **Usuarios y grupos**. Seleccione **+ Agregar usuarios y grupos** y, luego, en la página **Agregar usuarios y grupos**, seleccione los usuarios o grupos a los que quiere asignar la colección. También puede usar el cuadro **Buscar** para buscar usuarios o grupos. Cuando haya terminado de seleccionar usuarios y grupos, elija **Seleccionar**.

   ![Adición de usuarios y grupos](media/acces-panel-collections/add-users-and-groups.png)

11. Seleccione **Revisar + crear**. Aparecen las propiedades de la nueva colección.


## <a name="view-audit-logs"></a>Visualización de registros de auditoría

Los registros de auditoría registran las operaciones de colecciones de Mis aplicaciones, incluidas las acciones de los usuarios finales de creación de colecciones. Los siguientes eventos se generan desde Mis aplicaciones:

* Crear colección
* Editar colección
* Eliminar colección
* Inicio de una aplicación (usuario final)
* Adición de aplicaciones de autoservicio (usuario final)
* Eliminación de aplicaciones de autoservicio (usuario final)

Puede obtener acceso a los registros de auditoría de [Azure Portal](https://portal.azure.com) seleccionando **Azure Active Directory** > **Aplicaciones empresariales** > **Registros de auditoría** en la sección Actividad. En **Servicio**, seleccione **Mis aplicaciones**.

## <a name="get-support-for-my-account-pages"></a>Obtener soporte técnico para las páginas de Mi cuenta

En la página Mis aplicaciones, un usuario puede seleccionar **Mi cuenta** > **Ver mi cuenta** para abrir la configuración de su cuenta. En la página de Azure AD **Mi cuenta**, los usuarios pueden administrar la información de seguridad, los dispositivos, las contraseñas, etc. También pueden tener acceso a la configuración de su cuenta de Office.

En caso de que necesite enviar una solicitud de soporte técnico por un problema con la página de la cuenta de Azure AD o la cuenta de Office, siga estos pasos para que la solicitud se gestione correctamente: 

* En caso de tener problemas con la página **"Mi cuenta" de Azure AD**, abra una solicitud de soporte técnico desde Azure Portal. Vaya a **Azure Portal** > **Azure Active Directory** > **Nueva solicitud de soporte técnico**.

* En caso de tener problemas con la página **"Mi cuenta" de Office**, abra una solicitud de soporte técnico desde el centro de administración de Microsoft 365. Vaya a **Centro de administración de Microsoft 365** > **Soporte técnico**. 

## <a name="next-steps"></a>Pasos siguientes
[Experiencias de usuario final para aplicaciones en Azure Active Directory](end-user-experiences.md)