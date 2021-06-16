---
title: Administración de una conexión de punto de conexión privado en Azure
description: Aprenda a administrar conexiones de punto de conexión privado en Azure
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: how-to
ms.date: 09/16/2019
ms.author: allensu
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d9e402b2015370eca5782ec91dd6afb8f7c6d962
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110701269"
---
# <a name="manage-a-private-endpoint-connection"></a>Administrar una conexión de punto de conexión privado
Azure Private Link trabaja en un modelo de flujo de llamadas de aprobación en el que el consumidor del servicio Private Link puede solicitar una conexión al proveedor de servicios para consumir el servicio. A partir de ese momento, el proveedor de servicios puede decidir si va a permitir al consumidor conectarse o no. Azure Private Link permite a los proveedores de servicios administrar la conexión de punto de conexión privado en sus recursos. En este artículo se proporcionan instrucciones sobre cómo administrar las conexiones de punto de conexión privado.

![Administración de puntos de conexión privados](media/manage-private-endpoint/manage-private-endpoint.png)

Hay dos métodos de aprobación de conexión entre los que puede elegir un consumidor del servicio Private Link:
- **Automático**: si el consumidor del servicio tiene permisos de Azure RBAC en el recurso de proveedor de servicios, el consumidor puede elegir el método de aprobación automático. En este caso, cuando el recurso de proveedor de servicios recibe la solicitud, no se requerirá ninguna acción del proveedor de servicios y la conexión se aprobará automáticamente. 
- **Manual**: por el contrario, si el consumidor del servicio no tiene permisos de Azure RBAC en el recurso de proveedor de servicios, el consumidor puede elegir el método de aprobación manual. En este caso, la solicitud de conexión aparece en los recursos de servicio como **Pendiente**. El proveedor de servicios debe aprobar manualmente la solicitud antes de que puedan establecerse las conexiones. En los casos manuales, el consumidor del servicio también puede especificar un mensaje con la solicitud para proporcionar más contexto al proveedor de servicios. El proveedor de servicios tiene las siguientes opciones entre las que elegir para todas las conexiones de punto de conexión privado: **Aprobar**, **Rechazar**, **Quitar**.

En la siguiente tabla se muestran las distintas acciones del proveedor de servicios y los estados de conexión resultantes para los puntos de conexión privados.  El proveedor de servicios también puede cambiar el estado de conexión de la conexión de punto de conexión privado en un momento posterior sin la intervención del consumidor. La acción actualizará el estado del punto de conexión en el lado del consumidor. 


|Acción del proveedor de servicios   |Estado de punto de conexión privado del consumidor del servicio   |Descripción   |
|---------|---------|---------|
|None    |    Pending     |    La conexión se crea manualmente y está pendiente de aprobación por parte del propietario del recurso de Private Link.       |
|Aprobación    |  Aprobado       |  La conexión se aprobó de forma automática o manual y está lista para usarse.     |
|Reject     | Rechazada        | El propietario del recurso de vínculo privado rechazó la conexión.        |
|Remove    |  Escenario desconectado       | El propietario del recurso de Private Link quitó la conexión, el punto de conexión privado se vuelve informativo y debe eliminarse para la limpieza.        |
|   |         |         |
   
## <a name="manage-private-endpoint-connections-on-azure-paas-resources"></a>Administración de conexiones de punto de conexión privado en los recursos de PaaS de Azure
El portal es el método preferido para administrar conexiones de punto de conexión privado en los recursos de PaaS de Azure. Actualmente, no tenemos compatibilidad con PowerShell o la CLI para administrar conexiones en los recursos de PaaS de Azure.
1. Inicie sesión en Azure Portal en https://portal.azure.com.
2. Vaya al centro de Private Link.
3. En **Recursos**, seleccione el tipo de recurso que desea para administrar las conexiones de punto de conexión privado.
4. Para cada uno de sus tipos de recurso, puede ver el número de conexiones de punto de conexión privado asociadas a los mismos. Puede filtrar los recursos según sea necesario.
5. Seleccione las conexiones de punto de conexión privado.  En las conexiones mostradas, seleccione la conexión que desea administrar. 
6. Puede cambiar el estado de la conexión seleccionando entre las opciones de la parte superior.

## <a name="manage-private-endpoint-connections-on-a-customerpartner-owned-private-link-service"></a>Administración de conexiones de punto de conexión privado en un servicio Private Link propiedad del cliente o asociado

Azure PowerShell y la CLI de Azure son los métodos preferidos para administrar conexiones de punto de conexión privado en servicios de asociados de Microsoft o servicios propiedad del cliente. Actualmente, no tenemos compatibilidad del portal para administrar conexiones en un servicio Private Link.  
 
### <a name="powershell"></a>PowerShell 
  
Use los siguientes comandos de PowerShell para administrar conexiones de punto de conexión privado.  
#### <a name="get-private-link-connection-states"></a>Obtención de estados de conexión de Private Link 
Use el cmdlet `Get-AzPrivateLinkService` para obtener las conexiones de punto de conexión privado y sus estados.  
```azurepowershell
Get-AzPrivateLinkService -Name myPrivateLinkService -ResourceGroupName myResourceGroup 
 ```
 
#### <a name="approve-a-private-endpoint-connection"></a>Aprobación de una conexión de punto de conexión privado 
 
Use el cmdlet `Approve-AzPrivateEndpointConnection` para aprobar una conexión de punto de conexión privado. 
 
```azurepowershell
Approve-AzPrivateEndpointConnection -Name myPrivateEndpointConnection -ResourceGroupName myResourceGroup -ServiceName myPrivateLinkService
```
 
#### <a name="deny-private-endpoint-connection"></a>Denegación de la conexión de punto de conexión privado 
 
Use el cmdlet `Deny-AzPrivateEndpointConnection` para rechazar una conexión de punto de conexión privado. 
```azurepowershell
Deny-AzPrivateEndpointConnection -Name myPrivateEndpointConnection -ResourceGroupName myResourceGroup -ServiceName myPrivateLinkService 
```
#### <a name="remove-private-endpoint-connection"></a>Eliminación de la conexión de punto de conexión privado 
 
Use el cmdlet `Remove-AzPrivateEndpointConnection` para quitar una conexión de punto de conexión privado. 
```azurepowershell
Remove-AzPrivateEndpointConnection -Name myPrivateEndpointConnection1 -ResourceGroupName myResourceGroup -ServiceName myPrivateLinkServiceName 
```
 
### <a name="azure-cli"></a>Azure CLI 
 
Use `az network private-link-service update` para administrar sus conexiones de punto de conexión privado. El estado de conexión se especifica en el parámetro ```azurecli connection-status```. 
```azurecli
az network private-link-service connection update -g myResourceGroup -n myPrivateEndpointConnection1 --service-name myPLS --connection-status Approved 
```

   

## <a name="next-steps"></a>Pasos siguientes
- [Más información sobre los puntos de conexión privados](private-endpoint-overview.md)
 
