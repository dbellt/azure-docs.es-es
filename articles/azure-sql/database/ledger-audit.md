---
title: Eventos de auditoría de Azure SQL Database con tablas habilitadas para libro de contabilidad
description: Introducción a las funcionalidades de auditoría del libro de contabilidad de Azure SQL Database
ms.custom: ''
ms.date: 05/25/2021
ms.service: sql-database
ms.subservice: security
ms.reviewer: vanto
ms.topic: conceptual
author: JasonMAnderson
ms.author: janders
ms.openlocfilehash: eafca78dc22a01784eee79293fb7a2ec798818ad
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388782"
---
# <a name="azure-sql-database-audit-events-with-ledger-enabled-tables"></a>Eventos de auditoría de Azure SQL Database con tablas habilitadas para libro de contabilidad

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> El libro de contabilidad de Azure SQL Database se encuentra actualmente en **versión preliminar pública**.

Al realizar actividades forenses con tablas habilitadas para el libro de contabilidad, además de los datos capturados en la vista de libro de contabilidad y el libro de contabilidad de base de datos, se agregan identificadores de acción adicionales a los registros de auditoría de SQL.  En la tabla siguiente se describen estos nuevos eventos de registro de auditoría junto con las condiciones que los desencadenan.

## <a name="enable-ledger"></a>Habilitar libro de contabilidad

| Columna | Valor |
|--|--|
| **action_id** | ENLR |
| **name** | ENABLE LEDGER  |
| **class_desc** | OBJECT |
| **covering_action_desc** | NULL |
| **parent_class_desc** | DATABASE |
| **covering_parent_action_name** | LEDGER_OPERATION_GROUP |
| **configuration_level** | NULL |
| **configuration_group_name** | LEDGER_OPERATION_GROUP |
| **action_in_log** | 1 |

**Condición que desencadena el evento**: crear una tabla de libro de contabilidad o conversión de una tabla normal en una tabla de libro de contabilidad.

## <a name="alter-ledger"></a>Modificar libro de contabilidad

| Columna | Valor |
|--|--|
| **action_id** | ALLR |
| **name** | ALTER LEDGER |
| **class_desc** | OBJECT |
| **covering_action_desc** | NULL |
| **parent_class_desc** | DATABASE |
| **covering_parent_action_name** | LEDGER_OPERATION_GROUP |
| **configuration_level** | NULL |
| **configuration_group_name** | LEDGER_OPERATION_GROUP |
| **action_in_log** | 1 |

**Condición que desencadena el evento**: quitar o cambiar el nombre de una tabla de libro de contabilidad, convertir una tabla de libro de contabilidad en una tabla normal, agregar, quitar o cambiar el nombre de una columna de una tabla de libro de contabilidad.


## <a name="generate-ledger-digest"></a>Generar resumen de libro de contabilidad

| Columna | Valor |
|--|--|
| **action_id** | GDLR |
| **name** | GENERATE LEDGER DIGEST |
| **class_desc** | DATABASE |
| **covering_action_desc** | LEDGER_OPERATION_GROUP |
| **parent_class_desc** | SERVER |
| **covering_parent_action_name** | LEDGER_OPERATION_GROUP |
| **configuration_level** | NULL |
| **configuration_group_name** | LEDGER_OPERATION_GROUP  |
| **action_in_log** | 1 |

**Condición que desencadena el evento**: generar un resumen del libro de contabilidad.

## <a name="verify-ledger"></a>Comprobar libro de contabilidad

| Columna | Valor |
|--|--|
| **action_id** | VFLR |
| **name** | VERIFY LEDGER |
| **class_desc** | DATABASE |
| **covering_action_desc** | LEDGER_OPERATION_GROUP |
| **parent_class_desc** | SERVER |
| **covering_parent_action_name** | LEDGER_OPERATION_GROUP |
| **configuration_level** | NULL |
| **configuration_group_name** | LEDGER_OPERATION_GROUP |
| **action_in_log** | 1 |

**Condición que desencadena el evento**: comprobar un resumen del libro de contabilidad.

## <a name="ledger-operation-group"></a>Grupo de operaciones del libro de contabilidad

| Columna | Valor |
|--|--|
| **action_id** | OPLR |
| **name** | LEDGER_OPERATION_GROUP |
| **class_desc** | DATABASE |
| **covering_action_desc** | NULL |
| **parent_class_desc** | SERVER |
| **covering_parent_action_name** | NULL |
| **configuration_level** | GROUP |
| **configuration_group_name** | LEDGER_OPERATION_GROUP |
| **action_in_log** | 0 |

**Condición que desencadena el evento**: N/D.

| Columna | Valor |
|--|--|
| **action_id** | OPLR |
| **name** | LEDGER_OPERATION_GROUP |
| **class_desc** | SERVER |
| **covering_action_desc** | NULL |
| **parent_class_desc** | NULL |
| **covering_parent_action_name** | NULL |
| **configuration_level** | GROUP |
| **configuration_group_name** | LEDGER_OPERATION_GROUP |
| **action_in_log** | 0 |

**Condición que desencadena el evento**: N/D. 

## <a name="next-steps"></a>Pasos siguientes

- [Auditoría para Azure SQL Database y Azure Synapse Analytics](auditing-overview.md)
- [Introducción al libro de contabilidad de Azure SQL Database](ledger-overview.md)
- [Inicio rápido: Creación de una instancia de Azure SQL Database con el libro de contabilidad habilitado](ledger-create-a-single-database-with-ledger-enabled.md)
