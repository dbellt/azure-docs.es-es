---
title: Administración de puntos de conexión de red virtual de Azure Database for MySQL mediante la CLI de Azure
description: En este artículo se describe cómo crear y administrar reglas y puntos de conexión de servicio de red virtual de Azure Database for MySQL mediante la línea de comandos de la CLI de Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 0bc686efbd07cf39d7932b175b6f9800b1ff185f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107774672"
---
# <a name="create-and-manage-azure-database-for-mysql-vnet-service-endpoints-using-azure-cli"></a>Creación y administración de puntos de conexión de servicio de red virtual de Azure Database for MySQL mediante la CLI de Azure
Las reglas y los puntos de conexión de servicios de red virtual (VNet) amplían el espacio de direcciones privadas de una red virtual al servidor de Azure Database for MySQL. Con los comandos de la interfaz de la línea de comandos (CLI) de Azure adecuados, puede crear, actualizar, eliminar, enumerar y mostrar reglas y puntos de conexión de servicio de red virtual para administrar el servidor. Para obtener información general sobre los puntos de conexión de servicio de red virtual de Azure Database for MySQL, incluidas las limitaciones, consulte [Azure Database for MySQL Server VNet service endpoints](concepts-data-access-and-security-vnet.md) (Puntos de conexión de servicio de red virtual del servidor de Azure Database for MySQL). Los puntos de conexión del servicio de red virtual están disponibles en todas las regiones admitidas para Azure Database for MySQL.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Necesita un [servidor y una base de datos de Azure Database for MySQL](quickstart-create-mysql-server-database-using-azure-cli.md).
 
- En este artículo se necesita la versión 2.0 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

> [!NOTE]
> La compatibilidad con puntos de conexión de servicio de red virtual solo existe para servidores de uso general y optimizados para memoria.
> En el caso de emparejamiento de VNET, si el tráfico fluye a través de una instancia de VNet Gateway común con puntos de conexión de servicio y se supone que lo hace al elemento del mismo nivel, cree una regla de ACL o red virtual para permitir que Microsoft Azure Virtual Machines en la red virtual de puerta de enlace acceda al servidor de Azure Database for MySQL.

## <a name="configure-vnet-service-endpoints-for-azure-database-for-mysql"></a>Configuración de puntos de conexión de servicio de red virtual para Azure Database for MySQL
Los comandos [az network vnet](/cli/azure/network/vnet) se usan para configurar redes virtuales.

Si tiene varias suscripciones, elija la suscripción adecuada en la que se debe facturar el recurso. Seleccione el identificador de suscripción específico en su cuenta mediante el comando [az account set](/cli/azure/account#az_account_set). Sustituya la propiedad **id** de la salida **az login** para su suscripción en el marcador de posición de identificador de suscripción.

- La cuenta debe tener todos los permisos necesarios para crear una red virtual y un punto de conexión de servicio.

Los puntos de conexión de servicio pueden configurarse en redes virtuales de forma independiente por un usuario con acceso de escritura a la red virtual.

Para proteger los recursos de servicio de Azure en una red virtual, el usuario debe tener permiso en "Microsoft.Network/virtualNetworks/subnets/joinViaServiceEndpoint/" para las subredes que se agregan. De forma predeterminada, este permiso se incluye en los roles de administrador de servicios integrado y puede modificarse mediante la creación de roles personalizados.

Obtenga más información sobre los [roles integrados](../role-based-access-control/built-in-roles.md) y la asignación de permisos específicos a [roles personalizados](../role-based-access-control/custom-roles.md).

Las redes virtuales y los recursos de servicio de Azure pueden encontrarse en la misma o en diferentes suscripciones. Si los recursos de servicio de Azure y de red virtual se encuentran en distintas suscripciones, los recursos deben estar en el mismo inquilino de Active Directory (AD). Asegúrese de que ambas suscripciones tengan registrado el proveedor de recursos **Microsoft.Sql**. Para más información, consulte [resource-manager-registration][resource-manager-portal].

> [!IMPORTANT]
> Se recomienda encarecidamente que lea este artículo sobre las configuraciones y las consideraciones de los puntos de conexión de servicio antes de ejecutar el script de ejemplo siguiente, o configurar los puntos de conexión de servicio. **Punto de conexión de servicio de red virtual:** un [punto de conexión de servicio de red virtual](../virtual-network/virtual-network-service-endpoints-overview.md) es una subred cuyos valores de propiedad incluyen uno o más nombres formales de tipo de servicio de Azure. Los puntos de conexión de servicio de red virtual usan el nombre de tipo de servicio **Microsoft.Sql**, que hace referencia al servicio de Azure denominado SQL Database. Esta etiqueta de servicio también se aplica a los servicios Azure SQL Database, Azure Database for PostgreSQL y MySQL. Es importante tener en cuenta que, al aplicar la etiqueta de servicio de **Microsoft.Sql** a un punto de conexión de servicio de red virtual, se configura el tráfico de punto de conexión de servicio de todos los servicios de Azure Database, incluidos los servidores de Azure SQL Database, Azure Database for PostgreSQL y Azure Database for MySQL de la subred. 
> 

### <a name="sample-script-to-create-an-azure-database-for-mysql-database-create-a-vnet-vnet-service-endpoint-and-secure-the-server-to-the-subnet-with-a-vnet-rule"></a>Script de ejemplo para crear una base de datos de Azure Database for MySQL, crear una red virtual, un punto de conexión de servicio de red virtual y proteger el servidor a la subred con una regla de red virtual
En este script de ejemplo, cambie las líneas resaltadas para personalizar el nombre de usuario de administrador y la contraseña. Reemplace el identificador de suscripción usado en el comando `az account set --subscription` por su propio identificador de suscripción.
[!code-azurecli-interactive[main](../../cli_scripts/mysql/create-mysql-server-vnet/create-mysql-server.sh?highlight=5,20 "Create an Azure Database for MySQL, VNet, VNet service endpoint, and VNet rule.")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación
Después de ejecutar el script de ejemplo, se puede usar el comando siguiente para quitar el grupo de recursos y todos los recursos asociados.
[!code-azurecli-interactive[main](../../cli_scripts/mysql/create-mysql-server-vnet/delete-mysql.sh "Delete the resource group.")]

<!-- Link references, to text, Within this same GitHub repo. --> 
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
