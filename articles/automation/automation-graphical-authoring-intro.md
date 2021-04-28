---
title: Control de errores en runbooks gráficos de Azure Automation
description: En este artículo se indica cómo crear un runbook gráfico sin trabajar con código.
services: automation
ms.subservice: process-automation
ms.date: 03/16/2018
ms.topic: conceptual
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 88492d914b710c7a738dd6d7f501e22d490065b6
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107833821"
---
# <a name="author-graphical-runbooks-in-azure-automation"></a>Control de errores en runbooks gráficos de Azure Automation

Todos los runbooks de Azure Automation son flujos de trabajo de Windows PowerShell. Los runbooks gráficos y los runbooks gráficos de flujo de trabajo de PowerShell generan código de PowerShell que se ejecuta en los trabajos de Automation, pero que no se puede ver ni modificar. Puede convertir un runbook gráfico en un runbook gráfico de flujo de trabajo de PowerShell y viceversa. Sin embargo, no puede convertir estos runbooks en un runbook textual. Además, el editor gráfico de Automation no puede importar un runbook textual.

La creación gráfica permite crear runbooks para Azure Automation sin las complejidades del código subyacente de Windows PowerShell o del flujo de trabajo de esta herramienta. Puede agregar actividades al lienzo desde una biblioteca de cmdlets y runbooks, vincularlas y configurarlas para conformar un flujo de trabajo. Si alguna vez ha utilizado System Center Orchestrator o Service Management Automation (SMA), la creación gráfica le debería resultar familiar. Este artículo proporciona una introducción a los conceptos que necesita para comenzar a crear un runbook gráfico.

## <a name="overview-of-graphical-editor"></a>Información general del editor de gráficos

Puede abrir el editor de gráficos en el Portal de Azure a través de la creación o la edición de un Runbook gráfico.

![Área de trabajo gráfica](media/automation-graphical-authoring-intro/runbook-graphical-editor.png)

Las secciones siguientes describen los controles del editor de gráficos.

### <a name="canvas-control"></a>Control Lienzo

El control Lienzo permite diseñar el runbook. Puede agregar actividades desde los nodos del control Biblioteca al runbook y conectarlas con vínculos para definir la lógica del runbook. En la parte inferior del lienzo hay controles que permiten acercar y alejar.

### <a name="library-control"></a>Control de Biblioteca

El control Biblioteca permite seleccionar las [actividades](#use-activities) que se van a agregar al runbook. Se agregan al lienzo, donde puede conectarlas a otras actividades. El control Biblioteca incluye las secciones definidas en la tabla siguiente.

| Sección | Descripción |
|:--- |:--- |
| Cmdlets |Todos los cmdlets que se pueden usar en el runbook. Los cmdlets se organizan por módulo. Están disponibles todos los módulos que ha instalado en la cuenta de Automation. |
| Runbooks |Los runbooks de la cuenta de Automation. Estos runbooks se pueden agregar al lienzo para usarse como runbooks secundarios. Solo se muestran los runbooks del mismo tipo básico que el runbook que se está editando. En el caso de los runbooks gráficos, solo se muestran runbooks basados en PowerShell. En los runbooks gráficos de flujo de trabajo de PowerShell, solo se muestran los runbooks basados en el flujo de trabajo de PowerShell. |
| Recursos |Los [recursos de automatización](/previous-versions/azure/dn939988(v=azure.100)) de la cuenta de Automation que se pueden usar en el runbook. Cuando se agrega un recurso a un runbook, se agrega una actividad de flujo de trabajo que obtiene el recurso seleccionado. En el caso de los recursos de variables, puede seleccionar si agregar una actividad para obtener o seleccionar la variable. |
| Control de Runbook |Las actividades de control que se pueden usar en el runbook actual. Una actividad de unión toma varias entradas y espera hasta que todas se completan antes de continuar con el flujo de trabajo. Una actividad de código ejecuta una o varias líneas de código de PowerShell o de flujo de trabajo de PowerShell, en función del tipo de runbook gráfico. Puede usar esta actividad para ejecutar código personalizado o utilizar funcionalidades que sean difíciles de obtener con otras actividades. |

### <a name="configuration-control"></a>Control de Configuración

El control Configuración permite proporcionar detalles de un objeto seleccionado en el lienzo. Las propiedades disponibles en este control dependen del tipo de objeto seleccionado. Cuando selecciona una opción en el control Configuración, se abren hojas adicionales para ofrecer más información.

### <a name="test-control"></a>Control de Prueba

El control de Prueba no aparece cuando se inicia por primera vez el editor de gráficos. Se abre cuando se prueba un runbook gráfico de manera interactiva.

## <a name="use-activities"></a>Uso de actividades

Las actividades son los bloques de creación de un runbook. Una actividad puede ser un cmdlet de PowerShell, un runbook secundario o un flujo de trabajo. Puede agregar una actividad al runbook; para ello, haga clic en ella con el botón derecho en el control Biblioteca y seleccione **Agregar a lienzo**. Luego puede hacer clic en la actividad y arrastrarla para colocarla en cualquier lugar del lienzo que desee. La ubicación de la actividad en el lienzo no afecta de ningún modo a la operación del runbook. Puede distribuir el runbook de la forma que considere más adecuada para visualizar su funcionamiento.

![Agregar a lienzo](media/automation-graphical-authoring-intro/add-to-canvas-cmdlet.png)

Seleccione una actividad en el lienzo para configurar sus propiedades y parámetros en la hoja Configuración. Puede cambiar la etiqueta de la actividad a un nombre que sea descriptivo. El runbook sigue ejecutando el cmdlet original. Simplemente se cambia el nombre para mostrar, que usa el editor gráfico. Tenga en cuenta que la etiqueta debe ser única dentro del runbook.

### <a name="parameter-sets"></a>Conjuntos de parámetros

Un conjunto de parámetros define los parámetros obligatorios y opcionales que aceptan los valores de un cmdlet determinado. Todos los cmdlets tienen, al menos, un conjunto de parámetros y algunos tienen varios. Si un cmdlet tiene varios conjuntos de parámetros, debe seleccionar cuál va a usar para poder configurar los parámetros. Puede cambiar el conjunto de parámetros que una actividad usa mediante la selección de **Conjunto de parámetros** y luego de otro conjunto. En este caso, se pierden todos los valores de parámetro que ya había configurado.

En el ejemplo siguiente, el cmdlet [Get-AzVM](/powershell/module/az.compute/get-azvm) tiene tres conjuntos de parámetros. En el ejemplo se utiliza un conjunto denominado **ListVirtualMachineInResourceGroupParamSet**, con un único parámetro opcional, para devolver todas las máquinas virtuales de un grupo de recursos. En el ejemplo también se usa el conjunto de parámetros **GetVirtualMachineInResourceGroupParamSet** para especificar la máquina virtual que se va a devolver. Este conjunto tiene dos parámetros obligatorios y un parámetro opcional.

![Conjunto de parámetros](media/automation-graphical-authoring-intro/get-azvm-parameter-sets.png)

#### <a name="parameter-values"></a>Valores de parámetros

Cuando se especifica un valor para un parámetro, se selecciona un origen de datos para determinar cómo se especifica el valor. Los orígenes de datos disponibles para un parámetro concreto dependen de los valores válidos para dicho parámetro. Por ejemplo, Null no es una opción disponible para un parámetro que no permita valores nulos.

| Origen de datos | Descripción |
|:--- |:--- |
| Valor constante |Escriba un valor para el parámetro. Este origen de datos solo se encuentra disponible para los siguientes tipos de datos: Int32, Int64, String, Boolean, DateTime, Switch. |
| Salida de la actividad |Utilice la salida de una actividad que precede a la actividad actual en el flujo de trabajo. Se enumeran todas las actividades válidas. Como valor del parámetro, use solo la actividad que genera el resultado. Si la actividad genera un objeto con varias propiedades, puede escribir el nombre de una propiedad específica después de seleccionar la actividad. |
| Entrada de Runbook |Seleccione una entrada de runbook como entrada para el parámetro de actividad. |
| Activo de variable |Seleccione una variable de Automation como entrada. |
| Activo de credencial |Seleccione una credencial de Automation como entrada. |
| Activo de certificado |Seleccione un certificado de Automation como entrada. |
| Activo de conexión |Seleccione una conexión de Automation como entrada. |
| Expresión de PowerShell |Especifique una [expresión de PowerShell](#work-with-powershell-expressions) simple. La expresión se evalúa antes que la actividad y el resultado se usa como valor del parámetro. Puede usar variables para consultar la salida de una actividad o un parámetro de entrada de runbook. |
| No configurado |Borre cualquier valor configurado anteriormente. |

#### <a name="optional-additional-parameters"></a>Parámetros adicionales opcionales

Todos los cmdlets tienen la opción de proporcionar parámetros adicionales. Se trata de parámetros comunes de PowerShell u otros parámetros personalizados. El editor gráfico muestra un cuadro de texto en el que puede incluir parámetros con la sintaxis de PowerShell. Por ejemplo, para usar el parámetro común `Verbose`, debe especificar `-Verbose:$True`.

### <a name="retry-activity"></a>Vuelva a intentar la actividad

La funcionalidad de reintento permite que una actividad se ejecute varias veces hasta que se cumpla una condición determinada, de forma muy similar a un bucle. Esta característica se puede utilizar en las actividades que deben ejecutarse varias veces, que son propensas a errores y que pueden necesitar más de un intento para realizarse correctamente, o bien para probar que los datos de la información de salida son válidos.

Cuando se habilita el reintento de una actividad, puede establecer un retraso y una condición. El retraso es el tiempo (medido en segundos o minutos) que el runbook espera antes de volver a ejecutar la actividad. Si no se especifica un retraso, la actividad se ejecuta de nuevo inmediatamente después de completarse.

:::image type="content" source="media/automation-graphical-authoring-intro/retry-delay.png" alt-text="Captura de pantalla de la configuración de la característica Habilitar reintento.":::

La condición de reintento es una expresión de PowerShell que se evalúa después de cada vez que se ejecuta la actividad. Si la expresión se resuelve en True, la actividad se vuelve a ejecutar. Si la expresión se resuelve en False, la actividad no se vuelve a ejecutar y el runbook pasa a la actividad siguiente.

:::image type="content" source="media/automation-graphical-authoring-intro/retry-condition.png" alt-text="Captura de pantalla que muestra el campo Reintentar hasta que se cumpla esta condición y ejemplos de expresiones de PowerShell que se pueden usar en la condición de reintento.":::

La condición de reintento puede utilizar una variable denominada `RetryData` que proporciona acceso a información sobre los reintentos de actividad. Esta variable tiene las propiedades de la tabla siguiente:

| Propiedad | Descripción |
|:--- |:--- |
| `NumberOfAttempts` |Número de veces que se ha ejecutado la actividad. |
| `Output` |Salida de la última ejecución de la actividad. |
| `TotalDuration` |Tiempo transcurrido desde la primera vez que se inició la actividad. |
| `StartedAt` |Hora (en formato UTC) a la que se inició la actividad por primera vez. |

A continuación se muestran ejemplos de las condiciones de reintento de actividades.

```powershell-interactive
# Run the activity exactly 10 times.
$RetryData.NumberOfAttempts -ge 10
```

```powershell-interactive
# Run the activity repeatedly until it produces any output.
$RetryData.Output.Count -ge 1
```

```powershell-interactive
# Run the activity repeatedly until 2 minutes has elapsed.
$RetryData.TotalDuration.TotalMinutes -ge 2
```

Después de configurar una condición de reintento en una actividad, esta incluirá dos indicaciones visuales que sirven de recordatorio. Una se muestra en la actividad y la otra cuando se revisa la configuración de la actividad.

![Indicadores visuales de reintento de actividades](media/automation-graphical-authoring-intro/runbook-activity-retry-visual-cue.png)

### <a name="workflow-script-control"></a>Control de Script de flujo de trabajo

Un control de Script de flujo de trabajo es una actividad especial que acepta scripts de PowerShell o de flujo de trabajo de PowerShell, en función del tipo de runbook gráfico que se crea. Proporciona una funcionalidad que, de otro modo, podría no estar disponible. No puede aceptar parámetros, pero puede usar variables para los parámetros de entrada de runbook y de salida de actividad. Cualquier salida de la actividad se agrega al bus de datos. Las excepciones se generan sin vínculo saliente, en cuyo caso la salida se agrega a la salida del runbook.

Por ejemplo, el siguiente código realiza cálculos de fecha mediante una variable de entrada del runbook denominada `NumberOfDays`. Luego envía un valor DateTime calculado como salida para su uso por parte de las actividades subsiguientes del runbook.

```powershell-interactive
$DateTimeNow = (Get-Date).ToUniversalTime()
$DateTimeStart = ($DateTimeNow).AddDays(-$NumberOfDays)}
$DateTimeStart
```

## <a name="use-links-for-workflow"></a>Usar vínculos para el flujo de trabajo

Un vínculo en un runbook gráfico conecta dos actividades. Se muestra en el lienzo como una flecha que apunta desde la actividad de origen hasta la actividad de destino. Las actividades se ejecutan en la dirección de la flecha y la actividad de destino se inicia una vez que se completa la actividad de origen.

### <a name="create-a-link"></a>Creación de un vínculo

Puede crear un vínculo entre dos actividades; para ello, seleccione la actividad de origen y haga clic en el círculo que aparece en la parte inferior de la forma. Arrastre la flecha a la actividad de destino y suéltela.

![Creación de un vínculo](media/automation-graphical-authoring-intro/create-link-options.png)

Seleccione el vínculo para configurar sus propiedades en la hoja Configuración. Las propiedades incluyen el tipo de vínculo, que se describe en la tabla siguiente.

| Tipo de vínculo | Descripción |
|:--- |:--- |
| Canalización |La actividad de destino se ejecuta una vez para cada objeto generado desde la actividad de origen. La actividad de destino no se ejecuta si la actividad de origen no genera salida. La salida de la actividad de origen está disponible como objeto. |
| Secuencia |La actividad de destino se ejecuta una sola vez cuando recibe salida desde la actividad de origen. La salida de la actividad de origen está disponible como una matriz de objetos. |

### <a name="start-runbook-activity"></a>Actividad de iniciar runbook

Un runbook gráfico se inicia con cualquier actividad que no tenga un vínculo entrante. A menudo solo hay una actividad que actúa como actividad inicial del runbook. Si varias actividades no tienen un vínculo entrante, el runbook se inicia ejecutándolas en paralelo. Sigue los vínculos para ejecutar otras actividades a medida que finaliza cada una de ellas.

### <a name="specify-link-conditions"></a>Especificar condiciones de vínculo

Cuando especifica una condición en un vínculo, la actividad de destino solo se ejecuta si la condición se resuelve en True. Normalmente usa una variable `ActivityOutput` en una condición para recuperar la salida desde la actividad de origen.

En un vínculo de canalización, debe especificar una condición para un único objeto. El runbook evalúa la condición en cada objeto generado por la actividad de origen. A continuación, ejecuta la actividad de destino en cada objeto que cumple con la condición. Por ejemplo, con una actividad de origen de `Get-AzVM`, puede usar la siguiente sintaxis para que un vínculo de canalización condicional recupere solo las máquinas virtuales del grupo de recursos denominado Group1.

```powershell-interactive
$ActivityOutput['Get Azure VMs'].Name -match "Group1"
```

En el caso de un vínculo de secuencia, el runbook solo evalúa la condición una vez, debido a que se devuelve una única matriz que incluye todos los objetos de la actividad de origen. Por este motivo, el runbook no puede usar un vínculo de secuencia para el filtrado, como sucede con un vínculo de canalización. El vínculo de secuencia simplemente puede determinar si se ejecuta o no la siguiente actividad.

Por ejemplo, tomemos el siguiente conjunto de actividades de nuestro runbook **Iniciar VM**:

![Vínculo condicional con secuencias](media/automation-graphical-authoring-intro/runbook-conditional-links-sequence.png)

El runbook usa tres vínculos de secuencia diferentes que comprueban los valores de los parámetros de entrada `VMName` y `ResourceGroupName`, para determinar la acción adecuada que se debe llevar a cabo. Las posibles acciones consisten en iniciar una sola máquina virtual, iniciar todas las máquinas virtuales del grupo de recursos o iniciar todas las máquinas virtuales de una suscripción. En el vínculo de secuencia entre `Connect to Azure` y `Get single VM`, esta es la lógica de la condición:

```powershell-interactive
<#
Both VMName and ResourceGroupName runbook input parameters have values
#>
(
(($VMName -ne $null) -and ($VMName.Length -gt 0))
) -and (
(($ResourceGroupName -ne $null) -and ($ResourceGroupName.Length -gt 0))
)
```

Cuando usa un vínculo condicional, los datos disponibles desde la actividad de origen a otras actividades de dicha rama se filtran por la condición. Si una actividad es el origen de varios vínculos, los datos disponibles para las actividades de cada rama dependen de la condición del vínculo que se conecta a dicha rama.

Por ejemplo, la actividad `Start-AzVM` del siguiente runbook inicia todas las máquinas virtuales. Además, tiene dos vínculos condicionales. El primer vínculo condicional usa la expresión `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -eq $true` para filtrar si la actividad `Start-AzVM` se completa correctamente. El segundo vínculo condicional usa la expresión `$ActivityOutput['Start-AzVM'].IsSuccessStatusCode -ne $true` para filtrar si la actividad `Start-AzVm` no puede iniciar la máquina virtual.

![Ejemplo de vínculo condicional](media/automation-graphical-authoring-intro/runbook-conditional-links.png)

Una actividad que sigue el primer vínculo y usa la salida de la actividad de `Get-AzureVM` solo recupera las máquinas virtuales que estaban iniciadas en el momento en que se ejecutó `Get-AzureVM`. Todas las actividades que siguen el segundo vínculo solo obtienen las máquinas virtuales que estaban detenidas en el momento en que se ejecutó `Get-AzureVM`. Toda actividad que siga el tercer vínculo obtiene todas las máquinas virtuales, independientemente de cuál sea su estado de ejecución.

### <a name="use-junctions"></a>Usar uniones

Una unión es una actividad especial que espera hasta que se hayan completado todas las ramas entrantes. Esto permite que el runbook ejecute varias actividades en paralelo y garantiza que todas se hayan completado antes de continuar.

Si bien una unión puede tener una cantidad ilimitada de vínculos entrantes, solo uno de ellos puede ser una canalización. No se limita la cantidad de vínculos entrantes de secuencia. Puede crear la unión con varios vínculos entrantes de canalización y guardar el runbook, pero se producirá un error cuando se ejecute.

El ejemplo siguiente forma parte de un runbook que inicia un conjunto de máquinas virtuales, a la vez que descarga revisiones para aplicarlas a esas máquinas. Utiliza una unión para garantizar que ambos procesos se han completado antes de que el runbook continúe.

![unión](media/automation-graphical-authoring-intro/runbook-junction.png)

### <a name="work-with-cycles"></a>Trabajar con ciclos

Un ciclo se forma cuando una actividad de destino se vincula de vuelta a su actividad de origen o a otra actividad que, al final, se vincula a su origen. Actualmente, la creación gráfica no admite los ciclos. Si el runbook tiene un ciclo, lo guarda como corresponde, pero recibe un error cuando se ejecuta.

![Ciclo](media/automation-graphical-authoring-intro/runbook-cycle.png)

### <a name="share-data-between-activities"></a>Uso compartido de datos entre actividades

Todo dato generado por una actividad con un vínculo saliente se escribe en el bus de datos del runbook. Toda actividad del runbook puede usar datos del bus de datos para rellenar los valores de parámetros o para incluirlos en el código de script. Una actividad puede tener acceso a la salida de cualquier actividad anterior en el flujo de trabajo.

La manera en que los datos se escriben en el bus de datos depende del tipo de vínculo en la actividad. En el caso de un vínculo de canalización, los datos se generan como varios objetos. En un vínculo de secuencia, los datos se generan como una matriz. Si solo hay un valor, la salida es una matriz de un solo elemento.

El runbook tiene dos maneras de acceder a los datos del bus de datos: 
* Usar un origen de datos de salida de actividad.
* Usar un origen de datos de expresión de PowerShell.

El primer método usa un origen de datos de salida de actividad para rellenar un parámetro de otra actividad. Si la salida es un objeto, el runbook puede especificar una única propiedad.

![salida de la actividad](media/automation-graphical-authoring-intro/activity-output-datasource-revised20165.png)

El segundo mecanismo de acceso a los datos recupera la salida de una actividad en un origen de datos de expresión de PowerShell o de una actividad de script de flujo de trabajo con una variable `ActivityOutput`, por medio de la sintaxis aquí mostrada. Si la salida es un objeto, el runbook puede especificar una única propiedad.

```powershell-interactive
$ActivityOutput['Activity Label']
$ActivityOutput['Activity Label'].PropertyName
```

### <a name="use-checkpoints"></a>Usar puntos de comprobación

Puede establecer [puntos de control](automation-powershell-workflow.md#use-checkpoints-in-a-workflow) en un runbook gráfico de flujo de trabajo de PowerShell; para ello, seleccione **Runbook de punto de control** en cualquier actividad. Esto permite establecer un punto de control después de que se ejecuta la actividad.

![Punto de control](media/automation-graphical-authoring-intro/set-checkpoint.png)

Los puntos de control solo se habilitan en los runbooks gráficos de flujo de trabajo de PowerShell; no están disponibles en los runbooks gráficos. Si el runbook usa cmdlets de Azure, debe incluir una actividad `Connect-AzAccount` después de cualquier actividad con punto de control. La operación de conexión se usa en caso de que el runbook se suspenda y deba reiniciarse a partir de este punto de control en un trabajo diferente.

## <a name="handle-runbook-input"></a>Controlar entrada de runbook

Si el runbook actual se utiliza como secundario, requiere una entrada, ya sea de un usuario que inicia el runbook mediante Azure Portal o de otro runbook. Por ejemplo, si un runbook crea una máquina virtual, es posible que el usuario deba proporcionar información como el nombre de la máquina virtual y otras propiedades cada vez que se inicie el runbook.

El runbook acepta la entrada definiendo uno o más parámetros de entrada. El usuario proporciona valores para estos parámetros cada vez que se inicia el runbook. Cuando el usuario inicia el runbook mediante Azure Portal, se le solicita que proporcione valores para cada parámetro de entrada admitido por el runbook.

Al crear el runbook, puede acceder a sus parámetros de entrada haciendo clic en **Entrada y salida** en la barra de herramientas del runbook. Con esto se abre el control Entrada y salida, donde puede editar un parámetro de entrada existente o crear uno nuevo, mediante un clic en **Agregar entrada**.

![Agregar entrada](media/automation-graphical-authoring-intro/runbook-edit-add-input.png)

Las propiedades de la siguiente tabla definen los parámetros de entrada:

| Propiedad | Descripción |
|:--- |:--- |
| Nombre | Necesario. El nombre del parámetro. El nombre debe ser único en el runbook. Debe comenzar por una letra y solo puede contener letras, números y caracteres de subrayado. El nombre no puede contener ningún espacio. |
| Descripción |Opcional. Descripción del propósito del parámetro de entrada. |
| Tipo | Opcional. El tipo de datos que se espera para el valor del parámetro. Azure Portal proporciona un control adecuado para el tipo de datos de cada parámetro cuando se solicite una entrada. Los tipos de parámetro admitidos son String, Int32, Int64, Decimal, Boolean, DateTime y Object. Si no se selecciona ningún tipo de datos, el valor predeterminado es String.|
| Mandatory | Opcional. Esta propiedad especifica si se debe proporcionar un valor para el parámetro. Si elige `yes`, se debe proporcionar un valor cuando se inicia el runbook. Si elige `no`, no se necesita un valor cuando se inicia el runbook y se puede usar un valor predeterminado. No es posible iniciar el runbook si no se proporciona un valor para cada parámetro obligatorio que no tiene definido un valor predeterminado. |
| Valor predeterminado | Opcional. Valor utilizado para un parámetro si no se pasa uno cuando se inicia el runbook. Para establecer un valor predeterminado, elija `Custom`. Elija `None` si no desea proporcionar ningún valor predeterminado. |

## <a name="handle-runbook-output"></a>Controlar salida de runbook

La creación gráfica guarda los datos que crea cualquier actividad que no tiene un vínculo saliente en la [salida del runbook](./automation-runbook-output-and-messages.md). La salida se guarda con el trabajo del runbook y queda a disposición de un runbook principal cuando el runbook se usa como secundario.

## <a name="work-with-powershell-expressions"></a>Funcionamiento con expresiones de PowerShell

Una de las ventajas de la creación gráfica es que permite crear un runbook con un conocimiento mínimo de PowerShell. No obstante, actualmente es necesario conocer un poco PowerShell para rellenar determinados [valores de parámetro](#use-activities) y para configurar [condiciones de vínculo](#use-links-for-workflow). En esta sección se proporciona una introducción rápida a las expresiones de PowerShell. En [Scripting con Windows PowerShell](/powershell/scripting/overview)están disponibles todos los detalles sobre PowerShell.

### <a name="use-a-powershell-expression-as-a-data-source"></a>Usar una expresión de PowerShell como origen de datos

Puede usar una expresión de PowerShell como origen de datos para rellenar el valor de un [parámetro de actividad](#use-activities) con los resultados de código de PowerShell. La expresión puede ser una sola línea de código que realiza una función simple o varias líneas que desarrollan cierta lógica compleja. La salida de un comando que no está asignado a una variable se envía al valor del parámetro.

Por ejemplo, el siguiente comando genera la fecha actual.

```powershell-interactive
Get-Date
```

El fragmento de código siguiente crea una cadena a partir de la fecha actual y la asigna a una variable. El código envía el contenido de la variable a la salida.

```powershell-interactive
$string = "The current date is " + (Get-Date)
$string
```

Los siguientes comandos evalúan la fecha actual y devuelven una cadena que indica si el día actual cae en fin de semana o es un día entre semana.

```powershell-interactive
$date = Get-Date
if (($date.DayOfWeek = "Saturday") -or ($date.DayOfWeek = "Sunday")) { "Weekend" }
else { "Weekday" }
```

### <a name="use-activity-output"></a>Usar salida de actividad

Para usar la salida de una actividad anterior en el runbook, use la variable `ActivityOutput` con la siguiente sintaxis.

```powershell-interactive
$ActivityOutput['Activity Label'].PropertyName
```

Por ejemplo, puede tener una actividad con una propiedad que requiera el nombre de una máquina virtual. En este caso, el runbook puede usar la siguiente expresión.

```powershell-interactive
$ActivityOutput['Get-AzureVM'].Name
```

Si la propiedad requiere el objeto de máquina virtual en lugar de simplemente un nombre, el runbook devuelve todo el objeto con la sintaxis siguiente.

```powershell-interactive
$ActivityOutput['Get-AzureVM']
```

El runbook puede utilizar la salida de una actividad en una expresión más compleja, como la siguiente. Esta expresión concatena texto con el nombre de la máquina virtual.

```powershell-interactive
"The computer name is " + $ActivityOutput['Get-AzureVM'].Name
```

### <a name="compare-values"></a>Comparar valores

Utilice [operadores de comparación](/powershell/module/microsoft.powershell.core/about/about_comparison_operators) para comparar valores o determinar si un valor coincide con un patrón especificado. Una comparación devuelve un valor de True o False.

Por ejemplo, la siguiente condición determina si la máquina virtual de una actividad denominada `Get-AzureVM` está actualmente detenida.

```powershell-interactive
$ActivityOutput["Get-AzureVM"].PowerState -eq "Stopped"
```

La siguiente condición determina si la misma máquina virtual está en cualquier estado distinto de detenida.

```powershell-interactive
$ActivityOutput["Get-AzureVM"].PowerState -ne "Stopped"
```

Puede combinar varias condiciones en el runbook usando un [operador lógico](/powershell/module/microsoft.powershell.core/about/about_logical_operators), como `-and` u `-or`. Por ejemplo, la siguiente condición comprueba si la misma máquina virtual del ejemplo anterior se encuentra en un estado Detenido o Deteniendo.

```powershell-interactive
($ActivityOutput["Get-AzureVM"].PowerState -eq "Stopped") -or ($ActivityOutput["Get-AzureVM"].PowerState -eq "Stopping")
```

### <a name="use-hashtables"></a>Tablas hash

Las [tablas hash](/powershell/module/microsoft.powershell.core/about/about_hash_tables) son pares de nombre-valor útiles para devolver un conjunto de valores. También puede encontrar referencias a la tabla hash como diccionario. Las propiedades de ciertas actividades esperan una tabla hash en lugar de un valor simple.

Una tabla hash se crea con la siguiente sintaxis. Puede contener cualquier número de entradas, pero cada una se define mediante un nombre y un valor.

```powershell-interactive
@{ <name> = <value>; [<name> = <value> ] ...}
```

Por ejemplo, la siguiente expresión crea una tabla hash que se usará como origen de datos para un parámetro de actividad que espera una tabla hash de valores para una búsqueda en Internet.

```powershell-interactive
$query = "Azure Automation"
$count = 10
$h = @{'q'=$query; 'lr'='lang_ja';  'count'=$Count}
$h
```

En el ejemplo siguiente se usa la salida de una actividad llamada `Get Twitter Connection` para rellenar una tabla hash.

```powershell-interactive
@{'ApiKey'=$ActivityOutput['Get Twitter Connection'].ConsumerAPIKey;
    'ApiSecret'=$ActivityOutput['Get Twitter Connection'].ConsumerAPISecret;
    'AccessToken'=$ActivityOutput['Get Twitter Connection'].AccessToken;
    'AccessTokenSecret'=$ActivityOutput['Get Twitter Connection'].AccessTokenSecret}
```

## <a name="authenticate-to-azure-resources"></a>Autenticarse en los recursos de Azure

Los runbooks de Azure Automation que administran recursos de Azure tienen que autenticarse en Azure. La [cuenta de ejecución](./automation-security-overview.md), también denominada entidad de servicio, es el método predeterminado que utiliza un runbook de Automation para acceder a los recursos de Azure Resource Manager de la suscripción. Para agregar esta funcionalidad a un runbook gráfico, agregue al lienzo el recurso de conexión `AzureRunAsConnection`, que usa el cmdlet [Get-AutomationConnection](/system-center/sma/manage-global-assets) de PowerShell. También puede agregar el cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Este escenario se ilustra en el siguiente ejemplo.

![Actividades de autenticación de ejecución](media/automation-graphical-authoring-intro/authenticate-run-as-account.png)

La actividad `Get Run As Connection`, o `Get-AutomationConnection`, se configura con un origen de datos de valor constante denominado `AzureRunAsConnection`.

![Configuración de la conexión de ejecución](media/automation-graphical-authoring-intro/authenticate-runas-parameterset.png)

La siguiente actividad, `Connect-AzAccount`, agrega la cuenta de ejecución autenticada para que se pueda usar en el runbook.

![Conjunto de parámetros de Connect-AzAccount](media/automation-graphical-authoring-intro/authenticate-conn-to-azure-parameter-set.png)

>[!NOTE]
>En el caso de los runbooks de PowerShell, `Add-AzAccount` y `Add-AzureRMAccount` son alias de `Connect-AzAccount`. Tenga en cuenta que estos alias no están disponibles para los runbooks gráficos. Un runbook gráfico solo puede usar `Connect-AzAccount`.

En los campos de parámetro **APPLICATIONID**, **CERTIFICATETHUMBPRINT** y **TENANTID**, especifique el nombre de la propiedad en la ruta de campo, ya que la actividad genera un objeto con varias propiedades. De lo contrario, cuando se ejecute el runbook, se producirá un error al intentar realizar la autenticación. Estos son los requisitos mínimos para autenticar el runbook con la cuenta de ejecución.

Algunos suscriptores crean una cuenta de Automation mediante una [cuenta de usuario de Azure AD](./shared-resources/credentials.md) para administrar la implementación clásica de Azure o los recursos de Azure Resource Manager. Para mantener la compatibilidad con versiones anteriores de estos suscriptores, el mecanismo de autenticación que se debe usar en el runbook es el cmdlet `Add-AzureAccount` con un [recurso de credencial](./shared-resources/credentials.md). El recurso representa a un usuario de Active Directory con acceso a la cuenta de Azure.

Puede habilitar esta funcionalidad para el runbook gráfico si agrega un recurso de credencial al lienzo, seguido de una actividad `Add-AzureAccount` que use el recurso de credencial para su entrada. Consulte el ejemplo siguiente.

![Actividades de autenticación](media/automation-graphical-authoring-intro/authentication-activities.png)

El runbook debe autenticarse en su inicio y después de cada punto de control. Por lo tanto, debe usar una actividad `Add-AzureAccount` después de cualquier actividad `Checkpoint-Workflow`. No es necesario usar una actividad de credencial adicional.

![Salida de la actividad](media/automation-graphical-authoring-intro/authentication-activity-output.png)

## <a name="export-a-graphical-runbook"></a>Exportar un runbook gráfico

Solo puede exportar la versión publicada de un runbook gráfico. Si el runbook todavía no se ha publicado, el botón **Exportar** está deshabilitado. Al hacer clic en el botón **Exportar**, el runbook se descarga en el equipo local. El nombre del archivo coincide con el nombre del runbook, seguido de una extensión **.graphrunbook**.

## <a name="import-a-graphical-runbook"></a>Importar un runbook gráfico

Puede importar un archivo de runbook gráfico o de runbook gráfico de flujo de trabajo de PowerShell seleccionando la opción **Importar** al agregar un runbook. Al seleccionar el archivo para importar, puede mantener el mismo nombre o indicar otro nuevo. El campo **Tipo de Runbook** muestra el tipo de runbook después de evaluar el archivo seleccionado. Si intenta seleccionar un tipo diferente que no sea correcto, el editor gráfico mostrará un mensaje para indicar que hay posibles conflictos y que puede haber errores de sintaxis durante la conversión.

![Importar runbook](media/automation-graphical-authoring-intro/runbook-import.png)

## <a name="test-a-graphical-runbook"></a>Ejecutar un runbook gráfico

Cada runbook gráfico de Azure Automation tiene una versión de borrador y una versión publicada. Solo puede ejecutar la versión publicada y solo puede editar la versión de borrador. Los cambios realizados en la versión de borrador no afectan la versión publicada. Cuando la versión borrador está lista para usar, se publica, lo que sobrescribe la versión publicada actual.

Puede probar la versión de borrador de un runbook en Azure Portal, sin modificar la versión publicada. También puede probar un nuevo runbook antes de publicarlo, para comprobar que funciona correctamente antes de cualquier reemplazo de versión. La prueba de un runbook ejecuta la versión de borrador y garantiza que se completan todas las acciones que realiza. No se crea ningún historial de trabajos, pero el panel de salida de la prueba muestra la salida.

Para abrir el control Prueba del runbook gráfico, abra el runbook para editarlo y, a continuación, haga clic en el botón **Panel de prueba**. El control Prueba solicita los parámetros de entrada; puede iniciar el runbook haciendo clic en **Iniciar**.

## <a name="publish-a-graphical-runbook"></a>Publicar un runbook gráfico

Para publicar un runbook gráfico, abra el runbook para editarlo y, a continuación, haga clic en **Publicar**. A continuación se indican los posibles estados del runbook:

* Nuevo: el runbook aún no se ha publicado. 
* Publicado: el runbook se ha publicado.
* En edición: el runbook se ha editado después de haberlo publicado y la versión de borrador y la versión publicada son distintas.

![Estados de runbooks](media/automation-graphical-authoring-intro/runbook-statuses-revised20165.png)

Tiene la opción de revertir a la versión publicada de un runbook. Esta operación anula cualquier cambio realizado desde la última vez que se publicó el runbook. Reemplaza la versión de borrador del runbook con la versión publicada.

## <a name="next-steps"></a>Pasos siguientes

* Para empezar a trabajar con runbooks de PowerShell, vea el [Tutorial: Crear un runbook gráfico](learn/automation-tutorial-runbook-graphical.md).
* Para más información sobre los tipos de runbook, sus ventajas y sus limitaciones, consulte [Tipos de runbooks de Azure Automation](automation-runbook-types.md).
* Para comprender cómo se realiza la autenticación con la cuenta de ejecución de Automation, consulte [Cuentas de ejecución](automation-security-overview.md#run-as-account).
* Para ver una referencia de los cmdlets de PowerShell, consulte [Az.Automation](/powershell/module/az.automation/#automation).
