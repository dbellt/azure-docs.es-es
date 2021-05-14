---
title: Operaciones de plano de control y de plano de datos
description: Describe la diferencia entre las operaciones de plano de control y de plano de datos. Las operaciones de plano de control se gestionan mediante Azure Resource Manager. Las operaciones de plano de datos se gestionan mediante un servicio.
ms.topic: conceptual
ms.date: 09/10/2020
ms.openlocfilehash: 40bb3291e4eec589c88bcd6ffd0f94a718ce8d1f
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108132096"
---
# <a name="azure-control-plane-and-data-plane"></a>Plano de control y plano de datos de Azure

Las operaciones de Azure se pueden dividir en dos categorías: plano de control y plano de datos. En este artículo se describen las diferencias entre estos dos tipos de operaciones.

El plano de control se utiliza para administrar los recursos de su suscripción. El plano de datos se usa para utilizar las funcionalidades expuestas por la instancia de un tipo de recurso.

Por ejemplo:

* Crea una máquina virtual mediante el plano de control. Una vez creada la máquina virtual, puede interactuar con ella mediante las operaciones de plano de datos, como Protocolo de escritorio remoto (RDP).

* Crea una cuenta de almacenamiento mediante el plano de control. El plano de datos se usa para leer y escribir datos en la cuenta de almacenamiento.

* Crea una base de datos de Cosmos mediante el plano de control. Para consultar datos en la base de datos de Cosmos, utilice el plano de datos.

## <a name="control-plane"></a>Plano de control

Todas las solicitudes de las operaciones de plano de control se envían a la dirección URL de Azure Resource Manager. Esta dirección URL varía en función del entorno de Azure.

* En el caso de Azure Global, la dirección URL es `https://management.azure.com`.
* En el caso de Azure Government, la dirección URL es `https://management.usgovcloudapi.net/`.
* En el caso de Azure Germany, la dirección URL es `https://management.microsoftazure.de/`.
* En el caso de Microsoft Azure China 21Vianet, la dirección URL es `https://management.chinacloudapi.cn`.

Para saber qué operaciones usan la dirección URL de Azure Resource Manager, consulte [API REST de Azure](/rest/api/azure/). Por ejemplo, la [operación de creación o actualización](/rest/api/mysql/flexibleserver(preview)/servers/update) para MySql es una operación de plano de control porque la dirección URL de la solicitud es:

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{serverName}/databases/{databaseName}?api-version=2017-12-01
```

Azure Resource Manager gestiona todas las solicitudes del plano de control. Aplica automáticamente las características de Azure que ha implementado para administrar los recursos como, por ejemplo:

* [Control de acceso basado en roles de Azure (Azure RBAC)](../../role-based-access-control/overview.md)
* [Azure Policy](../../governance/policy/overview.md)
* [Bloqueos de administración](lock-resources.md)
* [Registros de actividad](view-activity-logs.md)

Después de autenticar la solicitud, Azure Resource Manager la envía al proveedor de recursos, que completa la operación.

El plano de control incluye dos escenarios para la gestión de solicitudes: "campo verde" y "campo marrón". El campo verde hace referencia a recursos nuevos. El campo marrón hace referencia a recursos existentes. A medida que implementa recursos, Azure Resource Manager entiende cuándo se deben crear nuevos recursos y cuándo se deben actualizar los recursos existentes. No tiene que preocuparse de que se creen recursos idénticos.

## <a name="data-plane"></a>Plano de datos

Las solicitudes de operaciones de plano de datos se envían a un punto de conexión específico de la instancia. Por ejemplo, la [operación Detectar idioma](../../cognitive-services/text-analytics/how-tos/text-analytics-how-to-language-detection.md) en Cognitive Services es una operación de plano de datos porque la dirección URL de la solicitud es:

```http
POST {Endpoint}/text/analytics/v2.0/languages
```

Las operaciones de plano de datos no se limitan a API REST. Pueden requerir credenciales adicionales, como iniciar sesión en una máquina virtual o un servidor de bases de datos.

Es posible que las características que exigen la administración y el gobierno no se apliquen a las operaciones de plano de datos. Debe tener en cuenta las distintas formas en que los usuarios interactúan con sus soluciones. Por ejemplo, un bloqueo que impide que los usuarios eliminen una base de datos no impide que los usuarios eliminen datos a través de consultas.

Puede usar algunas directivas para controlar las operaciones de plano de datos. Para más información, consulte [Modos del proveedor de recursos (versión preliminar)](../../governance/policy/concepts/definition-structure.md#resource-provider-modes) en Azure Policy.

## <a name="next-steps"></a>Pasos siguientes

* Para ver información general de Azure Resource Manager, consulte [¿Qué es Azure Resource Manager?](overview.md)

* Para más información sobre el efecto de las definiciones de directivas en los recursos nuevos y los recursos existentes, consulte [Evaluación del efecto de una nueva definición de Azure Policy](../../governance/policy/concepts/evaluate-impact.md).