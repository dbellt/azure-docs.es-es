---
title: Azure Automanage para Linux
description: Obtenga información sobre los procedimientos recomendados de Azure Automanage para máquinas virtuales para servicios que se incorporan y configuran automáticamente para máquinas Linux.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.collection: linux
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: deanwe
ms.openlocfilehash: 24b72afb7f685e50ac9452889b404fa8bda29644
ms.sourcegitcommit: 02bc06155692213ef031f049f5dcf4c418e9f509
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/03/2021
ms.locfileid: "106278327"
---
# <a name="azure-automanage-for-virtual-machines-best-practices---linux"></a>Procedimientos recomendados de Azure Automanage para máquinas virtuales - Linux

Estos servicios de Azure se incorporan automáticamente cuando usa Automanage para máquinas virtuales (VM) en una máquina virtual Linux. Estos servicios son fundamentales para nuestra nota del producto de procedimientos recomendados, que puede encontrar en nuestro marco [Cloud Adoption Framework](/azure/cloud-adoption-framework/manage/azure-server-management).

Todos estos servicios se incorporarán y configurarán automáticamente, se supervisarán en busca de desfases y, en caso de que se detecten, se corregirán. Para obtener más información sobre este proceso, consulte [Azure Automanage para máquinas virtuales](automanage-virtual-machines.md).

## <a name="supported-linux-distributions-and-versions"></a>Distribuciones y versiones de Linux compatibles

Automanage es compatible con las siguientes distribuciones y versiones de Linux:

- CentOS 7.3+
- RHEL 7.4 y versiones posteriores
- Ubuntu 16.04 y 18.04
- SLES 12 (solo SP3-SP5)

## <a name="participating-services"></a>Servicios participantes

>[!NOTE]
> Actualmente, Microsoft Antimalware no es compatible en máquinas virtuales Linux.

|Servicio    |Descripción    |Entornos admitidos<sup>1</sup>    |Preferencias admitidas<sup>1</sup>    |
|-----------|---------------|----------------------|-------------------------|
|[Supervisión de VM Insights](https://docs.microsoft.com/azure/azure-monitor/vm/vminsights-overview)    |Azure Monitor para VM supervisa el rendimiento y el mantenimiento de las máquinas virtuales, incluidos los procesos en ejecución y las dependencias de otros recursos. [Más información](../azure-monitor/vm/vminsights-overview.md).    |Producción    |No    |
|[Backup](https://docs.microsoft.com/azure/backup/backup-overview)   |Azure Backup proporciona copias de seguridad independientes y aisladas para impedir la destrucción accidental de los datos en las máquinas virtuales. [Más información](../backup/backup-azure-vms-introduction.md). Los cargos se basan en el número y el tamaño de las VM que se están protegiendo. [Más información](https://azure.microsoft.com/pricing/details/backup/).    |Producción    |Sí    |
|[Azure Security Center](https://docs.microsoft.com/azure/security-center/security-center-introduction)    |Azure Security Center es un sistema unificado de administración de la seguridad de infraestructura que fortalece el nivel de seguridad de sus centros de datos, y proporciona protección contra amenazas avanzada en las cargas de trabajo híbridas de la nube. [Más información](../security-center/security-center-introduction.md).  Automanage configurará la suscripción en la que reside la máquina virtual a la oferta de nivel Gratis de Azure Security Center. Si la suscripción ya se ha incorporado a Azure Security Center, Automanage no la volverá a configurar.    |Producción, desarrollo/pruebas    |No    |
|[Administración de actualizaciones](https://docs.microsoft.com/azure/automation/update-management/overview)    |Use Update Management en Azure Automation para administrar las actualizaciones del sistema operativo de las máquinas virtuales. Puede evaluar rápidamente el estado de las actualizaciones disponibles en todas las máquinas agente y administrar el proceso de instalación de las actualizaciones necesarias para los servidores. [Más información](../automation/update-management/overview.md).    |Producción, desarrollo/pruebas    |No    |
|[Change Tracking e inventario](https://docs.microsoft.com/azure/automation/change-tracking/overview) |Change Tracking e Inventario combina funciones de inventario y seguimiento de cambios que le permiten realizar un seguimiento de los cambios en la infraestructura de servidores y máquinas virtuales. El servicio admite el seguimiento de cambios en el registro, servicios, demonios, software y archivos del entorno para ayudarle a diagnosticar cambios no deseados y generar alertas. La compatibilidad con inventario le permite consultar recursos de los invitados para obtener una visualización de las aplicaciones instaladas y otros elementos de configuración.  [Más información](../automation/change-tracking/overview.md).    |Producción, desarrollo/pruebas    |No    |
|[Configuración de invitado de Azure](https://docs.microsoft.com/azure/governance/policy/concepts/guest-configuration)  | La directiva de configuración de invitado se usa para supervisar la configuración y notificar sobre el cumplimiento de la máquina. El servicio de Automanage instalará las bases de referencia Linux de Azure con la extensión de configuración de invitado. En el caso de máquinas Linux, el servicio de configuración de invitado instalará la base de referencia en modo de solo auditoría. Podrá ver dónde no cumple la máquina virtual con la base de referencia, pero el incumplimiento no se corregirá automáticamente. [Más información](../governance/policy/concepts/guest-configuration.md).    |Producción, desarrollo/pruebas    |No    |
|[Diagnósticos de arranque](https://docs.microsoft.com/azure/virtual-machines/boot-diagnostics)  | Diagnósticos de arranque es una característica de depuración para máquinas virtuales (VM) de Azure que permite el diagnóstico de errores de arranque de la máquina virtual. Los diagnósticos de arranque permiten a un usuario observar el estado de la máquina virtual cuando está arrancando mediante la recopilación de información de registro serie y capturas de pantallas. |Producción, desarrollo/pruebas    |No    |
|[Cuenta de Azure Automation](https://docs.microsoft.com/azure/automation/automation-create-standalone-account)    |Azure Automation permite la administración de la infraestructura y las aplicaciones a lo largo de su ciclo de vida. [Más información](../automation/automation-intro.md).    |Producción, desarrollo/pruebas    |No    |
|[Área de trabajo de Log Analytics](https://docs.microsoft.com/azure/azure-monitor/logs/log-analytics-overview) |Azure Monitor almacena los datos de registro en un área de trabajo de Log Analytics, que es un recurso de Azure y un contenedor en el que los datos se recopilan y se agregan, y que sirve como límite administrativo. [Más información](../azure-monitor/logs/design-logs-deployment.md).    |Producción, desarrollo/pruebas    |No    |


<sup>1</sup> La selección de entorno está disponible cuando se habilita Automanage. [Más información](automanage-virtual-machines.md#environment-configuration). También puede ajustar la configuración predeterminada del entorno y establecer sus propias preferencias dentro de las restricciones de los procedimientos recomendados.


## <a name="next-steps"></a>Pasos siguientes

Intente habilitar Automanage para máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)