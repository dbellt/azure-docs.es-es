---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: fauhse
ms.service: storage
ms.topic: include
ms.date: 2/20/2020
ms.author: fauhse
ms.custom: include file
ms.openlocfilehash: 532ac3b60f90a3d587a57ad8ea9663a6cfad3d90
ms.sourcegitcommit: ba8f0365b192f6f708eb8ce7aadb134ef8eda326
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2021
ms.locfileid: "109644919"
---
Un recurso compartido de archivos de Azure en la nube en una cuenta de almacenamiento de Azure.
Aquí se aplica otro nivel de consideraciones relativas al rendimiento.

Si tiene recursos compartidos muy activos (recursos compartidos que usan muchos usuarios o aplicaciones), dos recursos compartidos de archivos de Azure podrían alcanzar el límite de rendimiento de una cuenta de almacenamiento.

Un procedimiento recomendado es implementar cuentas de almacenamiento con un recurso compartido de archivos cada una.
Puede agrupar varios recursos compartidos de archivos de Azure en la misma cuenta de almacenamiento si tiene recursos compartidos de archivo o que espera que tengan escasa actividad diaria.

Estas consideraciones se aplican más al acceso directo a la nube (a través de una VM de Azure) que a Azure File Sync. Si tiene pensado usar solo Azure File Sync en estos recursos compartidos, es correcta la agrupación de varios en una sola cuenta de almacenamiento de Azure.

Si ha creado una lista de recursos compartidos, tiene que asignar cada recurso compartido a la cuenta de almacenamiento en la que se encontrará.

En la fase anterior, estableció el número adecuado de recursos compartidos. En este paso, tiene una asignación de cuentas de almacenamiento a recursos compartidos de archivos. Ahora debe implementar el número adecuado de cuentas de almacenamiento de Azure con el número adecuado de recursos compartidos de archivos de Azure en ellas.

Asegúrese de que la región de cada una de las cuentas de almacenamiento sea la misma y coincida con la región del recurso del servicio de sincronización de almacenamiento que ya ha implementado.

> [!CAUTION]
> Si crea un recurso compartido de archivos de Azure con un límite de 100 TiB, ese recurso compartido puede usar solo las opciones de redundancia de almacenamiento con redundancia local o de zona. Tenga en cuenta sus necesidades de redundancia de almacenamiento antes de usar recursos compartidos de archivos de 100 TiB.

Los recursos compartidos de archivos de Azure todavía se crean con un límite de 5 TiB de forma predeterminada. Dado que va a crear nuevas cuentas de almacenamiento, asegúrese de seguir las [instrucciones para crear cuentas de almacenamiento que permitan recursos compartidos de archivos de Azure con límites de 100 TiB](../articles/storage/files/storage-files-how-to-create-large-file-share.md).

Otra consideración a la hora de implementar una cuenta de almacenamiento es la redundancia de Azure Storage. Consulte las [Opciones de redundancia de Azure Storage](../articles/storage/common/storage-redundancy.md).

Los nombres de los recursos también son importantes. Por ejemplo, si agrupa varios recursos compartidos para el Departamento de Recursos Humanos en una cuenta de almacenamiento de Azure, debe asignar el nombre adecuado a la cuenta de almacenamiento. Del mismo modo, al asignar el nombre a los recursos compartidos de archivos de Azure, tiene que usar nombres similares a los que se usan para sus homólogos locales.