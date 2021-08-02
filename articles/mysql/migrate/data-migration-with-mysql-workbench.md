---
title: 'Guía de migración de MySQL local a Azure Database for MySQL: migración de datos con MySQL Workbench'
description: Siga todos los pasos de la guía de instalación para crear un entorno que admita los pasos siguientes.
ms.service: mysql
ms.subservice: migration-guide
ms.topic: how-to
author: arunkumarthiags
ms.author: arthiaga
ms.reviewer: maghan
ms.custom: ''
ms.date: 05/26/2021
ms.openlocfilehash: 24be41c768c808cb8d33a0fc74b2a950b6111797
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958647"
---
# <a name="mysql-on-premises-to-azure-database-for-mysql-migration-guide-data-migration-with-mysql-workbench"></a>Guía de migración de MySQL local a Azure Database for MySQL: migración de datos con MySQL Workbench

### <a name="setup"></a>Configurar

Siga todos los pasos de la guía de instalación para crear un entorno que admita los pasos siguientes.

### <a name="configuring-server-parameters-source"></a>Configuración de los parámetros de servidor (origen)

En función del tipo de migración que haya elegido (sin conexión o en línea), seguramente quiera evaluar si va a modificar los parámetros del servidor para permitir una salida rápida de los datos. Si trabaja en línea, puede que no sea necesario hacer nada en los parámetros del servidor, ya que es probable que realice una replicación `binlog` y que los datos se sincronicen por sí solos. Sin embargo, si va a realizar una migración sin conexión, una vez que detenga el tráfico de la aplicación, puede cambiar los parámetros del servidor para que pase de admitir la carga de trabajo a que admita la exportación.

### <a name="configuring-server-parameters-target"></a>Configuración de los parámetros del servidor (destino)

Revise los parámetros del servidor antes de iniciar el proceso de importación en Azure Database for MySQL. Los parámetros del servidor se pueden recuperar y establecer mediante [Azure Portal](../howto-server-parameters.md) o llamando a los [cmdlets de Azure PowerShell para MySQL](../howto-configure-server-parameters-using-powershell.md) para realizar los cambios.

Ejecute el siguiente script de PowerShell para obtener todos los parámetros:

```
\[Net.ServicePointManager\]::SecurityProtocol = \[Net.SecurityProtocolType\]::Tls
12

Install-Module -Name Az.MySql
Connect-AzAccount
$rgName = "{RESOURCE\_GROUP\_NAME}";
$serverName = "{SERVER\_NAME}";
Get-AzMySqlConfiguration -ResourceGroupName $rgName -ServerName $serverName
```

  - Para hacer lo mismo con la herramienta mysql, descargue la [certificación raíz de la entidad de certificación](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) en c:\\temp (crear este directorio).

    > [!NOTE] 
    > Recuerde que el certificado está sujeto a cambios. Para conocer la última información sobre los certificados, consulte [Configuración de la conectividad SSL en la aplicación para conectarse de forma segura a Azure Database for MySQL](../howto-configure-ssl.md).

  - Ejecute el siguiente código en un símbolo del sistema y asegúrese de actualizar los tokens:

```
mysql --host {servername}.mysql.database.azure.com --database mysql --user 
{u sername}@{servername} -p --ssl-ca=c:\\temp\\BaltimoreCyberTrustRoot.crt.cer
-A -e "SHOW GLOBAL VARIABLES;" \> c:\\temp\\settings\_azure.txt
```

En el nuevo archivo \`settings\_azure.txt\`, verá los parámetros de servidor predeterminados de Azure Database for MySQL, tal como se muestra en el apéndice A.

Para respaldar la migración, establezca los parámetros de la instancia de MySQL de destino para permitir una entrada más rápida. Se deben establecer los siguientes parámetros de servidor antes de iniciar la migración de datos:

  - `max\_allowed\_packet`: establezca el parámetro en `1073741824` (es decir, 1 GB) o el tamaño más grande de una fila de la base de datos para evitar problema de desbordamiento debido a filas largas. Considere la posibilidad de ajustar este parámetro si hay filas BLOB de gran tamaño que se deban extraer (o leer).

  - `innodb\_buffer\_pool\_size`: escale verticalmente el servidor a una SKU optimizada para memoria de 32 núcleos virtuales del plan de tarifa del portal durante la migración para aumentar el tamaño de innodb\_buffer\_pool\_size. El valor de innodb\_buffer\_pool\_size solo se puede aumentar escalando verticalmente el proceso del servidor de Azure Database for MySQL. Consulte [Parámetros de servidor de Azure Database for MySQL](../concepts-server-parameters.md#innodb_buffer_pool_size) para conocer el valor máximo del plan. El valor máximo de un sistema de 32 núcleos virtuales optimizados para memoria es `132070244352`.

  - `innodb\_io\_capacity` & `innodb\_io\_capacity\_max`: cambie el parámetro por `9000` para mejorar la utilización de E/S y así optimizar la velocidad de migración.

  - `max\_connections`: si usa una herramienta que genera varios subprocesos para aumentar el rendimiento, aumente las conexiones para admitir esa herramienta. El valor predeterminado es `151`; el valor máximo es `5000`.

    > [!NOTE] 
    > Tenga cuidado al realizar el escalado. Algunas operaciones no se pueden deshacer, como el escalado del almacenamiento.

Esta configuración se puede actualizar mediante estos cmdlets de Azure PowerShell:

```
Install-Module -Name Az.MySql
$rgName = " {RESOURCE\_GROUP\_NAME}";
$serverName = "{SERVER\_NAME}";

Select-AzSubscription -Subscription "{SUBSCRIPTION\_ID}"
Update-AzMySqlConfiguration -Name max\_allowed\_packet -ResourceGroupName
$rgna me -ServerName $serverName -Value 1073741824  
Update-AzMySqlConfiguration -Name innodb\_buffer\_pool\_size -ResourceGroupName 
$rgname -ServerName $serverName -Value 16106127360
Update-AzMySqlConfiguration -Name innodb\_io\_capacity -ResourceGroupName 
$rgna me -ServerName $serverName -Value 9000
Update-AzMySqlConfiguration -Name innodb\_io\_capacity\_max -ResourceGroupName 
$ rgname -ServerName $serverName -Value 9000

\#required if you have functions

Update-AzMySqlConfiguration -Name log\_bin\_trust\_function\_creators 
-ResourceGr oupName $rgname -ServerName $serverName -Value ON
```
### <a name="data"></a>data

#### <a name="tool-choice"></a>Elección de la herramienta

Una vez que se han migrado los objetos de base de datos y los usuarios del sistema de origen, la migración puede comenzar. Las bases de datos que se ejecutan en MySQL, versión 8.0 no pueden usar Azure DMS para migrar la carga de trabajo. En su lugar, los usuarios de migración deben utilizar MySQL Workbench.

#### <a name="manual-import-and-export-steps"></a>Pasos manuales de importación y exportación

  - Abra MySQL Workbench y conéctese como usuario raíz de la base de datos local.

  - En \*\*Administración\*\*, seleccione \*\*Exportación de datos\*\*. Seleccione el esquema **reg\_app**.

  - En **Objetos para exportar**, seleccione **Dump Stored Procedures and Functions** (Volcar procedimientos y funciones almacenados), **Dump Events** (Volcar eventos) y **Dump Triggers** (Volcar desencadenadores).

  - En **Export Options** (Opciones de exportación), seleccione **Export to Self-Contained File** (Exportar a archivo autocontenido).

  - Además, active la casilla **Include Create Schema** (Incluir esquema de creación). Consulte la imagen siguiente para observar la configuración correcta de mysqldump.

    ![Incluir esquema de creación](./media/image6.jpg)

    **Prueba**

  - Si alguna de estas opciones parece no estar disponible, es probable que el panel de salida las obstruya. Cambie solo el diseño del editor.

    ![diseño del editor](./media/image7.jpg)

    **Prueba**

  - Seleccione la pestaña **Exportar progreso**.

  - Seleccione **Iniciar exportación** y observe que MySQL Workbench realiza llamadas a la herramienta `mysqldump`.

  - Abra el script de exportación recién creado.

  - Busque instrucciones `DEFINER` y cambie a un usuario válido o quítelas por completo.

    > [!NOTE] 
    > Para ello, pase `--skip-definer` al comando mysqldump. Esta no es una opción de MySQL Workbench; por lo tanto, las líneas deberán quitarse manualmente en los comandos de exportación. Aunque aquí se señalan cuatro elementos para quitar, puede haber otros elementos que podrían producir un error al migrar de una versión de MySQL a otra (por ejemplo, nuevas palabras reservadas).

  - Busque instrucciones `SET GLOBAL` y cambie a un usuario válido o quítelas por completo.

  - Asegúrese de que `sql\_mode` no está establecido en `NO\_AUTO\_CREATE\_USER`.

  - Quite la función `hello\_world`.

  - En MySQL Workbench, cree una conexión a Azure Database for MySQL.

      - Como nombre de host, escriba el DNS completo del servidor (por ejemplo, `servername.mysql.database.azure.com`).

      - Escriba el nombre de usuario (por ejemplo, `sqlroot@servername`).

      - Seleccione la pestaña **SSL**.

      - Para encontrar el archivo de la entidad de certificación SSL, vaya al archivo de clave **BaltimoreCyberTrustRoot.crt.cer**.

      - Seleccione **Probar conexión** y asegúrese de que la conexión se completa.

      - Seleccione **Aceptar**.

        ![Cuadro de diálogo de conexión de MySQL](./media/image8.jpg)

        **Se muestra el cuadro de diálogo de conexión de MySQL.**

  - Seleccione **Archivo-\>Open SQL Script** (Abrir script de SQL).

  - Vaya al archivo de volcado y seleccione **Abrir**.

  - Seleccione **Execute**(Ejecutar).

### <a name="update-applications-to-support-ssl"></a>Actualización de aplicaciones para admitir SSL

  - Cambie a API de servidor de Java en Visual Studio Code.

  - Guarde el archivo **launch.json**.

  - Actualice **DB\_CONNECTION\_URL** a `jdbc:mysql://serverDNSname:3306/reg\_app?useUnicode=true\&useJDBCCompliantT imezoneShift=true\&useLegacyDatetimeCode=false\&serverTimezone=UTC\&verifySe rverCertificate=true\&useSSL=true\&requireSSL=true\&noAccessToProcedureBodie s=true.` y observe los parámetros SSL adicionales.

  - Actualice **DB\_USER\_NAME** a **conferenceuser@servername** .

  - Inicie la configuración de depuración y asegúrese de que la aplicación funciona localmente con la nueva base de datos.

### <a name="revert-server-parameters"></a>Reversión de los parámetros de servidor

Los parámetros siguientes se pueden cambiar en la instancia de destino de Azure Database for MySQL. Estos parámetros se pueden establecer mediante Azure Portal o los [cmdlets de Azure PowerShell para MySQL](../howto-configure-server-parameters-using-powershell.md).

```
$rgName = "YourRGName";
$serverName = "servername";
Update-AzMySqlConfiguration -Name max\_allowed\_packet -ResourceGroupName 
$rgna me -ServerName $serverName -Value 536870912
Update-AzMySqlConfiguration -Name innodb\_buffer\_pool\_size -ResourceGroupName 
$rgname -ServerName $serverName -Value 16106127360
Update-AzMySqlConfiguration -Name innodb\_io\_capacity -ResourceGroupName $rgna 
me -ServerName $serverName -Value 200
Update-AzMySqlConfiguration -Name innodb\_io\_capacity\_max -ResourceGroupName 
$ rgname -ServerName $serverName -Value 2000
```
### <a name="change-connection-string-for-the-java-api"></a>Cambio de la cadena de conexión para la API de Java

  - Use los siguientes comandos para cambiar la cadena de conexión de la API de Java de App Service.

```
$rgName = "YourRGName"; 
$app_name = "servername";
az webapp config appsettings set -g $rgName -n $app_name 
--settings DB_CONNECTION_URL={DB_CONNECTION_URL}
```

> [!NOTE] 
> Recuerde que puede usar el portal para establecer la cadena de conexión.

  - Reinicie la API de App Service.

```
az webapp restart -g $rgName -n $app\_name
```
Ha completado correctamente una migración del entorno local a Azure Database for MySQL\!.  


> [!div class="nextstepaction"]  
> [Administración posterior a la migración](./post-migration-management.md)