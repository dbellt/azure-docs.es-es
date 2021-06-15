---
title: Supervisión, métricas y registros sin procesar para Azure CDN
description: En este artículo se describe cómo configurar y usar la supervisión, las métricas y los registros sin procesar de Azure CDN.
services: cdn
author: asudbring
manager: KumudD
ms.service: azure-cdn
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 11/23/2020
ms.author: yuajia
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: fd117532267672e152b6fd6f6ce382c4bda39687
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693960"
---
# <a name="real-time-monitoring-metrics-and-access-logs-for-azure-cdn"></a>Supervisión en tiempo real, métricas y registros de acceso para Azure CDN
Con Azure CDN de Microsoft, puede supervisar los recursos de las siguientes maneras para ayudarle a solucionar problemas, realizar seguimientos y depurar incidencias. 

* Los registros sin procesar proporcionan abundante información sobre cada solicitud que CDN recibe. Los registros sin procesar son diferentes de los registros de actividad. Los registros de actividad proporcionan visibilidad sobre las operaciones llevadas a cabo en los recursos de Azure.
* Las métricas, que muestran cuatro métricas clave en CDN, incluida la proporción de aciertos de bytes, el recuento de solicitudes, el tamaño de respuesta y la latencia total. También proporciona diferentes dimensiones para desglosar las métricas.
* La alerta, que permite al cliente configurar alertas para las métricas clave.
* Las métricas adicionales, que permiten a los clientes usar Azure Log Analytics para habilitar métricas de valor adicionales. También se proporcionan ejemplos de consultas para otras métricas en Azure Log Analytics.

> [!IMPORTANT]
> La característica Registros sin procesar de HTTP está disponible para Azure CDN de Microsoft.

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar. 

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en [https://portal.azure.com](https://portal.azure.com).

## <a name="configuration---azure-portal"></a>Configuración: Azure Portal

Para configurar los registros sin procesar para Azure CDN desde el perfil de Microsoft: 

1. En el menú de Azure Portal, seleccione **Todos los recursos** >>  **\<your-CDN-profile>** .

2. En **Supervisión**, seleccione **Configuración de diagnóstico**.

3. Seleccione **+ Agregar configuración de diagnóstico**.

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-01.png" alt-text="Adición de configuración de diagnóstico para el perfil de CDN." border="true":::
    
    > [!IMPORTANT]
    > Los registros sin procesar solo están disponibles en el nivel de perfil, mientras que los registros de código de estado HTTP agregados están disponibles en el nivel de punto de conexión.

4. En **Configuración de diagnóstico**, escriba un nombre para la configuración de diagnóstico en **Nombre de la configuración de diagnóstico**.

5. Seleccione **AzureCdnAccessLog** y establezca la retención en días.

6. Seleccione **Detalles del destino**. Las opciones de destino son:
    * **Enviar a Log Analytics**
        * Seleccione la **Suscripción** y el **Área de trabajo de Log Analytics**.
    * **Archivar en una cuenta de almacenamiento**
        * Seleccione la **Suscripción** y la **Cuenta de almacenamiento**.
    * **Transmisión a un centro de eventos**
        * Seleccione la **Suscripción**, el **Espacio de nombres del centro de eventos**, el **Nombre del centro de eventos (opcional)** y el **Nombre de la directiva del centro de eventos**.

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-02.png" alt-text="Configuración del destino para la configuración de registro." border="true":::

7. Seleccione **Guardar**.

## <a name="configuration---azure-powershell"></a>Configuración: Azure PowerShell

Use [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) para configurar los valores de diagnóstico de los registros sin procesar.

Los datos de retención se definen mediante la opción **-RetentionInDays** del comando.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

### <a name="enable-diagnostic-logs-in-a-storage-account"></a>Habilitación de registros de diagnóstico en una cuenta de almacenamiento

1. Inicie sesión en Azure PowerShell:

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```

2. Para habilitar los registros de diagnóstico en una cuenta de almacenamiento, escriba estos comandos. Reemplace las variables por sus valores:

    ```azurepowershell-interactive
    ## Variables for the commands ##
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $storageacct = <your-storage-account-name>
    $diagname = <your-diagnostic-setting-name>
    $days = '30'

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $storage = Get-AzStorageAccount -ResourceGroupName $rsg -Name $storageacct

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -StorageAccountId $storage.id -Enabled $true -Category AzureCdnAccessLog -RetentionEnabled 1 -RetentionInDays $days
    ```

### <a name="enable-diagnostics-logs-for-log-analytics-workspace"></a>Habilitación de registros de diagnóstico en el área de trabajo de Log Analytics

1. Inicie sesión en Azure PowerShell:

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Para habilitar los registros de diagnóstico en un área de trabajo de Log Analytics, escriba estos comandos. Reemplace las variables por sus valores:

    ```azurepowershell-interactive
    ## Variables for the commands ##
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $workspacename = <your-log-analytics-workspace-name>
    $diagname = <your-diagnostic-setting-name>
    $days = '30'

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $workspace = Get-AzOperationalInsightsWorkspace -ResourceGroupName $rsg -Name $workspacename

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -WorkspaceId $workspace.ResourceId -Enabled $true -Category AzureCdnAccessLog -RetentionEnabled 1 -RetentionInDays $days
    ```
### <a name="enable-diagnostics-logs-for-event-hub-namespace"></a>Habilitación de registros de diagnóstico en un espacio de nombres del centro de eventos

1. Inicie sesión en Azure PowerShell:

    ```azurepowershell-interactive
    Connect-AzAccount 
    ```
2. Para habilitar los registros de diagnóstico en un espacio de nombres del centro de eventos, escriba estos comandos. Reemplace las variables por sus valores:

    ```azurepowershell-interactive
    ## Variables for the commands ##
    $rsg = <your-resource-group-name>
    $cdnprofile = <your-cdn-profile-name>
    $cdnendpoint = <your-cdn-endpoint-name>
    $evthubnamespace = <your-event-hub-namespace-name>
    $diagname = <your-diagnostic-setting-name>

    $cdn = Get-AzCdnEndpoint -ResourceGroupName $rsg -ProfileName $cdnprofile -EndpointName $cdnendpoint

    $eventhub = Get-AzEventHubNamespace -ResourceGroupName $rsg -Name $eventhubname

    Set-AzDiagnosticSetting -Name $diagname -ResourceId $cdn.id -EventHubName $eventhub.id -Enabled $true -Category AzureCdnAccessLog -RetentionEnabled 1 -RetentionInDays $days
    ```

## <a name="raw-logs-properties"></a>Propiedades de los registros sin procesar

En la actualidad, Azure CDN del servicio de Microsoft proporciona los registros sin procesar. Los registros sin procesar proporcionan las solicitudes de API individuales con el siguiente esquema para cada entrada: 

| Propiedad  | Descripción |
| ------------- | ------------- |
| BackendHostname | Si la solicitud se reenvía a un back-end, este campo representa el nombre de host del back-end. Este campo estará en blanco si la solicitud se redirige o reenvía a una caché regional (si el almacenamiento en caché está habilitado para la regla de enrutamiento). |
| CacheStatus | En el caso de los escenarios de almacenamiento en caché, este campo define el número de aciertos y errores de caché en el POP |
| ClientIp | Dirección IP del cliente que realizó la solicitud. Si hubiera un encabezado X-Forwarded-For en la solicitud, la dirección IP del cliente se seleccionaría del mismo. |
| ClientPort | Puerto IP del cliente que realizó la solicitud. |
| HttpMethod | Método HTTP utilizado por la solicitud. |
| HttpStatusCode | El código de estado HTTP devuelto desde el servidor proxy. |
| HttpStatusDetails | Estado resultante en la solicitud. El significado de este valor de cadena puede encontrarse en una tabla de referencia de estado. |
| HttpVersion | Tipo de la solicitud o conexión. |
| POP | Nombre corto del perímetro en el que ha aterrizado la solicitud. |
| RequestBytes | El tamaño del mensaje de solicitud HTTP en bytes, incluidos los encabezados de solicitud y el cuerpo de solicitud. |
| RequestUri | URI de la solicitud recibida. |
| ResponseBytes | Bytes enviados por el servidor back-end como respuesta.  |
| RoutingRuleName | El nombre de la regla de enrutamiento que coincidió con la solicitud. |
| RulesEngineMatchNames | Los nombres de las reglas que coincidieron con la solicitud. |
| SecurityProtocol | La versión del protocolo TLS/SSL utilizada por la solicitud o null si no hay cifrado. |
| SentToOriginShield </br> (en desuso)* **Consulte las notas sobre el desuso en la sección siguiente.**| Si es True, significa que la solicitud se respondió desde la memoria caché del escudo de origen en lugar del PoP perimetral. El escudo de origen es una memoria caché primaria que se usa para mejorar la proporción de aciertos de caché. |
| isReceivedFromClient | Si es true, significa que la solicitud procedía del cliente. Si es false, la solicitud es una línea no ejecutada en el servidor perimetral (POP secundario) y se responde desde el escudo de origen (POP primario). |
| TimeTaken | Período de tiempo desde el primer byte de la solicitud en Front Door hasta el último byte de la respuesta, en segundos. |
| TrackingReference | La cadena de referencia exclusiva que identifica una solicitud atendida por Front Door, que también se envía como encabezado X-Azure-Ref al cliente. Se requiere para buscar los detalles en los registros de acceso para una solicitud específica. |
| UserAgent | Tipo de explorador utilizado por el cliente. |
| ErrorInfo | Este campo contiene el tipo específico de error para restringir el área de solución de problemas. </br> Los valores posibles son: </br> **NoError**: indica que no se encontraron errores. </br> **CertificateError**: error genérico de certificado SSL.</br> **CertificateNameCheckFailed**: el nombre de host del certificado SSL no es válido o no coincide. </br> **ClientDisconnected**: error de solicitud debido a la conexión de red del cliente. </br> **UnspecifiedClientError**: error genérico de cliente. </br> **InvalidRequest**: solicitud no válida. Podría producirse debido a un encabezado, un cuerpo y una dirección URL con formato incorrecto. </br> **DNSFailure**: error de DNS. </br> **DNSNameNotResolved**: no se pudo resolver el nombre o la dirección del servidor. </br> **OriginConnectionAborted**: la conexión con el origen se detuvo repentinamente. </br> **OriginConnectionError**: error genérico de conexión con el origen. </br> **OriginConnectionRefused**: no se pudo establecer la conexión con el origen. </br> **OriginError**: error genérico del origen. </br> **OriginInvalidResponse**: el origen devolvió una respuesta no válida o desconocida. </br> **OriginTimeout**: el período de tiempo de espera de la solicitud del origen expiró. </br> **ResponseHeaderTooBig**: el origen devolvió un encabezado de respuesta demasiado grande. </br> **RestrictedIP**: la solicitud se bloqueó debido a una IP restringida. </br> **SSLHandshakeError**: no se puede establecer la conexión con el origen debido a un error del protocolo de enlace SSL. </br> **UnspecifiedError**: se produjo un error que no coincidía con ninguno de los errores de la tabla. |
| TimeToFirstByte | La duración en milisegundos desde que Microsoft CDN recibe la solicitud hasta el momento en que se envía el primer byte al cliente. El tiempo se mide solo desde el lado de Microsoft. No se miden los datos del lado cliente. |
> [!NOTE]
> Para ver los registros en el perfil de Log Analytics, ejecute una consulta. Una consulta de ejemplo tendría el siguiente aspecto:
    ```
    AzureDiagnostics | where Category == "AzureCdnAccessLog"
    ```

### <a name="sent-to-origin-shield-deprecation"></a>Desuso de la propiedad de envío al escudo de origen
La propiedad **isSentToOriginShield** del registro sin procesar ha quedado en desuso y se ha reemplazado por un nuevo campo, **isReceivedFromClient**. Use el nuevo campo si aún usa el campo en desuso. 

Los registros sin procesar incluyen los registros generados tanto desde el servidor perimetral de la red CDN (POP secundario) como desde el escudo de origen. El escudo de origen hace referencia a los nodos primarios que están ubicados de manera estratégica por todo el planeta. Estos nodos se comunican con los servidores de origen y reducen la carga de tráfico en el origen. 

Por cada solicitud que va al escudo de origen, hay dos entradas de registro:

* Una para los nodos perimetrales y
* otra para el escudo de origen. 

Para diferenciar la salida o las respuestas de los nodos perimetrales de las del escudo de origen, puede usar el campo **isReceivedFromClient** para obtener los datos correctos. 

Si el valor es false, significa que la solicitud se responde desde el escudo de origen a los nodos perimetrales. Este enfoque es eficaz para comparar los registros sin procesar con los datos de facturación. No se paga por la salida del escudo de origen a los nodos perimetrales. Se paga por la salida de los nodos perimetrales a los clientes. 

**Ejemplo de consulta Kusto para excluir los registros generados en el escudo de origen en Log Analytics.**

```kusto
AzureDiagnostics 
| where OperationName == "Microsoft.Cdn/Profiles/AccessLog/Write" and Category == "AzureCdnAccessLog"  
| where isReceivedFromClient == true

```

> [!IMPORTANT]
> La característica de registros sin formato HTTP está disponible automáticamente para los perfiles creados o actualizados después del **25 de febrero de 2020**. En el caso de los perfiles de CDN creados anteriormente, debe actualizar el punto de conexión de CDN después de configurar el registro. Por ejemplo, puede navegar al filtrado geográfico en puntos de conexión de CDN y bloquear cualquier país o región que no sea relevante para su carga de trabajo y pulsar Guardar.


## <a name="metrics"></a>Métricas
Azure CDN de Microsoft se integra con Azure Monitor y publica cuatro métricas de CDN para ayudar a realizar seguimientos, solucionar problemas y depurar incidencias. 

Las métricas se muestran en gráficos y se puede acceder a ellas a través de PowerShell, la CLI y la API. Las métricas de CDN son gratuitas.

Azure CDN de Microsoft mide y envía sus métricas a intervalos de 60 segundos. Las métricas pueden tardar hasta 3 minutos en aparecer en el portal. 

Para obtener más información, vea [Información general sobre las métricas en Microsoft Azure](../azure-monitor/essentials/data-platform-metrics.md).

**Métricas compatibles con Azure CDN de Microsoft**

| Métricas  | Descripción | Dimensions |
| ------------- | ------------- | ------------- |
| Proporción de aciertos de bytes* | El porcentaje de salida de la memoria caché de CDN, calculado con respecto a la salida total. | Punto de conexión |
| RequestCount | El número de solicitudes de cliente que atiende CDN. | Punto de conexión </br> País del cliente. </br> Región del cliente. </br> Estado de HTTP </br> Código de estado HTTP. |
| ResponseSize | Número de bytes enviados como respuestas desde el perímetro de CDN a los clientes. |Punto de conexión </br> País del cliente. </br> Región del cliente. </br> Estado de HTTP </br> Código de estado HTTP. |
| TotalLatency | El tiempo total desde la solicitud de cliente recibida por CDN **hasta el último byte de respuesta enviado desde CDN al cliente**. |Punto de conexión </br> País del cliente. </br> Región del cliente. </br> Estado de HTTP </br> Código de estado HTTP. |

***Proporción de aciertos de bytes = (salida desde el perímetro - salida desde el origen)/salida desde el perímetro**

Escenarios excluidos en el cálculo de la proporción de aciertos de bytes:

* No se configura ninguna caché explícitamente a través del motor de reglas o del comportamiento del almacenamiento en caché de cadenas de consulta.
* Se configura la directiva de control de caché con una caché privada o sin almacén.

### <a name="metrics-configuration"></a>Configuración de métricas

1. En el menú de Azure Portal, seleccione **Todos los recursos** >>  **\<your-CDN-profile>** .

2. En **Supervisión**, seleccione **Métricas**:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-03.png" alt-text="Métricas para el perfil de CDN." border="true":::

3. Seleccione **Agregar métrica** y luego la métrica que desea agregar:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-04.png" alt-text="Adición y selección de una métrica para el perfil de CDN." border="true":::

4. Seleccione **Agregar filtro** para agregar un filtro:
    
    :::image type="content" source="./media/cdn-raw-logs/raw-logs-05.png" alt-text="Aplicación de un filtro a la métrica." border="true":::

5. Seleccione **Aplicar** división para ver la tendencia por dimensiones diferentes:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-06.png" alt-text="Aplicación de la división a la métrica." border="true":::

6. Seleccione **Nuevo gráfico** para agregar un nuevo gráfico:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-07.png" alt-text="Adición de un nuevo gráfico a la vista de métrica." border="true":::

### <a name="alerts"></a>Alertas

Puede configurar alertas en CDN de Microsoft seleccionando **Supervisión** >> **Alertas**.

Seleccione **Nueva regla de alertas** para las métricas que aparecen en la sección Métricas:

:::image type="content" source="./media/cdn-raw-logs/raw-logs-08.png" alt-text="Configuración de alertas para el punto de conexión de CDN." border="true":::

La alerta se cobrará en función de Azure Monitor. Para más información sobre alertas, consulte [Alertas de Azure Monitor](../azure-monitor/alerts/alerts-overview.md).

### <a name="additional-metrics"></a>Métricas adicionales
Puede habilitar métricas adicionales con Azure Log Analytics y registros sin procesar por un costo adicional.

1. Siga los pasos anteriores de habilitación de los diagnósticos para enviar el registro sin procesar a Log Analytics.

2. Seleccione el área de trabajo de Log Analytics que creó:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-09.png" alt-text="Selección de un área de trabajo de Log Analytics" border="true":::   

3. Seleccione **Registros** en **General** en el área de trabajo de Log Analytics.  Luego seleccione **Tareas iniciales**:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-10.png" alt-text="Área de trabajo de recursos de Log Analytics." border="true":::   
 
4. Seleccione **Perfiles de CDN**.  Seleccione una consulta de ejemplo para ejecutar o cerrar la pantalla de ejemplo para escribir una consulta personalizada:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-11.png" alt-text="Pantalla de consulta de ejemplo." border="true":::   

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-12.png" alt-text="Ejecución de la consulta." border="true":::   

4. Para ver los datos por gráfico, seleccione **Gráfico**.  Seleccione **Anclar al panel** para anclar el gráfico al panel de Azure:

    :::image type="content" source="./media/cdn-raw-logs/raw-logs-13.png" alt-text="Ancle un gráfico al panel." border="true"::: 

## <a name="next-steps"></a>Pasos siguientes
En este artículo, ha habilitado los registros sin procesar de HTTP para el servicio CDN de Microsoft.

Para más información sobre Azure CDN y los otros servicios de Azure que se mencionan en este artículo, consulte:

* [Análisis](cdn-log-analysis.md) de patrones de uso de Azure CDN.

* Más información sobre [Azure Monitor](../azure-monitor/overview.md).

* Configuración de [Log Analytics en Azure Monitor](../azure-monitor/logs/log-analytics-tutorial.md).
