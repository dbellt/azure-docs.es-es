---
title: Arquitecturas de las soluciones con Azure NetApp Files | Microsoft Docs
description: Proporciona referencias a los procedimientos recomendados de las arquitecturas de soluciones que usa Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: b-juche
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/03/2021
ms.author: b-juche
ms.openlocfilehash: 358bddaffc90467d6ff636b0e9c88eeac04a2fed
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108742506"
---
# <a name="solution-architectures-using-azure-netapp-files"></a>Arquitecturas de las soluciones con Azure NetApp Files
En este artículo se proporcionan referencias a los procedimientos recomendados que pueden ayudarle a comprender las arquitecturas de soluciones para usar Azure NetApp Files.  

El diagrama siguiente resume las categorías de arquitecturas de soluciones que Azure NetApp Files ofrece:

![Categorías de la arquitectura de la solución](../media/azure-netapp-files/solution-architecture-categories.png)

## <a name="linux-oss-apps-and-database-solutions"></a>Aplicaciones OSS de Linux y soluciones de base de datos

En esta sección se proporcionan referencias para las soluciones de las bases de datos y las aplicaciones OSS de Linux. 

### <a name="oracle"></a>Oracle

* [Guía de procedimientos recomendados para la implementación de Oracle en Azure con Azure NetApp Files](https://www.netapp.com/us/media/tr-4780.pdf)
* [Imágenes de VM de Oracle y su implementación en Microsoft Azure: Opciones de configuración de almacenamiento compartido](../virtual-machines/workloads/oracle/oracle-vm-solutions.md#shared-storage-configuration-options)
* [Rendimiento de Oracle Database en volúmenes individuales de Azure NetApp Files](performance-oracle-single-volumes.md)
* [Ventajas del uso de Azure NetApp Files con Oracle Database](solutions-benefits-azure-netapp-files-oracle-database.md)

### <a name="machine-learning"></a>Machine Learning
*   [Aprendizaje automático de Cloudera](https://docs.cloudera.com/machine-learning/cloud/requirements-azure/topics/ml-requirements-azure.html)

### <a name="education"></a>Education
* [Moodle sobre el almacenamiento de NFS con Azure NetApp Files](https://techcommunity.microsoft.com/t5/azure-architecture-blog/azure-netapp-files-for-nfs-storage-with-moodle/ba-p/2300630)

## <a name="windows-apps-and-sql-server-solutions"></a>Aplicaciones de Windows y soluciones de SQL Server

En esta sección se proporcionan referencias para las aplicaciones de Windows y soluciones de SQL Server.

### <a name="file-sharing-and-global-file-caching"></a>Uso compartido de archivos y almacenamiento en caché global de archivos

* [¿Quiere crear su propia instancia de Azure NFS? Cómo lidiar con los recursos compartidos de archivos Linux en la nube](https://cloud.netapp.com/blog/ma-anf-blg-build-your-own-linux-nfs-file-shares)
* [Caché de archivos globales/implementación de Azure NetApp Files](https://youtu.be/91LKb1qsLIM)
* [Cumplimiento de la nube para Azure NetApp Files](https://cloud.netapp.com/hubfs/Cloud%20Compliance%20for%20Azure%20NetApp%20Files%20-%20November%202020.pdf)

### <a name="sql-server"></a>SQL Server

* [SQL Server en la guía de implementación de Azure con Azure NetApp Files](https://www.netapp.com/pdf.html?item=/media/27154-tr-4888.pdf)
* [Ventajas del uso de Azure NetApp Files para la implementación de SQL Server](solutions-benefits-azure-netapp-files-sql-server.md)
* [Implementar SQL Server a través de SMB con Azure NetApp Files](https://www.youtube.com/watch?v=x7udfcYbibs)
* [Implementar el clúster de conmutación por error AlwaysOn de SQL Server en SMB con Azure NetApp Files](https://www.youtube.com/watch?v=zuNJ5E07e8Q) 
* [Implementar grupos de disponibilidad AlwaysOn con Azure NetApp Files](https://www.youtube.com/watch?v=y3VQmzzeyvc) 

## <a name="sap-on-azure-solutions"></a>Soluciones de SAP en Azure

En esta sección se proporcionan referencias de soluciones de SAP en Azure. 

### <a name="generic-sap-and-sap-netweaver"></a>SAP genérico y SAP NetWeaver 

* [Aplicaciones SAP en Microsoft Azure usando Azure NetApp Files](https://www.netapp.com/us/media/tr-4746.pdf)
* [Alta disponibilidad de SAP NetWeaver en VM de Azure en SUSE Linux Enterprise Server con Azure NetApp Files para las aplicaciones de SAP](../virtual-machines/workloads/sap/high-availability-guide-suse-netapp-files.md)
* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Red Hat Enterprise Linux con Azure NetApp Files para aplicaciones SAP](../virtual-machines/workloads/sap/high-availability-guide-rhel-netapp-files.md)
* [Alta disponibilidad para SAP NetWeaver en máquinas virtuales de Azure en Windows con Azure NetApp Files (SMB) para aplicaciones SAP](../virtual-machines/workloads/sap/high-availability-guide-windows-netapp-files-smb.md)
* [Alta disponibilidad para SAP NetWeaver en VM de Azure en Red Hat Enterprise Linux para SAP Applications: guía de varios SID](../virtual-machines/workloads/sap/high-availability-guide-rhel-multi-sid.md)

### <a name="sap-hana"></a>SAP HANA 

* [Configuraciones de almacenamiento de máquinas virtuales de Azure en SAP HANA](../virtual-machines/workloads/sap/hana-vm-operations-storage.md)
* [Volúmenes NFS v4.1 en Azure NetApp Files para SAP HANA](../virtual-machines/workloads/sap/hana-vm-operations-netapp.md)
* [Alta disponibilidad del escalado vertical de SAP HANA con Azure NetApp Files en Red Hat Enterprise Linux](../virtual-machines/workloads/sap/sap-hana-high-availability-netapp-files-red-hat.md)
* [Escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en SUSE Linux Enterprise Server](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-suse.md)
* [Escalabilidad horizontal de SAP HANA con nodo en espera en máquinas virtuales de Azure con Azure NetApp Files en Red Hat Enterprise Linux](../virtual-machines/workloads/sap/sap-hana-scale-out-standby-netapp-files-rhel.md)
* [Escalabilidad horizontal de SAP HANA con HSR y Pacemaker en RHEL: Azure Virtual Machines](../virtual-machines/workloads/sap/sap-hana-high-availability-scale-out-hsr-rhel.md)
* [Herramienta Azure Application Consistent Snapshot (AzAcSnap)](azacsnap-introduction.md)

### <a name="sap-anydb"></a>SAP AnyDB

* [Implementación de DBMS de Azure Virtual Machines de Oracle para la carga de trabajo de SAP: Azure Virtual Machines](../virtual-machines/workloads/sap/dbms_guide_oracle.md#oracle-configuration-guidelines-for-sap-installations-in-azure-vms-on-linux)
* [Implementación de SAP AnyDB (Oracle 19c) con Azure NetApp Files](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/deploy-sap-anydb-oracle-19c-with-azure-netapp-files/ba-p/2064043)

### <a name="sap-iq-nls"></a>SAP IQ-NLS

*   [Implementación de la solución de alta disponibilidad SAP IQ-NLS con Azure NetApp Files en SUSE Linux Enterprise Server](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/deploy-sap-iq-nls-ha-solution-using-azure-netapp-files-on-suse/ba-p/1651172#.X2tDfpNzBh4.linkedin)
* [Administración de una licencia de SAP IQ en un escenario de alta disponibilidad](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/how-to-manage-sap-iq-license-in-ha-scenario/ba-p/2052583)

### <a name="sap-tech-community-and-blog-posts"></a>Publicaciones de blog y comunidad tecnológica de SAP 

* [Azure NetApp Files: copia de seguridad SAP HANA en segundos](https://blog.netapp.com/azure-netapp-files-sap-hana-backup-in-seconds/)
* [Azure NetApp Files: restaure la base de datos HANA a partir de una copia de seguridad de instantánea](https://blog.netapp.com/azure-netapp-files-backup-sap-hana)
* [Azure NetApp Files: Descarga de copias de seguridad de SAP HANA con sincronización en la nube](https://blog.netapp.com/azure-netapp-files-sap-hana)
* [Acelere las copias del sistema SAP HANA con Azure NetApp Files](https://blog.netapp.com/sap-hana-faster-using-azure-netapp-files/)
* [Volúmenes en la nube ONTAP y Azure NetApp Files: simplificación de la migración del sistema SAP HANA](https://blog.netapp.com/cloud-volumes-ontap-and-azure-netapp-files-sap-hana-system-migration-made-easy/)
* [Decisiones arquitectónicas para maximizar la inversión de ANF en la arquitectura con escalabilidad horizontal de HANA N + M, parte 1](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/architectural-decisions-to-maximize-anf-investment-in-hana-n-m/ba-p/2078737)
* [Decisiones arquitectónicas para maximizar la inversión de ANF en la arquitectura con escalabilidad horizontal de HANA N + M, parte 2](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/architectural-decisions-to-maximize-anf-investment-in-hana-n-m/ba-p/2117130)
* [Decisiones arquitectónicas para maximizar la inversión de ANF en la arquitectura con escalabilidad horizontal de HANA N + M, parte 3](https://techcommunity.microsoft.com/t5/running-sap-applications-on-the/architectural-decisions-to-maximize-anf-investment-in-hana-n-m/ba-p/2215948)
* [Cambio de tamaño y consolidación de volúmenes de SAP con Azure NetApp Files](https://techcommunity.microsoft.com/t5/sap-on-microsoft/sap-landscape-sizing-and-volume-consolidation-with-anf/m-p/2145572/highlight/true#M14)

## <a name="azure-vmware-solutions"></a>Soluciones de VMware en Azure

* [Azure NetApp Files con Azure VMware Solution: montajes de SO invitado](../azure-vmware/netapp-files-with-azure-vmware-solution.md)

## <a name="virtual-desktop-infrastructure-solutions"></a>Soluciones de infraestructura de escritorio virtual

En esta sección se proporcionan referencias para las soluciones de infraestructura de escritorio virtual.

### <a name="windows-virtual-desktop"></a>Windows Virtual Desktop

* [Ventajas de usar Azure NetApp Files con Windows Virtual Desktop](solutions-windows-virtual-desktop.md)
* [Opciones de almacenamiento para los contenedores de perfiles de FSLogix de Windows Virtual Desktop](../virtual-desktop/store-fslogix-profile.md#azure-platform-details)
* [Creación de un contenedor de perfiles de FSLogix para un grupo host mediante Azure NetApp Files](../virtual-desktop/create-fslogix-profile-container.md)
* [Windows Virtual Desktop a escala empresarial](/azure/architecture/example-scenario/wvd/windows-virtual-desktop)
* [Microsoft FSLogix para la empresa: procedimientos recomendados de Azure NetApp Files](/azure/architecture/example-scenario/wvd/windows-virtual-desktop-fslogix#azure-netapp-files-best-practices)
* [Configuración de Azure NetApp Files para la asociación de aplicaciones MSIX](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/setting-up-azure-netapp-files-for-msix-app-attach-step-by-step/m-p/1990021)

## <a name="hpc-solutions"></a>Soluciones HPC

En esta sección se proporcionan referencias para las soluciones de informática de alto rendimiento (HPC). 

### <a name="generic-hpc"></a>HPC genérico

* [Azure NetApp Files: Cómo sacar el máximo partido del almacenamiento en la nube](https://cloud.netapp.com/hubfs/Resources/ANF%20PERFORMANCE%20TESTING%20IN%20TEMPLATE.pdf)
* [Ejecución de cargas de trabajo de MPI con Azure Batch y Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
* [Azure Cycle Cloud: Entornos de HPC de CycleCloud en Azure NetApp Files](/azure/cyclecloud/overview)

### <a name="oil-and-gas"></a>Petróleo y gas

* [Informática de alto rendimiento (HPC): Petróleo y gas en Azure](https://techcommunity.microsoft.com/t5/azure-global/high-performance-computing-hpc-oil-and-gas-in-azure/ba-p/824926)
* [Ejecución de software de simulación de depósitos en Azure](/azure/architecture/example-scenario/infrastructure/reservoir-simulation)

### <a name="electronic-design-automation-eda"></a>Automatización de diseño electrónico (EDA)

* [Ventajas de usar Azure NetApp Files para la automatización de diseños electrónicos](solutions-benefits-azure-netapp-files-electronic-design-automation.md)
* [Azure CycleCloud: Laboratorio de HPC de EDA con Azure NetApp Files](https://github.com/Azure/cyclecloud-hands-on-labs/blob/master/EDA/README.md)
* [Azure para el sector de los semiconductores](https://azurecomcdn.azureedge.net/cvt-f40f39cd9de2d875ab0c198a8d7b186350cf0bca161e80d7896941389685d012/mediahandler/files/resourcefiles/azure-for-the-semiconductor-industry/Azure_for_the_Semiconductor_Industry.pdf)

### <a name="analytics"></a>Análisis

* [Azure NetApp Files: un sistema de archivos compartidos que se usará con la cuadrícula SAS en Microsoft Azure](https://communities.sas.com/t5/Administration-and-Deployment/Azure-NetApp-Files-A-shared-file-system-to-use-with-SAS-Grid-on/m-p/705192)
* [Azure NetApp Files: sistema de archivos compartidos para usar con SAS Grid en MS Azure: actualización de RHEL8.3/nconnect](https://communities.sas.com/t5/Administration-and-Deployment/Azure-NetApp-Files-A-shared-file-system-to-use-with-SAS-Grid-on/m-p/722261#M21648)
* [Prácticas recomendadas para usar Microsoft Azure con SAS®](https://communities.sas.com/t5/Administration-and-Deployment/Best-Practices-for-Using-Microsoft-Azure-with-SAS/m-p/676833#M19680)

## <a name="azure-platform-services-solutions"></a>Soluciones de servicios de la plataforma Azure

En esta sección se proporcionan soluciones para los servicios de la plataforma Azure. 

### <a name="azure-kubernetes-services-and-kubernetes"></a>Azure Kubernetes Service y Kubernetes

* [Integración de Azure NetApp Files con Azure Kubernetes Service](../aks/azure-netapp-files.md)
* [Rendimiento Kubernetes fuera de este mundo en Azure con Azure NetApp Files](https://cloud.netapp.com/blog/ma-anf-blg-configure-kubernetes-openshift)
* [Azure NetApp Files + Trident = almacenamiento dinámico y persistente para Kubernetes](https://anfcommunity.com/2021/02/16/azure-netapp-files-trident-dynamic-and-persistent-storage-for-kubernetes/)
* [Trident: orquestador de almacenamiento para contenedores](https://netapp-trident.readthedocs.io/en/stable-v20.04/kubernetes/operations/tasks/backends/anf.html)
* [Plataforma de comercio electrónico Magento en Azure Kubernetes Service (AKS)](/azure/architecture/example-scenario/magento/magento-azure)

### <a name="azure-batch"></a>Azure Batch

* [Ejecución de cargas de trabajo de MPI con Azure Batch y Azure NetApp Files](https://azure.microsoft.com/resources/run-mpi-workloads-with-azure-batch-and-azure-netapp-files/)
