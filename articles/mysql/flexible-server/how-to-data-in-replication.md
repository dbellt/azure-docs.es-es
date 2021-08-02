---
title: 'Configuración de la replicación de datos de entrada: Servidor flexible de Azure Database for MySQL'
description: En este artículo se describe la configuración de la replicación de datos internos para el servidor flexible de Azure Database for MySQL.
author: SudheeshGH
ms.author: sunaray
ms.service: mysql
ms.topic: how-to
ms.date: 06/08/2021
ms.openlocfilehash: 041e6e2b3a79fa639a00506c81fc3e7ab0a98cec
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111746778"
---
# <a name="how-to-configure-azure-database-for-mysql-flexible-server-data-in-replication"></a>Configuración de la replicación de datos de entrada del servidor flexible de Azure Database for MySQL

En este artículo se describe cómo configurar la [replicación de datos de entrada](concepts-data-in-replication.md) en el servidor flexible de Azure Database for MySQL mediante la configuración de los servidores de origen y de réplica. En este artículo se asume que tiene alguna experiencia previa con servidores y bases de datos MySQL.

> [!NOTE]
> Este artículo contiene referencias al término _esclavo_, un término que Microsoft ya no usa. Cuando se elimine el término del software, se eliminará también de este artículo.
>

Para crear una réplica en el servicio flexible de Azure Database for MySQL, la [replicación de datos de entrada](concepts-data-in-replication.md) sincroniza los datos que proceden de un servidor MySQL de origen local en máquinas virtuales (VM) o en servicios de base de datos en la nube. La replicación de datos de entrada se basa en la posición del archivo de registro binario (binlog). Para obtener más información acerca de la replicación de binlog, consulte la [Introducción a la replicación de binlog de MySQL](https://dev.mysql.com/doc/refman/5.7/en/binlog-replication-configuration-overview.html).

Revise las [limitaciones y los requisitos](concepts-data-in-replication.md#limitations-and-considerations) de la replicación de datos de entrada antes de seguir los pasos de este artículo.

## <a name="create-an-azure-database-for-mysql-flexible-server-instance-to-use-as-a-replica"></a>Creación de una instancia del servidor flexible de Azure Database for MySQL para usarla como réplica

1. Cree una instancia del servidor flexible de Azure Database for MySQL (por ejemplo, "replica.mysql.database.azure.com"). Consulte [Uso de Azure Portal para crear un servidor flexible de Azure Database for MySQL](quickstart-create-server-portal.md) para la creación del servidor. Este servidor es el servidor de "réplica" para la replicación de datos de entrada.

2. Cree las mismas cuentas de usuario y los privilegios correspondientes.

   Las cuentas de usuario no se replican desde el servidor de origen al servidor de réplica. Si planea proporcionar a los usuarios acceso al servidor de réplica, debe crear manualmente todas las cuentas y privilegios correspondientes en este servidor flexible de Azure Database for MySQL recién creado.

## <a name="configure-the-source-mysql-server"></a>Configuración del servidor MySQL

En los siguientes pasos se prepara y configura el servidor MySQL hospedado en el entorno local, en una máquina virtual o en un servicio de base de datos que se hospeda en otros proveedores de nube para la replicación de datos de entrada. Este servidor es el "origen" de la replicación de datos de entrada.

1. Revise los [requisitos del servidor de origen](concepts-data-in-replication.md#requirements) antes de continuar.

2. Requisitos de red 
    * Asegúrese de que el servidor de origen permite el tráfico entrante y saliente en el puerto 3306 y de que tiene una **dirección IP pública**, el DNS es accesible públicamente o tiene un nombre de dominio completo (FQDN).

    * Si el acceso privado está en uso, asegúrese de que hay conectividad entre el servidor de origen y la red virtual en la que se hospeda el servidor de réplica. 
    * Asegúrese de que proporcionamos conectividad sitio a sitio a sus servidores de origen locales mediante [ExpressRoute](../../expressroute/expressroute-introduction.md) o [VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md). Para más información sobre la creación de una red virtual, consulte la documentación de [Virtual Network](../../virtual-network/index.yml)y, especialmente, los artículos de inicio rápido con detalles paso a paso.
    * Si el acceso privado se usa en el servidor de réplica y el origen es la máquina virtual de Azure, asegúrese de que se establece la conectividad de red virtual a red virtual. Se admite el emparejamiento de red virtual a red virtual dentro de las regiones.**Actualmente no se admite el emparejamiento global.** Tendría que usar otros métodos de conectividad para comunicarse entre redes virtuales de regiones distintas, como la conexión de red virtual a red virtual. Para más información, consulte [Puerta de enlace de VPN de red virtual a red virtual](../../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md).
    * Asegúrese de que las reglas del grupo de seguridad de red de la red virtual no bloquean el puerto de salida 3306 (que también es de entrada si MySQL se ejecuta en la máquina virtual de Azure). Para más información sobre el filtrado del tráfico con grupos de seguridad de red para redes virtuales, vea el artículo [Filtrado del tráfico de red con grupos de seguridad de red](../../virtual-network/virtual-network-vnet-plan-design-arm.md).
    * Configure las reglas de firewall del servidor de origen para permitir la dirección IP del servidor de réplica.

  
3. Active el registro binario.

   Compruebe si se ha habilitado el registro binario en el servidor de origen mediante la ejecución del comando siguiente:

   ```sql
   SHOW VARIABLES LIKE 'log_bin';
   ```

   Si la variable [`log_bin`](https://dev.mysql.com/doc/refman/8.0/en/replication-options-binary-log.html#sysvar_log_bin) se devuelve con el valor "ON", el registro binario está habilitado en el servidor.

   Si se devuelve `log_bin` con el valor "OFF" y el servidor de origen se ejecuta de forma local o en máquinas virtuales en las que se puede tener acceso al archivo de configuración (my.cnf), puede seguir estos pasos:
   1. Busque el archivo de configuración de MySQL (my.cnf) en el servidor de origen. Por ejemplo: /etc/my.cnf.
   2. Abra el archivo de configuración para editarlo y busque la sección **mysqld**.
   3. En la sección mysqld, agregue la línea siguiente:

       ```bash
       log-bin=mysql-bin.log
       ```

   4. Reinicie el servicio MySQL en el servidor de origen (o reinicie) para que los cambios surtan efecto.
   5. Una vez que se haya reiniciado el servidor, compruebe que el registro binario está habilitado mediante la ejecución de la misma consulta que antes:

      ```sql
      SHOW VARIABLES LIKE 'log_bin';
      ```

4. Establezca la configuración del servidor de origen.

   La replicación de datos de entrada requiere el parámetro `lower_case_table_names` para ser coherente entre los servidores de origen y de réplica. De manera predeterminada, este parámetro es 1 en el servidor flexible de Azure Database for MySQL.

   ```sql
   SET GLOBAL lower_case_table_names = 1;
   ```

5. Cree un nuevo rol de replicación y configure el permiso.

   Cree una cuenta de usuario en el servidor de origen que está configurado con privilegios de replicación. Esto puede realizarse a través de los comandos SQL o una herramienta como MySQL Workbench. Tenga en cuenta si planea replicar con SSL, ya que necesitará especificarse al crear el usuario. Consulte la documentación de MySQL para entender cómo [agregar cuentas de usuario](https://dev.mysql.com/doc/refman/5.7/en/user-names.html) en el servidor de origen.

   En los comandos siguientes, el nuevo rol de replicación creado puede obtener acceso al origen desde cualquier máquina, no solo desde la máquina que hospeda al propio origen. Esto se hace especificando "syncuser@'%'" en el comando create user. Consulte la documentación de MySQL para más información acerca de la [especificación de nombres de cuenta](https://dev.mysql.com/doc/refman/5.7/en/account-names.html).

   **Comando SQL**

   *Replicación con SSL*

   Para requerir SSL a todas las conexiones de usuario, use el comando siguiente para crear un usuario:

   ```sql
   CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
   GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%' REQUIRE SSL;
   ```

   *Replicación sin SSL*

   Si SSL no es necesario para todas las conexiones, use el comando siguiente para crear un usuario:

   ```sql
   CREATE USER 'syncuser'@'%' IDENTIFIED BY 'yourpassword';
   GRANT REPLICATION SLAVE ON *.* TO ' syncuser'@'%';
   ```

   **MySQL Workbench**

   Para crear el rol de replicación en MySQL Workbench, abra el panel **Usuarios y privilegios** desde el panel **Administración** y, a continuación, seleccione **Agregar cuenta**.

   :::image type="content" source="./media/how-to-data-in-replication/users-privileges.png" alt-text="Usuarios y privilegios":::

   Escriba el nombre de usuario en el campo **Login Name** (Nombre de inicio de sesión).

   :::image type="content" source="./media/how-to-data-in-replication/sync-user.png" alt-text="Usuario de sincronización":::

   Seleccione el panel **Roles administrativos** y, a continuación, seleccione **Servidor subordinado de replicación** en la lista de **Privilegios globales**. A continuación, seleccione **Aplicar** para crear el rol de replicación.

   :::image type="content" source="./media/how-to-data-in-replication/replication-slave.png" alt-text="Servidor subordinado de replicación":::

6. Establezca el servidor de origen en el modo de solo lectura.

   Antes de comenzar a volcar la base de datos, el servidor debe colocarse en modo de solo lectura. En modo de solo lectura, el servidor de origen no podrá procesar ninguna transacción de escritura. Evalúe el impacto para su negocio y la programación de la ventana de solo lectura en un momento de poco tráfico, si es necesario.

   ```sql
   FLUSH TABLES WITH READ LOCK;
   SET GLOBAL read_only = ON;
   ```

7. Obtenga el nombre del archivo de registro binario y el desplazamiento.

   Ejecute el comando [`show master status`](https://dev.mysql.com/doc/refman/5.7/en/show-master-status.html) para determinar el nombre de archivo de registro binario actual y el desplazamiento.

   ```sql
    show master status;
   ```
   Los resultados deberían parecerse a los siguientes. Asegúrese de anotar el nombre del archivo binario, pues se utilizará en pasos posteriores.

   :::image type="content" source="./media/how-to-data-in-replication/master-status.png" alt-text="Resultados de estado del maestro":::

## <a name="dump-and-restore-the-source-server"></a>Volcado y restauración del servidor de origen

1. Determine qué bases de datos y tablas quiere replicar en el servidor flexible de Azure Database for MySQL y realice el volcado de memoria desde el servidor de origen.

    Puede utilizar mysqldump para volcar bases de datos desde el servidor principal. Para más información, consulte [Volcado y restauración](../concepts-migrate-dump-restore.md). No es necesario volcar la biblioteca de MySQL y la biblioteca de prueba.


3. Establezca el servidor de origen en modo de lectura/escritura.

   Cuando se haya volcado la base de datos, cambie el servidor MySQL de origen de nuevo al modo de lectura/escritura.

   ```sql
   SET GLOBAL read_only = OFF;
   UNLOCK TABLES;
   ```

4. Restaure el archivo de volcado en el nuevo servidor.

   Restaure el archivo de volcado en el servidor creado en el servicio de servidor flexible de Azure Database for MySQL. Consulte [Volcado y restauración](../concepts-migrate-dump-restore.md) para saber cómo restaurar un archivo de volcado en un servidor MySQL. Si el archivo de volcado es grande, cárguelo a una máquina virtual en Azure dentro de la misma región que el servidor de réplica. Restáurelo en el servidor flexible de Azure Database for MySQL desde la máquina virtual.


## <a name="link-source-and-replica-servers-to-start-data-in-replication"></a>Vinculación de los servidores de origen y de réplica para iniciar la replicación de datos de entrada

1. Establezca el servidor de origen.

   Todas las funciones de la replicación de datos de entrada se realizan mediante los procedimientos almacenados. Puede encontrar todos los procedimientos en [Procedimientos almacenados de replicación de datos de entrada](../reference-stored-procedures.md). Los procedimientos almacenados se pueden ejecutar en el shell de MySQL o en MySQL Workbench.

   Para vincular dos servidores e iniciar la replicación, inicie sesión en el servidor de réplica de destino en el servicio Azure Database for MySQL y establezca la instancia externa como servidor de origen. Esto se realiza mediante el procedimiento almacenado `mysql.az_replication_change_master` en el servidor Azure Database for MySQL.

   ```sql
   CALL mysql.az_replication_change_master('<master_host>', '<master_user>', '<master_password>', <master_port>, '<master_log_file>', <master_log_pos>, '<master_ssl_ca>');
   ```

 
   - master_host: nombre de host del servidor de origen
   - master_user: nombre de usuario para el servidor de origen
   - master_password: contraseña para el servidor de origen
   - master_port: número de puerto en el que el servidor de origen escucha las conexiones (3306 es el puerto predeterminado en el que escucha MySQL).
   - master_log_file: nombre del archivo de registro binario procedente de la ejecución de `show master status`
   - master_log_pos: posición del registro binario procedente de la ejecución de `show master status`
   - master_ssl_ca: contexto del certificado de entidad de certificación. Si no usa SSL, pase una cadena vacía.

     Se recomienda pasar este parámetro como una variable. Para más información, vea los ejemplos siguientes.

   > [!NOTE]
   > Si el servidor de origen se hospeda en una máquina virtual de Azure, establezca la opción "Permitir el acceso a servicios de Azure" en "Activado" para permitir que los servidores de origen y de réplica se comuniquen entre sí. Esta configuración se puede cambiar desde las opciones de **seguridad de conexión**. Para obtener más información, consulte el artículo sobre cómo [administrar reglas de firewall mediante el portal](how-to-manage-firewall-portal.md).

   **Ejemplos**

   *Replicación con SSL*

   La variable `@cert` se crea mediante la ejecución de los siguientes comandos de MySQL:

      ```sql
      SET @cert = '-----BEGIN CERTIFICATE-----
      PLACE YOUR PUBLIC KEY CERTIFICATE'`S CONTEXT HERE
      -----END CERTIFICATE-----'
      ```

   La replicación con SSL se configura entre un servidor de origen hospedado en el dominio "companya.com" y un servidor de réplica hospedado en el servidor flexible de Azure Database for MySQL. Este procedimiento almacenado se ejecuta en la réplica.

      ```sql
      CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mysql-bin.000002', 120, @cert);
      ```

   *Replicación sin SSL*

   La replicación sin SSL se configura entre un servidor de origen hospedado en el dominio "companya.com" y un servidor de réplica hospedado en el servidor flexible de Azure Database for MySQL. Este procedimiento almacenado se ejecuta en la réplica.

      ```sql
      CALL mysql.az_replication_change_master('master.companya.com', 'syncuser', 'P@ssword!', 3306, 'mysql-bin.000002', 120, '');
      ```

2. Configure el filtrado.

   Si desea omitir la replicación de algunas tablas del maestro, actualice el parámetro de servidor `replicate_wild_ignore_table` en el servidor de réplica. Puede proporcionar más de un patrón de tabla mediante una lista separada por comas.

   Consulte la [documentación de MySQL](https://dev.mysql.com/doc/refman/8.0/en/replication-options-replica.html#option_mysqld_replicate-wild-ignore-table) para más información acerca de este parámetro.

   Para actualizar el parámetro, use [Azure Portal](how-to-configure-server-parameters-portal.md) o la [CLI de Azure](how-to-configure-server-parameters-cli.md).

3. Inicie la replicación.

   Llame al procedimiento almacenado `mysql.az_replication_start` para iniciar la replicación.

   ```sql
   CALL mysql.az_replication_start;
   ```

4. Compruebe el estado de replicación.

   Llame al comando [`show slave status`](https://dev.mysql.com/doc/refman/5.7/en/show-slave-status.html) en el servidor de réplica para ver el estado de replicación.

   ```sql
   show slave status;
   ```

   Si el estado de `Slave_IO_Running` y `Slave_SQL_Running` es "yes" y el valor de `Seconds_Behind_Master` es "0", la replicación funciona correctamente. `Seconds_Behind_Master` indica el tiempo de retraso de la réplica. Si el valor no es "0", significa que la réplica procesa actualizaciones.

## <a name="other-useful-stored-procedures-for-data-in-replication-operations"></a>Otros procedimientos almacenados útiles para las operaciones de la replicación de datos de entrada

### <a name="stop-replication"></a>Detención replicación

Para detener la replicación entre el servidor de origen y de réplica, use el siguiente procedimiento almacenado:

```sql
CALL mysql.az_replication_stop;
```

### <a name="remove-replication-relationship"></a>Eliminación de la relación de replicación

Para eliminar la relación entre el servidor de origen y de réplica, use el siguiente procedimiento almacenado:

```sql
CALL mysql.az_replication_remove_master;
```

### <a name="skip-replication-error"></a>Omisión de error de replicación

Para omitir un error de replicación y permitir que continúe la replicación, use el siguiente procedimiento almacenado:

```sql
CALL mysql.az_replication_skip_counter;
```

```sql
SHOW BINLOG EVENTS [IN 'log_name'] [FROM pos][LIMIT [offset,] row_count]
```

  :::image type="content" source="./media/how-to-data-in-replication/show-binary-log.png" alt-text="Mostrar los resultados del registro binario":::

## <a name="next-steps"></a>Pasos siguientes

- Obtenga más información sobre la [replicación de datos de entrada](concepts-data-in-replication.md) para el servidor flexible de Azure Database for MySQL.
