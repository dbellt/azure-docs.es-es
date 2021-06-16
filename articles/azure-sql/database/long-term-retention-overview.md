---
title: Retención de copia de seguridad a largo plazo
titleSuffix: Azure SQL Database & Azure SQL Managed Instance
description: Obtenga información sobre cómo admiten Azure SQL Database y Azure SQL Managed Instance el almacenamiento de copias de seguridad de bases de datos completas durante un máximo de 10 años mediante la directiva de retención a largo plazo.
services: sql-database
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: shkale-msft
ms.author: shkale
ms.reviewer: mathoma
ms.date: 02/25/2021
ms.openlocfilehash: e49f6220a25a449221ccab70a7439722189378cd
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708492"
---
# <a name="long-term-retention---azure-sql-database-and-azure-sql-managed-instance"></a>Retención a largo plazo: Azure SQL Database y Azure SQL Managed Instance

Muchas aplicaciones tienen finalidades normativas, de cumplimiento u otras de carácter empresarial que requieren la conservación de las copias de seguridad de las bases de datos más allá del período de entre 7 y 35 días que ofrecen las [copias de seguridad automáticas](automated-backups-overview.md) de Azure SQL Database e Instancia administrada de Azure SQL. Con la característica Retención a largo plazo (LTR), puede almacenar copias de seguridad completas de SQL Database e Instancia administrada de SQL en Azure Blob Storage con [redundancia configurada](automated-backups-overview.md#backup-storage-redundancy) con acceso de lectura durante un máximo de 10 años. Las copias de seguridad de LTR se pueden restaurar luego como una nueva base de datos.

La retención a largo plazo se puede habilitar para Azure SQL Database y está disponible en versión preliminar pública para Azure SQL Managed Instance. En este artículo se proporciona información general conceptual de la retención a largo plazo. Para configurar la retención a largo plazo, consulte [Configuración de la retención a largo plazo en Azure SQL Database](long-term-backup-retention-configure.md) y [Configuración de retención a largo plazo en Instancia administrada de Azure SQL](../managed-instance/long-term-backup-retention-configure.md). 

> [!NOTE]
> Puede usar los trabajos del Agente SQL para programar [copias de seguridad de base de datos de solo copia](/sql/relational-databases/backup-restore/copy-only-backups-sql-server) como alternativa a la retención a largo plazo después de 35 días.

> [!IMPORTANT]
> La retención a largo plazo en Managed Instance solo está disponible actualmente en versión preliminar pública en las regiones públicas de Azure. 


## <a name="how-long-term-retention-works"></a>Cómo funciona la retención a largo plazo
     
La retención a largo plazo (LTR) aprovecha las copias de seguridad de base de datos completas que se [crean automáticamente](automated-backups-overview.md) para hacer posible la restauración a un momento dado (PITR). Si se configura una directiva LTR, estas copias de seguridad se copian en diferentes blobs para almacenarlas a largo plazo. La copia es un trabajo en segundo plano que no afecta al rendimiento de la carga de trabajo de la base de datos. La directiva de LTR de cada base de datos de SQL Database también puede especificar con qué frecuencia se crean las copias de seguridad LTR.

Para habilitar LTR, puede definir la directiva mediante una combinación de cuatro parámetros: retención de copia de seguridad semanal (W), retención de copia de seguridad mensual (M), retención de copia de seguridad anual (Y) y semana del año (WeekOfYear). Si especifica W, se copiará una copia de seguridad cada semana en el almacenamiento a largo plazo. Si especifica M, la primera copia de seguridad de cada mes se copiará en el almacenamiento a largo plazo. Si especifica Y, se copiará una copia de seguridad durante la semana especificada en WeekOfYear en el almacenamiento a largo plazo. Si el valor de WeekOfYear especificado pertenece al pasado al configurar la directiva, la primera copia de seguridad de LTR se creará el año siguiente. Cada copia de seguridad se conservará en el almacenamiento a largo plazo según los parámetros de la directiva configurados al crear la copia de seguridad de LTR.

> [!NOTE]
> Cualquier cambio en la directiva LTR se aplica solo a las copias de seguridad futuras. Por ejemplo, si se modifican los parámetros de retención de copia de seguridad semanal (W), retención de copia de seguridad mensual (M) o retención de copia de seguridad anual (Y), la nueva configuración de retención solo se aplicará a las copias de seguridad nuevas. La retención de las copias de seguridad existentes no se modificará. Si su intención es eliminar las copias de seguridad de LTR anteriores antes de que expire su período de retención, tendrá que [eliminar manualmente las copias de seguridad](./long-term-backup-retention-configure.md#delete-ltr-backups).
> 

Ejemplos de la directiva LTR:

-  W=0, M=0, Y=5, WeekOfYear=3

   La tercera copia de seguridad completa de cada año se conservará durante cinco años.
   
- W=0, M=3, Y=0

   La primera copia de seguridad completa de cada mes se conservará durante tres meses.

- W=12, M=0, Y=0

   Cada copia de seguridad completa semanal se conservará durante 12 semanas.

- W=6, M=12, Y=10, WeekOfYear=16

   Cada copia de seguridad completa semanal se conservará durante seis semanas. Excepto la primera copia de seguridad completa de cada mes, que se conservará durante 12 meses. Excepto la copia de seguridad completa realizada en la 16ª semana del año, que se conservará durante 10 años. 

En la tabla siguiente se muestra la cadencia y caducidad de las copias de seguridad a largo plazo para la siguiente directiva:

W=12 semanas (84 días), M=12 meses (365 días), Y=10 años (3650 días), WeekOfYear=15 (semana posterior al 15 de abril)

   ![Ejemplo de LTR](./media/long-term-retention-overview/ltr-example.png)


Si modifica la directiva anterior y establece W=0 (sin copias de seguridad semanales), la cadencia de las copias de seguridad cambiará tal y como se muestra en la tabla anterior en función de las fechas destacadas. La cantidad de almacenamiento necesaria para mantener estas copias de seguridad se reduciría según corresponda. 

> [!IMPORTANT]
> Azure controla los intervalos en que se hacen las copias de seguridad individuales de retención a largo plazo. No se puede crear una copia de seguridad de retención a largo plazo manualmente ni tampoco controlar los intervalos de creación de copias de seguridad. Después de configurar una directiva LTR, pueden transcurrir hasta siete días para que la primera copia de seguridad de LTR aparezca en la lista de copias de seguridad disponibles.  


## <a name="geo-replication-and-long-term-backup-retention"></a>Retención de copia de seguridad a largo plazo y replicación geográfica

Si usa grupos de conmutación por error o de replicación geográfica activa como solución de continuidad del negocio, debe prepararse para posibles conmutaciones por error y configurar la misma directiva de LTR en la instancia o base de datos secundaria. El costo de almacenamiento de LTR no aumentará, ya que las copias de seguridad no se generan desde las bases de datos secundarias. Las copias de seguridad solo se crean cuando la base de datos secundaria se convierte en principal. Esto garantiza una generación de las copias de seguridad LTR ininterrumpida cuando se desencadene la conmutación por error y la base de datos principal se mueva a la región secundaria. 

> [!NOTE]
> Cuando la base de datos principal original se recupere de una interrupción que provoque la conmutación por error, se convertirá en una nueva base de datos secundaria. Por lo tanto, no se reanudará la creación de copia de seguridad y la directiva LTR existente no surtirá efecto hasta que vuelva a ser la base de datos principal de nuevo. 


## <a name="configure-long-term-backup-retention"></a>Configuración de la retención de copia de seguridad a largo plazo

Puede configurar la retención de copias de seguridad a largo plazo mediante Azure Portal y PowerShell para Azure SQL Database y Azure SQL Managed Instance. Para restaurar una base de datos desde el almacenamiento de LTR, puede seleccionar una copia de seguridad específica en función de su marca de tiempo. La base de datos se puede restaurar en cualquier servidor existente de la misma instancia administrada y con la misma suscripción que la base de datos original.

Para aprender a configurar la retención a largo plazo o restaurar una base de datos a partir de una copia de seguridad de SQL Database mediante Azure Portal o PowerShell, consulte [Administración de la retención de copias de seguridad a largo plazo de Azure SQL Database](long-term-backup-retention-configure.md).

Para obtener información sobre cómo configurar la retención a largo plazo o cómo restaurar una base de datos a partir de una copia de seguridad de SQL Managed Instance mediante PowerShell, consulte [Administración de la retención de copias de seguridad a largo plazo en Azure SQL Managed Instance](../managed-instance/long-term-backup-retention-configure.md).

Para restaurar una base de datos desde el almacenamiento de LTR, puede seleccionar una copia de seguridad específica en función de su marca de tiempo. La base de datos se puede restaurar en cualquier servidor existente en la misma suscripción que la base de datos original. Para más información sobre cómo restaurar la base de datos a partir de una copia de seguridad de LTR mediante Azure Portal o PowerShell, vea [Administración de la retención de copias de seguridad a largo plazo de Azure SQL Database](long-term-backup-retention-configure.md). 

## <a name="next-steps"></a>Pasos siguientes

Dado que las copias de seguridad de bases de datos protegen los datos de eliminaciones o daños accidentales, son una parte esencial de cualquier estrategia de recuperación ante desastres y de continuidad empresarial. 

- Para descubrir otras soluciones de continuidad empresarial de SQL Database, consulte el artículo de [información general sobre la continuidad empresarial](business-continuity-high-availability-disaster-recover-hadr-overview.md).
- Para aprender sobre las copias de seguridad automáticas generadas por el servicio, consulte [copias de seguridad automáticas](../database/automated-backups-overview.md)
