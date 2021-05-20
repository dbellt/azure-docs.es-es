---
title: Conectarse a una red del mismo nivel en Azure Lab Services | Microsoft Docs
description: Aprenda a conectar la red de laboratorio a otra red como una red del mismo nivel. Por ejemplo, conecte la red local de la organización o la universidad a la red virtual de laboratorio en Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 645e99681f5615c3082d784e4e540a6200f618b8
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108755142"
---
# <a name="connect-your-labs-network-with-a-peer-virtual-network-in-azure-lab-services"></a>Conexión de la red del laboratorio con una red virtual del mismo nivel en Azure Lab Services

En este artículo encontrará información sobre cómo emparejar su red de laboratorio con otra red.

## <a name="overview"></a>Información general

El emparejamiento de redes virtuales permite conectar sin problemas redes virtuales de Azure. Una vez emparejadas, a efectos de conectividad las redes virtuales aparecen como una sola. El tráfico entre las máquinas virtuales de las redes virtuales emparejadas se enruta a través de la infraestructura de la red troncal de Microsoft, de forma muy parecida a como se enruta el tráfico entre máquinas virtuales de la misma red virtual a través únicamente de direcciones IP privadas. Para más información, consulte [Emparejamiento de redes virtuales](../virtual-network/virtual-network-peering-overview.md).

Puede que necesite conectar la red de laboratorio con una red virtual del mismo nivel en algunos escenarios, como los siguientes:

- Las máquinas virtuales del laboratorio tienen software que se conecta a servidores de licencias locales para adquirir una licencia.
- Las máquinas virtuales del laboratorio necesitan acceder a conjuntos de datos (o a cualquier otro archivo) en recursos compartidos de red de la universidad.

Algunas redes locales están conectadas a Azure Virtual Network ya sea a través de [ExpressRoute](../expressroute/expressroute-introduction.md) o de una [puerta de enlace de red virtual](../vpn-gateway/vpn-gateway-about-vpngateways.md). Estos servicios se deben configurar fuera de Azure Lab Services. Para obtener más información sobre cómo conectar una red local a Azure mediante ExpressRoute, consulte [Información general sobre ExpressRoute](../expressroute/expressroute-introduction.md). Para lograr la conectividad local mediante una puerta de enlace de red virtual, tanto la puerta de enlace como la red virtual y la cuenta de laboratorio deben estar en la misma región.

> [!NOTE]
> Al crear una instancia de Azure Virtual Network que se emparejará con una cuenta de laboratorio, es importante comprender cómo impacta la región de la red virtual en el lugar donde se crean los laboratorios.  Para más información, consulte la sección de la guía del administrador sobre [regiones o ubicaciones ](./administrator-guide.md#regionslocations).

> [!NOTE]
> Si la escuela necesita realizar filtrado de contenido, por ejemplo, para el cumplimiento de la [Ley de Protección de la Infancia en Internet (Children's Internet Protection Act, CIPA)](https://www.fcc.gov/consumers/guides/childrens-internet-protection-act), tiene que usar software de terceros.  Para obtener más información, lea las instrucciones sobre [filtrado de contenido con Lab Services](./administrator-guide.md#content-filtering).

## <a name="configure-at-the-time-of-lab-account-creation"></a>Configuración al crear la cuenta de laboratorio

Durante la [creación de la cuenta de laboratorio](tutorial-setup-lab-account.md), puede elegir una red virtual existente de la lista desplegable **Emparejar red virtual** en la pestaña **Opciones avanzadas**.  En la lista solo se mostrarán las redes virtuales de la misma región que la cuenta de laboratorio. La red virtual seleccionada se conecta (empareja) a los laboratorios creados con la cuenta de laboratorio.  Todas las máquinas virtuales de los laboratorios que se creen después de hacer este cambio tendrán acceso a los recursos de la red virtual emparejada.

![Selección de una red virtual para emparejamiento](./media/how-to-connect-peer-virtual-network/select-vnet-to-peer.png)

### <a name="address-range"></a>Intervalo de direcciones

También hay una opción para proporcionar un **intervalo de direcciones** para las máquinas virtuales de los laboratorios.  La propiedad **Intervalo de direcciones** solo se aplica si se ha habilitado **Emparejar Red virtual** para el laboratorio. Si se proporciona el intervalo de direcciones, todas las máquinas virtuales de los laboratorios de la cuenta de laboratorio se crearán en ese intervalo de direcciones. El intervalo de direcciones debe estar en notación CIDR (por ejemplo, 10.20.0.0/20) y no superponerse con ningún intervalo de direcciones existente.  Al proporcionar un intervalo de direcciones, es importante pensar en el número de *laboratorios* que se crearán e indicar un intervalo de direcciones para acomodarlos. Lab Services presupone un máximo de 512 máquinas virtuales por laboratorio.  Por ejemplo, un intervalo de direcciones IP con "/23" solo puede crear un laboratorio.  Un intervalo con "/21" permitirá la creación de cuatro laboratorios.

Si no se especifica el **intervalo de direcciones**, Lab Services usa el intervalo de direcciones predeterminado que proporciona Azure al crear la red virtual que se empareja con la suya.  El intervalo suele ser similar a 10.x.0.0/16.  Esto puede dar lugar a la superposición del intervalo de direcciones IP, por lo que debe asegurarse de especificar dicho intervalo en la configuración del laboratorio, o bien comprobar el intervalo de direcciones de la red virtual que se va a emparejar.

> [!NOTE]
> Se puede producir un error en la creación del laboratorio si la cuenta de laboratorio está emparejada a una red virtual, pero tiene un intervalo de direcciones IP demasiado estrecho. Puede quedarse sin espacio en el intervalo de direcciones si hay demasiados laboratorios en la cuenta de laboratorio (cada laboratorio usa 512 direcciones). 
> 
> Si se produce un error en la creación del laboratorio, póngase en contacto con el administrador o el propietario de la cuenta de laboratorio y solicite que se aumente el intervalo de direcciones. El administrador puede aumentar el intervalo de direcciones mediante los pasos mencionados en la sección [Especificación de un intervalo de direcciones para las VM de la cuenta de laboratorio](#specify-an-address-range-for-vms-in-the-lab-account). 

## <a name="configure-after-the-lab-account-is-created"></a>Configuración después de crear la cuenta de laboratorio

La misma propiedad se puede habilitar desde la pestaña **Configuración de laboratorios** de la página **Cuenta de laboratorio** si no configuró una red del mismo nivel cuando creó la cuenta de laboratorio. El cambio realizado en esta configuración tendrá efecto únicamente en los laboratorios creados después de dicho cambio. Como puede ver en la imagen, la opción **Asociar red virtual** se puede habilitar o deshabilitar en los laboratorios de la cuenta de laboratorio.

![Habilitar o deshabilitar la asociación de red virtual después de crear el laboratorio](./media/how-to-connect-peer-virtual-network/select-vnet-to-peer-existing-lab.png)

Al seleccionar una red virtual en el campo **Asociar red virtual**, se deshabilita la opción **Permitir al creador del laboratorio seleccionar la ubicación correspondiente**. El motivo es que los laboratorios de la cuenta de laboratorio deben estar en la misma región que dicha cuenta para que se puedan conectar con los recursos de la red virtual emparejada.

> [!IMPORTANT]
> La configuración de la red virtual emparejada solo se aplica a los laboratorios creados después de realizar el cambio, no a los laboratorios que ya existían.


## <a name="specify-an-address-range-for-vms-in-the-lab-account"></a>Especificación de un intervalo de direcciones para las VM de la cuenta de laboratorio
El procedimiento siguiente tiene pasos para especificar un intervalo de direcciones para las máquinas virtuales del laboratorio. Si actualiza el intervalo que ha especificado antes, el intervalo de direcciones modificado se aplica solo a las máquinas virtuales que se crean después de realizar el cambio. 

Estas son algunas restricciones al especificar el intervalo de direcciones que debe tener en cuenta. 

- El prefijo debe ser menor o igual a 23. 
- Si una red virtual se empareja con la cuenta de laboratorio, el intervalo de direcciones proporcionado no puede coincidir con el de la red virtual emparejada.

1. En la página **Cuenta de laboratorio**, seleccione **Configuración del laboratorio** en el menú de la izquierda.
2. Para el campo **Intervalo de direcciones**, especifique el intervalo de direcciones para las máquinas virtuales que se van a crear en el laboratorio. Dicho rango debe estar en la notación de enrutamiento de interdominios sin clases (CIDR) (por ejemplo: 10.20.0.0/23). Las máquinas virtuales del laboratorio se crearán en este rango de direcciones.
3. Seleccione **Guardar** en la barra de herramientas. 

    ![Configuración del intervalo de direcciones](./media/how-to-manage-lab-accounts/labs-configuration-page-address-range.png)

## <a name="next-steps"></a>Pasos siguientes

Vea los artículos siguientes:

- [Permiso para que el creador del laboratorio seleccione su ubicación](allow-lab-creator-pick-lab-location.md)
- [Asociación o desasociación de una galería de imágenes compartidas en Azure Lab Services](how-to-attach-detach-shared-image-gallery.md)
- [Cómo agregar un usuario como propietario de un laboratorio de clase en Azure Lab Services](how-to-add-user-lab-owner.md)
- [Configuración del firewall para Azure Lab Services](how-to-configure-firewall-settings.md)
- [Configuración de cuentas de laboratorio en Azure Lab Services](how-to-configure-lab-accounts.md)