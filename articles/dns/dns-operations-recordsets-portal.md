---
title: Administración de conjuntos de registros de DNS y de registros con Azure DNS
description: Azure DNS proporciona la funcionalidad de administrar registros y conjuntos de registros de DNS al hospedar un dominio.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: how-to
ms.date: 04/28/2021
ms.author: rohink
ms.openlocfilehash: 8ac76671dc16fb51cea35154cd7862ad1dee66f6
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108227027"
---
# <a name="manage-dns-records-and-record-sets-by-using-the-azure-portal"></a>Administración de registros y conjuntos de registros DNS mediante el Portal de Azure

En este artículo se explica cómo administrar conjuntos de registros y registros para la zona DNS mediante el Portal de Azure.

Es importante comprender la diferencia entre los conjuntos de registros de DNS y los registros DNS individuales. Un conjunto de registros es una colección de registros de una zona con el mismo nombre y el mismo tipo. Para más información, consulte [Creación de registros y conjuntos de registros de DNS mediante el Portal de Azure](./dns-getstarted-portal.md).

## <a name="create-a-new-record-set-and-record"></a>Creación de un nuevo conjunto de registros y un registro

Para crear un conjunto de registros en el Portal de Azure, consulte [Creación de registros y conjuntos de registros de DNS mediante el Portal de Azure](./dns-getstarted-portal.md).

## <a name="view-a-record-set"></a>Visualización de un conjunto de registros

1. En Azure Portal, vaya a la página de información general **Zona DNS**.

1. Busque el conjunto de registros y selecciónelo para abrir sus propiedades.

    :::image type="content" source="./media/dns-operations-recordsets-portal/overview.png" alt-text="Captura de pantalla de la página de información general de la zona contosotest.com.":::

## <a name="add-a-new-record-to-a-record-set"></a>Incorporación de un nuevo registro a un conjunto de registros

Puede agregar hasta 20 registros a cualquier conjunto de registros. Un conjunto de registros no puede contener dos registros idénticos. Se pueden crear conjuntos de registros vacíos (sin ningún registro), pero no aparecen en los servidores de nombres de Azure DNS. Los conjuntos de registros de tipo CNAME pueden contener, como máximo, un registro.

1. En la página **Record set properties** (Propiedades del conjunto de registros) de la zona DNS, seleccione el conjunto de registros al que quiere agregar un registro.

    :::image type="content" source="./media/dns-operations-recordsets-portal/select-record.png" alt-text="Captura de pantalla de la selección del conjunto de registros www.":::

1. Especifique las propiedades del conjunto de registros rellenando los campos.

    :::image type="content" source="./media/dns-operations-recordsets-portal/record-page.png" alt-text="Captura de pantalla de la página Agregar un registro.":::

1. Para guardar la configuración, seleccione **Guardar** en la parte superior de la página. A continuación, cierre la página.

Cuando se haya guardado el registro, los valores de la página **Zona DNS** reflejarán el nuevo registro.

## <a name="update-a-record"></a>Actualización de un registro

Cuando se actualiza un registro en un conjunto de registros existente, los campos que puede actualizar dependen del tipo de registro con el que está trabajando.

1. En la página **Record set properties** (Propiedades del conjunto de registros) correspondiente a su conjunto de registros, busque el registro.

1. Modifique el registro. Cuando se modifica un registro, puede cambiar la configuración disponible para dicho registro. En el ejemplo siguiente, se seleccionó el campo **Dirección IP** y la dirección IP se encuentra en proceso de modificación.

    :::image type="content" source="./media/dns-operations-recordsets-portal/update-record-page.png" alt-text="Captura de pantalla de la página Actualizar un registro.":::

1. Para guardar la configuración, seleccione **Guardar** en la parte superior de la página. En la esquina superior derecha verá la notificación de que el registro se ha guardado.

    :::image type="content" source="./media/dns-operations-recordsets-portal/record-saved.png" alt-text="Captura de pantalla de un registro guardado correctamente.":::

Cuando se haya guardado el registro, los valores del conjunto de registros de la página **Zona DNS** reflejarán el registro actualizado.

## <a name="remove-a-record-from-a-record-set"></a>Eliminación de un registro de un conjunto de registros

Puede usar el Portal de Azure para quitar registros de un conjunto de registros. Al eliminar el último registro de un conjunto de registros, no se elimina el conjunto de registros.

1. En la página **Record set properties** (Propiedades del conjunto de registros) correspondiente a su conjunto de registros, busque el registro.

1. Seleccione los puntos suspensivos ( **...** ) junto al registro y, luego, elija **Quitar** para eliminar el registro del conjunto de registros.

    :::image type="content" source="./media/dns-operations-recordsets-portal/delete-record.png" alt-text="Captura de pantalla de cómo eliminar un registro.":::

1. Para guardar la configuración, seleccione **Guardar** en la parte superior de la página.

1. Cuando el registro se haya eliminado, los valores del registro de la página **Zona DNS** reflejarán la eliminación.

## <a name="delete-a-record-set"></a><a name="delete"></a>Eliminación de un conjunto de registros

1. En la página **Record set properties** (Propiedades del conjunto de registros) correspondiente a su conjunto de registros, seleccione **Eliminar**.

    :::image type="content" source="./media/dns-operations-recordsets-portal/delete-record-set.png" alt-text="Captura de pantalla de cómo eliminar un conjunto de registros.":::

1. Aparece un mensaje en el que se pregunta si desea eliminar el conjunto de registros.

1. Compruebe que el nombre coincida con el conjunto de registros que quiere eliminar y, luego, haga clic en **Sí**.

1. En la página **Zona DNS**, compruebe que el conjunto de registros ya no está visible.

## <a name="work-with-ns-and-soa-records"></a>Trabajo con registros NS y SOA

Los registros NS y SOA creados automáticamente se administran de forma diferente de otros tipos de registros.

### <a name="modify-soa-records"></a>Modificación de registros SOA

No puede agregar ni eliminar registros del conjunto de registros SOA creado automáticamente en el vértice de zona (nombre = "\@"). Sin embargo, puede modificar cualquiera de los parámetros del registro SOA, excepto "Host" y del conjunto de registros TTL.

### <a name="modify-ns-records-at-the-zone-apex"></a>Modificación de los registros NS en el vértice de zona

El conjunto de registros NS en el vértice de zona se crea automáticamente con cada zona DNS. Este conjunto de registros contiene los nombres de los servidores de nombres de Azure DNS asignados a la zona.

Puede agregar más servidores de nombres a este conjunto de registros NS, para admitir dominios de hospedaje conjunto con más de un proveedor DNS. También puede modificar el TTL y los metadatos de este conjunto de registros. Sin embargo, no puede quitar ni modificar los servidores de nombres de Azure DNS rellenados previamente.

Esta restricción solo se aplica al conjunto de registros NS en el vértice de zona. Otros conjuntos de registros NS de su zona (como los que se usan para delegar zonas secundarias) se pueden modificar sin restricciones.

### <a name="delete-soa-or-ns-record-sets"></a>Eliminación de conjuntos de registros SOA o NS

No puede eliminar los conjuntos de registros SOA ni NS en el vértice de zona (nombre = "\@") que se crean automáticamente cuando se crea la zona. Se eliminan automáticamente al eliminar la zona.

## <a name="next-steps"></a>Pasos siguientes

* Para más información acerca de DNS de Azure, consulte la [Introducción a DNS de Azure](dns-overview.md).
* Para más información acerca de la automatización de DNS, consulte [Creación de conjuntos de registros y zonas DNS con el SDK de .NET](dns-sdk.md).
* Para más información acerca de los registros de DNS inversos, consulte [Introducción a DNS inverso y compatibilidad en Azure](dns-reverse-dns-overview.md).
* Para más información acerca de Azure DNS, consulte [Introducción a los registros de alias de Azure DNS](dns-alias.md).