---
title: Implementación de la plataforma de SAP BusinessObjects BI en Azure para Windows | Microsoft Docs
description: Implemente y configure la plataforma de SAP BusinessObjects BI en Azure para Windows.
services: virtual-machines-windows,virtual-network,storage,azure-netapp-files,azure-files,azure-sql
documentationcenter: saponazure
author: dennispadia
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-sap
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/08/2021
ms.author: depadia
ms.openlocfilehash: f520bc6615d9c7ea40612e7d52fb201d04c246e9
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111408126"
---
# <a name="sap-businessobjects-bi-platform-deployment-guide-for-windows-on-azure"></a>Guía de implementación de la plataforma de inteligencia empresarial SAP BusinessObjects para Windows en Azure

En este artículo se describe la estrategia para implementar la plataforma de SAP BusinessObjects Business Intelligence (SAP BOBI) en Azure para Windows. En este ejemplo, se configuran dos máquinas virtuales (VM) con discos administrados SSD prémium de Azure como directorio de instalación. Azure SQL Database, una oferta de plataforma como servicio (PaaS), se usa para el servidor de administración central (CMS) y las bases de datos de auditoría. Azure Premium Files, un protocolo SMB, se usa como un almacén de archivos que se comparte entre ambas máquinas virtuales. La aplicación web de Java de Tomcat predeterminada y la aplicación de la plataforma de inteligencia empresarial (BI) se instalan juntas en ambas máquinas virtuales. Para equilibrar la carga de las solicitudes de los usuarios, se usa Azure Application Gateway, que tiene capacidades de descarga nativas de TLS/SSL.

Este tipo de arquitectura es eficaz para entornos pequeños de implementación o de no producción. Para una implementación de producción o a gran escala, debe separar los hosts para las aplicaciones web, y puede tener varios hosts de aplicaciones de SAP BOBI que permitan al servidor procesar más información.

![Diagrama que muestra una implementación de SAP BOBI en Azure para Windows.](media/businessobjects-deployment-guide/businessobjects-deployment-windows.png)

En este ejemplo, se usan las versiones del producto y el diseño del sistema de archivos siguientes:

- Plataforma de SAP BusinessObjects 4.3 SP01, revisión 1
- Windows Server 2019
- SQL Database (versión 12.0.2000.8)
- Controlador ODBC de Microsoft: msodbcsql.msi (versión: 13.1)

| Sistema de archivos        | Descripción                                                                                                               | Tamaño (GB)             | Acceso necesario  | Storage                    |
|--------------------|---------------------------------------------------------------------------------------------------------------------------|-----------------------|---------------|----------------------------|
| F:          | El sistema de archivos para la instalación de la instancia de SAP BOBI, la aplicación web de Tomcat predeterminada y los controladores de base de datos (según proceda). | Directrices de dimensionamiento de SAP | Privilegios administrativos locales | Discos administrados SSD prémium de Azure|
| \\\azusbobi.file.core.windows.net\frsinput  | El directorio de montaje es para los archivos compartidos en todos los hosts de SAP BOBI que se usarán como directorio de almacén de archivos de entrada. | Necesidad empresarial         | Privilegios administrativos locales | Azure NetApp Files         |
| \\\azusbobi.file.core.windows.net\frsoutput | El directorio de montaje es para los archivos compartidos en todos los hosts de SAP BOBI que se usarán como directorio de almacén de archivos de salida. | Necesidad empresarial         | Privilegios administrativos locales | Azure NetApp Files         |

## <a name="deploy-a-windows-virtual-machine-via-the-azure-portal"></a>Implementación de una máquina virtual Windows en Azure Portal

En esta sección, crearemos dos máquinas virtuales con una imagen del sistema operativo (SO) Windows para la plataforma SAP BOBI. A continuación, se indican los pasos de nivel superior para crear máquinas virtuales:

1. Cree un [grupo de recursos](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups).

1. Cree una [red virtual](../../../virtual-network/quick-create-portal.md#create-a-virtual-network):

   - No use una única subred para todos los servicios de Azure en la implementación de la plataforma SAP BI. En función de la arquitectura de la plataforma SAP BI, puede que tenga que crear varias subredes. En esta implementación, crearemos dos subredes: una subred de aplicación de BI y otra subred de Application Gateway.
   - Siga la nota de SAP [2276646](https://launchpad.support.sap.com/#/notes/2276646) para identificar los puertos para la comunicación de la plataforma SAP BOBI entre distintos componentes.
   - SQL Database se comunica a través del puerto 1433. Se debe permitir el tráfico saliente a través del puerto 1433 desde los servidores de aplicación de SAP BOBI.
   - En Azure, Application Gateway debe estar en una subred independiente. Para más información, consulte [Introducción a la configuración de Application Gateway](../../../application-gateway/configuration-overview.md).
   - En caso de que use Azure NetApp Files para el almacén de archivos en lugar de Azure Files, cree una subred independiente para Azure NetApp Files. Para más información, vea [Directrices para el planeamiento de red de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-network-topologies.md).

1. Cree un conjunto de disponibilidad:

   - Para lograr redundancia para cada nivel en la implementación de varias instancias, coloque las máquinas virtuales de cada nivel en un conjunto de disponibilidad. Asegúrese de separar los conjuntos de disponibilidad para cada nivel en función de su arquitectura.

1. Cree la máquina virtual 1 (azuswinboap1):

   - Puede usar una imagen personalizada o elegir una imagen de Azure Marketplace. En función de sus necesidades, consulte [Implementación de una máquina virtual desde Azure Marketplace para SAP](deployment-guide.md) o [Implementación de una máquina virtual con una imagen personalizada para SAP](deployment-guide.md).

1. Cree la máquina virtual 2 (azuswinboap2).
1. Agregue un disco SSD Premium. Se usará como directorio de instalación de SAP BOBI.

## <a name="provision-azure-premium-files"></a>Aprovisionamiento de Azure Premium Files

Antes de continuar con la configuración de Azure Files, familiarícese con la [documentación correspondiente](../../../storage/files/storage-files-introduction.md).

Azure Files ofrece recursos compartidos de archivos estándar que se hospedan en hardware basado en HDD, y recursos compartidos de archivos prémium que se hospedan en hardware basado en SSD. Para un almacén de archivos de SAP BusinessObjects, use Azure Premium Files.

Los recursos compartidos de Azure Premium Files están disponibles con redundancia local y de zona en un subconjunto de regiones. Para averiguar si los recursos compartidos de archivos prémium están disponibles actualmente en su región, consulte [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage). Para información sobre las regiones que admiten el almacenamiento con redundancia de zona (ZRS), consulte [Redundancia de Azure Storage](../../../storage/common/storage-redundancy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="deploy-an-azure-files-storage-account-and-nfs-shares"></a>Implementación de una cuenta de almacenamiento de archivos de Azure y recursos compartidos de NFS

Los recursos compartidos de archivos de Azure se implementan en cuentas de almacenamiento, que son objetos de nivel superior que representan un grupo compartido de almacenamiento. Este grupo de almacenamiento se puede usar para implementar varios recursos compartidos de archivos. Azure admite varios tipos de cuentas de almacenamiento para los distintos escenarios de almacenamiento que los clientes pueden tener. Para el almacenamiento de archivos de SAP BusinessObjects, debe crear una cuenta de FileStorage. Se usa para implementar recursos compartidos de archivos de Azure en hardware basado en SSD prémium.

> [!NOTE]
> Las cuentas de FileStorage solo se pueden usar para almacenar recursos compartidos de archivos de Azure. No se puede implementar ningún otro recurso de almacenamiento, como blobs, contenedores, colas o tablas, en una cuenta de FileStorage.

Se accederá a la cuenta de almacenamiento a través de un [punto de conexión privado](../../../storage/files/storage-files-networking-endpoints.md), y se implementará en la misma red virtual que la plataforma SAP BOBI. Con esta configuración, el tráfico del sistema SAP nunca sobrepasa los límites de seguridad de la red virtual. A menudo, los sistemas SAP contienen datos confidenciales y críticos para la empresa, por lo que permanecer dentro de los límites de la red virtual es una consideración de seguridad importante para muchos clientes.

Si necesita acceder a la cuenta de almacenamiento desde otra red virtual, puede usar el [emparejamiento de red virtual de Azure](../../../virtual-network/virtual-network-peering-overview.md).

#### <a name="azure-files-storage-account"></a>Cuenta de almacenamiento de archivos de Azure

1. Para crear una cuenta de almacenamiento mediante Azure Portal, seleccione **Crear un recurso** > **Almacenamiento** > **Cuenta de Storage**.

1. En la pestaña **Aspectos básicos**, complete todos los campos obligatorios para crear una cuenta de almacenamiento:

   1. Seleccione **Suscripción** > **Grupo de recursos** > **Región**.

   1. Escriba el **nombre de la cuenta de almacenamiento**. Por ejemplo, escriba **azusbobi**. Este nombre debe ser único globalmente, pero, por lo demás, puede ser cualquier nombre que quiera.

   1. Seleccione **Prémium** como nivel de rendimiento y **FileStorage** como el tipo de cuenta.

   1. En la etiqueta **Replicación**, elija un nivel de redundancia, seleccione **Almacenamiento con redundancia local (LRS)**.

      Para FileStorage prémium, LRS y ZRS son las únicas opciones disponibles. En función de la estrategia de implementación (conjunto de disponibilidad o zona de disponibilidad), elija el nivel de redundancia adecuado. Para más información, vea [Redundancia de Azure Storage](../../../storage/common/storage-redundancy.md).

   1. Seleccione **Siguiente**.

1. En la pestaña **Redes**, seleccione [Punto de conexión privado](../../../storage/files/storage-files-networking-endpoints.md) para el método de conectividad. Para más información, vea [Consideraciones de redes para Azure Files](../../../storage/files/storage-files-networking-overview.md).

   1. Seleccione **Agregar** en la sección de puntos de conexión privados.

   1. Seleccione **Suscripción** > **Grupo de recursos** > **Ubicación**.

   1. Escriba el **Nombre** del punto de conexión privado. Por ejemplo, escriba **azusbobi-pe**.

   1. Seleccione **Archivo** en **Subrecurso de almacenamiento**.

   1. En la sección **Redes**, seleccione la **Red virtual** y la **Subred** donde se ha implementado la aplicación SAP BusinessObjects BI.

   1. Acepte el **valor predeterminado (sí)** para **Integrar con la zona DNS privada**.

   1. Seleccione la **Zona DNS privada** en la lista desplegable.

   1. Seleccione **Aceptar** para volver a la pestaña **Redes** en **Crear cuenta de almacenamiento**.

1. En la pestaña **Protección de datos**, configure la directiva de eliminación temporal para recursos compartidos de archivos de Azure en la cuenta de almacenamiento. De manera predeterminada, la funcionalidad de eliminación temporal está desactivada. Para más información sobre la eliminación temporal, consulte [Evitar la eliminación accidental de recursos compartidos de archivos de Azure](../../../storage/files/storage-files-prevent-file-share-deletion.md).

1. En la pestaña **Avanzadas**, seleccione las distintas opciones de seguridad.

   El campo **Se requiere transferencia segura** indica si la cuenta de almacenamiento requiere cifrado en tránsito para la comunicación con la cuenta de almacenamiento. Si requiere compatibilidad con SMB 2.1, tiene que deshabilitar este campo. Para la plataforma SAP BOBI, conserve el **valor predeterminado (habilitado)** .

1. Continúe y cree la cuenta de almacenamiento.

Para más información sobre cómo crear una cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento de FileStorage](../../../storage/files/storage-how-to-create-file-share.md).

#### <a name="create-azure-file-shares"></a>Crear recursos compartidos de archivos de Azure

El siguiente paso consiste en crear archivos de Azure en la cuenta de almacenamiento. Azure Files usa un modelo aprovisionado para los recursos compartidos de archivos prémium. En un modelo de negocio aprovisionado, se especifica de forma proactiva a los archivos de Azure cuáles son los requisitos de almacenamiento, en lugar de que se le aplique una factura basada en lo que usa. Para comprender mejor este modelo, consulte [Modelo aprovisionado](../../../storage/files/understanding-billing.md#provisioned-model). En este ejemplo, se crean dos instancias de archivos de Azure: frsinput (256 GB) y frsoutput (256 GB) para el almacén de archivos de SAP BOBI.

1. Vaya a la cuenta de almacenamiento **azusbobi** > **Recursos compartidos de archivos**.
1. Seleccione **Nuevo recurso compartido de archivos**.
1. Escriba el **Nombre** del recurso compartido de archivos. Por ejemplo, escriba **frsinput** o **frsoutput**.
1. Indique el tamaño de recurso compartido de archivos necesario en **Capacidad aprovisionada**. Por ejemplo, escriba **256 GB**.
1. Elija **SMB** como **Protocolo**.
1. Seleccione **Crear**.

## <a name="configure-a-data-disk-on-a-windows-virtual-machine"></a>Configuración de un disco de datos en una máquina virtual Windows

En los pasos de esta sección se usan los siguientes prefijos:

**[A]** : El paso se aplica a todos los hosts.

### <a name="initialize-a-new-data-disk"></a>Inicio de un nuevo disco de datos

La aplicación SAP BusinessObjects BI requiere una partición en la que se puedan instalar sus archivos binarios. Puede instalar la aplicación SAP BOBI en la partición del sistema operativo (C:), pero debe asegurarse de tener suficiente espacio para la implementación y el sistema operativo. Se recomienda tener al menos 2 GB disponibles para archivos temporales y aplicaciones web. También se aconseja separar los archivos binarios de instalación de SAP BOBI en una partición independiente.

En este ejemplo, una aplicación SAP BOBI se instala en una partición independiente (F:). Inicialice el disco SSD prémium que ha conectado durante el aprovisionamiento de la máquina virtual:

1. **[A]** Si no hay ningún disco de datos conectado a la máquina virtual (azuswinboap1 y azuswinboap2), siga los pasos descritos en [Agregar un disco de datos](../../windows/attach-managed-disk-portal.md#add-a-data-disk) para conectar un nuevo disco de datos administrado.
1. **[A]** Después de conectar el disco administrado a la máquina virtual, inicialice el disco siguiendo los pasos descritos en [Inicialización de un nuevo disco de datos](../../windows/attach-managed-disk-portal.md#initialize-a-new-data-disk).

### <a name="mount-azure-premium-files"></a>Montaje de Azure Premium Files

Para usar Azure Files como almacén de archivos, tiene que montarlo, lo que significa asignarle una letra de unidad o una ruta de acceso a un punto de montaje.

**[A]** Para montar un recurso compartido de archivos de Azure, siga los pasos de [Montaje del recurso compartido de archivos de Azure](../../../storage/files/storage-how-to-use-files-windows.md#mount-the-azure-file-share).

Para montar un recurso compartido de archivos de Azure en Windows Server, asegúrese de que el puerto 445 esté abierto. El protocolo SMB requiere que esté abierto el puerto TCP 445. Se producirá un error en las conexiones si se bloquea el puerto 445. Puede comprobar si el firewall está bloqueando el puerto 445 con el cmdlet `Test-NetConnection` mencionado en la guía de [solución de problemas](../../../storage/files/storage-troubleshoot-windows-file-connection-problems.md#cause-1-port-445-is-blocked).

## <a name="configure-a-cms-database-azure-sql"></a>Configuración de la base de datos CMS: Azure SQL

En esta sección se proporcionan detalles sobre cómo aprovisionar Azure SQL mediante Azure Portal. También se proporcionan instrucciones sobre cómo crear las bases de datos CMS y de auditoría para la plataforma SAP BOBI y una cuenta de usuario para acceder a la base de datos.

Las instrucciones solo se aplican si usa SQL Database. En el caso de otras bases de datos, vea la documentación específica de SAP o de la base de datos para obtener instrucciones.

### <a name="create-a-sql-database-server"></a>Creación de un servidor de SQL Database

SQL Database ofrece diferentes opciones de implementación: base de datos única, grupo elástico y servidor de bases de datos. Para una plataforma SAP BOBI, necesitamos dos bases de datos, CMS y de auditoría. En lugar de crear dos bases de datos únicas, puede crear un servidor de SQL Database que pueda administrar el grupo de bases de datos únicas y los grupos elásticos. Siga estos pasos para crear un servidor de SQL Database:

1. Vaya a la página [Seleccione una opción de implementación de SQL](https://portal.azure.com/#create/Microsoft.AzureSQL).
1. En **Bases de datos SQL**, cambie **Tipo de recurso** a **Servidor de bases de datos**. Seleccione **Crear**.
1. En la pestaña **Aspectos básicos**, rellene todos los campos necesarios para **Crear un servidor de SQL Database**:

   1. En **Detalles del proyecto**, seleccione la **Suscripción** y **Grupo de recursos**.

   1. Escriba el **Nombre de servidor**. Por ejemplo, escriba **azussqlbodb**. El nombre del servidor debe ser único globalmente, pero, por lo demás, puede ser cualquier nombre que quiera.

   1. Seleccione la **Ubicación**.

   1. Escriba el **Inicio de sesión del administrador del servidor**. Por ejemplo, escriba **boadmin**. A continuación, escriba una **Contraseña**.

1. En la pestaña **Redes**, cambie **Permitir que los servicios y recursos de Azure accedan a este servidor** a **No** en **Reglas de firewall**.
1. En **Configuración adicional**, mantenga la configuración predeterminada.
1. Continúe y cree el **Servidor de SQL Database**.

En el paso siguiente, cree las bases de datos CMS y de auditoría en el servidor de SQL Database (azussqlbodb.database.windows.net).

### <a name="create-the-cms-and-the-audit-database"></a>Creación de las bases de datos CMS y de auditoría

Después de aprovisionar el servidor de SQL Database, vaya al recurso azussqlbodb. A continuación, siga estos pasos para crear las bases de datos CMS y de auditoría.

1. En la página de **información general de azussqlbodb**, seleccione **Crear base de datos**.
1. En la pestaña **Aspectos básicos**, rellene todos los campos necesarios:

   1. Escriba el **Nombre de la base de datos**. Por ejemplo, escriba **bocms** o **boaudit**.

   1. En la opción **Proceso y almacenamiento**, seleccione **Configurar base de datos**. Elija el modelo adecuado en función del resultado de dimensionamiento. Para información sobre las opciones, vea [Modelos de ajuste de tamaño para Azure SQL Database](businessobjects-deployment-guide.md#sizing-models-for-azure-sql-database).
1. En la pestaña **Redes**, seleccione [Punto de conexión privado](../../../private-link/tutorial-private-endpoint-sql-portal.md) para el método de conectividad. El punto de conexión privado se usará para acceder a SQL Database dentro de la red virtual configurada.

   1. Seleccione **Agregar punto de conexión privado**.

   1. Seleccione **Suscripción** > **Grupo de recursos** > **Ubicación**.

   1. Escriba el **Nombre** del punto de conexión privado. Por ejemplo, escriba **azusbodb-pe**.

   1. En **Subrecurso de destino**, seleccione **SqlServer**.

   1. En la sección **Redes**, seleccione la **Red virtual** y la **Subred** donde se ha implementado la aplicación SAP BusinessObjects BI.

   1. Acepte el **valor predeterminado (sí)** para **Integrar con la zona DNS privada**.

   1. Seleccione la **Zona DNS privada** en la lista desplegable.

   1. Seleccione **Aceptar** para volver a la pestaña **Redes** en **Crear base de datos SQL**.
1. En la pestaña **Configuración adicional**, cambie la configuración de **Intercalación** a **SQL_Latin1_General_CP850_BIN2**.
1. Continúe y cree la base de datos CMS.

De forma similar, puede crear la base de datos de auditoría. Por ejemplo, escriba **boaudit**.

### <a name="download-and-install-an-odbc-driver"></a>Descarga e instalación de un controlador ODBC

Los servidores de aplicaciones de SAP BOBI requieren que el cliente o controladores de la base de datos accedan a la base de datos CMS o de auditoría. El controlador ODBC de Microsoft se usa para acceder a las bases de datos CMS y de auditoría que se ejecutan en SQL Database. En esta sección se proporcionan instrucciones sobre cómo descargar y configurar un controlador ODBC en Windows. 

1. Vea la sección **Compatibilidad del repositorio CMS + auditoría por SO** en [Matriz de disponibilidad de productos (PAM) para la plataforma SAP BusinessObjects BI](https://support.sap.com/pam) para averiguar los conectores de base de datos que son compatibles con SQL Database.
1. Descargue la versión del controlador ODBC desde este [vínculo](/sql/connect/odbc/windows/release-notes-odbc-sql-server-windows?preserve-view=true&view=sql-server-ver15). En este ejemplo, vamos a descargar el controlador ODBC [13.1](/sql/connect/odbc/windows/release-notes-odbc-sql-server-windows?preserve-view=true&view=sql-server-ver15#131).
1. Instale el controlador ODBC en todos los servidores de inteligencia empresarial (azuswinboap1 y azuswinboap2).
1. Después de instalar el controlador en **azuswinboap1**, vaya a **Inicio** > **Herramientas administrativas de Windows** > **Orígenes de datos ODBC (64 bits)** .
1. Vaya a la pestaña **Sistema DSN**. 
1. Seleccione **Agregar** para crear una conexión a la base de datos CMS.
1. Seleccione **Controlador ODBC 13 para SQL Server** y seleccione **Finalizar**.
1. Escriba la información de la base de datos CMS como se muestra a continuación y seleccione **Siguiente**:
   - **Nombre:** el nombre de la base de datos creada en la sección "Creación de las bases de datos CMS y de auditoría". Por ejemplo, escriba **bocms** o **boaudit**.
   - **Descripción:** la descripción que describe el origen de datos. Por ejemplo, escriba **Base de datos CMS** o **Base de datos de auditoría**.
   - **Servidor:** el nombre del servidor creado en la sección "Creación de un servidor de SQL Database". Por ejemplo, escriba azussqlbodb.database.windows.net.
1. Seleccione **With SQL Server authentication using a login ID and password entered by user** (Con la autenticación de SQL Server mediante un id. de inicio de sesión y una contraseña escritos por el usuario) para comprobar la autenticidad en Azure SQL Server. Escriba la credencial de usuario que se creó en el momento de la creación del servidor de SQL Database. Por ejemplo, escriba **boadmin**. Seleccione **Siguiente**.
1. Cambie la **base de datos predeterminada** a **bocms** y mantenga todo lo demás con los valores predeterminados. Seleccione **Siguiente**.
1. Seleccione la casilla **Usar cifrado de alta seguridad para los datos** y mantenga todo lo demás con los valores predeterminados. Seleccione **Finalizar**.
1. Se ha creado el origen de datos de la base de datos CMS. Ahora puede seleccionar **Origen de datos de prueba** para validar la conexión a la base de datos CMS desde la aplicación BI. Debe completarse con éxito. Si se produce un error, [solucione](../../../azure-sql/database/troubleshoot-common-errors-issues.md) el problema de conectividad.

>[!Note]
>SQL Database se comunica a través del puerto 1433. Se debe permitir el tráfico saliente a través del puerto 1433 desde los servidores de aplicación de SAP BOBI.

Repita los mismos pasos anteriores para crear una conexión para la base de datos de auditoría en el servidor azuswinboap1. De manera similar, instale y configure ambos orígenes de datos ODBC (bocms y boaudit) en todos los servidores de aplicaciones BI (azuswinboap2). 

## <a name="server-preparation"></a>Preparación del servidor

Siga la guía más reciente de SAP para preparar los servidores para la instalación de la plataforma BI. Para obtener la información más actualizada, consulte la sección "Preparación" del [Manual de instalación de la plataforma SAP Business Intelligence para Windows](https://help.sap.com/viewer/df8899896b364f6c880112f52e4d06c8/4.3.1/en-US/46b0d1a26e041014910aba7db0e91070.html).

## <a name="installation"></a>Instalación

Para instalar la plataforma de BI en un host de Windows, inicie sesión con un usuario que tenga privilegios administrativos locales.

Vaya a los medios de la plataforma de SAP BusinessObjects BI y ejecute `setup.exe`.

Siga las instrucciones del [Manual de instalación de la plataforma SAP Business Intelligence para Windows](https://help.sap.com/viewer/df8899896b364f6c880112f52e4d06c8/4.3.1/en-US/46ae62456e041014910aba7db0e91070.html) que son específicas de su versión. Estos son algunos puntos a tener en cuenta al instalar la plataforma SAP BOBI en Windows:

- En la pantalla **Configure Destination Folder** (Configurar carpeta de destino), proporcione la carpeta de destino donde quiere instalar la plataforma BI. Por ejemplo, especifique **F:\SAP BusinessObjects\*** . 
- En la pantalla **Configure Product Registration** (Configurar el registro del producto), puede usar una clave de licencia temporal para las soluciones de SAP BusinessObjects de la nota de SAP [1288121](https://launchpad.support.sap.com/#/notes/1288121), o puede generar una clave de licencia en el marketplace de servicios de SAP.
- En la pantalla **Select Install Type** (Seleccionar tipo de instalación), seleccione la instalación **Completa** en el primer servidor (azuswinboap1). Para el otro servidor (azuswinboap2), seleccione **Custom / Expand** (Personalizar/Expandir), que expande la configuración de SAP BOBI existente.
- En **Select Default or Existing Database** (Seleccionar base de datos predeterminada o existente), seleccione **Configure an existing database** (Configurar una base de datos existente), que le solicitará que seleccione la base de datos CMS y de auditoría. Seleccione **Microsoft SQL Server using ODBC** (Microsoft SQL Server con ODBC) para el tipo **CMS Database** (Base de datos CMS) y el tipo **Audit Database** (Base de datos de auditoría).

  También puede seleccionar **No auditing database** (Ninguna base de datos de auditoría) si no quiere configurar la auditoría durante la instalación.

- Seleccione las opciones adecuadas en la pantalla **Select Java Web Application Server** (Seleccionar servidor de aplicaciones web de Java) según la arquitectura de SAP BOBI. En este ejemplo, se ha seleccionado la opción 1, que instala el servidor de Tomcat en la misma plataforma SAP BOBI.
- Escriba la información de la base de datos CMS en **Configure CMS Repository Database - SQL Server (ODBC)** (Configurar la base de datos del repositorio de CMS: SQL Server [ODBC]). En la imagen siguiente se muestra una entrada de ejemplo para la información de la base de datos CMS para una instalación de Windows.
  
  ![Captura de pantalla que muestra la información de la base de datos CMS para Windows.](media/businessobjects-deployment-guide/businessobjects-deployment-windows-sql-cms.png)
- (Opcional) Escriba la información de la base de datos de auditoría en **Configure Auditing Database - SQL Server (ODBC)** (Configurar la base de datos de auditoría: SQL Server [ODBC]). En la imagen siguiente se muestra una entrada de ejemplo para la información de la base de datos de auditoría para una instalación de Windows.
  
  ![Captura de pantalla que muestra la información de la base de datos de auditoría para Windows.](media/businessobjects-deployment-guide/businessobjects-deployment-windows-sql-audit.png)

- Siga las instrucciones e introduzca las entradas necesarias para finalizar la instalación.

Para una implementación de varias instancias, ejecute la configuración de la instalación en el segundo host (azuswinboap2). En la pantalla **Select Install Type** (Seleccionar tipo de instalación), seleccione **Custom / Expand** (Personalizar/Expandir), que expande la configuración de SAP BOBI existente. Para más información, vea el blog de SAP [Configuración de la plataforma SAP BusinessObjects Business Intelligence con Azure SQL Database](https://blogs.sap.com/2020/06/19/sap-on-azure-sap-businessobjects-business-intelligence-platform-setup-with-azure-sql-db-managed-paas-database/).

> [!IMPORTANT]
> Los números de versión del motor de base de datos para SQL Server y SQL Database no son comparables entre sí. Son números de compilación internos para estos productos independientes. El motor de base de datos de SQL Database se basa en el mismo código base que el de SQL Server. Lo más importante es que el motor de base de datos de SQL Database siempre tiene los bits más recientes del motor de SQL Database. La versión 12 de SQL Database es más reciente que la versión 15 de SQL Server.

Para averiguar la versión actual de SQL Database, puede comprobar la configuración de la Consola de administración central (CMC), o puede ejecutar la consulta siguiente mediante [sqlcmd](/sql/tools/sqlcmd-utility?preserve-view=true&view=sql-server-ver15) o [SQL Server Management Studio (SSMS)](/sql/ssms/sql-server-management-studio-ssms?preserve-view=true&view=sql-server-ver15). La alineación de las versiones de SQL con la compatibilidad predeterminada se puede encontrar en el artículo sobre el [nivel de compatibilidad de bases de datos](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level?preserve-view=true&view=sql-server-ver15).

![Captura de pantalla que muestra la información de la base de datos en CMC.](media/businessobjects-deployment-guide/businessobjects-deployment-windows-sql-cmc.PNG)

```sql
1> select @@version as version;
2> go
version
------------------------------------------------------------------------------------------
Microsoft SQL Azure (RTM) - 12.0.2000.8
        Feb 20 2021 17:51:58
        Copyright (C) 2019 Microsoft Corporation

(1 rows affected)

1> select name, compatibility_level from sys.databases;
2> go
name                                                                  compatibility_level
--------------------------------------------------------------------- -------------------
master                                                                                150
bocms                                                                                 150
boaudit                                                                               150

(3 rows affected)
```

## <a name="post-installation"></a>Después de la instalación

Después de la instalación de varias instancias de la plataforma SAP BOBI, deben seguirse algunos pasos adicionales posteriores a la configuración para admitir la alta disponibilidad de la aplicación.

### <a name="configure-a-cluster-name"></a>Configuración de un nombre de clúster

En la implementación de varias instancias de la plataforma de SAP BOBI, se quiere ejecutar varios servidores CMS juntos en un clúster. Un clúster consta de dos o más servidores CMS que funcionan conjuntamente en una base de datos común del sistema CMS. Si se produce un error en un nodo que se ejecuta en CMS, un nodo con otro CMS seguirá prestando servicio a las solicitudes de la plataforma de BI. De manera predeterminada, en una plataforma de SAP BOBI, un nombre de clúster refleja el nombre de host del primer CMS que instale. 

Para configurar el nombre del clúster en Windows, siga las instrucciones de la [Guía del administrador de la plataforma de SAP Business Intelligence](https://help.sap.com/viewer/2e167338c1b24da9b2a94e68efd79c42/4.3.1/en-US). Después de configurar el nombre del clúster, siga la nota de SAP [1660440](https://launchpad.support.sap.com/#/notes/1660440) para establecer la entrada predeterminada del sistema en la página de inicio de sesión de Launchpad para CMC o BI. 

### <a name="configure-the-input-and-output-filestore-location-to-azure-premium-files"></a>Configuración de la ubicación del almacén de archivos de entrada y salida en Azure Premium Files

El almacén de archivos hace referencia a los directorios de disco en los que se encuentran los archivos reales de SAP BusinessObjects BI. La ubicación predeterminada del servidor de repositorios de archivos para la plataforma SAP BOBI se encuentra en el directorio de instalación local. En una implementación de varias instancias, es importante configurar el almacén de archivos en un almacenamiento compartido, como Azure Premium Files o Azure NetApp Files, para que se pueda acceder a él desde todos los servidores de capa de almacenamiento.

1. Si no se crea, siga las instrucciones proporcionadas en la sección anterior, "Aprovisionamiento de Azure Premium Files" para crear y montar Azure Premium Files.

   > [!Tip]
   > Elija la redundancia de almacenamiento para Azure Premium Files (LRS o ZRS) en función de la implementación de las máquinas virtuales (conjunto de disponibilidad o zona de disponibilidad).

1. Siga la nota de SAP [2512660](https://launchpad.support.sap.com/#/notes/0002512660) para cambiar la ruta de acceso del repositorio de archivos (entrada y salida).

### <a name="tomcat-clustering-session-replication"></a>Agrupación en clústeres de Tomcat: replicación de sesión

Tomcat admite la agrupación en clústeres de dos o más servidores de aplicaciones para la replicación de sesión y la conmutación por error. Las sesiones de la plataforma de SAP BOBI están serializadas, una sesión de usuario puede conmutar por error a otra instancia de Tomcat, incluso cuando se produce un error en un servidor de aplicaciones. Por ejemplo, un usuario puede estar conectado a un servidor web en el que se produce un error mientras el usuario navega por una jerarquía de carpetas en una aplicación de SAP BI. En un clúster configurado correctamente, el usuario puede seguir navegando por la jerarquía de carpetas sin que se le redirija a la página de inicio de sesión.

En la nota de SAP [2808640](https://launchpad.support.sap.com/#/notes/2808640), se proporcionan los pasos para configurar la agrupación en clústeres de Tomcat mediante multidifusión. Sin embargo, en Azure no se admite la multidifusión. Para que el clúster de Tomcat funcione en Azure, debe usar [StaticMembershipInterceptor](https://tomcat.apache.org/tomcat-8.0-doc/config/cluster-interceptor.html#Static_Membership) (nota de SAP [2764907](https://launchpad.support.sap.com/#/notes/2764907)). Para configurar un clúster de Tomcat en Azure, vea [Agrupación en clústeres de Tomcat mediante pertenencia estática para la plataforma de SAP BusinessObjects BI](https://blogs.sap.com/2020/09/04/sap-on-azure-tomcat-clustering-using-static-membership-for-sap-businessobjects-bi-platform/) en el blog de SAP.

### <a name="load-balance-a-web-tier-of-an-sap-bi-platform"></a>Equilibrio de la carga de un nivel web de una plataforma de SAP BI

En una implementación de varias instancias de SAP BOBI, los servidores de aplicaciones web de Java (nivel web) se ejecutan en dos o más hosts. Para distribuir la carga de usuarios uniformemente entre los servidores web, puede usar un equilibrador de carga entre los usuarios finales y los servidores web. Puede usar Azure Load Balancer o Azure Application Gateway para administrar el tráfico a los servidores de aplicaciones web. Las ofertas se explican en las siguientes secciones:

* [Load Balancer](../../../load-balancer/load-balancer-overview.md) es un equilibrador de carga de nivel 4 (TCP, UDP) de alto rendimiento y baja latencia que distribuye el tráfico entre máquinas virtuales correctas. Un sondeo de estado de equilibrador de carga supervisa un puerto determinado en cada máquina virtual y solo distribuye tráfico a las máquinas virtuales operativas. Puede elegir un equilibrador de carga público o un equilibrador de carga interno, en función de si quiere que la plataforma SAP BI sea accesible desde Internet o no. Tiene redundancia de zona, lo que garantiza una alta disponibilidad en las zonas de disponibilidad.

   En la siguiente figura, vea la sección "Internal Load Balancer", donde el servidor de aplicaciones web se ejecuta en el puerto 8080 (el puerto HTTP predeterminado de Tomcat) que un sondeo de estado supervisará. Cualquier solicitud entrante que proceda de usuarios se redirigirá a los servidores de aplicaciones web (azuswinboap1 o azuswinboap2) en el grupo de back-end. Load Balancer no admite la terminación TLS/SSL, que también se conoce como descarga TLS/SSL. Si usa Load Balancer para distribuir el tráfico entre servidores web, se recomienda usar Standard Load Balancer.

   > [!NOTE]
   > Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no habrá conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo conseguir la conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](high-availability-guide-standard-load-balancer-outbound-connections.md).

   ![Captura de pantalla que muestra una instancia de Load Balancer que se usa para equilibrar el tráfico entre servidores web.](media/businessobjects-deployment-guide/businessobjects-deployment-windows-load-balancer.png)

* [Application Gateway](../../../application-gateway/overview.md) proporciona un controlador de entrega de aplicaciones como servicio, que se usa para ayudar a las aplicaciones a dirigir el tráfico de usuario a uno o varios servidores de aplicaciones web. Ofrece diversas funcionalidades de equilibrio de carga de capa 7, como la descarga SSL/TLS, firewall de aplicaciones web (WAF) y afinidad de sesión basada en cookies para las aplicaciones.

   En una plataforma de SAP BI, Application Gateway dirige el tráfico web de la aplicación a los recursos especificados en un grupo de back-end. En este caso, es azuswinboap1 o azuswinboap2. Se asigna una escucha al puerto, se crean reglas y se agregan recursos a un grupo de back-end. En la figura siguiente, Application Gateway con una dirección IP (10.31.3.25) de front-end privada actúa como punto de entrada para los usuarios, controla las conexiones entrantes de TLS/SSL (HTTPS: TCP/443), descifra TLS/SSL y pasa la solicitud (HTTP: TCP/8080) a los servidores del grupo de back-end. Con la característica de terminación TLS/SSL integrada, es necesario mantener solo un certificado TLS/SSL en la puerta de enlace de aplicación, lo que simplifica las operaciones.

   ![Captura de pantalla que muestra una instancia de Application Gateway que se usa para equilibrar el tráfico entre servidores web.](media/businessobjects-deployment-guide/businessobjects-deployment-windows-application-gateway.png)

   Para configurar Application Gateway para un servidor web de SAP BOBI, vea [Servidores web de SAP BOBI de equilibrio de carga mediante Application Gateway](https://blogs.sap.com/2020/09/17/sap-on-azure-load-balancing-web-application-servers-for-sap-bobi-using-azure-application-gateway/) en el blog de SAP.

   > [!NOTE]
   > Se recomienda usar Application Gateway para equilibrar la carga del tráfico en el servidor web, ya que proporciona características como la descarga SSL, la centralización de la administración de SSL para reducir la sobrecarga de cifrado y descifrado en el servidor, el algoritmo Round-Robin para distribuir el tráfico, las capacidades de Web Application Firewall y la alta disponibilidad.

## <a name="sap-businessobjects-bi-platform-reliability-on-azure"></a>Confiabilidad de la plataforma de SAP BusinessObjects BI en Azure

La plataforma de SAP BusinessObjects BI incluye diferentes niveles, que están optimizados para operaciones y tareas específicas. Cuando un componente de cualquier nivel no esté disponible, la aplicación SAP BOBI dejará de estar accesible o no funcionará ninguna de sus funcionalidades. Debe asegurarse de que cada nivel esté diseñado para ser confiable y mantener la aplicación operativa sin interrupción empresarial.

En esta guía se explora cómo las características nativas de Azure en combinación con la configuración de una plataforma de SAP BOBI mejoran la disponibilidad de una implementación de SAP. Esta sección se centra en las siguientes opciones de la confiabilidad de la plataforma de SAP BOBI en Azure:

- **Copia de seguridad y restauración**: este proceso crea copias periódicas de datos y aplicaciones en ubicaciones independientes. Si los datos o aplicaciones originales se pierden o se dañan, las copias se pueden usar para restaurar o recuperar el estado anterior.
- **Alta disponibilidad**: una plataforma de alta disponibilidad tiene al menos dos de todo en la región de Azure para mantener la aplicación operativa si uno de los servidores deja de estar disponible.
- **Recuperación ante desastres (DR)** : este proceso restaura la funcionalidad de la aplicación si hay pérdidas catastróficas. Por ejemplo, toda una región de Azure podría dejar de estar disponible debido a un desastre natural.

La implementación de esta solución varía en función de la naturaleza de la configuración del sistema en Azure. Debe adaptar las soluciones de copia de seguridad y restauración, alta disponibilidad y recuperación ante desastres en función de sus requisitos empresariales.

## <a name="backup-and-restore"></a>Copia de seguridad y restauración

La copia de seguridad y restauración es un proceso de creación de copias periódicas de datos y aplicaciones en una ubicación independiente para que se puedan restaurar o recuperar en un estado anterior si se pierden o dañan los datos originales o las aplicaciones. También es un componente esencial de cualquier estrategia empresarial de recuperación ante desastres. Estas copias de seguridad permiten la restauración de aplicaciones y bases de datos a un momento dado dentro del período de retención configurado.

Para desarrollar una estrategia completa de copia de seguridad y restauración para la plataforma de SAP BOBI, identifique los componentes que provocan el tiempo de inactividad del sistema o la interrupción de la aplicación. En una plataforma de SAP BOBI, la copia de seguridad de los componentes siguientes es fundamental para proteger la aplicación:

- Directorio de instalación de SAP BOBI (Managed Disks Premium)
- Almacén de archivos (Azure Premium Files o Azure NetApp Files para una instalación distribuida)
- Base de datos CMS y de auditoría (SQL Database, Azure Database for MySQL o una base de datos en Azure Virtual Machines)

En la siguiente sección se describe cómo implementar una estrategia de copia de seguridad y restauración para cada componente en una plataforma de SAP BOBI.

### <a name="backup-and-restore-for-an-sap-bobi-installation-directory"></a>Copia de seguridad y restauración para un directorio de instalación de SAP BOBI

En Azure, la manera más sencilla de realizar copias de seguridad de las máquinas virtuales y de todos los discos conectados es mediante [Azure Backup](../../../backup/backup-azure-vms-introduction.md). Proporciona una copia de seguridad independiente y aislada para impedir la destrucción accidental de los datos en las VM. Las copias de seguridad se almacenan en un almacén de Recovery Services con administración integrada de puntos de recuperación. La configuración y el escalado son sencillos. Las copias de seguridad están optimizadas y puede restaurarlas fácilmente cuando sea necesario.

Como parte del proceso de copia de seguridad, se toma una instantánea. Los datos se transfieren al almacén de Recovery Services sin afectar a las cargas de trabajo de producción. La instantánea proporciona un nivel diferente de coherencia, como se describe en [Coherencia de instantáneas](../../../backup/backup-azure-vms-introduction.md#snapshot-consistency). Backup también ofrece compatibilidad en paralelo con la copia de seguridad de los discos administrados mediante [Azure Disk Backup](../../../backup/disk-backup-overview.md), además de una solución de [copia de seguridad de VM de Azure](../../../backup/backup-azure-vms-introduction.md). Esto resulta útil cuando es necesario hacer copias de seguridad coherentes de máquinas virtuales una vez al día, y copias de seguridad más frecuentes de los discos del SO, o de un disco de datos específico, que sean coherentes con los bloqueos. Para obtener más información, consulte [Acerca de la copia de seguridad de VM de Azure](../../../backup/backup-azure-vms-introduction.md), [Azure Disk Backup](../../../backup/disk-backup-overview.md) y [Preguntas más frecuentes sobre la copia de seguridad de VM de Azure](../../../backup/backup-azure-vm-backup-faq.yml).

### <a name="backup-and-restore-for-filestore"></a>Copia de seguridad y restauración del almacén de archivos

En función de la implementación, el almacén de archivos de la plataforma de SAP BOBI puede estar en Azure NetApp Files o Azure Files. Elija entre las siguientes opciones de copia de seguridad y restauración en función del almacenamiento que use para el almacén de archivos:

* **Azure NetApp Files**: para Azure NetApp Files, puede crear instantáneas a petición y programar la creación automática de instantáneas mediante el uso de directivas de instantáneas. Las copias de instantáneas proporcionan una copia de un momento dado del volumen de Azure NetApp Files. Para obtener más información, consulte [Administración de instantáneas mediante Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-manage-snapshots.md).
* **Azure Files**: la copia de seguridad de Azure Files se integra con una instancia nativa de [Backup](../../../backup/backup-overview.md), que centraliza la función de copia de seguridad y restauración junto con la copia de seguridad de las máquinas virtuales y simplifica el trabajo de las operaciones. Para más información, vea [Copia de seguridad de recursos compartidos de archivos de Azure](../../../backup/azure-file-share-backup-overview.md) y [Preguntas acerca de la copia de seguridad de archivos de Azure](../../../backup/backup-azure-files-faq.yml).

Si ha creado un servidor NFS independiente, asegúrese de implementar la estrategia de copia de seguridad y restauración para él.

### <a name="backup-and-restore-for-the-cms-and-audit-database"></a>Copia de seguridad y restauración de la base de datos CMS y de auditoría

En el caso de la plataforma de SAP BOBI que se ejecuta en máquinas virtuales Windows, las bases de datos CMS y de auditoría se pueden ejecutar en cualquiera de las bases de datos admitidas, como se describe en la [matriz de compatibilidad](businessobjects-deployment-guide.md#support-matrix) de la guía de implementación y planeamiento de la plataforma de SAP BOBI en Azure. Por lo tanto, es importante que adopte la estrategia de copia de seguridad y restauración en función de la base de datos usada para el almacenamiento de datos CMS y de auditoría.

* [SQL Database](/sql/relational-databases/backup-restore/transaction-log-backups-sql-server?preserve-view=true&view=sql-server-ver15) emplea tecnología de SQL Server para crear **copias de seguridad completas** cada semana, [copias de seguridad diferenciales](/sql/relational-databases/backup-restore/full-database-backups-sql-server?preserve-view=true&view=sql-server-ver15) cada 12 a 24 horas y copias de seguridad del [registro de transacciones](/sql/relational-databases/backup-restore/differential-backups-sql-server?preserve-view=true&view=sql-server-ver15) cada 5 a 10 minutos. La frecuencia de las copias de seguridad del registro de transacciones se basa en el tamaño de proceso y en la cantidad de actividad de la base de datos.
 
   Los usuarios pueden elegir una opción para configurar la redundancia del almacenamiento de copia de seguridad entre los blobs LRS, ZRS o GRS. Los mecanismos de redundancia de almacenamiento almacenan varias copias de los datos para protegerlos de eventos planeados y no planeados, que incluyen errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red, y desastres naturales masivos. De forma predeterminada, SQL Database almacena la copia de seguridad en [blobs GRS](../../../storage/common/storage-redundancy.md) que se replican en una [región emparejada](../../../best-practices-availability-paired-regions.md). Se puede cambiar en función del requisito empresarial a blobs LRS o ZRS. Para obtener información más actualizada sobre la programación, la retención y el consumo de almacenamiento de copias de seguridad de SQL Database, consulte [Copias de seguridad automatizadas: Azure SQL Database y Azure SQL Managed Instance](../../../azure-sql/database/automated-backups-overview.md).

* **Azure Database for MySQL** crea automáticamente copias de seguridad del servidor y las almacena en LRS o GRS configurado por el usuario. Azure Database for MySQL hace copias de seguridad de los archivos de datos y del registro de transacciones. En función del tamaño de almacenamiento máximo admitido, se realizan copias de seguridad completas y diferenciales (servidores de almacenamiento de hasta 4 TB) o copias de seguridad de instantáneas (servidores de almacenamiento de hasta 16 TB). Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. De manera predeterminada, el período de retención de la copia de seguridad es de 7 días, pero se puede [configurar opcionalmente](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. Estos archivos de copia de seguridad no están expuestos al usuario y no se pueden exportar. Estas copias de seguridad solo se pueden usar en operaciones de restauración de Azure Database for MySQL. Puede usar [mysqldump](../../../mysql/concepts-migrate-dump-restore.md) para copiar una base de datos. Para más información, consulte [Copia de seguridad y restauración en Azure Database for MySQL](../../../mysql/concepts-backup.md).

* **En el caso de una base de datos instalada en una máquina virtual de Azure**, puede usar las herramientas de copia de seguridad estándar o [Backup](../../../backup/sap-hana-db-about.md) para bases de datos compatibles. Además, si los servicios y las herramientas de Azure no cumplen sus requisitos, puede usar herramientas de copia de seguridad compatibles de terceros que proporcionen un agente para la copia de seguridad y recuperación de todos los componentes de la plataforma de SAP BOBI.

## <a name="high-availability"></a>Alta disponibilidad

Alta disponibilidad se refiere a un conjunto de tecnologías que pueden minimizar las interrupciones de TI ya que proporcionan continuidad empresarial de las aplicaciones o los servicios mediante componentes redundantes, con tolerancia a errores o protegidos mediante conmutación por error dentro del mismo centro de datos. En nuestro caso, el centro de datos se encuentra en una región de Azure. En el artículo [Escenarios y arquitectura de alta disponibilidad para SAP](sap-high-availability-architecture-scenarios.md) se proporciona información sobre las diferentes técnicas de alta disponibilidad y las recomendaciones ofrecidas en Azure para aplicaciones SAP, que complementan las instrucciones de esta sección.

En función del resultado del dimensionamiento de la plataforma de SAP BOBI, debe diseñar el panorama y determinar la distribución de los componentes de BI en máquinas virtuales de Azure y subredes. El nivel de redundancia en la arquitectura distribuida depende del objetivo de tiempo de recuperación (RTO) y del objetivo de punto de recuperación (RPO) necesarios para la empresa. La plataforma de SAP BOBI incluye diferentes niveles y los componentes de cada nivel se deben diseñar para lograr redundancia. A continuación, si se produce un error en un componente, no habrá ninguna interrupción en la aplicación SAP BOBI. Por ejemplo:

- Servidores de aplicaciones redundantes, como servidores de aplicaciones de BI y servidores web
- Componentes únicos, como la base de datos CMS, el almacén de archivos y Load Balancer

En la siguiente sección se describe cómo lograr una alta disponibilidad en cada componente de la plataforma de SAP BOBI.

### <a name="high-availability-for-application-servers"></a>Alta disponibilidad en los servidores de aplicaciones

Los servidores de aplicaciones web y de BI no necesitan una solución de alta disponibilidad específica, independientemente de si se instalan por separado o de manera conjunta. Puede lograr una alta disponibilidad mediante redundancia, es decir, configurando varias instancias de BI y servidores web en varias máquinas virtuales de Azure. Puede implementar las máquinas virtuales en [conjuntos de disponibilidad](sap-high-availability-architecture-scenarios.md#multiple-instances-of-virtual-machines-in-the-same-availability-set) o [zonas de disponibilidad](sap-high-availability-architecture-scenarios.md#azure-availability-zones) en función del RTO empresarial requerido. Para la implementación en zonas de disponibilidad, asegúrese de que todos los demás componentes de la plataforma de SAP BOBI también estén diseñados para tener redundancia de zona.

Actualmente, no todas las regiones de Azure ofrecen zonas de disponibilidad, por lo que debe adoptar la estrategia de implementación en función de su región. Las regiones de Azure que ofrecen zonas aparecen en [zonas de disponibilidad de Azure](../../../availability-zones/az-overview.md).

> [!Important]
> Los conceptos de zonas de disponibilidad de Azure y conjuntos de disponibilidad de Azure son mutuamente excluyentes. Esto significa que se pueden implementar dos o varias máquinas virtuales en una zona de disponibilidad específica o en un conjunto de disponibilidad de Azure, pero no en ambos.

### <a name="high-availability-for-the-cms-database"></a>Alta disponibilidad de la base de datos CMS

Si usa una base de datos de Azure como solución para las bases de datos CMS y de auditoría, el marco de alta disponibilidad con redundancia local se proporciona de manera predeterminada. Seleccione las capacidades de alta disponibilidad, redundancia y resistencia inherentes a la región y el servicio, sin necesidad de configurar ningún componente adicional. Si la estrategia de implementación de la plataforma de SAP BOBI se encuentra en la zona de disponibilidad, asegúrese de lograr la redundancia de zona para las bases de datos CMS y de auditoría. Para obtener más información sobre la alta disponibilidad para las ofertas de bases de datos admitidas en Azure, vea [Alta disponibilidad para Azure SQL Database](../../../azure-sql/database/high-availability-sla.md) y [Alta disponibilidad en Azure Database for MySQL](../../../mysql/concepts-high-availability.md). 

En el caso de otra implementación del sistema de administración de bases de datos (DBMS) para una base de datos CMS, vea [Guías de implementación de DBMS para la carga de trabajo de SAP](dbms_guide_general.md) para obtener información sobre una implementación de DBMS diferente y su enfoque para lograr alta disponibilidad.

### <a name="high-availability-for-filestore"></a>Alta disponibilidad para el almacén de archivos

El almacén de archivos hace referencia a los directorios del disco donde se almacena contenido como informes, universos y conexiones. Se comparte en todos los servidores de aplicaciones de ese sistema. Por lo tanto, debe asegurarse de que tenga alta disponibilidad, junto con otros componentes de la plataforma de SAP BOBI.

Para la plataforma de SAP BOBI que se ejecuta en Windows, puede elegir [Azure Premium Files](../../../storage/files/storage-files-introduction.md) o [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) para el almacén de archivos, que, por naturaleza, está diseñado para tener alta disponibilidad y alta durabilidad. Azure Premium Files admite ZRS, que puede ser útil para la implementación entre zonas de la plataforma de SAP BOBI. Para más información, consulte la sección [Redundancia](../../../storage/files/storage-files-planning.md#redundancy) para Azure Files.

Dado que el servicio de recursos compartidos de archivos no está disponible en todas las regiones, asegúrese de consultar la lista de [productos disponibles por región](https://azure.microsoft.com/en-us/global-infrastructure/services/) para buscar la información actualizada. Si el servicio no está disponible en su región, puede crear un servidor NFS desde el que pueda compartir el sistema de archivos con la aplicación SAP BOBI. Sin embargo, también debe tener en cuenta su alta disponibilidad.

### <a name="high-availability-for-the-load-balancer"></a>Alta disponibilidad para el equilibrador de carga

Para distribuir el tráfico a través de un servidor web, puede usar Load Balancer o Application Gateway. La redundancia para cualquiera de los equilibradores de carga se puede lograr en función de la SKU que elija para la implementación:

* **Load Balancer**: la redundancia se puede lograr mediante la configuración del front-end de Standard Load Balancer como con redundancia de zona. Para más información, consulte [Standard Load Balancer y Availability Zones](../../../load-balancer/load-balancer-standard-availability-zones.md).
* **Application Gateway**: se puede lograr alta disponibilidad en función del tipo de nivel seleccionado durante la implementación:
   * La SKU v1 admite escenarios de alta disponibilidad cuando se han implementado dos o más instancias. Azure distribuye estas instancias entre dominios de actualización y de errores para asegurarse de que las instancias no produzcan un error todas al mismo tiempo. Con esta SKU, se puede lograr redundancia dentro de la zona.
   * La SKU v2 garantiza automáticamente que las nuevas instancias se distribuyan entre dominios de error y dominios de actualización. Si elige la redundancia de zona, las instancias más recientes también se distribuyen entre las zonas de disponibilidad para ofrecer resistencia ante errores de zona. Para más información, consulte [Escalabilidad automática y Application Gateway con redundancia de zona v2](../../../application-gateway/application-gateway-autoscaling-zone-redundant.md).

### <a name="reference-high-availability-architecture-for-the-sap-businessobjects-bi-platform"></a>Arquitectura de alta disponibilidad de referencia para la plataforma de SAP BusinessObjects BI

En la arquitectura de referencia siguiente se describe la configuración de la plataforma de SAP BOBI en las zonas de disponibilidad que se ejecutan en servidores Windows. La arquitectura muestra el uso de diferentes servicios de Azure como Application Gateway, Azure Premium Files (almacén de archivos) y SQL Database (bases de datos CMS y de auditoría). La plataforma de SAP BOBI ofrece redundancia de zona integrada, lo que reduce la complejidad de administrar diferentes soluciones de alta disponibilidad.

En la figura siguiente, la carga del tráfico entrante (HTTPS - TCP/443) se equilibra mediante una SKU de Application Gateway v2, que abarca varias zonas de disponibilidad. Application Gateway distribuye la solicitud de usuario entre los servidores web, que se distribuyen en varias zonas de disponibilidad. El servidor web reenvía la solicitud a las instancias del servidor de administración y procesamiento que se han implementado en máquinas virtuales independientes entre las zonas de disponibilidad. Azure Premium Files con ZRS se conecta a través de un vínculo privado a las máquinas virtuales de capa de almacenamiento y administración para acceder al contenido, como informes, universo y conexiones. La aplicación accede a las bases de datos CMS y de auditoría que se ejecutan en una instancia con redundancia de zona de SQL Database, que replica las bases de datos en varias ubicaciones físicas dentro de una región de Azure.

![Diagrama que muestra la arquitectura de alta disponibilidad para una plataforma de SAP BOBI en Windows.](media/businessobjects-deployment-guide/businessobjects-deployment-windows-high-availability-availability-zone.png)

En la arquitectura anterior se proporciona información sobre cómo se puede realizar la implementación de SAP BOBI en Azure. Sin embargo, no cubre todas las posibles opciones de configuración de una plataforma de SAP BOBI en Azure. Puede adaptar su implementación en función de las necesidades empresariales, eligiendo diferentes productos o servicios para los componentes, como Load Balancer, el servidor de repositorio de archivos y DBMS.

En el caso de que las zonas de disponibilidad no estén disponibles en la región seleccionada, puede implementar máquinas virtuales de Azure en conjuntos de disponibilidad. Azure se asegura de que las VM colocadas en un conjunto de disponibilidad se ejecuten en varios servidores físicos, bastidores de proceso, unidades de almacenamiento y conmutadores de red. Si se produce un error de hardware o software, solo un subconjunto de las VM se ve afectado y la solución general permanece operativa.

## <a name="disaster-recovery"></a>Recuperación ante desastres

En esta sección se explica la estrategia para proporcionar protección de recuperación ante desastres para una plataforma de SAP BOBI. Complementa el documento [Recuperación ante desastres para SAP](../../../site-recovery/site-recovery-sap.md), que representa los recursos principales para el enfoque general de DR de SAP. Para la plataforma de SAP BOBI, consulte la nota de SAP [2056228](https://launchpad.support.sap.com/#/notes/2056228), en la que se describen los métodos siguientes para implementar un entorno de DR de forma segura:

 * Uso total o selectivo de la administración del ciclo de vida o de la federación para promover o distribuir el contenido del sistema principal.
 * Copia periódica sobre el contenido de CMS y FRS.

En esta guía, hablaremos sobre la segunda opción para implementar un entorno de DR. Esta guía no abarca una lista exhaustiva de todas las opciones de configuración posibles para DR. Se cubre una solución que incluye servicios nativos de Azure en combinación con la configuración de la plataforma de SAP BOBI.

>[!Important]
>La disponibilidad de cada componente en la plataforma de SAP BOBI debe tenerse en cuenta en la región secundaria. Toda la estrategia de DR debe probarse exhaustivamente.

### <a name="reference-dr-architecture-for-an-sap-businessobjects-bi-platform"></a>Arquitectura de DR de referencia para una plataforma de SAP BusinessObjects BI

Esta arquitectura de referencia ejecuta una implementación de varias instancias de la plataforma de SAP BOBI con servidores de aplicaciones redundantes. Para DR, debe conmutar por error todos los componentes de la plataforma de SAP BOBI a una región secundaria. En la figura siguiente, Azure Premium Files se usa como almacén de archivos, SQL Database como repositorio de CMS y de auditoría, y Application Gateway para equilibrar la carga del tráfico. La estrategia para lograr la protección de DR para cada componente es diferente, lo que se describe en la siguiente sección.

![Diagrama que muestra la recuperación ante desastres de la plataforma de SAP BusinessObjects BI para Windows.](media\businessobjects-deployment-guide\businessobjects-deployment-windows-disaster-recovery.png)

### <a name="load-balancer"></a>Load Balancer

Load Balancer se usa para distribuir el tráfico entre servidores de aplicaciones web de la plataforma de SAP BOBI. En Azure, puede usar Load Balancer o Application Gateway para equilibrar la carga del tráfico entre los servidores web. Para lograr la recuperación ante desastres de los servicios del equilibrador de carga, debe implementar otro equilibrador de carga o puerta de enlace de aplicación en una región secundaria. Para mantener la misma dirección URL después de la conmutación por error de DR, cambie la entrada en DNS de modo que apunte al servicio de equilibrio de carga que se ejecuta en la región secundaria.

### <a name="virtual-machines-that-run-web-and-bi-application-servers"></a>Máquinas virtuales que ejecutan servidores de aplicaciones web y de BI

[Azure Site Recovery](../../../site-recovery/site-recovery-overview.md) se puede usar para replicar máquinas virtuales que ejecutan servidores de aplicaciones web y de BI en la región secundaria. Replica en la región secundaria los servidores y todos los discos administrados conectados para que, cuando se produzcan desastres o interrupciones, pueda conmutar por error fácilmente a su entorno replicado y seguir trabajando. Para comenzar a replicar todas las máquinas virtuales de la aplicación SAP en el centro de datos de DR de Azure, siga las instrucciones de [Replicación de una máquina virtual en Azure](../../../site-recovery/azure-to-azure-tutorial-enable-replication.md).

### <a name="filestore"></a>Almacén de archivos

El almacén de archivos es un directorio de discos donde se almacenan los archivos reales, como los informes y los documentos de BI. Es importante que todos los archivos del almacén de archivos estén sincronizados con la región de DR. En función del tipo de servicio de recurso compartido de archivos que use para la plataforma de SAP BOBI que se ejecuta en Windows, es necesario adoptar una estrategia de DR para sincronizar el contenido. Por ejemplo:

- **Azure Premium Files** solo admite LRS y ZRS. Para la estrategia de recuperación ante desastres de los archivos de Azure Premium, use [AzCopy](../../../storage/common/storage-use-azcopy-v10.md) o [Azure PowerShell](/powershell/module/az.storage/?preserve-view=true&view=azps-5.8.0) para copiar los archivos en otra cuenta de almacenamiento en una región distinta. Para más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../../../storage/common/storage-disaster-recovery-guidance.md).
- **Azure NetApp Files** proporciona volúmenes NFS y SMB, por lo que se puede usar cualquier herramienta de copia basada en archivos para replicar datos entre regiones de Azure. Para más información sobre cómo copiar el volumen de Azure NetApp Files en otra región, consulte [Preguntas más frecuentes acerca de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-faqs.md#how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region).

  Puede usar la replicación entre regiones de Azure NetApp Files, actualmente en [versión preliminar](https://azure.microsoft.com/en-us/blog/azure-netapp-files-cross-region-replication-and-new-enhancements-in-preview/), que usa la tecnología de NetApp SnapMirror. Con esta tecnología, solo los bloques modificados se envían a través de la red en un formato comprimido y eficaz. Esta tecnología propietaria reduce la cantidad de datos necesarios para replicar entre regiones, lo que ahorra costos de transferencia de datos. También acorta el tiempo de replicación, por lo que puede lograr un RPO menor. Para obtener más información, consulte [Requisitos y consideraciones del uso de la replicación entre regiones](../../../azure-netapp-files/cross-region-replication-requirements-considerations.md).

### <a name="cms-database"></a>Base de datos CMS

Las bases de datos CMS y de auditoría de la región de DR deben ser una copia de las bases de datos que se ejecutan en la región primaria. En función del tipo de base de datos, es importante copiar la base de datos en la región de DR según el RTO y el RPO necesarios para la empresa. En esta sección se describen las distintas opciones disponibles para cada solución de base de datos en Azure, compatible con la aplicación de SAP BOBI que se ejecuta en Windows.

#### <a name="azure-sql-database"></a>Azure SQL Database

Para una estrategia de DR de [SQL Database](../../../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md), hay dos opciones disponibles para copiar la base de datos en la región secundaria. Ambas opciones de recuperación ofrecen diferentes niveles de RTO y RPO. Para más información sobre el RTO y el RPO para cada opción de recuperación, consulte [Recuperación de una base datos en un servidor existente](../../../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md#recover-a-database-to-the-existing-server).

Opción 1: [Restauración de una copia de seguridad de base de datos con redundancia geográfica](../../../azure-sql/database/recovery-using-backups.md#geo-restore)

   De forma predeterminada, SQL Database almacena los datos en [blobs GRS](../../../storage/common/storage-redundancy.md) que se replican en una [región emparejada](../../../best-practices-availability-paired-regions.md). En el caso de una base de datos SQL, la redundancia del almacenamiento de copia de seguridad se puede configurar al crear la base de datos CMS o de auditoría, o se puede actualizar para una base de datos existente. Los cambios realizados en una base de datos existente solo se aplican a las copias de seguridad futuras. Una base de datos situada en cualquier base de datos de SQL de cualquier región de Azure se puede restaurar desde las copias de seguridad con replicación geográfica más recientes. La restauración geográfica usa una copia de seguridad con replicación geográfica como su origen. Hay un retraso entre el momento en que se hace una copia de seguridad y el momento en que se replica geográficamente en un blob de Azure de otra región. Como resultado, la base de datos restaurada puede encontrarse hasta una hora por detrás de la base de datos original.

   >[!Important]
   >La restauración geográfica está disponible para bases de datos de SQL configuradas con [almacenamiento de copia de seguridad](../../../azure-sql/database/automated-backups-overview.md#backup-storage-redundancy) con redundancia geográfica.

Opción 2: [Replicación geográfica](../../../azure-sql/database/active-geo-replication-overview.md) o un [grupo de conmutación por error automática](../../../azure-sql/database/auto-failover-group-overview.md)

   La replicación geográfica es una característica de SQL Database que permite crear bases de datos secundarias legibles de bases de datos individuales en un servidor en la misma región o en otra distinta. Si la replicación geográfica está habilitada para las bases de datos CMS y de auditoría, la aplicación puede iniciar la conmutación por error en una base de datos secundaria de otra región de Azure. La replicación geográfica está habilitada para bases de datos individuales, pero para habilitar la conmutación por error transparente y coordinada de varias bases de datos (CMS y de auditoría) para la aplicación de SAP BOBI, es aconsejable usar un grupo de conmutación por error automática. Proporciona la semántica de grupo, además de la replicación geográfica activa, por lo que todo el servidor SQL (todas las bases de datos) se replica en otra región, en lugar de las bases de datos individuales. Consulte la tabla de funcionalidades que [compara la replicación geográfica con los grupos de conmutación por error](../../../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md#compare-geo-replication-with-failover-groups).

   Los grupos de conmutación por error automática proporcionan puntos de conexión de agentes de escucha de lectura y escritura y de solo lectura que no se modifican durante las conmutaciones por error. El punto de conexión de lectura y escritura se puede mantener como cliente de escucha en la entrada de conexión ODBC para las bases de datos CMS y de auditoría. Por lo tanto, ya sea que use la activación de conmutación por error automática o manual, la conmutación por error transforma todas las bases de datos secundarias del grupo en primarias. Después de que la conmutación por error de una base de datos finaliza, el registro de DNS se actualiza automáticamente para redirigir los puntos de conexión a la nueva región. La aplicación se conecta automáticamente a la base de datos de CMS, ya que el punto de conexión de lectura y escritura se mantiene como cliente de escucha en la conexión ODBC.

   En la imagen siguiente, el grupo de conmutación por error automática para un servidor de SQL (azussqlbodb) que se ejecuta en la región Este de EE. UU. 2 se replica en la región secundaria Este de EE. UU. (sitio de DR). El punto de conexión del cliente de escucha de lectura y escritura se mantiene como cliente de escucha en la conexión ODBC para el servidor de aplicaciones de BI que se ejecuta en Windows. Después de la conmutación por error, el punto de conexión seguirá siendo el mismo. No se requiere ninguna intervención manual para conectar la aplicación de BI a la base de datos de Azure SQL en la región secundaria.

   ![Captura de pantalla que muestra grupos de conmutación por error automática de SQL Database.](media\businessobjects-deployment-guide\businessobjects-deployment-windows-sql-failover-group.png)

   Esta opción proporciona un RTO y RPO inferiores a la opción 1. Para más información sobre esta opción, consulte [Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada](../../../azure-sql/database/auto-failover-group-overview.md).

#### <a name="azure-database-for-mysql"></a>Azure Database for MySQL

Azure Database for MySQL proporciona opciones para recuperar una base de datos si se produce algún desastre. Elija la opción adecuada que se adapte a su empresa:

* Habilite las réplicas de lectura entre regiones para mejorar la planificación de la continuidad empresarial y DR. Puede crear hasta cinco réplicas desde el servidor de origen. Las réplicas de lectura se actualizan de forma asincrónica mediante la tecnología de replicación de registros binarios de Azure Database for MySQL. Las réplicas son nuevos servidores que se administran de forma similar a los servidores de Azure Database for MySQL normales. Para más sobre las réplicas de lectura, las regiones disponibles, las restricciones y la conmutación por error, vea [Réplicas de lectura en Azure DB for MySQL](../../../mysql/concepts-read-replicas.md).

* Use la característica de restauración geográfica de Azure Database for MySQL que permite restaurar el servidor mediante copias de seguridad con redundancia geográfica. Se puede acceder a estas copias de seguridad incluso cuando la región en la que se hospeda el servidor está sin conexión. Puede realizar la restauración a partir de estas copias de seguridad en cualquier otra región y volver a poner en línea el servidor.

  > [!Important]
  > La restauración geográfica solo es posible si se ha aprovisionado el servidor con almacenamiento de copia de seguridad con redundancia geográfica. No se admite el cambio de las opciones de redundancia de copia de seguridad después de la creación del servidor. Para más información, vea [Redundancia de copia de seguridad](../../../mysql/concepts-backup.md#backup-redundancy-options).

En la tabla siguiente se enumeran las recomendaciones para DR en cada nivel usado en este ejemplo.

| Niveles de la plataforma de SAP BOBI                          | Recomendación                                               |
| ------------------------------------------------ | ------------------------------------------------------------ |
| Azure Application Gateway o Azure Load Balancer | Configuración paralela de Application Gateway en una región secundaria    |
| Servidores de aplicaciones web                          | Replicación con Azure Site Recovery                             |
| Servidores de aplicaciones de BI                           | Replicación mediante Site Recovery                             |
| Azure Premium Files                              | AzCopy *o* Azure PowerShell                               |
| Azure NetApp Files                               | Herramienta de copia basada en archivos para replicar datos en una región secundaria *o* replicación entre regiones de Azure NetApp Files en versión preliminar |
| Azure SQL Database                               | Grupos de replicación geográfica o conmutación por error automática *o* restauración geográfica     |
| Azure Database for MySQL                         | Réplicas de lectura entre regiones *o* restauración de la copia de seguridad de copias de seguridad con redundancia geográfica |

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la recuperación ante desastres para la implementación de una aplicación de SAP de niveles múltiples](../../../site-recovery/site-recovery-sap.md)
- [Planeamiento e implementación de Azure Virtual Machines para SAP](planning-guide.md)
- [Implementación de Azure Virtual Machines para SAP](deployment-guide.md)
- [Implementación de DBMS de Azure Virtual Machines para SAP](./dbms_guide_general.md)
