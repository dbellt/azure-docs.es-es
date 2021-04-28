---
title: Configuración del proxy del Centro de distribución de claves de Kerberos en Windows Virtual Desktop - Azure
description: Cómo configurar un grupo de hosts de Windows Virtual Desktop para usar un servidor proxy del Centro de distribución de claves Kerberos.
author: Heidilohr
ms.topic: how-to
ms.date: 03/20/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: 9dce264b7f2c88aed11f5b82a61f83cbac6c9697
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107785116"
---
# <a name="configure-a-kerberos-key-distribution-center-proxy-preview"></a>Configuración de un proxy del Centro de distribución de claves de Kerberos (versión preliminar)

> [!IMPORTANT]
> Esta característica actualmente está en su versión preliminar pública.
> Esta versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Los clientes conscientes de la seguridad, como las organizaciones financieras o gubernamentales, a menudo inician sesión con tarjetas inteligentes. Las tarjetas inteligentes hacen que las implementaciones sean más seguras, ya que requieren la autenticación multifactor (MFA). Sin embargo, para la parte de RDP de una sesión de Windows Virtual Desktop, las tarjetas inteligentes requieren una conexión directa o "línea de visión" con un controlador de dominio de Active Directory (AD) para la autenticación Kerberos. Sin esta conexión directa, los usuarios no pueden iniciar sesión automáticamente en la red de la organización desde las conexiones remotas. Los usuarios de una implementación de Windows Virtual Desktop pueden usar el servicio de proxy KDC para el proxy de este tráfico de autenticación e iniciar sesión de forma remota. El proxy KDC permite la autenticación para el Protocolo de escritorio remoto de una sesión de Windows Virtual Desktop, lo que permite que el usuario inicie sesión de forma segura. Esto hace que el trabajo desde casa sea mucho más fácil y permite que determinados escenarios de recuperación ante desastres se ejecuten de forma más fluida.

Sin embargo, la configuración del proxy KDC normalmente implica la asignación del rol de la puerta de enlace de Windows Server en Windows Server 2016 o posterior. ¿Cómo se usa un rol de Servicios de escritorio remoto para iniciar sesión en Windows Virtual Desktop? Para responder a eso, echemos un vistazo a los componentes.

Hay dos componentes en el servicio de Windows Virtual Desktop que deben autenticarse:

- La fuente del cliente de Windows Virtual Desktop que proporciona a los usuarios una lista de las aplicaciones o escritorios disponibles a los que tienen acceso. Este proceso de autenticación se produce en Azure Active Directory, lo que significa que este componente no es el centro de este artículo.
- La sesión de RDP que es el resultado de que un usuario seleccione uno de los recursos disponibles. Este componente usa la autenticación Kerberos y requiere un proxy KDC para los usuarios remotos.

En este artículo se muestra cómo configurar la fuente en el cliente de Windows Virtual Desktop en el Azure Portal. Si quiere obtener información sobre cómo configurar el rol de puerta de enlace RD, consulte [Implementar el rol de puerta de enlace de RD](/azure/virtual-desktop/rd-gateway-role).

## <a name="requirements"></a>Requisitos

Para configurar un host de sesión de Windows Virtual Desktop con un proxy KDC, necesitará lo siguiente:

- Acceso al Azure Portal y a una cuenta de administrador de Azure.
- Los equipos cliente remotos deben ejecutar Windows 10 o Windows 7 y tener instalado el [cliente de Escritorio de Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop). Actualmente, no se admite el cliente web.
- Debe tener un proxy KDC instalado en la máquina. Para obtener información sobre cómo hacerlo, consulte [Configuración del rol de puerta de enlace de RD para Windows Virtual Desktop](rd-gateway-role.md).
- El sistema operativo de la máquina debe ser Windows Server 2016 o posterior.

Una vez que haya asegurado de cumplir estos requisitos, estará listo para comenzar.

## <a name="how-to-configure-the-kdc-proxy"></a>Configuración del proxy de KDC

Para configurar el proxy de KDC:

1. Inicie sesión en Azure Portal como administrador.

2. Vaya a la página de Windows Virtual Desktop.

3. Seleccione el grupo de hosts para el que desea habilitar el proxy de KDC y, a continuación, seleccione **Propiedades de RDP**.

    > [!div class="mx-imgBorder"]
    > ![Captura de pantalla de la página de Azure Portal en la que se muestra un usuario que selecciona grupos de host, el nombre del grupo host de ejemplo y, a continuación, las propiedades de RDP.](media/rdp-properties.png)

4. Seleccione la pestaña **Opciones avanzadas** y, a continuación, escriba un valor con el siguiente formato sin espacios:

    
    > kdcproxyname:s:\<fqdn\>
    

    > [!div class="mx-imgBorder"]
    > ![Una captura de pantalla que muestra la pestaña de opciones avanzadas seleccionada, con el valor especificado como se describe en el paso 4.](media/advanced-tab-selected.png)

5. Seleccione **Guardar**.

6. El grupo de hosts seleccionado ahora debe comenzar a emitir archivos de conexión RDP que incluyen el valor de kdcproxyname que ingresó en el paso 4.

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo administrar el lado Servicios de escritorio remoto del proxy KDC y asignar el rol puerta de enlace de RD, consulte [implementar el rol de puerta de enlace de RD](rd-gateway-role.md).

Si está interesado en escalar los servidores proxy de KDC, aprenda a configurar la alta disponibilidad del proxy KDC en [Agregar alta disponibilidad a la Web RD y la puerta de enlace Web](/windows-server/remote/remote-desktop-services/rds-rdweb-gateway-ha).
