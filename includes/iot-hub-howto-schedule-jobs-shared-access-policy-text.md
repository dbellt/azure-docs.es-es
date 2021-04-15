---
title: archivo de inclusión
description: archivo de inclusión
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: include
ms.date: 07/17/2019
ms.author: robinsh
ms.custom: include file
ms.openlocfilehash: e493d1c4f5851ee510ea83e706afce5fbb6f487e
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "70049078"
---
<!-- This contains intro text for the "Get an IoT hub connection string" section in the iot-hub-lang-lang-schedule-jobs.md files-->

En este artículo se creará un servicio de back-end que programa un trabajo para que invoque un método directo en un dispositivo, programa un trabajo para actualizar el dispositivo gemelo y supervisa el progreso de cada trabajo. Para realizar estas operaciones, el servicio necesita los permisos de **lectura del registro** y **escritura en el registro**. De manera predeterminada, todas las instancias del centro de IoT se crean con una directiva de acceso compartido denominada **registryReadWrite** que concede estos permisos.
