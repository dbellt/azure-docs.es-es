---
title: Recuperación acelerada de bases de datos.
titleSuffix: Azure SQL
description: La Recuperación acelerada de la base de datos proporciona una recuperación de bases de datos rápida y coherente, reversión de transacciones instantánea y truncamiento de registros agresivo para bases de datos de la cartera de Azure SQL.
ms.service: sql-database
ms.subservice: backup-restore
ms.custom: sqldbrb=4
ms.devlang: ''
ms.topic: conceptual
author: kfarlee
ms.author: kfarlee
ms.reviewer: mathoma
ms.date: 05/19/2020
ms.openlocfilehash: aefc5be21dc60327baecbcc7614e9bc1a78f2ca1
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110708974"
---
# <a name="accelerated-database-recovery-in-azure-sql"></a>Recuperación de base de datos acelerada en Azure SQL 
[!INCLUDE[appliesto-sqldb-sqlmi](includes/appliesto-sqldb-sqlmi.md)]

**Recuperación acelerada de la base de datos (ADR)** es una característica del motor de base de datos de SQL Server que mejora considerablemente la disponibilidad de la base de datos, en especial en presencia de transacciones de larga duración, al rediseñar el proceso de recuperación del motor de base de datos de SQL Server. 

ADR está disponible actualmente para Azure SQL Database, Azure SQL Managed Instance, bases de datos de Azure Synapse Analytics y SQL Server en VM de Azure a partir de SQL Server 2019. 

> [!NOTE] 
> ADR está habilitado de forma predeterminada en Azure SQL Database y Azure SQL Managed Instance y no se admite su deshabilitación en ninguno de los productos. 

## <a name="overview"></a>Información general

Las principales ventajas de ADR son:

- **Recuperación de base de datos rápida y coherente**

  Con ADR, las transacciones de larga duración no afectan al tiempo de recuperación total, lo que permite una recuperación de base de datos rápida y coherente, con independencia del número de transacciones activas en el sistema o de sus tamaños.

- **Reversión de transacción instantánea**

  Con ADR, la reversión de la transacción es instantánea, independientemente del momento en que la transacción haya estado activa o del número de actualizaciones que haya realizado.

- **Truncamiento de registro agresivo**

  Con ADR, el registro de transacciones se trunca de manera agresiva, incluso en presencia de transacciones activas de larga duración, lo que evita que crezca fuera de control.

## <a name="standard-database-recovery-process"></a>Proceso estándar de recuperación de bases de datos

La recuperación de bases de datos sigue el modelo de recuperación [ARIES](https://people.eecs.berkeley.edu/~brewer/cs262/Aries.pdf) y consta de tres fases, que se muestran en el diagrama siguiente y se explican con más detalle después del diagrama.

![Proceso de recuperación actual](./media/accelerated-database-recovery/current-recovery-process.png)

- **Fase de análisis**

  Análisis hacia delante del registro de transacciones desde el principio del último punto de comprobación correcto (o el LSN de página desfasada más antiguo) hasta el final para determinar el estado de cada transacción en el momento en que la base de datos se ha detenido.

- **Fase de rehacer**

  Análisis hacia delante del registro de transacciones desde la transacción más antigua sin confirmar hasta el final para actualizar la base de datos al estado que tenía en el momento del bloqueo al poner al día todas las operaciones confirmadas.

- **Fase de deshacer**

  Para cada transacción activa en el momento del bloqueo, recorre el registro hacia atrás y deshace las operaciones realizadas por esta transacción.

En función de este diseño, el tiempo que tarda el motor de base de datos de SQL Server en recuperarse de un reinicio inesperado es (aproximadamente) proporcional al tamaño de la transacción activa más larga del sistema en el momento del bloqueo. La recuperación requiere la reversión de todas las transacciones incompletas. El período de tiempo necesario es proporcional al trabajo que ha realizado la transacción y el tiempo que ha estado activa. Por lo tanto, el proceso de recuperación puede tardar mucho tiempo en la presencia de transacciones de larga duración (como operaciones de inserción masiva grandes u operaciones de generación de índice en una tabla grande).

Además, la cancelación o la reversión de una transacción grande en función de este diseño puede tardar mucho porque usa la misma fase de recuperación de reversión descrita anteriormente.

Además, el motor de base de datos de SQL Server no puede truncar el registro de transacciones si hay transacciones de ejecución prolongada, ya que sus entradas de registro correspondientes son necesarias para los procesos de recuperación y reversión. Como resultado de este diseño del motor de base de datos de SQL Server, algunos clientes se enfrentan al problema de que el tamaño del registro de transacciones crece mucho y consume cantidades enormes de espacio de la unidad.

## <a name="the-accelerated-database-recovery-process"></a>El proceso de recuperación de base de datos acelerada

ADR aborda los problemas anteriores al rediseñar por completo el proceso de recuperación del motor de base de datos de SQL Server para:

- Sea constante/instantáneo, ya que evita tener que examinar el registro desde/hasta el principio de la transacción activa más antigua. Con ADR, el registro de transacciones solo se procesa desde el último punto de comprobación correcto (o un número de secuencia de registro [LSN] de página desfasada más antiguo). Como resultado, el tiempo de recuperación no se ve afectado por las transacciones de larga duración.
- Minimice el espacio del registro de transacciones necesario, ya que ya no es necesario procesar el registro para toda la transacción. Como resultado, el registro de transacciones se puede truncar de manera agresiva a medida que se producen puntos de comprobación y copias de seguridad.

En líneas generales, ADR consigue una recuperación de base de datos rápida mediante la creación de versiones de todas las modificaciones físicas de la base de datos y solo se deshacen las operaciones lógicas, que son limitadas y se pueden deshacer casi al instante. Cualquier transacción que estuviera activa en el momento de un bloqueo se marca como anulada y, por lo tanto, las consultas de usuario simultáneas pueden pasar por alto las versiones generadas por estas transacciones.

El proceso de recuperación de ADR tiene las mismas tres fases que el proceso de recuperación actual. El funcionamiento de estas fases con ADR se muestra en el siguiente diagrama y se explica con más detalle después del diagrama.

![Proceso de recuperación de ADR](./media/accelerated-database-recovery/adr-recovery-process.png)

- **Fase de análisis**

  El proceso sigue siendo el mismo de siempre con la adición de reconstruir sLog y copiar entradas de registro para las operaciones sin control de versiones.
  
- **Fase de rehacer**

  Se divide en dos fases
  - Fase 1

      Rehacer desde sLog (transacción sin confirmar más antigua hasta el último punto de control). La operación de rehacer es rápida, ya que solo necesita procesar algunos registros de sLog.

  - Fase 2

     La puesta al día desde el registro de transacciones se inicia desde el último punto de comprobación (en lugar de la transacción sin confirmar más antigua)

- **Fase de deshacer**

   La fase de reversión con ADR termina casi instantáneamente al usar sLog para deshacer las operaciones sin control de versiones y el almacén de versiones persistente (PVS) con reversión lógica para realizar la reversión basada en la versión del nivel de fila.

## <a name="adr-recovery-components"></a>Componentes de recuperación de ADR

Estos son los cuatro componentes clave de ADR:

- **Almacén de versiones persistentes (PVS)**

  El almacén de versiones persistente es un nuevo mecanismo del motor de base de datos de SQL Server para conservar las versiones de fila generadas en la propia base de datos en lugar del almacén de versiones `tempdb` tradicional. PVS habilita el aislamiento de recursos, además de mejorar la disponibilidad de las réplicas secundarias legibles.

- **Reversión lógica**

  La reversión lógica es el proceso asincrónico responsable de realizar la reversión basada en la versión de nivel de fila, lo que proporciona una reversión de transacción instantánea y deshace todas las operaciones con control de versiones. La reversión lógica se logra al:

  - Realizar el seguimiento de todas las transacciones anuladas y marcarlas como invisibles para otras transacciones. 
  - Revertir mediante PVS para todas las transacciones de usuario, en lugar de examinar físicamente el registro de transacciones y deshacer los cambios de uno en uno.
  - Liberar todos los bloqueos inmediatamente después de la anulación de transacciones. Dado que la anulación implica simplemente marcar los cambios en memoria, el proceso es muy eficaz y, por tanto, no es necesario mantener los bloqueos durante mucho tiempo.

- **sLog**

  sLog es una secuencia de registro en memoria secundaria que almacena entradas de registro para operaciones sin control de versiones (como la invalidación de la caché de metadatos, las adquisiciones de bloqueos, etc.). sLog tiene estas características:

  - Es de bajo volumen y en memoria
  - Se conserva en el disco mediante su serialización durante el proceso de punto de comprobación
  - Se trunca periódicamente a medida que se confirman las transacciones
  - Acelera las operaciones de rehacer y deshacer procesando solo las operaciones sin control de versiones  
  - Habilita el truncamiento del registro de transacciones agresivo conservando solo las entradas de registro necesarias

- **Limpiador**

  El limpiador es el proceso asincrónico que se reactiva periódicamente y limpia las versiones de página que no son necesarias.

## <a name="accelerated-database-recovery-patterns"></a>Recuperación de base de datos acelerada

Los siguientes tipos de cargas de trabajo se benefician en su mayor parte de ADR:

- Cargas de trabajo con transacciones de larga ejecución.
- Las cargas de trabajo que han detectado casos en los que las transacciones activas hacen que el registro de transacciones crezca de forma significativa.  
- Cargas de trabajo que han experimentado largos períodos de falta de disponibilidad de la base de datos debido a la recuperación de larga duración (por ejemplo, un reinicio inesperado del servicio o la reversión manual de transacciones).
