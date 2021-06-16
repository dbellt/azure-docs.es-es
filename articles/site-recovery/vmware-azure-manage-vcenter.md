---
title: Administración de servidores de VMware vCenter en Azure Site Recovery
description: En este artículo se describe cómo agregar y administrar VMware vCenter para la recuperación ante desastres de máquinas virtuales de VMware en Azure con Azure Site Recovery.
services: site-recovery
author: Sharmistha-Rai
manager: gaggupta
ms.service: site-recovery
ms.topic: conceptual
ms.author: sharrai
ms.date: 05/27/2021
ms.openlocfilehash: b7c0a8a6305f3142d170cb601ec23597a69fe5b0
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576781"
---
# <a name="manage-vmware-vcenter-server"></a>Administración de VMware vCenter Server

En este artículo se resumen las acciones de administración en una instancia de vCenter Server de VMware en [Azure Site Recovery](site-recovery-overview.md).

## <a name="verify-prerequisites-for-vcenter-server"></a>Comprobación de los requisitos previos para vCenter Server

Los requisitos previos para instancias de vCenter Server y máquinas virtuales durante la recuperación ante desastres de máquinas virtuales de VMware en Azure se enumeran en la [matriz de compatibilidad](vmware-physical-azure-support-matrix.md#replicated-machines).

## <a name="set-up-an-account-for-automatic-discovery"></a>Configuración de una cuenta de detección automática

Cuando se configura la recuperación ante desastres para máquinas virtuales WMware locales, Site Recovery necesita acceso el host de vCenter Server/vSphere. A partir de ahí, el servidor de procesos de Site Recovery puede detectar automáticamente máquinas virtuales y realizar la conmutación por error en ellas, en caso de que sea necesario. De forma predeterminada, el servidor de procesos se ejecuta en el servidor de configuración de Site Recovery. Agregue una cuenta para que el servidor de configuración se conecte a vCenter Server o al host de vSphere como se indica a continuación:

1. Inicie sesión en el servidor de configuración.
1. Abra la herramienta del servidor de configuración (_cspsconfigtool.exe_) desde el acceso directo del escritorio.
1. En la pestaña **Administrar cuenta**, haga clic en **Agregar cuenta**.

   ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)

1. Proporcione los detalles de la cuenta y haga clic en **Aceptar** para agregarla. La cuenta debe tener los privilegios que se resumen en la tabla de permisos de la cuenta.

   > [!NOTE]
   > Se tarda aproximadamente 15 minutos en sincronizar la información de la cuenta con Site Recovery.

### <a name="account-permissions"></a>Permisos de cuenta

|**Task** | **Cuenta** | **Permisos** | **Detalles**|
|--- | --- | --- | ---|
|**Detección y migración de máquinas virtuales (sin conmutación por recuperación)** | Al menos una cuenta de usuario de solo lectura. | Objeto de centro de datos  –> Propagar al objeto secundario, rol = solo lectura | El usuario se asigna en el nivel de centro de datos y tiene acceso a todos los objetos de este.<br/><br/> Para restringir el acceso, asigne el rol **Sin acceso** con **Propagate to child object** (Propagar a objeto secundario) a los objetos secundarios (hosts de vSphere, almacenes de datos, máquinas virtuales y redes).|
|**Replicación y conmutación por error** | Al menos una cuenta de usuario de solo lectura. | Objeto de centro de datos  –> Propagar al objeto secundario, rol = solo lectura | El usuario se asigna en el nivel de centro de datos y tiene acceso a todos los objetos de este.<br/><br/> Para restringir el acceso, asigne el rol **Sin acceso** con **Propagate to child object** (Propagar a objeto secundario) a los objetos secundarios (hosts de vSphere, almacenes de datos, máquinas virtuales y redes).<br/><br/> Es útil para la migración, pero no para la replicación completa, la conmutación por error o la conmutación por recuperación.|
|**Replicación, conmutación por error y conmutación por recuperación** | Se recomienda crear un rol (AzureSiteRecoveryRole) con los permisos necesarios y después asignarlo a un grupo o usuario de VMware. | Objeto de centro de datos –> Propagar a objeto secundario, rol=AzureSiteRecoveryRole<br/><br/> Almacén de datos -> Asignar espacio, examinar almacén de datos, operaciones de archivo de bajo nivel, quitar archivo, actualizar archivos de máquina virtual<br/><br/> Red -> Asignación de red<br/><br/> Recursos-> Asignar máquina virtual al grupo de recursos, migrar apagado de máquina virtual, migrar máquinas virtuales encendidas<br/><br/> Tareas -> Crear tarea, actualizar tarea<br/><br/> Máquina virtual -> Configuración<br/><br/> Máquina virtual -> Interactuar -> Responder a pregunta, conexión de dispositivos, configurar soporte de CD, Configurar soporte de disquete, apagar, encender, instalación de herramientas de VMware<br/><br/> Máquina virtual -> Inventario -> Crear, registrar, anular registro<br/><br/> Máquina virtual -> Aprovisionamiento -> Permitir descarga de máquina virtual, permitir carga de archivos de máquina virtual<br/><br/> Máquina virtual -> Instantáneas -> Quitar instantáneas | El usuario se asigna en el nivel de centro de datos y tiene acceso a todos los objetos de este.<br/><br/> Para restringir el acceso, asigne el rol **Sin acceso** con **Propagate to child object** (Propagar a objeto secundario) a los objetos secundarios (hosts de vSphere, almacenes de datos, máquinas virtuales y redes).|

## <a name="add-vmware-server-to-the-vault"></a>Adición de un servidor VMware al almacén

Al configurar la recuperación ante desastres para máquinas virtuales de VMware locales, se agrega a vCenter Server o al host de vSphere en el que se detectan las máquinas virtuales al almacén de Site Recovery, como se indica a continuación:

1. En el almacén > **Infraestructura de Site Recovery** > **Servidores de configuración**, abra el servidor de configuración.
1. En la página **Detalles**, haga clic en **vCenter**.
1. En **Agregar servidor vCenter**, especifique un nombre descriptivo para el host de vSphere o el servidor vCenter.
1. Especifique la dirección IP o FQDN del servidor.
1. Deje el puerto 443 a menos que los servidores de VMware estén configurados para escuchar las solicitudes en un puerto diferente.
1. Seleccione la cuenta usada para conectar al servidor de VMware vCenter o vSphere ESXi. A continuación, haga clic en **Aceptar**.

## <a name="modify-credentials"></a>Modificación de credenciales

Si fuera necesario, puede modificar las credenciales usadas para conectarse a vCenter Server o al host de vSphere de la siguiente manera:

1. Inicie sesión en el servidor de configuración.
1. Abra la herramienta del servidor de configuración (_cspsconfigtool.exe_) mediante el acceso directo del escritorio.
1. En **Administrar cuenta**, haga clic en la pestaña **Agregar cuenta**.

   ![add-account](./media/vmware-azure-manage-vcenter/addaccount.png)

1. Proporcione los detalles de la nueva cuenta y haga clic en **Aceptar**. La cuenta necesita los permisos que se enumeran en la tabla de [permisos de la cuenta](#account-permissions).
1. En el almacén > **Infraestructura de Site Recovery** > **Servidores de configuración**, abra el servidor de configuración.
1. En **Detalles**, haga clic en **Actualizar el servidor**.
1. Una vez finalizado el trabajo de actualización del servidor, seleccione la instancia de vCenter Server.
1. En **Summary** (Resumen), seleccione la cuenta recién agregada en **vCenter server/vSphere host account** (Cuenta de servidor vCenter o host de vSphere) y haga clic en **Save** (Guardar).

   ![modify-account](./media/vmware-azure-manage-vcenter/modify-vcente-creds.png)

## <a name="delete-a-vcenter-server"></a>Eliminación de una instancia de vCenter Server

1. En el almacén > **Infraestructura de Site Recovery** > **Servidores de configuración**, abra el servidor de configuración.
1. En la página **Detalles**, seleccione el servidor de vCenter.
1. Haga clic en el botón **Eliminar**.

   ![delete-account](./media/vmware-azure-manage-vcenter/delete-vcenter.png)

## <a name="modify-the-ip-address-and-port"></a>Modificación de la dirección IP y el puerto

Puede modificar la dirección IP de la instancia de vCenter Server o los puertos utilizados para la comunicación entre el servidor y Site Recovery. De forma predeterminada, Site Recovery accede a la información de vCenter Server o del host de vSphere a través del puerto 443.

1. En el almacén > **Infraestructura de Site Recovery** > **Servidores de configuración**, haga clic en el servidor de configuración al que se agrega la instancia de vCenter Server.
1. En **Servidores vCenter**, haga clic en la instancia de vCenter Server que desea modificar.
1. En **Summary** (Resumen), actualice la dirección IP y el puerto, y guarde los cambios.

   ![add_ip_new_vcenter](media/vmware-azure-manage-vcenter/add-ip.png)

1. Para que los cambios surtan efecto, espere 15 minutos o [actualice el servidor de configuración](vmware-azure-manage-configuration-server.md#refresh-configuration-server).

## <a name="migrate-all-vms-to-a-new-server"></a>Migración de todas las máquinas virtuales a un nuevo servidor

Si quiere migrar todas las máquinas virtuales para usar una nueva instancia de vCenter Server, solo tiene que actualizar la dirección IP asignada a dicha instancia. No agregue otra cuenta de VMware, ya que podrían aparecer entradas duplicadas. Actualice la dirección de la siguiente manera:

1. En el almacén > **Infraestructura de Site Recovery** > **Servidores de configuración**, haga clic en el servidor de configuración al que se agrega la instancia de vCenter Server.
1. En la sección **Servidores vCenter**, haga clic en la instancia de vCenter Server desde la que tiene previsto migrar.
1. En **Resumen**, actualice la dirección IP con la de la nueva instancia de vCenter Server y guarde los cambios.
1. En cuanto se actualice la dirección IP, Site Recovery comienza a recibir información de detección de máquinas virtuales de la nueva instancia de vCenter Server. Esto no afecta a las actividades de replicación en curso.

## <a name="migrate-a-few-vms-to-a-new-server"></a>Migración de unas pocas máquinas virtuales a un nuevo servidor

Si solo desea migrar unas pocas máquinas virtuales que se replican a un nueva instancia de vCenter Server, haga lo siguiente:

1. [Agregue](#add-vmware-server-to-the-vault) la nueva instancia de vCenter Server al servidor de configuración.
1. [Deshabilite la replicación](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) para las máquinas virtuales que se moverán al nuevo servidor.
1. En VMware, migre las máquinas virtuales a la nueva instancia de vCenter Server.
1. [Habilite la replicación](vmware-azure-tutorial.md#enable-replication) para las máquinas virtuales migradas de nuevo y seleccione la nueva instancia de vCenter Server.

## <a name="migrate-most-vms-to-a-new-server"></a>Migración de la mayoría de las máquinas virtuales a un nuevo servidor

Si el número de máquinas virtuales que desea migrar a una nueva instancia de vCenter Server es mayor que el número de máquinas virtuales que permanecerán en la instancia de vCenter Server original, haga lo siguiente:

1. [Actualice la dirección IP](#modify-the-ip-address-and-port) asignada a la instancia de vCenter Server en los ajustes del servidor de configuración, a la dirección de la nueva instancia de vCenter Server.
1. [Deshabilite la replicación](site-recovery-manage-registration-and-protection.md#disable-protection-for-a-vmware-vm-or-physical-server-vmware-to-azure) para las pocas máquinas virtuales que permanecen en el servidor anterior.
1. [Agregue el la instancia de vCenter Server anterior](#add-vmware-server-to-the-vault) y su dirección IP al servidor de configuración.
1. [Vuelva a habilitar la replicación](vmware-azure-tutorial.md#enable-replication) para las máquinas virtuales que permanecen en el servidor anterior.

## <a name="next-steps"></a>Pasos siguientes

Si surge algún problema, consulte [Solución de problemas de detección de vCenter Server](vmware-azure-troubleshoot-vcenter-discovery-failures.md).
