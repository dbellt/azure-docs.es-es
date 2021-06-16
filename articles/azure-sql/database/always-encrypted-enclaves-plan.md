---
title: Plan de atestación y enclaves de Intel SGX en Azure SQL Database
description: Planee la implementación de Always Encrypted con enclaves seguros en Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.devlang: ''
ms.topic: conceptual
author: jaszymas
ms.author: jaszymas
ms.reviwer: vanto
ms.date: 01/15/2021
ms.openlocfilehash: ccf4d00aa48edfc5cbe8df894d7d1a28387ecff2
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111411598"
---
# <a name="plan-for-intel-sgx-enclaves-and-attestation-in-azure-sql-database"></a>Plan de atestación y enclaves de Intel SGX en Azure SQL Database

[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

> [!NOTE]
> Always Encrypted con enclaves seguros para Azure SQL Database está actualmente en **versión preliminar pública.**

[Always Encrypted con enclaves seguros](/sql/relational-databases/security/encryption/always-encrypted-enclaves) en Azure SQL Database usa enclaves de [Intel software Guard Extensions (Intel SGX)](https://itpeernetwork.intel.com/microsoft-azure-confidential-computing/) y requiere [Microsoft Azure Attestation](/sql/relational-databases/security/encryption/always-encrypted-enclaves#secure-enclave-attestation).

## <a name="plan-for-intel-sgx-in-azure-sql-database"></a>Plan de Intel SGX en Azure SQL Database

Intel SGX es una tecnología de entorno de ejecución de confianza basado en hardware. Intel SGX está disponible para las bases de datos que usan el [modelo de núcleo virtual](service-tiers-sql-database-vcore.md) y la generación de hardware de [serie DC](service-tiers-sql-database-vcore.md?#dc-series). Por lo tanto, para asegurarse de que puede usar Always Encrypted con enclaves seguros en la base de datos, debe seleccionar la generación de hardware de serie DC al crear la base de datos. También puede actualizar la base de datos existente para usar la generación de hardware de serie DC.

> [!NOTE]
> Intel SGX no está disponible en las generaciones de hardware que no sean de serie DC. Por ejemplo, Intel SGX no está disponible para el hardware de Gen5 ni para las bases de datos que usan el [modelo de DTU](service-tiers-dtu.md).

> [!IMPORTANT]
> Antes de configurar la generación de hardware de serie DC para la base de datos, compruebe la disponibilidad regional de la serie DC y asegúrese de que comprende sus limitaciones de rendimiento. Para obtener información detallada, vea [Serie DC](service-tiers-sql-database-vcore.md#dc-series).

## <a name="plan-for-attestation-in-azure-sql-database"></a>Plan de atestación en Azure SQL Database

[Microsoft Azure Attestation](../../attestation/overview.md) (versión preliminar) es una solución para la atestación de entornos de ejecución de confianza (TEE), incluidos los enclaves de Intel SGX en Azure SQL Database mediante la generación de hardware de serie DC.

Para usar Azure Attestation para la atestación de enclaves de Intel SGX en Azure SQL Database, debe:

1. Cree un [proveedor de atestación](../../attestation/basic-concepts.md#attestation-provider) y configúrelo con una directiva de atestación. 

2. Conceda al servidor lógico de Azure SQL acceso al proveedor de atestación creado.

## <a name="roles-and-responsibilities-when-configuring-sgx-enclaves-and-attestation"></a>Roles y responsabilidades al configurar la atestación y los enclaves de SGX

La configuración del entorno para que sea compatible con los enclaves de Intel SGX y la atestación de Always Encrypted en Azure SQL Database implica la configuración de componentes de tipos diferentes: Microsoft Azure Attestation, Azure SQL Database y aplicaciones que desencadenan la atestación de enclaves. Los usuarios realizan la configuración de los componentes de cada tipo y asumen uno de los roles siguientes:

- Administrador de atestación: crea un proveedor de atestación en Microsoft Azure Attestation, crea la directiva de atestación, concede acceso al servidor lógico de Azure SQL al proveedor de atestación y comparte la dirección URL de atestación que apunta a la directiva con los administradores de la aplicación.
- Administrador de Azure SQL Database: habilita los enclaves de SGX en las bases de datos seleccionando la generación de hardware de serie DC y proporciona al administrador de atestación la identidad del servidor lógico de Azure SQL que necesita tener acceso al proveedor de atestación.
- Administrador de aplicaciones: configura las aplicaciones con la dirección URL de atestación obtenida del administrador de atestación.

En entornos de producción (donde se controlan datos confidenciales reales), es importante que la organización se adhiera a la separación de roles al configurar la atestación, donde cada rol distinto es asumido por usuarios diferentes. En concreto, si el objetivo de implementar Always Encrypted en la organización es reducir el área expuesta a ataques asegurándose de que los administradores de Azure SQL Database no puedan tener acceso a los datos confidenciales, los administradores de Azure SQL Database no deben controlar las directivas de atestación.

## <a name="next-steps"></a>Pasos siguientes

- [Habilitación de Intel SGX para Azure SQL Database](always-encrypted-enclaves-enable-sgx.md)

## <a name="see-also"></a>Consulte también

- [Tutorial: Introducción a Always Encrypted con enclaves seguros en Azure SQL Database](always-encrypted-enclaves-getting-started.md)