---
title: Descripción de los conectores de Azure Active Directory Application Proxy
description: Obtenga información sobre los conectores de Azure Active Directory Application Proxy.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: ae7da354b16dbea3e84f8d75fbf97e805387c4b0
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111952110"
---
# <a name="understand-azure-ad-application-proxy-connectors"></a>Descripción de los conectores del Proxy de aplicación de Azure AD

Los conectores son los que hacen posible el Proxy de aplicación de Azure AD. Son simples, fáciles de implementar y mantener, y muy eficaces. En este artículo se habla de qué son los conectores, cómo funcionan, y se ofrecen algunas sugerencias para optimizar la implementación.

## <a name="what-is-an-application-proxy-connector"></a>¿Qué es un conector de proxy de aplicación?

Los conectores son agentes ligeros que se colocan en local y facilitan la conexión saliente del servicio Proxy de aplicación. Los conectores deben instalarse en un servidor de Windows Server con acceso a la aplicación de back-end. Puede organizar los conectores en grupos; cada grupo controla el tráfico a aplicaciones concretas. Para obtener más información sobre el proxy de aplicación y una representación en diagramas de su arquitectura, consulte el documento sobre el [uso de Azure AD Application Proxy para publicar aplicaciones locales para usuarios remotos](what-is-application-proxy.md#application-proxy-connectors).

## <a name="requirements-and-deployment"></a>Requisitos e implementación

Para implementar correctamente Proxy de aplicación necesita al menos un conector, pero se recomiendan dos o más para conseguir una mayor resistencia. Instale el conector en una máquina con Windows Server 2012 R2 o posterior. El conector tiene que comunicarse con el servicio Application Proxy, así como con las aplicaciones locales que se publiquen.

### <a name="windows-server"></a>Windows Server
Se necesita un servidor en el que se ejecute Windows Server 2012 R2 o superior en el que se pueda instalar el conector del proxy de la aplicación. El servidor tiene que conectarse a los servicios de Application Proxy de Azure y a las aplicaciones locales que se publiquen.

El servidor de Windows debe tener habilitado TLS 1.2 antes de instalar el conector del proxy de aplicación. Para habilitar TLS 1.2 en el servidor:

1. Establezca las siguientes claves del Registro:

    ```
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2]
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Client] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\SCHANNEL\Protocols\TLS 1.2\Server] "DisabledByDefault"=dword:00000000 "Enabled"=dword:00000001
    [HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\.NETFramework\v4.0.30319] "SchUseStrongCrypto"=dword:00000001
    ```

1. Reinicio del servidor

Para más información sobre los requisitos de red del servidor del conector, vea [Get started with Application Proxy and install a connector (Información general sobre Proxy de aplicación e instalación de un conector)](application-proxy-add-on-premises-application.md).

## <a name="maintenance"></a>Mantenimiento

Los conectores y el servicio se encargan de todas las tareas de alta disponibilidad. Se pueden agregar o quitar de forma dinámica. Cada vez que llega una solicitud nueva, esta se enruta a uno de los conectores que estén disponibles en ese momento. Si un conector no está disponible temporalmente, no responde a este tráfico.

Los conectores no tienen estado ni datos de configuración en el equipo. Los únicos datos que almacenan son los valores de configuración para conectar el servicio y su certificado de autenticación. Cuando se conectan al servicio, extraen todos los datos de configuración requeridos y los actualizan cada dos minutos.

Los conectores también sondean al servidor para averiguar si hay una versión más reciente del conector. Si se encuentra alguna, los conectores se actualizan.

Puede supervisar los conectores desde el equipo en que se ejecutan, con el registro de eventos y los contadores de rendimiento. O puede ver su estado desde la página Application Proxy de Azure Portal:

![Ejemplo: Conectores de Azure AD Application Proxy](./media/application-proxy-connectors/app-proxy-connectors.png)

No es necesario que elimine manualmente los conectores que no se están utilizando. Cuando hay un conector en ejecución, este permanece activo tal y como se conecta al servicio. Los conectores que no se están usando se etiquetan como _inactivos_ y se quitan tras 10 días de inactividad. No obstante, si quiere desinstalar un conector, desinstale el servicio de conector y el servicio de actualizador del servidor. Reinicie el equipo para quitar completamente el servicio.

## <a name="automatic-updates"></a>Actualizaciones automáticas

Azure AD proporciona actualizaciones automáticas para todos los conectores que se implementen. Mientras se esté ejecutando el servicio de actualización de conectores de Application Proxy, los conectores se [actualizan automáticamente con las últimas versiones principales](application-proxy-faq.yml#why-is-my-connector-still-using-an-older-version-and-not-auto-upgraded-to-latest-version-). Si no ve el servicio de actualización de conectores en el servidor, debe [volver a instalar el conector](application-proxy-add-on-premises-application.md) con el fin de obtener las actualizaciones.

Si no quiere esperar a que haya una actualización automática para el conector, puede realizar una actualización manual. Vaya a la [página de descarga del conector](https://download.msappproxy.net/subscription/d3c8b69d-6bf7-42be-a529-3fe9c2e70c90/connector/download) en el servidor en el que se encuentra el conector y seleccione **Descargar**. Este proceso inicia una actualización del conector local.

Para los inquilinos con varios conectores, las actualizaciones automáticas se dirigen a un conector cada vez en cada grupo para evitar tiempos de inactividad en su entorno.

Puede experimentar un tiempo de inactividad cuando el conector se actualiza si:
  
- Solo tiene un conector. Le recomendamos que instale un segundo conector y [cree un grupo de conectores](application-proxy-connector-groups.md). De este modo, evitará tiempos de inactividad y proporcionará mayor disponibilidad.  
- Un conector estaba en medio de una transacción cuando comenzó la actualización. Aunque se pierde la transacción inicial, el explorador debería reintentar automáticamente la operación o el usuario podría actualizar la página. Cuando se vuelve a enviar la solicitud, el tráfico se enruta a un conector de copia de seguridad.

Para consultar información sobre las versiones publicadas anteriormente y qué cambios incluyen, vea [Application Proxy- Version Release History](./application-proxy-release-version-history.md) (Proxy de aplicación: Historial de lanzamiento de versiones).

## <a name="creating-connector-groups"></a>Creación de grupos de conectores

Los grupos de conectores permiten asignar conectores específicos para atender a aplicaciones específicas. Puede agrupar varios conectores y, a continuación, asignar cada aplicación a un grupo.

Los grupos de conectores facilitan la administración de implementaciones a gran escala. También mejoran la latencia para los inquilinos que tienen aplicaciones hospedadas en distintas regiones, porque se pueden crear grupos de conectores basados en la ubicación para atender solo a las aplicaciones locales.

Para más información acerca de los grupos de conectores, consulte [Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores](application-proxy-connector-groups.md).

## <a name="capacity-planning"></a>Planificación de capacidad

Es importante asegurarse de que la capacidad prevista entre los conectores es suficiente para administrar el volumen de tráfico esperado. Se recomienda que cada grupo de conectores tenga al menos dos conectores para proporcionar alta disponibilidad y escala. Tener tres conectores es una situación óptima en caso de que necesite realizar tareas de servicio en una máquina en cualquier momento.

En general, cuantos más usuarios tenga, mayor será la máquina que necesitará. A continuación se muestra una tabla que proporciona un esquema del volumen y la latencia que diferentes máquinas pueden controlar. Tenga en cuenta que todo se basa en las transacciones por segundo (TPS) que se esperan en len lugar de en las transacciones por usuario, ya que los patrones de uso varían y no se pueden utilizar para predecir la carga. Tenga en cuenta también que habrá algunas diferencias en función del tamaño de las respuestas y el tiempo de respuesta de la aplicación de back-end: cuanto mayor sea el tamaño de respuesta y menor el tiempo de respuesta, menor será el TPS máximo. También le recomendamos que tenga máquinas adicionales para que la carga distribuida entre las máquinas siempre proporcione un búfer amplio. La capacidad adicional garantizará que tanto la resistencia como la disponibilidad son elevadas.

|Núcleos|RAM|Latencia esperada (MS)-P99|TPS Máximo|
| ----- | ----- | ----- | ----- |
|2|8|325|586|
|4|16|320|1150|
|8|32|270|1190|
|16|64|245|1200*|

\* Esta máquina usa una configuración personalizada para generar algunos de los límites de conexión predeterminados más allá de la configuración recomendada de .NET. Se recomienda ejecutar una prueba con la configuración predeterminada antes de ponerse en contacto con el equipo de soporte técnico para obtener este límite cambiado para el inquilino.

> [!NOTE]
> No hay mucha diferencia en el TPS máximo entre máquinas de 4, 8 y 16 núcleos. La diferencia principal entre ellas está en la latencia esperada.
>
> Esta tabla también se centra en el rendimiento esperado de un conector en función del tipo de equipo en el que está instalado. Esto es independiente de los límites del servicio de Application Proxy. Consulte [Límites y restricciones del servicio](../enterprise-users/directory-service-limits-restrictions.md).

## <a name="security-and-networking"></a>Seguridad y redes

Los conectores pueden instalarse en cualquier ubicación de la red que les permita enviar solicitudes al servicio Proxy de aplicación. Lo importante es que el equipo que ejecuta el conector también tenga acceso a las aplicaciones. Puede instalar conectores dentro de la red corporativa o en una máquina virtual que se ejecute en la nube. Los conectores se pueden ejecutar en una red perimetral, también conocida como zona desmilitarizada (DMZ), pero no es necesario porque todo el tráfico es saliente, por lo que la red se mantiene protegida.

Los conectores solo envían solicitudes salientes. El tráfico saliente se envía al servicio de Proxy de aplicación y a las aplicaciones publicadas. No hay que abrir los puertos de entrada, porque una vez que se ha establecido una sesión, el tráfico fluye en ambos sentidos. Tampoco es necesario que configure el acceso de entrada en los firewalls.

Para más información sobre cómo configurar reglas de firewall de salida, vea [Trabajo con servidores proxy locales existentes](./application-proxy-configure-connectors-with-proxy-servers.md).

## <a name="performance-and-scalability"></a>Rendimiento y escalabilidad

La escala del servicio Proxy de aplicación es transparente, pero es un factor en lo que respecta a los conectores. Debe tener suficientes conectores para administrar el tráfico en sus momentos de pico. Como los conectores no tienen estado, no dependen del número de usuarios o de sesiones. Más bien dependen del número de solicitudes y del tamaño de la carga. En el caso del tráfico web estándar, una máquina puede controlar en promedio unas dos mil solicitudes por segundo. La capacidad específica depende de las características exactas de la máquina.

El rendimiento del conector viene determinado por las redes y la CPU. El rendimiento de la CPU es necesario para el cifrado y el descifrado TLS, mientras que las redes son importantes para conectar rápidamente con las aplicaciones y el servicio en línea de Azure.

En cambio, la memoria no supone un problema para los conectores. El servicio en línea se encarga de gran parte del procesamiento y de todo el tráfico no autenticado. Todo lo que puede realizarse en la nube se realiza en la nube.

Si por algún motivo ese conector o máquina no están disponibles, el tráfico comenzará a dirigirse a otro conector del grupo. Esta resistencia también es el motivo por el que se recomienda tener varios conectores.

Otro factor que incide el rendimiento es la calidad de las conexiones entre los conectores; en particular:

- **El servicio en línea**: unas conexiones de baja o alta latencia al servicio Proxy de aplicación de Azure influyen en el rendimiento del conector. Para optimizar el rendimiento, conecte la organización a Azure con Express Route. Si no, el equipo de redes debe asegurarse de que las conexiones a Azure se administren de la manera más eficaz posible.
- **Las aplicaciones de back-end:** en algunos casos, hay servidores proxy adicionales entre el conector y las aplicaciones de back-end que pueden ralentizar o evitar las conexiones. Para solucionar esta situación, abra un explorador desde el servidor del conector e intente acceder a la aplicación. Si ejecuta los conectores en Azure pero las aplicaciones están en local, la experiencia podría no corresponderse a lo que esperan los usuarios.
- **Los controladores de dominio:** si los conectores realizan el inicio de sesión único (SSO) utilizando la delegación restringida de Kerberos, se pondrán en contacto con los controladores de dominio antes de enviar la solicitud al back-end. Los conectores tienen una memoria caché de vales Kerberos, pero en un entorno ocupado, la capacidad de respuesta de los controladores de dominio puede repercutir en el rendimiento. Este problema es más común en el caso de los conectores que se ejecutan en Azure pero se comunican con controladores de dominio que están en local.

Para más información sobre la optimización de la red, vea [Consideraciones sobre la topología de red al utilizar el Proxy de aplicación de Azure Active Directory](application-proxy-network-topology.md).

## <a name="domain-joining"></a>Unión a dominio

Los conectores se pueden ejecutar en un equipo que no esté unido a dominio. Sin embargo, si desea usar el inicio de sesión único (SSO) para aplicaciones que usan la autenticación integrada de Windows (IWA), necesita una máquina unida a dominio. En este caso, los equipos del conector deben estar unidos a un dominio que pueda llevar a cabo la delegación limitada de [kerberos](https://web.mit.edu/kerberos) en nombre de los usuarios para las aplicaciones publicadas.

Los conectores también pueden estar unidos a dominios de bosques que tengan una relación de confianza parcial o a controladores de dominio de solo lectura.

## <a name="connector-deployments-on-hardened-environments"></a>Implementaciones del conector en entornos protegidos

Por lo general, la implementación del conector es sencilla y no requiere ninguna configuración especial. Sin embargo, hay algunas condiciones únicas que deben tenerse en cuenta:

- Las organizaciones que limitan el tráfico saliente deben [abrir los puertos necesarios](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).
- Podría ser necesario que las máquinas compatibles con FIPS cambiaran su configuración para permitir que los procesos del conector generaran y almacenaran un certificado.
- Las organizaciones que bloquean su entorno en función de los procesos que emiten las solicitudes de red tienen que asegurarse de que los servicios del conector estén habilitados para tener acceso a todas las direcciones IP y puertos necesarios.
- En algunos casos, los proxy de reenvío de salida pueden interrumpir la autenticación de certificado de dos direcciones y provocar un error en la comunicación.

## <a name="connector-authentication"></a>Autenticación de conector

Para proporcionar un servicio seguro, los conectores tienen que autenticarse hacia el servicio y el servicio tiene que autenticarse hacia el conector. Esta autenticación se lleva a cabo mediante certificados de cliente y servidor cuando los conectores inician la conexión. De este modo, el nombre de usuario y la contraseña del administrador no se almacenan en el equipo del conector.

Los certificados utilizados son específicos del servicio de Proxy de aplicación. Se crean durante el registro inicial y se renuevan automáticamente por los conectores cada dos meses.

Después de la primera renovación correcta del certificado, el servicio de conector de Azure Active Directory Application Proxy (servicio de red) no tiene permiso para quitar el certificado antiguo del almacén de la máquina local. Si el certificado ha expirado o el servicio ya no lo va a usar, no hay problema en que lo elimine.

Para evitar problemas con la renovación de certificados, asegúrese de que la comunicación de red del conector hacia los [destinos documentados](./application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment) está habilitada.

Si un conector no se conecta al servicio durante varios meses, puede que tenga los certificados caducados. En este caso, desinstale y vuelva a instalar el conector para provocar el registro. Puede ejecutar los siguientes comandos de PowerShell:

```
Import-module AppProxyPSModule
Register-AppProxyConnector -EnvironmentName "AzureCloud"
```

Para la administración pública, use `-EnvironmentName "AzureUSGovernment"`. Para más información, consulte [Instalación del agente para la nube de Azure Government](../hybrid/reference-connect-government-cloud.md#install-the-agent-for-the-azure-government-cloud).

Para más información acerca de cómo comprobar el certificado y solucionar problemas, consulte [Comprobación de que los componentes de máquina y back-end admitan el certificado de confianza del proxy de aplicación](./application-proxy-connector-installation-problem.md#verify-machine-and-backend-components-support-for-application-proxy-trust-certificate).

## <a name="under-the-hood"></a>En segundo plano

Los conectores se basan en el Proxy de aplicación web de Windows Server, por lo que coinciden mayoritariamente respecto a las herramientas de administración, incluidos los Registros de eventos de Windows.

![Administración de registros de eventos con el Visor de eventos](./media/application-proxy-connectors/event-view-window.png)

y contadores de rendimiento de Windows.

![Adición de contadores al conector con el Monitor de rendimiento](./media/application-proxy-connectors/performance-monitor.png)

Los conectores tienen registros de **administración** y **sesión**. Los registros de **administración** incluyen eventos importantes y sus errores. Los registros de **sesión** incluyen todas las transacciones y sus detalles de procesamiento.

Para ver los registros, abra **Visor de eventos** y vaya a **Registros de aplicaciones y servicios** > **Microsoft** > **AadApplicationProxy** > **Conector**. Para que el registro de **sesión** sea visible, en el menú **Ver**, seleccione **Mostrar registros analíticos y de depuración**. El registro de **sesión** se usa normalmente para solucionar problemas y está deshabilitado de forma predeterminada. Habilítelo para comenzar la recopilación de eventos y deshabilítelo cuando ya no se necesite.

Puede examinar el estado del servicio en la ventana Servicios. El conector consta de dos servicios de Windows: el conector real y el actualizador. Ambos deben ejecutarse todo el tiempo.

 ![Ejemplo: Ventana Servicios que muestra los servicios locales de Azure AD](./media/application-proxy-connectors/aad-connector-services.png)

## <a name="next-steps"></a>Pasos siguientes

- [Publicación de aplicaciones en redes independientes y ubicaciones mediante grupos de conectores](application-proxy-connector-groups.md)
- [Trabajo con servidores proxy locales existentes](./application-proxy-configure-connectors-with-proxy-servers.md)
- [Solución de problemas y mensajes de error de Proxy de aplicación](./application-proxy-troubleshoot.md)
- [Cómo instalar de forma silenciosa el conector de Proxy de aplicación de Azure AD](./application-proxy-register-connector-powershell.md)