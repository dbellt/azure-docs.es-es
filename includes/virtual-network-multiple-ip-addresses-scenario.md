---
author: genlin
ms.service: virtual-network
ms.topic: include
ms.date: 11/09/2018
ms.author: genli
ms.openlocfilehash: 3df4108907a4e1e65a444faf1049163966b7accf
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186237"
---
## <a name="scenario"></a>Escenario
Se crea una máquina virtual con una sola NIC y se conecta a una red virtual. La máquina virtual requiere tres direcciones IP *privadas* y dos direcciones IP *públicas*, todas diferentes. Las direcciones IP se asignan a las siguientes configuraciones de IP:

* **IPConfig-1:** asigna un dirección IP privada *estática* y una dirección IP pública *estática*.
* **IPConfig-2:** asigna un dirección IP privada *estática* y una dirección IP pública *estática*.
* **IPConfig-3:** asigna un dirección IP privada *estática* y ninguna dirección IP pública.
  
    ![Varias direcciones IP](./media/virtual-network-multiple-ip-addresses-scenario/multiple-ipconfigs.png)

Las configuraciones de IP se asocian a la NIC cuando esta se crea, y la NIC se conecta a la máquina virtual cuando esta se crea. Los tipos de direcciones IP usados para el escenario tienen fines ilustrativos. Puede asignar cualquier dirección IP y tipo de asignación que necesite.

> [!NOTE]
> Aunque los pasos de este artículo asignan todas las configuraciones de IP a una NIC única, también puede asignar varias configuraciones de IP a cualquiera de las NIC de una máquina virtual con varias NIC. Para aprender a crear una VM con varias NIC, lea el artículo [Implementación de máquinas virtuales con varias NIC mediante PowerShell](../articles/virtual-machines/windows/multiple-nics.md).