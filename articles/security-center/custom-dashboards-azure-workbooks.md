---
title: Galería de libros en Azure Security Center
description: Aprenda a crear informes completos e interactivos de los datos de Azure Security Center con la galería de Libros de Azure Monitor integrada.
author: memildin
ms.author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 06/13/2021
ms.openlocfilehash: 9aad86bc08d7190ae9fca9faab9dbfcfba93f206
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112062323"
---
# <a name="create-rich-interactive-reports-of-security-center-data"></a>Creación de informes completos e interactivos de los datos de Security Center

Los [libros de Azure Monitor](../azure-monitor/visualize/workbooks-overview.md) proporcionan un lienzo flexible para el análisis de datos y la creación de informes visuales completos en Azure Portal. Permiten acceder a varios orígenes de datos desde Azure y combinarlos en experiencias interactivas unificadas.

Los libros proporcionan un amplio conjunto de funcionalidades para visualizar los datos de Azure. Para obtener ejemplos detallados de cada tipo de visualización, consulte la [documentación y ejemplos de visualizaciones](../azure-monitor/visualize/workbooks-text-visualizations.md). 

Dentro de Azure Security Center, puede acceder a los informes integrados para realizar un seguimiento de la postura de seguridad de su organización. También puede crear informes personalizados para ver una amplia gama de datos de Security Center u otros orígenes de datos admitidos.

:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-snip.png" alt-text="Puntuación de seguridad a lo largo del tiempo":::

## <a name="availability"></a>Disponibilidad

| Aspecto                          | Detalles                                                                                                                                      |
|---------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------|
| Estado de la versión:                  | Disponibilidad general (GA)                                                                                                                    |
| Precios:                        | Gratuito                                                                                                                                         |
| Roles y permisos necesarios: | Para guardar los libros, debe tener al menos permisos de Colaborador de libros en el grupo de recursos de destino.                                      |
| Nubes:                         | ![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos) |
|                                 |                                                                                                                                              |

## <a name="workbooks-gallery-in-azure-security-center"></a>Galería de libros en Azure Security Center

Con la funcionalidad integrada de los libros de Azure, Azure Security Center facilita la creación de sus propios informes personalizados e interactivos. Security Center también incluye una galería de libros con los siguientes informes listos para su personalización:

- **Puntuación de seguridad a lo largo del tiempo**: Puede realizar el seguimiento de las puntuaciones de las suscripciones y de los cambios en las recomendaciones sobre sus recursos.
- **Actualizaciones del sistema**: puede ver las actualizaciones del sistema que faltan por recursos, sistema operativo, gravedad, etc.
- **Conclusiones de la evaluación de vulnerabilidad**: Puede ver los resultados de los exámenes de vulnerabilidades de los recursos de Azure.

:::image type="content" source="media/custom-dashboards-azure-workbooks/workbooks-gallery-security-center.png" alt-text="Galería de libros integrados en Azure Security Center":::

Elija uno de los informes proporcionados o cree uno propio.

> [!TIP]
> Use el botón **Editar** para personalizar a su gusto cualquiera de los informes proporcionados. Cuando haya terminado de editar, seleccione **Guardar**, y los cambios se guardarán en un nuevo libro.
> 
> :::image type="content" source="media/custom-dashboards-azure-workbooks/editing-supplied-workbooks.png" alt-text="Edición de los libros proporcionados para personalizarlos según sus necesidades particulares":::

### <a name="use-the-secure-score-over-time-report"></a>Uso del informe Puntuación de seguridad a lo largo del tiempo

En este informe se usan los datos de la puntuación de seguridad del área de trabajo Log Analytics. Esos datos se tienen que exportar desde la herramienta de exportación continua, como se describe en [Configuración de la exportación continua desde las páginas de Security Center en Azure Portal](continuous-export.md?tabs=azure-portal).

Al configurar la exportación continua, establezca la frecuencia de exportación en las **streaming updates** (actualizaciones de streaming) e **instantáneas**.

:::image type="content" source="media/custom-dashboards-azure-workbooks/export-frequency-both.png" alt-text="Para el libro de puntuación de seguridad a lo largo del tiempo, tendrá que seleccionar ambas opciones de la configuración de frecuencia de exportación en la configuración de exportación continua.":::

> [!NOTE]
> Las instantáneas se exportan semanalmente, por lo que tendrá que esperar al menos una semana para que se exporte la primera instantánea para poder ver los datos en este informe.

> [!TIP]
> Para configurar la exportación continua en toda la organización, use las directivas DeployIfNotExist de Azure Policy proporcionadas que se describen en [Configuración de la exportación continua a escala](continuous-export.md?tabs=azure-policy).

El informe de puntuación de seguridad a lo largo del tiempo tiene cinco gráficos para las suscripciones que informan a las áreas de trabajo seleccionadas:


|Grafo  |Ejemplo  |
|---------|---------|
|**Tendencias de puntuación de la última semana y el último mes**<br>Use esta sección para supervisar la puntuación actual y las tendencias generales de las puntuaciones de las suscripciones.|:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-table-1.png" alt-text="Tendencias para la puntuación de seguridad en el informe integrado":::|
|**Puntuación agregada para todas las suscripciones seleccionadas**<br>Mantenga el puntero sobre cualquier punto de la línea de tendencia para ver la puntuación agregada en cualquier fecha del intervalo de tiempo seleccionado.|:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-table-2.png" alt-text="Puntuación agregada para todas las suscripciones seleccionadas":::|
|**Recomendaciones con el mayor número de recursos incorrectos**<br>Esta tabla le ayuda a evaluar las recomendaciones para el mayor número de recursos cuyo estado ha cambiado a incorrecto durante el período seleccionado.|:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-table-3.png" alt-text="Recomendaciones con el mayor número de recursos incorrectos":::|
|**Puntuaciones de controles de seguridad específicos**<br>Los controles de seguridad de Security Center son agrupaciones lógicas de recomendaciones. En este gráfico se muestran, de un vistazo, las puntuaciones semanales de todos los controles.|:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-table-4.png" alt-text="Puntuaciones de los controles de seguridad durante el período de tiempo seleccionado":::|
|**Cambios en los recursos**<br>Aquí se enumeran las recomendaciones con el mayor número de recursos que han cambiado de estado (correcto, incorrecto o no aplicable) durante el período seleccionado. Seleccione cualquier recomendación de la lista para abrir una nueva tabla donde se muestren los recursos específicos.|:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-table-5.png" alt-text="Recomendaciones con el mayor número de recursos que han cambiado de estado de mantenimiento":::|

### <a name="use-the-system-updates-report"></a>Uso del informe Actualizaciones del sistema

Este informe se basa en la recomendación de seguridad "Las actualizaciones del sistema deben estar instaladas en las máquinas".

El informe le ayuda a identificar las máquinas con actualizaciones pendientes.

Puede ver la situación de las suscripciones seleccionadas según:

- La lista de recursos con actualizaciones pendientes
- La lista de actualizaciones que faltan en los recursos

:::image type="content" source="media/custom-dashboards-azure-workbooks/system-updates-report.png" alt-text="Informe Actualizaciones del sistema de Security Center según la recomendación de seguridad de las actualizaciones que faltan":::

### <a name="use-the-vulnerability-assessment-findings-report"></a>Uso del informe Conclusiones de la evaluación de vulnerabilidades

Security Center incluye examinadores de vulnerabilidades para sus máquinas, contenedores en registros de contenedor y servidores SQL Server.

Más información sobre el uso de estos examinadores:

- [Análisis de máquinas con el examinador de VA integrado](deploy-vulnerability-assessment-vm.md)
- [Búsqueda de vulnerabilidades en las imágenes del registro](defender-for-container-registries-usage.md)
- [Examen de recursos de SQL en busca de vulnerabilidades](defender-for-sql-on-machines-vulnerability-assessment.md)

Los resultados de cada uno de estos examinadores se informan en recomendaciones independientes:

- Es necesario corregir las vulnerabilidades de las máquinas virtuales
- Es necesario corregir las vulnerabilidades de las imágenes de Azure Container Registry (con tecnología de Qualys)
- Se deben corregir las conclusiones de la evaluación de vulnerabilidades de las bases de datos SQL
- Es necesario corregir los resultados de la evaluación de vulnerabilidades de los servidores SQL Server en las máquinas

Este informe recopila estas conclusiones y las organiza por gravedad, tipo de recurso y categoría.

:::image type="content" source="media/custom-dashboards-azure-workbooks/vulnerability-assessment-findings-report.png" alt-text="Informe Conclusiones de la evaluación de vulnerabilidades de Security Center":::


## <a name="import-workbooks-from-other-workbook-galleries"></a>Importación de libros desde otras galerías de libros

Si ha creado libros en otros servicios de Azure y quiere moverlos a la galería de libros de Azure Security Center:

1. Abra el libro de destino.

1. En la barra de herramientas, seleccione **Editar**.

    :::image type="content" source="media/custom-dashboards-azure-workbooks/editing-workbooks.png" alt-text="Edición de un libro de Azure Monitor":::

1. En la barra de herramientas, seleccione **</>** para ir al Editor avanzado.

    :::image type="content" source="media/custom-dashboards-azure-workbooks/editing-workbooks-advanced-editor.png" alt-text="Inicio del Editor avanzado para obtener el código JSON de la plantilla de la galería":::

1. Copie el archivo JSON de la plantilla de la galería del libro.

1. Abra la galería de libros en Security Center y, en la barra de menús, seleccione **Nuevo**.
1. Seleccione **</>** para ir al Editor avanzado.
1. Pegue todo el código JSON de la plantilla de la galería.
1. Seleccione **Aplicar**.
1. En la barra de herramientas, seleccione **Guardar como**.

    :::image type="content" source="media/custom-dashboards-azure-workbooks/editing-workbooks-save-as.png" alt-text="Guardado del libro en la galería en Security Center":::

1. Escriba los detalles necesarios para guardar el libro:
   1. Un nombre para el libro.
   2. La región deseada.
   3. Una suscripción, un grupo de recursos y el uso compartido, según corresponda.

Encontrará el libro guardado en la categoría **Libros modificados recientemente**.


## <a name="next-steps"></a>Pasos siguientes

En este artículo se describe la página de libros de Azure Monitor integrada de Security Center con los informes integrados y la opción de crear sus propios informes personalizados e interactivos.

- Obtenga más información sobre los [libros de Azure Monitor](../azure-monitor/visualize/workbooks-overview.md).
- Los informes integrados extraen sus datos de las recomendaciones de Security Center. Obtenga información sobre las diversas recomendaciones de seguridad en [Guía de referencia sobre las recomendaciones de seguridad](recommendations-reference.md).