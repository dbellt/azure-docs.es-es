---
title: Creación un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde la CLI
description: Creación un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc desde la CLI
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 06/02/2021
ms.topic: how-to
ms.openlocfilehash: d5e9b449aaff6bb14283184c2182d0e9de2ef0c5
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111407568"
---
# <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Creación de un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

En este documento se describen los pasos para crear un grupo de servidores Hiperescala de PostgreSQL en Azure Arc.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="getting-started"></a>Introducción
Si ya está familiarizado con los temas siguientes, puede omitir este párrafo.
Hay temas importantes que puede que le interese leer antes de continuar con la creación:
- [Información general sobre los servicios de datos habilitados para Azure Arc](overview.md)
- [Modos de conectividad y requisitos](connectivity.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

Si prefiere probar las cosas sin aprovisionar un entorno completo por su cuenta, empiece a trabajar rápidamente con [Azure Arc JumpStart](https://azurearcjumpstart.io/azure_arc_jumpstart/azure_arc_data/) en Azure Kubernetes Service (AKS), AWS Elastic Kubernetes Service (EKS), Google Cloud Kubernetes Engine (GKE) o en una máquina virtual de Azure.


## <a name="login-to-the-azure-arc-data-controller"></a>Inicio de sesión en el controlador de datos de Azure Arc

Antes de poder crear una instancia, primero debe iniciar sesión en el controlador de datos de Azure Arc. Si ya ha iniciado sesión en el controlador de datos, puede omitir este paso.

```console
azdata login
```

Después, se le pedirá el nombre de usuario, la contraseña y el espacio de nombres del sistema.  

> Si ha usado el script para crear el controlador de datos, el espacio de nombres debe ser **arc**.

```console
Namespace: arc
Username: arcadmin
Password:
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Paso preliminar y temporal solo para usuarios de OpenShift
Implemente este paso antes de pasar al siguiente. Para implementar el grupo de servidores Hiperescala de PostgreSQL en Red Hat OpenShift en un proyecto distinto del predeterminado, debe ejecutar los comandos siguientes en el clúster para actualizar las restricciones de seguridad. Este comando concede los privilegios necesarios a las cuentas de servicio que ejecutarán el grupo de servidores Hiperescala de PostgreSQL. La restricción de contexto de seguridad (SCC) arc-data-scc es la que ha agregado al implementar el controlador de datos de Azure Arc.

```Console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

**Server-group-name es el nombre del grupo de servidores que se va a crear en el siguiente paso.**

Para obtener más detalles sobre las SCC en OpenShift, consulte la [documentación de OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html). Ahora puede implementar el paso siguiente.


## <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Creación de un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

Para crear un grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc en el controlador de datos de Arc, usará el comando `azdata arc postgres server create` al que pasará varios parámetros.

Para más información sobre todos los parámetros que se pueden establecer en el momento de la creación, revise la salida del comando:
```console
azdata arc postgres server create --help
```

Los parámetros principales que debe tener en cuenta son:
- **el nombre del grupo de servidores** que quiere implementar; Indique `--name` o `-n` seguido de un nombre cuya longitud no debe superar los 11 caracteres.

- **la versión del motor de PostgreSQL** que quiere implementar: de forma predeterminada, es la versión 12; Para implementar la versión 12, puede omitir este parámetro o pasar uno de los siguientes parámetros: `--engine-version 12` o `-ev 12`. Para implementar la versión 11, indique `--engine-version 11` o `-ev 11`.

- **el número de nodos de trabajo** que quiere implementar para escalar horizontalmente y lograr mejores rendimientos; Antes de continuar, lea los [conceptos sobre Hiperescala de Postgres](concepts-distributed-postgres-hyperscale.md). Para indicar el número de nodos de trabajo que se van a implementar, use el parámetro `--workers` o `-w` seguido de un entero. En la tabla siguiente se indica el intervalo de valores admitidos y qué forma de implementación de Postgres obtiene con ellos. Por ejemplo, si quiere implementar un grupo de servidores con 2 nodos de trabajo, indique `--workers 2` o `-w 2`. Se crean tres pods, uno para el nodo o la instancia de coordinación y dos para los nodos o las instancias de trabajo (uno para cada uno de los trabajos).



|Necesita:   |Forma del grupo de servidores que se va a implementar   |Parámetro -w que se usará   |Nota   |
|---|---|---|---|
|Una forma de Postgres de escalado horizontal para satisfacer las necesidades de escalabilidad de las aplicaciones.   |3 o más instancias de Postgres, 1 es el coordinador, n son los trabajos con n >=2.   |Use -w n, con n>=2.   |Se carga la extensión Citus que proporciona la funcionalidad de Hiperescala.   |
|Una forma básica de Hiperescala de Postgres para que realice la validación funcional de la aplicación con un coste mínimo. No es válido para la validación del rendimiento y la escalabilidad. Para ello, debe usar el tipo de implementaciones descritas anteriormente.   |1 instancia de Postgres que es el coordinador y el trabajo.   |Use -w 0 y cargue la extensión Citus. Use los parámetros siguientes si realiza la implementación desde la línea de comandos: -w 0 --extensions Citus.   |Se carga la extensión Citus que proporciona la funcionalidad de Hiperescala.   |
|Una instancia sencilla de Postgres que está lista para escalar horizontalmente cuando la necesite.   |1 instancia de Postgres. Todavía no es consciente de la semántica para el coordinador y el trabajo. Para escalarla horizontalmente después de la implementación, edite la configuración, aumente el número de nodos de trabajo y distribuya los datos.   |Use -w 0 o no especifique -w.   |La extensión Citus que proporciona la funcionalidad Hiperescala está presente en la implementación, pero aún no está cargada.   |
|   |   |   |   |

Aunque el uso de -w 1 funciona, no se recomienda usarlo. Esta implementación no le proporcionará mucho valor. Con ella, se obtienen 2 instancias de Postgres: 1 coordinador y 1 trabajo. Con esta configuración, los datos no se escalan horizontalmente en realidad, ya que se implementa un único trabajo. Por lo tanto, no verá un aumento del nivel de rendimiento y escalabilidad. Quitaremos la compatibilidad de esta implementación en una versión futura.

- **las clases de almacenamiento** que quiere que use el grupo de servidores. Es importante que establezca la clase de almacenamiento justo en el momento de implementar un grupo de servidores, ya que no se puede cambiar después. Si va a cambiar la clase de almacenamiento después de la implementación, deberá extraer los datos, eliminar el grupo de servidores, crear otro grupo de servidores e importar los datos. Puede especificar las clases de almacenamiento que se usarán para los datos, los registros y las copias de seguridad. De forma predeterminada, si no indica las clases de almacenamiento, se usarán las clases de almacenamiento del controlador de datos.
    - Para establecer la clase de almacenamiento para los datos, indique el parámetro `--storage-class-data` o `-scd` seguido del nombre de la clase de almacenamiento.
    - Para establecer la clase de almacenamiento para los registros, indique el parámetro `--storage-class-logs` o `-scl` seguido del nombre de la clase de almacenamiento.
    - Para establecer la clase de almacenamiento para las copias de seguridad: en esta versión preliminar de Hiperescala de PostgreSQL habilitada para Azure Arc hay dos maneras de establecer las clases de almacenamiento en función de los tipos de operaciones de copia de seguridad o restauración que quiera realizar. Estamos trabajando para simplificar esta experiencia. Indicará una clase de almacenamiento o un montaje de notificación de volumen. Un montaje de notificación de volumen es un par formado por una notificación de volumen persistente existente (en el mismo espacio de nombres) y el tipo de volumen (y metadatos opcionales según el tipo de volumen) separados por dos puntos. El volumen persistente se montará en cada pod del grupo de servidores de PostgreSQL.
        - Si quiere planear solo restauraciones completas de bases de datos, establezca el parámetro `--storage-class-backups` o `-scb` seguido del nombre de la clase de almacenamiento.
        - Si planea realizar restauraciones completas de la base de datos y restauraciones a un momento dado, establezca el parámetro `--volume-claim-mounts` o `-vcm` seguido del nombre de una notificación de volumen y un tipo de volumen.

Tenga en cuenta que, al ejecutar el comando de creación, se le pedirá que escriba la contraseña del usuario administrativo predeterminado, `postgres`. El nombre de ese usuario no se puede cambiar en esta versión preliminar. Para omitir el aviso interactivo, establezca la variable de entorno de sesión `AZDATA_PASSWORD` antes de ejecutar el comando create.

### <a name="examples"></a>Ejemplos

**Para implementar un grupo de servidores de la versión 12 de Postgres denominado postgres01 con 2 nodos de trabajo que emplea las mismas clases de almacenamiento que el controlador de datos, ejecute el siguiente comando:**
```console
azdata arc postgres server create -n postgres01 --workers 2
```

**Para implementar un grupo de servidores de la versión 12 de Postgres denominado postgres01 con 2 nodos de trabajo que emplea las mismas clases de almacenamiento que el controlador de datos para datos y registros, pero su clase de almacenamiento específica para realizar restauraciones completas y restauraciones a un momento dado, siga estos pasos:**

 En este ejemplo se da por hecho que el grupo de servidores está hospedado en un clúster de Azure Kubernetes Service (AKS). En este ejemplo se usa azurefile-premium como nombre de la clase de almacenamiento. Puede ajustar el ejemplo siguiente para que coincida con su entorno. Tenga en cuenta que **AccessModes ReadWriteMany es necesario** para esta configuración.  

En primer lugar, cree un archivo YAML que contenga la descripción siguiente del PVC de copia de seguridad y asígnele el nombre CreateBackupPVC.yml, por ejemplo:
```console
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: backup-pvc
  namespace: arc
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  resources:
    requests:
      storage: 100Gi
  storageClassName: azurefile-premium
```

Luego, cree un PVC con la definición almacenada en el archivo YAML:

```console
kubectl create -f e:\CreateBackupPVC.yml -n arc
``` 

Seguidamente, cree el grupo de servidores:

```console
azdata arc postgres server create -n postgres01 --workers 2 -vcm backup-pvc:backup
```

> [!IMPORTANT]
> - Lea las [limitaciones actuales relacionadas con la copia de seguridad y la restauración](limitations-postgresql-hyperscale.md#backup-and-restore).


> [!NOTE]  
> - Si ha implementado el controlador de datos con las variables de entorno de sesión `AZDATA_USERNAME` y `AZDATA_PASSWORD` en la misma sesión de terminal, los valores de `AZDATA_PASSWORD` también se usarán para implementar el grupo de servidores de Hiperescala de PostgreSQL. Si prefiere usar otra contraseña, (1) actualice el valor de `AZDATA_PASSWORD`, (2) elimine la variable de entorno `AZDATA_PASSWORD` o (3) elimine su valor; se le pedirá que escriba una contraseña de forma interactiva al crear un grupo de servidores.
> - La creación de un grupo de servidores Hiperescala de PostgreSQL no registrará los recursos en Azure de forma inmediata. Como parte del proceso de carga de [inventario de recursos](upload-metrics-and-logs-to-azure-monitor.md) o [datos de uso](view-billing-data-in-azure.md) a Azure, los recursos se crearán en Azure y podrá verlos en Azure Portal.



## <a name="list-the-postgresql-hyperscale-server-groups-deployed-in-your-arc-data-controller"></a>Enumeración de los grupos de servidores de Hiperescala de PostgreSQL implementados en el controlador de datos de Arc

Para enumerar los grupos de servidores de Hiperescala de PostgreSQL implementados en el controlador de datos de Arc, ejecute el siguiente comando:

```console
azdata arc postgres server list
```


```output
Name        State     Workers
----------  --------  ---------
postgres01  Ready     2
```

## <a name="get-the-endpoints-to-connect-to-your-azure-arc-enabled-postgresql-hyperscale-server-groups"></a>Obtención de los puntos de conexión para conectarse a los grupos de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc

Para ver los puntos de conexión de un grupo de servidores de PostgreSQL, ejecute el comando siguiente:

```console
azdata arc postgres endpoint list -n <server group name>
```
Por ejemplo:
```console
[
  {
    "Description": "PostgreSQL Instance",
    "Endpoint": "postgresql://postgres:<replace with password>@12.345.123.456:1234"
  },
  {
    "Description": "Log Search Dashboard",
    "Endpoint": "https://12.345.123.456:12345/kibana/app/kibana#/discover?_a=(query:(language:kuery,query:'custom_resource_name:\"postgres01\"'))"
  },
  {
    "Description": "Metrics Dashboard",
    "Endpoint": "https://12.345.123.456:12345/grafana/d/postgres-metrics?var-Namespace=arc3&var-Name=postgres01"
  }
]
```

Puede usar el punto de conexión de la instancia de PostgreSQL para conectarse al grupo de servidores de Hiperescala de PostgreSQL desde la herramienta que prefiera: [Azure Data Studio](/sql/azure-data-studio/download-azure-data-studio), [pgcli](https://www.pgcli.com/) psql, pgAdmin, etc. Al hacerlo, se conectará al nodo o la instancia de coordinación que se encarga de enrutar la consulta a los nodos o las instancias de trabajo adecuados si ha creado tablas distribuidas. Para más información, lea los [conceptos de Hiperescala de PostgreSQL habilitada para Azure Arc](concepts-distributed-postgres-hyperscale.md).

## <a name="special-note-about-azure-virtual-machine-deployments"></a>Nota especial sobre las implementaciones de máquinas virtuales de Azure

Cuando se usa una máquina virtual de Azure, la dirección IP del punto de conexión no mostrará la _IP pública_. Para localizar la IP pública, use el comando siguiente:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

Después, puede combinar la IP pública con el puerto para establecer la conexión.

Es posible que también tenga que exponer el puerto del grupo de servidores Hiperescala de PostgreSQL a través de la puerta de enlace de seguridad de red (NSG). Para permitir el tráfico a través de NSG, tendrá que agregar una regla mediante el comando siguiente:

Para establecer una regla, tendrá que conocer el nombre de su NSG. Puede determinar el NSG con el comando siguiente:

```azurecli
az network nsg list -g azurearcvm-rg --query "[].{NSGName:name}" -o table
```

Una vez que tenga el nombre del NSG, puede agregar una regla de firewall mediante el comando siguiente. En estos valores de ejemplo se crea una regla de NSG para el puerto 30655 y se permite la conexión desde **cualquier** dirección IP de origen.  No es un procedimiento recomendado de seguridad.  Puede mejorar la operación si especifica un valor -source-address-prefixes específico de la dirección IP del cliente, o bien un intervalo de direcciones IP que abarque las direcciones IP del equipo o la organización.

Reemplace el valor del parámetro --destination-port-ranges siguiente por el número de puerto que ha obtenido del comando "azdata arc postgres server list" anterior.

```azurecli
az network nsg rule create -n db_port --destination-port-ranges 30655 --source-address-prefixes '*' --nsg-name azurearcvmNSG --priority 500 -g azurearcvm-rg --access Allow --description 'Allow port through for db access' --destination-address-prefixes '*' --direction Inbound --protocol Tcp --source-port-ranges '*'
```

## <a name="connect-with-azure-data-studio"></a>Conexión con Azure Data Studio

Abra Azure Data Studio y conéctese a la instancia con la dirección IP y el número de puerto del punto de conexión externo anterior, y la contraseña que haya especificado al crear la instancia.  Si PostgreSQL no está disponible en la lista desplegable *Tipo de conexión*, puede instalar la extensión PostgreSQL si busca PostgreSQL en la pestaña Extensiones.

> [!NOTE]
> Tendrá que hacer clic en el botón [Avanzado] del panel de conexión para escribir el número de puerto.

Recuerde que, si usa una máquina virtual de Azure, necesitará la _IP pública_ a la que se pueda acceder mediante el comando siguiente:

```azurecli
az network public-ip list -g azurearcvm-rg --query "[].{PublicIP:ipAddress}" -o table
```

## <a name="connect-with-psql"></a>Conexión con psql

Para acceder al grupo de servidores Hiperescala de PostgreSQL, pase el punto de conexión externo del grupo de servidores Hiperescala de PostgreSQL que ha recuperado antes:

Ahora puede conectarse a psql:

```console
psql postgresql://postgres:<EnterYourPassword>@10.0.0.4:30655
```

## <a name="next-steps"></a>Pasos siguientes

- Conéctese a su Hiperescala de PostgreSQL habilitada para Azure Arc: lea [Obtención de puntos de conexión y cadenas de conexión](get-connection-endpoints-and-connection-strings-postgres-hyperscale.md)
- Lea los conceptos y las guías paso a paso de Hiperescala de Azure Database for PostgreSQL para distribuir los datos entre varios nodos de Hiperescala de PostgreSQL y poder beneficiarse de posibles mejores rendimientos:
    * [Nodos y tablas](../../postgresql/concepts-hyperscale-nodes.md)
    * [Determinar el tipo de aplicación](../../postgresql/concepts-hyperscale-app-type.md)
    * [Elección de una columna de distribución](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocación de tabla](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribución y modificación de tablas](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Diseño de una base de datos multiinquilino](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Diseño de un panel de análisis en tiempo real](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* En los documentos anteriores, omita las secciones **Inicio de sesión en Azure Portal** y **Creación de una instancia de Azure Database for PostgreSQL: Hiperescala (Citus)** . Implemente los pasos restantes en la implementación de Azure Arc. Esas secciones son específicas de Hiperescala (Citus) de Azure Database for PostgreSQL que se ofrece como un servicio PaaS en la nube de Azure, pero las demás partes de los documentos se aplican directamente a Hiperescala de PostgreSQL habilitada para Azure Arc.

- [Escalado horizontal del grupo de servidores de Hiperescala de PostgreSQL habilitada para Azure Arc](scale-out-in-postgresql-hyperscale-server-group.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Expansión de notificaciones de volúmenes persistentes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#expanding-persistent-volumes-claims)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)
