---
title: 'Sugerencia a los usuarios que configuren una aplicación Microsoft Authenticator (versión preliminar): Azure Active Directory'
description: Información sobre cómo trasladar la organización de los métodos de autenticación menos seguros a la aplicación Microsoft Authenticator
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 06/01/2021
ms.author: justinha
author: mjsantani
manager: daveba
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3d8599554918af41ead7c862ee2c84f2afd4b18a
ms.sourcegitcommit: 7f59e3b79a12395d37d569c250285a15df7a1077
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2021
ms.locfileid: "110786245"
---
# <a name="how-to-nudge-users-to-set-up-microsoft-authenticator-preview---microsoft-authenticator-app"></a>Cómo sugerir a los usuarios que configuren Microsoft Authenticator (versión preliminar): aplicación Microsoft Authenticator

Puede sugerir a los usuarios que configuren Microsoft Authenticator durante el inicio de sesión. Los usuarios pasarán por su inicio de sesión normal, realizarán la autenticación multifactor como de costumbre y, a continuación, se les pedirá que configuren Microsoft Authenticator. Puede incluir o excluir usuarios o grupos para controlar a quién se sugiere que configure la aplicación. Esto permite que las campañas dirigidas trasladen los usuarios desde los métodos de autenticación menos seguros a Microsoft Authenticator.  

Además de elegir a quién se puede sugerir, puede definir cuántos días un usuario puede posponer, o "posponer", la advertencia. Si un usuario pulsa en **Ahora no** para posponer la configuración de la aplicación, se le volverá a sugerir en el siguiente intento de MFA, después de que haya transcurrido el tiempo de la posposición. 

## <a name="prerequisites"></a>Prerrequisitos 

- Su organización debe haber habilitado Azure MFA. 
- El usuario no debe haber configurado aún Microsoft Authenticator para notificaciones push en su cuenta. 
- Los administradores deben habilitar a los usuarios en Microsoft Authenticator mediante una de estas directivas:  
  - Directiva de registro de MFA: se deberá habilitar a los usuarios para la **Notificación a través de aplicación móvil**.  
  - Directiva de métodos de autenticación: los usuarios deberán estar habilitados para Microsoft Authenticator y el modo de autenticación establecido en **Cualquiera** o **Inserción**. Si la directiva se establece en **Sin contraseña**, el usuario no será apto para la sugerencia. 

## <a name="user-experience"></a>Experiencia del usuario

1. El usuario realiza correctamente la operación de MFA con Azure MFA. 

1. El usuario ve el mensaje para configurar la aplicación Microsoft Authenticator para mejorar su experiencia de inicio de sesión. Nota: Solo los usuarios que tienen permitidas las notificaciones push de Microsoft Authenticator y no las tienen configuradas actualmente verán el mensaje. 

   ![El usuario realiza la autenticación multifactor](./media/how-to-nudge-authenticator-app/user-mfa.png)

1. El usuario pulsa en **Siguiente** y recorre la configuración de Microsoft Authenticator. 
   1. En primer lugar, descargue la aplicación.  

      ![El usuario descarga Microsoft Authenticator](./media/how-to-nudge-authenticator-app/download.png)

   1. Consulte cómo se configura Microsoft Authenticator. 
   
      ![El usuario configura Microsoft Authenticator](./media/how-to-nudge-authenticator-app/setup.png)

   1. Escanee el código QR. 

      ![El usuario escanea el código QR](./media/how-to-nudge-authenticator-app/scan.png)

   1. Apruebe la notificación de prueba.

      ![El usuario aprueba la notificación de prueba](./media/how-to-nudge-authenticator-app/test.png)

   1. Notificación aprobada.

      ![Confirmación de la aprobación](./media/how-to-nudge-authenticator-app/approved.png)

   1. La aplicación Authenticator ahora está configurada correctamente como el método de inicio de sesión predeterminado del usuario.

      ![Instalación completada](./media/how-to-nudge-authenticator-app/finish.png)

1. Si un usuario no quiere instalar la aplicación Authenticator, puede pulsar en **No ahora** para posponer el mensaje durante el número de días que defina un administrador. 
 
   ![Posposición de la instalación](./media/how-to-nudge-authenticator-app/snooze.png)

## <a name="enable-the-nudge-policy"></a>Habilitación de la directiva de sugerencia

Para habilitar la sugerencia, debe usar la directiva de métodos de autenticación mediante Graph API o los comandos de PowerShell. Los **administradores globales**  y los **administradores de directivas de métodos de autenticación** pueden actualizar la directiva. 

Para configurar la directiva mediante el Explorador de Graph:

1. Inicie sesión en el Explorador de Graph y asegúrese de que ha dado su consentimiento a los permisos **Policy.Read.All** y **Policy.ReadWrite.AuthenticationMethod**.

   Abra el panel Permisos:

   ![Captura de pantalla del Explorador de Graph](./media/how-to-nudge-authenticator-app/permissions.png)

1. Recupere la directiva de métodos de autenticación: `GET https://graph.microsoft.com/beta/policies/authenticationmethodspolicy`

1. Actualice la sección registrationEnforcement y authenticationMethodsRegistrationCampaign de la directiva para habilitar la sugerencia en un usuario o grupo.

   ![Sección Campaña](./media/how-to-nudge-authenticator-app/campaign.png)

Para actualizar la directiva, realice una revisión en la directiva de métodos de autenticación con solo la sección registrationEnforcement actualizada: `PATCH https://graph.microsoft.com/beta/policies/authenticationmethodspolicy`

En la tabla siguiente se enumeran las propiedades **authenticationMethodsRegistrationCampaign**.

| Nombre | Valores posibles | Descripción |
|------|-----------------|-------------|
| state |   "enabled"<br>"disabled"<br>"predeterminado" | Permite habilitar o deshabilitar la característica.<br>El valor predeterminado se usa cuando no se ha establecido explícitamente la configuración y usará el valor predeterminado de Azure AD para esta configuración. Actualmente se asigna en disabled.<br>Cambie los estados a enabled o disabled según sea necesario.  |
| snoozeDurationInDays | Intervalo: de 0 a 14 | Define el número de días después de los cuales el usuario volverá a ver la sugerencia.<br>Si el valor es 0, se sugiere al usuario durante cada intento de MFA.<br>Valor predeterminado: 1 día |
| includeTargets | N/D | Permite incluir distintos usuarios y grupos a los que quiere que se dirija la característica. |
| excludeTargets | N/D | Permite excluir distintos usuarios y grupos que quiere omitir de la característica. Si un usuario está en un grupo excluido y un grupo incluido, el usuario se excluirá de la característica.|

En la tabla siguiente se enumeran las propiedades **includeTargets**.

| Nombre | Valores posibles | Descripción |
|------|-----------------|-------------|
| targetType| "user"<br>"grupo" | Tipo de entidad de destino. |
| Identificador | Un identificador GUID | El id. del usuario o grupo de destino. |
| targetedAuthenticationMethod | "microsoftAuthenticator" | Se solicita al usuario del método de autenticación que se registre. El único valor permitido es "microsoftAuthenticator". |

En la tabla siguiente se enumeran las propiedades **excludeTargets**.

| Nombre       | Valores posibles   | Descripción                           |
|------------|-------------------|---------------------------------------|
| targetType | "user"<br>"grupo" | Tipo de entidad de destino.          |
| Identificador         | Una cadena          | El id. del usuario o grupo de destino. |

### <a name="examples"></a>Ejemplos

Estos son algunos JSON de ejemplo que puede usar para empezar. 

- Incluir a todos los usuarios 
  
  Si desea incluir TODOS los usuarios en el inquilino, simplemente [descargue este JSON](https://download.microsoft.com/download/1/4/E/14E6151E-C40A-42FB-9F66-D8D374D13B40/All%20Users%20Enabled.json) y péguelo en el Explorador de Graph y ejecute `PATCH` en el punto de conexión. 

  ```json
  {
  "registrationEnforcement": {
          "authenticationMethodsRegistrationCampaign": {
              "snoozeDurationInDays": 0,
              "state": "enabled",
              "excludeTargets": [],
              "includeTargets": [
                  {
                      "id": "all_users",
                      "targetType": "group",
                      "targetedAuthenticationMethod": "microsoftAuthenticator"
                  }
              ]
          }
      }
  }
  ```

- Incluir usuarios o grupos de usuarios específicos

  Si quiere incluir determinados usuarios o grupos en el inquilino, [descargue este JSON](https://download.microsoft.com/download/1/4/E/14E6151E-C40A-42FB-9F66-D8D374D13B40/Multiple%20Includes.json) y actualícelo con los GUID pertinentes de los usuarios y grupos. A continuación, pegue el JSON en el Explorador de Graph y ejecute `PATCH` en el punto de conexión. 

  ```json
  {
  "registrationEnforcement": {
        "authenticationMethodsRegistrationCampaign": {
            "snoozeDurationInDays": 0,
            "state": "enabled",
            "excludeTargets": [],
            "includeTargets": [
                {
                    "id": "*********PLEASE ENTER GUID***********",
                    "targetType": "group",
                    "targetedAuthenticationMethod": "microsoftAuthenticator"
                },
        {
                    "id": "*********PLEASE ENTER GUID***********",
                    "targetType": "user",
                    "targetedAuthenticationMethod": "microsoftAuthenticator"
                }
            ]
        }
    }
  ```

- Incluir y excluir usuarios o grupos de usuarios específicos

  Si quiere incluir Y excluir determinados usuarios o grupos de usuarios en el inquilino, [descargue este JSON](https://download.microsoft.com/download/1/4/E/14E6151E-C40A-42FB-9F66-D8D374D13B40/Multiple%20Includes%20and%20Excludes.json) y péguelo en el Explorador de Graph y ejecute `PATCH` en el punto de conexión. Escriba los GUID correctos para los usuarios y grupos.

  ```json
  {
  "registrationEnforcement": {
          "authenticationMethodsRegistrationCampaign": {
              "snoozeDurationInDays": 0,
              "state": "enabled",
              "excludeTargets": [
                  {
                      "id": "*********PLEASE ENTER GUID***********",
                      "targetType": "group"
                  },
        {
                      "id": "*********PLEASE ENTER GUID***********",
                      "targetType": "user"
                  }
              ],
              "includeTargets": [
                  {
                      "id": "*********PLEASE ENTER GUID***********",
                      "targetType": "group",
                      "targetedAuthenticationMethod": "microsoftAuthenticator"
                  },
        {
                      "id": "*********PLEASE ENTER GUID***********",
                      "targetType": "user",
                      "targetedAuthenticationMethod": "microsoftAuthenticator"
                  }
              ]
          }
      }
  }
  ```

### <a name="identify-the-guids-of-users-to-insert-in-the-jsons"></a>Identificación de los GUID de usuarios que se insertan en los JSON

1. Vaya a Azure Portal.
1. Pulse en **Azure Active Directory**.
1. En la hoja **Administrar**, pulse en **Usuarios**.
1. En la página **Usuarios**, identifique el usuario específico al que desea dirigirse.
1. Al pulsar en el usuario específico, verá su **id. de objeto**, que es el GUID del usuario.

   ![Id. de objeto de usuario](./media/how-to-nudge-authenticator-app/object-id.png)

### <a name="identify-the-guids-of-groups-to-insert-in-the-jsons"></a>Identificación de los GUID de grupos que se insertan en los JSON

1. Vaya a Azure Portal.
1. Pulse en **Azure Active Directory**.
1. En la hoja **Administrar**, pulse en **Grupos**.
1. En la página **Grupos**, identifique el grupo específico al que quiere dirigirse.
1. Pulse en el grupo y obtenga el **Id. de objeto**.

   ![Sugerencia a grupo](./media/how-to-nudge-authenticator-app/group.png)

<!---comment out PS until ready>

### PowerShell

1. Install the module.
1. Ensure you pass the right roles:
   
   ```powershell
   Connect-MgGraph -Scopes "User.Read.All","Group.ReadWrite.All"
   ```

1. Select the beta profile.
1. Call `Update-MgPolicyAuthenticationMethod`.

<---->

## <a name="limitations"></a>Limitaciones

La sugerencia no aparecerá en los dispositivos móviles que ejecutan Android o iOS.

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

**¿Estará esta característica disponible para el servidor MFA?** No. Esta característica solo estará disponible para los usuarios que usen Azure MFA. 

**¿Durante cuánto tiempo se ejecutará la campaña?** Puede usar las API para habilitar la campaña durante el tiempo que quiera. Cuando quiera que la campaña se deje de ejecutar, simplemente use las API para deshabilitar la campaña.  
 
**¿Puede tener cada grupo de usuarios una duración de posposición diferente?** No. La duración de la posposición del mensaje es una configuración para todo el inquilino y se aplica a todos los grupos del ámbito. 

**¿Se puede proteger a los usuarios para configurar el inicio de sesión por teléfono sin contraseña?** La característica pretende capacitar a los administradores para que los usuarios se configuren con MFA mediante la aplicación Authenticator y no el inicio de sesión por teléfono sin contraseña.  

**¿Verá la sugerencia un usuario que tenga una configuración de aplicación de autenticación de terceros?** Si este usuario no tiene la aplicación Microsoft Authenticator configurada para notificaciones push y están habilitadas para ella mediante directiva, sí, el usuario verá la sugerencia. 

**¿Verá la sugerencia un usuario que tenga una aplicación Microsoft Authenticator configurada solo para códigos TOTP?** Sí. Si la aplicación Microsoft Authenticator no está configurada para notificaciones push y el usuario está habilitado para ella mediante directiva, sí, el usuario verá la sugerencia.

**Si un usuario acaba de pasar por el registro de MFA, ¿se le sugerirá en la misma sesión de inicio de sesión?** No. Para proporcionar una buena experiencia de usuario, no se sugerirá a los usuarios que configuren Authenticator en la misma sesión en la que registraron otros métodos de autenticación.  

**¿Puedo sugerir a mis usuarios que registren otro método de autenticación?** No. Por ahora, la característica pretende sugerir a los usuarios que configuren solo la aplicación Microsoft Authenticator. 

**¿Hay alguna manera de ocultar la opción de posposición y forzar a mis usuarios a configurar la aplicación Authenticator?**  
No hay ninguna manera de ocultar la opción de posposición en la sugerencia. Puede establecer snoozeDuration en 0, lo que garantiza que los usuarios verán la sugerencia durante cada intento de MFA.  

**¿Puedo sugerir a mis usuarios si no uso Azure MFA?** No. La sugerencia solo funcionará para los usuarios que están realizando la MFA mediante el servicio Azure MFA. 

**¿Se sugerirá a los usuarios invitados o B2B de mi inquilino?** Sí. Si se encuentran en el ámbito para la sugerencia según la directiva. 

**¿Qué ocurre si el usuario cierra el explorador?** Es lo mismo que la posposición.


## <a name="next-steps"></a>Pasos siguientes

[Habilitación del inicio de sesión sin contraseña con la aplicación Microsoft Authenticator](howto-authentication-passwordless-phone.md)