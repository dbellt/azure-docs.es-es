---
title: Preguntas sobre la detección, la valoración y el análisis de dependencias en Azure Migrate
description: Obtenga respuestas a preguntas comunes sobre detección, valoración y análisis de dependencias en Azure Migrate.
author: rashijoshi
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/09/2020
ms.openlocfilehash: ff7bd9145b64f82ca514897ebb67526bc28de867
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108143582"
---
# <a name="discovery-assessment-and-dependency-analysis---common-questions"></a>Detección, valoración y análisis de dependencias: preguntas comunes

En este artículo se responde a preguntas comunes sobre detección, valoración y análisis de dependencias en Azure Migrate. Si tiene otras preguntas, repase estos recursos:

- [Preguntas generales](resources-faq.md) sobre Azure Migrate
- Preguntas sobre el [dispositivo de Azure Migrate](common-questions-appliance.md)
- Preguntas sobre la [migración de servidores](common-questions-server-migration.md)
- Obtenga respuestas a sus preguntas en el [foro de Azure Migrate](https://aka.ms/AzureMigrateForum)


## <a name="what-geographies-are-supported-for-discovery-and-assessment-with-azure-migrate"></a>¿Qué zonas geográficas se admiten para la detección y evaluación con Azure Migrate?

Revise las zonas geográficas admitidas para [nubes públicas](migrate-support-matrix.md#supported-geographies-public-cloud) y [gubernamentales](migrate-support-matrix.md#supported-geographies-azure-government).


## <a name="how-many-servers-can-i-discover-with-an-appliance"></a>¿Cuántos servidores puedo detectar con un dispositivo?

Puede detectar hasta 10 000 servidores del entorno VMware, hasta 5 000 servidores del entorno Hyper-V y hasta 1000 servidores físicos mediante el uso de un solo dispositivo. Si tiene más servidores, lea acerca de [escalar una valoración de Hyper-V](scale-hyper-v-assessment.md), [escalar una valoración de VMware](scale-vmware-assessment.md) y [escalar una valoración de servidor físico](scale-physical-assessment.md).

## <a name="how-do-i-choose-the-assessment-type"></a>¿Cómo elegir el tipo de valoración?

- Utilice las **valoraciones de máquinas virtuales de Azure** cuando quiera evaluar servidores de su entorno local de [VMware](how-to-set-up-appliance-vmware.md) y [Hyper-V](how-to-set-up-appliance-hyper-v.md), y [servidores físicos](how-to-set-up-appliance-physical.md) para la migración a máquinas virtuales de Azure. [Más información](concepts-assessment-calculation.md)

- Use el tipo de evaluación de **Azure SQL** si desea evaluar la instancia local de SQL Server del entorno de VMware para la migración a Azure SQL Database o a Azure SQL Managed Instance. [Más información](concepts-assessment-calculation.md)

- Utilice las valoraciones de **Azure VMware Solution (AVS)** cuando quiera evaluar [las máquinas virtuales VMware](how-to-set-up-appliance-vmware.md) locales para la migración a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md) con este tipo de valoración. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

- Puede usar un grupo común con máquinas de VMware solo para ejecutar ambos tipos de valoraciones. Tenga en cuenta que si está ejecutando valoraciones de AVS en Azure Migrate por primera vez, es aconsejable crear un nuevo grupo de máquinas de VMware.
 

## <a name="why-is-performance-data-missing-for-someall-servers-in-my-azure-vm-andor-avs-assessment-report"></a>¿Por qué faltan los datos de rendimiento de algunos o todos los servidores de mi máquina virtual de Azure o el informe de evaluación de AVS?

En el caso de la evaluación "en función del rendimiento", la exportación del informe de evaluación indica "PercentageOfCoresUtilizedMissing" o "PercentageOfMemoryUtilizedMissing" cuando el dispositivo de Azure Migrate no puede recopilar datos de rendimiento de los servidores locales pertinentes. Compruebe:

- Si los servidores están encendidos durante el tiempo en que crea la evaluación
- Si solo faltan contadores de memoria y está intentando evaluar servidores en un entorno Hyper-V. En este caso, habilite la memoria dinámica en los servidores y "vuelva a calcular" la evaluación para reflejar los últimos cambios. El dispositivo solo puede recopilar los valores de uso para servidores en un entorno Hyper-V si el servidor tiene habilitada la memoria dinámica.

- Si faltan todos los contadores de rendimiento, asegúrese de que se permiten las conexiones de salida en los puertos 443 (HTTPS).

    > [!Note]
    > Si falta alguno de los contadores de rendimiento, Azure Migrate: evaluación de servidores vuelve a la memoria y los núcleos locales asignados, y recomienda un tamaño de máquina virtual acorde.


## <a name="why-is-performance-data-missing-for-someall-sql-instancesdatabases-in-my-azure-sql-assessment"></a>¿Por qué faltan datos de rendimiento en algunas o todas las instancias o bases de datos de SQL en la evaluación de Azure SQL?

Para asegurarse de que se recopilen los datos de rendimiento, compruebe lo siguiente:

- Si las instancias de SQL Server están encendidas durante el tiempo en que crea la evaluación
- Si el estado de la conexión del Agente SQL en Azure Migrate es "Conectado" y compruebe el último latido 
- Si el estado de conexión de Azure Migrate para todas las instancias de SQL es "Conectado" en la hoja de la instancia de SQL detectada
- Si faltan todos los contadores de rendimiento, asegúrese de que se permiten las conexiones de salida en los puertos 443 (HTTPS).

Si falta alguno de los contadores de rendimiento, la evaluación de Azure SQL recomienda la configuración de Azure SQL más pequeña para esa instancia o base de datos.

## <a name="why-is-the-confidence-rating-of-my-assessment-low"></a>¿Por qué la clasificación de confianza de mi valoración es baja?

La clasificación de confianza se calcula para las evaluaciones "en función del rendimiento" en función del porcentaje de [puntos de datos disponibles](./concepts-assessment-calculation.md#ratings) necesarios para calcular la evaluación. Estos son los motivos por los que una valoración puede obtener una clasificación de confianza baja:

- No generó un perfil de su entorno durante el tiempo que está creando la evaluación. Por ejemplo, si está creando una evaluación con la duración de rendimiento establecida en una semana, debe esperar al menos una semana después de iniciar la detección para que se recopilen todos los puntos de datos. Si no puede esperar el período de duración, puede cambiar la duración del rendimiento a un período más pequeño y **Recalcular** la evaluación.
 
- La evaluación no puede recopilar los datos de rendimiento de algunos o de todos los servidores en el período de evaluación. Para obtener una clasificación de confianza alta, asegúrese de que: 
    - Los servidores estén activados mientras dure la evaluación.
    - Se permiten las conexiones salientes en los puertos 443.
    - La memoria dinámica está habilitada en los servidores de Hyper-V. 
    - Si el estado de la conexión de los agentes en Azure Migrate es "Conectado" y compruebe el último latido.
    - Para las evaluaciones de Azure SQL, el estado de conexión de Azure Migrate para todas las instancias de SQL es "Conectado" en la hoja de la instancia de SQL detectada.

    **Recalcule** la evaluación para reflejar los cambios más recientes en la clasificación de confianza.

- En el caso de las evaluaciones de máquinas virtuales de Azure y AVS, se crearon pocos servidores después de iniciar la detección. Por ejemplo, si va a crear una evaluación para el historial de rendimiento del último mes, pero se crearon algunos servidores en el entorno hace solo una semana. En este caso, los datos de rendimiento de los nuevos servidores no estarán disponibles para todo el período y la clasificación de confianza será baja. [Más información](./concepts-assessment-calculation.md#confidence-ratings-performance-based)

- En el caso de las evaluaciones de Azure SQL, se crearon algunas instancias de SQL o bases de datos después de que se iniciara la detección. Por ejemplo, si va a crear una evaluación para el historial de rendimiento del último mes, pero se crearon algunas instancias o bases de datos de SQL en el entorno hace solo una semana. En este caso, los datos de rendimiento de los nuevos servidores no estarán disponibles para todo el período y la clasificación de confianza será baja. [Más información](./concepts-azure-sql-assessment-calculation.md#confidence-ratings)

## <a name="-the-number-of-azure-vm-or-avs-assessments-on-the-discovery-and-assessment-tool-are-incorrect"></a>> El número de evaluaciones de máquinas virtuales de Azure o AVS en la herramienta de detección y evaluación son incorrectos
 Para corregir esto, haga clic en el número total de evaluaciones para navegar a todas las evaluaciones y recalcular la máquina virtual de Azure o la evaluación de AVS. La herramienta de detección y evaluación mostrará el recuento correcto de ese tipo de evaluación.


## <a name="i-want-to-try-out-the-new-azure-sql-assessment"></a>Quiero probar la nueva evaluación de Azure SQL
La detección y evaluación de las instancias y bases de datos de SQL Server que se ejecutan en el entorno de VMware se encuentran ahora en versión preliminar. Para empezar, realice [este tutorial](tutorial-discover-vmware.md). Si quiere probar esta característica en un proyecto existente, asegúrese de haber completado los [requisitos previos](how-to-discover-sql-existing-project.md) de este artículo.

## <a name="i-cant-see-some-servers-when-i-am-creating-an-azure-sql-assessment"></a>No puedo ver algunos servidores cuando creo una evaluación de Azure SQL

- La evaluación de Azure SQL solo se puede realizar en servidores que se ejecutan donde se detectaron instancias de SQL. Si no ve los servidores y las instancias de SQL que desea evaluar, espere unos momentos hasta que se complete la detección y, a continuación, cree la evaluación. 
- Si no puede ver un grupo creado anteriormente durante la creación de la evaluación, quite los servidores que no sean de VMware o que no tengan una instancia de SQL del grupo.
- Si está ejecutando evaluaciones de Azure SQL en Azure Migrate por primera vez, es aconsejable crear un nuevo grupo de servidores.

## <a name="i-want-to-understand-how-was-the-readiness-for-my-instance-computed"></a>Quiero comprender cómo se ha calculado la preparación de la instancia
La preparación de las instancias de SQL se ha calculado después de realizar una comprobación de compatibilidad de características con el tipo de implementación de Azure SQL de destino (Azure SQL Database o Azure SQL Managed Instance). [Más información](./concepts-azure-sql-assessment-calculation.md#calculate-readiness)

## <a name="why-is-the-readiness-for-all-my-sql-instances-marked-as-unknown"></a>¿Por qué se marca la preparación de todas las instancias de SQL como desconocida?
Si la detección se ha iniciado recientemente y aún está en curso, es posible que vea la preparación de algunas o de todas las instancias de SQL como desconocida. Es recomendable que espere un tiempo para que el dispositivo pueda generar perfiles del entorno y, a continuación, volver a calcular la evaluación.
La detección de SQL se realiza una vez cada 24 horas y puede que tenga que esperar hasta un día para que se reflejen los últimos cambios de configuración. 

## <a name="why-is-the-readiness-for-some-of-my-sql-instances-marked-as-unknown"></a>¿Por qué se marca la preparación de algunas o todas las instancias de SQL como desconocida?
Esto puede suceder si: 
- La detección todavía está en curso. Es recomendable que espere un tiempo para que el dispositivo pueda generar perfiles del entorno y, a continuación, volver a calcular la evaluación.
- Hay algunos problemas de detección que debe corregir en la hoja de errores y notificaciones.

La detección de SQL se realiza una vez cada 24 horas y puede que tenga que esperar hasta un día para que se reflejen los últimos cambios de configuración.

## <a name="my-assessment-is-in-outdated-state"></a>Mi evaluación está en estado Obsoleto

### <a name="azure-vmavs-assessment"></a>Evaluación de máquina virtual de Azure o AVS
Si hay cambios en el entorno local de los servidores que se encuentran en un grupo que se ha evaluado, la evaluación se marca como obsoleta. Una valoración se puede marcar como "obsoleta" debido a uno o varios cambios en las siguientes propiedades:
- Número de núcleos de procesador
- Memoria asignada
- Tipo de arranque o firmware
- Nombre, versión y arquitectura del sistema operativo
- Número de discos
- Número de adaptadores de red
- Cambio de tamaño de disco (GB asignados)
- Actualización de las propiedades de NIC. Ejemplo: Cambios de dirección Mac, adición de direcciones IP, etc.

Debe **Recalcular** la evaluación para reflejar los cambios más recientes en la evaluación.

### <a name="azure-sql-assessment"></a>Evaluación de Azure SQL
Si hay cambios en las instancias de SQL del entorno local que se encuentran en un grupo que se ha evaluado, la evaluación se marca como **Obsoleta**.
- Se ha agregado o eliminado una instancia de SQL de un servidor
- Se ha agregado o eliminado una base de datos SQL de una instancia de SQL
- El tamaño total de la base de datos de una instancia de SQL ha cambiado en más del 20 %
- Cambio en el número de núcleos de procesador o memoria asignada

Debe **Recalcular** la evaluación para reflejar los cambios más recientes en la evaluación.

## <a name="why-was-i-recommended-a-particular-target-deployment-type"></a>¿Por qué se recomienda un tipo de implementación de destino determinado?
Azure Migrate recomienda un tipo de implementación de Azure SQL específico que es compatible con su instancia de SQL. La migración a un destino recomendado de Microsoft reduce el esfuerzo general de migración. Se ha recomendado esta configuración de Azure SQL (SKU) después de considerar las características de rendimiento de la instancia de SQL y de las bases de datos que administra. Si hay varias configuraciones de Azure SQL válidas, se recomienda la que resulte más rentable. [Más información](./concepts-azure-sql-assessment-calculation.md#calculate-sizing)

## <a name="what-deployment-target-should-i-choose-if-my-sql-instance-is-ready-for-azure-sql-db-and-azure-sql-mi"></a>¿Qué destino de implementación debo elegir si mi instancia de SQL está lista para Azure SQL Database y Azure SQL Managed Instance? 
Si la instancia está lista para Azure SQL Database y Azure SQL Managed Instance, se recomienda el tipo de implementación de destino para el que el costo estimado de la configuración de Azure SQL sea menor.

## <a name="why-is-my-instance-marked-as-potentially-ready-for-azure-vm-in-my-azure-sql-assessment"></a>¿Por qué mi instancia está marcada como Potencialmente lista para una máquina virtual de Azure en mi evaluación de Azure SQL?
Esto puede suceder si el tipo de implementación de destino elegido en las propiedades de evaluación es **Recomendado** y la instancia de SQL no está lista para Azure SQL Database y Azure SQL Managed Instance. Se recomienda que el usuario cree una evaluación en Azure Migrate con el tipo de evaluación como **Máquina virtual de Azure** para determinar si el servidor en el que se ejecuta la instancia está listo para migrarse a una máquina virtual de Azure.
Se recomienda que el usuario cree una evaluación en Azure Migrate con el tipo de evaluación como **Azure VM** para determinar si el servidor en el que se ejecuta la instancia está listo para migrarse a una máquina virtual de Azure en su lugar:
- Las evaluaciones de máquinas virtuales de Azure en Azure Migrate siguen actualmente el enfoque de migración mediante lift-and-shift y no tendrán en cuenta las métricas de rendimiento específicas para ejecutar instancias y bases de datos de SQL en la máquina virtual de Azure. 
- Al ejecutar una evaluación de máquinas virtuales de Azure en un servidor, el tamaño recomendado y las estimaciones de costos serán para todas las instancias que se ejecutan en ese servidor y se pueden migrar a una máquina virtual de Azure mediante la herramienta Server Migration. Antes de migrar, [revise las directrices de rendimiento](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices-checklist.md) para SQL Server en máquinas virtuales de Azure.

## <a name="i-cant-see-some-databases-in-my-assessment-even-though-the-instance-is-part-of-the-assessment"></a>No veo algunas bases de datos en mi evaluación, aunque la instancia forma parte de ella

La evaluación de Azure SQL solo incluye las bases de datos que presentan el estado En línea. En caso de que la base de datos esté en cualquier otro estado, la evaluación omite el cálculo de la preparación, el tamaño y el costo de esas bases de datos. En caso de que desee evaluar estas bases de datos, cambie el estado de la base de datos y vuelva a calcular la evaluación en algún momento.

## <a name="i-want-to-compare-costs-for-running-my-sql-instances-on-azure-vm-vs-azure-sql-databaseazure-sql-managed-instance"></a>Quiero comparar los costos de la ejecución de mis instancias de SQL en máquinas virtuales de Azure con los de hacerlo en Azure SQL Database o Azure SQL Managed Instance

Puede crear una evaluación con el tipo **Máquina virtual de Azure** en el mismo grupo que se usó en la evaluación de **Azure SQL**. Después, puede comparar los dos informes en paralelo. Sin embargo, las evaluaciones de máquinas virtuales de Azure en Azure Migrate siguen actualmente el enfoque de migración mediante lift-and-shift y no tendrán en cuenta las métricas de rendimiento específicas para ejecutar instancias de SQL y bases de datos en la máquina virtual de Azure. Al ejecutar una evaluación de máquinas virtuales de Azure en un servidor, el tamaño recomendado y las estimaciones de costos serán para todas las instancias que se ejecutan en ese servidor y se pueden migrar a una máquina virtual de Azure mediante la herramienta Server Migration. Antes de migrar, [revise las directrices de rendimiento](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices-checklist.md) para SQL Server en máquinas virtuales de Azure.

## <a name="the-storage-cost-in-my-azure-sql-assessment-is-zero"></a>El costo de almacenamiento en mi evaluación de Azure SQL es cero
Para Azure SQL Managed Instance, no se agrega ningún costo de almacenamiento para el almacenamiento de los primeros 32 GB, por instancia y mes, y se agrega un costo de almacenamiento adicional para el almacenamiento en incrementos de 32 GB. [Más información](https://azure.microsoft.com/pricing/details/azure-sql/sql-managed-instance/single/)

## <a name="i-cant-see-some-groups-when-i-am-creating-an-azure-vmware-solution-avs-assessment"></a>No puedo ver algunos grupos cuando creo una valoración de Azure VMware Solution (AVS).

- La valoración de AVS se puede realizar en grupos que solo tienen máquinas de VMware. Quite cualquier máquina que no sea de VMware del grupo si pretende realizar una valoración de AVS.
- Si está ejecutando valoraciones de AVS en Azure Migrate por primera vez, es aconsejable crear un nuevo grupo de máquinas de VMware.

## <a name="i-cant-see-some-vm-types-and-sizes-in-azure-government"></a>No puedo ver algunos tamaños y tipos de VM en Azure Government

Los tamaños y tipos de VM que se admiten para la evaluación y la migración dependen de la disponibilidad en la ubicación de Azure Government. Puede [revisar y comparar](https://azure.microsoft.com/global-infrastructure/services/?regions=usgov-non-regional,us-dod-central,us-dod-east,usgov-arizona,usgov-iowa,usgov-texas,usgov-virginia&products=virtual-machines) los tipos de VM en Azure Government.

## <a name="the-size-of-my-server-changed-can-i-run-an-assessment-again"></a>El tamaño de mi servidor ha cambiado. ¿Puedo volver a ejecutar una valoración?

El dispositivo de Azure Migrate recopila continuamente información sobre el entorno local.  Una valoración es una instantánea de un momento dado de los servidores locales. Si cambia la configuración de un servidor que quiere evaluar, use la opción de recalcular para actualizar la evaluación con los últimos cambios.

## <a name="how-do-i-discover-servers-in-a-multitenant-environment"></a>¿Cómo puedo detectar servidores en un entorno de varios inquilinos?

- **VMware**: Si un entorno se comparte entre los inquilinos y no quiere detectar los servidores de un inquilino en la suscripción de otro inquilino, cree credenciales de VMware vCenter Server con acceso solo a los servidores que quiere detectar. Después, use esas credenciales al iniciar la detección en el dispositivo de Azure Migrate.
- **Hyper-V**: la detección usa credenciales de host de Hyper-V. Si los servidores comparten el mismo host de Hyper-V, no hay actualmente ninguna manera de separar la detección.  

## <a name="do-i-need-vcenter-server"></a>¿vCenter Server es necesario?

Sí, Azure Migrate necesita vCenter Server para detectar un entorno de VMware a fin de realizar la detección. Azure Migrate no admite la detección de hosts de ESXi que no administra vCenter Server.

## <a name="what-are-the-sizing-options-in-an-azure-vm-assessment"></a>¿Cuáles son las opciones de dimensionamiento en una evaluación de máquinas virtuales de Azure?

Con el ajuste como tamaño local, Azure Migrate no tiene en cuenta los datos de rendimiento del servidor para la evaluación. Azure Migrate evalúa los tamaños de las VM en función de la configuración local. Con el ajuste de tamaño basado en el rendimiento, el ajuste de tamaño se basa en los datos de uso.

Por ejemplo, si un servidor local tiene 4 núcleos y 8 GB de memoria con un 50 % de uso de CPU y un 50 % de uso de memoria:
- El ajuste de tamaño como local recomienda una SKU de máquina virtual de Azure con cuatro núcleos y 8 GB de memoria.
- El ajuste de tamaño basado en el rendimiento recomienda una SKU de VM de 2 núcleos y 4 GB de memoria, ya que se tiene en cuenta el porcentaje de uso.

De forma similar, el ajuste de tamaño de disco depende del criterio de ajuste de tamaño y el tipo de almacenamiento:
- Si el criterio de ajuste de tamaño se basa en el rendimiento y el tipo de almacenamiento es automático, Azure Migrate tiene en cuenta los valores de rendimiento y de IOPS del disco al identificar el tipo de disco de destino (estándar o prémium).
- Si el criterio de ajuste de tamaño está basado en el rendimiento y el tipo de almacenamiento es prémium, Azure Migrate recomienda una SKU de disco premium, basada en el tamaño del disco local. Se aplica la misma lógica para ajustar el tamaño del disco cuando el criterio de ajuste de tamaño es como local y el tipo de almacenamiento es estándar o prémium.

## <a name="does-performance-history-and-utilization-affect-sizing-in-an-azure-vm-assessment"></a>¿El historial de rendimiento y el uso afectan al dimensionamiento en una valoración de máquinas virtuales de Azure?

Sí, el historial de rendimiento y el uso afectan al dimensionamiento de máquinas virtuales de Azure.

### <a name="performance-history"></a>Historial de rendimiento

Solo para el ajuste de tamaño basado en el rendimiento, Azure Migrate recopila el historial de rendimiento de las máquinas locales y lo usa para recomendar el tipo de disco y el tamaño de la VM en Azure:

1. El dispositivo realiza un perfil del entorno local continuamente para recopilar datos de uso en tiempo real cada 20 segundos.
2. El dispositivo acumula los ejemplos de 20 segundos recopilados y los usa para crear un único punto de datos cada 15 minutos.
3. Para crear el punto de datos, el dispositivo selecciona el valor máximo de todos los ejemplos de 20 segundos.
4. El dispositivo envía el punto de datos a Azure.

### <a name="utilization"></a>Utilización

Cuando se crea una evaluación en Azure, en función de la duración del rendimiento y el valor del percentil del historial de rendimiento, Azure Migrate calcula el valor del uso efectivo y lo usa para ajustar el tamaño.

Por ejemplo, si establece la duración del rendimiento en 1 día y el valor del percentil en el percentil 95, Azure Migrate ordena los puntos de ejemplo de 15 minutos que envía el recopilador para el último día de manera ascendente. Elige el valor del percentil 95 como el uso efectivo.

El uso del valor del percentil 95 garantiza la omisión de todos los valores atípicos. Los valores atípicos se pueden incluir si Azure Migrate usa el percentil 99. Para elegir el uso máximo para el período sin omitir ningún valor atípico, establezca Azure Migrate para que utilice el percentil 99.


## <a name="how-are-import-based-assessments-different-from-assessments-with-discovery-source-as-appliance"></a>¿En qué se diferencian las valoraciones basadas en la importación de las valoraciones con el origen de detección como dispositivo?

Las valoraciones de máquinas virtuales de Azure basadas en la importación son valoraciones creadas con máquinas que se importan en Azure Migrate mediante un archivo CSV. Solo cuatro campos son obligatorios para la importación: nombre del servidor, núcleos, memoria y sistema operativo. Estos son algunos aspectos que hay que tener en cuenta: 
 - Los criterios de preparación son menos rigurosos en las valoraciones basadas en la importación en el parámetro de tipo de arranque. Si no se proporciona el tipo de arranque, se supone que la máquina usa el tipo BIOS y que no está macada como **Condicionalmente preparada**. En las valoraciones con el origen de detección como dispositivo, la preparación se marca como **Condicionalmente preparada** si falta el tipo de arranque. Esta diferencia en el cálculo de la preparación se debe a la posibilidad de que los usuarios no tengan toda la información sobre las máquinas en las primeras fases del planeamiento de la migración cuando se realizan las valoraciones basadas en la importación. 
 - Las valoraciones de importación basadas en el rendimiento usan el valor de uso proporcionado por el usuario para el cálculo de tamaño correcto. Dado que el valor de uso lo proporciona el usuario, las opciones **Historial de rendimiento** y **Utilización del percentil** están deshabilitadas en las propiedades de la valoración. En las valoraciones con el origen de detección como dispositivo, el valor del percentil elegido se selecciona de los datos de rendimiento recopilados por el dispositivo.

## <a name="why-is-the-suggested-migration-tool-in-import-based-avs-assessment-marked-as-unknown"></a>¿Por qué la herramienta de migración sugerida en la evaluación de AVS basada en la importación está marcada como desconocida?

En el caso de las máquinas importadas mediante un archivo CSV, se desconoce la herramienta de migración predeterminada en una valoración de AVS. Sin embargo, para las máquinas de VMware, se recomienda usar la solución Hybrid Cloud Extension (HCX) de VMware. [Más información](../azure-vmware/tutorial-deploy-vmware-hcx.md).


## <a name="what-is-dependency-visualization"></a>¿Qué es la visualización de dependencias?

La visualización de dependencias puede ayudarle a evaluar grupos de servidores para realizar la migración con mayor confianza. La visualización de dependencias comprueba las dependencias de las máquinas antes de llevar a cabo una valoración. Este proceso ayuda a asegurar que nada se queda atrás y evita interrupciones inesperadas al migrar a Azure. Azure Migrate usa la solución Service Map en Azure Monitor para habilitar la visualización de dependencias. [Más información](concepts-dependency-visualization.md).

> [!NOTE]
> El análisis de dependencias basado en agente no está disponible en Azure Government. Puede usar el análisis de dependencias sin agente

## <a name="whats-the-difference-between-agent-based-and-agentless"></a>¿Cuál es la diferencia entre basado en agente y sin agente?

La diferencia entre la visualización sin agente y la visualización basada en agente se resume en la siguiente tabla.

**Requisito** | **Sin agente** | **Basado en agente**
--- | --- | ---
Soporte técnico | Esta opción se encuentra actualmente en versión preliminar y solo está disponible servidores en un entorno VMware. [Revise](migrate-support-matrix-vmware.md#dependency-analysis-requirements-agentless) los sistemas operativos compatibles. | En disponibilidad general (GA).
Agente | No es necesario instalar agentes en las máquinas que quiere comprobar. | Agentes que debe instalar en las máquinas locales que quiere analizar: [Microsoft Monitoring Agent (MMA)](../azure-monitor/agents/agent-windows.md) y el [agente de dependencia](../azure-monitor/agents/agents-overview.md#dependency-agent). 
Prerrequisitos | [Revise](concepts-dependency-visualization.md#agentless-analysis) los requisitos previos y los requisitos de implementación. | [Revise](concepts-dependency-visualization.md#agent-based-analysis) los requisitos previos y los requisitos de implementación.
Log Analytics | No se requiere. | Azure Migrate utiliza la solución [Service Map](../azure-monitor/vm/service-map.md) de los [registros de Azure Monitor](../azure-monitor/logs/log-query-overview.md) para la visualización de dependencias. [Más información](concepts-dependency-visualization.md#agent-based-analysis).
Funcionamiento | Captura los datos de conexión TCP en las máquinas habilitadas para la visualización de dependencias. Después de la detección, recopila datos en intervalos de cinco minutos. | Los agentes de Service Map instalados en una máquina recopilan datos acerca de los procesos de TCP, así como de las conexiones de entrada o salida para cada proceso.
data | Nombre de aplicación, proceso y nombre del servidor de la máquina de origen.<br/><br/> Puerto, nombre de aplicación, proceso y nombre del servidor de la máquina de destino. | Nombre de aplicación, proceso y nombre del servidor de la máquina de origen.<br/><br/> Puerto, nombre de aplicación, proceso y nombre del servidor de la máquina de destino.<br/><br/> Se recopila la información sobre el número de conexiones, la latencia y la transferencia de datos, y está disponible para las consultas de Log Analytics. 
Visualización | El mapa de dependencias de un solo servidor se puede ver durante un plazo de entre una hora y 30 días. | Mapa de dependencia de un solo servidor.<br/><br/> El mapa puede verse solo durante una hora.<br/><br/> Mapa de dependencias de un grupo de servidores.<br/><br/> Agregue y quite servidores de un grupo desde la vista de mapa.
Exportación de datos | Los datos de los últimos 30 días se pueden descargar en formato CSV. | Los datos se pueden consultar con Log Analytics.


## <a name="do-i-need-to-deploy-the-appliance-for-agentless-dependency-analysis"></a>¿Es necesario implementar el dispositivo para el análisis de dependencia sin agente?

Sí, se debe implementar el [dispositivo de Azure Migrate](migrate-appliance.md).

## <a name="do-i-pay-for-dependency-visualization"></a>¿Tengo que pagar por la visualización de dependencias?

No. Más información sobre los [precios de Azure Migrate](https://azure.microsoft.com/pricing/details/azure-migrate/).

## <a name="what-do-i-install-for-agent-based-dependency-visualization"></a>¿Qué tengo que instalar para la visualización de dependencias basada en agente?

Para usar la visualización de dependencias basada en agente, descargue e instale agentes en cada máquina local que vaya a evaluar:

- [Microsoft Monitoring Agent (MMA)](../azure-monitor/agents/agent-windows.md)
- [Dependency Agent](../azure-monitor/agents/agents-overview.md#dependency-agent)
- Si también tiene máquinas sin conectividad a Internet, descargue e instale en ellas la puerta de enlace de Log Analytics.

Solo necesita estos agentes si utiliza la visualización de dependencias basada en agente.

## <a name="can-i-use-an-existing-workspace"></a>¿Puedo usar un área de trabajo existente?

Sí, en la visualización de dependencias basada en agente puede asociar un área de trabajo existente al proyecto de migración y usarla para la visualización de dependencias. 

## <a name="can-i-export-the-dependency-visualization-report"></a>¿Puedo exportar el informe de visualización de dependencias?

No, no se puede exportar el informe de visualización de dependencias en la visualización basada en agente. Sin embargo, Azure Migrate usa Service Map, y puede usar la [API REST de Service Map](/rest/api/servicemap/machines/listconnections) para obtener las dependencias en formato JSON.

## <a name="can-i-automate-agent-installation"></a>¿Puedo automatizar la instalación de los agentes?

Para la visualización de dependencias basada en agente:

- Use un [script para instalar el agente de dependencias](../azure-monitor/vm/vminsights-enable-hybrid.md#dependency-agent).
- En el caso de MMA, [use la línea de comandos o la automatización](../azure-monitor/agents/log-analytics-agent.md#installation-options), o bien use un [script](https://gallery.technet.microsoft.com/scriptcenter/Install-OMS-Agent-with-2c9c99ab).
- Además de los scripts, puede usar herramientas de implementación como Microsoft Endpoint Configuration Manager e [Intigua](https://www.intigua.com/intigua-for-azure-migration) para implementar los agentes.

## <a name="what-operating-systems-does-mma-support"></a>¿Qué sistemas operativos admite MMA?

- Vea la lista de los [sistemas operativos Windows compatibles con MMA](../azure-monitor/agents/log-analytics-agent.md#installation-options).
- Vea la lista de los [sistemas operativos Linux compatibles con MMA](../azure-monitor/agents/log-analytics-agent.md#installation-options).

## <a name="can-i-visualize-dependencies-for-more-than-one-hour"></a>¿Puedo visualizar las dependencias durante más de una hora?

Para la visualización basada en agente, puede visualizar las dependencias durante un máximo de una hora. Puede retroceder hasta un mes a una fecha determinada del historial, pero la duración máxima de la visualización es de una hora. Por ejemplo, puede usar la característica de duración de tiempo en el mapa de dependencias para ver las dependencias de ayer, pero solo puede verlas durante un período de una hora. No obstante, puede usar los registros de Azure Monitor para [consultar los datos de dependencias](./how-to-create-group-machine-dependencies.md) durante un período más largo.

Para la visualización sin agente, puede ver el mapa de dependencias de un solo servidor durante un plazo de entre una hora y 30 días.

## <a name="can-i-visualize-dependencies-for-groups-of-more-than-10-servers"></a>¿Puedo visualizar dependencias de grupos de más de 10 servidores?

Puede [visualizar las dependencias](./how-to-create-a-group.md#refine-a-group-with-dependency-mapping) de grupos que tengan un máximo de 10 servidores. Si tiene un grupo con más de 10 servidores, se recomienda dividirlo en grupos más pequeños y luego visualizar las dependencias.

## <a name="next-steps"></a>Pasos siguientes

Consulte la sección [Información general de Azure Migrate](migrate-services-overview.md).