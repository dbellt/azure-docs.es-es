---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: f7af6962c3343cd9d3e35e96d88650aa6a42c6b3
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95555968"
---
1. Mientras está conectado a la máquina virtual con Escritorio remoto, busque **Administrador de configuración**:

    ![Abrir SSCM](./media/virtual-machines-sql-server-connection-tcp-protocol/sql-server-configuration-manager.png)

1. En el panel de la consola de Administrador de configuración de SQL Server, expanda **Configuración de red de SQL Server**.

1. En el panel de la consola, haga clic en **Protocols for MSSQLSERVER** (Protocolos de MSSQLSERVER) (el nombre de la instancia predeterminada). En el panel de detalles, haga clic con el botón derecho en **TCP** y haga clic en **Habilitar**, en caso de que no esté habilitado.

    ![Habilitar TCP](./media/virtual-machines-sql-server-connection-tcp-protocol/enable-tcp.png)

1. En el panel de la consola, haga clic en **Servicios de SQL Server**. En el panel de detalles, haga clic con el botón derecho en **SQL Server (*nombre de la instancia*)** (la instancia predeterminada es **SQL Server (MSSQLSERVER)**) y, después, haga clic en **Reiniciar** para detener y reiniciar la instancia de SQL Server.

    ![Reiniciar el motor de base de datos](./media/virtual-machines-sql-server-connection-tcp-protocol/restart-sql-server.png)

1. Cierre el Administrador de configuración de SQL Server.

Para obtener más información acerca de la habilitación de protocolos para el motor de base de datos de SQL Server, consulte [Habilitar o deshabilitar un protocolo de red de servidor](/sql/database-engine/configure-windows/enable-or-disable-a-server-network-protocol).