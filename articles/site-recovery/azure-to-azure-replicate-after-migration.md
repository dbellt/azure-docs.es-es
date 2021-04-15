---
title: Configuración de la recuperación ante desastres después de migrar a Azure con Azure Site Recovery
description: En este artículo se describe cómo preparar máquinas para configurar la recuperación ante desastres entre regiones de Azure después de la migración a Azure mediante Azure Site Recovery.
services: site-recovery
ms.service: site-recovery
ms.topic: article
ms.date: 11/14/2019
ms.openlocfilehash: 41fa4f5d042b5c284a58b0fd87486e2309b03053
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581739"
---
# <a name="set-up-disaster-recovery-for-azure-vms-after-migration-to-azure"></a>Configuración de la recuperación ante desastres de máquinas virtuales de Azure tras la migración a Azure 


Siga este artículo si ha [migrado máquinas locales a máquinas virtuales de Azure](./migrate-tutorial-on-premises-azure.md) mediante el servicio [Site Recovery](site-recovery-overview.md) y ahora quiere configurar las máquinas virtuales para la recuperación ante desastres en una región secundaria de Azure. En este artículo se describe cómo asegurarse de que el agente de máquina virtual de Azure se instala en máquinas virtuales migradas y cómo quitar Site Recovery Mobility Service, que ya no se necesita después de la migración.



## <a name="verify-migration"></a>Comprobación de la migración

Antes de configurar la recuperación ante desastres, asegúrese de que la migración se haya completado según lo previsto. Para completar una migración correctamente después de una conmutación por error, debe seleccionar la opción **Completar la migración** en cada máquina que desea migrar. 

## <a name="verify-the-azure-vm-agent"></a>Comprobación del agente de máquina virtual de Azure

Cada máquina virtual de Azure debe tener instalado el [agente de máquina virtual de Azure](../virtual-machines/extensions/agent-windows.md). Para replicar máquinas virtuales de Azure, Site Recovery instala una extensión en el agente.

- Si la máquina ejecuta la versión 9.7.0.0 o una posterior de Site Recovery Mobility Service, Mobility Service instala automáticamente el agente de máquina virtual de Azur en máquinas virtuales de Windows. En versiones anteriores de Mobility Service, debía instalar el agente de manera manual.
- En el caso de las máquinas virtuales Linux, debe instalar manualmente el agente de máquina virtual de Azure. Solo necesita instalar el agente de máquina virtual de Azure si la versión de Mobility Service instalada en la máquina migrada es la 9.6 o posterior.


### <a name="install-the-agent-on-windows-vms"></a>Instalación del agente en máquinas virtuales de Windows

Si ejecuta una versión de Site Recovery Mobility Service anterior a la 9.7.0.0 o tiene la necesidad de instalar al agente manualmente, haga lo siguiente:  

1. Asegúrese de que tiene permisos de administrador en la máquina virtual.
2. Descargue el [instalador del agente de máquina virtual de](https://go.microsoft.com/fwlink/?LinkID=394789&clcid=0x409).
3. Ejecute el archivo del instalador.

#### <a name="validate-the-installation"></a>Validación de la instalación
Para comprobar que el agente está instalado:

1. En la máquina virtual de Azure, en la carpeta C:\WindowsAzure\Package, debería ver el archivo WaAppAgent.exe.
2. Haga clic con el botón derecho en el archivo y, en **Propiedades**, seleccione la pestaña **Detalles**.
3. Verifique que en el campo **Versión del producto** aparece el valor 2.6.1198.718 o una versión posterior.

[Obtenga más información](../virtual-machines/extensions/agent-windows.md) a sobre la instalación del agente para Windows.

### <a name="install-the-agent-on-linux-vms"></a>Instalación del agente en máquinas virtuales de Linux

Instale el agente de [máquina virtual de Linux de Azure](../virtual-machines/extensions/agent-linux.md) manualmente, como se indica a continuación:

1. Asegúrese de que tiene permisos de administrador en la máquina.
2. Se recomienda encarecidamente que instale al agente de máquina virtual de Linux mediante un paquete de RPM o DEB de su repositorio de paquetes de distribución. Todos los [proveedores de distribución aprobada](../virtual-machines/linux/endorsed-distros.md) integran el paquete de agente Linux de Azure en sus imágenes y repositorios.
    - Se recomienda encarecidamente que actualice el agente solo a través de un repositorio de distribución.
    - No se recomienda que instale el agente de máquina virtual de Linux directamente desde GitHub y lo actualice.
    -  Si el último agente no está disponible para su distribución, póngase en contacto con el soporte técnico de distribución para obtener instrucciones sobre cómo instalarlo. 

#### <a name="validate-the-installation"></a>Validación de la instalación 

1. Ejecute el comando **ps -e** para asegurarse de que el agente de Azure se ejecuta en la máquina virtual de Linux.
2. Si el proceso no se está ejecutando, reinícielo mediante los siguientes comandos:
    - Para Ubuntu: **service walinuxagent start**
    - Para otras distribuciones: **service waagent start**


## <a name="uninstall-the-mobility-service"></a>Desinstalación de Mobility Service

1. Desinstale manualmente Mobility Service de la máquina virtual de Azure mediante uno de los siguientes métodos. 
    - En Windows, en Panel de control > **Agregar o quitar programas**, desinstale **Microsoft Azure Site Recovery Mobility Service/Servidor de destino maestro**. En un símbolo del sistema con privilegios elevados, ejecute:
        ```
        MsiExec.exe /qn /x {275197FC-14FD-4560-A5EB-38217F80CBD1} /L+*V "C:\ProgramData\ASRSetupLogs\UnifiedAgentMSIUninstall.log"
        ```
    - En Linux, inicie sesión como un usuario raíz. En un terminal, vaya a **/user/local/ASR** y ejecute el siguiente comando:
        ```
        ./uninstall.sh -Y
        ```
2. Reinicie la máquina virtual antes de configurar la replicación.

## <a name="next-steps"></a>Pasos siguientes

[Consulte la solución de problemas](site-recovery-extension-troubleshoot.md) para la extensión de Site Recovery en el agente de máquina virtual de Azure.
[Replicación rápida](azure-to-azure-quickstart.md) de una máquina virtual de Azure a una región secundaria.
