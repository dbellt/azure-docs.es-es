---
title: Configuración de un Pase de acceso temporal en Azure AD para registrar métodos de autenticación sin contraseña
description: Obtenga información sobre cómo configurar y permitir que los usuarios registren métodos de autenticación sin contraseña mediante un Pase de acceso temporal.
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 04/26/2021
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: inbarckms
ms.collection: M365-identity-device-management
ms.openlocfilehash: 0f0473a266cffb083a82ffc9afb333da7eecf59c
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108015706"
---
# <a name="configure-temporary-access-pass-in-azure-ad-to-register-passwordless-authentication-methods-preview"></a>Configuración de un Pase de acceso temporal en Azure AD para registrar métodos de autenticación sin contraseña (versión preliminar)

Los métodos de autenticación sin contraseña, como FIDO2 y el inicio de sesión telefónico sin contraseña a través de la aplicación Microsoft Authenticator, permiten a los usuarios iniciar sesión de manera segura sin una contraseña. Los usuarios pueden arrancar métodos sin contraseña de una de dos maneras:

- Con los métodos de autenticación multifactor de Azure AD existentes 
- Usando un Pase de acceso temporal (TAP) 

Un Pase de acceso temporal es un código de acceso de tiempo limitado que emite un administrador, que satisface los requisitos de autenticación sólida y que se puede usar para incorporar otros métodos de autenticación, incluidos aquellos sin contraseña. Asimismo, el Pase de acceso temporal facilita la recuperación cuando un usuario ha perdido u olvidado su factor de autenticación sólida, como una clave de seguridad FIDO2 o la aplicación Microsoft Authenticator, pero debe iniciar sesión para registrar nuevos métodos de autenticación sólida.


En este artículo se muestra cómo habilitar y usar el Pase de acceso temporal en Azure AD mediante Azure Portal. También puede realizar estas acciones con las API REST. 

>[!NOTE]
>El Pase de acceso temporal está actualmente en versión preliminar pública. Es posible que algunas características no se admitan o que tengan funcionalidades limitadas. 

## <a name="enable-the-temporary-access-pass-policy"></a>Habilitación de la directiva del Pase de acceso temporal

Una directiva de Pase de acceso temporal define la configuración, como la duración de los pases creados en el inquilino, o los usuarios y grupos que pueden usar un Pase de acceso temporal para iniciar sesión. Antes de que un usuario pueda iniciar sesión con un Pase de acceso temporal, debe habilitar la directiva del método de autenticación y elegir qué usuarios y grupos pueden iniciar sesión mediante un Pase de acceso temporal.
Aunque puede crear un Pase de acceso temporal para cualquier usuario, solo aquellos incluidos en la directiva pueden iniciar sesión con él.

Los titulares del rol Administrador global y Administrador de la directiva del método de autenticación pueden actualizar la directiva del método de autenticación del Pase de acceso temporal.
Para configurar la directiva del método de autenticación del Pase de acceso temporal:

1. Inicie sesión en Azure Portal como Administrador global y haga clic en **Azure Active Directory** > **Seguridad** > **Método de autenticación** > **Pase de acceso temporal**.
1. Haga clic en **Sí** para habilitar la directiva, seleccione qué usuarios tienen aplicada la directiva y los valores de **General**.

   ![Captura de pantalla que muestra cómo habilitar la directiva del método de autenticación del Pase de acceso temporal.](./media/how-to-authentication-temporary-access-pass/policy.png)

   En la tabla siguiente se describen el valor predeterminado y el intervalo de valores permitidos.


   | Configuración | Valores predeterminados | Valores permitidos | Comentarios |
   |---|---|---|---|
   | Duración mínima | 1 hora | De 10 a 43 200 minutos (30 días) | Número mínimo de minutos que el Pase de acceso temporal es válido. |
   | Duración máxima | 24 horas | De 10 a 43 200 minutos (30 días) | Número máximo de minutos que el Pase de acceso temporal es válido. |
   | Duración predeterminada | 1 hora | De 10 a 43 200 minutos (30 días) | Los valores predeterminados se pueden invalidar mediante los pases individuales, dentro de la vigencia mínima y máxima configurada por la directiva. |
   | Uso único | False | True o False | Cuando la directiva se establece en False, se pueden usar los pases en el inquilino una vez o más de una vez durante su validez (vigencia máxima). Al aplicar un uso único en la directiva del Pase de acceso temporal, todos los pases creados en el inquilino se crearán como de un solo uso. |
   | Length | 8 | De 8 a 48 caracteres | Define la longitud del código de acceso. |

## <a name="create-a-temporary-access-pass"></a>Creación de un Pase de acceso temporal

Después de habilitar una directiva, puede crear un Pase de acceso temporal para un usuario en Azure AD. Estos roles pueden realizar las siguientes acciones relacionadas con un Pase de acceso temporal.

- El Administrador global puede crear, eliminar y ver el Pase de acceso temporal de cualquier usuario (excepto el suyo).
- Los Administradores de autenticación con privilegios pueden crear, eliminar y ver el Pase de acceso temporal en administradores y miembros (excepto el suyo).
- Los Administradores de autenticación pueden crear, eliminar y ver el Pase de acceso temporal en miembros (excepto el suyo).
- El Administrador global puede ver los detalles del Pase de acceso temporal en el usuario (sin tener que leer el propio código).

1. Inicie sesión en Azure Portal como Administrador global, Administrador de autenticación con privilegios o Administrador de autenticación. 
1. Haga clic en **Azure Active Directory**, vaya a Usuarios, seleccione un usuario, como *Chris Green*, y, a continuación, elija **Métodos de autenticación**.
1. Si es necesario, seleccione la opción para **Try the new user authentication methods experience** (Probar la nueva experiencia de métodos de autenticación de usuario).
1. Seleccione la opción para **Add authentication methods** (Agregar métodos de autenticación).
1. Debajo de **Seleccionar método**, haga clic en **Temporary Access Pass (Preview)** (Pase de acceso temporal [versión preliminar]).
1. Defina una duración o una hora de activación personalizada y haga clic en **Agregar**.

   ![Captura de pantalla sobre cómo crear un Pase de acceso temporal.](./media/how-to-authentication-temporary-access-pass/create.png)

1. Una vez que se agregue, se muestran los detalles del Pase de acceso temporal. Tome nota del valor real del Pase de acceso temporal. Este valor se proporcionará al usuario. No puede ver este valor después de hacer clic en **Aceptar**.
   
   ![Captura de pantalla de los detalles del Pase de acceso temporal.](./media/how-to-authentication-temporary-access-pass/details.png)

Los comandos siguientes muestran cómo crear y obtener un Pase de acceso temporal mediante PowerShell:

```powershell
# Create a Temporary Access Pass for a user
$properties = @{}
$properties.isUsableOnce = $True
$properties.startDateTime = '2021-03-11 06:00:00'
$propertiesJSON = $properties | ConvertTo-Json

New-MgUserAuthenticationTemporaryAccessPassMethod -UserId user2@contoso.com -BodyParameter $propertiesJSON

Id                                   CreatedDateTime       IsUsable IsUsableOnce LifetimeInMinutes MethodUsabilityReason StartDateTime         TemporaryAccessPass
--                                   ---------------       -------- ------------ ----------------- --------------------- -------------         -------------------
c5dbd20a-8b8f-4791-a23f-488fcbde3b38 9/03/2021 11:19:17 PM False    True         60                NotYetValid           11/03/2021 6:00:00 AM TAPRocks!

# Get a user's Temporary Access Pass
Get-MgUserAuthenticationTemporaryAccessPassMethod -UserId user3@contoso.com

Id                                   CreatedDateTime       IsUsable IsUsableOnce LifetimeInMinutes MethodUsabilityReason StartDateTime         TemporaryAccessPass
--                                   ---------------       -------- ------------ ----------------- --------------------- -------------         -------------------
c5dbd20a-8b8f-4791-a23f-488fcbde3b38 9/03/2021 11:19:17 PM False    True         60                NotYetValid           11/03/2021 6:00:00 AM

```

## <a name="use-a-temporary-access-pass"></a>Uso de un Pase de acceso temporal

El uso más común de un Pase de acceso temporal es para que un usuario registre los detalles de autenticación durante el primer inicio de sesión, sin necesidad de completar mensajes de seguridad adicionales. Los métodos de autenticación se registran en [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo). Los usuarios también pueden actualizar los métodos de autenticación existentes aquí.

1. Abra un explorador web en [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo).
1. Escriba el UPN de la cuenta para la que creó el Pase de acceso temporal, como *tapuser@contoso.com* .
1. Si el usuario está incluido en la directiva del Pase de acceso temporal, verá una pantalla para escribir el Pase de acceso temporal.
1. Escriba el Pase de acceso temporal que se mostró en Azure Portal.

   ![Captura de pantalla sobre cómo escribir un Pase de acceso temporal.](./media/how-to-authentication-temporary-access-pass/enter.png)

>[!NOTE]
>En el caso de los dominios federados, se prefiere usar un Pase de acceso temporal en vez de la federación. Un usuario con un Pase de acceso temporal completará la autenticación en Azure AD y no se le redirigirá al proveedor de identidades federado (IdP).

El usuario ahora ha iniciado sesión y puede actualizar o registrar un método, como una llave de seguridad FIDO2. Los usuarios que actualicen sus métodos de autenticación debido a la pérdida de sus credenciales o dispositivos deben asegurarse de que quitan los métodos de autenticación antiguos.

Los usuarios también pueden usar su Pase de acceso temporal para registrarse en el inicio de sesión telefónico sin contraseña directamente desde la aplicación Authenticator. Para obtener más información, consulte [Agregar la cuenta profesional o educativa a la aplicación Microsoft Authenticator](../user-help/user-help-auth-app-add-work-school-account.md).

![Captura de pantalla de cómo escribir un Pase de acceso temporal mediante una cuenta profesional o educativa.](./media/how-to-authentication-temporary-access-pass/enter-work-school.png)

## <a name="delete-a-temporary-access-pass"></a>Eliminación de un Pase de acceso temporal

No se puede usar un Pase de acceso temporal que haya expirado. En la opción **Métodos de autenticación** de un usuario, la columna **Detalle** muestra cuándo expiró el Pase de acceso temporal. Puede eliminar un Pase de acceso temporal que haya expirado siguiendo estos pasos:

1. En el portal de Azure AD, vaya a **Usuarios**, seleccione un usuario, por ejemplo, *Usuario de TAP* y, después, elija **Métodos de autenticación**.
1. En el lado derecho del método de autenticación **Temporary Access Pass (Preview)** (Pase de acceso temporal [vista previa]) que se muestra en la lista, seleccione **Eliminar**.

También puede usar PowerShell:

```powershell
# Remove a user's Temporary Access Pass
Remove-MgUserAuthenticationTemporaryAccessPassMethod -UserId user3@contoso.com -TemporaryAccessPassAuthenticationMethodId c5dbd20a-8b8f-4791-a23f-488fcbde3b38
```

## <a name="replace-a-temporary-access-pass"></a>Reemplazo de un Pase de acceso temporal 

- Un usuario solo puede tener un Pase de acceso temporal. El código de acceso se puede usar durante las horas de inicio y finalización del Pase de acceso temporal.
- Si el usuario requiere un nuevo Pase de acceso temporal:
  - Si el Pase de acceso temporal existente es válido, el administrador debe eliminar el Pase de acceso temporal existente y crear un nuevo pase para el usuario. Si se elimina un Pase de acceso temporal válido, se revocarán las sesiones del usuario. 
  - Si el Pase de acceso temporal existente ha expirado, uno nuevo invalidará el Pase de acceso temporal existente y no revocará las sesiones del usuario.

Para obtener más información sobre los estándares de NIST para la incorporación y la recuperación, consulte la [publicación especial de NIST 800-63A](https://pages.nist.gov/800-63-3/sp800-63a.html#sec4).

## <a name="limitations"></a>Limitaciones

Tenga en cuenta las limitaciones siguientes:

- Cuando se usa un Pase de acceso temporal de un solo uso para registrar un método sin contraseña, como FIDO2 o el inicio de sesión telefónico, el usuario debe completar el registro en un plazo de 10 minutos a partir del inicio de sesión con el Pase de acceso temporal de un solo uso. Esta limitación no se aplica a un Pase de acceso temporal que se pueda usar más de una vez.
- Tenga en cuenta que los usuarios invitados no pueden iniciar sesión con un Pase de acceso temporal.
- Pase de acceso temporal está en versión preliminar pública y actualmente no está disponible en Azure para la Administración Pública de Estados Unidos.
- Los usuarios en el ámbito de la directiva de registro de autoservicio de restablecimiento de contraseña (SSPR) *o* la [directiva de registro de autenticación multifactor de protección de identidades](../identity-protection/howto-identity-protection-configure-mfa-policy.md) tienen que registrar los métodos de autenticación una vez que han iniciado sesión con un Pase de acceso temporal. Los usuarios en el ámbito de estas directivas se redirigen al [modo de interrupción del registro combinado](concept-registration-mfa-sspr-combined.md#combined-registration-modes). Esta experiencia no admite actualmente el registro con FIDO2 e inicio de sesión telefónico. 
- Un Pase de acceso temporal no se puede usar con la extensión del servidor de directivas de redes (NPS) ni el adaptador de los Servicios de federación de Active Directory (AD FS), ni durante el programa de instalación de Windows/experiencia de serie (OOBE) y AutoPilot. 
- Cuando el inicio de sesión único de conexión directa está habilitado en el inquilino, se pide a los usuarios que escriban una contraseña. El vínculo **Use el Pase de acceso temporal en su lugar** estará disponible para que el usuario inicie sesión con un Pase de acceso temporal.

  ![Captura de pantalla de la opción Use un Pase de acceso temporal en su lugar.](./media/how-to-authentication-temporary-access-pass/alternative.png)

## <a name="troubleshooting"></a>Solución de problemas    

- Si no se ofrece un Pase de acceso temporal a un usuario durante el inicio de sesión, compruebe lo siguiente:
  - El usuario está en el ámbito de la directiva del método de autenticación del Pase de acceso temporal.
  - El usuario tiene un Pase de acceso temporal válido y, si es de un solo uso, todavía no se ha usado.
- Si aparece la opción **Temporary Access Pass sign in was blocked due to User Credential Policy** (El inicio de sesión con el Pase de acceso temporal se bloqueó debido a la directiva de credenciales de usuario) durante el inicio de sesión con un Pase de acceso temporal, compruebe lo siguiente:
  - El usuario tiene un Pase de acceso temporal de varios usos, mientras que la directiva del método de autenticación requiere un Pase de acceso temporal de un solo uso.
  - Ya se ha usado un Pase de acceso temporal de un solo uso.

## <a name="next-steps"></a>Pasos siguientes

- [Planeamiento de una implementación de autenticación sin contraseña en Azure Active Directory](howto-authentication-passwordless-deployment.md)

