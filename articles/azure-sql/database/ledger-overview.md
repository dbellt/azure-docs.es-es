---
title: Introducción al libro de contabilidad de Azure SQL Database
description: Introducción al libro de contabilidad de Azure SQL Database
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: 36d7c5d6b97b54392be8c85d1b6a6088eda5f983
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388752"
---
# <a name="azure-sql-database-ledger"></a>Libro de contabilidad de Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

Establecer la confianza en torno a la integridad de los datos almacenados en los sistemas de base de datos ha sido un problema persistente para todas las organizaciones que administran datos financieros, médicos u otros datos confidenciales. La característica del libro de contabilidad de [Azure SQL Database](sql-database-paas-overview.md) proporciona a cualquier base de datos funcionalidades de evidencia de alteración, lo que permite certificar criptográficamente a otras partes, como auditores u otras partes empresariales, que los datos no se han alterado.

El libro de contabilidad ayuda a proteger los datos frente a cualquier atacante o usuario con privilegios elevados, incluidos los administradores de bases de datos (DBA) y los administradores del sistema y de la nube. Al igual que un libro de contabilidad tradicional, los datos históricos se conservan de forma que, si se actualiza una fila en la base de datos, su valor anterior se mantiene y protege en una tabla de historial. El libro de contabilidad proporciona una crónica de todos los cambios realizados en la base de datos a lo largo del tiempo. El libro de contabilidad y los datos históricos se administran de forma transparente, lo que ofrece protección sin cambios en la aplicación. Los datos históricos se mantienen en un formato relacional para admitir consultas SQL con fines de auditoría, análisis forense y otros fines. Ledger proporciona garantías de integridad de los datos criptográficos al tiempo que mantiene la eficacia, la flexibilidad y el rendimiento de Azure SQL Database.

:::image type="content" source="media/ledger/ledger-table-architecture.png" alt-text="arquitectura de tablas del libro de contabilidad":::

## <a name="use-case-for-azure-sql-database-ledger"></a>Caso de uso para el libro de contabilidad de Azure SQL Database 

### <a name="streamlining-audits"></a>Simplificación de auditorías

El valor de cualquier sistema de producción se basa en la capacidad de confiar en los datos que el sistema consume y produce. Si un usuario malintencionado ha alterado los datos de la base de datos, puede tener resultados desastrosos en los procesos empresariales que dependen de esos datos. Mantener la confianza en los datos requiere una combinación de habilitar los controles de seguridad adecuados para reducir los posibles ataques, las prácticas de copia de seguridad y restauración y los procedimientos exhaustivos de recuperación ante desastres. Las partes externas suelen auditar estas prácticas para garantizar su aplicación. Los procesos de auditoría son actividades que consumen mucho tiempo. La auditoría requiere la inspección en el sitio de prácticas implementadas, como revisar los registros de auditoría, inspeccionar los controles de autenticación y acceso, por nombrar algunos. Aunque estos procesos manuales pueden exponer posibles brechas en la seguridad, lo que no pueden proporcionar es una prueba atestable de que los datos no se han modificado de forma malintencionada. Ledger proporciona la prueba criptográfica de integridad de los datos a los auditores, lo que puede ayudar no solo a simplificar el proceso de auditoría, sino que también proporciona la capacidad de no rechazar la integridad de los datos del sistema.

### <a name="multi-party-business-processes"></a>Procesos empresariales de varias partes

Los sistemas en los que varias organizaciones tienen un proceso empresarial que deben compartir entre sí, como los sistemas de administración de la cadena de suministro, tienen dificultades para compartir los datos entre sí con confianza. Muchas organizaciones están recurriendo a cadenas de bloques tradicionales, como Ethereum o Hyperledger Fabric, para transformar digitalmente sus procesos empresariales de varias partes. La cadena de bloques es una excelente solución para redes de varias partes en las que la confianza es baja entre las partes que participan en la red. Sin embargo, muchas de estas redes son soluciones fundamentalmente centralizadas donde la confianza es importante, pero una infraestructura totalmente descentralizada es una solución de gran peso. Ledger proporciona una solución para estas redes en la que los participantes pueden comprobar la integridad de los datos de forma centralizada, en lugar de tener las implicaciones de complejidad y rendimiento que el consenso de la red introduce en una red de cadena de bloques.

### <a name="trusted-off-chain-storage-for-blockchain"></a>Almacenamiento fuera de la cadena de bloques de confianza

Cuando una red de cadena de bloques es necesaria para un proceso empresarial de varias partes, tener la capacidad de consultar los datos en la cadena de bloques sin sacrificar el rendimiento es un desafío. Los patrones típicos para resolver este problema implican la replicación de datos de la cadena de bloques a un almacén fuera de la cadena, como una base de datos. Sin embargo, una vez que los datos se replican en la base de datos desde la cadena de bloques, la integridad de los datos garantiza la pérdida de una oferta de cadena de bloques. Ledger proporciona la integridad de datos necesaria para el almacenamiento fuera de la cadena de bloques, lo que garantiza la plena confianza en los datos en todo el sistema.

## <a name="how-it-works"></a>Cómo funciona

Cada transacción que recibe la base está codificada con hash criptográfico (SHA-256). La función hash usa el valor de la transacción (incluidos los hashes de las filas contenidas en la transacción), junto con el hash de la transacción anterior como entrada para la función hash. La función vincula criptográficamente todas las transacciones, de forma similar a una cadena de bloques. Los hashes criptográficos ([resúmenes de base de datos](#database-digests)), que representan el estado de la base de datos, se generan y almacenan periódicamente fuera de Azure SQL Database en una ubicación de almacenamiento a prueba de alteraciones. Un ejemplo de una ubicación de almacenamiento sería [blobs inmutables de Azure Storage](../../storage/blobs/storage-blob-immutable-storage.md) o [Azure Confidential Ledger](/azure/confidential-ledger/). Los resúmenes de base de datos se usan posteriormente para comprobar la integridad de la base de datos comparando el valor del hash del resumen con los hash calculados de la base de datos. 

La funcionalidad del libro de contabilidad se introduce en las tablas en Azure SQL Database de dos formas:

- [**Tablas de libro de contabilidad actualizables**](ledger-updatable-ledger-tables.md), que permiten actualizar y eliminar filas de las tablas.
- [**Tablas de libro de contabilidad de solo anexión**](ledger-append-only-ledger-tables.md), que solo permiten inserciones en las tablas.

Tanto las **tablas de libro de contabilidad actualizables** como las **tablas de libro de contabilidad de solo anexión** proporcionan funcionalidades de evidencia de alteración y análisis forense digital. Comprender qué transacciones enviadas por los usuarios que provocaron cambios en la base de datos son importantes si se corrigen posibles eventos de alteración o demostrar a terceros que las transacciones enviadas al sistema proceden de usuarios autorizados. La característica de libro de contabilidad permite a los usuarios, sus asociados o auditores analizar todas las operaciones históricas y detectar posibles alteraciones. Cada operación de fila va acompañada del identificador de la transacción que la realizó, lo que permite a los usuarios recuperar más información sobre el momento en que se ejecutó la transacción, la identidad del usuario que la ejecutó y correlacionarla con otras operaciones realizadas por esta transacción.

Hay algunas limitaciones de las tablas de libro de contabilidad que debe tener en cuenta. Para más información sobre las limitaciones de las tablas de libro de contabilidad, vea [Limitaciones del libro de contabilidad de Azure SQL Database](ledger-limits.md).

### <a name="ledger-database"></a>Base de datos de libro de contabilidad

Una base de datos de libro de contabilidad es una base de datos, en la que es evidente la alteración de los datos del usuario y donde se almacenan en tablas de libro de contabilidad. Una base de datos de libro de contabilidad solo puede contener tablas de libro de contabilidad, y cada tabla se crea de forma predeterminada como una tabla de libro de contabilidad actualizable. Las bases de datos de libro de contabilidad proporcionan una solución fácil de usar para las aplicaciones que requieren que se proteja la integridad de todos los datos. 

### <a name="updatable-ledger-tables"></a>Tablas de libro de contabilidad actualizables

Las [tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md) son ideales para los patrones de aplicación que esperan emitir actualizaciones y eliminaciones en las tablas de la base de datos, como las aplicaciones del sistema de registro (SOR). Esto significa que los patrones de datos existentes de la aplicación no necesitan cambiar para habilitar la funcionalidad de libro de contabilidad.  

Las [tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md) realizan un seguimiento del historial de cambios de todas las filas de la base de datos cuando se producen transacciones que realizan actualizaciones o eliminaciones. Una tabla de libro de contabilidad actualizable es una tabla con versiones del sistema que contiene una referencia a otra tabla con un esquema reflejado. El sistema usa esta tabla para almacenar automáticamente la versión anterior de una fila de la tabla de libro de contabilidad cada vez que dicha fila se actualiza o elimina. Esta otra tabla se conoce como tabla de historial. La tabla de historial se genera automáticamente cuando se crea una tabla de libro de contabilidad actualizable. Los valores contenidos en la tabla de libro de contabilidad actualizable y su tabla de historial correspondiente proporcionan una crónica de los valores de la base de datos a lo largo del tiempo. Para consultar fácilmente esta crónica de la base de datos, se crea una vista de libro de contabilidad generada por el sistema, que une la tabla de libro de contabilidad actualizable y la tabla de historial.

Para obtener más información sobre cómo crear y usar tablas de libro de contabilidad actualizables, vea [Creación de tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md).

### <a name="append-only-ledger-tables"></a>Tablas de libro de contabilidad de solo anexión

Las [tablas de libro de contabilidad de solo anexión](ledger-append-only-ledger-tables.md) son ideales para los patrones de aplicación que son de solo inserción, como las aplicaciones de administración de eventos e información de seguridad (SEIM). Las tablas de libro de contabilidad de solo anexión bloquean las actualizaciones y eliminaciones en el nivel de interfaz de programación de aplicaciones (API), lo que proporciona una mayor protección contra la alteración por parte de usuarios con privilegios, como los administradores de sistemas y DBA. Puesto que solo se permiten inserciones en el sistema, las tablas de libro de contabilidad de solo anexión no tienen una tabla de historial correspondiente, ya que no hay ningún historial que capturar. Al igual que las tablas de libro de contabilidad actualizables, se crea una vista de libro de contabilidad que proporciona información sobre la transacción que insertó filas en la tabla de solo anexión y el usuario que realizó la inserción.

Para obtener más información sobre cómo crear y usar tablas de libro de contabilidad de solo anexión, vea [Creación y uso de tablas de libro de contabilidad de solo anexión](ledger-how-to-append-only-ledger-tables.md).

### <a name="database-ledger"></a>Libro de contabilidad de base de datos

El libro de contabilidad de base de datos consta de tablas del sistema que almacenan los hashes criptográficos de las transacciones procesadas en el sistema. Puesto que las transacciones son la unidad de [atomicidad](/windows/win32/cossdk/acid-properties) del motor de base de datos, esta es la unidad de trabajo que se captura en el libro de contabilidad de base de datos. En concreto, cuando se confirma una transacción, el hash SHA-256 de las filas modificadas por la transacción en la tabla del libro de contabilidad, junto con algunos metadatos para la transacción, como la identidad del usuario que la ejecutó y su marca de tiempo de confirmación, se anexa como una *entrada de transacción* en el libro de contabilidad de base de datos. Cada 30 segundos, se aplica un algoritmo hash SHA-256 a las transacciones procesadas por la base de datos mediante una estructura de datos en árbol Merkle, lo que genera un hash raíz. Esto forma un bloque, al que luego se aplica un hash SHA-256 mediante el hash raíz del bloque junto con el hash raíz del bloque anterior como entrada a la función hash, formando una cadena de bloques.

Para obtener más información sobre el libro de contabilidad de base de datos, vea [Libro de contabilidad de base de datos](ledger-database-ledger.md).

### <a name="database-digests"></a>Resúmenes de base de datos

El hash del bloque más reciente del libro de contabilidad de la base de datos se conoce como resumen de la base de datos y representa el estado de todas las tablas del libro de contabilidad de la base de datos en el momento en que se generó el bloque. Cuando se forma un bloque, su resumen de base de datos asociado se publica y almacena fuera de Azure SQL Database en un almacenamiento a prueba de alteraciones. Puesto que los resúmenes de base de datos representan el estado de la base de datos en el momento en que se generaron, es fundamental proteger los resúmenes de la alteración. Un atacante que tenga acceso para modificar los resúmenes podría alterar los datos de la base de datos, generar los hashes que representan la base de datos con los cambios alterados y, a continuación, modificar los resúmenes para representar el hash actualizado de las transacciones en el bloque. Ledger proporciona la capacidad de generar y almacenar automáticamente los resúmenes de base de datos en [blobs inmutables de Azure Storage](../../storage/blobs/storage-blob-immutable-storage.md) o en [Azure Confidential Ledger](/azure/confidential-ledger/) para evitar la alteración. Como alternativa, los usuarios pueden generar manualmente resúmenes de base de datos y almacenarlos en la ubicación que prefieran. Los resúmenes de base de datos se usan para comprobar posteriormente que los datos almacenados en tablas de libro de contabilidad no se han alterado.

Para obtener más información sobre los resúmenes de base de datos, vea [Administración de resúmenes y comprobación de base de datos](ledger-digest-management-and-database-verification.md).

### <a name="ledger-verification"></a>Verificación del libro de contabilidad

La característica de libro de contabilidad no permite a los usuarios modificar el contenido del libro de contabilidad. Sin embargo, un atacante o administrador del sistema que tenga el control de la máquina puede omitir todas las comprobaciones del sistema y alterar directamente los datos. Por ejemplo, un atacante o administrador del sistema puede editar los archivos de base de datos en el almacenamiento. Ledger no puede evitar estos ataques, pero garantiza que se detectará cualquier alteración cuando se comprueban los datos del libro de contabilidad. El proceso de comprobación del libro de contabilidad toma como entrada uno o varios resúmenes de base de datos generados previamente y vuelve a compilar los hash almacenados en el libro de contabilidad de base de datos, en función del estado actual de las tablas del libro de contabilidad. Si los hash calculados no coinciden con los resúmenes de entrada, se produce un error en la comprobación, lo que indica que los datos se han alterado e informa de todas las incoherencias detectadas.

Puesto que la comprobación del libro de contabilidad vuelve a calcular todos los hashes de las transacciones de la base de datos, puede ser un proceso que consume muchos recursos para las bases de datos con grandes cantidades de datos. La ejecución de la comprobación del libro de contabilidad solo debe realizarse cuando los usuarios necesitan comprobar la integridad de su base de datos y no de forma continua. Lo ideal es que la comprobación del libro de contabilidad solo se ejecute cuando la organización que hospeda los datos pase por una auditoría y necesite proporcionar evidencia criptográfica con respecto a la integridad de sus datos a otra parte. Para reducir el costo de comprobación, el libro de contabilidad expone opciones para comprobar tablas de libro de contabilidad individuales o solo un subconjunto del libro de contabilidad.

Para obtener más información sobre la comprobación del libro de contabilidad, vea [Administración de resúmenes y comprobación de base de datos](ledger-digest-management-and-database-verification.md).

## <a name="next-steps"></a>Pasos siguientes
 
- [Inicio rápido: Creación de una instancia de Azure SQL Database con el libro de contabilidad habilitado](ledger-create-a-single-database-with-ledger-enabled.md)
- [Tablas de libro de contabilidad actualizables](ledger-updatable-ledger-tables.md)   
- [Tablas de libro de contabilidad de solo anexión](ledger-append-only-ledger-tables.md)   
- [Libro de contabilidad de base de datos](ledger-database-ledger.md)   
- [Administración de resúmenes y comprobación de la base de datos](ledger-digest-management-and-database-verification.md)   
- [Limitaciones del libro de contabilidad de Azure SQL Database](ledger-limits.md)
- [Creación y uso de tablas de libro de contabilidad actualizables](ledger-how-to-updatable-ledger-tables.md)
- [Creación y uso de tablas de solo adición del libro de contabilidad](ledger-how-to-append-only-ledger-tables.md)
- [Acceso a los resúmenes almacenados en Azure Confidential Ledger (ACL)](ledger-how-to-access-acl-digest.md)
- [Comprobación de una tabla de libro de contabilidad para detectar alteraciones](ledger-verify-database.md)
