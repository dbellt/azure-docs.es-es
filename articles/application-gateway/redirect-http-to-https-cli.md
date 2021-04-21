---
title: Redireccionamiento de HTTP a HTTPS mediante la CLI
titleSuffix: Azure Application Gateway
description: Aprenda a crear un redireccionamiento de HTTP a HTTPS y a agregar un certificado para la terminación TLS mediante la CLI de Azure.
services: application-gateway
author: vhorne
ms.service: application-gateway
ms.topic: how-to
ms.date: 09/24/2020
ms.author: victorh
ms.openlocfilehash: e66eca305433a89496f72aac667512efd418a369
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107784774"
---
# <a name="create-an-application-gateway-with-http-to-https-redirection-using-the-azure-cli"></a>Creación de una puerta de enlace de aplicaciones con redirección de HTTP a HTTPS mediante la CLI de Azure

Puede usar la CLI de Azure para crear una instancia de [Application Gateway](overview.md) con un certificado para la terminación TLS/SSL. Para redirigir el tráfico HTTP al puerto HTTPS en la puerta de enlace de aplicaciones se usa una regla de enrutamiento. En este ejemplo, también crea un [conjunto de escalado de máquinas virtuales](../virtual-machine-scale-sets/overview.md) para el grupo de back-end de la puerta de enlace de aplicaciones que contiene dos instancias de máquina virtual.

En este artículo aprenderá a:

* Creación de un certificado autofirmado
* Configurar una red
* Crear una puerta de enlace de aplicaciones con el certificado
* Adición de un agente de escucha y una regla de redireccionamiento
* Crear un conjunto de escalado de máquinas virtuales con el grupo de servidores back-end predeterminado

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Este tutorial requiere la versión 2.0.4 o posterior de la CLI de Azure. Si usa Azure Cloud Shell, ya está instalada la versión más reciente.

## <a name="create-a-self-signed-certificate"></a>Creación de un certificado autofirmado

Para su uso en producción, debe importar un certificado válido firmado por un proveedor de confianza. En este tutorial, creará un certificado autofirmado y un archivo pfx con el comando openssl.

```console
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -keyout privateKey.key -out appgwcert.crt
```

Escriba valores que tengan sentido para su certificado. Puede aceptar los valores predeterminados.

```console
openssl pkcs12 -export -out appgwcert.pfx -inkey privateKey.key -in appgwcert.crt
```

Escriba la contraseña del certificado. En este ejemplo, se usa *Azure123456!* .

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. Para crear un grupo de recursos, use [az group create](/cli/azure/group).

En el ejemplo siguiente, se crea un grupo de recursos llamado *myResourceGroupAG* en la ubicación *eastus*.

```azurecli-interactive 
az group create --name myResourceGroupAG --location eastus
```

## <a name="create-network-resources"></a>Crear recursos de red

Cree la red virtual llamada *myVNet* y la subred llamada *myAGSubnet* mediante [az network vnet create](/cli/azure/network/vnet). Luego, puede agregar la subred llamada *myBackendSubnet* que necesitan los servidores back-end mediante [az network vnet subnet create](/cli/azure/network/vnet/subnet). Cree la dirección IP pública llamada *myAGPublicIPAddress* mediante [az network public-ip create](/cli/azure/network/public-ip).

```azurecli-interactive
az network vnet create \
  --name myVNet \
  --resource-group myResourceGroupAG \
  --location eastus \
  --address-prefix 10.0.0.0/16 \
  --subnet-name myAGSubnet \
  --subnet-prefix 10.0.1.0/24
az network vnet subnet create \
  --name myBackendSubnet \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --address-prefix 10.0.2.0/24
az network public-ip create \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress
```

## <a name="create-the-application-gateway"></a>Creación de la puerta de enlace de aplicaciones

Puede usar [az network application-gateway create](/cli/azure/network/application-gateway#az_network_application_gateway_create) para crear la puerta de enlace de aplicaciones llamada *myAppGateway*. Cuando se crea una puerta de enlace de aplicaciones mediante la CLI de Azure, se especifica información de configuración, como capacidad, SKU y HTTP. 

La puerta de enlace de aplicaciones se asigna a los elementos *myAGSubnet* y *myAGPublicIPAddress* creados anteriormente. En este ejemplo, asociará el certificado que creó y su contraseña al crear la puerta de enlace de aplicaciones. 

```azurecli-interactive
az network application-gateway create \
  --name myAppGateway \
  --location eastus \
  --resource-group myResourceGroupAG \
  --vnet-name myVNet \
  --subnet myAGsubnet \
  --capacity 2 \
  --sku Standard_Medium \
  --http-settings-cookie-based-affinity Disabled \
  --frontend-port 443 \
  --http-settings-port 80 \
  --http-settings-protocol Http \
  --public-ip-address myAGPublicIPAddress \
  --cert-file appgwcert.pfx \
  --cert-password "Azure123456!"

```

 La puerta de enlace de aplicaciones puede tardar varios minutos en crearse. Después de crear la puerta de enlace de aplicaciones, puede ver estas nuevas características de ella:

- *appGatewayBackendPool*: una puerta de enlace de aplicaciones debe tener al menos un grupo de direcciones de servidores back-end.
- *appGatewayBackendHttpSettings*: especifica que se use el puerto 80 y un protocolo HTTP para la comunicación.
- *appGatewayHttpListener*: agente de escucha predeterminado asociado con *appGatewayBackendPool*.
- *appGatewayFrontendIP*: asigna *myAGPublicIPAddress* a *appGatewayHttpListener*.
- *rule1*: la regla de enrutamiento predeterminada asociada a *appGatewayHttpListener*.

## <a name="add-a-listener-and-redirection-rule"></a>Adición de un agente de escucha y una regla de redireccionamiento

### <a name="add-the-http-port"></a>Adición del puerto HTTP

Puede usar [az network application-gateway frontend-port create](/cli/azure/network/application-gateway/frontend-port#az_network-application_gateway_frontend_port_create) para agregar el puerto HTTP a la puerta de enlace de aplicaciones.

```azurecli-interactive
az network application-gateway frontend-port create \
  --port 80 \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --name httpPort
```

### <a name="add-the-http-listener"></a>Adición del agente de escucha HTTP

Puede usar [az network application-gateway http-listener create](/cli/azure/network/application-gateway/http-listener#az_network_application_gateway_http_listener_create) para agregar el agente de escucha denominado *myListener* a la puerta de enlace de aplicaciones.

```azurecli-interactive
az network application-gateway http-listener create \
  --name myListener \
  --frontend-ip appGatewayFrontendIP \
  --frontend-port httpPort \
  --resource-group myResourceGroupAG \
  --gateway-name myAppGateway
```

### <a name="add-the-redirection-configuration"></a>Adición de la configuración de redireccionamiento

Agregue la configuración de redirección de HTTP a HTTPS a la puerta de enlace de aplicaciones con [az network application-gateway redirect-config create](/cli/azure/network/application-gateway/redirect-config#az_network_application_gateway_redirect_config_create).

```azurecli-interactive
az network application-gateway redirect-config create \
  --name httpToHttps \
  --gateway-name myAppGateway \
  --resource-group myResourceGroupAG \
  --type Permanent \
  --target-listener appGatewayHttpListener \
  --include-path true \
  --include-query-string true
```

### <a name="add-the-routing-rule"></a>Adición de la regla de enrutamiento

Agregue la regla de enrutamiento denominada *rule2* con la configuración de redirección a la puerta de enlace de aplicaciones con [az network application-gateway rule create](/cli/azure/network/application-gateway/rule#az_network_application_gateway_rule_create).

```azurecli-interactive
az network application-gateway rule create \
  --gateway-name myAppGateway \
  --name rule2 \
  --resource-group myResourceGroupAG \
  --http-listener myListener \
  --rule-type Basic \
  --redirect-config httpToHttps
```

## <a name="create-a-virtual-machine-scale-set"></a>Crear un conjunto de escalado de máquinas virtuales

En este ejemplo, creará un conjunto de escalado de máquinas virtuales denominado *myvmss* que proporcione servidores para el grupo de back-end en la puerta de enlace de aplicaciones. Las máquinas virtuales del conjunto de escalado están asociadas a *myBackendSubnet* y *appGatewayBackendPool*. Para crear el conjunto de escalado, puede usar [az vmss create](/cli/azure/vmss#az_vmss_create).

```azurecli-interactive
az vmss create \
  --name myvmss \
  --resource-group myResourceGroupAG \
  --image UbuntuLTS \
  --admin-username azureuser \
  --admin-password Azure123456! \
  --instance-count 2 \
  --vnet-name myVNet \
  --subnet myBackendSubnet \
  --vm-sku Standard_DS2 \
  --upgrade-policy-mode Automatic \
  --app-gateway myAppGateway \
  --backend-pool-name appGatewayBackendPool
```

### <a name="install-nginx"></a>Instalación de NGINX

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroupAG \
  --vmss-name myvmss \
  --settings '{ "fileUris": ["https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/application-gateway/iis/install_nginx.sh"],
  "commandToExecute": "./install_nginx.sh" }'
```

## <a name="test-the-application-gateway"></a>Prueba de la puerta de enlace de aplicaciones

Para obtener la dirección IP pública de la puerta de enlace de aplicaciones, puede usar [az network public-ip show](/cli/azure/network/public-ip). Copie la dirección IP pública y péguela en la barra de direcciones del explorador.

```azurecli-interactive
az network public-ip show \
  --resource-group myResourceGroupAG \
  --name myAGPublicIPAddress \
  --query [ipAddress] \
  --output tsv
```

![Advertencia de seguridad](./media/redirect-http-to-https-cli/application-gateway-secure.png)

Para aceptar la advertencia de seguridad si usó un certificado autofirmado, seleccione **Detalles** y, a continuación, **Acceder a la página web**. A continuación, se muestra el sitio de NGINX protegido, como en el ejemplo siguiente:

![Prueba de la dirección URL base en la puerta de enlace de aplicaciones](./media/redirect-http-to-https-cli/application-gateway-nginxtest.png)

## <a name="next-steps"></a>Pasos siguientes

- [Creación de una puerta de enlace de aplicaciones con redireccionamiento interno mediante la CLI de Azure](redirect-internal-site-cli.md)