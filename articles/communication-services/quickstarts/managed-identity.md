---
title: Uso de identidades administradas en Communication Services
titleSuffix: An Azure Communication Services quickstart
description: Las identidades administradas le permiten autorizar a Azure Communication Services el acceso desde aplicaciones que se ejecutan en máquinas virtuales de Azure, aplicaciones de funciones y otros recursos.
services: azure-communication-services
author: peiliu
ms.service: azure-communication-services
ms.topic: how-to
ms.date: 05/27/2021
ms.author: peiliu
ms.reviewer: mikben
zone_pivot_groups: acs-js-csharp-java-python
ms.openlocfilehash: 3433aceb4bc9bda10c11602a7a45f28a5882f479
ms.sourcegitcommit: e832f58baf0b3a69c2e2781bd8e32d4f1ae932c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110585751"
---
# <a name="use-managed-identities"></a>Uso de identidades administradas
Comience a trabajar con Azure Communication Services mediante identidades administradas. Los SDK de SMS e identidad de Communication Services admiten la autenticación de Azure Active Directory (Azure AD) con [identidades administradas para los recursos de Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Este inicio rápido le muestra cómo autorizar el acceso a los SDK de identidad y SMS desde un entorno de Azure que admita identidades administradas. También se describe cómo probar el código en un entorno de desarrollo.

## <a name="prerequisites"></a>Prerrequisitos

- Una cuenta de Azure con una suscripción activa. [Cree su cuenta de forma gratuita](https://azure.microsoft.com/free).
- Un recurso de Azure Communication Services activo, consulte [Creación de un recurso de Communication Services](./create-communication-resource.md) si no tiene uno.
- Para enviar un SMS, necesitará un [número de teléfono](./telephony-sms/get-phone-number.md).
- Una identidad administrada de configuración para un entorno de desarrollo, consulte [Autorización del acceso con identidad administrada](./managed-identity-from-cli.md).

::: zone pivot="programming-language-csharp"
[!INCLUDE [.NET](./includes/managed-identity/managed-identity-net.md)]
::: zone-end

::: zone pivot="programming-language-javascript"
[!INCLUDE [JavaScript](./includes/managed-identity/managed-identity-js.md)]
::: zone-end

::: zone pivot="programming-language-java"
[!INCLUDE [Java](./includes/managed-identity/managed-identity-java.md)]
::: zone-end

::: zone pivot="programming-language-python"
[!INCLUDE [Python](./includes/managed-identity/managed-identity-python.md)]
::: zone-end

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre el control de acceso basado en rol de Azure](../../../articles/role-based-access-control/index.yml).
- [Más información sobre la biblioteca de identidades de Azure para .NET](/dotnet/api/overview/azure/identity-readme)
- [Creación de tokens de acceso de usuario](../quickstarts/access-tokens.md)
- [Enviar un mensaje SMS](../quickstarts/telephony-sms/send.md)
- [Más información acerca de SMS](../concepts/telephony-sms/concepts.md)
