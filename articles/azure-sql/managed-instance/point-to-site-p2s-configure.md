---
title: Configuración de la conectividad de punto a sitio mediante SSMS
titleSuffix: Azure SQL Managed Instance
description: Conéctese a Instancia administrada de Azure SQL mediante SQL Server Management Studio (SSMS) con una conexión de punto a sitio desde un equipo cliente local.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, jovanpop
ms.date: 03/13/2019
ms.openlocfilehash: 791c8a754a83762b03290e98b2e05cd2626ad672
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694012"
---
# <a name="quickstart-configure-a-point-to-site-connection-to-azure-sql-managed-instance-from-on-premises"></a>Inicio rápido: Configuración de una conexión de punto a sitio a Instancia administrada de Azure SQL desde el entorno local
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este inicio rápido se muestra cómo conectarse a Instancia administrada de Azure SQL mediante [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) (SSMS) desde un equipo cliente local a través de una conexión de punto a sitio. Para más información sobre las conexiones de punto a sitio, consulte [Acerca de las conexiones VPN de punto a sitio](../../vpn-gateway/point-to-site-about.md).

## <a name="prerequisites"></a>Requisitos previos

En esta guía de inicio rápido:

- Se usan los recursos creados en [Creación de una Instancia administrada](instance-create-quickstart.md) como punto de partida.
- Se requiere PowerShell 5.1 y Azure PowerShell 1.4.0 o posterior en el equipo cliente local. Si es necesario, consulte las instrucciones para la [instalación del módulo de Azure PowerShell](/powershell/azure/install-az-ps#install-the-azure-powershell-module).
- Se requiere la versión más reciente de [SQL Server Management Studio](/sql/ssms/sql-server-management-studio-ssms) en el equipo cliente local.

## <a name="attach-a-vpn-gateway-to-a-virtual-network"></a>Asociación de una puerta de enlace de VPN a una red virtual

1. Abra PowerShell en el equipo cliente local.

2. Copie este script de PowerShell. Este script asocia una puerta de enlace de VPN a la red virtual de Instancia administrada de SQL que creó en el inicio rápido [Creación de una instancia administrada](instance-create-quickstart.md). Este script usa el módulo Az de Azure PowerShell y realiza las siguientes acciones en hosts basados en Windows o Linux:

   - Crea e instala certificados en una máquina cliente
   - Calcula el intervalo futuro de IP de subred de puerta de enlace de VPN
   - Crea la subred de puerta de enlace
   - Implementa la plantilla de Azure Resource Manager que asocia la puerta de enlace de VPN a la subred VPN

     ```powershell
     $scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/attach-vpn-gateway'

     $parameters = @{
       subscriptionId = '<subscriptionId>'
       resourceGroupName = '<resourceGroupName>'
       virtualNetworkName = '<virtualNetworkName>'
       certificateNamePrefix  = '<certificateNamePrefix>'
       }

     Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/attachVPNGateway.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters, $scriptUrlBase
     ```

3. Pegue el script en la ventana de PowerShell y proporcione los parámetros requeridos. Los valores de `<subscriptionId>`, `<resourceGroup>` y `<virtualNetworkName>` deben coincidir con los que se usan en el inicio rápido [Creación de una Instancia administrada](instance-create-quickstart.md). El valor de `<certificateNamePrefix>` puede ser la cadena que prefiera.

4. Ejecute el script de PowerShell.

> [!IMPORTANT]
> No continúe hasta que se complete el script de PowerShell.

## <a name="create-a-vpn-connection"></a>Creación de una conexión VPN

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
2. Abra el grupo de recursos en el que creó la puerta de enlace de red virtual y, a continuación, abra el recurso de puerta de enlace de red virtual.
3. Seleccione **Configuración de punto a sitio** y, después, **Descargar cliente VPN**.

    ![Descarga del cliente VPN](./media/point-to-site-p2s-configure/download-vpn-client.png)  
4. En el equipo cliente local, extraiga los archivos del archivo ZIP y, a continuación, abra la carpeta con los archivos extraídos.
5. Abra la carpeta **WindowsAmd64** y el archivo **VpnClientSetupAmd64.exe**.
6. Si recibe un mensaje que indica que **Windows protegió su PC**, haga clic en **Más información** y, a continuación, haga clic en **Ejecutar de todas formas**.

    ![Instalación del cliente VPN](./media/point-to-site-p2s-configure/vpn-client-defender.png)
7. Seleccione **Sí** en el cuadro de diálogo Control de cuentas de usuario para continuar.
8. En el cuadro de diálogo que hace referencia a la red virtual, seleccione **Sí** para instalar el cliente VPN de la red virtual.

## <a name="connect-to-the-vpn-connection"></a>Conexión a VPN

1. Vaya a **VPN** en **Red e Internet** en el equipo cliente local y seleccione la red virtual de Instancia administrada de SQL para establecer una conexión a esta red virtual. En la siguiente imagen, la red virtual se denomina **MyNewVNet**.

    ![Conexión VPN](./media/point-to-site-p2s-configure/vpn-connection.png)  
2. Seleccione **Conectar**.
3. En el cuadro de diálogo, seleccione **Conectar**.

    ![Captura de pantalla en la que se resalta el botón Conectar.](./media/point-to-site-p2s-configure/vpn-connection2.png)  
4. Cuando se le indique que Connection Manager necesita privilegios elevados para actualizar la tabla de rutas, elija **Continuar**.
5. Seleccione **Sí** en el cuadro de diálogo Control de cuentas de usuario para continuar.

   Ha establecido una conexión VPN a la red virtual de Instancia administrada de SQL.

    ![Captura de pantalla en la que se resalta el mensaje Conectado cuando se ha establecido la conexión.](./media/point-to-site-p2s-configure/vpn-connection-succeeded.png)  

## <a name="connect-with-ssms"></a>Conectarse con SSMS

1. En el equipo cliente local, abra SQL Server Management Studio.
2. En el cuadro de diálogo **Conectar con el servidor**, escriba el **nombre de host** completo para su instancia administrada en el cuadro **Nombre del servidor**.
3. Seleccione **Autenticación de SQL Server**, proporcione su nombre de usuario y contraseña y, a continuación, seleccione **Conectar**.

    ![Conexión a SSMS](./media/point-to-site-p2s-configure/ssms-connect.png)  

Después de conectarse, podrá ver las bases de datos del usuario y del sistema en el nodo Bases de datos. También podrá ver varios objetos en los nodos Seguridad, Objetos del servidor, Replicación, administración, Agente SQL Server y XEvent Profiler.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener una guía de inicio rápido que muestra cómo conectarse desde una máquina virtual de Azure, consulte [Configuración de una conexión de punto a sitio](point-to-site-p2s-configure.md).
- Para información general sobre las opciones de conexión de las aplicaciones, consulte [Conexión de las aplicaciones a Instancia administrada de SQL](connect-application-instance.md).
- Para restaurar una base de datos SQL Server existente del entorno local a una instancia administrada, puede usar [Azure Database Migration Service para la migración](../../dms/tutorial-sql-server-to-managed-instance.md), o bien el [comando T-SQL RESTORE](restore-sample-database-quickstart.md) para restaurarla desde un archivo de copia de seguridad de base de datos.