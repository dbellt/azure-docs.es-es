---
title: Control de falsos positivos en Azure Sentinel
description: Aprenda a resolver falsos positivos en Azure Sentinel mediante la creación de reglas de automatización o la modificación de reglas de análisis para especificar excepciones.
author: batamig
ms.author: bagol
ms.service: azure-sentinel
ms.topic: how-to
ms.date: 05/11/2021
ms.openlocfilehash: 88886f46f028da19f5e5b75f42e5d96a09f2213a
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109795634"
---
# <a name="handle-false-positives-in-azure-sentinel"></a>Control de falsos positivos en Azure Sentinel

Las [reglas de análisis de Azure Sentinel](tutorial-detect-threats-built-in.md) le notifican cuando se produce algo sospechoso en la red. Ninguna regla de análisis es perfecta y necesariamente obtendrá algunos falsos positivos que se deben controlar. En este artículo se describe cómo controlar los falsos positivos, ya sea mediante automatización o mediante la modificación de las reglas de análisis programadas.

## <a name="false-positive-causes-and-prevention"></a>Causas y prevención de falsos positivos

Incluso en una regla de análisis creada correctamente, los falsos positivos a menudo proceden de entidades específicas, como usuarios o direcciones IP que deben excluirse de la regla.

Entre los escenarios habituales se incluyen los siguientes:

- Las actividades normales de determinados usuarios, normalmente entidades de servicio, muestran un patrón que parece sospechoso.
- Se detecta como malintencionada actividad intencionada de examen de la seguridad procedente de direcciones IP conocidas.
- Una regla que excluye las direcciones IP privadas también debe excluir algunas direcciones IP internas que no son privadas.

En este artículo se describen dos métodos para evitar los falsos positivos:

- Las **reglas de automatización** crean excepciones sin modificar las reglas de análisis.
- Las **modificaciones de las reglas de análisis programadas** permiten excepciones más detalladas y permanentes.
  
En la tabla siguiente se describen las características de cada método:

  
|Método|Característica|
|-|-|
|**Regalas de automatización**|<ul><li>Se pueden aplicar a varias reglas de análisis.</li><li>Mantienen un registro de auditoría. Las excepciones impiden la creación de incidentes, pero las alertas se siguen registrando con fines de auditoría.</li><li>Las suelen generar los analistas.</li><li>Permiten la aplicación de excepciones durante un tiempo limitado. Por ejemplo, un trabajo de mantenimiento podría desencadenar falsos positivos que, fuera del período de tiempo de mantenimiento, se considerarían incidentes verdaderos.</li></ul>|
|**Modificaciones de las reglas de análisis**|<ul><li>Permiten expresiones booleanas avanzadas y excepciones basadas en subredes.</li><li>Le permiten usar listas de seguimiento para centralizar la administración de excepciones.</li><li>Normalmente, requieren la implementación por parte de los ingenieros del centro de operaciones de seguridad (SOC).</li><li>Constituyen la solución más flexible y completa a los falsos positivos, pero son más complejas.</li></ul>|

## <a name="add-exceptions-by-using-automation-rules"></a>Incorporación de excepciones mediante reglas de automatización

La manera más sencilla de agregar una excepción es [agregar una regla de automatización](automate-incident-handling-with-automation-rules.md#creating-and-managing-automation-rules) cuando se ve un incidente de falso positivo.

Para agregar una regla de automatización para controlar un falso positivo:

1. En Azure Sentinel, en **Incidentes**, seleccione el incidente para el que desea crear una excepción.
1. Seleccione **Crear regla de automatización**.
1. En la barra lateral **Creación de una regla de automatización**, puede modificar el nuevo nombre de la regla para identificar la excepción, en lugar de simplemente el nombre de la regla de alerta.
1. En **Condiciones**, puede agregar más **nombres de reglas de análisis** a los que aplicar la excepción.
1. En la barra lateral se presentan las entidades específicas del incidente actual que podrían haber causado el falso positivo. Mantenga las sugerencias automáticas o modifíquelas para ajustar la excepción. Por ejemplo, podría cambiar una condición en una dirección IP para aplicarla a una subred completa.
   
   :::image type="content" source="media/false-positives/create-rule.png" alt-text="Captura de pantalla que muestra cómo crear una regla de automatización para un incidente en Azure Sentinel.":::
   
1. Después de definir el desencadenador, puede seguir definiendo lo que hace la regla:
   
   :::image type="content" source="media/false-positives/apply-rule.png" alt-text="Captura de pantalla que muestra cómo terminar de crear y aplicar una regla de automatización en Azure Sentinel.":::
   
   - La regla ya está configurada para cerrar un incidente que cumple los criterios de excepción.
   - Puede agregar un comentario al incidente cerrado automáticamente que explique la excepción. Por ejemplo, podría especificar que el incidente se originó en una actividad administrativa conocida.
   - De forma predeterminada, la regla se establece para que expire automáticamente después de 24 horas. Puede elegir el período de tiempo que desee para la expiración y esta reducirá la posibilidad de errores de falsos negativos. Si desea una excepción más larga, establezca la opción **Expiración de la regla** en una hora posterior.
   
1. Seleccione **Aplicar** para activar la excepción.

> [!TIP]
> También puede crear una regla de automatización desde cero, sin empezar a partir de un incidente. Seleccione **Automatización** en el menú de navegación izquierdo de Azure Sentinel y, a continuación, seleccione **Crear** > **Agregar nueva regla**.

## <a name="add-exceptions-by-modifying-analytics-rules"></a>Adición de excepciones mediante la modificación de reglas de análisis

Otra opción para implementar excepciones es modificar la consulta de la regla de análisis. Puede incluir excepciones directamente en la regla o, preferiblemente, cuando sea posible, usar una referencia a una [lista de seguimiento](watchlists.md#use-watchlists-in-analytics-rules). A continuación, puede administrar la lista de excepciones en la lista de seguimiento.

### <a name="modify-the-query"></a>Modificación de la consulta

Para editar las reglas de análisis existentes, seleccione **Automatización** en el menú de navegación izquierdo de Azure Sentinel. Seleccione la regla que desea editar y, a continuación, seleccione **Editar** en la esquina inferior derecha para abrir el **asistente para reglas de análisis**.

Para obtener instrucciones detalladas sobre el uso del **asistente para reglas de análisis** para crear y editar reglas de análisis, consulte [Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas](tutorial-detect-threats-custom.md).

Para implementar una excepción en un preámbulo de regla típico, puede agregar una condición como `where IPAddress !in ('<ip addresses>')` cerca del principio de la consulta de regla. Esta línea excluye direcciones IP específicas de la regla.

```kusto
let timeFrame = 1d;
SigninLogs
| where TimeGenerated >= ago(timeFrame)
| where IPAddress !in ('10.0.0.8', '192.168.12.1')
...
```

Este tipo de excepción no se limita a las direcciones IP. Puede excluir usuarios específicos mediante el campo `UserPrincipalName` o excluir aplicaciones específicas mediante `AppDisplayName`.

También puede excluir varios atributos. Por ejemplo, para excluir alertas de la dirección IP `10.0.0.8` o del usuario `user@microsoft.com`, use:

```kusto
| where IPAddress !in ('10.0.0.8')
| where UserPrincipalName != 'user@microsoft.com'
```

Para implementar una excepción más específica cuando corresponda y reducir la posibilidad de falsos negativos, puede combinar los atributos. La siguiente excepción solo se aplica si ambos valores aparecen en la misma alerta:

```kusto
| where IPAddress != '10.0.0.8' and UserPrincipalName != 'user@microsoft.com'
```

### <a name="exclude-subnets"></a>Exclusión de subredes

La exclusión de los intervalos IP usados por una organización requiere una exclusión de subred. En el ejemplo siguiente se muestra la manera de excluir las subredes.

El operador `ipv4_lookup` es un operador de enriquecimiento, no de filtrado. La línea `where isempty(network)` es la que realiza realmente el filtrado, inspeccionando los eventos que no muestran una coincidencia.

```kusto
let subnets = datatable(network:string) [ "111.68.128.0/17", "5.8.0.0/19", ...];
let timeFrame = 1d;
SigninLogs
| where TimeGenerated >= ago(timeFrame)
| evaluate ipv4_lookup(subnets, IPAddress, network, return_unmatched = true)
| where isempty(network)
...
```

### <a name="use-watchlists-to-manage-exceptions"></a>Uso de listas de seguimiento para administrar excepciones

Puede usar una lista de seguimiento para administrar la lista de excepciones fuera de la propia regla. Cuando se puede aplicar, esta solución tiene las siguientes ventajas:

- Un analista puede agregar excepciones sin editar la regla, lo cual se adapta mejor a los procedimientos recomendados del centro de operaciones de seguridad.
- La misma lista de seguimiento se puede aplicar a varias reglas, lo que permite la administración central de las excepciones.

El uso de una lista de seguimiento es similar al uso de una excepción directa. Use `_GetWatchlist('<watchlist name>')` para llamar a la lista de seguimiento:

```kusto
let timeFrame = 1d;
let logonDiff = 10m;
let allowlist = (_GetWatchlist('ipallowlist') | project IPAddress);
SigninLogs
| where TimeGenerated >= ago(timeFrame)
| where IPAddress !in (allowlist)
...
```

También puede realizar el filtrado de subredes mediante una lista de seguimiento. Por ejemplo, en el código de exclusión de la subred anterior, podría reemplazar la definición de `datatable` de la subred por una lista de seguimiento:

```kusto
let subnets = _GetWatchlist('subnetallowlist');
```

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte:
- [Automatización del control de incidentes en Azure Sentinel con las reglas de automatización](automate-incident-handling-with-automation-rules.md)
- [Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas](tutorial-detect-threats-custom.md)
- [Uso de listas de seguimiento de Azure Sentinel](watchlists.md)
