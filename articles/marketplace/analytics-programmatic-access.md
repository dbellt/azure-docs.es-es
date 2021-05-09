---
title: Paradigma del acceso mediante programación
description: Descripción del flujo de alto nivel del patrón de llamada de API para el análisis mediante programación. También se incluyen las API para acceder a los informes de análisis en el marketplace comercial de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: article
author: sayantanroy83
ms.author: sroy
ms.date: 3/08/2021
ms.openlocfilehash: 907b3020091de1be9a037d149de737c14de125ad
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108207346"
---
# <a name="programmatic-access-paradigm"></a>Paradigma del acceso mediante programación

En este diagrama se muestra el patrón de llamada de API que se usa para crear una nueva plantilla de informe, programar el informe personalizado y recuperar los datos de error.

[![Ilustra el patrón de llamada de API que se usa para crear una nueva plantilla de informe, programar el informe personalizado y recuperar los datos de error.](./media/analytics-programmatic-access/analytics-high-level-flow.png)](./media/analytics-programmatic-access/analytics-high-level-flow.png#lightbox)
***Figura 1: Flujo de alto nivel del patrón de llamada de API***

En esta lista se proporcionan más detalles sobre la figura 1.

1. La aplicación cliente puede definir el esquema o plantilla del informe personalizado llamando a la [API de creación de consultas de informes](#create-report-query-api). Como alternativa, puede usar una plantilla de informe (`QueryId`) de la [lista de consultas del sistema](analytics-system-queries.md).
1. Si se ejecuta correctamente, la API de creación de plantillas de informes devuelve el elemento `QueryId`.
1. A continuación, la aplicación cliente llama a la [API de creación de informes](#create-report-api) con el elemento `QueryID` junto con la fecha de inicio del informe, el intervalo de repetición, la periodicidad y un URI de devolución de llamada opcional.
1. Si se ejecuta correctamente, la [API de creación de informes](#create-report-api) devuelve el elemento `ReportID`.
1. La aplicación cliente recibe una notificación en el URI de devolución de llamada en cuanto los datos del informe están listos para su descarga.
1. A continuación, la aplicación cliente usa la [API de obtención de ejecuciones de informes](#get-report-executions-api) para consultar el estado del informe con el `Report ID` y el intervalo de fechas.
1. Si se ejecuta correctamente, se devuelve el vínculo de descarga del informe y la aplicación puede iniciar la descarga de los datos.

## <a name="report-query-language-specification"></a>Especificación del lenguaje de consulta de informes

Aunque proporcionamos [consultas del sistema](analytics-system-queries.md) que puede usar para crear informes, también puede crear sus propias consultas en función de sus necesidades empresariales. Para obtener más información sobre las consultas personalizadas, consulte [Especificación de consultas personalizadas](analytics-custom-query-specification.md).

## <a name="create-report-query-api"></a>API de creación de consultas de informes

Esta API ayuda a crear consultas personalizadas que definen el conjunto de datos del que deben exportarse las columnas y métricas. La API proporciona la flexibilidad para crear una nueva plantilla de informes en función de sus necesidades empresariales.

También puede usar las [consultas del sistema](analytics-system-queries.md) que proporcionamos. Cuando no se necesiten plantillas de informe personalizadas, puede llamar a la [API de creación de informes](#create-report-api) directamente con los [QueryIds](analytics-system-queries.md) de las consultas del sistema que proporcionamos.

En el ejemplo siguiente se muestra cómo crear una consulta personalizada para obtener el _uso normalizado y los cargos financieros estimados de las SKU de pago_ del conjunto de datos [ISVUsage](analytics-make-your-first-api-call.md#programmatic-api-call) del último mes.

*Sintaxis de la solicitud*

| Método | URI de solicitud |
| ------------ | ------------- |
| POST | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledQueries` |
|||

*Encabezado de solicitud*

| Encabezado | Tipo | Descripción |
| ------------- | ------------- | ------------- |
| Authorization | string | Necesario. El token de acceso de Azure Active Directory (Azure AD). El formato es `Bearer <token>`. |
| Content-Type | `string` | `application/JSON` |
||||

*Parámetro de ruta de acceso*

Ninguno

*Parámetro de consulta*

Ninguno

*Ejemplo de solicitud de carga*

```json
{
    "Name": "ISVUsageQuery",
    "Description": "Normalized Usage and Estimated Financial Charges for PAID SKUs",
    "Query": "SELECT UsageDate, NormalizedUsage, EstimatedExtendedChargePC FROM ISVUsage WHERE SKUBillingType = 'Paid' ORDER BY UsageDate DESC TIMESPAN LAST_MONTH"
}
```

*Glosario*

En esta tabla se proporcionan las definiciones clave de los elementos de la carga de solicitud.

| Parámetro | Obligatorio | Descripción | Valores permitidos |
| ------------ | ------------- | ------------- | ------------- |
| `Name` | Sí | Nombre descriptivo de la consulta | string |
| `Description` | No | Descripción de lo que devuelve la consulta | string |
| `Query` | Sí | Cadena de consulta del informe. | Tipo de datos: cadena<br>[Consulta personalizada](analytics-sample-queries.md) basada en la necesidad empresarial |
|||||

> [!NOTE]
> Para ejemplos de consultas personalizadas, vea [Ejemplos de consultas](analytics-sample-queries.md).

*Respuesta de ejemplo*

La carga de respuesta tiene la estructura siguiente:

Códigos de respuesta: 200, 400, 401, 403, 500

Ejemplo de carga de respuesta:

```json
{
  "value": [
        {
            "queryId": "78be43f2-e35f-491a-8cd5-78fe14194f9c",
            "name": " ISVUsageQuery",
            "description": "Normalized Usage and Estimated Financial Charges for PAID SKUs”,
            "query": " SELECT UsageDate, NormalizedUsage, EstimatedExtendedChargePC FROM ISVUsage WHERE SKUBillingType = 'Paid' ORDER BY UsageDate DESC TIMESPAN LAST_MONTH",
            "type": "userDefined",
            "user": "142344300",
            "createdTime": "2021-01-06T05:38:34Z"
        }
    ],
    "totalCount": 1,
    "message": "Query created successfully",
    "statusCode": 200
}
```

*Glosario*

En esta tabla se proporcionan las definiciones clave de los elementos de la respuesta.

| Parámetro | Description |
| ------------ | ------------- |
| `QueryId` | Identificador único universal (UUID) de la consulta que creó |
| `Name` | Nombre descriptivo dado a la consulta en la carga de la solicitud |
| `Description` | Descripción proporcionada durante la creación de la consulta. |
| `Query` | Consulta de informe pasada como entrada durante la creación de la consulta |
| `Type` | Establézcala en `userDefined` |
| `User` | Id. de usuario usado para crear la consulta |
| `CreatedTime` | Hora UTC en que se creó la consulta en el formato aaaa-MM-ddTHH:mm:ssZ |
| `TotalCount` | Número de conjuntos de datos en la matriz de valores. |
| `StatusCode` | Código de resultado<br>Los valores posibles son 200, 400, 401, 403 y 500. |
| `message` | Mensaje de estado de la ejecución de la API. |
|||

## <a name="create-report-api"></a>API de creación de informes

Al crear una plantilla de informe personalizada correctamente y recibir el elemento `QueryID` como parte de la respuesta de [Creación de consultas de informes](#create-report-query-api), se puede llamar a esta API para programar la ejecución de una consulta a intervalos regulares. Puede establecer una frecuencia y una programación para la entrega del informe. En el caso de las consultas del sistema que proporcionamos, también se puede llamar a la API de creación de informes con [QueryId](analytics-sample-queries.md).

*Sintaxis de la solicitud*

| Método | URI de solicitud |
| ------------ | ------------- |
| POST | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport` |
|||

*Encabezado de solicitud*

| Encabezado | Tipo | Descripción |
| ------ | ---- | ----------- |
| Authorization | string | Necesario. El token de acceso de Azure Active Directory (Azure AD). El formato es `Bearer <token>`. |
| Tipo de contenido | string | `application/JSON` |
||||

*Parámetro de ruta de acceso*

Ninguno

*Parámetro de consulta*

Ninguno

*Ejemplo de solicitud de carga*

```json
{
  "ReportName": "ISVUsageReport",
  "Description": "Normalized Usage and Estimated Financial Charges for PAID SKUs",
  "QueryId": "78be43f2-e35f-491a-8cd5-78fe14194f9c ",
  "StartTime": "2021-01-06T19:00:00Z ",
  "RecurrenceInterval": 48,
  "RecurrenceCount": 20,
  "Format": "csv",
  "CallbackUrl": "https://<SampleCallbackUrl>"
}
```

*Glosario*

En esta tabla se proporcionan las definiciones clave de los elementos de la carga de solicitud.

| Parámetro | Obligatorio | Descripción | Valores permitidos |
| ------------ | ------------- | ------------- | ------------- |
| `ReportName` | Sí | Nombre asignado al informe. | string |
| `Description` | No | Descripción del informe creado | string |
| `QueryId` | Sí | Id. de consulta del informe. | string |
| `StartTime` | Sí | Marca de tiempo UTC en la que comenzará la generación del informe.<br>El formato debe ser aaaa-MM-ddTHH:mm:ssZ. | string |
| `RecurrenceInterval` | Sí | Frecuencia en horas en que se debe generar el informe.<br>El valor mínimo es 4 y el máximo es 90. | integer |
| `RecurrenceCount` | No | Número de informes que se van a generar. | integer |
| `Format` | No | Formato del archivo exportado.<br>El formato predeterminado es .CVS. | CSV/TSV |
| `CallbackUrl` | No | Dirección URL de acceso público que se puede configurar opcionalmente como el destino de la devolución de llamada. | Cadena (dirección URL http) |
| `ExecuteNow` | No | Este parámetro debe usarse para crear un informe que se ejecutará una sola vez. `StartTime`, `RecurrenceInterval` y `RecurrenceCount` se omiten si se establece en `true`. El informe se ejecuta inmediatamente de manera asincrónica. | true/false |
| `QueryStartTime` | No | Opcionalmente, especifica la hora de inicio de la consulta que extrae los datos. Este parámetro solo es aplicable a un informe de una sola ejecución que tiene `ExecuteNow` establecido en `true`. El formato debe ser aaaa-MM-ddTHH:mm:ssZ. | Marca de tiempo como string |
| `QueryEndTime` | No | Opcionalmente, especifica la hora de fin de la consulta que extrae los datos. Este parámetro solo es aplicable a un informe de una sola ejecución que tiene `ExecuteNow` establecido en `true`. El formato debe ser aaaa-MM-ddTHH:mm:ssZ. | Marca de tiempo como string |
|||||

*Respuesta de muestra*

La carga de respuesta tiene la estructura siguiente:

Código de respuesta: 200, 400, 401, 403, 404, 500

Carga de respuesta:

```json
{
  "Value": [
    {
            "reportId": "72fa95ab-35f5-4d44-a1ee-503abbc88003",
            "reportName": "ISVUsageReport",
            "description": "Normalized Usage and Estimated Financial Charges for PAID SKUs",
            "queryId": "78be43f2-e35f-491a-8cd5-78fe14194f9c",
            "query": "SELECT UsageDate, NormalizedUsage, EstimatedExtendedChargePC FROM ISVUsage WHERE SKUBillingType = 'Paid' ORDER BY UsageDate DESC TIMESPAN LAST_MONTH",
            "user": "142344300",
            "createdTime": "2021-01-06T05:46:00Z",
            "modifiedTime": null,
            "startTime": "2021-01-06T19:00:00Z",
            "reportStatus": "Active",
            "recurrenceInterval": 48,
            "recurrenceCount": 20,
            "callbackUrl": "https://<SampleCallbackUrl>",
            "format": "csv"    
    }
  ],
  "TotalCount": 1,
  "Message": "Report created successfully",
  "StatusCode": 200
}
```

*Glosario*

En esta tabla se proporcionan las definiciones clave de los elementos de la respuesta.

| Parámetro | Description |
| ------------ | ------------- |
| `ReportId` | Identificador único universal (UUID) del informe que creó. |
| `ReportName` | Nombre dado al informe en la carga útil de la solicitud. |
| `Description` | Descripción proporcionada durante la creación del informe. |
| `QueryId` | Id. de consulta pasado en el momento en que creó el informe. |
| `Query` | Texto de la consulta que se ejecutará para este informe. |
| `User` | Id. de usuario usado para crear el informe. |
| `CreatedTime` | Hora UTC en que se creó el informe en el formato aaaa-MM-ddTHH:mm:ssZ. |
| `ModifiedTime` | Hora UTC en que se modificó el informe en el formato aaaa-MM-ddTHH:mm:ssZ. |
| `StartTime` | Hora UTC en que se iniciará la ejecución del informe en el formato aaaa-MM-ddTHH:mm:ssZ |
| `ReportStatus` | Estado de la ejecución del informe. Los valores posibles son **En pausa**, **Activo** e **Inactivo**. |
| `RecurrenceInterval` | Intervalo de periodicidad proporcionado durante la creación del informe. |
| `RecurrenceCount` | Recuento de periodicidad proporcionado durante la creación del informe. |
| `CallbackUrl` | Dirección URL de devolución de llamada proporcionada en la solicitud. |
| `Format` | Formato de los archivos de informe. Los valores posibles son CSV o TSV. |
| `TotalCount` | Número de conjuntos de datos en la matriz de valores. |
| `StatusCode` | Código de resultado<br>Los valores posibles son 200, 400, 401, 403 y 500. |
| `message` | Mensaje de estado de la ejecución de la API. |
|||

## <a name="get-report-executions-api"></a>API de obtención de ejecuciones de informes

Puede usar este método para consultar el estado de la ejecución de un informe con el elemento `ReportId` recibido de la [API de creación de informes](#create-report-api). El método devuelve el vínculo de descarga del informe si el informe está listo para su descarga. De lo contrario, el método devolverá el estado. También puede usar esta API para obtener todas las ejecuciones que se han producido para un informe determinado.

> [!IMPORTANT]
> Esta API tiene parámetros de consulta predeterminados establecidos para `executionStatus=Completed` y `getLatestExecution=true`. Por lo tanto, si se llama a la API antes de la primera ejecución correcta del informe devolverá 404. Las ejecuciones pendientes se pueden obtener estableciendo `executionStatus=Pending`.

*Sintaxis de la solicitud*

| Método | URI de solicitud |
| ------------ | ------------- |
| Obtener | `https://api.partnercenter.microsoft.com/insights/v1/cmp/ScheduledReport/execution/{reportId}?executionId={executionId}&executionStatus={executionStatus}&getLatestExecution={getLatestExecution}` |
|||

*Encabezado de solicitud*

| Encabezado | Tipo | Descripción |
| ------ | ------ | ------ |
| Authorization | string | Necesario. El token de acceso de Azure Active Directory (Azure AD). El formato es `Bearer <token>`. |
| Tipo de contenido | string | `application/json` |
||||

*Parámetro de ruta de acceso*

Ninguno

*Parámetro de consulta*

| Nombre de parámetro | Obligatorio | Tipo | Descripción |
| ------------ | ------------- | ------------- | ------------- |
| `reportId` | Sí | string | Filtrar para obtener los detalles de ejecución solo de los informes con el elemento `reportId` especificado en este argumento. Se pueden especificar varios `reportIds` separándolos con un punto y coma ";". |
| `executionId` | No | string | Filtrar para obtener los detalles de solo los informes con el elemento `executionId` especificado en este argumento. Se pueden especificar varios `executionIds` separándolos con un punto y coma ";". |
| `executionStatus` | No | string/enum | Filtrar para obtener los detalles de solo los informes con el elemento `executionStatus` especificado en este argumento.<br>Los valores válidos son `Pending`, `Running`, `Paused` y `Completed`. <br>El valor predeterminado es `Completed`. Se pueden especificar varios estados separándolos con un punto y coma ";". |
| `getLatestExecution` | No | boolean | La API devolverá los detalles de la ejecución más reciente del informe.<br>De manera predeterminada, este parámetro está establecido en `true`. Si opta por pasar el valor de este parámetro como `false`, la API devolverá las instancias de ejecución de los últimos 90 días. |
|||||

*Carga de solicitud*

Ninguno

*Respuesta de muestra*

La carga de respuesta tiene la estructura siguiente:

Códigos de respuesta: 200, 400, 401, 403, 404, 500

Ejemplo de carga de respuesta:

```json
{
    "value": [
        {
            "executionId": "a0bd78ad-1a05-40fa-8847-8968b718d00f",
            "reportId": "72fa95ab-35f5-4d44-a1ee-503abbc88003",
            "recurrenceInterval": 4,
            "recurrenceCount": 10,
            "callbackUrl": null,
            "format": "csv",
            "executionStatus": "Completed",
            "reportAccessSecureLink": "https://<path to report for download>",
            "reportExpiryTime": null,
            "reportGeneratedTime": "2021-01-13T14:40:46Z"
        }
    ],
    "totalCount": 1,
    "message": null,
    "statusCode": 200
}
```

Una vez completada la ejecución del informe, se muestra el estado de ejecución `Completed`. Puede descargar el informe seleccionando la dirección URL en `reportAccessSecureLink`.

*Glosario*

Definiciones clave de los elementos de la respuesta.

| Parámetro | Description |
| ------------ | ------------- |
| `ExecutionId` | Identificador único universal (UUID) de la instancia de ejecución. |
| `ReportId` | Id. de informe asociado a la instancia de ejecución. |
| `RecurrenceInterval` | Intervalo de periodicidad proporcionado durante la creación del informe. |
| `RecurrenceCount` | Recuento de periodicidad proporcionado durante la creación del informe. |
| `CallbackUrl` | Dirección URL de devolución de llamada asociada a la instancia de ejecución. |
| `Format` | Formato del archivo generado al final de la ejecución. |
| `ExecutionStatus` | Estado de la instancia de ejecución del informe.<br>Los valores válidos son `Pending`, `Running`, `Paused` y `Completed`. |
| `ReportAccessSecureLink` | Vínculo a través del cual se puede acceder al informe de forma segura. |
| `ReportExpiryTime` | Hora UTC después de la cual el vínculo de informe expirará en el formato aaaa-MM-ddTHH:mm:ssZ. |
| `ReportGeneratedTime` | Hora UTC a la que se generó el informe en el formato aaaa-MM-ddTHH:mm:ssZ. |
| `TotalCount` | Número de conjuntos de datos en la matriz de valores. |
| `StatusCode` | Código de resultado <br>Los valores posibles son 200, 400, 401, 403, 404 y 500. |
| `message` | Mensaje de estado de la ejecución de la API. |
|||

## <a name="next-steps"></a>Pasos siguientes
- Puede probar las API a través de la [dirección URL de la API de Swagger](https://api.partnercenter.microsoft.com/insights/v1/cmp/swagger/index.html).
- [Introducción al acceso mediante programación a los datos de análisis](analytics-get-started.md)
