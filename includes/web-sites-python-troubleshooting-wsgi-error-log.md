---
title: archivo de inclusión
description: archivo de inclusión
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/11/2018
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 020e59f029b09f3c7656f67039731e4141e68d31
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186919"
---
Si Python encuentra un error al iniciar la aplicación, solo se devolverá una página de error simple (por ejemplo, "No se puede mostrar la página porque se ha producido un error interno del servidor").

Para capturar errores de aplicaciones Python:

1. En Azure Portal, en la aplicación web, seleccione **Configuración**.
2. En la pestaña **Configuración**, seleccione **Configuración de la aplicación**.
3. En **Configuración de la aplicación**, escriba el siguiente par clave-valor:
    * Clave: WSGI_LOG
    * Valor: D:\home\site\wwwroot\logs.txt (escriba el nombre de archivo que elija)

Ahora debería ver los errores en el archivo logs.txt en la carpeta wwwroot.
