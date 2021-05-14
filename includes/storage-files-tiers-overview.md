---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 08/28/2020
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: a5fd0394feb8b58688bd163d62af30b031125b46
ms.sourcegitcommit: 3de22db010c5efa9e11cffd44a3715723c36696a
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109663927"
---
Azure Files ofrece cuatro niveles diferentes de almacenamiento: Premium, Optimizado para transacciones, Frecuente y Esporádico, con el fin de que pueda adaptar sus recursos compartidos a los requisitos de rendimiento y precio de su escenario:

- **Premium**: Los recursos compartidos de archivos Premium tienen el respaldo de discos SSD y proporcionan alto rendimiento y baja latencia de forma consistente en menos de 10 milisegundos en la mayoría de las operaciones de E/S para las cargas de trabajo con mayor uso de E/S. Los recursos compartidos de archivos Premium son adecuados para una amplia variedad de cargas de trabajo como bases de datos, hospedaje de sitios web y entornos de desarrollo. Los recursos compartidos de archivos Premium se pueden usar con los protocolos Bloque de mensajes del servidor (SMB) y Network File System (NFS).
- **Optimizado para transacciones**: Los recursos compartidos de archivos con el nivel Optimizado para transacciones permiten cargas de trabajo con muchas transacciones que no necesitan la latencia que ofrecen los recursos compartidos de archivos Premium. Los recursos compartidos de archivos optimizados para transacciones se ofrecen en el hardware de almacenamiento estándar respaldado por unidades de disco duro (HDD). A los optimizados para transacciones se les ha llamado históricamente "estándar"; sin embargo, realmente es el tipo de soporte físico de almacenamiento, en lugar del propio nivel (tanto el acceso frecuente como el esporádico también son niveles "estándar", ya que se encuentran en el hardware de almacenamiento estándar).
- **Acceso frecuente**: Los recursos compartidos de nivel de acceso frecuente ofrecen almacenamiento optimizado para escenarios de uso compartido de archivos de uso general, como los recursos compartidos de los equipos. Los recursos compartidos de archivos de nivel de acceso frecuente se ofrecen en el hardware de almacenamiento estándar respaldado por unidades de disco duro.
- **Acceso esporádico**: Los recursos compartidos de archivos de acceso esporádico ofrecen un almacenamiento económico optimizado para escenarios de almacenamiento de archivo en línea. Los recursos compartidos de archivos de nivel de acceso esporádico se ofrecen en el hardware de almacenamiento estándar respaldado por unidades de disco duro.

Los recursos compartidos de archivos Premium se implementan en la **cuenta de almacenamiento de FileStorage** y solo están disponibles en un modelo de facturación aprovisionado. Para más información sobre el modelo de facturación aprovisionado para recursos compartidos de archivos Premium, consulte [Planeamiento de una implementación de Azure Files](../articles/storage/files/understanding-billing.md#provisioned-model). Los recursos compartidos de archivos estándar, que incluyen los optimizados para las transacciones, los de nivel de acceso frecuente y los de nivel de acceso esporádico, se implementan en la **cuenta de almacenamiento de uso general, versión 2 (GPv2)** y están disponibles en un modelo de pago por uso. 

Al seleccionar una capa de almacenamiento para la carga de trabajo, tenga en cuenta los requisitos de rendimiento y uso. Si la carga de trabajo requiere una latencia de un solo dígito o si usa medios de almacenamiento SSD en un entorno local, es probable que el nivel Premium sea el más adecuado. Si no es muy importante que la latencia sea baja, por ejemplo, en el caso de recursos compartidos de equipo montados localmente desde Azure o almacenados en la caché local mediante Azure File Sync, desde el punto de vista del costo es posible que sea más adecuado usar el almacenamiento estándar.

Una vez que se haya creado un recurso compartido de archivos en una cuenta de almacenamiento, no se podrá mover a niveles exclusivos de diferentes tipos de cuenta de almacenamiento. Por ejemplo, para cambiar un recurso compartido de archivos optimizado para transacciones al nivel Premium, es preciso crear un recurso compartido de archivos en una cuenta de almacenamiento de FileStorage y copiar los datos desde el recurso compartido original a un nuevo recurso compartido de archivos en la cuenta de FileStorage. Se recomienda usar AzCopy para copiar datos entre recursos compartidos de archivos de Azure, pero también se pueden usar herramientas como `robocopy` en Windows o `rsync` en macOS y Linux. 

Los recursos compartidos de archivos implementados en cuentas de almacenamiento GPv2 se pueden mover entre los niveles estándar (transacción optimizada, de acceso frecuente y de acceso esporádico) sin crear ninguna cuenta de almacenamiento y migrar los datos, pero será preciso abonar los costos de transacción al cambiar el nivel. Si un recurso compartido se pasa de un nivel de acceso frecuente a un nivel esporádico, incurrirá en la carga de transacciones de escritura del nivel más esporádico en cada archivo del recurso compartido. Por contra, si un recurso compartido se pasa de un nivel de acceso esporádico a un nivel frecuente, incurrirá en la carga de transacciones de lectura del nivel más esporádico en cada archivo del recurso compartido.

Para más información, consulte [Facturación de Azure Files](../articles/storage/files/understanding-billing.md).