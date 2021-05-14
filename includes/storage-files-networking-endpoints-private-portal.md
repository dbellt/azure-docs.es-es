---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 04/15/2021
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: 958c40ed311ebe8b2a83ce0d4d394d00f48d1afa
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107732510"
---
Vaya a la cuenta de almacenamiento para la que desea crear un punto de conexión privado. En la tabla de contenido de la cuenta de almacenamiento, seleccione **Redes**, **Private endpoint connections** (Conexiones de punto de conexión privado) y, por último, **+ Private endpoint** (+ Punto de conexión privado) para crear un punto de conexión privado. 

[![Captura de pantalla del elemento de conexiones del punto de conexión privado en la tabla de contenido de la cuenta de almacenamiento](media/storage-files-networking-endpoints-private-portal/create-private-endpoint-0.png)](media/storage-files-networking-endpoints-private-portal/create-private-endpoint-0.png#lightbox)

El asistente resultante tiene varias páginas que debe completar:

En la hoja **Aspectos básicos**, seleccione el grupo de recursos, el nombre y la región que quiera para el punto de conexión privado. Estos pueden ser cualquier cosa que desee y no tienen que coincidir con la cuenta de almacenamiento en ningún aspecto, aunque debe crear el punto de conexión privado en la misma región que la red virtual en la que quiere crear el punto de conexión privado.

![Una captura de pantalla de la sección Aspectos básicos de la sección Crear punto de conexión privado](media/storage-files-networking-endpoints-private-portal/create-private-endpoint-1.png)

En la hoja **Recurso**, seleccione el botón de radio **Connect to an Azure resource in my directory** (Conectarse a un recurso de Azure en mi directorio). En **Tipo de recurso**, seleccione **Microsoft.Storage/storageAccounts** para el tipo de recurso. El campo **Recurso** es la cuenta de almacenamiento con el recurso compartido de archivos de Azure al que quiere conectarse. El recurso secundario de destino es **archivo**, ya que es para Azure Files.

La hoja **Configuración** permite seleccionar la red virtual y la subred específicas a las que le gustaría agregar el punto de conexión privado. Debe seleccionar una subred distinta de la subred en la que agregó el punto de conexión de servicio anteriormente. La hoja Configuración también contiene la información para crear o actualizar la zona DNS privada. Se recomienda usar la zona `privatelink.file.core.windows.net` predeterminada.

![Captura de pantalla de la sección Configuración](media/storage-files-networking-endpoints-private-portal/create-private-endpoint-2.png)

Haga clic en **Revisar y crear** para crear el punto de conexión privado. 
