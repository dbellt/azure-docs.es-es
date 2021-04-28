---
title: Acerca de la recuperación ante desastres para aplicaciones locales con Azure Site Recovery
description: Describe las cargas de trabajo que pueden protegerse mediante la recuperación ante desastres con el servicio Azure Site Recovery.
ms.topic: conceptual
ms.date: 03/18/2020
ms.openlocfilehash: 1a5d20e6feacfe72052142c07dc45753b9bc3138
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107599118"
---
# <a name="about-disaster-recovery-for-on-premises-apps"></a>Acerca de la recuperación ante desastres en aplicaciones locales

En este artículo se describen las cargas de trabajo y las aplicaciones locales que se pueden proteger para la recuperación ante desastres con el servicio [Azure Site Recovery](site-recovery-overview.md).

## <a name="overview"></a>Información general

Las organizaciones necesitan una estrategia de recuperación ante desastres y continuidad empresarial (BCDR) para mantener las cargas de trabajo y los datos seguros y disponibles durante los tiempos de inactividad planeados y no planeados. Y, también, para recuperar las condiciones de funcionamiento normales.

Site Recovery es un servicio de Azure que contribuye a su estrategia de recuperación ante desastres y continuidad del negocio. Mediante Site Recovery, puede implementar la replicación con reconocimiento de aplicaciones en la nube o en un sitio secundario. Site Recovery se puede usar para administrar la replicación, realizar pruebas de recuperación ante desastres y ejecutar conmutaciones por error y conmutación por recuperación. Las aplicaciones pueden ejecutarse en equipos con Windows o Linux, servidores físicos, VMware o Hyper-V.

Site Recovery se integra con aplicaciones de Microsoft, como SharePoint, Exchange, Dynamics, SQL Server y Active Directory. Microsoft trabaja en estrecha colaboración con destacados proveedores como Oracle, SAP y Red Hat. Puede personalizar las soluciones de replicación en función de cada aplicación.

## <a name="why-use-site-recovery-for-application-replication"></a>¿Por qué es aconsejable usar Site Recovery para la replicación de aplicaciones?

Site Recovery contribuye a la recuperación y protección de nivel de aplicación como se indica a continuación:

- Es independiente de las aplicaciones y proporciona replicación de las cargas de trabajo que se ejecutan en una máquina compatible.
- Replicación casi sincrónica con objetivos de punto de recuperación (RPO) de solo 30 segundos, con el fin de cubrir las necesidades de las aplicaciones empresariales más críticas.
- Instantáneas coherentes con las aplicaciones para aplicaciones de uno o varios niveles.
- Integración con SQL Server AlwaysOn y asociación con otras tecnologías de replicación de nivel de aplicación. Por ejemplo, la replicación de Active Directory, SQL AlwaysOn y Grupos de disponibilidad de base de datos de Exchange (DAG).
- Planes de recuperación flexibles que permiten recuperar toda una pila de aplicaciones con un solo clic e incluyen scripts externos y acciones manuales en el plan.
- Administración de red avanzada en Site Recovery y Azure para simplificar los requisitos de red de las aplicaciones. Administración de red como la capacidad para reservar direcciones IP, configurar el equilibrio de carga y la integración con Azure Traffic Manager, lo que reduce los cambios de red de los objetivos de tiempo de recuperación (RTO).
- Una biblioteca de automatización enriquecida que proporciona scripts específicos de la aplicación y preparados para la producción que pueden descargarse e integrarse con los planes de recuperación.

## <a name="workload-summary"></a>Resumen de cargas de trabajo

Site Recovery puede replicar cualquier aplicación que se ejecute en una máquina compatible. Nos hemos asociado con los equipos de producto para realizar pruebas adicionales para las aplicaciones especificadas en la siguiente tabla.

| **Carga de trabajo** |**Replicación de máquinas virtuales de Azure en Azure** |**Replicación de máquinas virtuales de Hyper-V a un sitio secundario** | **Replicación de máquinas virtuales de Hyper-V en Azure** | **Replicación de máquinas virtuales VMware en un sitio secundario** | **Replicación de máquinas virtuales VMware en Azure** |
| --- | --- | --- | --- | --- |---|
| Active Directory, DNS |Sí |Sí |Sí |Sí |Sí|
| Aplicaciones web (IIS, SQL) |Sí |Sí |Sí |Sí |Sí|
| System Center Operations Manager |Sí |Sí |Sí |Sí |Sí|
| SharePoint |Sí |Sí |Sí |Sí |Sí|
| SAP<br/><br/>Replicación de un sitio de SAP en Azure para no clúster |Sí (probado por Microsoft) |Sí (probado por Microsoft) |Sí (probado por Microsoft) |Sí (probado por Microsoft) |Sí (probado por Microsoft)|
| Exchange (no DAG) |Sí |Sí |Sí |Sí |Sí|
| Escritorio remoto/VDI |Sí |Sí |Sí |Sí |Sí|
| Linux (sistema operativo y aplicaciones) |Sí (probado por Microsoft) |Sí (probado por Microsoft) |Sí (probado por Microsoft) |Sí (probado por Microsoft) |Sí (probado por Microsoft)|
| Dynamics AX |Sí |Sí |Sí |Sí |Sí|
| Servidor de archivos de Windows |Sí |Sí |Sí |Sí |Sí|
| Citrix XenApp y XenDesktop |No|N/D |No |N/D |No |

## <a name="replicate-active-directory-and-dns"></a>Replicación de Active Directory y DNS

Las infraestructuras de DNS y Active Directory son esenciales para la mayoría de las aplicaciones empresariales. Durante la recuperación ante desastres será preciso que proteja y recupere estos componentes de la infraestructura antes de recuperar las cargas de trabajo y las aplicaciones.

Puede usar Site Recovery para crear un plan de recuperación ante desastres automatizada completa para Active Directory y DNS. Por ejemplo, para conmutar por error SharePoint y SAP desde un sitio principal a uno secundario, puede configurar un plan de recuperación que primero conmute por error Active Directory. Luego use un plan de recuperación adicional específico de la aplicación para conmutar por error las restantes aplicaciones que usen Active Directory.

[Más información](site-recovery-active-directory.md) sobre la recuperación ante desastres para Active Directory y DNS.

## <a name="protect-sql-server"></a>Protección de SQL Server

SQL Server proporciona una base de servicios de datos para muchas aplicaciones empresariales de un centro de datos local. Site Recovery se puede utilizar con las tecnologías de alta disponibilidad y recuperación ante desastres de SQL Server para proteger las aplicaciones empresariales de varios niveles que utilizan SQL Server.

Site Recovery proporciona:

- Una solución de recuperación ante desastres simple y rentable para SQL Server. Replique varias versiones y ediciones de servidores independientes y clústeres de SQL Server en Azure o en un sitio secundario.
- Integración con grupos de disponibilidad AlwaysOn de SQL para administrar la conmutación por error y la conmutación por recuperación con planes de recuperación de Azure Site Recovery.
- Planes de recuperación de un extremo a otro para todos los niveles de una aplicación, incluidas las bases de datos de SQL Server.
- Escalado de SQL Server para cargas máximas con Site Recovery _protegiéndolos_ en tamaños de máquina virtual de IaaS más grandes en Azure.
- Pruebas fáciles de recuperación ante desastres de SQL Server. Puede ejecutar conmutaciones por error de prueba para analizar datos y realizar comprobaciones de cumplimento sin que ello afecte al entorno de producción.

[Más información](site-recovery-sql.md) sobre la recuperación ante desastres para SQL Server.

## <a name="protect-sharepoint"></a>Protección de SharePoint

Azure Site Recovery ayuda a proteger las implementaciones de SharePoint como se indica a continuación:

- Elimina la necesidad y los costos de infraestructura asociados a la recuperación ante desastres en una granja en espera. Use Site Recovery para replicar toda una granja (niveles de web, aplicación y base de datos) en Azure o en un sitio secundario.
- Simplifica la administración e implementación de las aplicaciones. Las actualizaciones implementadas en el sitio principal se replican automáticamente. Las actualizaciones están disponibles después de la conmutación por error y la recuperación de una granja en un sitio secundario. Reduce la complejidad de la administración y los costos asociados con el hecho de mantener actualizada una granja en espera.
- Simplifica el desarrollo y las pruebas de aplicaciones de SharePoint mediante la creación de un entorno de réplica de copia de producción a petición para realizar pruebas y depuraciones.
- Simplifica la transición a la nube mediante el uso de Site Recovery para migrar las implementaciones de SharePoint a Azure.

[Más información](site-recovery-sharepoint.md) sobre la recuperación ante desastres para SharePoint.

## <a name="protect-dynamics-ax"></a>Protección de Dynamics AX

Azure Site Recovery le ayuda a proteger soluciones ERP de Dynamics AX mediante:

- La organización de la replicación de todo el entorno de Dynamics AX (niveles web y de AOS, niveles de base de datos y SharePoint) en Azure o en un sitio secundario.
- La simplificación de la migración de las implementaciones de Dynamics AX a la nube (Azure).
- La simplificación del desarrollo y de las pruebas de aplicaciones de Dynamics AX mediante la creación de una copia de producción a petición para realizar pruebas y depuraciones.

[Más información](site-recovery-dynamicsax.md) sobre la recuperación ante desastres en Dynamic AX.

## <a name="protect-remote-desktop-services"></a>Protección de Servicios de Escritorio remoto

Servicios de Escritorio remoto (RDS) habilita Infraestructura de escritorio virtual (VDI), escritorios basados en sesión y aplicaciones, lo que permite a los usuarios trabajar desde cualquier lugar.

Con Azure Site Recovery se pueden replicar los siguientes servicios:

- Replicar escritorios virtuales agrupados administrados o no administrados en un sitio secundario.
- Replicar aplicaciones remotas y sesiones en un sitio secundario o en Azure.

En la tabla siguiente se muestran las opciones de replicación:

| **RDS** |**Replicación de máquinas virtuales de Azure en Azure** | **Replicación de máquinas virtuales de Hyper-V a un sitio secundario** | **Replicación de máquinas virtuales de Hyper-V en Azure** | **Replicación de máquinas virtuales VMware en un sitio secundario** | **Replicación de máquinas virtuales VMware en Azure** | **Replicación de servidores físicos en un sitio secundario** | **Replicación de servidores físicos a Azure** |
|---| --- | --- | --- | --- | --- | --- | --- |
| **Escritorio virtual agrupado (no administrado)** |No|Sí |No |Sí |No |Sí |No |
| **Escritorio virtual agrupado (administrado y sin UPD)** |No|Sí |No |Sí |No |Sí |No |
| **Aplicaciones remotas y sesiones de escritorio (sin UPD)** |Sí|Sí |Sí |Sí |Sí |Sí |Sí |

[Más información](/windows-server/remote/remote-desktop-services/rds-disaster-recovery-with-azure) sobre la recuperación ante desastres para RDS.

## <a name="protect-exchange"></a>Protección de Exchange

Site Recovery ayuda a proteger Exchange como se indica a continuación:

- En implementaciones de Exchange pequeñas, como servidor individual o independiente, Site Recovery puede realizar la replicación y la conmutación por error en Azure o en un sitio secundario.
- En implementaciones mayores Site Recovery se integra con los DAG de Exchange.
- Los DAG de Exchange son la solución recomendada para la recuperación ante desastres de Exchange en una empresa.  Los planes de recuperación de Site Recovery pueden incluir varios DAG para organizar la conmutación por error de DAG entre sitios.

Para más información sobre la recuperación ante desastres para Exchange, consulte [DAG de Exchange](/Exchange/high-availability/database-availability-groups/database-availability-groups) y [Recuperación ante desastres de Exchange](/Exchange/high-availability/disaster-recovery/disaster-recovery).

## <a name="protect-sap"></a>Protección de SAP

Use Site Recovery para proteger una implementación de SAP como se indica a continuación:

- Habilitar la protección de aplicaciones de producción de SAP NetWeaver y no NetWeaver que se ejecutan de forma local, mediante la replicación de sus componentes en Azure.
- Habilitar la protección de aplicaciones de producción de SAP NetWeaver y no NetWeaver que se ejecutan en Azure, mediante la replicación de componentes en otro centro de datos de Azure.
- Simplifique la migración a la nube con Site Recovery para migrar la implementación de SAP en Azure.
- Simplificar las actualizaciones de los proyectos SAP, las pruebas y la creación de prototipos, mediante la creación de una clon en producción a petición para probar las aplicaciones SAP.

[Más información](site-recovery-sap.md) sobre la recuperación ante desastres para SAP.

## <a name="protect-internet-information-services"></a>Protección de Internet Information Services

Use Site Recovery para proteger la implementación de Internet Information Services como se indica a continuación:

Azure Site Recovery proporciona la función de recuperación ante desastres mediante la replicación de los componentes críticos de su entorno en un sitio remoto inactivo o en una nube pública, como Microsoft Azure. Dado que las máquinas virtuales con el servidor web y la base de datos se replican en el sitio de recuperación, no es preciso realizar una copia de seguridad independiente de los archivos de configuración o certificados. Los enlaces y las asignaciones de las aplicaciones que dependen de las variables de entorno que se cambian después de la conmutación por error se pueden actualizar a través de los scripts integrados en los planes de la recuperación ante desastres. Las máquinas virtuales solo se muestran en el sitio de recuperación durante una conmutación por error. Azure Site Recovery también le ayuda a orquestar la conmutación por error de un extremo a otro, ya que proporciona las siguientes funcionalidades:

- Secuenciación del apagado e inicio de las máquinas virtuales en los distintos niveles.
- Incorporación de scripts que permiten las actualizaciones de las dependencias y los enlaces de las aplicaciones en las máquinas virtuales después de que se hayan iniciado. Los scripts también se pueden utilizar para actualizar el servidor DNS para que apunte al sitio de recuperación.
- Asignación de direcciones IP a máquinas virtuales antes de la conmutación por error mediante la asignación de las redes principal y de recuperación y uso de scripts que no es preciso actualizar después de la conmutación por error.
- Capacidad para una conmutación por error con un solo clic para varias aplicaciones web que elimina el ámbito de confusión durante un desastre.
- Funcionalidad de probar los planes de recuperación en un entorno aislado en los simulacros de recuperación ante desastres.

[Más información](site-recovery-iis.md) sobre la recuperación ante desastres para IIS.

## <a name="protect-citrix-xenapp-and-xendesktop"></a>Protección de Citrix XenApp y XenDesktop

A partir de marzo de 2020, Citrix anunció el desuso y el fin del soporte técnico de las cargas de trabajo hospedadas en la nube pública. Por lo tanto, no se recomienda usar Site Recovery para proteger cargas de trabajo de Citrix.

## <a name="next-steps"></a>Pasos siguientes

[Más información](azure-to-azure-quickstart.md) sobre la recuperación ante desastres para una máquina virtual de Azure.
