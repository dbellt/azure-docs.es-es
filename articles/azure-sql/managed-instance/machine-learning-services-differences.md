---
title: Diferencias clave de Machine Learning Services
description: En este artículo se describen las principales diferencias entre Machine Learning Services de Azure SQL Managed Instance y Machine Learning Services de SQL Server.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: machine-learning
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: garyericson
ms.author: garye
ms.reviewer: sstein, davidph
manager: cgronlun
ms.date: 03/17/2021
ms.openlocfilehash: a322efdf5ef21912a8e29e9b3dea4bc041c61cca
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108131898"
---
# <a name="key-differences-between-machine-learning-services-in-azure-sql-managed-instance-and-sql-server"></a>Principales diferencias entre Machine Learning Services de Instancia administrada de Azure SQL y SQL Server

En este artículo se describen las pocas diferencias clave que hay entre [Machine Learning Services de Azure SQL Managed Instance](machine-learning-services-overview.md) y [Machine Learning Services de SQL Server](/sql/advanced-analytics/what-is-sql-server-machine-learning).

## <a name="language-support"></a>Compatibilidad con idiomas

Las versiones de Machine Learning Services tanto de SQL Managed Instance como de SQL Server admiten el [marco de extensibilidad](/sql/machine-learning/concepts/extensibility-framework) de Python y R. Las principales diferencias de la versión de SQL Managed Instance son:

- Solo se admiten Python y R. No se pueden agregar lenguajes externos como Java.

- Las versiones iniciales de Python y R son distintas:

  | Plataforma                   | Versión del entorno de ejecución de Python           | Versiones del entorno de ejecución de R                   |
  |----------------------------|----------------------------------|--------------------------------------|
  | Instancia administrada de Azure SQL | 3.7.2                            | 3.5.2                                |
  | SQL Server 2019            | 3.7.1                            | 3.5.2                                |
  | SQL Server 2017            | 3.5.2 y 3.7.2 (CU22 y posteriores) | 3.3.3 y 3.5.2 (CU22 y posteriores)     |
  | SQL Server 2016            | No disponible                    | 3.2.2 y 3.5.2 (SP2 CU14 y posteriores) |

## <a name="python-and-r-packages"></a>Paquetes de Python y R

En SQL Managed Instance no hay compatibilidad con los paquetes que dependen de entornos de ejecución externos (por ejemplo, Java) o que necesitan acceso a las API del sistema operativo para su instalación o uso.

Para obtener más información sobre la administración de paquetes de Python y R, vea:

- [Obtención de información de paquetes de Python](/sql/machine-learning/package-management/python-package-information?context=%2fazure%2fazure-sql%2fmanaged-instance%2fcontext%2fml-context&preserve-view=true&view=azuresqldb-mi-current)
- [Obtención de información de paquetes de R](/sql/machine-learning/package-management/r-package-information?context=%2fazure%2fazure-sql%2fmanaged-instance%2fcontext%2fml-context&preserve-view=true&view=azuresqldb-mi-current)

## <a name="resource-governance"></a>Regulación de recursos

En SQL Managed Instance, no es posible limitar los recursos de R mediante [Resource Governor](/sql/relational-databases/resource-governor/resource-governor?view=azuresqldb-mi-current&preserve-view=true) y no se admiten grupos de recursos externos.

De forma predeterminada, los recursos de R se establecen en un máximo del 20 % de los recursos de SQL Managed Instance disponibles cuando la extensibilidad está habilitada. Para cambiar este porcentaje predeterminado, cree una incidencia de soporte técnico de Azure en [https://azure.microsoft.com/support/create-ticket/](https://azure.microsoft.com/support/create-ticket/).

La extensibilidad se habilita con los siguientes comandos SQL (SQL Managed Instance se reiniciará y no estará disponible durante unos segundos):

```sql
sp_configure 'external scripts enabled', 1;
RECONFIGURE WITH OVERRIDE;
```

Para deshabilitar la extensibilidad y restaurar el 100 % de los recursos de memoria y CPU en SQL Server, use los siguientes comandos:

```sql
sp_configure 'external scripts enabled', 0;
RECONFIGURE WITH OVERRIDE;
```

El número total de recursos disponibles para SQL Managed Instance dependen del nivel de servicio que se elija. Para obtener más información, consulte [Azure SQL Database purchasing models](../database/purchasing-models.md) (Modelos de compra de Azure SQL Database).

### <a name="insufficient-memory-error"></a>Error de memoria insuficiente

El uso de la memoria depende de cuánta se use en los scripts de R y del número de consultas paralelas que se ejecutan. Si no hay suficiente memoria disponible para R, recibirá un mensaje de error. Los mensajes comunes de error son:

- `Unable to communicate with the runtime for 'R' script for request id: *******. Please check the requirements of 'R' runtime`
- `'R' script error occurred during execution of 'sp_execute_external_script' with HRESULT 0x80004004. ...an external script error occurred: "..could not allocate memory (0 Mb) in C function 'R_AllocStringBuffer'"`
- `An external script error occurred: Error: cannot allocate vector of size.`

Si recibe uno de estos errores, puede resolverlo escalando la base de datos a un nivel de servicio superior.

## <a name="sql-managed-instance-pools"></a>Grupos de Instancia administrada de SQL

En la actualidad, Machine Learning Services no es compatible con los [grupos de Azure SQL Managed Instance (versión preliminar)](instance-pools-overview.md).

## <a name="next-steps"></a>Pasos siguientes

- Vea la introducción, [Machine Learning Services de Instancia administrada de Azure SQL](machine-learning-services-overview.md).
- Para obtener información sobre cómo usar Python en Machine Learning Services, vea [Ejecución de scripts de Python](/sql/machine-learning/tutorials/quickstart-python-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true).
- Para obtener información sobre cómo usar R en Machine Learning Services, vea [Ejecución de scripts de R](/sql/machine-learning/tutorials/quickstart-r-create-script?context=/azure/azure-sql/managed-instance/context/ml-context&view=azuresqldb-mi-current&preserve-view=true).