---
title: 'Inicio rápido: Eliminación de una aplicación del inquilino de Azure Active Directory (Azure AD)'
description: Este inicio rápido utiliza Azure Portal para eliminar una aplicación del inquilino de Azure Active Directory (Azure AD).
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: quickstart
ms.workload: identity
ms.date: 1/5/2021
ms.author: mtillman
ms.openlocfilehash: 70129123a842ef123992df4614fa7b9a35a774b9
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112075905"
---
# <a name="quickstart-delete-an-application-from-your-azure-active-directory-azure-ad-tenant"></a>Inicio rápido: Eliminación de una aplicación del inquilino de Azure Active Directory (Azure AD)

Este inicio rápido utiliza Azure Portal para eliminar una aplicación que se agregó al inquilino de Azure Active Directory (Azure AD).

Para más información sobre el inicio de sesión único y Azure, consulte [¿Qué es el inicio de sesión único?](what-is-single-sign-on.md)

## <a name="prerequisites"></a>Requisitos previos

Para eliminar una aplicación del inquilino de Azure AD, necesita:

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Uno de los siguientes roles: Administrador global, Administrador de aplicaciones en la nube, Administrador de aplicaciones o Propietario de la entidad de servicio.
- Opcional: Haber finalizado el inicio rápido de [Visualización de las aplicaciones](view-applications-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Incorporación de una aplicación](add-application-portal.md).
- Opcional: Haber finalizado el inicio rápido de [Configuración de una aplicación](add-application-portal-configure.md).
- Opcional: haber finalizado [Asignación de usuarios a una aplicación](add-application-portal-assign-users.md).
- Opcional: Haber finalizado el inicio rápido de [Configuración del inicio de sesión único](add-application-portal-setup-sso.md).

>[!IMPORTANT]
>Use un entorno que no sea de producción para probar los pasos de esta guía de inicio rápido.

## <a name="delete-an-application-from-your-azure-ad-tenant"></a>Eliminación de una aplicación del inquilino de Azure AD

Para eliminar una aplicación del inquilino de Azure AD:

1. En el portal de Azure AD, seleccione **Aplicaciones empresariales**. Busque y seleccione la aplicación que desea eliminar. En este caso, se eliminará la aplicación **GitHub_test** que agregamos en el inicio rápido anterior.
1. En la sección **Administrar** en el panel izquierdo, seleccione **Propiedades**.
1. Seleccione **Eliminar** y, a continuación, seleccione **Sí** para confirmar que quiere eliminar la aplicación de su inquilino de Azure AD.

> [!TIP]
> La administración de aplicaciones se puede automatizar mediante Graph API, consulte el artículo sobre la [automatización de la administración de aplicaciones con Microsoft Graph API](/graph/application-saml-sso-configure-api).

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando haya terminado con esta serie de inicios rápidos, considere la posibilidad de eliminar la aplicación para limpiar el inquilino de prueba. En este inicio rápido se ha indicado cómo eliminar la aplicación.

## <a name="next-steps"></a>Pasos siguientes

Ha completado la serie de inicios rápidos. A continuación, obtenga información acerca del inicio de sesión único (SSO), para lo que debe consultar [¿Qué es el inicio de sesión único (SSO)?](what-is-single-sign-on.md) O bien, lea los procedimientos recomendados en la administración de aplicaciones.
> [!div class="nextstepaction"]
> [Procedimientos recomendados de administración de aplicaciones](application-management-fundamentals.md)
