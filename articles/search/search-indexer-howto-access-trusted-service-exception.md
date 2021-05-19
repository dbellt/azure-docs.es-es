---
title: Acceso del indexador a Azure Storage mediante una excepción de servicio de confianza
titleSuffix: Azure Cognitive Search
description: Habilite el acceso a datos por parte de un indexador en Azure Cognitive Search a los datos almacenados de forma segura en Azure Storage.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 05/11/2021
ms.openlocfilehash: 16808be9b4a0c3a03a2a56a5255809b585aa404f
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109715801"
---
# <a name="indexer-access-to-azure-storage-using-the-trusted-service-exception-azure-cognitive-search"></a>Acceso del indexador a Azure Storage mediante una excepción de servicio de confianza (Azure Cognitive Search)

En un servicio Azure Cognitive Search, los indexadores que acceden a los datos de las cuentas de Azure Storage pueden utilizar la funcionalidad de [excepción de servicio de confianza](../storage/common/storage-network-security.md#exceptions) para acceder a los datos de forma segura. Este mecanismo ofrece a los clientes que no pueden conceder [al indexador acceso mediante las reglas de firewall de IP](search-indexer-howto-access-ip-restricted.md) una alternativa sencilla, segura y gratuita para acceder a los datos de las cuentas de almacenamiento.

> [!NOTE]
> La compatibilidad con el acceso a los datos de las cuentas de almacenamiento mediante una excepción de servicio de confianza se limita a Azure Blob Storage y al almacenamiento de Azure Data Lake Gen2. Azure Table Storage no se admite.

## <a name="step-1-configure-a-connection-using-a-managed-identity"></a>Paso 1: Configurar una conexión mediante una identidad administrada

Siga las instrucciones de [Configuración de una conexión a una cuenta de Azure Storage mediante una identidad administrada](search-howto-managed-identities-storage.md). Cuando haya terminado, habrá registrado el servicio Search en Azure Active Directory como un servicio de confianza y tendrá permisos concedidos en Azure Storage que proporcionan a la identidad de búsqueda permisos específicos para acceder a los datos o a la información.

> [!NOTE]
> Las instrucciones le enseñan un enfoque del portal para configurar Cognitive Search como un servicio de confianza. Para hacerlo mediante código, puede usar la [API REST](/rest/api/searchmanagement/services/createorupdate), [Azure PowerShell](search-manage-powershell.md#create-a-service-with-a-system-assigned-managed-identity) o la [CLI de Azure](search-manage-azure-cli.md#create-a-service-with-a-system-assigned-managed-identity).

## <a name="step-2-allow-trusted-microsoft-services-to-access-the-storage-account"></a>Paso 2: Permitir que los servicios de Microsoft de confianza accedan a la cuenta de almacenamiento

En Azure Portal, vaya a la pestaña **Firewalls y redes virtuales** de la cuenta de almacenamiento. Asegúrese de que la opción **Permitir que los servicios de Microsoft de confianza accedan a esta cuenta de almacenamiento** esté habilitada. Esta opción solo permitirá que la instancia de servicio de búsqueda específica tenga el acceso basado en rol adecuado a la cuenta de almacenamiento (autenticación segura) para acceder a los datos de dicha cuenta, aunque estén protegidos por las reglas de firewall de IP.

![Excepción de servicio de confianza](media\search-indexer-howto-secure-access\exception.png "Excepción de servicio de confianza")

Los indexadores ahora podrán acceder a los datos de la cuenta de almacenamiento, aunque la cuenta esté protegida mediante reglas de firewall de IP.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre los indexadores de Azure Storage:

- [Indexador de blobs de Azure](search-howto-indexing-azure-blob-storage.md)
- [Indexador de Azure Data Lake Storage Gen2](search-howto-index-azure-data-lake-storage.md)
- [Indexador de tablas de Azure](search-howto-indexing-azure-tables.md)