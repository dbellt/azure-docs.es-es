---
title: Cumplimiento de los requisitos de residencia de datos en Azure Cosmos DB
description: Aprenda a cumplir los requisitos de residencia de datos en Azure Cosmos DB para que los datos y las copias de seguridad permanezcan en una sola región.
author: kanshiG
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 04/05/2021
ms.author: govindk
ms.reviewer: sngun
ms.openlocfilehash: c5f8a4361774368e3934d1e2b16c8311876f5caa
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491872"
---
# <a name="how-to-meet-data-residency-requirements-in-azure-cosmos-db"></a>Cumplimiento de los requisitos de residencia de datos en Azure Cosmos DB

En Azure Cosmos DB, puede configurar los datos y las copias de seguridad para que permanezcan en una sola región para cumplir los[ requisitos de residencia.](https://azure.microsoft.com/en-us/global-infrastructure/data-residency/)

## <a name="residency-requirements-for-data"></a>Requisitos de residencia para los datos

En Azure Cosmos DB, debe configurar explícitamente la replicación de datos entre regiones. Obtenga información sobre cómo configurar la replicación geográfica mediante [Azure Portal](how-to-manage-database-account.md#addremove-regions-from-your-database-account) y la [CLI de Azure](scripts/cli/common/regions.md). Para cumplir los requisitos de residencia de datos, puede crear una directiva de Azure que permita que regiones concretas impidan la replicación de datos en las regiones no deseadas.

## <a name="residency-requirements-for-backups"></a>Requisitos de residencia para las copias de seguridad

**Copias de seguridad en modo continuo**: estas copias de seguridad residen de manera predeterminada, ya que se almacenan en el almacenamiento con redundancia local o con redundancia de zona. Para más información, consulte el artículo [Copia de seguridad continua](continuous-backup-restore-portal.md).

**Copias de seguridad en modo periódico**: de manera predeterminada, las copias de seguridad en modo periódico se almacenarán en el almacenamiento con redundancia geográfica. En el caso de los modos de copia de seguridad periódica, puede configurar la redundancia de datos en el nivel de cuenta. Existen tres opciones de redundancia para el almacenamiento de copia de seguridad. Estas son redundancia local, redundancia de zona o redundancia geográfica. Para obtener más información, consulte [Configuración de la redundancia de copia de seguridad](configure-periodic-backup-restore.md#configure-backup-interval-retention) mediante el portal.

## <a name="use-azure-policy-to-enforce-the-residency-requirements"></a>Uso de Azure Policy para aplicar los requisitos de residencia

Si tiene que cumplir requisitos de residencia de datos que le imponen que mantenga todos los datos en una única región de Azure, puede aplicar copias de seguridad con redundancia de zona o con redundancia local para su cuenta mediante Azure Policy.  También puede aplicar una directiva en la que las cuentas de Azure Cosmos DB no se replican geográficamente en otras regiones.

Azure Policy es un servicio que puede usar para crear, asignar y administrar directivas que aplican reglas a los recursos de Azure. Azure Policy le permite mantener esos recursos conforme a lo establecido en los estándares corporativos y los contratos de nivel de servicio. Para más información, consulte cómo usar [Azure Policy](policy.md) para implementar la gobernanza y los controles de recursos de Azure Cosmos DB.

## <a name="next-steps"></a>Pasos siguientes

* Configure y administre la copia de seguridad periódica mediante [Azure Portal](configure-periodic-backup-restore.md)
* Configure y administre la copia de seguridad continua mediante [Azure Portal](continuous-backup-restore-portal.md), [PowerShell](continuous-backup-restore-powershell.md), la [CLI](continuous-backup-restore-command-line.md) o [Azure Resource Manager](continuous-backup-restore-template.md).
