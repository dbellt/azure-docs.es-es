---
title: 'Tutorial: Configuración de Azure Active Directory para emitir credenciales verificables (versión preliminar)'
description: En este tutorial, creará el entorno necesario para implementar credenciales verificables en el inquilino.
documentationCenter: ''
author: barclayn
manager: daveba
ms.service: identity
ms.topic: tutorial
ms.subservice: verifiable-credentials
ms.date: 04/01/2021
ms.author: barclayn
ms.reviewer: ''
ms.openlocfilehash: c289e69345b2fe537fd80f2cd8b59bc13ce8287b
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108017308"
---
# <a name="tutorial---configure-your-azure-active-directory-to-issue-verifiable-credentials-preview"></a>Tutorial: Configuración de Azure Active Directory para emitir credenciales verificables (versión preliminar)

En este tutorial, se basará en el trabajo realizado como parte del artículo de [introducción](get-started-verifiable-credentials.md) y configurará Azure Active Directory (Azure AD) con su propio [identificador descentralizado](https://www.microsoft.com/security/business/identity-access-management/decentralized-identity-blockchain?rtc=1#:~:text=Decentralized%20identity%20is%20a%20trust,protect%20privacy%20and%20secure%20transactions.) (DID). El identificador descentralizado se usará para emitir una credencial verificable mediante la aplicación de ejemplo y el emisor. No obstante, en este tutorial aún se usará el inquilino de Azure B2C de ejemplo para la autenticación.  En el siguiente tutorial, realizaremos pasos adicionales para configurar la aplicación para que funcione con Azure AD.

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo:

> [!div class="checklist"]
> * Creará los servicios necesarios para incorporar Azure AD para las credenciales verificables. 
> * Creará su DID.
> * Personalizará las reglas y los archivos de visualización.
> * Configurará credenciales verificables en Azure AD.


## <a name="prerequisites"></a>Requisitos previos

Para completar correctamente este tutorial, necesitará lo siguiente:

- Complete la [Introducción](get-started-verifiable-credentials.md).
- Disponga de una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure AD con una [licencia](https://azure.microsoft.com/pricing/details/active-directory/) P2. Siga el [procedimiento para crear una cuenta de desarrollador gratuita](how-to-create-a-free-developer-account.md) si no tiene ninguna.
- Una instancia de [Azure Key Vault](../../key-vault/general/overview.md) en la que tenga derechos para crear claves y secretos.

## <a name="azure-active-directory"></a>Azure Active Directory

Antes de empezar, necesitaremos un inquilino de Azure AD. Cuando el inquilino está habilitado para credenciales verificables, se le asigna un identificador descentralizado (DID) y se habilita con un servicio de emisor para emitir credenciales verificables. Su inquilino y su DID emitirán todas las credenciales verificables que usted emita. El DID también se usa para comprobar las credenciales verificables.
Si acaba de crear una suscripción de Azure de prueba, no es necesario rellenar el inquilino con cuentas de usuario, pero debe tener al menos una cuenta de usuario de prueba para completar los tutoriales posteriores.

## <a name="create-a-key-vault"></a>Creación de un almacén de claves

Al trabajar con credenciales verificables, tiene el control total y la administración de las claves criptográficas que usa el inquilino para firmar digitalmente las credenciales verificables. Para emitir y comprobar las credenciales, debe proporcionar a Azure AD acceso a su propia instancia de Azure Key Vault.

1. En el menú de Azure Portal o en la **página principal**, seleccione **Crear un recurso**.
2. En el cuadro de búsqueda, escriba **key vault**.
3. En la lista de resultados, elija **Key Vault**.
4. En la sección Key Vault, elija **Crear**.
5. En la sección **Crear Key Vault**, proporcione la siguiente información:
    - **Suscripción**: Elija una suscripción.
    - En **Grupo de recursos**, elija **Crear nuevo** y escriba un nombre para el grupo de recursos, como **vc-resource-group**. En varios artículos se usa el mismo nombre para el grupo de recursos.
    - **Name**: se requiere un nombre único. Se usa **woodgroup-vc-kv**, por lo que debe reemplazar este valor por su propio nombre único.
    - En el menú desplegable **Ubicación**, elija una ubicación.
    - Deje las restantes opciones con sus valores predeterminados.
6. Después de proporcionar la información descrita anteriormente, seleccione **Directiva de acceso**.

    ![creación de una página de Key Vault](media/enable-your-tenant-verifiable-credentials/create-key-vault.png)

7. En la pantalla Directiva de acceso, elija **Agregar directiva de acceso**.

    >[!NOTE]
    > De forma predeterminada, la cuenta que crea el almacén de claves es la única con acceso. El servicio de credenciales verificables necesita tener acceso al almacén de claves. El almacén de claves debe tener una directiva de acceso que permita al administrador **crear claves**, tener la capacidad de **eliminar claves** si es necesario y **firmar** para crear el enlace de dominio para las credenciales verificables. Si usa la misma cuenta mientras realiza las pruebas, asegúrese de modificar la directiva predeterminada para conceder el permiso de **firma** de la cuenta, además de los permisos predeterminados concedidos a los creadores del almacén.

8. Para el administrador de usuarios, asegúrese de que **Crear**, **Eliminar** y **Firmar** estén habilitados en la sección de permisos de claves. Los permisos de creación y eliminación están habilitados de forma predeterminada, de modo que el permiso de firma debe ser el único que tenga que actualizarse. 

    ![Permisos del almacén de claves](media/enable-your-tenant-verifiable-credentials/keyvault-access.png)

9. Seleccione **Revisar + crear**.
10. Seleccione **Crear**.
11. Vaya al almacén y anote el nombre del almacén y el URI.

Tome nota de las dos propiedades siguientes:

- **Nombre del almacén**: en el ejemplo, el nombre del valor es **woodgrove-vc-kv**. Este nombre se usará en otros pasos.
- **URI de almacén**: en el ejemplo, es https://woodgrove-vc-kv.vault.azure.net/. Las aplicaciones que utilizan el almacén a través de su API de REST deben usar este identificador URI.

>[!NOTE]
> Cada transacción del almacén de claves genera costos de suscripción de Azure adicionales. Revise la [página precios de Key Vault](https://azure.microsoft.com/pricing/details/key-vault/) para obtener información detallada.

>[!IMPORTANT]
> Durante la versión preliminar de las credenciales verificables de Azure Active Directory, las claves y los secretos creados en el almacén no deben modificarse una vez creados. Al eliminar, deshabilitar o actualizar las claves y los secretos, se invalidan las credenciales emitidas. No modifique sus claves o secretos durante la versión preliminar.

## <a name="create-a-modified-rules-and-display-file"></a>Creación de un archivo de reglas y visualización modificado

En esta sección, usaremos las reglas y los archivos de visualización de la [aplicación del emisor de ejemplo](https://github.com/Azure-Samples/active-directory-verifiable-credentials/) y los modificaremos ligeramente para crear la primera credencial verificable del inquilino.

1. Copie los archivos JSON de visualización y reglas en una carpeta temporal y cámbieles el nombre a **MyFirstVC-display.json** y **MyFirstVC-rules.json** respectivamente. Puede encontrar ambos archivos en **issuer\issuer_config**.

   ![Archivos de visualización y reglas como parte del directorio de la aplicación de ejemplo](media/enable-your-tenant-verifiable-credentials/sample-app-rules-display.png)

   ![Archivos de visualización y reglas en una carpeta temporal](media/enable-your-tenant-verifiable-credentials/display-rules-files-temp.png)

2. Abra el archivo MyFirstVC-rules.json en el editor de código. 

    ```json
         {
          "attestations": {
            "idTokens": [
              {
                "mapping": {
                  "firstName": { "claim": "given_name" },
                  "lastName": { "claim": "family_name" }
                },
                "configuration": "https://didplayground.b2clogin.com/didplayground.onmicrosoft.com/B2C_1_sisu/v2.0/.well-known/openid-configuration",
                "client_id": "8d5b446e-22b2-4e01-bb2e-9070f6b20c90",
                "redirect_uri": "vcclient://openid/"
              }
            ]
          },
          "validityInterval": 2592000,
          "vc": {
            "type": ["VerifiedCredentialExpert"]
          }
        }
      
    ```

   Ahora cambiaremos el campo "type" a "MyFirstVC". 

   ```json
    "type": ["MyFirstVC"]
  
   ```

   Guarde este cambio.

   >[!NOTE]
   > En este punto del tutorial no cambiaremos **"configuration"** ni **"client_id"** . Seguiremos usando el inquilino de Microsoft B2C que usamos en la [introducción](get-started-verifiable-credentials.md). Usaremos su instancia de Azure AD en el siguiente tutorial.

3. Abra el archivo MyFirstVC-display.json en el editor de código.

   ```json
       {
          "default": {
           "locale": "en-US",
           "card": {
             "title": "Verified Credential Expert",
             "issuedBy": "Microsoft",
             "backgroundColor": "#000000",
             "textColor": "#ffffff",
             "logo": {
               "uri": "https://didcustomerplayground.blob.core.windows.net/public/VerifiedCredentialExpert_icon.png",
               "description": "Verified Credential Expert Logo"
             },
             "description": "Use your verified credential to prove to anyone that you know all about verifiable credentials."
           },
           "consent": {
             "title": "Do you want to get your Verified Credential?",
             "instructions": "Sign in with your account to get your card."
           },
           "claims": {
             "vc.credentialSubject.firstName": {
               "type": "String",
               "label": "First name"
             },
             "vc.credentialSubject.lastName": {
               "type": "String",
               "label": "Last name"
             }
           }
         }
      }
   ```

   Haremos algunas modificaciones para que esta credencial verificable se distinga visualmente de la versión del código de ejemplo. 

    ```json
         "card": {
            "title": "My First VC",
            "issuedBy": "Your Issuer Name",
            "backgroundColor": "#ffffff",
            "textColor": "#000000",
          }
    ```
 
   >[!NOTE]
   > Para asegurarse de que la credencial es legible y accesible, es muy recomendable seleccionar los colores para el texto y el fondo con una [relación de contraste](https://www.w3.org/WAI/WCAG21/Techniques/general/G18) que sea al menos de 4,5:1.  

   Guarde estos cambios.

## <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

Antes de crear la primera credencial verificable, es necesario crear un contenedor de Blob Storage que pueda contener nuestros archivos de configuración y reglas.

1. Cree una cuenta de almacenamiento con las opciones que se muestran a continuación. Para ver los pasos detallados, consulte el artículo [Creación de una cuenta de almacenamiento](../../storage/common/storage-account-create.md?tabs=azure-portal).

   - **Suscripción:** elija la suscripción que se está usando para estos tutoriales.
   - **Grupo de recursos:** : elija el mismo grupo de recursos que hemos usado en los tutoriales anteriores (**vc-resource-group**).
   - **Nombre**: un nombre único.
   - **Ubicación:** (EE. UU.) Este de EE. UU.
   - **Rendimiento:** estándar.
   - **Tipo de cuenta:** : almacenamiento V2.
   - **Replicación:** redundancia local.
 
   ![Creación de una cuenta de almacenamiento nueva](media/enable-your-tenant-verifiable-credentials/create-storage-account.png)

2. Después de crear la cuenta de almacenamiento, es necesario crear un contenedor. Seleccione **Contenedores** en **Blob Storage** y cree un contenedor con los valores que se indican a continuación:

    - **Nombre:** vc-container
    - **Nivel de acceso público:** Privado (sin acceso anónimo)

   ![Crear un contenedor](media/enable-your-tenant-verifiable-credentials/new-container.png)

3. Ahora seleccione el nuevo contenedor y cargue los nuevos archivos de visualización y reglas **MyFirstVC-display.json** y **MyFirstVC-rules.json** creados anteriormente.

   ![carga del archivo de reglas](media/enable-your-tenant-verifiable-credentials/blob-storage-upload-rules-display-files.png)

## <a name="assign-blob-role"></a>Asignación de rol de blob

Antes de crear la credencial, es necesario proporcionar al usuario que ha iniciado sesión la asignación de roles correcta para que pueda acceder a los archivos en Storage Blob.

1. Vaya a **Almacenamiento** > **Contenedor**.
2. Elija **Control de acceso (IAM)** en el menú de la izquierda.
3. Seleccione **Asignaciones de roles**.
4. Seleccione **Agregar**.
5. En la sección **Rol**, elija **Lector de datos de blobs de almacenamiento**.
6. En **Asignar acceso a** elija **User, group, or service principle** (Usuario, grupo o entidad de servicio).
7. En **Seleccionar**: elija la cuenta que está usando para realizar estos pasos.
8. Seleccione **Guardar** para completar la asignación de roles.


   ![Adición de una asignación de roles](media/enable-your-tenant-verifiable-credentials/role-assignment.png)

  >[!IMPORTANT]
  >De forma predeterminada, a los creadores de contenedores se les asigna el rol **Propietario**. El rol **Propietario** no es suficiente por sí solo. Su cuenta necesita el rol **Lector de datos de blobs de almacenamiento**. Para más información, consulte [Uso de Azure Portal para asignar un rol de Azure para el acceso a datos de blobs y colas](../../storage/common/storage-auth-aad-rbac-portal.md).

## <a name="set-up-verifiable-credentials-preview"></a>Configuración de credenciales verificables (versión preliminar)

Ahora es necesario realizar el último paso para configurar el inquilino para las credenciales verificables.

1. En Azure Portal, busque **credenciales verificables**.
2. Seleccione **Credenciales comprobables (versión preliminar)** .
3. Seleccione **Introducción**.
4. Es necesario configurar la organización y proporcionar el nombre de la organización, el dominio y el almacén de claves. Veamos cada uno de los valores.

      - **Nombre de organización**: este nombre es el modo en que hace referencia a su negocio en el servicio de credenciales verificables. Este valor no se muestra a los clientes.
      - **Dominio**: el dominio especificado se agrega a un punto de conexión de servicio en el documento de DID. [Microsoft Authenticator](../user-help/user-help-auth-app-download-install.md) y otras carteras usan esta información para validar que el DID se ha [vinculado a su dominio](how-to-dnsbind.md). Si la cartera puede verificar el DID, se muestra un símbolo de verificación. Si la cartera no puede verificarlo, informa al usuario de que la credencial fue emitida por una organización que no ha podido validar. El dominio es lo que enlaza su DID a algo tangible que el usuario puede saber sobre su negocio.
      - **Almacén de claves**: proporcione el nombre de la instancia de Key Vault que ha creado anteriormente.
 
   >[!IMPORTANT]
   > El dominio no puede ser una redirección; de lo contrario, el DID y el dominio no se pueden vincular. Asegúrese de usar el formato https://www.domain.com.
    
5. Seleccione **Save and create credential** (Guardar y crear credencial).

    ![configuración de la identidad de la organización](media/enable-your-tenant-verifiable-credentials/save-create.png)

El inquilino ya está habilitado para la versión preliminar de la credencial verificable.

## <a name="create-your-vc-in-the-portal"></a>Creación de la credencial verificable en el portal

El paso anterior le deja en la página de **creación de una nueva credencial**. 

   ![introducción a las credenciales verificables](media/enable-your-tenant-verifiable-credentials/create-credential-after-enable-did.png)

1. En Nombre de credencial, agregue el nombre **MyFirstVC**. Este nombre se usa en el portal para identificar las credenciales verificables y se incluye como parte del contrato de las credenciales verificables.
2. En la sección del archivo de visualización, elija **Configure display file** (Configurar archivo de visualización).
3. En la sección **Cuentas de almacenamiento**, seleccione **woodgrovevcstorage**.
4. En la lista de contenedores disponibles, seleccione **vc-container**.
5. Elija el archivo **MyFirstVC-display.json** que creó anteriormente.
6. En **Create a new credential** (Crear nueva credencial) en la sección del **archivo de reglas**, elija **Configure rules file** (Configurar archivo de reglas).
7. En la sección **Cuentas de almacenamiento**, seleccione **woodgrovecstorage**.
8. Elija **vc-container**.
9. Seleccione **MyFirstVC-rules.json**.
10. En la pantalla **Create a new credential** (Crear nueva credencial), elija **Crear**.

   ![creación de una credencial](media/enable-your-tenant-verifiable-credentials/create-my-first-vc.png)

### <a name="credential-url"></a>Dirección URL de la credencial

Ahora que tiene una nueva credencial, copie la dirección URL de la credencial.

   ![Dirección URL de la credencial de emisión](media/enable-your-tenant-verifiable-credentials/issue-credential-url.png)

>[!NOTE]
>La dirección URL de la credencial es la combinación de los archivos de reglas y visualización. Es la dirección URL que Authenticator evalúa antes de mostrar los requisitos de emisión de las credenciales verificables del usuario.  

## <a name="update-the-sample-app"></a>Actualización de la aplicación de ejemplo

Ahora realizaremos modificaciones en el código del emisor de la aplicación de ejemplo para actualizarlo con la dirección URL de la credencial verificable. Esto le permite emitir credenciales verificables mediante su propio inquilino.

1. Vaya a la carpeta donde colocó los archivos de la aplicación de ejemplo.
2. Abra la carpeta del emisor y, a continuación, abra app.js en Visual Studio Code.
3. Actualice la constante "credential" con la nueva dirección URL de la credencial y establezca la constante "credentialType" en "MyFirstVC" y guarde el archivo.

    ![imagen de Visual Studio Code que muestra las áreas pertinentes resaltadas](media/enable-your-tenant-verifiable-credentials/sample-app-vscode.png)

4. Abra un símbolo del sistema y abra la carpeta del emisor.
5. Ejecute la aplicación Node actualizada.

    ```terminal
    node app.js
    ```

6. En otro símbolo del sistema, ejecute ngrok para configurar una dirección URL en 8081. Puede instalar ngrok globalmente mediante el [paquete npm ngrok](https://www.npmjs.com/package/ngrok/).

    ```terminal
    ngrok http 8081
    ```
    
    >[!IMPORTANT]
    > Es posible que también vea una advertencia que indica que esta aplicación o sitio web puede ser de riesgo. Este mensaje es normal en este momento, ya que aún no hemos vinculado el DID a su dominio. Siga las instrucciones de [enlace de DNS](how-to-dnsbind.md) para configurar esto.

    
7. Abra la dirección URL HTTPS generada por ngrok.

    ![Puntos de conexión de reenvío de NGROK](media/enable-your-tenant-verifiable-credentials/ngrok-url-screen.png)

8. Seleccione **GET CREDENTIAL** (Obtener credencial).
9. En Authenticator, escanee el código QR.
10. En el mensaje de advertencia **Usar esta aplicación o este sitio web puede ser arriesgado**, elija **Avanzado**.

  ![Advertencia inicial](media/enable-your-tenant-verifiable-credentials/site-warning.png)

11. En la advertencia de sitio web de riesgo, seleccione **Continuar de todos modos (no seguro)** .

  ![Segunda advertencia sobre el emisor](media/enable-your-tenant-verifiable-credentials/site-warning-proceed.png)


12. En la pantalla **Adición de credenciales**, observe ciertos aspectos: 
    1. En la parte superior de la pantalla, puede ver el mensaje **Sin comprobar** en rojo.
    1. La credencial se personaliza en función de los cambios realizados en el archivo de visualización.
    1. La opción **Iniciar sesión en tu cuenta** está señalando a **didplayground.b2clogin.com**.
    
   ![pantalla de adición de credencial con advertencia](media/enable-your-tenant-verifiable-credentials/add-credential-not-verified.png)

13. Elija **Iniciar sesión en tu cuenta** y autentíquese con los datos de credenciales que proporcionó en el [tutorial de introducción](get-started-verifiable-credentials.md).
14. Después de autenticarse correctamente, el botón **Agregar** ya no estará atenuado. Seleccione **Agregar**.

  ![pantalla de adición de credencial después de la autenticación](media/enable-your-tenant-verifiable-credentials/add-credential-not-verified-authenticated.png)

Hemos emitido una credencial verificable usando nuestro inquilino para generar la credencial verificable mientras seguimos usando nuestro inquilino de B2C para la autenticación.

  ![credencial verificable emitida por Azure AD y autenticada por nuestra instancia de Azure B2C](media/enable-your-tenant-verifiable-credentials/my-vc-b2c.png)


## <a name="test-verifying-the-vc-using-the-sample-app"></a>Prueba de comprobación de la credencial verificable mediante la aplicación de ejemplo

Ahora que hemos emitido la credencial verificable desde nuestro propio inquilino, vamos a comprobarla con nuestra aplicación de ejemplo.

>[!IMPORTANT]
> Para la prueba, use el mismo correo electrónico y la misma contraseña que usó en el [artículo de introducción](get-started-verifiable-credentials.md). Mientras el inquilino emite la credencial verificable, la entrada procede de un token de identificación emitido por el inquilino de Microsoft B2C. En el segundo tutorial, vamos a cambiar la emisión de tokens a su inquilino.

1. Abra **Configuración** en la hoja de credenciales verificables de Azure Portal. Copie el identificador descentralizado (DID).

   ![copia del DID](media/enable-your-tenant-verifiable-credentials/issuer-identifier.png)

2. Ahora, abra la carpeta del verificador que forma parte de los archivos de la aplicación de ejemplo. Debemos actualizar el archivo app.js en el código de ejemplo del verificador y realizar los cambios siguientes:

    - **credential**: cambie este valor a la dirección URL de su credencial.
    - **credentialType**: "MyFirstVC".
    - **issuerDid**: copie este valor de Azure Portal > Credenciales comprobables (versión preliminar) > Ajustes > Identificador descentralizado (DID).
    
   ![actualización de la constante issuerDid para que coincida con el identificador del emisor](media/enable-your-tenant-verifiable-credentials/editing-verifier.png)

3. Detenga la ejecución del servicio ngrok del emisor.

    ```cmd
    control-c
    ```

4. Ahora ejecute ngrok con el puerto 8082 del verificador.

    ```cmd
    ngrok http 8082
    ```

5. En otra ventana de terminal, vaya a la aplicación verificadora y ejecútela de la misma forma que se ejecutó la aplicación del emisor.

    ```cmd
    cd ..
    cd verifier
    node app.js
    ```

6. Abra la dirección URL de ngrok en el explorador y escanee el código QR con Authenticator en su dispositivo móvil.
7. En su dispositivo móvil, elija **Permitir** en la pantalla **Nueva solicitud de permiso**.

    >[!NOTE]
    > El DID que firma esta credencial verificable sigue siendo Microsoft B2C. El DID del verificador sigue siendo del inquilino de la aplicación de ejemplo de Microsoft. Puesto que el DID de Microsoft se ha vinculado a un dominio de nuestra propiedad, no se mostrará la advertencia como ha ocurrido durante el flujo de emisión. Esto se actualizará en la sección siguiente.
    
   ![nueva solicitud de permiso](media/enable-your-tenant-verifiable-credentials/new-permission-request.png)

8. Ha comprobado correctamente la credencial.

## <a name="next-steps"></a>Pasos siguientes

Ahora que el código de ejemplo emite una credencial verificable desde el emisor, puede continuar con la siguiente sección, en la que usará su propio proveedor de identidades para autenticar a los usuarios que intentan obtener una credencial verificable y usar su DID para firmar solicitudes de presentación.

> [!div class="nextstepaction"]
> [Tutorial: emisión y comprobación de credenciales verificables mediante el inquilino](issue-verify-verifiable-credentials-your-tenant.md)


