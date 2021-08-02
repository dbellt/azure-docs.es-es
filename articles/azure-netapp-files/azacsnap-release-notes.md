---
title: Notas de la versión de la herramienta Instantánea coherente de aplicaciones de Azure para Azure NetApp Files | Microsoft Docs
description: Ofrece notas de la versión a la herramienta Instantánea de Azure Application Consistent que puede usar con Azure NetApp Files.
services: azure-netapp-files
documentationcenter: ''
author: Phil-Jensen
manager: ''
editor: ''
ms.assetid: ''
ms.service: azure-netapp-files
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 05/27/2021
ms.author: phjensen
ms.openlocfilehash: fd8d4e1bfed60aa8f3eae4d4d3c033247ab1268d
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110579912"
---
# <a name="release-notes-for-azure-application-consistent-snapshot-tool"></a>Notas de la versión de la herramienta Azure Application Consistent Snapshot

En esta página se enumeran los cambios importantes realizados en AzAcSnap para proporcionar una nueva funcionalidad o resolver los defectos.

## <a name="may-2021"></a>Mayo de 2021

### <a name="azacsnap-v501-build-2021052414837---patch-update-to-v50"></a>AzAcSnap v5.0.1 (compilación: 20210524.14837); actualización de revisión a v5.0

AzAcSnap v5.0.1 (compilación: 20210524.14837) se proporciona como una actualización de revisión a la rama v5.0 con las siguientes correcciones y mejoras:

- Se ha mejorado el control del código de salida.  En algunos casos, se emitía un código de salida de 0 (cero) incluso cuando se producía un error de ejecución donde debería haber sido distinto de cero.  Los códigos de salida ahora solo deben ser cero cuando `azacsnap` se ejecuta correctamente hasta el final y un número distinto de cero en caso de error.  Además, el control de errores internos de AzAcSnap se ha ampliado para capturar y emitir el código de salida de los comandos externos (por ejemplo, hdbsql, ssh) ejecutados por AzAcSnap, si son la causa del error.

Descargue la [versión más reciente](https://aka.ms/azacsnapdownload) del instalador y revise [cómo empezar](azacsnap-get-started.md).

## <a name="april-2021"></a>Abril de 2021

### <a name="azacsnap-v50-build-202104216349---ga-released-21-april-2021"></a>AzAcSnap v5.0 (compilación: 20210421.6349): versión de disponibilidad general (21 de abril de 2021)

AzAcSnap v5.0 (compilación: 20210421.6349) se ha publicado para disponibilidad general y, para esta compilación, se han realizado las siguientes correcciones y mejoras:

- El tiempo de espera de reintento de hdbsql (para esperar una respuesta de SAP HANA) se establece automáticamente en la mitad del valor de "savePointAbortWaitSeconds" para evitar condiciones de carrera.  El valor de "savePointAbortWaitSeconds" se puede modificar directamente en el archivo de configuración JSON y debe ser de 600 segundos como mínimo.

## <a name="march-2021"></a>Marzo de 2021

### <a name="azacsnap-v50-preview-build2021031830771"></a>Versión preliminar de AzAcSnap v 5.0 (compilación: 20210318.30771)

La versión preliminar de AzAcSnap v 5.0 (compilación: 20210318.30771) se ha publicado con las siguientes correcciones y mejoras:

- Se ha quitado la necesidad de agregar el usuario AZACSNAP al SAP HANA bases de los inquilinos, consulte la sección [Habilitar la comunicación con SAP Hana](azacsnap-installation.md#enable-communication-with-sap-hana).
- Corrección para permitir una [restauración](azacsnap-cmd-ref-restore.md) con volúmenes configurados con QoS manual.
- Se ha agregado un control mutex para limitar las conexiones SSH para Azure Large Instance.
- Corrija el instalador para controlar los nombres de ruta de acceso con espacios y otros problemas relacionados.
- Como preparación para la compatibilidad con otros servidores de bases de datos, cambió el parámetro opcional "--hanasid" a "--dbsid".

## <a name="next-steps"></a>Pasos siguientes

- [Primeros pasos con la herramienta Azure Application Consistent Snapshot](azacsnap-get-started.md)
