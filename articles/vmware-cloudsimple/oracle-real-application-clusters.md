---
title: 'Azure VMware Solution by CloudSimple: optimización de la nube privada de CloudSimple para Oracle RAC'
description: Se describe cómo implementar un nuevo clúster y optimizar una VM para la instalación y configuración de Oracle Real Application Clusters (RAC).
author: shortpatti
ms.author: v-patsho
ms.date: 08/06/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: a563067c62f56905d88186db77cbe2e039cfa409
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177465"
---
# <a name="optimize-your-cloudsimple-private-cloud-for-installing-oracle-rac"></a>Optimización de la nube privada de CloudSimple para la instalación de Oracle RAC

Puede implementar Oracle Real Application Clusters (RAC) en el entorno de nube privada de CloudSimple. En esta guía se describe cómo implementar un nuevo clúster y optimizar una VM para la solución de Oracle RAC. Después de completar los pasos de este tema, puede instalar y configurar Oracle RAC.

## <a name="storage-policy"></a>Directiva de almacenamiento

La implementación correcta de Oracle RAC requiere un número adecuado de nodos en el clúster.  En la directiva de almacenamiento de vSAN, se aplican errores tolerables (FTT) a los discos de datos que se usan para almacenar la base de datos, el registro y los discos de fase de puesta al día.  El número necesario de nodos para tolerar errores de forma eficaz es 2N + 1, donde N es el valor de FTT.

Ejemplo: Si el valor de FTT deseado es 2, el número total de nodos del clúster debe ser 2 * 2 + 1 = 5.

## <a name="overview-of-deployment"></a>Información general de la implementación

En las secciones siguientes se describe cómo configurar el entorno de nube privada de CloudSimple para Oracle RAC.

1. Procedimientos recomendados para la configuración del disco
2. Implementación del clúster de vSphere de nube privada de CloudSimple
3. Configuración de redes para Oracle RAC
4. Configuración de directivas de almacenamiento de vSAN
5. Creación de VM de Oracle y de discos de VM compartidos
6. Configuración de reglas de afinidad de VM a host

## <a name="best-practices-for-disk-configuration"></a>Procedimientos recomendados para la configuración del disco

Las máquinas virtuales de Oracle RAC tienen varios discos, que se usan para una función específica.  Los discos compartidos se montan en todas las máquinas virtuales que usa el clúster de Oracle RAC.  Los discos de instalación de software y de sistema operativo solo se montan en las máquinas virtuales individuales.  

![Información general sobre los discos de máquina virtual de Oracle RAC](media/oracle-vm-disks-overview.png)

En el ejemplo siguiente se usan los discos definidos en la tabla siguiente.

| Disco                                      | Propósito                                       | Disco compartido |
|-------------------------------------------|-----------------------------------------------|-------------|
| SO                                        | Disco del sistema operativo                         | No          |
| GRID                                      | Ubicación de instalación para el software de Oracle Grid     | No          |
| DATABASE                                  | Ubicación de instalación para el software de Oracle Database | No          |
| ORAHOME                                   | Ubicación de instalación para archivos binarios de Oracle Database    | No          |
| DATA1, DATA2, DATA3, DATA4                | Disco en el que se almacenan los archivos de Oracle Database   | Sí         |
| REDO1, REDO2, REDO3, REDO4, REDO5, REDO6  | Discos de registro de fase de puesta al día                                | Sí         |
| OCR1, OCR2, OCR3, OCR4, OCR5              | Discos de votación                                  | Sí         |
| FRA1, FRA2                                | Discos de área de recuperación rápida                      | Sí         |

![Configuración del disco de la máquina virtual de Oracle](media/oracle-vmdk.png)

### <a name="virtual-machine-configuration"></a>Configuración de la máquina virtual

* Cada máquina virtual está configurada con cuatro controladoras SCSI.
* El tipo de controladora SCSI se establece en VMware paravirtual.
* Se crean varios discos virtuales (.vmdk).
* Los discos se montan en diferentes controladoras SCSI.
* El tipo de uso compartido de varios escritores se ha establecido para los discos de clúster compartidos.
* La directiva de almacenamiento vSAN está definida para garantizar la alta disponibilidad de los discos.

### <a name="operating-system-and-software-disk-configuration"></a>Configuración del sistema operativo y del disco de software

Cada máquina virtual de Oracle está configurada con varios discos para el sistema operativo host, el intercambio, la instalación de software y otras funciones del sistema operativo.  Estos discos no se comparten entre las máquinas virtuales.  

* Se configuran tres discos para cada máquina virtual como discos virtuales y se montan en máquinas virtuales de Oracle RAC.
    * Disco de SO
    * Disco para almacenar archivos de instalación de Oracle Grid
    * Disco para almacenar archivos de instalación de Oracle Database
* Los discos se pueden configurar **con aprovisionamiento fino**.
* Cada disco se monta en al primera controladora SCSI (SCSI0).  
* El uso compartido se establece en **No sharing** (Sin uso compartido).
* La redundancia se define en el almacenamiento mediante directivas de vSAN.  

![Diagrama que muestra la configuración física del disco del sistema operativo de Oracle RAC.](media/oracle-vm-os-disks.png)

### <a name="data-disk-configuration"></a>Configuración del disco de datos

Los discos de datos se utilizan principalmente para almacenar archivos de base de datos.  

* Se configuran cuatro discos como discos virtuales y se montan en todas las máquinas virtuales de Oracle RAC.
* Cada disco se monta en una controladora SCSI diferente.
* Cada disco virtual se configura como un **Thick Provision Eager Zeroed** (valor diligente establecido en cero de aprovisionamiento grueso).  
* El uso compartido se establece en **Multi-writer** (Multiescritor).  
* Los discos deben configurarse como un grupo de discos de administración automática del almacenamiento (ASM).  
* La redundancia se define en el almacenamiento mediante directivas de vSAN.  
* La redundancia de ASM se establece en redundancia **externa**.

![Configuración del grupo de discos de datos de Oracle RAC](media/oracle-vm-data-disks.png)

### <a name="redo-log-disk-configuration"></a>Configuración del disco de registro de fase de puesta al día

Los archivos de registro de fase de puesta al día se utilizan para almacenar una copia de los cambios realizados en la base de datos.  Los archivos de registro se usan cuando es necesario recuperar los datos después de cualquier error.

* Los discos de registro de fase de puesta al día deben configurarse como varios grupos de discos.  
* Se crean seis discos y se montan en todas las máquinas virtuales de Oracle RAC.
* Los discos se montan en diferentes controladoras SCSI.
* Cada disco virtual se configura como un **Thick Provision Eager Zeroed** (valor diligente establecido en cero de aprovisionamiento grueso).
* El uso compartido se establece en **Multi-writer** (Multiescritor).  
* Los discos deben configurarse como dos grupos de discos de ASM.
* Cada grupo de discos de ASM contiene tres discos, que se encuentran en diferentes controladoras SCSI.  
* La redundancia de ASM se establece en redundancia **normal**.
* Se crean cinco archivos de registro de fase de puesta al día en el grupo de registro de fase de puesta al día de ASM.

```
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 1 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
SQL > alter database add logfile thread 2 ('+ORCLRAC_REDO1','+ORCLRAC_REDO2') size 1G;
```

![Configuración del grupo de discos de datos de fase de puesta al día de Oracle RAC](media/oracle-vm-redo-log-disks.png)

### <a name="oracle-voting-disk-configuration"></a>Configuración del disco de votación de Oracle

Los discos de votación proporcionan la funcionalidad de disco de cuórum como canal de comunicación adicional para evitar situaciones de cerebro dividido.

* Se crean cinco discos y se montan en todas las máquinas virtuales de Oracle RAC.
* Los discos se montan en una controladora SCSI.
* Cada disco virtual se configura como un **Thick Provision Eager Zeroed** (valor diligente establecido en cero de aprovisionamiento grueso).
* El uso compartido se establece en **Multi-writer** (Multiescritor).  
* Los discos deben configurarse como un grupo de discos de ASM.  
* La redundancia de ASM se establece en redundancia **alta**.

![Configuración del grupo de discos de votación de Oracle RAC](media/oracle-vm-voting-disks.png)

### <a name="oracle-fast-recovery-area-disk-configuration-optional"></a>Configuración del disco del área de recuperación rápida de Oracle (opcional)

El área de recuperación rápida (FRA) es el sistema de archivos administrado por el grupo de discos de ASM de Oracle.  FRA proporciona una ubicación de almacenamiento compartido para los archivos de copia de seguridad y recuperación. Oracle crea registros archivados y registros flashback en el área de recuperación rápida. Oracle Recovery Manager (RMAN) puede almacenar opcionalmente sus conjuntos de copia de seguridad y copias de imágenes en el área de recuperación rápida, y los usa al restaurar los archivos durante la recuperación de medios.

* Se crean dos discos y se montan en todas las máquinas virtuales de Oracle RAC.
* Los discos se montan en una controladora SCSI.
* Cada disco virtual se configura como un **Thick Provision Eager Zeroed** (valor diligente establecido en cero de aprovisionamiento grueso).
* El uso compartido se establece en **Multi-writer** (Multiescritor).  
* Los discos deben configurarse como un grupo de discos de ASM.  
* La redundancia de ASM se establece en redundancia **externa**.

![Diagrama que muestra la configuración del grupo de discos de votación de Oracle RAC.](media/oracle-vm-fra-disks.png)

## <a name="deploy-cloudsimple-private-cloud-vsphere-cluster"></a>Implementación del clúster de vSphere de nube privada de CloudSimple

Para implementar un clúster de vSphere en la nube privada, siga este proceso:

1. En el portal de CloudSimple [cree una nube privada](create-private-cloud.md). CloudSimple crea un usuario predeterminado de vCenter denominado "cloudowner" en la nube privada que se acaba de crear. Para obtener detalles sobre el modelo predeterminado de permisos y usuarios de la nube privada, consulte [Información sobre el modelo de permisos de la nube privada](learn-private-cloud-permissions.md).  En este paso se crea el clúster de administración principal para la nube privada.

2. En el portal de CloudSimple, [expanda la nube privada](expand-private-cloud.md) con un nuevo clúster.  Este clúster se usará para implementar Oracle RAC.  Seleccione el número de nodos en función de la tolerancia a errores deseada (tres nodos como mínimo).

## <a name="set-up-networking-for-oracle-rac"></a>Configuración de redes para Oracle RAC

1. En la nube privada, [cree dos VLAN](create-vlan-subnet.md), una para la red pública de Oracle y otra para la red privada de Oracle y asigne los CIDR de subred adecuados.
2. Una vez creadas las VLAN, cree los [grupos de puertos distribuidos en el vCenter de la nube privada](create-vlan-subnet.md#use-vlan-information-to-set-up-a-distributed-port-group-in-vsphere).
3. Configure una [máquina virtual de servidor DNS y DHCP](dns-dhcp-setup.md) en el clúster de administración para el entorno de Oracle.
4. [Configure el reenvío de DNS en el servidor DNS](on-premises-dns-setup.md#create-a-conditional-forwarder) instalado en la nube privada.

## <a name="set-up-vsan-storage-policies"></a>Configuración de directivas de almacenamiento de vSAN

Las directivas de vSAN definen los errores tolerables y la fragmentación de discos para los datos almacenados en los discos de VM.  La directiva de almacenamiento creada debe aplicarse a los discos de VM al crear la VM.

1. [Inicie sesión en el cliente de vSphere](./vcenter-access.md) de la nube privada.
2. En el menú superior, seleccione **Directivas y perfiles**.
3. En el menú de la izquierda, seleccione **VM Storage Policies** (Directivas de almacenamiento de VM) y, a continuación, seleccione **Create a VM storage Policy** (Crear una directiva de almacenamiento de VM).
4. Escriba un nombre descriptivo para la directiva y haga clic en **SIGUIENTE**.
5. En la sección **Policy structure** (Estructura de la directiva), seleccione **Enable rules for vSAN storage** (Habilitar reglas para el almacenamiento de vSAN) y haga clic en **SIGUIENTE**.
6. En la sección **vSAN** > **Disponibilidad**, seleccione **Ninguno** para la tolerancia ante desastres de sitios. En el caso de los errores tolerables, seleccione la opción **RAID - Mirroring** (RAID: creación de reflejo) para el valor de FTT deseado.
    ![Configuración de vSAN](media/oracle-rac-storage-wizard-vsan.png).
7. En la sección **Avanzado**, seleccione el número de bandas de disco por objeto. En Reserva del espacio de objetos, seleccione **Aprovisionamiento grueso**. Seleccione **Disable object checksum** (Deshabilitar la suma de comprobación de objetos). Haga clic en **SIGUIENTE**.
8. Siga las instrucciones que se muestran en la pantalla para ver la lista de almacenes de datos de vSAN compatibles, revisar la configuración y finalizar la instalación.

## <a name="create-oracle-vms-and-create-shared-vm-disks-for-oracle"></a>Creación de VM de Oracle y de discos de VM compartidos para Oracle

Para crear una VM para Oracle, clone una VM existente o cree una nueva.  En esta sección se describe cómo crear una VM nueva y, a continuación, clonarla para crear una segunda después de instalar el sistema operativo base.  Una vez creadas las VM, puede crear una para agregar discos.  El clúster de Oracle usa discos compartidos para almacenar datos, registros y registros de fase de puesta al día.

### <a name="create-vms"></a>Creación de máquinas virtuales

1. En vCenter, haga clic en el icono **Hosts and Clusters** (Hosts y clústeres). Seleccione el clúster que creó para Oracle.
2. Haga clic con el botón derecho en el clúster y seleccione **Nueva máquina virtual**.
3. Seleccione **Create new virtual machine** (Crear nueva máquina virtual) y haga clic en **Next** (Siguiente).
4. Asigne un nombre a la máquina virtual, seleccione la ubicación de la VN de Oracle y haga clic en **Siguiente**.
5. Seleccione el recurso de clúster y haga clic en **Siguiente**.
6. Seleccione el almacén de datos del clúster y haga clic en **Siguiente**.
7. Mantenga la selección de compatibilidad de ESXi 6.5 predeterminada y haga clic en **Next** (Siguiente).
8. Seleccione el SO invitado del archivo ISO para la máquina virtual que va a crear y haga clic en **Next** (Siguiente).
9. Seleccione el tamaño de disco duro que se requiere para instalar el sistema operativo.
10. Para instalar la aplicación en otro dispositivo, haga clic en **Agregar nuevo dispositivo**.
11. Seleccione las opciones de red y asigne el grupo de puertos distribuido creado para la red pública.
12. Para agregar interfaces de red adicionales, haga clic en **Agregar nuevo dispositivo** y seleccione el grupo de puertos distribuido creado para la red privada.
13. En Nueva unidad de CD o DVD, seleccione el archivo ISO del almacén de datos que contiene el ISO para la instalación del sistema operativo preferido. Seleccione el archivo que ha cargado previamente en la carpeta de archivos ISO y plantillas, y haga clic en **Aceptar**.
14. Revise la configuración y haga clic en **Aceptar** para crear la nueva VM.
15. Encienda la VM. Instale el sistema operativo y las actualizaciones necesarias.

Una vez instalado el sistema operativo, puede clonar una segunda VM. Haga clic con el botón derecho en la entrada de la VM y seleccione la opción Clonar.

### <a name="create-shared-disks-for-vms"></a>Creación de discos compartidos para máquinas virtuales

Oracle utiliza discos compartidos para almacenar los datos, registros y archivos de registro de fase de puesta al día.  Puede crear un disco compartido en vCenter y montarlo en ambas VM.  Para obtener un mayor rendimiento, coloque los discos de datos en diferentes controladoras SCSI. Los pasos que aparecen a continuación le muestran cómo crear un disco compartido en vCenter y luego adjuntarlo a una máquina virtual. El cliente flash de vCenter se usa para modificar las propiedades de la VM.

#### <a name="create-disks-on-the-first-vm"></a>Creación de discos en la primera VM

1. En vCenter, haga clic con el botón derecho en una de las VM de Oracle y seleccione **Editar configuración**.
2. En la sección de nuevo dispositivo, seleccione **Controladora SCSI** y haga clic en **Agregar**.
3. En la sección de nuevo dispositivo, seleccione **Nuevo disco duro** y haga clic en **Agregar**.
4. Expanda las propiedades del nuevo disco duro.
5. Especifique el tamaño del disco duro.
6. Especifique la directiva de almacenamiento de VM para que sea la directiva de almacenamiento de vSAN que definió anteriormente.
7. Seleccione la ubicación como una carpeta en el almacén de datos de vSAN. La ubicación ayuda a explorar y adjuntar los discos a una segunda VM.
8. Para el aprovisionamiento de discos, seleccione **Thick provision eager zeroed** (valor diligente establecido en cero de aprovisionamiento grueso).
9. Para compartir, especifique **Multi-writer** (Multiescritor).
10. Para el nodo de dispositivo virtual, seleccione la nueva controladora SCSI que se creó en el paso 2.

    ![Captura de pantalla en la que se resaltan los campos necesarios para crear discos en la primera máquina virtual.](media/oracle-rac-new-hard-disk.png)

Repita los pasos del 2 al 10 para todos los discos nuevos necesarios para los datos, registros y registros de fase de puesta al día.

#### <a name="attach-disks-to-second-vm"></a>Asociación de discos a la segunda VM

1. En vCenter, haga clic con el botón derecho en una de las VM de Oracle y seleccione **Editar configuración**.
2. En la sección de nuevo dispositivo, seleccione **Controladora SCSI** y haga clic en **Agregar**.
3. En la sección de nuevo dispositivo, seleccione **Disco duro existente** y haga clic en **Agregar**.
4. Vaya a la ubicación donde se creó el disco para la primera VM y seleccione el archivo VMDK.
5. Especifique la directiva de almacenamiento de VM para que sea la directiva de almacenamiento de vSAN que definió anteriormente.
6. Para el aprovisionamiento de discos, seleccione **Thick provision eager zeroed** (valor diligente establecido en cero de aprovisionamiento grueso).
7. Para compartir, especifique **Multi-writer** (Multiescritor).
8. Para el nodo de dispositivo virtual, seleccione la nueva controladora SCSI que se creó en el paso 2.

    ![Creación de discos en la primera VM](media/oracle-rac-existing-hard-disk.png)

Repita los pasos del 2 al 7 para todos los discos nuevos necesarios para los datos, registros y registros de fase de puesta al día de Oracle.

## <a name="set-up-vm-host-affinity-rules"></a>Configuración de reglas de afinidad de VM a host

Las reglas de afinidad de VM a host garantizan que la VM se ejecute en el host deseado.  Puede definir reglas en vCenter para asegurarse de que la VM de Oracle se ejecute en el host con los recursos adecuados y para cumplir con los requisitos de licencia específicos.

1. En el portal de CloudSimple, [escale los privilegios](escalate-private-cloud-privileges.md) del usuario cloudowner.
2. Inicie sesión en el cliente de vSphere de su nube privada.
3. En el cliente de vSphere, seleccione el clúster donde se implementan las VM de Oracle y haga clic en **Configurar**.
4. En Configurar, seleccione **VM/Host Groups** (Grupos host o de VM).
5. Haga clic en **+** .
6. Agregue un grupo de VM. Seleccione el **grupo de VM** como tipo. Escriba el nombre del grupo. Seleccione las VM y, a continuación, haga clic en **Aceptar** para crear el grupo.
6. Agregue un grupo host. Seleccione el **grupo host** como tipo. Escriba el nombre del grupo. Seleccione los hosts en los que se ejecutarán las VM y, a continuación, haga clic en **Aceptar** para crear el grupo.
7. Para crear una regla, haga clic en **VM/Host rules** (Reglas de host o VM).
8. Haga clic en **+** .
9. Escriba un nombre para la regla y seleccione **Habilitar**.
10. En el tipo de regla, seleccione **Virtual Machines to Host** (Máquinas virtuales para el host).
11. Seleccione el grupo de VM que contiene las VM de Oracle.
12. Seleccione **Must run on hosts in this group** (Se debe ejecutar en los hosts de este grupo).
13. Seleccione el grupo host que creó.
14. Haga clic en **Aceptar** para crear la regla.

## <a name="references"></a>Referencias

* [Acerca de las directivas de vSAN](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.virtualsan.doc/GUID-08911FD3-2462-4C1C-AE81-0D4DBC8F7990.html)
* [VMware Multi-Writer Attribute for Shared VMDKs](https://docs.vmware.com/en/VMware-Cloud-on-AWS/solutions/VMware-Cloud-on-AWS.df6735f8b729fee463802083d46fdc75/GUID-A7642A82B3D6C5F7806DB40A3F2766D9.html) (Atributo de multiescritor para archivos VMDK compartidos)
