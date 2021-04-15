---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 12/27/2019
ms.author: rogarana
ms.custom: include file
ms.openlocfilehash: b7c097306ba46d4f0024aecc55994508e2d8a090
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96011768"
---
Para proteger los datos de los recursos compartidos de archivos de Azure contra la pérdida o daños de los datos, todos los recursos compartidos de los archivos de Azure almacenan varias copias de cada archivo a medida que se escriben. En función de los requisitos de la carga de trabajo, puede seleccionar niveles adicionales de redundancia. Actualmente, Azure Files admite las siguientes opciones de redundancia de datos:

- **Redundancia local** El almacenamiento con redundancia local, que a menudo se denomina LRS, significa que todos los archivos se almacenan tres veces dentro de un clúster de almacenamiento de Azure. Esto protege contra la pérdida de datos debido a errores de hardware, como una unidad de disco incorrecta.
- **Redundancia de zona** El almacenamiento con redundancia de zona, que a menudo se conoce como ZRS, significa que cada archivo se almacena tres veces en tres clústeres de almacenamiento de Azure distintos. Al igual que con el almacenamiento con redundancia local, la redundancia de zona ofrece tres copias de cada archivo, sin embargo, estas copias están aisladas físicamente en tres clústeres de almacenamiento distintos en distintas *zonas de disponibilidad* de Azure. Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. No se acepta la escritura en el almacenamiento hasta que se escribe en los clústeres de almacenamiento en las tres zonas de disponibilidad. 
- **Redundancia** geográfica El almacenamiento con redundancia geográfica, que a menudo se conoce como GRS, es como el almacenamiento con redundancia local, en el que un archivo se almacena tres veces dentro de un clúster de almacenamiento de Azure en la región primaria. Todas las escrituras se replican de forma asincrónica en una región secundaria definida por Microsoft. El almacenamiento con redundancia geográfica proporciona seis copias de los datos distribuidos entre dos regiones de Azure. En caso de que se produzca un desastre importante, como la pérdida permanente de una región de Azure debido a un desastre natural o a otro evento similar, Microsoft realizará una conmutación por error para que el servidor secundario en vigor se convierta en el servidor principal, atendiendo a todas las operaciones. Dado que la replicación entre las regiones principal y secundaria es asincrónica, en caso de que se produzca un desastre importante, se perderán los datos que todavía no se hayan replicado en la región secundaria. También puede realizar una conmutación por error manual de una cuenta de almacenamiento con redundancia geográfica.
- **Redundancia de zona geográfica** El almacenamiento con redundancia de zona geográfica, que a menudo se conoce como GZRS, es como el almacenamiento con redundancia de zona, en el que un archivo se almacena tres veces en tres clústeres de almacenamiento distintos en la región primaria. Todas las escrituras se replican de forma asincrónica en una región secundaria definida por Microsoft. El proceso de conmutación por error para el almacenamiento con redundancia de zona geográfica funciona igual que para el almacenamiento con redundancia geográfica.

Los recursos compartidos de archivos de Azure estándar admiten los cuatro tipos de redundancia, mientras que los recursos compartidos de archivos Premium de Azure solo admiten almacenamiento con redundancia local y redundancia de zona

Las cuentas de almacenamiento de uso general versión 2 (GPv2) proporcionan dos opciones de redundancia adicionales que no son compatibles con Azure Files: almacenamiento con redundancia geográfica con acceso de lectura, que a menudo se conoce como RA-GRS, y almacenamiento con redundancia de zona geográfica accesible. se conoce como RA-GZRS. Puede aprovisionar recursos compartidos de archivos de Azure en cuentas de almacenamiento con estas opciones establecidas, pero Azure Files no admite la lectura desde la región secundaria. Los recursos compartidos de archivos de Azure implementados en cuentas de almacenamiento con redundancia geográfica o con redundancia de zona geográfica con acceso de lectura se facturarán como almacenamiento con redundancia geográfica o con redundancia de zona geográfica, respectivamente.