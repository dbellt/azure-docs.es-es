---
title: archivo de inclusión
description: archivo de inclusión
services: app-service
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 06/14/2019
ms.author: cephalin
ms.custom: include file
ms.openlocfilehash: 8f1fe6ae38c708c5205f8c1230da05457d6b7010
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107764315"
---
Se puede implementar FTP y Git local en una aplicación web de Azure mediante un *usuario de implementación*. Una vez configurado este usuario de implementación, podrá usarlo en todas las implementaciones de Azure. El nombre de usuario y la contraseña en el nivel de cuenta son diferentes de las credenciales de suscripción de Azure. 

Para configurar el usuario de implementación, ejecute el comando [az webapp deployment user set](/cli/azure/webapp/deployment/user#az_webapp_deployment_user_set) en Azure Cloud Shell. Reemplace \<username> y \<password> por un nombre de usuario y una contraseña de usuario de implementación. 

- El nombre de usuario debe ser único dentro de Azure y no debe contener el símbolo "\@" para las inserciones de Git local. 
- La contraseña debe tener al menos ocho caracteres y dos de los tres elementos siguientes: letras, números y símbolos. 

```azurecli-interactive
az webapp deployment user set --user-name <username> --password <password>
```

La salida JSON muestra la contraseña como `null`. Si se produce un error `'Conflict'. Details: 409`, cambie el nombre de usuario. Si se produce un error `'Bad Request'. Details: 400`, use una contraseña más segura. 

Anote el nombre de usuario y la contraseña que se usarán para implementar las aplicaciones web.
