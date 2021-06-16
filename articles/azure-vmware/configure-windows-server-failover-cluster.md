---
title: Configuración del clúster de conmutación por error de Windows Server en vSAN de Azure VMware Solution
description: Obtenga información sobre cómo configurar el clúster de conmutación por error de Windows Server (WSFC) en vSAN de Azure VMware Solution con discos compartidos nativos.
ms.topic: how-to
ms.date: 05/04/2021
ms.openlocfilehash: f2fc9e712d3f56aeddc6e66c12837794dceb9abe
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111954504"
---
# <a name="configure-windows-server-failover-cluster-on-azure-vmware-solution-vsan"></a>Configuración del clúster de conmutación por error de Windows Server en vSAN de Azure VMware Solution

En este artículo, aprenderá a configurar [clústeres de conmutación por error en Windows Server](/windows-server/failover-clustering/failover-clustering-overview) en vSAN de Azure VMware Solution con discos compartidos nativos. 

>[!IMPORTANT]
>La implementación de este artículo sirve como prueba de concepto y prueba piloto. Se recomienda usar una configuración Cluster-in-a-Box (CIB) hasta que las directivas de selección de ubicación estén disponibles.

El clúster de conmutación por error de Windows Server, conocido anteriormente como Microsoft Service Cluster Service (MSCS), es una característica del sistema operativo (SO) Windows Server. WSFC es una característica crítica para la empresa, necesaria para muchas aplicaciones. Por ejemplo, se requiere WSFC para las siguientes configuraciones:

- SQL Server configurado como:
  - Instancia de clúster de conmutación por error (FCI) Always On, para una alta disponibilidad a nivel de instancia.
  - Grupo de disponibilidad (AG) Always On, para una alta disponibilidad a nivel de base de datos.
- Servicios de archivos de Windows:
  - Recurso compartido de archivos genérico que se ejecuta en el nodo de clúster activo.
  - Servidor de archivos de escalabilidad horizontal (SOFS), que almacena archivos en volúmenes compartidos de clúster (CSV).
  - Espacios de almacenamiento directo (S2D); discos locales utilizados para crear bloques de almacenamiento en distintos nodos del clúster.

Puede hospedar el clúster de WSFC en distintas instancias de Azure VMware Solution, conocidas como Cluster-Across-Box (CAB). También puede colocar el clúster de WSFC en un solo nodo de Azure VMware Solution. Esta configuración se conoce como Cluster-in-a-Box (CIB). No se recomienda usar una solución CIB para una implementación en producción. En caso de error del nodo único de Azure VMware Solution, todos los nodos del clúster de WSFC se apagarían y la aplicación experimentaría tiempo de inactividad. Azure VMware Solution requiere un mínimo de tres nodos en un clúster de nube privada.

Es importante implementar una configuración de WSFC compatible. Querrá que la solución sea compatible con vSphere y Azure VMware Solution. VMware proporciona un documento detallado sobre WSFC en vSphere 6.7, titulado [Configurar clústeres de conmutación por error y el Servicio de clúster de Microsoft](https://docs.vmware.com/en/VMware-vSphere/6.7/vsphere-esxi-vcenter-server-67-setup-mscs.pdf).

Este artículo se centra en WSFC en Windows Server 2016 y Windows Server 2019. Las versiones anteriores de Windows Server no tienen [soporte técnico estándar](https://support.microsoft.com/lifecycle/search?alpha=windows%20server), por lo que no las consideramos aquí.

Primero debe [crear un WSFC](/windows-server/failover-clustering/create-failover-cluster). Use la información que se proporciona en este artículo para conocer las características de una implementación de WSFC en Azure VMware Solution.

## <a name="prerequisites"></a>Requisitos previos

- Entorno de Azure VMware Solution
- Soporte de instalación del SO Microsoft Windows Server

## <a name="reference-architecture"></a>Arquitectura de referencia

Azure VMware Solution proporciona compatibilidad nativa con WSFC virtualizado. Es compatible con las reservas persistentes de SCSI-3 (SCSI3PR) en un nivel de disco virtual. WSFC requiere esta compatibilidad para arbitrar el acceso a un disco compartido entre los nodos. La compatibilidad de SCSI3PR permite la configuración de WSFC con un recurso de disco compartido entre las máquinas virtuales de forma nativa en almacenes de datos de vSAN.

En el diagrama siguiente se ilustra la arquitectura de los nodos virtuales de WSFC en una nube privada de Azure VMware Solution. Muestra dónde reside Azure VMware Solution, incluidos los servidores virtuales de WSFC (cuadro rojo) con respecto a la plataforma de Azure más amplia. En este diagrama se muestra una arquitectura radial típica, pero se puede realizar una configuración similar con el uso de Azure Virtual WAN. Ambas opciones ofrecen todo el valor que otros servicios de Azure pueden aportarle.

:::image type="content" source="media/windows-server-failover-cluster/windows-server-failover-architecture.svg" alt-text="Diagrama de arquitectura de los nodos virtuales del clúster de conmutación por error de Windows Server en una nube privada de Azure VMware Solution." border="false" lightbox="media/windows-server-failover-cluster/windows-server-failover-architecture.svg":::

## <a name="supported-configurations"></a>Configuraciones admitidas

Actualmente, las configuraciones admitidas son:

- Microsoft Windows Server 2012 o posterior
- Hasta cinco nodos de clústeres de conmutación por error por clúster
- Hasta cuatro adaptadores de PVSCSI por VM
- Hasta 64 discos por adaptador de PVSCSI

## <a name="virtual-machine-configuration-requirements"></a>Requisitos de configuración de la máquina virtual

### <a name="wsfc-node-configuration-parameters"></a>Parámetros de configuración de los nodos de WSFC

- Instale las herramientas de VMware más recientes en cada nodo de WSFC.
- No se admite la combinación de discos no compartidos y compartidos en un único adaptador SCSI virtual. Por ejemplo, si el disco del sistema (unidad C:) está asociado a SCSI0:0, el primer disco compartido se conectaría a SCSI1:0. Un nodo de máquina virtual de un WSFC tiene el mismo controlador SCSI virtual máximo que una máquina virtual normal, hasta cuatro (4) controladores SCSI virtuales.
- Los identificadores SCSI de los discos virtuales deben ser coherentes entre todas las máquinas virtuales que hospedan nodos del mismo WSFC.

| **Componente** | **Requisitos** |
| --- | --- |
| Versión de hardware de la VM | 11 o posterior para admitir vMotion en vivo. |
| NIC virtual | Tarjeta de interfaz de red (NIC) paravirtualizada VMXNET3; habilite el escalado en lado de recepción (RSS) de Windows en el invitado en la NIC virtual. |
| Memoria | Use la memoria de reserva de VM completa para los nodos del clúster de WSFC. |
| Aumentar el tiempo de espera de E/S de cada nodo de WSFC | Modifique HKEY\_LOCAL\_MACHINE\System\CurrentControlSet\Services\Disk\TimeOutValueSet a 60 segundos o más. (Si vuelve a crear el clúster, es posible que este valor se restablezca a su valor predeterminado, por lo que debe cambiarlo de nuevo). |
| Seguimiento de estado del clúster de Windows | El valor del parámetro SameSubnetThreshold del seguimiento de estado del clúster de Windows debe modificarse para permitir 10 latidos perdidos como mínimo. Este es [el valor predeterminado en Windows Server 2016](https://techcommunity.microsoft.com/t5/failover-clustering/tuning-failover-cluster-network-thresholds/ba-p/371834). Esta recomendación se aplica a todas las aplicaciones que usan WSFC, incluidos los discos compartidos y no compartidos. |

### <a name="wsfc-node---boot-disks-configuration-parameters"></a>Nodo de WSFC: parámetros de configuración de discos de arranque


| **Componente** | **Requisitos** |
| --- | --- |
| Tipo de controlador SCSI | LSI Logic SAS |
| Modo de disco | Las máquinas |
| Uso compartido del bus SCSI | Ninguno |
| Modificar la configuración avanzada de un controlador SCSI virtual que hospeda el dispositivo de arranque | Agregue la siguiente configuración avanzada a cada nodo de WSFC:<br /> scsiX.returnNoConnectDuringAPD = "TRUE"<br />scsiX.returnBusyOnNoConnectStatus = "FALSE"<br />Donde X es el número de id. del controlador de bus SCSI del dispositivo de arranque. De forma predeterminada, X se estable en 0. |

### <a name="wsfc-node---shared-disks-configuration-parameters"></a>Nodo de WSFC: parámetros de configuración de discos compartidos


| **Componente** | **Requisitos** |
| --- | --- |
| Tipo de controlador SCSI | VMware paravirtualizado (PVSCSI) |
| Modo de disco | Independiente: persistente (paso 2 en la ilustración siguiente). Con esta configuración, se asegura de que todos los discos se excluyan de las instantáneas. Las instantáneas no se admiten para las máquinas virtuales basadas en WSFC. |
| Uso compartido del bus SCSI | Físico (paso 1 de la ilustración siguiente) |
| Marca Multi-writer | No se usa |
| Formato de disco | Aprovisionamiento grueso. (El aprovisionamiento grueso de puesta a cero rápida (EZT) no es necesario con vSAN). |

:::image type="content" source="media/windows-server-failover-cluster/edit-settings-virtual-hardware.png" alt-text="Captura de pantalla que muestra la página Editar configuración del hardware virtual":::.

## <a name="non-supported-scenarios"></a>Escenarios no compatibles

Las siguientes funcionalidades no son compatibles con WSFC en Azure VMware Solution:

- Almacenes de datos de NFS
- Espacios de almacenamiento
- vSAN con el servicio iSCSI
- Clúster extendido de vSAN
- Compatibilidad mejorada con vMotion (EVC)
- Tolerancia a errores (FT) de vSphere
- Instantáneas
- Storage vMotion en vivo (en línea)
- Virtualización de id. de N-Port (NPIV)

Los cambios frecuentes en el hardware de las máquinas virtuales pueden interrumpir el latido entre los nodos de WSFC.

No se admiten las siguientes actividades y podrían provocar la conmutación por error del nodo de WSFC:

- Adición de memoria de acceso frecuente
- Adición de CPU de acceso frecuente
- Uso de instantáneas
- Aumento del tamaño de un disco compartido
- Pausa y reanudación del estado de la máquina virtual
- Compromiso excesivo de memoria que provoca al intercambio de ESXi o la dilatación de la memoria de la máquina virtual
- Extensión frecuente del archivo VMDK local, aunque no esté asociado con el controlador de uso compartido del bus SCSI

## <a name="configure-wsfc-with-shared-disks-on-azure-vmware-solution-vsan"></a>Configuración de WSFC con discos compartidos en vSAN de Azure VMware Solution

1. Asegúrese de que haya un entorno de Active Directory disponible.
2. Cree máquinas virtuales (VM) en el almacén de datos de vSAN.
3. Encienda todas las máquinas virtuales, configure el nombre de host, las direcciones IP, una todas las máquinas virtuales a un dominio de Active Directory e instale las actualizaciones del sistema operativo disponibles más recientes.
4. Instale las herramientas de VMware más recientes.
5. Habilite y configure la característica del clúster de conmutación por error de Windows Server en cada máquina virtual.
6. Configure un testigo del clúster para el cuórum (un testigo de recurso compartido de archivos funciona correctamente).
7. Apague todos los nodos del clúster de WSFC.
8. Agregue uno o varios controladores SCSI paravirtuales (hasta cuatro) a cada parte de la máquina virtual del WSFC. Use la configuración de los párrafos anteriores.
9. En el primer nodo del clúster, agregue todos los discos compartidos necesarios mediante **Agregar nuevo dispositivo** > **Disco duro**. El uso compartido de discos debe dejarse como **No especificado** (valor predeterminado) y el modo de disco como **Independiente: persistente**. Conéctelo a los controladores creados en los pasos anteriores.
10. Continúe con los demás nodos de WSFC. Agregue los discos creados en el paso anterior; para ello, seleccione **Agregar nuevo dispositivo** > **Disco duro existente**. Asegúrese de mantener los mismos identificadores SCSI de disco en todos los nodos de WSFC.
11. Encienda el primer nodo de WSFC. Inicie sesión y abra la consola de administración de discos (MMC). Asegúrese de que el sistema operativo pueda administrar los discos compartidos agregados y de que se hayan inicializado. Formatee los discos y asigne una letra de unidad.
12. Encienda los otros nodos de WSFC.
13. Agregue el disco al clúster de WSFC mediante el **Asistente para agregar discos** y agréguelos a un volumen compartido de clúster.
14. Pruebe una conmutación por error con el **Asistente para migrar discos** y asegúrese de que el clúster de WSFC con discos compartidos funciona correctamente.
15. Ejecute el **Asistente para validación del clúster** para confirmar si el clúster y sus nodos funcionan correctamente.

    Es importante tener en cuenta los siguientes elementos específicos de la prueba de validación del clúster:

       - **Validar la reserva persistente de espacios de almacenamiento**. Si no usa espacios de almacenamiento con el clúster (por ejemplo, vSAN de Azure VMware Solution), esta prueba no es aplicable. Puede omitir los resultados de la prueba Validar la reserva persistente de espacios de almacenamiento, incluida esta advertencia. Para evitar las advertencias, puede excluir esta prueba.
        
      - **Validar la comunicación de red**. La prueba de Validación de clúster lanzará una advertencia que indica que solo hay disponible una interfaz de red por nodo de clúster. Puede ignorar esta advertencia. Azure VMware Solution proporciona la disponibilidad y el rendimiento necesarios, ya que los nodos se conectan a uno de los segmentos de NSX-T. Sin embargo, conserve este elemento como parte de la prueba de validación del clúster, ya que validará otros aspectos de la comunicación de red.

16. Cree una regla de DRS para colocar las máquinas virtuales de WSFC en los mismos nodos de Azure VMware Solution. Para ello, necesita una regla de afinidad entre el host y la máquina virtual. De esta manera, los nodos de clúster se ejecutarán en el mismo host de Azure VMware Solution. Una vez más, se trata de una prueba piloto hasta que las directivas de selección de ubicación estén disponibles.

    >[!NOTE]
    > Para ello, debe crear un vale de solicitud de soporte técnico. Nuestra organización de soporte técnico de Azure podrá ayudarle con esto.

## <a name="related-information"></a>Información relacionada

- [Clústeres de conmutación por error de Windows Server](/windows-server/failover-clustering/failover-clustering-overview)
- [Directrices para la agrupación en clústeres de Microsoft en vSphere (1037959) (vmware.com)](https://kb.vmware.com/s/article/1037959)
- [Acerca de la configuración de los clústeres de conmutación por error y el Servicio de clúster de Microsoft (vmware.com)](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.mscs.doc/GUID-1A2476C0-CA66-4B80-B6F9-8421B6983808.html)
- [vSAN 6.7 U3 - WSFC con discos compartidos y reservas persistentes de SCSI-3 (vmware.com)](https://blogs.vmware.com/virtualblocks/2019/08/23/vsan67-u3-wsfc-shared-disksupport/)
- [Límites de Azure VMware Solution](../azure-resource-manager/management/azure-subscription-service-limits.md#azure-vmware-solution-limits)

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha tratado la configuración de un WSFC en Azure VMware Solution, puede que quiera obtener información sobre lo siguiente:

- Configurar el nuevo WSFC agregando más aplicaciones que requieran la funcionalidad de WSFC. Por ejemplo, SQL Server y ASCS de SAP.
- Configurar una solución de copia de seguridad.
  - [Configuración de Azure Backup Server para Azure VMware Solution](../backup/backup-azure-microsoft-azure-backup.md?context=%2fazure%2fazure-vmware%2fcontext%2fcontext)
  - [Soluciones de copia de seguridad para máquinas virtuales de Azure VMware Solution](../backup/backup-azure-backup-server-vmware.md?context=%2fazure%2fazure-vmware%2fcontext%2fcontext)