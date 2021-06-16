---
title: Preguntas más frecuentes sobre los certificados de administración de aplicaciones de Azure Active Directory
description: Obtenga respuestas a las preguntas más frecuentes (P + F) sobre la administración de certificados para aplicaciones mediante Azure Active Directory como proveedor de identidades (IdP).
services: active-directory
author: mtillman
manager: CelesteDG
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: reference
ms.date: 03/19/2021
ms.author: mtillman
ms.reviewer: secherka, mifarca, shchaur, shravank, sureshja
ms.openlocfilehash: 0f0bfe38dc28586d2fdfc40777dc0c599ab7b307
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112077417"
---
# <a name="azure-active-directory-azure-ad-application-management-certificates-frequently-asked-questions"></a>Preguntas más frecuentes sobre los certificados de administración de aplicaciones de Azure Active Directory (Azure AD)

Esta página responde a las preguntas más frecuentes sobre la administración de certificados para aplicaciones que usan Azure Active Directory (Azure AD) como proveedor de identidades (IdP).

## <a name="is-there-a-way-to-generate-a-list-of-expiring-saml-signing-certificates"></a>¿Hay alguna manera de generar una lista de los certificados de firma de SAML que van a expirar?

Puede exportar todas las aplicaciones con secretos y certificados que van a expirar y sus propietarios para las aplicaciones especificadas desde el directorio a un archivo CSV mediante los [scripts de PowerShell](app-management-powershell-samples.md). 

## <a name="where-can-i-find-the-information-about-soon-to-expire-certificates-renewal-steps"></a>¿Dónde puedo encontrar la información sobre los pasos de renovación de los certificados que expirarán pronto?

Puede encontrar los pasos [aquí](manage-certificates-for-federated-single-sign-on.md#renew-a-certificate-that-will-soon-expire).

## <a name="how-can-i-customize-the-expiration-date-for-the-certificates-issued-by-azure-ad"></a>¿Cómo puedo personalizar la fecha de expiración de los certificados que emitió Azure AD?

De manera predeterminada, Azure AD configura un certificado para que expire después de tres años cuando se crea automáticamente durante la configuración de inicio de sesión único de SAML. Dado que no se puede cambiar la fecha de un certificado después de guardarlo, tendrá que crear un certificado nuevo. Para ver qué pasos tiene que dar para hacerlo, consulte [Personalización de la fecha de expiración para el certificado de federación y sustituirlo por un certificado nuevo](manage-certificates-for-federated-single-sign-on.md#customize-the-expiration-date-for-your-federation-certificate-and-roll-it-over-to-a-new-certificate).

## <a name="how-can-i-automate-the-certificates-expiration-notifications"></a>¿Cómo puedo automatizar las notificaciones de expiración de los certificados?

Azure AD enviará una notificación por correo electrónico 60, 30 y 7 días antes de que expire el certificado de SAML. Puede agregar varias direcciones de correo electrónico para recibir notificaciones. 

> [!NOTE]
> Puede agregar hasta 5 direcciones de correo electrónico a la lista de notificaciones (incluida la dirección de correo electrónico del administrador que agregó la aplicación). Si necesita enviar notificaciones a más personas, use los mensajes de correo electrónico de la lista de distribución. 

Para especificar los correos electrónicos a los que quiere que se envíen las notificaciones, consulte [Inclusión de direcciones de correo electrónico para notificar la expiración del certificado](manage-certificates-for-federated-single-sign-on.md#add-email-notification-addresses-for-certificate-expiration).

Tenga en cuenta que no hay ninguna opción para editar o personalizar estas notificaciones de correo electrónico que se reciben de `aadnotification@microsoft.com`. Sin embargo, puede exportar los registros de aplicaciones con secretos y certificados que van a expirar a través de los [scripts de PowerShell](app-management-powershell-samples.md).

## <a name="who-can-update-the-certificates"></a>¿Quién puede actualizar los certificados?

El propietario de la aplicación, el administrador global o el administrador de la aplicación pueden actualizar los certificados a través de la interfaz de usuario de Azure Portal, PowerShell o Microsoft Graph.

## <a name="i-need-more-details-about-certificate-signing-options"></a>Necesito más detalles sobre las opciones de firma de certificados.

En Azure AD, puede configurar las opciones de firma de certificado y el algoritmo correspondiente. Para obtener más información, consulte [Opciones avanzadas de firma de certificados de token SAML para aplicaciones de Azure AD](certificate-signing-options.md).

## <a name="i-need-to-replace-the-certificate-for-azure-ad-application-proxy-applications-and-need-more-instructions"></a>Necesito reemplazar el certificado para las aplicaciones de Application Proxy de Azure AD y necesito más instrucciones.

Para reemplazar los certificados de las aplicaciones Application Proxy de Azure AD, consulte [Muestra de PowerShell: reemplazar el certificado en aplicaciones de Application Proxy](../app-proxy/scripts/powershell-get-custom-domain-replace-cert.md).

## <a name="how-do-i-manage-certificates-for-custom-domains-in-azure-ad-application-proxy"></a>¿Cómo administro los certificados de dominios personalizados en Application Proxy de Azure AD?

Para configurar una aplicación local para que use un dominio personalizado, necesita un dominio personalizado de Azure Active Directory comprobado, un certificado PFX para dicho dominio y una aplicación local para configurar. Para obtener más información, consulte [Dominios personalizados en Application Proxy de Azure AD](../app-proxy/application-proxy-configure-custom-domain.md). 

## <a name="i-need-to-update-the-token-signing-certificate-on-the-application-side-where-can-i-get-it-on-azure-ad-side"></a>Necesito actualizar el certificado de firma de tokens en el lado de la aplicación. ¿Dónde puedo obtenerlo en Azure AD?

Para renovar un certificado SAML X.509, consulte [Certificado de firma de SAML](configure-saml-single-sign-on.md#saml-signing-certificate).

## <a name="what-is-azure-ad-signing-key-rollover"></a>¿Qué es la sustitución de la clave de firma de Azure AD?

Puede encontrar más información [aquí](../develop/active-directory-signing-key-rollover.md). 

## <a name="how-do-i-renew-application-token-encryption-certificate"></a>¿Cómo renuevo el certificado de cifrado de tokens de la aplicación?

Para renovar un certificado de cifrado de tokens de la aplicación, consulte [Cómo renovar un certificado de cifrado de tokens para una aplicación empresarial](howto-saml-token-encryption.md). 

## <a name="how-do-i-renew-application-token-signing-certificate"></a>¿Cómo renuevo el certificado de firma de tokens de la aplicación?

Para renovar un certificado de firma de tokens de la aplicación, consulte [Cómo renovar un certificado de firma de tokens para una aplicación empresarial](manage-certificates-for-federated-single-sign-on.md).

## <a name="how-do-i-update-azure-ad-after-changing-my-federation-certificates"></a>Cómo actualizo Azure AD después de cambiar los certificados de federación?

Para actualizar Azure AD después de cambiar los certificados de federación, consulte [Renovación de certificados de federación para Microsoft 365 y Azure Active Directory](../hybrid/how-to-connect-fed-o365-certs.md).
