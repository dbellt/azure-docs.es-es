---
author: alkohli
ms.service: databox
ms.topic: include
ms.date: 08/21/2019
ms.author: alkohli
ms.openlocfilehash: eb55d993ad8960f821c2b72f0a53602166b7cc7e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "69900606"
---
Para los datos en reposo:

- El acceso a los datos almacenados en recursos compartidos está restringido.

    - Los clientes SMB que tienen acceso a datos del recurso compartido necesitan credenciales de usuario asociadas al recurso compartido. Estas credenciales se definen cuando se crea el recurso compartido.
    - Las direcciones IP de los clientes NFS con acceso a un recurso compartido deben agregarse cuando se crea el recurso compartido.
