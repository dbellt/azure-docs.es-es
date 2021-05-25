---
title: Azure Automanage para Windows Server
description: Obtenga información sobre los procedimientos recomendados de Azure Automanage para máquinas virtuales para los servicios que se incorporan y configuran automáticamente para las máquinas Windows Server.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 02/22/2021
ms.author: deanwe
ms.openlocfilehash: 3af0c8c99607675de3166c8747755791dbba5820
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109483864"
---
# <a name="azure-automanage-for-virtual-machines-best-practices---windows-server"></a>Procedimientos recomendados de Azure Automanage para máquinas virtuales: Windows Server

Estos servicios de Azure se incorporan automáticamente cuando usa Automanage para máquinas virtuales (VM) en una VM Windows Server. Estos servicios son fundamentales para nuestra nota del producto de procedimientos recomendados, que puede encontrar en nuestro marco [Cloud Adoption Framework](/azure/cloud-adoption-framework/manage/azure-server-management).

Todos estos servicios se incorporarán y configurarán automáticamente, se supervisarán en busca de desfases y, en caso de que se detecten, se corregirán. Para obtener más información sobre este proceso, consulte [Azure Automanage para máquinas virtuales](automanage-virtual-machines.md).

## <a name="supported-windows-server-versions"></a>Versiones de Windows Server admitidas

Automanage es compatible con las siguientes versiones de Windows Server:

- Windows Server 2012/R2
- Windows Server 2016
- Windows Server 2019
- Windows Server 2019 Azure Edition

## <a name="participating-services"></a>Servicios participantes

|Servicio    |Descripción    |Entornos admitidos<sup>1</sup>    |Preferencias admitidas<sup>1</sup>    |
|-----------|---------------|----------------------|-------------------------|
|[Supervisión de VM Insights](../azure-monitor/vm/vminsights-overview.md)    |Azure Monitor para VM supervisa el rendimiento y el mantenimiento de las máquinas virtuales, incluidos los procesos en ejecución y las dependencias de otros recursos. [Más información](../azure-monitor/vm/vminsights-overview.md).    |Producción    |No    |
|[Backup](../backup/backup-overview.md)    |Azure Backup proporciona copias de seguridad independientes y aisladas para impedir la destrucción accidental de los datos en las máquinas virtuales. [Más información](../backup/backup-azure-vms-introduction.md). Los cargos se basan en el número y el tamaño de las VM que se están protegiendo. [Más información](https://azure.microsoft.com/pricing/details/backup/).    |Producción    |Sí    |
|[Azure Security Center](../security-center/security-center-introduction.md)    |Azure Security Center es un sistema unificado de administración de la seguridad de infraestructura que fortalece el nivel de seguridad de sus centros de datos, y proporciona protección contra amenazas avanzada en las cargas de trabajo híbridas de la nube. [Más información](../security-center/security-center-introduction.md).  Automanage configurará la suscripción en la que reside la máquina virtual a la oferta de nivel Gratis de Azure Security Center. Si la suscripción ya se ha incorporado a Azure Security Center, Automanage no la volverá a configurar.    |Producción, desarrollo/pruebas    |No    |
|[Microsoft Antimalware](../security/fundamentals/antimalware.md)    |Microsoft Antimalware para Azure es una protección gratuita en tiempo real que ayuda a identificar y eliminar virus, spyware y otro software malintencionado. Genera alertas cuando software no deseado o malintencionado intenta instalarse o ejecutarse en los sistemas de Azure. **Nota:** Microsoft Antimalware requiere que no haya ningún otro software antimalware instalado o puede que no funcione.  [Más información](../security/fundamentals/antimalware.md). |Producción, desarrollo/pruebas    |Sí    |
|[Administración de actualizaciones](../automation/update-management/overview.md)    |Use Update Management en Azure Automation para administrar las actualizaciones del sistema operativo de las máquinas virtuales. Puede evaluar rápidamente el estado de las actualizaciones disponibles en todas las máquinas agente y administrar el proceso de instalación de las actualizaciones necesarias para los servidores. [Más información](../automation/update-management/overview.md).    |Producción, desarrollo/pruebas    |No    |
|[Change Tracking e inventario](../automation/change-tracking/overview.md) |Change Tracking e Inventario combina funciones de inventario y seguimiento de cambios que le permiten realizar un seguimiento de los cambios en la infraestructura de servidores y máquinas virtuales. El servicio admite el seguimiento de cambios en el registro, servicios, demonios, software y archivos del entorno para ayudarle a diagnosticar cambios no deseados y generar alertas. La compatibilidad con inventario le permite consultar recursos de los invitados para obtener una visualización de las aplicaciones instaladas y otros elementos de configuración.  [Más información](../automation/change-tracking/overview.md).    |Producción, desarrollo/pruebas    |No    |
|[Configuración de invitado de Azure](../governance/policy/concepts/guest-configuration.md) | La directiva de configuración de invitado se usa para supervisar la configuración y notificar sobre el cumplimiento de la máquina. El servicio de Automanage instalará las [líneas base de seguridad de Windows](/windows/security/threat-protection/windows-security-baselines) con la extensión de configuración de invitado. En el caso de las máquinas Windows, el servicio de configuración de invitado volverá a aplicar automáticamente la configuración de la base de referencia si no son compatibles. [Más información](../governance/policy/concepts/guest-configuration.md).    |Producción, desarrollo/pruebas    |No    |
|[Diagnósticos de arranque](../virtual-machines/boot-diagnostics.md)    | Diagnósticos de arranque es una característica de depuración para máquinas virtuales (VM) de Azure que permite el diagnóstico de errores de arranque de la máquina virtual. Los diagnósticos de arranque permiten a un usuario observar el estado de la máquina virtual cuando está arrancando mediante la recopilación de información de registro serie y capturas de pantallas. Esta característica solo se habilitará en las máquinas que usan discos administrados. |Producción, desarrollo/pruebas    |No    |
|[Cuenta de Azure Automation](../automation/automation-create-standalone-account.md)    |Azure Automation permite la administración de la infraestructura y las aplicaciones a lo largo de su ciclo de vida. [Más información](../automation/automation-intro.md).    |Producción, desarrollo/pruebas    |No    |
|[Área de trabajo de Log Analytics](../azure-monitor/logs/log-analytics-overview.md) |Azure Monitor almacena los datos de registro en un área de trabajo de Log Analytics, que es un recurso de Azure y un contenedor en el que los datos se recopilan y se agregan, y que sirve como límite administrativo. [Más información](../azure-monitor/logs/design-logs-deployment.md).    |Producción, desarrollo/pruebas    |No    |


<sup>1</sup> La selección de entorno está disponible cuando se habilita Automanage. [Más información](automanage-virtual-machines.md#environment-configuration). También puede ajustar la configuración predeterminada del entorno y establecer sus propias preferencias dentro de las restricciones de los procedimientos recomendados.


## <a name="next-steps"></a>Pasos siguientes

Intente habilitar Automanage para máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)