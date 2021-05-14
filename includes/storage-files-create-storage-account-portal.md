---
title: storage-files-create-storage-account-portal
description: Creación de una cuenta de almacenamiento en Azure Files.
services: storage
author: wmgries
ms.service: storage
ms.topic: include
ms.date: 04/15/2021
ms.author: wgries
ms.custom: include file
ms.openlocfilehash: 33159ab6dd014a153c8fd317fd291aeca033d6e9
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107717876"
---
Una cuenta de almacenamiento es un grupo compartido de almacenamiento en el que puede implementar un recurso compartido de archivos de Azure u otros recursos de almacenamiento como blobs o colas. Una cuenta de almacenamiento puede contener un número ilimitado de recursos compartidos. Un recurso compartido puede almacenar un número ilimitado de archivos, hasta los límites de capacidad de la cuenta de almacenamiento.

Para crear una cuenta de almacenamiento:

1. En el menú de la izquierda, seleccione **+** para crear un recurso.
1. Seleccione **Cuenta de almacenamiento** para crear una cuenta de almacenamiento.

    :::image type="content" source="../articles/storage/files/media/storage-how-to-use-files-portal/create-storage-account-1.png" alt-text="Captura de pantalla de la opción de cuenta de almacenamiento en la hoja Crear un recurso." lightbox="../articles/storage/files/media/storage-how-to-use-files-portal/create-storage-account-1.png":::

1. En **Nombre**, escriba *mystorageacct* seguido de algunos números aleatorios, hasta que aparezca una marca de verificación verde que indica que es un nombre único. Un nombre de cuenta de almacenamiento debe estar en minúsculas y ser único globalmente. Anote el nombre de la cuenta de almacenamiento. Lo usará más adelante. 
1. En **Rendimiento**, conserve el valor predeterminado de **Estándar**.
1. En **Replicación**, seleccione **Almacenamiento con redundancia local (LRS)** .
1. En **Suscripción**, seleccione la suscripción que se usó para crear la cuenta de almacenamiento. Si solo tiene una suscripción, debe ser la predeterminada.
1. En **Grupo de recursos**, seleccione **Crear nuevo**. Como nombre, escriba *myResourceGroup*.
1. En **Ubicación**: seleccione **Este de EE. UU**.
1. Cuando termine, haga clic en **Crear** para iniciar la implementación.