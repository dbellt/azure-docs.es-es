---
title: Cálculos de evaluación de Azure VMware Solution en Azure Migrate | Microsoft Docs
description: En este artículo se proporciona una introducción a los cálculos de evaluación de Azure VMware Solution en el servicio Azure Migrate.
author: rashi-ms
ms.author: rajosh
ms.manager: abhemraj
ms.topic: conceptual
ms.date: 06/25/2020
ms.openlocfilehash: b55f0562ae08a6e5a6b38bd0b6fc9f781b94ab64
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106078696"
---
# <a name="assessment-overview-migrate-to-azure-vmware-solution"></a>Introducción a la evaluación (migración a Azure VMware Solution)

[Azure Migrate](migrate-services-overview.md) proporciona un centro principal para realizar el seguimiento de la detección, evaluación y migración de las aplicaciones y cargas de trabajo locales. También realiza un seguimiento de las instancias de nube privada y pública en Azure. El centro proporciona herramientas de Azure Migrate para la evaluación y la migración, así como ofertas de proveedores de software independientes (ISV).

La herramienta de detección y evaluación de Azure Migrate evalúa los servidores locales para la migración a máquinas virtuales de IaaS de Azure y a instancias de Azure VMware Solution (AVS). En este artículo se proporciona información sobre cómo se calculan las evaluaciones de Azure VMware Solution (AVS).

> [!NOTE]
> La evaluación de Azure VMware Solution (AVS) solo se puede crear para máquinas virtuales con VMware.

## <a name="types-of-assessments"></a>Tipos de evaluaciones

Las evaluaciones que crea con Azure Migrate son una instantánea de los datos de un momento dado. Con Azure Migrate: Server Assessment se pueden crear dos

**Tipo de evaluación** | **Detalles**
--- | --- 
**MV de Azure** | Evaluaciones para la migración de los servidores locales a máquinas virtuales de Azure. Con este tipo de evaluación, puede evaluar los servidores locales en los entornos de [VMware](how-to-set-up-appliance-vmware.md) e [Hyper-V](how-to-set-up-appliance-hyper-v.md) y los [servidores físicos](how-to-set-up-appliance-physical.md) para la migración a máquinas virtuales de Azure.
**SQL de Azure** | Evaluaciones para migrar las instancias locales de SQL Server desde su entorno de VMware a Azure SQL Database o a Azure SQL Managed Instance.
**Azure VMware Solution (AVS)** | Evaluaciones para la migración de los servidores locales a [Azure VMware Solution (AVS)](../azure-vmware/introduction.md). Puede evaluar las [máquinas virtuales de VMware](how-to-set-up-appliance-vmware.md) locales para la migración a Azure VMware Solution (AVS) con este tipo de evaluación. [Más información](concepts-azure-vmware-solution-assessment-calculation.md)

> [!NOTE]
> Si el número de evaluaciones de la máquina virtual de Azure o de AVS no es correcto en la herramienta de detección y evaluación, haga clic en el número total de evaluaciones para desplazarse a todas ellas y volver a calcularlas. La herramienta de detección y evaluación mostrará el recuento correcto de ese tipo de evaluación. 

Una evaluación de Azure VMware Solution (AVS) proporciona dos opciones de criterios de dimensionamiento:

| **Valoración** | **Detalles** | **Data** |
| - | - | - |
| **Basada en el rendimiento** | Evaluaciones basadas en los datos de rendimiento recopilados de las máquinas virtuales locales. | **Tamaño del nodo recomendado**: se basa en los datos de uso de la CPU y de la memoria junto con el tipo de nodo, el tipo de almacenamiento y la configuración de FTT seleccionados para la evaluación. |
| **Como local** | Evaluaciones que se basan en el tamaño local. | **Tamaño del nodo recomendado**: se basa en el tamaño de la máquina virtual local junto con el tipo de nodo, el tipo de almacenamiento y la configuración de FTT seleccionados para la evaluación. |

## <a name="how-do-i-run-an-assessment"></a>¿Cómo se ejecuta una evaluación?

Existen dos formas de realizar una evaluación:

- Evaluar las máquinas con los metadatos de servidor recopilados por un dispositivo de Azure Migrate ligero. El dispositivo detecta servidores locales. A continuación, envía continuamente los metadatos y los datos de rendimiento de las máquinas a Azure Migrate. Esto permite más precisión.
- Evaluar los servidores mediante los metadatos de servidor importados en un formato de valores separados por comas (CSV).

## <a name="how-do-i-assess-with-the-appliance"></a>¿Cómo se evalúa con el dispositivo?

Haga lo siguiente si va a implementar un dispositivo de Azure Migrate para detectar servidores locales:

1. Configure Azure y el entorno local para que funcionen con Azure Migrate.
2. Para realizar la primera evaluación, cree un proyecto de Azure y, luego, agregue la herramienta Discovery and Assessment.
3. Implemente un dispositivo de Azure Migrate ligero. El dispositivo detecta continuamente los servidores locales y envía los metadatos y los datos de rendimiento de los servidores a Azure. Implemente el dispositivo como una máquina virtual. No es necesario instalar nada en los servidores que quiera evaluar.

Cuando el dispositivo inicia la detección de los servidores, puede reunir aquellos que quiera evaluar en un grupo y realizar una evaluación del grupo con el tipo de evaluación de **Azure VMware Solution (AVS)** .

Cree la primera evaluación de Azure VMware Solution (AVS) mediante los pasos que se indican [aquí](how-to-create-azure-vmware-solution-assessment.md).

## <a name="how-do-i-assess-with-imported-data"></a>¿Cómo se evalúa con datos importados?

Si va a evaluar los servidores con un archivo CSV, no necesita un dispositivo. En su lugar, siga estos pasos:

1. Configure Azure para que funcione con Azure Migrate.
2. Para realizar la primera evaluación, cree un proyecto de Azure y, luego, agregue la herramienta Discovery and Assessment.
3. Descargue una plantilla CSV y agregue los datos de los servidores.
4. Importe la plantilla en Azure Migrate.
5. Detecte los servidores agregados con la importación, reúnalos en un grupo y realice una evaluación del grupo con el tipo de evaluación de **Azure VMware Solution (AVS)** .

## <a name="what-data-does-the-appliance-collect"></a>¿Qué datos recopila el dispositivo?

Si usa el dispositivo Azure Migrate para la evaluación, obtenga información sobre los metadatos y los datos de rendimiento que se recopilan para [VMware](migrate-appliance.md#collected-data---vmware).

## <a name="how-does-the-appliance-calculate-performance-data"></a>¿Cómo calcula el dispositivo los datos de rendimiento?

Si usa el dispositivo para la detección, los datos de rendimiento de la configuración de proceso se recopilan de la siguiente manera:

1. El dispositivo recopila un punto de ejemplo en tiempo real.

   - **VM de VMware**: se recopila un punto de ejemplo cada 20 segundos.
2. El dispositivo combina los puntos de ejemplo para crear un único punto de datos cada 10 minutos. Para crear el punto de datos, el dispositivo selecciona los valores máximos de todos los ejemplos. A continuación, envía el punto de datos a Azure.
3. Azure Migrate almacena todos los puntos de datos de 10 minutos del último mes.
4. Al crear una evaluación, esta identifica el punto de datos adecuado que se va a usar para elegir el tamaño adecuado. La identificación se basa en los valores de percentil de *historial de rendimiento* y *uso de percentil*.

   - Por ejemplo, si el historial de rendimiento es de una semana y el uso de percentil es el percentil 95, la evaluación ordena los puntos de muestra de 10 minutos de la última semana. Los clasifica en orden ascendente y selecciona el valor de percentil 95 para la elección del tamaño adecuado.
   - El valor del percentil 95 garantiza que se van a omitir los valores atípicos, que podrían incluirse si eligiera el percentil 99.
   - Si quiere elegir el uso máximo en el período y no quiere omitir ningún valor atípico, seleccione el percentil 99 como uso de percentil.
5. Este valor se multiplica por el factor de confort para obtener los datos de uso efectivos del rendimiento para estas métricas que recopila el dispositivo:

   - Uso de CPU
   - Uso de RAM

Los siguientes datos de rendimiento se recopilan, pero no se usan en las recomendaciones de ajuste de tamaño de las valoraciones de AVS:

- Los datos de IOPS y rendimiento de cada disco conectado a la máquina virtual.
- La entrada o la salida de red de cada adaptador de red conectado a una máquina virtual.

## <a name="how-are-avs-assessments-calculated"></a>¿Cómo se calculan las evaluaciones de AVS?

La evaluación de AVS utiliza los datos de rendimiento y los metadatos de los servidores locales para calcular las evaluaciones. Si implementa el dispositivo de Azure Migrate, la evaluación usa los datos que este recopila. Sin embargo, si ejecuta una evaluación importada mediante un archivo CSV, se proporcionan los metadatos para el cálculo.

Los cálculos se producen en tres fases:

1. **Cálculo de la preparación de Azure VMware Solution (AVS)** : para ver si las máquinas virtuales locales son adecuadas para la migración a Azure VMware Solution (AVS).
2. **Cálculo del número de nodos de AVS y el uso de estos**: el número estimado de nodos de AVS necesarios para ejecutar las máquinas virtuales de VMware y el uso previsto de la CPU, la memoria y el almacenamiento en todos los nodos.
3. **Estimación del costo mensual**: los costos mensuales estimados de todos los nodos de Azure VMware Solution que ejecutan las máquinas virtuales locales.

Los cálculos se encuentran en el orden anterior. Un servidor solo se mueve a una fase posterior si supera la anterior. Por ejemplo, si un servidor no supera la fase de preparación de AVS, se marca como no adecuado para Azure. No se realizan los cálculos de dimensionamiento y costo para ese servidor.

## <a name="whats-in-an-azure-vmware-solution-avs-assessment"></a>¿En qué consiste una evaluación de Azure VMware Solution?

Esto es lo que se incluye en una evaluación de AVS:

| **Propiedad** | **Detalles** |
| - | - |
| **Ubicación de destino** | Especifica la ubicación de la nube privada de AVS a la que se desea realizar la migración. |
| **Tipo de almacenamiento** | Especifica el motor de almacenamiento que se va a utilizar en AVS. AVS actualmente solo admite vSAN como tipo de almacenamiento predeterminado, pero habrá más opciones de almacenamiento en función del mapa de ruta. |
| **Instancias reservadas (RI)** | Esta propiedad le ayuda a especificar instancias reservadas en AVS si se compra y el término de la instancia reservada. Se usa para calcular los costos. |
| **Tipo de nodo** | Especifica el [tipo de nodo de AVS](../azure-vmware/concepts-private-clouds-clusters.md) que se usa en Azure. El tipo de nodo predeterminado es AV36. Es posible que haya más tipos de nodos disponibles en el futuro.  Azure Migrate le recomendará un número necesario de nodos para las máquinas virtuales que se vayan a migrar a AVS. |
| **Configuración de errores tolerables, nivel de RAID** | Especifica la combinación válida de errores tolerables y combinaciones RAID. La opción de errores tolerables seleccionada junto con el requisito de disco de la máquina virtual local determinará el almacenamiento de vSAN total que se requiere en AVS. El almacenamiento total disponible después de los cálculos también incluye: a) un espacio reservado para los objetos de administración como vCenter y b) el 25 % de la demora de almacenamiento necesaria para las operaciones de vSAN. |
| **Criterio de ajuste de tamaño** | Establece los criterios que se van a utilizar para determinar los requisitos de memoria, CPU y almacenamiento para los nodos de AVS. Puede optar por un dimensionamiento *basado en el rendimiento* o *como local*, sin tener en cuenta el historial de rendimiento. Para simplemente migrar mediante "lift-and-shift", elija "como local". Para obtener el dimensionamiento basado en el uso, elija "Basado en el rendimiento". |
| **Historial de rendimiento** | Establece la duración que se debe tener en cuenta para evaluar los datos de rendimiento de los servidores. Esta propiedad solo se aplica cuando el criterio de dimensionamiento está *basado en el rendimiento*. |
| **Uso de percentil** | Especifica el valor de percentil del ejemplo de rendimiento establecido para determinar el tamaño adecuado. Esta propiedad solo es aplicable cuando el ajuste de tamaño se basa en el rendimiento. |
| **Factor de confort** | Azure Migrate tiene en cuenta un búfer (factor de confort) durante la evaluación. Dicho búfer se aplica además de los datos de uso del servidor en las máquinas virtuales (CPU, memoria y disco). El factor de confort se tiene en cuenta en problemas como el uso estacional, un historial de rendimiento corto y los posibles aumentos en el uso futuro. Por ejemplo, una máquina virtual de 10 núcleos con un uso del 20 % normalmente genera una máquina virtual de 2 núcleos. Sin embargo, con un factor de confort de 2.0 x, el resultado es una máquina virtual de 4 núcleos. |
| **Oferta** | Muestra la [oferta de Azure](https://azure.microsoft.com/support/legal/offer-details/) en la que está inscrito. Azure Migrate calcula el costo en consecuencia. |
| **Moneda** | Muestra la moneda de facturación de la cuenta. |
| **Descuento (%)** | Muestra cualquier descuento específico de la suscripción que recibe además de la oferta de Azure. La configuración predeterminada es 0 %. |
| **Ventaja híbrida de Azure** | Especifique si dispone de Software Assurance y tiene derecho a la [Ventaja híbrida de Azure](https://azure.microsoft.com/pricing/hybrid-use-benefit/). Aunque no tiene ningún impacto en los precios de Azure VMware Solution debido a que el precio se basa en los nodos, los clientes pueden seguir utilizando las licencias de sistema operativo o SQL locales (basadas en Microsoft) en AVS mediante el programa Ventajas híbridas de Azure. Otros proveedores de sistemas operativos de software tendrán que proporcionar sus propios términos de licencia como, por ejemplo, RHEL. |
| **Suscripción excesiva de CPU virtual** | Especifica la relación entre el número de núcleos virtuales vinculados a un núcleo físico del nodo de AVS. El valor predeterminado en los cálculos es 4 vCPU: 1 núcleo físico en AVS. Los usuarios de la API pueden establecer este valor como un entero. Tenga en cuenta que la suscripción excesiva de vCPU > 4:1 puede afectar a las cargas de trabajo según su uso de la CPU. Al dimensionar, siempre asumimos el uso del 100 % de los núcleos elegidos. |
| **Factor de sobreconfirmación de memoria** | Especifica la proporción de memoria sobre la confirmación en el clúster. Un valor de 1 representa el 100 % del uso de memoria; 0,5 el 50 % y 2 sería usar el 200 % de la memoria disponible. Solo puede agregar valores del 0,5 al 10 con hasta una posición decimal. |
| **Factor de compresión y desduplicación** | Especifica el factor de compresión y desduplicación previsto para las cargas de trabajo. El valor real se puede obtener del vSAN local o de la configuración de almacenamiento. Este varía por carga de trabajo. Un valor de 3 significaría el triple, por lo que para un disco de 300 GB solo se utilizarían 100 GB de almacenamiento. Un valor de 1 significa que no hay eliminación de duplicados ni compresión. Solo puede agregar valores del 1 al 10 con hasta una posición decimal. |

## <a name="azure-vmware-solution-avs-suitability-analysis"></a>Análisis de idoneidad para Azure VMware Solution

Las evaluaciones de AVS evalúan cada máquina virtual local para comprobar su idoneidad para AVS mediante la revisión de las propiedades del servidor. También asigna cada servidor evaluado a una de las siguientes categorías de idoneidad:

- **Listo para AVS**: el servidor se puede migrar como está a Azure AVS sin realizar ningún cambio. Se iniciará en AVS con soporte técnico de AVS completo.
- **Preparado con condiciones**: Es posible que la máquina virtual tenga problemas de compatibilidad con la versión actual de vSphere y que, además, requiera herramientas de VMware y otras opciones de configuración antes de que pueda conseguirse la funcionalidad completa de la máquina virtual en AVS.
- **No está listo para AVS**: la máquina virtual no se iniciará en AVS. Por ejemplo, si la máquina virtual de VMware local tiene un dispositivo externo conectado, como un CD-ROM, se producirá un error en la operación de VMware VMotion (si se utiliza este).
- **Preparación desconocida**: Azure Migrate no ha podido encontrar la preparación del servidor debido a una recopilación insuficiente de metadatos en el entorno local.

La evaluación revisa las propiedades del servidor para determinar la preparación para Azure del servidor local.

### <a name="server-properties"></a>Propiedades del servidor

La evaluación revisa la siguiente propiedad de la máquina virtual local para determinar si se puede ejecutar en Azure VMware Solution.

| **Propiedad** | **Detalles** | **Estado de preparación de AVS** |
| - | - | - |
| **Protocolo de Internet** | Actualmente, Azure no admite direcciones de Internet IPv6 de un extremo a otro. Póngase en contacto con el equipo de MSFT AVS GBB local para obtener ayuda sobre las instrucciones de corrección si el servidor se detecta con IPv6. | Protocolo de Internet preparado condicionalmente |

### <a name="guest-operating-system"></a>Sistema operativo invitado

Actualmente, las evaluaciones de AVS no revisan el sistema operativo como parte del análisis de idoneidad. Es probable que todos los sistemas operativos que se ejecutan en máquinas virtuales locales se ejecuten en Azure VMware Solution (AVS).

Junto con las propiedades de la máquina virtual, la evaluación examina el sistema operativo invitado de los servidores para determinar si se puede ejecutar en Azure.

## <a name="sizing"></a>Ajuste de tamaño

Cuando el servidor está marcado como listo para AVS, la evaluación de AVS realiza recomendaciones de dimensionamiento del nodo, lo cual implica la identificación de los requisitos adecuados de la máquina virtual local y la búsqueda del número total de nodos de AVS necesarios. Estas recomendaciones varían en función de las propiedades de evaluación especificadas.

- Si la evaluación utiliza el *dimensionamiento basado en el rendimiento*, Azure Migrate tiene en cuenta el historial de rendimiento del servidor para realizar la recomendación de dimensionamiento adecuada para AVS. Este método es especialmente útil si ha asignado en exceso la máquina virtual local pero el uso es bajo, y desea ajustar adecuadamente el tamaño de la máquina virtual en AVS para ahorrar costos. Este método le ayudará a optimizar los tamaños durante la migración.
- Si no desea tener en cuenta los datos de rendimiento para el ajuste de tamaño de la máquina virtual y desea integrar los servidores locales tal cual en AVS, puede establecer el criterio de dimensionamiento en *Como local*. Esto hará que la evaluación ajuste el tamaño de las máquinas virtuales en función de la configuración local, sin tener en cuenta los datos de uso.

### <a name="ftt-sizing-parameters"></a>Parámetros de ajuste de tamaño de los errores tolerables

El motor de almacenamiento que se usa en AVS es vSAN. Las directivas de almacenamiento de vSAN definen los requisitos de almacenamiento de los servidores. Estas directivas garantizan el nivel de servicio requerido para las máquinas virtuales, ya que determinan cómo se asigna el almacenamiento a la máquina virtual. Estas son las combinaciones FTT-RAID disponibles:

| **Errores tolerables (FTT)** | **Configuración de RAID** | **Hosts mínimos requeridos** | **Consideración de dimensionamiento** |
| - | - | - | - |
| 1 | RAID-1 (Creación de reflejo) | 3 | Una máquina virtual de 100 GB consumiría 200 GB. |
| 1 | RAID-5 (codificación de borrado) | 4 | Una máquina virtual de 100 GB consumiría 133,33 GB. |
| 2 | RAID-1 (Creación de reflejo) | 5 | Una máquina virtual de 100 GB consumiría 300 GB. |
| 2 | RAID-6 (codificación de borrado) | 6 | Una máquina virtual de 100 GB consumiría 150 GB. |
| 3 | RAID-1 (Creación de reflejo) | 7 | Una máquina virtual de 100 GB consumirá 400 GB. |

### <a name="performance-based-sizing"></a>Ajuste de tamaño según el rendimiento

En el caso del dimensionamiento basado en el rendimiento, el dispositivo de Azure Migrate realiza un perfil del entorno local para recopilar datos de rendimiento de la CPU, de la memoria y del disco. Por lo tanto, el dimensionamiento basado en el rendimiento de AVS tendrá en cuenta el espacio en disco asignado y el uso del percentil elegido de la memoria y la CPU. Por ejemplo, si una máquina virtual tiene cuatro núcleos virtuales asignados pero solo usa un 25 %, AVS tendrá un tamaño de un núcleo virtual para esa máquina virtual.

**Pasos de la recopilación de datos de rendimiento:**

1. En el caso de las máquinas virtuales de VMware, el dispositivo de Azure Migrate recopila un punto de ejemplo en tiempo real en intervalos de 20 segundos.
2. El dispositivo acumula los puntos de ejemplo recopilados cada 10 minutos y envía el valor máximo de los últimos 10 minutos a Azure Migrate.
3. Azure Migrate almacena todos los puntos de ejemplo de 10 minutos del último mes. Después, en función de las propiedades de la evaluación especificadas para *Historial de rendimiento* y *Uso del percentil*, identifica el punto de datos adecuado que se debe usar para el dimensionamiento adecuado. Por ejemplo, si el historial de rendimiento está establecido en un día y el uso del percentil es 95, Azure Migrate utiliza los puntos de ejemplo almacenados cada 10 minutos del último día, los clasifica en orden ascendente y selecciona el percentil 95 para el dimensionamiento adecuado.
4. Este valor se multiplica por el factor de confort para obtener los datos de uso efectivos del rendimiento para cada métrica (utilización de CPU y de memoria) que recopila el dispositivo.

Una vez determinado el valor de uso efectivo, el ajuste de tamaño del almacenamiento, la red y el proceso se determinan de la siguiente manera.

**Ajuste de tamaño de almacenamiento**: Azure Migrate usa el espacio de disco de la máquina virtual local total como parámetro de cálculo para determinar los requisitos de almacenamiento de vSAN de AVS además del valor de errores tolerables seleccionado por el cliente. FTT: el número de errores tolerables así como la necesidad de un número mínimo de nodos por opción de FTT determinarán el almacenamiento total de vSAN que se requiere junto con el requisito de disco de máquina virtual. Actualmente no se tiene en cuenta el uso del almacenamiento y la lógica solo examina el almacenamiento asignado por la máquina virtual.

**Dimensionamiento de red**: actualmente, las evaluaciones de AVS no tienen en cuenta ninguna configuración de red.

**Dimensionamiento de proceso**: después de calcular los requisitos de almacenamiento, la evaluación de AVS tiene en cuenta los requisitos de CPU y de memoria para determinar el número de nodos necesarios para AVS según el tipo de nodo.

- Según los criterios de dimensionamiento, la evaluación de AVS examina los datos de la máquina virtual basados en el rendimiento o la configuración de la máquina virtual local. La configuración del factor de confort permite especificar el factor de crecimiento del clúster. El hyperthreading está habilitado de forma predeterminada y, por lo tanto, un nodo de 36 núcleos tendrá 72 núcleos virtuales. Se utilizan cuatro núcleos virtuales por núcleo físico para determinar los umbrales de CPU por clúster mediante el estándar de VMware que no supera el 80 % de uso; el objetivo es permitir el mantenimiento o los errores que se van a controlar sin poner en peligro la disponibilidad del clúster. Actualmente no hay ninguna invalidación disponible para cambiar los valores de suscripción excesiva, aunque es posible que se incluya en versiones futuras.

### <a name="as-on-premises-sizing"></a>Ajuste de tamaño como local

Si usa el *dimensionamiento como local*, la evaluación de AVS no tiene en cuenta el historial de rendimiento de las máquinas virtuales ni los discos. En su lugar, asigna nodos de AVS según el tamaño asignado en el entorno local. El tipo de almacenamiento predeterminado es vSAN en AVS.

[Obtenga más información](./tutorial-assess-vmware-azure-vmware-solution.md#review-an-assessment) sobre cómo revisar una evaluación de Azure VMware Solution.

### <a name="cpu-utilization-on-avs-nodes"></a>Uso de CPU en nodos de AVS

El uso de CPU presupone el uso del 100 % de los núcleos disponibles. Para reducir el número de nodos necesarios, puede aumentar la suscripción de 4:1 a 6:1 en función de las características de la carga de trabajo y de la experiencia local. A diferencia del disco, AVS no impone ningún límite en el uso de la CPU, por lo que los clientes deben asegurarse de que el clúster funciona de forma óptima y, si se requiere una "ejecución activa", se debe ajustar en consecuencia. Para permitir más espacio para el crecimiento, reduzca la sobresuscripción o aumente el valor de factor de crecimiento.

El uso de CPU también tiene en cuenta la sobrecarga de administración de vCenter, del administrador de NSX y de otros recursos más pequeños.

### <a name="memory-utilization-on-avs-nodes"></a>Uso de memoria en nodos de AVS

El uso de memoria muestra la memoria total de todos los nodos y los requisitos del servidor o las cargas de trabajo. La memoria puede estar sobresuscrita y, de nuevo, AVS no impone ningún límite y es el cliente el que ejecuta un rendimiento óptimo del clúster para sus cargas de trabajo.

El uso de la memoria también tiene en cuenta la sobrecarga de administración de vCenter, del administrador de NSX y de otros recursos más pequeños.

### <a name="storage-utilization-on-avs-nodes"></a>Uso del almacenamiento en nodos de AVS

El uso del almacenamiento se calcula en función de la siguiente secuencia:

1. Tamaño necesario para las máquinas virtuales (asignadas tal cual o con espacio de uso basado en el rendimiento)
2. Aplicación del factor de crecimiento si lo hubiera
3. Adición de la sobrecarga de administración y aplicación de la relación de errores tolerables
4. Aplicación del factor de compresión y desduplicación
5. Aplicación del margen de demora necesario del 25 % para vSAN
6. Resultado del almacenamiento disponible para las máquinas virtuales sobre el total del almacenamiento, incluidos los gastos de administración.

El almacenamiento disponible en un clúster de tres nodos se basará en la directiva de almacenamiento predeterminada, que es RAID-1 y utiliza el aprovisionamiento grueso. Al calcular la codificación de borrado o RAID-5, por ejemplo, se requiere un mínimo de cuatro nodos. Tenga en cuenta que en AVS la directiva de almacenamiento debe cambiarla el administrador para permitir más espacio.

## <a name="confidence-ratings"></a>Clasificaciones de confianza

Cada evaluación basada en el rendimiento de Azure Migrate se asocia a una clasificación de confianza comprendida entre una estrella (la más baja) y cinco estrellas (la más alta).

- La clasificación de confianza se asigna a una valoración que se basa en la disponibilidad de puntos de datos necesarios para calcular tal valoración.
- La clasificación de confianza de una valoración le ayuda a calcular la confiabilidad de las recomendaciones de tamaño que proporciona Azure Migrate.
- Las clasificaciones de confianza no son aplicables a evaluaciones *como en local*.
- Para dimensionamiento basado en el rendimiento, las evaluaciones de AVS necesitan los datos de uso de la CPU y de la memoria de la máquina virtual. Los siguientes datos se recopilan, pero no se usan en las recomendaciones de dimensionamiento de AVS:

  - Los datos de IOPS y rendimiento de cada disco conectado a la máquina virtual.
  - La entrada o la salida de red de cada adaptador de red conectado a una máquina virtual.

  Si alguna de las cifras de uso anteriores no está disponible en vCenter Server, la recomendación de tamaño podría no ser confiable.

A continuación se muestra la clasificación de confianza para la evaluación según el porcentaje de puntos de datos disponibles.

| **Disponibilidad de puntos de datos** | **Clasificación de confianza** |
| - | - |
| De 0 a 20 % | 1 estrella |
| De 21 a 40 % | 2 estrellas |
| De 41 a 60 % | 3 estrellas |
| De 61 a 80 % | 4 estrellas |
| De 81 a 100 % | 5 estrellas |

### <a name="low-confidence-ratings"></a>Clasificaciones de confianza bajas

Estos son algunos de los motivos por los que una evaluación puede obtener una clasificación de confianza baja:

- No ha generado un perfil de su entorno durante el tiempo que ha estado creando la evaluación. Por ejemplo, si crea la evaluación con la duración de rendimiento establecida en un día, debe esperar al menos un día después de empezar la detección para que se recopilen todos los puntos de datos.
- La evaluación no puede recopilar los datos de rendimiento de algunas o de todas las VM en el período de evaluación. Para obtener una clasificación de confianza alta, asegúrese de que:

  - Las VM estén activadas mientras dure la evaluación.
  - Se permiten las conexiones salientes en los puertos 443.
  - La memoria dinámica está habilitada en las VM dinámicas de Hyper-V.

  "Recalcule" la evaluación para reflejar los cambios más recientes en la clasificación de confianza.
- Se crearon algunas máquinas virtuales en el período durante el que se calculó la evaluación. Por ejemplo, supongamos que ha creado una evaluación para el historial de rendimiento del último mes, pero algunas máquinas virtuales se han creado hace solo una semana. En este caso, los datos de rendimiento de las nuevas máquinas virtuales no estarán disponibles en ningún momento y la clasificación de confianza sería baja.

> [!NOTE]
> Si la clasificación de confianza de una evaluación no llega a las cinco estrellas, se recomienda que espere al menos un día para que el dispositivo genere el perfil del entorno y luego recalcule la evaluación. En caso contrario, es posible que el ajuste de tamaño basado en el rendimiento no sea confiable. En ese caso, se recomienda cambiar la evaluación a un ajuste de tamaño local.

## <a name="monthly-cost-estimation"></a>Estimación del costo mensual

Una vez completadas las recomendaciones de dimensionamiento, Azure Migrate calcula el costo total de la ejecución de las cargas de trabajo locales en AVS multiplicando el número de nodos necesarios de AVS por el precio del nodo. El costo por máquina virtual se calcula dividiendo el costo total por el número de máquinas virtuales en la evaluación.

- El cálculo tiene en cuenta el número de nodos necesario, el tipo de nodo y la ubicación.
- Para calcular el costo mensual total, suma el costo de todos los nodos.
- Los costos se muestran en la moneda especificada en la configuración de evaluación.

Como los precios de Azure VMware Solution (AVS) son por nodo, el costo total no tiene costo de proceso ni distribución de costos de almacenamiento. [Más información](../azure-vmware/introduction.md)

## <a name="migration-tool-guidance"></a>Guía sobre la herramienta de migración

En el informe preparación para Azure para la evaluación de Azure VMware Solution (AVS), puede ver las siguientes herramientas sugeridas:

- **VMware HCX o Enterprise**: En el caso de los servidores de VMware, la solución Hybrid Cloud Extension (HCX) de VMware es la herramienta de migración sugerida para migrar la carga de trabajo local a la nube privada de Azure VMware Solution (AVS). [Más información](../azure-vmware/tutorial-deploy-vmware-hcx.md).
- **Desconocido**: en el caso de los servidores importados mediante un archivo CSV, se desconoce la herramienta de migración predeterminada. Sin embargo, para los servidores de VMware, se recomienda usar la solución Hybrid Cloud Extension (HCX) de VMware.

## <a name="next-steps"></a>Pasos siguientes

Cree una evaluación para [máquinas virtuales de VMware de Azure VMware Solution](how-to-create-azure-vmware-solution-assessment.md).
