---
title: archivo de inclusión
description: archivo de inclusión
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 08/07/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: e81ce2743d2509ce52f3190218decfa4e518bdad
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "70050472"
---
<!-- This contains intro text for the "Get an IoT hub connection string" section in the iot-hub-lang-lang-twin-getstarted.md files-->

En este artículo, se crea un servicio back-end que agrega propiedades deseadas a un dispositivo gemelo y luego consulta el registro de identidades para buscar todos los dispositivos con propiedades notificadas que se han actualizado en consecuencia. El servicio necesita el permiso **Conectar al servicio** para modificar las propiedades deseadas de un dispositivo gemelo y el permiso **Lectura del Registro** para consultar el registro de identidades. No hay ninguna directiva de acceso compartido predeterminada que contenga solo estos dos permisos, por lo que tendrá que crearla.
