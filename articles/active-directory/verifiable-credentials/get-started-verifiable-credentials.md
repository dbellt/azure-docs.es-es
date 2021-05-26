---
title: 'Tutorial: Introducción a Azure Active Directory Verifiable Credentials con una aplicación de ejemplo (versión preliminar)'
description: En este tutorial, aprenderá a emitir credenciales verificables mediante nuestra aplicación de ejemplo y el inquilino de prueba.
ms.service: active-directory
ms.subservice: verifiable-credentials
author: barclayn
ms.author: barclayn
ms.topic: tutorial
ms.date: 04/01/2021
ms.openlocfilehash: 2fba830b93be35b1bc55ad85531737ebe8a8c982
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110466239"
---
#  <a name="tutorial---get-started-with-azure-active-directory-verifiable-credentials-using-a-sample-app-preview"></a>Tutorial: Introducción a Azure Active Directory Verifiable Credentials con una aplicación de ejemplo (versión preliminar)

En este tutorial, se describen los pasos necesarios para emitir la primera credencial verificable: una tarjeta de experto en credenciales verificado. A continuación, puede usar esta tarjeta para demostrar a un verificador que es un experto en credenciales verificado que domina el arte de la emisión de credenciales digitales. Empiece a usar Azure Active Directory Verifiable Credentials mediante la aplicación de ejemplo para emitir su primera credencial verificable.

![Esta es la imagen de una tarjeta de ejemplo.](media/get-started-verifiable-credentials/verifiedcredentialexpert-card.png)

> [!IMPORTANT]
> Azure Active Directory Verifiable Credentials se encuentra actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

- [NodeJS](https://nodejs.org/en/download/) versión 10.14 o posterior instalada en nuestro sistema de prueba.
- Necesita que [GIT](https://git-scm.com/downloads) esté instalado si desea clonar el repositorio que hospeda la aplicación de ejemplo.
- [Visual Studio Code](https://code.visualstudio.com/Download)
- Un sistema para hospedar nuestro sitio de ejemplo.
- Un dispositivo móvil con Microsoft Authenticator versión 6.2005.3599 o posterior instalada.
- [NGROK](https://ngrok.com/) gratis.

## <a name="download-the-sample-code"></a>Descarga del código de ejemplo

Para emitir una tarjeta de experto en credenciales verificado, debe ejecutar un sitio web en la máquina local. El sitio web se usa para iniciar un proceso de emisión de credenciales verificables. Hemos proporcionado un sitio web sencillo, escrito en NodeJS, que usaremos en este tutorial.

En primer lugar, descargue el código de ejemplo de Github [aquí](https://github.com/Azure-Samples/active-directory-verifiable-credentials), o bien, clone el repositorio en la máquina local:

```terminal
git clone https://github.com/Azure-Samples/active-directory-verifiable-credentials.git
```

Puede que desee familiarizarse con el código de los sitios web de ejemplo. La carpeta `issuer` contiene todo el código que se usa para emitir una credencial verificable. Puede encontrar más detalles en el [archivo Léame](https://github.com/Azure-Samples/active-directory-verifiable-credentials) del ejemplo.

## <a name="run-the-issuer-website"></a>Ejecución del sitio web del emisor

Puede ejecutar los pasos desde Visual Studio Code o cualquier línea de comandos disponible en el sistema operativo. 

1. Vaya a la carpeta `issuer`. 

    ```terminal
    cd issuer
    ```

2. Una vez allí, es necesario instalar todos los paquetes necesarios e iniciar el sitio.

   ```terminal
    npm install
    node app.js
    ```

3. En el terminal, ahora verá que la aplicación emisora está escuchando en el puerto 8081. Ahora vamos a configurar un proxy inverso con ngrok para que Authenticator pueda comunicarse con la aplicación. 

## <a name="creating-a-reverse-proxy-with-ngrok"></a>Creación de un proxy inverso con ngrok

Al ejecutar el sitio web de ejemplo, el dispositivo debe comunicarse con el servidor de Node que se ejecuta en la máquina local. Se recomienda usar [ngrok](https://ngrok.com/) como una manera fácil de hacer que el servidor de desarrollo local esté disponible en Internet.

1.  Después de descargar y extraer **ngrok**, es necesario ejecutar:

    ```terminal
     ngrok http 8081
    ```

De forma predeterminada, el sitio web de ejemplo se ejecuta en el puerto `8081`. **ngrok** genera dos direcciones URL de reenvío para el servidor. Copie la dirección URL con el prefijo `https://`.

![ngrok ayuda a que los puntos de conexión de la aplicación estén disponibles a través de Internet.](media/get-started-verifiable-credentials/ngrok.png)

>[!NOTE] 
> Si usa PowerShell, puede que tenga que escribir `./ngrok` para que se reconozca el comando.

Ahora que el puerto local está expuesto a Internet mediante ngrok, el sitio de ejemplo usará automáticamente el nombre de host que genere ngrok. Abra el explorador y vaya a la dirección URL de reenvío de HTTPS de ngrok. Debe poder visitar correctamente la página principal del sitio de ejemplo. Si se abre la página, el dispositivo puede comunicarse con la aplicación de ejemplo que se ejecuta en el servidor local. Ya está listo para emitir una tarjeta de experto en credenciales verificado.

## <a name="issue-a-credential"></a>Emisión de una credencial

1. Instale Authenticator en el dispositivo móvil. Microsoft Authenticator se usa para recibir, almacenar y presentar las credenciales verificables a las partes interesadas.

2. A continuación, emita una credencial verificable. **Haga clic** en el botón **Get Credential** (Obtener credencial). Al hacer clic en el botón **Get Credential** (Obtener credencial), el sitio web de ejemplo muestra un código QR que se puede digitalizar mediante Authenticator. Si ve el sitio desde el explorador del dispositivo móvil, al hacer clic en el botón **Get Credential** (Obtener credencial), se desencadena un vínculo profundo que abre la aplicación Authenticator y no requiere la digitalización de un código QR.

   ![Botón Get Credential (Obtener credencial)](media/get-started-verifiable-credentials/credential-expert-get.png)

3. Digitalice el código QR del sitio web mediante Authenticator, o si tiene acceso al sitio web a través de un dispositivo móvil, haga clic en el botón Get Credential (Obtener credencial) para desencadenar el vínculo profundo. 

   ![Código QR ](media/get-started-verifiable-credentials/credential-expert-issue.png)

4. Observe que el botón **Add** (Agregar) está atenuado en este momento. Elija **Iniciar sesión en tu cuenta** debajo de la imagen de la tarjeta.

   ![Iniciar sesión ](media/get-started-verifiable-credentials/add-verified-credential-expert.png)

5. Antes de obtener la tarjeta de experto en credenciales, el inquilino que estamos usando requiere que proporcione información de autenticación. Si esta es la primera vez que realiza el tutorial, no tiene una cuenta de experto en credenciales. Cree una nueva cuenta de usuario en el inquilino de B2C.

   ![autenticación antes de continuar](media/get-started-verifiable-credentials/authenticate-credential-expert.png)

6. Una vez que haya iniciado sesión, el botón **Add** (Agregar) ya no estará atenuado. Elija **Add** (Agregar) para aceptar el nuevo VC.

   ![Elegir Add (Agregar) después de la autenticación](media/get-started-verifiable-credentials/add-verified-credential-expert-after-auth.png) 


7. Felicidades. Ahora tiene un VC de experto en credenciales verificado.

   ![VC de experto en credenciales agregado](media/get-started-verifiable-credentials/credential-expert-add-card.png) 
 
Después, es el momento de comprobar la credencial.

## <a name="validate-credentials"></a>Validar las credenciales,

Ahora que ha completado la parte de emisión del tutorial y que tiene una credencial verificable en Authenticator, es el momento de validarla en su propia aplicación verificadora.

1.  Detenga la ejecución del servicio ngrok del emisor.

    ```terminal
     control+c
    ```


2. En otra ventana de terminal, abra la carpeta de la aplicación verificadora y ejecútela de la misma forma que se ejecutó la aplicación del emisor.

    ```terminal
     cd verifier
     npm install
     node app.js
    ```

3. Ahora ejecute ngrok con el puerto del verificador 8082.

    ```terminal
    ngrok http 8082
    ```

4. Abra la dirección URL de reenvío de HTTPS de ngrok en el explorador y pulse en el botón **VERIFY CREDENTIAL** (VERIFICAR CREDENCIAL).  

   ![botón verificar el experto en credenciales](media/get-started-verifiable-credentials/prove-credential-expert.png)

5. Abra Authenticator y digitalice el código QR.

   ![digitalizar el código QR para verificar la credencial](media/get-started-verifiable-credentials/scan-verify.png)

  > [!IMPORTANT]
  > En iOS está en la parte superior derecha y en Android en la parte inferior derecha. Digitalice el código QR.

6. Elija **Allow** (Permitir) en la pantalla nueva solicitud de permiso en Authenticator. Al presionar Allow (Permitir), está firmando una presentación verificable con su DID (identidad descentralizada) para demostrar que controla, de hecho, esta credencial verificable.

   ![nueva solicitud de permiso](media/get-started-verifiable-credentials/new-permission-request.png)

    Después de una presentación correcta, se deben haber actualizado tres elementos:

   1. En la página web debería aparecer ahora "Enhorabuena, su nombre" + es un experto en credenciales verificado".
   
    ![Enhorabuena, verificar de nuevo](media/get-started-verifiable-credentials/congratulations.png)


   2. El terminal de la aplicación del verificador también debe mostrar el mismo mensaje de los registros.
   
    ![actividad de la aplicación en el símbolo del sistema](media/get-started-verifiable-credentials/cmd-verified-expert.png)

   3. En Authenticator, debe haber una entrada para la actividad reciente de esta presentación.

    ![Actividad en Authenticator](media/get-started-verifiable-credentials/recent-activity.png)

   
>[!NOTE]
> Mientras se ejecuta la aplicación verificadora, ngrok puede dejar de funcionar y mostrar un error que indica que hay demasiadas conexiones. Para evitar esto, puede registrar la cuenta con ngrok. 

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha completado correctamente la guía de inicio rápido, es el momento de crear su propia identidad no centralizada en el servicio Azure Active Directory Verifiable Credentials y emitir sus propias credenciales verificables.

>[!div class="nextstepaction"] 
>[Configuración de su propio emisor mediante la aplicación de ejemplo de credenciales verificables](./enable-your-tenant-verifiable-credentials.md)
