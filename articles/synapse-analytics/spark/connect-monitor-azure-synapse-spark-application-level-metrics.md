---
title: 'Tutorial: Conexión y supervisión de las métricas de nivel de aplicación de Azure Synapse Spark'
description: 'Tutorial: Aprenda a integrar su servidor local existente de Prometheus con el área de trabajo de Azure Synapse para obtener métricas de aplicación de Azure Spark casi en tiempo real mediante el conector de Prometheus para Azure Synapse.'
services: synapse-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 01/22/2021
ms.openlocfilehash: 45ccced6f083e0d304651a0cea7df90c6396fd88
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143258"
---
# <a name="tutorial-connect-and-monitor-azure-synapse-spark-application-level-metrics"></a>Conexión y supervisión de las métricas de nivel de aplicación de Azure Synapse Spark

## <a name="overview"></a>Información general

En este tutorial aprenderá a integrar su servidor local existente de Prometheus con el área de trabajo de Azure Synapse para obtener métricas de aplicación de Azure Spark casi en tiempo real mediante el conector de Prometheus para Azure Synapse. 

En este tutorial también se presentan las API de métricas de REST de Azure Synapse. Puede capturar datos de métricas de aplicaciones Spark mediante las API REST para crear su propio kit de herramientas de supervisión y diagnóstico, o bien para integrarlos con sus sistemas de supervisión.

## <a name="use-azure-synapse-prometheus-connector-for-your-on-premises-prometheus-servers"></a>Uso del conector de Prometheus para Azure Synapse con los servidores de Prometheus locales

El [conector de Prometheus para Azure Synapse](https://github.com/microsoft/azure-synapse-spark-metrics) es un proyecto de código abierto. Este conector usa un método de detección de servicios basado en archivos que le permite:
 - Autenticarse en el área de trabajo de Azure Synapse con una entidad de servicio de AAD.
 - Capturar la lista de aplicaciones de Apache Spark del área de trabajo. 
 - Extraer métricas de aplicación de Spark mediante la configuración basada en archivos de Prometheus. 

### <a name="1-prerequisite"></a>1. Requisito previo

Necesita un servidor de Prometheus implementado en una máquina virtual de Linux.

### <a name="2-create-a-service-principal"></a>2. Creación de una entidad de servicio

Para usar el conector de Prometheus para Azure Synapse en su servidor de Prometheus local, debe crear una entidad de servicio mediante los pasos siguientes.

#### <a name="21-create-a-service-principal"></a>2.1 Creación de una entidad de servicio:

```bash
az ad sp create-for-rbac --name <service_principal_name>
```

El resultado debería tener este aspecto:

```json
{
  "appId": "abcdef...",
  "displayName": "<service_principal_name>",
  "name": "http://<service_principal_name>",
  "password": "abc....",
  "tenant": "<tenant_id>"
}
```

Anote el identificador de la aplicación, la contraseña y el identificador de inquilino.

#### <a name="22-add-corresponding-permissions-to-the-service-principal-created-in-the-above-step"></a>2.2 Adición de los permisos correspondientes a la entidad de servicio creada en el paso anterior.

![Captura de pantalla de la concesión de permiso de SRBAC](./media/monitor-azure-synapse-spark-application-level-metrics/screenshot-grant-permission-srbac-new.png)

1. Inicie sesión en el [área de trabajo de Azure Synapse Analytics](https://web.azuresynapse.net/) como administrador de Azure Synapse.

2. En Synapse Studio, en el panel de la izquierda, seleccione **Administrar > Control de acceso**.

3. Haga clic en el botón Agregar situado en la parte superior izquierda para **agregar una asignación de roles**.

4. Como ámbito, elija **Área de trabajo**.

5. Seleccione el rol **Operador de procesos de Synapse**.

6. En Seleccionar usuario, escriba el **<nombre_de_la_entidad_de_servicio>** y haga clic en ella.

7. Haga clic en **Aplicar** (espere 3 minutos para que se apliquen los permisos).


### <a name="3-download-the-azure-synapse-prometheus-connector"></a>3. Descarga del conector de Prometheus para Azure Synapse

Use los comandos para instalar el conector de Prometheus para Azure Synapse.

```bash
git clone https://github.com/microsoft/azure-synapse-spark-metrics.git
cd ./azure-synapse-spark-metrics/synapse-prometheus-connector/src
python pip install -r requirements.txt
```

### <a name="4-create-a-config-file-for-azure-synapse-workspaces"></a>4. Creación de un archivo de configuración para las áreas de trabajo de Azure Synapse

Cree un archivo config.yaml en la carpeta config y rellene los campos siguientes: workspace_name, tenant_id, service_principal_name y service_principal_password.
Puede agregar varias áreas de trabajo en el archivo config.yaml.

```yaml
workspaces:
  - workspace_name: <your_workspace_name>
    tenant_id: <tenant_id>
    service_principal_name: <service_principal_app_id>
    service_principal_password: "<service_principal_password>"
```

### <a name="5-update-the-prometheus-config"></a>5. Actualización de la configuración de Prometheus

Agregue la siguiente configuración a la sección scrape_config de Prometheus y reemplace los campos <your_workspace_name> y <path_to_synapse_connector> por el nombre de su área de trabajo y por su carpeta synapse-prometheus-connector clonada, respectivamente.

```yaml
- job_name: synapse-prometheus-connector
  static_configs:
  - labels:
      __metrics_path__: /metrics
      __scheme__: http
    targets:
    - localhost:8000
- job_name: synapse-workspace-<your_workspace_name>
  bearer_token_file: <path_to_synapse_connector>/output/workspace/<your_workspace_name>/bearer_token
  file_sd_configs:
  - files:
    - <path_to_synapse_connector>/output/workspace/<your_workspace_name>/application_discovery.json
    refresh_interval: 10s
  metric_relabel_configs:
  - source_labels: [ __name__ ]
    target_label: __name__
    regex: metrics_application_[0-9]+_[0-9]+_(.+)
    replacement: spark_$1
  - source_labels: [ __name__ ]
    target_label: __name__
    regex: metrics_(.+)
    replacement: spark_$1
```


### <a name="6-start-the-connector-in-the-prometheus-server-vm"></a>6. Inicio del conector en la máquina virtual del servidor de Prometheus

Inicie un servidor de conector en la máquina virtual del servidor de Prometheus tal y como se indica a continuación.

```
python main.py
```

El conector debería empezar a funcionar pasados unos segundos. Y podrá ver la carpeta "synapse-prometheus-connector" en la página de detección de servicios de Prometheus.

## <a name="use-azure-synapse-prometheus-or-rest-metrics-apis-to-collect-metrics-data"></a>Uso de las API de métricas de REST o de Prometheus de Azure Synapse para recopilar datos de métricas

### <a name="1-authentication"></a>1. Authentication
Puede usar el flujo de credenciales de cliente para obtener un token de acceso. Para acceder a las API de métricas, debe obtener un token de acceso de Azure AD para la entidad de servicio, la cual debe tener el permiso adecuado para acceder a las API.

| Parámetro     | Obligatorio | Descripción                                                                                                   |
| ------------- | -------- | ------------------------------------------------------------------------------------------------------------- |
| tenant_id     | Verdadero     | Identificador de inquilino de la entidad de servicio (aplicación) de Azure.                                                          |
| grant_type    | Verdadero     | Especifica el tipo de concesión solicitado. En un flujo de concesión de credenciales de cliente, el valor debe ser client_credentials. |
| client_id     | Verdadero     | Identificador de la aplicación (entidad de servicio) que ha registrado en Azure Portal o en la CLI de Azure.        |
| client_secret | True     | Secreto generado para la aplicación (entidad de servicio).                                                  |
| resource      | Verdadero     | Identificador URI de recursos de Synapse, debería ser "https://dev.azuresynapse.net"                                                  |

```bash
curl -X GET -H 'Content-Type: application/x-www-form-urlencoded' \
  -d 'grant_type=client_credentials&client_id=<service_principal_app_id>&resource=<azure_synapse_resource_id>&client_secret=<service_principal_secret>' \
  https://login.microsoftonline.com/<tenant_id>/oauth2/token
```

La respuesta tiene el siguiente aspecto:

```json
{
  "token_type": "Bearer",
  "expires_in": "599",
  "ext_expires_in": "599",
  "expires_on": "1575500666",
  "not_before": "1575499766",
  "resource": "2ff8...f879c1d",
  "access_token": "ABC0eXAiOiJKV1Q......un_f1mSgCHlA"
}
```

### <a name="2-list-running-applications-in-the-azure-synapse-workspace"></a>2. Enumeración de las aplicaciones en ejecución en el área de trabajo de Azure Synapse

Para obtener una lista de las aplicaciones de Spark en un área de trabajo de Azure Synapse, consulte el artículo [Supervisión: obtención de la lista de trabajos de Spark](/rest/api/synapse/data-plane/monitoring/getsparkjoblist).


### <a name="3-collect-spark-application-metrics-with-the-prometheus-or-rest-apis"></a>3. Recopilación de métricas de aplicaciones de Spark con las API REST o de Prometheus


#### <a name="collect-spark-application-metrics-with-the-prometheus-api"></a>Recopilación de métricas de aplicaciones de Spark con la API de Prometheus

Obtención de las métricas más recientes de la aplicación Spark especificada mediante la API de Prometheus

```
GET https://{endpoint}/livyApi/versions/{livyApiVersion}/sparkpools/{sparkPoolName}/sessions/{sessionId}/applications/{sparkApplicationId}/metrics/executors/prometheus?format=html
```

| Parámetro          | Obligatorio | Descripción                                                                                 |
| ------------------ | -------- | --------------------------------------------------------------------------------------------|
| endpoint           | Verdadero     | El punto de conexión de desarrollo del área de trabajo. Por ejemplo `https://myworkspace.dev.azuresynapse.net.`. |
| livyApiVersion     | Verdadero     | Versión de API válida para la solicitud. Actualmente es 2019-11-01-preview.                      |
| sparkPoolName      | Verdadero     | Nombre del grupo de Spark.                                                                     |
| sessionID          | Verdadero     | Identificador de la sesión.                                                                 |
| sparkApplicationId | Verdadero     | Identificador de la aplicación Spark.                                                                        |

Solicitud de ejemplo: 

```
GET https://myworkspace.dev.azuresynapse.net/livyApi/versions/2019-11-01-preview/sparkpools/mysparkpool/sessions/1/applications/application_1605509647837_0001/metrics/executors/prometheus?format=html
```

Respuesta de ejemplo:

Código de estado: 200 La respuesta tiene el siguiente aspecto:

```
metrics_executor_rddBlocks{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_memoryUsed_bytes{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 74992
metrics_executor_diskUsed_bytes{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_totalCores{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_maxTasks{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_activeTasks{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 1
metrics_executor_failedTasks_total{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 0
metrics_executor_completedTasks_total{application_id="application_1605509647837_0001", application_name="mynotebook_mysparkpool_1605509570802", executor_id="driver"} 2
...

```

#### <a name="collect-spark-application-metrics-with-the-rest-api"></a>Recopilación de métricas de aplicaciones de Spark con la API REST

```
GET https://{endpoint}/livyApi/versions/{livyApiVersion}/sparkpools/{sparkPoolName}/sessions/{sessionId}/applications/{sparkApplicationId}/executors
```

| Parámetro          | Obligatorio | Descripción                                                                                 |
| ------------------ | -------- | --------------------------------------------------------------------------------------------|
| endpoint           | Verdadero     | El punto de conexión de desarrollo del área de trabajo. Por ejemplo `https://myworkspace.dev.azuresynapse.net.`. |
| livyApiVersion     | Verdadero     | Versión de API válida para la solicitud. Actualmente es 2019-11-01-preview.                      |
| sparkPoolName      | Verdadero     | Nombre del grupo de Spark.                                                                     |
| sessionID          | Verdadero     | Identificador de la sesión.                                                                 |
| sparkApplicationId | Verdadero     | Identificador de la aplicación Spark.                                                                        |

Solicitud de ejemplo

```
GET https://myworkspace.dev.azuresynapse.net/livyApi/versions/2019-11-01-preview/sparkpools/mysparkpool/sessions/1/applications/application_1605509647837_0001/executors
```

Respuesta de ejemplo: Código de estado: 200

```json
[
    {
        "id": "driver",
        "hostPort": "f98b8fc2aea84e9095bf2616208eb672007bde57624:45889",
        "isActive": true,
        "rddBlocks": 0,
        "memoryUsed": 75014,
        "diskUsed": 0,
        "totalCores": 0,
        "maxTasks": 0,
        "activeTasks": 0,
        "failedTasks": 0,
        "completedTasks": 0,
        "totalTasks": 0,
        "totalDuration": 0,
        "totalGCTime": 0,
        "totalInputBytes": 0,
        "totalShuffleRead": 0,
        "totalShuffleWrite": 0,
        "isBlacklisted": false,
        "maxMemory": 15845975654,
        "addTime": "2020-11-16T06:55:06.718GMT",
        "executorLogs": {
            "stdout": "http://f98b8fc2aea84e9095bf2616208eb672007bde57624:8042/node/containerlogs/container_1605509647837_0001_01_000001/trusted-service-user/stdout?start=-4096",
            "stderr": "http://f98b8fc2aea84e9095bf2616208eb672007bde57624:8042/node/containerlogs/container_1605509647837_0001_01_000001/trusted-service-user/stderr?start=-4096"
        },
        "memoryMetrics": {
            "usedOnHeapStorageMemory": 75014,
            "usedOffHeapStorageMemory": 0,
            "totalOnHeapStorageMemory": 15845975654,
            "totalOffHeapStorageMemory": 0
        },
        "blacklistedInStages": []
    },
    // ...
]
```


### <a name="4-build-your-own-diagnosis-and-monitoring-tools"></a>4. Creación de sus propias herramientas de diagnóstico y supervisión

La API de Prometheus y las API REST proporcionan datos de métricas completos sobre la aplicación Spark en ejecución. Puede recopilar los datos de métricas relacionados con la aplicación mediante la API de Prometheus y las API REST. También puede crear sus propias herramientas de diagnóstico y supervisión para que se adapten mejor a sus necesidades.
