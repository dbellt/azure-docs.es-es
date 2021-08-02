---
title: Uso de máquinas virtuales y grupos de seguridad de red en Azure Bastion
description: Aprenda a usar grupos de seguridad de red con Azure Bastion.
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: conceptual
ms.date: 12/09/2020
ms.author: cherylmc
ms.openlocfilehash: 84f32755a4838fbcb29b3d85d8308b5288d746ea
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110537911"
---
# <a name="working-with-nsg-access-and-azure-bastion"></a>Trabajo con acceso a grupos de seguridad de red y Azure Bastion

Al trabajar con Azure Bastion, puede usar grupos de seguridad de red (NSG). Para más información, consulte [Grupos de seguridad](../virtual-network/network-security-groups-overview.md).

:::image type="content" source="./media/bastion-nsg/figure-1.png" alt-text="NSG":::

En este diagrama:

* El host de Bastion se implementa en la red virtual.
* El usuario se conecta a Azure Portal con cualquier explorador HTML5.
* El usuario va a la máquina virtual de Azure para RDP/SSH.
* Integración de conexión: sesión de RDP/SSH con un solo clic dentro del explorador
* No se requiere ninguna dirección IP pública en la máquina virtual de Azure.

## <a name="network-security-groups"></a><a name="nsg"></a>Grupos de seguridad de red

En esta sección se muestra el tráfico de red entre el usuario y Azure Bastion, y para dirigirse a las máquinas virtuales de la red virtual:

> [!IMPORTANT]
> Si decide usar un grupo de seguridad de red con el recurso de Azure Bastion, **tendrá** que crear todas las reglas de tráfico de entrada y salida siguientes. Si se omite cualquiera de las reglas siguientes en el grupo de seguridad de red, el recurso de Azure Bastion no recibirá las actualizaciones necesarias en el futuro y, por tanto, estará abierto a posibles vulnerabilidades de seguridad.
> 

### <a name="azurebastionsubnet"></a><a name="apply"></a>AzureBastionSubnet

Azure Bastion se implementa específicamente en ***AzureBastionSubnet***.

* **Tráfico de entrada:**

   * **Tráfico de entrada procedente de Internet pública:** Azure Bastion creará una dirección IP pública que necesita el puerto 443 habilitado en la dirección IP pública para el tráfico de entrada. El puerto 3389/22 no tiene que abrirse en la subred AzureBastionSubnet.
   * **Plano de control del tráfico de entrada procedente de Azure Bastion:** Para la conectividad del plano de control, habilite el puerto 443 entrante desde la etiqueta de servicio de **GatewayManager**. De este modo, se permite que el plano de control, es decir, el administrador de puerta de enlace, pueda comunicarse con Azure Bastion.
   * **Tráfico de entrada desde el plano de datos de Azure Bastion:** Para la comunicación del plano de datos entre los componentes subyacentes de Azure Bastion, habilite los puertos 8080 y 5701 de entrada desde la etiqueta de servicio **VirtualNetwork** a la etiqueta de servicio **VirtualNetwork**. Esto permite que los componentes de Azure Bastion se comuniquen entre sí.
   * **Tráfico de entrada desde Azure Load Balancer:** en el caso de los sondeos de estado, habilite el puerto 443 de entrada desde la etiqueta de servicio **AzureLoadBalancer**. Esto permite que Azure Load Balancer detecte la conectividad.


   :::image type="content" source="./media/bastion-nsg/inbound.png" alt-text="Captura de pantalla que muestra las reglas de seguridad de entrada para la conectividad de Azure Bastion.":::

* **Tráfico de salida:**

   * **Tráfico de salida a máquinas virtuales de destino:** Azure Bastion se comunicará con las máquinas virtuales de destino a través de la dirección IP privada. Los grupos de seguridad de red tienen que permitir el tráfico de salida a otras subredes de máquinas virtuales de destino para el puerto 3389 y 22.
   * **Tráfico de salida al plano de datos de Azure Bastion:** Para la comunicación del plano de datos entre los componentes subyacentes de Azure Bastion, habilite los puertos 8080 y 5701 de salida desde la etiqueta de servicio **VirtualNetwork** a la etiqueta de servicio **VirtualNetwork**. Esto permite que los componentes de Azure Bastion se comuniquen entre sí.
   * **Salida del tráfico a otros puntos de conexión públicos de Azure:** Azure Bastion debe ser capaz de conectarse a varios puntos de conexión públicos dentro de Azure (por ejemplo, para almacenar registros de diagnóstico y los registros de medición). Por esta razón, Azure Bastion necesita una salida hacia 443 para la etiqueta de servicio **AzureCloud**.
   * **Tráfico de salida a Internet:** Azure Bastion debe ser capaz de comunicarse con Internet para la validación de la sesión y el certificado. Por esta razón, se recomienda habilitar el puerto 80 de salida a **Internet.**


   :::image type="content" source="./media/bastion-nsg/outbound.png" alt-text="Captura de pantalla que muestra las reglas de seguridad de salida para la conectividad de Azure Bastion.":::

### <a name="target-vm-subnet"></a>Subred de máquina virtual de destino
Se trata de una subred que contiene la máquina virtual de destino a la que quiere conectarse mediante RDP/SSH.

   * **Tráfico de entrada procedente de Azure Bastion:** Azure Bastion se comunicará con la máquina virtual de destino a través de la dirección IP privada. Los puertos RDP/SSH (puertos 3389/22, respectivamente) tienen que abrirse en la máquina virtual de destino a través de la dirección IP privada. Como procedimiento recomendado, puede agregar el intervalo de direcciones IP de la subred de Azure Bastion en esta regla para permitir que solo Bastion pueda abrir estos puertos en las máquinas virtuales de destino de la subred de la máquina virtual de destino.


## <a name="next-steps"></a>Pasos siguientes

Para más información sobre Azure Bastion, consulte las [preguntas frecuentes](bastion-faq.md).
