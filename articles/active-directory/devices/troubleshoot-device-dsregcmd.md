---
title: 'Solución de problemas con el comando dsregcmd: Azure Active Directory'
description: Uso de la salida de dsregcmd para conocer el estado de los dispositivos en Azure AD
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 11/21/2019
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: spunukol
ms.collection: M365-identity-device-management
ms.openlocfilehash: ea502deee0caf5418bf5554473180eb405792567
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287057"
---
# <a name="troubleshooting-devices-using-the-dsregcmd-command"></a>Solución de problemas de dispositivos con el comando dsregcmd

La utilidad dsregcmd /status debe ejecutarse como una cuenta de usuario de dominio.

## <a name="device-state"></a>Estado del dispositivo

En esta sección se enumeran los parámetros de estado de la combinación de dispositivos. En la tabla siguiente se enumeran los criterios para que el dispositivo esté en varios estados de unión.

| AzureAdJoined | EnterpriseJoined | DomainJoined | Estado del dispositivo |
| ---   | ---   | ---   | ---   |
| SÍ | No | No | Unido a Azure AD |
| No | No | SÍ | Unido a dominio |
| SÍ | No | SÍ | Unido a AD híbrido |
| No | SÍ | SÍ | Unido a DRS en el entorno local |

> [!NOTE]
> El estado de Workplace Join (registrado en Azure AD) se muestra en la sección "Estado del usuario".

- **AzureAdJoined:** Establézcalo en "YES" (Sí) si el dispositivo está unido a Azure AD. De lo contrario, establézcalo en "NO".
- **EnterpriseJoined:** Establézcalo en "YES" (Sí) si el dispositivo está unido a un DRS en el entorno local. Un dispositivo no puede ser EnterpriseJoined y AzureAdJoined.
- **DomainJoined:** Establézcalo en "YES" (Sí) si el dispositivo está unido a un dominio (AD).
- **DomainName:** Establézcalo en el nombre del dominio si el dispositivo está unido a un dominio.

### <a name="sample-device-state-output"></a>Salida del estado del dispositivo de muestra

```
+----------------------------------------------------------------------+
| Device State                                                         |
+----------------------------------------------------------------------+
             AzureAdJoined : YES
          EnterpriseJoined : NO
              DomainJoined : YES
                DomainName : HYBRIDADFS
+----------------------------------------------------------------------+
```

## <a name="device-details"></a>Detalles del dispositivo

Solo se muestra cuando el dispositivo está unido a Azure AD o a Azure AD híbrido (no registrado en Azure AD). En esta sección se enumeran los detalles de identificación de los dispositivos almacenados en la nube.

- **DeviceId:** Identificador único del dispositivo en el inquilino de Azure AD.
- **Thumbprint:** Huella digital del certificado de dispositivo.
- **DeviceCertificateValidity:** Validez del certificado de dispositivo.
- **KeyContainerId:** ContainerId de la clave privada del dispositivo asociada con el certificado de dispositivo.
- **KeyProvider:** Proveedor de claves (hardware/software) usado para almacenar la clave privada del dispositivo.
- **TpmProtected:** "YES" (Sí) si la clave privada del dispositivo se almacena en un TPM de hardware.

### <a name="sample-device-details-output"></a>Salida de los detalles del dispositivo de muestra

```
+----------------------------------------------------------------------+
| Device Details                                                       |
+----------------------------------------------------------------------+

                  DeviceId : e92325d0-xxxx-xxxx-xxxx-94ae875dxxxx
                Thumbprint : D293213EF327483560EED8410CAE36BB67208179
 DeviceCertificateValidity : [ 2019-01-11 21:02:50.000 UTC -- 2029-01-11 21:32:50.000 UTC ]
            KeyContainerId : 13e68a58-xxxx-xxxx-xxxx-a20a2411xxxx
               KeyProvider : Microsoft Software Key Storage Provider
              TpmProtected : NO
+----------------------------------------------------------------------+
```

## <a name="tenant-details"></a>Detalles de inquilino

Solo se muestra cuando el dispositivo está unido a Azure AD o a Azure AD híbrido (no registrado en Azure AD). En esta sección se enumeran los detalles comunes del inquilino cuando un dispositivo está unido a Azure AD.

> [!NOTE]
> Si las direcciones URL de MDM de esta sección están vacías, indica que la MDM no se configuró o que el usuario actual no está en el ámbito de la inscripción de MDM. Compruebe la configuración de movilidad en Azure AD para revisar la configuración de MDM.

> [!NOTE]
> Aunque vea las direcciones URL MDM, esto no significa que el dispositivo esté administrado por MDM. La información se muestra si el inquilino tiene la configuración de MDM para la inscripción automática incluso si el dispositivo es no administrado.

### <a name="sample-tenant-details-output"></a>Salida de los detalles del inquilino de muestra

```
+----------------------------------------------------------------------+
| Tenant Details                                                       |
+----------------------------------------------------------------------+

                TenantName : HybridADFS
                  TenantId : 96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx
                       Idp : login.windows.net
               AuthCodeUrl : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/oauth2/authorize
            AccessTokenUrl : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/oauth2/token
                    MdmUrl : https://enrollment.manage-beta.microsoft.com/EnrollmentServer/Discovery.svc
                 MdmTouUrl : https://portal.manage-beta.microsoft.com/TermsOfUse.aspx
          MdmComplianceUrl : https://portal.manage-beta.microsoft.com/?portalAction=Compliance
               SettingsUrl : eyJVxxxxIjpbImh0dHBzOi8va2FpbGFuaS5vbmUubWljcm9zb2Z0LmNvbS8iLCJodHRwczovL2thaWxhbmkxLm9uZS5taWNyb3NvZnQuY29tLyxxxx==
            JoinSrvVersion : 1.0
                JoinSrvUrl : https://enterpriseregistration.windows.net/EnrollmentServer/device/
                 JoinSrvId : urn:ms-drs:enterpriseregistration.windows.net
             KeySrvVersion : 1.0
                 KeySrvUrl : https://enterpriseregistration.windows.net/EnrollmentServer/key/
                  KeySrvId : urn:ms-drs:enterpriseregistration.windows.net
        WebAuthNSrvVersion : 1.0
            WebAuthNSrvUrl : https://enterpriseregistration.windows.net/webauthn/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/
             WebAuthNSrvId : urn:ms-drs:enterpriseregistration.windows.net
    DeviceManagementSrvVer : 1.0
    DeviceManagementSrvUrl : https://enterpriseregistration.windows.net/manage/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx/
     DeviceManagementSrvId : urn:ms-drs:enterpriseregistration.windows.net
+----------------------------------------------------------------------+
```

## <a name="user-state"></a>Estado de usuario

En esta sección se muestra el estado de varios atributos para el usuario que ha iniciado sesión actualmente en el dispositivo.

> [!NOTE]
> El comando debe ejecutarse en un contexto de usuario para recuperar el estado válido.

- **NgcSet:** Establézcalo en "YES" (Sí) si se establece una clave de Windows Hello para el usuario que ha iniciado sesión.
- **NgcKeyId:** Identificador de la clave de Windows Hello si se ha establecida una para el usuario que ha iniciado sesión.
- **CanReset:** Indica si el usuario puede restablecer la clave de Windows Hello.
- **Valores posibles:** DestructiveOnly, NonDestructiveOnly, DestructiveAndNonDestructive o Unknown, si se ha producido un error.
- **WorkplaceJoined:** Establézcalo en "YES" (Sí) si se han agregado cuentas registradas de Azure AD al dispositivo en el contexto de NTUSER actual.
- **WamDefaultSet:** Establézcalo en "YES" (Sí) si se creó una WebAccount predeterminada de WAM para el usuario que ha iniciado sesión. Este campo podría mostrar un error si se ejecuta dsreg /status desde un símbolo del sistema con privilegios elevados.
- **WamDefaultAuthority:** Establézcalo en "organizations" (organizaciones) para Azure AD.
- **WamDefaultId:** Siempre "https://login.microsoft.com" para Azure AD.
- **WamDefaultGUID:** GUID del proveedor de WAM (cuenta de Azure AD/Microsoft) para la cuenta web predeterminada de WAM.

### <a name="sample-user-state-output"></a>Salida del estado del usuario de muestra

```
+----------------------------------------------------------------------+
| User State                                                           |
+----------------------------------------------------------------------+

                    NgcSet : YES
                  NgcKeyId : {FA0DB076-A5D7-4844-82D8-50A2FB42EC7B}
                  CanReset : DestructiveAndNonDestructive
           WorkplaceJoined : NO
             WamDefaultSet : YES
       WamDefaultAuthority : organizations
              WamDefaultId : https://login.microsoft.com
            WamDefaultGUID : { B16898C6-A148-4967-9171-64D755DA8520 } (AzureAd)

+----------------------------------------------------------------------+
```

## <a name="sso-state"></a>Estado de SSO

Esta sección se puede pasar por alto para los dispositivos registrados de Azure AD.

> [!NOTE]
> El comando debe ejecutarse en un contexto de usuario para recuperar el estado válido para dicho usuario.

- **AzureAdPrt:** Establézcalo en "YES" (Sí) si hay un PRT en el dispositivo para el usuario que ha iniciado sesión.
- **AzureAdPrtUpdateTime:** Establézcalo en la hora en formato UTC cuando se actualizó por última vez el PRT.
- **AzureAdPrtExpiryTime:** Establézcalo en la hora en formato UTC en que el PRT va a expirar si no se renueva.
- **AzureAdPrtAuthority:** Dirección URL de la autoridad de Azure AD.
- **EnterprisePrt:** Establézcalo en "YES" (Sí) si el dispositivo tiene PRT desde ADFS en el entorno local. En el caso de los dispositivos unidos a Azure AD híbrido, el dispositivo podría tener PRT tanto de Azure AD como de Azure AD en el entorno local simultáneamente. Los dispositivos unidos a un entorno local solo tendrán un PRT empresarial.
- **EnterprisePrtUpdateTime:** Establézcalo en la hora en formato UTC cuando se actualizó por última vez el PRT empresarial.
- **EnterprisePrtExpiryTime:** Establézcalo en la hora en formato UTC en que el PRT va a expirar si no se renueva.
- **EnterprisePrtAuthority:** Dirección URL de la autoridad ADFS

### <a name="sample-sso-state-output"></a>Salida del estado SSO de muestra

```
+----------------------------------------------------------------------+
| SSO State                                                            |
+----------------------------------------------------------------------+

                AzureAdPrt : YES
      AzureAdPrtUpdateTime : 2019-01-24 19:15:26.000 UTC
      AzureAdPrtExpiryTime : 2019-02-07 19:15:26.000 UTC
       AzureAdPrtAuthority : https://login.microsoftonline.com/96fa76d0-xxxx-xxxx-xxxx-eb60cc22xxxx
             EnterprisePrt : YES
   EnterprisePrtUpdateTime : 2019-01-24 19:15:33.000 UTC
   EnterprisePrtExpiryTime : 2019-02-07 19:15:33.000 UTC
    EnterprisePrtAuthority : https://fs.hybridadfs.nttest.microsoft.com:443/adfs

+----------------------------------------------------------------------+
```

## <a name="diagnostic-data"></a>Datos de diagnóstico

### <a name="pre-join-diagnostics"></a>Diagnóstico previo a la unión

Esta sección solo se muestra si el dispositivo está unido a un dominio y no puede realizar una unión a Azure AD híbrido.

En esta sección se realizan una serie de pruebas para ayudar a diagnosticar errores de la unión. En esta sección también se incluyen los detalles de la anterior (?). Esta información incluye la fase de error, el código de error, el identificador de la solicitud de servidor, el estado HTTP de respuesta del servidor y el mensaje de error de la respuesta del servidor.

- **User Context:** Contexto en el que se ejecutan los diagnósticos. Valores posibles: SYSTEM, usuario UN-ELEVATED (sin privilegios), usuario ELEVATED (con privilegios).

   > [!NOTE]
   > Dado que la unión real se realiza en el contexto de SYSTEM, la ejecución del diagnóstico en el contexto de SYSTEM está más cercana al escenario de unión real. Para ejecutar diagnósticos en el contexto de SYSTEM, el comando dsregcmd /status se debe ejecutar desde un símbolo del sistema con privilegios elevados.

- **Client Time:** Hora del sistema en UTC.
- **AD Connectivity Test:** Se realiza una prueba de conectividad con el controlador de dominio. Si se produce un error en esta prueba, es probable que se produzcan errores de unión en la fase previa a la comprobación.
- **AD Configuration Test:** Se lee y se comprueba si el objeto SCP está configurado correctamente en el bosque de AD en el entorno local. Los errores de esta prueba podrían producir errores de unión en la fase de detección con el código 0x801c001d.
- **DS Discovery Test:** La prueba obtiene los puntos de conexión de DRS del punto de conexión de los metadatos de detección y realiza una solicitud del dominio Kerberos del usuario. Los errores de esta prueba podrían producir errores de unión en la fase de detección.
- **DRS Connectivity Test:** Se realiza una prueba de conectividad básica del punto de conexión DRS.
- **Token acquisition Test:** La prueba intenta obtener un token de autenticación de Azure AD si el inquilino del usuario está federado. Los errores de esta prueba podrían producir errores de unión en la fase de autorización. Si la autorización no puede realizarse, se intentará la unión de sincronización como reserva, a menos que la reserva se deshabilite explícitamente con la configuración siguiente de clave del Registro.

```
Keyname: Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\CDJ
Value: FallbackToSyncJoin
Type:  REG_DWORD
Value: 0x0 -> Disabled
Value: 0x1 -> Enabled
Default (No Key): Enabled
```

- **Fallback to Sync-Join:** Establézcalo en "Enabled" (Habilitado) si la siguiente clave del Registro, a fin de evitar que la reserva realice la unión de sincronización con errores de autenticación, NO está presente. Esta opción está disponible en Windows 10 1803 y versiones posteriores.
- **Previous Registration:** Hora en que se produjo el intento de unión anterior. Solo se registran los intentos de unión erróneos.
- **Error Phase:** Fase de la unión en la que se anuló. Los valores posibles son pre-check (comprobación previa), discover (detección), auth (autorización), join (unión).
- **Client ErrorCode:** Se devolvió el código de error del cliente (HRESULT).
- **Server ErrorCode:** Código de error del servidor si se envió una solicitud al servidor y este respondió de nuevo con un código de error.
- **Server Message:** Mensaje del servidor devuelto junto con el código de error.
- **HTTPS Status:** Estado HTTP devuelto por el servidor.
- **Request ID:** Identificador de solicitud del cliente enviado al servidor. Resulta de utilidad para poner en correlación con los registros del lado servidor.

### <a name="sample-pre-join-diagnostics-output"></a>Muestra de la salida de diagnóstico previo a la unión

En el ejemplo siguiente se muestra un error de detección en la prueba de diagnóstico.

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

     Diagnostics Reference : www.microsoft.com/aadjerrors
              User Context : SYSTEM
               Client Time : 2019-01-31 09:25:31.000 UTC
      AD Connectivity Test : PASS
     AD Configuration Test : PASS
        DRS Discovery Test : FAIL [0x801c0021/0x801c000c]
     DRS Connectivity Test : SKIPPED
    Token acquisition Test : SKIPPED
     Fallback to Sync-Join : ENABLED

     Previous Registration : 2019-01-31 09:23:30.000 UTC
               Error Phase : discover
          Client ErrorCode : 0x801c0021

+----------------------------------------------------------------------+
```

En el ejemplo siguiente se muestran que las pruebas de diagnóstico se están superando, pero el intento de registro produjo un error de directorio, lo que se espera para la unión de sincronización. Una vez que se complete el trabajo de sincronización de Azure AD Connect, el dispositivo podrá combinarse.

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

     Diagnostics Reference : www.microsoft.com/aadjerrors
              User Context : SYSTEM
               Client Time : 2019-01-31 09:16:50.000 UTC
      AD Connectivity Test : PASS
     AD Configuration Test : PASS
        DRS Discovery Test : PASS
     DRS Connectivity Test : PASS
    Token acquisition Test : PASS
     Fallback to Sync-Join : ENABLED

     Previous Registration : 2019-01-31 09:16:43.000 UTC
         Registration Type : sync
               Error Phase : join
          Client ErrorCode : 0x801c03f2
          Server ErrorCode : DirectoryError
            Server Message : The device object by the given id (e92325d0-7ac4-4714-88a1-94ae875d5245) is not found.
              Https Status : 400
                Request Id : 6bff0bd9-820b-484b-ab20-2a4f7b76c58e

+----------------------------------------------------------------------+
```

### <a name="post-join-diagnostics"></a>Diagnóstico posterior a la unión

En esta sección se muestra el resultado de las comprobaciones de validez realizadas en un dispositivo unido a la nube.

- **AadRecoveryEnabled:** Si es "YES" (Sí), las claves almacenadas en el dispositivo no se pueden usar y el dispositivo está marcado para recuperación. El siguiente inicio de sesión desencadenará el flujo de recuperación y se volverá a registrar el dispositivo.
- **KeySignTest:** Si es "PASSED" (Superado) las claves del dispositivo están en buen estado. Si se produce un error en KeySignTest, normalmente el dispositivo se marcará para la recuperación. El siguiente inicio de sesión desencadenará el flujo de recuperación y se volverá a registrar el dispositivo. En el caso de los dispositivos unidos a Azure AD híbrido, la recuperación es silenciosa. Aunque se unan a Azure AD o se registren en Azure AD, los dispositivos solicitarán la autenticación del usuario para recuperar y volver a registrar el dispositivo, si es necesario. **KeySignTest requiere privilegios elevados.**

#### <a name="sample-post-join-diagnostics-output"></a>Muestra de la salida de diagnóstico posterior a la unión

```
+----------------------------------------------------------------------+
| Diagnostic Data                                                      |
+----------------------------------------------------------------------+

         AadRecoveryEnabled: NO
               KeySignTest : PASSED
+----------------------------------------------------------------------+
```

## <a name="ngc-prerequisite-check"></a>Comprobación de los requisitos previos de NGC

En esta sección se realizan las comprobaciones de los requisitos previos para el aprovisionamiento de Windows Hello para empresas (WHFB).

> [!NOTE]
> Es posible que no vea los detalles de la comprobación de requisitos previos de NGC en dsregcmd /status si el usuario ya ha configurado correctamente WHFB.

- **IsDeviceJoined:** Establézcalo en "YES" (Sí) si el dispositivo está unido a Azure AD.
- **IsUserAzureAD:** Establézcalo en "YES" (Sí) si el usuario que ha iniciado sesión está presente en Azure AD.
- **PolicyEnabled:** Establézcalo en "YES" (Sí) si la directiva de WHFB está habilitada en el dispositivo.
- **PostLogonEnabled:** Establézcalo en "YES" (Sí) si la plataforma desencadena la inscripción de WHFB de forma nativa. Si se establece en "NO", indica que la inscripción de Windows Hello para empresas se desencadena mediante un mecanismo personalizado.
- **DeviceEligible:** Establézcalo en "YES" (Sí) si el dispositivo cumple los requisitos de hardware para inscribirse en WHFB.
- **SessionIsNotRemote:** Establézcalo en "YES" (Sí) si el usuario actual ha iniciado sesión directamente en el dispositivo y no de forma remota.
- **CertEnrollment**: Elemento específico de la implementación de certificados de confianza de WHFB que indica la entidad de inscripción de certificados de WHFB. Establézcalo en "enrollment authority" (entidad de inscripción) si el origen de la directiva de WHFB es Directiva de grupo o en "mobile device management" (administración de dispositivos móviles) si el origen es MDM. De lo contrario, establézcalo en "none" (ninguno).
- **AdfsRefreshToken:** Elemento específico de la implementación de certificados de confianza de WHFB. Solo está presente si CertEnrollment es "enrollment authority" (entidad de inscripción). Indica si el dispositivo tiene un PRT empresarial para el usuario.
- **AdfsRaIsReady:** Elemento específico de la implementación de certificados de confianza de WHFB.  Solo está presente si CertEnrollment es "enrollment authority" (entidad de inscripción). Establézcalo en "YES" (Sí) si ADFS indica en los metadatos de detección que admite WHFB *y* si la plantilla de certificado de inicio de sesión está disponible.
- **LogonCertTemplateReady:** Elemento específico de la implementación de certificados de confianza de WHFB. Solo está presente si CertEnrollment es "enrollment authority" (entidad de inscripción). Establézcalo en "YES" (Sí) si el estado de la plantilla de certificado de inicio de sesión es válido y ayuda a solucionar los problemas de ADFS RA.
- **PreReqResult:** Proporciona el resultado de toda la evaluación de requisitos previos de WHFB. Establézcalo en "Will Provision" (Aprovisionará) si la inscripción de WHFB se iniciará como una tarea posterior al inicio de sesión cuando el usuario inicie sesión la próxima vez.

### <a name="sample-ngc-prerequisite-check-output"></a>Resultado de la comprobación de los requisitos previos de NGC de muestra

```
+----------------------------------------------------------------------+
| Ngc Prerequisite Check                                               |
+----------------------------------------------------------------------+

            IsDeviceJoined : YES
             IsUserAzureAD : YES
             PolicyEnabled : YES
          PostLogonEnabled : YES
            DeviceEligible : YES
        SessionIsNotRemote : YES
            CertEnrollment : enrollment authority
          AdfsRefreshToken : YES
             AdfsRaIsReady : YES
    LogonCertTemplateReady : YES ( StateReady )
              PreReqResult : WillProvision
+----------------------------------------------------------------------+
```

## <a name="next-steps"></a>Pasos siguientes

- [Herramienta de búsqueda de errores de Microsoft](/windows/win32/debug/system-error-code-lookup-tool)
