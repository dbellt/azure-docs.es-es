---
title: 'Supervisión de la disponibilidad con pruebas de ping de URL: Azure Monitor'
description: Configure pruebas de ping en Application Insights. Obtenga alertas si un sitio web deja de estar disponible o responde con lentitud.
ms.topic: conceptual
ms.date: 05/25/2021
ms.reviewer: sdash
ms.openlocfilehash: e7e52cb5db6db5fbd1e73713d3d3558be9e859f0
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111968389"
---
# <a name="monitor-availability-with-url-ping-tests"></a>Supervisión de la disponibilidad con pruebas de ping de URL

El nombre "prueba de ping de dirección URL" es un poco inapropiado. Para que quede claro, estas pruebas no usan el protocolo de mensajes de control de Internet (ICMP) para comprobar la disponibilidad del sitio. En cambio, usan funcionalidad más avanzada de solicitud HTTP para validar si un punto de conexión responde. También miden el rendimiento asociado con esa respuesta y permiten establecer criterios de éxito personalizados junto con características más avanzadas, como analizar solicitudes dependientes y facilitar los reintentos.

Para crear una prueba de disponibilidad, debe usar un recurso existente de Application Insight o crear el [recurso de Application Insights](create-new-resource.md).

Para crear la primera solicitud de disponibilidad, abra el panel Disponibilidad y seleccione   **Create Test** (Crear prueba).

:::image type="content" source="./media/monitor-web-app-availability/availability-create-test-001.png" alt-text="Captura de pantalla de la creación de una prueba.":::

## <a name="create-a-test"></a>Creación de una prueba

|Configuración| Explicación
|----|----|----|
|**URL** |  La dirección URL puede ser cualquier página web que desee probar, pero debe ser visible desde la red pública de Internet. La dirección URL puede incluir una cadena de consulta. Así, por ejemplo, se puede ejercitar un poco la base de datos. Si la dirección URL se resuelve en una redirección, la seguimos, hasta 10 redirecciones.|
|**Analizar solicitudes dependientes**| La prueba solicitará imágenes, scripts, archivos de estilo y otros archivos que forman parte de la página web en pruebas. El tiempo de respuesta registrado incluye el tiempo dedicado a obtener estos archivos. La prueba da error si cualquiera de estos recursos no se puede descargar correctamente dentro del tiempo de espera de la prueba entera. Si la opción no está activada, la prueba solo solicita el archivo en la dirección URL que especificó. Si se habilita esta opción, se realiza una comprobación más estricta. Podría producirse un error en la prueba para los casos que puede que no sean evidentes al examinar el sitio de forma manual.
|**Habilitar reintentos**|cuando la prueba da error, se reintenta tras un corto intervalo. Se notifica un error únicamente si los tres intentos sucesivos producen un error. Las sucesivas pruebas se realizan según la frecuencia habitual de la prueba. El reintento se suspende temporalmente hasta que uno se complete correctamente. Esta regla se aplica independientemente en cada ubicación de la prueba. **Se recomienda esta opción**. Como media, cerca del 80 % de los errores desaparecen al reintentar.|
|**Frecuencia de prueba**| establece la frecuencia con que se ejecuta la prueba desde cada ubicación de prueba. Con una frecuencia predeterminada de cinco minutos y cinco ubicaciones de prueba, el sitio se prueba, de media, cada minuto.|
|**Ubicaciones de prueba**| Son los lugares desde donde nuestros servidores envían solicitudes web a la dirección URL. **El número mínimo de ubicaciones de prueba recomendadas es cinco** con el fin de asegurarse de que puede distinguir los problemas del sitio web de los problemas de la red. Puede seleccionar hasta 16 ubicaciones.

**Si la dirección URL no es visible desde la red Internet pública, tiene la opción de abrir selectivamente el firewall para permitir solo las transacciones de prueba**. Para más información sobre las excepciones de firewall para nuestros agentes de prueba de disponibilidad, consulte la [guía de direcciones IP](./ip-addresses.md#availability-tests).

> [!NOTE]
> Se recomienda probar desde varias ubicaciones con un **mínimo de cinco ubicaciones**. Esto es para evitar falsas alarmas que pueden deberse a problemas transitorios con una ubicación específica. Además, hemos descubierto que la configuración óptima es que el **número de ubicaciones de prueba sea igual que el umbral de ubicación de la alerta + 2**.

## <a name="success-criteria"></a>Criterios de éxito

|Configuración| Explicación
|----|----|----|
| **Tiempo de espera de prueba** |reduzca este valor para recibir una alerta sobre las respuestas lentas. La prueba se considera un error si no se han recibido respuestas de su sitio dentro de este período. Si seleccionó **Analizar solicitudes dependientes**, todas las imágenes, archivos de estilo, scripts y otros recursos dependientes se deben haber recibido durante este período.|
| **Respuesta HTTP** | el código de estado devuelto que se considera correcto. 200 es el código que indica que se ha devuelto una página web normal.|
| **Coincidencia de contenido** | Una cadena, como "Bienvenido". Probamos que se produce una coincidencia exacta entre mayúsculas y minúsculas en todas las respuestas. Debe ser una cadena sin formato, sin caracteres comodín. No se olvide de que si el contenido cambia, es posible que tenga que actualizarla. **En la coincidencia de contenido solo se admiten caracteres en inglés** |

## <a name="alerts"></a>Alertas

|Configuración| Explicación
|----|----|----|
|**Casi en tiempo real (versión preliminar)** | Se recomienda usar alertas casi en tiempo real. La configuración de este tipo de alertas se realiza después de crear la prueba de disponibilidad.  |
|**Umbral de la ubicación de la alerta**|se recomienda un mínimo de 3/5 ubicaciones. La relación óptima entre el umbral de ubicación de la alerta y el número de ubicaciones de prueba es **umbral de ubicación de la alerta** = **número de ubicaciones de prueba - 2, con un mínimo de cinco ubicaciones de prueba.**|

## <a name="location-population-tags"></a>Etiquetas para rellenar la ubicación

Se pueden usar las siguientes etiquetas para rellenar el atributo de ubicación geográfica al implementar una prueba de ping de la dirección URL para averiguar la disponibilidad mediante Azure Resource Manager.

### <a name="azure-gov"></a>Azure Gov

| Display Name (Nombre para mostrar)   | Nombre de rellenado     |
|----------------|---------------------|
| USGov Virginia | usgov-va-azr        |
| USGov: Arizona  | usgov-phx-azr       |
| USGov Texas    | usgov-tx-azr        |
| Departamento de Defensa del este de EE. UU     | usgov-ddeast-azr    |
| Departamento de Defensa de centro de EE. UU.  | usgov-ddcentral-azr |

### <a name="azure-china"></a>Azure China

| Display Name (Nombre para mostrar)   | Nombre de rellenado     |
|----------------|---------------------|
| Este de China     | mc-cne-azr          |
| Este de China 2   | mc-cne2-azr         |
| Norte de China    | mc-cnn-azr          |
| Norte de China 2  | mc-cnn2-azr         |

#### <a name="azure"></a>Azure

| Display Name (Nombre para mostrar)                           | Nombre de rellenado   |
|----------------------------------------|-------------------|
| Este de Australia                         | emea-au-syd-edge  |
| Sur de Brasil                           | latam-br-gru-edge |
| Centro de EE. UU.                             | us-fl-mia-edge    |
| Este de Asia                              | apac-hk-hkn-azr   |
| Este de EE. UU.                                | us-va-ash-azr     |
| Sur de Francia (anteriormente Centro de Francia) | emea-ch-zrh-edge  |
| Centro de Francia                         | emea-fr-pra-edge  |
| Japón Oriental                             | apac-jp-kaw-edge  |
| Norte de Europa                           | emea-gb-db3-azr   |
| Centro-Norte de EE. UU                       | us-il-ch1-azr     |
| Centro-sur de EE. UU.                       | us-tx-sn1-azr     |
| Sudeste de Asia                         | apac-sg-sin-azr   |
| Oeste de Reino Unido                                | emea-se-sto-edge  |
| Oeste de Europa                            | emea-nl-ams-azr   |
| Oeste de EE. UU.                                | us-ca-sjc-azr     |
| Sur de Reino Unido                               | emea-ru-msa-edge  |

## <a name="see-your-availability-test-results"></a>Visualización de los resultados de las pruebas de disponibilidad

Los resultados de la prueba de disponibilidad se pueden visualizar con vistas de línea o vistas de trazado de dispersión.

Después de unos minutos, haga clic en **Actualizar** para ver los resultados.

:::image type="content" source="./media/monitor-web-app-availability/availability-refresh-002.png" alt-text="Captura de pantalla que muestra la página Disponibilidad con el botón Actualizar resaltado.":::

En la vista de dispersión se muestran ejemplos de los resultados de las pruebas, que incluyen detalles sobre los pasos de las pruebas de diagnóstico. El motor de pruebas almacena los detalles de diagnóstico de las pruebas con errores. En el caso de las pruebas correctas, los detalles de diagnóstico se almacenan para un subconjunto de las ejecuciones. Mantenga el mouse sobre cualquiera de los puntos rojos o verdes para ver la prueba, el nombre de esta y su ubicación.

:::image type="content" source="./media/monitor-web-app-availability/availability-scatter-plot-003.png" alt-text="Vista de línea" border="false":::

Seleccione una prueba o una ubicación determinadas, o bien reduzca el período de tiempo para ver más resultados del período de tiempo que le interese. Use el Explorador de búsqueda para ver los resultados de todas las ejecuciones, o use consultas de Analytics para ejecutar informes personalizados en estos datos.

## <a name="inspect-and-edit-tests"></a>Inspección y edición de pruebas

Para editar, eliminar o deshabilitar temporalmente una prueba, haga clic en el botón de puntos suspensivos junto al nombre de la prueba. Pueden tardar hasta 20 minutos para que los cambios se propaguen a todos los agentes de prueba después de realizar un cambio.

:::image type="content" source="./media/monitor-web-app-availability/edit.png" alt-text="Vea los detalles de la prueba. Edite y deshabilitar una prueba web." border="false":::

Tal vez le interese deshabilitar las pruebas de disponibilidad o las reglas de alerta asociadas a ellas mientras esté realizando el mantenimiento del servicio.

## <a name="if-you-see-failures"></a>Si ve errores

Seleccione un punto rojo.

:::image type="content" source="./media/monitor-web-app-availability/end-to-end.png" alt-text="Captura de pantalla de la pestaña de detalles de una transacción completa." border="false":::

Puede ver los detalles de transacción en todos los componentes desde el resultado de la prueba de disponibilidad. Aquí puede:

* Revise el informe de solución de problemas para determinar qué puede haber provocado que la prueba fallase, pero la aplicación siga estando disponible.
* Inspeccionar la respuesta recibida desde el servidor.
* Diagnosticar errores con la telemetría de lado servidor correlacionada que se recopiló durante el procesamiento de la prueba de disponibilidad con error.
* Registrar un problema o elemento de trabajo en GIT o Azure Boards para realizar un seguimiento del problema. El error contiene un vínculo a este evento.
* Abra el resultado de la prueba web en Visual Studio.

Para obtener más información sobre la experiencia completa de diagnóstico de transacciones, consulte la [documentación sobre el diagnóstico de transacciones](./transaction-diagnostics.md).

Haga clic en la fila de excepciones para ver los detalles de la excepción del lado servidor que ha provocado un error en la prueba de disponibilidad sintética. También puede obtener la [instantánea de depuración](./snapshot-debugger.md) para realizar diagnósticos de nivel de código más completos.

:::image type="content" source="./media/monitor-web-app-availability/open-instance-4.png" alt-text="Diagnósticos del servidor":::

Además de los resultados sin formato, también puede ver dos métricas de disponibilidad clave en el [Explorador de métricas](../essentials/metrics-getting-started.md):

1. Disponibilidad: Porcentaje de las pruebas que obtuvieron resultados satisfactorios en todas las ejecuciones de prueba.
2. Duración de la prueba: Duración media de las pruebas en todas las ejecuciones de prueba.

## <a name="automation"></a>Automation

* [Use scripts de PowerShell para configurar una prueba de disponibilidad](./powershell.md#add-an-availability-test) automáticamente.
* Configure un [webhook](../alerts/alerts-webhooks.md) que se llama cuando se genera una alerta.


## <a name="next-steps"></a>Pasos siguientes

* [Alertas de disponibilidad](availability-alerts.md)
* [Pruebas web de varios pasos](availability-multistep.md)
* [Solución de problemas](troubleshoot-availability.md)
* [Plantilla de Azure Resource Manager para pruebas web](/azure/templates/microsoft.insights/webtests?tabs=json)