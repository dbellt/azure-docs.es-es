---
title: Creación y uso de un dominio personalizado
description: Conecte un dominio personalizado a una máquina virtual en Azure.
author: mimckitt
ms.service: virtual-machines
ms.subservice: networking
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 04/01/2021
ms.author: jamesser
ms.reviewer: cynthn
ms.openlocfilehash: c4797420904b6dc03550f658c2aa950a4de99c9c
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107250956"
---
# <a name="add-custom-domain-to-azure-vm-or-resource"></a>Incorporación de un dominio personalizado a una máquina virtual o un recurso de Azure

En Azure hay varias maneras de conectar un dominio personalizado a una máquina virtual o un recurso. Para cualquier recurso con una IP pública (máquina virtual, equilibrador de carga o puerta de enlace de aplicación), la manera más sencilla es crear un conjunto de registros A en el registrador de dominios correspondiente. 

## <a name="prerequisites"></a>Requisitos previos 
- Necesita una máquina virtual con un servidor web en ejecución. Puede usar la [guía de inicio rápido](./linux/quick-create-cli.md) para crear una máquina virtual y agregar NGINX.

- La máquina virtual debe ser accesible desde la Web (puerto abierto 80 o 443). Para realizar una implementación más segura, coloque primero la máquina virtual detrás de un equilibrador de carga o de una puerta de enlace de aplicación. Para obtener más información, consulte [Inicio rápido: Equilibrador de carga](https://docs.microsoft.com/azure/load-balancer/quickstart-load-balancer-standard-public-portal?tabs=option-1-create-load-balancer-standard).

- Debe tener un dominio existente y acceso a la configuración de DNS. Para obtener más información, consulte [Compra de un nombre de dominio personalizado para Azure App Service](../app-service/manage-custom-dns-buy-domain.md).


## <a name="add-custom-domain-to-vm-public-ip-address"></a>Incorporación de un dominio personalizado a la dirección IP pública de una máquina virtual

Cuando se crea una máquina virtual en Azure Portal, se crea automáticamente un recurso de IP pública para dicha máquina. Su dirección IP pública se muestra en la página de información general de la máquina virtual. 
 
:::image type="content" source="media/custom-domain/essentials.png" alt-text="Se muestra la dirección IP pública en la sección “Essentials” de la página de información general de la máquina virtual.":::

Si selecciona la dirección IP, puede ver más información sobre ella. Asegúrese de que **Asignación de IP** esté establecida en **Estática**. Una dirección IP estática no cambia si la máquina virtual o el recurso se reinician o se apagan.

:::image type="content" source="media/custom-domain/ip-config.png" alt-text="Se muestra la configuración de IP pública para que pueda ver si la dirección IP es estática.":::

Si la dirección IP no es estática, deberá crear un nombre de dominio completo. 

1. Seleccione la máquina virtual en el portal. 
1. En el menú de la izquierda, seleccione **Propiedades**.
1. En **Dirección IP pública\Etiqueta de nombre DNS**, seleccione su dirección IP.
2. En **Etiqueta de nombre de DNS**, escriba el prefijo que desea usar.
3. En la parte superior de la página, seleccione **Guardar**.
4. Seleccione **Información general** en el menú de la izquierda para volver a la hoja de información general de la máquina virtual.
5. Compruebe que el *nombre DNS* se muestra correctamente. 

Abra un explorador, escriba su dirección IP o su nombre de dominio completo y compruebe que aparece el contenido web que se ejecuta en la máquina virtual.
 
Una vez que compruebe su IP estática o su nombre de dominio completo, vaya al proveedor del dominio y acceda a la configuración de DNS.

Una vez allí, agregue un *registro A* que apunte a su dirección IP pública o a su nombre de dominio completo. Por ejemplo, este es el procedimiento para el registrador de dominios GoDaddy:
1. Inicie sesión y seleccione el dominio personalizado que desea usar.
2. En la sección **Domains** (Dominios), seleccione **Manage All** (Administrar todo) y, después, seleccione **DNS | Manage Zones** (DNS | Administrar zonas).
3. En **Domain Name** (Nombre de dominio), escriba el dominio personalizado y seleccione **Search** (Buscar).
4. En la página “DNS Management” (Administración de DNS), seleccione “Add” (Agregar); después, seleccione “A” en la lista “Type” (Tipo).
5. Rellene los campos de la entrada de “A”:
    - “Type” (Tipo): deje **A** seleccionado.
    - “Host”: escriba **@** .
    - “Points to” (Apunta a): escriba la dirección IP pública o el nombre de dominio completo de su máquina virtual. 
    - “TTL”: deje seleccionada la opción de una hora.
6. Seleccione **Guardar**.

La entrada del registro A se agrega a la tabla de registros de DNS.
 
Una vez creado el registro, el DNS suele tardar aproximadamente una hora en propagarse, pero a veces puede tardar hasta 48. 


 
## <a name="next-steps"></a>Pasos siguientes
[Introducción a la terminación TLS y a TLS de extremo a extremo con Application Gateway](../application-gateway/ssl-overview.md).

 
