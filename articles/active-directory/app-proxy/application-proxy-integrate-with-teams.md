---
title: Acceso a aplicaciones de Azure Active Directory Application Proxy en Teams
description: Use Azure Active Directory Application Proxy para acceder a la aplicación local a través de Microsoft Teams.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: how-to
ms.date: 04/28/2021
ms.author: kenwith
ms.reviewer: harshja
ms.openlocfilehash: ff79c1f6f1e646ef6261769a9bb6c10b0e696688
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108230069"
---
# <a name="access-your-on-premises-applications-through-microsoft-teams-with-azure-active-directory-application-proxy"></a>Acceso a las aplicaciones locales a través de Microsoft Teams con Azure Active Directory Application Proxy

Proxy de aplicación de Azure Active Directory permite usar el inicio de sesión único en las aplicaciones locales con independencia de dónde se encuentre. Microsoft Teams optimiza la colaboración para efectuarse desde un solo lugar. La integración de ambos supone que los usuarios pueden ser más productivos con sus compañeros de equipo en cualquier situación.

Los usuarios pueden agregar aplicaciones en la nube a sus canales de Teams [mediante pestañas](https://support.office.com/article/Video-Using-Tabs-7350a03e-017a-4a00-a6ae-1c9fe8c497b3?ui=en-US&rs=en-US&ad=US) pero ¿qué ocurre con los sitios de SharePoint o la herramienta de planeamiento que se hospedan de manera local? Proxy de aplicación es la solución. Puede agregar las aplicaciones publicadas a través de Proxy de aplicación a sus canales utilizando las mismas direcciones URL externas que usan siempre para tener acceso a sus aplicaciones de forma remota. Y, dado que Proxy de aplicación se autentica a través de Azure Active Directory, los usuarios obtienen una experiencia de inicio de sesión único.

## <a name="install-the-application-proxy-connector-and-publish-your-app"></a>Instalación del conector de Proxy de aplicación y publicación de la aplicación

Si no lo ha hecho ya, [configure Proxy de aplicación para el inquilino e instale el conector](../app-proxy/application-proxy-add-on-premises-application.md). A continuación, publique la aplicación local para acceso remoto. Cuando vaya a publicar la aplicación, tome nota de la dirección URL externa porque se utiliza para agregar la aplicación a Teams.

Si ya tiene sus aplicaciones publicadas, pero no recuerda sus direcciones URL externas, búsquelas en [Azure Portal](https://portal.azure.com). Inicie sesión, vaya a **Azure Active Directory** > **Aplicaciones empresariales** > **Todas las aplicaciones** > seleccione la aplicación > **Proxy de aplicación**.

## <a name="add-your-app-to-teams"></a>Incorporación de la aplicación a Teams

Una vez que publique la aplicación a través de Proxy de aplicación, haga saber a los usuarios que pueden agregarla como una pestaña directamente en sus canales de Teams; luego, la aplicación estará disponible para que la utilicen todos los miembros del equipo. Haga que sigan estos tres pasos:

1. Vaya hasta el canal de Teams donde desee agregar esta aplicación y seleccione **+** para agregar una pestaña.

   ![Seleccione + para agregar una pestaña en Teams.](./media/application-proxy-integrate-with-teams/add-tab.png)

1. Seleccione **Sitio Web** en las opciones de la pestaña.

   ![Seleccione el sitio web en la pantalla Agregar pestaña.](./media/application-proxy-integrate-with-teams/website.png)

1. Asigne un nombre a la pestaña y establezca la dirección URL para la dirección URL externa de Proxy de aplicación.

   ![Asigne un nombre a la pestaña y agregue la dirección URL externa.](./media/application-proxy-integrate-with-teams/tab-name-url.png)

Una vez que un miembro de un equipo agrega la pestaña, se muestra para todos los usuarios del canal. Los usuarios que accedan a la aplicación obtendrán acceso de inicio de sesión único con las credenciales que se usen para Microsoft Teams. Los usuarios que no tengan acceso a la aplicación pueden ver la pestaña en Teams, pero no podrán usarla hasta que les proporcione permisos en la aplicación local y la versión publicada en Azure Portal de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

- Aprenda a [publicar sitios de SharePoint locales](application-proxy-integrate-with-sharepoint-server.md) con Proxy de aplicación.
- Configure las aplicaciones para usar [dominios personalizados](application-proxy-configure-custom-domain.md) para su dirección URL externa.
