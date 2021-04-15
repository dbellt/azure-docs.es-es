---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: cb160a140b5c0cb184a5172da10ade0de37c4fed
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186359"
---
<!--v-sharos 10/13/2105 virtual device security-->

Tenga en cuenta las siguientes consideraciones de seguridad al utilizar el dispositivo virtual de StorSimple:

* El dispositivo virtual se protege mediante su suscripción de Microsoft Azure. Esto significa que si está utilizando el dispositivo virtual y se pone en peligro su suscripción de Azure, los datos almacenados en el dispositivo virtual también se ponen en peligro.
* La clave pública del certificado usada para cifrar los datos almacenados en Azure StorSimple está disponible de forma segura en el Portal de Azure clásico; la clave privada se conserva con el dispositivo StorSimple. En el dispositivo virtual de StorSimple, las claves públicas y privadas se almacenan en Azure.
* El dispositivo virtual está hospedado en el centro de datos de Microsoft Azure.

