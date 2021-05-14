---
title: Extensión de subred en Azure | Microsoft Docs
description: Más información sobre la extensión de subred en Azure.
services: virtual-network
documentationcenter: na
author: anupam-p
manager: narayan
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/31/2019
ms.author: anupand
ms.openlocfilehash: e39859e3cc4d28ac4b1456fcdaec65ecfb9c7f31
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107728219"
---
# <a name="subnet-extension"></a>Extensión de subred
La migración de cargas de trabajo a la nube pública requiere una cuidadosa planeación y coordinación. Una de las consideraciones clave puede ser la capacidad de conservar las direcciones IP. Esto puede ser importante especialmente si las aplicaciones tienen dependencias de dirección IP o si tiene requisitos de cumplimiento para usar direcciones IP específicas. Azure Virtual Network resuelve este problema, ya que le permite crear una red virtual y subredes con el intervalo de direcciones IP que usted prefiera.

Las migraciones pueden llegar a ser un reto cuando el requisito anterior se combina con un requisito adicional para mantener algunas aplicaciones en el entorno local. En tal caso, tendrá que dividir las aplicaciones entre Azure y el entorno local, sin tener que volver a numerar las direcciones IP en ambos lados. Además, tendrá que permitir que las aplicaciones se comuniquen como si estuvieran en la misma red.

Una solución al problema anterior es la extensión de subred. La extensión de una red permite a las aplicaciones comunicarse en el mismo dominio de difusión cuando existen en diferentes ubicaciones físicas, lo que elimina la necesidad de rediseñar la topología de red. 

Aunque la extensión de la red no es una buena práctica en general, los casos de uso siguientes pueden hacer que sea necesario.

- **Migración por fases**: el escenario más común es que desee escalonar la migración. Desea llevar algunas aplicaciones primero y, con el tiempo, migrar el resto de las aplicaciones a Azure.
- **Latencia**: los requisitos de baja latencia pueden ser otro motivo para mantener algunas aplicaciones en el entorno local con el fin de asegurarse de que están lo más cerca posible del centro de recursos.
- **Cumplimiento**: otro caso de uso es que podría tener requisitos de cumplimiento para mantener algunas de sus aplicaciones en el entorno local.
 
> [!NOTE] 
> No debe extender las subredes a menos que sea necesario. En los casos en que extienda las subredes, debe intentar que sea un paso intermedio. Con el tiempo, debe intentar volver a numerar las aplicaciones en la red local y migrarlas a Azure.

En la siguiente sección analizaremos cómo puede extender las subredes a Azure.


## <a name="extend-your-subnet-to-azure"></a>Extensión de la subred a Azure
 Puede extender las subredes locales a Azure con una solución basada en la red de superposición de nivel 3. La mayoría de las soluciones usan una tecnología de superposición, como VXLAN, para extender la red de nivel 2 mediante una red de superposición de nivel 3. En el diagrama siguiente se muestra una solución generalizada. En esta solución, la misma subred existe en ambos lados, es decir, en Azure y en el entorno local. 

![Ejemplo de extensión de subred](./media/subnet-extension/subnet-extension.png)

Las direcciones IP de la subred se asignan a las máquinas virtuales en Azure y en el entorno local. Tanto Azure como el entorno local tienen un NVA insertado en sus redes. Cuando una máquina virtual de Azure intenta comunicarse con una máquina virtual en una red local, el NVA de Azure captura el paquete, lo encapsula y lo envía a través de VPN/ExpressRoute a la red local. El NVA local recibe el paquete, lo desencapsula y lo reenvía al destinatario previsto en su red. El tráfico de retorno utiliza una ruta y lógica similares.

En el ejemplo anterior, el NVA de Azure y el NVA local se comunican y aprenden sobre las direcciones IP que tienen detrás de ellos. Las redes más complejas también pueden tener un servicio de asignación, que mantiene la asignación entre los NVA y las direcciones IP detrás de ellos. Cuando un NVA recibe un paquete, consulta el servicio de asignación para averiguar la dirección del NVA que tiene la dirección IP de destino detrás de él.

En la sección siguiente, encontrará información sobre las soluciones de extensión de subred que hemos probado en Azure.

## <a name="next-steps"></a>Pasos siguientes 
[Ampliación de las subredes locales a Azure mediante Azure Extended Network](https://docs.microsoft.com/windows-server/manage/windows-admin-center/azure/azure-extended-network).