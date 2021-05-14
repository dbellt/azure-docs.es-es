---
title: Preferencias de enrutamiento en Azure
description: Obtenga información acerca de cómo seleccionar el enrutamiento del tráfico entre Azure e Internet con las preferencias de enrutamiento.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/01/2021
ms.author: mnayak
ms.openlocfilehash: 588af3ff739d82029260909d23a7639bc2cbcb8d
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108069946"
---
# <a name="what-is-routing-preference"></a>¿Qué es la preferencia de enrutamiento?

Las preferencias de enrutamiento de Azure le permiten elegir cómo se enruta el tráfico entre Azure e Internet. Puede optar por enrutar el tráfico a través de la red de Microsoft o a través de una red de ISP (Internet público). Estas opciones también se conocen como *enrutamiento de tipo "cold-potato"* y *enrutamiento de tipo "hot-potato"* respectivamente. El precio de la transferencia de datos de salida varía en función de la selección de enrutamiento. Puede elegir la opción de enrutamiento al crear una dirección IP pública. Asimismo, la dirección IP pública se puede asociar a recursos como máquinas virtuales, conjuntos de escalado de máquinas virtuales, equilibradores de carga accesibles desde Internet, etc. También puede establecer la preferencia de enrutamiento para los recursos de Azure Storage, como blobs, archivos, web y Azure Data Lake. De forma predeterminada, el tráfico se enruta a través de la red global de Microsoft para todos los servicios de Azure.

## <a name="routing-via-microsoft-global-network"></a>Enrutamiento a través de la red global de Microsoft

Al enrutar el tráfico a través de la *red global de Microsoft*, este se entrega en una de las redes más grandes del mundo que abarca más de 160.000 kilómetros de fibra con más de 165 puntos de presencia (POP) perimetral. La red está bien aprovisionada y dispone de varias rutas de acceso de fibra redundantes para garantizar una confiabilidad y disponibilidad excepcionalmente altas. Igualmente, la ingeniería del tráfico se administra mediante un controlador WAN que define un software; este controlador se encarga de garantizar la selección de la ruta de acceso de baja latencia para el tráfico y ofrece un rendimiento de red Premium.

![Enrutamiento a través de la red global de Microsoft](media/routing-preference-overview/route-via-microsoft-global-network.png)

**Tráfico de entrada:** el anuncio de difusión por proximidad de BGP global garantiza que el tráfico de entrada acceda a la red de Microsoft más cercana al usuario. Por ejemplo, si un usuario de Singapur obtiene acceso a los recursos de Azure hospedados en Chicago (EE. UU.), el tráfico accede a la red global de Microsoft en el protocolo POP perimetral de Singapur y viaja por la red de Microsoft hasta el servicio hospedado en Chicago.

**Tráfico de salida:** el tráfico de salida sigue el mismo principio. El tráfico pasa la mayor parte de su recorrido en la red global de Microsoft y sale por la más cercana al usuario. Por ejemplo, si el tráfico de Azure Chicago está destinado a un usuario de Singapur, el tráfico viaja a través de la red de Microsoft desde Chicago a Singapur y sale de la red de Microsoft en el protocolo POP perimetral de Singapur.

Tanto el tráfico de entrada como el de salida se mantiene la mayor parte del tiempo en la red global de Microsoft. Esto también se conoce como *enrutamiento de tipo "cold potato"* .


## <a name="routing-over-public-internet-isp-network"></a>Enrutamiento a través de la red pública de Internet (red de ISP)

La nueva opción de enrutamiento denominada *enrutamiento de Internet* minimiza el viaje a través de la red global de Microsoft y utiliza la red de ISP de tránsito para enrutar el tráfico. Esta opción de enrutamiento le permite ahorrar costos y ofrece un rendimiento de red comparable a otros proveedores de la nube.

![Enrutamiento a través de la red pública de Internet](media/routing-preference-overview/route-via-isp-network.png)

**Tráfico de entrada:** la ruta de acceso de entrada usa un *enrutamiento de tipo "hot potato"* , lo que significa que el tráfico entra en la red de Microsoft más cercana a la región de servicio hospedada. Por ejemplo, si un usuario de Singapur obtiene acceso a los recursos de Azure hospedados en Chicago, el tráfico viaja a través de la red pública de Internet y entra en la red global de Microsoft en Chicago.

**Tráfico de salida:** el tráfico de salida sigue el mismo principio. El tráfico sale de la red de Microsoft en la misma región en la que se hospeda el servicio. Por ejemplo, si el tráfico del servicio en Azure Chicago está destinado a un usuario de Singapur, el tráfico sale de la red de Microsoft en Chicago y viaja a través de la red pública de Internet hasta el usuario en Singapur.

## <a name="supported-services"></a>Servicios admitidos

La dirección IP pública con la opción de preferencia de enrutamiento "red global de Microsoft" puede asociarse a cualquier servicio de Azure. Sin embargo, la dirección IP pública con la opción de preferencia de enrutamiento **Internet** puede asociarse a los siguientes recursos de Azure:

* Máquina virtual
* Conjunto de escalado de máquina virtual
* Azure Kubernetes Service (AKS)
* Equilibrador de carga accesible desde Internet
* Application Gateway
* Azure Firewall

En cuanto al almacenamiento, los puntos de conexión principales siempre usan la **red global de Microsoft**. Puede habilitar los puntos de conexión secundarios que prefiera con **Internet** para realizar el enrutamiento del tráfico. Los servicios de almacenamiento admitidos son:

* Datos BLOB
* Archivos
* Web
* Azure Data Lake

## <a name="pricing"></a>Precios
La diferencia de precio entre ambas opciones se refleja en los precios de transferencia de datos de salida de Internet. El precio de transferencia de datos para el enrutamiento a través de la **red global de Microsoft** es el mismo que el precio de salida actual de Internet. Consulte la [página de precios de ancho de banda de Azure](https://azure.microsoft.com/pricing/details/bandwidth/) para obtener la información más reciente sobre los precios.

## <a name="limitations"></a>Limitaciones


* La preferencia de enrutamiento solo es compatible con la SKU estándar con redundancia de zona de la dirección IP pública. No se admite la SKU básica de la dirección IP pública.
* Actualmente, la preferencia de enrutamiento solo admite direcciones IP públicas IPv4. No se admiten direcciones IP públicas IPv6.


## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre cómo optimizar la conectividad a los servicios de Microsoft Azure a través de Internet (vídeo)](https://www.youtube.com/watch?v=j6A_Mbpuh6s&list=PLLasX02E8BPA5V-waZPcelhg9l3IkeUQo&index=12) 
* [Configuración de la preferencia de enrutamiento de una VM mediante Azure PowerShell](configure-routing-preference-virtual-machine-powershell.md)
* [Configuración de la preferencia de enrutamiento de una VM mediante la CLI de Azure](configure-routing-preference-virtual-machine-cli.md)
