---
title: Administración del servidor de configuración para la recuperación ante desastres con Azure Site Recovery
description: Obtenga información sobre las tareas comunes para administrar un servidor de configuración local para la recuperación ante desastres de máquinas virtuales VMware y servidores físicos en Azure con Azure Site Recovery.
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: e260b9fa7b24a3cb91e107bac6b01537879857c0
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576902"
---
# <a name="manage-the-configuration-server-for-vmware-vmphysical-server-disaster-recovery"></a>Administración del servidor de configuración para la recuperación ante desastres del servidor físico o VM de VMware

Se configura un servidor de configuración local cuando se usa [Azure Site Recovery](site-recovery-overview.md) para realizar la recuperación ante desastres de servidores físicos y máquinas virtuales de VMware en Azure. El servidor de configuración coordina la comunicación entre Azure y VMware local, además de administrar la replicación de datos. En este artículo se resumen las tareas comunes para administrar el servidor de configuración después de que se haya implementado.


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="update-windows-license"></a>Actualización de la licencia de Windows

La licencia que se proporciona con la plantilla de OVF es una licencia de evaluación válida durante 180 días. Para el uso ininterrumpido, debe activar Windows con una licencia adquirida. La actualización de la licencia se puede realizar mediante una clave independiente o una clave estándar de KMS. Hay instrucciones disponibles en la [línea de comandos de DISM Windows para el sistema operativo en ejecución](/windows-hardware/manufacture/desktop/dism-windows-edition-servicing-command-line-options). Para obtener las claves, consulte la [configuración del cliente KMS](/windows-server/get-started/kmsclientkeys).

## <a name="access-configuration-server"></a>Acceso al servidor de configuración

Puede acceder al servidor de configuración como se indica a continuación:

* Inicie sesión en la máquina virtual en la que está implementado e inicie el **administrador de configuración de Azure Site Recovery** desde el acceso directo del escritorio.
* Como alternativa, puede acceder al servidor de configuración de forma remota desde https://*NombreDeServidorDeConfiguración*/:44315/. Inicie sesión con las credenciales de administrador.

## <a name="modify-vmware-server-settings"></a>Modificación de la configuración del servidor de VMware

1. Para asociar otro servidor de VMware con el servidor de configuración, después de [iniciar sesión](#access-configuration-server), seleccione **Add vCenter Server/vSphere ESXi server** (Agregar servidor vCenter/servidor vSphere ESXi).
2. Escriba los detalles y seleccione **Aceptar**.

## <a name="modify-credentials-for-automatic-discovery"></a>Modificación de las credenciales para la detección automática

1. Para actualizar las credenciales usadas para conectarse al servidor de VMware para detectar automáticamente máquinas virtuales de VMware, después de [iniciar sesión](#access-configuration-server), elija la cuenta y haga clic en **Edit** (Editar).
2. Escriba las nuevas credenciales y, a continuación, seleccione **OK** (Aceptar).

    ![Modificación para VMware](./media/vmware-azure-manage-configuration-server/modify-vmware-server.png)

También puede modificar las credenciales con CSPSConfigtool.exe.

1. Inicie sesión en el servidor de configuración y ejecute CSPSConfigtool.exe.
2. Elija la cuenta que desea modificar y haga clic en **Edit** (Editar).
3. Escriba las credenciales modificadas y haga clic en **Ok** (Aceptar)

## <a name="modify-credentials-for-mobility-service-installation"></a>Modificación de las credenciales para la instalación de Mobility Service

Modifique las credenciales usadas para instalar automáticamente Mobility Service en las máquinas virtuales de VMware que habilite para la replicación.

1. Después de [iniciar sesión](#access-configuration-server), seleccione **Manage virtual machine credentials** (Administrar las credenciales de la máquina virtual)
2. Elija la cuenta que desea modificar y haga clic en **Edit** (Editar).
3. Escriba las nuevas credenciales y, a continuación, seleccione **OK** (Aceptar).

    ![Modificación de las credenciales de Mobility Service](./media/vmware-azure-manage-configuration-server/modify-mobility-credentials.png)

También puede modificar las credenciales con CSPSConfigtool.exe.

1. Inicie sesión en el servidor de configuración y ejecute CSPSConfigtool.exe.
2. Elija la cuenta que desea modificar y haga clic en **Edit** (Editar).
3. Escriba las nuevas credenciales y haga clic en **OK** (Aceptar).

## <a name="add-credentials-for-mobility-service-installation"></a>Adición de las credenciales para la instalación de Mobility Service

Si no agregó las credenciales durante la implementación de OVF del servidor de configuración,

1. Después de [iniciar sesión](#access-configuration-server), seleccione **Manage virtual machine credentials** (Administrar las credenciales de la máquina virtual)
2. Haga clic en **Add virtual machine credentials** (Agregar credenciales de máquina virtual).
    ![Captura de pantalla que muestra el panel Administrar las credenciales de la máquina virtual con el vínculo Agregar credenciales de máquina virtual.](media/vmware-azure-manage-configuration-server/add-mobility-credentials.png)
3. Escriba las nuevas credenciales y haga clic en **Add** (Agregar).

También puede agregar las credenciales con CSPSConfigtool.exe.

1. Inicie sesión en el servidor de configuración y ejecute CSPSConfigtool.exe.
2. Haga clic en **Add** (Agregar), escriba las nuevas credenciales y haga clic en **Ok** (Aceptar).

## <a name="modify-proxy-settings"></a>Modificación de la configuración de proxy

Modifique la configuración de proxy que utiliza la máquina del servidor de configuración para acceder por Internet a Azure. Si, además del servidor de procesos predeterminado que se ejecuta en la máquina del servidor de configuración, tiene una máquina de servidor de procesos adicional, modifique la configuración en ambas máquinas.

1. Después de [iniciar sesión](#access-configuration-server) en el servidor de configuración, seleccione **Manage connectivity** (Administrar la conectividad).
2. Actualice los valores del proxy. A continuación, seleccione **Save** (Guardar) para actualizar la configuración.

## <a name="add-a-network-adapter"></a>Incorporación de un adaptador de red

La plantilla OVF (Open Virtualization Format) implementa la máquina virtual del servidor de configuración con un único adaptador de red.

- Puede [agregar otro adaptador a la máquina virtual](vmware-azure-deploy-configuration-server.md#add-an-additional-adapter), pero debe hacerlo antes de registrar el servidor de configuración en el almacén.
- Para agregar un adaptador después de registrar el servidor de configuración en el almacén, agregue el adaptador en las propiedades de la máquina virtual. A continuación, es necesario [volver a registrar](#reregister-a-configuration-server-in-the-same-vault) el servidor en el almacén.

## <a name="how-to-renew-ssl-certificates"></a>Renovación de certificados SSL

El servidor de configuración lleva integrado un servidor web, que coordina las actividades de los agentes de Mobility en todas las máquinas protegidas, los servidores de procesos integrados o de escalabilidad horizontal y los servidores de destino maestros conectados a él. El servidor web usa un certificado SSL para autenticar clientes. El certificado expira después de tres años y se puede renovar en cualquier momento.

### <a name="check-expiry"></a>Comprobación de expiración

La fecha de expiración aparece en **Configuration Server health** (Mantenimiento del servidor de configuración). En lo que respecta a las implementaciones de servidores de configuración antes de mayo de 2016, la expiración del certificado se estableció en 1 año. Si tiene un certificado que va a expirar, ocurre lo siguiente:

- Cuando la fecha de expiración es dos meses o menos, el servicio comienza a enviar notificaciones en el portal y por correo electrónico (si está suscrito a las notificaciones de Azure Site Recovery).
- Aparece un banner de notificación en la página de recursos de almacén. Para más información, seleccione el banner.
- Si ve el botón **Actualizar ahora**, quiere decir que algunos componentes en el entorno no se han actualizado a la versión 9.4.xxxx.x o superior. Actualice los componentes antes de renovar el certificado. La renovación no es posible en versiones anteriores.

### <a name="if-certificates-are-yet-to-expire"></a>Si los certificados aún no han expirado

1. Para renovarlos, en el almacén, abra **Infraestructura de Site Recovery** > **Servidor de configuración**. Seleccione el servidor de configuración necesario.
2. Asegúrese de que los servidores de procesos de escalabilidad horizontal, los servidores de destino maestros y los agentes de movilidad de todos los componentes en todas las máquinas protegidas tengan las versiones más recientes y estén conectados.
3. A continuación, seleccione **Renovar certificados**.
4. Siga detenidamente las instrucciones de esta página y haga clic en Aceptar para renovar los certificados en el servidor de configuración seleccionado y en sus componentes asociados.

### <a name="if-certificates-have-already-expired"></a>Si los certificados ya han expirado

1. Si ya han expirado, los certificados **no se pueden renovar desde Azure Portal**. Antes de continuar, asegúrese de que los servidores de procesos de escalabilidad horizontal, los servidores de destino maestros y los agentes de movilidad de todos los componentes en todas las máquinas protegidas tengan las versiones más recientes y estén conectados.
2. **Siga este procedimiento solo si los certificados ya han expirado.** Inicie sesión en el servidor de configuración, vaya a la unidad C > ProgramData > Site Recovery > home > svsystems > bin y ejecute la herramienta de ejecutor "RenewCerts" como administrador.
3. Aparecerá una ventana de ejecución de PowerShell y se desencadenará la renovación de certificados. Esta operación puede tardar hasta 15 minutos. No cierre la ventana hasta que finalice la renovación.

:::image type="content" source="media/vmware-azure-manage-configuration-server/renew-certificates.png" alt-text="RenewCertificates":::

## <a name="reregister-a-configuration-server-in-the-same-vault"></a>Proceso para volver a registrar el servidor de configuración en el mismo almacén

Si lo necesita, puede volver a registrar el servidor de configuración en el mismo almacén. Si, además del servidor de procesos predeterminado que se ejecuta en la máquina del servidor de configuración, tiene una máquina del servidor de procesos adicional, vuelva a registrar ambas máquinas.


1. En el almacén, abra **Administrar** > **Infraestructura de Site Recovery** > **Servidores de configuración**.
2. En **Servers** (Servidores), seleccione **Download registration key** (Descargar clave de registro) para descargar el archivo de credenciales de almacén.
3. Inicie sesión en la máquina del servidor de configuración.
4. En **%ProgramData%\ASR\home\svsystems\bin**, abra **cspsconfigtool.exe**.
5. En la pestaña **Vault Registration** (Registro del almacén), seleccione **Browse** (Examinar) y busque el archivo de credenciales de almacén que ha descargado.
6. Si lo necesita, proporcione los detalles del servidor proxy. Después, seleccione **Register** (Registrar).
7. Abra la ventana de comandos de administrador de PowerShell y ejecute el comando siguiente:
   ```
    $pwd = ConvertTo-SecureString -String MyProxyUserPassword
    Set-OBMachineSetting -ProxyServer http://myproxyserver.domain.com -ProxyPort PortNumber – ProxyUserName domain\username -ProxyPassword $pwd
   ```

    >[!NOTE]
    >Para **extraer los últimos certificados** del servidor de configuración y escalar horizontalmente el servidor de procesos, ejecute el comando *"\<Installation Drive\Microsoft Azure Site Recovery\agent\cdpcli.exe>"--registermt*

8. Por último, reinicie el servicio OBEngine con la ejecución del siguiente comando.
   ```
        net stop obengine
        net start obengine
   ```


## <a name="register-a-configuration-server-with-a-different-vault"></a>Registro de un servidor de configuración con otro almacén

> [!WARNING]
> El siguiente paso desasocia el servidor de configuración del almacén actual y se detiene la replicación de todas las máquinas virtuales protegidas en el servidor de configuración.

1. Inicie sesión en el servidor de configuración.
2. Abra la ventana de comandos de administrador de PowerShell y ejecute el comando siguiente:

    ```
    reg delete "HKLM\Software\Microsoft\Azure Site Recovery\Registration"
    net stop dra
    ```
3. Inicie el portal de explorador del dispositivo del servidor de configuración mediante el acceso directo del escritorio.
4. Realice los pasos de registro, de forma similar al [registro](vmware-azure-tutorial.md#register-the-configuration-server) de un nuevo servidor de configuración.

## <a name="upgrade-the-configuration-server"></a>Actualización del servidor de configuración

Los paquetes acumulativos se ejecutan para actualizar el servidor de configuración. Las actualizaciones se pueden aplicar para hasta las versiones N-4. Por ejemplo:

- Si ejecuta 9.7 9.8, 9,9 o 9.10, puede actualizar directamente a 9.11.
- Si ejecuta la versión 9.6 o anterior, y desea actualizar a 9.11, primero debe actualizar a la versión 9.7 antes de hacerlo a la versión 9.11.

Para obtener instrucciones detalladas sobre la instrucción de soporte técnico de los componentes de Azure Site Recovery, consulte [aquí](./service-updates-how-to.md#support-statement-for-azure-site-recovery).
[Aquí](./service-updates-how-to.md#links-to-currently-supported-update-rollups) hay disponibles vínculos a paquetes acumulativos para actualizar a todas las versiones del servidor de configuración.

> [!IMPORTANT]
> Con cada nueva versión "N" de un componente de Azure Site Recovery que se publica, todas las versiones anteriores a "N-4" se consideran excluidas del soporte técnico. Siempre es aconsejable actualizar a las versiones más recientes disponibles.</br>
> Para obtener instrucciones detalladas sobre la instrucción de soporte técnico de los componentes de Azure Site Recovery, consulte [aquí](./service-updates-how-to.md#support-statement-for-azure-site-recovery).

Actualice el servidor como se indica a continuación:

1. En el almacén, vaya a **Manage** > **Site Recovery Infrastructure** > **Configuration Servers** (Administrar > Infraestructura de Site Recovery > Servidores de configuración).
2. Si hay alguna actualización disponible, aparecerá un vínculo en la columna **Agent Version** (Versión del agente) >.
    ![Actualizar](./media/vmware-azure-manage-configuration-server/update2.png)
3. Descargue el archivo del instalador de actualizaciones en el servidor de configuración.

    ![Captura de pantalla que muestra dónde hacer clic para descargar el archivo del instalador de actualizaciones.](./media/vmware-azure-manage-configuration-server/update1.png)

4. Haga doble clic para ejecutar el instalador.
5. El instalador detecta la versión actual que se ejecuta en la máquina. Haga clic en **Sí** para iniciar la actualización.
6. Cuando la actualización finaliza se valida la configuración del servidor.

    ![Captura de pantalla que muestra la configuración completada de validación del servidor.](./media/vmware-azure-manage-configuration-server/update3.png)

7. Haga clic en **Finish** (Finalizar) para cerrar el instalador.
8. Para actualizar el resto de los componentes de Site Recovery, consulte nuestra [guía de actualización](./service-updates-how-to.md#vmware-vmphysical-server-disaster-recovery-to-azure).

## <a name="upgrade-configuration-serverprocess-server-from-the-command-line"></a>Actualización del servidor de configuración o del servidor de procesos desde la línea de comandos

Ejecute el archivo de instalación del modo siguiente:

  ```
  UnifiedSetup.exe [/ServerMode <CS/PS>] [/InstallDrive <DriveLetter>] [/MySQLCredsFilePath <MySQL credentials file path>] [/VaultCredsFilePath <Vault credentials file path>] [/EnvType <VMWare/NonVMWare>] [/PSIP <IP address to be used for data transfer] [/CSIP <IP address of CS to be registered with>] [/PassphraseFilePath <Passphrase file path>]
  ```

### <a name="sample-usage"></a>Ejemplo de uso
  ```
  MicrosoftAzureSiteRecoveryUnifiedSetup.exe /q /x:C:\Temp\Extracted
  cd C:\Temp\Extracted
  UNIFIEDSETUP.EXE /AcceptThirdpartyEULA /servermode "CS" /InstallLocation "D:\" /MySQLCredsFilePath "C:\Temp\MySQLCredentialsfile.txt" /VaultCredsFilePath "C:\Temp\MyVault.vaultcredentials" /EnvType "VMWare"
  ```


### <a name="parameters"></a>Parámetros

|Nombre de parámetro| Tipo | Descripción| Valores|
|-|-|-|-|
| /ServerMode|Obligatorio|Especifica si se deben instalar los servidores de configuración y de procesos, o solo el servidor de procesos|CS<br>PS|
|/InstallLocation|Obligatorio|La carpeta donde se instalan los componentes| Cualquier carpeta del equipo|
|/MySQLCredsFilePath|Obligatorio|La ruta de acceso al archivo en que se almacenan las credenciales del servidor MySQL|El archivo debe tener el formato especificado a continuación|
|/VaultCredsFilePath|Obligatorio|La ruta de acceso del archivo de credenciales del almacén|Ruta de acceso de archivo válido|
|/EnvType|Obligatorio|El tipo de entorno que quiere proteger |VMware<br>NonVMware|
|/PSIP|Obligatorio|Dirección IP de la NIC que se usará para la transferencia de datos de replicación| Cualquier dirección IP válida|
|/CSIP|Obligatorio|Dirección IP de la NIC en la que el servidor de configuración realiza la escucha| Cualquier dirección IP válida|
|/PassphraseFilePath|Obligatorio|Ruta de acceso completa a la ubicación del archivo de frase de contraseña|Ruta de acceso de archivo válido|
|/BypassProxy|Opcional|Especifica si el servidor de configuración se conecta a Azure sin proxy|Para hacerlo, obtenga este valor desde Venu|
|/ProxySettingsFilePath|Opcional|Configuración del proxy (el proxy predeterminado requiere autenticación, o un proxy personalizado)|El archivo debe tener el formato especificado a continuación|
|DataTransferSecurePort|Opcional|Número de puerto en el PSIP que se usará para los datos de replicación| Número de puerto válido (el valor predeterminado es 9433)|
|/SkipSpaceCheck|Opcional|Omitir comprobación de espacio para disco de caché| |
|/AcceptThirdpartyEULA|Obligatorio|La marca implica la aceptación de los términos de licencia de terceros| |
|/ShowThirdpartyEULA|Opcional|Muestra los términos de licencia de terceros. Si se proporciona como entrada, se omiten todos los demás parámetros| |



### <a name="create-file-input-for-mysqlcredsfilepath"></a>Creación de entrada de archivo para MYSQLCredsFilePath

El parámetro MySQLCredsFilePath toma como entrada un archivo. Cree el archivo con el siguiente formato y páselo como parámetro de entrada MySQLCredsFilePath.
```ini
[MySQLCredentials]
MySQLRootPassword = "Password>"
MySQLUserPassword = "Password"
```
### <a name="create-file-input-for-proxysettingsfilepath"></a>Creación de entrada de archivo para ProxySettingsFilePath
El parámetro ProxySettingsFilePath toma un archivo como entrada. Cree el archivo con el siguiente formato y páselo como parámetro de entrada ProxySettingsFilePath.

```ini
[ProxySettings]
ProxyAuthentication = "Yes/No"
Proxy IP = "IP Address"
ProxyPort = "Port"
ProxyUserName="UserName"
ProxyPassword="Password"
```

## <a name="delete-or-unregister-a-configuration-server"></a>Eliminación o anulación del registro de un servidor de configuración

1. [Deshabilite la protección](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) de todas las máquinas virtuales del servidor de configuración.
2. [Desasocie](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) y [elimine](vmware-azure-set-up-replication.md#disassociate-or-delete-a-replication-policy) todas las directivas de replicación del servidor de configuración.
3. [Elimine](vmware-azure-manage-vcenter.md#delete-a-vcenter-server) todos los servidores vCenter y hosts de vSphere que estén asociados con el servidor de configuración.
4. En el almacén, abra **Site Recovery Infrastructure** >  (Infraestructura de Site Recovery) **Configuration Servers** (Servidores de configuración).
5. Seleccione el servidor de configuración que quiere eliminar. En la página **Details** (Detalles), haga clic en **Delete** (Eliminar).

    ![Eliminación de un servidor de configuración](./media/vmware-azure-manage-configuration-server/delete-configuration-server.png)


### <a name="delete-with-powershell"></a>Eliminación con PowerShell

También puede eliminar el servidor de configuración con PowerShell.

1. [Instale](/powershell/azure/install-Az-ps) el módulo de Azure PowerShell.
2. Inicie sesión en su cuenta de Azure mediante este comando:

    `Connect-AzAccount`
3. Seleccione la suscripción de almacén.

     `Get-AzSubscription –SubscriptionName <your subscription name> | Select-AzSubscription`
3.  Establezca el contexto de almacén.

    ```
    $vault = Get-AzRecoveryServicesVault -Name <name of your vault>
    Set-AzSiteRecoveryVaultSettings -ARSVault $vault
    ```
4. Recupere el servidor de configuración.

    `$fabric = Get-AzSiteRecoveryFabric -FriendlyName <name of your configuration server>`
6. Elimine el servidor de configuración.

    `Remove-AzSiteRecoveryFabric -Fabric $fabric [-Force]`

> [!NOTE]
> Puede usar la opción **-Force** de Remove-AzSiteRecoveryFabric para forzar la eliminación del servidor de configuración.

## <a name="generate-configuration-server-passphrase"></a>Generación de frase de contraseña del servidor de configuración

1. Inicie sesión en el servidor de configuración y, a continuación, abra una ventana del símbolo del sistema como administrador.
2. Para cambiar el directorio a la carpeta bin, ejecute el comando **cd %ProgramData%\ASR\home\svsystems\bin**
3. Para generar el archivo de frase de contraseña, ejecute **genpassphrase.exe -v > MobSvc.passphrase**.
4. La frase de contraseña se almacenará en el archivo ubicado en **%ProgramData%\ASR\home\svsystems\bin\MobSvc.passphrase**.

## <a name="refresh-configuration-server"></a>Actualización del servidor de configuración

1. En Azure Portal, navegue a **Almacén de Recovery Services** > **Administrar** > **Infraestructura de Site Recovery** > **For VMware & Physical machines** (Para máquinas físicas y de VMware) > **Servidores de configuración**.
2. Haga clic en el servidor de configuración que quiere actualizar.
3. En la hoja con detalles del servidor de configuración elegido, haga clic en **Más** > **Actualizar servidor**.
4. Supervise el progreso del trabajo en **Almacén de Recovery Services** > **Supervisión** > **Trabajos de Site Recovery**.

## <a name="failback-requirements"></a>Requisitos de conmutación por recuperación

Durante la reprotección y la conmutación por recuperación, el servidor de configuración local debe estar ejecutándose y en un estado de conexión. Para la conmutación por recuperación correcta, la máquina virtual que se conmutó debe existir en la base de datos del servidor de configuración.

Asegúrese de realizar una copia de seguridad programada periódica del servidor de configuración. Si se produce un desastre y se pierde el servidor de configuración, primero debe restaurar el servidor de configuración a partir de una copia de seguridad y asegurarse de que el servidor de configuración restaurado tenga la misma dirección IP con la que se registró en el almacén. La conmutación por recuperación no funcionará si se usa una dirección IP diferente para el servidor de configuración restaurado.

## <a name="next-steps"></a>Pasos siguientes

Revise los tutoriales para configurar la recuperación ante desastres de [máquinas virtuales de VMware](vmware-azure-tutorial.md) en Azure.
