---
title: 'Autenticación de Azure Virtual Desktop: Azure'
description: Métodos de autenticación de Azure Virtual Desktop.
services: virtual-desktop
author: Heidilohr
ms.service: virtual-desktop
ms.topic: conceptual
ms.date: 05/26/2021
ms.author: helohr
manager: femila
ms.openlocfilehash: bb5341f1f80350ce14b99a6ca5cf27343bd20675
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745392"
---
# <a name="supported-authentication-methods"></a>Métodos de autenticación admitidos

En este artículo, le mostraremos una breve descripción de los tipos de autenticación que puede usar en Azure Virtual Desktop.

## <a name="session-host-authentication"></a>Autenticación de host de sesión

Azure Virtual Desktop admite NT LAN Manager (NTLM) y Kerberos para la autenticación del host de sesión. Sin embargo, para utilizar Kerberos, el cliente debe obtener vales de seguridad de Kerberos de un servicio de Centro de distribución de claves (KDC) que se ejecute en un controlador de dominio. Para obtener vales, el cliente necesita una línea de visión directa al controlador de dominio. Para ello, puede usar la red corporativa. También puede usar una conexión VPN a la red corporativa o configurar un [servidor proxy KDC](key-distribution-center-proxy.md).

Estos son los métodos de inicio de sesión admitidos actualmente:

- Cliente de escritorio de Windows
    - Nombre de usuario y contraseña
    - Tarjeta inteligente
    - Windows Hello para empresas (solo confianza de certificados)
- Cliente de Microsoft Store
    - Nombre de usuario y contraseña
- Cliente web
    - Nombre de usuario y contraseña
- Android
    - Nombre de usuario y contraseña
- iOS
    - Nombre de usuario y contraseña
- macOS
    - Nombre de usuario y contraseña

>[!NOTE]
>La tarjeta inteligente y Windows Hello para empresas solo pueden usar Kerberos para iniciar sesión. Para iniciar la sesión con Kerberos se necesita una línea de visión al controlador de dominio o un [servidor proxy KDC](key-distribution-center-proxy.md).

## <a name="hybrid-identity"></a>Identidad híbrida

Azure Virtual Desktop admite [identidades híbridas](../active-directory/hybrid/whatis-hybrid-identity.md) a través de Azure Active Directory (AD), incluidas las federadas mediante Servicios de federación de Active Directory (AD FS). Dado que debe ser posible detectar a los usuarios a través de Azure AD, Azure Virtual Desktop no admite implementaciones de Active Directory independientes con ADFS.

## <a name="single-sign-on-sso"></a>Inicio de sesión único (SSO)

Azure Virtual Desktop admite una [inicio de sesión único mediante Servicios de federación de Active Directory (AD FS)](configure-adfs-sso.md) para Windows y clientes web.

De lo contrario, la única manera de evitar que le pidan sus credenciales del host de sesión es guardarlas en el cliente. Se recomienda hacer esto solo con dispositivos seguros a fin de evitar que otros usuarios tengan acceso a los recursos.

## <a name="next-steps"></a>Pasos siguientes

¿Tiene curiosidad sobre otras formas de proteger la implementación? Consulte [Procedimientos recomendados de seguridad](security-guide.md).
