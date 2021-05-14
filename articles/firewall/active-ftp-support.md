---
title: Compatibilidad de FTP activo con Azure Firewall
description: De manera predeterminada, el FTP activo está deshabilitado en Azure Firewall. Puede habilitarlo mediante PowerShell, la CLI y las plantillas de ARM.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: conceptual
ms.date: 04/12/2021
ms.author: victorh
ms.openlocfilehash: 18b3680e47fe808413998144259e033a4cbcaa27
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107864476"
---
# <a name="azure-firewall-active-ftp-support"></a>Compatibilidad de FTP activo con Azure Firewall

Con el FTP activo, el servidor FTP inicia la conexión de datos con puerto de datos del cliente FTP designado. Normalmente, los firewalls de la red del cliente bloquean una solicitud de conexión externa a un puerto de cliente interno. Para más información, consulte [FTP activo frente a FTP pasivo, la explicación definitiva](https://slacksite.com/other/ftp.html).

De manera predeterminada, la compatibilidad con el FTP activo está deshabilitada en Azure Firewall para protegerse frente a ataques de devolución FTP mediante el comando `PORT` de FTP. Sin embargo, puede habilitar el FTP activo durante la implementación mediante Azure PowerShell, la CLI de Azure o una plantilla de Azure ARM.

Para admitir FTP en modo activo, es necesario abrir los siguientes puertos TCP:

- Puerto 21 del servidor FTP desde cualquier lugar (el cliente inicia la conexión)
- Puerto 21 del servidor FTP hacia los puertos > 1023 (el servidor responde al puerto de control del cliente)
- Puerto 20 del servidor FTP hacia los puertos > 1023 en clientes (el servidor inicia la conexión de datos hacia el puerto de datos del cliente)
- Puerto 20 del servidor FTP desde los puertos > 1023 en los clientes (el cliente envía los ACK al puerto de datos del servidor)

## <a name="azure-powershell"></a>Azure PowerShell

Para realizar la implementación mediante Azure PowerShell, use el parámetro `AllowActiveFTP`. Para más información, consulte [Creación de un firewall con Permitir FTP activo](/powershell/module/az.network/new-azfirewall#16---create-a-firewall-with-allow-active-ftp-).

## <a name="azure-cli"></a>Azure CLI

Para realizar la implementación mediante la CLI de Azure, use el parámetro `--allow-active-ftp`. Para más información, consulte [az network firewall create](/cli/azure/network/firewall#az_network_firewall_create-optional-parameters). 

## <a name="azure-resource-manager-arm-template"></a>Plantilla de Azure Resource Manager (ARM)

Para realizar la implementación con una plantilla de ARM, use el campo `AdditionalProperties`:

```json
"additionalProperties": {
            "Network.FTP.AllowActiveFTP": "True"
        },
```
Para obtener más información, consulte [Microsoft.Network azureFirewalls](/azure/templates/microsoft.network/azurefirewalls).

## <a name="next-steps"></a>Pasos siguientes

Para aprender a implementar una instancia de Azure Firewall, consulte [Implementación y configuración de Azure Firewall mediante Azure PowerShell](deploy-ps.md).