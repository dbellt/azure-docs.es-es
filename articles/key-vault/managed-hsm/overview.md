---
title: 'Introducción a HSM administrado de Azure: HSM administrado de Azure | Microsoft Docs'
description: HSM administrado de Azure es un servicio en la nube que protege las claves criptográficas de las aplicaciones en la nube.
services: key-vault
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: overview
ms.custom: mvc
ms.date: 04/01/2021
ms.author: mbaldwin
author: msmbaldwin
ms.openlocfilehash: a9df83535bf0ee7a6c484a7682392d2ad0ef59f7
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107815331"
---
# <a name="what-is-azure-key-vault-managed-hsm"></a>¿Qué es HSM administrado de Azure Key Vault?

HSM administrado de Azure Key Vault es un servicio en la nube que cumple los estándares totalmente administrado, de alta disponibilidad y de un solo inquilino que le permite proteger las claves criptográficas de las aplicaciones en la nube mediante HSM validados de **FIPS 140-2 de nivel 3**. Para obtener información sobre precios, consulte la sección Grupos de HSM administrados en la [página de precios de Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/). 

## <a name="why-use-managed-hsm"></a>Motivos para usar Managed HSM

### <a name="fully-managed-highly-available-single-tenant-hsm-as-a-service"></a>HSM totalmente administrado, de alta disponibilidad y de un solo inquilino como servicio

- **Totalmente administrado** El servicio controla el aprovisionamiento, la configuración, la revisión y el mantenimiento de HSM. 
- **Alta disponibilidad y resistencia a la zona** (donde se admiten zonas de disponibilidad): Cada clúster de HSM consta de varias particiones de HSM que abarcan al menos dos zonas de disponibilidad. Si se produce un error de hardware, las particiones de miembro de su clúster de HSM se migrarán automáticamente a los nodos correctos.
- **Inquilino único**: cada instancia de HSM administrada está dedicada a un solo cliente y consta de un clúster de varias particiones de HSM. Cada clúster de HSM usa un dominio de seguridad independiente específico del cliente que aísla de forma criptográfica el clúster de HSM de cada cliente.


### <a name="access-control-enhanced-data-protection--compliance"></a>Control de acceso, protección de datos mejorada y cumplimiento

- **Administración de claves centralizada**: administre claves críticas y de alto valor en su organización en un solo lugar. Con permisos pormenorizados por clave, controle el acceso a todas las claves en base al principio de "acceso con menos privilegios".
- **Control de acceso aislado**: El modelo de control de acceso "RBAC local" de HSM administrado permite a los administradores de clústeres de HSM designados tener un control completo sobre los HSM que ni siquiera los administradores de grupos de administración, de suscripciones o de grupos de recursos pueden invalidar.
- **HSM validados con FIPS 140-2 nivel 3**: Proteja sus datos y cumpla los requisitos de cumplimiento con los HSM validados con FIPS (Federal Information Protection Standard) 140-2 de nivel 3. Los HSM administrados usan adaptadores HSM de Marvell LiquidSecurity.
- **Supervisión y auditoría**: completamente integrado con Azure Monitor. Obtenga registros completos de toda la actividad a través de Azure Monitor. Use Azure Log Analytics para los análisis y las alertas.
- **Residencia de datos**: HSM administrado no almacena ni procesa datos del cliente fuera de la región en la que el cliente implementa la instancia de HSM.

### <a name="integrated-with-azure-and-microsoft-paassaas-services"></a>Integración con los servicios PaaS y SaaS de Microsoft y Azure 

- Genere (o importe mediante [BYOK](hsm-protected-keys-byok.md)) claves y úselas para cifrar los datos en reposo en servicios de Azure como [Azure Storage](../../storage/common/customer-managed-keys-overview.md), [Azure SQL](../../azure-sql/database/transparent-data-encryption-byok-overview.md)y [Azure Information Protection](/azure/information-protection/byok-price-restrictions).

### <a name="uses-same-api-and-management-interfaces-as-key-vault"></a>Usa la misma API y las mismas interfaces de administración que Key Vault

- Migre fácilmente las aplicaciones existentes que usen un almacén (multiinquilino) para utilizar HSM administrados.
- Utilice los mismos patrones de implementación y desarrollo de aplicaciones para todas las aplicaciones, con independencia de la solución de administración de claves en uso: almacenes multiinquilino o HSM administrados con un solo inquilino.

### <a name="import-keys-from-your-on-premise-hsms"></a>Importación de claves desde HSM locales

- Genere claves protegidas con HSM en el HSM local e impórtelas de forma segura en HSM administrados.

## <a name="next-steps"></a>Pasos siguientes
- Consulte [Quickstart: Aprovisionamiento y activación de un HSM administrado mediante la CLI de Azure](quick-create-cli.md) para crear y activar un HSM administrado.
- Consulte [Procedimientos recomendados para usar Azure Key Vault Managed HSM](best-practices.md)
