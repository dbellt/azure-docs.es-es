---
title: archivo de inclusión
description: archivo de inclusión
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 07/15/2019
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: 39b61c78cbd0333183c80234256d6041a109d846
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "68249911"
---
Un identificador de recurso de Azure AD indica la audiencia con la que se puede usar un token emitido para proporcionar acceso a un recurso de Azure. En el caso de Azure Storage, el identificador de recurso puede ser específico de una sola cuenta de almacenamiento o puede aplicarse a cualquiera de ellas. En la tabla siguiente se describen los valores que se pueden proporcionar para el identificador de recurso:

|Id. de recurso  |Descripción  |
|---------|---------|
|`https://<account>.blob.core.windows.net` <br /><br /> `https://<account>.queue.core.windows.net`    | El punto de conexión de servicio para una cuenta de almacenamiento determinada. Use este valor para adquirir un token con el fin de autorizar solicitudes solo a esa cuenta y a ese servicio de Azure Storage específicos. Reemplace el valor entre corchetes por el nombre de la cuenta de almacenamiento.      |
|`https://storage.azure.com/`     | Úselo para adquirir un token para autorizar solicitudes a cualquier cuenta de Azure Storage.        |
