---
author: IEvangelist
ms.author: dapine
ms.date: 06/25/2019
ms.service: cognitive-services
ms.topic: include
ms.openlocfilehash: 7f51b7fe95445cbd3a8aff530f89ce55b5abfb1e
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105629890"
---
Si necesita configurar un proxy HTTP para realizar solicitudes de salida, use estos dos argumentos:

| Nombre | Tipo de datos | Descripción |
|--|--|--|
|HTTP_PROXY|string|El proxy que se va a utilizar, por ejemplo, `http://proxy:8888`<br>`<proxy-url>`|
|HTTP_PROXY_CREDS|string|Las credenciales necesarias para autenticarse en el proxy, por ejemplo, `username:password`. Este valor **debe estar en minúsculas**. |
|`<proxy-user>`|string|El usuario del proxy.|
|`<proxy-password>`|string|La contraseña asociada con `<proxy-user>` para el proxy.|
||||


```bash
docker run --rm -it -p 5000:5000 \
--memory 2g --cpus 1 \
--mount type=bind,src=/home/azureuser/output,target=/output \
<registry-location>/<image-name> \
Eula=accept \
Billing=<endpoint> \
ApiKey=<api-key> \
HTTP_PROXY=<proxy-url> \
HTTP_PROXY_CREDS=<proxy-user>:<proxy-password> \
```
