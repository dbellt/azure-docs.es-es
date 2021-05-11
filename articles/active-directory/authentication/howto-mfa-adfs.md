---
title: 'Protección de los recursos con Azure AD MFA y ADFS: Azure Active Directory'
description: En esta página de Azure AD Multi-Factor Authentication se describe cómo empezar a trabajar con Azure AD MFA y AD FS 2.0 en la nube.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 04/29/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: d8c0628d9e2d0d89fcb84569d227f592b5ad0943
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108285581"
---
# <a name="securing-cloud-resources-with-azure-ad-multi-factor-authentication-and-ad-fs"></a>Protección de recursos en la nube con Azure AD Multi-Factor Authentication y AD FS

Si su organización está federada con Azure Active Directory, use Azure AD Multi-Factor Authentication o los servicios de federación de Active Directory (AD FS) para proteger los recursos a los que se accede mediante Azure AD. Utilice los siguientes procedimientos para proteger recursos de Azure Active Directory mediante Azure AD Multi-Factor Authentication o Servicios de federación de Active Directory.

>[!NOTE]
>Para proteger el recurso de Azure AD, se recomienda exigir MFA a través de una [directiva de acceso condicional](https://docs.microsoft.com/azure/active-directory/conditional-access/howto-conditional-access-policy-all-users-mfa), establecer el valor de dominio SupportsMfa en $True y [emitir la notificación multipleauthn](https://docs.microsoft.com/azure/active-directory/authentication/howto-mfa-adfs#secure-azure-ad-resources-using-ad-fs) cuando un usuario completa correctamente la verificación en dos pasos.

## <a name="secure-azure-ad-resources-using-ad-fs"></a>Protección de los recursos de Azure AD mediante AD FS

Para proteger los recursos de la nube, configure una regla de notificaciones para que los servicios de federación de Active Directory emitan la notificación multipleauthn cuando un usuario realice correctamente la verificación en dos pasos. Esta notificación se transmitirá a Azure AD. Siga este procedimiento para realizar los pasos:

1. Abra Administración de AD FS.
2. A la izquierda, seleccione **Relaciones de confianza para usuario autenticado**.
3. Haga clic con el botón derecho en **Plataforma de identidad de Microsoft Office 365** y seleccione **Editar reglas de notificaciones**.

   ![Consola de ADFS: confianzas de usuarios de confianza](./media/howto-mfa-adfs/trustedip1.png)

4. En Reglas de transformación de emisión, haga clic en **Agregar regla**.

   ![Edición de las reglas de transformación de emisión](./media/howto-mfa-adfs/trustedip2.png)

5. En el Asistente para agregar regla de notificaciones de transformación, seleccione **Pasar por una notificación entrante o filtrarla** en la lista desplegable y haga clic en **Siguiente**.

   ![Captura de pantalla que muestra el Asistente para agregar regla de notificaciones de transformación donde se selecciona una plantilla de regla de notificaciones.](./media/howto-mfa-adfs/trustedip3.png)

6. Asigne un nombre a la regla. 
7. Seleccione **Referencias de métodos de autenticación** como tipo de notificación entrante.
8. Seleccione **Pasar a través todos los valores de notificaciones**.
    ![Captura de pantalla que muestra el Asistente para agregar regla de notificaciones de transformación donde se selecciona Pasar a través todos los valores de notificaciones.](./media/howto-mfa-adfs/configurewizard.png)
9. Haga clic en **Finalizar** Cierre la consola de administración de AD FS.

## <a name="trusted-ips-for-federated-users"></a>Direcciones IP de confianza para usuarios federados

Las direcciones IP de confianza permiten a los administradores omitir la verificación en dos pasos para una dirección IP específica o para usuarios federados que tienen solicitudes que se originan dentro de su propia intranet. En las secciones siguientes se describe cómo configurar IP fiables de Azure AD Multi-Factor Authentication con usuarios federados y omitir la verificación en dos pasos cuando una solicitud se origina en una intranet de usuarios federados. Esto se consigue configurando AD FS para usar un paso a través o filtrar una plantilla de notificación entrante con el tipo de notificación dentro de la red corporativa.

En este ejemplo se utiliza Microsoft 365 para las relaciones de confianza del usuario de confianza.

### <a name="configure-the-ad-fs-claims-rules"></a>Configurar las reglas de notificaciones de AD FS

Utilice el procedimiento siguiente para configurar las notificaciones de AD FS. Cree dos reglas de notificaciones, una para el tipo de notificación dentro de la red corporativa y otra adicional para mantener a nuestros usuarios con la sesión iniciada.

1. Abra Administración de AD FS.
2. A la izquierda, seleccione **Relaciones de confianza para usuario autenticado**.
3. Haga clic con el botón derecho en **Plataforma de identidad de Microsoft Office 365** y seleccione **Consola de ADFS Console - Editar reglas de notificaciones**
   ![](./media/howto-mfa-adfs/trustedip1.png)
4. En Reglas de transformación de emisión, haga clic en **Agregar regla**.
   ![Agregar una regla de notificaciones](./media/howto-mfa-adfs/trustedip2.png)
5. En el Asistente para agregar regla de notificaciones de transformación, seleccione **Pasar por una notificación entrante o filtrarla** en la lista desplegable y haga clic en **Siguiente**.
   ![Captura de pantalla que muestra el Asistente para agregar regla de notificaciones de transformación donde se selecciona Pasar por una notificación entrante o filtrarla.](./media/howto-mfa-adfs/trustedip3.png)
6. En el cuadro situado junto al nombre de la regla de notificación, asigne un nombre a la regla. Por ejemplo: InsideCorpNet.
7. En la lista desplegable, junto a Tipo de notificación entrante, seleccione **Dentro de la red corporativa**.
   ![Agregar una notificación de una red corporativa](./media/howto-mfa-adfs/trustedip4.png)
8. Haga clic en **Finalizar**
9. En Reglas de transformación de emisión, haga clic en **Agregar regla**.
10. En el Asistente para agregar regla de notificaciones de transformación, seleccione **Enviar notificaciones mediante regla personalizada** en la lista desplegable y haga clic en **Siguiente**.
11. En el cuadro situado bajo el nombre de la regla de notificación: escriba *Mantener a los usuarios con la sesión iniciada*.
12. En el cuadro de regla personalizada, escriba:

```ad-fs-claim-rule
        c:[Type == "http://schemas.microsoft.com/2014/03/psso"]
            => issue(claim = c);
    ![Create custom claim to keep users signed in](./media/howto-mfa-adfs/trustedip5.png)
```

13. Haga clic en **Finalizar**
14. Haga clic en **Aplicar**.
15. Haga clic en **Aceptar**.
16. Cierre Administración de AD FS.

### <a name="configure-azure-ad-multi-factor-authentication-trusted-ips-with-federated-users"></a>Configuración de las IP de confianza de Azure AD Multi-Factor Authentication con usuarios federados

Ahora que las notificaciones están listas, podemos configurar direcciones IP de confianza.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione **Azure Active Directory** > **Seguridad** > **Acceso condicional** > **Ubicaciones con nombre**.
3. Desde la hoja **Acceso condicional: ubicaciones con nombre**, seleccione **Configurar direcciones IP de confianza de MFA**

   ![Acceso condicional de Azure AD, ubicaciones con nombre, Configurar direcciones IP de confianza de MFA](./media/howto-mfa-adfs/trustedip6.png)

4. En la página Configuración del servicio, en **direcciones IP de confianza**, seleccione **Omitir autenticación multifactor para solicitudes de usuarios federados en mi intranet**.  
5. Haga clic en **guardar**.

Eso es todo. En este punto, los usuarios federados de Microsoft 365 solo deberán usar MFA cuando una notificación se origine fuera de la intranet corporativa.
