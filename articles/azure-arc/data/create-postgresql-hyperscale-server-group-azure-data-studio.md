---
title: Creación de Hiperescala de PostgreSQL habilitada para Azure Arc mediante Azure Data Studio
description: Creación de Hiperescala de PostgreSQL habilitada para Azure Arc mediante Azure Data Studio
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 06/02/2021
ms.topic: how-to
ms.openlocfilehash: afcd4eb8327ff806e76b635d8be3715d93b15ed6
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111407712"
---
# <a name="create-azure-arc-enabled-postgresql-hyperscale-using-azure-data-studio"></a>Creación de Hiperescala de PostgreSQL habilitada para Azure Arc mediante Azure Data Studio

En este documento se le guía por los pasos para usar Azure Data Studio a fin de aprovisionar grupos de servidores Hiperescala de PostgreSQL habilitados para Azure Arc.

[!INCLUDE [azure-arc-common-prerequisites](../../../includes/azure-arc-common-prerequisites.md)]

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="connect-to-the-azure-arc-data-controller"></a>Conexión al controlador de datos de Azure Arc

Antes de poder crear una instancia, inicie sesión en el controlador de datos de Azure Arc si todavía no lo ha hecho.

```console
azdata login
```

Después, se le solicitará el espacio de nombres en el que se ha creado el controlador de datos, el nombre de usuario y la contraseña para iniciar sesión en el controlador.

> Si tiene que validar el espacio de nombres, puede ejecutar ```kubectl get pods -A``` para obtener una lista de todos los espacios de nombres del clúster.

```console
Username: arcadmin
Password:
Namespace: arc
Logged in successfully to `https://10.0.0.4:30080` in namespace `arc`. Setting active context to `arc`
```

## <a name="preliminary-and-temporary-step-for-openshift-users-only"></a>Paso preliminar y temporal solo para usuarios de OpenShift

Implemente este paso antes de pasar al siguiente. Para implementar el grupo de servidores Hiperescala de PostgreSQL en Red Hat OpenShift en un proyecto distinto del predeterminado, debe ejecutar los comandos siguientes en el clúster para actualizar las restricciones de seguridad. Este comando concede los privilegios necesarios a las cuentas de servicio que ejecutarán el grupo de servidores Hiperescala de PostgreSQL. La restricción de contexto de seguridad (SCC) **_arc-data-scc_** es la que ha agregado al implementar el controlador de datos de Azure Arc.

```console
oc adm policy add-scc-to-user arc-data-scc -z <server-group-name> -n <namespace name>
```

_**Server-group-name** es el nombre del grupo de servidores que se va a implementar en el siguiente paso._
   
Para obtener más detalles sobre las SCC en OpenShift, consulte la [documentación de OpenShift](https://docs.openshift.com/container-platform/4.2/authentication/managing-security-context-constraints.html).
Ahora puede implementar el paso siguiente.

## <a name="create-an-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Creación de un grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc

1. Inicio de Azure Data Studio
1. En la pestaña Conexiones, haga clic en los tres puntos de la parte superior izquierda y elija "Nueva implementación".
1. En las opciones de implementación, seleccione **Grupo de servidores Hiperescala de PostgreSQL: Azure Arc**
    >[!NOTE]
    > Es posible que se le pida que instale [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)] si no lo está actualmente.
1. Acepte los términos de privacidad y de licencia, y haga clic en **Seleccionar** en la parte inferior.
1. En la hoja Implementar grupo de servidores Hiperescala de PostgreSQL: Azure Arc, escriba la información siguiente:
   - Escriba un nombre para el grupo de servidores
   - El número de nodos de trabajo
   - Escriba y confirme una contraseña para el usuario administrador _postgres_ del grupo de servidores
   - Seleccione la clase de almacenamiento adecuada para los datos
   - Seleccione la clase de almacenamiento adecuada para los registros
   - Seleccione la clase de almacenamiento adecuada para las copias de seguridad
   - Seleccione el número de nodos de trabajo que quiere aprovisionar
1. Haga clic en el botón **Implementar**

Esto inicia la creación del grupo de servidores Hiperescala de PostgreSQL habilitado para Azure Arc en el controlador de datos.

Tras unos minutos, la creación se habrá completado de forma correcta.

### <a name="important-parameters-you-should-consider"></a>Parámetros importantes que debe tener en cuenta:

- **el número de nodos de trabajo** que quiere implementar para escalar horizontalmente y lograr mejores rendimientos; Antes de continuar, lea los [conceptos sobre Hiperescala de Postgres](concepts-distributed-postgres-hyperscale.md). En la tabla siguiente se indica el intervalo de valores admitidos y la forma de implementación de Postgres que obtiene con ellos. Por ejemplo, si quiere implementar un grupo de servidores con 2 nodos de trabajo, indique 2. Se crean tres pods, uno para el nodo o la instancia de coordinación y dos para los nodos o las instancias de trabajo (uno para cada uno de los trabajos).



|Necesita:   |Forma del grupo de servidores que se va a implementar   |Número de nodos de trabajo que se deben indicar   |Nota   |
|---|---|---|---|
|Un formulario de Postgres escalado horizontalmente para satisfacer las necesidades de escalabilidad de las aplicaciones.   |3 o más instancias de Postgres, 1 es coordinador, n son trabajos con n >=2.   |n, con n>=2.   |Se carga la extensión Citus que proporciona la funcionalidad Hiperescala.   |
|Un formulario básico de Hiperescala de Postgres para que realice la validación funcional de la aplicación con un costo mínimo. No es válido para la validación de rendimiento y escalabilidad. Para ello, debe usar el tipo de implementaciones descritas anteriormente.   |1 instancia de Postgres que es coordinador y trabajo.   |0 y agregue Citus a la lista de extensiones que se cargarán.   |Se carga la extensión Citus que proporciona la funcionalidad Hiperescala.   |
|Una instancia sencilla de Postgres que está lista para escalar horizontalmente cuando la necesite.   |1 instancia de Postgres. Todavía no es consciente de la semántica para el coordinador y el trabajo. Para escalarla horizontalmente después de la implementación, edite la configuración, aumente el número de nodos de trabajo y distribuya los datos.   |0   |La extensión Citus que proporciona la funcionalidad Hiperescala está presente en la implementación, pero aún no está cargada.   |
|   |   |   |   |

Aunque indica que 1 trabajo funciona, no se recomienda su uso. Esta implementación no le proporcionará mucho valor. Con ella, se obtienen 2 instancias de Postgres: 1 coordinador y 1 trabajador. Con esta configuración, realmente no se escalan horizontalmente los datos desde que se implementa un único trabajo. Por lo tanto, no verá un mayor nivel de rendimiento y escalabilidad. Quitaremos la compatibilidad de esta implementación en una versión futura.

- **las clases de almacenamiento** que quiere que use el grupo de servidores. Es importante que establezca la clase de almacenamiento justo en el momento de implementar un grupo de servidores, ya que no se puede cambiar después. Si va a cambiar la clase de almacenamiento después de la implementación, deberá extraer los datos, eliminar el grupo de servidores, crear otro grupo de servidores e importar los datos. Puede especificar las clases de almacenamiento que se usarán para los datos, los registros y las copias de seguridad. De forma predeterminada, si no indica las clases de almacenamiento, se usarán las clases de almacenamiento del controlador de datos.
    - Para establecer la clase de almacenamiento para los datos, indique el parámetro `--storage-class-data` o `-scd` seguido del nombre de la clase de almacenamiento.
    - Para establecer la clase de almacenamiento para los registros, indique el parámetro `--storage-class-logs` o `-scl` seguido del nombre de la clase de almacenamiento.
    - Para establecer la clase de almacenamiento para las copias de seguridad: en esta versión preliminar de Hiperescala de PostgreSQL habilitada para Azure Arc hay dos maneras de establecer las clases de almacenamiento en función de los tipos de operaciones de copia de seguridad o restauración que quiera realizar. Estamos trabajando para simplificar esta experiencia. Indicará una clase de almacenamiento o un montaje de notificación de volumen. Un montaje de notificación de volumen es un par formado por una notificación de volumen persistente existente (en el mismo espacio de nombres) y el tipo de volumen (y metadatos opcionales según el tipo de volumen) separados por dos puntos. El volumen persistente se montará en cada pod del grupo de servidores de PostgreSQL.
        - Si quiere planear solo restauraciones completas de bases de datos, establezca el parámetro `--storage-class-backups` o `-scb` seguido del nombre de la clase de almacenamiento.
        - Si planea realizar restauraciones completas de la base de datos y restauraciones a un momento dado, establezca el parámetro `--volume-claim-mounts` o `-vcm` seguido del nombre de una notificación de volumen y un tipo de volumen.


## <a name="next-steps"></a>Pasos siguientes
- [Administración del grupo de servidores mediante Azure Data Studio](manage-postgresql-hyperscale-server-group-with-azure-data-studio.md)
- [Supervisión del grupo de servidores](monitor-grafana-kibana.md)
- Lea los conceptos y las guías paso a paso de Hiperescala de Azure Database for PostgreSQL para distribuir los datos entre varios nodos de Hiperescala de PostgreSQL y poder beneficiarse de toda la eficacia de esta opción. :
    * [Nodos y tablas](../../postgresql/concepts-hyperscale-nodes.md)
    * [Determinar el tipo de aplicación](../../postgresql/concepts-hyperscale-app-type.md)
    * [Elección de una columna de distribución](../../postgresql/concepts-hyperscale-choose-distribution-column.md)
    * [Colocación de tabla](../../postgresql/concepts-hyperscale-colocation.md)
    * [Distribución y modificación de tablas](../../postgresql/howto-hyperscale-modify-distributed-tables.md)
    * [Diseño de una base de datos multiinquilino](../../postgresql/tutorial-design-database-hyperscale-multi-tenant.md)*
    * [Diseño de un panel de análisis en tiempo real](../../postgresql/tutorial-design-database-hyperscale-realtime.md)*

    > \* En los documentos anteriores, omita las secciones **Inicio de sesión en Azure Portal** y **Creación de una instancia de Azure Database for PostgreSQL: Hiperescala (Citus)** . Implemente los pasos restantes en la implementación de Azure Arc. Esas secciones son específicas de Hiperescala (Citus) de Azure Database for PostgreSQL que se ofrece como un servicio PaaS en la nube de Azure, pero las demás partes de los documentos se aplican directamente a Hiperescala de PostgreSQL habilitada para Azure Arc.

- [Escalado del grupo de servidores Hiperescala de Azure Database for PostgreSQL](scale-out-in-postgresql-hyperscale-server-group.md)
- [Conceptos de almacenamiento de Kubernetes y configuración de almacenamiento](storage-configuration.md)
- [Modelo de recursos de Kubernetes](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/scheduling/resources.md#resource-quantities)

