---
title: Configuración de la autenticación mutua en Azure Application Gateway mediante el portal
description: Aprenda a configurar una instancia de Application Gateway para que tenga autenticación mutua mediante el portal.
services: application-gateway
author: mscatyao
ms.service: application-gateway
ms.topic: how-to
ms.date: 04/02/2021
ms.author: caya
ms.openlocfilehash: b2118a257f029a63445b09dfa618d6e3ceacddda
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106230944"
---
# <a name="configure-mutual-authentication-with-application-gateway-through-portal-preview"></a>Configuración de la autenticación mutua con Application Gateway mediante el portal (versión preliminar)

En este artículo se describe cómo usar Azure Portal para configurar la autenticación mutua en Application Gateway. La autenticación mutua significa que Application Gateway autentica al cliente que envía la solicitud mediante el certificado de cliente que se carga en Application Gateway. 

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="before-you-begin"></a>Antes de empezar

Para configurar la autenticación mutua con una instancia de Application Gateway, necesita cargar un certificado de cliente en la puerta de enlace. El certificado de cliente se utilizará para validar el certificado que el cliente va a presentar a Application Gateway. Para fines de prueba, puede usar un certificado autofirmado. Sin embargo, estos certificados no se recomiendan para las cargas de trabajo de producción, ya que son más difíciles de administrar y no son completamente seguros. 

Para más información, en concreto, sobre qué tipo de certificados de cliente se pueden cargar, consulte [Introducción a la autenticación mutua con Application Gateway](./mutual-authentication-overview.md#certificates-supported-for-mutual-authentication).

## <a name="create-a-new-application-gateway"></a>Crear una nueva instancia de Application Gateway

En primer lugar, cree una instancia de Application Gateway como lo haría normalmente mediante el portal; no se necesitan pasos adicionales en esta fase para habilitar la autenticación mutua. Para más información sobre cómo crear una instancia de Application Gateway en el portal, consulte nuestro [tutorial de inicio rápido del portal](./quick-create-portal.md).

## <a name="configure-mutual-authentication"></a>Configuración de la autenticación mutua 

Para configurar una instancia de Application Gateway existente con la autenticación mutua, primero debe ir a la pestaña **SSL settings (Preview)** (Configuración de SSL [versión preliminar]) del portal y crear un perfil SSL. Cuando cree un perfil SSL, verá dos pestañas: **Autenticación de cliente** y **Directiva SSL**. En la pestaña **Autenticación de cliente** se cargan los certificados de cliente. La pestaña **Directiva SSL** sirve para configurar una directiva SSL específica del cliente de escucha. Para más información, consulte [Configuración de una directiva SSL específica del cliente de escucha](./application-gateway-configure-listener-specific-ssl-policy.md).

> [!IMPORTANT]
> Asegúrese de cargar toda la cadena de certificados de la entidad de certificación de cliente en un archivo y solo una cadena por archivo.

1. Busque **Application Gateway** en el portal, seleccione **Puertas de enlace de aplicaciones** y haga clic en la instancia de Application Gateway existente.

2. Seleccione **SSL settings (Preview)** (Configuración de SSL [versión preliminar]) en el menú de la izquierda.

3. Haga clic en el signo más situado junto a **SSL Profiles** (Perfiles SSL) en la parte superior para crear un perfil SSL.

4. En **SSL Profile Name** (Nombre del perfil SSL), escriba un nombre. En este ejemplo, el perfil se llama *applicationGatewaySSLProfile*. 

5. Permanezca en la pestaña **Autenticación de cliente**. Cargue el certificado PEM que quiera usar para la autenticación mutua entre el cliente y Application Gateway mediante el botón **Upload a new certificate** (Cargar un nuevo certificado). 

    Para más información sobre cómo extraer cadenas de certificados de entidad de certificación de cliente de confianza para cargarlas aquí, consulte [Extracción de cadenas de certificados de entidad de certificación de cliente de confianza](./mutual-authentication-certificate-management.md).

   > [!NOTE]
   > Si este no es el primer perfil SSL y ha cargado otros certificados de cliente en su instancia de Application Gateway, puede decidir reutilizar un certificado existente en la puerta de enlace mediante el menú desplegable. 

6. Active la casilla **Verify client certificate issuer's DN** (Comprobar el nombre distintivo del emisor del certificado de cliente) solo si quiere que Application Gateway compruebe el nombre distintivo del emisor inmediato del certificado de cliente. 

7. Considere la posibilidad de agregar una directiva específica del cliente de escucha. Consulte las instrucciones en [Configuración de directivas SSL específicas del cliente de escucha](./application-gateway-configure-listener-specific-ssl-policy.md).

8. Seleccione **Agregar** para guardar la configuración.
    > [!div class="mx-imgBorder"]
    > ![Adición de autenticación de cliente al perfil SSL](./media/mutual-authentication-portal/mutual-authentication-portal.png)

## <a name="associate-the-ssl-profile-with-a-listener"></a>Asociación del perfil SSL con un cliente de escucha

Ahora que hemos creado un perfil SSL con la autenticación mutua, es necesario asociarlo al cliente de escucha para completar la configuración de la autenticación mutua. 

1. Vaya a su instancia de Application Gateway. Si acaba de completar los pasos anteriores, no es necesario hacer nada aquí. 

2. Seleccione **Clientes de escucha** en el menú de la izquierda. 

3. Haga clic en **Agregar agente de escucha** si aún no tiene configurado un cliente de escucha HTTPS. Si ya tiene un cliente de escucha HTTPS, haga clic en él en la lista. 

4. Rellene los campos **Nombre del agente de escucha**, **IP de front-end**, **Puerto** y **Protocolo**, y configure otros **valores HTTPS** para que se adapten a sus necesidades.

5. Active la casilla **Enable SSL Profile** (Habilitar perfil SSL) para poder seleccionar el perfil SSL que se va a asociar con el cliente de escucha. 

6. En la lista desplegable, seleccione el perfil SSL que acaba de crear. En este ejemplo, elegimos el perfil SSL que hemos creado en los pasos anteriores: *applicationGatewaySSLProfile*. 

7. Siga configurando el resto del cliente de escucha de acuerdo con sus requisitos. 

8. Haga clic en **Agregar** para guardar el nuevo cliente de escucha con el perfil SSL asociado. 

    > [!div class="mx-imgBorder"]
    > ![Asociación del perfil SSL al nuevo cliente de escucha](./media/mutual-authentication-portal/mutual-authentication-listener-portal.png)

## <a name="renew-expired-client-ca-certificates"></a>Renovación de certificados de entidad de certificación de cliente expirados

En el caso de que el certificado de entidad de certificación de cliente haya expirado, puede actualizarlo en la puerta de enlace mediante los pasos siguientes: 

1. Vaya a su instancia de Application Gateway y diríjase a la pestaña **SSL settings (Preview)** (Configuración de SSL [versión preliminar]) en el menú de la izquierda. 
 
1. Seleccione los perfiles SSL existentes con el certificado de cliente expirado. 
 
1. Seleccione **Upload a new certificate** (Cargar un nuevo certificado) en la pestaña **Autenticación de cliente** y cargue el nuevo certificado de cliente. 
 
1. Seleccione el icono de papelera junto al certificado que ha expirado. Esta acción eliminará la asociación de ese certificado del perfil SSL. 

1. Repita los pasos del 2 al 4 anteriores con cualquier otro perfil SSL que vaya a usar el mismo certificado de cliente expirado. Podrá elegir el nuevo certificado que ha cargado en el paso 3 en el menú desplegable de otros perfiles SSL.

## <a name="next-steps"></a>Pasos siguientes

- [Administrar el tráfico web con Application Gateway mediante la CLI de Azure](./tutorial-manage-web-traffic-cli.md)