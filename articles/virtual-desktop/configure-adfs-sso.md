---
title: 'Configuración del inicio de sesión único de AD FS de Azure Virtual Desktop: Azure'
description: Configuración del inicio de sesión único de AD FS en un entorno de Azure Virtual Desktop.
services: virtual-desktop
author: Heidilohr
manager: lizross
ms.service: virtual-desktop
ms.topic: how-to
ms.date: 05/28/2021
ms.author: helohr
ms.openlocfilehash: c85186d8338918dbcf2af56abd959f5cbff6ad56
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111967278"
---
# <a name="configure-ad-fs-single-sign-on-for-azure-virtual-desktop"></a>Configuración del inicio de sesión único de AD FS para Azure Virtual Desktop

> [!IMPORTANT]
> El inicio de sesión único de AD FS está actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin contrato de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.
> Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

En este artículo obtendrá información del proceso de configuración del inicio de sesión único (SSO) de los Servicios de federación de Active Directory (AD FS) para Azure Virtual Desktop.

> [!NOTE]
> Tenga en cuenta que Azure Virtual Desktop (clásico) no admite esta característica.

## <a name="requirements"></a>Requisitos

> [!IMPORTANT]
> Durante la versión preliminar pública, debe configurar el grupo host para que esté en el [entorno de validación](create-validation-host-pool.md).

Antes de configurar el inicio de sesión único de AD FS, debe ejecutar la siguiente configuración en el entorno:

* Debe implementar el rol **Servicios de certificados de Active Directory (CA)** . Todos los servidores que ejecutan el rol deben estar unidos a un dominio, tener instaladas las actualizaciones más recientes de Windows y configurarse como una [entidad de certificación de empresa](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc731183%28v%3dws.10%29).
* Asimismo, debe implementar el rol **Servicios de federación de Active Directory (AD FS)** . Todos los servidores que ejecutan el rol deben estar unidos a un dominio, tener instaladas las actualizaciones más recientes de Windows y ejecutar una versión de Windows Server 2016 o posterior. Consulte nuestro [tutorial de federación](../active-directory/hybrid/tutorial-federation.md) para empezar a configurar este rol.
* Se recomienda configurar el rol de **Proxy de aplicación web** para proteger la conexión del entorno con los servidores de AD FS. Todos los servidores que ejecutan el rol deben tener instaladas las actualizaciones más recientes de Windows y ejecutar una versión de Windows Server 2016 o posterior. Consulte esta [guía de Proxy de aplicación web](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn383662(v=ws.11)) para empezar a configurar el rol.
* Debe implementar **Azure AD Connect** para sincronizar usuarios con Azure AD. Igualmente, Azure AD Connect debe configurarse en el [modo de federación](../active-directory/connect/active-directory-aadconnect-get-started-custom.md).
* [Configure el entorno de PowerShell](powershell-module.md) para Azure Virtual Desktop en el servidor de AD FS.
* Al usar Windows 10 20H1 o 20H2 para conectarse a Azure Virtual Desktop, debe instalar la **actualización acumulativa 2021-04 para Windows 10 (KB5001330)** o una versión posterior para que el inicio de sesión único funcione correctamente.

> [!NOTE]
> Esta solución no se admite con Azure AD Domain Services. Debe usar controlador de dominio de Active Directory.

## <a name="supported-clients"></a>Clientes compatibles

Los siguientes clientes de Azure Virtual Desktop admiten esta característica:

* [Cliente de escritorio de Windows](connect-windows-7-10.md)
* [Cliente web](connect-web.md)

## <a name="configure-the-certificate-authority-to-issue-certificates"></a>Configuración de la entidad de certificación para emitir certificados

Debe crear correctamente las siguientes plantillas de certificado para que AD FS pueda usar SSO:

* En primer lugar, deberá crear la plantilla de certificado del **Agente de inscripción de Exchange (solicitud sin conexión)** . AD FS usa la plantilla de certificado del Agente de inscripción de Exchange para solicitar certificados en nombre del usuario.
* También deberá crear la plantilla de certificado **Inicio de sesión de tarjeta inteligente** que AD FS usará para crear el certificado de inicio de sesión.

Después de crear estas plantillas de certificado, deberá habilitarlas en la entidad de certificación para que AD FS pueda solicitarlas.

> [!NOTE]
> Esta solución genera nuevos certificados a corto plazo para cada inicio de sesión de usuario que puede rellenar la base de datos de la entidad de certificación a lo largo del tiempo, si tiene muchos usuarios. Para evitarlo, [puede configurar una entidad de certificación para el procesamiento de certificados no persistente](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ff934598(v=ws.10)).

### <a name="create-the-enrollment-agent-certificate-template"></a>Creación de la plantilla de certificado del agente de inscripción

En función de su entorno, es posible que ya haya configurado una plantilla de certificado de agente de inscripción para otros fines, como Windows Hello para empresas, los certificados de inicio de sesión o los certificados VPN. Si es así, deberá modificarlo para admitir el inicio de sesión único. Si no es así, puede crear una plantilla.

Para determinar si ya usa una plantilla de certificado de agente de inscripción, ejecute el siguiente comando de PowerShell en el servidor AD FS y compruebe si se devuelve un valor. Si está vacío, cree una nueva plantilla de certificado de agente de inscripción. Si está vacío, cree una nueva plantilla de certificado de agente de inscripción.

```powershell
Import-Module adfs
(Get-AdfsCertificateAuthority).EnrollmentAgentCertificateTemplateName
```

Para crear una nueva plantilla de certificado de agente de inscripción:

1. En la entidad de certificación, ejecute **mmc.exe** desde el menú Inicio para iniciar **Consola de administración de Microsoft**.
2. Seleccione **Archivo...**  > **Agregar o complemento remoto...**  > **Plantillas de certificado** > **Agregar >**  > **Aceptar** para ver la lista de plantillas de certificado.
3. Expanda las **plantillas de certificado**, haga clic con el botón derecho en **Agente de inscripción de Exchange (solicitud sin conexión)** y seleccione **Duplicar plantilla**.
4. Seleccione la pestaña **General** y escriba "Agente de inscripción de ADFS" en el campo **Nombre para mostrar de plantilla**. Esto establecerá automáticamente el nombre de la plantilla en "ADFSEnrollmentAgent".
5. Seleccione la pestaña **Seguridad** y, a continuación, **Agregar...** .
6. A continuación, seleccione **Tipos de objeto...** , **Cuentas de servicio** y, a continuación, **Aceptar**.
7. Escriba el nombre de la cuenta de servicio de AD FS y seleccione **Aceptar**.
   * En una configuración AD FS aislada, la cuenta de servicio se denominará "adfssvc$".
   * Si configura AD FS con Azure AD Connect, la cuenta de servicio se denominará "aadcsvc$".
8. Una vez agregada la cuenta de servicio y cuando esté visible en la pestaña **Seguridad**, selecciónela en el panel **Nombres de grupos o usuarios**, seleccione **Permitir** tanto para "Inscribir" como para "Inscribir automáticamente" en el panel **Permisos para la cuenta de servicio de AD FS** y, a continuación, seleccione **Aceptar** para guardar.

   :::image type="content" source="media/adfs-enrollment-properties-security.png" alt-text="Captura de pantalla que muestra la pestaña de seguridad de la plantilla de certificado del agente de inscripción, una vez configurada correctamente.":::

Para actualizar una plantilla de certificado de agente de inscripción existente:

1. En la entidad de certificación, ejecute **mmc.exe** desde el menú Inicio para iniciar **Consola de administración de Microsoft**.
2. Seleccione **Archivo...**  > **Agregar o complemento remoto...**  > **Plantillas de certificado** > **Agregar >**  > **Aceptar** para ver la lista de plantillas de certificado.
3. Expanda las **plantillas de certificado**, haga doble clic en la plantilla que corresponde a la que está configurada en el servidor de AD FS. En la pestaña **General**, el nombre de la plantilla debe coincidir con el nombre que encontró anteriormente.
4. Seleccione la pestaña **Seguridad** y, a continuación, **Agregar...** .
5. A continuación, seleccione **Tipos de objeto...** , **Cuentas de servicio** y, a continuación, **Aceptar**.
6. Escriba el nombre de la cuenta de servicio de AD FS y seleccione **Aceptar**.
   * En una configuración AD FS aislada, la cuenta de servicio se denominará "adfssvc$".
   * Si configura AD FS con Azure AD Connect, la cuenta de servicio se denominará "aadcsvc$".
7. Una vez agregada la cuenta de servicio y cuando esté visible en la pestaña **Seguridad**, selecciónela en el panel **Nombres de grupos o usuarios**, seleccione **Permitir** tanto para "Inscribir" como para "Inscribir automáticamente" en el panel **Permisos para la cuenta de servicio de AD FS** y, a continuación, seleccione **Aceptar** para guardar.

### <a name="create-the-smartcard-logon-certificate-template"></a>Creación de la plantilla de certificado de inicio de sesión de tarjeta inteligente

Para crear la plantilla de certificado de inicio de sesión de tarjeta inteligente:

1. En la entidad de certificación, ejecute **mmc.exe** desde el menú Inicio para iniciar **Consola de administración de Microsoft**.
2. Seleccione **Archivo...**  > **Agregar o complemento remoto...**  > **Plantillas de certificado** > **Agregar** > **Aceptar** para ver la lista de plantillas de certificado.
3. Expanda **plantillas de certificado**, haga clic con el botón derecho en **Inicio de sesión de tarjeta inteligente** y seleccione **Duplicar plantilla**.
4. Seleccione la pestaña **General** y escriba "Inicio de sesión único de ADFS" en el campo **Nombre para mostrar de plantilla**. Esto establecerá automáticamente el nombre de la plantilla en "ADFSSSO".
   > [!NOTE]
   > Puesto que este certificado se solicita a petición, se recomienda acortar el período de validez a 8 horas y el período de renovación a 1 hora.

5. Seleccione la pestaña **Nombre del asunto** y, a continuación, seleccione **Proporcionar en la solicitud**. Cuando vea un mensaje de advertencia, seleccione **Aceptar**.

   :::image type="content" source="media/adfs-sso-properties-subject-inline.png" alt-text="Captura de pantalla en la que se muestra la pestaña de nombre del asunto de la plantilla de certificado de SSO y su aspecto cuando se configura correctamente." lightbox="media/adfs-sso-properties-subject-expanded.png":::

6. Selecciona la pestaña **Requisitos de emisión** .
7. Seleccione **Este número de firmas autorizadas** y escriba el valor de **1**.

   :::image type="content" source="media/adfs-sso-properties-issuance-inline.png" alt-text="Captura de pantalla en la que se muestra la pestaña de nombre del tema de la plantilla de certificado de SSO y su aspecto cuando se configura correctamente." lightbox="media/adfs-sso-properties-issuance-expanded.png":::

8. En **Directiva de la aplicación**, seleccione **Agente de solicitud de certificado**.
9.  Seleccione la pestaña **Seguridad** y, a continuación, **Agregar...** .
10. Seleccione **Tipos de objeto...** , **Cuentas de servicio** y **Aceptar**.
11. Escriba el nombre de la cuenta de servicio de AD FS igual que hizo en la sección [Creación de la plantilla de certificado del agente de inscripción](#create-the-enrollment-agent-certificate-template).
    * En una configuración AD FS aislada, la cuenta de servicio se denominará "adfssvc$".
    * Si configura AD FS con Azure AD Connect, la cuenta de servicio se denominará "aadcsvc$".
12. Una vez agregada la cuenta de servicio y cuando esté visible en la pestaña **Seguridad**, selecciónela en el panel **Nombres de usuario o grupo**, seleccione **Permitir** tanto para "Inscripción" como para "Inscripción automática" y, a continuación, seleccione **Aceptar** para guardar.

   :::image type="content" source="media/adfs-sso-properties-security.png" alt-text="Captura de pantalla que muestra la pestaña de seguridad de la plantilla de certificado de SSO, una vez configurada correctamente.":::

### <a name="enable-the-new-certificate-templates"></a>Habilite las nuevas plantillas de certificado:

Para habilitar las nuevas plantillas de certificado:

1. En la entidad de certificación, ejecute **mmc.exe** desde el menú Inicio para iniciar **Consola de administración de Microsoft**.
2. Seleccione **Archivo...**  > **Agregar o quitar complemento...**  > **Entidad de certificación** > **Agregar >**  > **Finalizar** > y **Aceptar** para ver la entidad de certificación.
3. Expanda la entidad de certificación en el panel izquierdo y abra **Plantillas de certificado**.
4. Haga clic con el botón derecho en el panel central que muestra la lista de plantillas de certificado, seleccione **Nuevo** y, a continuación, seleccione **Plantilla de certificado para emitir**.
5. Seleccione **Agente de inscripción de ADFS** y **SSO de ADFS** y, a continuación, seleccione **Aceptar**. Debería ver ambas plantillas en el panel central.

   :::image type="content" source="media/adfs-certificate-templates.png" alt-text="Captura de pantalla que muestra la lista de plantillas de certificado que se pueden emitir, incluidos el nuevo agente de inscripción de ADFS y el inicio de sesión único de ADFS.":::

   > [!NOTE]
   > Si ya tiene configurada una plantilla de certificado de agente de inscripción, solo tiene que agregar la plantilla SSO de ADFS.

## <a name="configure-the-ad-fs-servers"></a>Configuración de los servidores AD FS

Debe configurar los servidores de Servicios de federación de Active Directory (AD FS) para usar las nuevas plantillas de certificado y establecer la confianza de usuario autenticado para admitir el inicio de sesión único.

La confianza de usuario autenticado entre el servidor de AD FS y el servicio Azure Virtual Desktop permite que las solicitudes de certificado de inicio de sesión único se reenvíen correctamente al entorno de dominio.

Al configurar el inicio de sesión único de AD FS, debe elegir una clave o certificado compartidos:

* Si tiene un único servidor de AD FS, puede elegir una clave o un certificado compartidos.
* Si tiene varios servidores de AD FS, es necesario elegir el certificado.

La clave compartida o el certificado que se usa para generar el token para iniciar sesión en Windows debe almacenarse de forma segura en [Azure Key Vault](../key-vault/general/overview.md). Puede almacenar el secreto en un archivo de Key Vault o implementar uno nuevo. En cualquier caso, debe asegurarse de establecer la directiva de acceso correcta para que el servicio Azure Virtual Desktop pueda acceder a ella.

Cuando se usa un certificado, puede usar cualquier certificado de uso general; asimismo, no hay ningún requisito en el nombre del tema o en el nombre alternativo del tema (SAN). Aunque no es necesario, se recomienda crear un certificado que haya emitido una entidad de certificación válida. Este certificado se puede crear directamente en Azure Key Vault y debe tener una clave privada exportable. La clave pública se puede exportar y usar para configurar el servidor AD FS mediante el script siguiente. Tenga en cuenta que este certificado es diferente del certificado SSL de AD FS que debe tener un nombre de tema adecuado y una entidad de certificación válida.

El script de PowerShell **ConfigureWVDSSO.ps1** disponible en la [Galería de PowerShell](https://www.powershellgallery.com/packages/ConfigureWVDSSO) configurará el servidor de AD FS para la relación de confianza e instalará el certificado si es necesario.

Este script solo tiene un parámetro necesario, *ADFSAuthority*, que es la dirección URL que se resuelve en AD FS y usa "/adfs" como sufijo. Por ejemplo, `https://adfs.contoso.com/adfs`.

1. En las VM de AD FS virtuales, ejecute el siguiente cmdlet de PowerShell para configurar AD FS y así poder usar las plantillas de certificado de la sección anterior:
  
   ```powershell
   Set-AdfsCertificateAuthority -EnrollmentAgentCertificateTemplate "ADFSEnrollmentAgent" -LogonCertificateTemplate "ADFSSSO" -EnrollmentAgent
   ```
 
   > [!NOTE]
   > Si ya tiene un elemento enrollmentAgentCertificateTemplate configurado, asegúrese de usar el nombre de plantilla existente en lugar de ADFSEnrollmentAgent.

2. Ejecute el script ConfigureWVDSSO.ps1.
   > [!NOTE]
   > Necesita los valores de la variable `$config` para completar la siguiente parte de las instrucciones, por lo que no debe cerrar la ventana de PowerShell que usó para completar las instrucciones anteriores. Puede seguir usando la misma ventana de PowerShell o dejarla abierta al iniciar una nueva sesión de PowerShell.
   
   * Si usa una clave compartida en Key Vault, ejecute el siguiente cmdlet de PowerShell en el servidor de AD FS; para ello, reemplace ADFSServiceUrl con la dirección URL completa para conectarse al servicio de AD FS:

     ```powershell
     Install-Script ConfigureWVDSSO
     $config = ConfigureWVDSSO.ps1 -ADFSAuthority "<ADFSServiceUrl>" [-WvdWebAppAppIDUri "<WVD Web App URI>"] [-RdWebURL "<RDWeb URL>"]
     ```

     > [!NOTE]
     > Necesita las propiedades WvdWebAppAppIDUri y RdWebURL para configurar un entorno en una nube soberana como Azure Government. En la nube de Azure Commercial, estas propiedades se establecen automáticamente en `https://www.wvd.microsoft.com` y `https://rdweb.wvd.microsoft.com` respectivamente.

   * Si usa un certificado en Key Vault, ejecute el siguiente cmdlet de PowerShell en el servidor de AD FS; para ello, reemplace ADFSServiceUrl con la dirección URL completa para conectarse al servicio de AD FS:

     ```powershell
     Install-Script ConfigureWVDSSO
     $config = ConfigureWVDSSO.ps1 -ADFSAuthority "<ADFSServiceUrl>" -UseCert -CertPath "<Path to the pfx file>" -CertPassword <Password to the pfx file> [-WvdWebAppAppIDUri "<WVD Web App URI>"] [-RdWebURL "<RDWeb URL>"]
     ```

     > [!NOTE]
     > Necesita las propiedades WvdWebAppAppIDUri y RdWebURL para configurar un entorno en una nube soberana como Azure Government. En la nube de Azure Commercial, estas propiedades se establecen automáticamente en `https://www.wvd.microsoft.com` y `https://rdweb.wvd.microsoft.com` respectivamente.

3. Establezca la directiva de acceso en Azure Key Vault mediante la ejecución del siguiente cmdlet de PowerShell:

   ```powershell
   Set-AzKeyVaultAccessPolicy -VaultName "<Key Vault Name>" -ServicePrincipalName 9cdead84-a844-4324-93f2-b2e6bb768d07 -PermissionsToSecrets get -PermissionsToKeys sign
   ```

4. Almacene la clave o el certificado compartidos en Azure Key Vault con una etiqueta que contenga una lista de id. de suscripción separados por comas y que tengan permiso para usar el secreto.

   * Si usa una clave compartida en Key Vault, ejecute el siguiente cmdlet de PowerShell para almacenar la clave compartida y establecer la etiqueta:

     ```powershell
     $hp = Get-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>" 
     $secret = Set-AzKeyVaultSecret -VaultName "<Key Vault Name>" -Name "adfsssosecret" -SecretValue (ConvertTo-SecureString -String $config.SSOClientSecret  -AsPlainText -Force) -Tag @{ 'AllowedWVDSubscriptions' = $hp.Id.Split('/')[2]}
     ```

   * Si el certificado ya está en Key Vault, ejecute el siguiente cmdlet de PowerShell para establecer la etiqueta:

     ```powershell
     $hp = Get-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>"
     $secret = Update-AzKeyVaultCertificate -VaultName "<Key Vault Name>" -Name "<Certificate Name>" -Tag @{ 'AllowedWVDSubscriptions' = $hp.Id.Split('/')[2]} -PassThru
     ```

   * Si tiene un certificado local, ejecute el siguiente cmdlet de PowerShell para importar el certificado en Key Vault y establezca la etiqueta :

     ```powershell
     $hp = Get-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>" 
     $secret = Import-AzKeyVaultCertificate -VaultName "<Key Vault Name>" -Name "adfsssosecret" -Tag @{ 'AllowedWVDSubscriptions' = $hp.Id.Split('/')[2]} -FilePath "<Path to pfx>" -Password (ConvertTo-SecureString -String "<pfx password>"  -AsPlainText -Force)
     ```

> [!NOTE]
> Opcionalmente, puede configurar la frecuencia con la que se solicitan credenciales a los usuarios cambiando la [configuración del inicio de sesión único de AD FS](/windows-server/identity/ad-fs/operations/ad-fs-single-sign-on-settings#keep-me-signed-in-for-unauthenticated-devices). De forma predeterminada, se solicitará a los usuarios cada 8 horas en dispositivos no registrados.

## <a name="configure-your-azure-virtual-desktop-host-pool"></a>Configuración del grupo de host de Azure Virtual Desktop

> [!IMPORTANT]
> Durante la versión preliminar pública, debe configurar el grupo host para que esté en el [entorno de validación](create-validation-host-pool.md).

Es el momento de configurar los parámetros de SSO de AD FS en el grupo de host de Azure Virtual Desktop. Para ello, [configure el entorno de PowerShell](powershell-module.md) para Azure Virtual Desktop, si aún no lo ha hecho, y conéctese a su cuenta.

A continuación, actualice la información de SSO del grupo de host mediante la ejecución de uno de los dos cmdlets siguientes en la misma ventana de PowerShell en la VM de AD FS:

* Si usa una clave compartida en Key Vault, ejecute el siguiente cmdlet de PowerShell:

  ```powershell
  Update-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>" -SsoadfsAuthority "<ADFSServiceUrl>" -SsoClientId "<WVD Web App URI>" -SsoSecretType SharedKeyInKeyVault -SsoClientSecretKeyVaultPath $secret.Id
  ```

  > [!NOTE]
  > Debe establecer la propiedad SsoClientId para que coincida con la nube de Azure en la que va a implementar el inicio de sesión único. En la nube de Azure Commercial, esta propiedad debe establecerse en `https://www.wvd.microsoft.com`. Sin embargo, la configuración necesaria para esta propiedad será diferente para otras nubes, como la nube de Azure Government.

* Si usa un certificado en Key Vault, ejecute el siguiente cmdlet de PowerShell:

  ```powershell
  Update-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>" -SsoadfsAuthority "<ADFSServiceUrl>" -SsoClientId "<WVD Web App URI>" -SsoSecretType CertificateInKeyVault -SsoClientSecretKeyVaultPath $secret.Id
  ```

  > [!NOTE]
  > Debe establecer la propiedad SsoClientId para que coincida con la nube de Azure en la que va a implementar el inicio de sesión único. En la nube de Azure Commercial, esta propiedad debe establecerse en `https://www.wvd.microsoft.com`. Sin embargo, la configuración necesaria para esta propiedad será diferente para otras nubes, como la nube de Azure Government.

### <a name="configure-additional-host-pools"></a>Configuración de grupos de hosts adicionales

Cuando necesite configurar grupos de hosts adicionales, puede recuperar la configuración que usó para configurar un grupo de hosts existente para configurar el nuevo.

Para recuperar la configuración del grupo de hosts existente, abra una ventana de PowerShell y ejecute este cmdlet:

```powershell
Get-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>" | fl *
```

Puede seguir los pasos para [configurar el grupo de hosts de Azure Virtual Desktop](#configure-your-azure-virtual-desktop-host-pool) con los mismos valores de *SsoClientId*, *SsoClientSecretKeyVaultPath*, *SsoSecretType* y *SsoadfsAuthority*.

## <a name="removing-sso"></a>Eliminación del inicio de sesión único

Para deshabilitar el SSO en el grupo de hosts, ejecute el siguiente cmdlet:

```powershell
Update-AzWvdHostPool -Name "<Host Pool Name>" -ResourceGroupName "<Host Pool Resource Group Name>" -SsoadfsAuthority ''
```

Si también quiere deshabilitar el inicio de sesión único en el servidor de AD FS, ejecute este cmdlet:

```powershell
Install-Script UnConfigureWVDSSO
UnConfigureWVDSSO.ps1 -WvdWebAppAppIDUri "<WVD Web App URI>" -WvdClientAppApplicationID "a85cf173-4192-42f8-81fa-777a763e6e2c"
```

> [!NOTE]
> La propiedad WvdWebAppAppIDUri debe coincidir con la nube de Azure en la que se va a implementar. En la nube de Azure Commercial, esta propiedad es `https://www.wvd.microsoft.com`. Será diferente en otras nubes, como la nube de Azure Government.

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha configurado el inicio de sesión único, puede iniciar sesión en un cliente de Azure Virtual Desktop admitido para probarlo como parte de una sesión de usuario. Si quiere obtener información sobre cómo conectarse a una sesión mediante sus nuevas credenciales, consulte estos artículos:

* [Conexión con el cliente de Escritorio de Windows](connect-windows-7-10.md)
* [Conexión con el cliente web](connect-web.md)