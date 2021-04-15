---
author: alkohli
ms.service: databox
ms.subservice: pod
ms.topic: include
ms.date: 09/06/2020
ms.author: alkohli
ms.openlocfilehash: b9ff5968b4bb406f1a96780985b5c6fe64ca976c
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "89505913"
---
| N.º de puerto| Dentro o fuera | Ámbito de puerto| Obligatorio| Notas |
|--------|-----|-----|-----------|----------|
| TCP 80 (HTTP)|En|LAN|Sí|Este puerto se usa para conectarse a las API REST de Blob Storage de Data Box a través de HTTP. Si no se conecta a las API REST, se redirige automáticamente a la interfaz de usuario web local a través de 8443. |
| TCP 443 (HTTPS)|En|LAN|Sí|Este puerto se usa para conectarse a las API REST de Blob Storage de Data Box a través de HTTPS. Si no se conecta a las API REST, se redirige automáticamente a la interfaz de usuario web local a través de 8443. |
| TCP 8443 (HTTPS-Alt)|En|LAN|Sí|Se trata de un puerto alternativo para HTTPS y se usa al conectarse a la interfaz de usuario web local para la administración de dispositivos. |
| TCP 445 (SMB)|Salida/Entrada|LAN|En algunos casos.<br>Vea las notas|Este puerto solo es necesario si se va a conectar mediante SMB. |
| TCP 2049 (NFS)|Salida/Entrada|LAN|En algunos casos.<br>Vea las notas|Este puerto solo es necesario si se va a conectar mediante NFS. |
| TCP 111 (NFS)|Salida/Entrada|LAN|En algunos casos.<br>Vea las notas|Este puerto se usa para la asignación de rpcbind/puerto y solo es necesario si se está conectando a través de NFS.  |
