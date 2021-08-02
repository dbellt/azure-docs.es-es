---
title: Creación de una instancia de Integration Runtime autohospedada
description: Aprenda a crear un entorno de ejecución de integración autohospedado en Azure Data Factory, lo que permite que las factorías de datos accedan a almacenes de datos en una red privada.
ms.service: data-factory
ms.topic: conceptual
author: lrtoyou1223
ms.author: lle
ms.date: 02/10/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 92e7f2af175182886dbc5904c5a50b485ca87d64
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110681215"
---
# <a name="create-and-configure-a-self-hosted-integration-runtime"></a>Creación y configuración de un entorno de ejecución de integración autohospedado

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

El entorno de ejecución de integración (IR) es la infraestructura de proceso que Azure Data Factory usa para proporcionar funcionalidades de integración de datos en distintos entornos de red. Para más información acerca del entorno de ejecución de integración, consulte [Introducción al entorno de ejecución de integración](concepts-integration-runtime.md).

Un entorno de ejecución de integración autohospedado puede ejecutar actividades de copia entre un almacén de datos en la nube y un almacén de datos en una red privada. También puede distribuir las siguientes actividades de transformación frente a los recursos de proceso en una red local o en Azure Virtual Network. La instalación de un entorno de ejecución de integración autohospedado debe realizarse en una máquina local o en una máquina virtual dentro de una red privada.  

En este artículo se describe cómo crear y configurar un IR autohospedado.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="considerations-for-using-a-self-hosted-ir"></a>Consideraciones a la hora de usar un IR autohospedado

- Puede utilizar un solo entorno de ejecución de integración autohospedado para varios orígenes de datos locales. También puede compartirlo con otra factoría de datos con el mismo inquilino de Azure Active Directory (Azure AD). Para más información, consulte [Uso compartido de un entorno de ejecución de integración autohospedado](./create-shared-self-hosted-integration-runtime-powershell.md).
- En cada máquina solo puede instalar una instancia del entorno de ejecución de integración autohospedado. Si tiene dos factorías de datos que necesitan obtener acceso a los orígenes de datos locales, use la [característica de uso compartido del IR autohospedado](./create-shared-self-hosted-integration-runtime-powershell.md) para compartir el IR autohospedada, o instale el IR autohospedado en dos equipos locales, uno para cada fábrica de datos.  
- No es preciso que el entorno de ejecución de integración autohospedado se encuentre en la misma máquina que el origen de datos. Sin embargo, cuanto más cerca estén ambos, menos tiempo necesitará el primero para conectarse al segundo. Le recomendamos que instale el entorno de ejecución de integración autohospedado en una máquina diferente de la que hospeda el origen de datos local. Si el entorno de ejecución de integración autohospedado y el origen de datos están en máquinas diferentes, el primero no compite con el segundo por recursos.
- Puede tener varios entornos de ejecución de integración autohospedados en diferentes equipos que se conecten al mismo origen de datos local. Por ejemplo, si tiene dos entornos de ejecución de integración autohospedados que sirven a dos factorías de datos el mismo origen de datos local puede estar registrado en ambas factorías de datos.
- Use un entorno de ejecución de integración autohospedado para admitir la integración de datos en Azure Virtual Network.
- Considere el origen de datos como un origen de datos local, que está detrás de un firewall, incluso cuando use Azure ExpressRoute. Use el entorno de ejecución de integración autohospedado para conectarse al origen de datos.
- Use el entorno de ejecución de integración autohospedado aunque el almacén de datos esté en la nube en una máquina virtual de infraestructura como servicio (IaaS) de Azure.
- Las tareas pueden generar error en un entorno de ejecución de integración autohospedado que esté instalado en un equipo con Windows Server para el que está habilitado el cifrado compatible con FIPS. Para solucionar este problema, tiene dos opciones: almacenar los valores de las credenciales y los secretos en una instancia de Azure Key Vault o deshabilitar el cifrado compatible con FIPS en el servidor. Para deshabilitar el cifrado compatible con FIPS, cambie el valor de la subclave del registro siguiente de 1 (habilitado) a 0 (deshabilitado): `HKLM\System\CurrentControlSet\Control\Lsa\FIPSAlgorithmPolicy\Enabled`. Si usa el [entorno de ejecución de integración autohospedado como proxy en el entorno de integración de SSIS](./self-hosted-integration-runtime-proxy-ssis.md), se puede habilitar el cifrado conforme a FIPS y se usará al mover datos de un entorno local a Azure Blob Storage como área de almacenamiento temporal.

## <a name="command-flow-and-data-flow"></a>Flujo de comandos y flujo de datos

Cuando mueve datos entre un entorno local y la nube, la actividad utiliza un entorno de ejecución de integración autohospedado para transferir los datos entre un origen de datos local y la nube.

A continuación se muestra un resumen de alto nivel de los pasos del flujo de datos para copiar con un IR autohospedado:

![Información general de alto nivel del flujo de datos](media/create-self-hosted-integration-runtime/high-level-overview.png)

1. Un desarrollador de datos crea primero un entorno de ejecución de integración autohospedado en una instancia de Azure Data Factory mediante Azure Portal o el cmdlet de PowerShell.  A continuación, el desarrollador de datos crea un servicio vinculado para un almacén de datos local mediante la especificación de la instancia del entorno de ejecución de integración autohospedado que debe usar el servicio para conectarse a los almacenes de datos.

2. El nodo del entorno de ejecución de integración autohospedado cifra las credenciales mediante la interfaz de programación de aplicaciones de protección de datos de Windows (DPAPI) y las guarda localmente. Si se establecen varios nodos para la alta disponibilidad, las credenciales se sincronizan de nuevo en otros nodos. Cada nodo cifra las credenciales mediante DPAPI y las almacena localmente. La sincronización de credenciales es transparente para el desarrollador de datos y la controla el IR autohospedado.

3. Azure Data Factory se comunica con el entorno de ejecución de integración autohospedado para programar y administrar trabajos. La comunicación se realiza a través de un canal de control que usa una conexión compartida de [Azure Relay](../azure-relay/relay-what-is-it.md#wcf-relay). Cuando es necesario ejecutar un trabajo de actividad, Data Factory pone en cola la solicitud junto con la información de credenciales. Sucede esto en caso de que las credenciales aún no estén almacenadas en el entorno de ejecución de integración autohospedado. El entorno de ejecución de integración autohospedado inicia el trabajo después de sondear la cola.

4. El entorno de ejecución de integración autohospedado copia datos entre un almacenamiento local y un almacenamiento en la nube. La dirección de la copia depende de la configuración de la actividad de copia en la canalización de los datos. En este paso, el entorno de ejecución de integración autohospedado se comunica directamente con servicios de almacenamiento basados en la nube, como Azure Blob Storage, a través de un canal HTTPS seguro.

## <a name="prerequisites"></a>Prerrequisitos

- Las versiones compatibles de Windows son:
  - Windows 8.1
  - Windows 10
  - Windows Server 2012
  - Windows Server 2012 R2
  - Windows Server 2016
  - Windows Server 2019

No se admite la instalación del entorno de ejecución de integración autohospedado en un controlador de dominio.

- El entorno de ejecución de integración autohospedado requiere un sistema operativo de 64 bits con .NET Framework 4.7.2 o posterior. Consulte [Requisitos de sistema de .NET Framework](/dotnet/framework/get-started/system-requirements) para más información.
- La configuración mínima recomendada para la máquina del entorno de ejecución de integración autohospedado es un procesador de 2 GHz con 4 núcleos, 8 GB de RAM y 80 GB de espacio disponible en disco duro. Para obtener información detallada sobre los requisitos del sistema, consulte la [descarga](https://www.microsoft.com/download/details.aspx?id=39717).
- Si el equipo host está en hibernación, el entorno de ejecución de integración autohospedado no responde a las solicitudes de datos. Configure un plan de energía adecuado en el equipo antes de instalar el entorno de ejecución de integración autohospedado. Si la máquina está configurada para hibernar, se mostrará un mensaje en el instalador del entorno de ejecución de integración autohospedado.
- Debe ser administrador de la máquina para instalar y configurar correctamente el entorno de ejecución de integración autohospedado.
- Las ejecuciones de la actividad de copia se producen con una frecuencia concreta. El uso del procesador y la RAM en la máquina sigue el mismo patrón en los momentos de máxima actividad y en los de inactividad. El uso de recursos también depende en gran medida de la cantidad de datos que se mueven. Cuando hay varios trabajos de copia en curso, puede ver que el uso de los recursos aumenta durante las horas pico.
- Puede que las tareas produzcan errores durante la extracción de datos en formatos Parquet, ORC o Avro. Para obtener más información sobre Parquet, consulte [Formato Parquet en Azure Data Factory](./format-parquet.md#using-self-hosted-integration-runtime). La creación del archivo se ejecuta en la máquina de integración autohospedada. Para que funcione según lo esperado, la creación del archivo requiere los siguientes requisitos previos:
  - Paquete [redistribuible de Visual C++ 2010](https://download.microsoft.com/download/3/2/2/3224B87F-CFA0-4E70-BDA3-3DE650EFEBA5/vcredist_x64.exe) (x64)
  - Versión 8 de Java Runtime (JRE) de un proveedor de JRE, como [AdoptOpenJDK](https://adoptopenjdk.net/). Asegúrese de que la variable de entorno `JAVA_HOME` esté establecida en la carpeta JRE (y no solo en la carpeta JDK).

## <a name="setting-up-a-self-hosted-integration-runtime"></a>Configuración de un entorno de ejecución de integración autohospedado

Para crear y configurar un entorno de ejecución de integración autohospedado, use los procedimientos siguientes.

### <a name="create-a-self-hosted-ir-via-azure-powershell"></a>Creación de un IR autohospedado mediante Azure PowerShell

1. Puede usar Azure PowerShell para esta tarea. Este es un ejemplo:

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $selfHostedIntegrationRuntimeName -Type SelfHosted -Description "selfhosted IR description"
    ```
  
2. [Descargue](https://www.microsoft.com/download/details.aspx?id=39717) e instale el entorno de ejecución de integración autohospedado en un equipo.

3. Recupere la clave de autenticación y registre el entorno de ejecución de integración autohospedado con la clave. Este es un ejemplo con PowerShell:

    ```powershell

    Get-AzDataFactoryV2IntegrationRuntimeKey -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $selfHostedIntegrationRuntimeName  

    ```

### <a name="create-a-self-hosted-ir-via-azure-data-factory-ui"></a>Creación de un IR autohospedado mediante la interfaz de usuario de Azure Data Factory

Use los pasos siguientes para crear un IR autohospedado mediante la interfaz de usuario de Azure Data Factory.

1. En la página **Comencemos** de la interfaz de usuario de Azure Data Factory, seleccione la [pestaña Administrar](./author-management-hub.md) en el panel izquierdo.

   ![Botón Administrar de la página principal](media/doc-common-process/get-started-page-manage-button.png)

1. Seleccione **Entornos de ejecución de integración** en el panel izquierdo y, a continuación, seleccione **+ Nuevo**.

   ![Creación de una instancia de Integration Runtime](media/doc-common-process/manage-new-integration-runtime.png)

1. En la página **Configuración de Integration Runtime**, seleccione **Azure, Self-Hosted** (Azure, autohospedado) y, luego, seleccione **Continuar**.

1. En la página siguiente, seleccione **Self-Hosted** (Autohospedado) para crear un IR autohospedado y, luego, seleccione **Continuar**.
   ![Creación de un IR autohospedado](media/create-self-hosted-integration-runtime/new-selfhosted-integration-runtime.png)

1. Escriba un nombre para el entorno de ejecución de integración y seleccione **Create** (Crear).

1. En la página **Configuración de Integration Runtime**, seleccione el vínculo bajo la **Opción 1** para abrir la configuración rápida en el equipo. O siga los pasos descritos en **Opción 2** para realizar la configuración manualmente. Las siguientes instrucciones se basan en el método de configuración manual:

   ![Configuración de Integration Runtime](media/create-self-hosted-integration-runtime/integration-runtime-setting-up.png)

    1. Copie y pegue la clave de autenticación. Seleccione **Download and install integration runtime** (Descargar e instalar Integration Runtime).

    1. Descargue el entorno de ejecución de integración autohospedado en una máquina Windows local. Ejecute al programa de instalación.

    1. En la página **Registro de Integration Runtime (autohospedado)** , pegue la clave guardada anteriormente y seleccione **Registrar**.

       ![Registro de Integration Runtime](media/create-self-hosted-integration-runtime/register-integration-runtime.png)

    1. En la página **Nuevo nodo de Integration Runtime (autohospedado)** , seleccione **Finalizar**.

1. Verá la siguiente ventana cuando el entorno de ejecución de integración autohospedado se haya registrado correctamente:

    ![El registro se completó correctamente.](media/create-self-hosted-integration-runtime/registered-successfully.png)

### <a name="set-up-a-self-hosted-ir-on-an-azure-vm-via-an-azure-resource-manager-template"></a>Configuración de un IR autohospedado en una VM de Azure mediante una plantilla de Azure Resource Manager

Puede automatizar la configuración del IR autohospedado en una máquina virtual de Azure mediante la [plantilla Crear IR autohospedado](https://github.com/Azure/azure-quickstart-templates/tree/master/101-vms-with-selfhost-integration-runtime). La plantilla proporciona una forma fácil de tener un IR autohospedado totalmente funcional dentro de Azure Virtual Network. El IR tiene características de escalabilidad y alta disponibilidad siempre que configure el número de nodos en dos o más.

### <a name="set-up-an-existing-self-hosted-ir-via-local-powershell"></a>Configuración de un IR autohospedado existente mediante PowerShell local

Puede usar la línea de comandos para configurar o administrar un IR autohospedado. Este uso puede resultar especialmente útil para automatizar la instalación y el registro de nodos de IR autohospedado.

Dmgcmd.exe se incluye en la instalación autohospedada. Normalmente se ubica en la carpeta C:\Archivos de programa\Microsoft Integration Runtime\4.0\Shared\. Esta aplicación admite varios parámetros y puede invocarse a través de una línea de comandos mediante scripts por lotes para la automatización.

Use la aplicación de la manera siguiente:

```powershell
dmgcmd ACTION args...
```

Estos son los detalles de las acciones y los argumentos de la aplicación: 

|ACTION|args|Descripción|
|------|----|-----------|
|`-rn`,<br/>`-RegisterNewNode`|"`<AuthenticationKey>`" ["`<NodeName>`"]|Registre un nodo del entorno de ejecución de integración autohospedado con la clave de autenticación y el nombre de nodo especificados.|
|`-era`,<br/>`-EnableRemoteAccess`|"`<port>`" ["`<thumbprint>`"]|Permite habilitar el acceso remoto al nodo actual para configurar un clúster de alta disponibilidad. También permite la configuración de credenciales directamente en el IR autohospedado sin necesidad de acceder a través de Azure Data Factory. Para ello, use el cmdlet **New-AzDataFactoryV2LinkedServiceEncryptedCredential** desde una máquina remota en la misma red.|
|`-erac`,<br/>`-EnableRemoteAccessInContainer`|"`<port>`" ["`<thumbprint>`"]|Permite habilitar el acceso remoto al nodo actual cuando el nodo se ejecuta en un contenedor.|
|`-dra`,<br/>`-DisableRemoteAccess`||Permite deshabilitar el acceso remoto al nodo actual. Es necesario obtener acceso remoto para realizar la configuración de varios nodos. El cmdlet de PowerShell **New-AzDataFactoryV2LinkedServiceEncryptedCredential** funcionará incluso cuando el acceso remoto esté deshabilitado. Este comportamiento se produce siempre y cuando el cmdlet se ejecute en la misma máquina que el nodo de IR autohospedado.|
|`-k`,<br/>`-Key`|"`<AuthenticationKey>`"|Permite sobrescribir o actualizar la clave de autenticación anterior. Debe tener cuidado con esta acción. El nodo de IR autohospedado anterior puede desconectarse si la clave es un nuevo entorno de ejecución de integración.|
|`-gbf`,<br/>`-GenerateBackupFile`|"`<filePath>`" "`<password>`"|Permite generar un archivo de copia de seguridad del nodo actual. El archivo de copia de seguridad incluye la clave del nodo y las credenciales del almacén de datos.|
|`-ibf`,<br/>`-ImportBackupFile`|"`<filePath>`" "`<password>`"|Permite restaurar el nodo desde un archivo de copia de seguridad.|
|`-r`,<br/>`-Restart`||Permite reiniciar el servicio de host del entorno de ejecución de integración autohospedado.|
|`-s`,<br/>`-Start`||Permite iniciar el servicio de host del entorno de ejecución de integración autohospedado.|
|`-t`,<br/>`-Stop`||Permite detener el servicio de host del entorno de ejecución de integración autohospedado.|
|`-sus`,<br/>`-StartUpgradeService`||Permite iniciar el servicio de actualización del entorno de ejecución de integración autohospedado.|
|`-tus`,<br/>`-StopUpgradeService`||Permite detener el servicio de actualización del entorno de ejecución de integración autohospedado.|
|`-tonau`,<br/>`-TurnOnAutoUpdate`||Permite activar la actualización automática del entorno de ejecución de integración autohospedado.|
|`-toffau`,<br/>`-TurnOffAutoUpdate`||Permite desactivar la actualización automática del entorno de ejecución de integración autohospedado.|
|`-ssa`,<br/>`-SwitchServiceAccount`|"`<domain\user>`" ["`<password>`"]|Permite configurar DIAHostService para que se ejecute como una cuenta nueva. Use la contraseña vacía "" para cuentas del sistema o cuentas virtuales.|

## <a name="install-and-register-a-self-hosted-ir-from-microsoft-download-center"></a>Instalación y registro de un IR autohospedado desde el Centro de descarga de Microsoft

1. Vaya a la [página de descarga del entorno de ejecución de integración Microsoft](https://www.microsoft.com/download/details.aspx?id=39717).
2. Seleccione **Descargar**, haga clic en la versión de 64 bits y seleccione **Siguiente**. La versión de 32 bits no se admite.
3. Ejecute el archivo de identidad administrada directamente o guárdelo en el disco duro para ejecutarlo más adelante.
4. En la ventana **principal**, seleccione un idioma y, después, seleccione **Siguiente**.
5. Acepte los términos de licencia del software de Microsoft y seleccione **Siguiente**.
6. Seleccione la **carpeta** en la que se va a instalar el entorno de ejecución de integración autohospedado y seleccione **Siguiente**.
7. En la página **Preparado para instalar**, seleccione **Instalar**.
8. Seleccione **Finalizar** para completar la instalación.
9. Para obtener la clave de autenticación, use PowerShell. Este es un ejemplo de PowerShell para recuperar la clave de autenticación:

    ```powershell
    Get-AzDataFactoryV2IntegrationRuntimeKey -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Name $selfHostedIntegrationRuntime
    ```

10. En la ventana **Registro de Integration Runtime (autohospedado)** de Microsoft Integration Runtime Configuration Manager que se ejecuta en la máquina, siga estos pasos:

    1. Pegue la clave de autenticación en el área de texto.

    2. Si lo desea, seleccione **Show authentication key** (Mostrar clave de autenticación) para ver el texto de la clave.

    3. Seleccione **Registrar**.

> [!NOTE]
> Hay notas de la versión disponibles en la misma [página de descarga del entorno de ejecución de integración de Microsoft](https://www.microsoft.com/download/details.aspx?id=39717).

## <a name="service-account-for-self-hosted-integration-runtime"></a>Cuenta de servicio para el entorno de ejecución de integración autohospedado

La cuenta de servicio de inicio de sesión predeterminada del entorno de ejecución de integración autohospedado es **NT SERVICE\DIAHostService**. Puede verla en **Servicios -> Servicio del entorno de ejecución -> Propiedades -> Inicio de sesión**.

![Cuenta de servicio para el entorno de ejecución de integración autohospedado](media/create-self-hosted-integration-runtime/shir-service-account.png)

Asegúrese de que la cuenta tenga permiso de inicio de sesión como servicio. De lo contrario, el entorno de ejecución de integración autohospedado no puede iniciarse correctamente. Puede comprobar el permiso en **Directiva de seguridad local -> Configuración de seguridad -> Directivas locales-> Asignación de permisos de usuario-> Iniciar sesión como servicio**

![Captura de pantalla de la directiva de seguridad local: asignación de derechos de usuario](media/create-self-hosted-integration-runtime/shir-service-account-permission.png)

![Captura de pantalla de la asignación de derechos de usuario Iniciar sesión como servicio](media/create-self-hosted-integration-runtime/shir-service-account-permission-2.png)

## <a name="notification-area-icons-and-notifications"></a>Iconos y notificaciones del área de notificación

Si mueve el cursor sobre el icono o el mensaje en el área de notificación, podrá ver los detalles del estado del entorno de ejecución de integración autohospedado.

![Notificaciones en el área de notificación](media/create-self-hosted-integration-runtime/system-tray-notifications.png)

## <a name="high-availability-and-scalability"></a>Alta disponibilidad y escalabilidad

Puede asociar un entorno de ejecución de integración autohospedado con varias máquinas locales o máquinas virtuales en Azure. Estas máquinas se llaman nodos. Puede tener hasta cuatro nodos asociados con un entorno de ejecución de integración autohospedado. Las ventajas de tener varios nodos en máquinas locales que tienen una puerta de enlace instalada para una puerta de enlace lógica son:

- Mayor disponibilidad del entorno de ejecución de integración autohospedado para que deje de ser el único punto de error de la solución de macrodatos o la integración de datos en la nube con Data Factory. Esta disponibilidad garantiza la continuidad al usar un máximo de cuatro nodos.
- Rendimiento mejorado durante el movimiento de datos entre almacenes de datos locales y en la nube. Obtenga más información sobre las [comparaciones de rendimiento](copy-activity-performance.md).

Puede asociar varios nodos mediante la instalación del software del entorno de ejecución de integración autohospedado desde el [Centro de descarga](https://www.microsoft.com/download/details.aspx?id=39717). A continuación, regístrelo mediante cualquiera de las claves de autenticación obtenidas del cmdlet **New-AzDataFactoryV2IntegrationRuntimeKey**, tal como se describe en el [tutorial](tutorial-hybrid-copy-powershell.md).

> [!NOTE]
> No es preciso crear un nuevo entorno de ejecución de integración autohospedado para asociar cada nodo. Puede instalar el entorno de ejecución de integración autohospedado en otro equipo y registrarlo con la misma clave de autenticación.

> [!NOTE]
> Antes de agregar otro nodo para aumentar la disponibilidad y escalabilidad, asegúrese de que la opción **Remote access to intranet** (Acceso remoto a la intranet) está habilitada en el primer nodo. Para ello, seleccione **Microsoft Integration Runtime Configuration Manager** > **Configuración** > **Remote access to intranet** (Acceso remoto a la intranet).

### <a name="scale-considerations"></a>Consideraciones de escala

#### <a name="scale-out"></a>Escalado horizontal

Cuando el uso del procesador sea alto y quede poca memoria disponible en el IR autohospedado, agregue un nuevo nodo para ayudar a escalar horizontalmente la carga en las máquinas. Si se producen errores en las actividades porque se agota su tiempo de espera o el nodo del IR autohospedado está sin conexión, será útil agregar un nodo a la puerta de enlace.

#### <a name="scale-up"></a>Escalado vertical

Cuando el procesador y la memoria RAM disponible no se usan correctamente, pero la ejecución de trabajos simultáneos alcanza los límites de un nodo, aumente el número de trabajos simultáneos que puede ejecutar un nodo para escalar verticalmente. También puede realizar el escalado verticalmente cuando se agota el tiempo de espera de las actividades porque el IR autohospedado está sobrecargado. Como se muestra en la siguiente imagen, puede aumentar la capacidad máxima de los nodos:  

![Aumento del número de trabajos simultáneos que se pueden ejecutar en un nodo](media/create-self-hosted-integration-runtime/scale-up-self-hosted-IR.png)

### <a name="tlsssl-certificate-requirements"></a>Requisitos del certificado TLS/SSL

Estos son los requisitos para el certificado TLS/SSL que se usa para proteger la comunicación entre los nodos del entorno de ejecución de integración:

- El certificado debe ser un certificado de confianza pública X509 v3. Se recomienda utilizar los certificados que emita alguna entidad de certificación (CA) de asociado pública.
- Todos los nodos del entorno de ejecución de integración deben confiar en este certificado.
- No se recomiendan los certificados de nombre alternativo del firmante (SAN) porque solo se usa el último elemento de SAN. Todos los demás elementos de SAN se omiten. Por ejemplo, si tiene un certificado de SAN cuyos nombres alternativos del firmante son **node1.domain.contoso.com** y **node2.domain.contoso.com**, solo puede usarlo en una máquina cuyo nombre de dominio completo (FQDN) es **node2.domain.contoso.com**.
- El certificado puede usar cualquier tamaño de clave compatible con Windows Server 2012 R2 para los certificados TLS/SSL.
- No se admiten certificados que utilicen claves CNG.  

> [!NOTE]
> Usos del certificado:
>
> - Para cifrar puertos en un nodo de IR autohospedado.
> - Para la comunicación de nodo a nodo para la sincronización de estado, que incluye la sincronización de credenciales de servicios vinculados entre nodos.
> - Cuando se usa un cmdlet de PowerShell para la configuración de credenciales del servicio vinculado desde una red local.
>
> Es conveniente usar este certificado si el entorno de la red privada no es seguro o si quiere proteger la comunicación entre nodos en la red privada.
>
> El movimiento de datos en tránsito desde un IR autohospedado y otros almacenes de datos siempre tiene lugar en un canal cifrado, independientemente de si este certificado está configurado o no.

## <a name="proxy-server-considerations"></a>Consideraciones acerca del servidor proxy

Si en su entorno de red corporativo se usa un servidor proxy para acceder a Internet, configure el entorno de ejecución de integración autohospedado para utilizar la configuración del proxy adecuada. Puede establecer el proxy durante la fase de registro inicial.

![Especificación del proxy](media/create-self-hosted-integration-runtime/specify-proxy.png)

Cuando se configura, el entorno de ejecución de integración autohospedado usa el servidor proxy para conectarse al origen y destino del servicio en la nube (que usan el protocolo HTTP o HTTPS). Por este motivo, debe seleccionar **Cambiar vínculo** durante la configuración inicial.

![Configuración del proxy](media/create-self-hosted-integration-runtime/set-http-proxy.png)

Hay tres opciones de configuración:

- **No utilizar proxy**: el entorno de ejecución de integración autohospedado no usa expresamente ningún proxy para conectarse a servicios en la nube.
- **Usar proxy del sistema**: el entorno Integration Runtime autohospedado utiliza la configuración de proxy de diahost.exe.config y diawp.exe.config. Si estos archivos no especifican ninguna configuración de proxy, el entorno de ejecución de integración autohospedado se conecta al servicio en la nube directamente sin pasar por ningún proxy.
- **Usar proxy personalizado**: establezca la configuración del proxy HTTP que se utilizará para el entorno de ejecución de integración autohospedado, en lugar de usar las configuraciones de diahost.exe.config y diawp.exe.config. Se requieren los valores de **Dirección** y **Puerto**. Los valores de **Nombre de usuario** y **Contraseña** son opcionales, y dependen de la configuración de la autenticación del proxy. Todas las configuraciones se cifran con DPAPI de Windows en el entorno de ejecución de integración autohospedado y se almacenan localmente en el equipo.

El servicio host del entorno de ejecución de integración se reinicia automáticamente después de guardar la configuración de proxy actualizada.

Tras registrar correctamente el entorno de ejecución de integración autohospedado, si quiere ver o actualizar la configuración de proxy, utilice Microsoft Integration Runtime Configuration Manager.

1. Inicie el **Administrador de configuración de Microsoft Integration Runtime**.
1. Seleccione la pestaña **Configuración**.
1. En **Proxy HTTP**, seleccione el vínculo **Cambiar** para abrir el cuadro de diálogo **Establecer proxy HTTP**.
1. Seleccione **Next** (Siguiente). En ese momento ve una advertencia en la que se le pide permiso para guardar la configuración del proxy y reiniciar el servicio de host del entorno de ejecución de integración.

Puede usar la herramienta Configuration Manager para ver y actualizar el proxy HTTP.

![Visualización y actualización del proxy](media/create-self-hosted-integration-runtime/view-proxy.png)

> [!NOTE]
> Si configura un servidor proxy con la autenticación NTLM, el servicio de host del entorno de ejecución de integración se ejecutará en la cuenta del dominio. Si más adelante cambia la contraseña de dicha cuenta, no olvide actualizar la configuración del servicio y reiniciarlo. Debido a este requisito, se recomienda acceder al servidor proxy con una cuenta de dominio dedicada que no requiera que se actualice la contraseña con frecuencia.

### <a name="configure-proxy-server-settings"></a>Configuración de un servidor proxy

Si selecciona la opción **Usar proxy del sistema** para el proxy HTTP, el entorno de ejecución de integración autohospedado utilizará la configuración de proxy de diahost.exe.config y diawp.exe.config. Cuando estos archivos no especifican ningún proxy, el entorno de ejecución de integración autohospedado se conecta al servicio en la nube directamente sin pasar por ningún proxy. En el procedimiento siguiente se proporcionan instrucciones para actualizar el archivo diahost.exe.config:

1. En el Explorador de archivos, cree una copia segura de C:\Archivos de programa\Microsoft Integration Runtime\4.0\Shared\diahost.exe.config como copia de seguridad del archivo original.
1. Abra el Bloc de notas ejecutándolo como administrador.
1. En el Bloc de notas, abra el archivo de texto C:\Archivos de programa\Microsoft Integration Runtime\4.0\Shared\diahost.exe.config.
1. Encuentre la etiqueta predeterminada de **system.net** como se muestra en el siguiente código:

    ```xml
    <system.net>
        <defaultProxy useDefaultCredentials="true" />
    </system.net>
    ```

    Después, puede agregar los detalles del servidor proxy tal y como se muestran en el ejemplo siguiente:

    ```xml
    <system.net>
        <defaultProxy enabled="true">
              <proxy bypassonlocal="true" proxyaddress="http://proxy.domain.org:8888/" />
        </defaultProxy>
    </system.net>
    ```

    La etiqueta de proxy permite propiedades adicionales para especificar la configuración requerida, como `scriptLocation`. Consulte [\<proxy\>Elemento (Configuración de red)](/dotnet/framework/configure-apps/file-schema/network/proxy-element-network-settings) para ver la sintaxis.

    ```xml
    <proxy autoDetect="true|false|unspecified" bypassonlocal="true|false|unspecified" proxyaddress="uriString" scriptLocation="uriString" usesystemdefault="true|false|unspecified "/>
    ```

1. Guarde el archivo de configuración en su ubicación original. Luego reinicie el servicio de host del entorno de ejecución de integración autohospedado, lo que aplica los cambios.

   Para reiniciar el servicio, use el applet de servicios del panel de control. O bien en Integration Runtime Configuration Manager, seleccione el botón **Detener servicio** y, después, seleccione **Iniciar servicio**.

   Si el servicio no se inicia, es probable que se deba a que se ha agregado una sintaxis de etiqueta XML incorrecta en el archivo de configuración de la aplicación que se ha editado.

> [!IMPORTANT]
> No olvide actualizar tanto diahost.exe.config como diawp.exe.config.

También tiene que asegurarse de que Microsoft Azure se encuentra en la lista de permitidos de su empresa. La lista de direcciones IP válidas de Azure se puede descargar del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=41653).

### <a name="possible-symptoms-for-issues-related-to-the-firewall-and-proxy-server"></a>Posibles síntomas de problemas relacionados con el firewall y el servidor proxy

Si ve mensajes de error como los siguientes, lo más probable es que se deba a una configuración incorrecta del firewall o el servidor proxy. Dicha configuración impide que el entorno de ejecución de integración autohospedado se conecte a Data Factory para autenticarse. Para asegurarse de que tanto el firewall como el servidor proxy están configurados correctamente, consulte la sección anterior.

- Al intentar registrar el entorno de ejecución de integración autohospedado, recibe el siguiente mensaje de error: "No se pudo registrar este nodo de Integration Runtime. Confirme que la clave de autenticación es válida y que el servicio host del servicio de integración se ejecuta en este equipo".
- Al abrir Integration Runtime Configuration Manager, se ve uno de estos dos estados, **Desconectado** o **Conectando**. Al ver los registros de eventos de Windows, en **Visor de eventos** > **Registros de aplicaciones y servicios** > **Microsoft Integration Runtime** aparecen mensajes de error como este:

  ```output
  Unable to connect to the remote server
  A component of Integration Runtime has become unresponsive and restarts automatically. Component name: Integration Runtime (Self-hosted).
  ```

### <a name="enable-remote-access-from-an-intranet"></a>Activación del acceso remoto desde una intranet

Si usa PowerShell para cifrar credenciales desde una máquina de la red que no sea la que tiene instalado el entorno de ejecución de integración autohospedado, puede habilitar la opción **Acceso remoto desde la intranet**. Si usa PowerShell para cifrar credenciales en la máquina en la que esté instalado el entorno de ejecución de integración autohospedado, no podrá habilitar el **Acceso remoto desde la intranet**.

Habilite el **Acceso remoto desde la intranet** para agregar otro nodo para que tanto la disponibilidad como la escalabilidad sean altas.  

Al ejecutar el entorno de ejecución de integración autohospedado versión 3.3 o posterior, el instalador del entorno de ejecución de integración autohospedado deshabilita de manera predeterminada la opción **Acceso remoto desde la intranet** en la máquina de dicho entorno.

Al usar un firewall de un asociado comercial u otros, puede abrir manualmente el puerto 8060 o el puerto configurado por el usuario. Si se presenta un problema de firewall durante la configuración del entorno de ejecución de integración autohospedado, use el comando siguiente para instalarlo sin configurar el firewall:

```cmd
msiexec /q /i IntegrationRuntime.msi NOFIREWALL=1
```

Si elige no abrir el puerto 8060 en la máquina del entorno de ejecución de integración autohospedado, use mecanismos que no sean la aplicación de Establecer credenciales para configurar las credenciales del almacén de datos. Por ejemplo, puede usar el cmdlet de PowerShell **New-AzDataFactoryV2LinkedServiceEncryptCredential**.

## <a name="ports-and-firewalls"></a>Puertos y firewalls

Estos son dos firewalls a considerar:

- El *firewall corporativo* que se ejecuta en el enrutador central de la organización.
- El *firewall de Windows* que está configurado como demonio en la máquina local en la que está instalado el entorno de ejecución de integración autohospedado.

![Firewalls](media/create-self-hosted-integration-runtime/firewall.png)

A nivel de firewall corporativo, es preciso configurar los siguientes dominios y puertos de salida:

[!INCLUDE [domain-and-outbound-port-requirements](./includes/domain-and-outbound-port-requirements-internal.md)]

En el nivel del firewall de Windows o nivel de máquina, normalmente estos puertos de salida están habilitados. Si no lo están, puede configurar los puertos y los dominios en la máquina del entorno de ejecución de integración autohospedado.

> [!NOTE]
> Ya que actualmente Azure Relay no admite la etiqueta de servicio, debe usar la etiqueta de servicio **AzureCloud** o **Internet** en las reglas de NSG para la comunicación con Azure Relay.
> Para comunicarse con Azure Data Factory, puede usar la etiqueta de servicio **DataFactoryManagement** en la configuración de la regla de NSG.

En función del origen y de los receptores, es posible que tenga que permitir más dominios y puertos de salida al firewall corporativo o al firewall de Windows.

[!INCLUDE [domain-and-outbound-port-requirements](./includes/domain-and-outbound-port-requirements-external.md)]

En algunas bases de datos en la nube, como Azure SQL Database y Azure Data Lake, es posible que tenga que permitir las direcciones IP de las máquinas del entorno de ejecución de integración autohospedado en la configuración de su firewall.

### <a name="get-url-of-azure-relay"></a>Obtención de la URL de Azure Relay

Debe incluirse un dominio y puerto obligatorios en la lista de permitidos del firewall para la comunicación con Azure Relay. El entorno de ejecución de integración autohospedado usa estos elementos para la creación interactiva, como, por ejemplo, las conexiones de prueba, el examen de la lista de carpetas y tablas, la obtención de esquemas y la vista previa de los datos. Si no quiere permitir **.servicebus.windows.net** y quiere tener direcciones URL más específicas, puede ver todos los nombres de dominio completo necesarios para el entorno de ejecución de integración autohospedado desde el portal de ADF. Siga estos pasos:

1. Vaya al portal de ADF y seleccione su entorno de ejecución de integración autohospedado.
2. En la página Editar, seleccione **Nodos**.
3. Haga clic en **View Service URL** (Ver direcciones URL de servicio) para obtener todos los nombres de dominio completos.

   ![Direcciones URL de Azure Relay](media/create-self-hosted-integration-runtime/Azure-relay-url.png)

4. Puede agregar estos nombres de dominio completos en la lista de permitidos de las reglas de firewall.

### <a name="copy-data-from-a-source-to-a-sink"></a>Copia de datos desde un origen a un receptor

Asegúrese de habilitar correctamente las reglas del firewall en el firewall corporativo, en el firewall de Windows de la máquina del entorno de ejecución de integración autohospedado y en el propio almacén de datos. De este modo, el entorno de ejecución de integración autohospedado podrá conectarse al origen y al receptor. Habilite las reglas de cada almacén de datos que participe en la operación de copia.

Por ejemplo, para copiar información de un almacén de datos local en un receptor de SQL Database o un receptor de Azure Synapse Analytics, debe realizar los siguientes pasos:

1. Permita la comunicación TCP saliente en el puerto 1433 tanto para el firewall corporativo como para el firewall de Windows.
2. Configure los valores del firewall de SQL Database para agregar la dirección IP de la máquina del entorno de ejecución de integración autohospedado a la lista de direcciones IP permitidas.

> [!NOTE]
> Si el firewall no permite el puerto de salida 1433, el entorno de ejecución de integración autohospedado no podrá acceder directamente a SQL Database. En este caso, puede usar una [copia de almacenamiento temporal](copy-activity-performance.md) en SQL Database y Azure Synapse Analytics. En este escenario, se requiere solo HTTPS (puerto 443) para el movimiento de datos.

## <a name="installation-best-practices"></a>Procedimientos recomendados de instalación

Para instalar el entorno de ejecución de integración autohospedado, descargue un paquete de instalación de identidad administrada del [Centro de descarga de Microsoft](https://www.microsoft.com/download/details.aspx?id=39717). Consulte el artículo [Movimiento de datos entre orígenes locales y la nube](tutorial-hybrid-copy-powershell.md) para obtener instrucciones detalladas.

- Configure un plan de energía en el equipo host para el entorno de ejecución de integración autohospedado con el fin de que el equipo no hiberne. Si el equipo host hiberna, el entorno de ejecución de integración autohospedado se pone en modo sin conexión.
- Realice regularmente una copia de las credenciales asociadas con el entorno de ejecución de integración autohospedado.
- Para automatizar las operaciones de configuración del entorno de ejecución de integración autohospedado, consulte [Configuración de un entorno de ejecución de integración autohospedado existente mediante PowerShell](#setting-up-a-self-hosted-integration-runtime).

## <a name="next-steps"></a>Pasos siguientes

Para obtener instrucciones paso a paso, consulte [Tutorial: Copia de datos locales en la nube](tutorial-hybrid-copy-powershell.md).
