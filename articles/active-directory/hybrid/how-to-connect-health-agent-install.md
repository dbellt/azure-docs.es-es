---
title: Instalación de agentes de Connect Health en Azure Active Directory
description: En este artículo de Azure AD Connect Health se describe la instalación del agente de Servicios de federación de Active Directory (AD FS) y de Sincronización.
services: active-directory
documentationcenter: ''
author: billmath
manager: daveba
editor: curtand
ms.assetid: 1cc8ae90-607d-4925-9c30-6770a4bd1b4e
ms.service: active-directory
ms.subservice: hybrid
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 10/20/2020
ms.topic: how-to
ms.author: billmath
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 08aad681160a4f0d369fd312ba58c3ea6c7e8780
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110454129"
---
# <a name="azure-ad-connect-health-agent-installation"></a>Instalación del agente de Azure AD Connect Health

En este artículo, aprenderá a instalar y configurar los agentes de Azure Active Directory (Azure AD) Connect Health. Para descargar los agentes, consulte [estas instrucciones](how-to-connect-install-roadmap.md#download-and-install-azure-ad-connect-health-agent).

## <a name="requirements"></a>Requisitos

En la tabla siguiente aparece una lista de requisitos para utilizar Azure AD Connect Health.

| Requisito | Descripción |
| --- | --- |
| Hay una suscripción de Azure AD Premium (P1 o P2).  |Azure AD Connect Health es una característica de Azure AD Premium (P1 o P2). Para más información, consulte [Suscripción a las ediciones Azure Active Directory Premium](../fundamentals/active-directory-get-started-premium.md). <br /><br />Para iniciar una prueba gratis de 30 días, consulte cómo [iniciar una prueba](https://azure.microsoft.com/trial/get-started-active-directory/). |
| Es administrador global en Azure AD. |De forma predeterminada, solo los administradores globales pueden instalar y configurar los agentes de mantenimiento, acceder al portal y realizar operaciones en Azure AD Connect Health. Para más información, consulte [Administración del directorio de Azure AD](../fundamentals/active-directory-whatis.md). <br /><br /> Con el control de acceso basado en rol de Azure (RBAC de Azure) puede permitir que otros usuarios de la organización accedan a Azure AD Connect Health. Para más información, consulte el artículo sobre [RBAC de Azure para Azure AD Connect Health](how-to-connect-health-operations.md#manage-access-with-azure-rbac). <br /><br />**Importante**: Utilice una cuenta profesional o educativa para instalar los agentes. No puede usar una cuenta de Microsoft. Para más información, consulte [Registro en Azure AD como organización](../fundamentals/sign-up-organization.md). |
| El agente de Azure AD Connect Health está instalado en cada servidor de destino. | Los agentes de mantenimiento se deben instalar y configurar en los servidores de destino para que puedan recibir datos y proporcionar las funcionalidades de supervisión y análisis. <br /><br />Por ejemplo, para obtener datos de la infraestructura de Servicios de federación de Active Directory (AD FS), debe instalar el agente en el servidor de AD FS y en el servidor de Proxy de aplicación web. Del mismo modo, para obtener datos de la infraestructura de Azure AD Domain Services local (Azure AD DS), debe instalar el agente en los controladores de dominio.  |
| Los puntos de conexión del servicio de Azure tienen una conectividad de salida. | Durante la instalación y el tiempo de ejecución, el agente requiere conectividad a los puntos de conexión del servicio de Azure AD Connect Health. Si los firewalls bloquean la conectividad de salida, agregue los [puntos de conexión de conectividad de salida](how-to-connect-health-agent-install.md#outbound-connectivity-to-the-azure-service-endpoints) a la lista de permitidos. |
|La conectividad de salida se basa en direcciones IP. | Para más información sobre el filtrado de firewall basado en direcciones IP, consulte los [intervalos IP de Azure](https://www.microsoft.com/download/details.aspx?id=56519).|
| La inspección de TLS del tráfico de salida está filtrada o deshabilitada. | Las operaciones de carga de datos o de paso de registro de agente pueden generar un error si la inspección de TLS o la finalización del tráfico de salida se producen en el nivel de red. Para más información, consulte el artículo sobre la [configuración de la inspección de TLS](/previous-versions/tn-archive/ee796230(v=technet.10)). |
| Los puertos del firewall en el servidor están ejecutando el agente. |El agente requiere que los siguientes puertos de firewall estén abiertos para que se pueda comunicar con los puntos de conexión del servicio de Azure AD Connect Health: <br /><li>Puerto TCP 443</li><li>Puerto TCP 5671</li> <br />La versión más reciente del agente no requiere el puerto 5671. Actualice a la versión más reciente, para que solo sea necesario el puerto 443. Para más información, consulte [La identidad híbrida requería puertos y protocolos](./reference-connect-ports.md). |
| Si está habilitada la seguridad mejorada de Internet Explorer, permita los sitios web especificados.  |Si está habilitada la seguridad mejorada de Internet Explorer, permita los siguientes sitios web en el servidor en el que instala el agente:<br /><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com</li><li>https:\//login.windows.net</li><li>https:\//aadcdn.msftauth.net</li><li>El servidor de federación de su organización que sea de confianza para Azure AD (por ejemplo, https:\//sts.contoso.com).</li> <br />Para más información, consulte el artículo sobre la [configuración de Internet Explorer](https://support.microsoft.com/help/815141/internet-explorer-enhanced-security-configuration-changes-the-browsing). Si tiene un proxy en la red, consulte la nota que aparece al final de esta tabla.|
| Está instalada la versión 4.0 o posterior de PowerShell. | Windows Server 2012 incluye la versión 3.0 de PowerShell. Esta versión *no* es suficiente para el agente.</br></br> Windows Server 2012 R2 y las versiones posteriores incluyen una versión lo suficientemente reciente de PowerShell.|
|FIPS (Estándar federal de procesamiento de información) está deshabilitado.|Los agentes de Azure AD Connect Health no admiten FIPS.|

> [!IMPORTANT]
> Windows Server Core no admite la instalación del agente de Azure AD Connect Health.


> [!NOTE]
> Si tiene un entorno muy bloqueado y restringido, debe agregar más direcciones URL que las que aparecen en la tabla para una seguridad mejorada de Internet Explorer. Agregue también las direcciones URL que aparecen en la tabla de la siguiente sección.  


### <a name="outbound-connectivity-to-the-azure-service-endpoints"></a>Conectividad saliente a los extremos del servicio de Azure

Durante la instalación y el tiempo de ejecución, el agente necesita conectividad con los puntos de conexión del servicio de Azure AD Connect Health. Si los firewalls bloquean la conectividad de salida, asegúrese de que las direcciones URL de la tabla siguiente no estén bloqueadas de forma predeterminada. 

No deshabilite la supervisión de seguridad ni la inspección de estas direcciones URL. En su lugar, permítalas como lo haría con otro tipo de tráfico de Internet. 

Estas direcciones URL permiten la comunicación con los puntos de conexión de servicio de Azure AD Connect Health. Más adelante en este artículo, aprenderá a [comprobar la conectividad de salida](#test-connectivity-to-azure-ad-connect-health-service) mediante `Test-AzureADConnectHealthConnectivity`.

| Entorno de dominio | Puntos de conexión del servicio de Azure necesarios |
| --- | --- |
| Público general | <li>&#42;.blob.core.windows.net </li><li>&#42;.aadconnecthealth.azure.com </li><li>&#42;.servicebus.windows.net - Puerto: 5671 (este punto de conexión no es necesario en la versión más reciente del agente).</li><li>&#42;.adhybridhealth.azure.com/</li><li>https:\//management.azure.com </li><li>https:\//policykeyservice.dc.ad.msft.net/</li><li>https:\//login.windows.net</li><li>https:\//login.microsoftonline.com</li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com (este punto de conexión solo se utiliza para la detección durante el registro).</li> <li>https://aadcdn.msftauth.net</li><li>https://aadcdn.msauth.net</li> |
| Azure Alemania | <li>&#42;.blob.core.cloudapi.de </li><li>&#42;.servicebus.cloudapi.de </li> <li>&#42;.aadconnecthealth.microsoftazure.de </li><li>https:\//management.microsoftazure.de </li><li>https:\//policykeyservice.aadcdi.microsoftazure.de </li><li>https:\//login.microsoftonline.de </li><li>https:\//secure.aadcdn.microsoftonline-p.de </li><li>https:\//www.office.de (este punto de conexión solo se utiliza para la detección durante el registro).</li> <li>https://aadcdn.msftauth.net</li><li>https://aadcdn.msauth.net</li> |
| Azure Government | <li>&#42;.blob.core.usgovcloudapi.net </li> <li>&#42;.servicebus.usgovcloudapi.net </li> <li>&#42;.aadconnecthealth.microsoftazure.us </li> <li>https:\//management.usgovcloudapi.net </li><li>https:\//policykeyservice.aadcdi.azure.us </li><li>https:\//login.microsoftonline.us </li><li>https:\//secure.aadcdn.microsoftonline-p.com </li><li>https:\//www.office.com (este punto de conexión solo se utiliza para la detección durante el registro).</li> <li>https://aadcdn.msftauth.net</li><li>https://aadcdn.msauth.net</li> |


## <a name="install-the-agent"></a>Instalación del agente

Para descargar e instalar un agente de Azure AD Connect Health: 

* Asegúrese de que cumple los [requisitos](how-to-connect-health-agent-install.md#requirements) de Azure AD Connect Health.
* Introducción a Azure AD Connect Health para AD FS:
    * [Descargue el agente de Azure AD Connect Health para AD FS.](https://go.microsoft.com/fwlink/?LinkID=518973)
    * Consulte las [instrucciones de instalación](#install-the-agent-for-ad-fs).
* Introducción al uso de Azure AD Connect Health para Sincronización:
    * [Descargue e instale la versión más reciente de Azure AD Connect](https://go.microsoft.com/fwlink/?linkid=615771). El agente de mantenimiento para Sincronización se instala como parte de la instalación de Azure AD Connect (versión 1.0.9125.0 o posterior).
* Introducción a Azure AD Connect Health para Azure AD DS:
    * [Descargue el agente de Azure AD Connect Health para AD DS.](https://go.microsoft.com/fwlink/?LinkID=820540)
    * Consulte las [instrucciones de instalación](#install-the-agent-for-azure-ad-ds).

## <a name="install-the-agent-for-ad-fs"></a>Instalación del agente para AD FS

> [!NOTE]
> El servidor de AD FS no puede ser el mismo que el servidor de Sincronización. No instale el agente de AD FS en su servidor de Sincronización.
>

Antes de que instale el agente, asegúrese de que el nombre de host del servidor de AD FS sea único y no esté presente en el servicio de AD FS.
Para iniciar la instalación del agente, haga doble clic en el archivo *.exe* que ha descargado. En la primera ventana, seleccione **Instalar**.

![Captura de pantalla que muestra la ventana de instalación del agente de Azure AD Connect Health para AD FS.](./media/how-to-connect-health-agent-install/install1.png)

Cuando finalice la instalación, seleccione **Configurar ahora**.

![Captura de pantalla que muestra el mensaje de confirmación para la instalación del agente de Azure AD Connect Health para AD FS.](./media/how-to-connect-health-agent-install/install2.png)

Se abre una ventana de PowerShell para iniciar el proceso de registro del agente. Cuando se le solicite, inicie sesión con una cuenta de Azure AD que tenga permisos para registrar el agente. De forma predeterminada, la cuenta de administrador global tiene permisos.

![Captura de pantalla que muestra la ventana de inicio de sesión de Azure AD Connect Health para AD FS.](./media/how-to-connect-health-agent-install/install3.png)

Después de que inicie sesión, PowerShell continúa. Cuando PowerShell termine, puede cerrarlo. Se ha completado la configuración.

En este momento, se deben iniciar automáticamente los servicios del agente que le permiten cargar de forma segura los datos necesarios en el servicio en la nube.

Si no ha cumplido todos los requisitos previos, aparecerán advertencias en la ventana de PowerShell. Asegúrese de completar todos los [requisitos](how-to-connect-health-agent-install.md#requirements) antes de instalar el agente. La siguiente captura de pantalla muestra un ejemplo de estas advertencias.

![Captura de pantalla que muestra el script de configuración de Azure AD Connect Health para AD FS.](./media/how-to-connect-health-agent-install/install4.png)

Para comprobar que se ha instalado el agente, busque los siguientes servicios en el servidor. Si completó la configuración, estos servicios deben aparecer en ejecución. De lo contrario, estarán detenidos hasta que se complete la configuración.

* Servicio de diagnóstico de AD FS de Azure AD Connect Health
* Servicio de análisis de AD FS de Azure AD Connect Health
* Servicio de supervisión de AD FS de Azure AD Connect Health

![Captura de pantalla que muestra los servicios de Azure AD Connect Health para AD FS.](./media/how-to-connect-health-agent-install/install5.png)


### <a name="enable-auditing-for-ad-fs"></a>Habilitación de la auditoría de AD FS

> [!NOTE]
> Esta sección se aplica solo a los servidores de AD FS. No es necesario seguir estos pasos en los servidores de Proxy de aplicación web.
>

La característica Análisis de uso necesita recopilar y analizar datos. Por lo tanto, el agente de Azure AD Connect Health necesita la información de los registros de auditoría de AD FS. Estos registros no están habilitados de forma predeterminada. Utilice los procedimientos siguientes para habilitar la auditoría de AD FS y localizar los registros de auditoría de AD FS en los servidores de AD FS.

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2012-r2"></a>Para habilitar la auditoría de AD FS en Windows Server 2012 R2

1. En la pantalla Inicio, abra **Administrador del servidor** y, a continuación, **Directiva de seguridad local**. O bien, en la barra de tareas, abra **Administrador del servidor** y seleccione **Herramientas/Directiva de seguridad local**.
2. Vaya a la carpeta *Configuración de seguridad\Directivas locales\Configuración de seguridad\Asignación de derechos de usuario*. Haga doble clic en **Generar auditorías de seguridad**.
3. En la pestaña **Configuración de seguridad local** , compruebe que aparezca la cuenta de servicio de AD FS. Si no aparece, seleccione **Agregar usuario o grupo** y agréguela a la lista. Después, seleccione **Aceptar**.
4. Para habilitar una auditoria, abra una ventana del símbolo del sistema con privilegios elevados. Luego, ejecute el siguiente comando: 
    
    `auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable`
1. Cierre **Directiva de seguridad local**.
    >[!Important]
    >Los pasos siguientes solo son necesarios en los servidores principales de AD FS. 
1. Abra el complemento **Administración de AD FS**. (En **Administrador del servidor**, seleccione **Herramientas** > **Administración de AD FS**.)
1. En el panel **Acciones**, seleccione **Modificar las propiedades del Servicio de federación**.
1. En el cuadro de diálogo **Propiedades del servicio de federación**, seleccione la pestaña **Eventos**.
1. Active las casillas **Auditorías de aciertos y Auditorías de errores** y seleccione luego **Aceptar**.
1. Para habilitar el registro detallado mediante PowerShell, utilice el siguiente comando: 

    `Set-AdfsProperties -LOGLevel Verbose`

#### <a name="to-enable-auditing-for-ad-fs-on-windows-server-2016"></a>Para habilitar la auditoría de AD FS en Windows Server 2016

1. En la pantalla Inicio, abra **Administrador del servidor** y, a continuación, **Directiva de seguridad local**. O bien, en la barra de tareas, abra **Administrador del servidor** y seleccione **Herramientas/Directiva de seguridad local**.
2. Vaya a la carpeta *Configuración de seguridad\Directivas locales\Asignación de derechos de usuario* y haga doble clic en **Generar auditorías de seguridad**.
3. En la pestaña **Configuración de seguridad local** , compruebe que aparezca la cuenta de servicio de AD FS. Si no aparece, seleccione **Agregar usuario o grupo** y agregue la cuenta de servicio de AD FS a la lista. Después, seleccione **Aceptar**.
4. Para habilitar una auditoria, abra una ventana del símbolo del sistema con privilegios elevados. Luego, ejecute el siguiente comando: 

    `auditpol.exe /set /subcategory:{0CCE9222-69AE-11D9-BED3-505054503030} /failure:enable /success:enable`
1. Cierre **Directiva de seguridad local**.
    >[!Important]
    >Los pasos siguientes solo son necesarios en los servidores principales de AD FS.
1. Abra el complemento **Administración de AD FS**. (En **Administrador del servidor**, seleccione **Herramientas** > **Administración de AD FS**.)
1. En el panel **Acciones**, seleccione **Modificar las propiedades del Servicio de federación**.
1. En el cuadro de diálogo **Propiedades del servicio de federación**, seleccione la pestaña **Eventos**.
1. Active las casillas **Auditorías de aciertos y Auditorías de errores** y seleccione luego **Aceptar**. Las auditorías de aciertos y las auditorías de errores deberían estar habilitadas de forma predeterminada.
1. Abra una ventana de PowerShell y ejecute el siguiente comando: 

    `Set-AdfsProperties -AuditLevel Verbose`

El nivel de auditoría "básico" está habilitado de forma predeterminada. Para más información, consulte [Mejoras de auditorías de AD FS en Windows Server 2016](/windows-server/identity/ad-fs/technical-reference/auditing-enhancements-to-ad-fs-in-windows-server).


#### <a name="to-locate-the-ad-fs-audit-logs"></a>Para buscar los registros de auditoría de AD FS

1. Abra **Visor de eventos**.
2. Vaya a **Registros de Windows** y seleccione **Seguridad**.
3. A la derecha, seleccione **Filter Current Logs** (Filtrar registros actuales).
4. En **Orígenes de eventos**, seleccione **AD FS Auditing**(Auditoría de AD FS).

    Para más información acerca de los registros de auditoría, consulte [Preguntas sobre operaciones](reference-connect-health-faq.md#operations-questions).

    ![Captura de pantalla que muestra la ventana Filter Current Logs (Filtrar registros actuales). En el campo "Orígenes de eventos", está seleccionada la opción "AD FS Auditing" (Auditoría de AD FS).](./media/how-to-connect-health-agent-install/adfsaudit.png)

> [!WARNING]
> Una directiva de grupo puede deshabilitar la auditoría de AD FS. Si se deshabilita la auditoría de AD FS, no estará disponible el análisis de uso sobre las actividades de inicio de sesión. Asegúrese de no tener ninguna directiva de grupo que deshabilite la auditoría de AD FS.
>


## <a name="install-the-agent-for-sync"></a>Instalación del agente para Sincronización

El agente de Azure AD Connect Health para Sincronización se instala automáticamente en la versión más reciente de Azure AD Connect. Para utilizar Azure AD Connect para Sincronización, [descargue la versión más reciente de Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) e instálela.

Para comprobar que se instaló el agente, busque los siguientes servicios en el servidor. Si ha completado la configuración, los servicios deben aparecer ya en ejecución. De lo contrario, estarán detenidos hasta que se complete la configuración.

* Servicio de análisis de sincronización de Azure AD Connect Health
* Servicio de supervisión de sincronización de Azure AD Connect Health

![Captura de pantalla que muestra Azure AD Connect Health en ejecución para los servicios de Sincronización en el servidor.](./media/how-to-connect-health-agent-install/services.png)

> [!NOTE]
> Recuerde que debe tener Azure AD Premium (P1 o P2) para utilizar Azure AD Connect Health. Si no lo tiene, no podrá completar la configuración en Azure Portal. Para más información, consulte los [requisitos](how-to-connect-health-agent-install.md#requirements).
>
>

## <a name="manually-register-azure-ad-connect-health-for-sync"></a>Registro manual de Azure AD Connect Health para Sincronización

Si se produce un error del agente de registro de Azure AD Connect Health para Sincronización después de instalar correctamente Azure AD Connect, puede utilizar un comando de PowerShell para registrar el agente manualmente.

> [!IMPORTANT]
> Utilice este comando de PowerShell solo si se produce un error en el registro del agente después de instalar Azure AD Connect.
>
>

Registre manualmente el agente de Azure AD Connect Health para Sincronización con el siguiente comando de PowerShell. Los servicios de Azure AD Connect Health se iniciarán después de que el agente se registre correctamente.

`Register-AzureADConnectHealthSyncAgent -AttributeFiltering $true -StagingMode $false`

El comando toma los siguientes parámetros:

* **AttributeFiltering**: `$true` (predeterminado) si Azure AD Connect no está sincronizando el conjunto de atributos predeterminado y se ha personalizado para usar un conjunto de atributos filtrados. De lo contrario, use `$false`.
* **StagingMode**: `$false` (predeterminado) si el servidor de Azure AD Connect *no* está en modo de ensayo. Si el servidor está configurado para estar en modo de ensayo, utilice `$true`.

Cuando se le solicite autenticación, use la misma cuenta de administrador global (como admin@domain.onmicrosoft.com) que utilizó para configurar Azure AD Connect.

## <a name="install-the-agent-for-azure-ad-ds"></a>Instalación del agente para Azure AD DS

Para iniciar la instalación del agente, haga doble clic en el archivo *.exe* que ha descargado. En la primera ventana, seleccione **Instalar**.

![Captura de pantalla que muestra el agente de Azure AD Connect Health para la ventana de instalación del agente de AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install1.png)

Cuando finalice la instalación, seleccione **Configurar ahora**.

![Captura de pantalla que muestra la ventana que finaliza la instalación del agente de Azure AD Connect Health para Azure AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install2.png)

Se abre una ventana del símbolo del sistema. PowerShell ejecuta `Register-AzureADConnectHealthADDSAgent`. Cuando se le solicite, inicie sesión en Azure.

![Captura de pantalla que muestra la ventana de inicio de sesión del agente de Azure AD Connect Health para Azure AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install3.png)

Después de que inicie sesión, PowerShell continúa. Cuando PowerShell termine, puede cerrarlo. Se ha completado la configuración.

En este punto, los servicios se deberían iniciar automáticamente y permiten que el agente supervise y recopile datos. Si no ha cumplido todos los requisitos previos descritos en las secciones anteriores, verá unas advertencias en la ventana de PowerShell. Asegúrese de completar todos los [requisitos](how-to-connect-health-agent-install.md#requirements) antes de instalar el agente. La siguiente captura de pantalla muestra un ejemplo de estas advertencias.

![Captura de pantalla que muestra una advertencia para la configuración del agente de Azure AD Connect Health para Azure AD DS.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install4.png)

Para comprobar que el agente está instalado, busque los siguientes servicios en el controlador de dominio:

* Servicio de análisis de AD DS de Azure AD Connect Health
* Servicio de supervisión de AD DS de Azure AD Connect Health

Si completó la configuración, estos servicios deben aparecer ya en ejecución. De lo contrario, estarán detenidos hasta que finalice la configuración.

![Captura de pantalla que muestra los servicios en ejecución en el controlador de dominio.](./media/how-to-connect-health-agent-install/aadconnect-health-adds-agent-install5.png)

### <a name="quickly-install-the-agent-on-multiple-servers"></a>Instalación rápida del agente en varios servidores

1. Cree una cuenta de usuario en Azure AD. Protéjala mediante una contraseña.
2. Asigne el rol **Propietario** a esta cuenta de Azure AD local en Azure AD Connect Health mediante el portal. Siga [estos pasos](how-to-connect-health-operations.md#manage-access-with-azure-rbac). Asigne el rol a todas las instancias de servicio. 
3. Descargue el archivo de MSI *.exe* en el controlador de dominio local para la instalación.
4. Ejecute el siguiente script. Reemplace los parámetros por la nueva cuenta de usuario y su contraseña. 

    ```powershell
    AdHealthAddsAgentSetup.exe /quiet
    Start-Sleep 30
    $userName = "NEWUSER@DOMAIN"
    $secpasswd = ConvertTo-SecureString "PASSWORD" -AsPlainText -Force
    $myCreds = New-Object System.Management.Automation.PSCredential ($userName, $secpasswd)
    import-module "C:\Program Files\Azure Ad Connect Health Adds Agent\PowerShell\AdHealthAdds"
     
    Register-AzureADConnectHealthADDSAgent -Credential $myCreds
    
    ```

Cuando haya terminado, puede quitar el acceso a la cuenta local realizando una o varias de las siguientes tareas: 
* Quite la asignación de rol de la cuenta local para Azure AD Connect Health.
* Cambie la contraseña de la cuenta local. 
* Deshabilite la cuenta local de Azure AD.
* Elimine la cuenta local de Azure AD.  

## <a name="register-the-agent-by-using-powershell"></a>Registro del agente mediante PowerShell

Después de instalar el archivo *setup.exe* del agente adecuado, puede registrar el agente con los siguientes comandos de PowerShell en función del rol. Abra una ventana de PowerShell y ejecute el comando adecuado:

```powershell
    Register-AzureADConnectHealthADFSAgent
    Register-AzureADConnectHealthADDSAgent
    Register-AzureADConnectHealthSyncAgent

```

> [!NOTE]
> Para registrarse en nubes soberanas, use las siguientes líneas de comandos:
>
> ```powershell
> Register-AzureADConnectHealthADFSAgent -UserPrincipalName upn-of-the-user
> Register-AzureADConnectHealthADDSAgent -UserPrincipalName upn-of-the-user
> Register-AzureADConnectHealthSyncAgent -UserPrincipalName upn-of-the-user
> ```
>


Estos comandos aceptan `Credential` como parámetro para completar el registro de forma no interactiva o para completarlo en una máquina que ejecuta Server Core. Tenga en cuenta que:
* Puede capturar `Credential` en una variable de PowerShell que se pasa como parámetro.
* Puede proporcionar cualquier identidad de Azure AD que tenga permisos para registrar los agentes y que *no* tenga la autenticación multifactor habilitada.
* De forma predeterminada, los administradores globales tienen permisos para registrar los agentes. También puede permitir que otras identidades con menos privilegios realicen este paso. Para más información, consulte [Azure RBAC](how-to-connect-health-operations.md#manage-access-with-azure-rbac).

```powershell
    $cred = Get-Credential
    Register-AzureADConnectHealthADFSAgent -Credential $cred

```

## <a name="configure-azure-ad-connect-health-agents-to-use-http-proxy"></a>Configuración de agentes de Azure AD Connect Health para usar el proxy HTTP

Puede configurar agentes de Azure AD Connect Health para trabajar con un proxy HTTP.

> [!NOTE]
> * No se admite `Netsh WinHttp set ProxyServerAddress`. El agente utiliza System.Net en lugar de los servicios HTTP de Windows para crear solicitudes web.
> * La dirección del proxy HTTP configurada se utiliza para transmitir mensajes HTTPS cifrados.
> * No se admiten los servidores proxy autenticados (mediante HTTPBasic).
>
>

### <a name="change-the-agent-proxy-configuration"></a>Cambio de configuración de proxy del agente

Para configurar el agente de Azure AD Connect Health a fin de que utilice un proxy HTTP, puede hacer lo siguiente:
* Importación de configuración de proxy existente
* Especificación manual de direcciones de proxy
* Borrado de la configuración de proxy existente

> [!NOTE]
> Para actualizar la configuración de proxy, debe reiniciar todos los servicios del agente de Azure AD Connect Health. Ejecute el siguiente comando:
>
> `Restart-Service AzureADConnectHealth*`

#### <a name="import-existing-proxy-settings"></a>Importación de configuración de proxy existente

Puede importar la configuración de proxy HTTP de Internet Explorer para que los agentes de Azure AD Connect Health puedan utilizar la configuración. En cada uno de los servidores que ejecutan el agente de mantenimiento, ejecute el siguiente comando de PowerShell:

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromInternetSettings
```

Puede importar la configuración de proxy WinHTTP para que los agentes de Azure AD Connect Health puedan utilizarla. En cada uno de los servidores que ejecutan el agente de mantenimiento, ejecute el siguiente comando de PowerShell:

```powershell
Set-AzureAdConnectHealthProxySettings -ImportFromWinHttp
```

#### <a name="specify-proxy-addresses-manually"></a>Especificación manual de direcciones de proxy

Puede especificar un servidor proxy de forma manual. En cada uno de los servidores que ejecutan el agente de mantenimiento, ejecute el siguiente comando de PowerShell:

```powershell
Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress address:port
```

Este es un ejemplo: 

`Set-AzureAdConnectHealthProxySettings -HttpsProxyAddress myproxyserver: 443`

En este ejemplo:
* El valor `address` puede ser un nombre de dominio DNS que se pueda resolver o una dirección IPv4.
* Puede omitir `port`. Si es así, el puerto predeterminado es 443.

#### <a name="clear-the-existing-proxy-configuration"></a>Borrado de la configuración de proxy existente

Puede borrar la configuración de proxy existente ejecutando el comando siguiente:

```powershell
Set-AzureAdConnectHealthProxySettings -NoProxy
```

### <a name="read-current-proxy-settings"></a>Lectura de la configuración de proxy actual

Puede ejecutar el comando siguiente para leer la configuración de proxy actual:

```powershell
Get-AzureAdConnectHealthProxySettings
```

## <a name="test-connectivity-to-azure-ad-connect-health-service"></a>Prueba de la conectividad con el servicio Azure AD Connect Health

Ocasionalmente, el agente de Azure AD Connect Health puede perder la conectividad con el servicio de Azure AD Connect Health. Las causas de esta pérdida de conectividad pueden incluir problemas de red, problemas de permisos y otros problemas.

Si el agente no puede enviar datos al servicio de Azure AD Connect Health durante más de dos horas, aparecerá la siguiente alerta en el portal: "Los datos del Servicio de mantenimiento no están actualizados". 

Puede averiguar si el agente de Azure AD Connect Health afectado puede cargar datos en el servicio de Azure AD Connect Health ejecutando el siguiente comando de PowerShell:

```powershell
Test-AzureADConnectHealthConnectivity -Role ADFS
```

El parámetro de rol tiene actualmente los siguientes valores:

* ADFS
* Sync
* ADDS

> [!NOTE]
> Para poder utilizar la herramienta de conectividad, primero debe registrar el agente. Si no puede completar el registro del agente, asegúrese de que cumple todos los [requisitos](how-to-connect-health-agent-install.md#requirements) para Azure AD Connect Health. La conectividad se prueba de forma predeterminada durante el registro del agente.
>
>

## <a name="next-steps"></a>Pasos siguientes

Consulte los artículos relacionados siguientes:

* [Azure AD Connect Health](./whatis-azure-ad-connect.md)
* [Operaciones de Azure AD Connect Health](how-to-connect-health-operations.md)
* [Uso de Azure AD Connect Health con AD FS](how-to-connect-health-adfs.md)
* [Uso de Azure AD Connect Health para sincronización](how-to-connect-health-sync.md)
* [Uso de Azure AD Connect Health con Azure AD DS](how-to-connect-health-adds.md)
* [Preguntas más frecuentes de Azure AD Connect Health](reference-connect-health-faq.md)
* [Historial de versiones de Azure AD Connect Health](reference-connect-health-version-history.md)
