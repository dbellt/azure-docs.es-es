---
title: Incorporación de los servidores locales del entorno virtual de VMware en Azure Arc
description: Incorporación de los servidores locales del entorno virtual de VMware en Azure Arc
author: deseelam
ms.author: deseelam
ms.manager: bsiva
ms.topic: how-to
ms.date: 04/27/2021
ms.openlocfilehash: 27b1253b2d2808e01e5dae542e82211d96add216
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108281084"
---
# <a name="onboard-on-premises-servers-in-vmware-virtual-environment-to-azure-arc"></a>Incorporación de los servidores locales del entorno virtual de VMware en Azure Arc   

En este artículo se describe cómo incorporar máquinas virtuales de VMware locales en Azure Arc para la administración de Azure mediante la herramienta de detección y evaluación de Azure Migrate. 

Azure Arc permite administrar el entorno de TI híbrido con un único panel al ampliar la experiencia de administración de Azure a los servidores locales que no son candidatos ideales para la migración. [Más información](https://docs.microsoft.com/azure/azure-arc/servers/overview) sobre Azure Arc 

## <a name="before-you-get-started"></a>Antes de comenzar

- [Consulte los requisitos](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#prerequisites) para detectar los servidores que se ejecutan en el entorno de VMware con la herramienta de detección y evaluación de Azure Migrate.  
- Prepare [VMware vCenter](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#prepare-vmware) para su uso y revise los [requisitos de VMware](migrate-support-matrix-vmware.md#vmware-requirements) para realizar el inventario de software. El inventario de software debe completarse para empezar a incorporar los servidores detectados a Azure Arc.   
- Revise los [requisitos de detección de aplicaciones](migrate-support-matrix-vmware.md#application-discovery-requirements) antes de iniciar el inventario de software en los servidores. Los servidores de Windows deben tener instalada la versión 3.0 de PowerShell u otra posterior.
- Asegúrese de comprobar los [requisitos previos para Azure Arc](https://docs.microsoft.com/azure/azure-arc/servers/agent-overview#prerequisites) y revise las consideraciones siguientes:
    - La incorporación a Azure Arc solo se puede iniciar una vez realizados el inventario de software y la detección de vCenter Server. El inventario de software puede tardar hasta 6 horas en completarse después de que se haya activado.
    -  El [agente de máquina híbrida conectada para Azure Arc](https://docs.microsoft.com/azure/azure-arc/servers/learn/quick-enable-hybrid-vm) se instalará en los servidores detectados durante el proceso de incorporación de Arc. Asegúrese de proporcionar credenciales con permisos de administrador en los servidores para instalar y configurar el agente. En Linux, proporcione la cuenta raíz, y en Windows, proporcione una cuenta que pertenezca al grupo de administradores locales. 
    - Compruebe que los servidores ejecutan un [sistema operativo compatible](https://docs.microsoft.com/azure/azure-arc/servers/agent-overview#supported-operating-systems).
    - Asegúrese de que a la cuenta de Azure se le ha concedido una asignación a los [roles de Azure requeridos](https://docs.microsoft.com/azure/azure-arc/servers/agent-overview#required-permissions).
    - Asegúrese de que las [direcciones URL necesarias](https://docs.microsoft.com/azure/azure-arc/servers/agent-overview#networking-configuration) no se bloquean si los servidores detectados se conectan mediante un firewall o servidor proxy para comunicarse a través de Internet.
    - Revise las [regiones admitidas](https://docs.microsoft.com/azure/azure-arc/servers/overview#supported-regions) para Azure Arc. 
    - Los servidores habilitados para Azure Arc admiten hasta 5000 instancias de máquina en un grupo de recursos.


## <a name="set-up-the-azure-migrate-project"></a>Configuración de un proyecto de Azure Migrate  

1. Antes de empezar, prepare la [cuenta de usuario de Azure](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#prepare-an-azure-user-account) y compruebe que tiene los [roles necesarios](./create-manage-projects.md#verify-permissions) en la suscripción para crear los recursos requeridos por Azure Migrate. 
2. [Use este artículo](https://docs.microsoft.com/azure/migrate/create-manage-projects) para configurar un nuevo proyecto de Azure Migrate que tenga incorporada la herramienta de detección y evaluación de Azure Migrate.  

    > [!Note]
    > También puede usar un proyecto de Migrate existente e incorporar el inventario de servidores detectados a Azure Arc. Para ello, inicie el administrador de configuración del dispositivo desde el servidor del dispositivo y asegúrese de que los servicios se han actualizado a las últimas versiones. [Más información](https://docs.microsoft.com/azure/migrate/migrate-appliance#appliance-upgrades) <br/> <br/> A continuación, [siga estas instrucciones](#onboard-to-azure-arc) para incorporar los servidores.  

## <a name="deploy-and-register-the-azure-migrate-appliance"></a>Implementación y registro del dispositivo de Azure Migrate 

La herramienta Azure Migrate: Discovery and assessment usa un dispositivo de Azure Migrate ligero. El dispositivo de Azure Migrate detecta los servidores y envía los metadatos de configuración y rendimiento a Azure Migrate. 

Antes de configurar el dispositivo, haga lo siguiente: 

1. [Revise](migrate-appliance.md#appliance---vmware) los requisitos para implementar el dispositivo de Azure Migrate.
2. Revise las direcciones URL de Azure a las que el dispositivo necesitará acceder en las nubes [públicas](migrate-appliance.md#public-cloud-urls) y [gubernamentales](migrate-appliance.md#government-cloud-urls). 
3. Tenga en cuenta los [requisitos de acceso a puertos](migrate-support-matrix-vmware.md#port-access-requirements) del dispositivo. 


Después,

- Siga este artículo para [configurar el dispositivo de Azure Migrate](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#set-up-the-appliance) para iniciar la detección de vCenter Server. Para implementar el dispositivo, puede descargar e importar una plantilla OVA a VMware para crear un servidor que se ejecute en vCenter Server.  
- Después de implementar el dispositivo, debe registrarlo en el proyecto antes de iniciar la detección. Siga [estas instrucciones](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#register-the-appliance-with-azure-migrate) para registrar el dispositivo. 

## <a name="configure-the-appliance-and-start-discovery"></a>Configuración del dispositivo e inicio de la detección  

Use [este artículo](https://docs.microsoft.com/azure/migrate/tutorial-discover-vmware#start-continuous-discovery) para configurar el dispositivo de Azure Migrate e iniciar la detección de vCenter Server. 

Al configurar el dispositivo para la detección, debe especificar los detalles en el administrador de configuración del dispositivo:

- Los detalles de la instancia de vCenter Server a la que se quiere conectar.  
- Las credenciales de vCenter Server en el ámbito para detectar los servidores del entorno de VMware. 
- Las credenciales del servidor con los permisos de administrador. La incorporación a Azure Arc requiere credenciales con permisos de administrador en el servidor para instalar y configurar el agente de máquina híbrida conectada para Azure Arc. [Obtenga más información](add-server-credentials.md) sobre cómo proporcionar credenciales y cómo administrarlas.

Una vez completada correctamente la detección, los servidores detectados tardan unos 15 minutos en aparecer en el portal.


## <a name="onboard-to-azure-arc"></a>Incorporación a Azure Arc

>[!Important]
>El inventario de software debe completarse antes de incorporar los servidores detectados a Azure Arc. 

Una vez completada la detección de vCenter Server, el inventario de software (detección de las aplicaciones instaladas) se iniciará automáticamente. Durante el inventario de software, las credenciales de servidor proporcionadas se iterarán y validarán en los servidores detectados. Puede iniciar la incorporación una vez que se haya completado el inventario de software y se hayan asignado las credenciales. El inventario de software puede tardar hasta 6 horas en completarse después de que se haya activado.  
1. Vaya al panel **Onboard to Azure Arc** (Incorporar a Azure Arc). 

    ![Incorporación a Arc](./media/onboard-to-azure-arc-with-azure-migrate/azure-arc-onboarding-panel-after-being-enabled.png)

2. Proporcione la **suscripción** y el **grupo de recursos** donde quiere que se administren los servidores en Azure.

3. En la lista desplegable **Región**, seleccione la región de Azure en la que se almacenarán los metadatos de los servidores.

4. Proporcione los detalles de la **entidad de servicio de Azure Active Directory** para la incorporación a gran escala. Revise este artículo para [crear una entidad de servicio mediante Azure Portal o Azure PowerShell](https://docs.microsoft.com/azure/azure-arc/servers/onboard-service-principal#create-a-service-principal-for-onboarding-at-scale). <br/>

    Se requieren las entradas siguientes:
    - **Id. de directorio (inquilino)** : [identificador único (GUID)](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-tenant-and-app-id-values-for-signing-in) que representa la instancia dedicada de Azure AD. 
    - **Id. de aplicación (cliente)** : [identificador único (GUID)](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#get-tenant-and-app-id-values-for-signing-in) que representa el identificador de aplicación de la entidad de servicio.
    - **Secreto de la entidad de servicio (secreto de aplicación)** : [secreto de cliente](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#option-2-create-a-new-application-secret) para la autenticación basada en contraseña. 
    
5. _Opcional_: proporcione la **dirección IP del servidor proxy** o el nombre y el **número de puerto** si los servidores detectados requieren un servidor proxy para conectarse a Internet. Escriba el valor con el formato `http://<proxyURL>:<proxyport>`. Este servidor proxy usado por los servidores detectados puede ser distinto al requerido por el servidor del dispositivo para conectarse a Internet (proporcionado en la sección de requisitos previos del administrador de configuración del dispositivo).   

    > [!Note]
    > No se admite la configuración de la autenticación del proxy. 

6. Seleccione **Iniciar incorporación** para iniciar el proceso de incorporación a Azure Arc. El proceso de incorporación puede tardar un tiempo. Una vez completada la incorporación, se le mostrará el estado de finalización y un informe de estado de incorporación detallado.    

    ![Informe de incorporación a Arc](./media/onboard-to-azure-arc-with-azure-migrate/onboarding-completion-report.png)

    > [!Note]
    > Si el inicio de sesión expira, seleccione **Iniciar sesión de nuevo**. A continuación, se abrirá una ventana modal con el código del dispositivo. Seleccione **Copiar código e iniciar sesión** para copiar el código del dispositivo y abrir un símbolo del sistema de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.

7. Una vez completada la incorporación, vaya a la [página principal de Azure Arc](https://portal.azure.com/#blade/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/servers) para ver y administrar los servidores incorporados.   

8. Vea el informe de incorporación detallado para conocer el estado de incorporación de los servidores y tomar las medidas adecuadas. 

> [!Note]
> El dispositivo de Azure Migrate iniciará sesiones de WinRM para ejecutar el script de incorporación a Azure Arc. Asegúrese de que no hay ninguna configuración que restrinja el acceso al servicio WinRM en los servidores de destino.    


## <a name="next-steps"></a>Pasos siguientes 

- Para obtener más información y detalles del error, revise el informe de incorporación detallado. Resuelva los errores, si los hay, para incorporar correctamente los servidores. 
- Puede seleccionar **Iniciar incorporación** para volver a ejecutar el proceso de incorporación a Azure Arc. La incorporación se realizará para todos los servidores recién detectados y los servidores que no se pudieron incorporar correctamente en la ejecución anterior.   

## <a name="troubleshooting-azure-arc-onboarding-errors"></a>Solución de los errores de incorporación a Azure Arc

Si recibe un error al realizar la incorporación a Azure Arc mediante el dispositivo de Azure Migrate, la siguiente sección puede ayudarle a identificar la posible causa y los pasos sugeridos para resolver el problema. 

Si no ve el código de error que aparece a continuación o si el código de error comienza por **_AZCM_**, consulte [esta guía para solucionar problemas de Azure Arc](https://docs.microsoft.com/azure/azure-arc/servers/troubleshoot-agent-onboard).

### <a name="error-60001---unabletoconnecttophysicalserver"></a>Error 60001: UnableToConnectToPhysicalServer  

**Causas posibles:**  
No se cumplen los [requisitos previos](https://go.microsoft.com/fwlink/?linkid=2134728) para conectarse al servidor o hay problemas de red para conectarse al servidor, como por ejemplo, alguna configuración del proxy.

**Acciones recomendadas**   
- Asegúrese de que el servidor cumple los requisitos previos para la detección y evaluación, como se documenta [aquí](https://go.microsoft.com/fwlink/?linkid=2134728). 
- Asegúrese de que ha elegido el método de autenticación correcto en el dispositivo para conectarse al servidor. 
   > [!Note] 
   > Azure Migrate admite la autenticación basada en claves SSH y en contraseñas para los servidores Linux.
- Si el problema persiste, envíe un caso de soporte técnico de Microsoft, donde debe proporcionar el identificador de la máquina del dispositivo (disponible en el pie de página del administrador de configuración del dispositivo).    

### <a name="error-60002---invalidservercredentials"></a>Error 60002: InvalidServerCredentials  

**Causas posibles:**  
No se puede establecer la conexión con el servidor. Ha proporcionado credenciales incorrectas en el dispositivo o las credenciales proporcionadas anteriormente han expirado.

**Acciones recomendadas**  
- Asegúrese de que ha proporcionado las credenciales correctas para el servidor en el dispositivo. Puede comprobarlo intentando conectarse al servidor con esas credenciales.
- Si las credenciales agregadas son incorrectas o han expirado, edite las credenciales en el dispositivo y vuelva a validar los servidores agregados. Si la validación se realiza correctamente, se resuelve el problema.
- Si el problema persiste, envíe un caso de soporte técnico de Microsoft, donde debe proporcionar el identificador de la máquina del dispositivo (disponible en el pie de página del administrador de configuración del dispositivo).

### <a name="error-60005---sshoperationtimeout"></a>Error 60005: SSHOperationTimeout  

**Causas posibles:**  
- La operación tardó más de lo esperado debido a problemas de latencia de red o a la falta de las últimas actualizaciones en el servidor. 

**Acciones recomendadas**  
- Asegúrese de que el servidor afectado tenga instaladas las últimas actualizaciones del kernel y del sistema operativo.
- Asegúrese de que no hay latencia de red entre el dispositivo y el servidor. Se recomienda que el dispositivo y el servidor de origen estén en el mismo dominio para evitar problemas de latencia.
- Conéctese al servidor afectado desde el dispositivo y ejecute los comandos [documentados aquí](https://go.microsoft.com/fwlink/?linkid=2152600) para comprobar si devuelven datos nulos o vacíos.
- Si el problema persiste, envíe un caso de soporte técnico de Microsoft, donde debe proporcionar el identificador de la máquina del dispositivo (disponible en el pie de página del administrador de configuración del dispositivo).  

### <a name="error-60108---softwareinventorycredentialnotassociated"></a>Error 60108: SoftwareInventoryCredentialNotAssociated  

**Causas posibles:**  
- No se encontraron credenciales asociadas con el servidor.

**Acciones recomendadas**  
- El inventario de software debe completarse para empezar a incorporar los servidores detectados a Azure Arc. [Más información](https://docs.microsoft.com/azure/migrate/how-to-discover-applications#add-credentials-and-initiate-discovery)
- Asegúrese de que las credenciales proporcionadas en el administrador de configuración del dispositivo son válidas y de que se puede acceder al servidor con las credenciales.
- Vuelva al administrador de configuración del dispositivo para proporcionar otro conjunto de credenciales o editar uno existente.  

### <a name="error-60109---arcosnotsupported"></a>Error 60109: ArcOsNotSupported  

**Causas posibles:**  
- El servidor hospeda un sistema operativo no compatible con la incorporación a Azure Arc.

**Acciones recomendadas**  
- [Consulte los sistemas operativos compatibles](https://docs.microsoft.com/azure/azure-arc/servers/agent-overview#supported-operating-systems) con Azure Arc. 
 
### <a name="error-10002---scriptexecutiontimedoutonvm"></a>Error 10002: ScriptExecutionTimedOutOnVm  

**Causas posibles:**  
- La tarea de incorporación no se ha completado a tiempo. Por tanto, la ejecución tardó más de lo esperado. 

**Acciones recomendadas**  
- El problema debería resolverse automáticamente en unos minutos. Si el problema persiste, póngase en contacto con el Soporte técnico de Microsoft.  
 
### <a name="error-50000---accessdenied"></a>Error 50000: AccessDenied 

**Causas posibles:**  
- No se pudo completar la operación porque el acceso al servidor estaba prohibido. La cuenta de usuario proporcionada en el dispositivo para acceder al servidor no tiene los permisos necesarios o las credenciales son incorrectas. Código de error de WinRM: 0x80070005

**Acciones recomendadas**  
- Valide las posibles causas y vuelva a intentar la operación. Si el problema persiste, póngase en contacto con el soporte técnico.

### <a name="error-9609516000960078---nullresultunhandledexceptionserverunknownerrorunhandlederror"></a>Error 960/951/60009/60078: NullResult/UnhandledException/ServerUnknownError/UnhandledError

**Causas posibles:**  
- Error en la operación debido a un error interno. 

**Acciones recomendadas**  
- Vuelva a intentar la operación más tarde. Si el problema persiste, póngase en contacto con soporte técnico y proporcione el identificador de la máquina del dispositivo (disponible en el **pie de página** del administrador de configuración del dispositivo).
