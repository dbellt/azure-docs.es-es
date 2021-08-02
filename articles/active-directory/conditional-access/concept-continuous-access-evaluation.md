---
title: Evaluación continua de acceso en Azure AD
description: Respuesta más rápida a los cambios en el estado del usuario con la evaluación continua de acceso en Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: conceptual
ms.date: 04/27/2021
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jlu
ms.collection: M365-identity-device-management
ms.openlocfilehash: c615c3b57d0c4ebfdbffdc1461f2289d4b8c4256
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111438277"
---
# <a name="continuous-access-evaluation"></a>Evaluación continua de acceso

La expiración y actualización de los tokens es un mecanismo estándar del sector. Cuando una aplicación cliente como Outlook se conecta a un servicio como Exchange Online, las solicitudes de API se autorizan mediante tokens de acceso de OAuth 2.0. De forma predeterminada, los tokens de acceso son válidos durante una hora; cuando expiran, se redirige al cliente de nuevo a Azure AD para actualizarlos. Ese período de actualización proporciona una oportunidad para volver a evaluar las directivas de acceso de los usuarios. Por ejemplo: podemos optar por no actualizar el token debido a una directiva de acceso condicional o porque el usuario se ha deshabilitado en el directorio. 

Los clientes han manifestado dudas sobre el retraso entre el momento en que cambian las condiciones para el usuario, como la ubicación de red o el robo de credenciales y cuándo se pueden aplicar directivas relacionadas con ese cambio. Aunque hemos experimentado con el enfoque directo de duraciones de tokens reducidas, hemos descubierto que pueden degradar las experiencias y la confiabilidad de los usuarios y no eliminan los riesgos.

La respuesta oportuna a las infracciones de las directivas o a los problemas de seguridad requiere realmente una "conversación" entre el emisor del token, como Azure AD, y el usuario de confianza, como Exchange Online. Esta conversación bidireccional nos proporciona dos funcionalidades importantes. El usuario de confianza puede advertir cuándo han cambiado las cosas, como un cliente que procede de una nueva ubicación, e indicárselo al emisor del token. También proporciona al emisor del token una manera de indicar al usuario de confianza que deje de respetar los tokens de un usuario determinado debido a que la cuenta esté en peligro, se haya deshabilitado u otros problemas. El mecanismo para esta conversación es la evaluación continua de acceso (CAE). Aunque el objetivo es que la respuesta sea casi en tiempo real, en algunos casos se puede observar una latencia de hasta 15 minutos debido al tiempo de propagación de los eventos.

La implementación inicial de la evaluación continua de acceso se centra en Exchange, Teams y SharePoint Online.

Para preparar las aplicaciones para el uso de CAE, consulte [Uso de las API habilitadas para la evaluación continua de acceso en las aplicaciones](../develop/app-resilience-continuous-access-evaluation.md).

### <a name="key-benefits"></a>Ventajas principales

- Terminación del usuario y cambio o restablecimiento de la contraseña: la revocación de la sesión de usuario se aplicará casi en tiempo real.
- Cambio de ubicación de red: las directivas de ubicación del acceso condicional se aplicarán casi en tiempo real.
- La exportación de tokens a una máquina fuera de una red de confianza se puede evitar con las directivas de ubicación del acceso condicional.

## <a name="scenarios"></a>Escenarios 

Hay dos escenarios que conforman la evaluación continua de acceso: la evaluación de eventos críticos y la evaluación de directivas de acceso condicional.

### <a name="critical-event-evaluation"></a>Evaluación de eventos críticos

La evaluación continua de acceso se implementa mediante la habilitación de servicios, como Exchange Online, SharePoint Online y Teams, para suscribirse a eventos críticos en Azure AD de modo que dichos eventos se puedan evaluar y aplicar casi en tiempo real. La evaluación de eventos críticos no se basa en las directivas de acceso condicional, por lo que está disponible en cualquier inquilino. Actualmente se evalúan los siguientes eventos:

- La cuenta de usuario se ha eliminado o deshabilitado.
- La contraseña de un usuario ha cambiado o se ha restablecido.
- Se habilita la autenticación multifactor para el usuario.
- El administrador revoca explícitamente todos los tokens de actualización de un usuario.
- Riesgo de usuario elevado detectado por Azure AD Identity Protection

Este proceso habilita el escenario en el que los usuarios pierden el acceso a los archivos, el correo electrónico, el calendario o las tareas de SharePoint Online de la organización y Teams desde las aplicaciones cliente de Microsoft 365 en cuestión de minutos después de uno de estos eventos críticos. 

> [!NOTE] 
> Teams y SharePoint Online no admiten aún evento de riesgo de usuario.

### <a name="conditional-access-policy-evaluation-preview"></a>Evaluación de directivas de acceso condicional (versión preliminar)

Exchange y SharePoint pueden sincronizar las directivas de acceso condicional principales para que se puedan evaluar dentro del propio servicio.

Este proceso habilita el escenario en el que los usuarios pierden el acceso a los archivos, el correo electrónico, el calendario o las tareas de la organización desde las aplicaciones cliente de Microsoft 365 o SharePoint Online inmediatamente después de un cambio de ubicación de red.

> [!NOTE]
> No se admiten todas las combinaciones de proveedores de recursos y aplicaciones. Consulte la tabla siguiente. Office hace referencia a Word, Excel y PowerPoint.

| | Outlook Web | Outlook Win32 | Outlook iOS | Outlook Android | Outlook Mac |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **SharePoint Online** | Compatible | Compatible | Compatible | Compatible | Compatible |
| **Exchange Online** | Compatible | Compatible | Compatible | Compatible | Compatible |

| | Aplicaciones web de Office | Aplicaciones Win32 de Office | Office para iOS | Office para Android | Office para Mac |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **SharePoint Online** | No compatible | Compatible | Compatible | Compatible | Compatible |
| **Exchange Online** | No compatible | Compatible | Compatible | Compatible | Compatible |

| | OneDrive web | OneDrive Win32 | OneDrive iOS | OneDrive Android | OneDrive Mac |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **SharePoint Online** | Compatible | Compatible | Compatible | Compatible | Compatible |

### <a name="client-side-claim-challenge"></a>Desafío de notificaciones del lado cliente

Antes de la evaluación continua de acceso, los clientes siempre intentaban reproducir el token de acceso desde su memoria caché, siempre y cuando no haya expirado. Con CAE, presentamos un nuevo caso en el que un proveedor de recursos puede rechazar un token aunque no haya expirado. Para informar a los clientes de que omitan su memoria caché aunque los tokens almacenados en caché no hayan expirado, presentamos un mecanismo llamado **desafío de notificaciones** para indicar que se ha rechazado el token y que Azure AD debe emitir un nuevo token de acceso. CAE requiere una actualización de cliente para comprender el desafío de notificaciones. La versión más reciente de las siguientes aplicaciones es compatible con el desafío de notificaciones:

| | Web | Win32 | iOS | Android | Mac |
| :--- | :---: | :---: | :---: | :---: | :---: |
| **Outlook** | Compatible | Compatible | Compatible | Compatible | Compatible |
| **Teams** | Compatible | Compatible | Compatible | Compatible | Compatible |
| **Office** | No compatible | Compatible | Compatible | Compatible | Compatible |
| **OneDrive** | Compatible | Compatible | Compatible | Compatible | Compatible |

### <a name="token-lifetime"></a>Duración del token

Dado que el riesgo y la directiva se evalúan en tiempo real, los clientes que negocien sesiones con reconocimiento de evaluación continua de acceso confiarán en CAE en lugar de en las directivas de vigencia del token de acceso estático existente, lo que significa que la directiva de vigencia del token configurable ya no se respetará en los clientes compatibles con CAE que negocien las sesiones con reconocimiento de CAE.

La vigencia del token se incrementa para tener una duración larga, hasta 28 horas, en las sesiones de CAE. La revocación se basa en los eventos críticos y en la evaluación de directivas, no solo en un período de tiempo arbitrario. Este cambio aumenta la estabilidad de las aplicaciones sin afectar a la postura de seguridad. 

Si no usa clientes compatibles con CAE, la vigencia del token de acceso predeterminada seguirá siendo de 1 hora, a menos que haya configurado la vigencia del token de acceso con la característica en versión preliminar [Vigencia de token configurable (CTL)](../develop/active-directory-configurable-token-lifetimes.md).

## <a name="example-flows"></a>Flujos de ejemplo

### <a name="user-revocation-event-flow"></a>Flujo de eventos de revocación de usuario:

![Flujo de eventos de revocación de usuario](./media/concept-continuous-access-evaluation/user-revocation-event-flow.png)

1. Un cliente compatible con CAE presenta credenciales o un token de actualización a Azure AD para solicitar un token de acceso para algún recurso.
1. Se devuelve un token de acceso junto con otros artefactos al cliente.
1. El administrador [revoca explícitamente todos los tokens de actualización de un usuario](/powershell/module/azuread/revoke-azureaduserallrefreshtoken). Se enviará un evento de revocación al proveedor de recursos desde Azure AD.
1. Se presenta un token de acceso al proveedor de recursos. El proveedor de recursos evalúa la validez del token y comprueba si hay algún evento de revocación para el usuario. El proveedor de recursos utiliza esta información para decidir si se concede acceso al recurso.
1. En este caso, el proveedor de recursos deniega el acceso y envía un error 401 y un desafío de notificaciones al cliente.
1. El cliente compatible con CAE comprende el desafío de notificaciones 401+. Omite las cachés y vuelve al paso 1, enviando su token de actualización junto con el desafío de notificaciones de vuelta a Azure AD. A continuación, Azure AD volverá a evaluar todas las condiciones y solicitará al usuario que vuelva a autenticarse en este caso.

### <a name="user-condition-change-flow-preview"></a>Flujo del cambio de condición del usuario (versión preliminar):

En el ejemplo siguiente, un administrador de acceso condicional ha configurado una directiva de acceso condicional basada en la ubicación para permitir solo el acceso desde intervalos IP específicos:

![Flujo de eventos de condición del usuario](./media/concept-continuous-access-evaluation/user-condition-change-flow.png)

1. Un cliente compatible con CAE presenta credenciales o un token de actualización a Azure AD para solicitar un token de acceso para algún recurso.
1. Azure AD evalúa todas las directivas de acceso condicional para ver si el usuario y el cliente cumplen las condiciones.
1. Se devuelve un token de acceso junto con otros artefactos al cliente.
1. El usuario sale de un intervalo de direcciones IP permitido.
1. El cliente presenta un token de acceso al proveedor de recursos desde fuera de un intervalo de direcciones IP permitido.
1. El proveedor de recursos evalúa la validez del token y comprueba la directiva de ubicación sincronizada desde Azure AD.
1. En este caso, el proveedor de recursos deniega el acceso y envía un error 401 y un desafío de notificaciones al cliente porque no proviene del intervalo IP permitido.
1. El cliente compatible con CAE comprende el desafío de notificaciones 401+. Omite las cachés y vuelve al paso 1, enviando su token de actualización junto con el desafío de notificaciones de vuelta a Azure AD. Azure AD vuelve a evaluar todas las condiciones y denegará el acceso en este caso.

## <a name="enable-or-disable-cae-preview"></a>Habilitación o deshabilitación de CAE (versión preliminar)

1. Inicie sesión en **Azure Portal** como administrador de acceso condicional, administrador de seguridad o administrador global.
1. Vaya a **Azure Active Directory** > **Seguridad** > **Evaluación continua de acceso**.
1. Elija **Habilitar versión preliminar**.
1. Seleccione **Guardar**.

En esta página, puede limitar opcionalmente los usuarios y grupos que estarán sujetos a la versión preliminar.

> [!WARNING]
> Para deshabilitar la evaluación continua de acceso, seleccione **Habilitar versión preliminar**, **Deshabilitar versión preliminar** y **Guardar**.

> [!NOTE]
>Puede consultar Microsoft Graph mediante [**continuousAccessEvaluationPolicy**](/graph/api/continuousaccessevaluationpolicy-get?view=graph-rest-beta&tabs=http#request-body) para comprobar la configuración de CAE en el inquilino. Una respuesta HTTP 200 y el cuerpo de respuesta asociado indican si CAE está habilitado o deshabilitado en el inquilino. CAE no está configurado si Microsoft Graph devuelve una respuesta HTTP 404.

![Habilitar la versión preliminar de CAE en Azure Portal](./media/concept-continuous-access-evaluation/enable-cae-preview.png)

## <a name="troubleshooting"></a>Solución de problemas

### <a name="supported-location-policies"></a>Directivas de ubicación admitidas

En el caso de CAE, solo tenemos información sobre ubicaciones con nombre basadas en IP. No tenemos información sobre otras opciones de configuración de ubicación, como las [direcciones IP de confianza de MFA](../authentication/howto-mfa-mfasettings.md#trusted-ips) o las ubicaciones basadas en países. Cuando el usuario procede de una dirección IP de confianza de MFA o de ubicaciones de confianza que incluyen las direcciones IP de confianza de MFA o la ubicación de país, CAE no se aplica después de que el usuario se mueva a otra ubicación. En esos casos, se emitirá un token de CAE de 1 hora sin comprobación de cumplimiento de IP instantánea.

> [!IMPORTANT]
> Al configurar ubicaciones para la evaluación continua de acceso, use solo la [condición de ubicación de acceso condicional basado en IP](../conditional-access/location-condition.md) y configure todas las direcciones IP, **incluidas las IPv4 e IPv6**, que el proveedor de identidades y el proveedor de recursos podrán ver. No use condiciones de ubicación de país o la característica de direcciones IP de confianza que está disponible en la página de configuración del servicio Multi-Factor Authentication de Azure AD.

### <a name="ip-address-configuration"></a>Configuración de dirección IP

El proveedor de identidades y los proveedores de recursos pueden ver diferentes direcciones IP. Esta falta de coincidencia puede ocurrir debido a las implementaciones del proxy de red de la organización o a configuraciones de IPv4/IPv6 incorrectas entre el proveedor de identidades y el proveedor de recursos. Por ejemplo:

- El proveedor de identidades ve una dirección IP del cliente.
- El proveedor de recursos ve otra dirección IP del cliente después de pasar por un servidor proxy.
- La dirección IP que ve el proveedor de identidades forma parte de un intervalo IP permitido en la directiva, pero la dirección IP que ve el proveedor de recursos no.

Si se produce este escenario en el entorno, Azure AD emitirá un token de CAE de una hora y no aplicará el cambio de ubicación del cliente con el fin de evitar bucles infinitos. Incluso en este caso, se mejora la seguridad en comparación con los tokens de una hora tradicionales, ya que se evalúan [otros eventos](#critical-event-evaluation) además de los eventos de cambio de ubicación del cliente.

### <a name="office-and-web-account-manager-settings"></a>Configuración de Office y del Administrador de cuentas web

| Canal de actualización de Office | DisableADALatopWAMOverride | DisableAADWAM |
| --- | --- | --- |
| Canal semestral para empresas | Si se establece en Enabled o en 1, no se admite CAE. | Si se establece en Enabled o en 1, no se admite CAE. |
| Canal actual <br> or <br> Canal mensual para empresas | Se admite CAE independientemente de la configuración. | Se admite CAE independientemente de la configuración. |

Para obtener una explicación de los canales de actualización de Office, consulte [Información general sobre los canales de actualización de Aplicaciones de Microsoft 365](/deployoffice/overview-update-channels). Se recomienda que las organizaciones no deshabiliten el Administrador de cuentas web (WAM).

### <a name="group-membership-and-policy-update-effective-time"></a>Tiempo válido de actualización de las directivas y de la pertenencia a grupos

La actualización de las directivas y de la pertenencia a grupos realizada por los administradores puede tardar hasta un día en aplicarse. Se han realizado algunas optimizaciones de las actualizaciones de directivas que reducen el retraso a dos horas. Sin embargo, no cubren aún todos los escenarios. 

Si se produce una emergencia y necesita actualizar las directivas o que el cambio de la pertenencia a un grupo se aplique a determinados usuarios inmediatamente, debe usar este [comando de PowerShell](/powershell/module/azuread/revoke-azureaduserallrefreshtoken) o "Revocar sesión" en la página del perfil de usuario para revocar la sesión de los usuarios, lo que garantizará que las directivas actualizadas se apliquen inmediatamente.

### <a name="coauthoring-in-office-apps"></a>Coautoría en aplicaciones de Office

Cuando varios usuarios colaboran en el mismo documento al mismo tiempo, es posible que CAE no pueda revocar inmediatamente el acceso del usuario al documento en función de la revocación del usuario o los eventos de cambio de directiva. En este caso, el usuario pierde el acceso por completo después de cerrar el documento, cerrar Word, Excel o PowerPoint o después de un período de 10 horas.

Para reducir este tiempo, un administrador de SharePoint puede reducir opcionalmente la duración máxima de las sesiones de coautoría para los documentos almacenados en SharePoint Online y OneDrive para la Empresa mediante la [configuración de una directiva de ubicación de red en SharePoint Online](/sharepoint/control-access-based-on-network-location). Una vez que se cambia esta configuración, la vigencia máxima de las sesiones de coautoría se reducirá a 15 minutos, pero se puede ajustar más mediante el comando de PowerShell de SharePoint Online "Set-SPOTenant –IPAddressWACTokenLifetime".

### <a name="enable-after-a-user-is-disabled"></a>Habilitar después de deshabilitar un usuario

Si habilita un usuario justo después de deshabilitarlo. Habrá una cierta latencia hasta que la cuenta se pueda habilitar. SPO and Teams tendrán 15 minutos de retraso. El retraso es de 35 a 40 minutos para EXO.

## <a name="faqs"></a>Preguntas más frecuentes

### <a name="how-will-cae-work-with-sign-in-frequency"></a>¿Cómo funcionará CAE con la frecuencia de inicio de sesión?

La frecuencia de inicio de sesión se respetará con o sin CAE.

## <a name="next-steps"></a>Pasos siguientes

- [Anuncio de evaluación continua de acceso](https://techcommunity.microsoft.com/t5/azure-active-directory-identity/moving-towards-real-time-policy-and-security-enforcement/ba-p/1276933)
- [Uso de las API habilitadas para la evaluación continua de acceso en las aplicaciones](../develop/app-resilience-continuous-access-evaluation.md)
- [Desafíos de notificaciones, solicitudes de notificaciones y funcionalidades de cliente](../develop/claims-challenge.md)
