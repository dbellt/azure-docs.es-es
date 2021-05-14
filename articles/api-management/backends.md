---
title: Back-ends de Azure API Management | Microsoft Docs
description: Más información sobre los back-ends personalizados de API Management
services: api-management
documentationcenter: ''
author: dlepow
editor: ''
ms.service: api-management
ms.topic: article
ms.date: 01/29/2021
ms.author: apimpm
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: a0ef3a2c1f2f1fc5cdf00737d1984f6cb13c40d0
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107813027"
---
# <a name="backends-in-api-management"></a>Back-ends de API Management

Un *back-end* (o *back-end de API*) de API Management es un servicio HTTP que implementa la API de front-end y sus operaciones.

Al importar ciertas API, API Management configura el back-end de la API automáticamente. Por ejemplo, API Management configura el back-end al importar una [especificación de OpenAPI](import-api-from-oas.md), [API de SOAP](import-soap-api.md) o recursos de Azure como una [aplicación de funciones de Azure](import-function-app-as-api.md) o una [aplicación lógica](import-logic-app-as-api.md) desencadenada mediante HTTP.

API Management también admite el uso de otros recursos de Azure, como un [clúster de Service Fabric](how-to-configure-service-fabric-backend.md) o un servicio personalizado, como back-end de API. El uso de estos back-ends personalizados requiere una configuración adicional, por ejemplo, para autorizar las credenciales de las solicitudes al servicio back-end y para definir las operaciones de la API. Estos back-ends se configuran y administran en Azure Portal o mediante las API o las herramientas de Azure.

Después de crear un back-end, puede hacer referencia a la dirección URL del back-end en las API. Use la directiva [`set-backend-service`](api-management-transformation-policies.md#SetBackendService) para redirigir una solicitud de API entrante al back-end personalizado en lugar de al back-end predeterminado para esa API.

## <a name="benefits-of-backends"></a>Ventajas de los back-ends

Un back-end personalizado tiene varias ventajas, entre las que se incluyen:

* Ofrece un resumen de la información sobre el servicio back-end y promueve la reutilización entre API y una gobernanza mejorada.  
* Fácil de usar mediante la configuración de una directiva de transformación en una API existente.
* Aprovecha la funcionalidad de API Management para mantener secretos en Azure Key Vault si los [valores con nombre](api-management-howto-properties.md) están configurados para la autenticación de parámetros de consulta o encabezado.

## <a name="next-steps"></a>Pasos siguientes

* Configuración de un [back-end de Service Fabric](how-to-configure-service-fabric-backend.md) mediante Azure Portal.
* Los back-ends también se pueden configurar mediante la [API REST](/rest/api/apimanagement) de API Management, [Azure PowerShell](/powershell/module/az.apimanagement/new-azapimanagementbackend) o [plantillas de Azure Resource Manager](../service-fabric/service-fabric-tutorial-deploy-api-management.md).

