---
title: 'Azure VMware Solution by CloudSimple: configuración de DNS y DHCP de carga de trabajo para la nube privada'
description: Describe cómo configurar DNS y DHCP para las aplicaciones y cargas de trabajo que se ejecutan en el entorno de la nube privada de CloudSimple.
author: shortpatti
ms.author: v-patsho
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 4b8e5cddb8616cc346cce16cc64c6770699a94c6
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176907"
---
# <a name="set-up-dns-and-dhcp-applications-and-workloads-in-your-cloudsimple-private-cloud"></a>Configuración de cargas de trabajo y aplicaciones de DNS y DHCP en la nube privada de CloudSimple

Las aplicaciones y cargas de trabajo que se ejecutan en un entorno de nube privada requieren servicios de resolución de nombres y DHCP para la búsqueda y asignación de direcciones IP.  Para proporcionar estos servicios se requiere una infraestructura de DHCP y DNS adecuada.  Puede configurar una máquina virtual para proporcionar estos servicios en el entorno de nube privada.  

## <a name="prerequisites"></a>Prerrequisitos

* Un grupo de puertos distribuidos con VLAN configurada
* Ruta configurada a servidores DNS basados en Internet o locales
* Plantilla de máquina virtual o archivo ISO para crear una máquina virtual

## <a name="linux-based-dns-server-setup"></a>Configuración del servidor DNS basado en Linux

Linux ofrece diversos paquetes para la configuración de servidores DNS.  A continuación se muestra un [ejemplo de configuración de DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-configure-bind-as-a-private-network-dns-server-on-ubuntu-18-04) con instrucciones para configurar un servidor DNS BIND de código abierto.

## <a name="windows-based-setup"></a>Configuración basada en Windows

En estos temas de Microsoft se describe cómo configurar un servidor de Windows como un servidor DNS y como un servidor DHCP.

* [Windows Server como servidor DNS](/windows-server/networking/dns/dns-top)
* [Windows Server como servidor DHCP](/windows-server/networking/technologies/dhcp/dhcp-top)
