---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 04/15/2021
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 5798ad8721d8bf2924aa97876d0c8354681d3568
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107718283"
---
Para crear una cuenta de almacenamiento de uso general v2 en Azure Portal, siga estos pasos:

1. En el menú de Azure Portal, seleccione **Todos los servicios**. En la lista de recursos, escriba **Cuentas de almacenamiento**. Cuando comience a escribir, la lista se filtrará en función de la entrada. Seleccione **Cuentas de almacenamiento**.
1. En la ventana **Cuentas de almacenamiento** que aparece, elija **+ Nueva**.
1. En la hoja **Aspectos básicos**, seleccione la suscripción en la que se va a crear la cuenta de almacenamiento.
1. En el campo **Grupo de recursos**, seleccione el grupo de recursos deseado o cree uno nuevo.  Para más información sobre los grupos de recursos de Azure, consulte [Información general de Azure Resource Manager](../articles/azure-resource-manager/management/overview.md).
1. Después, escriba un nombre para la cuenta de almacenamiento. El nombre que elija debe ser único en Azure. El nombre también debe tener una longitud de entre 3 y 24 caracteres, y solo puede contener números y letras minúsculas.
1. Seleccione una región para la cuenta de almacenamiento o utilice la región predeterminada.
1. Seleccione un nivel de rendimiento. El valor predeterminado es *Estándar*.
1. Especifique cómo se replicará la cuenta de almacenamiento. La opción de redundancia predeterminada es *Almacenamiento con redundancia geográfica (GRS)* . Para más información acerca de las opciones de replicación disponibles, consulte [Redundancia de Azure Storage](../articles/storage/common/storage-redundancy.md).
1. Hay otras opciones disponibles en las hojas **Opciones avanzadas**, **Redes**, **Protección de datos** y **Etiquetas**. Para utilizar Azure Data Lake Storage, elija la hoja **Opciones avanzadas** y, después, establezca **Espacio de nombres jerárquico** en **Habilitado**. Para más información, consulte [Introducción a Azure Data Lake Storage Gen2](../articles/storage/blobs/data-lake-storage-introduction.md).
1. Seleccione **Revisar y crear** para revisar la configuración de la cuenta de almacenamiento y crear la cuenta.
1. Seleccione **Crear**.

En la imagen siguiente se muestra la configuración de la hoja **Aspectos básicos** de una nueva cuenta de almacenamiento:

:::image type="content" source="media/storage-create-account-portal-include/account-create-portal.png" alt-text="Captura de pantalla en la que se muestra cómo crear una cuenta de almacenamiento en Azure Portal" lightbox="media/storage-create-account-portal-include/account-create-portal.png":::.
