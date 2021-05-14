---
title: 'Azure VMware Solution by CloudSimple: uso de la nube privada como sitio de recuperación ante desastres de cargas de trabajo locales'
description: Describe cómo configurar la nube privada de CloudSimple como un sitio de recuperación ante desastres de las cargas de trabajo locales de VMware.
author: shortpatti
ms.author: v-patsho
ms.date: 08/20/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: e7305557944e00121c54112704f67f861d2aa789
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108177789"
---
# <a name="set-up-cloudsimple-private-cloud-as-a-disaster-recovery-site-for-on-premises-vmware-workloads"></a>Configuración de la nube privada de CloudSimple como un sitio de recuperación ante desastres de las cargas de trabajo locales de VMware

La nube privada de CloudSimple se puede configurar como un sitio de recuperación de aplicaciones locales para procurar la continuidad empresarial en caso de que se produzca un desastre. La solución de recuperación se basa en Zerto Virtual Replication como plataforma de replicación y orquestación. La infraestructura y las máquinas virtuales de aplicación críticas se pueden replicar continuamente desde la instancia de vCenter local a la nube privada. Puede usar la nube privada para realizar pruebas de conmutación por error y para garantizar la disponibilidad de la aplicación durante un desastre. Se puede seguir un método similar para configurar la nube privada como sitio primario protegido por un sitio de recuperación en otra ubicación.

> [!NOTE]
> Consulte el documento de Zerto sobre las [consideraciones de tamaño de Zerto Virtual Replication](https://s3.amazonaws.com/zertodownload_docs/5.5U3/Zerto%20Virtual%20Replication%20Sizing.pdf) para obtener instrucciones sobre cómo cambiar el tamaño del entorno de recuperación ante desastres.

La solución de CloudSimple:

* Elimina la necesidad de configurar un centro de datos específico para la recuperación ante desastres.
* Permite aprovechar las ubicaciones de Azure en que se implementa CloudSimple para lograr resistencia geográfica internacional.
* Ofrece una opción para reducir los costos de implementación y el costo total de propiedad a fin de establecer la recuperación ante desastres.

La solución requiere lo siguiente:

* Instalar, configurar y administrar Zerto en la nube privada.
* Proporcionar sus propias licencias de Zerto cuando la nube privada sea el sitio protegido. La instancia de Zerto que se ejecuta en el sitio de CloudSimple se puede emparejar con el sitio local para la concesión de licencias.

En la siguiente imagen se muestra la arquitectura de la solución de Zerto.

![Arquitectura](media/cloudsimple-zerto-architecture.png)

## <a name="how-to-deploy-the-solution"></a>Implementación de la solución

En las secciones siguientes se describe cómo implementar una solución de recuperación ante desastres con Zerto Virtual Replication en la nube privada.

1. [Requisitos previos](#prerequisites)
2. [Configuración opcional en la nube privada de CloudSimple](#optional-configuration-on-your-private-cloud)
3. [Configuración de ZVM y VRA en la nube privada de CloudSimple](#set-up-zvm-and-vra-on-your-private-cloud)
4. [Configuración de un grupo de protección virtual de Zerto](#set-up-zerto-virtual-protection-group)

### <a name="prerequisites"></a>Requisitos previos

Complete los siguientes requisitos previos para habilitar Zerto Virtual Replication desde el entorno local a la nube privada.

1. [Configure una conexión VPN de sitio a sitio entre la red local y la nube privada de CloudSimple](set-up-vpn.md).
2. [Configure la búsqueda DNS para que los componentes de administración de la nube privada se reenvíen a servidores DNS de la nube privada](on-premises-dns-setup.md).  Para habilitar el reenvío de búsqueda DNS, cree una entrada de zona de reenvío en el servidor DNS local para `*.cloudsimple.io` en los servidores DNS de CloudSimple.
3. Configure la búsqueda DNS para que los componentes de vCenter locales se reenvíen a los servidores DNS locales.  Los servidores DNS deben ser accesibles desde la nube privada CloudSimple a través de la conexión VPN de sitio a sitio. Para obtener ayuda, envíe una [solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) y facilite la siguiente información.  

    * Nombre del dominio DNS local
    * Direcciones IP del servidor DNS local

4. Instale un servidor de Windows en la nube privada. Este servidor servirá para instalar Zerto Virtual Manager.
5. [Escale sus privilegios de CloudSimple](escalate-private-cloud-privileges.md).
6. Cree un usuario con el rol administrativo en la instancia de vCenter de la nube privada para usarlo como cuenta de servicio en Zerto Virtual Manager.

### <a name="optional-configuration-on-your-private-cloud"></a>Configuración opcional en la nube privada de CloudSimple

1. Cree uno o varios grupos de recursos en la instancia de vCenter de la nube privada para usarlos como grupos de recursos de destino de las máquinas virtuales del entorno local.
2. Cree una o varias carpetas en la instancia de vCenter de la nube privada para usarlas como carpetas de destino de las máquinas virtuales del entorno local.
3. Cree redes VLAN como redes de conmutación por error y configure reglas de firewall. Tramite una [solicitud de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) para obtener ayuda.
4. Cree grupos de puertos distribuidos de la red de prueba y la red de conmutación por error para probar la conmutación por error de las máquinas virtuales.
5. Instale [servidores DHCP y DNS](dns-dhcp-setup.md), o bien use un controlador de dominio de Active Directory en el entorno de nube privada.

### <a name="set-up-zvm-and-vra-on-your-private-cloud"></a>Configuración de ZVM y VRA en la nube privada

1. Instale Zerto Virtual Manager (ZVM) en el servidor de Windows en la nube privada.
2. Inicie sesión en ZVM con la cuenta de servicio creada en los pasos anteriores.
3. Configure las licencias de Zerto Virtual Manager.
4. Instale Zerto virtual Replication Appliance (VRA) en los hosts ESXi de la nube privada.
5. Empareje la instancia de ZVM de la nube privada con la instancia de ZVM local.

### <a name="set-up-zerto-virtual-protection-group"></a>Configuración de un grupo de protección virtual de Zerto

1. Cree un grupo de protección virtual (VPG) y establezca su prioridad.
2. Seleccione las máquinas virtuales que requieren protección para la continuidad empresarial y personalice el orden de arranque, si es necesario.
3. Seleccione el sitio de recuperación como la nube privada y el servidor de recuperación predeterminado como el clúster de nube privada o el grupo de recursos que creó. Seleccione **vsanDatastore** para el almacén de datos de recuperación de la nube privada.

    ![VPG](media/cloudsimple-zerto-vpg.png)

    > [!NOTE]
    > Puede personalizar la opción de host de máquinas virtuales concretas en la opción Configuración de la VM.

4. Personalice las opciones de almacenamiento según sea necesario.
5. Especifique las redes de recuperación que se usarán como red de conmutación por error y red de prueba de conmutación por error como los grupos de puertos distribuidos creados anteriormente, y personalice los scripts de recuperación según sea necesario.
6. Personalice la configuración de red de máquinas virtuales concretas si procede y cree el VPG.
7. Pruebe la conmutación por error una vez que la replicación haya terminado.

## <a name="reference"></a>Referencia

[Documentación de Zerto](https://www.zerto.com/myzerto/technical-documentation/)
