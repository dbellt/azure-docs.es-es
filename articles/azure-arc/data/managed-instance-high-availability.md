---
title: Alta disponibilidad de Instancia administrada habilitada para Azure Arc
titleSuffix: Deploy Azure Arc enabled Managed Instance with high availability
description: Aprenda a implementar la Instancia administrada habilitada para Azure Arc con alta disponibilidad.
author: rothja
ms.author: jroth
ms.reviewer: mikeray
ms.date: 03/02/2021
ms.topic: conceptual
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.openlocfilehash: a847eb04745895fb903385715296a2c5c5d8898e
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110061523"
---
# <a name="azure-arc-enabled-managed-instance-high-availability"></a>Alta disponibilidad de Instancia administrada habilitada para Azure Arc

La Instancia administrada habilitada para Azure Arc se implementa en Kubernetes como una aplicación en contenedor y usa construcciones de Kubernetes como conjuntos con estado y almacenamiento persistente para proporcionar seguimiento de estado integrado, detección de errores y mecanismos de conmutación por error para mantener el estado del servicio. Para aumentar la confiabilidad, también puede configurar la Instancia administrada habilitada para Azure Arc para implementar con réplicas adicionales en una configuración de alta disponibilidad. La supervisión, la detección de errores y la conmutación automática por error se administran mediante el controlador de datos de servicios de datos de Arc. Este servicio se proporciona sin intervención del usuario, desde la configuración del grupo de disponibilidad y la configuración de puntos de conexión de creación de reflejo de la base de datos, hasta la incorporación de bases de datos al grupo de disponibilidad, o conmutación por error y coordinación de actualizaciones. En este documento se exploran ambos tipos de alta disponibilidad.

## <a name="built-in-high-availability"></a>Alta disponibilidad integrada 

Kubernetes proporciona una alta disponibilidad integrada cuando se configura el almacenamiento persistente remoto y se comparte con los nodos usados por la implementación del servicio de datos de Arc. En esta configuración, Kubernetes desempeña el papel de orquestador de clústeres. Si se produce un error en una instancia administrada de un contenedor o el nodo subyacente, el orquestador arranca otra instancia del contenedor y se adjunta al mismo almacenamiento persistente. Este tipo está habilitado de forma predeterminada al implementar la Instancia administrada habilitada para Azure Arc.

### <a name="verify-built-in-high-availability"></a>Comprobación de la alta disponibilidad integrada

En esta sección, comprobará la alta disponibilidad integrada que proporciona Kubernetes. Cuando siga los pasos para probar esta funcionalidad, elimine el pod de una instancia administrada existente y compruebe que Kubernetes se recupera de esta acción. 

### <a name="prerequisites"></a>Requisitos previos

- El clúster de Kubernetes debe tener [almacenamiento remoto compartido](storage-configuration.md#factors-to-consider-when-choosing-your-storage-configuration). 
- Una Instancia administrada habilitada para Azure Arc implementada con una réplica (valor predeterminado)

1. Vea los pods. 

   ```console
   kubectl get pods -n <namespace of data controller>
   ```

2. Elimine el pod de la instancia administrada.

   ```console
   kubectl delete pod <name of managed instance>-0 -n <namespace of data controller>
   ```

   Por ejemplo,

   ```output
   user@pc:/# kubectl delete pod sql1-0 -n arc
   pod "sql1-0" deleted
   ```

3. Vea los pods para comprobar que la instancia administrada se está recuperando.

   ```console
   kubectl get pods -n <namespace of data controller>
   ```

   Por ejemplo,

   ```output
   user@pc:/# kubectl get pods -n arc
   NAME                 READY   STATUS    RESTARTS   AGE
   sql1-0               2/3     Running   0          22s
   ```

Una vez recuperados todos los contenedores dentro del pod, puede conectarse a la instancia administrada.

## <a name="deploy-with-always-on-availability-groups"></a>Implementación con Grupos de disponibilidad AlwaysOn

Para aumentar la confiabilidad, puede configurar la Instancia administrada habilitada para Azure Arc para implementar con réplicas adicionales en una configuración de alta disponibilidad. 

Funcionalidades que permiten los grupos de disponibilidad:

- Cuando se implementa con varias réplicas, se crea un único grupo de disponibilidad denominado `containedag`. De forma predeterminada, `containedag` tiene tres réplicas, incluida la principal. Todas las operaciones CRUD del grupo de disponibilidad se administran internamente, incluida la creación del grupo de disponibilidad o la unión de réplicas al grupo de disponibilidad creado. No se pueden crear grupos de disponibilidad adicionales en la Instancia administrada habilitada para Azure Arc.

- Todas las bases de datos se agregan automáticamente al grupo de disponibilidad, incluidas todas las bases de datos del usuario y el sistema, como `master` y `msdb`. Esta funcionalidad proporciona una vista de un solo sistema de las réplicas del grupo de disponibilidad. Observe las bases de datos `containedag_master` y `containedag_msdb` si se conecta directamente a la instancia. Las bases de datos `containedag_*` representan `master` y `msdb` dentro del grupo de disponibilidad.

- Se aprovisiona automáticamente un punto de conexión externo para conectarse a las bases de datos del grupo de disponibilidad. Este punto de conexión `<managed_instance_name>-external-svc` desempeña el rol de escucha de grupo de disponibilidad.

### <a name="deploy"></a>Implementación

Para implementar una instancia administrada con grupos de disponibilidad, ejecute el siguiente comando.

```console
azdata arc sql mi create -n <name of instance> --replicas 3
```

### <a name="check-status"></a>Comprobar estado
Una vez implementada la instancia, ejecute los siguientes comandos para comprobar el estado de la instancia:

```console
azdata arc sql mi list
azdata arc sql mi show -n <name of instance>
```

Salida de ejemplo:

```output
user@pc:/# azdata arc sql mi list
ExternalEndpoint    Name    Replicas    State
------------------  ------  ----------  -------
20.131.31.58,1433   sql2    3/3         Ready

user@pc:/#  azdata arc sql mi show -n sql2
{
...
  "status": {
    "AGStatus": "Healthy",
    "externalEndpoint": "20.131.31.58,1433",
    "logSearchDashboard": "link to logs dashboard",
    "metricsDashboard": "link to metrics dashboard",
    "readyReplicas": "3/3",
    "state": "Ready"
  }
}
```

Observe el número adicional de `Replicas` y el campo `AGstatus` que indica el estado del grupo de disponibilidad. Si todas las réplicas están activadas y sincronizadas, este valor es `healthy`. 

### <a name="restore-a-database"></a>Restaurar una base de datos 
Se requieren pasos adicionales para restaurar una base de datos en un grupo de disponibilidad. En los pasos siguientes se muestra cómo restaurar una base de datos en una instancia administrada y agregarla a un grupo de disponibilidad. 

1. Exponga el punto de conexión externo de la instancia principal mediante la creación de un nuevo servicio Kubernetes.

    Determine el pod que hospeda la réplica principal; para ello, conéctese a la instancia administrada y ejecute:

    ```sql
    SELECT @@SERVERNAME
    ```
    Cree el servicio Kubernetes en la instancia principal ejecutando el comando siguiente si el clúster de Kubernetes usa los servicios de nodePort. Reemplace `podName` por el nombre del servidor devuelto en el paso anterior, `serviceName` por el nombre preferido para el servicio Kubernetes creado.

    ```bash
    kubectl -n <namespaceName> expose pod <podName> --port=1533  --name=<serviceName> --type=NodePort
    ```

    Para un servicio LoadBalancer, ejecute el mismo comando, con la excepción de que el tipo del servicio creado es `LoadBalancer`. Por ejemplo: 

    ```bash
    kubectl -n <namespaceName> expose pod <podName> --port=1533  --name=<serviceName> --type=LoadBalancer
    ```

    Este es un ejemplo de este comando en ejecución en Azure Kubernetes Service, donde el pod que hospeda la réplica principal es `sql2-0`:

    ```bash
    kubectl -n arc-cluster expose pod sql2-0 --port=1533  --name=sql2-0-p --type=LoadBalancer
    ```

    Obtenga la dirección IP del servicio Kubernetes creado:

    ```bash
    kubectl get services -n <namespaceName>
    ```
2. Restaure la base de datos en el punto de conexión de la instancia principal.

    Agregue el archivo de copia de seguridad de la base de datos en el contenedor de instancia principal.

    ```console
    kubectl cp <source file location> <pod name>:var/opt/mssql/data/<file name> -n <namespace name>
    ```

    Ejemplo

    ```console
    kubectl cp /home/WideWorldImporters-Full.bak sql2-1:var/opt/mssql/data/WideWorldImporters-Full.bak -c arc-sqlmi -n arc
    ```

    Restaure el archivo de copia de seguridad de la base de datos mediante la ejecución del comando siguiente.

    ```sql 
    RESTORE DATABASE test FROM DISK = '/var/opt/mssql/data/<file name>.bak'
    WITH MOVE '<database name>' to '/var/opt/mssql/data/<file name>.mdf'  
    ,MOVE '<database name>' to '/var/opt/mssql/data/<file name>_log.ldf'  
    ,RECOVERY, REPLACE, STATS = 5;  
    GO
    ```
    
    Ejemplo

    ```sql
    RESTORE Database WideWorldImporters
    FROM DISK = '/var/opt/mssql/data/WideWorldImporters-Full.BAK'
    WITH
    MOVE 'WWI_Primary' TO '/var/opt/mssql/data/WideWorldImporters.mdf',
    MOVE 'WWI_UserData' TO '/var/opt/mssql/data/WideWorldImporters_UserData.ndf',
    MOVE 'WWI_Log' TO '/var/opt/mssql/data/WideWorldImporters.ldf',
    MOVE 'WWI_InMemory_Data_1' TO '/var/opt/mssql/data/WideWorldImporters_InMemory_Data_1',
    RECOVERY, REPLACE, STATS = 5;  
    GO
    ```

3. Agregue la base de datos al grupo de disponibilidad.

    Para que la base de datos se agregue al grupo de disponibilidad, debe ejecutarse en modo de recuperación completa y debe realizarse una copia de seguridad del registro. Ejecute las instrucciones TSQL siguientes para agregar la base de datos restaurada al grupo de disponibilidad.

    ```sql
    ALTER DATABASE <databaseName> SET RECOVERY FULL;
    BACKUP DATABASE <databaseName> TO DISK='<filePath>'
    ALTER AVAILABILITY GROUP containedag ADD DATABASE <databaseName>
    ```

    En el ejemplo siguiente se agrega una base de datos denominada `WideWorldImporters` restaurada en la instancia:

    ```sql
    ALTER DATABASE WideWorldImporters SET RECOVERY FULL;
    BACKUP DATABASE WideWorldImporters TO DISK='/var/opt/mssql/data/WideWorldImporters.bak'
    ALTER AVAILABILITY GROUP containedag ADD DATABASE WideWorldImporters
    ```

> [!IMPORTANT]
> Como procedimiento recomendado, debe limpiar mediante la eliminación del servicio Kubernetes creado anteriormente al ejecutar este comando:
>
>```bash
>kubectl delete svc sql2-0-p -n arc
>```

### <a name="limitations"></a>Limitaciones

Los grupos de disponibilidad de Instancia administrada habilitada para Azure Arc tienen las mismas limitaciones [ que los grupos de disponibilidad del clúster de macrodatos. Para más información, haga clic aquí.](/sql/big-data-cluster/deployment-high-availability#known-limitations)

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las [Características y funcionalidades de SQL Managed Instance habilitada para Azure Arc](managed-instance-features.md).
