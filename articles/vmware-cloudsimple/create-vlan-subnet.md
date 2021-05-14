---
title: 'Creación de redes VLAN y subredes: Azure VMware Solution by CloudSimple'
description: 'Azure VMware Solution by CloudSimple: se describe cómo crear y administrar redes VLAN y subredes para las nubes privadas y luego aplicar reglas de firewall.'
author: shortpatti
ms.author: v-patsho
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 86dfc14f92f3afc1632cf6e5c033cf682f2ddc67
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176961"
---
# <a name="create-and-manage-vlanssubnets-for-your-private-clouds"></a>Creación y administración de redes VLAN y subredes para las nubes privadas

Abra la pestaña VLANs/Subnets (Redes VLAN o subredes) en la página Network (Red) para crear y administrar VLAN o subredes para las nubes privadas. Después de crear una VLAN o subred, puede aplicar reglas de firewall.

## <a name="create-a-vlansubnet"></a>Creación de una VLAN o subred

1. [Acceda al portal de CloudSimple](access-cloudsimple-portal.md) y seleccione **Network** (Red) en el menú lateral.
2. Seleccione **VLANs/subnets** (Redes VLAN o subredes).
3. Haga clic en **Create VLAN/Subnet** (Crear VLAN o subred).

    ![Página VLAN/subnet (VLAN o subred)](media/vlan-subnet-page.png)

4. Seleccione la nube privada de la nueva VLAN o subred.
5. Escriba un identificador de VLAN.
6. Escriba el nombre de la subred.
7. Para habilitar el enrutamiento en la VLAN (subred), especifique el intervalo de CIDR de la subred. Asegúrese de que el intervalo de CIDR no se superponga con ninguna de las subredes locales, subredes de Azure o subredes de puerta de enlace.
8. Haga clic en **Enviar**.

    ![Creación de una VLAN o subred](media/create-new-vlan-subnet-details.png)


> [!IMPORTANT]
> Existe una cuota de 30 VLAN por cada nube privada. Estos límites se pueden incrementar [poniéndose en contacto con soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="use-vlan-information-to-set-up-a-distributed-port-group-in-vsphere"></a>Uso de la información de VLAN para configurar un grupo de puertos distribuido en vSphere

Para crear un grupo de puertos distribuido en vSphere, siga las instrucciones del tema de VMware "Agregar un grupo de puertos distribuido" de la <a href="https://docs.vmware.com/en/VMware-vSphere/6.5/vsphere-esxi-vcenter-server-65-networking-guide.pdf" target="_blank">guía de redes de vSphere</a>. Al configurar el grupo de puertos distribuido, proporcione la información de VLAN de la configuración de CloudSimple.

![Grupo de puertos distribuido](media/distributed-port-group.png)

## <a name="select-a-firewall-table"></a>Selección de una tabla de firewall

Las tablas de firewall y las reglas asociadas se definen en la página **Network > Firewall tables** (Red > Tablas de firewall). Para seleccionar la tabla de firewall que se va a aplicar a la VLAN o subred de una nube privada, seleccione la VLAN o subred, haga clic en **Firewall table attachment** (Datos adjuntos de la tabla de firewall) en la página **VLANs/Subnets** (VLAN o subredes). Consulte [Tablas de firewall](firewall.md) para obtener instrucciones sobre cómo configurar tablas de firewall y definir reglas.

![Vínculo de tabla de firewall](media/vlan-subnet-firewall-link.png)

> [!NOTE]
> Una subred puede estar asociada a una tabla de firewall. Una tabla de firewall puede estar asociada a varias subredes.

## <a name="edit-a-vlansubnet"></a>Edición de una VLAN o subred

Para editar la configuración de una VLAN o subred, selecciónela en la página **VLANs/Subnets** (VLAN o subredes) y haga clic en el icono **Edit** (Editar). Realice los cambios y haga clic en **Submit** (Enviar).

## <a name="delete-a-vlansubnet"></a>Eliminación de una VLAN o subred

Para eliminar una VLAN o subred, selecciónela en la página **VLANs/Subnets** (VLAN o subredes) y haga clic en el icono **Delete** (Eliminar). Haga clic en **Delete** (Eliminar) para confirmar.
