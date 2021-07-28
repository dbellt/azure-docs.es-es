---
title: 'Proveedores de almacenamiento de Durable Functions: Azure'
description: Obtenga información sobre los distintos proveedores de almacenamiento de Durable Functions y cómo se comparan.
author: cgillum
ms.topic: conceptual
ms.date: 05/05/2021
ms.author: azfuncdf
ms.openlocfilehash: bf50f0bdc3c8e654a3d2f780bb7f0c32533948eb
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110465806"
---
# <a name="durable-functions-storage-providers"></a>Proveedores de almacenamiento de Durable Functions

Durable Functions conserva automáticamente los parámetros de una función, los valores devueltos y otros estados en un almacenamiento duradero para garantizar una ejecución de confianza. La configuración predeterminada de Durable Functions almacena este estado del entorno de ejecución en una cuenta (clásica) de Azure Storage. Sin embargo, es posible configurar Durable Functions v2.0 y versiones posteriores para usar un proveedor de almacenamiento duradero alternativo.

Durable Functions es un conjunto de desencadenadores y enlaces de Azure Functions con tecnología interna de [Durable Task Framework](https://github.com/Azure/durabletask) (DTFx). DTFx admite varios proveedores de almacenamiento de back-end, incluido el proveedor de Azure Storage que usa Durable Functions. A partir de la **versión 2.4.3** de Durable Functions, los usuarios pueden configurar sus aplicaciones de funciones para que usen proveedores de almacenamiento de DTFx distintos del proveedor de Azure Storage.

> [!NOTE]
> La opción de usar proveedores de almacenamiento diferentes de Azure Storage se debe realizar con cuidado. La mayoría de las aplicaciones de funciones que se ejecutan en Azure deben usar el proveedor predeterminado de Azure Storage para Durable Functions. Sin embargo, hay importantes contrapartidas relacionadas con los costos, la escalabilidad y la administración de datos que se deben tener en cuenta al decidir si se debe usar un proveedor de almacenamiento alternativo. En este artículo se describen muchas de estas contrapartidas en detalle.

Se desarrollaron dos proveedores de almacenamiento de DTFx alternativos para su uso con Durable Functions: el proveedor de almacenamiento de _Netherite_ y el proveedor de almacenamiento de _Microsoft SQL Server (MSSQL)._ En este artículo se describen los tres proveedores admitidos, se comparan entre sí y se proporciona información básica sobre cómo empezar a usarlos.

## <a name="azure-storage"></a>Azure Storage

Azure Storage es el proveedor de almacenamiento predeterminado para Durable Functions. Usa colas, tablas y blobs para conservar la orquestación y el estado de la entidad. También usa blobs y concesiones de blobs para administrar particiones. En muchos casos, la cuenta de almacenamiento que se usa para almacenar el estado del entorno de ejecución de Durable Functions es la misma que la cuenta de almacenamiento predeterminada que usa Azure Functions (`AzureWebJobsStorage`). Sin embargo, también es posible configurar Durable Functions con una cuenta de almacenamiento independiente. El proveedor de Azure Storage está integrado en la extensión de Durable Functions y no tiene ninguna otra dependencia.

Entre las principales ventajas del proveedor de Azure Storage se incluyen:

* No se requiere ninguna configuración: puede usar la cuenta de almacenamiento que se creó automáticamente mediante la experiencia de configuración de la aplicación de funciones.
* Modelo de facturación sin servidor de menor costo: Azure Storage tiene un modelo de precios basado en el consumo basado completamente en el uso ([más información](durable-functions-billing.md#azure-storage-transactions)).
* Mejor compatibilidad con herramientas: Azure Storage ofrece emulación local multiplataforma y se integra con Visual Studio, Visual Studio Code y Azure Functions Core Tools.
* Opción más consolidad: Azure Storage era el back-end de almacenamiento original y más probado para Durable Functions.

El código fuente de los componentes de DTFx del proveedor de almacenamiento de Azure Storage se puede encontrar en el repositorio de GitHub [Azure/durabletask](https://github.com/Azure/durabletask/tree/main/src/DurableTask.AzureStorage).

> [!NOTE]
> Las cuentas estándar de uso general de Azure Storage son necesarias cuando se usa el proveedor de Azure Storage. No se admite ningún otro tipo de cuenta de almacenamiento. Se recomienda encarecidamente usar cuentas de almacenamiento de uso general v1 heredadas, ya que las cuentas de almacenamiento v2 más recientes pueden ser significativamente más costosas para las cargas de trabajo de Durable Functions. Para obtener más información sobre los tipos de cuentas de Azure Storage, consulte la documentación de [información general acerca de la cuenta de Azure Storage](../../storage/common/storage-account-overview.md).

## <a name="netherite-preview"></a><a name="netherite"></a>Netherite (versión preliminar)

[Microsoft Research](https://www.microsoft.com/research) diseñó y desarrolló el back-end de almacenamiento de Netherite. Usa [Azure Event Hubs](../../event-hubs/event-hubs-about.md) y la tecnología de base de datos [FASTER](https://www.microsoft.com/research/project/faster/) además de [blobs en páginas de Azure](../../storage/blobs/storage-blob-pageblob-overview.md). El diseño de Netherite permite un procesamiento de orquestaciones y entidades significativamente más elevado en comparación con otros proveedores. En algunos escenarios de punto de referencia, se ha demostrado que el rendimiento aumenta en más de un orden de magnitud en comparación con el proveedor predeterminado de Azure Storage.

Entre las principales ventajas del proveedor de Netherite se incluyen:

* Rendimiento significativamente mayor a un costo menor en comparación con otros proveedores de almacenamiento.
* Admite la optimización del rendimiento de precios, lo que le permite escalar verticalmente el rendimiento según sea necesario.
* Admite hasta 32 particiones de datos con SKU básicas y estándar de Event Hubs.
* Más rentable que otros proveedores para cargas de trabajo de alto rendimiento.

Puede obtener más información sobre los detalles técnicos del proveedor de almacenamiento de Netherite, incluida información sobre cómo empezar a usarlo, en la [documentación de Netherite](https://microsoft.github.io/durabletask-netherite). El código fuente del proveedor de almacenamiento de Netherite se puede encontrar en el repositorio de GitHub [microsoft/durabletask-netherite](https://github.com/microsoft/durabletask-netherite). También hay disponible una evaluación más detallada del proveedor de almacenamiento de Netherite en el siguiente documento de investigación: [Flujos de trabajo sin servidor con Durable Functions y Netherite](https://arxiv.org/abs/2103.00033).

> [!NOTE]
> El nombre de _Netherite_ tiene origen en el mundo de [Minecraft](https://minecraft.fandom.com/wiki/Netherite).

## <a name="microsoft-sql-server-mssql-preview"></a><a name="mssql"></a>Microsoft SQL Server (MSSQL) (versión preliminar)

El proveedor de almacenamiento de Microsoft SQL Server (MSSQL) conserva todo el estado en una base de datos de Microsoft SQL Server. Es compatible con las implementaciones locales y hospedadas en la nube de SQL Server, incluida [Azure SQL Database](../../azure-sql/database/sql-database-paas-overview.md).

Entre las principales ventajas del proveedor de MSSQL se incluyen:

* Admite entornos desconectados: no se requiere conectividad de Azure cuando se usa una instalación de SQL Server.
* Portátil en varios entornos y nubes, incluidos los entornos locales y hospedados en Azure.
* Coherencia de datos fuerte, habilitación de copias de seguridad y restauración y conmutación por error sin pérdida de datos.
* Compatibilidad nativa con el cifrado de datos personalizado (una característica de SQL Server).
* Se integra con las aplicaciones de base de datos existentes a través de procedimientos almacenados integrados.

Puede obtener más información sobre los detalles técnicos del proveedor de almacenamiento de MSSQL, incluida información sobre cómo empezar a usarlo, en la [documentación del proveedor de Microsoft SQL](https://microsoft.github.io/durabletask-mssql). El código fuente del proveedor de almacenamiento de MSSQL se puede encontrar en el repositorio de GitHub [microsoft/durabletask-mssql](https://github.com/microsoft/durabletask-mssql).

## <a name="configuring-alternate-storage-providers"></a>Configuración de proveedores de almacenamiento alternativos

La configuración de proveedores de almacenamiento alternativos suele ser un proceso de dos pasos:

1. Agregue el paquete NuGet adecuado a la aplicación de funciones (este requisito es temporal para las aplicaciones que usan agrupaciones de extensiones).
1. Actualice el archivo **host.json** para especificar qué proveedor de almacenamiento desea usar.

Si no hay ningún proveedor de almacenamiento configurado explícitamente en host.json, el proveedor de Azure Storage se habilitará de manera predeterminada.

### <a name="configuring-the-azure-storage-provider"></a>Configuración del proveedor de Azure Storage

El proveedor de Azure Storage es el proveedor de almacenamiento predeterminado y no requiere ninguna configuración explícita, referencias de paquetes NuGet ni referencias de agrupaciones de extensiones. Puede encontrar el conjunto completo de opciones de configuración de **host.json** [aquí](durable-functions-bindings.md#hostjson-settings), en la ruta de acceso `extensions/durableTask/storageProvider`.

### <a name="configuring-the-netherite-storage-provider"></a>Configuración del proveedor de almacenamiento de Netherite

Para usar el proveedor de almacenamiento de Netherite, primero debe agregar una referencia al paquete NuGet [Microsoft.Azure.DurableTask.Netherite.AzureFunctions](https://www.nuget.org/packages/Microsoft.Azure.DurableTask.Netherite.AzureFunctions) en el archivo **csproj** (aplicaciones .NET) o en el archivo **extensions.proj** (aplicaciones de JavaScript, Python y PowerShell).

> [!NOTE]
> El proveedor de almacenamiento de Netherite aún no se admite en las aplicaciones que usan [agrupaciones de extensiones](../functions-bindings-register.md#extension-bundles).

En el ejemplo siguiente de host.json se muestra la configuración mínima necesaria para habilitar el proveedor de almacenamiento de Netherite.

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "type": "Netherite",
        "storageConnectionName": "AzureWebJobsStorage",
        "eventHubsConnectionName": "EventHubsConnection"
      }
    }
  }
}
```

Para obtener instrucciones de configuración más detalladas, consulte la [documentación de introducción a Netherite](https://microsoft.github.io/durabletask-netherite/#/?id=getting-started).

### <a name="configuring-the-mssql-storage-provider"></a>Configuración del proveedor de almacenamiento de MSSQL

Para usar el proveedor de almacenamiento de MSSQL, primero debe agregar una referencia al paquete NuGet [Microsoft.DurableTask.SqlServer.AzureFunctions](https://www.nuget.org/packages/Microsoft.DurableTask.SqlServer.AzureFunctions) en el archivo **csproj** (aplicaciones .NET) o en el archivo **extensions.proj** (aplicaciones de JavaScript, Python y PowerShell).

> [!NOTE]
> El proveedor de almacenamiento de MSSQL aún no se admite en las aplicaciones que usan [agrupaciones de extensiones](../functions-bindings-register.md#extension-bundles).

En el ejemplo siguiente se muestra la configuración mínima necesaria para habilitar el proveedor de almacenamiento de MSSQL.

```json
{
  "version": "2.0",
  "extensions": {
    "durableTask": {
      "storageProvider": {
        "type": "mssql",
        "connectionStringName": "SQLDB_Connection"
      }
    }
  }
}
```

Para obtener instrucciones de configuración más detalladas, consulte la [documentación de introducción a MSSQL](https://microsoft.github.io/durabletask-mssql/#/quickstart).

## <a name="comparing-storage-providers"></a>Comparación de proveedores de almacenamiento

Hay muchos contrapartidas entre los distintos proveedores de almacenamiento admitidos. La tabla siguiente se puede usar para ayudarle a comprender estas contrapartidas y decidir qué proveedor de almacenamiento es mejor para sus necesidades.

| Proveedor de almacenamiento | Azure Storage | Netherite | MSSQL |
|- |-              |-          |-      |
| Estado de soporte técnico oficial | ✅ Disponible con carácter general | ⚠ Versión preliminar pública | ⚠ Versión preliminar pública |
| Dependencias externas | Cuenta de Azure Storage (uso general v1) | Azure Event Hubs<br/>Cuenta de Azure Storage (de uso general) | [SQL Server 2019](https://www.microsoft.com/sql-server/sql-server-2019) o Azure SQL Database |
| Opciones de emulación y desarrollo local | [Azurite v3.12+](../../storage/common/storage-use-azurite.md) (multiplataforma)<br/>[Emulador de Azure Storage](../../storage/common/storage-use-emulator.md) (solo Windows) | Emulación en memoria ([más información](https://microsoft.github.io/durabletask-netherite/#/emulation)) | SQL Server Developer Edition (admite contenedores de [Windows](/sql/database-engine/install-windows/install-sql-server), [Linux](/sql/linux/sql-server-linux-setup) y [Docker](/sql/linux/sql-server-linux-docker-container-deployment)) |
| Configuración de la central de tareas | Explícito | Explícito | Implícita de manera predeterminada ([más información](https://microsoft.github.io/durabletask-mssql/#/taskhubs)) |
| Rendimiento máximo | Moderada | Muy alto | Moderada |
| Escalabilidad horizontal máxima de orquestación o entidad (nodos) | 16 | 32 | N/D |
| Escalabilidad horizontal máxima de actividad (nodos) | N/D | 32 | N/D |
| Compatibilidad con el plan de consumo | ✅ Totalmente compatible | ❌ No se admite | ❌ No se admite |
| Compatibilidad con el plan Elástico Premium | ✅ Totalmente compatible | ⚠ Requiere [supervisión de la escala en tiempo de ejecución](../functions-networking-options.md#premium-plan-with-virtual-network-triggers) | ⚠ Requiere [supervisión de la escala en tiempo de ejecución](../functions-networking-options.md#premium-plan-with-virtual-network-triggers) |
| Compatibilidad con el escalado de [KEDA 2.0](https://keda.sh/)<br/>([más información](../functions-kubernetes-keda.md)) | ❌ No se admite | ❌ No se admite | ✅ Compatible con [Scaler de MSSQL](https://keda.sh/docs/scalers/mssql/) ([más información](https://microsoft.github.io/durabletask-mssql/#/scaling)) |
| Compatibilidad con [agrupaciones de extensiones](../functions-bindings-register.md#extension-bundles) (recomendado para aplicaciones que no son .NET) | ✅ Totalmente compatible | ❌ No se admite | ❌ No se admite |
| ¿Se puede configurar el precio y el rendimiento? | ❌ No | ✅ Sí (TU y CPU de Event Hubs) | ✅ Sí (vCPU de SQL) |
| Compatibilidad con entornos desconectados | ❌ Se requiere conectividad de Azure | ❌ Se requiere conectividad de Azure | ✅ Totalmente compatible |

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre el rendimiento y la escala de Durable Functions](durable-functions-perf-and-scale.md)
