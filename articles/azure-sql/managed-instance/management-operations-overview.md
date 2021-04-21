---
title: Introducción a las operaciones de administración
titleSuffix: Azure SQL Managed Instance
description: Más información sobre la duración de las operaciones de administración de Azure SQL Managed Instance y los procedimientos recomendados.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: overview
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, MashaMSFT
ms.date: 07/10/2020
ms.openlocfilehash: bd66c10bb1d6316bbe90e7ba4092d79c6a43a75d
ms.sourcegitcommit: c6a2d9a44a5a2c13abddab932d16c295a7207d6a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107285301"
---
# <a name="overview-of-azure-sql-managed-instance-management-operations"></a>Introducción a las operaciones de administración de Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Instancia administrada de SQL proporciona operaciones de administración que puede usar para implementar automáticamente instancias administradas nuevas, actualizar las propiedades de una instancia y eliminar instancias que ya no son necesarias.

## <a name="what-are-management-operations"></a>¿Qué son las operaciones de administración?

Todas las operaciones de administración se pueden clasificar de la siguiente manera:

- Implementación de una instancia (creación de instancia nueva).
- Actualización de una instancia (cambio de las propiedades de una instancia, como núcleos virtuales o almacenamiento reservado)
- Eliminación de una instancia

Para admitir las [implementaciones dentro de máquinas virtuales de Azure](../../virtual-network/virtual-network-for-azure-services.md) y proporcionar aislamiento y seguridad para los clientes, SQL Managed Instance depende de los [clústeres virtuales](connectivity-architecture-overview.md#high-level-connectivity-architecture). Un clúster virtual representa un conjunto dedicado de máquinas virtuales aisladas implementadas dentro de la subred de la red virtual del cliente. Básicamente, cada implementación de instancia administrada en una subred vacía da como resultado la creación de un clúster virtual.

Las operaciones de administración posteriores en instancias administradas pueden afectar al clúster virtual subyacente. Los cambios que afectan al clúster virtual subyacente también pueden afectar a la duración de las operaciones de administración debido a que la implementación de las máquinas virtuales adicionales conlleva una sobrecarga que se debe tener en cuenta cuando planea implementaciones nuevas o actualizaciones de instancias administradas existentes.


## <a name="duration"></a>Duration

La duración de las operaciones en el clúster virtual puede variar, pero normalmente tienen la duración más larga. 

Los valores siguientes son los valores normalmente previsibles en función de los datos de telemetría de los servicios existentes:

- **Creación de un clúster virtual**:  es un paso sincrónico en las operaciones de administración de una instancia. <br/> **El 90 % de las operaciones finaliza en 4 horas**.
- **Cambio de tamaño del clúster virtual (expansión o reducción)** : La expansión es un paso sincrónico, mientras que la reducción se realiza de manera asincrónica (sin afectar la duración de las operaciones de administración de la instancia). <br/>**El 90 % de las expansiones de un clúster finaliza en menos de 2,5 horas**.
- **Eliminación de un clúster virtual**: La eliminación es un paso asincrónico, pero también se puede [iniciar manualmente](virtual-cluster-delete.md) en un clúster virtual vacío, en cuyo caso se ejecuta de manera sincrónica. <br/>**El 90 % de las eliminaciones de clúster virtual finaliza en 1,5 horas**.

Además, la administración de las instancias también puede incluir una de las operaciones en bases de datos hospedadas, lo que provoca duraciones más largas:

- **Asociación de archivos de base de datos desde Azure Storage**:  un paso sincrónico, como el escalado o la reducción vertical de un proceso (núcleos virtuales) o de un almacenamiento en el nivel de servicio De uso general. <br/>**El 90 % de estas operaciones finaliza en 5 minutos**.
- **Inicialización de un grupo de disponibilidad AlwaysOn**: un paso sincrónico, como el escalado o la reducción vertical de un proceso (núcleo virtual) o de un almacenamiento en el nivel de servicio Crítico para la empresa, así como en el cambio del nivel de servicio De uso general a Crítico para la empresa (o viceversa). La duración de esta operación es proporcional al tamaño total de la base de datos, así como la actividad de base de datos actual (el número de transacciones activas). La actividad de la base de datos cuando se actualiza una instancia puede introducir una varianza considerable en la duración total. <br/>**El 90 % de estas operaciones se ejecuta a 220 GB/hora o más**.

En las tablas siguientes se resumen las operaciones y duraciones generales normales según la categoría de la operación:

**Categoría: Implementación**

|Operación  |Segmento de larga duración  |Duración estimada  |
|---------|---------|---------|
|Primera instancia en una subred vacía|Creación de un clúster virtual|El 90 % de las operaciones finaliza en 4 horas.|
|Primera instancia de otra generación de hardware en una subred no vacía (por ejemplo, primera instancia Gen 5 en una subred con instancias Gen 4)|Creación de un clúster virtual<sup>1</sup>|El 90 % de las operaciones finaliza en 4 horas.|
|Creación de instancia subsiguiente dentro de la subred no vacía (segunda instancia, tercera instancia, etc.)|Cambio de tamaño de un clúster virtual|El 90 % de las operaciones finaliza en 2,5 horas.|
| | | 

<sup>1</sup> El clúster virtual se crea por generación de hardware.

**Categoría: Actualización**

|Operación  |Segmento de larga duración  |Duración estimada  |
|---------|---------|---------|
|Cambio de una propiedad de una instancia (contraseña de administrador, inicio de sesión de Azure AD, marca de Ventaja híbrida de Azure)|N/D|Hasta 1 minuto.|
|Escalado o reducción vertical del almacenamiento de una instancia (nivel de servicio De uso general)|No hay segmento de larga duración<sup>1</sup>|El 99 % de las operaciones finaliza en 5 minutos.|
|Escalado o reducción vertical del almacenamiento de una instancia (nivel de servicio Crítico para la empresa)|- Cambio de tamaño de un clúster virtual<br>- Inicialización de un grupos de disponibilidad AlwaysOn|El 90 % de las operaciones finaliza en 2,5 horas + tiempo para inicializar todas las bases de datos (220 GB/hora).|
|Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (De uso general)|- Cambio de tamaño de un clúster virtual<br>- Adjuntar archivos de base de datos|El 90 % de las operaciones finaliza en 2,5 horas.|
|Escalado y reducción vertical del proceso de una instancia (núcleos virtuales) (Crítico para la empresa)|- Cambio de tamaño de un clúster virtual<br>- Inicialización de un grupos de disponibilidad AlwaysOn|El 90 % de las operaciones finaliza en 2,5 horas + tiempo para inicializar todas las bases de datos (220 GB/hora).|
|Cambio en el nivel de servicio de una instancia (De uso general a Crítico para la empresa y viceversa)|- Cambio de tamaño de un clúster virtual<br>- Inicialización de un grupos de disponibilidad AlwaysOn|El 90 % de las operaciones finaliza en 2,5 horas + tiempo para inicializar todas las bases de datos (220 GB/hora).|
| | | 

<sup>1</sup> El escalado del almacenamiento de una instancia administrada de uso general no provocará una conmutación por error al final de la operación. En este caso, la operación consiste en actualizar los metadatos y propagar la respuesta para la solicitud enviada.

**Categoría: Eliminación**

|Operación  |Segmento de larga duración  |Duración estimada  |
|---------|---------|---------|
|Eliminación de una instancia|Copia del final del registro para todas las bases de datos|El 90 % de las operaciones finaliza en hasta 1 minuto.<br>Nota: Si se elimina la última instancia de la subred, esta operación programará la eliminación del clúster virtual después de 12 horas.<sup>1</sup>|
|Eliminación de un clúster virtual (como operación iniciada por el usuario)|Eliminación de un clúster virtual|El 90 % de las operaciones finaliza en hasta 1,5 hora.|
| | | 

<sup>1</sup>12 horas es la configuración actual pero está sujeta a cambios en el futuro. Si necesita eliminar un clúster virtual anterior (por ejemplo, para liberar la subred), consulte [Eliminación de una subred después de eliminar una instancia administrada](virtual-cluster-delete.md).

## <a name="instance-availability"></a>Disponibilidad de instancias

SQL Managed Instance **está disponible durante las operaciones de actualización**, excepto durante un tiempo de inactividad breve provocado por la conmutación por error que se produce al final de la actualización. Normalmente tarda 10 segundos como máximo, incluso en el caso de transacciones de larga duración interrumpidas, gracias a la [recuperación acelerada de bases de datos](../accelerated-database-recovery.md).

> [!NOTE]
> El escalado del almacenamiento de una instancia administrada de uso general no provocará una conmutación por error al final de la actualización.

Instancia administrada de SQL no está disponible para las aplicaciones cliente durante las operaciones de implementación y eliminación.

> [!IMPORTANT]
> No se recomienda escalar el proceso ni el almacenamiento de Azure SQL Managed Instance ni cambiar el nivel de servicio al mismo tiempo que las transacciones de larga duración (importación de datos, trabajos de procesamiento de datos, regeneración del índice, etc.). La conmutación por error de la base de datos al final de la operación cancela todas las transacciones en curso. 

## <a name="management-operations-steps"></a>Pasos de las operaciones de administración

Las operaciones de administración constan de varios pasos. Con la [introducción de Operations API](management-operations-monitor.md) estos pasos se exponen para el subconjunto de operaciones (implementación y actualización). La operación de implementación consta de tres pasos mientras que la de actualización se realiza en seis pasos. Para más información sobre la duración de las operaciones, consulte la sección [Duración de las operaciones de administración](#duration). Los pasos se enumeran por orden de ejecución.

### <a name="managed-instance-deployment-steps"></a>Pasos de implementación de la instancia administrada

|Nombre del paso  |Descripción del paso  |
|----|---------|
|Validación de solicitudes |Se validan los parámetros enviados. En caso de configuración errónea, se producirá un error en la operación. |
|Cambio de tamaño o creación de un clúster virtual |Según el estado de la subred, se podrá crear el clúster virtual o cambiar su tamaño. |
|Nuevo inicio de instancia de SQL |El proceso de SQL se inicia en el clúster virtual implementado. |

### <a name="managed-instance-update-steps"></a>Pasos de actualización de la instancia administrada

|Nombre del paso  |Descripción del paso  |
|----|---------|
|Validación de solicitudes | Se validan los parámetros enviados. En caso de configuración errónea, se producirá un error en la operación. |
|Cambio de tamaño o creación de un clúster virtual |Según el estado de la subred, se podrá crear el clúster virtual o cambiar su tamaño. |
|Nuevo inicio de instancia de SQL | El proceso de SQL se inicia en el clúster virtual implementado. |
|Inicializar o asociar archivos de base de datos |Según el tipo de operación de actualización, se inicializará la base de datos o se asociarán los archivos de esta. |
|Preparación de la conmutación por error y conmutación por error |Una vez que se han inicializado los datos o se han vuelto a asociar los archivos de la base de datos, se prepara el sistema para la conmutación por error. Cuando todo está configurado, se realiza la conmutación por error **con un breve tiempo de inactividad**. |
|Limpieza de instancias de SQL antiguas |Proceso de eliminación de instancias antiguas de SQL del clúster virtual |

> [!NOTE]
> Como resultado del escalado de las instancias, el clúster virtual subyacente pasará por el proceso de liberar capacidad no usada y de una posible desfragmentación de la capacidad, lo que podría afectar a las instancias que no participaron en las operaciones de creación y escalado. 


## <a name="management-operations-cross-impact"></a>Impacto de las operaciones de administración

Las operaciones de administración de una instancia administrada pueden afectar a otras operaciones de administración de las instancias colocadas en el mismo clúster virtual:

- Las **operaciones de restauración de larga ejecución** de un clúster virtual pondrán en espera otras operaciones de creación de instancias o de escalado en la misma subred.<br/>**Ejemplo**: si hay una operación de restauración de larga duración y hay una solicitud de creación o escalado en la misma subred, esta solicitud tardará más tiempo en completarse, ya que esperará a que se complete la operación de restauración antes de continuar.

- La operación de creación o escalado de instancias que inició el cambio de tamaño del clúster virtual pondrá en espera una operación de **creación o escalado de instancias posterior**.<br/>**Ejemplo**: si hay varias solicitudes de creación o escalado en la misma subred del mismo clúster virtual y una de ellas inicia el cambio de tamaño de este, todas las solicitudes que se enviaron más de cinco minutos después de la solicitud de operación inicial tardarán más de lo esperado, ya que estas solicitudes tendrán que esperar a que el cambio de tamaño se complete antes de reanudarse.

- **Las operaciones de creación y escalado enviadas en una ventana de 5 minutos** se procesarán por lotes y se ejecutarán en paralelo.<br/>**Ejemplo**: solo se realizará un cambio de tamaño del clúster virtual para todas las operaciones enviadas en la ventana de 5 minutos (medida a partir del momento de la ejecución de la primera solicitud de operación). Si otra solicitud se envía más de 5 minutos después de enviar la primera, tendrá que esperar a que el cambio de tamaño del clúster virtual se complete antes de iniciar la ejecución.

> [!IMPORTANT]
> Las operaciones de administración que se ponen en espera debido a otra operación en curso se reanudarán automáticamente una vez que se cumplan las condiciones para continuar. No se necesita ninguna acción del usuario para reanudar las operaciones de administración temporalmente en pausa.

## <a name="monitoring-management-operations"></a>Supervisión de las operaciones de administración

Para más información sobre cómo supervisar el progreso y el estado de la operación de administración, consulte [Supervisión de las operaciones de administración](management-operations-monitor.md).

## <a name="canceling-management-operations"></a>Cancelación de operaciones de administración

Para más información sobre cómo cancelar la operación de administración, consulte [Cancelación de operaciones de administración](management-operations-cancel.md).


## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre cómo crear su primera instancia administrada, vea la [Guía de inicio rápido](instance-create-quickstart.md).
- Para obtener una lista de características y una comparación, consulte [Características comunes de SQL](../database/features-comparison.md).
- Para obtener más información sobre la configuración de redes virtuales, vea [Configuración de una red virtual de Instancia administrada de SQL](connectivity-architecture-overview.md).
- Para ver un inicio rápido en el que se crea una instancia administrada y se restaura una base de datos desde un archivo de copia de seguridad, consulte [Creación de una instancia administrada](instance-create-quickstart.md).
- Para consultar un tutorial sobre el uso de Azure Database Migration Service para la migración, consulte [Migración de Instancia administrada de SQL mediante Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
