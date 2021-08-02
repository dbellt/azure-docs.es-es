---
title: Capacidades de búsqueda en Azure Sentinel| Microsoft Docs
description: Use las consultas de búsqueda integradas de Azure Sentinel a modo de guía para formular las preguntas adecuadas que lleven a detectar problemas en los datos.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.assetid: 6aa9dd27-6506-49c5-8e97-cc1aebecee87
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.custom: mvc
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/27/2021
ms.author: yelevin
ms.openlocfilehash: a8bf6ec24a67327d715b4a5d09b1d0d633b980c5
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109810414"
---
# <a name="hunt-for-threats-with-azure-sentinel"></a>Búsqueda de amenazas con Azure Sentinel

> [!IMPORTANT]
>
> - Las actualizaciones del **panel de búsqueda** están actualmente en **VERSIÓN PRELIMINAR**. Los elementos siguientes relacionados con esta actualización se marcarán como "(versión preliminar)". Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

Tanto los analistas como los investigadores de seguridad, quieren buscar amenazas de seguridad de forma proactiva, pero los distintos sistemas y dispositivos de seguridad generan gran cantidad de datos que pueden ser difíciles de analizar y filtrar para convertirlos en eventos significativos. Azure Sentinel tiene eficaces herramientas de búsqueda y consulta que permiten buscar amenazas de seguridad en los orígenes de datos de cualquier organización. Para ayudar a los analistas de seguridad a buscar proactivamente nuevas anomalías que ni las aplicaciones de seguridad ni las reglas de análisis programadas han sido capaces de identificar, las consultas de búsqueda integradas de Azure Sentinel servirán de guía para formular las preguntas adecuadas para detectar problemas en los datos que ya hay en la red. 

Por ejemplo, una consulta integrada proporciona datos sobre los procesos menos habituales que se ejecutan en la infraestructura. No desea recibir una alerta cada vez que se ejecuten (pueden ser totalmente inofensivos), pero puede echar un vistazo a la consulta de vez en cuando para ver si hay algo inusual. 

Con la función de búsqueda de Azure Sentinel, puede aprovechar las siguientes capacidades:

- **Consultas integradas**: la página de búsqueda principal, a la que se puede acceder desde el menú de navegación de Azure Sentinel, proporciona ejemplos de consultas listas para usar diseñadas no solo para que pueda empezar a trabajar, sino también para que se familiarice con las tablas y el lenguaje de consulta. Estas consultas de búsqueda integradas las desarrollan los investigadores de seguridad de Microsoft de forma continua mediante la incorporación de nuevas consultas y el ajuste de las consultas existentes para proporcionar un punto de entrada para buscar nuevas detecciones y averiguar dónde empezar a buscar los comienzos de nuevos ataques. 

- **Eficaz lenguaje de consulta con IntelliSense**: las consultas de búsqueda se integran en el [lenguaje de consulta Kusto (KQL)](/azure/data-explorer/kusto/query/), un lenguaje de consulta que proporciona la eficacia y flexibilidad necesarias para llevar la búsqueda a un nuevo nivel. Es el mismo lenguaje que usan las consultas en las reglas de análisis y en otros lugares de Azure Sentinel.

- **Panel de búsqueda (versión preliminar)** : esta actualización de la página principal permite ejecutar todas las consultas, o un subconjunto seleccionado, con un solo clic. Identifique dónde desea empezar la búsqueda. Para ello, debe examinar el recuento de resultados, los picos o el cambio en el recuento de resultados en un período de 24 horas. También puede ordenar y filtrar por favoritos, origen de datos, táctica o técnica MITRE ATT&CK, resultados o diferencias de resultados. Vea las consultas que aún no tienen los orígenes de datos necesarios conectados y obtenga recomendaciones necesarias para habilitar estas consultas.

- **Crear sus propios marcadores**: durante el proceso de búsqueda, puede encontrar resultados de consulta que puedan parecer inusuales o sospechosos. Puede incluir estos elementos en los "marcadores" (guardarlos y quitarlos para que pueda volver a consultarlos en el futuro). Los elementos que se hayan incluido en los marcadores se pueden usar para crear o enriquecer cualquier incidente para su investigación. Para obtener más información sobre los marcadores, vea el tema sobre el [uso de marcadores de búsqueda](bookmarks.md).

- **Usar cuadernos para que la investigación sea eficaz**: los cuadernos proporcionan un tipo de entorno de espacio aislado virtual, que se completa con su propio kernel. Puede usar cuadernos para mejorar la búsqueda y las investigaciones con aprendizaje automático, visualización y análisis de datos. Puede realizar una investigación completa en un cuaderno, encapsulando los datos sin procesar, el código que se ejecuta en él, los resultados y sus visualizaciones, y guardar todo el contenido para que se pueda compartir con otros usuarios y estos puedan reutilizarlo. 

- **Consultar los datos almacenados**: los datos se encuentran en tablas y se pueden consultar. Así, por ejemplo, se puede consultar la creación de procesos, los eventos DNS y otros muchos tipos de eventos.

- **Vínculos a la comunidad**: aproveche la eficacia de disfrutar de una comunidad más amplia para buscar más consultas y orígenes de datos.
 
## <a name="get-started-hunting"></a>Empezar a realizar búsquedas

En el portal de Azure Sentinel, haga clic en **Hunting** (Búsqueda).

   :::image type="content" source="media/hunting/hunting-start.png" alt-text="Azure Sentinel empieza a buscar" lightbox="media/hunting/hunting-start.png":::

- Al abrir la página **Hunting** (Búsqueda), todas las consultas de búsqueda se muestran en una misma tabla. En esa tabla se enumeran todas las consultas escritas por el equipo de analistas de seguridad de Microsoft, así como otras consultas adicionales que haya creado o modificado. En cada consulta se proporciona una descripción de lo que se busca y en qué tipo de datos se ejecuta. Estas plantillas se agrupan según diversas tácticas; así, los iconos de la derecha clasifican el tipo de amenaza, como acceso inicial, persistencia y filtración.

- (Versión preliminar) Para ver cómo se aplican las consultas en su entorno, haga clic en el botón **Run all queries (Preview)** [Ejecutar todas las consultas (versión preliminar)], o bien seleccione un subconjunto de consultas mediante las casillas que hay a la izquierda de cada fila y, después seleccione el botón **Run selected queries (Preview)** [Ejecutar las consultas seleccionadas (versión preliminar)]. La ejecución de las consultas puede tardar entre unos segundos y varios minutos, en función del número de consultas seleccionadas, el intervalo de tiempo y la cantidad de datos que se consultan.

- (Versión preliminar) Una vez que las consultas hayan acabado de ejecutarse, puede ver qué consultas han devuelto resultados mediante el filtro **Results** (Resultados). Luego, puede ordenar para ver qué consultas han tenido más el mayor o menor número de resultados. También puede ver qué consultas no están activas en su entorno. Para ello, debe seleccionar *N/A* en el filtro **Results** (Resultados). Mantenga el puntero sobre el icono de información (i) que aparece al lado de *N/A* para ver qué orígenes de datos se necesitan para que esta consulta sea activa.

- (Versión preliminar) Para identificar picos en los datos, ordene o filtre por **Results delta** (Diferencia de resultados). De esta forma se comparan los resultados de las últimas 24 horas con los resultados de las 24-48 horas anteriores para que sea fácil ver grandes diferencias en el volumen.

- (Versión preliminar) La **barra de tácticas de MITRE ATT&CK**, al principio de la tabla, muestra cuántas consultas se asignan a cada táctica de MITRE ATT&CK. La barra de tácticas se actualiza dinámicamente en función del actual conjunto de filtros aplicado. Se trata de una manera sencilla de ver qué tácticas de MITRE ATT&CK se muestran al filtrar por un recuento de resultados determinado, una diferencia de resultados alta, resultados de *N/A* o cualquier otro conjunto de filtros.

- (Versión preliminar) Las consultas también se pueden asignar a técnicas de MITRE ATT&CK. Para filtrar u ordenar por técnicas de MITRE ATT&CK, utilice el filtro **Technique** (Técnica). Al abrir una consulta, podrá hacer clic en la técnica para ver la descripción de MITRE ATT&CK de la técnica.

- Puede guardar cualquier consulta en sus favoritos. Las consultas guardadas en favoritos se ejecutan automáticamente cada vez que se accede a la página **Hunting** (Búsqueda). Puede crear sus propias consultas de búsqueda o clonar y personalizar una plantilla de consulta de búsqueda ya existente.
 
- Si hace clic en **Run query** (Ejecutar consulta) en la página de detalles de la consulta de búsqueda, puede ejecutar cualquier consulta sin salir de la página de búsqueda. El número de coincidencias se muestra en la tabla, en la columna **Results** (Resultados). Revise la lista de consultas de búsqueda y las coincidencias encontradas correspondientes.

- Puede realizar una revisión rápida de la consulta subyacente en el panel de detalles de la consulta. Para ver los resultados, haga clic en el vínculo **View query results** (Ver resultados de la consulta) (debajo de la ventana de consulta) o en el botón **View Results** (Ver resultados) (en la parte inferior del panel). La consulta se abrirá en la hoja **Logs** (Registros) (Log Analytics) y, debajo de la consulta, puede revisar las coincidencias de la consulta.

- Para conservar los resultados sospechosos o interesantes de una consulta en Log Analytics, marque las casillas de las filas que desea conservar y seleccione **Agregar marcador**. De esta forma se crea un registro (un marcador) para cada fila marcada que contiene los resultados de la fila, la consulta que creó los resultados y las asignaciones de entidades para extraer usuarios, hosts y direcciones IP. Puede agregar sus propias etiquetas (consulte a continuación) y notas a cada marcador.

- Para ver todos los resultados marcados haciendo clic en la pestaña **Bookmarks** (Marcadores) de la página principal de **Hunting** (Búsqueda). Puede agregar etiquetas a los marcadores para clasificarlas para el filtrado. Por ejemplo, si va a investigar una campaña de ataques, puede crear una etiqueta para la campaña, aplicar la etiqueta a todos los marcadores pertinentes y, después, filtrar todos los marcadores por esa campaña.

- Para investigar un único resultado incluido en los marcadores, seleccione el marcador y, después, haga clic en **Investigate** (Investigar) en el panel de detalles para abrir la experiencia de investigación. También puede crear un incidente a partir de uno o varios marcadores, o bien agregar uno o varios marcadores a un incidente existente. Para ello, marque las casillas situadas a la izquierda de los marcadores deseados y, después, seleccione **Create new incident** (Crear incidente) o **Add to existing incident** (Agregar a incidente existente) en el menú desplegable **Incident actions** (Acciones de incidente) cerca de la parte superior de la pantalla. Luego puede realizar una evaluación de prioridades del incidente e investigarlo como cualquier otro.

- Tras haber detectado o creado una consulta de búsqueda que proporciona información muy valiosas sobre posibles ataques, puede crear reglas de detección personalizadas en función de esa consulta y exponer dicha información como alertas para los respondedores a los incidentes de seguridad. Vea los resultados de la consulta en Log Analytics (más arriba), haga clic en el botón **New alert rule** (Nueva regla de alertas) en la parte superior del panel y seleccione **Create Azure Sentinel alert** (Crear una alerta de Azure Sentinel). Se abrirá el **Asistente para reglas de Analytics**. Complete los pasos necesarios como se explica en [Tutorial: Creación de reglas de análisis personalizadas para detectar amenazas](tutorial-detect-threats-custom.md).

## <a name="query-language"></a>Lenguaje de consulta 

Las búsquedas de Azure Sentinel se basan en el lenguaje de consultas de Kusto. Para obtener más información sobre el lenguaje de consulta y los operadores admitidos, consulte la [referencia de idioma de consulta](../azure-monitor/logs/get-started-queries.md).

## <a name="public-hunting-query-github-repository"></a>Repositorio de GitHub de consultas de búsquedas públicas

Eche un vistazo al [repositorio de consultas de búsqueda](https://github.com/Azure/Azure-Sentinel/tree/master/Hunting%20Queries). Use las consultas de ejemplo compartidas por nuestros clientes y aporte las suyas propias.

 ## <a name="sample-query"></a>Consulta de ejemplo

Una consulta típica empieza por un nombre de tabla, seguido de una serie de operadores separados por \|.

En el ejemplo anterior, empiece con la tabla SecurityEvent y agregue elementos canalizados según sea necesario.

1. Defina un filtro de tiempo para revisar solo los registros de los últimos siete días.

1. Agregue un filtro a la consulta para mostrar únicamente el evento con el identificador 4688.

1. Agregue un filtro a la consulta en CommandLine para que contenga solo instancias de cscript.exe.

1. Proyecte solo las columnas que esté interesado en explorar, limite los resultados a 1000 y haga clic en **Run query** (Ejecutar consulta).

1. Haga clic en el triángulo verde y ejecute la consulta. Puede comprobar la consulta y ejecutarla para hallar comportamientos anómalos.

## <a name="useful-operators"></a>Operadores útiles

El lenguaje de consulta es muy eficaz y cuenta con numerosos operadores disponibles. Estos son algunos de ellos, que son muy útiles:

**where**: filtra una tabla por el subconjunto de filas que cumplen un predicado.

**summarize**: crea una tabla que agrega el contenido de la tabla de entrada.

**join**: combina las filas de dos tablas para formar una nueva tabla, haciendo coincidir los valores de las columnas especificadas de cada tabla.

**count**: devuelve el número de registros en el conjunto de registros de entrada.

**top**: devuelve los primeros N registros ordenados por las columnas especificadas.

**limit**: devuelve hasta el número de filas especificado.

**project**: selecciona las columnas que se incluirán, las cambia de nombre o las quita e inserta nuevas columnas calculadas.

**extend**: crea columnas calculadas y las anexa al conjunto de resultados.

**makeset**: devuelve una matriz dinámica (JSON) del conjunto de valores distintos que una expresión toma en el grupo.

**find**: busca filas que coinciden con un predicado a través de un conjunto de tablas.

## <a name="save-a-query"></a>Almacenamiento de una consulta

Puede crear o modificar una consulta y guardarla como consulta propia o compartirla con los usuarios que están en el mismo inquilino.

:::image type="content" source="./media/hunting/save-query.png" alt-text="Guardar consulta" lightbox="./media/hunting/save-query.png":::

### <a name="create-a-new-hunting-query"></a>Creación de una consulta de búsqueda

1. Haga clic en **New query** (Nueva consulta).

1. Rellene todos los campos en blanco y seleccione **Create** (Crear).

    :::image type="content" source="./media/hunting/new-query.png" alt-text="Nueva consulta" lightbox="./media/hunting/new-query.png":::

### <a name="clone-and-modify-an-existing-hunting-query"></a>Clonación y modificación de una consulta de búsqueda existente

1. Seleccione la consulta de búsqueda en la tabla que quiere modificar.

1. Seleccione el botón de puntos suspensivos (...) en la línea de la consulta que quiere modificar y seleccione **Clone query** (Clonar consulta).

    :::image type="content" source="./media/hunting/clone-query.png" alt-text="Clonar una consulta" lightbox="./media/hunting/clone-query.png":::

1. Modifique la consulta y seleccione **Create** (Crear).

    :::image type="content" source="./media/hunting/custom-query.png" alt-text="Consulta personalizada" lightbox="./media/hunting/custom-query.png":::

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a realizar una investigación de búsqueda en Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:

- [Uso de cuadernos de Jupyter Notebook para buscar amenazas de seguridad](notebooks.md)
- [Realizar un seguimiento de los datos durante una búsqueda](bookmarks.md)
