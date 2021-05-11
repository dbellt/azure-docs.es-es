---
title: Administración de zonas DNS en DNS de Azure - Azure Portal | Microsoft Docs
description: Puede administrar zonas DNS con Azure Portal. Este artículo describe cómo actualizar, eliminar y crear zonas DNS en Azure DNS.
services: dns
documentationcenter: na
author: rohinkoul
ms.service: dns
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 04/28/2021
ms.author: rohink
ms.openlocfilehash: 58118480c16f2e318bab7435a79e27629880acc5
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108203233"
---
# <a name="how-to-manage-dns-zones-in-the-azure-portal"></a>Administración de zonas DNS en Azure Portal

> [!div class="op_single_selector"]
> * [Portal](dns-operations-dnszones-portal.md)
> * [PowerShell](dns-operations-dnszones.md)
> * [CLI de Azure clásica](./dns-operations-dnszones-cli.md)
> * [CLI de Azure](dns-operations-dnszones-cli.md)

En este artículo se muestra cómo administrar sus zonas DNS mediante Azure Portal. También se pueden administrar las zonas DNS mediante la [CLI de Azure](dns-operations-dnszones-cli.md) multiplataforma o Azure [PowerShell](dns-operations-dnszones.md).

## <a name="create-a-dns-zone"></a>Creación de una zona DNS

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. En la parte superior izquierda de la pantalla, seleccione **Crear un recurso** y busque **Zona DNS**. Seleccione **Crear**.

    :::image type="content" source="./media/dns-operations-dnszones-portal/search-dns-zone.png" alt-text="Captura de pantalla de la búsqueda para crear un recurso para la zona DNS.":::

1. En la página **Crear zona DNS**, escriba los siguientes valores y, luego, seleccione **Crear**.

    | Configuración | Detalles |
    | --- | --- |
    | **Suscripción** | Seleccione la suscripción en la que se creará la zona DNS.|
    | **Grupos de recursos** | Seleccione o cree un grupo de recursos. Para más información sobre los grupos de recursos, lea el artículo [Información general de Azure Resource Manager](../azure-resource-manager/management/overview.md?toc=%2fazure%2fdns%2ftoc.json#resource-groups).|
    | **Nombre** | Escriba un nombre para la zona DNS. Por ejemplo: **contoso.com**. |
    | **Ubicación** | Seleccione la ubicación del grupo de recursos. La ubicación ya estará seleccionada si usa un grupo de recursos creado anteriormente.  |

> [!NOTE]
> El grupo de recursos se refiere a la ubicación del grupo de recursos y no tiene efecto alguno sobre la zona DNS. La ubicación de la zona DNS siempre es "global" y no se muestra.

## <a name="list-dns-zones"></a>Enumeración de zonas DNS

En la búsqueda de recursos de la parte superior de Azure Portal, busque **Zonas DNS**. Cada zona DNS es su propio recurso. La información como el número de conjuntos de registros y los servidores de nombres se puede ver en esta página. La columna **Servidores de nombres** no está en la vista predeterminada. Para agregarla, seleccione **Vista administrada > Editar columnas > + Agregar columna** y, en la lista desplegable, seleccione **Servidores de nombres**. Seleccione **Guardar** para aplicar la nueva columna.

:::image type="content" source="./media/dns-operations-dnszones-portal/list-zones.png" alt-text="Captura de pantalla de la página de lista Zona DNS.":::

## <a name="delete-a-dns-zone"></a>Eliminar una zona DNS

Navegue a una zona DNS en el portal. En la página de información general de la **Zona DNS** seleccionada, seleccione **Eliminar zona**. A continuación, se le pedirá que confirme que quiere eliminar la zona DNS. Al eliminar una zona DNS, también se eliminan todos los registros contenidos en la zona.

:::image type="content" source="./media/dns-operations-dnszones-portal/delete-zone.png" alt-text="Captura de pantalla del botón Eliminar zona DNS en la página de información general.":::

## <a name="next-steps"></a>Pasos siguientes

Aprenda a trabajar con la zona y registros DNS en [Introducción a DNS de Azure con Azure Portal](dns-getstarted-portal.md).
