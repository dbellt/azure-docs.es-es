---
title: Indexación de datos mediante conectores de Power Query (versión preliminar)
titleSuffix: Azure Cognitive Search
description: Importe datos de distintos orígenes de datos mediante los conectores de Power Query.
manager: luisca
author: MarkHeff
ms.author: maheff
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/25/2021
ms.custom: references_regions
ms.openlocfilehash: d2e82dd33235db2722118fb11d650862ce00c8a3
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110483180"
---
# <a name="index-data-using-power-query-connectors-preview"></a>Indexación de datos mediante conectores de Power Query (versión preliminar)

> [!IMPORTANT] 
> La compatibilidad con los conectores de Power Query se encuentra actualmente en **versión preliminar pública validada**. [Regístrese](https://aka.ms/azure-cognitive-search/indexer-preview) para solicitar acceso.

Si usa un indizador para rastrear orígenes de datos externos para la indexación, ahora puede usar conectores de [Power Query](https://docs.microsoft.com/power-query/power-query-what-is-power-query) específicos para la conexión del origen de datos en Azure Cognitive Search.

Los conectores de Power Query pueden acceder a una gama más amplia de orígenes de datos, incluidos los de otros proveedores de nube. Los nuevos orígenes de datos admitidos en esta versión preliminar incluyen:

+ Amazon Redshift
+ Elasticsearch
+ PostgreSQL
+ Objetos de Salesforce
+ Informes de Salesforce
+ Smartsheet
+ Snowflake

En este artículo se muestra un enfoque basado en Azure Portal para configurar un indizador mediante conectores de Power Query. Actualmente, no hay compatibilidad con el SDK.

> [!NOTE]
> La funcionalidad de versión preliminar se ofrece sin un Acuerdo de Nivel de Servicio y no es aconsejable usarla para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="supported-functionality"></a>Funcionalidad admitida
Los conectores de Power Query se usan en indizadores. Un indizador de Azure Cognitive Search es un rastreador (crawler) que extrae datos y metadatos utilizables en búsquedas de un origen de datos externo y rellena un índice basado en las asignaciones de un campo a otro entre el índice y su origen de datos. Este enfoque se denomina a veces "modelo de extracción", porque el servicio extrae datos sin que sea preciso escribir código que agregue datos en un índice. Los indizadores proporcionan un método cómodo para que los usuarios indexen el contenido de su origen de datos sin tener que escribir su propio rastreador o modelo de inserción.

Los indizadores que hacen referencia a los orígenes de datos de Power Query tienen el mismo nivel de compatibilidad con conjuntos de aptitudes, programaciones, lógica de detección de cambios de límite máximo y la mayoría de los parámetros que admiten otros indizadores.

## <a name="prerequisites"></a>Requisitos previos
Antes de empezar a extraer datos de uno de los orígenes de datos admitidos, deberá asegurarse de que tiene todos los recursos configurados.
+ Servicio Azure Cognitive Search
    + El servicio Azure Cognitive Search configurado en una [región admitida](search-how-to-index-power-query-data-sources.md#regional-availability).
    + Asegúrese de que el equipo de Azure Cognitive Search haya habilitado el servicio de búsqueda para la versión preliminar. También puede registrarse para obtener la versión preliminar, rellenando [este formulario](https://aka.ms/azure-cognitive-search/indexer-preview). 
+ Cuenta de Almacenamiento de blobs de Azure
    + Se requiere una cuenta de Blob Storage para que la versión preliminar se utilice como intermediaria para los datos. Los datos fluirán desde el origen de datos, luego a Blob Storage y, a continuación, al índice. Este requisito solo existe con la versión preliminar validada inicial.

## <a name="getting-started-using-the-azure-portal"></a>Introducción al uso de Azure Portal
Azure Portal proporciona compatibilidad con los conectores de Power Query. Mediante el muestre de datos y la lectura de metadatos en el contenedor, el asistente Importar datos de Azure Cognitive Search puede crear un índice predeterminado, asignar campos de origen a campos de índice de destino y cargar el índice en una sola operación. Según el tamaño y la complejidad del origen de datos, puede tener un índice de búsqueda de texto completo y operativo en cuestión de minutos.

### <a name="step-1--prepare-source-data"></a>Paso 1: preparación de los datos de origen
Asegúrese de que el origen de datos contenga datos. El Asistente para importar datos lee los metadatos y realiza el muestreo de datos para inferir un esquema de índice, pero también carga los datos del origen de datos. Si faltan los datos, el asistente se detendrá y devolverá un error. 

### <a name="step-2--start-import-data-wizard"></a>Paso 2: inicio del asistente para la importación de datos
Una vez aprobada la versión preliminar, el equipo de Azure Cognitive Search le proporcionará un vínculo de Azure Portal que usa una marca de características para que pueda acceder a los conectores de Power Query. Abra esta página e inicie el asistente desde la barra de comandos de la página del servicio Azure Cognitive Search seleccionando **Importar datos**.

![Comando de importación de datos en el portal](./media/search-import-data-portal/import-data-cmd2.png "Inicio del Asistente para la importación de datos")

### <a name="step-3--select-your-data-source"></a>Paso 3: selección de los datos
Hay algunos orígenes de datos de los que puede extraer datos mediante esta versión preliminar. Todos los orígenes de datos que usan Power Query incluirán la leyenda "Con tecnología de Power Query" en su icono. Seleccione el origen de datos. 
 
![Selección de un origen de datos](./media/search-power-query-connectors/power-query-import-data.png "Selección de un origen de datos")

Una vez que haya seleccionado el origen de datos, seleccione **Siguiente: Configuración de los datos** para pasar a la sección siguiente.

### <a name="step-4--configure-your-data"></a>Paso 4: configuración de los datos
Una vez que haya seleccionado el origen de datos, configurará la conexión. Cada origen de datos requerirá información diferente. Para algunos orígenes de datos, la documentación de Power Query proporciona detalles adicionales sobre cómo conectarse a los datos. 

+ [PostgreSQL](https://docs.microsoft.com/power-query/connectors/postgresql)
+ [Objetos de Salesforce](https://docs.microsoft.com/power-query/connectors/salesforceobjects)
+ [Informes de Salesforce](https://docs.microsoft.com/power-query/connectors/salesforcereports)

Una vez que haya proporcionado las credenciales de conexión, seleccione **Siguiente**.

### <a name="step-5--select-your-data"></a>Paso 5: selección de los datos
El Asistente para importación mostrará una vista previa de varias tablas que están disponibles en el origen de datos. En este paso, comprobará una tabla que contiene los datos que desea importar en el índice.
 
![Vista previa de los datos](./media/search-power-query-connectors/power-query-preview-data.png "Vista previa de los datos")

Una vez que haya seleccionado la tabla, seleccione **Siguiente**.

### <a name="step-6--transform-your-data-optional"></a>Paso 6: transformación de los datos (opcional)
Los conectores de Power Query proporcionan una experiencia de interfaz de usuario enriquecida que le permite manipular los datos para que pueda enviar los datos correctos al índice. Puede quitar columnas, filtrar filas y mucho más. 

No es necesario transformar los datos antes de importarlos en Azure Cognitive Search.

![Transformación de los datos](./media/search-power-query-connectors/power-query-transform-your-data.png "Transformación de los datos") 

Para obtener más información sobre cómo transformar datos con Power Query, consulte [Uso de Power Query en Power BI Desktop](https://docs.microsoft.com/power-query/power-query-quickstart-using-power-bi). 

Cuando haya terminado de transformar los datos, seleccione **Siguiente**.

### <a name="step-7--add-azure-blob-storage"></a>Paso 7: adición de Azure Blob Storage
Actualmente, la versión preliminar del conector de Power Query requiere que proporcione una cuenta de almacenamiento de blobs. Este paso solo existe con la versión preliminar validada inicial. Esta cuenta de almacenamiento de blobs servirá como almacenamiento temporal para los datos que se mueven del origen de datos a un índice de Azure Cognitive Search.

Se recomienda proporcionar una cadena de conexión de acceso completo para la cuenta de almacenamiento: 
```
{ "connectionString" : "DefaultEndpointsProtocol=https;AccountName=<your storage account>;AccountKey=<your account key>;" }
```

Para obtener la cadena de conexión del portal de Azure, vaya a la hoja de la cuenta de almacenamiento > Configuración > Claves (para las cuentas de almacenamiento del modelo clásico) o Configuración > Claves de acceso (para las cuentas de almacenamiento de Azure Resource Manager).

Después de proporcionar un nombre de origen de datos y una cadena de conexión, seleccione "Siguiente: adición de aptitudes cognitivas (opcional)". 

### <a name="step-8--add-cognitive-skills-optional"></a>Paso 8: adición de aptitudes cognitivas (opcional)
El [enriquecimiento con IA](cognitive-search-concept-intro.md) es una extensión de indizadores que se puede usar para que el contenido sea más fácil de buscar.

Este es un paso opcional para esta versión preliminar. Cuando haya terminado, seleccione **Siguiente: personalización del índice de destino**.

### <a name="step-9--customize-target-index"></a>Paso 9: personalización del índice de destino
En la página Índice, debería mostrarse una lista de campos con un tipo de datos y una serie de casillas de verificación para configurar los atributos del índice. El asistente puede generar una lista de campos basada en metadatos y mediante el muestreo de los datos de origen.

Puede seleccionar atributos de forma masiva si activa la casilla situada en la parte superior de una columna de atributos. Elija Se puede recuperar y Se puede buscar para cada campo que se debe devolver a una aplicación cliente y está sujeto a un proceso de búsqueda de texto completo. Observará que los enteros no permiten búsquedas de texto completo o de texto parcial (los números se evalúan literalmente y suelen ser útiles en filtros).

Revise la descripción de atributos de índice y analizadores de lenguaje para más información.

Dedique un momento a la revisión de las selecciones. Una vez que se ejecuta al asistente, se crean las estructuras de datos físicas y no podrá modificar la mayoría de las propiedades de estos campos sin quitar y volver a crear todos los objetos.

![Creación del índice](./media/search-power-query-connectors/power-query-index.png "Creación del índice")

Cuando haya terminado, seleccione **Siguiente: creación de un indizador**.

### <a name="step-10--create-an-indexer"></a>Paso 10: creación de un indizador
El último paso crea el indizador. Al asignarle un nombre al indizador, este puede existir como un recurso independiente que se puede programar y administrar independientemente de los objetos de origen de datos y del de índice que se crearon en la misma secuencia del asistente.

La salida del Asistente para importar datos es un indizador que rastrea el origen de datos e importa los datos seleccionados a un índice en Azure Cognitive Search.

Al crear el indizador, opcionalmente puede elegir ejecutarlo según una programación y agregar la detección de cambios. Para agregar la detección de cambios, designe una columna como "límite máximo".

![Creación del indizador](./media/search-power-query-connectors/power-query-indexer-configuration.png "Creación del indizador")

Cuando haya terminado de rellenar esta página, seleccione **Enviar**.

## <a name="high-water-mark-change-detection-policy"></a>Directiva de detección de cambios de límite superior
Esta directiva de detección de cambios se basa en una columna de "marca de límite superior" que captura la versión o la hora en que se actualizó por última vez una fila.

### <a name="requirements"></a>Requisitos
+ Todas las inserciones especifican un valor para la columna.
+ Todas las actualizaciones de un elemento también cambian el valor de la columna.
+ El valor de esta columna aumenta con cada inserción o actualización.

## <a name="unsupported-column-names"></a>Nombres de columna no admitidos
Los nombres de campo de un índice de Azure Cognitive Search deben cumplir ciertos requisitos. Uno de estos requisitos es que no se permiten algunos caracteres, como "/". Si un nombre de columna de la base de datos no cumple estos requisitos, la detección del esquema de índice no reconocerá la columna como un nombre de campo válido, ni se mostrará esa columna como un campo sugerido para el índice. Normalmente, el uso de [asignaciones de campos](search-indexer-field-mappings.md) solucionaría este problema, pero no se admiten en el portal.

Para indexar el contenido de una columna de la tabla que tiene un nombre de campo no admitido, cambie el nombre de la columna durante la fase "Transformación de los datos" del proceso de importación de datos. Por ejemplo, puede cambiar el nombre de una columna denominada "Código postal o código postal de facturación" por "código postal". Al cambiar el nombre de la columna, la detección del esquema de índice la reconocerá como un nombre de campo válido y la agregará como una sugerencia a la definición del índice.

## <a name="regional-availability"></a>Disponibilidad regional
La versión preliminar solo está disponible para los clientes con servicios de búsqueda en las siguientes regiones:
+ Centro de EE. UU.
+ Este de EE. UU.
+ Este de EE. UU. 2
+ Centro-Norte de EE. UU
+ Norte de Europa
+ Centro-sur de EE. UU.
+ Centro-Oeste de EE. UU.
+ Oeste de Europa
+ Oeste de EE. UU.
+ Oeste de EE. UU. 2

## <a name="preview-limitations"></a>Limitaciones de vista previa
Esta versión preliminar tiene muchas características emocionantes, pero existen algunas limitaciones. En esta sección se describen las limitaciones específicas de la versión preliminar actual.
+ La extracción de datos binarios desde el origen de datos no se admite en esta versión preliminar. 
+ No se admiten otras [sesiones de depuración](cognitive-search-debug-session.md) en este momento.

## <a name="next-steps"></a>Pasos siguientes
Aprendió a extraer datos de nuevos orígenes de datos mediante conectores de Power Query. Para obtener más información sobre los indizadores, consulte [Indizadores en Azure Cognitive Search](search-indexer-overview.md).
