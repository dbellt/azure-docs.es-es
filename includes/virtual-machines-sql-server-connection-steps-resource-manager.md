---
author: rothja
ms.service: virtual-machines-sql
ms.topic: include
ms.date: 10/26/2018
ms.author: jroth
ms.openlocfilehash: 51dc04fbef8d09878f33d7fda6f15039d3afba3e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96026628"
---
### <a name="configure-a-dns-label-for-the-public-ip-address"></a>Configuración de una etiqueta DNS para la dirección IP pública

Para conectarse al motor de base de datos de SQL Server desde Internet, considere configurar una etiqueta DNS para la dirección IP pública. Puede conectarse mediante una dirección IP, pero la etiqueta DNS crea un registro de dirección que es más fácil de identificar y abstrae la dirección IP pública subyacente.

> [!NOTE]
> Si solo piensa conectarse a la instancia de SQL Server desde de la misma red virtual o de forma local, no necesita etiquetas DNS.

Para crear una etiqueta DNS, seleccione primero **Máquinas virtuales** en el portal. Seleccione su máquina virtual de SQL Server para que aparezcan sus propiedades.

1. En la introducción de la máquina virtual, seleccione **Dirección IP pública**.

    ![dirección ip pública](./media/virtual-machines-sql-server-connection-steps/rm-public-ip-address.png)

1. En las propiedades de la dirección IP pública, expanda **Configuración**.

1. Escriba un nombre para la etiqueta DNS. Este nombre es un registro A que se puede usar para conectarse a la máquina virtual de SQL Server utilizando el nombre en lugar de usar directamente la dirección IP.

1. Haga clic en el botón **Guardar**.

    ![etiqueta dns](./media/virtual-machines-sql-server-connection-steps/rm-dns-label.png)

### <a name="connect-to-the-database-engine-from-another-computer"></a>Conexión al motor de base de datos desde otro equipo

1. En otro equipo que esté conectado a Internet, abra SQL Server Management Studio (SSMS). Si no tiene SQL Server Management Studio, puede descargarla [aquí](/sql/ssms/download-sql-server-management-studio-ssms).

1. En el cuadro de diálogo **Conectar al servidor** o **Conectarse al motor de base de datos**, edite el valor de **Nombre del servidor**. Escriba la dirección IP o el nombre DNS completo de la máquina virtual (que se determinó en la tarea anterior). También puede agregar una coma y proporcionar el puerto TCP de SQL Server. Por ejemplo, `mysqlvmlabel.eastus.cloudapp.azure.com,1433`.

1. En el cuadro **Autenticación**, seleccione **Autenticación de SQL Server**.

1. En el cuadro **Inicio de sesión** , escriba un nombre de inicio de sesión de SQL válido.

1. En el cuadro **Contraseña** , escriba la contraseña de inicio de sesión.

1. Haga clic en **Conectar**.

    ![conexión ssms](./media/virtual-machines-sql-server-connection-steps/rm-ssms-connect.png)