---
title: Implementación de Azure Monitor para soluciones de SAP con Azure Portal
description: Implementación de Azure Monitor para soluciones de SAP con Azure Portal
author: sameeksha91
ms.author: sakhare
ms.topic: how-to
ms.service: virtual-machines-sap
ms.date: 08/17/2020
ms.openlocfilehash: a9208101777cd88f0237e661a414550759a069b0
ms.sourcegitcommit: aaba99b8b1c545ad5d19f400bcc2d30d59c63f39
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/26/2021
ms.locfileid: "108007388"
---
# <a name="deploy-azure-monitor-for-sap-solutions-with-azure-portal"></a>Implementación de Azure Monitor para soluciones de SAP con Azure Portal

Los recursos de Azure Monitor para soluciones de SAP se pueden crear mediante [Azure Portal](https://azure.microsoft.com/features/azure-portal). Este método proporciona una interfaz de usuario basada en explorador para implementar Azure Monitor para soluciones de SAP y configurar los proveedores.

## <a name="sign-in-to-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-monitoring-resource"></a>Creación de recurso de supervisión

1. Seleccione **Azure Monitor para soluciones de SAP** en **Azure Marketplace**.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-1.png" alt-text="La imagen muestra la selección de la oferta Azure Monitor para soluciones de SAP en Azure Marketplace." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-1.png":::

2. En la pestaña **Aspectos básicos**, proporcione los valores necesarios. Si corresponde, puede usar un área de trabajo de Log Analytics existente.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-2.png" alt-text="Visualización de las opciones de configuración de Azure Portal." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-2.png":::

3. Al seleccionar una red virtual, asegúrese de que los sistemas que quiere supervisar son accesibles desde dentro de esa red virtual. 

   > [!IMPORTANT]
   > La selección de **Compartir** para el uso compartido de datos con Microsoft permite que nuestros equipos de soporte técnico proporcionen soporte adicional.

## <a name="configure-providers"></a>Configurar proveedores

### <a name="sap-netweaver-provider"></a>Proveedor de SAP NetWeaver

#### <a name="prerequisites-for-adding-netweaver-provider"></a>Requisitos previos para agregar el proveedor de NetWeaver

El "servicio de inicio de SAP" proporciona una serie de servicios, incluida la supervisión del sistema SAP. Aprovecharemos "SAPControl", que es una interfaz de servicio web SOAP que expone estas funcionalidades. Esta interfaz de servicio web de SAPControl diferencia entre los métodos de [servicio web protegidos y no protegidos](https://wiki.scn.sap.com/wiki/display/SI/Protected+web+methods+of+sapstartsrv). Para poder capturar métricas específicas, deberá desproteger algunos métodos. Para desproteger los métodos necesarios para la versión actual, siga los pasos que se indican a continuación para cada sistema SAP:

1. Abra una conexión de GUI de SAP al servidor SAP.
2. Inicie sesión con una cuenta administrativa.
3. Ejecute la transacción RZ10.
4. Seleccione el perfil adecuado (DEFAULT.PFL).
5. Seleccione "Mantenimiento extendido" y haga clic en Cambiar. 
6. Modifique el valor del parámetro afectado "service/protectedwebmethods" a "SDEFAULT -GetQueueStatistic –ABAPGetWPTable –EnqGetStatistic –GetProcessList" a la configuración recomendada y haga clic en Copiar.
7. Vuelva atrás y seleccione Perfil->Guardar.
8. Reinicie el sistema para que el parámetro surta efecto.

>[!Tip]
> Use una lista de control de acceso (ACL) para filtrar el acceso a un puerto de servidor. Consulte esta [nota de SAP](https://launchpad.support.sap.com/#/notes/1495075).

#### <a name="installing-netweaver-provider-on-the-azure-portal"></a>Instalación del proveedor de NetWeaver en Azure Portal
1.  Asegúrese de que se han completado los pasos previos y de que se ha reiniciado el servidor.
2.  En Azure Portal, en AMS, seleccione Agregar proveedor y elija SAP NetWeaver en la lista desplegable.
3.  Especifique el nombre de host del sistema SAP y el subdominio (si procede).
4.  Escriba el número de instancia correspondiente al nombre de host especificado. 
5.  Escriba el id. del sistema (SID).
6.  Cuando termine, seleccione Agregar proveedor.
7.  Siga agregando proveedores adicionales según sea necesario o seleccione Revisar y crear para completar la implementación.

![imagen](https://user-images.githubusercontent.com/75772258/114583569-5c777d80-9c9f-11eb-99a2-8c60987700c2.png)

### <a name="sap-hana-provider"></a>Proveedor de SAP HANA 

1. Seleccione la pestaña **Proveedor** para agregar los proveedores que quiere configurar. Puede agregar varios proveedores uno tras otro o agregarlos después de implementar el recurso de supervisión. 

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-3.png" alt-text="Muestra la pestaña de proveedor para agregar proveedores adicionales a Azure Monitor para soluciones de SAP." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-3.png":::

2. Seleccione **Agregar proveedor** y elija **SAP HANA** en el menú desplegable. 

   > [!IMPORTANT]
   > Asegúrese de que el proveedor de SAP HANA esté configurado para el nodo "maestro" de SAP HANA.

3. Escriba la dirección IP privada del servidor HANA.

4. Escriba el nombre del inquilino de la base de datos que quiere usar. Puede elegir cualquier inquilino, pero se recomienda usar **SYSTEMDB** porque permite una matriz más amplia de áreas de supervisión. 

5. Escriba el número de puerto de SQL asociado con la base de datos HANA. El número de puerto debe tener el formato de **[3]**  +  **[instance#]**  +  **[13]** . Por ejemplo, 30013. 

6. Escriba el nombre de usuario de la base de datos que quiere usar. Asegúrese de que el usuario de la base de datos tiene asignados los roles de **supervisión** y **lectura de catálogo**. 

7. Cuando termine, seleccione **Agregar proveedor**. Siga agregando más proveedores según sea necesario o seleccione **Revisar y crear** para completar la implementación.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-4.png" alt-text="Imagen de las opciones de configuración al agregar información del proveedor." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-4.png":::
   
### <a name="microsoft-sql-server-provider"></a>Proveedor de Microsoft SQL Server

1. Antes de agregar el proveedor de Microsoft SQL Server, debe ejecutar el script siguiente en SQL Server Management Studio para crear un usuario con los permisos adecuados necesarios para configurar el proveedor.

   ```sql
   USE [<Database to monitor>]
   DROP USER [AMS]
   GO
   USE [master]
   DROP USER [AMS]
   DROP LOGIN [AMS]
   GO
   CREATE LOGIN [AMS] WITH PASSWORD=N'<password>', DEFAULT_DATABASE=[<Database to monitor>], DEFAULT_LANGUAGE=[us_english], CHECK_EXPIRATION=OFF, CHECK_POLICY=OFF
   CREATE USER AMS FOR LOGIN AMS
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GRANT CONNECT TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW SERVER STATE TO AMS
   GRANT VIEW ANY DEFINITION TO AMS
   GRANT EXEC ON xp_readerrorlog TO AMS
   GO
   USE [<Database to monitor>]
   CREATE USER [AMS] FOR LOGIN [AMS]
   ALTER ROLE [db_datareader] ADD MEMBER [AMS]
   ALTER ROLE [db_denydatawriter] ADD MEMBER [AMS]
   GO
   ``` 

2. Seleccione **Agregar proveedor** y elija **Microsoft SQL Server** en el menú desplegable. 

3. Rellene los campos con la información asociada a su instancia de Microsoft SQL Server. 

4. Cuando termine, seleccione **Agregar proveedor**. Siga agregando más proveedores según sea necesario o seleccione **Revisar y crear** para completar la implementación.

     :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-6.png" alt-text="La imagen muestra información relacionada con la incorporación del proveedor de Microsoft SQL Server." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-6.png":::

### <a name="high-availability-cluster-pacemaker-provider"></a>Proveedor de clúster de alta disponibilidad (Pacemaker)

1. Seleccione **Clúster de alta disponibilidad (Pacemaker)** en la lista desplegable. 

   > [!IMPORTANT]
   > Para configurar el proveedor de clúster de alta disponibilidad (Pacemaker), asegúrese de que ha_cluster_provider esté instalado en cada nodo. Para más información, consulte el artículo sobre el [exportador de clúster de alta disponibilidad](https://github.com/ClusterLabs/ha_cluster_exporter#installation).

2. Escriba el punto de conexión de Prometheus con el formato http://IP:9664/metrics. 
 
3. Escriba el id. del sistema (SID), el nombre de host y el nombre del clúster.

4. Cuando termine, seleccione **Agregar proveedor**. Siga agregando más proveedores según sea necesario o seleccione **Revisar y crear** para completar la implementación.

   :::image type="content" source="./media/azure-monitor-sap/azure-monitor-quickstart-5.png" alt-text="La imagen muestra las opciones relacionadas con el proveedor de Pacemaker del clúster de alta disponibilidad." lightbox="./media/azure-monitor-sap/azure-monitor-quickstart-5.png":::

### <a name="os-linux-provider"></a>Proveedor del SO (Linux) 

1. Selección del SO (Linux) en la lista desplegable 

   >[!IMPORTANT]
   > Para configurar el proveedor de sistema operativo (Linux), asegúrese de que la versión más reciente de Node_Exporter esté instalada en cada host (BareMetal o VM) que desee supervisar. Instale la [versión más reciente de Node Exporter.](https://prometheus.io/download/#node_exporter) Haga clic aquí para [obtener más información](https://github.com/prometheus/node_exporter).

2. Escriba un nombre, que será el identificador de la instancia de BareMetal.
3. Escriba el punto de conexión Node_Exporter con el formato http://IP:9100/metrics.

   >[!IMPORTANT]
   >Use la dirección IP privada del host de Linux. Asegúrese de que el host y el recurso de AMS se encuentran en la misma red virtual. 

   >[!Note]
   > El puerto de firewall "9100" debe estar abierto en el host de Linux.
   >Si usa firewall-cmd: firewall-cmd --permanent --add-port=9100/tcp firewall-cmd --reload. Si usa ufw: ufw allow 9100/tcp ufw reload

    >[!Tip]
    > Si el host de Linux es una máquina virtual de Azure, asegúrese de que todos los NSG aplicables permiten el tráfico entrante en el puerto 9100 desde "VirtualNetwork" como origen.
 
5. Cuando termine, seleccione  **Agregar proveedor**. Continúe agregando más proveedores según sea necesario o seleccione  **Revisar y crear**  para completar la implementación. 


## <a name="next-steps"></a>Pasos siguientes

Más información sobre [Azure Monitor para soluciones de SAP](azure-monitor-overview.md)
