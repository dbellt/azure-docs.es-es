---
title: 'Solución de errores comunes: Azure Database for MySQL'
description: Aprenda a solucionar errores comunes de migración que encuentran los usuarios nuevos en el servicio Azure Database for MySQL.
author: savjani
ms.service: mysql
ms.author: pariks
ms.custom: mvc
ms.topic: overview
ms.date: 5/21/2021
ms.openlocfilehash: d095eddf150990ac5ab76f0753cae2bc44537c88
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110471817"
---
# <a name="commonly-encountered-errors-during-or-post-migration-to-azure-database-for-mysql"></a>Errores que se detectan normalmente durante la migración al servicio Azure Database for MySQL o después de esta.

[!INCLUDE[applies-to-single-flexible-server](includes/applies-to-single-flexible-server.md)]

Azure Database for MySQL es un servicio totalmente administrado que usa la tecnología de la versión de comunidad de MySQL. La experiencia de MySQL en un entorno de servicio administrado puede diferir de ejecutarlo en su propio entorno. En este artículo, verá algunos de los errores habituales que pueden encontrar los usuarios la primera vez que migran a Azure Database for MySQL, o que desarrollan en dicho servicio.

## <a name="common-connection-errors"></a>Errores de conexión comunes

#### <a name="error-1184-08s01-aborted-connection-22-to-db-db-name-user-user-host-hostip-init_connect-command-failed"></a>ERROR 1184 (08S01): Aborted connection 22 to db: 'db-name' user: 'user' host: 'hostIP' (init_connect command failed)

El error anterior se produce después de un inicio de sesión correcto, pero antes de que se ejecute cualquier comando cuando se establece la sesión. El mensaje anterior indica que ha establecido un valor incorrecto del parámetro de servidor `init_connect`, que provoca un error en la inicialización de la sesión.

Hay algunos parámetros de servidor, como `require_secure_transport`, que no se admiten en el nivel de sesión y, por consiguiente, intentar cambiar los valores de estos parámetros mediante `init_connect` puede generar el error 1184 al conectarse al servidor MySQL, como se muestra a continuación:

mysql> show databases; ERROR 2006 (HY000): MySQL server has gone away No connection. Trying to reconnect... Connection id:    64897 Current database: *** NONE ***_ ERROR 1184 (08S01): Aborted connection 22 to db: 'db-name' user: 'user' host: 'hostIP' (init_connect command failed)

**Solución**: restablezca el valor `init_connect` en la pestaña Parámetros del servidor de Azure Portal y establezca solo los parámetros de servidor admitidos mediante el parámetro init_connect.

## <a name="errors-due-to-lack-of-super-privilege-and-dba-role"></a>Errores debidos a la falta del privilegio SUPER y del rol DBA

El privilegio SUPER y el rol DBA no se admiten en el servicio. En consecuencia, pueden producirse algunos errores comunes que se enumeran a continuación:

#### <a name="error-1419-you-do-not-have-the-super-privilege-and-binary-logging-is-enabled-you-might-want-to-use-the-less-safe-log_bin_trust_function_creators-variable"></a>ERROR 1419: You do not have the SUPER privilege and binary logging is enabled (you *might* want to use the less safe log_bin_trust_function_creators variable) (No tiene el privilegio SUPER y el registro binario está habilitado [puede] usar la variable log_bin_trust_function_creators, pero es menos segura)

El error anterior puede aparecer al crear una función, al desencadenar como se muestra a continuación o al importar un esquema. Las instrucciones del DDL como CREATE FUNCTION o CREATE TRIGGER se escriben en el registro binario, por lo que la réplica secundaria puede ejecutarlas. El subproceso de réplica de SQL tiene privilegios totales, lo que se puede aprovechar para elevar los privilegios. Para protegerse frente a este riesgo en los servidores que tienen habilitado el registro binario, el motor de MySQL requiere que los creadores de funciones almacenadas tengan el privilegio SUPER, además del privilegio CREATE ROUTINE habitual.

```sql
CREATE FUNCTION f1(i INT)
RETURNS INT
DETERMINISTIC
READS SQL DATA
BEGIN
  RETURN i;
END;
```

**Solución**: para resolver el error, establezca `log_bin_trust_function_creators` en 1 en la hoja [Parámetros del servidor](howto-server-parameters.md) del portal, ejecute las instrucciones de DDL o importe el esquema para crear los objetos deseados. Puede seguir manteniendo en `log_bin_trust_function_creators` 1 para el servidor a fin de evitar el error en el futuro. Se recomienda establecer `log_bin_trust_function_creators`, ya que el riesgo de seguridad resaltado en la [documentación de la comunidad de MySQL](https://dev.mysql.com/doc/refman/5.7/en/replication-options-binary-log.html#sysvar_log_bin_trust_function_creators) es mínimo en el servicio Azure Database for MySQL porque el registro de bin no está expuesto a ninguna amenaza.

#### <a name="error-1227-42000-at-line-101-access-denied-you-need-at-least-one-of-the-super-privileges-for-this-operation-operation-failed-with-exitcode-1"></a>ERROR 1227 (42000) en la línea 101: Access denied; you need (at least one of) the SUPER privilege(s) for this operation [Acceso denegado; necesita (al menos uno de) los privilegios SUPER para esta operación]. Error en la operación, exitcode 1

El error anterior puede producirse tanto al importar un archivo de volcado como al crear un procedimiento que contenga [depuradores](https://dev.mysql.com/doc/refman/5.7/en/create-procedure.html).

**Solución:**  para resolver este error, el usuario administrador puede conceder privilegios para crear o ejecutar procedimientos mediante la ejecución del comando GRANT, como en los ejemplos siguientes:

```sql
GRANT CREATE ROUTINE ON mydb.* TO 'someuser'@'somehost';
GRANT EXECUTE ON PROCEDURE mydb.myproc TO 'someuser'@'somehost';
```

Como alternativa, puede reemplazar los depuradores por el nombre del usuario administrador que ejecuta el proceso de importación, como se muestra a continuación.

```sql
DELIMITER;;
/*!50003 CREATE*/ /*!50017 DEFINER=`root`@`127.0.0.1`*/ /*!50003
DELIMITER;;

/* Modified to */

DELIMITER ;;
/*!50003 CREATE*/ /*!50017 DEFINER=`AdminUserName`@`ServerName`*/ /*!50003
DELIMITER ;
```

#### <a name="error-1227-42000-at-line-295-access-denied-you-need-at-least-one-of-the-super-or-set_user_id-privileges-for-this-operation"></a>ERROR 1227 (42000) en la línea 295: Access denied; you need (at least one of) the SUPER privilege(s) for this operation [Acceso denegado; necesita (al menos uno de) los privilegios SUPER o SET_USER_ID para esta operación].

El error anterior puede producirse al ejecutar instrucciones CREATE VIEW con DEFINER como parte de la importación de un archivo de copia de seguridad o la ejecución de un script. Azure Database for MySQL no permite los privilegios SUPER ni el privilegio SET_USER_ID a ningún usuario.

**Solución**:

* Use el usuario del definidor para ejecutar CREATE VIEW, en caso de que sea posible. Es probable que haya muchas vistas con distintos depuradores que tengan permisos diferentes, por lo que es posible que esto no sea factible.  O
* Edite el archivo de copia de seguridad o el script CREATE VIEW y elimine la instrucción DEFINER= del archivo, o bien 
* Edite el archivo de copia de seguridad o el script CREATE VIEW y reemplace los valores del definidor por el usuario con permisos de administrador que realiza la importación o ejecute el archivo de script.

> [!Tip]
> Use sed o Perl para modificar un archivo de copia de seguridad o un script SQL para reemplazar la instrucción DEFINER=

#### <a name="error-1227-42000-at-line-18-access-denied-you-need-at-least-one-of-the-super-privileges-for-this-operation"></a>ERROR 1227 (42000) en la línea 18: Access denied; you need (at least one of) the SUPER privilege(s) for this operation [Acceso denegado; necesita (al menos uno de) los privilegios SUPER para esta operación].

El error anterior puede producirse si está intentando importar el archivo de volcado desde el servidor de MySQL con GTID habilitado en el servidor de destino de Azure Database for MySQL. Mysqldump agrega la instrucción SET @@SESSION.sql_log_bin=0 a un archivo de volcado de memoria desde un servidor donde GTID está en uso, lo que deshabilita el registro binario mientras se recarga el archivo de volcado de memoria.

**Solución**: para resolver este error durante la importación, quite o comente las líneas siguientes del archivo mysqldump y vuelva a ejecutar la importación para asegurarse de que se ha realizado correctamente.

SET @MYSQLDUMP_TEMP_LOG_BIN = @@SESSION.SQL_LOG_BIN; SET @@SESSION.SQL_LOG_BIN= 0; SET @@GLOBAL.GTID_PURGED=''; SET @@SESSION.SQL_LOG_BIN = @MYSQLDUMP_TEMP_LOG_BIN;

## <a name="common-connection-errors-for-server-admin-sign-in"></a>Errores de conexión comunes en el inicio de sesión de administrador del servidor

Cuando se crea un servidor de Azure Database for MySQL, el usuario final proporciona un inicio de sesión de administrador del servidor durante la operación. El inicio de sesión de administrador del servidor permite crear bases de datos, agregar nuevos usuarios y conceder permisos. Si se elimina el inicio de sesión de administrador del servidor, se revocan sus permisos o se cambia su contraseña, es posible que empiece a ver errores de conexión en la aplicación durante las conexiones. Estos son algunos de los errores comunes:

#### <a name="error-1045-28000-access-denied-for-user-usernameip-address-using-password-yes"></a>ERROR 1045 (28000): Access denied for user 'username'@'IP address' (using password: SÍ) [Acceso denegado para el usuario "nombre de usuario"@"dirección IP" (usa contraseña: SÍ)].

El error anterior se produce si:

* El nombre de usuario no existe.
* El nombre de usuario se ha eliminado.
* Su contraseña se ha cambiado o restablecido.

**Solución**:

* Compruebe si "nombre de usuario" existe como usuario válido en el servidor o si se elimina accidentalmente. Puede ejecutar la consulta siguiente iniciando sesión en el usuario de Azure Database for MySQL:

  ```sql
  select user from mysql.user;
  ```

* Si no puede iniciar sesión en MySQL para ejecutar la consulta anterior, es aconsejable que [restablezca la contraseña de administrador desde Azure Portal](howto-create-manage-server-portal.md). La opción de restablecer la contraseña desde Azure Portal le ayudará a volver a crear el usuario, restablecer la contraseña y restaurar los permisos de administrador, lo que le permitirá iniciar sesión con el administrador del servidor y realizar otras operaciones.

## <a name="next-steps"></a>Pasos siguientes

Si no encontró la respuesta que estaba buscando, tenga en cuenta las siguientes opciones:

* Publique su pregunta en la [página de preguntas y respuestas de Microsoft](/answers/topics/azure-database-mysql.html) o en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-database-mysql).
* Envíe un correo electrónico al equipo de Azure Database for MySQL [@Ask Azure DB for MySQL](mailto:AskAzureDBforMySQL@service.microsoft.com). Esta dirección de correo electrónico no es un alias de soporte técnico.
* Póngase en contacto con el soporte técnico de Azure, [presente una incidencia de soporte técnico en Azure Portal](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade). Para corregir un problema con la cuenta, envíe una [solicitud de soporte técnico](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) en Azure Portal.
* Para proporcionar comentarios o solicitar nuevas características, cree una entrada mediante [UserVoice](https://feedback.azure.com/forums/597982-azure-database-for-mysql).
