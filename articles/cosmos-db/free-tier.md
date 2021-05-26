---
title: Nivel Gratis de Azure Cosmos DB
description: Use el nivel Gratis de Azure Cosmos DB para comenzar a usar, desarrollar y probar sus aplicaciones. Con el nivel Gratis, recibirá las primeras 1000 RU/s y 25 GB de almacenamiento en la cuenta de forma gratuita.
author: SnehaGunda
ms.service: cosmos-db
ms.topic: how-to
ms.date: 05/25/2021
ms.author: sngun
ms.openlocfilehash: faa931348d94575754b2d369b8ce50840f2f35a8
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386383"
---
# <a name="azure-cosmos-db-free-tier"></a>Nivel Gratis de Azure Cosmos DB 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

El nivel Gratis de Azure Cosmos DB hace que sea fácil empezar a usar, desarrollar y probar las aplicaciones, o incluso ejecutar pequeñas cargas de trabajo de producción de forma gratuita. Cuando el nivel Gratis esté habilitado en una cuenta, recibirá en ella las primeras 1000 RU/s y 25 GB de almacenamiento de forma gratuita. El rendimiento y el almacenamiento consumidos por encima de estos límites se facturan a un precio normal. El nivel Gratis está disponible para todas las cuentas de API con rendimiento aprovisionado, rendimiento de escalado automático y una o varias regiones de escritura.

El nivel Gratis se mantiene indefinidamente durante la vigencia de la cuenta e incluye todas las [ventajas y características](introduction.md#key-benefits) de una cuenta de Azure Cosmos DB normal. Estas ventajas incluyen almacenamiento y rendimiento ilimitados (RU/s), Acuerdos de Nivel de Servicio, alta disponibilidad, distribución global inmediata en todas las regiones de Azure, etc.

Puede tener una cuenta de Azure Cosmos DB de nivel Gratis por cada suscripción de Azure y debe elegir esta ventaja al crear la cuenta. Si no ve la opción para aplicar el descuento por nivel Gratis, significará que en otra cuenta de la suscripción ya se ha habilitado el nivel Gratis. Al crear una cuenta, se recomienda habilitar el descuento del nivel Gratis si está disponible.

> [!NOTE]
> El nivel Gratis no está disponible actualmente para las cuentas sin servidor.

## <a name="free-tier-with-shared-throughput-database"></a>Nivel gratis con base de datos de rendimiento compartido

En el modelo de rendimiento compartido, al aprovisionar el rendimiento en una base de datos, este se comparte entre todos sus contenedores. Al usar el nivel Gratis, puede aprovisionar una base de datos compartida con hasta 1000 RU/s de forma gratuita. Todos los contenedores de la base de datos compartirán el rendimiento. 

Al igual que con la cuenta normal, en la cuenta de nivel Gratis, una base de datos de rendimiento compartido puede tener hasta 25 contenedores. Las bases de datos adicionales con rendimiento compartido o contenedores con un rendimiento por encima de 1000 RU/s se facturan al precio normal. En una cuenta de nivel Gratis, puede crear hasta 5 bases de datos de rendimiento compartido.

## <a name="free-tier-with-azure-discount"></a>Nivel Gratis con descuento de Azure

El nivel Gratis de Azure Cosmos DB es compatible con la [cuenta gratuita de Azure](optimize-dev-test.md#azure-free-account). Para optar a esta ventaja, cree una cuenta de Azure Cosmos DB de nivel Gratis en la suscripción de la cuenta gratuita de Azure. Durante los primeros 12 meses, recibirá un descuento combinado de 1400 RU/s (1000 RU/s del nivel Gratis de Azure Cosmos DB y 400 RU/s de la cuenta gratuita de Azure) y 50 GB de almacenamiento (25 GB del nivel Gratis de Azure Cosmos DB y 25 GB de la cuenta gratuita de Azure). Transcurridos los 12 meses, seguirá recibiendo 1000 RU/s y 25 GB del nivel Gratis de Azure Cosmos DB durante el período de vigencia de la cuenta de Azure Cosmos DB. Para ver un ejemplo de cómo se apilan los cargos, consulte [Ejemplos de facturación con cuentas de nivel Gratis](understand-your-bill.md#azure-free-tier).

> [!NOTE]
> El nivel Gratis de Azure Cosmos DB es diferente de la cuenta gratuita de Azure. La cuenta gratuita de Azure ofrece créditos y recursos de Azure de forma gratuita durante un tiempo limitado. Al usar Azure Cosmos DB como parte de esta cuenta gratuita, obtendrá 25 GB de almacenamiento y 400 RU/s de rendimiento aprovisionado durante 12 meses.

## <a name="best-practices-to-keep-your-account-free"></a>Procedimientos recomendados para mantener la cuenta gratuita

Al usar el nivel Gratis de Azure Cosmos DB, para mantener la cuenta completamente gratuita, esta no debe tener ningún consumo adicional de RU/s o almacenamiento que no sea el que ofrece el nivel Gratis.

Por ejemplo, estas son algunas opciones que no generan un cargo mensual:

* Una base de datos con un rendimiento aprovisionado máximo de 1000 RU/s.
* Dos contenedores, uno con 400 RU/s como máximo y otro con un rendimiento aprovisionado máximo de 600 RU/s.
* Cuenta con 2 regiones donde una sola región tenga un contenedor con hasta 500 RU/s de rendimiento aprovisionado.

## <a name="create-an-account-with-free-tier"></a>Creación de una cuenta con el nivel Gratis

Puede crear una cuenta de nivel Gratis desde Azure Portal, PowerShell, la CLI o plantillas de Azure Resource Manager (ARM). Debe elegir el nivel Gratis al crear la cuenta, ya que, una vez creada, no se puede establecer.

### <a name="azure-portal"></a>Azure Portal

Al crear la cuenta mediante Azure Portal, establezca la opción **Apply Free Tier Discount** (Aplicar descuento del nivel Gratis) en **Aplicar**. Consulte el artículo [Creación de una cuenta con el nivel Gratis](create-cosmosdb-resources-portal.md) para obtener instrucciones paso a paso.

### <a name="arm-template"></a>Plantilla ARM

Para crear una cuenta de nivel Gratis mediante una plantilla de ARM, establezca la propiedad `"enableFreeTier": true`. Para ver la plantilla completa, consulte un ejemplo en [Implementación de una plantilla de ARM con el nivel Gratis](manage-with-templates.md#free-tier).

### <a name="cli"></a>CLI

Para crear una cuenta con el nivel Gratis mediante la CLI, establezca el parámetro `--enable-free-tier` en true:

```azurecli-interactive
# Create a free tier account for SQL API
az cosmosdb create \
    -n "Myaccount" \
    -g "MyResourcegroup" \
    --enable-free-tier true \
    --default-consistency-level "Session"
    
```

### <a name="powershell"></a>PowerShell

Para crear una cuenta con el nivel Gratis mediante Azure PowerShell, establezca el parámetro `-EnableFreeTier` en true:

```powershell-interactive
# Create a free tier account for SQL API. 
New-AzCosmosDBAccount -ResourceGroupName MyResourcegroup" `
    -Name "Myaccount" `
    -ApiKind "sql" `
    -EnableFreeTier true `
    -DefaultConsistencyLevel "Session" `
```

## <a name="next-steps"></a>Pasos siguientes

Después de crear una cuenta de nivel Gratis, puede empezar a compilar aplicaciones con Azure Cosmos DB siguiendo estos artículos:

* [Compilación de una aplicación de consola mediante el SDK de .NET V4](create-sql-api-dotnet-v4.md) para administrar recursos de Azure Cosmos DB.
* [Compilación de una aplicación web .NET mediante la API de Azure Cosmos DB para MongoDB](create-mongodb-dotnet.md)
* [Descargue un cuaderno de la galería](publish-notebook-gallery.md#download-a-notebook-from-the-gallery) y analice los datos.
* Obtenga más información sobre [la factura de Azure Cosmos DB](understand-your-bill.md).
