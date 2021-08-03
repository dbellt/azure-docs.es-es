---
title: 'Solución de problemas de migración de Hive de 3.6 a 4.0: Azure HDInsight'
description: Guía de solución de problemas para la migración de cargas de trabajo de Hive desde HDInsight 3.6 a 4.0
ms.service: hdinsight
ms.topic: troubleshooting
ms.date: 05/24/2021
ms.openlocfilehash: 55927e15db913cde5feaa82e2f4dd6b71c800d56
ms.sourcegitcommit: bb9a6c6e9e07e6011bb6c386003573db5c1a4810
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110501378"
---
# <a name="troubleshooting-guide-for-migration-of-hive-workloads-from-hdinsight-36-to-hdinsight-40"></a>Guía de solución de problemas para la migración de cargas de trabajo de Hive desde HDInsight 3.6 a HDInsight 4.0

En este artículo se proporcionan respuestas a algunos de los problemas más comunes a los que se enfrentan los clientes al migrar cargas de trabajo de Hive de HDInsight 3.6 a HDInsight 4.0.

## <a name="change-parquet-files-to-store-timestamp-in-utc-time-zone"></a>Cambio de los archivos de Parquet para almacenar la marca de tiempo en la zona horaria UTC

Solución:
1. Use ssh en los nodos principales y haga una copia de seguridad de los archivos jar subyacentes por si tuviera que revertir. No mantenga la copia de seguridad en la misma ruta de acceso.
    `/usr/hdp/4.1.2.5/hive/lib/hive-common-3.1.2.4.1.2.5.jar` y `hive-exec-3.1.2.4.1.2.5.jar`

2. Descargue los dos archivos jar.
    - [https://wenjmshare.blob.core.windows.net/public/hive-common-3.1.2.4.1.2.5.jar](https://wenjmshare.blob.core.windows.net/public/hive-common-3.1.2.4.1.2.5.jar) 
    - [https://wenjmshare.blob.core.windows.net/public/hive-exec-3.1.2.4.1.2.5.jar](https://wenjmshare.blob.core.windows.net/public/hive-exec-3.1.2.4.1.2.5.jar)

3. Mueva los archivos jar descargados a la ubicación `/usr/hdp/4.1.2.5/hive/lib` en ambos nodos principales.

4. Reinicie HS2 en la interfaz de usuario de Ambari.


## <a name="reduce-latency-when-running-describe-table_name"></a>Reduzca la latencia al ejecutar `DESCRIBE TABLE_NAME`.

Solución:
* Aumente el número máximo de objetos (tablas o particiones) que se pueden recuperar del metastore en un lote. Establézcalo en un número grande (el valor predeterminado es 300) hasta que se alcancen los niveles de latencia satisfactorios. Cuanto mayor sea el número, menos recorridos de ida y vuelta se necesitan en el servidor de metastore de Hive, si bien también puede provocar un mayor requisito de memoria en el lado cliente.

    ```hive.metastore.batch.retrieve.max=2000```
* Reinicio de Hive y todos los servicios obsoletos.

## <a name="unable-to-query-gzipped-text-file-if-skipheaderlinecount-and-skipfooterlinecount-are-set-for-table"></a>No se puede consultar el archivo de texto Gzipped si skip.header.line.count y skip.footer.line.count están configurados para la tabla.

El problema se ha corregido en Interactive Query 4.0, pero aún persiste en Interactive Query 3.1.0.

Solución:
* Cree una tabla sin usar ```"skip.header.line.count"="1"``` y ```"skip.footer.line.count"="1"```, luego cree una vista de la tabla original que excluya la fila del encabezado/pie en la consulta.

## <a name="unable-to-use-unicode-characters"></a>No se pueden usar caracteres Unicode.

Solución:
1. Conéctese a la base de datos del metastore de Hive del clúster.

2. Realice una copia de seguridad de la tabla `TBLS` y `TABLE_PARAMS` con el siguiente comando:
    ```sql
        select * into tbls_bak from tbls;
        select * into table_params_bak from table_params;
    ```

3. Cambie manualmente los tipos de columna afectados a `nvarchar(max)`.
    ```sql 
        alter table TABLE_PARAMS alter column PARAM_VALUE nvarchar(max);
        alter table TBLS alter column VIEW_EXPANDED_TEXT nvarchar(max) null;    
        alter table TBLS alter column VIEW_ORIGINAL_TEXT nvarchar(max) null;
    ```

## <a name="create-table-as-select-ctas-creates-a-new-table-with-same-uuid"></a>Create table as select (CTAS) crea una nueva tabla con el mismo UUID.

Hive 3.1 (HDInsight 4.0) ofrece una UDF integrada para generar UUID únicos. El método UUID() de Hive genera identificadores únicos incluso con CTAS. Puede aprovecharlo de la siguiente manera.
```hql
create table rhive as
select uuid() as UUID
from uuid_test
```

## <a name="hive-job-output-format-differs-from-hdinsight-36"></a>El formato de salida del trabajo de Hive difiere de HDInsight 3.6.

Se debe a la diferencia de WebHCat(Templeton) entre HDInsight 3.6 y HDInsight 4.0.

* API REST de Hive: agregue ```arg=--showHeader=false -d arg=--outputformat=tsv2 -d```.

* SDK de .NET: inicialice argumentos de ```HiveJobSubmissionParameters```.
    ```csharp
    List<string> args = new List<string> { { "--showHeader=false" }, { "--outputformat=tsv2" } };
                var parameters = new HiveJobSubmissionParameters
                {
                    Query = "SELECT clientid,market from hivesampletable LIMIT 10",
                    Defines = defines,
                    Arguments = args
                };
    ```

## <a name="reduce-hive-internal-table-creation-latency"></a>Reducción de la latencia de creación de tablas internas de Hive

1. En el sitio de Hive avanzado y el sitio de Hivemetastore avanzado, elimine el valor ```org.apache.hive.hcatalog.listener.DbNotificationListener``` de ```hive.metastore.transactional.event.listeners```. 

2. Si ```hive.metastore.event.listeners``` tiene un valor, quítelo.

3. DbNotificationListener solo es necesario si usa comandos REPL y, si no es así, es seguro quitarlo.

    :::image type="content" source="./media/apache-hive-40-migration-guide/hive-reduce-internal-table-creation-latency.png" alt-text="Reducción de la latencia de tabla interna en HDInsight 4.0" border="true":::

## <a name="change-hive-default-table-location"></a>Cambio de la ubicación predeterminada de la tabla de Hive

Se trata de un cambio de comportamiento por diseño en HDInsight 4.0 (Hive 3.1). Este cambio está motivado por el control de permisos de archivo. 

Para crear tablas externas en una ubicación personalizada, especifique la ubicación en la instrucción create table.

## <a name="disable-acid-in-hdinsight-40"></a>Deshabilitación de ACID en HDInsight 4.0

Se recomienda habilitar ACID en HDInsight 4.0, ya que la mayoría de las mejoras recientes (tanto funcionales como de rendimiento) en Hive están disponibles solo para las tablas ACID.

Pasos para deshabilitar ACID en HDInsight 4.0:
1. Cambie las siguientes configuraciones de Hive en Ambari:

    ```text
    hive.strict.managed.tables=false
    hive.support.concurrency=false; 
    hive.txn.manager=org.apache.hadoop.hive.ql.lockmgr.DummyTxnManager;
    hive.enforce.bucketing=false;
    hive.compactor.initiator.on=false;
    hive.compactor.worker.threads=0;
    hive.create.as.insert.only=false;
    metastore.create.as.acid=false;
    ```

2. Reinicie el servicio de Hive.

> [!IMPORTANT]
> Microsoft recomienda no compartir los mismos datos o almacenamiento con las tablas administradas por Hive de HDInsight 3.6 y HDInsight 4.0. Es un escenario no admitido.

* Normalmente, las configuraciones anteriores deben establecerse incluso antes de crear tablas de Hive en el clúster de HDInsight 4.0. No debemos deshabilitar ACID una vez creadas las tablas administradas. Podría provocar la pérdida de datos o resultados incoherentes. Por lo tanto, se recomienda establecerlo una vez al crear un nuevo clúster y no cambiarlo más adelante.

* La deshabilitación de ACID después de crear tablas es arriesgado; sin embargo, en caso de que quiera hacerlo, siga estos pasos para evitar posibles pérdidas de datos o incoherencias:

    1. Cree una tabla externa con el mismo esquema y copie los datos de la tabla administrada original mediante el comando CTAS ```create external table e_t1 select * from m_t1```.    
    2. Quite la tabla administrada mediante ```drop table m_t1```.
    3. Deshabilite ACID mediante las configuraciones sugeridas.        
    4. Cree m_t1 y copie los datos de la tabla externa mediante el comando CTAS ```create table m_t1 select * from e_t1```.        
    5. Quite la tabla externa mediante ```drop table e_t1```.

Asegúrese de que todas las tablas administradas se convierten en tablas externas y se descartan antes de deshabilitar ACID. Además, compare el esquema y los datos después de cada paso para evitar cualquier discrepancia.

## <a name="create-hive-external-table-with-755-permission"></a>Creación de una tabla externa de Hive con permiso 755

Este problema se puede resolver mediante cualquiera de las dos opciones siguientes:

1. Establezca manualmente el permiso de carpeta en 757 o 777 para permitir que el usuario de Hive escriba en el directorio.

2. Cambie el "Administrador de autorización de Hive" de ```org.apache.hadoop.hive.ql.security.authorization.StorageBasedAuthorizationProvider``` a ```org.apache.hadoop.hive.ql.security.authorization.MetaStoreAuthzAPIAuthorizerEmbedOnly```.

MetaStoreAuthzAPIAuthorizerEmbedOnly deshabilita eficazmente las comprobaciones de seguridad porque el metastore de Hive no está insertado en HDInsight 4.0. Sin embargo, esto puede traer otros posibles problemas. Tenga cuidado al usar esta opción.

## <a name="permission-errors-in-hive-job-after-upgrading-to-hdinsight-40"></a>Errores de permisos en el trabajo de Hive después de actualizar a HDInsight 4.0

* En HDInsight 4.0, todas las formas de clúster con componentes de Hive se configuran con un nuevo proveedor de autorización: ```org.apache.hadoop.hive.ql.security.authorization.StorageBasedAuthorizationProvider```.

* Se deben asignar permisos de archivo HDFS al usuario de Hive para el archivo al que se accede. El mensaje de error proporciona los detalles necesarios para resolver el problema.

* También puede cambiar al proveedor ```MetaStoreAuthzAPIAuthorizerEmbedOnly``` que se usa en clústeres de Hive de HDInsight 3.6.
```org.apache.hadoop.hive.ql.security.authorization.MetaStoreAuthzAPIAuthorizerEmbedOnly```

    :::image type="content" source="./media/apache-hive-40-migration-guide/hive-job-permission-errors.png" alt-text="Establezca la autorización en MetaStoreAuthzAPIAuthorizerEmbedOnly" border="true":::.

## <a name="next-steps"></a>Pasos siguientes

[!INCLUDE [troubleshooting next steps](../../../includes/hdinsight-troubleshooting-next-steps.md)]