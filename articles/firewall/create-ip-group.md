---
title: Creación de grupos de IP en Azure Firewall
description: Los grupos de IP permiten agrupar y administrar direcciones IP en las reglas de Azure Firewall.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 06/23/2020
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: e19c27c130b59339ec46038007eb82b3fd7ac16b
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693726"
---
# <a name="create-ip-groups"></a>Creación de grupos de IP

Los grupos de IP permiten agrupar y administrar direcciones IP en las reglas de Azure Firewall. Pueden tener una sola dirección IP, varias direcciones IP o uno o varios intervalos de direcciones IP.

## <a name="create-an-ip-group---azure-portal"></a>Creación de un grupo IP: Azure Portal

Para crear un grupo de IP con Azure Portal, siga estos pasos:

1. En la página principal de Azure Portal, seleccione **Crear un recurso**.
1. En el cuadro de búsqueda, escriba **Grupos de IP** y, a continuación, seleccione **Grupos de IP**.
1. Seleccione **Crear**.
1. Seleccione su suscripción.
1. seleccione un grupo de recursos o cree uno.
1. Escriba un nombre único para el grupo de IP y, después, seleccione una región.
1. Seleccione **Siguiente: Direcciones IP**.
1. Escriba una dirección IP, varias direcciones IP o intervalos de direcciones IP.

   Hay dos maneras de escribir direcciones IP:
   - Puede escribirlas manualmente.
   - Puede importarlas desde un archivo.

   Para importarlas desde un archivo, seleccione **Import from a file** (Importar de un archivo). Puede arrastrar el archivo al cuadro o seleccionar **Examinar archivos**. Si es necesario, puede revisar y editar las direcciones IP cargadas.

   Al escribir una dirección IP, el portal la valida para comprobar que no haya problemas de duplicados, superposición y formato.

1. Cuando termine, seleccione **Revisar y crear**.
1. Seleccione **Crear**.

## <a name="create-an-ip-group---azure-powershell"></a>Creación de un grupo de IP: Azure PowerShell

En este ejemplo se crea un grupo de IP con un prefijo de dirección y una dirección IP mediante Azure PowerShell:

```azurepowershell
$ipGroup = @{
    Name              = 'ipGroup'
    ResourceGroupName = 'ipGroupRG'
    Location          = 'West US'
    IpAddress         = @('10.0.0.0/24', '192.168.1.10') 
}

New-AzIpGroup @ipGroup
```

## <a name="create-an-ip-group---azure-cli"></a>Creación de un grupo de IP: CLI de Azure

En este ejemplo se crea un grupo de IP con un prefijo de dirección y una dirección IP mediante la CLI de Azure:

```azurecli-interactive
az network ip-group create \
    --name ipGroup \ 
    --resource-group ipGroupRG \
    --location westus \
    --ip-addresses '10.0.0.0/24' '192.168.1.10'
```

## <a name="next-steps"></a>Pasos siguientes

- [Más información sobre los grupos de IP](ip-groups.md)
