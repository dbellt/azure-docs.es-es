---
title: Límites y limitación de solicitudes
description: En este artículo se describe cómo usar la limitación con las solicitudes de Azure Resource Manager cuando se han alcanzado los límites de suscripción.
ms.topic: conceptual
ms.date: 12/15/2020
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: cb562d6f6489ff30c6b940963a20974eb987b031
ms.sourcegitcommit: 52491b361b1cd51c4785c91e6f4acb2f3c76f0d5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108322192"
---
# <a name="throttling-resource-manager-requests"></a>Limitación de solicitudes de Resource Manager

En este artículo se describe cómo Azure Resource Manager regula las solicitudes. Se muestra cómo hacer el seguimiento del número de solicitudes que quedan para alcanzar el límite y cómo responder cuando se alcance.

La regulación se produce en dos niveles. Azure Resource Manager regula las solicitudes para la suscripción y el inquilino. Si la solicitud está por debajo de los límites de regulación para la suscripción y el inquilino, Resource Manager dirige la solicitud al proveedor de recursos. El proveedor de recursos aplica los límites de regulación adaptados a sus operaciones. En la imagen siguiente se muestra cómo se aplica la regulación a medida que una solicitud va del usuario a Azure Resource Manager y al proveedor de recursos.

![Regulación de solicitudes](./media/request-limits-and-throttling/request-throttling.svg)

## <a name="subscription-and-tenant-limits"></a>Límites de la suscripción y el inquilino

Cada operación de nivel de suscripción y de nivel de inquilino está sujeta a límites de regulación. Las solicitudes de la suscripción son aquellas en las que se pasa el identificador de la suscripción; por ejemplo, las solicitudes para recuperar los grupos de recursos de la suscripción. Las solicitudes del inquilino no incluyen el identificador de la suscripción; por ejemplo, las solicitudes para recuperar ubicaciones válidas de Azure.

Los límites de regulación predeterminados por hora se muestran en la tabla siguiente.

| Ámbito | Operaciones | Límite |
| ----- | ---------- | ------- |
| Subscription | Lecturas | 12000 |
| Subscription | Eliminaciones | 15000 |
| Subscription | Escrituras | 1200 |
| Inquilino | Lecturas | 12000 |
| Inquilino | Escrituras | 1200 |

Estas cifras están limitadas a la entidad de seguridad (usuario o aplicación) que realiza las solicitudes y al id. de suscripción o de inquilino. Si las solicitudes proceden de más de una entidad de seguridad, el límite en toda la suscripción o inquilino es superior a 12 000 y 1200 por hora.

Estos límites se aplican a cada instancia de Azure Resource Manager. En cada región de Azure hay varias instancias, y Azure Resource Manager está implementado en todas las regiones de Azure.  Por lo tanto, en la práctica, los límites son superiores a estos. Normalmente, las solicitudes de un usuario se controlan mediante diferentes instancias de Azure Resource Manager.

## <a name="resource-provider-limits"></a>Límites del proveedor de recursos

Los proveedores de recursos aplican sus propios límites de regulación. Dado que Resource Manager regula por el id. de entidad de seguridad y por instancia de Resource Manager, es posible que el proveedor de recursos reciba más solicitudes que los límites predeterminados de la sección anterior.

En esta sección se describen los límites de regulación de algunos proveedores de recursos ampliamente utilizados.

### <a name="storage-throttling"></a>Limitación del almacenamiento

[!INCLUDE [azure-storage-limits-azure-resource-manager](../../../includes/azure-storage-limits-azure-resource-manager.md)]

### <a name="network-throttling"></a>Limitación de la red

El proveedor de recursos Microsoft.Network aplica los siguientes límites de regulación:

| Operación | Límite |
| --------- | ----- |
| Escritura/eliminación (PUT) | 1000 por cada 5 minutos |
| Lectura (GET) | 10000 por cada 5 minutos |

> [!NOTE]
> **DNS privado de Azure** tiene un límite de 500 operaciones de lectura (GET) cada 5 minutos.
>

### <a name="compute-throttling"></a>Regulación de proceso

Para obtener información sobre los límites de regulación para las operaciones de proceso, consulte [Solución de errores de limitación de API - Proceso](/troubleshoot/azure/virtual-machines/troubleshooting-throttling-errors).

Para comprobar las instancias de máquina virtual dentro de un conjunto de escalado de máquinas virtuales, use las [operaciones de conjuntos de escalado de máquinas virtuales](/rest/api/compute/virtualmachinescalesetvms). Por ejemplo, use las [máquinas virtuales del conjunto de escalado de máquinas virtuales: lista](/rest/api/compute/virtualmachinescalesetvms/list) con parámetros para comprobar el estado de energía de las instancias de máquina virtual. Esta API reduce el número de solicitudes.

### <a name="azure-resource-graph-throttling"></a>Regulación de Azure Resource Graph

[Azure Resource Graph](../../governance/resource-graph/overview.md) limita el número de solicitudes para sus operaciones. Los pasos descritos en este artículo para determinar las solicitudes restantes y sobre cómo responder cuando se alcanza el límite también se aplican a Resource Graph. Sin embargo, Resource Graph establece su límite y frecuencia de restablecimiento propios. Para más información, consulte [Encabezados de limitación de Resource Graph](../../governance/resource-graph/concepts/guidance-for-throttled-requests.md#understand-throttling-headers).

### <a name="other-resource-providers"></a>Otros proveedores de recursos

Para más información acerca de la limitación en otros proveedores de recursos, consulte:

* [Guía de las limitaciones de Azure Key Vault](../../key-vault/general/overview-throttling.md)
* [Solución de problemas de AKS](../../aks/troubleshooting.md#im-receiving-429---too-many-requests-errors)

## <a name="error-code"></a>Código de error

Cuando se alcanza este límite, recibirá el código de estado HTTP **429 Demasiadas solicitudes**. En la respuesta se incluye el valor **Retry-After** que especifica el número de segundos que debe esperar (o estar en estado de suspensión) la aplicación antes de enviar la siguiente solicitud. Si envía una solicitud antes de que haya transcurrido el tiempo especificado en el valor de reintento, la solicitud no se procesará y se devolverá un nuevo valor de reintento.

Después de esperar el tiempo especificado, también puede cerrar y volver a abrir la conexión a Azure. Al restablecer la conexión, puede conectarse a una instancia diferente de Azure Resource Manager.

Si utiliza una instancia de Azure SDK, es posible que el SDK tenga una configuración de reintento automático. Para más información, consulte la [Guía de reintentos para servicios de Azure](/azure/architecture/best-practices/retry-service-specific).

Algunos proveedores de recursos devuelven 429 para informar de un problema temporal. El problema podría ser una condición de sobrecarga que no se deba directamente a la solicitud. O bien, podría representar un error temporal sobre el estado del recurso de destino o del recurso dependiente. Por ejemplo, el proveedor de recursos de red devuelve 429 con el código de error **RetryableErrorDueToAnotherOperation** cuando otra operación bloquea el recurso de destino. Para determinar si el error proviene de la regulación o de una condición temporal, vea los detalles del error en la respuesta.

## <a name="remaining-requests"></a>Solicitudes restantes

Puede determinar el número de solicitudes restantes examinando los encabezados de respuesta. Las solicitudes de lectura devuelven un valor en el encabezado para el número de solicitudes de lectura restantes. Las solicitudes de escritura incluyen un valor para el número de solicitudes de escritura restantes. En la tabla siguiente se describen los encabezados de respuesta que puede examinar para esos valores:

| Encabezado de respuesta | Descripción |
| --- | --- |
| x-ms-ratelimit-Remaining-Subscription-Reads |Lecturas restantes del ámbito de la suscripción. Este valor lo devuelven las operaciones de lectura. |
| x-ms-ratelimit-Remaining-Subscription-Writes |Escrituras restantes del ámbito de la suscripción. Este valor lo devuelven las operaciones de escritura. |
| x-ms-ratelimit-Remaining-tenant-Reads |Lecturas restantes del ámbito del inquilino |
| x-ms-ratelimit-Remaining-tenant-Writes |Escrituras restantes del ámbito del inquilino |
| x-ms-ratelimit-Remaining-Subscription-Resource-Requests |Solicitudes de tipos de recursos restantes del ámbito de la suscripción<br /><br />Este valor de encabezado solo se devuelve si un servicio ha invalidado el límite predeterminado. Resource Manager agrega este valor en lugar de las lecturas o escrituras de la suscripción. |
| x-ms-ratelimit-Remaining-Subscription-Resource-Entities-Read |Solicitudes de colección de tipos de recursos restantes del ámbito de la suscripción<br /><br />Este valor de encabezado solo se devuelve si un servicio ha invalidado el límite predeterminado. Este valor proporciona el número de solicitudes de colección restantes (recursos de lista). |
| x-ms-ratelimit-Remaining-tenant-Resource-Requests |Solicitudes de tipos de recurso restantes del ámbito del inquilino<br /><br />Este encabezado solo se agrega para las solicitudes en el nivel del inquilino, y solo si un servicio ha invalidado el límite predeterminado. Resource Manager agrega este valor en lugar de las lecturas o escrituras del inquilino. |
| x-ms-ratelimit-Remaining-tenant-Resource-Entities-Read |Solicitudes de colección de tipos de recursos restantes del ámbito del inquilino<br /><br />Este encabezado solo se agrega para las solicitudes en el nivel del inquilino, y solo si un servicio ha invalidado el límite predeterminado. |

El proveedor de recursos también puede devolver encabezados de respuesta con información sobre las solicitudes restantes. Para obtener información sobre los encabezados de respuesta devueltos por el proveedor de recursos de proceso, vea [Encabezados de respuesta que informan sobre la frecuencia de llamadas](/troubleshoot/azure/virtual-machines/troubleshooting-throttling-errors#call-rate-informational-response-headers).

## <a name="retrieving-the-header-values"></a>Recuperación de los valores de encabezado

El proceso de recuperación de estos valores de encabezado del código o el script es igual que el de cualquier valor de encabezado. 

Por ejemplo, en **C#**, recupere el valor del encabezado de un objeto **HttpWebResponse** denominado "**response**"con el código siguiente:

```cs
response.Headers.GetValues("x-ms-ratelimit-remaining-subscription-reads").GetValue(0)
```

En **PowerShell**, recupere el valor del encabezado de una operación Invoke-WebRequest.

```powershell
$r = Invoke-WebRequest -Uri https://management.azure.com/subscriptions/{guid}/resourcegroups?api-version=2016-09-01 -Method GET -Headers $authHeaders
$r.Headers["x-ms-ratelimit-remaining-subscription-reads"]
```

Para obtener un ejemplo completo de PowerShell, consulte [comprobación de los límites de Resource Manager para una suscripción](https://github.com/Microsoft/csa-misc-utils/tree/master/psh-GetArmLimitsViaAPI).

Si quiere ver las solicitudes restantes de depuración, puede proporcionar el parámetro **-Debug** en el cmdlet **PowerShell**.

```powershell
Get-AzResourceGroup -Debug
```

Que devuelve muchos valores, incluido el siguiente valor de respuesta:

```output
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
OK

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-reads: 11999
```

Para obtener los límites de escritura, utilice una operación de escritura: 

```powershell
New-AzResourceGroup -Name myresourcegroup -Location westus -Debug
```

Esta operación devuelve muchos valores, incluidos los siguientes:

```output
DEBUG: ============================ HTTP RESPONSE ============================

Status Code:
Created

Headers:
Pragma                        : no-cache
x-ms-ratelimit-remaining-subscription-writes: 1199
```

En la **CLI de Azure**, recupere el valor del encabezado mediante la opción más detallada.

```azurecli
az group list --verbose --debug
```

Esta operación devuelve muchos valores, incluidos los siguientes:

```output
msrest.http_logger : Response status: 200
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Content-Encoding': 'gzip'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'Vary': 'Accept-Encoding'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-reads': '11998'
```

Para obtener los límites de escritura, utilice una operación de escritura: 

```azurecli
az group create -n myresourcegroup --location westus --verbose --debug
```

Esta operación devuelve muchos valores, incluidos los siguientes:

```output
msrest.http_logger : Response status: 201
msrest.http_logger : Response headers:
msrest.http_logger :     'Cache-Control': 'no-cache'
msrest.http_logger :     'Pragma': 'no-cache'
msrest.http_logger :     'Content-Length': '163'
msrest.http_logger :     'Content-Type': 'application/json; charset=utf-8'
msrest.http_logger :     'Expires': '-1'
msrest.http_logger :     'x-ms-ratelimit-remaining-subscription-writes': '1199'
```

## <a name="next-steps"></a>Pasos siguientes

* Para obtener un ejemplo completo de PowerShell, consulte [comprobación de los límites de Resource Manager para una suscripción](https://github.com/Microsoft/csa-misc-utils/tree/master/psh-GetArmLimitsViaAPI).
* Para más información acerca de límites y cuotas, consulte [Límites, cuotas y restricciones de suscripción y servicios de Microsoft Azure](../../azure-resource-manager/management/azure-subscription-service-limits.md).
* Para obtener información sobre el control de solicitudes asincrónicas de REST, vea [Seguimiento de las operaciones asincrónicas de Azure](async-operations.md).
