---
title: 'Aplicación SaaS: Supervisión del rendimiento de muchas bases de datos'
description: Supervisión y administración del rendimiento de Azure SQL Database en una aplicación SaaS multiinquilino
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 01/25/2019
ms.openlocfilehash: 8f60e2d075a3911183ebc9b9423bd51c0aca7960
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699286"
---
# <a name="monitor-and-manage-performance-of-azure-sql-database-in-a-multi-tenant-saas-app"></a>Supervisión y administración del rendimiento de Azure SQL Database en una aplicación SaaS multiinquilino
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial se describen varios escenarios clave de administración de rendimiento que se usan en aplicaciones de SaaS. Mediante un generador de carga para simular la actividad en todas las bases de datos de inquilino, se demuestran las características integradas de supervisión y alertas de SQL Database y los grupos elásticos.

La aplicación Wingtip Tickets SaaS Database Per Tenant utiliza un modelo de datos de inquilino único, donde cada lugar (inquilino) tiene su propia base de datos. Al igual que en muchas aplicaciones SaaS, el patrón de carga de trabajo de inquilino previsto es imprevisible y esporádico. En otras palabras, las ventas de entradas pueden producirse en cualquier momento. Para aprovechar las ventajas de este patrón de uso típico de base de datos, las bases de datos de inquilinos se implementan en grupos elásticos. Los grupos elásticos optimizan el costo de las soluciones mediante el uso compartido de los recursos entre distintas bases de datos. Con este tipo de patrón, es importante supervisar la base de datos y el uso de los recursos del grupo para asegurarse de que las cargas están razonablemente repartidas entre los grupos. También debe asegurarse de que las bases de datos individuales tienen los recursos suficientes y de que los grupos no están al límite de [unidades de transacción de bases de datos elásticas](purchasing-models.md#dtu-based-purchasing-model). En este tutorial se exploran métodos para supervisar y administrar las bases de datos y los grupos, y se explica cómo tomar medidas correctivas en respuesta a las variaciones en la carga de trabajo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> 
> * Simular el uso de las bases de datos de inquilinos mediante la ejecución de un generador de carga proporcionado
> * Supervisar las bases de datos de inquilinos a medida que responden al aumento de carga
> * Escalar verticalmente el grupo elástico en respuesta al aumento de la carga en la base de datos
> * Aprovisionar un segundo grupo elástico para equilibrar la actividad de la base de datos


Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* Aplicación Wingtip Tickets SaaS Database Per Tenant implementada. Para implementarla en menos de cinco minutos, consulte el artículo de [implementación y exploración de la aplicación Wingtip Tickets SaaS Database Per Tenant](./saas-dbpertenant-get-started-deploy.md).
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)

## <a name="introduction-to-saas-performance-management-patterns"></a>Introducción a los patrones de administración del rendimiento de SaaS

La administración del rendimiento de la base de datos consiste en compilar y analizar los datos de rendimiento y, a continuación, reaccionar a estos datos mediante el ajuste de parámetros para mantener un tiempo de respuesta aceptable de la aplicación. Cuando se hospedan a varios inquilinos, los grupos elásticos son una manera rentable de proporcionar y administrar los recursos para un grupo de bases de datos con cargas de trabajo impredecibles. Con ciertos patrones de carga de trabajo, se pueden beneficiar de la administración en grupo un mínimo de dos bases de datos S3.

![diagrama de aplicaciones](./media/saas-dbpertenant-performance-monitoring/app-diagram.png)

Se deben supervisar los grupos y las bases de datos de estos para garantizar que permanecen dentro de intervalos de rendimiento aceptables. Ajuste la configuración del grupo para satisfacer las necesidades de la carga de trabajo agregada de todas las bases de datos, garantizando que las eDTU del grupo son adecuadas para la carga de trabajo global. Ajuste los valores mínimo y máximo de unidades de transacción de bases de datos elásticas por base de datos adecuados para los requisitos de su aplicación.

### <a name="performance-management-strategies"></a>Estrategias de administración del rendimiento

* Para evitar tener que supervisar el rendimiento manualmente, resulta más eficaz **establecer alertas que se activan cuando las bases de datos o los grupos se alejan de los intervalos normales**.
* Para responder a las fluctuaciones a corto plazo en el tamaño de proceso de un grupo, **se puede escalar vertical u horizontalmente el nivel de eDTU del grupo**. Si esta fluctuación es habitual o previsible, **se puede programar el escalado automático del grupo**. Por ejemplo, cuando haya poca carga de trabajo (por la noche o durante el fin de semana), redúzcalo verticalmente.
* Para responder a las fluctuaciones a largo plazo o a cambios en el número de bases de datos, **las bases de datos se pueden mover a otros grupos**.
* Para responder a aumentos breves en la carga *individual* de una base de datos, **se puede sacar de un grupo para asignarle un tamaño de proceso individual**. Una vez reducida la carga, la base de datos se puede devolver al grupo. Cuando se sepa de antemano, las bases de datos se pueden mover con antelación para garantizar que siempre tienen los recursos necesarios y evitar que otras bases de datos del grupo resulten afectadas. Si es predecible, como para un establecimiento, una avalancha de ventas de entradas para un evento popular, este comportamiento de administración puede integrarse en la aplicación.

[Azure Portal](https://portal.azure.com) proporciona supervisión y alertas integradas en la mayoría de recursos. La supervisión y las alertas están disponibles para las bases de datos y los grupos. La supervisión y las alertas integradas son específicas de los recursos, por lo que es conveniente usarlas con un número reducido de recursos, pero no cuando se trabaja con muchos recursos.

Para escenarios de gran volumen, donde se trabaja con muchos recursos, se puede usar los [registros de Azure Monitor](./saas-dbpertenant-log-analytics.md). Se trata de un servicio de Azure independiente que proporciona análisis de los registros recopilados en un área de trabajo de Log Analytics. Los registros de Azure Monitor pueden recopilar la telemetría de muchos servicios y sirve para consultar y establecer alertas.

## <a name="get-the-wingtip-tickets-saas-database-per-tenant-application-scripts"></a>Obtención de los scripts de la aplicación Wingtip Tickets SaaS Database Per Tenant

Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant). Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.

## <a name="provision-additional-tenants"></a>Aprovisionamiento de inquilinos adicionales

Aunque los grupos pueden ser rentables con solo dos bases de datos S3, cuantas más bases de datos en el grupo, más rentable será la media. Para una buena comprensión de cómo funciona la administración y la supervisión del rendimiento a escala, para este tutorial se necesitan al menos 20 bases de datos implementadas.

Si ya aprovisionó un lote de inquilinos en un tutorial anterior, vaya directamente a la sección [Simulación de uso en todas las bases de datos de inquilinos](#simulate-usage-on-all-tenant-databases).

1. En **PowerShell ISE**, abra \\Learning Modules\\Performance Monitoring and Management\\*Demo-PerformanceMonitoringAndManagement.ps1*. Mantenga este script abierta, ya que se van a ejecutar varios escenarios en este tutorial.
1. Establezca **$DemoScenario** = **1**, **Aprovisionamiento de un lote de inquilinos**
1. Presione **F5** para ejecutar el script.

El script implementará 17 inquilinos en menos de cinco minutos.

El script *New-TenantBatch* usa un conjunto anidado o vinculado de plantillas de [Resource Manager](../../azure-resource-manager/index.yml) que crean un lote de inquilinos, que, de forma predeterminada, copia la base de datos **basetenantdb** en el servidor de catálogo para crear las bases de datos de inquilinos y las registra en el catálogo; finalmente, las inicializa con el nombre de inquilino y el tipo de lugar. Esto es coherente con la manera en que la aplicación aprovisiona un nuevo inquilino. Los cambios realizados en *basetenantdb* se aplican a los nuevos inquilinos que se aprovisionen a partir de ese momento. Vea el [tutorial de administración de esquemas](saas-tenancy-schema-management.md) para ver cómo realizar cambios de esquema en bases de datos de inquilinos *existentes* (incluida la base de datos *basetenantdb*).

## <a name="simulate-usage-on-all-tenant-databases"></a>Simulación de uso en todas las bases de datos de inquilinos

Se proporciona el script *Demo-PerformanceMonitoringAndManagement.ps1* que simula una carga de trabajo que se ejecuta en todas las bases de datos de los inquilinos. La carga se genera mediante uno de los escenarios de carga disponibles:

| Demostración | Escenario |
|:--|:--|
| 2 | Generación de una carga de intensidad normal (aprox. 40 DTU) |
| 3 | Generación de una carga con ráfagas más prolongadas y frecuentes por base de datos|
| 4 | Generación de una carga con mayores ráfagas de DTU por base de datos (aprox. 80 DTU)|
| 5 | Generación de una carga normal y una carga elevada en un solo inquilino (aprox. 95 DTU)|
| 6 | Generación de una carga desequilibrada en varios grupos|

El generador de carga se aplica una carga *sintética* exclusiva de CPU para cada base de datos de inquilinos. El generador de inicia un trabajo para cada base de datos de inquilinos, que llama a un procedimiento almacenado periódicamente que genera la carga. Los niveles de carga (en eDTU), la duración y los intervalos varían en todas las bases de datos, lo cual simula una actividad de inquilinos imprevisible.

1. En **PowerShell ISE**, abra \\Learning Modules\\Performance Monitoring and Management\\*Demo-PerformanceMonitoringAndManagement.ps1*. Mantenga este script abierta, ya que se van a ejecutar varios escenarios en este tutorial.
1. Establezca **$DemoScenario** = **2**, *Generación de una carga de intensidad normal*.
1. Presione **F5** para aplicar una carga de todas las bases de datos de inquilinos.

Wingtip Tickets SaaS Database Per Tenant es una aplicación SaaS y, en el mundo real, la carga que tiene una aplicación SaaS habitualmente es esporádica e impredecible. Para realizar la simulación, el generador de carga crea una carga aleatoria que distribuye entre todos los inquilinos. El patrón de carga tarda varios minutos en crearse, por lo que se recomienda dejar que el generador de carga se ejecute durante unos 3-5 minutos antes de intentar supervisar la carga en las secciones siguientes.

> [!IMPORTANT]
> El generador de carga se ejecuta como una serie de trabajos en la sesión local de PowerShell. Mantenga abierta la pestaña *Demo-PerformanceMonitoringAndManagement.ps1*. Si la cierra o suspende la máquina, el generador de carga se detendrá. El generador de carga permanece en un estado *de invocación de trabajo* en el que genera carga en los nuevos inquilinos que se aprovisionaron después de iniciar el generador. Presione *Ctrl-C* para detener la invocación de nuevos trabajos y salir del script. El generador de carga continuará ejecutándose, pero solo en los inquilinos existentes.

## <a name="monitor-resource-usage-using-the-azure-portal"></a>Supervisión del uso de los recursos mediante Azure Portal

Para supervisar el uso de los recursos derivado de la carga que se aplica, abra el portal por el grupo que contiene las bases de datos de inquilinos:

1. Abra [Azure Portal](https://portal.azure.com) y vaya al servidor *tenants1-dpt-&lt;USUARIO&gt;* .
1. Desplácese hacia abajo, busque los grupos elásticos y haga clic en **Pool1**. Este grupo contiene todas las bases de datos de inquilinos creadas hasta ahora.

Observe los gráficos **Supervisión de grupo elástico** y **Supervisión de base de datos elástica**.

El uso de los recursos del grupo es la suma del uso de todas las bases de datos del grupo. El gráfico de base de datos muestra las cinco bases de datos más recientes:

![gráfico de base de datos](./media/saas-dbpertenant-performance-monitoring/pool1.png)

Como hay otras bases de datos en el grupo aparte de las cinco principales, el uso del grupo muestra la actividad que no se refleja en el gráfico de las cinco bases de datos principales. Para ver detalles adicionales, haga clic en **Uso de recursos de base de datos**:

![utilización de recursos de base de datos](./media/saas-dbpertenant-performance-monitoring/database-utilization.png)


## <a name="set-performance-alerts-on-the-pool"></a>Establecimiento de alertas de rendimiento en el grupo

Establezca una alerta en el grupo que se desencadene cuando el uso sea de \>75 % como sigue:

1. Abra *Pool1* (en el servidor *tenants1-dpt-\<user\>* ) en [Azure Portal](https://portal.azure.com).
1. Haga clic en **Reglas de alerta** y en **+ Agregar alerta**:

   ![agregar alerta](./media/saas-dbpertenant-performance-monitoring/add-alert.png)

1. Proporcione un nombre, como **High DTU**,
1. Establezca los valores siguientes:
   * **Métrica = eDTU percentage**
   * **Condición = greater than**
   * **Umbral = 75**
   * **Período = Over the last 30 minutes**
1. Agregue una dirección de correo al cuadro *Correos electrónicos adicionales del administrador* y haga clic en **Aceptar**.

   ![Establecer alerta](./media/saas-dbpertenant-performance-monitoring/alert-rule.png)


## <a name="scale-up-a-busy-pool"></a>Escalado vertical de un grupo ocupado

Si aumenta el nivel de carga global en un grupo hasta el punto de llegar al máximo del grupo y alcanza el 100 % del uso de eDTU, el rendimiento de las bases de datos se verá afectado, lo cual podría ralentizar los tiempos de respuesta de consulta para todas las bases de datos del grupo.

**A corto plazo**, considere la posibilidad de escalar verticalmente el grupo para proporcionar recursos adicionales o de quitar bases de datos del grupo (moverlas a otros grupos o fuera del grupo a un nivel de servicio independiente).

**A largo plazo**, considere la posibilidad de optimizar las consultas o el uso de índices para mejorar el rendimiento de las bases de datos. En función de la sensibilidad de la aplicación a los problemas de rendimiento, es un procedimiento recomendado el escalado vertical de un grupo antes de que llegue al 100 % del uso de eDTU. Use una alerta para que le avise con antelación.

Puede simular un grupo ocupado si aumenta la carga que produce el generador. Hacer que las bases de datos generen ráfagas con más frecuencia y más prolongadas aumenta la carga global del grupo sin cambiar los requisitos de las bases de datos individualmente. El escalado vertical del grupo se realiza fácilmente en el portal o desde PowerShell. En este ejercicio se usa el portal.

1. Establezca *$DemoScenario* = **3**, _Generación de una carga con ráfagas más prolongadas y frecuentes por base de datos_ para aumentar la intensidad de la carga global del grupo sin cambiar la carga máxima necesaria para cada base de datos.
1. Presione **F5** para aplicar una carga de todas las bases de datos de inquilinos.

1. Vaya a **Pool1** en Azure Portal.

Supervise el aumento del uso de eDTU del grupo en el gráfico superior. La carga mayor tarda unos minutos en entrar en vigor, pero debería ver cómo rápidamente el grupo empieza a llegar al máximo uso y, según la carga se va estabilizando en el nuevo modelo, sobrecarga rápidamente el grupo.

1. Para escalar verticalmente el grupo, haga clic en **Configurar grupo** en la parte superior de la página **Pool1**.
1. Ajuste el valor **eDTU del grupo** a **100**. Cambiar la eDTU del grupo no cambia la configuración por base de datos (que sigue siendo 50 eDTU máx. por base de datos). Puede ver la configuración por base de datos en el lado derecho de la página **Configurar grupo**.
1. Haga clic en **Guardar** para enviar la solicitud de escala del grupo.

Vuelva a **Pool1** > **Introducción** para ver los gráficos de supervisión. Supervise el efecto de proporcionar más recursos al grupo (aunque con pocas bases de datos y una carga aleatoria no siempre es fácil verlo bien hasta que se ejecuta durante un tiempo). Mientras observe los gráficos, tenga en cuenta que el 100 % del gráfico superior representa ahora 100 eDTU, mientras que en el gráfico inferior, el 100 % sigue siendo 50 eDTU, ya que el máximo por base de datos sigue siendo 50 eDTU.

Las bases de datos permanecen en línea y están totalmente disponibles durante el proceso. En el último momento, como las bases de datos están preparadas para la habilitación con la nueva eDTU de grupo, todas las conexiones activas se interrumpen. El código de la aplicación debe escribirse para siempre reintente recuperar las conexiones interrumpidas, por lo tanto, se volverá a conectar a la base de datos del grupo escalado verticalmente.

## <a name="load-balance-between-pools"></a>Equilibrio de carga entre grupos

Como alternativa al escalado vertical del grupo, cree un segundo grupo y mueva bases de datos a este para equilibrar la carga entre los dos grupos. Para ello, el nuevo grupo debe crearse en el mismo servidor que el primero.

1. En [Azure Portal](https://portal.azure.com), abra el servidor **tenants1-dpt-&lt;USUARIO&gt;** .
1. Haga clic en **+ Grupo nuevo** para crear un grupo en el servidor actual.
1. En la plantilla **Grupo elásticos**:

   1. Establezca **Nombre** en *Pool2*.
   1. Deje el plan de tarifa como **Grupo Estándar**.
   1. Haga clic en **Configurar grupo**.
   1. Establezca **eDTU del grupo** en *50 eDTU*.
   1. Haga clic en **Agregar bases de datos** para ver una lista de bases de datos en el servidor que se pueden agregar a *Pool2*.
   1. Seleccione 10 bases de datos cualquiera para moverlas al nuevo grupo y, después, haga clic en **Seleccionar**. Si ha estado ejecutando el generador de carga, el servicio ya sabe que el perfil de rendimiento requiere un grupo mayor que el tamaño predeterminado de 50 eDTU y recomienda empezar con una configuración de 100 eDTU.

      ![recomendación](./media/saas-dbpertenant-performance-monitoring/configure-pool.png)

   1. Para este tutorial, deje el valor predeterminado en 50 eDTU y haga clic en **Seleccionar** de nuevo.
   1. Haga clic en **Aceptar** para crear el grupo y para mover las bases de datos seleccionadas a este.

Crear el grupo y mover las bases de datos tarda unos minutos. Al mover las bases de datos, permanecen en línea y totalmente accesibles hasta el último momento, cuando se cierran todas las conexiones abiertas. Siempre y cuando tenga alguna lógica de reintento, los clientes se conectarán a la base de datos en el nuevo grupo.

Vaya a **Pool2** (en el servidor *tenants1-dpt-\<user\>* ) para abrir el grupo y supervisar su rendimiento. Si no lo ve, espere a que finalice el aprovisionamiento del nuevo grupo.

Ahora verá que el uso de los recursos en *Pool1* ha disminuido y que *Pool2* tiene una carga similar.

## <a name="manage-performance-of-an-individual-database"></a>Administración del rendimiento de una base de datos individual

Si una base de datos individual de un grupo experimenta una carga elevada durante un tiempo prolongado, según la configuración de grupo, puede tender a dominar los recursos del grupo, lo cual puede afectar a otras bases de datos. Si es probable que la actividad continúe durante un tiempo, la base de datos se puede sacar temporalmente del grupo. Esto permite que la base de datos tenga los recursos adicionales necesarios y la aísla de las otras bases de datos.

En este ejercicio se simula el efecto de una carga elevada en Contoso Concert Hall cuando se venden entradas para un concierto popular.

1. En **PowerShell ISE**, abra el script \\*Demo-PerformanceMonitoringAndManagement.ps1*.
1. Establezca **$DemoScenario = 5, generar una carga normal y una carga elevada en un solo inquilino (aprox. 95 DTU).**
1. Establezca **$SingleTenantDatabaseName = contosoconcerthall**
1. Ejecute el script con **F5**.


1. En [Azure Portal](https://portal.azure.com), vaya a la lista de bases de datos en el servidor *tenants1-dpt-\<user\>* . 
1. Haga clic en la base de datos **contosoconcerthall**.
1. Haga clic en el grupo en el que se encuentra **contosoconcerthall**. Busque el grupo en la sección **Grupo elásticos**.

1. Inspeccione el gráfico **Supervisión de grupo elástico** y busque el aumento de uso de eDTU en el grupo. Pasado un minuto o dos, la carga mayor entrará en vigor rápidamente y verá que el grupo alcanza el 100 % de uso.
2. Observe la pantalla **Supervisión de base de datos elástica** que muestra las bases de datos principales de la última hora. La base de datos *contosoconcerthall* pronto debería aparecer como una de las cinco bases de datos principales.
3. **Haga clic en el gráfico Supervisión de bases de datos elásticas** y se abre la **página** **Uso de recursos de base de datos**, donde puede supervisar cualquier base de datos. Esto le permite aislar la presentación de la base de datos *contosoconcerthall*.
4. En la lista de bases de datos, haga clic en **contosoconcerthall**.
5. Haga clic en **Plan de tarifa (escalar DTU)** para abrir la página **Configurar rendimiento** donde puede establecer un tamaño de proceso independiente para la base de datos.
6. Haga clic en la pestaña **Estándar** para abrir las opciones de escalado del nivel Estándar.
7. Deslice el **control deslizante de las DTU** a la derecha para seleccionar **100** DTU. Tenga en cuenta que esto se corresponde con el objetivo de servicio, **S3**.
8. Haga clic en **Aplicar** para sacar la base de datos del grupo y convertirla en una base de datos *S3 Estándar*.
9. Una vez completado el escalado, supervise el efecto sobre la base de datos contosoconcerthall y Pool1 en las hojas de base de datos y grupos elásticos.

Una vez que desaparezca el exceso de carga de la base de datos contosoconcerthall debe devolverla rápidamente al grupo para reducir el coste. Si no es evidente cuándo ocurrirá esto, podría establecer una alerta en la base de datos que se active cuando el uso de DTU descienda por debajo del máximo por base de datos establecido para el grupo. El movimiento de una base de datos a un grupo se describe en el ejercicio 5.

## <a name="other-performance-management-patterns"></a>Otros patrones de administración del rendimiento

**Escalado preventivo** En el ejercicio anterior donde exploramos cómo escalar una base de datos aislada, sabía la base de datos que buscaba. Si la administración de Contoso Concert Hall había informado a Wingtips de la venta inminente de entradas, la base de datos podía haberse sacado del grupo de manera preventiva. De lo contrario, probablemente se hubiera necesitado una alerta en el grupo o la base de datos permite detectar lo que estaba pasando. No le gustaría que le informaran de esto los otros inquilinos del grupo con quejas sobre el rendimiento. Y si el inquilino puede predecir durante cuánto tiempo necesita recursos adicionales, podrá configurar un runbook de Azure Automation para sacar la base de datos del grupo y volverla a meter según un programa determinado.

**Autoservicio de escalado de inquilinos**. Como el escalado es una tarea que se llama fácilmente a través de la API de administración, puede crear fácilmente la capacidad de escalar las bases de datos de inquilinos en la aplicación de inquilino y ofrecerla como característica del servicio SaaS. Por ejemplo, puede permitir que los inquilinos se autoadministren el escalado y la reducción vertical y quizá vincularlo directamente a su facturación.

**Escalado y reducción vertical de un grupo según un programa para cumplir patrones de uso**

Cuando el uso global de los inquilinos siga patrones previsibles, puede utilizar Azure Automation para escalar y reducir verticalmente un grupo según un programa. Por ejemplo, puede reducir un grupo verticalmente después de las 6 p.m. y escalarlo de nuevo antes de las 6 a.m. entre semana porque sabe que se necesitan menos recursos.



## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Simular el uso de las bases de datos de inquilinos mediante la ejecución de un generador de carga proporcionado
> * Supervisar las bases de datos de inquilinos a medida que responden al aumento de carga
> * Escalar verticalmente el grupo elástico en respuesta al aumento de la carga en la base de datos
> * Aprovisionar un segundo grupo elástico para equilibrar la actividad de la base de datos

[Tutorial sobre la restauración de un único inquilino](saas-dbpertenant-restore-single-tenant.md)


## <a name="additional-resources"></a>Recursos adicionales

* Otros [tutoriales basados en la implementación de la aplicación Wingtip Tickets SaaS Database Per Tenant](saas-dbpertenant-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)
* [Grupos elásticos de SQL](elastic-pool-overview.md)
* [Azure Automation](../../automation/automation-intro.md)
* [Registros de Azure Monitor](./saas-dbpertenant-log-analytics.md): tutorial de configuración y uso de los registros de Azure Monitor