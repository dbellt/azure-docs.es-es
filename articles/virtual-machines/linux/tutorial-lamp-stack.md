---
title: 'Tutorial: Implementación de LAMP y WordPress en una máquina virtual'
description: En este tutorial, aprenderá a instalar la pila LAMP y WordPress en una máquina virtual Linux en Azure.
author: cynthn
ms.collection: linux
ms.service: virtual-machines
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 04/20/2021
ms.author: cynthn
ms.openlocfilehash: 5365bad5fdea2a8213defc103f0cdd966ebe50a5
ms.sourcegitcommit: 260a2541e5e0e7327a445e1ee1be3ad20122b37e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107816357"
---
# <a name="tutorial-install-a-lamp-stack-on-an-azure-linux-vm"></a>Tutorial: Instalación de una pila LAMP en una máquina virtual Linux en Azure

En este artículo se ofrecen instrucciones paso a paso para implementar un servidor web Apache, MySQL y PHP (la pila LAMP) en una máquina virtual de Ubuntu en Azure. Para ver el servidor LAMP en acción, si lo desea, puede instalar y configurar un sitio de WordPress. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Creación de una máquina virtual de Ubuntu 
> * Apertura del puerto 80 para el tráfico web
> * Instalación de Apache, MySQL y PHP
> * Comprobación de la instalación y configuración
> * Instalación de WordPress 

Esta configuración es para pruebas rápidas o prueba de concepto. Para más información sobre la pila LAMP, incluyendo recomendaciones para un entorno de producción, consulte la [documentación de Ubuntu](https://help.ubuntu.com/community/ApacheMySQLPHP).

En este tutorial se usa la CLI dentro de [Azure Cloud Shell](../../cloud-shell/overview.md), que se actualiza constantemente a la versión más reciente. Para abrir Cloud Shell, seleccione **Pruébelo** en la esquina superior de cualquier bloque de código.

Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la CLI de Azure de la versión 2.0.30, u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Para crear un grupo de recursos, use el comando [az group create](/cli/azure/group). Un grupo de recursos de Azure es un contenedor lógico en el que se implementan y se administran los recursos de Azure. 

En el ejemplo siguiente, se crea un grupo de recursos denominado *myResourceGroup* en la ubicación *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>Creación de una máquina virtual

Cree la máquina virtual con el comando [az vm create](/cli/azure/vm). 

En el ejemplo siguiente, se crea una máquina virtual denominada *myVM* y las claves SSH si aún no existen en una ubicación de claves predeterminada. Para utilizar un conjunto específico de claves, utilice la opción `--ssh-key-value`. El comando también establece *azureuser* como nombre de usuario de administrador. Use este nombre más adelante para conectarse a la VM. 

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Cuando se ha creado la máquina virtual, la CLI de Azure muestra información similar al ejemplo siguiente. Anote el valor de `publicIpAddress`. Esta dirección se utiliza para acceder a la VM en los pasos posteriores.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```



## <a name="open-port-80-for-web-traffic"></a>Apertura del puerto 80 para el tráfico web 

De forma predeterminada, solo se permiten conexiones mediante SSH con las máquinas virtuales Linux implementadas en Azure. Dado que esta máquina virtual va a ser un servidor web, debe abrir el puerto 80 desde Internet. Use el comando [az vm open-port](/cli/azure/vm) para abrir el puerto deseado.  
 
```azurecli-interactive
az vm open-port --port 80 --resource-group myResourceGroup --name myVM
```

Para obtener más información sobre cómo abrir puertos en la máquina virtual, consulte [Apertura de puertos](nsg-quickstart.md).

## <a name="ssh-into-your-vm"></a>Conexión SSH con la máquina virtual

Si no conoce la dirección IP pública de la máquina virtual, ejecute el comando [az network public-ip list](/cli/azure/network/public-ip). Necesitará esta dirección IP para varios pasos posteriores.

```azurecli-interactive
az network public-ip list --resource-group myResourceGroup --query [].ipAddress
```

Ejecute el comando siguiente para crear una sesión SSH con la máquina virtual. Sustituya la dirección IP pública correcta de la máquina virtual. En este ejemplo, la dirección IP es *40.68.254.142*. *azureuser* se establece el nombre de usuario de administrador establecido al crear la VM.

```bash
ssh azureuser@40.68.254.142
```


## <a name="install-apache-mysql-and-php"></a>Instalación de Apache, MySQL y PHP

Ejecute el siguiente comando para actualizar los orígenes de paquetes de Ubuntu e instalar Apache, MySQL y PHP. Tenga en cuenta el símbolo de intercalación (^) al final del comando, que forma parte del nombre del paquete `lamp-server^`. 


```bash
sudo apt update && sudo apt install lamp-server^
```

Se le pide que instale los paquetes y otras dependencias. Este proceso instala las extensiones PHP mínimas necesarias para utilizar PHP con MySQL.  

## <a name="verify-apache"></a>Comprobación de Apache

Compruebe la versión de Apache con el comando siguiente:
```bash
apache2 -v
```

Con Apache instalado y el puerto 80 abierto para la máquina virtual, ahora se puede acceder al servidor web desde Internet. Para ver la página predeterminada de Apache2 Ubuntu, abra un explorador web y escriba la dirección IP pública de la máquina virtual. Use la dirección IP pública utilizada para SSH en la máquina virtual:

![Página predeterminada de Apache][3]


## <a name="verify-and-secure-mysql"></a>Comprobación y protección de MySQL

Compruebe la versión de MySQL con el siguiente comando (tenga en cuenta el parámetro `V` en mayúsculas):

```bash
mysql -V
```

Para ayudar a proteger la instalación de MySQL, incluido el establecimiento de una contraseña raíz, ejecute el script `mysql_secure_installation`. 

```bash
sudo mysql_secure_installation
```

Si lo desea, puede configurar el complemento para validar contraseña (se recomienda). A continuación, establezca una contraseña para el usuario raíz de MySQL y configure las opciones de seguridad restantes para su entorno. Se recomienda que responda "Y" (Yes o Sí) para todas las preguntas.

Si desea crear probar características de MySQL (crear una base de datos MySQL, agregar usuarios o cambiar la configuración), inicie sesión en MySQL. Este paso no es necesario para completar este tutorial.

```bash
sudo mysql -u root -p
```

Cuando haya terminado, escriba `\q` para salir del símbolo del sistema de mysql.

## <a name="verify-php"></a>Comprobación de PHP

Compruebe la versión de PHP con el comando siguiente:

```bash
php -v
```

Si desea realizar más pruebas, cree una página PHP de información rápida para verla en un explorador. El siguiente comando crea la página de información de PHP:

```bash
sudo sh -c 'echo "<?php phpinfo(); ?>" > /var/www/html/info.php'
```

Ahora puede comprobar la página de información de PHP que creó. Abra un explorador y vaya a `http://yourPublicIPAddress/info.php`. Sustituya la dirección por la dirección IP pública de la máquina virtual. Debe tener un aspecto similar a esta imagen.

![Página de información de PHP][2]

[!INCLUDE [virtual-machines-linux-tutorial-wordpress.md](../../../includes/virtual-machines-linux-tutorial-wordpress.md)]

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, implementó un servidor LAMP en Azure. Ha aprendido a:

> [!div class="checklist"]
> * Creación de una máquina virtual de Ubuntu
> * Apertura del puerto 80 para el tráfico web
> * Instalación de Apache, MySQL y PHP
> * Comprobación de la instalación y configuración
> * Instalación de WordPress en el servidor LAMP

Pase al siguiente tutorial para aprender a proteger servidores web con certificados TLS/SSL.

> [!div class="nextstepaction"]
> [Protección de servidores web con TLS](tutorial-secure-web-server.md)

[2]: ./media/tutorial-lamp-stack/phpsuccesspage.png
[3]: ./media/tutorial-lamp-stack/apachesuccesspage.png
