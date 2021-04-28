---
title: Rotación de emergencia de los certificados de AD FS | Microsoft Docs
description: En este artículo se explica cómo revocar y actualizar los certificados de AD FS inmediatamente.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 03/22/2021
ms.subservice: hybrid
ms.author: billmath
ms.openlocfilehash: 9741c2e85a7cd3523ffe7fe8262e5f5d821b62c3
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108126604"
---
# <a name="emergency-rotation-of-the-ad-fs-certificates"></a>Rotación de emergencia de los certificados de AD FS
En caso de que necesite rotar los certificados de AD FS inmediatamente, puede seguir los pasos que se describen a continuación en esta sección.

> [!IMPORTANT]
> Al realizar los pasos siguientes en el entorno de AD FS, se revocarán inmediatamente los certificados antiguos.  Como esto se hace inmediatamente, se evita el tiempo normal que suele concederse a sus asociados de la federación para usar su nuevo certificado. Podría producirse una interrupción del servicio mientras que las confianzas se actualizan para usar los nuevos certificados.  Esto debería resolverse una vez que todos los asociados de la federación tengan los nuevos certificados.

> [!NOTE]
> Microsoft recomienda encarecidamente usar un módulo de seguridad de hardware (HSM) para proteger los certificados.
> Para obtener más información, vea [Módulo de seguridad de hardware](/windows-server/identity/ad-fs/deployment/best-practices-securing-ad-fs#hardware-security-module-hsm) en los procedimientos recomendados para proteger AD FS.

## <a name="determine-your-token-signing-certificate-thumbprint"></a>Determinación de la huella digital del certificado de firma de tokens
Para revocar el antiguo certificado de firma de tokens que AD FS está usando actualmente, debe determinar la huella digital del certificado de firma de tokens.  Para ello, siga estos pasos:

 1. Conéctese al servicio `PS C:\>Connect-MsolService` de Microsoft Online Services.
 2. Documente la huella digital y las fechas de expiración de su certificado de firma de tokens tanto local como en la nube.
`PS C:\>Get-MsolFederationProperty -DomainName <domain>` 
 3.  Copie el valor de la huella digital.  Se usará más adelante para quitar los certificados existentes.

También puede obtener la huella digital mediante el uso de la Administración de AD FS; para ello, vaya a Servicio/Certificados, haga clic con el botón derecho en el certificado, seleccione Ver certificado y, a continuación, seleccione Detalles. 

## <a name="determine-whether-ad-fs-renews-the-certificates-automatically"></a>Determinación de si AD FS renueva automáticamente los certificados
De forma predeterminada, AD FS está configurado para generar certificados de firma y descifrado de tokens automáticamente, tanto en el momento de la configuración inicial como cuando los certificados se aproximan a su fecha de expiración.

Puede ejecutar el siguiente comando de Windows PowerShell: `PS C:\>Get-AdfsProperties | FL AutoCert*, Certificate*`.

La propiedad AutoCertificateRollover describe si AD FS está configurado para renovar automáticamente los certificados de firma y descifrado de tokens.  Si AutoCertificateRollover está establecido en TRUE, siga las instrucciones que se describen a continuación en [Generación de un nuevo certificado autofirmado si AutoCertificateRollover está establecido en TRUE].  Si AutoCertificateRollover está establecido en FALSE, siga las instrucciones que se describen a continuación en [Generación de nuevos certificados manualmente si AutoCertificateRollover está establecido en FALSE].


## <a name="generating-new-self-signed-certificate-if-autocertificaterollover-is-set-to-true"></a>Generación de un nuevo certificado autofirmado si AutoCertificateRollover está establecido en TRUE
En esta sección, creará **dos** certificados de firma de tokens.  El primero usará la marca **-urgent**, que reemplazará inmediatamente al certificado principal actual.  El segundo se usará para el certificado secundario.  

>[!IMPORTANT]
>La razón por la que estamos creando dos certificados es porque Azure conserva la información relativa al certificado anterior.  Al crear un segundo certificado, estamos forzando a Azure a liberar la información sobre el antiguo y sustituirla por la del segundo.
>
>Si no se crea el segundo certificado y actualiza Azure con él, es posible que el antiguo certificado de firma de tokens autentique a los usuarios.

Puede usar los pasos siguientes para generar los nuevos certificados de firma de tokens.

 1. Compruebe que la sesión en el servidor de AD FS principal está iniciada.
 2. Abra Windows PowerShell como administrador. 
 3. Asegúrese de que el valor de AutoCertificateRollover está establecido en True.
`PS C:\>Get-AdfsProperties | FL AutoCert*, Certificate*`
 4. Para generar un nuevo certificado de firma de tokens: `Update-ADFSCertificate –CertificateType token-signing -Urgent`.
 5. Compruebe la actualización mediante la ejecución del comando siguiente: `Get-ADFSCertificate –CertificateType token-signing`.
 6. Ahora, genere el segundo certificado de firma de tokens: `Update-ADFSCertificate –CertificateType token-signing`.
 7. Puede verificar la actualización mediante la ejecución de nuevo del comando siguiente: `Get-ADFSCertificate –CertificateType token-signing`.


## <a name="generating-new-certificates-manually-if-autocertificaterollover-is-set-to-false"></a>Generación de nuevos certificados manualmente si AutoCertificateRollover está establecido en FALSE
Si no usa los certificados de firma y descifrado de tokens autofirmados y generados automáticamente, debe renovar y configurar estos certificados manualmente.  Esto implica crear dos nuevos certificados de firma de tokens e importarlos.  A continuación, se promueve uno a principal, se revoca el certificado antiguo y se configura el segundo certificado como secundario.

En primer lugar, debe obtener dos nuevos certificados de la entidad de certificación e importarlos en el almacén de certificados personales del equipo local en cada servidor de la federación. Para obtener instrucciones, consulte el artículo [Importación de un certificado](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754489(v=ws.11)).

>[!IMPORTANT]
>La razón por la que estamos creando dos certificados es porque Azure conserva la información relativa al certificado anterior.  Al crear un segundo certificado, estamos forzando a Azure a liberar la información sobre el antiguo y sustituirla por la del segundo.
>
>Si no se crea el segundo certificado y actualiza Azure con él, es posible que el antiguo certificado de firma de tokens autentique a los usuarios.

### <a name="to-configure-a-new-certificate-as-a-secondary-certificate"></a>Para configurar un nuevo certificado como certificado secundario
A continuación, debe configurar un certificado como el certificado de firma o descifrado de tokens de AD FS secundario y, a continuación, promoverlo a principal.

1. Una vez que hayas importado el certificado. Abra la consola **Administración de AD FS**.
2. Expande **Servicios** y selecciona **Certificados**.
3. En el panel Acciones, haga clic en **Add Token-Signing Certificate** (Agregar certificado de firma de tokens).
4. Selecciona el nuevo certificado en la lista de certificados que se muestran y haz clic en Aceptar.

### <a name="to-promote-the-new-certificate-from-secondary-to-primary"></a>Para promover el nuevo certificado de secundario a principal
Ahora que el nuevo certificado se ha importado y configurado en AD FS, debe establecer como certificado principal.
1. Abra la consola **Administración de AD FS**.
2. Expande **Servicios** y selecciona **Certificados**.
3. Haz clic en el certificado de firma de tokens secundario.
4. En el panel **Acciones**, haz clic en **Establecer como principal**. Haz clic en Sí en el aviso de confirmación.
5. Una vez que ha promocionado el nuevo certificado como certificado principal, debe quitar el certificado antiguo porque todavía se puede usar. Vea la sección [Retirada de los certificados antiguos](#remove-your-old-certificates) más adelante.  

### <a name="to-configure-the-second-certificate-as-a-secondary-certificate"></a>Para configurar el segundo certificado como certificado secundario
Ahora que ha agregado el primer certificado, lo ha convertido en principal y ha eliminado el anterior, importe el segundo certificado.  A continuación, debe configurar el certificado como certificado de firma de tokens de AD FS secundario.

1. Una vez que hayas importado el certificado. Abra la consola **Administración de AD FS**.
2. Expande **Servicios** y selecciona **Certificados**.
3. En el panel Acciones, haga clic en **Add Token-Signing Certificate** (Agregar certificado de firma de tokens).
4. Selecciona el nuevo certificado en la lista de certificados que se muestran y haz clic en Aceptar.

## <a name="update-azure-ad-with-the-new-token-signing-certificate"></a>Actualización de Azure AD con el nuevo certificado de firma de tokens
Abra el módulo Microsoft Azure Active Directory para Windows PowerShell. También puede abrir Windows PowerShell y ejecutar el comando `Import-Module msonline`.

Conéctese a Azure AD ejecutando el comando `Connect-MsolService` y, luego, escriba sus credenciales de administrador global.

>[!Note]
> Si ejecuta estos comandos en un equipo que no es el servidor de federación principal, ejecute primero el comando siguiente: `Set-MsolADFSContext –Computer <servername>`. Sustituya <servername> por el nombre del servidor de AD FS. A continuación, escriba las credenciales de administrador del servidor de AD FS cuando se le pida.

Opcionalmente, verifique si es necesaria una actualización comprobando la información actual del certificado en Azure AD. Para ello, ejecute el siguiente comando: `Get-MsolFederationProperty`. Escriba el nombre del dominio federado cuando se le pida.

Para actualizar la información del certificado en Azure AD, ejecute el comando `Update-MsolFederatedDomain` y, a continuación, escriba el nombre de dominio cuando se le solicite.

>[!Note]
> Si ve un error al ejecutar este comando, ejecute el comando `Update-MsolFederatedDomain –SupportMultipleDomain` y, a continuación, escriba el nombre de dominio cuando se le solicite.

## <a name="replace-ssl-certificates"></a>Reemplazo de certificados SSL
En caso de que necesite reemplazar el certificado de firma de tokens debido a un riesgo, también debe revocar y reemplazar los certificados SSL para AD FS y los servidores WAP.  

La revocación de los certificados SSL debe realizarse en la entidad de certificación (CA) que emitió el certificado.  Estos certificados suelen emitirlos proveedores de terceros como GoDaddy.  Para obtener un ejemplo, consulte (Revocación de un certificado | Certificados SSL - GoDaddy Help US).  Para obtener más información, vea [Funcionamiento de la revocación de certificados](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee619754(v=ws.10)).

Una vez que se ha revocado el certificado SSL anterior y se ha emitido uno nuevo, puede reemplazar los certificados SSL. Para obtener más información, vea [Reemplazar el certificado SSL para AD FS](/windows-server/identity/ad-fs/operations/manage-ssl-certificates-ad-fs-wap#replacing-the-ssl-certificate-for-ad-fs).


## <a name="remove-your-old-certificates"></a>Retirada de los certificados antiguos
Una vez que haya reemplazado los certificados antiguos, debe quitarlos porque todavía se pueden usar. . Para ello, siga estos pasos:  Para ello, siga estos pasos:

1. Compruebe que la sesión en el servidor de AD FS principal está iniciada.
2. Abra Windows PowerShell como administrador. 
4. Para quitar el certificado de firma de tokens antiguo: `Remove-ADFSCertificate –CertificateType token-signing -thumbprint <thumbprint>`.

## <a name="updating-federation-partners-who-can-consume-federation-metadata"></a>Actualización de los asociados de la federación que pueden usar metadatos de la federación
Si ha renovado y configurado un nuevo certificado de firma o descifrado de tokens, debe asegurarse de que todos los asociados de la federación (asociados de la organización de recursos o la organización de cuentas que están representados en AD FS por las confianzas de un usuario de confianza y las confianzas de un proveedor de notificaciones) han seleccionado los nuevos certificados.

## <a name="updating-federation-partners-who-can-not-consume-federation-metadata"></a>Actualización de los asociados de la federación que NO pueden usar metadatos de la federación
Si los asociados de la federación no pueden usar los metadatos de la federación, debe enviarles manualmente la clave pública del nuevo certificado de firma o descifrado de tokens. Envíe la nueva clave pública de certificado (archivo. cer o. p7b si desea incluir toda la cadena) a todos los asociados de la organización de recursos o la organización de cuentas (representados en AD FS por las confianzas de un usuario de confianza y las confianzas de un proveedor de notificaciones). Haga que los asociados implementen los cambios en su lado para confiar en los nuevos certificados.



## <a name="revoke-refresh-tokens-via-powershell"></a>Revocación de tokens de actualización a través de PowerShell
Ahora queremos revocar los tokens de actualización para los usuarios que puedan tenerlos y obligarles a volver a iniciar sesión y obtener nuevos tokens.  Esto hará que los usuarios cierren la sesión en su teléfono y en las sesiones actuales de correo web, junto con otros elementos que usen tokens y tokens de actualización.  Puede encontrar información [aquí](/powershell/module/azuread/revoke-azureaduserallrefreshtoken?preserve-view=true&view=azureadps-2.0), además de en el artículo [Revocación del acceso de usuario en Azure Active Directory](../../active-directory/enterprise-users/users-revoke-access.md).

## <a name="next-steps"></a>Pasos siguientes

- [Administración de certificados SSL en AD FS y WAP en Windows Server 2016](/windows-server/identity/ad-fs/operations/manage-ssl-certificates-ad-fs-wap#replacing-the-ssl-certificate-for-ad-fs)
- [Obtención y configuración de certificados de descifrado y firma de tokens para AD FS](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn781426(v=ws.11)#updating-federation-partners)
- [Renovación de certificados de federación para Microsoft 365 y Azure Active Directory](how-to-connect-fed-o365-certs.md)