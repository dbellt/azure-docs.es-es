---
title: Directivas de nube por niveles de Azure File Sync | Microsoft Docs
description: Detalles sobre cómo las directivas de espacio disponible del volumen y fecha funcionan conjuntamente en diferentes escenarios.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 04/13/2021
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 4376a57b677b95b2de7d261b30ac4c0ad24956cc
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107796522"
---
# <a name="cloud-tiering-policies"></a>Directivas de nube por niveles

La nube por niveles tiene dos directivas que determinan qué archivos se almacenan por niveles en la nube: **directiva de espacio disponible del volumen** y **directiva de fecha**.

La **directiva de espacio disponible del volumen** garantiza que un porcentaje especificado del volumen local en el que se encuentra el punto de conexión de servidor siempre se mantiene disponible. 

La **directiva de fecha** almacena por niveles los archivos a los que se ha accedido hace x días o más. La directiva de espacio disponible del volumen siempre tiene prioridad; cuando no haya espacio suficiente en el volumen para conservar todos los archivos que especifica la directiva de fecha, Azure File Sync invalidará la directiva de fecha y seguirá almacenando por niveles los archivos con acceso más esporádico hasta alcanzar el porcentaje de espacio libre en el volumen.

## <a name="how-both-policies-work-together"></a>Funcionamiento conjunto de ambas directivas

Para ilustrar cómo funcionan estas directivas, se usará un ejemplo: supongamos que ha configurado Azure File Sync en un volumen local de 500 GB y que la nube por niveles nunca se ha habilitado. Estos son los archivos del recurso compartido de archivos:

|Nombre de archivo |Hora de último acceso  |Tamaño de archivo  |Se almacena en |
|----------|------------------|-----------|----------|
|Archivo 1    | Hace 2 días  | 10 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 2    | Hace 10 días | 30 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 3    | Hace 1 año | 200 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 4    | Hace 1 año y 2 días | 130 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 5    | Hace 2 años y 1 día | 140 GB | Servidor y recurso compartido de archivos de Azure

**Cambio 1:** Ha habilitado la nube por niveles, ha establecido una directiva de espacio disponible del volumen del 20 % y ha mantenido deshabilitada la directiva de fecha. Con esa configuración, la nube por niveles garantiza que el 20 % (en este caso, 100 GB) del espacio se mantiene libre y disponible en la máquina local. Como resultado, la capacidad total de la caché local es de 400 GB. Esos 400 GB almacenarán los archivos a los que se ha accedido más recientemente y con mayor frecuencia en el volumen local.

Con esta configuración, solo los archivos del 1 al 4 se almacenarían en la caché local y el archivo 5 se almacenaría por niveles. Esto supone 370 GB de los 400 GB que se pueden usar. El archivo 5 ocupa 140 GB y superaría el límite de 400 GB si se almacenara en la caché local. 

**Cambio 2:** Imagine que un usuario accede al archivo 5. Esto convierte al archivo 5 en el archivo al que se ha accedido más recientemente en el recurso compartido. Como resultado, el archivo 5 se almacenaría en la caché local y, para no superar el límite de 400 GB, el archivo 4 se almacenaría por niveles. En la tabla siguiente se muestra dónde se almacenan los archivos, con estas actualizaciones:

|Nombre de archivo |Hora de último acceso  |Tamaño de archivo  |Se almacena en |
|----------|------------------|-----------|----------|
|Archivo 5    | hace 2 horas | 140 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 1    | Hace 2 días  | 10 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 2    | Hace 10 días | 30 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 3    | Hace 1 año | 200 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 4    | Hace 1 año y 2 días | 130 GB | Recurso compartido de archivos de Azure, almacenado por niveles localmente

**Cambio 3:** Supongamos que ha actualizado las directivas para que la directiva de almacenamiento por niveles basada en la fecha sea de 60 días y la directiva de espacio disponible del volumen sea del 70 %. Ahora, solo se pueden almacenar hasta 150 GB en la caché local. Aunque el acceso al archivo 2 se ha producido hace menos de 60 días, la directiva de espacio disponible del volumen invalidará la directiva de fecha y el archivo 2 se almacenará por niveles para mantener el espacio libre local del 70 %.

**Cambio 4:** Si ha cambiado la directiva de espacio disponible del volumen al 20 % y después ha usado `Invoke-StorageSyncFileRecall` para recuperar todos los archivos que caben en la unidad local, pero respetando al mismo tiempo las directivas de la nube por niveles, la tabla sería similar a la siguiente:

|Nombre de archivo |Hora de último acceso  |Tamaño de archivo  |Se almacena en |
|----------|------------------|-----------|----------|
|Archivo 5    | Hace 1 hora  | 140 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 1    | Hace 2 días  | 10 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 2    | Hace 10 días | 30 GB | Servidor y recurso compartido de archivos de Azure
|Archivo 3    | Hace 1 año | 200 GB | Recurso compartido de archivos de Azure, almacenado por niveles localmente
|Archivo 4    | Hace 1 año y 2 días | 130 GB | Recurso compartido de archivos de Azure, almacenado por niveles localmente

En este caso, los archivos 1, 2 y 5 se almacenarían en la caché local y los archivos 3 y 4 por niveles. Dado que la directiva de fecha es de 60 días, los archivos 3 y 4 están almacenados por niveles, aunque la directiva de espacio disponible del volumen permite hasta 400 GB localmente.

> [!NOTE] 
> Los archivos no se recuperan automáticamente cuando los clientes cambian la directiva de espacio disponible del volumen por un valor más pequeño (por ejemplo, del 20 al 10 %) o cambian la directiva de fecha por un valor mayor (por ejemplo, de 20 a 50 días).

## <a name="multiple-server-endpoints-on-a-local-volume"></a>Varios puntos de conexión de servidor en un volumen local

La nube por niveles se puede habilitar para varios puntos de conexión de servidor en un único volumen local. En esta configuración, debe establecer el espacio disponible del volumen en la misma cantidad en todos los puntos de conexión de servidor del mismo volumen. Si establece diferentes directivas de espacio disponible del volumen en varios puntos de conexión de servidor del mismo volumen, el porcentaje de espacio disponible en el volumen más grande tendrá prioridad. A esto se le llama **directiva de espacio disponible del volumen efectiva**. Por ejemplo, si tiene tres puntos de conexión de servidor en el mismo volumen local, uno establecido en el 15 %, otro en el 20 % y un tercero en el 30 %, todos comenzarán a almacenar por niveles los archivos a los que se accede con menos frecuencia cuando tengan menos del 30 % de espacio libre disponible.

## <a name="next-steps"></a>Pasos siguientes

* [Supervisión de la nube por niveles](file-sync-monitor-cloud-tiering.md)
