---
title: Bibliotecas cliente para Azure Key Vault | Microsoft Docs
description: Bibliotecas cliente para Azure Key Vault
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 08/14/2020
ms.author: mbaldwin
ms.openlocfilehash: b8690b5ce3e2aee9a554d3e0682161b86264e124
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107818540"
---
# <a name="client-libraries-for-azure-key-vault"></a>Bibliotecas cliente para Azure Key Vault

Las bibliotecas cliente para Azure Key Vault permiten el acceso mediante programación a la funcionalidad de Key Vault con diversos lenguajes, como .NET, Python, Java y JavaScript.

## <a name="client-libraries-per-language-and-object"></a>Bibliotecas cliente por lenguaje y objeto

Cada SDK tiene bibliotecas cliente independientes para el almacén de claves, los secretos, las claves y los certificados, según la tabla siguiente.

| Idioma | Secretos | Claves | Certificados | Key Vault (plano de administración) |
|--|--|--|--|--|
| .NET | - [Referencia de API](/dotnet/api/azure.security.keyvault.secrets)<br>- [Paquete NuGet](https://www.nuget.org/packages/Azure.Security.KeyVault.Secrets/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Azure.Security.KeyVault.Secrets)<br>- [Inicio rápido](../secrets/quick-create-net.md) | - [Referencia de API](/dotnet/api/azure.security.keyvault.keys)<br>- [Paquete NuGet](https://www.nuget.org/packages/Azure.Security.KeyVault.Keys/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Azure.Security.KeyVault.Keys)<br>- [Inicio rápido](../keys/quick-create-net.md) | - [Referencia de API](/dotnet/api/azure.security.keyvault.certificates)<br>- [Paquete NuGet](https://www.nuget.org/packages/Azure.Security.KeyVault.Certificates/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Azure.Security.KeyVault.Certificates)<br>- [Inicio rápido](../certificates/quick-create-net.md) | - [Referencia de API](/dotnet/api/microsoft.azure.management.keyvault)<br>- [Paquete NuGet](https://www.nuget.org/packages/Microsoft.Azure.Management.KeyVault/)<br> - [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/keyvault/Microsoft.Azure.Management.KeyVault)|
| Python| - [Referencia de API](/python/api/overview/azure/keyvault-secrets-readme)<br>- [Paquete PyPi](https://pypi.org/project/azure-keyvault-secrets/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-secrets)<br>- [Inicio rápido](../secrets/quick-create-python.md) |- [Referencia de API](/python/api/overview/azure/keyvault-keys-readme)<br>- [Paquete PyPi](https://pypi.org/project/azure-keyvault-keys/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-keys)<br>- [Inicio rápido](../keys/quick-create-python.md) | - [Referencia de API](/python/api/overview/azure/keyvault-certificates-readme)<br>- [Paquete PyPi](https://pypi.org/project/azure-keyvault-certificates/)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-keyvault-certificates)<br>- [Inicio rápido](../certificates/quick-create-python.md) | - [Referencia de API](/python/api/azure-mgmt-keyvault/azure.mgmt.keyvault)<br> - [Paquete PyPi](https://pypi.org/project/azure-mgmt-keyvault/)<br> - [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/keyvault/azure-mgmt-keyvault)|
| Java | - [Referencia de API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-secrets/4.2.0/index.html)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-secrets)<br>- [Inicio rápido](../secrets/quick-create-java.md) |- [Referencia de API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-keys/4.2.0/index.html)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-keys)<br>- [Inicio rápido](../keys/quick-create-java.md) | - [Referencia de API](https://azuresdkdocs.blob.core.windows.net/$web/java/azure-security-keyvault-certificates/4.1.0/index.html)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/azure-security-keyvault-certificates)<br>- [Inicio rápido](../certificates/quick-create-java.md) |- [Referencia de API](/java/api/com.microsoft.azure.management.keyvault)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/keyvault/mgmt-v2016_10_01)|
| Node.js | - [Referencia de API](/javascript/api/@azure/keyvault-secrets/)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/keyvault-secrets)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/keyvault-secrets)<br>- [Inicio rápido](../secrets/quick-create-node.md) |- [Referencia de API](/javascript/api/@azure/keyvault-keys/)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/keyvault-keys)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/keyvault-keys)<br>- [Inicio rápido](../keys/quick-create-node.md)| - [Referencia de API](/javascript/api/@azure/keyvault-certificates/)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/keyvault-certificates)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/keyvault-certificates)<br>- [Inicio rápido](../certificates/quick-create-node.md) |  - [Referencia de API](/javascript/api/@azure/arm-keyvault/)<br>- [Paquete NPM](https://www.npmjs.com/package/@azure/arm-keyvault)<br>- [Código fuente de la biblioteca](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/keyvault/arm-keyvault)

## <a name="next-steps"></a>Pasos siguientes

- Consulte la [guía del desarrollador de Azure Key Vault](developers-guide.md).
- Más información acerca de la [Autenticación en Key Vault](authentication.md)
- Más información acerca de la [Protección del acceso a Key Vault](security-features.md)
