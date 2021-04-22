---
title: Habilitación de la seguridad de la capa de transporte de un extremo a otro
titleSuffix: Azure Spring Cloud
description: Habilitación de la seguridad de la capa de transporte de un extremo a otro para una aplicación
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 03/24/2021
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 647cf6f0b1af6a5858bbf1147cc03ecc4637ed25
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227813"
---
# <a name="enable-end-to-end-tls-for-an-application"></a>Habilitación de TLS de un extremo a otro para una aplicación

En este tema se muestra cómo habilitar SSL/TLS de un extremo a otro para proteger el tráfico desde un controlador de entrada a aplicaciones que admiten HTTPS. Después de habilitar TLS de un extremo a otro y cargar un certificado desde el almacén de claves, todas las comunicaciones en Azure Spring Cloud se protegen con TLS.

   ![Gráfico de comunicaciones protegidas por TLS](media/enable-end-to-end-tls/secured-tls.png)

## <a name="prerequisites"></a>Requisitos previos 

- Una instancia de Azure Spring Cloud implementada. Para comenzar, siga nuestro [inicio rápido sobre la implementación mediante la CLI de Azure](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-quickstart-launch-app-cli).
- Si no está familiarizado con TLS de un extremo a otro, consulte el [ejemplo de TLS de un extremo a otro](https://github.com/Azure-Samples/spring-boot-secure-communications-using-end-to-end-tls-ssl).
- Para cargar de forma segura los certificados necesarios en aplicaciones de Spring Boot, puede usar el [programa de inicio de Spring Boot de Key Vault](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/spring/azure-spring-boot-starter-keyvault-certificates).


## <a name="enable-end-to-end-tls-on-an-existing-app"></a>Habilitación de TLS de un extremo a otro en una aplicación existente

Use el comando `az spring-cloud app update --enable-end-to-end-tls` para habilitar o deshabilitar TLS de un extremo a otro para una aplicación.

```azurecli
az spring-cloud app update --enable-end-to-end-tls -n app_name -s service_name -g resource_group_name
az spring-cloud app update --enable-end-to-end-tls false -n app_name -s service_name -g resource_group_name
```

## <a name="enable-end-to-end-tls-when-you-bind-custom-domain"></a>Habilitación de TLS de un extremo a otro al enlazar un dominio personalizado

Use el comando `az spring-cloud app custom-domain update --enable-end-to-end-tls` o `az spring-cloud app custom-domain bind --enable-end-to-end-tls` para habilitar o deshabilitar TLS de un extremo a otro para una aplicación.

```azurecli
az spring-cloud app custom-domain update --enable-end-to-end-tls -n app_name -s service_name -g resource_group_name
az spring-cloud app custom-domain bind --enable-end-to-end-tls -n app_name -s service_name -g resource_group_name
```

## <a name="verify-end-to-end-tls-status"></a>Comprobación del estado de TLS de un extremo a otro

Use el comando `az spring-cloud app show` para comprobar el valor de `enableEndToEndTls`.
```
az spring-cloud app show -n app_name -s service_name -g resource_group_name
```

## <a name="next-steps"></a>Pasos siguientes
* [Acceso a Config Server y Service Registry](how-to-access-data-plane-azure-ad-rbac.md)
