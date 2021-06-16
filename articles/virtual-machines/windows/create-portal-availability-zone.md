---
title: Creación de una máquina virtual con zona con Azure Portal
description: Creación de una máquina virtual en una zona de disponibilidad con Azure Portal
documentationcenter: virtual-machines
author: mimckitt
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 5/10/2021
ms.author: mimckitt
ms.reviewer: cynthn
ms.custom: ''
ms.openlocfilehash: f1c37751e8f633c6d7dfab88b9dbe524626483a8
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112079182"
---
# <a name="create-a-virtual-machine-in-an-availability-zone-using-the-azure-portal"></a>Creación de una máquina virtual en una zona de disponibilidad con Azure Portal

En este artículo se describe el uso de Azure Portal para crear una máquina virtual en una zona de disponibilidad de Azure. Una [zona de disponibilidad](../../availability-zones/az-overview.md) es una zona separada físicamente en una región de Azure. Use zonas de disponibilidad para proteger sus datos y aplicaciones de la improbable pérdida o error de todo un centro de datos.

Para usar una zona de disponibilidad, cree la máquina virtual [en una región de Azure compatible](../../availability-zones/az-region.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure 

1. Inicie sesión en Azure Portal en https://portal.azure.com.

1. Haga clic en **Crear un recurso** > **Proceso** > **Máquina virtual**. 

3. Escriba la información de la máquina virtual. El nombre de usuario y la contraseña se usarán para iniciar sesión en la máquina virtual. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](faq.yml#what-are-the-password-requirements-when-creating-a-vm-). 

4. Elija una ubicación que admita zonas de disponibilidad, como Este de EE. UU. 2. 

5. En **Opciones de disponibilidad**, seleccione la lista desplegable **Zona de disponibilidad**. 

1. En **Zona de disponibilidad**, seleccione una zona en la lista desplegable.
        
4. Elija un tamaño para la máquina virtual. Seleccione un tamaño recomendado o filtre en función de las características. Confirme que el tamaño esté disponible en la zona que desea usar.

6. Termine de rellenar la información de la máquina virtual. Cuando haya terminado, seleccione **Revisar y crear**.

7. Una vez comprobada la información, seleccione **Crear**.

1. Una vez creada la máquina virtual, puede ver la zona de disponibilidad en la **sección Essentials** de la página de la máquina virtual.

## <a name="confirm-zone-for-managed-disk-and-ip-address"></a>Confirmación de zona para disco administrado y dirección IP

Cuando la máquina virtual se implementa en una zona de disponibilidad, se crea un disco administrado para la máquina virtual en la misma zona de disponibilidad. De forma predeterminada, también se crea una dirección IP pública en dicha zona.

Puede confirmar la configuración de zona para estos recursos en el portal.  

1. Seleccione **Discos** en el menú de la izquierda y, a continuación, seleccione el disco del sistema operativo. La página del disco incluye detalles sobre la ubicación y la zona de disponibilidad del disco.

1. De nuevo en la página de la máquina virtual, seleccione la dirección IP pública. En el menú de la izquierda, seleccione **Propiedades**. La página de propiedades incluye los detalles sobre la ubicación y la zona de disponibilidad de la dirección IP pública.

    
## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a crear una máquina virtual en una zona de disponibilidad. Aprenda más sobre la [disponibilidad](../availability.md) de las máquinas virtuales de Azure.
