---
title: 'Servidor FHIR con Postman en Azure: Azure API for FHIR'
description: En este tutorial, se le guía por los pasos necesarios para usar Postman para acceder a un servidor de FHIR. Postman es útil para depurar aplicaciones que tienen acceso a las API.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: cavoeg
author: matjazl
ms.date: 03/26/2021
ms.openlocfilehash: 72e5711ca813378e291d48bdaaa5803693d91482
ms.sourcegitcommit: 91fdedcb190c0753180be8dc7db4b1d6da9854a1
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2021
ms.locfileid: "112284030"
---
# <a name="access-azure-api-for-fhir-with-postman"></a>Acceso a Azure API for FHIR con Postman

Una aplicación cliente puede acceder a la Azure API for FHIR mediante una [API REST](https://www.hl7.org/fhir/http.html). Para enviar solicitudes, ver respuestas y depurar la aplicación a medida que se compila, use la herramienta de pruebas de API que prefiera. En este tutorial, le guíamos por los pasos para acceder al servidor de FHIR mediante [Postman](https://www.getpostman.com/). 

## <a name="prerequisites"></a>Requisitos previos

- Un punto de conexión de FHIR en Azure. 

  Para implementar el Azure API for FHIR (un servicio administrado), puede usar el Azure Portal [,](fhir-paas-portal-quickstart.md) [PowerShell](fhir-paas-powershell-quickstart.md) [o CLI de Azure](fhir-paas-cli-quickstart.md).

- Aplicación cliente [confidencial registrada para](register-confidential-azure-ad-client-app.md) acceder al servicio FHIR.
- Ha concedido permisos a la aplicación cliente confidencial, por ejemplo, "Colaborador de datos de FHIR", para acceder al servicio FHIR. Para más información, consulte [Configuración de RBAC de Azure para FHIR.](./configure-azure-rbac.md)
- Postman instalado. 
    
  Para obtener más información sobre Postman, vea [Introducción a Postman.](https://www.getpostman.com)

## <a name="fhir-server-and-authentication-details"></a>Detalles de autenticación y servidor de FHIR

Para usar Postman, se requieren los siguientes parámetros de autenticación:

- La dirección URL del servidor de FHIR, por ejemplo, `https://MYACCOUNT.azurehealthcareapis.com`

- El valor `Authority` del proveedor de identidades para el servidor de FHIR, por ejemplo `https://login.microsoftonline.com/{TENANT-ID}`

- Configurado que `audience` suele ser la dirección URL del servidor de FHIR, por ejemplo, o `https://<FHIR-SERVER-NAME>.azurehealthcareapis.com` `https://azurehealthcareapis.com` .

- Identificador `client_id` de aplicación o de la aplicación cliente confidencial que se [usa](register-confidential-azure-ad-client-app.md) para acceder al servicio FHIR.

- Secreto `client_secret` de aplicación o de la aplicación cliente confidencial.

Por último, debe comprobar que `https://www.getpostman.com/oauth2/callback` es una dirección URL de respuesta registrada para la aplicación cliente.

## <a name="connect-to-fhir-server"></a>Conexión al servidor de FHIR

Abra Postman y, a continuación, **seleccione GET** para realizar una solicitud a `https://fhir-server-url/metadata` .

![Instrucción de funcionalidad de metadatos de Postman](media/tutorial-postman/postman-metadata.png)

La dirección URL de metadatos para Azure API for FHIR es `https://MYACCOUNT.azurehealthcareapis.com/metadata`. 

En este ejemplo, la dirección URL del servidor de FHIR es y la instrucción de funcionalidad `https://fhirdocsmsft.azurewebsites.net` del servidor está disponible en `https://fhirdocsmsft.azurewebsites.net/metadata` . Este punto de conexión es accesible sin autenticación.

Si intenta acceder a recursos restringidos, se produce una respuesta "Error de autenticación".

![Error de autenticación](media/tutorial-postman/postman-authentication-failed.png)

## <a name="obtaining-an-access-token"></a>Obtención de un token de acceso
Seleccione **Obtener token de acceso nuevo**.

Para obtener un token de acceso válido, seleccione **Autorización** y **OAuth 2.0** en **el** menú desplegable TIPO.

![Establecimiento de OAuth 2.0](media/tutorial-postman/postman-select-oauth2.png)

Seleccione **Obtener token de acceso nuevo**.

![Solicitar nuevo token de acceso](media/tutorial-postman/postman-request-token.png)

En el **cuadro de diálogo Obtener nuevo token** de acceso, escriba los detalles siguientes:

| Campo                 | Valor de ejemplo                                                                                                   | Comentario                    |
|-----------------------|-----------------------------------------------------------------------------------------------------------------|----------------------------|
| Nombre del token            | MYTOKEN                                                                                                         | Un nombre que elija          |
| Tipo de concesión            | Código de autorización                                                                                              |                            |
| Dirección URL de devolución de llamadas          | `https://www.getpostman.com/oauth2/callback`                                                                    |                            |
| Dirección URL de autenticación              | `https://login.microsoftonline.com/{TENANT-ID}/oauth2/authorize?resource=<audience>` | El valor`audience` es `https://MYACCOUNT.azurehealthcareapis.com` para Azure API for FHIR |
| Dirección URL del token de acceso      | `https://login.microsoftonline.com/{TENANT ID}/oauth2/token`                                                    |                            |
| Id. de cliente             | `XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXXXXX`                                                                           | Identificador de aplicación             |
| Secreto del cliente         | `XXXXXXXX`                                                                                                      | Clave de cliente secreta          |
| Ámbito | `<Leave Blank>` | No se usa el ámbito; por lo tanto, se puede dejar en blanco.  
| Estado                 | `1234`     | [El](https://learning.postman.com/docs/sending-requests/authorization/) estado es un valor opaco para evitar la falsificación de solicitud entre sitios. Es opcional y puede tomar un valor arbitrario como "1234".                           |
| Autenticación de clientes | Enviar credenciales de cliente en el cuerpo                                                                                 |                 

Seleccione **Solicitar token para** guiarse por el flujo Azure Active Directory autenticación y se devolverá un token a Postman. Si se produce un error de autenticación, consulte la consola de Postman para obtener más detalles. **Nota:** En la cinta de opciones, seleccione **Ver** y, a continuación, **seleccione Mostrar consola de Postman.** El método abreviado de teclado para la consola de Postman **es Alt-Ctrl+C.**

Desplácese hacia abajo para ver la pantalla de token devuelta y, a continuación, **seleccione Usar token.**

![Usar token](media/tutorial-postman/postman-use-token.png)

Consulte el campo **Token de** acceso para ver el token recién rellenado. Si selecciona Enviar **para** repetir la búsqueda `Patient` de recursos, se devuelve **un** `200 OK` estado. Un estado devuelto `200 OK` indica una solicitud HTTP correcta.

![200 OK](media/tutorial-postman/postman-200-OK.png)

En el *ejemplo de búsqueda de* pacientes, no hay ningún paciente en la base de datos de modo que el resultado de la búsqueda esté vacío.

Puede inspeccionar el token de acceso mediante una herramienta como [jwt.ms](https://jwt.ms). A continuación se muestra un ejemplo del contenido.

```json
{
  "aud": "https://MYACCOUNT.azurehealthcareapis.com",
  "iss": "https://sts.windows.net/{TENANT-ID}/",
  "iat": 1545343803,
  "nbf": 1545343803,
  "exp": 1545347703,
  "acr": "1",
  "aio": "AUQAu/8JXXXXXXXXXdQxcxn1eis459j70Kf9DwcUjlKY3I2G/9aOnSbw==",
  "amr": [
    "pwd"
  ],
  "appid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "oid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "appidacr": "1",

  ...// Truncated
}
```

En situaciones de solución de problemas, un buen punto de partida es validar que tiene la audiencia correcta (notificación `aud`). Si el token es del emisor correcto (notificación) y tiene la audiencia correcta (notificación), pero todavía no puede acceder a la API de FHIR, es probable que el usuario o la entidad de servicio (notificación) no tenga acceso al plano de datos de `iss` `aud` `oid` FHIR. Se recomienda usar el [control de acceso basado en rol de Azure (RBAC de Azure)](configure-azure-rbac.md) para asignar roles de plano de datos a los usuarios. Si usa un inquilino externo de Azure Active Directory secundario para el plano de datos, deberá configurar RBAC local para las asignaciones [de FHIR.](configure-local-rbac.md)

También es posible obtener un token para el Azure API for FHIR [mediante el CLI de Azure](get-healthcare-apis-access-token-cli.md). Si usa un token obtenido con el CLI de Azure, debe usar el tipo de autorización *Bearer Token*. Pegue el token directamente.

## <a name="inserting-a-patient"></a>Inserción de un paciente

Con un token de acceso válido, ahora puede insertar un nuevo paciente. En Postman, cambie el método **seleccionando Publicar** y agregue el siguiente documento JSON en el cuerpo de la solicitud.

[!code-json[](../samples/sample-patient.json)]

Seleccione **Enviar** para determinar que el paciente se ha creado correctamente.

![Captura de pantalla que muestra que el paciente se ha creado correctamente.](media/tutorial-postman/postman-patient-created.png)

Si repite la búsqueda de pacientes, ahora debería ver el registro de pacientes.

![Paciente creado](media/tutorial-postman/postman-patient-found.png)

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha accedido al Azure API for FHIR mediante Postman. Para obtener más información sobre las características Azure API for FHIR, vea
 
>[!div class="nextstepaction"]
>[Características compatibles](fhir-features-supported.md)
