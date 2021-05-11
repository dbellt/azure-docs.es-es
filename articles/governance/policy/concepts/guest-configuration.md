---
title: Información sobre cómo auditar el contenido de máquinas virtuales
description: Obtenga información sobre la forma en que Azure Policy usa el cliente de configuración de invitado para auditar la configuración dentro de las máquinas virtuales.
ms.date: 05/01/2021
ms.topic: conceptual
ms.openlocfilehash: 863f85c1eeeed381eda12a066a247c2605a1d68f
ms.sourcegitcommit: f6b76df4c22f1c605682418f3f2385131512508d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108326136"
---
# <a name="understand-azure-policys-guest-configuration"></a>Información sobre Guest Configuration de Azure Policy


Azure Policy puede auditar la configuración dentro de un equipo, tanto para las máquinas que se ejecutan en Azure como para las [conectadas a Arc](../../../azure-arc/servers/overview.md). La validación se realiza mediante el cliente y la extensión Guest Configuration. La extensión, a través del cliente, valida la configuración como:

- La configuración del sistema operativo
- Configuración de la aplicación o presencia
- Configuración del entorno

En este momento, la mayoría de definiciones de directivas de configuración de invitado de Azure Policy solo realiza la auditoría de la configuración dentro de la máquina. No se aplica a configuraciones. La excepción es una directiva integrada [a la que se hace referencia a continuación](#applying-configurations-using-guest-configuration).

[Hay disponible un tutorial de vídeo de este documento](https://youtu.be/Y6ryD3gTHOs).

## <a name="enable-guest-configuration"></a>Habilitar configuración de invitado

Para auditar el estado de las máquinas de su entorno, incluidas las de Azure y las conectadas a Arc, revise los detalles siguientes.

## <a name="resource-provider"></a>Proveedor de recursos

Para poder usar Guest Configuration debe registrar el proveedor de recursos.
Si la asignación de una directiva de configuración de invitado se realiza en el portal, o si la suscripción está inscrita en Azure Security Center, el proveedor de recursos se registra automáticamente. Puede registrarse manualmente mediante el [portal](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal), [Azure PowerShell](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-powershell) o la [CLI de Azure](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-cli).

## <a name="deploy-requirements-for-azure-virtual-machines"></a>Requisitos de implementación de Azure Virtual Machines

Para auditar la configuración en una máquina, se habilita una [extensión de máquina virtual](../../../virtual-machines/extensions/overview.md) y la máquina debe tener una identidad administrada por el sistema. La extensión descarga la asignación de directiva aplicable y la definición de configuración correspondiente. La identidad se usa para autenticar la máquina a medida que lee y escribe en el servicio de configuración de invitado. La extensión no es necesaria para las máquinas conectadas a Arc porque se incluye en el agente Connected Machine de Arc.

> [!IMPORTANT]
> La extensión de configuración de invitado y una identidad administrada son necesarias para realizar auditorías en las máquinas virtuales de Azure. Para implementar la extensión a gran escala, asigne la siguiente iniciativa de directiva:
> 
> `Deploy prerequisites to enable Guest Configuration policies on virtual machines`

### <a name="limits-set-on-the-extension"></a>Límites establecidos en la extensión

Para limitar que la extensión afecte a las aplicaciones que se ejecutan en la máquina, no se permite que la configuración de invitado supere el 5 % de la CPU. Esta limitación existe tanto para definiciones integradas como personalizadas. Lo mismo se aplica al servicio de configuración de invitado en el agente Connected Machine de Arc.

### <a name="validation-tools"></a>Herramientas de validación

Dentro de la máquina, el cliente de Guest Configuration usa herramientas locales para ejecutar la auditoría.

En la tabla siguiente se muestra una lista de las herramientas locales usadas en cada sistema operativo compatible. En el caso de contenido integrado, la configuración de invitado controla la carga de estas herramientas automáticamente.

|Sistema operativo|Herramienta de validación|Notas|
|-|-|-|
|Windows|[Configuración de estado deseado de PowerShell](/powershell/scripting/dsc/overview/overview) v2| Instalado de prueba en una carpeta que solo usa Azure Policy. No entrará en conflicto con DSC de Windows PowerShell. PowerShell Core no se agrega a la ruta de acceso del sistema.|
|Linux|[Chef InSpec](https://www.chef.io/inspec/)| Instala Chef InSpec versión 2.2.61 en la ubicación predeterminada y lo agrega a la ruta de acceso del sistema. También se instalan las dependencias del paquete InSpec, que incluyen Ruby y Python. |

### <a name="validation-frequency"></a>Frecuencia de validación

El cliente de Guest Configuration busca asignaciones de invitados nuevas o modificadas cada 5 minutos. Una vez que se recibe una asignación de invitado, se comprueban los valores de esa configuración en un intervalo de 15 minutos. Los resultados se envían al proveedor de recursos de Guest Configuration cuando finaliza la auditoría. Cuando se produce una directiva del tipo [desencadenador evaluación](../how-to/get-compliance-data.md#evaluation-triggers), el estado de la máquina se escribe en el proveedor de recursos de Guest Configuration. Esta actualización hace que Azure Policy evalúe las propiedades de Azure Resource Manager. Una evaluación de Azure Policy a petición recupera el valor más reciente del proveedor de recursos de la configuración de invitado. Sin embargo, no desencadena una nueva auditoría de la configuración en la máquina. El estado se escribe simultáneamente en Azure Resource Graph.

## <a name="supported-client-types"></a>Tipos de cliente admitidos

Las definiciones de directivas de configuración de invitado son inclusivas de nuevas versiones. Las versiones anteriores de los sistemas operativos disponibles en Azure Marketplace se excluyen si el cliente de configuración de invitado no es compatible. En la tabla siguiente se muestra una lista de sistemas operativos compatibles en imágenes de Azure:

|Publicador|Nombre|Versiones|
|-|-|-|
|Canonical|Ubuntu Server|14.04 - 20.04|
|Credativ|Debian|8 - 10|
|Microsoft|Windows Server|2012 - 2019|
|Microsoft|Cliente Windows|Windows 10|
|OpenLogic|CentOS|7.3 -8|
|Red Hat|Red Hat Enterprise Linux|7.4 - 8|
|Suse|SLES|12 SP3-SP5, 15|

Las definiciones de directivas de configuración de invitado admiten imágenes de máquina virtual personalizadas, siempre y cuando se trate de uno de los sistemas operativos de la tabla anterior.

## <a name="network-requirements"></a>Requisitos de red

Las máquinas virtuales de Azure pueden usar su adaptador de red local o un vínculo privado para comunicarse con el servicio de configuración de invitado.

Las máquinas de Azure Arc se conectan mediante la infraestructura de red local para llegar a los servicios de Azure e informar del estado de cumplimiento.

### <a name="communicate-over-virtual-networks-in-azure"></a>Comunicación a través de redes virtuales en Azure

Para comunicarse con el proveedor de recursos de la configuración de invitado en Azure, las máquinas requieren acceso de salida a los centros de datos Azure en el puerto **443**. Si una red en Azure no permite el tráfico saliente, las excepciones deben configurarse con las reglas del [grupo de seguridad de red](../../../virtual-network/manage-network-security-group.md#create-a-security-rule). La [etiqueta de servicio](../../../virtual-network/service-tags-overview.md) "GuestAndHybridManagement" se puede usar para hacer referencia al servicio de configuración de invitado en lugar de mantener manualmente la [lista de intervalos IP](https://www.microsoft.com/en-us/download/details.aspx?id=56519) para los centros de datos de Azure.

### <a name="communicate-over-private-link-in-azure"></a>Comunicación a través de un vínculo privado en Azure

Las máquinas virtuales pueden usar un [vínculo privado](../../../private-link/private-link-overview.md) para la comunicación con el servicio de configuración de invitado. Aplique la etiqueta con el nombre `EnablePrivateNetworkGC` y el valor `TRUE` para habilitar esta característica. La etiqueta se puede aplicar antes o después de que se apliquen las definiciones de directivas de configuración de invitado a la máquina.

El tráfico se enruta mediante la [dirección IP pública virtual](../../../virtual-network/what-is-ip-address-168-63-129-16.md) de Azure para establecer un canal seguro y autenticado con recursos de la plataforma Azure.

### <a name="azure-arc-connected-machines"></a>Máquinas conectadas de Azure Arc

Los nodos que se encuentran fuera de Azure y que están conectados mediante Azure Arc requieren conectividad con el servicio de configuración de invitado. Detalles sobre los requisitos de red y proxy que se proporcionan en la [documentación de Azure Arc](../../../azure-arc/servers/overview.md).

En el caso de los servidores conectados a Arc en centros de datos privados, permita el tráfico con los siguientes patrones:

- Puerto: solo se requiere el puerto TCP 443 para el acceso a Internet
- Dirección URL global: `*.guestconfiguration.azure.com`

## <a name="managed-identity-requirements"></a>Requisitos de identidad administrada

Las definiciones de directiva de la iniciativa _Implementar los requisitos previos para habilitar directivas de configuración de invitado en máquinas virtuales_ habilitan una identidad administrada asignada por el sistema, si no existe ninguna. Hay dos definiciones de directivas en la iniciativa que administran la creación de identidades. Las condiciones IF en las definiciones de directiva garantizan el comportamiento correcto en función del estado actual del recurso de máquina en Azure.

Si la máquina no tiene ninguna identidad administrada actualmente, la directiva efectiva será: [Agregar una identidad administrada asignada por el sistema para habilitar las asignaciones de configuración de invitado en máquinas virtuales sin identidades](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F3cf2ab00-13f1-4d0c-8971-2ac904541a7e)

Si la maquina tiene actualmente una identidad del sistema asignada por el usuario, la directiva efectiva será: [Agregar una identidad administrada asignada por el sistema para habilitar las asignaciones de configuración de invitado en VM con una identidad asignada por el usuario](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F497dff13-db2a-4c0f-8603-28fa3b331ab6)

## <a name="guest-configuration-definition-requirements"></a>Requisitos de definición de Guest Configuration

Las definiciones de directivas de invitado deusan el efecto **AuditIfNotExists**. Cuando se asigna la definición, un servicio back-end administra automáticamente el ciclo de vida de todos los requisitos del proveedor de recursos de Azure `Microsoft.GuestConfiguration`.

Las definiciones de directivas **AuditIfNotExists** no devolverán resultados de cumplimiento hasta que se cumplan todos los requisitos en la máquina. Los requisitos se describen en la sección [Requisitos de implementación de Azure Virtual Machines](#deploy-requirements-for-azure-virtual-machines).

> [!IMPORTANT]
> En una versión anterior de Guest Configuration, una iniciativa debía combinar las definiciones **DeployIfNotExists** y **AuditIfNotExists**. Las definiciones **DeployIfNotExists** ya no son necesarias. Las definiciones e iniciativas se etiquetan como `[Deprecated]`, pero las asignaciones existentes seguirán funcionando. Para obtener más información, consulte la entrada de blog: [Cambio importante publicado para las directivas de auditoría de configuración de invitado](https://techcommunity.microsoft.com/t5/azure-governance-and-management/important-change-released-for-guest-configuration-audit-policies/ba-p/1655316)

### <a name="what-is-a-guest-assignment"></a>¿Qué es una asignación de invitado?

Cuando se asigna una instancia de Azure Policy, si se encuentra en la categoría "Configuración de invitado", se incluyen metadatos para describir una asignación de invitado.
Puede pensar en una asignación de invitado como un vínculo entre una máquina y un escenario de Azure Policy.
Por ejemplo, el fragmento de código siguiente asocia la configuración de línea de base de Windows de Azure con la versión mínima `1.0.0` a cualquier máquina en el ámbito de la directiva. De forma predeterminada, la asignación de invitado solo realizará una auditoría de la máquina.

```json
"metadata": {
    "category": "Guest Configuration",
    "guestConfiguration": {
        "name": "AzureWindowsBaseline",
        "version": "1.*"
    }
//additional metadata properties exist
```

El servicio de configuración de invitado crea automáticamente las asignaciones de invitado por cada máquina. El tipo de recurso es `Microsoft.GuestConfiguration/guestConfigurationAssignments`.
Azure Policy usa la propiedad **complianceStatus** del recurso de asignación de invitado para informar del estado de cumplimiento. Para más información, vea [Obtención de datos de cumplimiento](../how-to/get-compliance-data.md).

#### <a name="auditing-operating-system-settings-following-industry-baselines"></a>Auditoría de la configuración del sistema operativo siguiendo las líneas de base del sector

Una iniciativa de Azure Policy permite realizar la auditoría de la configuración del sistema operativo en función de una "línea de base". La definición, _\[Versión preliminar\]: Windows machines should meet requirements for the Azure security baseline_ (Versión preliminar: Las máquinas Windows deben cumplir los requisitos de la línea de base de seguridad de Azure) incluye un conjunto de reglas basadas en la directiva de grupo de Active Directory.

La mayoría de las opciones de configuración están disponibles como parámetros. Los parámetros le permiten personalizar lo que se audita.
Alinee la directiva con sus requisitos o asigne la directiva a información de terceros, como estándares normativos del sector.

Algunos parámetros admiten un intervalo de valores enteros. Por ejemplo, el valor de Vigencia máxima de la contraseña puede auditar la configuración de directiva de grupo efectiva. Un intervalo "1,70" confirmaría que los usuarios deben cambiar sus contraseñas al menos cada 70 días, pero no menos de un día.

Si asigna la directiva mediante una plantilla de Azure Resource Manager (plantilla de ARM), use un archivo de parámetros para administrar excepciones. Proteja los archivos en un sistema de control de versiones, como Git. Los comentarios sobre los cambios en los archivos proporcionan evidencias sobre el motivo por el que una asignación es una excepción al valor esperado.

#### <a name="applying-configurations-using-guest-configuration"></a>Aplicación de configuraciones mediante la configuración de invitado

La definición _Configurar la zona horaria de las máquinas Windows_ realiza cambios en la máquina mediante la configuración de la zona horaria. No se admiten las definiciones de directiva personalizadas para configurar los valores dentro de las máquinas.

Al asignar definiciones que empiezan por _Configurar_, también debe asignar la definición _Implementar los requisitos previos para habilitar la directiva de configuración de invitado en máquinas virtuales Windows._ . Puede combinar estas definiciones en una iniciativa si así lo decide.

> [!NOTE]
> La directiva de zona horaria integrada es la única definición que admite la configuración dentro de las máquinas. Las definiciones de directivas personalizadas que configuran los valores dentro de las máquinas no se admiten.

#### <a name="assigning-policies-to-machines-outside-of-azure"></a>Asignación de directivas a máquinas fuera de Azure

Las definiciones de directivas de auditoría disponibles para la configuración de invitado incluyen el tipo de recurso **Microsoft.HybridCompute/machines**. Todas las máquinas incorporadas a [Azure Arc para servidores](../../../azure-arc/servers/overview.md) que se encuentran en el ámbito de asignación de directivas se incluyen automáticamente.

## <a name="troubleshooting-guest-configuration"></a>Solución de problemas de la configuración de invitado

Para obtener más información sobre la solución de problemas de la configuración de invitado, consulte [Solución de problemas de Azure Policy](../troubleshoot/general.md).

### <a name="multiple-assignments"></a>Asignaciones múltiples

Actualmente, las definiciones de directivas de configuración de invitado solo admiten la asignación de la misma asignación de invitado una vez por cada máquina, incluso si la asignación de directiva usa parámetros diferentes.

### <a name="client-log-files"></a>Archivos de registro de cliente

La extensión de la configuración de invitado escribe archivos de registro en las siguientes ubicaciones:

Windows: `C:\ProgramData\GuestConfig\gc_agent_logs\gc_agent.log`

Linux

- VM de Azure: `/var/lib/GuestConfig/gc_agent_logs/gc_agent.log`
- VM de Azure: `/var/lib/GuestConfig/arc_policy_logs/gc_agent.log`

### <a name="collecting-logs-remotely"></a>Recopilar registros de forma remota

El primer paso para solucionar problemas de configuraciones o módulos de Configuración de invitado debe ser usar el cmdlet `Test-GuestConfigurationPackage` siguiendo los pasos para [crear una directiva de auditoría de Configuración de invitado para Windows](../how-to/guest-configuration-create.md#step-by-step-creating-a-custom-guest-configuration-audit-policy-for-windows).
Si no se realiza correctamente, la recopilación de registros de cliente puede ayudar a diagnosticar problemas.

#### <a name="windows"></a>Windows

Capture información de archivos de registro con el [comando de ejecución de VM de Azure](../../../virtual-machines/windows/run-command.md). El siguiente script de PowerShell de ejemplo puede ser útil.

```powershell
$linesToIncludeBeforeMatch = 0
$linesToIncludeAfterMatch = 10
$logPath = 'C:\ProgramData\GuestConfig\gc_agent_logs\gc_agent.log'
Select-String -Path $logPath -pattern 'DSCEngine','DSCManagedEngine' -CaseSensitive -Context $linesToIncludeBeforeMatch,$linesToIncludeAfterMatch | Select-Object -Last 10
```

#### <a name="linux"></a>Linux

Capture información de archivos de registro con el [comando de ejecución de VM de Azure](../../../virtual-machines/linux/run-command.md). El siguiente script de Bash de ejemplo puede ser útil.

```Bash
linesToIncludeBeforeMatch=0
linesToIncludeAfterMatch=10
logPath=/var/lib/GuestConfig/gc_agent_logs/gc_agent.log
egrep -B $linesToIncludeBeforeMatch -A $linesToIncludeAfterMatch 'DSCEngine|DSCManagedEngine' $logPath | tail
```

### <a name="client-files"></a>Archivos de cliente

El cliente de configuración de invitado descarga paquetes de contenido en una máquina y extrae el contenido.
Para comprobar qué contenido se ha descargado y almacenado, consulte las ubicaciones de carpeta que se indican a continuación.

Windows: `c:\programdata\guestconfig\configuration`

Linux: `/var/lib/GuestConfig/Configuration`

## <a name="guest-configuration-samples"></a>Ejemplos de configuración de invitado

Hay ejemplos de directivas integradas de la configuración de invitado disponibles en las siguientes ubicaciones:

- [Definiciones de directivas integradas: configuración de invitado](../samples/built-in-policies.md#guest-configuration)
- [Iniciativas integradas: configuración de invitado](../samples/built-in-initiatives.md#guest-configuration)
- [Repositorio de GitHub de ejemplos de Azure Policy](https://github.com/Azure/azure-policy/tree/master/built-in-policies/policySetDefinitions/Guest%20Configuration)

### <a name="video-overview"></a>Introducción en vídeo

La siguiente información general de la configuración de invitado de Azure Policy es de las charlas de ITOps de 2021.

[Control de las líneas de base en entornos de servidor híbridos con la configuración de invitado de Azure Policy](https://techcommunity.microsoft.com/t5/itops-talk-blog/ops114-governing-baselines-in-hybrid-server-environments-using/ba-p/2109245)

## <a name="next-steps"></a>Pasos siguientes

- Aprenda cómo ver los detalles de cada opción de configuración en la [vista de compatibilidad de la configuración de invitado](../how-to/determine-non-compliance.md#compliance-details-for-guest-configuration).
- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](./definition-structure.md).
- Vea la [Descripción de los efectos de directivas](./effects.md).
- Obtenga información acerca de cómo se pueden [crear directivas mediante programación](../how-to/programmatically-create.md).
- Obtenga información sobre cómo [obtener datos de cumplimiento](../how-to/get-compliance-data.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](../how-to/remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
