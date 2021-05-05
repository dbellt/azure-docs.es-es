---
title: Asegurar la alta disponibilidad de una aplicación al ejecutarse en VMware en Azure
description: Describe las características de alta disponibilidad de CloudSimple para abordar escenarios comunes de error de aplicaciones que se ejecutan en una nube privada de CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 20cd474c36749cfbe9848dd54f2a38892bd80ec0
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177753"
---
# <a name="ensure-application-high-availability-when-running-in-vmware-on-azure"></a>Asegurar la alta disponibilidad de una aplicación al ejecutarse en VMware en Azure

La solución CloudSimple proporciona alta disponibilidad para las aplicaciones que se ejecutan en VMware en el entorno de Azure. En la tabla siguiente se enumeran los escenarios de error y las características de alta disponibilidad asociadas.

|  Escenario de error  |  ¿Aplicación protegida?  |  Característica de alta disponibilidad de la plataforma  |  Característica de alta disponibilidad de VMware  |  Característica de alta disponibilidad de Azure  |
|----------------------------------------|------------------------|-------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------------------------------------|
|  Error de disco  |  SÍ  |  Sustitución rápida del nodo con errores  |  [Acerca de la directiva de almacenamiento predeterminada de vSAN](https://docs.vmware.com/en/VMware-vSphere/6.7/com.vmware.vsphere.virtualsan.doc/GUID-C228168F-6807-4C2A-9D74-E584CAF49A2A.html)  |  |
|  Error del ventilador  |  SÍ  |  Ventiladores redundantes, sustitución rápida del nodo con errores  |  |  |
|  Error de NIC  |  SÍ  |  NIC redundante, sustitución rápida del nodo con errores  |  |  |
|  Error de alimentación del host  |  SÍ  |  Fuente de alimentación redundante  |  |  |
|  Error del host ESXi  |  SÍ  |  Sustitución rápida del nodo con errores  |  [Alta disponibilidad de VMware vSphere](https://www.vmware.com/products/vsphere/high-availability.html)  |  |
|  Error de máquina virtual  |  SÍ  |  [Equilibradores de carga](load-balancers.md)  |  [Alta disponibilidad de VMware vSphere](https://www.vmware.com/products/vsphere/high-availability.html)  |  Azure Load Balancer para máquinas virtuales de VMware sin estado  |
|  Error del puerto de conmutador de hoja  |  SÍ  |  NIC redundante  |  |  |
|  Error del conmutador de hoja  |  SÍ  |  Conmutadores de hoja redundantes  |  |  |
|  Error de bastidor  |  SÍ  |  Grupos de selección de ubicación  |  |  |
|  Conectividad de red a un CD local  |  SÍ  |  Servicios de red redundantes  |  |  Circuitos de ER redundantes  |
|  Conectividad de red a Azure  |  SÍ  |  |  |  Circuitos de ER redundantes  |
|  Error del centro de datos  |  SÍ  |  |  |  Zonas de disponibilidad  |
|  Error regional  |  SÍ  |  |  |  Regiones de Azure  |

Azure VMware Solution by CloudSimple proporciona las siguientes características de alta disponibilidad.

## <a name="fast-replacement-of-failed-node"></a>Sustitución rápida del nodo con errores

El software de plano de control de CloudSimple supervisa continuamente el estado de los clústeres de VMware y detecta cuándo se produce un error en un nodo ESXi. Luego, agrega automáticamente un nuevo host ESXi al clúster de VMware afectado a partir del grupo de nodos de disponibilidad inmediata y retira el nodo con errores del clúster. Esta funcionalidad garantiza que la capacidad de reserva en el clúster de VMware se restaure rápidamente para que se restaure la resistencia del clúster proporcionada por la alta disponibilidad de vSAN y VMware HA.

## <a name="placement-groups"></a>Grupos de selección de ubicación

Un usuario que cree una nube privada puede seleccionar una región de Azure y un grupo de selección de ubicación dentro de la región seleccionada. Un grupo de selección de ubicación es un conjunto de nodos distribuidos entre varios bastidores, pero dentro del mismo segmento de red vertebral. Los nodos dentro del mismo grupo de selección de ubicación pueden comunicarse entre sí con un máximo de dos saltos de conmutador adicionales. Un grupo de selección de ubicación siempre se encuentra dentro de una sola zona de disponibilidad de Azure y abarca varios bastidores. El plano de control de CloudSimple distribuye los nodos de una nube privada entre varios bastidores en función del mejor esfuerzo. Se garantiza que los nodos de grupos de selección de ubicación diferentes se colocarán en distintos bastidores.

## <a name="availability-zones"></a>Zonas de disponibilidad

Las zonas de disponibilidad son una oferta de alta disponibilidad que protege las aplicaciones y datos de los errores del centro de datos. Las zonas de disponibilidad son ubicaciones físicas especiales dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Cada región tiene una zona de disponibilidad. Para obtener más información, consulte [¿Qué son las zonas de disponibilidad en Azure?](../availability-zones/az-overview.md)

## <a name="redundant-azure-expressroute-circuits"></a>Circuitos ExpressRoute redundantes

La conectividad del centro de datos a una red virtual de Azure mediante ExpressRoute tiene circuitos redundantes para proporcionar un vínculo de conectividad de red de alta disponibilidad.

## <a name="redundant-networking-services"></a>Servicios de red redundantes

Todos los servicios de red de CloudSimple para la nube privada (incluidas la VLAN, el firewall, las direcciones IP públicas, Internet y VPN) están diseñados para tener una alta disponibilidad y poder admitir el contrato de nivel de servicio.

## <a name="azure-layer-7-load-balancer-for-stateless-vmware-vms"></a>Equilibrador de carga de capa 7 de Azure para VM de VMware sin estado

Los usuarios pueden colocar un equilibrador de carga de nivel 7 de Azure delante de las VM de nivel web sin estado que se ejecutan en el entorno de VMware para lograr una alta disponibilidad para el nivel web.

## <a name="azure-regions"></a>Regiones de Azure

Una región de Azure es un conjunto de centros de datos implementados dentro de un perímetro definido por la latencia y conectados a través de una red regional dedicada de baja latencia. Para más detalles, consulte [Regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions).
