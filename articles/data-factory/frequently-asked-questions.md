---
title: 'Azure Data Factory: Preguntas más frecuentes '
description: Obtenga respuestas a las preguntas más frecuentes acerca de Azure Data Factory.
author: ssabat
ms.author: susabat
ms.service: data-factory
ms.topic: conceptual
ms.date: 04/29/2021
ms.openlocfilehash: d3cc2d73fb3f1076af62b8ea028260bfd5e600ed
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108317998"
---
# <a name="azure-data-factory-faq"></a>Preguntas más frecuentes de Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Este artículo ofrece respuestas a las preguntas más frecuentes sobre Azure Data Factory.  

## <a name="what-is-azure-data-factory"></a>¿Qué es Azure Data Factory?

Data Factory es un servicio ETL de integración de datos basado en la nube y completamente administrado que automatiza el movimiento y la transformación de los datos. Al igual que las máquinas de una fábrica transforman las materias primas en productos acabados, Azure Data Factory organiza los servicios existentes que recopilan datos sin procesar y los transforman en información lista para utilizar. 

Con Azure Data Factory, puede crear flujos de trabajo controlados por datos para mover datos entre almacenes de datos locales y en la nube. Además, puede procesar y transformar los datos con los flujos de datos. ADF también admite motores de proceso externos para transformaciones codificadas a mano mediante servicios de proceso como Azure HDInsight, Azure Databricks y el entorno de ejecución de integración de SQL Server Integration Services (SSIS).

Con Data Factory, puede ejecutar el procesamiento de datos en un servicio en la nube basado en Azure o en su propio entorno de proceso autohospedado, como SSIS, SQL Server u Oracle. Después de crear una canalización que realice la acción que necesita, puede programarla para que se ejecute periódicamente (por ejemplo, cada hora, diaria o semanalmente), programar una ventana de tiempo o desencadenarla a partir de la aparición de un evento. Para más información, consulte [Introducción a Azure Data Factory](introduction.md).

## <a name="compliance-and-security-considerations"></a>Consideraciones sobre cumplimiento y seguridad

Azure Data Factory está certificado para una variedad de certificaciones de cumplimiento, como _SOC 1, 2, 3_, _HIPAA BAA_ y _HITRUST_. Puede encontrar una lista completa y creciente de certificaciones [aquí](data-movement-security-considerations.md). Se pueden encontrar copias digitales para informes de auditoría y certificaciones de cumplimiento en el [Centro de confianza del servicio](https://servicetrust.microsoft.com/).

### <a name="control-flows-and-scale"></a>Escala y flujos de control

Para admitir los diferentes patrones y flujos de integración en el almacenamiento de datos actual, Data Factory habilita un modelo de canalización de datos flexible. Esto conlleva paradigmas de programación de flujo de control total, que incluyen la ejecución condicional, la bifurcación en canalizaciones de datos y la capacidad de pasar explícitamente los parámetros dentro y entre estos flujos. El flujo de control también abarca la transformación de datos mediante la distribución de actividades a motores de ejecución externos y las funcionalidades de flujo de datos que incluyen el movimiento de datos a escala mediante la actividad de copia.

Data Factory proporciona la libertad de modelar cualquier estilo de flujo necesario para la integración de datos y que se puede enviar a petición o varias veces según una programación. Estos son algunos de los flujos habituales que permite este modelo:

- Flujos de control:
    - Las actividades se pueden encadenar juntas en una secuencia dentro de una canalización.
    - Las actividades se pueden bifurcar dentro de una canalización.
    - Parámetros:
        * Los parámetros se pueden definir en la canalización y los argumentos se pueden pasar al invocar la canalización a petición o desde un desencadenador.
        * Las actividades pueden consumir los argumentos que se pasan a la canalización.
    - Paso a estado personalizado:
        * Los resultados de la actividad, incluido el estado, se pueden usar en una posterior actividad de la canalización.
    - Contenedores de bucle:
        * La actividad foreach iterará en una colección especificada de actividades en un bucle. 
- Flujos basados en el desencadenador:
    - Las canalizaciones se pueden desencadenar a petición, por hora de reloj, o bien en respuesta a temas controlados por Event Grid.
- Flujos delta:
    - Los parámetros se pueden usar para definir el límite máximo de la copia delta al mover tablas de dimensiones o de referencia desde un almacén relacional de forma local o en la nube para cargar los datos en Data Lake.

Para más información, consulte el [Tutorial: Flujos de control](tutorial-control-flow.md).

### <a name="data-transformed-at-scale-with-code-free-pipelines"></a>Datos transformados a escala con canalizaciones sin código

La nueva experiencia de herramientas basadas en explorador proporciona la creación e implementación de canalizaciones sin código con una experiencia actual, interactiva y basada en la web.

Para los desarrolladores de datos visuales y los ingenieros de datos, la interfaz de usuario web de Data Factory es el entorno de desarrollo sin código que usará para crear canalizaciones. Está totalmente integrado con el repositorio de Git de Visual Studio Online y proporciona integración de CI/CD y desarrollo iterativo con opciones de depuración.

### <a name="rich-cross-platform-sdks-for-advanced-users"></a>Variados SDK multiplataforma para usuarios avanzados

Data Factory V2 proporciona un conjunto más completo de SDK que se pueden usar para crear, administrar y supervisar canalizaciones mediante su IDE favorito, como son:

* SDK de Python
* CLI de PowerShell
* SDK DE C#

Los usuarios también pueden usar las API REST documentadas para interactuar con Data Factory V2.

### <a name="iterative-development-and-debugging-by-using-visual-tools"></a>Desarrollo y depuración iterativos mediante herramientas visuales

Las herramientas visuales de Azure Data Factory le permiten realizar un desarrollo y depuración iterativos. Puede crear canalizaciones y realizar series de pruebas con la funcionalidad **Depurar** en el lienzo de la canalización sin escribir ni una sola línea de código. Puede ver los resultados de las series de pruebas en la ventana **Salida** del lienzo de la canalización. Después de realizar correctamente una serie de pruebas, puede agregar más actividades a la canalización y continuar con la depuración de manera iterativa. También puede cancelar las series de pruebas una vez que están en curso.

No es necesario que publique los cambios en el servicio Data Factory antes de seleccionar **Depurar**. Esto resulta útil en escenarios en los que se desea garantizar que los cambios o nuevas adiciones funcionarán según lo esperado antes de actualizar los flujos de trabajo de la factoría de datos en entornos de desarrollo, pruebas o producción.

### <a name="ability-to-deploy-ssis-packages-to-azure"></a>Capacidad de Implementación de paquetes SSIS en Azure

Si desea mover las cargas de trabajo de SSIS, puede crear una instancia de Data Factory y aprovisionar una instancia de Integration Runtime para la integración de SSIS en Azure. Un entorno de ejecución de integración de SSIS de Azure es un clúster totalmente administrado de máquinas virtuales de Azure (nodos) que se dedican a ejecutar los paquetes SSIS en la nube. Para obtener instrucciones paso a paso, vea el tutorial [Implementación de paquetes SSIS en Azure](./tutorial-deploy-ssis-packages-azure.md). 

### <a name="sdks"></a>SDK

Si es un usuario avanzado y busca una interfaz programática, Data Factory le proporciona un amplio conjunto de SDK que puede usar para crear, administrar o supervisar canalizaciones mediante su IDE favorito. Entre los lenguajes compatibles se incluye: .NET, PowerShell, Python y REST.

### <a name="monitoring"></a>Supervisión

Puede supervisar las factorías de datos mediante PowerShell, SDK o las herramientas de supervisión visual de la interfaz de usuario del explorador. Puede supervisar y administrar a petición flujos personalizados basados en desencadenadores y controlados por tiempo de una manera eficaz. Cancele tareas que ya existen, vea los errores de un solo vistazo, explore en profundidad para obtener mensajes de error detallados y depure los problemas, todo ello desde un único panel sin cambios de contexto y sin tener que desplazarse entre pantallas.

### <a name="new-features-for-ssis-in-data-factory"></a>Nuevas características de SSIS en Data Factory

Desde el lanzamiento inicial de la versión preliminar pública en 2017, Data Factory ha agregado las siguientes características para SSIS:

-    Compatibilidad con tres configuraciones o variantes más de Azure SQL Database para hospedar los paquetes o proyectos de la base de datos SSIS (SSISDB):
-    SQL Database con puntos de conexión de servicio de red virtual
-    Instancia administrada de SQL
-    Grupo elástico
-    Compatibilidad con una red virtual de Azure Resource Manager en una red virtual clásica que caerá en desuso en el futuro, lo que le permite insertar o unir su entorno de ejecución de integración de Azure-SSIS a una red virtual que esté configurada para SQL Database con acceso a puntos de conexión de servicio de red virtual, Instancias administradas o datos locales. Para más información, consulte también [Unión de una instancia de Integration Runtime de SSIS de Azure a una red virtual](join-azure-ssis-integration-runtime-virtual-network.md).
-    Compatibilidad con la autenticación de Azure Active Directory (Azure AD) y la autenticación de SQL para conectarse a SSISDB, lo que le permite usar la autenticación de Azure AD con su identidad administrada de Data Factory para los recursos de Azure
-    Compatibilidad para usar su propia licencia de SQL Server para conseguir ahorros sustanciales de costos con la opción Ventaja híbrida de Azure
-    Compatibilidad con la edición Enterprise del entorno de ejecución de integración de Azure-SSIS que le permite usar características avanzadas o premium, una interfaz de configuración personalizada para instalar componentes o extensiones adicionales y un ecosistema de terceros. Para más información, consulte también [Enterprise Edition, Custom Setup, and 3rd Party Extensibility for SSIS in ADF](https://blogs.msdn.microsoft.com/ssis/2018/04/27/enterprise-edition-custom-setup-and-3rd-party-extensibility-for-ssis-in-adf/) (Enterprise Edition, instalación personalizada y extensibilidad de terceros para SSIS en ADF). 
-    Una integración más profunda de SSIS en Data Factory que le permite invocar o desencadenar actividades de primera clase de ejecución de paquetes SSIS en canalizaciones de Data Factory y programarlas mediante SSMS. Para más información, consulte también [Modernize and extend your ETL/ELT workflows with SSIS activities in ADF pipelines](https://blogs.msdn.microsoft.com/ssis/2018/05/23/modernize-and-extend-your-etlelt-workflows-with-ssis-activities-in-adf-pipelines/) (Modernización y ampliación de los flujos de trabajo ETL/ETL con actividades de SSIS en las canalizaciones de ADF)

## <a name="what-is-the-integration-runtime"></a>¿Qué es el entorno de ejecución de integración?

El entorno de ejecución de integración es la infraestructura de proceso que Azure Data Factory usa para proporcionar las siguientes funcionalidades de integración de datos en diversos entornos de red:

- **Movimiento de datos**: Para el movimiento de datos, el entorno de ejecución de integración mueve los datos entre los almacenes de origen y de destino e incluye funcionalidad de conectores integrados, conversión de formato, asignación de columnas y transferencia de datos eficaz y escalable.
- **Distribución de actividades**: Para la transformación, permite ejecutar paquetes SSIS de forma nativa.
- **Ejecución de paquetes SSIS**: El entorno de ejecución de integración ejecuta de forma nativa paquetes SSIS en un entorno de proceso de Azure administrado. También permite distribuir y supervisar actividades de transformación que se ejecutan en una gran variedad de servicios de proceso, como Azure HDInsight, Azure Machine Learning, Azure SQL Database y SQL Server.

Puede implementar una o varias instancias del entorno de ejecución de integración, según sea necesario para mover y transformar los datos. El entorno de ejecución de integración se puede ejecutar en una red de Azure pública o en una red privada (local, Azure Virtual Network o en la nube privada virtual de Amazon Web Services [VPC]).

Para más información, consulte [Integration Runtime en Azure Data Factory](concepts-integration-runtime.md).

## <a name="what-is-the-limit-on-the-number-of-integration-runtimes"></a>¿Cuál es el límite del número de Integration Runtimes?

No hay ningún límite estricto acerca del número de instancias de Integration Runtime que puede tener en una factoría de datos. Sin embargo, existe un límite acerca del número de núcleos de máquina virtual que Integration Runtime puede usar por suscripción para la ejecución de paquetes SSIS. Para más información, consulte los [Límites de Data Factory](../azure-resource-manager/management/azure-subscription-service-limits.md#data-factory-limits).

## <a name="what-are-the-top-level-concepts-of-azure-data-factory"></a>¿Cuáles son los conceptos de nivel superior de Azure Data Factory?

Una suscripción de Azure puede tener una o varias instancias de Azure Data Factory (o factorías de datos). Azure Data Factory contiene cuatro componentes principales que funcionan juntos como plataforma en la que pueda crear flujos de trabajo orientados a datos con pasos para moverlos y transformarlos.

### <a name="pipelines"></a>Procesos

Una factoría de datos puede tener una o más canalizaciones. Una canalización es una agrupación lógica de actividades para realizar una unidad de trabajo. Juntas, las actividades de una canalización realizan una tarea. Por ejemplo, una canalización puede contener un grupo de actividades que ingiere datos de un blob de Azure y luego ejecuta una consulta de Hive en un clúster de HDInsight para particionar los datos. La ventaja es que puede usar una canalización para administrar las actividades como un conjunto en lugar de tener que administrar individualmente cada actividad. Puede encadenar juntas las actividades en una canalización para hacerlas funcionar de forma secuencial o puede hacerlas funcionar de forma independiente en paralelo.

### <a name="data-flows"></a>Flujos de datos

Los flujos de datos son objetos que se compilan visualmente en Data Factory que transforman los datos a escala de los servicios Spark de back-end. No es necesario saber programar ni conocer los elementos internos de Spark. Solo tiene que diseñar su intención de transformación de datos mediante gráficos (asignación) u hojas de cálculo (limpieza y transformación).

### <a name="activities"></a>Actividades

Las actividades representan un paso del procesamiento en una canalización. Por ejemplo, puede utilizar una actividad de Copia para copiar datos de un almacén de datos a otro. De igual forma, puede usar una actividad de Hive, que ejecuta una consulta de Hive en un clúster de Azure HDInsight para transformar o analizar los datos. Data Factory admite tres tipos de actividades: actividades de movimiento de datos, actividades de transformación de datos y actividades de control.

### <a name="datasets"></a>Conjuntos de datos

Los conjuntos de datos representan las estructuras de datos de los almacenes de datos que simplemente apuntan o hacen referencia a los datos que desea utilizar en sus actividades como entradas o salidas. 

### <a name="linked-services"></a>Servicios vinculados

Los servicios vinculados son muy similares a las cadenas de conexión que definen la información de conexión necesaria para que Data Factory se conecte a recursos externos. Considérelos de esta forma: un servicio vinculado define la conexión al origen de datos y un conjunto de datos representa la estructura de los datos. Por ejemplo, un servicio vinculado de Azure Storage especifica la cadena de conexión para conectarse a la cuenta de Azure Storage. Además, un conjunto de datos de Azure Blob especifica el contenedor de blobs y la carpeta que contiene los datos.

Los servicios vinculados tienen dos fines en Data Factory:

- Para representar un *almacén de datos* que incluye, entre otros, una instancia de SQL Server, una instancia de base de datos de Oracle, un recurso compartido de archivos o una cuenta de Azure Blob Storage. Para obtener una lista de almacenes de datos compatibles, consulte [Actividad de copia en Azure Data Factory](copy-activity-overview.md).
- Para representar un *recurso de proceso* que puede hospedar la ejecución de una actividad. Por ejemplo, la actividad HDInsight Hive se ejecuta en un clúster de Hadoop para HDInsight. Para ver una lista de actividades de transformación y de entornos de proceso admitidos, consulte el artículo [Transformar datos en Azure Data Factory](transform-data.md).

### <a name="triggers"></a>Desencadenadores

Los desencadenadores representan unidades de procesamiento que determinan cuándo se pone en marcha una ejecución de canalización. Existen diferentes tipos de desencadenadores para diferentes tipos de eventos. 

### <a name="pipeline-runs"></a>Ejecuciones de la canalización

Una ejecución de una canalización es una instancia de la ejecución de la canalización. Normalmente, crea instancias de una ejecución de canalización al pasar argumentos a los parámetros definidos en la canalización. Puede pasar los argumentos manualmente o dentro de la definición del desencadenador.

### <a name="parameters"></a>Parámetros

Los parámetros son pares clave-valor en una configuración de solo lectura. Defina parámetros en una canalización y pase los argumentos para los parámetros definidos durante la ejecución de un contexto de ejecución. El contexto de ejecución se crea mediante un desencadenador o desde una canalización que ejecuta manualmente. Las actividades dentro de la canalización consumen los valores de parámetro.

Un conjunto de datos es un parámetro fuertemente tipado y una entidad que puede reutilizar o a la que puede hacer referencia. Una actividad puede hacer referencia a conjuntos de datos y puede consumir las propiedades definidas en la definición del conjunto de datos.

Un servicio vinculado también es un parámetro fuertemente tipado que contiene información de conexión a un almacén de datos o a un entorno de proceso. También es una entidad que puede reutilizar o a la que puede hacer referencia.

### <a name="control-flows"></a>Flujos de control

Los flujos de control organizan actividades de canalización que incluyen el encadenamiento de actividades en una secuencia, la creación de ramas, los parámetros que define en el nivel de canalización y los argumentos que pasa al invocar la canalización a petición o desde un desencadenador. Los flujos de control incluyen además el paso a un estado personalizado y contenedores de bucle (es decir, los iteradores Para cada).


Para más información sobre los conceptos de Data Factory, consulte los siguientes artículos:

- [Dataset and linked services](concepts-datasets-linked-services.md) (Conjuntos de datos y servicios vinculados)
- [Canalizaciones y actividades](concepts-pipelines-activities.md)
- [Integration runtime](concepts-integration-runtime.md) (Tiempo de ejecución de integración)

## <a name="what-is-the-pricing-model-for-data-factory"></a>¿Qué es el modelo de precios de Data Factory?

Consulte los [detalles de precios de Azure Data Factory](https://azure.microsoft.com/pricing/details/data-factory/) para obtener información al respecto.

## <a name="how-can-i-stay-up-to-date-with-information-about-data-factory"></a>¿Cómo puedo mantenerme actualizado con información acerca de Data Factory?

Para ver la información más actualizada acerca de Azure Data Factory, vaya a los sitios siguientes:

- [Blog](https://azure.microsoft.com/blog/tag/azure-data-factory/)
- [Página principal de la documentación](./index.yml)
- [Página principal del producto](https://azure.microsoft.com/services/data-factory/)

## <a name="technical-deep-dive"></a>Inmersión técnica profunda

### <a name="how-can-i-schedule-a-pipeline"></a>¿Cómo puedo programar una canalización?

Puede usar el desencadenador de programador o el desencadenador de ventana de tiempo para programar una canalización. El desencadenador utiliza una programación del calendario del reloj, que puede programar canalizaciones periódicamente o mediante patrones periódicos basados en calendarios (por ejemplo, semanalmente los lunes a las 6:00 p.m. y los jueves a las 9:00 p.m.). Para obtener más información, consulte [Desencadenadores y ejecución de la canalización](concepts-pipeline-execution-triggers.md).

### <a name="can-i-pass-parameters-to-a-pipeline-run"></a>¿Puedo pasar parámetros a una ejecución de canalización?

Sí, los parámetros son conceptos de primera clase de nivel superior en Data Factory. Puede definir parámetros en el nivel de canalización y pasar argumentos al ejecutar la canalización a petición o mediante un desencadenador.  

### <a name="can-i-define-default-values-for-the-pipeline-parameters"></a>¿Puedo definir valores predeterminados para los parámetros de la canalización?

Sí. Puede definir valores predeterminados para los parámetros de las canalizaciones.

### <a name="can-an-activity-in-a-pipeline-consume-arguments-that-are-passed-to-a-pipeline-run"></a>¿Una actividad de una canalización puede consumir los argumentos que se pasan a una ejecución de canalización?

Sí. Cada actividad dentro de la canalización puede utilizar el valor del parámetro que se pasa a la canalización y que se ejecuta con la construcción de `@parameter`. 

### <a name="can-an-activity-output-property-be-consumed-in-another-activity"></a>¿Puede utilizarse una propiedad de salida de actividad en otra actividad?

Sí. Un resultado de la actividad se puede utilizar en una actividad posterior con la construcción de `@activity`.
 
### <a name="how-do-i-gracefully-handle-null-values-in-an-activity-output"></a>¿Cómo puedo controlar correctamente los valores NULL en una salida de actividad?

Puede usar la construcción de `@coalesce` en las expresiones para controlar correctamente los valores NULL.

## <a name="mapping-data-flows"></a>Asignación de flujos de datos

### <a name="i-need-help-troubleshooting-my-data-flow-logic-what-info-do-i-need-to-provide-to-get-help"></a>Necesito ayuda para solucionar mi lógica de flujo de datos. ¿Qué información tengo que proporcionar para obtener ayuda?

Cuando Microsoft facilita ayuda o soluciona problemas con los flujos de datos, proporcione el script de flujo de datos. Este es el script de código subyacente del gráfico de flujo de datos. En la interfaz de usuario de ADF, abra el flujo de datos y, a continuación, haga clic en el botón "Script" en la esquina superior derecha. Copie y pegue este script o guárdelo en un archivo de texto.

### <a name="how-do-i-access-data-by-using-the-other-90-dataset-types-in-data-factory"></a>¿Cómo accedo a los datos con los otros 90 tipos de conjunto de datos en Data Factory?

Actualmente, la característica de flujo de datos de asignación permite los archivos de texto delimitados, de Azure SQL Database y Azure Synapse Analytics desde Azure Blob Storage o Azure Data Lake Storage Gen2 y los archivos Parquet desde Blob Storage o Data Lake Storage Gen2 de forma nativa para el origen y el receptor. 

Utilice la actividad de copia para almacenar provisionalmente los datos desde cualquiera de los demás conectores y, a continuación, ejecutar una actividad de Data Flow para transformar los datos después de haberlos almacenado provisionalmente. Por ejemplo, la canalización se copiará en primer lugar al almacenamiento de blobs y, a continuación, una actividad de Data Flow utilizará un conjunto de datos del origen para transformar los datos.

### <a name="is-the-self-hosted-integration-runtime-available-for-data-flows"></a>¿Está disponible el entorno de ejecución de integración autohospedado para flujos de datos?

E entorno de ejecución de integración autohospedado es una construcción de canalización de ADF que se puede usar con la actividad de copia para adquirir o trasladar datos hacia y desde orígenes locales o receptores de datos basados en máquinas virtuales. Las máquinas virtuales que se usan para un IR autohospedado también se pueden colocar en la misma red virtual que los almacenes de datos protegidos para acceder a esos almacenes de datos desde ADF. Con los flujos de datos logrará estos mismos resultados finales pero mediante Azure IR con red virtual administrada en su lugar.

### <a name="does-the-data-flow-compute-engine-serve-multiple-tenants"></a>¿El motor de proceso de flujo de datos atiende a varios inquilinos?

Los clústeres nunca se comparten. Garantizamos el aislamiento de cada trabajo ejecutado en las ejecuciones en producción. En caso de los escenarios de depuración, una persona obtiene un clúster, lo inicia y todos los depuradores irán a él.

### <a name="is-there-a-way-to-write-attributes-in-cosmos-db-in-the-same-order-as-specified-in-the-sink-in-adf-data-flow"></a>¿Hay alguna manera de escribir atributos en cosmos DB en el mismo orden que se especifica en el receptor en el flujo de datos de ADF?    

Para cosmos DB, el formato subyacente de cada documento es un objeto JSON que es un conjunto desordenado de pares nombre-valor, por lo que el orden no se puede reservar. El flujo de datos activa un clúster incluso en el entorno de ejecución de integración con un aviso de flujo de datos de configuración del período de vida de 15 minutos sobre período de vida y costos. Esto soluciona los problemas de rendimiento recogidos en el documento de [rendimiento del flujo de datos](https://docs.microsoft.com/azure/data-factory/concepts-data-flow-performance#time-to-live).


###  <a name="why-an-user-is-unable-to-use-data-preview-in-the-data-flows"></a>¿Por qué un usuario no puede usar la vista previa de datos en los flujos de datos?   

Debe comprobar los permisos para el rol personalizado. Hay varias acciones implicadas en la vista previa de datos del flujo de datos. Empiece por comprobar el tráfico de red durante la depuración en el explorador. Siga todas las acciones; para más información, consulte la información sobre el [proveedor de recursos](https://docs.microsoft.com/azure/role-based-access-control/resource-provider-operations#microsoftdatafactory).

### <a name="does-the-data-flow-compute-engine-serve-multiple-tenants"></a>¿El motor de proceso de flujo de datos atiende a varios inquilinos?   

Este documento de solución de problemas puede ayudar a resolver el problema de [varios inquilinos](https://docs.microsoft.com/azure/data-factory/frequently-asked-questions#does-the-data-flow-compute-engine-serve-multiple-tenants).


###  <a name="in-adf-can-i-calculate-value-for-a-new-column-from-existing-column-from-mapping"></a>En ADF, ¿puedo calcular el valor de una nueva columna a partir de una columna existente de la asignación?  

Puede usar la transformación de derivación en el flujo de datos de asignación para crear una nueva columna en la lógica que desee. Para crear una columna derivada, puede generar una nueva columna o actualizar una existente. En el cuadro de texto Columna, especifique la columna que está creando. Para reemplazar una columna existente en el esquema, puede usar la lista desplegable de columnas. Para generar la expresión de la columna derivada, haga clic en el cuadro de texto Escribir expresión. Puede empezar a escribir la expresión o abrir el generador de expresiones para crear la lógica.

### <a name="why-mapping-data-flow-preview-failing-with-gateway-timeout"></a>¿Por qué la vista previa del flujo de datos de asignación da error con el tiempo de espera de la puerta de enlace? 

Pruebe a usar un clúster más grande y aprovechar los límites de fila de la configuración de depuración en un valor más pequeño para reducir el tamaño de la salida de depuración.

### <a name="how-to-parameterize-column-name-in-dataflow"></a>¿Cómo se puede parametrizar el nombre de columna en el flujo de datos?

El nombre de columna se puede parametrizar de forma similar a otras propiedades. Al igual que en la columna derivada, el cliente puede usar **$ColumnNameParam = toString(byName($myColumnNameParamInData)).** Estos parámetros se pueden pasar desde la ejecución de la canalización hacia los flujos de datos.



## <a name="wrangling-data-flow-data-flow-power-query"></a>Flujo de datos de limpieza y transformación (Power Query de flujo de datos)

### <a name="what-are-the-supported-regions-for-wrangling-data-flow"></a>¿Cuáles son las regiones admitidas para el flujo de datos de limpieza y transformación?

Data Factory está disponible en las [siguientes regiones](https://azure.microsoft.com/global-infrastructure/services/?products=data-factory).
La característica Power Query se está implantando en todas las regiones. Si la característica no está disponible en su región, póngase en contacto con el soporte técnico.

### <a name="what-are-the-limitations-and-constraints-with-wrangling-data-flow-"></a>¿Cuáles son las limitaciones y restricciones con el flujo de datos de limpieza y transformación?

Los nombres de los conjuntos de datos solo pueden contener caracteres alfanuméricos. Se admiten los siguientes almacenes de datos:

* Conjunto de datos DelimitedText en Azure Blob Storage mediante la autenticación de clave de cuenta
* Conjunto de datos DelimitedText de Azure Data Lake Storage gen2 con la clave de cuenta o autenticación de la entidad de servicio
* Conjunto de datos de DelimitedText en Azure Data Lake Storage gen1 mediante la autenticación de la entidad de servicio
* Azure SQL Database y Data Warehouse mediante la autenticación de SQL. Consulte los tipos de SQL admitidos a continuación. No hay compatibilidad con PolyBase o ensayo para el almacenamiento de datos.

En este momento, la integración de Key Vault de servicio vinculado no se admite en los flujos de datos de limpieza y transformación.

### <a name="what-is-the-difference-between-mapping-and-wrangling-data-flows"></a>¿Cuál es la diferencia entre los flujos de datos de asignación y de limpieza y transformación?

Los flujos de datos de asignación proporcionan una manera de transformar los datos a escala sin necesidad de programar. Puede diseñar un trabajo de transformación de datos en el lienzo de flujos de datos realizando una serie de transformaciones. Comience realice todas las transformaciones que desee en el origen y, después, continúe con los pasos de transformación de datos. Complete el flujo de datos con un receptor para enviar los resultados a un destino. El flujo de datos de asignación es excelente para asignar y transformar datos con esquemas conocidos y desconocidos en los receptores y orígenes.

Los flujos de datos de limpieza y transformación permite realizar tareas de preparación y exploración de datos ágiles mediante el editor de mashup de Power Query en línea a escala a través de la ejecución de Spark. Con el aumento de los lagos de datos, a veces solo necesita explorar un conjunto de datos o crear un conjunto de datos en el lago. No está asignando a un destino conocido. Los flujos de datos de limpieza y transformación se usan a menudo para escenarios de análisis basados en modelos menos formales.

### <a name="what-is-the-difference-between-power-platform-dataflows-and-wrangling-data-flows"></a>¿Cuál es la diferencia entre los flujos de datos de Power Platform y los flujos de datos de limpieza y transformación?

Los flujos de datos de Power Platform permiten a los usuarios importar y transformar datos de una amplia gama de orígenes de datos en Common Data Service y Azure Data Lake para crear aplicaciones de PowerApps, informes de Power BI o automatizaciones de flujo. Los flujos de datos de Power Platform usan las experiencias de preparación de datos de Power Query establecidas, similares a Power BI y Excel. Los flujos de entrada de Power Platform también permiten una reutilización sencilla dentro de una organización y controlan automáticamente la orquestación (por ejemplo, la actualización automática de los flujos de datos que dependen de otro flujo de entrada cuando se actualiza el primero).

Azure Data Factory (ADF) es un servicio de integración de datos administrado que permite a los ingenieros de datos y al integrador de datos de componentes para crear flujos de trabajo de extracción, transformación y carga de datos (ETL) híbridos complejos y de extracción, carga y transformación (ELT). El flujo de datos de limpieza y transformación en ADF permite a los usuarios un entorno sin servidor ni código que simplifica la preparación de datos en la nube y lo escala a cualquier tamaño de datos, sin la necesidad de administración de infraestructura. Usa la tecnología de preparación de datos de Power Query (también utilizada en flujos de datos de Power Platform, Excel y Power BI) para preparar y dar forma a los datos. Creados para tratar todas las complejidades y desafíos a escala de la integración de macrodatos, los flujos de datos de limpieza y transformación permiten a los usuarios transformar rápidamente los datos a gran escala mediante la ejecución de Spark. Los usuarios pueden crear canalizaciones de datos resistentes en un entorno visual accesible con nuestra interfaz basada en el explorador y permitir que ADF se haga cargo de las complejidades de la ejecución de Spark. Cree programaciones para sus canalizaciones y supervise las ejecuciones del flujo de datos desde el portal de supervisión de ADF. Administre fácilmente los Acuerdos de Nivel de Servicio de la disponibilidad de los datos con la supervisión y las alertas de disponibilidad enriquecidas de ADF, y aproveche las funcionalidades incorporadas de integración continua e implementación para guardar y administrar sus flujos en un entorno administrado. Establezca alertas y vea planes de ejecución para validar que su lógica tenga el rendimiento esperado mientras optimiza sus flujos de datos.

### <a name="supported-sql-types"></a>Tipos de SQL admitidos

El flujo de datos de limpieza y transformación admite los siguientes tipos de datos en SQL. Obtendrá un error de validación para usar un tipo de datos que no se admite.

* short
* double
* real
* FLOAT
* char
* NCHAR
* varchar
* NVARCHAR
* integer
* int
* bit
* boolean
* SMALLINT
* TINYINT
* bigint
* long
* text
* date
* datetime
* datetime2
* smalldatetime
* timestamp
* UNIQUEIDENTIFIER
* Xml


## <a name="next-steps"></a>Pasos siguientes

Para obtener instrucciones paso a paso para crear una factoría de datos, consulte los siguientes tutoriales:

- [Guía de inicio rápido: Crear una factoría de datos](quickstart-create-data-factory-dot-net.md)
- [Tutorial: Copia de datos en la nube](tutorial-copy-data-dot-net.md)
