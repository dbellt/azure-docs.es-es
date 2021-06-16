---
title: Supervisión del rendimiento de una base de datos multiinquilino con particiones
description: Supervisión y administración del rendimiento de una base de datos multiinquilino con particiones de Azure SQL Database en una aplicación SaaS multiinquilino
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
ms.openlocfilehash: cecd9f262d2a2a968659eb08a1a892bfa330428b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110699066"
---
# <a name="monitor-and-manage-performance-of-sharded-multi-tenant-azure-sql-database-in-a-multi-tenant-saas-app"></a>Supervisión y administración del rendimiento de una base de datos multiinquilino con particiones de Azure SQL Database en una aplicación SaaS multiinquilino
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este tutorial se describen varios escenarios clave de administración de rendimiento que se usan en aplicaciones de SaaS. Mediante un generador de carga para simular actividad en todas las bases de datos multiinquilino con particiones, se demuestran las características integradas de supervisión y alertas de Azure SQL Database.

La aplicación SaaS de base de datos multiinquilino llamada Wingtip Tickets usa un modelo de datos multiinquilino con particiones, en el que los datos de ubicación (inquilino) se distribuyen según el identificador de inquilino entre varias bases de datos. Al igual que en muchas aplicaciones SaaS, el patrón de carga de trabajo de inquilino previsto es imprevisible y esporádico. En otras palabras, las ventas de entradas pueden producirse en cualquier momento. Para sacar el máximo partido a este patrón típico de uso de base datos, se pueden escalar y reducir verticalmente las bases de datos para optimizar el costo de una solución. Con este tipo de patrón, es importante supervisar el uso de los recursos de la base de datos para asegurarse de que las cargas están razonablemente repartidas entre varias bases de datos. También debe asegurarse de que las bases de datos individuales tienen los recursos adecuados y de que no han alcanzado el límite de [unidades de transmisión de datos](purchasing-models.md#dtu-based-purchasing-model). En este tutorial se exploran métodos para supervisar y administrar las bases de datos, y se explica cómo tomar medidas correctivas en respuesta a las variaciones en la carga de trabajo.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> 
> * Simular el uso de una base de datos multiinquilino con particiones mediante la ejecución de un generador de carga proporcionado
> * Supervisar la base de datos a medida que responde al aumento de carga
> * Escalar verticalmente la base de datos en respuesta al aumento de la carga de esta
> * Aprovisionar un inquilino en una base de datos de un único inquilino

Para completar este tutorial, asegúrese de cumplir estos requisitos previos:

* La aplicación SaaS de base de datos multiinquilino Wingtip Tickets está implementada. Para implementarla en menos de cinco minutos, consulte el artículo sobre la [implementación y exploración de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets](./saas-multitenantdb-get-started-deploy.md)
* Azure PowerShell está instalado. Para más información, consulte [Introducción a Azure PowerShell](/powershell/azure/get-started-azureps)

## <a name="introduction-to-saas-performance-management-patterns"></a>Introducción a los patrones de administración del rendimiento de SaaS

La administración del rendimiento de la base de datos consiste en compilar y analizar los datos de rendimiento y, a continuación, reaccionar a estos datos mediante el ajuste de parámetros para mantener un tiempo de respuesta aceptable de la aplicación. 

### <a name="performance-management-strategies"></a>Estrategias de administración del rendimiento

* Para evitar tener que supervisar el rendimiento manualmente, resulta más eficaz **establecer alertas que se activan cuando las bases de datos se alejan de los intervalos normales**.
* Para responder a las fluctuaciones en el tamaño de proceso de una base de datos a corto plazo, **se puede escalar horizontal o verticalmente el nivel de DTU**. Si esta fluctuación es habitual o previsible, **se puede programar el escalado automático de la base de datos**. Por ejemplo, cuando haya poca carga de trabajo (por la noche o durante el fin de semana), redúzcalo verticalmente.
* Para responder a las fluctuaciones a largo plazo o a cambios en los inquilinos, **los inquilinos individuales se pueden mover a otra base de datos**.
* Para responder a aumentos breves a corto plazo en la carga *individual* de un inquilino, **se pueden sacar los inquilinos individuales de una base de datos para asignarle un tamaño de proceso individual**. Una vez reducida la carga, el inquilino se puede devolver a la base de datos multiinquilino. Si se sabe de antemano, los inquilinos se pueden mover con antelación para garantizar que la base de datos siempre tiene los recursos necesarios y evitar que otros inquilinos de la base de datos multiinquilino se vean afectados. Si es predecible, como para un establecimiento, una avalancha de ventas de entradas para un evento popular, este comportamiento de administración puede integrarse en la aplicación.

[Azure Portal](https://portal.azure.com) proporciona supervisión y alertas integradas en la mayoría de recursos. Para SQL Database, la supervisión y las alertas están disponibles para las bases de datos. La supervisión y las alertas integradas son específicas de los recursos, por lo que es conveniente usarlas con un número reducido de recursos, pero no cuando se trabaja con muchos recursos.

Para escenarios de gran volumen, donde se trabaja con muchos recursos, se puede usar los [registros de Azure Monitor](https://azure.microsoft.com/services/log-analytics/). Se trata de un servicio de Azure independiente que proporciona análisis de los registros recopilados en un área de trabajo de Log Analytics. Los registros de Azure Monitor pueden recopilar la telemetría de muchos servicios y sirve para consultar y establecer alertas.

## <a name="get-the-wingtip-tickets-saas-multi-tenant-database-application-source-code-and-scripts"></a>Obtención del código fuente y los scripts de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets

Los scripts y el código fuente de la aplicación SaaS de base de datos multiinquilino Wingtip Tickets están disponibles en el repositorio de GitHub [WingtipTicketsSaaS-MultitenantDB](https://github.com/microsoft/WingtipTicketsSaaS-MultiTenantDB). Consulte las [instrucciones generales](saas-tenancy-wingtip-app-guidance-tips.md) para saber cuáles son los pasos para descargar y desbloquear los scripts SaaS de Wingtip Tickets.

## <a name="provision-additional-tenants"></a>Aprovisionamiento de inquilinos adicionales

Para una buena comprensión de cómo funciona la administración y la supervisión del rendimiento a escala, en este tutorial es necesario que disponga de varios inquilinos en una base de datos multiinquilino con particiones.

Si ya ha aprovisionado un lote de inquilinos en un tutorial anterior, vaya directamente a la sección [Simulación de uso en todas las bases de datos de inquilinos](#simulate-usage-on-all-tenant-databases).

1. En **PowerShell ISE**, abra \\Learning Modules\\Performance Monitoring and Management\\*Demo-PerformanceMonitoringAndManagement.ps1*. Mantenga este script abierta, ya que se van a ejecutar varios escenarios en este tutorial.
1. Establezca **$DemoScenario** = **1**, _Aprovisionamiento de un lote de inquilinos_
1. Presione **F5** para ejecutar el script.

El script implementa 17 inquilinos en la base de datos multiinquilino en unos minutos. 

El script *New-TenantBatch* crea nuevos inquilinos con claves de inquilino exclusivas dentro de la base de datos multiinquilino con particiones y los inicializa con el nombre del inquilino y el tipo de ubicación. Esto es coherente con la manera en que la aplicación aprovisiona un nuevo inquilino. 

## <a name="simulate-usage-on-all-tenant-databases"></a>Simulación de uso en todas las bases de datos de inquilinos

Se proporciona el script *Demo-PerformanceMonitoringAndManagement.ps1* que simula una carga de trabajo que se ejecuta en la base de datos multiinquilino. La carga se genera mediante uno de los escenarios de carga disponibles:

| Demostración | Escenario |
|:--|:--|
| 2 | Generación de una carga de intensidad normal (aproximadamente 30 DTU) |
| 3 | Generación de una carga con más ráfagas por inquilino|
| 4 | Generación de una carga con mayores ráfagas de DTU por inquilino (aproximadamente 70 DTU)|
| 5 | Generación de una carga de alta intensidad (aproximadamente 90 DTU) en un solo inquilino y de una carga de intensidad normal en todos los demás |

El generador de carga se aplica una carga *sintética* exclusiva de CPU para cada base de datos de inquilinos. El generador de inicia un trabajo para cada base de datos de inquilinos, que llama a un procedimiento almacenado periódicamente que genera la carga. Los niveles de carga (en DTU), la duración y los intervalos varían en todas las bases de datos, lo cual simula una actividad de inquilinos imprevisible.

1. En **PowerShell ISE**, abra \\Learning Modules\\Performance Monitoring and Management\\*Demo-PerformanceMonitoringAndManagement.ps1*. Mantenga este script abierta, ya que se van a ejecutar varios escenarios en este tutorial.
1. Establezca **$DemoScenario** = **2**, _Generación de una carga de intensidad normal_.
1. Presione **F5** para aplicar una carga en todos los inquilinos.

La aplicación SaaS de base de datos multiinquilino Wingtip Tickets es una aplicación SaaS y, en el mundo real, la carga que tiene una aplicación SaaS habitualmente es esporádica e impredecible. Para realizar la simulación, el generador de carga crea una carga aleatoria que distribuye entre todos los inquilinos. El patrón de carga tarda varios minutos en crearse, por lo que se recomienda dejar que el generador de carga se ejecute durante unos 3-5 minutos antes de intentar supervisar la carga en las secciones siguientes.

> [!IMPORTANT]
> El generador de carga se ejecuta como una serie de trabajos en una nueva ventana de PowerShell. Si cierra la sesión, se detiene el generador de carga. El generador de carga permanece en un estado *de invocación de trabajo* en el que genera carga en los nuevos inquilinos que se aprovisionaron después de iniciar el generador. Presione *Ctrl-C* para detener la invocación de nuevos trabajos y salir del script. El generador de carga continuará ejecutándose, pero solo en los inquilinos existentes.

## <a name="monitor-resource-usage-using-the-azure-portal"></a>Supervisión del uso de los recursos mediante Azure Portal

Para supervisar el uso de los recursos derivado de la carga que se aplica, abra el portal por la base de datos multiinquilino, **tenants1**, que contiene los inquilinos:

1. Abra [Azure Portal](https://portal.azure.com) y vaya al servidor *tenants1-mt-&lt;USER&gt;* .
1. Desplácese hacia abajo y busque las bases de datos y haga clic en **tenants1**. Esta base de datos multiinquilino con particiones contiene todos los inquilinos creados hasta ahora.

![gráfico de base de datos](./media/saas-multitenantdb-performance-monitoring/multitenantdb.png)

Observe el gráfico de **DTU**.

## <a name="set-performance-alerts-on-the-database"></a>Establecimiento de alertas de rendimiento en la base de datos

Establezca una alerta en la base de datos que se desencadene cuando el uso sea de \>75 % como sigue:

1. Abra la base de datos *tenants1* (en el servidor *tenants1-mt-&lt;USER&gt;* ) de [Azure Portal](https://portal.azure.com).
1. Haga clic en **Reglas de alerta** y en **+ Agregar alerta**:

   ![agregar alerta](./media/saas-multitenantdb-performance-monitoring/add-alert.png)

1. Proporcione un nombre, como **High DTU**,
1. Establezca los valores siguientes:
   * **Métrica = Porcentaje de DTU**
   * **Condición = greater than**
   * **Umbral = 75**.
   * **Período = Over the last 30 minutes**
1. Agregue una dirección de correo al cuadro *Correos electrónicos adicionales del administrador* y haga clic en **Aceptar**.

   ![Establecer alerta](./media/saas-multitenantdb-performance-monitoring/set-alert.png)

## <a name="scale-up-a-busy-database"></a>Escalado vertical de una base de datos ocupada

Si aumenta el nivel de carga en una base de datos hasta el punto de llegar al máximo y se alcanza el 100 % del uso de DTU, el rendimiento de la base de datos se verá afectado, lo cual podría ralentizar los tiempos de respuesta de consulta.

**A corto plazo**, considere la posibilidad de escalar verticalmente la base de datos para proporcionar recursos adicionales o eliminar inquilinos de la base de datos multiinquilino (moverlos de esta a una base de datos independiente).

**A largo plazo**, considere la posibilidad de optimizar las consultas o el uso de índices para mejorar el rendimiento de las bases de datos. En función de la sensibilidad de la aplicación a los problemas de rendimiento, es un procedimiento recomendado el escalado vertical de una base de datos antes de que llegue al 100 % del uso de DTU. Use una alerta para que le avise con antelación.

Puede simular una base de datos ocupada si aumenta la carga que produce el generador. Hacer que los inquilinos generen ráfagas con más frecuencia y más prolongadas aumenta la carga de la base de datos multiinquilino sin cambiar los requisitos de los inquilinos individualmente. El escalado vertical de la base de datos se realiza fácilmente en el portal o desde PowerShell. En este ejercicio se usa el portal.

1. Establezca *$DemoScenario* = **3**, _Generación de una carga con ráfagas más prolongadas y frecuentes por base de datos_ para aumentar la intensidad de la carga global de la base de datos sin cambiar la carga máxima necesaria para cada inquilino.
1. Presione **F5** para aplicar una carga de todas las bases de datos de inquilinos.
1. Vaya a la base de datos **tenants1** en Azure Portal.

Supervise el aumento del uso de DTU de la base de datos en el gráfico superior. La carga mayor tarda unos minutos en entrar en vigor, pero debería ver cómo rápidamente la base de datos empieza a llegar al máximo uso y, según la carga se va estabilizando en el nuevo modelo, sobrecarga rápidamente la base de datos.

1. Para escalar verticalmente la base de datos, haga clic en **Plan de tarifa (escalar DTU)** en la hoja de configuración.
1. Ajuste el valor de **DTU** a **100**. 
1. Haga clic en **Aplicar** para enviar la solicitud para escalar la base de datos.

Vuelva a **tenants1** > **Introducción** para ver los gráficos de supervisión. Supervise el efecto de proporcionar más recursos a la base de datos (aunque con pocos inquilinos y una carga aleatoria no siempre es fácil verlo bien hasta que se ejecuta durante un tiempo). Mientras observe los gráficos, tenga en cuenta que el 100 % del gráfico superior representa ahora 100 DTU, mientras que en el gráfico inferior, el 100 % sigue siendo 50 DTU.

Las bases de datos permanecen en línea y están totalmente disponibles durante el proceso. El código de la aplicación debe escribirse para siempre reintente recuperar las conexiones interrumpidas, por lo tanto, se volverá a conectar a la base de datos.

## <a name="provision-a-new-tenant-in-its-own-database"></a>Aprovisionamiento de un inquilino nuevo en su propia base de datos 

El modelo multiinquilino con particiones le permite elegir entre aprovisionar un nuevo inquilino en una base de datos multiinquilino junto con otros inquilinos o aprovisionar el inquilino en su propia base de datos. Gracias al aprovisionamiento de un inquilino en su propia base de datos, este se beneficia del aislamiento inherente de la base de datos independiente, lo que le permite administrar el rendimiento de ese inquilino de forma independiente al de otros, restaurar ese inquilino de forma independiente al de otros, etc. Por ejemplo, puede decidir colocar los clientes de versiones de evaluación gratuita o los clientes habituales en una base de datos multiinquilino, y los clientes Premium en bases de datos individuales.  Aunque se creen bases de datos aisladas de un solo inquilino, se podrán seguir administrando de forma colectiva en un grupo elástico para optimizar el costo de los recursos.

Si ya ha aprovisionado un nuevo inquilino en su propia base de datos, omita los pasos siguientes.

1. En **PowerShell ISE**, abra...\\Learning Modules\\ProvisionTenants\\*Demo-ProvisionTenants.ps1*. 
1. Modifique **$TenantName = "Salix Salsa"** y **$VenueType  = "dance"** .
1. Establezca **$Scenario** = **2**, _Aprovisionamiento de un inquilino en una nueva base de datos de un solo inquilino_
1. Presione **F5** para ejecutar el script.

El script aprovisionará este inquilino en una base de datos independiente, registrará la base de datos y el inquilino con el catálogo y, a continuación, abrirá la página Eventos del inquilino en el explorador. Actualice la página Centro de eventos y verá que "Salix Salsa" se ha agregado como una ubicación.

## <a name="manage-performance-of-an-individual-database"></a>Administración del rendimiento de una base de datos individual

Si un único inquilino de una base de datos multiinquilino experimenta una gran carga de forma continuada, puede que tienda a dominar los recursos de la base de datos y que esto afecte al resto de los inquilinos de esa base datos. Si es probable que la actividad continúe durante un tiempo, ese inquilino se puede sacar temporalmente de la base de datos y colocarlo en su propia base de datos de un solo inquilino. Esto permite que el inquilino tenga los recursos adicionales que necesita y lo aísla de los otros inquilinos.

En este ejercicio se simula el efecto de una carga elevada en Salix Salsa cuando se venden entradas para un evento popular.

1. Abra el script …\\*Demo-PerformanceMonitoringAndManagement.ps1*.
1. Establezca **$DemoScenario = 5 _,_ genere una carga normal y una carga elevada en un solo inquilino (aproximadamente 90 DTU).**
1. Establezca **$SingleTenantName = Salix Salsa**
1. Ejecute el script con **F5**.

Vaya al portal y, después, vaya a **salixsalsa** > **Introducción** para ver los gráficos de supervisión. 

## <a name="other-performance-management-patterns"></a>Otros patrones de administración del rendimiento

**Autoservicio de escalado de inquilinos**

Como el escalado es una tarea que se llama fácilmente a través de la API de administración, puede crear fácilmente la capacidad de escalar las bases de datos de inquilinos en la aplicación de inquilino y ofrecerla como característica del servicio SaaS. Por ejemplo, puede permitir que los inquilinos se autoadministren el escalado y la reducción vertical y quizá vincularlo directamente a su facturación.

**Escalado y reducción vertical de una base de datos según un programa para cumplir patrones de uso**

Cuando el uso global de los inquilinos siga patrones previsibles, puede utilizar Azure Automation para escalar y reducir verticalmente una base de datos según un programa. Por ejemplo, puede reducir una base de datos verticalmente después de las 6 p.m. y escalarla de nuevo antes de las 6 a.m. entre semana porque sabe que se necesitan menos recursos.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Simular el uso de una base de datos multiinquilino con particiones mediante la ejecución de un generador de carga proporcionado
> * Supervisar la base de datos a medida que responde al aumento de carga
> * Escalar verticalmente la base de datos en respuesta al aumento de la carga de esta
> * Aprovisionar un inquilino en una base de datos de un único inquilino

## <a name="additional-resources"></a>Recursos adicionales

<!--* [Additional tutorials that build upon the Wingtip Tickets SaaS Multi-tenant Database application deployment](saas-multitenantdb-wingtip-app-overview.md#sql-database-wingtip-saas-tutorials)-->
* [Azure Automation](../../automation/automation-intro.md)