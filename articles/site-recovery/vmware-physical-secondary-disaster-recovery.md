---
title: Recuperación ante desastres de máquinas virtuales de VMware o de servidores físicos en un sitio secundario con Azure Site Recovery
description: Obtenga información sobre cómo configurar la recuperación ante desastres de máquinas virtuales de VMware o servidores físicos Windows y Linux en un sitio secundario con Azure Site Recovery.
ms.service: site-recovery
services: site-recovery
ms.topic: conceptual
ms.date: 11/05/2019
ms.openlocfilehash: c2456fcfffd9645f94856998142f262298b5eef9
ms.sourcegitcommit: d63f15674f74d908f4017176f8eddf0283f3fac8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106581459"
---
# <a name="set-up-disaster-recovery-of-on-premises-vmware-virtual-machines-or-physical-servers-to-a-secondary-site"></a>Configuración de la recuperación ante desastres de máquinas virtuales de VMware o de servidores físicos locales en un sitio secundario

InMage Scout en [Azure Site Recovery](site-recovery-overview.md) proporciona características de replicación en tiempo real entre los sitios locales de VMware. InMage Scout se incluye en las suscripciones al servicio Azure Site Recovery.

## <a name="end-of-support-announcement"></a>Anuncio de fin del soporte técnico

El escenario de Azure Site Recovery para la replicación entre los centros de datos físicos o de VMware locales está llegando al fin del soporte técnico.

-   Desde agosto de 2018, no se puede configurar el escenario en el almacén de Recovery Services y el software InMage Scout no se puede descargar desde el almacén. Se seguirá ofreciendo soporte técnico a las implementaciones existentes. 
-   Desde el 31 de diciembre de 2020, no se ofrecerá soporte técnico al escenario.
- Los asociados existentes pueden incorporar nuevos clientes al escenario hasta que finalice el soporte técnico.

Durante la 2018 y 2019, se lanzarán dos actualizaciones: 

-   Actualización 7: corrige problemas de cumplimiento y configuración de red y proporciona compatibilidad con TLS 1.2.
-   Actualización 8: agrega compatibilidad para sistemas operativos Linux RHEL/CentOS 7.3/7.4/7.5 y para SUSE 12.

Después de la actualizaciones 8, no se publicarán más actualizaciones. Habrá soporte técnico de revisiones limitadas para los sistemas operativos agregados en la actualización 8 y correcciones de errores basadas en el mejor esfuerzo.

Azure Site Recovery continúa innovando; para ello, proporciona a los clientes de VMware e Hyper-V una solución de DRaaS avanzada y sin problemas con Azure como un sitio de recuperación ante desastres. Microsoft recomienda que los clientes de InMage y ASR Count existentes consideren el uso del escenario de VMware a Azure de Azure Site Recovery para sus necesidades de continuidad del negocio. El escenario de VMware a Azure de Azure Site Recovery es una solución de recuperación ante desastres de clase empresarial para aplicaciones de VMware que ofrece RPO y RTO de minutos, compatibilidad con la replicación y recuperación de la aplicación de varias máquinas virtuales, incorporación sin problemas, supervisión completa una ventaja de costo total de propiedad considerable.

### <a name="scenario-migration"></a>Migración del escenario
Como alternativa, se recomienda configurar la recuperación ante desastres para máquinas virtuales de VMware locales y máquinas físicas replicándolas en Azure. Para ello, realice lo siguiente:

1.  Revise la comparación rápida a continuación. Antes de poder replicar máquinas locales, debe comprobar que cumplen los [requisitos](./vmware-physical-azure-support-matrix.md#replicated-machines) para la replicación en Azure. Si va a replicar máquinas virtuales de VMware, se recomienda que revise las [pautas de planeamiento de capacidad](./site-recovery-plan-capacity-vmware.md) y ejecute la [herramienta Deployment Planner](./site-recovery-deployment-planner.md) para identificar los requisitos de capacidad de identidad y comprobar el cumplimiento.
2.  Después de ejecutar Deployment Planner, puede configurar la replicación: o   Para máquinas virtuales de VMware, siga estos tutoriales para [preparar Azure](./tutorial-prepare-azure.md), [preparar el entorno de VMware local](./vmware-azure-tutorial-prepare-on-premises.md) y [configurar la recuperación ante desastres](./vmware-azure-tutorial-prepare-on-premises.md).
o   Para máquinas físicas, siga este [tutorial](./physical-azure-disaster-recovery.md).
3.  Después de que las máquinas se estén replicando en Azure, puede ejecutar un [simulacro de recuperación ante desastres](./site-recovery-test-failover-to-azure.md) para asegurarse de que todo funciona según lo previsto.

### <a name="quick-comparison"></a>Comparación rápida

**Característica** | **Replicación en Azure** |**Replicación entre centros de datos de VMware**
--|--|--
**Componentes necesarios** |Servicio de movilidad en las máquinas replicadas. Servidor de configuración local, servidor de procesos, servidor de destino maestro. Servidor de procesos temporal en Azure para conmutación por recuperación.|Servidor de movilidad, servidor de procesos, servidor de configuración y destino principal
**Configuración y orquestación** |Almacén de Recovery Services en Azure Portal | Usando vContinuum 
**Replicado** |Disco (Windows y Linux) |Volumen-Windows<br> Disco-Linux
**Clúster de discos compartido** |No compatible|Compatible
**Límites de renovación de datos (promedio)** |10 MB/s de datos por disco<br> 25 MB/s de datos por máquina virtual<br> [Más información](./site-recovery-vmware-deployment-planner-analyze-report.md#azure-site-recovery-limits) | > 10 MB/s de datos por disco  <br> > 25 MB/s de datos por máquina virtual
**Supervisión** |Desde Azure Portal|Desde CX (servidor de configuración)
**Matrices compatibles** | [Haga clic aquí para obtener información detallada](./vmware-physical-azure-support-matrix.md)|[Descarga de matriz compatible de ASR Scout](https://aka.ms/asr-scout-cm)


## <a name="prerequisites"></a>Requisitos previos
Para completar este tutorial:

- [Revise](vmware-physical-secondary-support-matrix.md) los requisitos de compatibilidad de todos los componentes.
- Asegúrese de que las máquinas que desea replicar cumplen con la [compatibilidad con máquinas replicadas](vmware-physical-secondary-support-matrix.md#replicated-vm-support).


## <a name="download-and-install-component-updates"></a>Descarga e instalación de actualizaciones de componentes

 Revise e instale las [actualizaciones](#updates) más recientes. Las actualizaciones se deben instalar en los servidores en este orden:

1. Servidor RX (si corresponde)
2. Servidores de configuración
3. Servidores de proceso
4. Servidores de destino maestros
5. Servidores de vContinuum
6. Servidor de origen (tanto servidor Windows como Linux)

Instale las actualizaciones de la siguiente manera:

> [!NOTE]
>La versión de actualización del archivo de todos los componentes de Scout no puede ser la misma que la del archivo .zip de la actualización. La versión anterior indica que no hay ningún cambio en el componente desde la actualización anterior a esta.

Descargue el archivo .zip de [actualización](https://aka.ms/asr-scout-update7) y los archivos de configuración de [actualización de MySQL y PHP](https://aka.ms/asr-scout-u7-mysql-php-manualupgrade). El archivo .zip de actualización contiene todos los binarios base y los binarios de actualización acumulativa de los componentes siguientes: 
- InMage_ScoutCloud_RX_8.0.1.0_RHEL6-64_GA_02Mar2015.tar.gz
- RX_8.0.7.0_GA_Update_7_2965621_28Dec18.tar.gz
- InMage_CX_8.0.1.0_Windows_GA_26Feb2015_release.exe
- InMage_CX_TP_8.0.1.0_Windows_GA_26Feb2015_release.exe
- CX_Windows_8.0.7.0_GA_Update_7_2965621_28Dec18.exe
- InMage_PI_8.0.1.0_Windows_GA_26Feb2015_release.exe
- InMage_Scout_vContinuum_MT_8.0.7.0_Windows_GA_27Dec2018_release.exe
- InMage_UA_8.0.7.0_Windows_GA_27Dec2018_release.exe
- InMage_UA_8.0.7.0_OL5-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_OL5-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_OL6-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_OL6-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_RHEL5-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_RHEL5-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_RHEL6-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_RHEL6-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_RHEL7-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP1-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP1-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP2-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP2-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP3-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP3-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP4-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES10-SP4-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-64_GA_04Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP1-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP1-64_GA_04Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP2-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP2-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP3-32_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP3-64_GA_03Dec2018_release.tar.gz
- InMage_UA_8.0.7.0_SLES11-SP4-64_GA_03Dec2018_release.tar.gz
  1. Extraiga los archivos .zip.
  2. **Servidor de RX**: Copie **RX_8.0.7.0_GA_Update_7_2965621_28Dec18.tar.gz** en el servidor RX y extráigalo. En la carpeta extraída, ejecute **/Install**.
  3. **Servidor de configuración y servidor de procesos**: Copie **CX_Windows_8.0.7.0_GA_Update_7_2965621_28Dec18.exe** en el servidor de configuración y el servidor de procesos. Haga doble clic para ejecutarlo.<br>
  4. **Servidor de destino principal de Windows**: Para actualizar el agente unificado, copie **InMage_UA_8.0.7.0_Windows_GA_27Dec2018_release.exe** en el servidor. Haga doble clic en él para ejecutarlo. También puede utilizarse el mismo archivo para una instalación nueva. La misma actualización del agente unificado también se aplica al servidor de origen.
  La actualización no necesita aplicarse en el destino maestro preparado con **InMage_Scout_vContinuum_MT_8.0.7.0_Windows_GA_27Dec2018_release.exe** puesto que este es el nuevo instalador de disponibilidad general con todos los cambios más recientes.
  5. **Servidor de vContinuum**:  Copie **InMage_Scout_vContinuum_MT_8.0.7.0_Windows_GA_27Dec2018_release.exe** en el servidor.  Asegúrese de que ha cerrado el asistente de vContinuum. Haga doble clic en el archivo para ejecutarlo.
  6. **Servidor de destino maestro Linux**: Para actualizar el agente unificado, copie **InMage_UA_8.0.7.0_RHEL6-64_GA_03Dec2018_release.tar.gz** en el servidor de destino maestro de Linux y extráigalo. En la carpeta extraída, ejecute **/Install**.
  7. **Servidor de origen Windows**: Para actualizar el agente unificado, copie **InMage_UA_8.0.7.0_Windows_GA_27Dec2018_release.exe** en el servidor de origen. Haga doble clic en el archivo para ejecutarlo. 
  8. **Servidor de origen Linux**: para actualizar el agente unificado, copie la versión correspondiente del archivo del agente unificado en el servidor Linux y extráigalo. En la carpeta extraída, ejecute **/Install**.  Ejemplo: Para el servidor RHEL 6.7 de 64 bits, copie **InMage_UA_8.0.7.0_RHEL6-64_GA_03Dec2018_release.tar.gz** en el servidor y extráigalo. En la carpeta extraída, ejecute **/Install**.
  9. Después de actualizar el servidor de configuración, el servidor de procesos y el servidor RX con los instaladores mencionados anteriormente, debe actualizar manualmente las bibliotecas de PHP y MySQL con los pasos mencionados en la sección 7.4 de la [guía de instalación rápida](https://aka.ms/asr-scout-quick-install-guide).

## <a name="enable-replication"></a>Habilitar replicación

1. Configure la replicación entre los sitios de origen y de destino de VMware.
2. Consulte los documentos siguientes para más información sobre la instalación, la protección y la recuperación:

   * [Notas de la versión](https://aka.ms/asr-scout-release-notes)
   * [Matriz de compatibilidad](https://aka.ms/asr-scout-cm)
   * [Guía de usuario](https://aka.ms/asr-scout-user-guide)
   * [Guía de usuario de RX](https://aka.ms/asr-scout-rx-user-guide)
   * [Guía de instalación rápida](https://aka.ms/asr-scout-quick-install-guide)
   * [Actualización de bibliotecas PHP y MYSQL](https://aka.ms/asr-scout-u7-mysql-php-manualupgrade)

## <a name="updates"></a>Actualizaciones

### <a name="site-recovery-scout-801-update-7"></a>Site Recovery Scout 8.0.1 Update 7 
Actualizado: 31 de diciembre de 2018 Descargue [Scout Update 7](https://aka.ms/asr-scout-update7).
Scout Update 7 es un instalador completo que se puede usar para una instalación nueva, así como para actualizar los agentes/MT existentes que se encuentran en las actualizaciones anteriores (desde las versiones de Update 1 a 6). Contiene todas las correcciones desde Update 1 hasta Update 6, además de las correcciones nuevas y mejoras que se describen a continuación.
 
#### <a name="new-features"></a>Nuevas características
* Cumplimiento con PCI
* Compatibilidad con TLS v1.2

#### <a name="bug-and-security-fixes"></a>Errores y revisiones de seguridad
* Solucionado: Las máquinas independientes o en clúster de Windows tienen una configuración IP incorrecta en la exploración de DR o la recuperación.
* Solucionado: La operación de agregar disco a veces da error en el clúster V2V.
* Problema corregido: El Asistente de vContinuum se bloquea durante la fase de recuperación si el destino maestro es Windows Server 2016
* Solucionado: Los problemas de seguridad de MySQL se mitigan al actualizar MySQL a la versión 5.7.23

#### <a name="manual-upgrade-for-php-and-mysql-on-csps-and-rx"></a>Actualización manual para PHP y MySQL en CS, PS y RX
La plataforma de scripting PHP debe actualizarse a la versión 7.2.10 en el servidor de configuración, el servidor de procesos y el servidor RX.
El sistema de administración de bases de datos de MySQL debe actualizarse a la versión 5.7.23 en el servidor de configuración, el servidor de procesos y el servidor RX.
Siga los pasos manuales de la [guía de instalación rápida](https://aka.ms/asr-scout-quick-install-guide) para actualizar las versiones PHP y MySQL.

### <a name="site-recovery-scout-801-update-6"></a>Site Recovery Scout 8.0.1 Update 6 
Actualizado: 12 de octubre de 2017

Descargue la [actualización 6 de Scout](https://aka.ms/asr-scout-update6).

La actualización 6 de Scout es una actualización acumulativa. Contiene todas las correcciones desde Update 1 hasta Update 5, además de las correcciones nuevas y mejoras que se describen a continuación. 

#### <a name="new-platform-support"></a>Nueva compatibilidad con plataformas
* Se ha agregado compatibilidad con Windows Server 2016 de origen
* Se ha agregado compatibilidad con los siguientes sistemas operativos Linux:
    - Red Hat Enterprise Linux (RHEL) 6.9
    - CentOS 6.9
    - Oracle Linux 5.11
    - Oracle Linux 6.8
* Se ha agregado compatibilidad con VMware Center 6.5.

Instale las actualizaciones de la siguiente manera:

> [!NOTE]
>La versión de actualización del archivo de todos los componentes de Scout no puede ser la misma que la del archivo .zip de la actualización. La versión anterior indica que no hay ningún cambio en el componente desde la actualización anterior a esta.

Descargue el archivo .zip de la [actualización](https://aka.ms/asr-scout-update6). El archivo contiene los siguientes componentes: 
- RX_8.0.4.0_GA_Update_4_8725872_16Sep16.tar.gz
- CX_Windows_8.0.6.0_GA_Update_6_13746667_18Sep17.exe
- UA_Windows_8.0.5.0_GA_Update_5_11525802_20Apr17.exe
- UA_RHEL6-64_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz
- vCon_Windows_8.0.6.0_GA_Update_6_11525767_21Sep17.exe
- Actualización de UA de 4 bits para RHEL5, OL5, OL6, SUSE 10, SUSE 11: UA_\<Linux OS>_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz
  1. Extraiga los archivos .zip.
  2. **Servidor de RX**: copie **RX_8.0.4.0_GA_Update_4_8725872_16Sep16.tar.gz** en el servidor RX y extráigalo. En la carpeta extraída, ejecute **/Install**.
  3. **Servidor de configuración y servidor de procesos**: copie **CX_Windows_8.0.6.0_GA_Update_6_13746667_18Sep17.exe** en el servidor de configuración y el servidor de procesos. Haga doble clic para ejecutarlo.<br>
  4. **Servidor de destino principal de Windows**: Para actualizar el agente unificado, copie **UA_Windows_8.0.5.0_GA_Update_5_11525802_20Apr17.exe** en el servidor. Haga doble clic en él para ejecutarlo. La misma actualización del agente unificado también se aplica al servidor de origen. Si el origen no se ha actualizado a Update 4, debe actualizar el agente unificado.
  La actualización no necesita aplicarse en el destino maestro preparado con **InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_10Oct2017_release.exe** puesto que este es el nuevo instalador de disponibilidad general con todos los cambios más recientes.
  5. **Servidor de vContinuum**:  copie **vCon_Windows_8.0.6.0_GA_Update_6_11525767_21Sep17.exe** en el servidor.  Asegúrese de que ha cerrado el asistente de vContinuum. Haga doble clic en el archivo para ejecutarlo.
  La actualización no tiene que aplicarse en el destino maestro preparado con **InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_10Oct2017_release.exe** puesto que este es el nuevo instalador de GA con todos los cambios más recientes.
  6. **Servidor de destino maestro Linux**: para actualizar el agente unificado, copie **UA_RHEL6-64_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz** en el servidor de destino maestro y extráigalo. En la carpeta extraída, ejecute **/Install**.
  7. **Servidor de origen Windows**: Para actualizar el agente unificado, copie **UA_Windows_8.0.5.0_GA_Update_5_11525802_20Apr17.exe** en el servidor de origen. Haga doble clic en el archivo para ejecutarlo. 
  No es necesario instalar el agente de Update 5 en el servidor de origen si ya se ha actualizado a Update 4, o si se ha instalado el agente de origen con el instalador de base más reciente **InMage_UA_8.0.1.0_Windows_GA_28Sep2017_release.exe**.
  8. **Servidor de origen Linux**: para actualizar el agente unificado, copie la versión correspondiente del archivo del agente unificado en el servidor Linux y extráigalo. En la carpeta extraída, ejecute **/Install**.  Ejemplo: Para el servidor RHEL 6.7 de 64 bits, copie **UA_RHEL6-64_8.0.4.0_GA_Update_4_9035261_26Sep16.tar.gz** en el servidor y extráigalo. En la carpeta extraída, ejecute **/Install**.


> [!NOTE]
> * Se ha actualizado el instalador de base del agente unificado para Windows para admitir Windows Server 2016. El nuevo instalador **InMage_UA_8.0.1.0_Windows_GA_28Sep2017_release.exe** está empaquetado con el paquete de base de Scout GA (**InMage_Scout_Standard_8.0.1 GA-Oct17.zip**). El mismo instalador se usará para todas las versiones compatibles de Windows. 
> * El instalador de destino maestro y vContinuum de Windows de base se ha actualizado para admitir Windows Server 2016. El nuevo instalador **InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_10Oct2017_release.exe** está empaquetado con el paquete de base de disponibilidad general de Scout (**InMage_Scout_Standard_8.0.1 GA-Oct17.zip**). El mismo instalador se utilizará para implementar el destino maestro de Windows 2016 y el destino maestro de Windows 2012 R2.
> * ASR Scout no admite Windows Server 2016 en un servidor físico. Solo admite la máquina virtual de VMware de Windows Server 2016. 
>

#### <a name="bug-fixes-and-enhancements"></a>Mejoras y correcciones de errores
- Se ha producido un error de protección de conmutación por recuperación para la máquina virtual Linux con la lista de discos que se van a replicar está vacía al final de la configuración.

### <a name="site-recovery-scout-801-update-5"></a>Site Recovery Scout 8.0.1 Update 5
La actualización 5 de Scout es una actualización acumulativa. Contiene todas las correcciones desde Update 1 hasta Update 4 y las correcciones nuevas que se describen a continuación.
- Las correcciones de Site Recovery Scout Update 4 a Update 5 son específicamente para los componentes de vContinuum y destino maestro.
- Si los servidores de origen, el destino maestro, la configuración, el proceso y los servidores RX ya ejecutan Update 4, aplíquela solo en el servidor de destino maestro. 

#### <a name="new-platform-support"></a>Nueva compatibilidad con plataformas
* SUSE Linux Enterprise Server 11 Service Pack 4 (SP4)
* **InMage_UA_8.0.1.0_SLES11-SP4-64_GA_13Apr2017_release.tar.gz** de SLES 11 SP4 de 64 bits se incluye con el paquete base de Scout GA (**InMage_Scout_Standard_8.0.1 GA.zip**). Descargue el paquete de GA del portal, como se describe en el artículo sobre la creación de un almacén.


#### <a name="bug-fixes-and-enhancements"></a>Mejoras y correcciones de errores

* Correcciones para aumentar la confiabilidad de la compatibilidad con clústeres de Windows:
    * Corregido: algunos de los discos de clúster de MSCS P2V se convierten en RAW después de la recuperación.
    * Corregido: se produce un error en la recuperación de un clúster de MSCS P2V debido a una falta de coincidencia en el orden del disco.
    * Corregido: error durante la operación de clúster de MSCS para agregar discos debido a una falta de coincidencia en el tamaño del disco.
    * Corregido: error en la comprobación de la preparación de la asignación del clúster de MSCS de origen con LUN de RDM en la comprobación del tamaño.
    * Corregido: error en la protección del clúster de nodo único debido a un problema de falta de coincidencia de SCSI. 
    * Corregido: error al volver a proteger el servidor de clúster de Windows P2V si hay discos del clúster de destino. 
    
* Solucionado: durante la protección de conmutación por recuperación, si el servidor de destino maestro seleccionado no está en el mismo servidor ESXi que el de la máquina de origen protegida (durante la protección directa), vContinuum elige el servidor de destino maestro equivocado durante la recuperación de la conmutación por recuperación y, posteriormente, se produce un error en la operación de recuperación.

> [!NOTE]
> * Las correcciones de los clústeres P2V solo son aplicables a los clústeres de MSCS físicos que se acaban de proteger con Site Recovery Scout Update 5. Para instalar las correcciones de los clústeres en clústeres de MSCS P2V protegidos con actualizaciones anteriores, siga los pasos de actualización mencionados en la sección 12 de las [notas de la versión de Site Recovery Scout](https://aka.ms/asr-scout-release-notes).
> * Si en el momento de volver a protegerlos, el mismo conjunto de discos está activo en cada uno de los nodos de clúster que cuando los protegió inicialmente, por lo que la nueva protección de un clúster de MSCS físico solo puede reusar discos de destino existentes. En caso contrario, use los pasos manuales de la sección 12 de las [notas de la versión de Site Recovery Scout](https://aka.ms/asr-scout-release-notes) para mover los discos del lado de destino a la ruta de acceso correcta al almacén de datos, para volver a usarlos cuando se vuelva a realizar la protección. Si vuelve a proteger el clúster de MSCS en modo P2V sin seguir los pasos de actualización, crea un disco en el servidor ESXi de destino. Deberá eliminar manualmente los discos antiguos del almacén de datos.
> * Cuando un origen SLES11 o SLES11 de origen (con cualquier Service Pack) se reinicia correctamente, debe marcar manualmente los pares de replicación de disco **raíz** para volver a sincronizarlos. No hay ninguna notificación en la interfaz de CX.  Si no marca el disco raíz para resincronización, es posible que note problemas con la integridad de datos.


### <a name="azure-site-recovery-scout-801-update-4"></a>Azure Site Recovery Scout 8.0.1, actualización 4
La actualización 4 de Scout es una actualización acumulativa. Incluye todas las correcciones desde Update 1 hasta Update 3 y las correcciones nuevas que se describen a continuación.

#### <a name="new-platform-support"></a>Nueva compatibilidad con plataformas

* Se ha agregado compatibilidad con vCenter/vSphere 6.0, 6.1 y 6.2.
* Se agregó compatibilidad con estos sistemas operativos Linux:
  * Red Hat Enterprise Linux (RHEL) 7.0, 7.1 y 7.2
  * CentOS 7.0, 7.1 y 7.2
  * Red Hat Enterprise Linux (RHEL) 6.8
  * CentOS 6.8

> [!NOTE]
> **InMage_UA_8.0.1.0_RHEL7-64_GA_06Oct2016_release.tar.gz** de RHEL/CentOS 7 de 64 bits se incluye con el paquete base de Scout GA **InMage_Scout_Standard_8.0.1 GA.zip**. Descargue el paquete de Scout GA del portal, como se describe en el artículo sobre la creación de un almacén.

#### <a name="bug-fixes-and-enhancements"></a>Mejoras y correcciones de errores

* Mejora en la administración del apagado para los siguientes clones y sistemas operativos Linux para evitar problemas no deseados de resincronización:
    * Red Hat Enterprise Linux (RHEL) 6.x
    * Oracle Linux (OL) 6.x
* Para Linux, todos los permisos de acceso a carpetas del directorio de instalación del agente unificado ahora están restringidos exclusivamente al usuario local.
* En Windows, una corrección para un problema de agotamiento del tiempo de espera al emitir marcadores comunes de coherencia distribuida en aplicaciones distribuidas de alta carga, como los clústeres de SQL Server y SharePoint.
* Una corrección relacionada con el registro en el instalador de la base del servidor de configuración.
* Se agregó un vínculo de descarga a VMware vCLI 6.0 al instalador de la base de destino maestro de Windows.
* Se agregaron registros y comprobaciones adicionales para los cambios en la configuración de red durante maniobras de conmutación por error y recuperación ante desastres.
* Corrección de un problema que ocasionaba que la información de retención no se informaba al servidor de configuración.  
* En el caso de los clústeres físicos, la corrección de un problema que provocaba un error del redimensionamiento del volumen en el asistente de vContinuum al reducir el volumen de origen.
* Una corrección para un problema de protección de clústeres con error: "No se pudo encontrar la firma de disco", cuando el disco de clúster es un disco PRDM.
* Corrección de un bloqueo del servidor de transporte cxps provocado por una excepción fuera de intervalo.
* Ahora se puede modificar el tamaño de las columnas Nombre del servidor y Dirección IP en la página **Instalación de inserción** del asistente de vContinuum.
* Mejoras de la API de RX:
  * Ahora están disponibles los cinco puntos de coherencia comunes más recientes (solo las etiquetas garantizadas).
  * Se muestran los detalles de espacio libre y capacidad para todos los dispositivos protegidos.
  * El estado del controlador de Scout en el servidor de origen está disponible.

> [!NOTE]
> * El paquete base **InMage_Scout_Standard_8.0.1_GA.zip** incluye:
>     * Un instalador de base del servidor de configuración actualizado (**InMage_CX_8.0.1.0_Windows_GA_26Feb2015_release.exe**)
>     * Un instalador de base de destino maestro de Windows (**InMage_Scout_vContinuum_MT_8.0.1.0_Windows_GA_26Feb2015_release.exe**).
>     * Para todas las instalaciones nuevas, use el servidor de configuración nuevo y los bits de GA del destino maestro de Windows.
> * Update 4 se puede aplicar directamente en 8.0.1 GA.
> * Las actualizaciones del servidor de configuración y RX no se pueden revertir una vez aplicadas.


### <a name="azure-site-recovery-scout-801-update-3"></a>Azure Site Recovery Scout 8.0.1 actualización 3

Todas las actualizaciones de Site Recovery son acumulativas. Update 3 contiene todas las correcciones de Update 1 y Update 2. La actualización 3 se puede aplicar directamente en 8.0.1 GA. Las actualizaciones del servidor de configuración y RX no se pueden revertir una vez aplicadas.

#### <a name="bug-fixes-and-enhancements"></a>Mejoras y correcciones de errores
Update 3 corrige los problemas siguientes:

* El servidor de configuración y RX no se registran en el almacén cuando están detrás del proxy.
* El número de horas en que no se alcanzaba el objetivo de punto de recuperación (RPO) no se actualiza en el informe de mantenimiento.
* El servidor de configuración no se sincroniza con RX cuando los detalles del hardware ESX o de la red contienen cualquier carácter UTF-8.
* Los controladores de dominio de Windows Server 2008 R2 no se inician después de la recuperación.
* La sincronización sin conexión no funciona según lo previsto.
* Después de la conmutación por error de la máquina virtual, la eliminación depares de replicación no avanza durante largo tiempo en la consola del servidor de configuración. Los usuarios no pueden completar las operaciones de conmutación por recuperación o reanudación.
* Las operaciones de instantáneas globales que realiza el trabajo de coherencia se han optimizado para ayudar a reducir las desconexiones de la aplicación como clientes de SQL Server.
* Se mejoró el rendimiento de la herramienta de coherencia (VACP.exe). Se redujo el uso de memoria requerido para crear instantáneas en Windows.
* El servicio de instalación de inserción se bloquea cuando la contraseña tiene más de 16 caracteres.
* vContinuum no comprueba las credenciales de vCenter ni solicita nuevas cuando estas se modifican.
* En Linux, el administrador de caché (cachemgr) de destino maestro no descarga los archivos del servidor de proceso. Esto genera una limitación de pares de replicación.
* Cuando el orden del disco físico del clúster de conmutación por error (MSCS) no es el mismo en todos los nodos, la replicación no se establece para algunos de los volúmenes del clúster. Tenga en cuenta que el clúster debe protegerse de nuevo para aprovechar esta corrección.  
* La funcionalidad de SMTP no funciona como se esperaba después de actualizar RX de Scout 7.1 a Scout 8.0.1.
* Se han agregado más estadísticas en el registro de la operación de reversión para medir el tiempo que ha tardado en completarse.
* Se ha agregado compatibilidad para los sistemas operativos Linux en el servidor de origen:
  * Red Hat Enterprise Linux (RHEL) 6 actualización 7
  * CentOS 6 actualización 7
* Las consolas RX y del servidor de configuración ahora muestran notificaciones para el par, el que va en modo de mapa de bits.
* Las siguientes revisiones de seguridad se han agregado a RX:
    * Omisión de la autorización mediante la alteración de parámetros: Se ha restringido el acceso a usuarios no aplicables.
    * Falsificación de solicitud entre sitios: se ha implementado el concepto de token de página que se genera aleatoriamente para cada página. Esto significa que solo hay una instancia de inicio de sesión único para el mismo usuario y que la actualización de la página no funciona. En lugar de eso, redirige al panel.
    * Carga de archivos malintencionados: los archivos están restringidos a extensiones específicas: z, aiff, asf, avi, bmp, csv, doc, docx, fla, flv, gif, gz, gzip, jpeg, jpg, log, mid, mov, mp3, mp4, mpc, mpeg, mpg, ods, odt, pdf, png, ppt, pptx, pxd, qt, ram, rar, rm, rmi, rmvb, rtf, sdc, sitd, swf, sxc, sxw, tar, tgz, tif, tiff, txt, vsd, wav, wma, wmv, xls, xlsx, xml y zip.
    * Scripting entre sitios persistente: se han agregado las validaciones de entradas.

### <a name="azure-site-recovery-scout-801-update-2-update-03dec15"></a>Azure Site Recovery Scout 8.0.1 actualización 2 (actualización del 3 de diciembre de 2015)

Las revisiones de la actualización 2 incluyen:

* **Servidor de configuración**: problemas que impedían que la característica de disponibilidad gratuita durante 31 días funcionase según lo esperado al registrar el servidor de configuración en el almacén de Azure Site Recovery.
* **Agente unificado**: corrige un problema en la actualización 1, cuyo resultado era que la actualización no se instalaba en el servidor de destino maestro durante la actualización desde la versión 8.0 a 8.0.1.

### <a name="azure-site-recovery-scout-801-update-1"></a>Azure Site Recovery Scout 8.0.1 actualización 1
La actualización 1 incluye las siguientes revisiones de errores y características nuevas:

* 31 días de protección gratuita por instancia de servidor. Esto le permite probar la funcionalidad o configurar una prueba de concepto.
* Todas las operaciones en el servidor, incluida la conmutación por error y la conmutación por recuperación, son gratis los primeros 31 días. El tiempo se inicia cuando se protege por primera vez un servidor con Site Recovery Scout. Desde el 32º día, a cada servidor protegido se le cobra la tarifa de instancia estándar de la protección de Site Recovery en un sitio propiedad de un cliente.
* En cualquier momento, el número de servidores protegidos que se están cargando actualmente está disponible en el **Panel** del almacén.
* Se agregó compatibilidad con la interfaz de la línea de comandos (vCLI) de vSphere 5.5 Update 2.
* Se agregó compatibilidad con los sistemas operativos Linux en el servidor de origen:
    * RHEL 6 Update 6
    * RHEL 5 Update 11
    * CentOS 6 Update 6
    * CentOS 5 Update 11
* Correcciones de errores para resolver los problemas siguientes:
  * Error en el registro del almacén para el servidor de configuración o el servidor RX.
  * Los volúmenes de clúster no aparecen del modo esperado cuando las máquinas virtuales agrupadas en clústeres se vuelven a proteger al reanudarse.
  * La conmutación por recuperación falla cuando el servidor de destino maestro se hospeda en un servidor ESXi diferente de las máquinas virtuales de producción locales.
  * Los permisos de archivo de configuración se cambian al actualizar a la versión 8.0.1. Este cambio afecta la protección y las operaciones.
  * El umbral de resincronización no se aplica del modo esperado, lo que genera un comportamiento de replicación incoherente.
  * La configuración de RPO no aparece correctamente en la consola del servidor de configuración. El valor de datos sin comprimir muestra incorrectamente el valor comprimido.
  * La operación de eliminación no efectúa la eliminación como se esperaba en el asistente de vContinuum y la replicación no se elimina de la consola del servidor de configuración.
  * En el asistente de vContinuum, el disco deja automáticamente de estar seleccionado al hacer clic en **Detalles** en la vista de disco durante la protección de las máquinas virtuales MSCS.
  * En el escenario físico a virtual (P2V), algunos servicios HP requeridos (como CIMnotify y CqMgHost) no se mueven a manual en la recuperación de la máquina virtual. Este problema genera que se produzca un tiempo de arranque adicional.
  * Se produce un error en la protección de la máquina virtual Linux cuando hay más de 26 discos en el servidor de destino maestro.

