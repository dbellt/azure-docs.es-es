---
title: Condición de ubicación del acceso condicional de Azure Active Directory
description: Use la condición de ubicación para controlar el acceso en función de la ubicación del usuario.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 04/28/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: calebb, olhuan
ms.collection: M365-identity-device-management
ms.custom: contperf-fy20q4
ms.openlocfilehash: e417ec70c39ff17a62237361a68dde5bb633dcdf
ms.sourcegitcommit: 42ac9d148cc3e9a1c0d771bc5eea632d8c70b92a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2021
ms.locfileid: "109845418"
---
# <a name="using-the-location-condition-in-a-conditional-access-policy"></a>Uso la condición de ubicación en una directiva de acceso condicional 

Como se explica en el [artículo de información general](overview.md), las directivas de acceso condicional se encuentran en la instrucción if-then más básica, que combina señales para tomar decisiones y aplicar directivas de la organización. Una de esas señales que se puede incorporar en el proceso de toma de decisiones es la ubicación.

![Señal condicional conceptual más la decisión de obtener el cumplimiento](./media/location-condition/conditional-access-signal-decision-enforcement.png)

Las organizaciones pueden usar esta ubicación para tareas comunes como: 

- Solicitar la autenticación multifactor a los usuarios que obtengan acceso a un servicio desde fuera de la red corporativa.
- Bloquear el acceso a los usuarios que accedan a un servicio desde determinados países o regiones.

La ubicación viene determinada por la dirección IP pública que proporciona un cliente a Azure Active Directory o las coordenadas GPS que proporciona la aplicación Microsoft Authenticator. De forma predeterminada, las directivas de acceso condicional se aplican a todas las direcciones IPv4 e IPv6. 

## <a name="named-locations"></a>Ubicaciones con nombre

Las ubicaciones se designan en Azure Portal en **Azure Active Directory** > **Seguridad** > **Acceso condicional** > **Ubicaciones con nombre**. Estas ubicaciones de red con nombre pueden incluir ubicaciones, como intervalos de redes de la sede central de una organización, intervalos de redes VPN o intervalos que desea bloquear. Las ubicaciones con nombre se pueden definir por intervalos de direcciones IPv4/IPv6 o por países. 

![Ubicación con nombre en Azure Portal](./media/location-condition/new-named-location.png)

### <a name="ip-address-ranges"></a>Intervalos de direcciones IP

Para definir una ubicación con nombre por intervalos de direcciones IPv4/IPv6, deberá proporcionar: 

- Un **nombre** para la ubicación.
- Uno o varios intervalos IP.
- Opcionalmente, **Marcar como ubicación de confianza**.

![Nuevas ubicaciones de IP en Azure Portal](./media/location-condition/new-trusted-location.png)

Las ubicaciones con nombre definidas por los intervalos de direcciones IPv4/IPv6 están sujetas a las siguientes limitaciones: 

- Configurar hasta 195 ubicaciones con nombre.
- Se pueden configurar hasta 2000 intervalos IP por ubicación con nombre.
- Se admiten tanto IPv4 como IPv6.
- No se pueden configurar intervalos IP privados.
- El número de direcciones IP que se pueden incluir en un intervalo es limitado. Solo se permiten las máscaras CIDR mayores que /8 al definir un intervalo de direcciones IP. 

#### <a name="trusted-locations"></a>Ubicaciones de confianza

Los administradores pueden designar ubicaciones definidas por intervalos de direcciones IP para que sean ubicaciones con nombre de confianza. 

Los inicios de sesión desde ubicaciones con nombre de confianza mejoran la precisión del cálculo de riesgos de Azure AD Identity Protection, lo que reduce el riesgo del inicio de sesión de los usuarios cuando se autentican desde una ubicación marcada como de confianza. Además, las ubicaciones con nombre de confianza pueden ser el destino de directivas de acceso condicional. Por ejemplo, puede [restringir el registro de autenticación multifactor a ubicaciones de confianza](howto-conditional-access-policy-registration.md). 

### <a name="countries"></a>Países

Las organizaciones pueden determinar la ubicación del país por dirección IP o coordenadas de GPS. 

Para definir una ubicación con nombre por país, es preciso proporcionar: 

- Un **nombre** para la ubicación.
- Elegir determinar la ubicación por dirección IP o coordenadas de GPS.
- Agregar uno o varios países.
- Opcionalmente, elegir **Incluir países o regiones desconocidos**.

![País como ubicación en Azure Portal](./media/location-condition/new-named-location-country-region.png)

Si selecciona **Determine location by IP address (IPv4 only)** [Determinar ubicación por dirección IP (solo IPv4)], el sistema recopilará la dirección IP del dispositivo en el que el usuario inicia sesión. Cuando un usuario inicia sesión, Azure AD resuelve la dirección IPv4 del usuario en un país o región, y la asignación se actualiza periódicamente. Las organizaciones pueden usar ubicaciones con nombre definidas por países para bloquear el tráfico de los países en los que no hacen negocios. 

> [!NOTE]
> Los inicios de sesión desde direcciones IPv6 no se pueden asignar a países o regiones y se consideran áreas desconocidas. Solo las direcciones IPv4 se pueden asignar a países o regiones.

Si selecciona **Determine location by GPS coordinates (Preview)** [Determinar ubicación por coordenadas de GPS (versión preliminar)], el usuario deberá tener la aplicación Microsoft Authenticator instalada en su dispositivo móvil. Cada hora, el sistema se pondrá en contacto con la aplicación de Microsoft Authenticator del usuario para recopilar la ubicación GPS del dispositivo móvil del usuario.

La primera vez que se requiera que el usuario comparta su ubicación desde la aplicación Microsoft Authenticator, el usuario recibirá una notificación en la aplicación. El usuario tendrá que abrir la aplicación y conceder permisos de ubicación. 

En las 24 horas siguientes, si el usuario sigue accediendo al recurso y ha concedido a la aplicación permiso para ejecutarse en segundo plano, la ubicación se compartirá de forma silenciosa una vez por hora desde el dispositivo, por lo que el usuario no tendrá que seguir sacando su dispositivo móvil. Después de 24 horas, el usuario tendrá que abrir la aplicación y aprobar manualmente la notificación.

#### <a name="include-unknown-countriesregions"></a>Inclusión de países o regiones desconocidos

Algunas direcciones IP no están asignadas a ningún país o región determinados, incluidas todas las direcciones IPv6. Para capturar estas ubicaciones IP, active la casilla **Incluir países o regiones desconocidos** al definir una ubicación geográfica. Esta opción le permite elegir si estas direcciones IP deberían estar incluidas en la ubicación con nombre. Use esta configuración cuando la directiva que usa la ubicación con nombre deba aplicarse en ubicaciones desconocidas.

### <a name="configure-mfa-trusted-ips"></a>Configurar IP de confianza de MFA

También puede configurar los intervalos de direcciones IP que representen a la intranet local de su organización en la [configuración del servicio de la autenticación multifactor](https://account.activedirectory.windowsazure.com/usermanagement/mfasettings.aspx). Esta función le permite configurar hasta 50 intervalos de direcciones IP. Los intervalos de direcciones IP están en formato CIDR. Para obtener más información, vea [IP de confianza](../authentication/howto-mfa-mfasettings.md#trusted-ips).  

Si se han configurado IP de confianza, se mostrarán como **IP de confianza de MFA** en la lista de ubicaciones de la condición de ubicación.

#### <a name="skipping-multi-factor-authentication"></a>Omisión de la autenticación multifactor

En la página de configuración del servicio de la autenticación multifactor, podrá identificar a los usuarios de la intranet corporativa seleccionando **Omitir la autenticación multifactor para solicitudes de usuarios federados en mi intranet**. Esta configuración indica que la notificación interna de la red corporativa, la cual emiten los Servicios de federación de Active Directory (AD FS), es de confianza y se utiliza para identificar al usuario como si estuviera en la red corporativa. Para obtener más información, vea [Habilitar la función de direcciones IP de confianza mediante el acceso condicional](../authentication/howto-mfa-mfasettings.md#enable-the-trusted-ips-feature-by-using-conditional-access).

Tras activar esta opción, incluida la ubicación con nombre, las **IP de confianza de MFA** se aplicarán a todas las directivas que tengan esta opción seleccionada.

Para las aplicaciones móviles y de escritorio que hayan tenido sesiones de larga duración, el acceso condicional se vuelve a evaluar periódicamente. El valor predeterminado es de una hora. Cuando la notificación dentro de la red corporativa solo se emite en el momento de la autenticación inicial, puede que Azure AD no tenga ninguna lista de intervalos de IP de confianza. En este caso, es más difícil determinar si el usuario sigue conectado a la red corporativa:

1. Compruebe si la dirección IP del usuario está en uno de los intervalos IP de confianza.
1. Compruebe si los tres primeros octetos de la dirección IP del usuario coinciden con los tres primeros octetos de la dirección IP de la autenticación inicial. La dirección IP se compara con la autenticación inicial cuando se emitió la notificación interna de red corporativa y se validó la ubicación del usuario.

Si se produce un error en los dos pasos, se considera que el usuario ya no está conectado a ninguna dirección IP de confianza.

## <a name="location-condition-in-policy"></a>Condición de ubicación de la directiva

Al configurar la condición de ubicación, puede distinguir entre:

- Cualquier ubicación
- Todas las ubicaciones de confianza
- Ubicaciones seleccionadas

### <a name="any-location"></a>Cualquier ubicación

De manera predeterminada, al seleccionar **Cualquier ubicación**, se aplicará una directiva a todas las direcciones IP, lo que incluye cualquier dirección de Internet. Esta configuración no está limitada a las direcciones IP que haya configurado como ubicación con nombre. Al seleccionar **Cualquier ubicación**, todavía puede excluir determinadas ubicaciones de una directiva. Por ejemplo, puede aplicar una directiva en todas las ubicaciones (excepto en aquellas que sean de confianza) para establecer el ámbito en todas las ubicaciones menos en la red corporativa.

### <a name="all-trusted-locations"></a>Todas las ubicaciones de confianza

Esta opción se aplica a:

- Todas las ubicaciones que se hayan marcado como ubicación de confianza
- **IP de confianza de MFA** (si están configuradas)

### <a name="selected-locations"></a>Ubicaciones seleccionadas

Con esta opción, puede seleccionar una o varias ubicaciones con nombre. Para una directiva a la que se aplicará esta configuración, el usuario debe conectarse desde cualquiera de las ubicaciones seleccionadas. Al **seleccionar** el control de selección de la red con nombre que se muestra, se abre la lista de redes con nombre. En la lista también se muestra si la ubicación de red se ha marcado como de confianza. La ubicación con nombre llamada **IP de confianza de MFA** se utiliza para incluir los valores de IP que pueden configurarse en la página de configuración del servicio de la autenticación multifactor.

## <a name="ipv6-traffic"></a>Tráfico de IPv6

De forma predeterminada, las directivas de acceso condicional se aplicarán a todo el tráfico de IPv6. Puede excluir intervalos de direcciones IPv6 específicos de una directiva de acceso condicional si no quiere que se apliquen directivas para intervalos IPv6 específicos. Por ejemplo, si no quiere aplicar una directiva en la red corporativa, y la red corporativa está hospedada en intervalos IPv6 públicos.

### <a name="identifying-ipv6-traffic-in-the-azure-ad-sign-in-activity-reports"></a>Identificación del tráfico IPv6 en los informes de actividad de inicio de sesión de Azure AD

Para detectar el tráfico IPv6 en el inquilino, vaya a [Informes de actividad de inicio de sesión en Azure AD](../reports-monitoring/concept-sign-ins.md). Una vez que haya abierto el informe de actividad, agregue la columna "dirección IP". Esta columna le permitirá identificar el tráfico IPv6.

También puede ver cuál es la dirección IP del cliente; para ello, haga clic en una fila del informe y, después, vaya a la pestaña "Ubicación" en los detalles de la actividad de inicio de sesión. 

### <a name="when-will-my-tenant-have-ipv6-traffic"></a>¿Cuándo tendrá mi inquilino tráfico IPv6?

Azure Active Directory (Azure AD) no admite actualmente conexiones de red directas que usan IPv6. No obstante, hay algunos casos en los que el tráfico de autenticación se realiza mediante proxy a través de otro servicio. En estos casos, la dirección IPv6 se usará durante la evaluación de la directiva.

La mayor parte del tráfico IPv6 que se dirige mediante proxy a Azure AD procede de Microsoft Exchange Online. Cuando estén disponibles, Exchange preferirá conexiones IPv6. **Por lo tanto, si tiene directivas de acceso condicional para Exchange que se han configurado para intervalos IPv4 específicos, querrá asegurarse de que también ha agregado los intervalos IPv6 de las organizaciones.** Si no se incluyen intervalos IPv6, se producirá un comportamiento inesperado en los dos casos siguientes:

- Cuando se usa un cliente de correo para conectarse a Exchange Online con autenticación heredada, Azure AD puede recibir una dirección IPv6. La solicitud de autenticación inicial va a Exchange y, después, se dirige mediante proxy a Azure AD.
- Si se usa Outlook Web Access (OWA) en el explorador, esta aplicación comprueba periódicamente que se están cumpliendo todas las directivas de acceso condicional. Esta comprobación sirve para detectar los casos en los que un usuario puede haber pasado de una dirección IP permitida a una nueva ubicación, como una cafetería de la calle. En este caso, si se usa una dirección IPv6 que no está en un intervalo configurado, es posible que el usuario vea su sesión interrumpida y se le redirija a Azure AD para que se vuelva a autenticar. 

Además, si usa redes virtuales de Azure, tendrá tráfico procedente de una dirección IPv6. Si el tráfico de red virtual está bloqueado por una directiva de acceso condicional, consulte el registro de inicio de sesión de Azure AD. Una vez que haya identificado el tráfico, puede obtener la dirección IPv6 que se está usando y excluirla de la directiva. 

> [!NOTE]
> Si quiere especificar un intervalo CIDR de IP para una sola dirección, aplique la máscara de /128 bits. Por ejemplo, si tiene la dirección IPv6 2607:fb90:b27a:6f69:f8d5:dea0:fb39:74a y quiere excluirla como un intervalo, usará 2607:fb90:b27a:6f69:f8d5:dea0:fb39:74a/128.

## <a name="what-you-should-know"></a>Qué debería saber

### <a name="when-is-a-location-evaluated"></a>¿Cuando se evalúa una ubicación?

Las directivas de acceso condicional se evalúan cuando:

- Un usuario inicia sesión por primera vez en una aplicación web, móvil o de escritorio.
- Una aplicación móvil o de escritorio que usa la autenticación moderna, con un token de actualización para adquirir un nuevo token de acceso. De forma predeterminada, esta comprobación se realiza una vez por hora.

Esta comprobación significa que, para que las aplicaciones móviles y de escritorio usen la autenticación moderna, se detecta un cambio en la ubicación dentro de la hora siguiente al cambio de la ubicación de red. Para las aplicaciones de escritorio y móviles que no usan la autenticación moderna, la directiva se aplica para cada solicitud de token. La frecuencia de la solicitud varía en función de la aplicación. De forma similar, para las aplicaciones web, la directiva se aplica en el primer inicio de sesión y es adecuada para la duración de la sesión en la aplicación web. Debido a las diferencias en la duración de la sesión de las aplicaciones, el tiempo de una evaluación de directiva a otra también varía. Cada vez que la aplicación solicita un nuevo token de inicio de sesión, la directiva se aplica.

De manera predeterminada, Azure AD emite un token cada hora. Tras salir de la red corporativa, la directiva se aplica a las aplicaciones que usan la autenticación moderna en un plazo de una hora.

### <a name="user-ip-address"></a>Dirección IP del usuario

La dirección IP que se usa en la evaluación de directivas es la dirección IP pública del usuario. Para los dispositivos conectados a una red privada, esta no es la dirección IP de cliente del dispositivo del usuario conectado a la intranet, sino la dirección que utiliza la red para conectarse a la red pública de Internet.

### <a name="bulk-uploading-and-downloading-of-named-locations"></a>Carga y descarga masiva de ubicaciones con nombre

Al crear o actualizar ubicaciones con nombre, en el caso de las actualizaciones masivas, puede cargar o descargar un archivo CSV con los intervalos IP. Una carga reemplaza los intervalos IP que aparecen en la lista por los del archivo. Cada fila del archivo contiene un intervalo de direcciones IP en formato CIDR.

### <a name="cloud-proxies-and-vpns"></a>Servidores proxy en la nube y soluciones VPN

Cuando utiliza un proxy hospedado en la nube o una solución VPN, la dirección IP que utiliza Azure AD al evaluar una directiva es la dirección IP del proxy. No se usa el encabezado X-Forwarded-For (XFF) que contiene la dirección IP pública del usuario, ya que no hay ninguna validación de que provenga de un origen de confianza y podría ser un método de falsificación de la dirección IP.

Cuando hay en funcionamiento un servidor proxy en la nube, se puede utilizar una directiva que se usa para requerir un dispositivo unido a Azure AD híbrido o la notificación interna de la red corporativa desde AD FS.

### <a name="api-support-and-powershell"></a>Compatibilidad con la API y PowerShell

Una versión preliminar de Graph API para ubicaciones con nombre está disponible. Para más información, consulte [namedLocation API](/graph/api/resources/namedlocation).

## <a name="next-steps"></a>Pasos siguientes

- Configure una directiva de acceso condicional mediante la ubicación; consulte el artículo [Acceso condicional: Bloquear el acceso por ubicación.](howto-conditional-access-policy-location.md)
