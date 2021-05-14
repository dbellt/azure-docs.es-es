---
title: Configuración del proveedor de identidades de la cuenta local
titleSuffix: Azure AD B2C
description: Defina los tipos de identidad que puede usar (correo electrónico, nombre de usuario, número de teléfono) para la autenticación de cuentas locales al configurar los flujos de usuario en el inquilino de Azure Active Directory B2C.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/22/2021
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: abe9d9cd46c7d4cbcb383ee9a37fc84cd64fcea8
ms.sourcegitcommit: 19dcad80aa7df4d288d40dc28cb0a5157b401ac4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107896016"
---
# <a name="set-up-phone-sign-up-and-sign-in-for-user-flows"></a>Configuración del registro e inicio de sesión telefónico para flujos de usuario

Además del correo electrónico y el nombre de usuario, puede habilitar el número de teléfono como opción de registro para todo el inquilino agregando el registro e inicio de sesión telefónico al proveedor de identidades de la cuenta local. Después de habilitar el registro e inicio de sesión telefónico para cuentas locales, puede agregar el registro telefónico a los flujos de usuario.

La configuración del registro e inicio de sesión telefónico en un flujo de usuario implica los siguientes pasos:

- [Configure el registro e inicio de sesión telefónico para todo el inquilino](#configure-phone-sign-up-and-sign-in-tenant-wide) en el proveedor de identidades de la cuenta local para aceptar un número de teléfono como identidad del usuario. 

- [Agregue el registro telefónico al flujo de usuario](#add-phone-sign-up-to-a-user-flow) para permitir a los usuarios registrarse en la aplicación con su número de teléfono.

- [Habilite la solicitud de correo electrónico de recuperación (versión preliminar)](#enable-the-recovery-email-prompt-preview) para permitir a los usuarios especificar un correo electrónico que se pueda usar para recuperar su cuenta cuando no tengan su teléfono.

- [Muestre información de consentimiento](#enable-consent-information) al usuario durante el flujo de registro o de inicio de sesión. Puede mostrar la información de consentimiento predeterminada o personalizar su propia información de consentimiento.

La autenticación multifactor (MFA) se deshabilita de forma predeterminada al configurar un flujo de usuario con el registro telefónico. Puede habilitar MFA en los flujos de usuario con el registro telefónico, pero como el número de teléfono se usa como identificador principal, el código de acceso de un solo uso de correo electrónico es la única opción disponible para el segundo factor de autenticación.

## <a name="configure-phone-sign-up-and-sign-in-tenant-wide"></a>Configuración del registro e inicio de sesión telefónico para todo el inquilino

El registro de correo electrónico se habilita de forma predeterminada en la configuración del proveedor de identidades de la cuenta local. Puede cambiar los tipos de identidad que admitirá en el inquilino seleccionando o anulando la selección del registro de correo electrónico, el nombre de usuario o el número de teléfono.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).

2. Asegúrese de usar el directorio que contiene el inquilino de Azure AD B2C. Para ello, seleccione el filtro **Directorio y suscripción** en el menú superior y luego el directorio que contiene el inquilino de Azure AD.

3. Elija **Todos los servicios** en la esquina superior izquierda de Azure Portal, y busque y seleccione **Azure AD B2C**.

4. En **Administrar**, seleccione **Proveedores de identidades**.

5. En la lista de proveedores de identidades, seleccione **Cuenta local**.

   ![Selección de Cuenta local en Proveedores de identidades](media/phone-authentication-user-flows/identity-provider-local-account.png)

1. En la página **Configuración de IDP local**, asegúrese de que **Teléfono** esté seleccionado como uno de los tipos de identidad permitidos que los consumidores pueden usar para crear sus cuentas locales en su inquilino de Azure AD B2C.

   ![Selección de los tipos de identidad permitidos](media/phone-authentication-user-flows/configure-local-idp.png)

1. Seleccione **Guardar**.

## <a name="add-phone-sign-up-to-a-user-flow"></a>Incorporación del registro telefónico a un flujo de usuario

Después de agregar el registro telefónico como una opción de identidad para las cuentas locales, puede agregarlo a los flujos de usuario, siempre y cuando sean las versiones del flujo de usuario **recomendadas** más recientes. A continuación se puede ver un ejemplo en el que se muestra cómo agregar el registro telefónico a nuevos flujos de usuario. Sin embargo, también puede agregar el registro telefónico a los flujos de usuario de la versión recomendada existentes (seleccione **Flujos de usuario** > *nombre de flujo de usuario* > **Proveedores de identidades** > **Local Account Phone signup** (Registro telefónico de la cuenta local)). 

Este es un ejemplo en el que se muestra cómo agregar el registro telefónico a un nuevo flujo de usuario.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.

    ![Inquilino de B2C, panel de directorio y suscripción, Azure Portal](./media/phone-authentication-user-flows/directory-subscription-pane.png)

3. En Azure Portal, busque y seleccione **Azure AD B2C**.
4. En **Directivas**, seleccione **Flujos de usuario** y **Nuevo flujo de usuario**.

    ![Página Flujos de usuario del portal con el botón Nuevo flujo de usuario resaltado](./media/phone-authentication-user-flows/signup-signin-user-flow.png)

5. En la página **Crear un flujo de usuario**, seleccione el flujo de usuario **Registrarse e iniciar sesión**.

    ![Página Selección de un flujo de usuario con el flujo de registro e inicio de sesión resaltado](./media/phone-authentication-user-flows/select-user-flow-type.png)

6. En **Seleccione una versión**, elija **Recomendada** y, luego, seleccione **Crear**. [Más información](user-flow-versions.md) sobre las versiones del flujo de usuario.

    ![Botón Creación de flujo de usuario](./media/phone-authentication-user-flows/select-version.png)

7. Escriba un **nombre** para el flujo de usuario. Por ejemplo, *signupsignin1*.
8. En la sección **Proveedores de identidades**, en **Cuentas locales**, seleccione **Phone signup**(Registro telefónico).

   ![Opción de registro telefónico del flujo de usuario seleccionada](media/phone-authentication-user-flows/user-flow-phone-signup.png)

9. En **Proveedores de identidades sociales**, seleccione cualquier otro proveedor de identidades que desee permitir para este flujo de usuario.

   > [!NOTE]
   > La autenticación multifactor (MFA) está deshabilitada de forma predeterminada para los flujos de registro de usuarios. Puede habilitar MFA para un flujo de usuario de registro telefónico, pero como un número de teléfono se usa como identificador principal, el código de acceso de un solo uso de correo electrónico es la única opción disponible para el segundo factor de autenticación.

1. En la sección **User attributes and token claims** (Atributos de usuario y notificaciones de token), elija los atributos y las notificaciones que desea recopilar y enviar del usuario durante el registro. Por ejemplo, seleccione **Mostrar más** y elija los atributos y las notificaciones de **País o región**, **Nombre para mostrar** y **Código postal**. Seleccione **Aceptar**.

1. Seleccione **Crear** para agregar el flujo de usuario. El prefijo *B2C_1* se anexa automáticamente al nombre.

## <a name="enable-the-recovery-email-prompt-preview"></a>Habilitación de la solicitud de correo electrónico de recuperación (versión preliminar)

Al habilitar el registro e inicio de sesión telefónico para los flujos de usuario, también es buena idea habilitar la característica de correo electrónico de recuperación. Con esta característica, un usuario puede proporcionar una dirección de correo electrónico que se puede usar para recuperar su cuenta cuando no tiene su teléfono. Esta dirección de correo electrónico se usa solo para la recuperación de cuenta. No se puede usar para iniciar sesión.

- Cuando la solicitud de correo electrónico de recuperación está **activada**, a un usuario que se registre por primera vez se le pedirá que compruebe un correo electrónico de copia de seguridad. A un usuario que no haya proporcionado un correo electrónico de recuperación antes se le pedirá que compruebe un correo electrónico de copia de seguridad durante el siguiente inicio de sesión.

- Cuando el correo electrónico de recuperación está **desactivado**, la solicitud de correo electrónico no se mostrará a un usuario que se registre o inicie sesión.
 
Puede habilitar la solicitud de correo electrónico de recuperación en las propiedades del flujo de usuario.

> [!NOTE]
> Antes de comenzar, asegúrese de que ha [agregado el registro telefónico al flujo de usuario](#add-phone-sign-up-to-a-user-flow) tal y como se ha descrito anteriormente.

### <a name="to-enable-the-recovery-email-prompt"></a>Para habilitar la solicitud de correo electrónico de recuperación

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
3. En Azure Portal, busque y seleccione **Azure AD B2C**.
4. En Azure AD B2C, en **Directivas**, seleccione **Flujos de usuario**.
5. Seleccione el flujo de usuario de la lista.
6. En **Configuración**, seleccione **Propiedades**.
7. Junto a **Habilitar solicitud de correo electrónico de recuperación para la suscripción del número de teléfono y el inicio de sesión (versión preliminar)** , seleccione:

   - **Activado** para mostrar la solicitud de correo electrónico de recuperación durante el registro e inicio de sesión.
   - **Desactivado** para ocultar la solicitud de correo electrónico de recuperación.

    ![Propiedades de flujos de usuario con la opción de habilitación del correo electrónico de recuperación habilitada](./media/phone-authentication-user-flows/recovery-email-settings.png)

8. Seleccione **Guardar**.

### <a name="to-test-the-recovery-email-prompt"></a>Para probar la solicitud de correo electrónico de recuperación

Una vez que haya habilitado el registro e inicio de sesión telefónico y la solicitud de correo electrónico de recuperación en el flujo de usuario, puede usar **Ejecutar flujo de usuario** para probar la experiencia del usuario.

1. Seleccione **Directivas** > **Flujos de usuario** y, a continuación, seleccione el flujo de usuario que ha creado. En la página de información general del flujo de usuario, seleccione **Ejecutar flujo de usuario**.

2. En **Aplicación**, seleccione la aplicación web que registró en el paso 1. La **dirección URL de respuesta** debe mostrar `https://jwt.ms`.

3. Seleccione **Ejecutar flujo de usuario** y compruebe el comportamiento siguiente:

   - A un usuario que se registre por primera vez se le pedirá que proporcione un correo electrónico de recuperación. 
   - A un usuario que ya se haya registrado pero no haya proporcionado un correo electrónico de recuperación se le pedirá que proporcione uno al iniciar sesión.

4. Escriba una dirección de correo electrónico y, a continuación, seleccione **Enviar código de verificación**. Compruebe que se envía un código a la bandeja de entrada del correo electrónico que ha proporcionado. Recupere el código y escríbalo en el cuadro **Código de verificación**. A continuación, seleccione **Verificar código**.

## <a name="enable-consent-information"></a>Habilitación de la información de consentimiento

Se recomienda encarecidamente incluir información de consentimiento en el flujo de registro e inicio de sesión. Se proporciona texto de ejemplo. Consulte el documento Short Code Monitoring Handbook en el [sitio web de CTIA](https://www.ctia.org/programs) y póngase en contacto con sus propios expertos legales o de cumplimiento para obtener instrucciones sobre el texto final y la configuración de características que satisfacen sus propias necesidades de cumplimiento:
>
> *Al proporcionar su número de teléfono, da su consentimiento a recibir un código de acceso de un solo uso enviado por mensaje de texto para ayudarle a iniciar sesión en *&lt;insert: el nombre de la aplicación&gt;* . Se pueden aplicar tarifas de datos y mensajes.*
>
> *&lt;insert: un vínculo a la declaración de privacidad&gt;*<br/>*&lt;insert: un vínculo a los términos de servicio&gt;*

Para habilitar la información de consentimiento

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Seleccione el icono **Directorio y suscripción** en la barra de herramientas del portal y, luego, elija el directorio que contiene el inquilino de Azure AD B2C.
3. En Azure Portal, busque y seleccione **Azure AD B2C**.
4. En Azure AD B2C, en **Directivas**, seleccione **Flujos de usuario**.
5. Seleccione el flujo de usuario de la lista.
6. En **Personalizar**, seleccione **Idiomas.**
7. Para mostrar el texto de consentimiento, seleccione **Habilitación de la personalización de idioma**.
  
    ![Habilitación de la personalización de idioma](./media/phone-authentication-user-flows/enable-language-customization.png)

8. Para personalizar la información de consentimiento, seleccione un idioma en la lista.
9. En el panel de idiomas, seleccione **Phone signIn page** (Página de inicio de sesión por teléfono).
10. Seleccione Descargar valores predeterminados.

    ![Descargar valores predeterminados](./media/phone-authentication-user-flows/phone-sign-in-language-override.png)

11. Abra el archivo JSON descargado. Busque el texto siguiente y personalícelo:

    - **disclaimer_link_1_url**: cambie **override** a "true" y agregue la dirección URL de la información de privacidad.

    - **disclaimer_link_2_url**: cambie **override** a "true" y agregue la dirección URL para los términos de uso.  

    - **disclaimer_msg_intro**: cambie **override** a "true" y cambie **value** a las cadenas de declinación de responsabilidades deseadas.  

12. Guarde el archivo. En **Upload new overrides** (Cargar nuevos reemplazos) busque el archivo y selecciónelo. Confirme que ve la notificación "Se cargaron correctamente los reemplazos.".

13. Seleccione **Phone signUp page** (Página de registro por teléfono) y repita los pasos del 10 al 12. 

## <a name="next-steps"></a>Pasos siguientes

- [Agregar proveedores de identidades externos](add-identity-provider.md)
- [Crear flujos de usuario](tutorial-create-user-flows.md)