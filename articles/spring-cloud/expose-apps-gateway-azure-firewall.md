---
title: Exposición de las aplicaciones a Internet mediante Application Gateway y Azure Firewall
description: Cómo exponer las aplicaciones a Internet mediante Application Gateway y Azure Firewall
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 11/17/2020
ms.custom: devx-track-java
ms.openlocfilehash: 7def685cb9e17ff253ade10714ece2259b432db1
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108129016"
---
# <a name="expose-applications-to-the-internet-using-application-gateway-and-azure-firewall"></a>Exposición de las aplicaciones a Internet mediante Application Gateway y Azure Firewall

En este documento se explica cómo exponer las aplicaciones a Internet mediante Application Gateway y Azure Firewall. Cuando se implementa una instancia de servicio de Azure Spring Cloud en la red virtual, solo se puede acceder a las aplicaciones de la instancia de servicio en la red privada. Para que las aplicaciones sean accesibles en Internet, debe integrarlas con **Azure Application Gateway** y, opcionalmente, con **Azure Firewall**.

## <a name="prerequisites"></a>Requisitos previos

- [CLI de Azure, versión 2.0.4 o posterior](/cli/azure/install-azure-cli).

## <a name="define-variables"></a>Definición de variables

Defina las variables para el grupo de recursos y la red virtual que creó como se indicó en [Implementación de Azure Spring Cloud en una red virtual de Azure (inserción de red virtual)](./how-to-deploy-in-azure-virtual-network.md). Personalice los valores en función de su entorno real.

```
SUBSCRIPTION='subscription-id'
RESOURCE_GROUP='my-resource-group'
LOCATION='eastus'
SPRING_APP_PRIVATE_FQDN='my-azure-spring-cloud-hello-vnet.private.azuremicroservices.io'
VIRTUAL_NETWORK_NAME='azure-spring-cloud-vnet'
APPLICATION_GATEWAY_SUBNET_NAME='app-gw-subnet'
APPLICATION_GATEWAY_SUBNET_CIDR='10.1.2.0/24'
```

## <a name="login-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en la CLI de Azure y elija una suscripción activa.

```
az login
az account set --subscription ${SUBSCRIPTION}
```

## <a name="create-network-resources"></a>Crear recursos de red

La instancia de **Azure Application Gateway** que se va a crear unirá la misma red virtual que la instancia de servicio de Azure Spring Cloud —o a la red virtual emparejada a ella—. En primer lugar, cree una nueva subred para Application Gateway en la red virtual con `az network vnet subnet create`, y cree también una dirección IP pública como front-end de Application Gateway con `az network public-ip create`.

```
APPLICATION_GATEWAY_PUBLIC_IP_NAME='app-gw-public-ip'
az network vnet subnet create \
    --name ${APPLICATION_GATEWAY_SUBNET_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --vnet-name ${VIRTUAL_NETWORK_NAME} \
    --address-prefix ${APPLICATION_GATEWAY_SUBNET_CIDR}
az network public-ip create \
    --resource-group ${RESOURCE_GROUP} \
    --location ${LOCATION} \
    --name ${APPLICATION_GATEWAY_PUBLIC_IP_NAME} \
    --allocation-method Static \
    --sku Standard
```

## <a name="create-application-gateway"></a>Creación de una puerta de enlace de aplicaciones

Cree una puerta de enlace de aplicación mediante `az network application-gateway create` y especifique el nombre de dominio completo (FQDN) privado de la aplicación como servidores en el grupo de back-end. A continuación, actualice la configuración de HTTP mediante `az network application-gateway http-settings update` para usar el nombre de host del grupo de back-end.

```
APPLICATION_GATEWAY_NAME='my-app-gw'
az network application-gateway create \
    --name ${APPLICATION_GATEWAY_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --location ${LOCATION} \
    --capacity 2 \
    --sku Standard_v2 \
    --http-settings-cookie-based-affinity Enabled \
    --http-settings-port 443 \
    --http-settings-protocol Https \
    --public-ip-address ${APPLICATION_GATEWAY_PUBLIC_IP_NAME} \
    --vnet-name ${VIRTUAL_NETWORK_NAME} \
    --subnet ${APPLICATION_GATEWAY_SUBNET_NAME} \
    --servers ${SPRING_APP_PRIVATE_FQDN}
az network application-gateway http-settings update \
    --gateway-name ${APPLICATION_GATEWAY_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --name appGatewayBackendHttpSettings \
    --host-name-from-backend-pool true
```

La puerta de enlace de aplicaciones puede tardar hasta 30 minutos en crearse. Una vez creado, compruebe el estado del back-end mediante `az network application-gateway show-backend-health`.  Esto examina si la puerta de enlace de aplicación llega a la aplicación a través de su FQDN privado.

```
az network application-gateway show-backend-health \
    --name ${APPLICATION_GATEWAY_NAME} \
    --resource-group ${RESOURCE_GROUP}
```

La salida indica el estado correcto del grupo de back-end.

```
{
  "backendAddressPools": [
    {
      "backendHttpSettingsCollection": [
        {
          "servers": [
            {
              "address": "my-azure-spring-cloud-hello-vnet.private.azuremicroservices.io",
              "health": "Healthy",
              "healthProbeLog": "Success. Received 200 status code",
              "ipConfiguration": null
            }
          ]
        }
      ]
    }
  ]
}
```

## <a name="access-your-application-using-the-frontend-public-ip-of-the-application-gateway"></a>Acceso a la aplicación mediante la dirección IP pública de front-end de la puerta de enlace de aplicación

Obtenga la dirección IP pública de la puerta de enlace de aplicación mediante `az network public-ip show`.

```
az network public-ip show \
    --resource-group ${RESOURCE_GROUP} \
    --name ${APPLICATION_GATEWAY_PUBLIC_IP_NAME} \
    --query [ipAddress] \
    --output tsv
```

Copie la dirección IP pública y péguela en la barra de direcciones del explorador.

  ![Aplicación en dirección IP pública](media/spring-cloud-expose-apps-gateway-az-firewall/app-gateway-public-ip.png)

## <a name="see-also"></a>Consulte también

- [Solución de problemas de Azure Spring Cloud en una red virtual](./troubleshooting-vnet.md)
- [Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual](./vnet-customer-responsibilities.md)