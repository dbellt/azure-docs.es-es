---
title: Configuración de directivas SSL específicas del cliente de escucha en Azure Application Gateway mediante el portal
description: Aprenda a configurar directivas SSL específicas del cliente de escucha en Application Gateway mediante el portal
services: application-gateway
author: mscatyao
ms.service: application-gateway
ms.topic: how-to
ms.date: 03/30/2021
ms.author: caya
ms.openlocfilehash: b0bda1921c63b4e353365c3b391e17f620740396
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106231046"
---
# <a name="configure-listener-specific-ssl-policies-on-application-gateway-through-portal-preview"></a>Configuración de directivas SSL específicas del cliente de escucha en Application Gateway mediante el portal (versión preliminar)

En este artículo se describe cómo usar Azure Portal para configurar directivas SSL específicas del cliente de escucha en Application Gateway. Las directivas SSL específicas del cliente de escucha permiten configurar clientes de escucha determinados para usar diferentes directivas SSL entre sí. Todavía podrá establecer una directiva SSL predeterminada que usarán todos los clientes de escucha a menos que la directiva SSL específica del cliente de escucha la sobrescriba. 

> [!NOTE]
> Solo las SKU Standard_v2 y WAF_v2 admiten directivas específicas del cliente de escucha, ya que estas directivas forman parte de los perfiles SSL, y estos solo se admiten en las puertas de enlace v2. 



Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="create-a-new-application-gateway"></a>Crear una nueva instancia de Application Gateway

En primer lugar, cree una instancia de Application Gateway como lo haría normalmente mediante el portal: no se necesitan pasos adicionales en esta fase para configurar directivas SSL específicas del cliente de escucha. Para más información sobre cómo crear una instancia de Application Gateway en el portal, consulte nuestro [tutorial de inicio rápido del portal](./quick-create-portal.md).

## <a name="set-up-a-listener-specific-ssl-policy"></a>Configuración de una directiva SSL específica del cliente de escucha

Para configurar una directiva SSL específica del cliente de escucha, primero debe ir a la pestaña **SSL settings (Preview)** (Configuración de SSL [versión preliminar]) del portal y crear un perfil SSL. Cuando cree un perfil SSL, verá dos pestañas: **Autenticación de cliente** y **Directiva SSL**. La pestaña **Directiva SSL** sirve para configurar una directiva SSL específica del cliente de escucha. En la pestaña **Autenticación de cliente** se cargan los certificados de cliente para la autenticación mutua. Para más información, consulte [Configuración de una autenticación mutua](./mutual-authentication-portal.md).

> [!NOTE]
> Se recomienda usar TLS 1.2, ya que esta versión de TLS será obligatoria en el futuro. 

1. Busque **Application Gateway** en el portal, seleccione **Puertas de enlace de aplicaciones** y haga clic en la instancia de Application Gateway existente.

2. Seleccione **SSL settings (Preview)** (Configuración de SSL [versión preliminar]) en el menú de la izquierda.

3. Haga clic en el signo más situado junto a **SSL Profiles** (Perfiles SSL) en la parte superior para crear un perfil SSL.

4. En **SSL Profile Name** (Nombre del perfil SSL), escriba un nombre. En este ejemplo, el perfil se llama *applicationGatewaySSLProfile*. 

5. Vaya a la pestaña **Directiva SSL** y active la casilla **Enable listener-specific SSL Policy** (Habilitar directiva SSL específica del cliente de escucha). 

6. Configure la directiva SSL específica del cliente de escucha según sus requisitos. Puede elegir entre las directivas SSL predefinidas o personalizar la suya propia. Para más información sobre las directivas SSL, consulte [Introducción a la directiva SSL](./application-gateway-ssl-policy-overview.md). Se recomienda el uso de TLS 1.2.

7. Seleccione **Agregar** para guardar la configuración.

    > [!NOTE]
    > No es necesario configurar la autenticación de cliente en un perfil SSL para asociarlo a un cliente de escucha. Puede tener configurada únicamente la autenticación de cliente, o solo la directiva SSL específica del cliente de escucha, o las dos en el perfil SSL.  

    ![Adición de la directiva SSL específica del cliente de escucha al perfil SSL](./media/application-gateway-configure-listener-specific-ssl-policy/listener-specific-ssl-policy-ssl-profile.png)
    
## <a name="associate-the-ssl-profile-with-a-listener"></a>Asociación del perfil SSL con un cliente de escucha

Ahora que hemos creado un perfil SSL con una directiva SSL específica del cliente de escucha, es necesario asociar el perfil SSL al cliente de escucha para poner en marcha la directiva. 

1. Vaya a su instancia de Application Gateway. Si acaba de completar los pasos anteriores, no es necesario hacer nada aquí. 

2. Seleccione **Clientes de escucha** en el menú de la izquierda. 

3. Haga clic en **Agregar agente de escucha** si aún no tiene configurado un cliente de escucha HTTPS. Si ya tiene un cliente de escucha HTTPS, haga clic en él en la lista. 

4. Rellene los campos **Nombre del agente de escucha**, **IP de front-end**, **Puerto** y **Protocolo**, y configure otros **valores HTTPS** para que se adapten a sus necesidades.

5. Active la casilla **Enable SSL Profile** (Habilitar perfil SSL) para poder seleccionar el perfil SSL que se va a asociar con el cliente de escucha. 

6. En la lista desplegable, seleccione el perfil SSL que ha creado. En este ejemplo, elegimos el perfil SSL que hemos creado en los pasos anteriores: *applicationGatewaySSLProfile*. 

7. Siga configurando el resto del cliente de escucha de acuerdo con sus requisitos. 

8. Haga clic en **Agregar** para guardar el nuevo cliente de escucha con el perfil SSL asociado. 

    ![Asociación del perfil SSL al nuevo cliente de escucha](./media/mutual-authentication-portal/mutual-authentication-listener-portal.png)        

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Administrar el tráfico web con Application Gateway mediante la CLI de Azure](./tutorial-manage-web-traffic-cli.md)