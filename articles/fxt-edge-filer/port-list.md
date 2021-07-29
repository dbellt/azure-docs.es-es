---
title: Lista de puertos para Azure FXT Edge Filer
description: Lista de los puertos TCP/UDP que los entornos de clúster de FXT usan
author: ekpgh
ms.author: v-erkel
ms.service: fxt-edge-filer
ms.topic: conceptual
ms.date: 05/26/2021
ms.openlocfilehash: 28de9732f6a22f730059c08b5be939c23e52ebaa
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111415211"
---
# <a name="required-network-ports"></a>Puertos de red necesarios

En esta lista se muestran los puertos TCP/UDP necesarios para los entornos de clúster de FXT. Configure los firewalls que use para asegurarse de que estos puertos son accesibles.

Los requisitos específicos del sistema variarán en función del tipo de almacenamiento de back-end que use.

Para más información, póngase en contacto con el servicio y el soporte técnico de Microsoft.

## <a name="api-ports"></a>Puertos de API

| Dirección | Tipo | Número de puerto | Protocolo |
|-----------|------|-------------|----------|
| Entrada   | TCP  | 22          | SSH      |
| Salida  | TCP  | 80          | HTTP     |
| Entrante y saliente  | TCP  | 443         | HTTPS    |

## <a name="nfs-ports"></a>Puertos NFS

Puertos NFS entrantes:

| Tipo    | Número de puerto | Servicio  |
|---------|-------------|----------|
| TCP/UDP | 111         | RPCBIND  |
| TCP/UDP | 2049        | NFS      |
| TCP/UDP | 4045        | NLOCKMGR |
| TCP/UDP | 4046        | MOUNTD   |
| TCP/UDP | 4047        | STATUS   |

Puertos NFS salientes:

| Tipo    | Número de puerto | Servicio  |
|---------|-------------|----------|
| TCP/UDP | 111         | RPCBIND  |
| TCP/UDP | 2049        | NFS      |

El tráfico del puerto NFS saliente varía en función del tipo de almacenamiento utilizado como un archivador principal. (Algunos sistemas no usan el puerto 2049, aunque es lo suficientemente común para que aparezca aquí. Todos los clústeres necesitan acceso en el puerto 111). Consulte la documentación de los sistemas de almacenamiento o use la técnica de consulta que se describe a continuación en [Requisitos de puertos adicionales](#additional-port-requirements).

Algún tráfico NFS saliente de los nodos FXT usan puertos efímeros. El tráfico FXT saliente por encima de los puertos conocidos no debe restringirse en el nivel de transporte.

## <a name="smb-ports"></a>Puertos SMB

| Dirección | Tipo | Número de puerto | Protocolo |
|-----------|------|-------------|----------|
| Entrante y saliente  | UDP  | 137         | NetBIOS  |
| Entrada   | UDP  | 138         | NetBIOS  |
| Entrante y saliente  | TCP  | 139         | SMB      |
| Entrante y saliente  | TCP  | 445         | SMB      |

<!--| Outbound  | UDP  | 137         | NetBIOS  | 
| Outbound  | TCP  | 139         | SMB      |
| Outbound  | TCP  | 445         | SMB      |
-->

## <a name="general-traffic-ports"></a>Puertos de tráfico generales

| Dirección | Tipo    | Número de puerto | Protocolo |
|-----------|---------|-------------|----------|
| Salida  | TCP/UDP | 53          | DNS      |
| Salida  | TCP/UDP | 88          | Kerberos |
| Salida  | UDP     | 123         | NTP      |
| Salida  | TCP/UDP | 389         | LDAP     |
| Salida  | TCP     | 686         | LDAPS    |

## <a name="additional-port-requirements"></a>Requisitos de puertos adicionales

Los archivadores principales pueden requerir acceso en puertos adicionales. Este requisito varía en función del tipo de almacenamiento utilizado.

Puede usar el comando `rpcinfo` para saber qué puertos usa un servidor determinado. Emita este comando desde un sistema cliente que no tenga un firewall:

`rpcinfo -p <server_IP_address>`

## <a name="next-steps"></a>Pasos siguientes

* Información sobre cómo [agregar almacenamiento](add-storage.md) al clúster de Azure FXT Edge Filer
* [Póngase en contacto con el soporte técnico](support-ticket.md) para obtener más información sobre la configuración de puertos.
