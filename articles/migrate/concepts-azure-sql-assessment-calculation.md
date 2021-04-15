---
title: Evaluaciones de Azure SQL en la herramienta de detección y evaluación de Azure Migrate
description: Obtenga información sobre las evaluaciones de Azure SQL en la herramienta de detección y evaluación de Azure Migrate
author: rashi-ms
ms.author: rajosh
ms.topic: conceptual
ms.date: 02/07/2021
ms.openlocfilehash: a22fa184f91cb409f7a4d7795a4bc34bdd83e598
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106077812"
---
# <a name="assessment-overview-migrate-to-azure-sql"></a>Información general sobre la evaluación (migración a Azure SQL)

En este artículo se proporciona información general sobre las evaluaciones para migrar las instancias locales de SQL Server desde un entorno de VMware a bases de datos o instancias administradas de Azure SQL mediante la [herramienta de detección y evaluación de Azure Migrate](./migrate-services-overview.md#azure-migrate-discovery-and-assessment-tool).

## <a name="whats-an-assessment"></a>¿Qué es una evaluación?
Una evaluación con la herramienta de detección y evaluación es una instantánea en un momento dado de los datos, mide la preparación y estima el efecto de la migración de servidores locales a Azure.

## <a name="types-of-assessments"></a>Tipos de evaluaciones

Existen tres tipos de evaluaciones que puede crear con la herramienta de detección y evaluación de Azure Migrate.

**Tipo de evaluación** | **Detalles**
--- | --- 
**MV de Azure** | Evaluaciones para la migración de los servidores locales a máquinas virtuales de Azure. <br/><br/> Con este tipo de evaluación, puede evaluar los servidores locales en los entornos de [VMware](how-to-set-up-appliance-vmware.md) e [Hyper-V](how-to-set-up-appliance-hyper-v.md) y los [servidores físicos](how-to-set-up-appliance-physical.md) para la migración a máquinas virtuales de Azure.
**SQL de Azure** | Evaluaciones para migrar las instancias locales de SQL Server desde su entorno de VMware a Azure SQL Database o a Azure SQL Managed Instance.
**Azure VMware Solution (AVS)** | Evaluaciones para la migración de los servidores locales a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). <br/><br/> Puede evaluar las [máquinas virtuales de VMware](how-to-set-up-appliance-vmware.md) locales para la migración a Azure VMware Solution (AVS) con este tipo de evaluación. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

> [!NOTE]
> Si el número de evaluaciones de la máquina virtual de Azure o de AVS no es correcto en la herramienta de detección y evaluación, haga clic en el número total de evaluaciones para desplazarse a todas ellas y volver a calcularlas. La herramienta de detección y evaluación mostrará el recuento correcto de ese tipo de evaluación. 

Una evaluación de Azure SQL proporciona un criterio de dimensionamiento:

**Criterio de tamaño** | **Detalles** | **Data**
--- | --- | ---
**Basada en el rendimiento** | Evaluaciones que realizan recomendaciones basadas en datos de rendimiento recopilados | La configuración de Azure SQL se basa en los datos de rendimiento de las instancias y bases de datos de SQL, lo que incluye el uso de CPU, el uso de memoria, la IOPS (archivos de datos y de registro), el rendimiento y la latencia de las operaciones de E/S.

## <a name="how-do-i-assess-my-on-premises-sql-servers"></a>¿Cómo evalúo mis instancias locales de SQL Server?

Puede evaluar las instancias locales de SQL Server mediante los datos de configuración y de uso recopilados por un dispositivo ligero de Azure Migrate. El dispositivo detecta las instancias y bases de datos locales de SQL Server y envía los datos de configuración y rendimiento a Azure Migrate. [Más información](how-to-set-up-appliance-vmware.md)

## <a name="how-do-i-assess-with-the-appliance"></a>¿Cómo se evalúa con el dispositivo?
Haga lo siguiente si va a implementar un dispositivo de Azure Migrate para detectar servidores locales:
1.  Configure Azure y el entorno local para que funcionen con Azure Migrate.
2.  Para su primera evaluación, cree un proyecto de Azure Migrate y agregue la herramienta Azure Migrate: detección y evaluación.
3.  Implemente un dispositivo de Azure Migrate ligero. El dispositivo detecta los servidores locales de forma continuada y envía los datos de configuración y rendimiento a Azure Migrate. Implemente el dispositivo como una máquina virtual o un servidor físico. No es necesario instalar nada en los servidores que quiera evaluar.

Una vez que el dispositivo inicia la detección, puede reunir los servidores que quiera evaluar en un grupo y ejecutar una evaluación del grupo con el tipo de evaluación de **Azure SQL**.

Siga nuestro tutorial para la evaluación de [instancias de SQL Server](tutorial-assess-sql.md) para probar estos pasos.

## <a name="how-does-the-appliance-calculate-performance-data-for-sql-instances-and-databases"></a>¿Cómo calcula el dispositivo los datos de rendimiento de las instancias y bases de datos de SQL?

El dispositivo recopila los datos de rendimiento de la configuración del proceso con estos pasos:
1. El dispositivo recopila un punto de ejemplo en tiempo real. Para instancias de SQL Server, se recopila un punto de muestra cada 30 segundos.
2. El dispositivo agrega los puntos de datos de muestra recopilados cada 30 segundos durante 10 minutos. Para crear el punto de datos, el dispositivo selecciona los valores máximos de todos los ejemplos. Envía el máximo, el promedio y la varianza de cada contador a Azure.
3. Azure Migrate almacena todos los puntos de datos de 10 minutos del último mes.
4. Al crear una evaluación, Azure Migrate identifica el punto de datos adecuado que se va a usar para elegir el tamaño adecuado. La identificación se basa en los valores de percentil de historial de rendimiento y uso de percentil.
    - Por ejemplo, si el historial de rendimiento es de una semana y el uso de percentil es el percentil 95, la evaluación ordena los puntos de muestra de 10 minutos de la última semana. Los clasifica en orden ascendente y selecciona el valor de percentil 95 para la elección del tamaño adecuado.
    - El valor del percentil 95 garantiza que se van a omitir los valores atípicos, que podrían incluirse si eligiera el percentil 99.
    - Si quiere elegir el uso máximo en el período y no quiere omitir ningún valor atípico, seleccione el percentil 99 como uso de percentil.
5. Este valor se multiplica por el factor de confort para obtener los datos de uso efectivos del rendimiento para estas métricas que recopila el dispositivo:
    - Uso de la CPU (%)
    - Uso de memoria (%)
    - E/S de lectura y E/S de escritura (archivos de datos y de registro)
    - MB/s de lectura y MB/s de escritura (rendimiento)
    - Latencia de operaciones de E/S

## <a name="what-properties-are-used-to-create-and-customize-an-azure-sql-assessment"></a>¿Qué propiedades se usan para crear y personalizar una evaluación de Azure SQL?

En las propiedades de evaluación de Azure SQL se incluye lo siguiente:

**Propiedad** | **Detalles**
--- | ---
**Ubicación de destino** | La región de Azure a la que quiere realizar la migración. La configuración de Azure SQL y los costos recomendados dependen de la ubicación que especifique.
**Tipo de implementación de destino** | El tipo de implementación de destino en el que desea ejecutar la evaluación: <br/><br/> Seleccione **Opción recomendada** si desea que Azure Migrate evalúe el grado de preparación de los servidores de SQL para su migración a Azure SQL MI y Azure SQL DB, y recomiende la opción de implementación de destino, el nivel de destino, la configuración de Azure SQL y las estimaciones mensuales que mejor se ajusten.<br/><br/>Seleccione **Azure SQL DB** si quiere evaluar las instancias de SQL Server para la migración exclusivamente a instancias de Azure SQL Database y revisar el nivel de destino, la configuración de Azure SQL DB y las estimaciones mensuales.<br/><br/>Seleccione **Azure SQL MI** si quiere evaluar las instancias de SQL Server para la migración exclusivamente a instancias de Azure SQL Database y revisar el nivel de destino, la configuración de Azure SQL MI y las estimaciones mensuales.
**Capacidad reservada** | Especifica la capacidad reservada para que se tengan en cuenta las estimaciones de los costos en la evaluación.<br/><br/> Si selecciona una opción de capacidad reservada, no podrá especificar "Descuento (%)".
**Criterio de tamaño** | Esta propiedad se usa para ajustar el tamaño correcto de la configuración de Azure SQL. <br/><br/> Su valor predeterminado es **Basado en el rendimiento**, lo que significa que la evaluación recopilará las métricas de rendimiento de las instancias y bases de datos de SQL Server para recomendar el tamaño óptimo de Azure SQL Managed Instance o para una recomendación de configuración o de nivel de Azure SQL Database.
**Historial de rendimiento** | El historial de rendimiento especifica la duración que se usa cuando se evalúan los datos de rendimiento.
**Uso de percentil** | El uso de percentil especifica el valor de percentil de la muestra de rendimiento que se usa para elegir el tamaño adecuado.
**Factor de confort** | Búfer usado durante la evaluación. Tiene en cuenta problemas como el uso estacional, el historial de rendimiento corto y los posibles aumentos en el uso futuro.<br/><br/> Por ejemplo, una instancia de 10 núcleos con un uso del 20 % normalmente genera una instancia de dos núcleos. Con un factor de confort de 2.0, el resultado es una instancia de cuatro núcleos.
**Programa de ofertas o licencias** | La [oferta de Azure](https://azure.microsoft.com/support/legal/offer-details/) en la que está inscrito. En la actualidad, solo puede elegir entre Pago por uso y Desarrollo/pruebas - Pago por uso. Tenga en cuenta que puede beneficiarse de descuentos adicionales combinando capacidad reservada y la Ventaja híbrida de Azure con la oferta de pago por uso.
**Nivel de servicio** | La opción de nivel de servicio más adecuada para satisfacer sus necesidades de negocio y realizar la migración a Azure SQL Database o a Azure SQL Managed Instance:<br/><br/>**Opción recomendada** si desea que Azure Migrate recomiende el nivel de servicio que se ajuste mejor a los servidores. Puede ser Uso general o Crucial para la empresa. <br/><br/> **De uso general** si desea utilizar una configuración de Azure SQL diseñada para cargas de trabajo orientadas al presupuestos. [Más información](../azure-sql/database/service-tier-general-purpose.md) <br/><br/> **Crítico para la empresa** si desea utilizar una configuración de Azure SQL diseñada para cargas de trabajo de baja latencia con alta resistencia a errores y conmutaciones por error rápidas. [Más información](../azure-sql/database/service-tier-business-critical.md)
**Moneda** | Moneda de facturación de la cuenta.
**Descuento (%)** | Cualquier descuento específico de la suscripción que recibe además de la oferta de Azure. La configuración predeterminada es 0 %.
**Ventaja híbrida de Azure** | Especifica si ya tiene una licencia de SQL Server. <br/><br/> Si la tiene y está incluida en las suscripciones activas de Software Assurance para SQL Server, podrá solicitar la Ventaja híbrida de Azure cuando traiga sus licencias a Azure.

[Revise los procedimientos recomendados](best-practices-assessment.md) para crear una evaluación con Azure Migrate.

## <a name="calculate-readiness"></a>Cálculo de la preparación

> [!NOTE]
Esta evaluación solo incluye las bases de datos que están en estado en línea. En caso de que la base de datos esté en cualquier otro estado, la evaluación omite el cálculo de la preparación, el tamaño y el costo de esas bases de datos. En caso de que desee evaluar estas bases de datos, cambie el estado de la base de datos y vuelva a calcular la evaluación en algún momento.

### <a name="azure-sql-readiness"></a>Preparación para Azure SQL

La preparación para Azure SQL de instancias y bases de datos de SQL se basa en una comprobación de la compatibilidad de las características con Azure SQL Database y Azure SQL Managed Instance:
1. La evaluación de Azure SQL tiene en cuenta las características de la instancia de SQL Server que las cargas de trabajo de SQL Server de origen (trabajos del Agente SQL, servidores vinculados, etc.) y los esquemas de las bases de datos de usuario (tablas, vistas, desencadenadores, procedimientos almacenados, etc.) usan actualmente para identificar problemas de compatibilidad.
1. Si no se encuentra ningún problema de compatibilidad, la preparación se marca como **Preparado** para el tipo de implementación de destino (Azure SQL Database o Azure SQL Managed Instance).
1. Si hay problemas de compatibilidad que no son críticos, como características degradadas o no compatibles que no bloquean la migración a un tipo de implementación de destino específico, la preparación se marca como **Preparado** (icono de información azul y con hipervínculo) con los detalles de **advertencia** y la guía de corrección recomendada.
1. Si hay algún problema de compatibilidad que puede bloquear la migración a un tipo de implementación de destino específico, la preparación se marca como **No preparado** con los detalles del **problema** y la guía de corrección recomendada.
    - Incluso si hay una sola base de datos en una instancia de SQL que no está preparada para un tipo de implementación de destino determinado, la instancia se marca como **No preparado** para ese tipo de implementación.
1. Si la detección aún está en curso o si hay problemas de detección para una instancia o base de datos de SQL, la preparación se marca como **Desconocido**, ya que la evaluación no pudo calcular la preparación de esa instancia de SQL.

### <a name="recommended-deployment-type"></a>Tipo de implementación recomendado

Si selecciona el tipo de implementación de destino como **Recomendado** en las propiedades de evaluación de Azure SQL, Azure Migrate recomienda un tipo de implementación de Azure SQL que sea compatible con su instancia de SQL. La migración a un destino recomendado por Microsoft reduce el esfuerzo general de la migración. 

#### <a name="recommended-deployment-type-based-on-azure-sql-readiness"></a>Tipo de implementación recomendado basado en la preparación para Azure SQL

 **Preparación para Azure SQL DB** | **Preparación para Azure SQL MI** | **Tipo de implementación recomendado** | **¿Se han calculado los costos estimados y la configuración de Azure SQL?**
 --- | --- | --- | --- |
 Ready | Ready | Azure SQL DB o <br/>Azure SQL MI  | Sí
 Ready | No está listo o<br/> Unknown | Azure SQL DB | Sí
 No está listo o<br/>Unknown | Ready | Azure SQL MI  | Sí
 No está listo | No está listo | Potencialmente listo para VM de Azure | No
 No está listo o<br/>Unknown | No está listo o<br/>Desconocido | Desconocido | No

> [!NOTE]
> Si el tipo de implementación recomendado se selecciona como **Recomendado** en las propiedades de evaluación y si la instancia SQL Server de origen es adecuada para la base de datos única de Azure SQL DB y Azure SQL Managed Instance, la evaluación recomienda una opción específica que optimice el costo y se ajuste a los límites de tamaño y rendimiento.

#### <a name="potentially-ready-for-azure-vm"></a>Potencialmente listo para VM de Azure

Si la instancia de SQL no está lista para Azure SQL Database y Azure SQL Managed Instance, el tipo de implementación recomendado se marca como *Potentially ready for Azure VM* (Potencialmente listo para VM de Azure).
- Se recomienda que el usuario cree una evaluación en Azure Migrate con el tipo de evaluación como "Azure VM" para determinar si el servidor en el que se ejecuta la instancia está listo para migrarse a una máquina virtual de Azure en su lugar. Observe lo siguiente:
    - Las evaluaciones de máquinas virtuales de Azure en Azure Migrate siguen actualmente el enfoque de migración mediante lift-and-shift y no tendrán en cuenta las métricas de rendimiento específicas para ejecutar instancias y bases de datos de SQL en la máquina virtual de Azure. 
    - Al ejecutar una evaluación de máquinas virtuales de Azure en un servidor, el tamaño recomendado y las estimaciones de costos serán para todas las instancias que se ejecutan en ese servidor y se pueden migrar a una máquina virtual de Azure mediante la herramienta Server Migration. Antes de migrar, [revise las directrices de rendimiento](../azure-sql/virtual-machines/windows/performance-guidelines-best-practices.md) para SQL Server en máquinas virtuales de Azure.


## <a name="calculate-sizing"></a>Cálculo del dimensionamiento

### <a name="azure-sql-configuration"></a>Configuración de Azure SQL

Una vez que la evaluación determina la preparación y el tipo recomendado de implementación de Azure SQL, calcula un nivel de servicio específico y una configuración de Azure SQL (tamaño de SKU) que pueda alcanzar o superar el rendimiento de la instancia de SQL local:
1. Durante el proceso de detección, Azure Migrate recopila la configuración y el rendimiento de la instancia de SQL que incluye:
    - Núcleos virtuales (asignados) y uso de CPU (%)
        - El uso de CPU para una instancia de SQL es el porcentaje de CPU asignada que la instancia en SQL Server utiliza.
        - El uso de CPU para una base de datos es el porcentaje de CPU asignada que la base de datos en la instancia de SQL utiliza.
    - Memoria (asignada) y uso de memoria (%)
    - E/S de lectura y E/S de escritura (archivos de datos y de registro)
        - Las ES/s de lectura y de escritura en un nivel de instancia de SQL se calculan agregando las ES/s de lectura y de escritura de todas las bases de datos detectadas en esa instancia.
    - MB/s de lectura y MB/s de escritura (rendimiento)
    - Latencia de operaciones de E/S
    - Tamaño total de base de datos y organizaciones de archivos de base de datos
        - El tamaño de la base de datos se calcula agregando todos los archivos de datos y de registro.
1. La evaluación agrega todos los datos de configuración y rendimiento, intenta encontrar la mejor coincidencia entre varios niveles y configuraciones de servicio de Azure SQL y elige una configuración que pueda coincidir o superar los requisitos de rendimiento de la instancia de SQL y optimizar el costo.

### <a name="confidence-ratings"></a>Clasificaciones de confianza 
Cada evaluación de Azure SQL está asociada a una clasificación de confianza. La clasificación oscila entre una (más baja) y cinco estrellas (más alta). La clasificación de confianza sirve de ayuda para calcular la confiabilidad de las recomendaciones de tamaño que proporciona Azure Migrate.
- La clasificación de confianza se asigna a una evaluación. Esta clasificación se basa en la disponibilidad de los puntos de datos necesarios para calcular tal evaluación.
- Para el dimensionamiento basado en el rendimiento, la evaluación recopila los datos de rendimiento de todas las instancias y bases de datos de SQL, entre las que se incluyen:
    - Uso de la CPU (%)
    - Uso de memoria (%)
    - E/S de lectura y E/S de escritura (archivos de datos y de registro)
    - MB/s de lectura y MB/s de escritura (rendimiento)
    - Latencia de operaciones de E/S
    
Si alguna de las cifras de uso anteriores no está disponible, las recomendaciones de tamaño podrían no ser confiables.
En esta tabla se muestran las clasificaciones de confianza de la evaluación, que dependen del porcentaje de puntos de datos disponibles:

**Disponibilidad del punto de datos** | **Clasificación de confianza**
--- | ---
0 % - 20 % | 1 estrella
21 % - 40 % | 2 estrellas
41 % - 60 % | 3 estrellas
61 % - 80 % | 4 estrellas
81 % - 100 % | 5 estrellas

#### <a name="low-confidence-ratings"></a>Clasificaciones de confianza bajas
Estos son algunos de los motivos por los que una evaluación puede obtener una clasificación de confianza baja:
- No ha generado un perfil de su entorno durante el tiempo que ha estado creando la evaluación. Por ejemplo, si crea la evaluación con la duración de rendimiento establecida en un día, debe esperar al menos un día después de empezar la detección para que se recopilen todos los puntos de datos.
- La evaluación no puede recopilar los datos de rendimiento de algunos o de todos los servidores en el período de evaluación. Para obtener una clasificación de confianza alta, asegúrese de que:
    - Los servidores estén activados mientras dure la evaluación.
    - Se permiten las conexiones salientes en los puertos 443.
    - Si el estado de la conexión de Azure Migrate del Agente SQL en Azure Migrate es "Conectado" y compruebe el último latido. 
    - Si el estado de conexión de Azure Migrate para todas las instancias de SQL es "Conectado" en la hoja de la instancia de SQL detectada.

    "Recalcule" la evaluación para reflejar los cambios más recientes en la clasificación de confianza.
- Se crearon algunas bases de datos o instancias durante el período en el que se calculó la evaluación. Por ejemplo, suponga que creó una evaluación para el historial de rendimiento del último mes, pero algunas bases de datos se crearon hace solo una semana. En este caso, los datos de rendimiento de los nuevos servidores no estarán disponibles para todo el período y la clasificación de confianza será baja.

> [!NOTE]
> Dado que las evaluaciones de Azure SQL se basan en el rendimiento, si la clasificación de confianza de una evaluación no llega a las cinco estrellas, se recomienda que espere al menos un día para que el dispositivo genere el perfil del entorno y luego vuelva a calcular la evaluación. De lo contrario, el tamaño basado en el rendimiento podría ser poco confiable.

## <a name="calculate-monthly-costs"></a>Cálculo de los costos mensuales
Una vez completadas las recomendaciones de dimensionamiento, la evaluación de Azure SQL calcula los costos de proceso y almacenamiento para las configuraciones de Azure SQL recomendadas mediante una API de precios interna. Para calcular el costo de proceso mensual total, suma el costo de proceso y almacenamiento de todas las instancias. 
### <a name="compute-cost"></a>Costo del proceso
- Para calcular el costo del proceso de una configuración de Azure SQL, la evaluación tiene en cuenta las siguientes propiedades:
    - Ventaja híbrida de Azure para licencias de SQL
    - Capacidad reservada
    - Ubicación de destino de Azure
    - Moneda
    - Programa de ofertas o licencias
    - Descuento (%)

### <a name="storage-cost"></a>Coste del almacenamiento
- Las estimaciones del costo del almacenamiento solo incluyen archivos de datos y no archivos de registro. 
- Para calcular el costo del almacenamiento de una configuración de Azure SQL, la evaluación tiene en cuenta las siguientes propiedades:
    - Ubicación de destino de Azure
    - Moneda
    - Programa de ofertas o licencias
    - Descuento (%)
- El costo del almacenamiento de copia de seguridad no se incluye en la evaluación.
- **Azure SQL Database**
    - Se agrega costo de almacenamiento de 5 GB como mínimo en la estimación del costo y se agrega un costo de almacenamiento adicional para el almacenamiento en incrementos de 1 GB. [Más información](https://azure.microsoft.com/pricing/details/sql-database/single/)
- **Instancia administrada de Azure SQL**
    - No se agrega ningún costo de almacenamiento para el almacenamiento de los primeros 32 GB, por instancia y mes, y se agrega un costo de almacenamiento adicional para el almacenamiento en incrementos de 32 GB. [Más información](https://azure.microsoft.com/pricing/details/azure-sql/sql-managed-instance/single/)
        
## <a name="next-steps"></a>Pasos siguientes
- [Revise](best-practices-assessment.md) los procedimientos recomendados para crear evaluaciones. 
- Obtenga información sobre cómo ejecutar una [evaluación de Azure SQL](how-to-create-azure-sql-assessment.md).