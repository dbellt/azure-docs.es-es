---
title: Supervisión de métricas y registro en Azure Front Door | Microsoft Docs
description: Este artículo describe las diferentes métricas y registros de acceso que admite Azure Front Door.
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: yuajia
ms.openlocfilehash: a972123604cafbfc4c6753c1fe3ad21147b53b2d
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106550666"
---
# <a name="monitoring-metrics-and-logs-in-azure-front-door"></a>Supervisión de métricas y registro en Azure Front Door

Con Azure Front Door, puede supervisar los recursos de las siguientes maneras:

- **Métricas**. Azure Front Door tiene actualmente ocho métricas para ver los contadores de rendimiento.
- **Registros**. Los registros de actividad y diagnóstico permiten que un recurso guarde o consuma datos de rendimiento, acceso u otros con fines de supervisión.

##  <a name="metrics"></a><a name="metrics"></a>Métricas

Las métricas son una característica de determinados recursos de Azure en los que puede ver contadores de rendimiento en el portal. Las métricas siguientes están disponibles en Front Door:

| Métrica | Nombre de métrica para mostrar | Unidad | Dimensions | Descripción |
| --- | --- | --- | --- | --- |
| RequestCount | Recuento de solicitudes | Count | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Número de solicitudes de cliente que atiende Front Door.  |
| RequestSize | Tamaño de la solicitud | Bytes | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Número de bytes enviados como solicitudes de clientes a Front Door. |
| ResponseSize | Tamaño de la respuesta | Bytes | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | Número de bytes enviados como respuestas de Front Door a los clientes. |
| TotalLatency | Latencia total | Milisegundos | HttpStatus</br>HttpStatusGroup</br>ClientRegion</br>ClientCountry | El tiempo total desde la solicitud de cliente recibida por Front Door hasta el último byte de respuesta enviado desde ADF al cliente. |
| BackendRequestCount | Recuento de solicitudes de back-end | Count | HttpStatus</br>HttpStatusGroup</br>Back-end | Número de solicitudes enviadas de Front Door a los servidores back-end. |
| BackendRequestLatency | Latencia de las solicitudes de back-end | Milisegundos | Back-end | Tiempo calculado desde que Front Door envía la solicitud al servidor back-end hasta que Front Door recibe el último byte de respuesta del servidor back-end. |
| BackendHealthPercentage | Porcentaje de estado del back-end | Percent | Back-end</br>BackendPool | El porcentaje de sondeos de estado correctos de Front Door a los servidores back-ends. |
| WebApplicationFirewallRequestCount | Recuento de solicitudes del firewall de aplicaciones web | Count | PolicyName</br>RuleName</br>Acción | Número de solicitudes de cliente procesadas por la seguridad del nivel de aplicación de Front Door. |

## <a name="activity-logs"></a><a name="activity-log"></a>Registros de actividad

Los registros de actividad proporcionan información sobre las operaciones realizadas en Front Door. También determinan qué, quién y cuándo para cualquier operación de escritura (put, post o delete) realizada en Front Door.

>[!NOTE]
>Los registros de actividad no incluyen operaciones de lectura (get). Tampoco incluyen operaciones realizadas con Azure Portal o la Management API original.

Acceda a registros de actividad en Front Door o a los registros de todos los recursos de Azure en Azure Monitor. Para ver los registros de actividad:

1. Seleccione la instancia de Front Door.
2. Seleccione **Registro de actividad**.

    :::image type="content" source="./media/front-door-diagnostics/activity-log.png" alt-text="Registro de actividad":::

3. Elija un ámbito de filtrado y, a continuación, seleccione **Aplicar**.

## <a name="diagnostic-logs"></a><a name="diagnostic-logging"></a>Registros de diagnóstico
Los registros de diagnóstico proporcionan información valiosa acerca de las operaciones y los errores que son importantes para la auditoría, así como para solucionar problemas. Los registros de diagnóstico son diferentes de los registros de actividad.

Los registros de actividad proporcionan información sobre las operaciones llevadas a cabo en los recursos de Azure. Los registros de diagnóstico proporcionan conclusiones detalladas sobre las operaciones que ha hecho el recurso. Para más información, vea [Información general sobre los registros de diagnóstico de Azure](../azure-monitor/essentials/platform-logs-overview.md).

:::image type="content" source="./media/front-door-diagnostics/diagnostic-log.png" alt-text="Registros de diagnóstico":::

Para configurar los registros de diagnóstico para Front Door:

1. Seleccione su instancia del servicio Azure Front Door.

2. Seleccione **Configuración de diagnóstico**.

3. Seleccione **Activar diagnósticos**. Archive los registros de diagnóstico junto con las métricas en una cuenta de almacenamiento, transmítalos en secuencias a un centro de eventos o envíelos a los registros de Azure Monitor.

Front Door actualmente proporciona los registros de diagnóstico. Los registros de diagnóstico proporcionan solicitudes API individuales con cada entrada con el siguiente esquema:

| Propiedad  | Descripción |
| ------------- | ------------- |
| BackendHostname | Si la solicitud se reenvió a un back-end, este campo representa el nombre de host del back-end. Este campo estará en blanco si la solicitud se redirige o reenvía a una caché regional (si el almacenamiento en caché está habilitado para la regla de enrutamiento). |
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
| ErrorInfo | Este campo contiene el tipo específico de error para facilitar la solución de problemas. </br> Los valores posibles son: </br> **NoError**: indica que no se encontró ningún error. </br> **CertificateError**: error genérico de certificado SSL.</br> **CertificateNameCheckFailed**: el nombre de host del certificado SSL no es válido o no coincide. </br> **ClientDisconnected**: error de solicitud debido a la conexión de red del cliente. </br> **UnspecifiedClientError**: error genérico de cliente. </br> **InvalidRequest**: solicitud no válida. Podría producirse debido a un encabezado, un cuerpo y una dirección URL con formato incorrecto. </br> **DNSFailure**: error de DNS. </br> **DNSNameNotResolved**: no se pudo resolver el nombre o la dirección del servidor. </br> **OriginConnectionAborted**: la conexión con el origen se detuvo repentinamente. </br> **OriginConnectionError**: error genérico de conexión con el origen. </br> **OriginConnectionRefused**: no se pudo establecer la conexión con el origen. </br> **OriginError**: error genérico del origen. </br> **OriginInvalidResponse**: el origen devolvió una respuesta no válida o desconocida. </br> **OriginTimeout**: el período de tiempo de espera de la solicitud del origen expiró. </br> **ResponseHeaderTooBig**: el origen devolvió un encabezado de respuesta demasiado grande. </br> **RestrictedIP**: la solicitud se bloqueó debido a una IP restringida. </br> **SSLHandshakeError**: no se puede establecer la conexión con el origen debido a un error del protocolo de enlace SSL. </br> **UnspecifiedError**: se produjo un error que no coincidía con ninguno de los errores de la tabla. |

### <a name="sent-to-origin-shield-deprecation"></a>Desuso de la propiedad de envío al escudo de origen
La propiedad **isSentToOriginShield** del registro sin procesar ha quedado en desuso y se ha reemplazado por un nuevo campo, **isReceivedFromClient**. Use el nuevo campo si aún usa el campo en desuso. 

Los registros sin procesar incluyen los registros generados tanto desde el servidor perimetral de la red CDN (POP secundario) como desde el escudo de origen. El escudo de origen hace referencia a los nodos primarios que están ubicados de manera estratégica por todo el planeta. Estos nodos se comunican con los servidores de origen y reducen la carga de tráfico en el origen. 

Por cada solicitud que va al escudo de origen, hay dos entradas de registro:

* Una para los nodos perimetrales y
* otra para el escudo de origen. 

Para diferenciar la salida o las respuestas de los nodos perimetrales de las del escudo de origen, puede usar el campo **isReceivedFromClient** para obtener los datos correctos. 

Si el valor es false, significa que la solicitud se responde desde el escudo de origen a los nodos perimetrales. Este enfoque es eficaz para comparar los registros sin procesar con los datos de facturación. No se paga por la salida del escudo de origen a los nodos perimetrales. Se paga por la salida de los nodos perimetrales a los clientes. 

**Ejemplo de consulta Kusto para excluir los registros generados en el escudo de origen en Log Analytics.**

`AzureDiagnostics 
| where Category == "FrontdoorAccessLog" and isReceivedFromClient_b == true`

> [!NOTE]
> En el caso de varias configuraciones de enrutamiento y comportamientos de tráfico, algunos de los campos, como backendHostname, cacheStatus, isReceivedFromClient y POP, pueden responder con valores diferentes. En la tabla siguiente se explican los distintos valores que estos campos tendrán para diversos escenarios:

| Escenarios | Recuento de entradas de registro | POP | BackendHostname | isReceivedFromClient | CacheStatus |
| ------------- | ------------- | ------------- | ------------- | ------------- | ------------- |
| Regla de enrutamiento sin almacenamiento en caché habilitado | 1 | Código POP perimetral | El back-end al que se reenvió la solicitud | Verdadero | CONFIG_NOCACHE |
| Regla de enrutamiento con almacenamiento en caché habilitado. Aciertos de caché en el POP perimetral | 1 | Código POP perimetral | Vacío | Verdadero | HIT |
| Regla de enrutamiento con almacenamiento en caché habilitado. Errores de caché en el POP perimetral pero acierto en el POP de la caché primaria | 2 | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. Nombre de host de POP de caché primaria</br>2. Vacío | 1. True</br>2. False | 1. MISS</br>2. HIT |
| Regla de enrutamiento con almacenamiento en caché habilitado. Error de cachés en el POP perimetral pero acierto PARTIAL en el POP de la caché primaria | 2 | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. Nombre de host de POP de caché primaria</br>2. Back-end que ayuda a rellenar la memoria caché | 1. True</br>2. False | 1. MISS</br>2. PARTIAL_HIT |
| Regla de enrutamiento con almacenamiento en caché habilitado. PARTIAL_HIT en el POP perimetral pero acierto en el POP de la caché primaria | 2 | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. True</br>2. False | 1. PARTIAL_HIT</br>2. HIT |
| Regla de enrutamiento con almacenamiento en caché habilitado. Errores de caché en el POP perimetral y en el de caché primaria | 2 | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. Código POP perimetral</br>2. Código POP de caché primaria | 1. True</br>2. False | 1. MISS</br>2. MISS |

> [!NOTE]
> En el caso de los escenarios de almacenamiento en caché, el valor del estado de la memoria caché será partial_hit cuando se sirvan algunos de los bytes para una solicitud desde la memoria caché del escudo de origen o perimetral de Front Door, mientras que algunos de los bytes se sirven desde el origen de los objetos grandes.

Front Door usa una técnica denominada fragmentación de objetos. Cuando se solicita un archivo grande, Front Door recupera partes más pequeñas del origen. Una vez que el servidor POP de Front Door recibe una solicitud de archivo completa o de intervalo de bytes, el servidor perimetral de Front Door solicita el archivo al origen en fragmentos de 8 MB.

Una vez que llega el fragmento al perímetro de la red Front Door, se almacena en caché y se sirve inmediatamente al usuario. Después, Front Door realiza una captura previa del siguiente fragmento en paralelo. Este captura previa garantiza que el contenido sigue estando un fragmento por delante del usuario, lo que reduce la latencia. Este proceso continúa hasta que se descarga todo el archivo (si se solicita), todos los intervalos de bytes están disponibles (si se solicitan) o el cliente cierra la conexión. Para más información sobre la solicitud de intervalo de bytes, vea RFC 7233. La red Front Door almacena en caché los fragmentos cuando se reciben. No es necesario almacenar el archivo entero en la caché de Front Door. Las solicitudes subsiguientes del archivo o los intervalos de bytes se sirven desde la caché de Front Door. Si no se almacenan en caché todos los fragmentos en la red Front Door, se usa la captura previa para solicitar fragmentos del origen. Esta optimización se basa en la capacidad del servidor de origen de admitir solicitudes de intervalo de bytes. Si el servidor de origen no admite solicitudes de intervalo de bytes, esta optimización no es efectiva.

## <a name="next-steps"></a>Pasos siguientes

- [Crear un perfil de Front Door](quickstart-create-front-door.md)
- Información acerca de cómo [funciona Front Door](front-door-routing-architecture.md)
