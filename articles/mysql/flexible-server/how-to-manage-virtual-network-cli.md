---
title: 'Administración de redes virtuales de Azure Database for MySQL mediante la CLI de Azure: servidor flexible'
description: 'Creación y administración de redes virtuales de Azure Database for MySQL: servidor flexible mediante la CLI de Azure'
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: 7da8062f18d737af9d19df54863bc56c7268910c
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107776924"
---
# <a name="create-and-manage-virtual-networks-for-azure-database-for-mysql---flexible-server-using-the-azure-cli"></a>Creación y administración de redes virtuales de Azure Database for MySQL: servidor flexible mediante la CLI de Azure

> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

Se admiten dos métodos de conectividad de red mutuamente excluyentes para conectarse a su servidor flexible de Azure Database for MySQL. Las dos opciones son las siguientes:

- Acceso público (direcciones IP permitidas)
- Acceso privado (integración con red virtual)

En este artículo nos vamos a centrar en la creación de un servidor de MySQL con **acceso privado (integración con red virtual)** mediante la CLI de Azure. Con *acceso privado (Integración con red virtual)* , puede implementar su servidor flexible en su propio [Azure Virtual Network](../../virtual-network/virtual-networks-overview.md). Las redes de Azure Virtual Network proporcionan una comunicación de red privada y segura. En el acceso privado, las conexiones con el servidor de MySQL están restringidas a dentro de la red virtual. Para obtener más información al respecto, consulte [Acceso privado (Integración con red virtual)](./concepts-networking.md#private-access-vnet-integration).

En Azure Database for MySQL: servidor flexible, solo puede implementar el servidor en una red virtual y una subred durante la creación del servidor. Una vez implementado el servidor flexible en una red virtual y una subred, no se puede trasladar a otra red virtual, a una subred o a *acceso público (direcciones IP permitidas)* .

## <a name="launch-azure-cloud-shell"></a>Inicio de Azure Cloud Shell

[Azure Cloud Shell](../../cloud-shell/overview.md) es un shell interactivo gratuito que puede usar para ejecutar los pasos de este artículo. Tiene las herramientas comunes de Azure preinstaladas y configuradas para usarlas en la cuenta.

Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior derecha de un bloque de código. También puede abrir Cloud Shell en una pestaña independiente acudiendo a [https://shell.azure.com/bash](https://shell.azure.com/bash). Seleccione **Copiar** para copiar los bloques de código, péguelos en Cloud Shell y, después, seleccione **Entrar** para ejecutarlos.

Si prefiere instalar y usar la CLI de forma local, en este inicio rápido se requiere la versión 2.0 o posterior de la CLI de Azure. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Requisitos previos

Deberá iniciar sesión en la cuenta con el comando [az login](/cli/azure/reference-index#az_login). Tenga en cuenta la propiedad **id**, que hace referencia al **id. de suscripción** de su cuenta de Azure.

```azurecli-interactive
az login
```

Seleccione la suscripción específica en su cuenta mediante el comando [az account set](/cli/azure/account#az_account_set). Anote el valor de **id** de la salida de **az login** para usarlo como valor del argumento **subscription** del comando. Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Para obtener todas las suscripciones, use [az account list](/cli/azure/account#az_account_list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-azure-database-for-mysql-flexible-server-using-cli"></a>Creación de Azure Database for MySQL: servidor flexible mediante la CLI
Puede usar el comando `az mysql flexible-server` para crear el servidor flexible con *acceso privado (Integración con red virtual)* . Este comando usa acceso privado (Integración con red virtual) como método de conectividad predeterminado. Si no se proporciona ninguna, se creará una red virtual y una subred. También puede proporcionar la red virtual y la subred ya existentes con el id. de subred. <!-- You can provide the **vnet**,**subnet**,**vnet-address-prefix** or**subnet-address-prefix** to customize the virtual network and subnet.--> Existen varias opciones para crear un servidor flexible mediante la CLI, tal y como se muestra en los ejemplos siguientes.

>[!Important]
> El uso de este comando delega la subred a **Microsoft.DBforMySQL/flexibleServers**. Esta delegación significa que solo los servidores flexibles de Azure Database for MySQL pueden usar esa subred. No puede haber otros tipos de recursos de Azure en la subred delegada.
>

Consulte en la [documentación de referencia](/cli/azure/mysql/flexible-server) de la CLI de Azure la lista completa de parámetros configurables de la CLI. Por ejemplo, en los siguientes comandos puede especificar opcionalmente el grupo de recursos.

- Creación de un servidor flexible mediante la red virtual predeterminada y la subred con prefijo de dirección predeterminado
    ```azurecli-interactive
    az mysql flexible-server create
    ```
- Cree un servidor flexible mediante la red virtual existente y la subred. Si la red virtual y la subred proporcionadas no existen, se creará la red virtual y la subred con prefijo de dirección predeterminado.
    ```azurecli-interactive
    az mysql flexible-server create --vnet myVnet --subnet mySubnet
    ```

- Cree un servidor flexible mediante la red virtual existente, la subred y el id. de subred. La subred proporcionada no debe tener ningún otro recurso implementado y se delega a **Microsoft.DBforMySQL/flexibleServers**, si aún no se ha delegado.
    ```azurecli-interactive
    az mysql flexible-server create --subnet /subscriptions/{SubID}/resourceGroups/{ResourceGroup}/providers/Microsoft.Network/virtualNetworks/{VNetName}/subnets/{SubnetName}
    ```
    > [!Note]
    > La red virtual y la subred deben estar en la misma región y suscripción que el servidor flexible.
<
- Cree un servidor flexible mediante una nueva red virtual y subred con prefijo de dirección no predeterminado.
    ```azurecli-interactive
    az mysql flexible-server create --vnet myVnet --address-prefixes 10.0.0.0/24 --subnet mySubnet --subnet-prefixes 10.0.0.0/24
    ```
Consulte en la [documentación de referencia](/cli/azure/mysql/flexible-server) de la CLI de Azure la lista completa de parámetros configurables de la CLI.


## <a name="next-steps"></a>Pasos siguientes
- Obtenga más información sobre las [redes de Azure Database for MySQL: servidor flexible](./concepts-networking.md).
- [Cree y administre redes virtuales de Azure Database for MySQL: servidor flexible mediante Azure Portal](./how-to-manage-virtual-network-portal.md).
- Obtenga más información sobre las [redes virtuales de Azure Database for MySQL: servidor flexible](./concepts-networking.md#private-access-vnet-integration).