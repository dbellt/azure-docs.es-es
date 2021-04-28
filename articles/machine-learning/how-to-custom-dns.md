---
title: Uso de un servidor DNS personalizado
titleSuffix: Azure Machine Learning
description: Cómo configurar un servidor DNS personalizado para trabajar con un área de trabajo de Azure Machine Learning y un punto de conexión privado.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.reviewer: larryfr
ms.author: jhirono
author: jhirono
ms.date: 04/01/2021
ms.topic: how-to
ms.custom: contperf-fy21q3
ms.openlocfilehash: 2a55fedd0fe059e7bff8203924389956dce36f3b
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107889815"
---
# <a name="how-to-use-your-workspace-with-a-custom-dns-server"></a>Uso de un área de trabajo con un servidor DNS personalizado

Al usar un área de trabajo de Azure Machine Learning con un punto de conexión privado, hay [varias maneras de controlar la resolución de nombres DNS](../private-link/private-endpoint-dns.md). De manera predeterminada, Azure controla automáticamente la resolución de nombres para el área de trabajo y el punto de conexión privado. Si, en cambio, __usa un servidor DNS personalizado propio__, tendrá que crear manualmente entradas DNS o usar reenviadores condicionales para el área de trabajo.

> [!IMPORTANT]
> En este artículo solo se explica cómo buscar el nombre de dominio completo (FQDN) y las direcciones IP para estas entradas; NO se proporciona información sobre cómo configurar los registros DNS para estos elementos. Consulte la documentación del software de DNS para obtener información sobre cómo agregar registros.

## <a name="prerequisites"></a>Prerrequisitos

- Una instancia de Azure Virtual Network que use [su propio servidor DNS](../virtual-network/virtual-networks-name-resolution-for-vms-and-role-instances.md#name-resolution-that-uses-your-own-dns-server).

- Un área de trabajo de Azure Machine Learning con un punto de conexión privado. Para más información, consulte [Creación de un área de trabajo de Azure Machine Learning](how-to-manage-workspace.md).

- Familiaridad con el uso del [aislamiento de red durante el entrenamiento e inferencia](./how-to-network-security-overview.md).

- Familiaridad con la [configuración de la zona DNS de puntos de conexión privados de Azure](../private-link/private-endpoint-dns.md).

- De manera opcional, la [CLI de Azure](/cli/azure/install-azure-cli) o [Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="public-regions"></a>Regiones públicas

La lista siguiente contiene los nombres de dominio completos (FQDN) que usa el área de trabajo si se encuentra en una región pública:

* `<workspace-GUID>.workspace.<region>.cert.api.azureml.ms`
* `<workspace-GUID>.workspace.<region>.api.azureml.ms`
* `ml-<workspace-name, truncated>-<region>-<workspace-guid>.notebooks.azure.net`

    > [!NOTE]
    > El nombre del área de trabajo de este nombre de dominio completo puede estar truncado. El truncamiento se realiza para mantener 63 caracteres `ml-<workspace-name, truncated>-<region>-<workspace-guid>`.
* `<instance-name>.<region>.instances.azureml.ms`

    > [!NOTE]
    > * Solo se puede acceder a la instancia de proceso desde dentro de la red virtual.
    > * La dirección IP de este FQDN **no** es la dirección IP de la instancia de proceso. En su lugar, use la dirección IP privada del punto de conexión privado del área de trabajo (la dirección IP de las entradas `*.api.azureml.ms`).

## <a name="azure-china-21vianet-regions"></a>Regiones de Azure China 21Vianet

Los siguientes nombres de dominio completo son para regiones de Azure China 21Vianet:

* `<workspace-GUID>.workspace.<region>.cert.api.ml.azure.cn`
* `<workspace-GUID>.workspace.<region>.api.ml.azure.cn`
* `ml-<workspace-name, truncated>-<region>-<workspace-guid>.notebooks.chinacloudapi.cn`

    > [!NOTE]
    > El nombre del área de trabajo de este nombre de dominio completo puede estar truncado. El truncamiento se realiza para mantener 63 caracteres `ml-<workspace-name, truncated>-<region>-<workspace-guid>`.
* `<instance-name>.<region>.instances.ml.azure.cn`
## <a name="find-the-ip-addresses"></a>Búsqueda de direcciones IP

Para buscar las direcciones IP internas de los FQDN en la red virtual, use uno de los métodos siguientes:

> [!NOTE]
> Los nombres de dominio completos y las direcciones IP serán diferentes en función de la configuración. Por ejemplo, el valor de GUID en el nombre de dominio será específico del área de trabajo.

# <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

```azurecli
az network private-endpoint show --endpoint-name <endpoint> --resource-group <resource-group> --query 'customDnsConfigs[*].{FQDN: fqdn, IPAddress: ipAddresses[0]}' --output table
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
$workspaceDns=Get-AzPrivateEndpoint -Name <endpoint> -resourcegroupname <resource-group>
$workspaceDns.CustomDnsConfigs | format-table
```

# <a name="azure-portal"></a>[Azure Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), seleccione el __área de trabajo__ de Azure Machine Learning.
1. En la sección __Configuración__, seleccione __Conexiones de punto de conexión privado__.
1. Seleccione el vínculo en la columna __Punto de conexión privado__ que se muestra.
1. Al final de la página se muestra una lista de los nombres de dominio completos (FQDN) y las direcciones IP del punto de conexión privado del área de trabajo.

:::image type="content" source="./media/how-to-custom-dns/private-endpoint-custom-dns.png" alt-text="Lista de los FQDN en el portal":::

---

La información devuelta por todos los métodos es la misma; una lista de los FQDN y las direcciones IP privadas para los recursos. El ejemplo siguiente es de una región global de Azure:

| FQDN | Dirección IP |
| ----- | ----- |
| `fb7e20a0-8891-458b-b969-55ddb3382f51.workspace.eastus.api.azureml.ms` | `10.1.0.5` |
| `ml-myworkspace-eastus-fb7e20a0-8891-458b-b969-55ddb3382f51.notebooks.azure.net` | `10.1.0.6` |

> [!IMPORTANT]
> El punto de conexión privado no enumera algunos FQDN, pero estos son necesarios para el área de trabajo en eastus, southcentralus y westus2. Estos FQDN se muestran en la tabla siguiente y también se deben agregar al servidor DNS o a una zona de DNS privado de Azure:
>
> * `<workspace-GUID>.workspace.<region>.cert.api.azureml.ms`
> * `<workspace-GUID>.workspace.<region>.experiments.azureml.net`
> * `<workspace-GUID>.workspace.<region>.modelmanagement.azureml.net`
> * `<workspace-GUID>.workspace.<region>.aether.ms`
> * Si tiene una instancia de proceso, use `<instance-name>.<region>.instances.azureml.ms`, donde `<instance-name>` es el nombre de la instancia de proceso. Use la dirección IP privada del punto de conexión privado del área de trabajo. Solo se puede acceder a la instancia de proceso desde dentro de la red virtual.
>
> Para todas estas direcciones IP, use la misma dirección que las entradas `*.api.azureml.ms` devueltas en los pasos anteriores.

En la tabla siguiente se muestran las direcciones IP de ejemplo de las regiones de Azure China 21Vianet:

| FQDN | Dirección IP |
| ----- | ----- |
| `52882c08-ead2-44aa-af65-08a75cf094bd.workspace.chinaeast2.api.ml.azure.cn` | `10.1.0.5` |
| `ml-mype-pltest-chinaeast2-52882c08-ead2-44aa-af65-08a75cf094bd.notebooks.chinacloudapi.cn` | `10.1.0.6` |
## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre el uso de Azure Machine Learning con una red virtual, consulte la [información general sobre la red virtual](how-to-network-security-overview.md).

Para obtener más información sobre la integración de puntos de conexión privados en la configuración de DNS, vea [Configuración de DNS para puntos de conexión privados de Azure](../private-link/private-endpoint-dns.md).
