---
title: Clasificación y detección de datos
description: Clasificación y detección de datos para Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics
services: sql-database
ms.service: sql-db-mi
ms.subservice: security
ms.custom: sqldbrb=1
titleSuffix: Azure SQL Database, Azure SQL Managed Instance, and Azure Synapse
ms.devlang: ''
ms.topic: conceptual
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 02/17/2021
tags: azure-synapse
ms.openlocfilehash: d6b51d32b691919e0f0947d3ddb58d184424b9ad
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110534416"
---
# <a name="data-discovery--classification"></a>Clasificación y detección de datos
[!INCLUDE[appliesto-sqldb-sqlmi-asa](../includes/appliesto-sqldb-sqlmi-asa.md)]

La clasificación y detección de datos está integrada en Azure SQL Database, Instancia administrada de Azure SQL y Azure Synapse Analytics. Ofrece funcionalidades básicas para detectar, clasificar, etiquetar e informar de los datos confidenciales de las bases de datos.

Los datos más confidenciales pueden incluir información empresarial, financiera, sanitaria o personal. La detección y clasificación de estos datos puede representar un rol fundamental en el enfoque de la protección de la información de la organización. Puede servir como infraestructura para lo siguiente:

- Ayudar a satisfacer los estándares de privacidad de datos y los requisitos de cumplimiento normativo.
- Varios escenarios de seguridad, como el acceso de supervisión (auditoría) a información confidencial.
- Controlar el acceso y mejorar la seguridad de las bases de datos que contienen información altamente confidencial.

> [!NOTE]
> Para información sobre SQL Server local, consulte [Clasificación y detección de datos de SQL](/sql/relational-databases/security/sql-data-discovery-and-classification).

## <a name="what-is-data-discovery--classification"></a><a id="what-is-dc"></a>¿Qué es la clasificación y detección de datos?

La clasificación y detección de datos constituye un nuevo paradigma de protección de la información para SQL Database, SQL Managed Instance y Azure Synapse, destinado no solo a proteger la base de datos, sino también los datos. Actualmente admite las siguientes funcionalidades:

- **Detección y recomendaciones:** el motor de clasificación examina la base de datos e identifica las columnas que contienen datos potencialmente confidenciales. A continuación, le proporciona una manera sencilla de revisar y aplicar la clasificación recomendada a través de Azure Portal.

- **Etiquetado:** puede aplicar etiquetas de clasificación de confidencialidad de forma persistente a las columnas, usando los nuevos atributos de metadatos que se han agregado al motor de base de datos de SQL Server. A continuación, estos metadatos se pueden utilizar en escenarios de auditoría y protección basados en la confidencialidad.

- **Confidencialidad del conjunto de resultados de consulta:** la confidencialidad del conjunto de resultados de consulta se calcula en tiempo real con fines de auditoría.

- **Visibilidad**: puede ver el estado de clasificación de la base de datos en un panel detallado en Azure Portal. Además, puede descargar un informe en formato Excel para usarlo con fines de auditoría y cumplimiento de normas, así como para otras necesidades.

## <a name="discover-classify-and-label-sensitive-columns"></a><a id="discover-classify-columns"></a>Detección, clasificación y etiquetado de columnas confidenciales

En esta sección se describen los pasos para:

- La detección, clasificación y etiquetado de columnas que contienen información confidencial en la base de datos.
- La visualización del estado de clasificación actual de la base de datos y exportación de informes.

La clasificación incluye dos atributos de metadatos:

- **Etiquetas**: los atributos principales de clasificación, que se usan para definir el nivel de confidencialidad de los datos almacenados en la columna.  
- **Tipos de información**: atributos que proporcionan información más detallada sobre el tipo de datos almacenados en la columna.

### <a name="define-and-customize-your-classification-taxonomy"></a>Definir y personalizar la taxonomía de clasificación

La función de clasificación y detección de datos incluye un conjunto integrado de etiquetas de confidencialidad y un conjunto integrado de tipos de información y lógica de detección. Puede personalizar esta taxonomía y definir un conjunto y la categoría de construcciones de clasificación específicamente para su entorno.

Se define y personaliza la taxonomía de clasificación en una ubicación central para toda la organización de Azure. Esa ubicación se encuentra en [Azure Security Center](../../security-center/security-center-introduction.md), como parte de la directiva de seguridad. Solo un usuario con derechos administrativos en el grupo de administración raíz de la organización puede realizar esta tarea.

Como parte de la administración de directivas de protección de la información, puede definir etiquetas personalizadas, clasificarlas y asociarlas con un conjunto seleccionado de tipos de información. También puede agregar sus propios tipos de información personalizados y configurarlos con patrones de cadena. Los patrones se agregan a la lógica de detección para identificar este tipo de datos en las bases de datos.

Para más información, consulte [Personalización de la directiva de SQL Information Protection en Azure Security Center (versión preliminar)](../../security-center/security-center-info-protection-policy.md).

Una vez definida la directiva de toda la organización, puede continuar con la clasificación de bases de datos individuales mediante la directiva personalizada.

### <a name="classify-your-database"></a>Clasificación de la base de datos

> [!NOTE]
> En el ejemplo siguiente se usa Azure SQL Database, pero debe seleccionar el producto adecuado para el que quiera configurar la clasificación y detección de datos.

1. Vaya a [Azure Portal](https://portal.azure.com).

1. Vaya a **Clasificación y detección de datos** en el encabezado **Seguridad** del panel de Azure SQL Database. La pestaña Información general incluye un resumen del estado de clasificación actual de la base de datos. El resumen incluye una lista detallada de todas las columnas clasificadas, que también se pueden filtrar para mostrar solo partes específicas de esquema, tipos de información y etiquetas. Si aún no ha clasificado ninguna columna, [vaya al paso 4](#step-4).

    ![Información general](./media/data-discovery-and-classification-overview/data-discovery-and-classification.png)

1. Para descargar un informe en formato de Excel, seleccione **Exportar** en el menú superior del panel.

1. <a id="step-4"></a>Para empezar a clasificar los datos, seleccione la pestaña **Clasificación** en la página **Clasificación y detección de datos**.

    El motor de clasificación examina la base de datos en busca de columnas que contengan datos potencialmente confidenciales y proporciona una lista de clasificaciones de columna recomendadas.

1. Visualice y aplique las recomendaciones de clasificación:

   - Para ver la lista de clasificaciones de columnas recomendadas, seleccione el panel de recomendaciones en la parte inferior del panel.

   - Para aceptar una recomendación para una columna específica, seleccione la casilla en la columna izquierda de la fila correspondiente. Para marcar todas las recomendaciones como aceptadas, seleccione la casilla más a la izquierda del encabezado de la tabla de recomendaciones.

   - Para aplicar las recomendaciones seleccionadas, seleccione **Aceptar las recomendaciones seleccionadas**.

   ![Recomendaciones para la clasificación](./media/data-discovery-and-classification-overview/recommendation.png)

1. También puede clasificar las columnas manualmente, como alternativa a la clasificación basada en recomendaciones, o además de ella:

   1. Seleccione **Agregar clasificación** en el menú superior del panel.

   1. En la ventana contextual que se abre, seleccione el esquema, la tabla y la columna que quiera clasificar, así como el tipo de información y la etiqueta de confidencialidad.

   1. Seleccione **Agregar clasificación** en la parte inferior de la ventana contextual.

   ![Agregue clasificación de forma manual](./media/data-discovery-and-classification-overview/manually-add-classification.png)


1. Para completar la clasificación y etiquetar de forma persistente las columnas de la base de datos con los nuevos metadatos de clasificación, seleccione **Guardar** en la página **Clasificación**.

## <a name="audit-access-to-sensitive-data"></a><a id="audit-sensitive-data"></a>Auditoría del acceso a datos confidenciales

Un aspecto importante del paradigma de protección de la información es la capacidad de supervisar el acceso a información confidencial. [Auditoría de Azure SQL](../../azure-sql/database/auditing-overview.md) se ha mejorado para incluir un nuevo campo en el registro de auditoría denominado `data_sensitivity_information`. Este campo registra las clasificaciones de confidencialidad (etiquetas) de los datos devueltos por una consulta. Este es un ejemplo:

![Registro de auditoría](./media/data-discovery-and-classification-overview/11_data_classification_audit_log.png)

## <a name="permissions"></a><a id="permissions"></a>Permisos

Estos roles integrados pueden leer la clasificación de datos de una base de datos:

- Propietario
- Lector
- Colaborador
- Administrador de seguridad SQL
- Administrador de acceso de usuario

Estos roles integrados pueden modificar la clasificación de datos de una base de datos:

- Propietario
- Colaborador
- Administrador de seguridad SQL

Obtenga información sobre los permisos basados en roles en [RBAC de Azure](../../role-based-access-control/overview.md).

## <a name="manage-classifications"></a><a id="manage-classification"></a>Administración de clasificaciones

Puede usar T-SQL, una API de REST o PowerShell para administrar las clasificaciones.

### <a name="use-t-sql"></a>Uso de T-SQL

Puede utilizar T-SQL para agregar o quitar las clasificaciones de columna, y para recuperar todas las clasificaciones para toda la base de datos.

> [!NOTE]
> Al usar T-SQL para administrar las etiquetas, no se valida que las etiquetas que se agregan a una columna existan en la directiva de protección de información de la organización (el conjunto de etiquetas que aparecen en las recomendaciones del portal). Por lo tanto, validarlas es su decisión.

Para obtener información sobre el uso de T-SQL para las clasificaciones, vea las siguientes referencias:

- Para agregar o actualizar la clasificación de una o varias columnas: [ADD SENSITIVITY CLASSIFICATION](/sql/t-sql/statements/add-sensitivity-classification-transact-sql) (Agregar clasificación de confidencialidad)
- Para quitar la clasificación de una o varias columnas: [DROP SENSITIVITY CLASSIFICATION](/sql/t-sql/statements/drop-sensitivity-classification-transact-sql) (Eliminar clasificación de la confidencialidad)
- Para ver todas las clasificaciones de la base de datos: [sys.sensitivity_classifications](/sql/relational-databases/system-catalog-views/sys-sensitivity-classifications-transact-sql)

### <a name="use-powershell-cmdlets"></a>Uso de cmdlets de PowerShell
Administre clasificaciones y las recomendaciones de Azure SQL Database e Instancia administrada de Azure SQL mediante PowerShell.

#### <a name="powershell-cmdlets-for-azure-sql-database"></a>Cmdlets de PowerShell para Azure SQL Database

- [Get-AzSqlDatabaseSensitivityClassification](/powershell/module/az.sql/get-azsqldatabasesensitivityclassification)
- [Set-AzSqlDatabaseSensitivityClassification](/powershell/module/az.sql/set-azsqldatabasesensitivityclassification)
- [Remove-AzSqlDatabaseSensitivityClassification](/powershell/module/az.sql/remove-azsqldatabasesensitivityclassification)
- [Get-AzSqlDatabaseSensitivityRecommendation](/powershell/module/az.sql/get-azsqldatabasesensitivityrecommendation)
- [Enable-AzSqlDatabaSesensitivityRecommendation](/powershell/module/az.sql/enable-azsqldatabasesensitivityrecommendation)
- [Disable-AzSqlDatabaseSensitivityRecommendation](/powershell/module/az.sql/disable-azsqldatabasesensitivityrecommendation)

#### <a name="powershell-cmdlets-for-azure-sql-managed-instance"></a>Cmdlets de PowerShell para Instancia administrada de Azure SQL

- [Get-AzSqlInstanceDatabaseSensitivityClassification](/powershell/module/az.sql/get-azsqlinstancedatabasesensitivityclassification)
- [Set-AzSqlInstanceDatabaseSensitivityClassification](/powershell/module/az.sql/set-azsqlinstancedatabasesensitivityclassification)
- [Remove-AzSqlInstanceDatabaseSensitivityClassification](/powershell/module/az.sql/remove-azsqlinstancedatabasesensitivityclassification)
- [Get-AzSqlInstanceDatabaseSensitivityRecommendation](/powershell/module/az.sql/get-azsqlinstancedatabasesensitivityrecommendation)
- [Enable-AzSqlInstanceDatabaseSensitivityRecommendation](/powershell/module/az.sql/enable-azsqlinstancedatabasesensitivityrecommendation)
- [Disable-AzSqlInstanceDatabaseSensitivityRecommendation](/powershell/module/az.sql/disable-azsqlinstancedatabasesensitivityrecommendation)

### <a name="use-the-rest-api"></a>Uso de la API de REST

Puede usar las API REST para administrar las clasificaciones y recomendaciones mediante programación. Las API de REST publicadas admiten las siguientes operaciones:

- [Crear o actualizar](/rest/api/sql/sensitivitylabels/createorupdate): crea o actualiza la etiqueta de confidencialidad de la columna especificada.
- [Eliminar](/rest/api/sql/sensitivitylabels/delete): elimina la etiqueta de confidencialidad de la columna especificada.
- [Deshabilitar la recomendación](/rest/api/sql/sensitivitylabels/disablerecommendation): deshabilita las recomendaciones de confidencialidad en la columna especificada.
- [Habilitar la recomendación](/rest/api/sql/sensitivitylabels/enablerecommendation): habilita las recomendaciones de confidencialidad en la columna especificada. (Las recomendaciones están habilitadas de forma predeterminada en todas las columnas).
- [Obtener](/rest/api/sql/sensitivitylabels/get): obtiene la etiqueta de confidencialidad de la columna especificada.
- [Enumerar las actuales por base de datos](/rest/api/sql/sensitivitylabels/listcurrentbydatabase): obtiene las etiquetas de confidencialidad actuales de la base de datos especificada.
- [Enumerar las recomendadas por base de datos](/rest/api/sql/sensitivitylabels/listrecommendedbydatabase): obtiene las etiquetas de confidencialidad recomendadas de la base de datos especificada.


## <a name="faq---advanced-classification-capabilities"></a>Preguntas más frecuentes: funcionalidades de clasificación avanzadas

**Pregunta**: ¿[Azure Purview](../../purview/overview.md) reemplazará a Clasificación y detección de datos de SQL o Clasificación y detección de datos de SQL se retirará pronto?
**Respuesta**: Seguimos admitiendo Clasificación y detección de datos de SQL y le animamos a que adopte [Azure Purview](../../purview/overview.md), que tiene capacidades más ricas para impulsar las funciones de clasificación avanzadas y la gobernanza de datos. Si decidimos retirar cualquier servicio, característica, API o SKU, recibirá un aviso previo con una ruta de migración o de transición. Obtenga más información sobre las directivas del ciclo de vida de Microsoft aquí.


## <a name="next-steps"></a><a id="next-steps"></a>Pasos siguientes

- Considere la posibilidad de configurar [Auditoría de Azure SQL](../../azure-sql/database/auditing-overview.md) para supervisar y auditar el acceso a los datos confidenciales clasificados.
- Para ver una presentación que incluye la detección y clasificación de datos, consulte [Detección, clasificación, etiquetado y protección de datos de SQL | Datos expuestos](https://www.youtube.com/watch?v=itVi9bkJUNc).
- Para clasificar las instancias de Azure SQL Database y Azure Synapse Analytics con las etiquetas de Azure Purview mediante comandos T-SQL, consulte [Clasificación de los datos de Azure SQL mediante etiquetas de Azure Purview](../../sql-database/scripts/sql-database-import-purview-labels.md).
