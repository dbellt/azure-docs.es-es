---
title: Acceso a aplicaciones de Proxy de aplicación de Azure AD en Teams | Microsoft Docs
description: Utilice Proxy de aplicación de Azure AD para tener acceso a la aplicación local a través de Microsoft Teams.
services: active-directory
documentationcenter: ''
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 09/05/2017
ms.author: kenwith
ms.reviewer: harshja
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: b2834dff789aac95e19f7d1229fc3b3c9449688b
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108224552"
---
# <a name="access-your-on-premises-applications-through-microsoft-teams"></a>Acceso a aplicaciones locales a través de Microsoft Teams

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
