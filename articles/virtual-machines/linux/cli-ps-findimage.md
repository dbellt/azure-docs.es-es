---
title: Búsqueda y uso de la información del plan de compra de Marketplace mediante la CLI
description: Aprenda a usar el CLI de Azure para buscar URN de imagen y parámetros de plan de compra, como el publicador, la oferta, la SKU y la versión, para las imágenes de máquina virtual de Marketplace.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.date: 03/22/2021
ms.author: cynthn
ms.collection: linux
ms.custom: contperf-fy21q3-portal, devx-track-azurecli
ms.openlocfilehash: db21cf9e4dc28a83b98f58e73db9a5d76f80aef2
ms.sourcegitcommit: 5ce88326f2b02fda54dad05df94cf0b440da284b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107886683"
---
# <a name="find-azure-marketplace-image-information-using-the-azure-cli"></a>Busque información de la imagen de Azure Marketplace mediante el CLI de Azure

Este tema describe cómo usar la CLI de Azure para buscar imágenes de máquina virtual en Azure Marketplace. Use esta información para especificar una imagen de Marketplace cuando se crea una máquina virtual mediante programación con la CLI, plantillas de Resource Manager u otras herramientas.

Puede examinar también las imágenes y ofertas disponibles mediante[Azure Marketplace](https://azuremarketplace.microsoft.com/)o [Azure PowerShell](../windows/cli-ps-findimage.md). 

## <a name="terminology"></a>Terminología

Una imagen de Marketplace de Azure tiene los atributos siguientes:

* **Publicador**: organización que ha creado la imagen. Ejemplos: Canonical, MicrosoftWindowsServer
* **Oferta**: nombre de un grupo de imágenes relacionadas creado por un publicador. Ejemplos: Ubuntu Server, WindowsServer
* **SKU**: instancia de una oferta, por ejemplo, una versión principal de una distribución. Ejemplos: 18.04-LTS, 2019-Datacenter
* **Versión**: número de versión de una SKU de imagen. 

Estos valores se pueden pasar individualmente o como un *URN* de imagen, combinando los valores separados por dos puntos (:). Por ejemplo: *Publicador*:*Oferta*:*SKU*:*Versión*. Puede reemplazar el número de versión del URN por `latest` para usar la versión más reciente de la imagen. 

Si el publicador de imágenes proporciona una licencia adicional y términos de compra, debe aceptarlos para poder usar la imagen.  Para obtener más información, vea [Comprobar la información del plan de compra](#check-the-purchase-plan-information).



## <a name="list-popular-images"></a>Enumeración de imágenes populares

Ejecute el comando [az vm image list](/cli/azure/vm/image) sin la opción `--all` para ver una lista de imágenes de VM populares en Azure Marketplace. Por ejemplo, ejecute el siguiente comando para mostrar una lista de almacenamiento en caché de imágenes populares en el formato de tabla:

```azurecli
az vm image list --output table
```

La salida incluye la imagen URN. También puede usar el *UrnAlias*, que es una versión abreviada creada para imágenes populares como *UbuntuLTS*.

```output
Offer          Publisher               Sku                 Urn                                                             UrnAlias             Version
-------------  ----------------------  ------------------  --------------------------------------------------------------  -------------------  ---------
CentOS         OpenLogic               7.5                 OpenLogic:CentOS:7.5:latest                                     CentOS               latest
CoreOS         CoreOS                  Stable              CoreOS:CoreOS:Stable:latest                                     CoreOS               latest
debian-10      Debian                  10                  Debian:debian-10:10:latest                                      Debian               latest
openSUSE-Leap  SUSE                    42.3                SUSE:openSUSE-Leap:42.3:latest                                  openSUSE-Leap        latest
RHEL           RedHat                  7-LVM               RedHat:RHEL:7-LVM:latest                                        RHEL                 latest
SLES           SUSE                    15                  SUSE:SLES:15:latest                                             SLES                 latest
UbuntuServer   Canonical               18.04-LTS           Canonical:UbuntuServer:18.04-LTS:latest                         UbuntuLTS            latest
WindowsServer  MicrosoftWindowsServer  2019-Datacenter     MicrosoftWindowsServer:WindowsServer:2019-Datacenter:latest     Win2019Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2016-Datacenter     MicrosoftWindowsServer:WindowsServer:2016-Datacenter:latest     Win2016Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2012-R2-Datacenter  MicrosoftWindowsServer:WindowsServer:2012-R2-Datacenter:latest  Win2012R2Datacenter  latest
WindowsServer  MicrosoftWindowsServer  2012-Datacenter     MicrosoftWindowsServer:WindowsServer:2012-Datacenter:latest     Win2012Datacenter    latest
WindowsServer  MicrosoftWindowsServer  2008-R2-SP1         MicrosoftWindowsServer:WindowsServer:2008-R2-SP1:latest         Win2008R2SP1         latest
```

## <a name="find-specific-images"></a>Búsqueda de imágenes específicas

Para buscar una imagen de máquina virtual determinada en Marketplace, use el comando `az vm image list` con la opción `--all`. Esta versión del comando tarda algún tiempo en completarse y puede devolver una salida larga, por lo que normalmente la lista se filtra por `--publisher` u otro parámetro. 

Por ejemplo, el siguiente comando muestra todas las ofertas de Debian (recuerde que sin el conmutador `--all`, solo busca en la caché local de las imágenes comunes):

```azurecli
az vm image list --offer Debian --all --output table 
```

Salida parcial: 

```output
Offer                                    Publisher                         Sku                                      Urn                                                                                                   Version
---------------------------------------  --------------------------------  ---------------------------------------  ----------------------------------------------------------------------------------------------------  --------------
apache-solr-on-debian                    apps-4-rent                       apache-solr-on-debian                    apps-4-rent:apache-solr-on-debian:apache-solr-on-debian:1.0.0                                         1.0.0
atomized-h-debian10-v1                   atomizedinc1587939464368          hdebian10plan                            atomizedinc1587939464368:atomized-h-debian10-v1:hdebian10plan:1.0.0                                   1.0.0
atomized-h-debian9-v1                    atomizedinc1587939464368          hdebian9plan                             atomizedinc1587939464368:atomized-h-debian9-v1:hdebian9plan:1.0.0                                     1.0.0
atomized-r-debian10-v1                   atomizedinc1587939464368          rdebian10plan                            atomizedinc1587939464368:atomized-r-debian10-v1:rdebian10plan:1.0.0                                   1.0.0
atomized-r-debian9-v1                    atomizedinc1587939464368          rdebian9plan                             atomizedinc1587939464368:atomized-r-debian9-v1:rdebian9plan:1.0.0                                     1.0.0
cis-debian-linux-10-l1                   center-for-internet-security-inc  cis-debian10-l1                          center-for-internet-security-inc:cis-debian-linux-10-l1:cis-debian10-l1:1.0.7                         1.0.7
cis-debian-linux-10-l1                   center-for-internet-security-inc  cis-debian10-l1                          center-for-internet-security-inc:cis-debian-linux-10-l1:cis-debian10-l1:1.0.8                         1.0.8
cis-debian-linux-10-l1                   center-for-internet-security-inc  cis-debian10-l1                          center-for-internet-security-inc:cis-debian-linux-10-l1:cis-debian10-l1:1.0.9                         1.0.9
cis-debian-linux-9-l1                    center-for-internet-security-inc  cis-debian9-l1                           center-for-internet-security-inc:cis-debian-linux-9-l1:cis-debian9-l1:1.0.18                          1.0.18
cis-debian-linux-9-l1                    center-for-internet-security-inc  cis-debian9-l1                           center-for-internet-security-inc:cis-debian-linux-9-l1:cis-debian9-l1:1.0.19                          1.0.19
cis-debian-linux-9-l1                    center-for-internet-security-inc  cis-debian9-l1                           center-for-internet-security-inc:cis-debian-linux-9-l1:cis-debian9-l1:1.0.20                          1.0.20
apache-web-server-with-debian-10         cognosys                          apache-web-server-with-debian-10         cognosys:apache-web-server-with-debian-10:apache-web-server-with-debian-10:1.2019.1008                1.2019.1008
docker-ce-with-debian-10                 cognosys                          docker-ce-with-debian-10                 cognosys:docker-ce-with-debian-10:docker-ce-with-debian-10:1.2019.0710                                1.2019.0710
Debian                                   credativ                          8                                        credativ:Debian:8:8.0.201602010                                                                       8.0.201602010
Debian                                   credativ                          8                                        credativ:Debian:8:8.0.201603020                                                                       8.0.201603020
Debian                                   credativ                          8                                        credativ:Debian:8:8.0.201604050                                                                       8.0.201604050
...
```


## <a name="look-at-all-available-images"></a>Examinar todas las imágenes disponibles
 
Otra forma de buscar una imagen en una ubicación es ejecutar los comandos [az vm image list-publishers](/cli/azure/vm/image), [az vm image list-offers](/cli/azure/vm/image) y [az vm image list-skus](/cli/azure/vm/image) en orden. Con estos comandos, determine estos valores:

1. Enumere los publicadores de imágenes de una ubicación. En este ejemplo, estamos examinando la región *Oeste de EE. UU.*
    
    ```azurecli
    az vm image list-publishers --location westus --output table
    ```

1. Para un publicador determinado, enumeración de sus ofertas. En este ejemplo, se agrega *Canonical* como el publicador.
    
    ```azurecli
    az vm image list-offers --location westus --publisher Canonical --output table
    ```

1. Para una oferta determinada, enumeración de sus SKU. En este ejemplo, agregamos *UbuntuServer* como oferta.
    ```azurecli
    az vm image list-skus --location westus --publisher Canonical --offer UbuntuServer --output table
    ```

1. Para un publicador, una oferta y una SKU determinados, muestre todas las versiones de la imagen. En este ejemplo, se agrega *18.04-LTS* como SKU.

    ```azurecli
    az vm image list \
        --location westus \
        --publisher Canonical \  
        --offer UbuntuServer \    
        --sku 18.04-LTS \
        --all --output table
    ```

Pase este valor de la columna URN con el parámetro `--image` cuando cree una máquina virtual con el [comando az vm create](/cli/azure/vm). También puede reemplazar el número de versión del URN por "más actualizado", para usar simplemente la versión más reciente de la imagen. 

Si implementa una máquina virtual con una plantilla de Resource Manager, los parámetros de imagen se establecen individualmente en las propiedades `imageReference`. Consulte la [referencia de plantilla](/azure/templates/microsoft.compute/virtualmachines).


## <a name="check-the-purchase-plan-information"></a>Comprobar la información del plan de compra

Algunas imágenes de máquina virtual en Azure Marketplace tienen términos adicionales de licencia y compra que debe aceptar antes de poder implementarlas mediante programación.  

Para implementar una máquina virtual desde una imagen de este tipo, tendrá que aceptar los términos de la imagen la primera vez que la use, una vez por suscripción. También deberá especificar parámetros de *plan de compra* para implementar una máquina virtual a partir de esa imagen

Para ver la información del plan de compra de una imagen, ejecute el comando [az vm image show](/cli/azure/image) con el URN de la imagen. Si la propiedad `plan` de la salida no es `null`, la imagen tienen términos que debe aceptar antes de la implementación mediante programación.

Por ejemplo, la imagen de Canonical Ubuntu Server 18.04 LTS no tiene términos adicionales, porque la información de `plan` es `null`:

```azurecli
az vm image show --location westus --urn Canonical:UbuntuServer:18.04-LTS:latest
```

Salida:

```output
{
  "dataDiskImages": [],
  "id": "/Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/Canonical/ArtifactTypes/VMImage/Offers/UbuntuServer/Skus/18.04-LTS/Versions/18.04.201901220",
  "location": "westus",
  "name": "18.04.201901220",
  "osDiskImage": {
    "operatingSystem": "Linux"
  },
  "plan": null,
  "tags": null
}
```

Ejecutar un comando similar para la imagen RabbitMQ Certified by Bitnami muestra las propiedades de `plan` siguientes: `name`, `product` y `publisher`. (Algunas imágenes también tienen una propiedad `promotion code`). 

```azurecli
az vm image show --location westus --urn bitnami:rabbitmq:rabbitmq:latest
```
Salida:

```output
{
  "dataDiskImages": [],
  "id": "/Subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/Providers/Microsoft.Compute/Locations/westus/Publishers/bitnami/ArtifactTypes/VMImage/Offers/rabbitmq/Skus/rabbitmq/Versions/3.7.1901151016",
  "location": "westus",
  "name": "3.7.1901151016",
  "osDiskImage": {
    "operatingSystem": "Linux"
  },
  "plan": {
    "name": "rabbitmq",
    "product": "rabbitmq",
    "publisher": "bitnami"
  },
  "tags": null
}
```

Para implementar esta imagen, debe aceptar los términos y proporcionar los parámetros del plan de compra al implementar una máquina virtual con esa imagen.

## <a name="accept-the-terms"></a>Aceptación de los términos

Para ver y aceptar los términos de licencia, use el comando [az vm image accept-terms](/cli/azure/vm/image/terms). Cuando acepta los términos, habilita la implementación mediante programación en la suscripción. Solo debe aceptar los términos una vez por suscripción para la imagen. Por ejemplo:

```azurecli
az vm image terms show --urn bitnami:rabbitmq:rabbitmq:latest
``` 

La salida incluye un `licenseTextLink` a los términos de licencia e indica que el valor de `accepted` es `true`:

```output
{
  "accepted": true,
  "additionalProperties": {},
  "id": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/providers/Microsoft.MarketplaceOrdering/offertypes/bitnami/offers/rabbitmq/plans/rabbitmq",
  "licenseTextLink": "https://storelegalterms.blob.core.windows.net/legalterms/3E5ED_legalterms_BITNAMI%253a24RABBITMQ%253a24RABBITMQ%253a24IGRT7HHPIFOBV3IQYJHEN2O2FGUVXXZ3WUYIMEIVF3KCUNJ7GTVXNNM23I567GBMNDWRFOY4WXJPN5PUYXNKB2QLAKCHP4IE5GO3B2I.txt",
  "name": "rabbitmq",
  "plan": "rabbitmq",
  "privacyPolicyLink": "https://bitnami.com/privacy",
  "product": "rabbitmq",
  "publisher": "bitnami",
  "retrieveDatetime": "2019-01-25T20:37:49.937096Z",
  "signature": "XXXXXXLAZIK7ZL2YRV5JYQXONPV76NQJW3FKMKDZYCRGXZYVDGX6BVY45JO3BXVMNA2COBOEYG2NO76ONORU7ITTRHGZDYNJNXXXXXX",
  "type": "Microsoft.MarketplaceOrdering/offertypes"
}
```

Para aceptar los términos, escriba:

```azurecli
az vm image terms accept --urn bitnami:rabbitmq:rabbitmq:latest
``` 

## <a name="deploy-a-new-vm-using-the-image-parameters"></a>Implementar una máquina virtual nueva mediante los parámetros de imagen

Con información sobre la imagen, puede implementarla mediante el comando `az vm create`. 

Para implementar una imagen que no tiene información de plan, como la última imagen de Ubuntu Server 18.04 de Canonical, pase el URN para `--image`:

```azurecli-interactive
az group create --name myURNVM --location westus
az vm create \
   --resource-group myURNVM \
   --name myVM \
   --admin-username azureuser \
   --generate-ssh-keys \
   --image Canonical:UbuntuServer:18.04-LTS:latest 
```


En el caso de una imagen con parámetros de plan de compra, como la imagen RabbitMQ Certified by Bitnami, se pasa el URN para `--image` y también se proporcionan los parámetros del plan de compra:

```azurecli
az group create --name myPurchasePlanRG --location westus

az vm create \
   --resource-group myPurchasePlanRG \
   --name myVM \
   --admin-username azureuser \
   --generate-ssh-keys \
   --image bitnami:rabbitmq:rabbitmq:latest \
   --plan-name rabbitmq \
   --plan-product rabbitmq \
   --plan-publisher bitnami
```

Si aparece un mensaje sobre la aceptación de los términos de la imagen, consulte la sección [Aceptación de los términos](#accept-the-terms). Asegúrese de que la salida de `az vm image accept-terms` devuelve el valor `"accepted": true,` que indica que ha aceptado los términos de la imagen.


## <a name="using-an-existing-vhd-with-purchase-plan-information"></a>Uso de un disco duro virtual existente con la información del plan de compra

Si tiene un disco duro virtual existente de una máquina virtual que se creó mediante una imagen de Azure Marketplace de pago, es posible que tenga que proporcionar la información del plan de compra al crear una nueva máquina virtual a partir de ese VHD. 

Si aún tiene la máquina virtual original, u otra máquina virtual creada que use la misma imagen de Marketplace, puede obtener de ella información sobre el nombre del plan, el editor e información del producto mediante [az vm get-instance-view](/cli/azure/vm#az_vm_get_instance_view). En este ejemplo se obtiene una máquina virtual llamada *myVM* en el grupo de recursos *myResourceGroup* y, después, se muestra la información del plan de compra.

```azurepowershell-interactive
az vm get-instance-view -g myResourceGroup -n myVM --query plan
```

Si no recibió la información del plan antes de que se eliminara la máquina virtual original, puede presentar una [solicitud de soporte técnico](https://ms.portal.azure.com/#create/Microsoft.Support). Deberá proporcionar el nombre de la máquina virtual, el identificador de la suscripción y la marca de tiempo de la operación de eliminación.

Una vez que tenga la información del plan, puede crear la nueva máquina virtual y utilizar el parámetro `--attach-os-disk` para especificar el disco duro virtual.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myNewVM \
  --nics myNic \
  --size Standard_DS1_v2 --os-type Linux \
  --attach-os-disk myVHD \
  --plan-name planName \
  --plan-publisher planPublisher \
  --plan-product planProduct 
```


## <a name="next-steps"></a>Pasos siguientes
Para crear una máquina virtual rápidamente con la información de la imagen, consulte [Creación y administración de máquinas virtuales Linux con la CLI de Azure](tutorial-manage-vm.md).
