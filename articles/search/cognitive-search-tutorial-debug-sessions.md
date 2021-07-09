---
title: 'Tutorial: Uso de sesiones de depuración para reparar conjuntos de aptitudes'
titleSuffix: Azure Cognitive Search
description: Las sesiones de depuración (versión preliminar) son una herramienta de Azure Portal que se usa para detectar, diagnosticar y reparar los problemas de un conjunto de aptitudes.
author: HeidiSteen
ms.author: heidist
manager: nitinme
ms.service: cognitive-search
ms.topic: tutorial
ms.date: 02/02/2021
ms.openlocfilehash: 0560c7e1f8770dd27d8ad462d10a2668d69d5865
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111559256"
---
# <a name="tutorial-debug-a-skillset-using-debug-sessions"></a>Tutorial: Depuración de un conjunto de aptitudes mediante sesiones de depuración

Los conjuntos de aptitudes coordinan una serie de acciones que analizan o transforman el contenido, donde la salida de una aptitud se convierte en la entrada de otra. En los casos en que las entradas dependen de las salidas, los errores en las definiciones de los conjuntos de aptitudes y las asociaciones de campos pueden dar lugar a operaciones y datos que faltan.

Las **sesiones de depuración** de Azure Portal proporcionan una visualización holística de un conjunto de aptitudes. Con esta herramienta, puede explorar en profundidad los pasos específicos para ver fácilmente dónde se puede estar produciendo un error en una acción.

En este artículo, usará **sesiones de depuración** para buscar y corregir 1) una entrada que falta y 2) las asignaciones de campos de salida. Este es un tutorial completo. Proporciona datos para indexar (datos de pruebas clínicas), una colección de Postman que crea objetos e instrucciones para usar las **sesiones de depuración** para detectar y corregir los problemas de un conjunto de aptitudes.

> [!Important]
> Las sesiones de depuración constituyen una característica en versión preliminar que se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).
>

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar, tiene que cumplir los siguientes requisitos previos:

+ Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/).

+ Un servicio de Azure Cognitive Search. [Cree un servicio](search-create-service-portal.md) o [busque uno existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Search%2FsearchServices) en su suscripción actual. Puede usar un servicio gratuito para este inicio rápido. 

+ Una cuenta de Azure Storage con [Blob Storage](../storage/blobs/index.yml) que se usará para hospedar los datos de ejemplo y conservar los datos temporales que se creen durante una sesión de depuración.

+ Una [aplicación de escritorio de Postman](https://www.getpostman.com/) y una [colección de Postman](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Debug-sessions) para crear objetos mediante las API REST.

+ [Datos de ejemplo (pruebas clínicas)](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/clinical-trials-pdf-19).

> [!NOTE]
> En esta guía de inicio rápido también se usa [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/) para la inteligencia artificial. Dado que la carga de trabajo es tan pequeña, Cognitive Services se aprovecha en segundo plano del procesamiento gratuito de hasta 20 transacciones. Esto significa que puede completar este ejercicio sin tener que crear un recurso de Cognitive Services adicional.

## <a name="set-up-your-data"></a>Configuración de los datos

En esta sección se crea el conjunto de datos de ejemplo en Azure Blob Storage para que el indexador y el conjunto de aptitudes tengan contenido con el que trabajar.

1. [Descargue los datos de ejemplo (clinical-trials-pdf-19)](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/clinical-trials-pdf-19), que constan de 19 archivos.

1. [Cree una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal) o [busque una cuenta existente](https://ms.portal.azure.com/#blade/HubsExtension/BrowseResourceBlade/resourceType/Microsoft.Storage%2storageAccounts/). 

   + Elija la misma región de Azure Cognitive Search para evitar cargos de ancho de banda.

   + Elija el tipo de cuenta StorageV2 (uso general V2).

1. Vaya a las páginas de servicios de Azure Storage en el portal y cree un contenedor de blobs. El procedimiento recomendado es especificar el nivel de acceso "privado". Asigne al contenedor el nombre `clinicaltrialdataset`.

1. En el contenedor, haga clic en **Cargar** para cargar los archivos de ejemplo que descargó y descomprimió en el primer paso.

1. En el portal, obtenga y guarde la cadena de conexión para Azure Storage. La necesitará para las llamadas API REST que indexan los datos. Puede obtener la cadena de conexión en **Configuración** > **Claves de acceso** en el portal.

## <a name="get-a-key-and-url"></a>Obtención de una clave y una dirección URL

Las llamadas de REST requieren la dirección URL del servicio y una clave de acceso en cada solicitud. Con ambos se crea un servicio de búsqueda, por lo que, si ha agregado Azure Cognitive Search a su suscripción, siga estos pasos para obtener la información necesaria:

1. [Inicie sesión en Azure Portal](https://portal.azure.com/) y en la página **Introducción** del servicio de búsqueda, obtenga la dirección URL. Un punto de conexión de ejemplo podría ser similar a `https://mydemo.search.windows.net`.

1. En **Configuración** > **Claves**, obtenga una clave de administrador para tener derechos completos en el servicio. Se proporcionan dos claves de administrador intercambiables para lograr la continuidad empresarial, por si necesitara sustituir una de ellas. Puede usar la clave principal o secundaria en las solicitudes para agregar, modificar y eliminar objetos.

   :::image type="content" source="media/search-get-started-rest/get-url-key.png" alt-text="Obtención de una clave de acceso y un punto de conexión HTTP" border="false":::

Todas las solicitudes requieren una clave de API en cada solicitud enviada al servicio. Tener una clave válida genera la confianza, solicitud a solicitud, entre la aplicación que envía la solicitud y el servicio que se encarga de ella.

## <a name="create-data-source-skillset-index-and-indexer"></a>Creación de un origen de datos, un conjunto de aptitudes, un índice y un indexador

En esta sección, se usan Postman y una colección proporcionada para crear el origen de datos de Cognitive Search, el conjunto de aptitudes, el índice y el indexador. Si no está familiarizado con Postman, consulte [este inicio rápido](search-get-started-rest.md).

Para completar esta tarea, necesitará la [colección de Postman](https://github.com/Azure-Samples/azure-search-postman-samples/tree/master/Debug-sessions) creada para este tutorial. 

1. Inicie Postman e importe la colección. En **Archivos** > **Nuevo**, seleccione la colección que desee importar.
1. Una vez importada la colección, expanda la lista de acciones (...).
1. Haga clic en **Editar**.
1. En Valor actual, escriba el nombre del `searchService` (por ejemplo, si el punto de conexión es `https://mydemo.search.windows.net`, el nombre del servicio es "`mydemo`").
1. Especifique la `apiKey` de la clave principal o secundaria del servicio de búsqueda.
1. Escriba el valor de `storageConnectionString` de la página de claves de la cuenta de Azure Storage.
1. Escriba el valor de `containerName` del contenedor que creó en la cuenta de almacenamiento y, después, haga clic en **Actualizar**.

   :::image type="content" source="media/cognitive-search-debug/postman-enter-variables.png" alt-text="Edición de variables en Postman":::

La colección contiene cuatro llamadas REST diferentes que se usan para crear los objetos que se utilizan en este tutorial.

La primera llamada crea el origen de datos. `clinical-trials-ds`. La segunda llamada crea el conjunto de aptitudes, `clinical-trials-ss`. La tercera llamada crea el índice, `clinical-trials`. La cuarta y última llamada crea el indexador, `clinical-trials-idxr`.

+ Abra cada solicitud, una a una, y haga clic en **Enviar** para enviar cada solicitud al servicio de búsqueda. 

Una vez completadas todas las llamadas de la colección, cierre Postman y vuelva a Azure Portal.

## <a name="check-results-in-the-portal"></a>Comprobación de los resultados en el portal

El código de ejemplo crea intencionadamente un índice erróneo como consecuencia de problemas que se produjeron durante la ejecución del conjunto de aptitudes. El problema consiste en que faltan datos. 

1. En Azure Portal, en la página de información general del servicio de búsqueda, seleccione la pestaña **Índices**. 
1. Seleccione el índice `clinical-trials`.
1. Escriba esta cadena de consulta: `$select=metadata_storage_path, organizations, locations&$count=true` para devolver los campos de documentos específicos (identificados por el campo `metadata_storage_path` exclusivo).
1. Haga clic en **Buscar** para ejecutar la consulta, la cual devolverá los 19 documentos, mostrando los valores vacíos de "organizaciones" y "ubicaciones".

Estos campos se deben haber rellenado mediante la [aptitud de reconocimiento de entidades](cognitive-search-skill-entity-recognition-v3.md) del conjunto de aptitudes que se usa para buscar organizaciones y ubicaciones en el contenido del blob. En el siguiente ejercicio, se usará una sesión de depuración para determinar qué salió mal.

Otra manera de investigar los errores y advertencias es mediante Azure Portal.

1. Abra la pestaña **Indizadores** y seleccione `clinical-trials-idxr`.
1. Tenga en cuenta que aunque el trabajo del indexador se completó correctamente en general, hubo 57 advertencias.
1. Haga clic en **Operación correcta** para ver las advertencias (si se produjeron principalmente errores, el vínculo de detalle sería **Con errores**). Verá una lista larga de cada advertencia que emite el indexador.

   :::image type="content" source="media/cognitive-search-debug/portal-indexer-errors-warnings.png" alt-text="Ver advertencias":::

## <a name="start-your-debug-session"></a>Inicio de la sesión de depuración

:::image type="content" source="media/cognitive-search-debug/new-debug-session-screen-required.png" alt-text="Inicio de una nueva sesión de depuración":::

1. En la página de información general de la búsqueda, haga clic en la pestaña **Sesiones de depuración**.
1. Seleccione **+ Nueva sesión de depuración**.
1. Asigne un nombre a la sesión. 
1. Conecte la sesión a su cuenta de almacenamiento. 
1. En la plantilla del indexador, proporcione el nombre de este. El indexador hace referencia al origen de datos, al conjunto de aptitudes y al índice.
1. Acepte la opción de documento predeterminada para el primer documento de la colección. 
1. **Guarde** la sesión. Al guardar la sesión se iniciará la canalización de enriquecimiento con IA tal y como se define en el conjunto de aptitudes.

> [!Important]
> Una sesión de depuración solo funciona con un único documento. Puede elegir qué documento quiere depurar o simplemente utilizar el primero.

<!-- > :::image type="content" source="media/cognitive-search-debug/debug-execution-complete1.png" alt-text="New debug session started"::: -->

Una vez completada la inicialización de la sesión de depuración, la sesión muestra de forma predeterminada la pestaña **Enriquecimientos con IA**, resaltando el **gráfico de aptitudes**. El gráfico de aptitudes proporciona una jerarquía visual del conjunto de aptitudes y su orden de ejecución secuencial y en paralelo.

## <a name="find-issues-with-the-skillset"></a>Búsqueda de los problemas del conjunto de aptitudes

Los problemas que notifica el indexador se pueden encontrar en la pestaña adyacente **Errores o advertencias**. 

Observe que la pestaña **Errores o advertencias** proporcionará una lista mucho más reducida que la mostrada anteriormente, ya que esta lista solo detalla los errores de un único documento. Al igual que la lista mostrada por el indexador, puede hacer clic en un mensaje de advertencia y ver los detalles de dicha advertencia.

Seleccione **Errores o advertencias** para revisar las notificaciones. Debería ver tres:

   + "No se pudo asignar el campo de salida "locations" al índice de búsqueda. Compruebe la propiedad "outputFieldMappings" del indexador.
Falta el valor "/document/merged_content/locations"."

   + "No se pudo asignar el campo de salida "organizations" al índice de búsqueda. Compruebe la propiedad "outputFieldMappings" del indexador.
Falta el valor "/document/merged_content/organizations"."

   + "La aptitud se ejecutó pero puede tener resultados inesperados porque una o varias entradas de aptitudes no eran válidas.
Falta la entrada de aptitud opcional. Nombre: "languageCode", origen: "/document/languageCode". Problemas de análisis del lenguaje de expresiones: Falta el valor "/document/languageCode"."

   Muchas aptitudes tienen un parámetro "languageCode". Al inspeccionar la operación, puede ver que falta esta entrada de código de idioma de la aptitud `Enrichment.NerSkillV2.#1`, que es la misma aptitud de reconocimiento de entidades que tiene problemas con la salida de "locations" (ubicaciones) y "organizations" (organizaciones). 

Dado que las tres notificaciones son sobre esta aptitud, el siguiente paso es depurarla. Si es posible, empiece resolviendo primero los problemas de la entrada antes de pasar a los problemas de outputFieldMapping.

 :::image type="content" source="media/cognitive-search-debug/debug-session-errors-warnings.png" alt-text="Nueva sesión de depuración iniciada":::

<!-- + The Skill Graph provides a visual hierarchy of the skillset and its order of execution from top to bottom. Skills that are side by side in the graph are executed in parallel. Color coding of skills in the graph indicate the types of skills that are being executed in the skillset. In the example, the green skills are text and the red skill is vision. Clicking on an individual skill in the graph will display the details of that instance of the skill in the right pane of the session window. The skill settings, a JSON editor, execution details, and errors/warnings are all available for review and editing.

+ The Enriched Data Structure details the nodes in the enrichment tree generated by the skills from the source document's contents. -->

## <a name="fix-missing-skill-input-value"></a>Corrección de valores de entrada de aptitud que faltan

En la pestaña **Errores o advertencias**, se muestra un error en una operación con la etiqueta `Enrichment.NerSkillV2.#1`. Los detalles de este error explican que hay un problema con un valor de entrada de la aptitud "/document/languageCode". 

1. Vuelva a la pestaña **Enriquecimientos con IA**.
1. Haga clic en el **gráfico de aptitudes**.
1. Haga clic en la aptitud **n.º 1** para que aparezcan sus detalles en el panel derecho.
1. Busque la entrada de "languageCode".
1. Seleccione el símbolo **</>** al principio de la línea y abra el evaluador de expresiones.
1. Haga clic en el botón **Evaluar** para confirmar que esta expresión genera un error. Esto confirmará que la propiedad "languageCode" no es una entrada válida.

   :::image type="content" source="media/cognitive-search-debug/expression-evaluator-language.png" alt-text="Evaluador de expresiones":::

Existen dos maneras de investigar este error en la sesión. La primera es examinar de dónde procede la entrada. ¿Qué aptitud de la jerarquía se supone que genera este resultado? En la pestaña Ejecuciones del panel de detalles de la aptitud debe aparecer el origen de la entrada. Si no hay ningún origen, esto indica un error de asignación de campos.

1. Haga clic en la pestaña **Ejecuciones**.
1. Examine las entradas y busque "languageCode". No hay ningún origen para esta entrada en la lista. 
1. Cambie el panel izquierdo para mostrar la estructura de datos enriquecidos. No hay ninguna ruta de acceso asignada que se corresponda con "languageCode".

   :::image type="content" source="media/cognitive-search-debug/enriched-data-structure-language.png" alt-text="Estructura de datos enriquecidos":::

Hay una ruta de acceso asignada para "language". Por lo tanto, hay un error tipográfico en la configuración de aptitudes. Para corregirlo, se debe actualizar la expresión de la aptitud #1 con la expresión "/document/language".

1. Abra el evaluador de expresiones **</>** para la ruta de acceso "language".
1. Copie la expresión. Cierre la ventana.
1. Vaya a la configuración de aptitudes de la aptitud #1 y abra el evaluador de expresiones **</>** para la entrada "languageCode".
1. Pegue el nuevo valor, "/document/language", en el cuadro Expresión y haga clic en **Evaluar**.
1. Debe mostrar la entrada correcta "en". Haga clic en Aplicar para actualizar la expresión.
1. Haga clic en **Guardar** en el panel derecho de detalles de la aptitud.
1. Haga clic en **Ejecutar** en el menú de la ventana de la sesión. Esto iniciará otra ejecución del conjunto de aptitudes con el documento. 

Una vez finalizada la ejecución de la sesión de depuración, haga clic en la pestaña Errores o advertencias, que mostrará que el error "Enrichment.NerSkillV2.#1" ha desaparecido. Sin embargo, sigue habiendo dos advertencias que indican que el servicio no pudo asignar campos de salida para organizaciones y ubicaciones al índice de búsqueda. Faltan los valores "/document/merged_content/organizations" y "/document/merged_content/locations".

## <a name="fix-missing-skill-output-values"></a>Corrección de valores de salida de aptitud que faltan

:::image type="content" source="media/cognitive-search-debug/warnings-missing-value-locations-organizations.png" alt-text="Errores y advertencias":::

Faltan valores de salida de una aptitud. Para identificar la aptitud que tiene el error, vaya a la estructura de datos enriquecidos, busque el nombre del valor y observe su origen. En el caso de valores de organizaciones y ubicaciones que faltan, son salidas de la aptitud #1. Al abrir el evaluador de expresiones </> para cada ruta de acceso, se mostrarán las expresiones enumeradas como "/document/content/organizations" y "/document/content/locations" respectivamente.

:::image type="content" source="media/cognitive-search-debug/expression-eval-missing-value-locations-organizations.png" alt-text="Entidad de organizaciones del evaluador de expresiones":::

La salida de estas entidades está vacía y no debería ser así. ¿Cuáles son las entradas que producen este resultado?

1. Vaya al **gráfico de aptitudes** y seleccione la aptitud #1.
1. En el panel de detalles de la aptitud de la derecha, seleccione la pestaña **Ejecuciones**.
1. Abra el evaluador de expresiones **</>** para la entrada "text".

   :::image type="content" source="media/cognitive-search-debug/input-skill-missing-value-locations-organizations.png" alt-text="Entrada de aptitud de texto":::

El resultado mostrado para esta entrada no parece una entrada de texto. Parece una imagen que está rodeada por líneas nuevas. La falta de texto indica que no se puede identificar ninguna entidad. Al examinar la jerarquía del conjunto de aptitudes, se muestra que la aptitud #6 (OCR) procesa primero el contenido que, a continuación, se pasa a la aptitud #5 (Combinar). 

1. Seleccione la aptitud #5 (Combinar) en el **gráfico de aptitudes**.
1. Seleccione la pestaña **Ejecuciones** en el panel de detalles de la aptitud de la derecha y abra el evaluador de expresiones **</>** para las salidas "mergedText".

   :::image type="content" source="media/cognitive-search-debug/merge-output-detail-missing-value-locations-organizations.png" alt-text="Salida de la aptitud de combinación":::

Aquí el texto se empareja con la imagen. Al examinar la expresión "/document/merged_content", se puede ver el error en las rutas de acceso de las entidades "organizations" y "locations" de la aptitud #1. En lugar de usar "/document/content", debe usar "/document/merged_content" para las entradas de "text".

1. Copie la expresión de la salida "mergedText" y cierre la ventana del evaluador de expresiones.
1. Seleccione la aptitud #1 en el **gráfico de aptitudes**.
1. En el panel de detalles de la aptitud de la derecha, seleccione la pestaña **Configuración de las aptitudes**.
1. Abra el evaluador de expresiones **</>** para la entrada "text".
1. Pegue la nueva expresión en el cuadro. Haga clic en **Evaluar**.
1. Se debe mostrar la entrada correcta con el texto agregado. Haga clic en **Aplicar** para actualizar la configuración de las aptitudes.
1. Haga clic en **Guardar** en el panel derecho de detalles de la aptitud.
1. Haga clic en **Ejecutar** en el menú de la ventana de sesiones. Esto iniciará otra ejecución del conjunto de aptitudes con el documento.

Una vez finalizada la ejecución del indexador, los errores siguen estando ahí. Vuelva a la aptitud #1 e investigue. La entrada de la aptitud se cambió de "content" a "merged_content". ¿Cuáles son las salidas de estas entidades en la aptitud?

1. Seleccione la pestaña **Enriquecimientos con IA**.
1. Seleccione el **gráfico de aptitudes** y haga clic en la aptitud #1.
1. Navegue por la **configuración de las aptitudes** y busque "outputs".
1. Abra el evaluador de expresiones **</>** para la entidad "organizations".

   :::image type="content" source="media/cognitive-search-debug/skill-output-detail-missing-value-locations-organizations.png" alt-text="Salida para la entidad &quot;organizations&quot;":::

Al evaluar el resultado de la expresión, se obtiene el resultado correcto. La aptitud trabaja para identificar el valor correcto de la entidad "organizations". Sin embargo, la asignación de salida en la ruta de acceso de la entidad sigue produciendo un error. Al comparar la ruta de acceso de salida de la aptitud con la ruta de acceso de salida del error, la aptitud es el objeto primario en una relación jerárquica de las salidas, organizaciones y ubicaciones en el nodo /document/content. Sin embargo, la asignación de campos de salida espera los resultados en el nodo /document/merged_content. En el paso anterior, la entrada cambió de "/document/content" a "/document/merged_content". El contexto de la configuración de aptitudes debe cambiarse para asegurarse de que la salida se genera con el contexto correcto.

1. Seleccione la pestaña **Enriquecimientos con IA**.
1. Seleccione el **gráfico de aptitudes** y haga clic en la aptitud #1.
1. Navegue por la **configuración de las aptitudes** y busque "context".
1. Haga doble clic en el valor de "context" y cámbielo a "/document/merged_content".
1. Haga clic en **Guardar** en el panel derecho de detalles de la aptitud.
1. Haga clic en **Ejecutar** en el menú de la ventana de sesiones. Esto iniciará otra ejecución del conjunto de aptitudes con el documento.

   :::image type="content" source="media/cognitive-search-debug/skill-setting-context-correction-missing-value-locations-organizations.png" alt-text="Corrección de contexto en la configuración de aptitudes":::

Todos los errores se han resuelto.

## <a name="commit-changes-to-the-skillset"></a>Confirmación de los cambios en el conjunto de aptitudes

Cuando se inició la sesión de depuración, el servicio de búsqueda creó una copia del conjunto de aptitudes. Esto se hizo para proteger el conjunto de aptitudes original en el servicio de búsqueda. Ahora que ha finalizado la depuración del conjunto de aptitudes, las correcciones se pueden confirmar (sobrescribir el conjunto de aptitudes original). 

Como alternativa, si no está listo para confirmar los cambios, puede guardar la sesión de depuración y volver a abrirla más adelante.

1. Haga clic en **Confirmar cambios** en el menú principal de sesiones de depuración.
1. Haga clic en **Aceptar** para confirmar que desea actualizar el conjunto de aptitudes.
1. Cierre la sesión de depuración y seleccione la pestaña **Indizadores**.
1. Abra 'clinical-trials-idxr'.
1. Haga clic en **Restablecer**.
1. Haga clic en **Ejecutar**. Haga clic en **ACEPTAR** para continuar.

Cuando el indexador haya terminado de ejecutarse, aparecerá una marca de verificación verde y la palabra Correcto junto a la marca de tiempo de la última ejecución en la pestaña **Historial de ejecución**. Para asegurarse de que se han aplicado los cambios:

1. En la página de información general de la búsqueda, seleccione la pestaña **Índice**.
1. Abra el índice de "clinical-trials" y, en la pestaña Explorador de búsqueda, escriba esta cadena de consulta: `$select=metadata_storage_path, organizations, locations&$count=true` para que devuelva los campos de documentos específicos (identificados mediante el campo `metadata_storage_path` único).
1. Haga clic en **Buscar**.

Los resultados deberían mostrar que las organizaciones y las ubicaciones ahora contienen los valores esperados.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando trabaje con su propia suscripción, es una buena idea al final de un proyecto identificar si todavía se necesitan los recursos que ha creado. Los recursos que se dejan en ejecución pueden costarle mucho dinero. Puede eliminar los recursos de forma individual o eliminar el grupo de recursos para eliminar todo el conjunto de recursos.

Puede encontrar y administrar recursos en el portal, mediante el vínculo **Todos los recursos** o **Grupos de recursos** en el panel de navegación izquierdo.

Si está usando un servicio gratuito, recuerde que está limitado a tres índices, indexadores y orígenes de datos. Puede eliminar elementos individuales en el portal para mantenerse por debajo del límite. 

## <a name="next-steps"></a>Pasos siguientes

Este tutorial ha tratado sobre varios aspectos de la definición y el procesamiento de los conjuntos de aptitudes. Para más información sobre conceptos y flujos de trabajo, consulte los siguientes artículos:

+ [Asignación de campos de salida de conjuntos de aptitudes a campos de un índice de búsqueda](cognitive-search-output-field-mapping.md)

+ [Conjuntos de aptitudes de Azure Cognitive Search](cognitive-search-working-with-skillsets.md)

+ [Configuración del almacenamiento en caché para el enriquecimiento incremental](cognitive-search-incremental-indexing-conceptual.md)