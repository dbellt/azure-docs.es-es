---
author: jovanpop-msft
ms.service: sql-database
ms.subservice: deployment-configuration
ms.topic: include
ms.date: 12/17/2020
ms.author: jovanpop
ms.openlocfilehash: a253cae49bcb8fff1b14736302810bffc9d1ecb0
ms.sourcegitcommit: b11257b15f7f16ed01b9a78c471debb81c30f20c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2021
ms.locfileid: "111593986"
---
  Puede utilizar uno de estos inicios rápidos para crear y configurar una base de datos:

  | Acción | SQL Database | Instancia administrada de SQL | SQL Server en máquina virtual de Azure | Azure Synapse Analytics |
  |:--- |:--- |:---|:---|:---|
  | Crear| [Portal](../database/single-database-create-quickstart.md) | [Portal](../managed-instance/instance-create-quickstart.md) | [Portal](../virtual-machines/windows/sql-vm-create-portal-quickstart.md) | [Portal](../../synapse-analytics/quickstart-create-workspace.md) |
  || [CLI](../database/scripts/create-and-configure-database-cli.md) | | | [CLI](../../synapse-analytics/quickstart-create-workspace-cli.md) |
  || [PowerShell](../database/scripts/create-and-configure-database-powershell.md) | [PowerShell](../managed-instance/scripts/create-configure-managed-instance-powershell.md) | [PowerShell](../virtual-machines/windows/sql-vm-create-powershell-quickstart.md) | [PowerShell](../../synapse-analytics/quickstart-create-workspace-powershell.md) |
  || | | [Plantilla de implementación](../virtual-machines/windows/create-sql-vm-resource-manager-template.md) | [Plantilla de implementación](../../synapse-analytics/quickstart-deployment-template-workspaces.md) | 
  | Configuración | [Regla de firewall de IP en el nivel de servidor](../database/firewall-create-server-level-portal-quickstart.md)| [Conectividad desde una máquina virtual](../managed-instance/connect-vm-instance-configure.md)| |
  |||[Conectividad desde un entorno local](../managed-instance/point-to-site-p2s-configure.md) | [Conexión a una instancia de SQL Server](../virtual-machines/windows/sql-vm-create-portal-quickstart.md) |
  | Obtención de información sobre la conexión | [SQL de Azure](../database/connect-query-content-reference-guide.md#get-server-connection-information)|[SQL de Azure](../database/connect-query-content-reference-guide.md#get-server-connection-information)| [VM con SQL](../virtual-machines/windows/sql-vm-create-portal-quickstart.md?#connect-to-sql-server)| [Synapse SQL](../../synapse-analytics/sql/connect-overview.md#find-your-server-name)|
  |||||
