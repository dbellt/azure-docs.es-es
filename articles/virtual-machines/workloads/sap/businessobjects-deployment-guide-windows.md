---
title: Implementación de la plataforma de inteligencia empresarial SAP BusinessObjects en Azure para Windows | Microsoft Docs
description: Implementación y configuración de la plataforma de inteligencia empresarial SAP BusinessObjects en Azure para Windows
services: virtual-machines-windows,virtual-network,storage,azure-netapp-files,azure-files,azure-sql
documentationcenter: saponazure
author: dennispadia
manager: juergent
editor: ''
tags: azure-resource-manager
keywords: ''
ms.service: virtual-machines-windows
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 04/08/2021
ms.author: depadia
ms.openlocfilehash: a13b62da91e3ce6f18ba4dcda2b3b0f22c815026
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108144392"
---
# <a name="sap-businessobjects-bi-platform-deployment-guide-for-windows-on-azure"></a>Guía de implementación de la plataforma de inteligencia empresarial SAP BusinessObjects para Windows en Azure

En este artículo se describe la estrategia para implementar la plataforma de inteligencia empresarial SAP BusinessObjects en Azure para Windows. En este ejemplo, se configuran dos máquinas virtuales con discos administrados SSD prémium como directorio de instalación. La base de datos de Azure SQL (oferta PaaS) se usa para la base de datos del servicio de administración central (CMS) y la base de datos de auditoría, y Azure Premium Files (protocolo SMB) se usa como almacén de archivos que comparten ambas máquinas virtuales. La aplicación web de Java de Tomcat predeterminada y la aplicación de plataforma de inteligencia empresarial se instalan juntas en ambas máquinas virtuales. Para equilibrar la carga de las solicitudes de los usuarios, se usa Azure Application Gateway, que tiene capacidades de descarga nativas de TLS/SSL.

Este tipo de arquitectura es eficaz para entornos pequeños de implementación o que no sean de producción. Para una implementación de producción o a gran escala, debería separar los hosts para la aplicación web, y puede tener varios hosts de aplicaciones de BOBI que permitan al servidor procesar más información.

![Implementación de SAP BOBI en Azure para Windows](media/businessobjects-deployment-guide/businessobjects-deployment-windows.png)

En este ejemplo, se usa la versión del producto y el diseño del sistema de archivos siguientes:

- Plataforma SAP BusinessObjects 4.3 SP01, revisión 1
- Windows Server 2019
- Azure SQL Database (versión: 12.0.2000.8)
- Controlador ODBC de Microsoft: msodbcsql.msi (versión: 13.1)

| Sistema de archivos        | Descripción                                                                                                               | Tamaño (GB)             | Acceso necesario  | Storage                    |
|--------------------|---------------------------------------------------------------------------------------------------------------------------|-----------------------|---------------|----------------------------|
| F:          | Sistema de archivos para la instalación de la instancia de SAP BOBI, aplicación web de Tomcat predeterminada y controladores de base de datos (si es necesario). | Directrices de ajuste de tamaño de SAP | Privilegios administrativos locales | Managed Disks Premium: SSD |
| \\\azusbobi.file.core.windows.net\frsinput  | El directorio de montaje es para los archivos compartidos en todos los hosts de BOBI que se usarán como directorio de almacén de archivos de entrada. | Necesidad empresarial         | Privilegios administrativos locales | Azure NetApp Files         |
| \\\azusbobi.file.core.windows.net\frsoutput | El directorio de montaje es para los archivos compartidos en todos los hosts de BOBI que se usarán como directorio del almacén de archivos de salida. | Necesidad empresarial         | Privilegios administrativos locales | Azure NetApp Files         |

## <a name="deploy-windows-virtual-machine-via-azure-portal"></a>Implementación de máquinas virtuales Windows mediante Azure Portal

En esta sección, crearemos dos máquinas virtuales (VM) con la imagen del sistema operativo (SO) Windows para la plataforma SAP BOBI. A continuación, se indican los pasos de alto nivel para crear máquinas virtuales:

1. Cree un [grupo de recursos](../../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups).

2. Cree una [red virtual](../../../virtual-network/quick-create-portal.md#create-a-virtual-network).

   - No use una única subred para todos los servicios de Azure en la implementación de la plataforma de inteligencia empresarial SAP. En función de la arquitectura de la plataforma de inteligencia empresarial de SAP, puede que tenga que crear varias subredes. En esta implementación, crearemos dos subredes: subred de aplicación de inteligencia empresarial y subred de Application Gateway.
   - Siga la nota de SAP [2276646](https://launchpad.support.sap.com/#/notes/2276646) para identificar los puertos para la comunicación de la plataforma de inteligencia empresarial SAP BusinessObjects entre distintos componentes.
   - Azure SQL Database se comunica a través del puerto 1433. Por lo tanto, se debe permitir el tráfico saliente a través del puerto 1433 desde los servidores de aplicación de SAP BOBI.
   - En Azure, Application Gateway tiene que estar en una subred independiente. Consulte el artículo [Azure Application Gateway](../../../application-gateway/configuration-overview.md) para obtener más información.
   - En caso de que use Azure NetApp Files para el almacén de archivos en lugar de Azure Files, tiene que crear una subred independiente para Azure NetApp Files. Consulte el artículo de [Directrices para el planeamiento de red de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-network-topologies.md) para obtener más detalles.

3. Cree un conjunto de disponibilidad.

   - Para lograr redundancia para cada nivel en la implementación de varias instancias, coloque las máquinas virtuales de cada nivel en un conjunto de disponibilidad. Asegúrese de separar los conjuntos de disponibilidad para cada nivel en función de su arquitectura.

4. Cree la máquina virtual 1 **(azuswinboap1)** .

   - Puede usar una imagen personalizada o elegir una imagen de Azure Marketplace. Consulte [Implementación de una máquina virtual desde Azure Marketplace para SAP](deployment-guide.md) o [Implementación de una máquina virtual con una imagen personalizada para SAP](deployment-guide.md) en función de sus necesidades.

5. Cree la máquina virtual 2 **(azuswinboap2)** .
6. Agregue un disco SSD Premium. Se usará como directorio de instalación de SAP BOBI.

## <a name="provision-azure-premium-files"></a>Aprovisionamiento de Azure Premium Files

Antes de continuar con la configuración de Azure Files, familiarícese con la [documentación correspondiente](../../../storage/files/storage-files-introduction.md).

Azure Files ofrece recursos compartidos de archivos estándar que se hospedan en hardware basado en disco duro (HDD), y recursos compartidos de archivos prémium que se hospedan en hardware basado en disco de estado sólido (SSD). Para el almacén de archivos de SAP BusinessObjects, se recomienda usar Azure Premium Files.

Los recursos compartidos de Azure Premium Files están disponibles con redundancia local y de zona en un subconjunto de regiones. Para averiguar si los recursos compartidos de archivos prémium están disponibles actualmente en su región, consulte la página [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=storage) para Azure. Para obtener información sobre las regiones que admiten ZRS, consulte [Redundancia de Azure Storage](../../../storage/common/storage-redundancy.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="deploy-azure-files-storage-account-and-nfs-shares"></a>Implementación de la cuenta de almacenamiento de archivos de Azure y recursos compartidos de NFS

Los recursos compartidos de archivos de Azure se implementan en cuentas de almacenamiento, que son objetos de nivel superior que representan un grupo compartido de almacenamiento. Este grupo de almacenamiento se puede usar para implementar varios recursos compartidos de archivos. Azure admite varios tipos de cuentas de almacenamiento para diferentes escenarios de almacenamiento que puedan tener los clientes, pero para el almacén de archivos de SAP BusinessObjects debe crear la cuenta de almacenamiento **FileStorage**. Le permite implementar recursos compartidos de archivos de Azure en hardware prémium o basado en unidades de estado sólido (SSD).

> [!NOTE]
> Las cuentas FileStorage solo se pueden usar para almacenar recursos compartidos de archivos de Azure. No se puede implementar ningún otro recurso de almacenamiento (blobs, contenedores, colas, tablas, etc.) en una cuenta FileStorage.

Se accederá a la cuenta de almacenamiento a través de un [punto de conexión privado](../../../storage/files/storage-files-networking-endpoints.md), implementado en la misma red virtual que la plataforma SAP BOBI. Con esta configuración, el tráfico del sistema SAP nunca sobrepasa los límites de seguridad de la red virtual. A menudo, los sistemas SAP contienen datos confidenciales y críticos para la empresa, por lo que permanecer dentro de los límites de la red virtual es una consideración de seguridad importante para muchos clientes.

Si necesita acceder a la cuenta de almacenamiento desde otra red virtual, puede usar el [emparejamiento de redes virtuales de Azure](../../../virtual-network/virtual-network-peering-overview.md).

#### <a name="azure-files-storage-account"></a>Cuenta de almacenamiento de archivos de Azure

1. Para crear una cuenta de almacenamiento mediante Azure Portal, **Crear un recurso** > **Almacenamiento** > **Cuenta de almacenamiento**.

2. En la pestaña **Aspectos básicos**, complete todos los campos obligatorios para crear una cuenta de almacenamiento.

   a. Seleccione **Suscripción**, **Grupo de recursos** y **Región**.

   b. Escriba el **Nombre de la cuenta de almacenamiento** (por ejemplo, **azusbobi**). Este nombre debe ser único globalmente, pero, por lo demás, puede ser cualquier nombre que quiera.

   c. Seleccione **Premium** nivel de rendimiento y **FileStorage** como tipo de cuenta.

   d. En la etiqueta de replicación, elija el nivel de redundancia, seleccione **Almacenamiento con redundancia local (LRS)**.

   Para Premium FileStorage, el almacenamiento con redundancia local (LRS) y el almacenamiento con redundancia de zona (ZRS) son las únicas opciones disponibles. Por lo tanto, en función de la estrategia de implementación (conjunto de disponibilidad o zona de disponibilidad), elija el nivel de redundancia adecuado. Para obtener más información, consulte [Redundancia de Azure Storage](../../../storage/common/storage-redundancy.md).

   e. Seleccione Siguiente.

3. En la pestaña **Redes**, seleccione [Punto de conexión privado](../../../storage/files/storage-files-networking-endpoints.md) como método de conectividad. Consulte [Consideraciones de redes para Azure Files](../../../storage/files/storage-files-networking-overview.md) para obtener información detallada.

   a. Seleccione **Agregar** en la sección de puntos de conexión privados.

   b. Seleccione **Suscripción**, **Grupo de recursos** y **Ubicación**.

   c. Escriba el **Nombre** del punto de conexión privado (por ejemplo, azusbobi-pe).

   d. Seleccione **Archivo** en **Subrecurso de almacenamiento**.

   e. En la sección **Redes**, seleccione la **Red virtual** y la **Subred** en la que se ha implementado la aplicación de inteligencia empresarial de SAP BusinessObjects.

   f. Acepte el **valor predeterminado (sí)** para **Integrar con la zona DNS privada**.

   g. Seleccione la **Zona DNS privada** de la lista desplegable.

   h. Seleccione **Aceptar** para volver a la pestaña Redes en Crear cuenta de almacenamiento.

4. En la pestaña **Protección de datos**, configure la directiva de eliminación temporal para recursos compartidos de archivos de Azure en la cuenta de almacenamiento. De manera predeterminada, la funcionalidad de eliminación temporal está desactivada. Para más información sobre la eliminación temporal, consulte [Cómo evitar la eliminación accidental de recursos compartidos de archivos de Azure](../../../storage/files/storage-files-prevent-file-share-deletion.md).

5. En la pestaña **Avanzadas**, compruebe las distintas opciones de seguridad.

   El campo **Se requiere transferencia segura** indica si la cuenta de almacenamiento requiere cifrado en tránsito para la comunicación con la cuenta de almacenamiento. Si requiere compatibilidad con SMB 2.1, tiene que deshabilitar este campo. Para la plataforma de inteligencia empresarial SAP BusinessObjects, mantenga el **valor predeterminado (habilitado)** .

6. Continúe y cree la cuenta de almacenamiento.

Para obtener más información sobre cómo crear una cuenta de almacenamiento, consulte [Creación de una cuenta de almacenamiento de FileStorage](../../../storage/files/storage-how-to-create-file-share.md).

#### <a name="create-azure-file-shares"></a>Creación de recursos compartidos de archivos de Azure

El siguiente paso consiste en crear la instancia de Azure Files en la cuenta de almacenamiento. Azure Files usa un modelo aprovisionado para los recursos compartidos de archivos prémium. En un modelo de negocio aprovisionado, debe especificar de forma proactiva cuáles son los requisitos de almacenamiento del servicio Azure Files, en lugar de que se le aplique una factura basada en lo que usa. Para comprender más sobre este modelo, consulte [Modelo aprovisionado](../../../storage/files/understanding-billing.md#provisioned-model). En este ejemplo, se crean dos instancias de Azure Files: frsinput (256 GB) y frsoutput (256 GB) para el almacén de archivos de SAP BOBI.

Vaya a la cuenta de almacenamiento **azusbobi** > **Recursos compartidos de archivos**.

1. Seleccione **Nuevo recurso compartido de archivos**.
2. Escriba el **Nombre** del recurso compartido de archivos (por ejemplo, frsinput, frsouput).
3. Indique el tamaño de recurso compartido de archivos necesario en **Capacidad aprovisionada** (por ejemplo, 256 GB)
4. Elija **SMB** como **Protocolo**.
5. Seleccione **Crear**.

## <a name="configure-data-disk-on-windows-virtual-machine"></a>Configuración de un disco de datos en una máquina virtual Windows

En los pasos de esta sección se usan los siguientes prefijos:

**[A]** : El paso se aplica a todos los hosts.

### <a name="initialize-a-new-data-disk"></a>Inicio de un nuevo disco de datos

La aplicación de inteligencia empresarial SAP BusinessObjects requiere una partición en la que se puedan instalar sus archivos binarios. Puede instalar la aplicación SAP BOBI en la partición del sistema operativo (C:), pero asegúrese de tener suficiente espacio para la implementación y el sistema operativo. Se recomienda tener al menos 2 GB disponibles para archivos temporales y aplicaciones web. Teniendo todo esto en cuenta, se aconseja separar los archivos binarios de instalación de SAP BOBI en una partición independiente.

En este ejemplo, la aplicación SAP BOBI se instalará en una partición independiente (F:). Inicialice el disco SSD prémium que ha conectado durante el aprovisionamiento de las máquinas virtuales.

1. **[A]** En caso de que no haya ningún disco de datos conectado a las máquinas virtuales (azuswinboap1 y azuswinboap2), siga los pasos mencionados en [Agregar un disco de datos](../../windows/attach-managed-disk-portal.md#add-a-data-disk) para conectar un nuevo disco de datos administrado.
2. **[A]** Después de conectar el disco administrado a la máquina virtual, inicialice el disco siguiendo los pasos mencionados en el documento [Inicio de un nuevo disco de datos](../../windows/attach-managed-disk-portal.md#initialize-a-new-data-disk).

### <a name="mount-azure-premium-files"></a>Montaje de Azure Premium Files

Para usar Azure Files como almacén de archivos, tiene que montarlo, lo que significa asignarle una letra de unidad o una ruta de acceso a un punto de montaje.

1. **[A]** Para montar un recurso compartido de archivos de Azure, siga los pasos descritos en el documento [Montaje del recurso compartido de archivos de Azure](../../../storage/files/storage-how-to-use-files-windows.md#mount-the-azure-file-share).

Para montar un recurso compartido de archivos de Azure en Windows Server, asegúrese de que el puerto 445 esté abierto. el protocolo SMB requiere que esté abierto el puerto TCP 445; las conexiones producirán errores si el puerto 445 está bloqueado. Puede comprobar si el firewall está bloqueando el puerto 445 con el cmdlet `Test-NetConnection` mencionado en la guía de [solución de problemas](../../../storage/files/storage-troubleshoot-windows-file-connection-problems.md#cause-1-port-445-is-blocked).

## <a name="configure-cms-database---azure-sql"></a>Configuración de la base de datos CMS: Azure SQL

En esta sección se proporcionan detalles sobre cómo aprovisionar Azure SQL mediante Azure Portal. También se proporcionan instrucciones sobre cómo crear las bases de datos CMS y de auditoría para la plataforma SAP BOBI y una cuenta de usuario para acceder a la base de datos.

La guía solo es aplicable si usa Azure SQL (oferta de DBaaS en Azure). En el caso de otras bases de datos, consulte la documentación específica de SAP o de la base de datos para obtener instrucciones.

### <a name="create-sql-database-server"></a>Creación de un servidor de bases de datos SQL

La base de datos de Azure SQL ofrece diferentes opciones de implementación: base de datos única, grupo elástico y servidor de bases de datos. Para SAP BOBI, se necesitan dos bases de datos (CMS y auditoría). Por lo tanto, en lugar de crear dos bases de datos únicas, puede crear un servidor de bases de datos SQL que pueda administrar el grupo de bases de datos únicas y los grupos elásticos.  A continuación se indican los pasos para crear un servidor de base de datos SQL:

1. Vaya a la página [Seleccione una opción de implementación de SQL](https://portal.azure.com/#create/Microsoft.AzureSQL).
2. En **Bases de datos SQL**, cambie **Tipo de recurso** a **Servidor de bases de datos** y seleccione **Crear**.
3. En la pestaña **Aspectos básicos**, complete todos los campos obligatorios para **Crear un servidor de SQL Database**.

   a. Seleccione la **Suscripción** y el **Grupo de recursos** en **Detalles del proyecto**.

   b. Escriba el **Nombre del servidor** (por ejemplo, azussqlbodb). El nombre del servidor debe ser único globalmente, pero, por lo demás, puede ser cualquier nombre que quiera.

   c. Seleccione la **Ubicación**.

   d. Escriba el **Inicio de sesión del administrador del servidor** (por ejemplo, boadmin) y la **Contraseña**.

4. En la pestaña **Redes**, cambie **Permitir que los servicios y recursos de Azure accedan a este servidor** a **No** en **Reglas de firewall**.
5. En **Configuración adicional**, mantenga la configuración predeterminada.
6. Continúe y cree el **Servidor de SQL Database**.

En el paso siguiente, cree las bases de datos CMS y de auditoría en el servidor de bases de datos SQL (azussqlbodb.database.windows.net).

### <a name="create-the-cms-and-the-audit-database"></a>Creación de las bases de datos CMS y de auditoría

Después de aprovisionar el servidor de bases de datos SQL, vaya al recurso **azussqlbodb**, y luego siga los pasos a continuación para crear las bases de datos CMS y de auditoría.

1. En la página **Información general** de azussqlbodb, seleccione **Crear base de datos**.
2. En la pestaña **Aspectos básicos**, complete todos los campos obligatorios:

   a. Escriba el **Nombre de la base de datos** (por ejemplo, bocms o boaudit).

   b. En la opción **Proceso y almacenamiento**, seleccione **Configurar base de datos** y elija el modelo adecuado en función del resultado de tamaño. Consulte [Modelos de ajuste de amaño para la base de datos de Azure SQL](businessobjects-deployment-guide.md#sizing-models-for-azure-sql-database) para obtener información sobre las opciones.
3. En la pestaña **Redes**, seleccione [Punto de conexión privado](../../../private-link/tutorial-private-endpoint-sql-portal.md) como método de conectividad. El punto de conexión privado se usará para acceder a la base de datos de Azure SQL dentro de la red virtual configurada.

   a. Seleccione **Agregar punto de conexión privado**.

   b. Seleccione **Suscripción**, **Grupo de recursos** y **Ubicación**.

   c. Escriba el **Nombre** del punto de conexión privado (por ejemplo, azusbodb-pe).

   e. Seleccione **SqlServer** en **Subrecurso de destino**.

   f. En la sección **Redes**, seleccione la **Red virtual** y la **Subred** en la que se ha implementado la aplicación de inteligencia empresarial de SAP BusinessObjects.

   g. Acepte el **valor predeterminado (sí)** para **Integrar con la zona DNS privada**.

   h. Seleccione la **Zona DNS privada** de la lista desplegable.

   i. Seleccione **Aceptar** para volver a la pestaña Redes en Crear SQL Database.
4. En la pestaña **Configuración adicional**, cambie la **Intercalación** a **SQL_Latin1_General_CP850_BIN2**.
5. Continúe y cree la base de datos CMS.

De manera similar, puede crear una base de datos **Audit** (por ejemplo, boaudit).

### <a name="download-and-install-odbc-driver"></a>Descarga e instalación del controlador ODBC

Para que el servidor de aplicaciones de SAP BOBI acceda a la base de datos CMS o de auditoría, requiere cliente o controladores de base de datos. El controlador ODBC de Microsoft se usa para acceder a las bases de datos CMS y de auditoría que se ejecutan en la base de datos de Azure SQL. En esta sección se proporcionan instrucciones sobre cómo descargar y configurar el controlador ODBC en Windows. 

1. Consulte la sección **Compatibilidad del repositorio CMS + auditoría por SO** en [Matriz de disponibilidad de productos (PAM) para la plataforma de inteligencia empresarial SAP BusinessObjects](https://support.sap.com/pam) para averiguar los conectores de base de datos que son compatibles con la base de datos de Azure SQL.
2. Descargue la versión del controlador ODBC desde el [vínculo](/sql/connect/odbc/windows/release-notes-odbc-sql-server-windows?preserve-view=true&view=sql-server-ver15). En este ejemplo, vamos a descargar el controlador ODBC [13.1](/sql/connect/odbc/windows/release-notes-odbc-sql-server-windows?preserve-view=true&view=sql-server-ver15#131).
3. Instale el controlador ODBC en todos los servidores de inteligencia empresarial (azuswinboap1 y azuswinboap2).
4. Después de instalar el controlador en **azuswinboap1**, vaya a **Inicio** > **Herramientas administrativas de Windows** > **Orígenes de datos ODBC (64 bits)** .  
5. Vaya a la pestaña **DNS del sistema**. 
6. Seleccione **Agregar** para crear una conexión a la base de datos CMS.
7. Seleccione **Controlador ODBC 13 para SQL Server** y seleccione **Finalizar**.
8. Escriba la información de la base de datos CMS como se muestra a continuación y seleccione **Siguiente**.
   - **Nombre:** *nombre de la base de datos creada en la sección Creación de las bases de datos CMS y de auditoría* (por ejemplo, bocms o boaudit)
   - **Descripción:** *descripción del origen de datos* (por ejemplo, base de datos CMS o base de datos de auditoría)
   - **Servidor:** *nombre del servidor SQL creado en la sección Creación de un servidor de bases de datos SQL* (por ejemplo, azussqlbodb.database.windows.net).
9. Seleccione "**Con la autenticación de SQL Server, mediante un identificador de inicio de sesión y una contraseña escritos por el usuario**" para comprobar la autenticidad en SQL Server. Escriba la credencial de usuario que se creó en el momento de crear el servidor de bases de datos de Azure SQL (por ejemplo, boadmin) y seleccione **Siguiente**.
10. **Cambie la base de datos predeterminada** a **bocms** y mantenga todo lo demás como viene predeterminado. Seleccione **Next** (Siguiente).
11. Active **Usar cifrado de alta seguridad para los datos** y mantenga todo lo demás como viene predeterminado. Seleccione **Finalizar**.
12. Se ha creado el origen de datos en la base de datos CMS, y ahora puede seleccionar **Origen de datos de prueba** para validar la conexión a la base de datos CMS desde la aplicación de inteligencia empresarial. Debe completarse correctamente. Si se produce un error, [solucione](../../../azure-sql/database/troubleshoot-common-errors-issues.md) el problema de conectividad.

>[!Note]
>Azure SQL Database se comunica a través del puerto 1433. Por lo tanto, se debe permitir el tráfico saliente a través del puerto 1433 desde los servidores de aplicación de SAP BOBI.

Repita los mismos pasos anteriores para crear la conexión para la base de datos Audit en el servidor azuswinboap1. De manera similar, instale y configure ambos orígenes de datos ODBC (bocms y boaudit) en todos los servidores de aplicaciones de inteligencia empresarial (azuswinboap2). 

## <a name="server-preparation"></a>Preparación del servidor

Siga la guía más reciente de SAP para preparar los servidores para la instalación de la plataforma de inteligencia empresarial. Para obtener la información más actualizada, consulte la sección Preparación de [Guía de instalación de la plataforma de inteligencia empresarial para Windows](https://help.sap.com/viewer/df8899896b364f6c880112f52e4d06c8/4.3.1/en-US/46b0d1a26e041014910aba7db0e91070.html).

## <a name="installation"></a>Instalación

Para instalar la plataforma de inteligencia empresarial en un host de Windows, inicie sesión con un usuario que tenga privilegios administrativos locales.

Vaya a los medios de la plataforma de inteligencia empresarial de SAP BusinessObjects y ejecute `setup.exe`.

Siga la [Guía de instalación de la plataforma de inteligencia empresarial de SAP para Windows](https://help.sap.com/viewer/df8899896b364f6c880112f52e4d06c8/4.3.1/en-US/46ae62456e041014910aba7db0e91070.html) específica para su versión. Hay algunos puntos a tener en cuenta durante la instalación de la plataforma SAP BOBI en Windows. 

- En la pantalla **Configure Destination Folder** (Configurar carpeta de destino), proporcione la carpeta de destino donde quiere instalar la plataforma de inteligencia empresarial, (por ejemplo, F:\SAP BusinessObjects\). 

- En la pantalla **Configure Product Registration** (Configurar el registro del producto), puede usar una clave de licencia temporal para las soluciones de SAP BusinessObjects de la nota de SAP [1288121](https://launchpad.support.sap.com/#/notes/1288121), o puede generar la clave de licencia en el marketplace de servicios de SAP.

- En la pantalla **Select Install Type** (Seleccionar tipo de instalación), seleccione la instalación **Full** (Completa) en el primer servidor (azuswinboap1). Para el otro servidor (azuswinboap2), seleccione la opción **Custom/Expand** (Personalizar/Expandir), que expandirá la instalación de BOBI existente.

- En **Seleccionar base de datos predeterminada o existente**, seleccione **Configurar una base de datos existente**, que le solicitará que seleccione base de datos CMS y de auditoría. Seleccione **Microsoft SQL Server using ODBC** (Microsoft SQL Server con ODBC) para el tipo de base de datos CMS y el tipo de base de datos de auditoría.

  También puede seleccionar Ninguna base de datos de auditoría si no quiere configurar la auditoría durante la instalación.

- Seleccione las opciones adecuadas en la **pantalla Seleccionar servidor de aplicaciones web de Java** según la arquitectura de SAP BOBI. En este ejemplo, se ha seleccionado la opción 1, que instala el servidor de Tomcat en la misma plataforma SAP BOBI.

- Escriba la información de la base de datos CMS en **Configure CMS Repository Database - SQL Server (ODBC)** (Configurar la base de datos del repositorio de CMS: SQL Server [ODBC]). Entrada de ejemplo para la información de la base de datos CMS para la instalación en Windows.
  
  ![Información de la base de datos CMS para Windows](media/businessobjects-deployment-guide/businessobjects-deployment-windows-sql-cms.png)

- (Opcional) Escriba la información de la base de datos Audit en **Configure Audit Repository Database - SQL Server (ODBC)** (Configurar la base de datos del repositorio de auditoría: SQL Server [ODBC]). Entrada de ejemplo para la información de la base de datos Audit para la instalación en Windows.
  
  ![Información de la base de datos Audit para Windows](media/businessobjects-deployment-guide/businessobjects-deployment-windows-sql-audit.png)

- Siga las instrucciones e introduzca las entradas necesarias para completar la instalación.

Para la implementación de varias instancias, ejecute la configuración de instalación en el segundo host (azuswinboap2). En la pantalla **Seleccionar tipo de instalación**, seleccione **Personalizar/Expandir**, opción que expandirá la configuración de BOBI existente. Consulte el blog de SAP en [Instalación de la plataforma de inteligencia empresarial SAP BusinessObjects con Azure SQL DB](https://blogs.sap.com/2020/06/19/sap-on-azure-sap-businessobjects-business-intelligence-platform-setup-with-azure-sql-db-managed-paas-database/).

> [!IMPORTANT]
> Los números de versión del motor de base de datos para SQL Server y Azure SQL Database no son comparables entre sí y, en su lugar, son números de compilación internos para estos productos independientes. El motor de base de datos de Azure SQL Database se basa en el mismo código base que el de SQL Server. Lo más importante es que el motor de base de datos de Azure SQL Database siempre tiene los bits más recientes del motor de base de datos SQL. La versión 12 de Azure SQL Database es más reciente que la versión 15 de SQL Server.

Para averiguar la versión actual de la base de datos de Azure SQL, puede comprobar la configuración de la Consola de administración central (CMC), o puede ejecutar la consulta siguiente mediante [sqlcmd](/sql/tools/sqlcmd-utility?preserve-view=true&view=sql-server-ver15) o [SQL Server Management Studio (SSMS)](/sql/ssms/sql-server-management-studio-ssms?preserve-view=true&view=sql-server-ver15). La alineación de las versiones de SQL con la compatibilidad predeterminada se puede encontrar en el artículo sobre el [nivel de compatibilidad de bases de datos](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level?preserve-view=true&view=sql-server-ver15).

![Información de bases de datos en CMC](media/businessobjects-deployment-guide/businessobjects-deployment-windows-sql-cmc.PNG)

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

Después de la instalación de instancias múltiples de la plataforma SAP BOBI, deben seguirse pasos adicionales posteriores a la configuración para admitir la alta disponibilidad de la aplicación.

### <a name="configuring-cluster-name"></a>Configuración del nombre del clúster

En la implementación de instancias múltiples de la plataforma SAP BOBI, quiere ejecutar varios servidores CMS juntos en un clúster. Un clúster consta de dos o más servidores CMS que funcionan conjuntamente en una base de datos común del sistema CMS. Si se produce un error en un nodo que se ejecuta en CMS, un nodo con otro CMS seguirá prestando servicio a las solicitudes de la plataforma de inteligencia empresarial. De manera predeterminada, en la plataforma SAP BOBI, un nombre de clúster refleja el nombre de host del primer CMS que instale. 

Para configurar el nombre del clúster en Windows, siga las instrucciones mencionadas en [Guía del administrador de la plataforma de inteligencia empresarial de SAP](https://help.sap.com/viewer/2e167338c1b24da9b2a94e68efd79c42/4.3.1/en-US). Después de configurar el nombre del clúster, siga la nota de SAP [1660440](https://launchpad.support.sap.com/#/notes/1660440) para establecer la entrada predeterminada del sistema en la página de inicio de sesión de Launchpad para CMC o BI. 

### <a name="configure-input-and-output-filestore-location-to-azure-premium-files"></a>Configuración de la ubicación del almacén de archivos de entrada y salida en Azure Premium Files

El almacén de archivos hace referencia a los directorios de disco en los que se encuentran los archivos reales de SAP BusinessObjects. La ubicación predeterminada del servidor de repositorios de archivos para la plataforma SAP BOBI se encuentra en el directorio de instalación local. En una implementación de instancias múltiples, es importante configurar el almacén de archivos en un almacenamiento compartido, como Azure Premium Files o Azure NetApp Files, para que se pueda acceder a él desde todos los servidores de capa de almacenamiento.

1. Si no se crea, siga las instrucciones proporcionadas en la sección anterior > **Aprovisionamiento de Azure Premium Files** para crear y montar Azure Premium Files.

   > [!Tip]
   > Elija la redundancia de almacenamiento para Azure Premium Files (LRS o ZRS) en función de la implementación de máquinas virtuales (conjunto de disponibilidad o zona de disponibilidad).

2. Siga la nota de SAP [2512660](https://launchpad.support.sap.com/#/notes/0002512660) para cambiar la ruta de acceso del repositorio de archivos (entrada y salida).

### <a name="tomcat-clustering---session-replication"></a>Agrupación en clústeres de Tomcat: replicación de sesión

Tomcat admite la agrupación en clústeres de dos o más servidores de aplicaciones para la replicación de sesión y la conmutación por error. Las sesiones de la plataforma SAP BOBI están serializadas, una sesión de usuario puede conmutar por error a otra instancia de Tomcat, incluso cuando se produce un error en un servidor de aplicaciones. Por ejemplo, si un usuario está conectado a un servidor web que produce un error mientras el usuario navega por una jerarquía de carpetas en la aplicación de inteligencia empresarial SAP. En un clúster configurado correctamente, el usuario puede seguir navegando por la jerarquía de carpetas sin que se redirija a la página de inicio de sesión.

En la nota de SAP [2808640](https://launchpad.support.sap.com/#/notes/2808640), se proporcionan los pasos para configurar la agrupación en clústeres de Tomcat mediante multidifusión. Sin embargo, en Azure no se admite la multidifusión. Para que el clúster de Tomcat funcione en Azure, debe usar [StaticMembershipInterceptor](https://tomcat.apache.org/tomcat-8.0-doc/config/cluster-interceptor.html#Static_Membership) (nota de SAP [2764907](https://launchpad.support.sap.com/#/notes/2764907)). Consulte [Agrupación en clústeres de Tomcat mediante pertenencia estática para la plataforma de inteligencia empresarial SAP BusinessObjects](https://blogs.sap.com/2020/09/04/sap-on-azure-tomcat-clustering-using-static-membership-for-sap-businessobjects-bi-platform/) en el blog de SAP para configurar el clúster de Tomcat en Azure.

### <a name="load-balancing-web-tier-of-sap-bi-platform"></a>Nivel web de equilibrio de carga de la plataforma de inteligencia empresarial SAP

En la implementación de varias instancias de SAP BOBI, los servidores de aplicaciones web de Java (nivel web) se ejecutan en dos o más hosts. Para distribuir la carga de usuarios uniformemente entre los servidores web, puede usar un equilibrador de carga entre los usuarios finales y los servidores web. En Azure, puede usar Azure Load Balancer o Azure Application Gateway para administrar el tráfico a los servidores de aplicaciones web. En la sección siguiente se explican los detalles de cada oferta.

1. [Azure Load Balancer](../../../load-balancer/load-balancer-overview.md) es un equilibrador de carga de nivel 4 (TCP, UDP) de alto rendimiento y baja latencia que distribuye el tráfico entre máquinas virtuales correctas. Un sondeo de estado de equilibrador de carga supervisa un puerto determinado en cada máquina virtual y solo distribuye tráfico a máquinas virtuales operativas. Puede elegir un equilibrador de carga público o un equilibrador de carga interno, en función de si quiere que la plataforma de inteligencia empresarial SAP sea accesible desde Internet o no. Tiene redundancia de zona, lo que garantiza una alta disponibilidad en las instancias de Availability Zones.

   Consulte la sección Equilibrador de carga interno en la ilustración siguiente, donde el servidor de aplicaciones web se ejecuta en el puerto 8080 (el puerto HTTP predeterminado de Tomcat) que lo supervisará el sondeo de estado. Por lo tanto, cualquier solicitud entrante que proceda de usuarios finales se redirigirá a los servidores de aplicaciones web (azuswinboap1 o azuswinboap2) en el grupo de back-end. El equilibrador de carga no admite la terminación TLS/SSL (también conocida como descarga TLS/SSL). Si usa Azure Load Balancer para distribuir el tráfico entre servidores web, se recomienda usar Standard Load Balancer.

   > [!NOTE]
   > Cuando las máquinas virtuales sin direcciones IP públicas se colocan en el grupo de back-end de Standard Load Balancer interno (sin dirección IP pública), no hay conectividad saliente de Internet, a menos que se realice una configuración adicional para permitir el enrutamiento a puntos de conexión públicos. Para obtener más información sobre cómo obtener conectividad saliente, vea [Conectividad de punto de conexión público para máquinas virtuales con Azure Standard Load Balancer en escenarios de alta disponibilidad de SAP](high-availability-guide-standard-load-balancer-outbound-connections.md).

   ![Azure Load Balancer para equilibrar el tráfico entre servidores web](media/businessobjects-deployment-guide/businessobjects-deployment-windows-load-balancer.png)

2. [Azure Application Gateway](../../../application-gateway/overview.md) proporciona un controlador de entrega de aplicaciones (ADC) como servicio, que se usa para ayudar a la aplicación a dirigir el tráfico de usuario a uno o varios servidores de aplicaciones web. Ofrece diversas funcionalidades de equilibrio de carga de capa 7, como la descarga SSL/TLS, firewall de aplicaciones web (WAF), afinidad de sesión basada en cookies y otras, para las aplicaciones.

   En la plataforma de inteligencia empresarial SAP, Application Gateway dirige el tráfico web de la aplicación a los recursos especificados en un grupo de back-end: azuswinboap1 o azuswinboap2. Se asignan escuchas a los puertos, se crean reglas y se agregan recursos a un grupo de back-end. En la ilustración siguiente, la puerta de enlace de aplicación con una dirección IP (10.31.3.25) de front-end privada actúa como punto de entrada para los usuarios, controla las conexiones de TLS/SSL (HTTPS: TCP/443) entrantes, descifra TLS/SSL y pasa la solicitud (HTTP: TCP/8080) a los servidores del grupo de back-end. Con la característica de terminación TLS/SSL integrada, solo es necesario mantener un certificado TLS/SSL en la puerta de enlace de aplicación, lo que simplifica las operaciones.

   ![Application Gateway para equilibrar el tráfico entre servidores web](media/businessobjects-deployment-guide/businessobjects-deployment-windows-application-gateway.png)

   Para configurar Application Gateway para el servidor web de SAP BOBI, puede consultar [Servidores web de SAP BOBI de equilibrio de carga mediante Azure Application Gateway](https://blogs.sap.com/2020/09/17/sap-on-azure-load-balancing-web-application-servers-for-sap-bobi-using-azure-application-gateway/) en el blog de SAP.

   > [!NOTE]
   > Se recomienda usar Azure Application Gateway para equilibrar la carga del tráfico en el servidor web, ya que proporciona características parecidas, como la descarga SSL, la centralización de la administración de SSL para reducir la sobrecarga de cifrado y descifrado en el servidor, el algoritmo Round-Robin para distribuir el tráfico, las capacidades de Web Application Firewall (WAF), la alta disponibilidad, etc.

## <a name="sap-businessobjects-bi-platform-reliability-on-azure"></a>Confiabilidad de la plataforma de inteligencia empresarial SAP BusinessObjects en Azure

La plataforma de inteligencia empresarial SAP BusinessObjects incluye diferentes niveles, que están optimizados para operaciones y tareas específicas. Cuando un componente de cualquier nivel no esté disponible, la aplicación SAP BOBI dejará de estar accesible o no funcionará ninguna de sus funcionalidades. Por lo tanto, debe asegurarse de que cada nivel esté diseñado para ser confiable y mantener la aplicación operativa sin interrupción empresarial.

En esta guía se explorará cómo las características nativas de Azure en combinación con la configuración de la plataforma SAP BOBI mejoran la disponibilidad de la implementación de SAP. Esta sección se centra en las siguientes opciones de la confiabilidad de la plataforma SAP BOBI en Azure:

- **Copia de seguridad y restauración:** Proceso de creación de copias periódicas de datos y aplicaciones en una ubicación independiente. Por lo tanto, se puede restaurar o recuperar al estado anterior si se pierden o dañan los datos o las aplicaciones originales.

- **Alta disponibilidad:** Una plataforma de alta disponibilidad tiene al menos dos de todo en la región de Azure para mantener la aplicación operativa si uno de los servidores deja de estar disponible.
- **Recuperación ante desastres:** Se trata de un proceso de restauración de la funcionalidad de la aplicación si hay alguna pérdida catastrófica, como cuando toda la región de Azure deja de estar disponible debido a algún desastre natural.

La implementación de esta solución varía en función de la naturaleza de la configuración del sistema en Azure. Por lo tanto, el cliente debe adaptar la solución de copia de seguridad y restauración, de alta disponibilidad y recuperación ante desastres en función de sus necesidades empresariales.

## <a name="back-up-and-restore"></a>Copia de seguridad y restauración

La copia de seguridad y restauración es un proceso de creación de copias periódicas de datos y aplicaciones en una ubicación independiente. Por lo tanto, se puede restaurar o recuperar al estado anterior si se pierden o dañan los datos o las aplicaciones originales. También es un componente esencial de cualquier estrategia de recuperación ante desastres empresarial. Estas copias de seguridad permiten la restauración de aplicaciones y bases de datos a un momento dado dentro del período de retención configurado.

Para desarrollar una estrategia completa de copia de seguridad y restauración para la plataforma SAP BOBI, identifique los componentes que provocan el tiempo de inactividad del sistema o la interrupción de la aplicación. En la plataforma SAP BOBI, la copia de seguridad de los componentes siguientes es fundamental para proteger la aplicación.

- Directorio de instalación de SAP BOBI (Managed Disks Premium)
- Almacén de archivos (Azure Premium Files o Azure NetApp Files para una instalación distribuida)
- Base de datos CMS y de auditoría (base de datos de Azure SQL, Azure Database for MySQL o base de datos en una máquina virtual de Azure)

En la siguiente sección se describe cómo implementar una estrategia de copia de seguridad y restauración para cada componente en la plataforma SAP BOBI.

### <a name="backup--restore-for-sap-bobi-installation-directory"></a>Copia de seguridad y restauración para el directorio de instalación de SAP BOBI

En Azure, la manera más sencilla de hacer copias de seguridad de máquinas virtuales, incluidos todos los discos conectados, es mediante el servicio de [copia de seguridad de VM de Azure](../../../backup/backup-azure-vms-introduction.md). Proporciona una copia de seguridad independiente y aislada para impedir la destrucción accidental de los datos en las VM. Las copias de seguridad se almacenan en un almacén de Recovery Services con administración integrada de puntos de recuperación. La configuración y la escalabilidad son sencillas, las copias de seguridad están optimizadas y puede restaurarlas fácilmente cuando sea necesario.

Como parte del proceso de copia de seguridad, se realiza una instantánea y los datos se transfieren al almacén de Recovery Services sin afectar a las cargas de trabajo de producción. La instantánea proporciona diferentes niveles de coherencia, como se describe en el artículo [Coherencia de instantáneas](../../../backup/backup-azure-vms-introduction.md#snapshot-consistency). Azure Backup también ofrece compatibilidad en paralelo con la copia de seguridad de los discos administrados mediante [Azure Disk Backup](../../../backup/disk-backup-overview.md), además de soluciones de [copia de seguridad de VM de Azure](../../../backup/backup-azure-vms-introduction.md). Esto resulta útil cuando es necesario hacer copias de seguridad coherentes de máquinas virtuales una vez al día, y copias de seguridad más frecuentes de los discos del SO, o de un disco de datos específico, que sean coherentes con los bloqueos. Para obtener más información, consulte [Copia de seguridad de VM de Azure](../../../backup/backup-azure-vms-introduction.md), [Azure Disk Backup](../../../backup/disk-backup-overview.md) y [Preguntas más frecuentes sobre la copia de seguridad de VM de Azure](../../../backup/backup-azure-vm-backup-faq.yml).

### <a name="backup--restore-for-filestore"></a>Copia de seguridad y restauración del almacén de archivos

En función de la implementación, el almacén de archivos de la plataforma SAP BOBI puede estar en Azure NetApp Files o Azure Files. Elija entre las siguientes opciones de copia de seguridad y restauración en función del almacenamiento que use para el almacén de archivos.

1. Para **Azure NetApp Files**, puede crear instantáneas a petición y programar la creación automática de instantáneas mediante el uso de directivas de instantáneas. Las copias de instantáneas proporcionan una copia de un momento dado del volumen de ANF. Para obtener más información, consulte [Administración de instantáneas mediante Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-manage-snapshots.md).
2. La copia de seguridad de **Azure Files** se integra con el servicio nativo de [Azure Backup](../../../backup/backup-overview.md), que centraliza la función de copia de seguridad y restauración junto con la copia de seguridad de las máquinas virtuales y simplifica el trabajo de las operaciones. Para obtener más información, vea [Copia de seguridad de recursos compartidos de archivos de Azure](../../../backup/azure-file-share-backup-overview.md) y [Preguntas acerca de la copia de seguridad de archivos de Azure](../../../backup/backup-azure-files-faq.yml).

Si ha creado un servidor NFS independiente, asegúrese de implementar la estrategia de copia de seguridad y restauración para aquel.

### <a name="backup--restore-for-cms-and-audit-database"></a>Copia de seguridad y restauración de las bases de datos CMS y de auditoría

En el caso de la plataforma SAP BOBI que se ejecuta en máquinas virtuales Windows, las bases de datos CMS y de auditoría se pueden ejecutar en cualquiera de las bases de datos admitidas, como se describe en la [matriz de compatibilidad](businessobjects-deployment-guide.md#support-matrix) de la guía de implementación y planeamiento de la plataforma de inteligencia empresarial de SAP BusinessObjects en Azure. Por lo tanto, es importante que adopte la estrategia de copia de seguridad y restauración en función de la base de datos usada para el almacén de datos CMS y de auditoría.

1. La **base de datos de Azure SQL** emplea tecnología de SQL Server para crear [copias de seguridad completas](/sql/relational-databases/backup-restore/full-database-backups-sql-server?preserve-view=true&view=sql-server-ver15) cada semana, [copias de seguridad diferenciales](/sql/relational-databases/backup-restore/differential-backups-sql-server?preserve-view=true&view=sql-server-ver15) cada 12 a 24 horas y copias de seguridad del [registro de transacciones](/sql/relational-databases/backup-restore/transaction-log-backups-sql-server?preserve-view=true&view=sql-server-ver15) cada 5 a 10 minutos. La frecuencia de las copias de seguridad del registro de transacciones se basa en el tamaño de proceso y en la cantidad de actividad de la base de datos. El usuario puede elegir una opción para configurar la redundancia de almacenamiento de copias de seguridad entre blobs de almacenamiento con redundancia local, con redundancia de zona o con redundancia geográfica. Los mecanismos de redundancia de almacenamiento almacenan varias copias de los datos para protegerlos de eventos planeados y no planeados, como errores transitorios del hardware, interrupciones del suministro eléctrico o cortes de la red, y desastres naturales masivos. De manera predeterminada, la base de datos de Azure SQL almacena las copias de seguridad en [blobs de almacenamiento](../../../storage/common/storage-redundancy.md) con redundancia geográfica que se replican en una [región emparejada](../../../best-practices-availability-paired-regions.md). Se puede cambiar en función del requisito empresarial a blobs de almacenamiento con redundancia local o con redundancia de zona. Para obtener información más actualizada sobre la programación, la retención y el consumo de almacenamiento de copias de seguridad de bases de datos de Azure SQL, consulte [Copias de seguridad automatizadas - Azure SQL Database e Instancia administrada de SQL](../../../azure-sql/database/automated-backups-overview.md).

2. **Azure Database for MySQL** crea automáticamente copias de seguridad del servidor y las guarda en un almacenamiento con redundancia local o con redundancia geográfica configurado por el usuario. Azure Database for MySQL realiza copias de seguridad de los archivos de datos y del registro de transacciones. En función del tamaño de almacenamiento máximo admitido, se realizan copias de seguridad completas y diferenciales (servidores de almacenamiento de hasta 4 TB) o copias de seguridad de instantáneas (servidores de almacenamiento de hasta 16 TB). Estas copias de seguridad permiten restaurar un servidor a un momento dado dentro del período de retención de copias de seguridad configurado. De manera predeterminada, el período de retención de la copia de seguridad es de siete días, pero se puede [configurar opcionalmente](../../../mysql/howto-restore-server-portal.md#set-backup-configuration) hasta 35 días. Todas las copias de seguridad se cifran mediante cifrado AES de 256 bits. Estos archivos de copia de seguridad no están expuestos al usuario y no se pueden exportar. Estas copias de seguridad solo se pueden usar en operaciones de restauración de Azure Database for MySQL. Puede usar [mysqldump](../../../mysql/concepts-migrate-dump-restore.md) para copiar una base de datos. Para más información, consulte [Copia de seguridad y restauración en Azure Database for MySQL](../../../mysql/concepts-backup.md).

3. Para una base de datos instalada en una máquina virtual de Azure, puede usar herramientas de copia de seguridad estándar o [Azure Backup](../../../backup/sap-hana-db-about.md) para las bases de datos admitidas. Además, si los servicios y las herramientas de Azure no cumplen sus requisitos, puede usar herramientas de copia de seguridad compatibles de terceros que proporcionen un agente para la copia de seguridad y recuperación de todos los componentes de la plataforma SAP BOBI.

## <a name="high-availability"></a>Alta disponibilidad

Alta disponibilidad se refiere a un conjunto de tecnologías que minimiza las interrupciones de TI al proporcionar una continuidad empresarial de los servicios o la aplicación mediante componentes redundantes, con tolerancia de errores o protegidos mediante conmutación por error dentro del mismo centro de datos. En nuestro caso, el centro de datos se encuentra en una región de Azure. En el artículo [Escenarios y arquitectura de alta disponibilidad para SAP](sap-high-availability-architecture-scenarios.md) se proporciona información sobre las diferentes técnicas de alta disponibilidad y la recomendación ofrecida en Azure para aplicaciones SAP, que complementan las instrucciones de esta sección.

En función del resultado de tamaño de la plataforma SAP BOBI, debe diseñar el panorama y determinar la distribución de los componentes de inteligencia empresarial en Azure Virtual Machines y las subredes. El nivel de redundancia en la arquitectura distribuida depende del objetivo de tiempo de recuperación (RTO) y del objetivo de punto de recuperación (RPO) necesarios para la empresa. La plataforma SAP BOBI incluye diferentes niveles y componentes en cada nivel que se deben diseñar para lograr redundancia. Por lo tanto, si se produce un error en un componente, no habrá ninguna interrupción en la aplicación SAP BOBI. Por ejemplo,

- Servidores de aplicaciones redundantes, como servidores de aplicaciones de inteligencia empresarial y servidores web.
- Componentes únicos, como la base de datos CMS, el almacén de archivos y Load Balancer.

En la siguiente sección se describe cómo lograr una alta disponibilidad en cada componente de la plataforma SAP BOBI.

### <a name="high-availability-for-application-servers"></a>Alta disponibilidad en los servidores de aplicaciones

En el caso de los servidores de aplicaciones web y de inteligencia empresarial, independientemente de si se instalan de forma independiente o conjunta, no se necesita una solución específica de alta disponibilidad. Puede lograr una alta disponibilidad mediante redundancia, es decir, configurando varias instancias de inteligencia empresarial y servidores web en varias máquinas virtuales de Azure. Puede implementar esta máquina virtual en [conjuntos de disponibilidad](sap-high-availability-architecture-scenarios.md#multiple-instances-of-virtual-machines-in-the-same-availability-set) o [Availability Zones](sap-high-availability-architecture-scenarios.md#azure-availability-zones) en función del RTO empresarial requerido. Para la implementación en Availability Zones, asegúrese de que todos los demás componentes de la plataforma SAP BOBI también estén diseñados para tener redundancia de zona.

Actualmente, no todas las regiones de Azure ofrecen zonas de disponibilidad, por lo que debe adoptar la estrategia de implementación en función de su región. Las regiones de Azure que ofrecen zonas aparecen en [Azure Availability Zones](../../../availability-zones/az-overview.md).

> [!Important]
> Los conceptos de zonas de disponibilidad de Azure y de conjuntos de disponibilidad de Azure son mutuamente excluyentes. Esto significa que se pueden implementar dos o varias máquinas virtuales en una zona de disponibilidad específica o en un conjunto de disponibilidad de Azure, pero no en ambos.

### <a name="high-availability-for-cms-database"></a>Alta disponibilidad de la base de datos CMS

Si usa el servicio de base de datos como servicio (DBaaS) de Azure para las bases de datos CMS y de auditoría, el marco de alta disponibilidad con redundancia local se proporciona de manera predeterminada. Solo tiene que seleccionar las capacidades de alta disponibilidad, redundancia y resistencia inherentes a la región y el servicio, sin necesidad de configurar ningún componente adicional. Si la estrategia de implementación de la plataforma SAP BOBI se encuentra en la zona de disponibilidad, debe asegurarse de lograr la redundancia de zona para las bases de datos CMS y de auditoría. Para obtener más información sobre la alta disponibilidad para las ofertas de DBaaS admitida en Azure, consulte [Alta disponibilidad para Azure SQL Database](../../../azure-sql/database/high-availability-sla.md) y [Alta disponibilidad en Azure Database for MySQL](../../../mysql/concepts-high-availability.md). 

Para otras implementaciones de DBMS para la base de datos CMS, consulte las [guías de implementación de DBMS para la carga de trabajo de SAP](dbms_guide_general.md), que proporciona información sobre las distintas implementaciones de DBMS y su enfoque para lograr alta disponibilidad.

### <a name="high-availability-for-filestore"></a>Alta disponibilidad para el almacén de archivos

El almacén de archivos hace referencia a los directorios del disco donde se almacena contenido como informes, universos y conexiones. Se comparte en todos los servidores de aplicaciones de ese sistema. Por lo tanto, debe asegurarse de que tenga alta disponibilidad, junto con otros componentes de la plataforma SAP BOBI.

Para la plataforma SAP BOBI que se ejecuta en Windows, puede elegir [Azure Premium Files](../../../storage/files/storage-files-introduction.md) o [Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-introduction.md) para el almacén de archivos, que, por naturaleza, está diseñado para tener alta disponibilidad y alta durabilidad. Azure Premium Files admite el almacenamiento con redundancia de zona, que puede ser útil para la implementación entre zonas de la plataforma SAP BOBI. Para más información, consulte la sección [Redundancia](../../../storage/files/storage-files-planning.md#redundancy) para Azure Files.

Dado que el servicio de recursos compartidos de archivos no está disponible en todas las regiones, asegúrese de consultar el sitio de [Productos disponibles por región](https://azure.microsoft.com/en-us/global-infrastructure/services/) para buscar información actualizada. Si el servicio no está disponible en su región, puede crear un servidor NFS desde el que pueda compartir el sistema de archivos con la aplicación SAP BOBI. Sin embargo, también debe tener en cuenta su alta disponibilidad.

### <a name="high-availability-for-load-balancer"></a>Alta disponibilidad para Load Balancer

Para distribuir el tráfico a través de un servidor web, puede usar Azure Load Balancer o Azure Application Gateway. La redundancia para cualquiera de los equilibradores de carga se puede lograr en función de la SKU que elija para la implementación.

1. Para Azure Load Balancer, la redundancia se puede lograr mediante la configuración del front-end de Standard Load Balancer como con redundancia de zona. Para más información, consulte [Standard Load Balancer y Availability Zones](../../../load-balancer/load-balancer-standard-availability-zones.md).
2. Para Application Gateway, se puede lograr alta disponibilidad en función del tipo de nivel seleccionado durante la implementación.
   1. La SKU v1 admite escenarios de alta disponibilidad cuando se han implementado dos o más instancias. Azure distribuye estas instancias entre dominios de actualización y de errores para asegurarse de que las instancias no produzcan un error todas al mismo tiempo. Por lo tanto, con esta SKU, se puede lograr redundancia dentro de la zona.
   2. La SKU v2 garantiza automáticamente que las nuevas instancias se distribuyan entre dominios de error y dominios de actualización. Si elige la redundancia de zona, las instancias más recientes también se distribuyen entre las zonas de disponibilidad para ofrecer resistencia ante errores de zona. Para más información, consulte [Escalabilidad automática y Application Gateway con redundancia de zona v2](../../../application-gateway/application-gateway-autoscaling-zone-redundant.md).

### <a name="reference-high-availability-architecture-for-sap-businessobjects-bi-platform"></a>Arquitectura de alta disponibilidad de referencia para la plataforma de inteligencia empresarial SAP BusinessObjects

En la arquitectura de referencia siguiente se describe la configuración de la plataforma SAP BOBI en las zonas de disponibilidad que se ejecutan en servidores Windows. La arquitectura muestra el uso de diferentes servicios de Azure, como Azure Application Gateway, Azure Premium Files (almacén de archivos), base de datos de Azure SQL (base de datos CMS y de auditoría) para la plataforma SAP BOBI, que ofrece redundancia de zona integrada, lo que reduce la complejidad de administrar diferentes soluciones de alta disponibilidad.

En la ilustración siguiente, la carga del tráfico entrante (HTTPS - TCP/443) se equilibra mediante una SKU de Azure Application Gateway v2, que abarca varias zonas de disponibilidad. Application Gateway distribuye la solicitud de usuario entre los servidores web, que se distribuyen en varias zonas de disponibilidad. El servidor web reenvía la solicitud a las instancias del servidor de administración y procesamiento que se han implementado en máquinas virtuales independientes entre las zonas de disponibilidad. Azure Premium Files con almacenamiento con redundancia de zona se conecta a través de un vínculo privado a las máquinas virtuales de capa de almacenamiento y administración para acceder al contenido, como informes, universo y conexiones. La aplicación accede a las bases de datos CMS y de auditoría que se ejecutan en una base de datos de Azure SQL (DBaaS) con redundancia de zona, que replica las bases de datos en varias ubicaciones físicas dentro de la región de Azure.

![Arquitectura de alta disponibilidad para la plataforma SAP BOBI en Windows](media/businessobjects-deployment-guide/businessobjects-deployment-windows-high-availability-availability-zone.png)

En la arquitectura anterior se proporciona información sobre cómo se puede realizar la implementación de SAP BOBI en Azure. Sin embargo, no cubre todas las posibles opciones de configuración de la plataforma SAP BOBI en Azure. El cliente puede personalizar su implementación en función de las necesidades empresariales, eligiendo diferentes productos o servicios para los componentes, como Load Balancer, el servidor de repositorio de archivos y DBMS.

En caso de que las zonas de disponibilidad no estén disponibles en la región seleccionada, puede implementar máquinas virtuales de Azure en conjuntos de disponibilidad. Azure se asegura de que las VM colocadas en un conjunto de disponibilidad se ejecuten en varios servidores físicos, grupos de proceso, unidades de almacenamiento y conmutadores de red. De modo que si se produce un error de hardware o software, solo un subconjunto de las VM se verá afectado y la solución general permanecerá operativa.

## <a name="disaster-recovery"></a>Recuperación ante desastres

En la instrucción de esta sección se explica la estrategia para proporcionar protección de recuperación ante desastres para la plataforma SAP BOBI. Complementa el documento [Recuperación ante desastres para SAP](../../../site-recovery/site-recovery-sap.md), que representa los recursos principales para el enfoque general de recuperación ante desastres de SAP. Para la plataforma de inteligencia empresarial SAP BusinessObjects, consulte la nota de SAP [2056228](https://launchpad.support.sap.com/#/notes/2056228),que describe los métodos siguientes para implementar el entorno de recuperación ante desastres de forma segura.

 1. Uso total o selectivo de la administración del ciclo de vida (LCM) o de la federación para promover o distribuir el contenido del sistema principal.
 2. Copia periódica sobre el contenido de CMS y FRS.

En esta guía, hablaremos sobre la segunda opción para implementar el entorno de recuperación ante desastres. No se trata una lista exhaustiva de las opciones de configuración posibles para la recuperación ante desastres, sino que trata la solución que incluye los servicios nativos de Azure en combinación con la configuración de la plataforma SAP BOBI.

>[!Important]
>La disponibilidad de cada componente en la plataforma de inteligencia empresarial SAP BusinessObjects debe tenerse en cuenta en la región secundaria, y toda la estrategia de recuperación ante desastres debe probarse exhaustivamente.

### <a name="reference-disaster-recovery-architecture-for-sap-businessobjects-bi-platform"></a>Arquitectura de recuperación ante desastres de referencia para la plataforma de inteligencia empresarial SAP BusinessObjects

Esta arquitectura de referencia está ejecutando una implementación de varias instancias de la plataforma SAP BOBI con servidores de aplicaciones redundantes. Para la recuperación ante desastres, debe conmutar por error todos los componentes de la plataforma SAP BOBI a una región secundaria. En la ilustración siguiente, Azure Premium Files se usa como almacén de archivos; la base de datos de Azure SQL, como repositorio de CMS/auditoría; y Azure Application Gateway, para equilibrar la carga del tráfico. La estrategia para lograr la protección de recuperación antes desastres para cada componente es diferente, lo que se describe en los detalles de la sección siguiente.

![Recuperación ante desastres de la plataforma de inteligencia empresarial SAP BusinessObjects para Windows](media\businessobjects-deployment-guide\businessobjects-deployment-windows-disaster-recovery.png)

### <a name="load-balancer"></a>Equilibrador de carga

Load Balancer se usa para distribuir el tráfico entre servidores de aplicaciones web de la plataforma SAP BOBI. En Azure, puede usar Azure Load Balancer o Azure Application Gateway para equilibrar la carga del tráfico entre los servidores web. Para lograr la recuperación ante desastres para los servicios del equilibrador de carga, debe implementar otra instancia de Azure Load Balancer o Azure Application Gateway en la región secundaria. Para mantener la misma dirección URL después de la conmutación por error de recuperación ante desastres, debe cambiar la entrada en DNS, que apunte al servicio de equilibrio de carga que se ejecuta en la región secundaria.

### <a name="virtual-machines-running-web-and-bi-application-servers"></a>Máquinas virtuales que ejecutan servidores de aplicaciones web y de inteligencia empresarial

El servicio [Azure Site Recovery](../../../site-recovery/site-recovery-overview.md) se puede usar para replicar máquinas virtuales que ejecutan servidores de aplicaciones web y de inteligencia empresarial en la región secundaria. Replica en la región secundaria los servidores y todos sus discos administrados conectados para que, cuando se produzcan desastres o interrupciones, pueda conmutar por error fácilmente a su entorno replicado y seguir trabajando. Para comenzar a replicar todas las máquinas virtuales de la aplicación SAP en el centro de datos de recuperación ante desastres de Azure, siga las instrucciones que aparecen en el artículo [Replicación de una máquina virtual en Azure](../../../site-recovery/azure-to-azure-tutorial-enable-replication.md).

### <a name="filestore"></a>Almacén de archivos

El almacén de archivos es un directorio de discos donde se almacenan los archivos reales, como los informes o los documentos de inteligencia empresarial. Es importante que todos los archivos del almacén de archivos estén sincronizados con la región de recuperación ante desastres. En función del tipo de servicio de recurso compartido de archivos que use para la plataforma SAP BOBI que se ejecuta en Windows, es necesario adoptar una estrategia de recuperación ante desastres para sincronizar el contenido.

- **Los archivos de Azure Premium** solo admiten el almacenamiento con redundancia local (LRS) y con redundancia de zona (ZRS). Para la estrategia de recuperación ante desastres de los archivos de Azure Premium, use [AzCopy](../../../storage/common/storage-use-azcopy-v10.md) o [Azure PowerShell](/powershell/module/az.storage/?preserve-view=true&view=azps-5.8.0) para copiar los archivos en otra cuenta de almacenamiento en una región distinta. Para obtener más información, consulte [Recuperación ante desastres y conmutación por error de la cuenta de almacenamiento](../../../storage/common/storage-disaster-recovery-guidance.md).

- **Azure NetApp Files** proporciona volúmenes NFS y SMB, por lo que se puede usar cualquier herramienta de copia basada en archivos para replicar datos entre regiones de Azure. Para obtener más información sobre cómo copiar el volumen de ANF en otra región, consulte [Preguntas más frecuentes acerca de Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-faqs.md#how-do-i-create-a-copy-of-an-azure-netapp-files-volume-in-another-azure-region).

  Puede usar la replicación entre regiones de Azure NetApp Files, que se encuentra actualmente en [versión preliminar](https://azure.microsoft.com/en-us/blog/azure-netapp-files-cross-region-replication-and-new-enhancements-in-preview/), que usa la tecnología de NetApp SnapMirror®. Por tanto, solo los bloques modificados se envían a través de la red en un formato comprimido y eficaz. Esta tecnología propietaria reduce la cantidad de datos necesarios para replicar entre regiones, lo que ahorra costos de transferencia de datos. También acorta el tiempo de replicación, por lo que puede lograr un objetivo de punto de restauración (RPO) más pequeño. Para obtener más información, consulte [Requisitos y consideraciones del uso de la replicación entre regiones](../../../azure-netapp-files/cross-region-replication-requirements-considerations.md).

### <a name="cms-database"></a>Base de datos CMS

Las bases de datos CMS y de auditoría de la región de recuperación ante desastres deben ser una copia de las bases de datos que se ejecutan en la región primaria. En función del tipo de base de datos, es importante copiar la base de datos en la región de recuperación ante desastres según el RTO y el RPO necesarios para la empresa. En esta sección se describen las distintas opciones disponibles para cada servicio de base de datos como servicio (DBaaS) en Azure, compatible con la aplicación de SAP BOBI que se ejecuta en Windows.

#### <a name="azure-sql-database"></a>Azure SQL Database

Para la estrategia de recuperación ante desastres e [Azure SQL Database](../../../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md), hay dos opciones disponibles para copiar la base de datos en la región secundaria. Ambas opciones de recuperación ofrecen distintos niveles de RTO y RPO. Para obtener más información sobre el RTO y el RPO para cada opción de recuperación, consulte [Recuperación de una base datos en el servidor existente](../../../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md#recover-a-database-to-the-existing-server).

1. [Restauración de una copia de seguridad de base de datos con redundancia geográfica](../../../azure-sql/database/recovery-using-backups.md#geo-restore)

   De manera predeterminada, la base de datos de Azure SQL almacena los datos en [blobs de almacenamiento](../../../storage/common/storage-redundancy.md) con redundancia geográfica que se replican en una [región emparejada](../../../best-practices-availability-paired-regions.md). En el caso de una instancia de SQL Database, la redundancia del almacenamiento de copia de seguridad se puede configurar al crear la base de datos CMS o de auditoría, o se puede actualizar para una base de datos existente; los cambios hechos en una base de datos existente solo se aplican a las copias de seguridad futuras. Una base de datos situada en cualquier base de datos de Azure SQL de cualquier región de Azure se puede restaurar desde las copias de seguridad con replicación geográfica más recientes. La restauración geográfica usa una copia de seguridad con replicación geográfica como su origen. Pero hay un retraso entre el momento en que se hace una copia de seguridad y el momento en que se replica geográficamente en un blob de Azure de otra región. Como resultado, la base de datos restaurada puede encontrarse hasta una hora por detrás de la base de datos original.

   >[!Important]
   >La restauración geográfica está disponible para bases de datos de Azure SQL configuradas con [almacenamiento de copia de seguridad](../../../azure-sql/database/automated-backups-overview.md#backup-storage-redundancy) con redundancia geográfica.

2. [Replicación geográfica](../../../azure-sql/database/active-geo-replication-overview.md) o un [grupo de conmutación por error automática](../../../azure-sql/database/auto-failover-group-overview.md)

   La replicación geográfica es una característica de Azure SQL Database que permite crear bases de datos secundarias legibles de bases de datos individuales en un servidor en la misma región o en otra distinta. Si la replicación geográfica está habilitada para las bases de datos CMS y de auditoría, la aplicación puede iniciar la conmutación por error en una base de datos secundaria de otra región de Azure. La replicación geográfica está habilitada para bases de datos individuales, pero para habilitar la conmutación por error transparente y coordinada de varias bases de datos (CMS y auditoría) para la aplicación de SAP BOBI, es aconsejable usar un grupo de conmutación por error automática. Proporciona la semántica de grupo, además de la replicación geográfica activa, por lo que todo el servidor Azure SQL (todas las bases de datos) se replica en otra región, en lugar de bases de datos individuales. Consulte las funcionalidades en la tabla de [Comparación de la replicación geográfica con los grupos de conmutación por error](../../../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md#compare-geo-replication-with-failover-groups).

   Los grupos de conmutación por error automática proporcionan puntos de conexión de agentes de escucha de lectura-escritura y de solo lectura que no se modifican durante las conmutaciones por error. El punto de conexión de lectura y escritura se puede mantener como cliente de escucha en la entrada de conexión ODBC para las bases de datos CMS y de auditoría. Por lo tanto, ya sea que use la activación de conmutación por error automática o manual, la conmutación por error transforma todas las bases de datos secundarias del grupo en primarias. Después de que la conmutación por error de una base de datos finaliza, el registro de DNS se actualiza automáticamente para redirigir los puntos de conexión a la nueva región. Por lo tanto, la aplicación se conectará automáticamente a la base de datos CMS, ya que los puntos de conexión de lectura y escritura se mantienen como clientes de escucha en la conexión ODBC.

   En la ilustración siguiente, el grupo de conmutación por error automática para un servidor de Azure SQL (azussqlbodb) que se ejecuta en la región Este de EE. UU. 2 se replica en la región secundaria Este de EE. UU. (sitio de recuperación ante desastres). El punto de conexión del cliente de escucha de lectura y escritura se mantiene como cliente de escucha en la conexión ODBC para el servidor de aplicaciones de inteligencia empresarial que se ejecuta en Windows. Después de la conmutación por error, el punto de conexión seguirá siendo el mismo y no se requiere ninguna intervención manual para conectar la aplicación de inteligencia empresarial a la base de datos de Azure SQL en la región secundaria.

   ![Grupos de conmutación por error automática de base de datos de Azure SQL](media\businessobjects-deployment-guide\businessobjects-deployment-windows-sql-failover-group.png)

   Esta opción proporciona un RTO y RPO inferiores a la opción 1. Para obtener más información sobre esta opción, consulte [Uso de grupos de conmutación por error automática para permitir la conmutación por error de varias bases de datos de manera transparente y coordinada](../../../azure-sql/database/auto-failover-group-overview.md).

#### <a name="azure-database-for-mysql"></a>Azure Database for MySQL

Azure Database for MySQL proporciona varias opciones para recuperar la base de datos si se produce algún desastre. Elija la opción adecuada que se adapte a su empresa.

1. Habilite las réplicas de lectura entre regiones para mejorar el planeamiento de la continuidad empresarial y recuperación ante desastres. Puede crear hasta cinco réplicas desde el servidor de origen. Las réplicas de lectura se actualizan de manera asincrónica mediante la tecnología de replicación de registros binarios de MySQL. Las réplicas son nuevos servidores que se administran de forma similar a los servidores de Azure Database for MySQL normales. Obtenga más información sobre las réplicas de lectura, las regiones disponibles, las restricciones y la conmutación por error en el [artículo sobre los conceptos de las réplicas de lectura](../../../mysql/concepts-read-replicas.md).

2. Use la característica de replicación geográfica de Azure Database for MySQL que permite restaurar el servidor mediante copias de seguridad con redundancia geográfica. Se puede acceder a estas copias de seguridad incluso cuando la región en la que se hospeda el servidor está sin conexión. Puede realizar la restauración a partir de estas copias de seguridad en cualquier otra región y volver a poner en línea el servidor.

  > [!Important]
  > La restauración geográfica solo es posible si se ha aprovisionado el servidor con almacenamiento de copia de seguridad con redundancia geográfica. No se admite el cambio de las **opciones de redundancia de copia de seguridad** después de la creación del servidor. Para obtener más información, consulte el artículo [Redundancia de copia de seguridad](../../../mysql/concepts-backup.md#backup-redundancy-options).

A continuación, se muestra la recomendación para la recuperación ante desastres de cada nivel usado en este ejemplo.

| Niveles de la plataforma SAP BOBI                          | Recomendación                                               |
| ------------------------------------------------ | ------------------------------------------------------------ |
| Azure Application Gateway o Azure Load Balancer | Configuración paralela de Application Gateway en la región secundaria    |
| Servidores de aplicaciones web                          | Replicación mediante Site Recovery                             |
| Servidores de aplicaciones de inteligencia empresarial                           | Replicación mediante Site Recovery                             |
| Azure Premium Files                              | AzCopy **o** Azure PowerShell                               |
| Azure NetApp Files                               | Herramienta de copia basada en archivos para replicar datos en la región secundaria **o** replicación entre regiones de ANF (versión preliminar) |
| Azure SQL Database                               | Grupos de replicación geográfica/conmutación por error automática **o** restauración geográfica.     |
| Azure Database for MySQL                         | Réplicas de lectura entre regiones **o** restauración de la copia de seguridad de copias de seguridad con redundancia geográfica. |
## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la recuperación ante desastres para la implementación de una aplicación de SAP de niveles múltiples](../../../site-recovery/site-recovery-sap.md)
- [Planeamiento e implementación de Azure Virtual Machines para SAP](planning-guide.md)
- [Implementación de Azure Virtual Machines para SAP](deployment-guide.md)
- [Implementación de DBMS de Azure Virtual Machines para SAP](./dbms_guide_general.md)