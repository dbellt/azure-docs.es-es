---
title: Inicio rápido de Java para Metrics Advisor
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: mrbullwinkle
manager: nitinme
ms.service: applied-ai-services
ms.subservice: metrics-advisor
ms.topic: include
ms.date: 11/09/2020
ms.author: mbullwin
ms.openlocfilehash: 5abdea905f38096a523ffb0e0aa2a7b1460c8a4a
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111894501"
---
[Documentación de referencia](https://westus2.dev.cognitive.microsoft.com/docs/services/MetricsAdvisor/) | [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src) | [Artifact (Maven)](https://search.maven.org/artifact/com.azure/azure-ai-metricsadvisor) | [Ejemplos](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src/samples)

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/cognitive-services/)
* La versión actual de [Java Development Kit (JDK)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* La [herramienta de compilación de Gradle](https://gradle.org/install/) u otro administrador de dependencias.
* Una vez que tenga la suscripción de Azure, <a href="https://go.microsoft.com/fwlink/?linkid=2142156"  title="cree un recurso de Metrics Advisor"  target="_blank">create a Metrics Advisor resource </a> en Azure Portal para implementar la instancia de Metrics Advisor.  
* Su propia base de datos SQL con datos de series temporales.
  
  
> [!TIP]
> * Puede encontrar ejemplos de Metrics Advisor de Java en [GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src/samples).
> * El recurso de Metrics Advisor puede tardar entre 10 y 30 minutos en implementar una instancia de servicio para que pueda usarla. Haga clic en **Ir al recurso** una vez que se implemente correctamente. Después de la implementación, puede empezar a usar la instancia de Metrics Advisor con el portal web y la API de REST. 
> * Puede encontrar la dirección URL de la API de REST en Azure Portal, en la sección **Información general** de su recurso. Tendrá este aspecto:
>    * `https://<instance-name>.cognitiveservices.azure.com/`
    
## <a name="setting-up"></a>Instalación

### <a name="create-a-new-gradle-project"></a>Creación de un proyecto de Gradle

En este inicio rápido se usa el administrador de dependencias Gradle. Puede encontrar más información de la biblioteca cliente en [Maven Central Repository](https://search.maven.org/artifact/com.azure/azure-ai-metricsadvisor).

En una ventana de la consola (como cmd, PowerShell o Bash), cree un directorio para la aplicación y vaya a él. 

```console
mkdir myapp && cd myapp
```

Ejecute el comando `gradle init` desde el directorio de trabajo. Este comando creará archivos de compilación esenciales para Gradle, como *build.gradle.kts*, que se usa en el runtime para crear y configurar la aplicación.

```console
gradle init --type basic
```

Cuando se le solicite que elija un **DSL**, seleccione **Kotlin**.

### <a name="install-the-client-library"></a>Instalación de la biblioteca cliente

Busque *build.gradle.kts* y ábralo con el IDE o el editor de texto que prefiera. A continuación, se copia en esta configuración de compilación. Asegúrese de incluir las dependencias del proyecto.

```kotlin
dependencies {
    compile("com.azure:azure-ai-metricsadvisor:1.0.0-beta.2")
}
```

### <a name="create-a-java-file"></a>Creación de un archivo Java

Cree una carpeta para la aplicación de ejemplo. En el directorio de trabajo, ejecute el siguiente comando:

```console
mkdir -p src/main/java
```

Vaya a la nueva carpeta y cree un archivo denominado *MetricsAdvisorQuickstarts.java*. Ábralo en el editor o el IDE que prefiera y agregue las siguientes instrucciones `import`:

> [!TIP]
> ¿Desea ver todo el archivo de código de inicio rápido de una vez? Puede encontrarlo en [GitHub](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/metricsadvisor/azure-ai-metricsadvisor/src/samples), que contiene los ejemplos de código de este inicio rápido.

En la clase `MetricsAdvisorQuickstarts` de la aplicación, cree variables para el punto de conexión y la clave del recurso.


> [!IMPORTANT]
> Vaya a Azure Portal. Si el recurso de Metrics Advisor que creó en la sección **Requisitos previos** se implementó correctamente, haga clic en el botón **Ir al recurso** en **Pasos siguientes**. Sus claves de suscripción y su punto de conexión se encuentran en la página **Key and Endpoint** (Clave y punto de conexión) del recurso, en **Administración de recursos**. <br><br>Para recuperar la clave de API, debe ir a [https://metricsadvisor.azurewebsites.net](https://metricsadvisor.azurewebsites.net). Seleccione los valores apropiados: **Directory** (Directorio), **Subscriptions** (Suscripciones) y **Workspace** (Área de trabajo) del recurso y seleccione **Comenzar**. A continuación, podrá recuperar las claves de API de [https://metricsadvisor.azurewebsites.net/api-key](https://metricsadvisor.azurewebsites.net/api-key).   
>
> Recuerde quitar la clave del código cuando haya terminado y no hacerla nunca pública. En el caso de producción, considere la posibilidad de usar alguna forma segura de almacenar las credenciales, y acceder a ellas. Para más información, consulte el artículo sobre la [seguridad](../../../cognitive-services-security.md) de Cognitive Services.

```java
private static String SUBSCRIPTION_KEY = "<replace-with-your-metrics-advisor-subscription-key-here>";
private static String API_KEY = "<replace-with-your-metrics-advisor-api-key-here>"
private static String ENDPOINT = "<replace-with-your-metrics-advisor-endpoint-here>";
```

En el método `Main()` de la aplicación, agregue llamadas para los métodos que se usan en este inicio rápido. Las creará más adelante.

```java
static void Main(string[] args){

    // You will create the below methods later in the quickstart
    exampleTask1();
}
```

## <a name="object-model"></a>Modelo de objetos

Las siguientes clases controlan algunas de las características principales del SDK de Java para Metrics Advisor.

|Nombre|Descripción|
|---|---|
| [MetricsAdvisorClient](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/MetricsAdvisorClient.html) | **Se usa para:** <br> - Enumerar incidentes anómalos <br> - Enumerar la causa raíz de los incidentes <br> - Recuperar los datos de series temporales originales y los datos de series temporales enriquecidos por el servicio. <br> - Enumerar alertas <br> - Agregar comentarios para optimizar el modelo |
| [MetricsAdvisorAdministrationClient](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/administration/MetricsAdvisorAdministrationClient.html)| **Permite:** <br> - Administrar fuentes de distribución de datos <br> - Configurar la detección de anomalías <br> - Configurar las alertas de anomalías <br> - Administrar enlaces  |
| [DataFeed](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/models/DataFeed.html) | **Lo que Metrics Advisor ingiere de su origen de datos. `DataFeed` contiene filas de:** <br> - Marcas de tiempo <br> - Cero o más dimensiones <br> - Una o varias medidas  |
| [DataFeedMetric](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-ai-metricsadvisor/1.0.0-beta.2/com/azure/ai/metricsadvisor/models/DataFeedMetric.html) | `DataFeedMetric` es una medida cuantificable que se usa para supervisar y evaluar el estado de un proceso empresarial específico. Puede ser una combinación de varios valores de serie temporal divididos en dimensiones. Por ejemplo, una métrica de mantenimiento de la Web podría contener dimensiones para recuento de usuarios y el mercado es-es. |

## <a name="code-examples"></a>Ejemplos de código

Estos fragmentos de código muestran cómo realizar las siguientes tareas con la biblioteca cliente de Metrics Advisor para Java:

* [Autenticar el cliente](#authenticate-the-client)
* [Adición de una fuente de distribución de datos](#add-a-data-feed)
* [Comprobación del estado de la ingesta](#check-the-ingestion-status)
* [Configuración de la detección de anomalías](#configure-anomaly-detection)
* [Crear un enlace](#create-a-hook)
* [Crear una configuración de alertas](#create-an-alert-configuration)
* [Consultar la alerta](#query-the-alert)

## <a name="authenticate-the-client"></a>Autenticar el cliente

### <a name="create-a-metrics-advisor-client-using-metricsadvisorkeycredential"></a>Creación de un cliente de Metrics Advisor mediante MetricsAdvisorKeyCredential

```java
MetricsAdvisorKeyCredential credential = new MetricsAdvisorKeyCredential(SUBSCRIPTION_KEY, API_KEY);
MetricsAdvisorClient metricsAdvisorClient = new MetricsAdvisorClientBuilder()
    .endpoint(ENDPOINT)
    .credential(credential)
    .buildClient();
```

### <a name="create-a-metrics-administration-client-using-metricsadvisorkeycredential"></a>Creación de un cliente de Metrics Administration mediante MetricsAdvisorKeyCredential

```java
MetricsAdvisorKeyCredential credential = new MetricsAdvisorKeyCredential(SUBSCRIPTION_KEY, API_KEY);
MetricsAdvisorAdministrationClient metricsAdvisorAdministrationClient =
    new MetricsAdvisorAdministrationClientBuilder()
        .endpoint(ENDPOINT)
        .credential(credential)
        .buildClient();

``` 

## <a name="add-a-data-feed"></a>Adición de una fuente de distribución de datos

Reemplace `sql_server_connection_string` por su propia cadena de conexión de SQL Server y reemplace `query` por una consulta que devuelva los datos en una sola marca de tiempo. También tendrá que ajustar los valores de `DataFeedMetric` y `DataFeedDimension` en función de los datos personalizados.

> [!IMPORTANT]
> La consulta debe devolver como máximo un registro para cada combinación de dimensión, en cada marca de tiempo. Y todos los registros devueltos por la consulta deben tener las mismas marcas de tiempo. El asesor de métricas ejecutará esta consulta para cada marca de tiempo que deba ingerir los datos. Consulte [Preguntas frecuentes sobre las consultas](../../faq.md#how-do-i-write-a-valid-query-for-ingesting-my-data) para más información y ejemplos. 

```java
DataFeed dataFeed = new DataFeed()
    .setName("dataFeedName")
    .setSource(new MySqlDataFeedSource("sql_server_connection_string", "query"))
    .setGranularity(new DataFeedGranularity().setGranularityType(DataFeedGranularityType.DAILY))
    .setSchema(new DataFeedSchema(
        Arrays.asList(
            new DataFeedMetric().setName("cost"),
            new DataFeedMetric().setName("revenue")
        )).setDimensions(
        Arrays.asList(
            new DataFeedDimension().setName("city"),
            new DataFeedDimension().setName("category")
        ))
    )
    .setIngestionSettings(new DataFeedIngestionSettings(OffsetDateTime.parse("2020-01-01T00:00:00Z")))
    .setOptions(new DataFeedOptions()
        .setDescription("data feed description")
        .setRollupSettings(new DataFeedRollupSettings()
            .setRollupType(DataFeedRollupType.AUTO_ROLLUP)));
final DataFeed createdSqlDataFeed = metricsAdvisorAdminClient.createDataFeed(dataFeed);

System.out.printf("Data feed Id : %s%n", createdSqlDataFeed.getId());
System.out.printf("Data feed name : %s%n", createdSqlDataFeed.getName());
System.out.printf("Is the query user is one of data feed administrator : %s%n", createdSqlDataFeed.isAdmin());
System.out.printf("Data feed created time : %s%n", createdSqlDataFeed.getCreatedTime());
System.out.printf("Data feed granularity type : %s%n",
    createdSqlDataFeed.getGranularity().getGranularityType());
System.out.printf("Data feed granularity value : %d%n",
    createdSqlDataFeed.getGranularity().getCustomGranularityValue());
System.out.println("Data feed related metric Ids:");
createdSqlDataFeed.getMetricIds().forEach(System.out::println);
System.out.printf("Data feed source type: %s%n", createdSqlDataFeed.getSourceType());

if (SQL_SERVER_DB == createdSqlDataFeed.getSourceType()) {
    System.out.printf("Data feed sql server query: %s%n",
        ((SQLServerDataFeedSource) createdSqlDataFeed.getSource()).getQuery());
}
```

## <a name="check-the-ingestion-status"></a>Comprobación del estado de la ingesta

En este ejemplo se comprueba el estado de la ingesta de un origen de una fuente de distribución de datos proporcionado previamente.

```java
String dataFeedId = "<use-the-data-feed-id-from-createdSqlDataFeed.getId()"; 

metricsAdvisorAdminClient.listDataFeedIngestionStatus(
    dataFeedId,
    new ListDataFeedIngestionOptions(
        OffsetDateTime.parse("2020-01-01T00:00:00Z"),
        OffsetDateTime.parse("2020-09-09T00:00:00Z"))
).forEach(dataFeedIngestionStatus -> {
    System.out.printf("Message : %s%n", dataFeedIngestionStatus.getMessage());
    System.out.printf("Timestamp value : %s%n", dataFeedIngestionStatus.getTimestamp());
    System.out.printf("Status : %s%n", dataFeedIngestionStatus.getStatus());
});
```
## <a name="configure-anomaly-detection"></a>Configuración de la detección de anomalías 

En este ejemplo se muestra la forma en que un usuario puede configurar una configuración de detección de anomalías para sus datos.

```java
String metricId = "<metric-id-from-adding-data-feed>";

ChangeThresholdCondition changeThresholdCondition = new ChangeThresholdCondition()
    .setAnomalyDetectorDirection(AnomalyDetectorDirection.BOTH)
    .setChangePercentage(20)
    .setShiftPoint(10)
    .setWithinRange(true)
    .setSuppressCondition(new SuppressCondition().setMinNumber(1).setMinRatio(2));

HardThresholdCondition hardThresholdCondition = new HardThresholdCondition()
    .setAnomalyDetectorDirection(AnomalyDetectorDirection.DOWN)
    .setLowerBound(5.0)
    .setSuppressCondition(new SuppressCondition().setMinNumber(1).setMinRatio(1));

SmartDetectionCondition smartDetectionCondition = new SmartDetectionCondition()
    .setAnomalyDetectorDirection(AnomalyDetectorDirection.UP)
    .setSensitivity(10.0)
    .setSuppressCondition(new SuppressCondition().setMinNumber(1).setMinRatio(2));

final AnomalyDetectionConfiguration anomalyDetectionConfiguration =
    metricsAdvisorAdminClient.createMetricAnomalyDetectionConfig(
        metricId,
        new AnomalyDetectionConfiguration("My dataPoint anomaly detection configuration")
            .setDescription("anomaly detection config description")
            .setWholeSeriesDetectionCondition(
                new MetricWholeSeriesDetectionCondition()
                    .setChangeThresholdCondition(changeThresholdCondition)
                    .setHardThresholdCondition(hardThresholdCondition)
                    .setSmartDetectionCondition(smartDetectionCondition)
                    .setCrossConditionOperator(DetectionConditionsOperator.OR))
    );
```

## <a name="create-a-hook"></a>Creación de un enlace

En este ejemplo se crea un enlace de correo electrónico que recibe alertas de incidentes de anomalías.

```java
NotificationHook emailNotificationHook = new EmailNotificationHook("email Hook")
    .setDescription("my email Hook")
    .addEmailToAlert("alertme@alertme.com")
    .setExternalLink("https://example.com/handleAlerts"); // you must enter a valid webhook url to post the alert payload

final NotificationHook notificationHook = metricsAdvisorAdminClient.createHook(emailNotificationHook);
EmailNotificationHook createdEmailHook = (EmailNotificationHook) notificationHook;
System.out.printf("Email Hook Id: %s%n", createdEmailHook.getId());
System.out.printf("Email Hook name: %s%n", createdEmailHook.getName());
System.out.printf("Email Hook description: %s%n", createdEmailHook.getDescription());
System.out.printf("Email Hook external Link: %s%n", createdEmailHook.getExternalLink());
System.out.printf("Email Hook emails to alert: %s%n",
    String.join(",", createdEmailHook.getEmailsToAlert()));
```

##  <a name="create-an-alert-configuration"></a>Creación de una configuración de alertas

En este ejemplo se muestra la forma en que un usuario puede configurar una configuración de alertas para las anomalías detectadas en sus datos.

```java
String detectionConfigurationId1 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
String detectionConfigurationId2 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
String hookId1 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
String hookId2 = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";

final AnomalyAlertConfiguration anomalyAlertConfiguration
    = metricsAdvisorAdminClient.createAnomalyAlertConfig(
        new AnomalyAlertConfiguration("My anomaly alert config name")
            .setDescription("alert config description")
            .setMetricAlertConfigurations(
                Arrays.asList(
                    new MetricAnomalyAlertConfiguration(detectionConfigurationId1,
                        MetricAnomalyAlertScope.forWholeSeries()),
                    new MetricAnomalyAlertConfiguration(detectionConfigurationId2,
                        MetricAnomalyAlertScope.forWholeSeries())
                        .setAlertConditions(new MetricAnomalyAlertConditions()
                            .setSeverityRangeCondition(new SeverityCondition()
                                .setMaxAlertSeverity(AnomalySeverity.HIGH)))
                ))
            .setCrossMetricsOperator(MetricAnomalyAlertConfigurationsOperator.AND)
            .setIdOfHooksToAlert(Arrays.asList(hookId1, hookId2)));
```

## <a name="query-the-alert"></a>Consulta de la alerta

En este ejemplo se muestra la forma en que un usuario puede consultar las alertas desencadenadas para una configuración de detección de alertas y obtener anomalías para esa alerta.

```java
String alertConfigurationId = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx";
final OffsetDateTime startTime = OffsetDateTime.parse("2020-01-01T00:00:00Z");
final OffsetDateTime endTime = OffsetDateTime.parse("2020-09-09T00:00:00Z");
metricsAdvisorClient.listAlerts(
    alertConfigurationId,
        startTime, endTime)
    .forEach(alert -> {
        System.out.printf("AnomalyAlert Id: %s%n", alert.getId());
        System.out.printf("AnomalyAlert created on: %s%n", alert.getCreatedTime());

        // List anomalies for returned alerts
        metricsAdvisorClient.listAnomaliesForAlert(
            alertConfigurationId,
            alert.getId())
            .forEach(anomaly -> {
                System.out.printf("DataPoint Anomaly was created on: %s%n", anomaly.getCreatedTime());
                System.out.printf("DataPoint Anomaly severity: %s%n", anomaly.getSeverity().toString());
                System.out.printf("DataPoint Anomaly status: %s%n", anomaly.getStatus());
                System.out.printf("DataPoint Anomaly related series key: %s%n", anomaly.getSeriesKey().asMap());
            });
    });
```

Puede compilar la aplicación con:

```console
gradle build
```
### <a name="run-the-application"></a>Ejecución de la aplicación

Ejecute la aplicación con el objetivo `run`:

```console
gradle run
```