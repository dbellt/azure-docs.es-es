---
title: Creación de un nombre de dominio completo para una VM en Azure Portal
description: Aprenda a crear un nombre de dominio completo para una máquina virtual en Azure Portal.
author: cynthn
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 05/07/2021
ms.author: cynthn
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c63bb64cac0642c0472862a9c3272b9939f39bb9
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109732621"
---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal-for-a-linux-vm"></a>Creación de un nombre de dominio completo en Azure Portal para una máquina virtual Linux

Cuando crea una máquina virtual (VM) en [Azure Portal](https://portal.azure.com), se crea automáticamente un recurso de IP pública para la máquina virtual. Use esta dirección IP pública para acceder de forma remota a la máquina virtual. Aunque el portal no crea [un nombre de dominio completo](https://en.wikipedia.org/wiki/Fully_qualified_domain_name), o FQDN, puede agregar uno cuando se crea la máquina virtual. Este artículo muestra los pasos para crear un nombre DNS o FQDN. Si crea una máquina virtual sin una dirección IP pública, no puede crear un FQDN.

## <a name="create-a-fqdn"></a>Creación de un FQDN
En este artículo se supone que ya ha creado una máquina virtual. Si es necesario, puede crear una máquina virtual [Linux](./linux/quick-create-portal.md) o [Windows](./windows/quick-create-portal.md) en el portal. Una vez que la máquina virtual está en funcionamiento, siga estos pasos:


1. Seleccione la máquina virtual en el portal. 
1. En el menú de la izquierda, seleccione **Propiedades**.
1. En **Dirección IP pública\Etiqueta de nombre DNS**, seleccione su dirección IP.
2. En **Etiqueta de nombre de DNS**, escriba el prefijo que desea usar.
3. En la parte superior de la página, seleccione **Guardar**.
4. Seleccione **Información general** en el menú de la izquierda para volver a la hoja de información general de la máquina virtual.
5. Compruebe que el **nombre DNS** se muestra correctamente. 

## <a name="next-steps"></a>Pasos siguientes

También puede administrar DNS con [zonas de Azure DNS](../dns/dns-getstarted-portal.md).

