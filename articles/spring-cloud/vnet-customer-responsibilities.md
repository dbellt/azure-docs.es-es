---
title: Responsabilidades del cliente al ejecutar Azure Spring Cloud en una red virtual
description: En este artículo se describen las responsabilidades del cliente al ejecutar Azure Spring Cloud en una red virtual.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 12/02/2020
ms.custom: devx-track-java
ms.openlocfilehash: ed906c32e91ae168e17d9eb769d3fd1985098f9c
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109634642"
---
# <a name="customer-responsibilities-for-running-azure-spring-cloud-in-vnet"></a>Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual
En este documento se incluyen especificaciones para el uso de Azure Spring Cloud en una red virtual.

Cuando se implementa Azure Spring Cloud en la red virtual, tiene dependencias de salida de los servicios fuera de la red virtual. Para fines operativos y de administración, Azure Spring Cloud debe tener acceso a determinados puertos y nombres de dominio completo (FQDN). Estos puntos de conexión son obligatorios para la comunicación con el plano de administración de Azure Spring Cloud y para la descarga e instalación de las actualizaciones de seguridad y componentes principales del clúster de Kubernetes.

De forma predeterminada, Azure Spring Cloud tiene acceso de salida a Internet ilimitado. Este nivel de acceso a la red permite que las aplicaciones que ejecuta accedan a recursos externos según sea necesario. Si desea restringir el tráfico de salida, es necesario el acceso a un número limitado de puertos y direcciones para las tareas de mantenimiento. La solución más sencilla para proteger las direcciones de salida consiste en usar un dispositivo de firewall que pueda controlar el tráfico de salida en función de los nombres de dominio. Azure Firewall, por ejemplo, puede restringir el tráfico saliente HTTP y HTTPS en función del FQDN de destino. También puede configurar las reglas de seguridad y de firewall que prefiera para permitir estos puertos y direcciones necesarios.

## <a name="azure-spring-cloud-resource-requirements"></a>Requisitos de recursos de Azure Spring Cloud 

A continuación, se muestra una lista de los requisitos de recursos para los servicios de Azure Spring Cloud. Como requisito general, no debe modificar los grupos de recursos creados por Azure Spring Cloud y los recursos de red subyacentes.
- No modifique los grupos de recursos creados y que son propiedad de Azure Spring Cloud.
  - De forma predeterminada, estos grupos de recursos se denominan ap-svc-rt_[NOMBRE_INSTANCIA_SERVICIO] _[REGIÓN]* y ap_[NOMBRE-INSTANCIA-SERVICIO]_[REGIÓN]*.
- No modifique las subredes usadas por Azure Spring Cloud.
- No cree más de una instancia de servicio de Azure Spring Cloud en la misma subred.
- Cuando se usa un firewall para controlar el tráfico, *no* bloquee el siguiente tráfico de salida hacia los componentes de Azure Spring Cloud que operan, mantienen y admiten la instancia de servicio.

## <a name="azure-spring-cloud-network-requirements"></a>Requisitos de red de Azure Spring Cloud

  | Punto de conexión de destino | Port | Uso | Nota: |
  |------|------|------|------|
  | *:1194 *o* [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): AzureCloud:1194 | UDP:1194 | Administración de un clúster de Kubernetes subyacente. | |
  | *:443 *o* [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): AzureCloud:443 | TCP:443 | Administración del servicio de Azure Spring Cloud. | La información de la instancia de servicio "requiredTraffics" se puede conocer en la carga de recursos, en la sección "networkProfile". |
  | *:9000 *o* [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): AzureCloud:9000 | TCP:9000 | Administración de un clúster de Kubernetes subyacente. |
  | *:123 *o* ntp.ubuntu.com:123 | UDP:123 | Sincronización de la hora NTP en nodos Linux. | |
  | *.azure.io:443 *o* [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): AzureContainerRegistry:443 | TCP:443 | Azure Container Registry. | Se puede reemplazar habilitando el punto de conexión de servicio de *Azure Container Registry* [en la red virtual](../virtual-network/virtual-network-service-endpoints-overview.md). |
  | *.core.windows.net:443 and *.core.windows.net:445 *o* [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): Storage:443 y Storage:445 | TCP:443, TCP:445 | Azure File Storage | Se puede reemplazar habilitando el punto de conexión de servicio de *Azure Storage* [en la red virtual](../virtual-network/virtual-network-service-endpoints-overview.md). |
  | *.servicebus.windows.net:443 *o* [ServiceTag](../virtual-network/service-tags-overview.md#available-service-tags): EventHub:443 | TCP:443 | Centro de eventos de Azure. | Se puede reemplazar habilitando el punto de conexión de servicio de *Azure Event Hubs* [en la red virtual](../virtual-network/virtual-network-service-endpoints-overview.md). |
  

## <a name="azure-spring-cloud-fqdn-requirements--application-rules"></a>Requisitos y reglas de aplicación del FQDN de Azure Spring Cloud

Azure Firewall proporciona la etiqueta de nombre de dominio completo (FQDN) **AzureKubernetesService** para simplificar las siguientes configuraciones.

  | FQDN de destino | Port | Uso |
  |------|------|------|
  | *.azmk8s.io | HTTPS:443 | Administración de un clúster de Kubernetes subyacente. |
  | <i>mcr.microsoft.com</i> | HTTPS:443 | Microsoft Container Registry (MCR). |
  | *.cdn.mscr.io | HTTPS:443 | Almacenamiento de MCR respaldado por Azure CDN. |
  | *.data.mcr.microsoft.com | HTTPS:443 | Almacenamiento de MCR respaldado por Azure CDN. |
  | <i>management.azure.com</i> | HTTPS:443 | Administración de un clúster de Kubernetes subyacente. |
  | <i>*login.microsoftonline.com</i> | HTTPS:443 | Autenticación de Azure Active Directory. |
  | <i>*login.microsoft.com</i> | HTTPS:443 | Autenticación de Azure Active Directory. |
  |<i>packages.microsoft.com</i>    | HTTPS:443 | Repositorio de paquetes de Microsoft. |
  | <i>acs-mirror.azureedge.net</i> | HTTPS:443 | Repositorio necesario para instalar los archivos binarios necesarios, como kubenet y Azure CNI. |
  | *mscrl.microsoft.com* | HTTPS:80 | Rutas de acceso de la cadena de certificados de Microsoft obligatorias. |
  | *crl.microsoft.com* | HTTPS:80 | Rutas de acceso de la cadena de certificados de Microsoft obligatorias. |
  | *crl3.digicert.com* | HTTPS:80 | Rutas de acceso de la cadena de certificados SSL de terceros. |

## <a name="see-also"></a>Vea también
* [Acceso a una aplicación en una red privada](access-app-virtual-network.md)
* [Exposición de las aplicaciones mediante Application Gateway y Azure Firewall](expose-apps-gateway-azure-firewall.md)
