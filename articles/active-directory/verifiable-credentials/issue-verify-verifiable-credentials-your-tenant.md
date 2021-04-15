---
title: 'Tutorial: emisión y comprobación de credenciales verificables mediante el inquilino de Azure (versión preliminar)'
description: Cambio del ejemplo de código de credencial verificable para que funcione con el inquilino de Azure
documentationCenter: ''
author: barclayn
manager: daveba
ms.service: identity
ms.topic: how-to
ms.subservice: verifiable-credentials
ms.date: 04/01/2021
ms.author: barclayn
ms.reviewer: ''
ms.openlocfilehash: e4772b6701065a44416d849faa9a501bd7895f27
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106553386"
---
# <a name="tutorial---issue-and-verify-verifiable-credentials-using-your-tenant-preview"></a>Tutorial: emisión y comprobación de credenciales verificables mediante el inquilino (versión preliminar)

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory son una característica que se encuentra actualmente en versión preliminar.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Ahora que tiene el inquilino de Azure configurado con el servicio de credenciales verificables, se le guiará por los pasos necesarios para habilitar Azure Active Directory (Azure AD) para emitir credenciales verificables mediante la aplicación de ejemplo.

En este artículo, aprenderá a:

> [!div class="checklist"]
> * Registrar la aplicación de ejemplo en Azure AD
> * Crear un archivo de reglas y visualización
> * Cargar archivos de reglas y visualización
> * Configurar el servicio de emisor de credenciales verificables para usar Azure Key Vault
> * Actualizar el ejemplo de código con la información del inquilino

Nuestro código de ejemplo requiere que los usuarios se autentiquen en un proveedor de identidades, específicamente Azure AD B2C, antes de que se pueda emitir la credencial verificable del experto comprobado en credenciales. No todos los emisores de credenciales verificables requieren autenticación antes de emitir credenciales.

La autenticación de tokens de identificador permite a los usuarios demostrar quién son antes de recibir su credencial. Cuando los usuarios inician sesión correctamente, el proveedor de identidades devuelve un token de seguridad que contiene notificaciones sobre el usuario. Después, el servicio del emisor transforma estos tokens de seguridad y sus notificaciones en una credencial verificable. La credencial verificable se firma con el DID del emisor.

Se admite cualquier proveedor de identidades que admita el protocolo OpenID Connect. Algunos ejemplos de proveedores de identidades admitidos son [Azure Active Directory](../fundamentals/active-directory-whatis.md) y [Azure AD B2C](../../active-directory-b2c/overview.md). En este tutorial, usamos AAD.

## <a name="prerequisites"></a>Prerrequisitos

En este tutorial se da por hecho que ya ha realizado los pasos de los [tutoriales anteriores](enable-your-tenant-verifiable-credentials.md) y que tiene acceso al entorno que ha usado.

## <a name="register-an-app-to-enable-did-wallets-to-sign-in-users"></a>Registro de una aplicación para permitir que las carteras de DID inicien la sesión de los usuarios

Para emitir una credencial verificable, debe registrar una aplicación para que Authenticator, o cualquier otra cartera de credenciales verificables, tenga permiso para iniciar la sesión de los usuarios.  

Registre una aplicación denominada "Aplicación VC Wallet" en Azure AD y obtenga un id. de cliente.

1. Siga las instrucciones para registrar una aplicación con [Azure AD](../develop/quickstart-register-app.md). Al registrarse, use los valores siguientes.

   - Nombre: "Aplicación VC Wallet"
   - Tipos de cuenta admitidos: Solo las cuentas de este directorio organizativo
   - URI de redireccionamiento: vcclient://openid/

   ![registro de una aplicación](media/issue-verify-verifable-credentials-your-tenant/register-application.png)

2. Después de registrar la aplicación, anote el id. de aplicación (cliente). Este valor lo necesitará más adelante.

   ![id. de aplicación (cliente)](media/issue-verify-verifable-credentials-your-tenant/client-id.png)

3. Seleccione el botón **Puntos de conexión** y copie el URI del documento de metadatos de OpenID Connect. Necesita esta información para la sección siguiente. 

   ![puntos de conexión del emisor](media/issue-verify-verifable-credentials-your-tenant/application-endpoints.png)

## <a name="set-up-your-node-app-with-access-to-azure-key-vault"></a>Configuración de la aplicación Node con acceso a Azure Key Vault

Para autenticar la solicitud de emisión de credenciales de un usuario, el sitio web del emisor utiliza las claves criptográficas en Azure Key Vault. Para obtener acceso a Azure Key Vault, el sitio web necesita un identificador de cliente y un secreto de cliente que se pueda usar para autenticarse en Azure Key Vault.

1. Al ver la página de información general de la aplicación VC Wallet, seleccione **Certificados y secretos**.
    ![Certificados y secretos](media/issue-verify-verifable-credentials-your-tenant/vc-wallet-app-certs-secrets.png)
1. En la sección **Secretos de cliente**, seleccione **Nuevo secreto de cliente**.
    1. Agregue una descripción como "Secreto de cliente de la credencial verificable de Node"
    1. Expira: en un año.
  ![Secreto de aplicación con una expiración de un año](media/issue-verify-verifable-credentials-your-tenant/add-client-secret.png)
1. Copie el SECRETO. Necesita esta información para actualizar la aplicación Node de ejemplo.

>[!WARNING]
> Tiene una oportunidad para copiar el secreto. Después, el secreto es un valor con hash unidireccional. No copie el id. 

Después de crear la aplicación y el secreto de cliente en Azure AD, debe conceder a la aplicación los permisos necesarios para realizar operaciones en Key Vault. Es necesario realizar estos cambios de permisos para permitir que el sitio web tenga acceso a las claves privadas almacenadas allí y las use.

1. Vaya a Key Vault.
2. Seleccione el almacén de claves que estamos usando para estos tutoriales.
3. Elija **Directivas de acceso** en el panel de navegación izquierdo.
4. Elija **+Agregar directiva de acceso**.
5. En la sección **Permisos de la clave**, elija **Obtener** y **Firmar**.
6. Seleccione **Entidad de seguridad** y use el id. de aplicación para buscar la aplicación que registró anteriormente. Selecciónela.
7. Seleccione **Agregar**.
8. Elija **GUARDAR**.

Para más información sobre los permisos de Key Vault y el control de acceso, consulte la [Guía de RBAC de Key Vault](../../key-vault/general/rbac-guide.md).

![asignación de los permisos del almacén de claves](media/issue-verify-verifable-credentials-your-tenant/key-vault-permissions.png)
## <a name="make-changes-to-match-your-environment"></a>Realización de cambios para que coincidan con el entorno

Hasta ahora, hemos estado trabajando con nuestra aplicación de ejemplo. La aplicación usa [Azure Active Directory B2C](../../active-directory-b2c/overview.md) y ahora estamos cambiando para usar Azure AD, por lo que necesitamos hacer algunos cambios, no solo para que coincidan con su entorno, sino también para admitir notificaciones adicionales que no se usaron antes.

1. Copie el archivo de reglas siguiente y guárdelo en **modified-expertRules.json**. 

    > [!NOTE]
    > **"scope": "openid profile"** se incluye en este archivo de reglas y no se incluyó en el archivo de reglas de la aplicación de ejemplo. En la siguiente sección se explica cómo habilitar las notificaciones opcionales en el inquilino de Azure Active Directory. 
    
    ```json
    {
      "attestations": {
        "idTokens": [
          {
            "mapping": {
              "firstName": { "claim": "given_name" },
              "lastName": { "claim": "family_name" }
            },
            "configuration": "https://dIdPlayground.b2clogin.com/dIdPlayground.onmicrosoft.com/B2C_1_sisu/v2.0/.well-known/openid-configuration",
            "client_id": "8d5b446e-22b2-4e01-bb2e-9070f6b20c90",
            "redirect_uri": "vcclient://openid/",
             "scope": "openid profile"
          }
        ]
      },
      "validityInterval": 2592000,
      "vc": {
        "type": ["VerifiedCredentialExpert"]
      }
    }
    ```

2. Abra el archivo y reemplace los valores de **client_id** y **configuration** por los dos valores que hemos copiado en la sección anterior.

    ![resaltado de los dos valores que deben modificarse como parte de este paso](media/issue-verify-verifable-credentials-your-tenant/rules-file.png)

  El valor de **configuration** es el URI del documento de metadatos de OpenID Connect.

  Dado que el código de ejemplo usa Azure Active Directory B2C y estamos usando Azure Active Directory, necesitamos agregar notificaciones opcionales a través de ámbitos para que estas notificaciones se incluyan en el token de identificador que se va a escribir en la credencial verificable. 

3. Vuelva a Azure Portal y abra Azure Active Directory.
4. Elija **Registros de aplicaciones**.
5. Abra la aplicación VC Wallet que hemos creado anteriormente.
6. Elija **Configuración del token**.
7. Elija **+Agregar notificación opcional**.

     ![en Configuración del token, Agregar una notificación opcional](media/issue-verify-verifable-credentials-your-tenant/token-configuration.png)

8. En **Tipo de token**, elija **Id.** y, en la lista de notificaciones disponibles, elija **given_name** y **family_name**.

     ![Agregar notificación opcional](media/issue-verify-verifable-credentials-your-tenant/add-optional-claim.png)

9. Presione **Agregar**.
10. Si recibe una advertencia de permisos, como se muestra a continuación, active la casilla y seleccione **Agregar**.

     ![agregar permisos para las notificaciones opcionales](media/issue-verify-verifable-credentials-your-tenant/add-optional-claim-permissions.png)

Ahora, cuando se presenta a un usuario el "Inicio de sesión" para obtener la credencial verificable, la aplicación VC Wallet tiene capacidad para incluir las notificaciones específicas a través del parámetro de ámbito que se va a escribir en la credencial verificable.

## <a name="create-new-vc-with-this-rules-file-and-the-old-display-file"></a>Creación de una nueva credencial verificable con este archivo de reglas y el archivo de visualización anterior

1. Cargue el nuevo archivo de reglas en el contenedor.
1. En la página de credenciales verificables, cree una nueva credencial denominada **modifiedCredentialExpert** con el archivo de visualización anterior y el nuevo archivo de reglas (**modified-credentialExpert.json**).
1. Una vez completado el proceso de creación de credenciales desde la página **Información general**, copie la **dirección URL de emisión de credenciales** y guárdela porque la necesitamos en la sección siguiente.

## <a name="before-we-continue"></a>Antes de continuar

Necesitamos poner algunos valores juntos para poder realizar los cambios de código necesarios. Usamos estos valores en la sección siguiente para que el código de ejemplo use sus propias claves almacenadas en el almacén. Hasta ahora, debemos tener listos los siguientes valores.

- **URI del contrato** de la credencial que acabamos de crear (dirección URL de emisión de credenciales)
- **Identificador de cliente de aplicación**: se obtiene cuando se registra la aplicación de nodo.
- **Secreto de cliente**: lo hemos creado anteriormente cuando se concedió a la aplicación acceso al almacén de claves.

Hay algunos otros valores que necesitamos obtener antes de poder realizar los cambios una vez volvamos a nuestra aplicación de ejemplo. Vamos a obtenerlos ahora.

### <a name="verifiable-credentials-settings"></a>Configuración de las credenciales verificables

1. Vaya a la página Credenciales verificables y elija **Configuración**.  
1. Copie los valores siguientes:

    - Identificador de inquilino 
    - Identificador del emisor (el DID)
    - Almacén de claves (URI)

1. En el identificador de clave de firma, hay un URI, pero solo se necesita una parte del mismo. Copie la parte que indica **issuerSigningKeyION** tal como aparece resaltada por el rectángulo rojo en la imagen siguiente.

   ![identificador de la clave de inicio de sesión](media/issue-verify-verifable-credentials-your-tenant/issuer-signing-key-ion.png)

### <a name="did-document"></a>Documento de DID

1. Abra el [Explorador de red de DIF ION](https://identity.foundation/ion/explorer/).

2. Pegue el DID en la barra de búsqueda.

4. En la respuesta con formato, busque la sección denominada **verificationMethod**.
5. En "verificationMethod", copie el identificador y etiquételo como kvSigningKeyId.
    
    ```json=
    "verificationMethod": [
          {
            "id": "#sig_25e48331",
    ```

Ahora tenemos todo lo que necesitamos para realizar los cambios en el código de ejemplo.

- **Emisor:** app.js. Actualice la credencial de const con el nuevo URI de contrato.
- **Comprobador:** app.js. Actualice issuerDid con el identificador del emisor.
- **Emisor y comprobador**: actualice didconfig.json con los siguientes valores:


```json=
{
    "azTenantId": "Your tenant ID",
    "azClientId": "Your client ID",
    "azClientSecret": "Your client secret",
    "kvVaultUri": "your keyvault uri",
    "kvSigningKeyId": "The verificationMethod ID from your DID Document",
    "kvRemoteSigningKeyId" : "The snippet of the issuerSigningKeyION we copied ",
    "did": "Your DID"
}
```

>[!IMPORTANT]
>Se trata de una aplicación de demostración y, normalmente, nunca debe proporcionar el secreto directamente a la aplicación.


Ahora tiene todo lo necesario para emitir y comprobar su propia credencial verificable desde el inquilino de Azure Active Directory con sus propias claves. 

## <a name="issue-and-verify-the-vc"></a>Emisión y comprobación de la credencial verificable

Siga los mismos pasos que hemos seguido en el tutorial anterior para emitir la credencial verificable y validarla con la aplicación. Una vez que haya completado correctamente el proceso de comprobación, ya está listo para seguir aprendiendo sobre las credenciales verificables.

1. Abra un símbolo del sistema y abra la carpeta del emisor.
2. Ejecute la aplicación Node actualizada.

    ```terminal
    node app.js
    ```

3. Con otro símbolo del sistema, ejecute ngrok para configurar una dirección URL en 8081.

    ```terminal
    ngrok http 8081
    ```
    
    >[!IMPORTANT]
    > Es posible que también vea una advertencia que indica que esta aplicación o sitio web puede ser de riesgo. Este mensaje es normal en este momento, ya que aún no hemos vinculado el DID a su dominio. Siga las instrucciones de [enlace de DNS](how-to-dnsbind.md) para configurar esto.

    
4. Abra la dirección URL HTTPS generada por ngrok.

    ![Puntos de conexión de reenvío de NGROK](media/enable-your-tenant-verifiable-credentials/ngrok-url-screen.png)

5. Seleccione **GET CREDENTIAL** (Obtener credencial).
6. En Authenticator, escanee el código QR.
7. En el mensaje de advertencia **Usar esta aplicación o este sitio web puede ser arriesgado**, elija **Avanzado**.

  ![Advertencia inicial](media/enable-your-tenant-verifiable-credentials/site-warning.png)

8. En la advertencia de sitio web de riesgo, seleccione **Continuar de todos modos (no seguro)** .

  ![Segunda advertencia sobre el emisor](media/enable-your-tenant-verifiable-credentials/site-warning-proceed.png)


9. En la pantalla **Adición de credenciales**, observe ciertos aspectos: 
    1. En la parte superior de la pantalla, puede ver el mensaje **Sin comprobar** en rojo.
    1. La credencial se personaliza en función de los cambios realizados en el archivo de visualización.
    1. La opción **Iniciar sesión en su cuenta** apunta a la página de inicio de sesión de Azure AD.
    
   ![pantalla de adición de credencial con advertencia](media/enable-your-tenant-verifiable-credentials/add-credential-not-verified.png)

10. Elija **Iniciar sesión en su cuenta** y realice la autenticación con un usuario en el inquilino de Azure AD.
11. Después de autenticarse correctamente, el botón **Agregar** ya no estará atenuado. Seleccione **Agregar**.

  ![pantalla de adición de credencial después de la autenticación](media/enable-your-tenant-verifiable-credentials/add-credential-not-verified-authenticated.png)

Hemos emitido una credencial verificable usando nuestro inquilino para generar la credencial verificable mientras seguimos usando nuestro inquilino de B2C para la autenticación.

  ![credencial verificable emitida por Azure AD y autenticada por nuestra instancia de Azure B2C](media/enable-your-tenant-verifiable-credentials/my-vc-b2c.png)


## <a name="test-verifying-the-vc-using-the-sample-app"></a>Prueba de comprobación de la credencial verificable mediante la aplicación de ejemplo

Ahora que hemos emitido la credencial verificable desde nuestro propio inquilino con notificaciones desde Azure AD, vamos a comprobarla con nuestra aplicación de ejemplo.

1. Detenga la ejecución del servicio ngrok del emisor.

    ```cmd
    control-c
    ```

2. Ahora ejecute ngrok con el puerto 8082 del verificador.

    ```cmd
    ngrok http 8082
    ```

3. En otra ventana de terminal, vaya a la aplicación verificadora y ejecútela de la misma forma que se ejecutó la aplicación del emisor.

    ```cmd
    cd ..
    cd verifier
    node app.js
    ```

4. Abra la dirección URL de ngrok en el explorador y escanee el código QR con Authenticator en su dispositivo móvil.
5. En su dispositivo móvil, elija **Permitir** en la pantalla **Nueva solicitud de permiso**.

   >[!IMPORTANT]
    > Como la aplicación de ejemplo también usa el DID para firmar la solicitud de presentación, observará una advertencia de que esta aplicación o este sitio web pueden ser arriesgados. Este mensaje es normal en este momento, ya que aún no hemos vinculado el DID a su dominio. Siga las instrucciones de [enlace de DNS](how-to-dnsbind.md) para configurar esto.
    
   ![nueva solicitud de permiso](media/enable-your-tenant-verifiable-credentials/new-permission-request.png)

8. Ha comprobado correctamente las credenciales y el sitio web debería mostrar su nombre y apellidos de la cuenta de usuario de Azure AD. 

Ya ha completado el tutorial y es oficialmente un experto comprobado en credenciales. La aplicación de ejemplo usa el DID para emitir y comprobar, al tiempo que escribe las notificaciones en una credencial verificable desde Azure AD. 

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo crear [credenciales personalizadas](credential-design.md).
- [Ejemplos](issuer-openid.md) de comunicación del servicio emisor.
