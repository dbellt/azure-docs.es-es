---
title: Configuración de Ethernet de BareMetal para Oracle
description: Obtenga información acerca de la configuración de interfaces Ethernet en instancias de BareMetal para cargas de trabajo de Oracle.
ms.topic: reference
ms.subservice: workloads
ms.date: 04/14/2021
ms.openlocfilehash: c57cbc86d17090d6960a334c2790d80b43420aca
ms.sourcegitcommit: 272351402a140422205ff50b59f80d3c6758f6f6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2021
ms.locfileid: "107588894"
---
# <a name="ethernet-configuration-of-baremetal-for-oracle"></a>Configuración de Ethernet de BareMetal para Oracle

En este artículo, se explicará la configuración de interfaces Ethernet en instancias de BareMetal para cargas de trabajo de Oracle.

Cada instancia de BareMetal aprovisionada para Oracle está preconfigurada con conjuntos de interfaces Ethernet. Las interfaces Ethernet se clasifican en cuatro tipos:

- se usa para el acceso de cliente.
- se usa para la comunicación nodo a nodo. Esta interfaz se configura en todos los servidores, sin importar la topología solicitada. Solo se usa para los escenarios de escalabilidad horizontal.
- se usa para la conectividad nodo a almacenamiento.
- Se usa para la configuración de la recuperación ante desastres (DR) y para la conectividad con Global Reach para la conectividad entre regiones.

## <a name="architecture"></a>Architecture

En el diagrama siguiente se muestra la arquitectura de las interfaces Ethernet preconfiguradas en BareMetal Infrastructure. 

[![Diagrama que muestra la arquitectura de las interfaces Ethernet preconfiguradas para cargas de trabajo de Oracle.](media/oracle-baremetal-ethernet/architecture-ethernet.png)](media/oracle-baremetal-ethernet/architecture-ethernet.png#lightbox)

La configuración predeterminada incluye una interfaz IP de cliente (eth1) que se conecta desde la instancia de Azure Virtual Network (VNET) con la que puede usar Secure Shell (SSH) para acceder a una instancia de BareMetal.

> [!NOTE]
> En el caso de otra interfaz de cliente (eth10) de otra VNET de Azure, póngase en contacto con su CSA de Microsoft para enviar una solicitud de servicio. Por ejemplo, si quiere entornos de desarrollo/pruebas, así como de producción o recuperación ante desastres.

| **Interfaz lógica de NIC** | **Nombre con sistema operativo RHEL** | **Caso de uso** |
| --- | --- | --- |
| A | net1.inquilino | Del cliente a la instancia de BareMetal. |
| C | net2.inquilino | Del nodo al almacenamiento; admite la coordinación y el acceso a los controladores de almacenamiento para la administración del entorno de almacenamiento. |
| B | net3.inquilino | De nodo a nodo (interconexión privada) |
| C | net4.inquilino | Reservado/iSCSI |
| C | net5.inquilino | Reservado/Copia de seguridad de registros |
| C | net6.inquilino | Del nodo al almacenamiento_copia de seguridad de datos (RMAN, instantánea). |
| C | net7.inquilino | Del nodo al almacenamiento storage_DNFS-Pri; proporciona conectividad con la matriz de almacenamiento de NetApp. |
| C | net8.inquilino | Del nodo al almacenamiento_DNFS-Sec; proporciona conectividad con la matriz de almacenamiento de NetApp. |
| D | net9.inquilino | Conectividad de recuperación ante desastres para la configuración de Global Reach a fin de acceder a BMI en otra región. |
| A | \*net10.inquilino | \* Del cliente a la instancia de BareMetal.
 |

Si es necesario, puede definir más interfaces de red por su cuenta. Sin embargo, las configuraciones de NIC existentes *no se pueden* cambiar.

## <a name="usage-rules"></a>Reglas de uso

En el caso de las instancias de BareMetal, el valor predeterminado tendrá nueve direcciones IP asignadas en las cuatro interfaces de red lógicas. Se aplican las siguientes reglas de uso:

- Ethernet "A" debe tener asignada una dirección IP que esté fuera del intervalo de direcciones del grupo de direcciones IP de servidor que envió a Microsoft. Esta dirección IP no se debe mantener en el directorio /etc/hosts del sistema operativo.
- Ethernet "B" se debe mantener exclusivamente en el directorio etc/hosts para la comunicación entre las distintas instancias. Mantenga estas direcciones IP en las configuraciones de escalabilidad horizontal de Oracle Real Application Clusters (RAC) como las direcciones IP que se usarán para la configuración entre nodos.
- Ethernet "C" debe tener asignada una dirección IP que se use para la comunicación con el almacenamiento NFS. Este tipo de dirección no se debe mantener en el directorio etc/hosts.
- Ethernet "D" debe usarse exclusivamente para la configuración de Global Reach con el objetivo de acceder a las instancias de BareMetal en la región de recuperación ante desastres.

## <a name="next-step"></a>Paso siguiente

Más información sobre la arquitectura de BareMetal Infrastructure para Oracle.

> [!div class="nextstepaction"]
> [Arquitectura de BareMetal Infrastructure para Oracle](oracle-baremetal-architecture.md)
