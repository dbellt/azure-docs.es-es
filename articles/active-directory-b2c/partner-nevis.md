---
title: Tutorial para configurar Azure Active Directory B2C con Nevis
titleSuffix: Azure AD B2C
description: Aprenda a integrar la autenticación de Azure AD B2C con Nevis para la autenticación sin contraseña.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 11/23/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 42e244249ecb0539637918ae2439bdb4f5da4b38
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588486"
---
# <a name="tutorial-to-configure-nevis-with-azure-active-directory-b2c-for-passwordless-authentication"></a>Tutorial para configurar Nevis con Azure Active Directory B2C para la autenticación sin contraseña

En este tutorial de ejemplo, aprenderá a ampliar Azure AD B2C con [Nevis](https://www.nevis.net/en/solution/authentication-cloud) para habilitar la autenticación sin contraseña. Nevis proporciona una experiencia de usuario final de marca completa, que da prioridad a los dispositivos móviles, con la aplicación Nevis Access, que proporciona una autenticación sólida de clientes y cumple los requisitos de transacciones de la Directiva de servicios de pago 2 (PSD2).

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una [cuenta de prueba](https://www.nevis-security.com/aadb2c/) de Nevis

- Una suscripción de Azure AD. Si no tiene una, obtenga una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md) vinculado a la suscripción de Azure.

- Un entorno configurado de Azure AD B2C para el uso de [directivas personalizadas](./tutorial-create-user-flows.md?pivots=b2c-custom-policy) si quiere integrar Nevis en el flujo de directivas de registro.

## <a name="scenario-description"></a>Descripción del escenario

En este escenario, agregue una aplicación de acceso de marca completa a la aplicación back-end para la autenticación sin contraseña. En la solución, se incluyen los componentes siguientes:

- Un inquilino de Azure AD B2C, con una directiva de inicio de sesión y registro combinada para back-end
- La instancia de Nevis y su API REST para mejorar Azure AD B2C
- Su propia aplicación de acceso de marca propia

En el diagrama se muestra la implementación.

![Flujo de inicio de sesión de contraseña de alto nivel con Azure AD B2C y Nevis](./media/partner-nevis/nevis-architecture-diagram.png)

|Paso | Descripción |
|:-----| :-----------|
| 1. | Un usuario intenta iniciar sesión o registrarse en una aplicación mediante la directiva de inicio de sesión y registro de Azure AD B2C.
| 2. | Durante el registro, la aplicación Nevis Access se registra en el dispositivo del usuario mediante un código QR. Se genera una clave privada en el dispositivo del usuario, que se utiliza para firmar las solicitudes del usuario.
| 3. |  Azure AD B2C utiliza un perfil técnico de RESTful para poner en marcha el inicio de sesión con la solución Nevis.
| 4. | La solicitud de inicio de sesión se envía a la aplicación de acceso, ya sea en forma de mensaje de inserción, código QR o vínculo profundo.
| 5. | El usuario aprueba el intento de inicio de sesión con su biometría. A continuación, se devuelve un mensaje a Nevis, que comprueba el inicio de sesión con la clave pública almacenada.
| 6. | Azure AD B2C envía una última solicitud a Nevis para confirmar que el inicio de sesión se ha completado correctamente.
| 7. |En función del mensaje de error o de operación correcta de Azure AD B2C, se concede o deniega el acceso al usuario a la aplicación.

## <a name="integrate-your-azure-ad-b2c-tenant"></a>Integración del inquilino de Azure AD B2C

### <a name="onboard-to-nevis"></a>Incorporación a Nevis 

[Regístrese para crear una cuenta en Nevis](https://www.nevis-security.com/aadb2c/).
Recibirá dos mensajes de correo electrónico:

1. Una notificación de cuenta de administración

2. Una invitación de aplicación móvil

### <a name="add-your-azure-ad-b2c-tenant-to-your-nevis-account"></a>Incorporación del inquilino de Azure AD B2C a la cuenta de Nevis

1. En el mensaje de correo electrónico de prueba de la cuenta de administración de Nevis, copie la clave de administración en el portapapeles.

2. Abra https://console.nevis.cloud/ en un explorador.

3. Inicie sesión en la consola de administración con la clave.

4. Seleccione **Add Instance** (Agregar instancia).

5. Seleccione la instancia recién creada para abrirla.

6. En la barra de navegación lateral, seleccione **Custom Integrations** (Integraciones personalizadas).

7. Seleccione **Add custom integration** (Agregar integración personalizada).

8. En el nombre de la organización, escriba un nombre para el inquilino de Azure AD B2C.

9. En la dirección URL/dominio, especifique `https://yourtenant.onmicrosoft.com`.

10. Seleccione **Siguiente**.

>[!NOTE]
>Más adelante necesitará el token de acceso a Nevis.

11. Seleccione **Listo**.

### <a name="install-the-nevis-access-app-on-your-phone"></a>Instalación de la aplicación Nevis Access en el teléfono

1. En la dirección de correo electrónico de prueba de la aplicación móvil de Nevis, abra la invitación **Test Flight app** (Aplicación Test Flight).

2. Instale la aplicación.

3. Siga las instrucciones especificadas para instalar la aplicación Nevis Access.

### <a name="integrate-azure-ad-b2c-with-nevis"></a>Integración de Azure AD B2C con Nevis

1. Abra [Azure Portal](https://portal.azure.com/).

2. Cambie al inquilino de Azure AD B2C. Asegúrese de que haya seleccionado el inquilino adecuado, ya que el inquilino de Azure AD B2C se encuentra normalmente en un inquilino independiente.

3. En el menú, seleccione **Identity Experience Framework (IEF)** .

4. Seleccione **Claves de directiva**.

5. Seleccione **Agregar** y cree una nueva clave con la siguiente configuración:

      a. En Opciones, seleccione **Manual**.

      b. Especifique **AuthCloudAccessToken** en Nombre.

      c. Pegue el **token de Nevis Access** previamente almacenado en el campo Secreto.

      d. En Uso de la clave, seleccione **Cifrado**.

      e. Seleccione **Crear**

### <a name="configure-and-upload-the-nevishtml-to-azure-blob-storage"></a>Configuración y carga de nevis.html en Azure Blob Storage

1. En Identity Environment (IDE), vaya a la carpeta [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy) (directiva).

2. Abra el archivo [**nevis.html**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/nevis.html).

3. Reemplace **authentication_cloud_url** por la dirección URL de la consola de administración de Nevis: `https://<instance_id>.mauth.nevis.cloud`.

4. **Guarde** los cambios en el archivo.

5. Siga las [instrucciones](./customize-ui-with-html.md#2-create-an-azure-blob-storage-account) y cargue el archivo **nevis.html** en Azure Blob Storage.

6. Siga las [instrucciones ](./customize-ui-with-html.md#3-configure-cors) y habilite el uso compartido de recursos entre orígenes (CORS) para este archivo.

7. Una vez que la carga se haya completado, y se haya habilitado CORS, seleccione el archivo **nevis.html** en la lista.

8. En la pestaña **Información general**, junto a **Dirección URL**, seleccione el icono **Copiar vínculo**.

9. Abra el vínculo en una nueva pestaña del explorador para asegurarse de que muestra un cuadro gris.

>[!NOTE]
>Más adelante necesitará el vínculo del blob.

### <a name="customize-your-trustframeworkbasexml"></a>Personalización de TrustFrameworkBase.xml

1. En el IDE, vaya a la carpeta [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy) (directiva).

2. Abra el archivo [**TrustFrameworkBase.xml**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/TrustFrameworkBase.xml).

3. Reemplace **yourtenant** por el nombre de su cuenta de inquilino de Azure en **TenantId**.

4. Reemplace **yourtenant** por el nombre de su cuenta de inquilino de Azure en **PublicPolicyURI**.

5. Reemplace todas las instancias de **authentication_cloud_url** por la dirección URL de la consola de administración de Nevis

6. **Guarde** los cambios en el archivo.

### <a name="customize-your-trustframeworkextensionsxml"></a>Personalización de TrustFrameworkExtensions.xml

1. En el IDE, vaya a la carpeta [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy) (directiva).

2. Abra el archivo [**TrustFrameworkExtensions.xml**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/TrustFrameworkExtensions.xml).

3. Reemplace **yourtenant** por el nombre de su cuenta de inquilino de Azure en **TenantId**.

4. Reemplace **yourtenant** por el nombre de su cuenta de inquilino de Azure en **PublicPolicyURI**.

5. En **BasePolicy**, en **TenantId**, reemplace también _yourtenant_ por el nombre de su cuenta de inquilino de Azure.

6. En **BuildingBlocks**, reemplace **LoadUri** por la dirección URL del vínculo de blob de _nevis.html_ en su cuenta de almacenamiento de blobs.

7. **Guarde** el archivo.

### <a name="customize-your-signuporsigninxml"></a>Personalización de SignUpOrSignin.xml

1. En el IDE, vaya a la carpeta [**policy**](https://github.com/azure-ad-b2c/partner-integrations/tree/master/samples/Nevis/policy) (directiva).

2. Abra el archivo [**SignUpOrSignin.xml**](https://github.com/azure-ad-b2c/partner-integrations/blob/master/samples/Nevis/policy/SignUpOrSignin.xml).

3. Reemplace **yourtenant** por el nombre de su cuenta de inquilino de Azure en **TenantId**.

4. Reemplace **yourtenant** por el nombre de su cuenta de inquilino de Azure en **PublicPolicyUri**.

5. En **BasePolicy**, en **TenantId**, reemplace también **yourtenant** por el nombre de su cuenta de inquilino de Azure.

6. **Guarde** el archivo.

### <a name="upload-your-custom-policies-to-azure-ad-b2c"></a>Carga de las directivas personalizadas en Azure AD B2C

1. Abra la página principal del [inquilino de Azure AD B2C](https://portal.azure.com/#blade/Microsoft_AAD_B2CAdmin/TenantManagementMenuBlade/overview).

2. Seleccione **Marco de experiencia de identidad**.

3. Seleccione **Cargar directiva personalizada**.

4. Seleccione el archivo **TrustFrameworkBase.xml** que ha modificado.

5. Active la casilla **Sobrescribir la directiva personalizada si ya existe**.
6. Seleccione **Cargar**.

7. Repita los pasos 5 y 6 para **TrustFrameworkExtensions.xml**.

8. Repita los pasos 5 y 6 para **SignUpOrSignin.xml**.

## <a name="test-the-user-flow"></a>Prueba del flujo de usuario

### <a name="test-account-creation-and-nevis-access-app-setup"></a>Creación de la cuenta de prueba e instalación de la aplicación Nevis Access

1. Abra la página principal del [inquilino de Azure AD B2C](https://portal.azure.com/#blade/Microsoft_AAD_B2CAdmin/TenantManagementMenuBlade/overview).

2. Seleccione **Marco de experiencia de identidad**.

3. Desplácese a Directivas personalizadas y seleccione **B2C_1A_signup_signin**.

4. Seleccione **Ejecutar ahora**.

5. En la ventana emergente, seleccione **Registrarse ahora**.

6. Agregue su dirección de correo electrónico.

7. Seleccione **Enviar código de verificación**.

8. Copie el código de verificación del correo electrónico.

9. Seleccione **Comprobar**.

10. Rellene el formulario con la nueva contraseña y el nombre para mostrar.

11. Seleccione **Crear**.

12. Se le dirigirá a la página de digitalización del código QR.

13. En el teléfono, abra la **aplicación Nevis Access**.

14. Seleccione **Face ID**.

15. Cuando la pantalla indique **Authenticator registration was successful** (El registro del autenticador se ha realizado correctamente), seleccione **Continue** (Continuar).

16. En el teléfono, vuelva a autenticarse con su cara.

17. Se le dirigirá a la página de aterrizaje de [jwt.ms](https://jwt.ms) que muestra los detalles del token descodificado.

### <a name="test-the-pure-passwordless-sign-in"></a>Prueba del inicio de sesión sin contraseña puro

1. En **Identity Experience Framework**, seleccione **B2C_1A_signup_signin**.

2. Seleccione **Ejecutar ahora**.

3. En la ventana emergente, seleccione **Autenticación sin contraseña**.

4. Especifique su dirección de correo electrónico.

5. Seleccione **Continuar**.

6. En el teléfono, en notificaciones, seleccione **Nevis Access app notification** (Notificación de la aplicación Nevis Access).

7. Autentíquese con su cara.

8. Se le dirigirá automáticamente a la página de aterrizaje de [jwt.ms](https://jwt.ms) que muestra los tokens.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)
