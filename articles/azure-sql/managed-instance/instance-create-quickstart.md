---
title: 'Inicio rápido: Creación de una instancia de Azure SQL Managed Instance (portal)'
description: En este inicio rápido, creará una instancia administrada, un entorno de red y una máquina virtual cliente para el acceso mediante Azure Portal.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: deployment-configuration
ms.custom: ''
ms.devlang: ''
ms.topic: quickstart
author: urosmil
ms.author: urmilano
ms.reviewer: mathoma
ms.date: 1/29/2021
ms.openlocfilehash: 77b43cfe781499c408f1952aed532ddb12c621e5
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082350"
---
# <a name="quickstart-create-an-azure-sql-managed-instance"></a>Inicio rápido: Creación de una instancia administrada de Azure SQL
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

En este inicio rápido se le enseña a crear una [instancia administrada de Azure SQL](sql-managed-instance-paas-overview.md) en Azure Portal.

> [!IMPORTANT]
> Para conocer las limitaciones, consulte las [regiones admitidas](resource-limits.md#supported-regions) y los [tipos de suscripción admitidos](resource-limits.md#supported-subscription-types).

## <a name="create-an-azure-sql-managed-instance"></a>Creación de una instancia administrada de Azure SQL

Para crear una instancia de SQL Managed Instance, siga estos pasos: 

### <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Si no tiene una suscripción a Azure, [cree una cuenta gratuita](https://azure.microsoft.com/free/).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione **Azure SQL** en el menú de la izquierda de Azure Portal. Si **Azure SQL** no está en la lista, seleccione **Todos los servicios** y escriba **Azure SQL** en el cuadro de búsqueda.
1. Seleccione **+ Agregar** para abrir la página **Select SQL deployment option** (Seleccionar la opción de implementación de SQL). Para ver información adicional acerca de una instancia administrada de Azure SQL Managed Instance, seleccione **Mostrar detalles** en el icono **SQL Managed Instance**.
1. Seleccione **Crear**.

   ![Creación de una instancia administrada](./media/instance-create-quickstart/create-azure-sql-managed-instance.png)

4. Use las pestañas del formulario de aprovisionamiento **Crear instancia administrada de Azure SQL** para agregar la información obligatoria y la opcional. En las siguientes secciones se describen estas pestañas.

### <a name="basics-tab"></a>Pestaña Aspectos básicos

- Rellene la información obligatoria necesaria en la pestaña **Aspectos básicos**. Es un conjunto mínimo de información necesaria para aprovisionar una instancia administrada de SQL.

   ![Pestaña "Aspectos básicos" para crear una instancia administrada de SQL](./media/instance-create-quickstart/azure-sql-managed-instance-create-tab-basics.png)

   Use la tabla siguiente como referencia para obtener la información necesaria en esta pestaña.

   | Configuración| Valor sugerido | Descripción |
   | ------ | --------------- | ----------- |
   | **Suscripción** | Su suscripción. | Una suscripción en la que tenga permiso para crear recursos. |
   | **Grupos de recursos** | un grupo de recursos nuevo o existente.|Para conocer cuáles son los nombres de grupo de recursos válidos, consulte el artículo [Convenciones de nomenclatura](/azure/architecture/best-practices/resource-naming).|
   | **Nombre de la instancia administrada** | Cualquier nombre válido.|Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura](/azure/architecture/best-practices/resource-naming).|
   | **Región** |La región en la que desea crear la instancia administrada.|Para obtener información sobre las regiones, consulte [Regiones de Azure](https://azure.microsoft.com/regions/).|
   | **Inicio de sesión de administrador de la instancia administrada** | Cualquier nombre de usuario válido. | Para conocer cuáles son los nombres válidos, consulte el artículo [Convenciones de nomenclatura](/azure/architecture/best-practices/resource-naming). No utilice "serveradmin", ya es un rol de nivel de servidor reservado.|
   | **Contraseña** | Cualquier contraseña válida.| La contraseña debe tener al menos 16 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../../virtual-machines/windows/faq.yml#what-are-the-password-requirements-when-creating-a-vm-).|

- Seleccione **Configurar instancia administrada**  para cambiar el tamaño de los recursos de almacenamiento y de proceso, así como para revisar los planes de tarifa. Utilice los controles deslizantes o cuadros de texto para especificar la cantidad de almacenamiento y el número de núcleos virtuales. Cuando haya terminado, seleccione **Aplicar** para guardar las opciones que haya elegido. 

   ![Formulario de instancia administrada](./media/instance-create-quickstart/azure-sql-managed-instance-create-tab-configure-performance.png)

| Valor| Valor sugerido | Descripción |
| ------ | --------------- | ----------- |
| **Nivel de servicio** | Seleccione una de las opciones. | En función del escenario, seleccione una de las siguientes opciones: </br> <ul><li>**De uso general**: para la mayoría de las cargas de trabajo de producción y la opción predeterminada.</li><li>**Crítico para la empresa**: este nivel está diseñado para cargas de trabajo de baja latencia con alta resistencia a errores y conmutaciones por error rápidas.</li></ul><BR>Para más información, consulte [Niveles de servicio de Azure SQL Database y Azure SQL Managed Instance](../../azure-sql/database/service-tiers-general-purpose-business-critical.md) y revise [Introducción a los límites de recursos de Azure SQL Managed Instance](../../azure-sql/managed-instance/resource-limits.md).|
| **Generación de hardware** | Seleccione una de las opciones. | En general, la generación de hardware define los límites de proceso y de memoria, así como otras características que afectan al rendimiento de la carga de trabajo. **Gen5** es el valor predeterminado.|
| **vCore compute model** (Modelo de proceso de núcleo virtual) | Seleccione una opción. | Los núcleos virtuales representan la cantidad exacta de recursos de proceso que se aprovisionan siempre para la carga de trabajo. **Ocho núcleos virtuales** es el valor predeterminado.|
| **Almacenamiento en GB** | Seleccione una opción. | Tamaño de almacenamiento en GB; seleccione un valor en función del tamaño de datos esperado. Si va a migrar los datos existentes desde el entorno local o en varias plataformas en la nube, consulte [Información general sobre la migración: SQL Server a SQL Managed Instance](../../azure-sql/migration-guides/managed-instance/sql-server-to-managed-instance-overview.md).|
| **Ventaja híbrida de Azure** | Active la opción si es aplicable. | Para aprovechar una licencia existente de Azure. Para más información, consulte [Ventaja híbrida de Azure: Azure SQL Database y SQL Managed Instance](../../azure-sql/azure-hybrid-benefit.md). |
| **Redundancia del almacenamiento de copia de seguridad** | Seleccione **Geo-redundant backup storage** (Almacenamiento de copia de seguridad con redundancia geográfica). | Redundancia del almacenamiento dentro de Azure para el almacenamiento de copia de seguridad. Tenga en cuenta que este valor no se puede cambiar más tarde. El almacenamiento de copia de seguridad con redundancia geográfica es el predeterminado y recomendado, aunque la redundancia de zona y local permite mayor flexibilidad en los costos y residencia de datos en una única región. Para más información, consulte [Redundancia de almacenamiento de copia de seguridad](../database/automated-backups-overview.md?tabs=managed-instance#backup-storage-redundancy).|


- Para examinar las opciones antes de crear una instancia administrada de SQL, puede seleccionar **Revisar y crear**. O bien, configure las opciones de red seleccionando **Siguiente: Redes**.

### <a name="networking-tab"></a>Pestaña Redes

- Rellene la información opcional de la pestaña **Redes**. Si omite esta información, el portal aplicará la configuración predeterminada.

   ![Pestaña "Redes" para crear una instancia administrada](./media/instance-create-quickstart/azure-sql-managed-instance-create-tab-networking.png)

   Use la tabla siguiente como referencia para obtener la información necesaria en esta pestaña.

   | Configuración| Valor sugerido | Descripción |
   | ------ | --------------- | ----------- |
   | **Red virtual** | Seleccione **Crear una nueva red virtual** o una subred y red virtual válidas.| Si hay una red o subred que no esté disponible, se debe [modificar para cumplir los requisitos de la red](vnet-existing-add-subnet.md) antes de seleccionarla como destino de la nueva instancia administrada. Para más información acerca de los requisitos necesarios para configurar el entorno de red de una instancia administrada de SQL, consulte el artículo en el que se indica cómo [configurar una red virtual para una instancia administrada de Azure SQL](connectivity-architecture-overview.md). |
   | **Tipo de conexión** | Elija entre los tipos de conexión proxy y redirigir.|Para más información acerca de los tipos de conexión, consulte el artículo sobre el [tipo de conexión de Instancia administrada de Azure SQL](../database/connectivity-architecture.md#connection-policy).|
   | **Punto de conexión público**  | Seleccione **Deshabilitar**. | Para que se pueda acceder a una instancia administrada mediante el punto de conexión de datos público, es preciso habilitar esta opción. | 
   | **Permitir acceso desde** (si la opción **Punto de conexión público**  está habilitada) | Seleccione **Sin acceso**.  |La experiencia del portal permite configurar un grupo de seguridad con un punto de conexión público. </br> </br> En función del escenario, seleccione una de las siguientes opciones: </br> <ul> <li>**Servicios de Azure**: esta opción es aconsejable cuando se establece conexión desde Power BI u otro servicio multiinquilino. </li> <li> **Internet**: se usa para realizar pruebas cuando se desee poner en marcha rápidamente una instancia administrada. No es aconsejable para entornos de producción. </li> <li> **Sin acceso**: esta opción crea una regla de seguridad **Denegar**. Modifique esta regla para que se pueda acceder a la instancia administrada mediante un punto de conexión público. </li> </ul> </br> Para más información acerca de la seguridad de los puntos de conexión públicos, consulte el artículo sobre el [uso de Instancia administrada de Azure SQL de forma segura con puntos de conexión públicos](public-endpoint-overview.md).|

- Para revisar las opciones antes de crear una instancia administrada, seleccione **Revisar y crear**. O bien, configure más opciones personalizadas, para lo que debe seleccionar **Siguiente: Configuración adicional**.


### <a name="additional-settings"></a>Configuración adicional

- Rellene la información opcional de la pestaña **Configuración adicional**. Si omite esta información, el portal aplicará la configuración predeterminada.

   ![Pestaña "Configuración adicional" para crear una instancia administrada](./media/instance-create-quickstart/azure-sql-managed-instance-create-tab-additional-settings.png)

   Use la tabla siguiente como referencia para obtener la información necesaria en esta pestaña.

   | Configuración| Valor sugerido | Descripción |
   | ------ | --------------- | ----------- |
   | **Intercalación** | Elija la intercalación que desea usar para la instancia administrada. Si migra bases de datos desde SQL Server, compruebe la intercalación de origen mediante `SELECT SERVERPROPERTY(N'Collation')` y use ese valor.| Para obtener información acerca de las intercalaciones, consulte [Configurar o cambiar la intercalación del servidor](/sql/relational-databases/collations/set-or-change-the-server-collation).|   
   | **Zona horaria** | Seleccione la zona horaria que la instancia administrada observará.|Para más información, consulte las [zonas horarias](timezones-overview.md).|
   | **Use as failover secondary** (Usar como secundaria de conmutación por error) | Seleccione **Sí**. | Habilite esta opción para usar la instancia administrada como secundaria del grupo de conmutación por error.|
   | **Instancia administrada de SQL principal** (si en **Usar como secundario de conmutación por error** se ha seleccionado **Sí**) | Elija la instancia administrada principal existente que se unirá en la misma zona DNS con la instancia administrada que está creando. | Este paso habilitará la configuración posterior a la creación del grupo de conmutación por error. Para más información, consulte el [Tutorial: Adición de una instancia administrada a un grupo de conmutación por error](failover-group-add-instance-tutorial.md).|

- Para revisar las opciones antes de crear una instancia administrada, seleccione **Revisar y crear**. O bien, configure etiquetas de Azure seleccionando **Siguiente: Etiquetas** (recomendado).

### <a name="tags"></a>Etiquetas

- Agregue etiquetas a los recursos de la plantilla de Azure Resource Manager (ARM). Las [etiquetas](../../azure-resource-manager/management/tag-resources.md) le ayudan a organizar los recursos de forma lógica. Los valores de etiqueta se muestran en los informes de costos y permiten otras actividades de administración por etiqueta. 

- Considere la posibilidad de etiquetar al menos la nueva instancia de SQL Managed Instance con la etiqueta Propietario para identificar quién la ha creado y la etiqueta Entorno para identificar si este sistema es de producción, desarrollo, etc. Para más información, consulte [Desarrollo de la estrategia de nomenclatura y etiquetado de los recursos de Azure](/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging).
 
- Seleccione **Revisar y crear** para continuar.

## <a name="review--create"></a>Revisar y crear

1. Para revisar las opciones antes de crear una instancia administrada, seleccione la pestaña **Revisar y crear**.

   ![Pestaña para examinar y crear una instancia administrada](./media/instance-create-quickstart/azure-sql-managed-instance-create-tab-review-create.png)

1. Seleccione **Crear** para iniciar el aprovisionamiento de la instancia administrada.

> [!IMPORTANT]
> La implementación de una instancia administrada es una operación de ejecución prolongada. La implementación de la primera instancia en la subred suele tardar mucho más que la implementación en una subred en la que ya existen instancias administradas. Para conocer los tiempos de aprovisionamiento medios, consulte [Introducción a las operaciones de administración de Azure SQL Managed Instance](management-operations-overview.md#duration).

## <a name="monitor-deployment-progress"></a>Supervisión del progreso de la implementación

1. Seleccione el icono **Notificaciones** para ver el estado de la implementación.

   ![Progreso de la implementación de una instancia administrada de SQL](./media/instance-create-quickstart/azure-sql-managed-instance-create-deployment-in-progress.png)

1. Seleccione **Implementación en curso**  en la notificación para abrir la ventana de la instancia administrada de SQL y supervisar el progreso de la implementación. 

> [!TIP]
> - Si ha cerrado el explorador web o ha salido de la pantalla de progreso de la implementación, puede supervisar la operación de aprovisionamiento a través de la página **Información general** de la instancia administrada o mediante PowerShell o la CLI de Azure. Para más información, consulte [Operaciones de supervisión](management-operations-monitor.md#monitor-operations). 
> - Puede cancelar el proceso de aprovisionamiento mediante Azure Portal, PowerShell, la CLI de Azure u otras herramientas con la API REST. Consulte [Cancelación de operaciones de administración de Azure SQL Managed Instance](management-operations-cancel.md).

> [!IMPORTANT]
> - El inicio de la creación de SQL Managed Instance se puede retrasar cuando existan otras operaciones que afecten a la ejecución prolongada, como operaciones de restauración o escalado de larga duración en otras instancias administradas de la misma subred. Para más información, consulte la sección [Impacto de las operaciones de administración](management-operations-overview.md#management-operations-cross-impact).
> - Para poder obtener el estado de creación de instancia administrada, tiene que tener **permisos de lectura** para el grupo de recursos. Si no tiene este permiso o lo revoca mientras la instancia administrada se encuentra en proceso de creación, es posible que SQL Managed Instance no se vea en la lista de implementaciones del grupo de recursos.
>

## <a name="view-resources-created"></a>Visualización de los recursos creados

Tras una implementación correcta de una instancia administrada, para ver los recursos creados:

1. Abra el grupo de recursos de la instancia administrada. 

   ![Recursos de Instancia administrada de SQL](./media/instance-create-quickstart/azure-sql-managed-instance-resources.png)

## <a name="view-and-fine-tune-network-settings"></a>Visualización y ajuste de la configuración de red

Para ajustar la configuración de red de forma opcional, revise lo siguiente:

1. En la lista de recursos, seleccione la tabla de rutas para revisar el objeto de tabla de rutas definida por el usuario (UDR) que se creó.

2. En la tabla de rutas, revise las entradas desde las que se enruta el tráfico hacia la red virtual de la instancia administrada de SQL y dentro de esta. Si crea o configura la tabla de rutas de forma manual, cree estas entradas en la tabla de rutas de SQL Managed Instance.

   ![Entrada de una subred de Instancia administrada de SQL en local](./media/instance-create-quickstart/azure-sql-managed-instance-route-table-user-defined-route.png)

    Para cambiar o agregar rutas, abra **Rutas** en la opción de configuración Tabla de rutas.

3. Vuelva al grupo de recursos y seleccione el objeto de grupo de seguridad de red (NSG) que se creó.

4. Revise las reglas de seguridad entrantes y salientes. 

   ![Reglas de seguridad](./media/instance-create-quickstart/azure-sql-managed-instance-security-rules.png)

    Para cambiar o agregar reglas, abra las **reglas de seguridad de entrada** y las **reglas de seguridad de salida** en la configuración del grupo de seguridad de red.

> [!IMPORTANT]
> Si ha configurado un punto de conexión público para SQL Managed Instance, es preciso que abra los puertos necesarios para que haya tráfico de red que permita establecer conexiones a SQL Managed Instance desde la red pública de Internet. Para más información, consulte [Configuración de un punto de conexión público para SQL Managed Instance](public-endpoint-configure.md#allow-public-endpoint-traffic-on-the-network-security-group).
>

## <a name="retrieve-connection-details-to-sql-managed-instance"></a>Recuperación de los detalles de conexión en la instancia administrada de SQL

Para conectarse a SQL Managed Instance, siga estos pasos para recuperar el nombre de host y el nombre de dominio completo (FQDN):

1. Vuelva al grupo de recursos y seleccione el objeto de instancia administrada de SQL que se creó.

2. En la pestaña **Información general**, busque la propiedad **Host**. Copie el nombre de host de la instancia administrada en el portapapeles, ya que lo usará en el siguiente inicio rápido. Para ello, haga clic en el botón **Copiar al Portapapeles**.

   ![Nombre de host](./media/instance-create-quickstart/azure-sql-managed-instance-host-name.png)

   El valor copiado representa un nombre de dominio completo (FQDN) que se puede usar para conectarse a una instancia administrada de SQL. Es similar al siguiente ejemplo de dirección: *your_host_name.a1b2c3d4e5f6.database.windows.net*.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre cómo conectarse a SQL Managed Instance:
- Para información general sobre las opciones de conexión de las aplicaciones, consulte [Conexión de las aplicaciones a Instancia administrada de SQL](connect-application-instance.md).
- Para ver un inicio rápido que muestre cómo conectarse a SQL Managed Instance desde una máquina virtual de Azure, consulte el documento sobre la [configuración de la conexión a una máquina virtual de Azure](connect-vm-instance-configure.md).
- Para ver un inicio rápido que muestre cómo conectarse a SQL Managed Instance desde un equipo cliente local mediante una conexión de punto a sitio, consulte el artículo acerca de la [configuración de una conexión de punto a sitio](point-to-site-p2s-configure.md).

Para restaurar una base de datos de SQL Server desde un entorno local a SQL Managed Instance: 
- Use [Azure Database Migration Service para la migración](../../dms/tutorial-sql-server-to-managed-instance.md) para realizar la restauración de un archivo de copia de seguridad de base de datos. 
- Use el [comando T-SQL RESTORE](restore-sample-database-quickstart.md) para realizar la restauración desde un archivo de copia de seguridad de base de datos.

Para obtener información acerca de la supervisión avanzada del rendimiento de las bases de datos de Instancia administrada de SQL con inteligencia para la solución de problemas integrada, consulte [Supervisión de instancias de Azure SQL Database mediante Azure SQL Analytics](../../azure-monitor/insights/azure-sql.md).