---
title: Uso de Azure Log Analytics para recopilar y visualizar métricas y registros (versión preliminar)
description: Aprenda a habilitar el conector de Azure Log Analytics integrado de Synapse para recopilar y enviar los registros y métricas de la aplicación Apache Spark al área de trabajo de Azure Log Analytics.
services: synapse-analytics
author: jejiang
ms.author: jejiang
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: spark
ms.date: 03/25/2021
ms.custom: references_regions
ms.openlocfilehash: 243618192593d93bba9d5229e7becfb2af62ce32
ms.sourcegitcommit: 5f482220a6d994c33c7920f4e4d67d2a450f7f08
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/08/2021
ms.locfileid: "107107745"
---
# <a name="tutorial-use-azure-log-analytics-to-collect-and-visualize-metrics-and-logs-preview"></a>Tutorial: Uso de Azure Log Analytics para recopilar y visualizar métricas y registros (versión preliminar)

En este tutorial, aprenderá a habilitar el conector de Azure Log Analytics integrado de Synapse para recopilar y enviar los registros y métricas de la aplicación Apache Spark al [área de trabajo de Azure Log Analytics](/azure/azure-monitor/logs/quick-create-workspace). A continuación, puede aprovechar los libros de Azure Monitor para visualizar las métricas y los registros.

## <a name="configure-azure-log-analytics-workspace-information-in-synapse-studio"></a>Configuración de la información del área de trabajo de Azure Log Analytics en Synapse Studio

### <a name="step-1-create-an-azure-log-analytics-workspace"></a>Paso 1: Creación de un área de trabajo de Azure Log Analytics

Puede seguir los documentos siguientes para crear un área de trabajo de Log Analytics:
- [Creación de un área de trabajo de Log Analytics en Azure Portal](https://docs.microsoft.com/azure/azure-monitor/logs/quick-create-workspace)
- [Creación de un área de trabajo de Log Analytics con la CLI de Azure](https://docs.microsoft.com/azure/azure-monitor/logs/quick-create-workspace-cli)
- [Crear y configurar el área de trabajo de Log Analytics en Azure Monitor con PowerShell](https://docs.microsoft.com/azure/azure-monitor/logs/powershell-workspace-configuration)

### <a name="step-2-prepare-a-spark-configuration-file"></a>Paso 2: Preparación de un archivo de configuración de Spark

#### <a name="option-1-configure-with-azure-log-analytics-workspace-id-and-key"></a>Opción 1. Configuración con el identificador y la clave del área de trabajo de Azure Log Analytics 

Copie la siguiente configuración de Spark, guárdela como **"spark_loganalytics_conf.txt"** y rellene los parámetros:

   - `<LOG_ANALYTICS_WORKSPACE_ID>`: identificador del área de trabajo de Azure Log Analytics.
   - `<LOG_ANALYTICS_WORKSPACE_KEY>`: clave de Azure Log Analytics: **Azure Portal > Área de trabajo de Azure Log Analytics > Agents management (Administración de agentes) > Clave principal**

```properties
spark.synapse.logAnalytics.enabled true
spark.synapse.logAnalytics.workspaceId <LOG_ANALYTICS_WORKSPACE_ID>
spark.synapse.logAnalytics.secret <LOG_ANALYTICS_WORKSPACE_KEY>
```

#### <a name="option-2-configure-with-an-azure-key-vault"></a>Opción 2. Configuración con Azure Key Vault

> [!NOTE]
>
> Debe conceder permiso de lectura de secretos a los usuarios que vayan a enviar aplicaciones Spark. Consulte [cómo proporcionar acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure](https://docs.microsoft.com/azure/key-vault/general/rbac-guide).

Para configurar una instancia de Azure Key Vault a fin de almacenar la clave del área de trabajo, siga estos pasos:

1. Cree un almacén de claves en Azure Portal y desplácese hasta él.
2. En las páginas de configuración de Key Vault, seleccione **Secretos**.
3. Haga clic en **Generar o Importar**.
4. En la pantalla **Crear un secreto**, elija los siguientes valores:
   - **Nombre**: escriba un nombre para el secreto; escriba `"SparkLogAnalyticsSecret"` como valor predeterminado.
   - **Valor**: escriba el valor de **<LOG_ANALYTICS_WORKSPACE_KEY>** del secreto.
   - Deje las restantes opciones con sus valores predeterminados. Haga clic en **Crear**.
5. Copie la siguiente configuración de Spark, guárdela como **"spark_loganalytics_conf.txt"** y rellene los parámetros:

   - `<LOG_ANALYTICS_WORKSPACE_ID>`: identificador del área de trabajo de Azure Log Analytics.
   - `<AZURE_KEY_VAULT_NAME>`: el nombre de Azure Key Vault que ha configurado.
   - `<AZURE_KEY_VAULT_SECRET_KEY_NAME>` (Opcional): el nombre del secreto en Azure Key Vault de la clave del área de trabajo; el valor predeterminado es "SparkLogAnalyticsSecret".

```properties
spark.synapse.logAnalytics.enabled true
spark.synapse.logAnalytics.workspaceId <LOG_ANALYTICS_WORKSPACE_ID>
spark.synapse.logAnalytics.keyVault.name <AZURE_KEY_VAULT_NAME>
spark.synapse.logAnalytics.keyVault.key.secret <AZURE_KEY_VAULT_SECRET_KEY_NAME>
```

> [!NOTE]
>
> También puede almacenar el identificador del área de trabajo de Log Analytics en Azure Key Vault. Consulte los pasos anteriores y almacene el identificador del área de trabajo con el nombre del secreto `"SparkLogAnalyticsWorkspaceId"`. O bien, use la configuración `spark.synapse.logAnalytics.keyVault.key.workspaceId` para especificar el nombre del secreto del identificador del área de trabajo en Azure Key Vault.

#### <a name="option-3-configure-with-an-azure-key-vault-linked-service"></a>Opción 3. Configuración con un servicio vinculado de Azure Key Vault

> [!NOTE]
>
> Debe conceder permiso de lectura de secretos al área de trabajo de Synapse. Consulte [cómo proporcionar acceso a las claves, los certificados y los secretos de Key Vault con un control de acceso basado en rol de Azure](https://docs.microsoft.com/azure/key-vault/general/rbac-guide).

Para configurar un servicio vinculado de Azure Key Vault en Synapse Studio para almacenar la clave del área de trabajo, siga estos pasos:

1. Siga los pasos descritos en la sección `Option 2. Configure with an Azure Key Vault`.
2. Cree un servicio vinculado de Azure Key Vault en Azure Synapse Studio:

    a. Vaya a **Synapse Studio > Administrar > Servicios vinculados** y haga clic en el botón **Nuevo**.

    b. Busque **Azure Key Vault** en el cuadro de búsqueda.

    c. Escriba un nombre para el servicio vinculado.

    d. Elija el almacén de claves de Azure. Haga clic en **Crear**.

3. Agregue un elemento `spark.synapse.logAnalytics.keyVault.linkedServiceName` a la configuración de Spark.

```properties
spark.synapse.logAnalytics.enabled true
spark.synapse.logAnalytics.workspaceId <LOG_ANALYTICS_WORKSPACE_ID>
spark.synapse.logAnalytics.keyVault.name <AZURE_KEY_VAULT_NAME>
spark.synapse.logAnalytics.keyVault.key.secret <AZURE_KEY_VAULT_SECRET_KEY_NAME>
spark.synapse.logAnalytics.keyVault.linkedServiceName <LINKED_SERVICE_NAME>
```

#### <a name="available-spark-configuration"></a>Configuración de Spark disponible

| Nombre de la configuración                                  | Valor predeterminado                | Descripción                                                                                                                                                                                                |
| --------------------------------------------------- | ---------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| spark.synapse.logAnalytics.enabled                  | false                        | True para habilitar el receptor de Azure Log Analytics para las aplicaciones Spark. En caso contrario, es false.                                                                                                                  |
| spark.synapse.logAnalytics.workspaceId              | -                            | Identificador del área de trabajo de Azure Log Analytics de destino.                                                                                                                                                          |
| spark.synapse.logAnalytics.secret                   | -                            | Secreto del área de trabajo de Azure Log Analytics de destino.                                                                                                                                                      |
| spark.synapse.logAnalytics.keyVault.linkedServiceName   | -                            | Nombre del servicio vinculado de Azure Key Vault para el identificador y la clave del área de trabajo de Azure Log Analytics.                                                                                                                       |
| spark.synapse.logAnalytics.keyVault.name            | -                            | Nombre del almacén de claves de Azure para el identificador y la clave de Azure Log Analytics.                                                                                                                                                |
| spark.synapse.logAnalytics.keyVault.key.workspaceId | SparkLogAnalyticsWorkspaceId | Nombre del secreto de Azure Key Vault para el identificador del área de trabajo de Azure Log Analytics.                                                                                                                                       |
| spark.synapse.logAnalytics.keyVault.key.secret      | SparkLogAnalyticsSecret      | Nombre del secreto de Azure Key Vault para la clave del área de trabajo de Azure Log Analytics.                                                                                                                                      |
| spark.synapse.logAnalytics.keyVault.uriSuffix       | ods.opinsights.azure.com     | [Sufijo del URI][uri_suffix] del área de trabajo de Azure Log Analytics de destino. Si el área de trabajo de Azure Log Analytics no está en Azure global, debe actualizar el sufijo del URI según la nube correspondiente. |

> [!NOTE]  
> - En las nubes de Azure China, el parámetro "spark.synapse.logAnalytics.keyVault.uriSuffix" debe ser "ods.opinsights.azure.cn". 
> - En las nubes de Azure Gov, el parámetro "spark.synapse.logAnalytics.keyVault.uriSuffix" debe ser "ods.opinsights.azure.us". 

[uri_suffix]: https://docs.microsoft.com/azure/azure-monitor/logs/data-collector-api#request-uri


### <a name="step-3-upload-your-spark-configuration-to-a-spark-pool"></a>Paso 3: Carga de la configuración de Spark en un grupo de Spark
Puede cargar el archivo de configuración en el grupo de Spark de Synapse en Synapse Studio.

   1. Vaya al grupo de Apache Spark en Azure Synapse Studio (Administrar > Grupos de Apache Spark).
   2. Haga clic en el botón **"..."** situado a la derecha del grupo de Apache Spark.
   3. Seleccione la configuración de Apache. 
   4. Haga clic en **Cargar** y elija el archivo **"spark_loganalytics_conf.txt"** creado.
   5. Haga clic en **Cargar** y, luego, en **Aplicar**.

      > [!div class="mx-imgBorder"]
      > ![Configuración del grupo de Spark](./media/apache-spark-azure-log-analytics/spark-pool-configuration.png)

> [!NOTE] 
>
> Toda la aplicación Spark enviada al grupo de Spark anterior usará los parámetros de configuración para insertar las métricas y los registros de la aplicación Spark en el área de trabajo de Azure Log Analytics especificada.

## <a name="submit-a-spark-application-and-view-the-logs-and-metrics-in-azure-log-analytics"></a>Envío de una aplicación Spark y visualización de los registros y las métricas en Azure Log Analytics

 1. Puede enviar una aplicación Spark al grupo de Spark configurado en el paso anterior, mediante una de las siguientes maneras:
    - Ejecute un cuaderno de Synapse Studio. 
    - Envíe un trabajo por lotes de Apache Spark de Synapse mediante la definición de trabajo de Spark.
    - Ejecute una canalización que contenga la actividad de Spark.

 2. Vaya al área de trabajo de Azure Log Analytics especificada y, luego, vea las métricas y los registros de la aplicación cuando la aplicación Spark empiece a ejecutarse.

## <a name="use-the-sample-azure-log-analytics-workbook-to-visualize-the-metrics-and-logs"></a>Uso del libro de ejemplo de Azure Log Analytics para visualizar las métricas y los registros

1. [Descargue el libro](https://aka.ms/SynapseSparkLogAnalyticsWorkbook) aquí.
2. Abra y **copie** el contenido del archivo de libro.
3. Vaya al libro de Azure Log Analytics ([Azure Portal](https://portal.azure.com/) > Área de trabajo de Log Analytics > Libros).
4. Abra el libro de Azure Log Analytics **"Vacío"** en modo **"Editor avanzado"** (presione el icono </>).
5. **Pegue** cualquier código JSON que exista.
6. Luego, presione **Aplicar** y **Edición finalizada**.

    > [!div class="mx-imgBorder"]
    > ![nuevo libro](./media/apache-spark-azure-log-analytics/new-workbook.png)

    > [!div class="mx-imgBorder"]
    > ![importar libro](./media/apache-spark-azure-log-analytics/import-workbook.png)

A continuación, envíe la aplicación de Apache Spark al grupo de Spark configurado. Cuando la aplicación pase al estado de ejecución, elija la aplicación en ejecución en la lista desplegable del libro.

> [!div class="mx-imgBorder"]
> ![imagen del libro](./media/apache-spark-azure-log-analytics/workbook.png)

Además, puede personalizar el libro mediante consultas de Kusto y configurar alertas.

> [!div class="mx-imgBorder"]
> ![consultas y alertas de Kusto](./media/apache-spark-azure-log-analytics/kusto-query-and-alerts.png)

## <a name="sample-kusto-queries"></a>Ejemplos de consultas de Kusto

1. Ejemplo de consulta de eventos de Spark.

   ```kusto
   SparkListenerEvent_CL
   | where workspaceName_s == "{SynapseWorkspace}" and clusterName_s == "{SparkPool}" and livyId_s == "{LivyId}"
   | order by TimeGenerated desc
   | limit 100 
   ```

2. Ejemplo de consulta de registros de ejecutores y controladores de aplicaciones de Spark.

   ```kusto
   SparkLoggingEvent_CL
   | where workspaceName_s == "{SynapseWorkspace}" and clusterName_s == "{SparkPool}" and livyId_s == "{LivyId}"
   | order by TimeGenerated desc
   | limit 100
   ```

3. Ejemplo de consulta de métricas de Spark.

   ```kusto
   SparkMetrics_CL
   | where workspaceName_s == "{SynapseWorkspace}" and clusterName_s == "{SparkPool}" and livyId_s == "{LivyId}"
   | where name_s endswith "jvm.total.used"
   | summarize max(value_d) by bin(TimeGenerated, 30s), executorId_s
   | order by TimeGenerated asc
   ```

## <a name="create-and-manage-alerts-using-azure-log-analytics"></a>Creación y administración de alertas mediante Azure Log Analytics

Las alertas de Azure Monitor permiten a los usuarios usar una consulta de Log Analytics para evaluar los registros y las métricas según una frecuencia establecida y activar una alerta en función de los resultados.

Para más información, consulte [Creación, visualización y administración de alertas de registro mediante Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/alerts/alerts-log).

## <a name="limitation"></a>Limitación

 - No se admite un área de trabajo de Azure Synapse Analytics con la [red virtual administrada](https://docs.microsoft.com/azure/synapse-analytics/security/synapse-workspace-managed-vnet) habilitada.
 - Actualmente, no se admiten las siguientes regiones:
   - Este de EE. UU. 2
   - Este de Noruega
   - Norte de Emiratos Árabes Unidos

## <a name="next-steps"></a>Pasos siguientes

 - Aprenda a [usar el grupo de Apache Spark sin servidor en Synapse Studio](https://docs.microsoft.com/azure/synapse-analytics/quickstart-create-apache-spark-pool-studio).
 - Aprenda a [ejecutar una aplicación Spark en un cuaderno](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-development-using-notebooks).
 - Aprenda a [crear una definición de trabajo de Apache Spark en Synapse Studio](https://docs.microsoft.com/azure/synapse-analytics/spark/apache-spark-job-definitions).