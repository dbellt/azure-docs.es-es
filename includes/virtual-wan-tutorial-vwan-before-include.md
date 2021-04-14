---
title: archivo de inclusión
description: archivo de inclusión
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: include
ms.date: 09/12/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 945d701a2a7dffc259c601b4dab9fa1333ccc066
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "96023537"
---
Antes de iniciar la configuración, compruebe que cumple los siguientes criterios:

* Si ya tiene una red virtual a la que quiere conectarse, compruebe que ninguna de las subredes de la red local se superponga con ella. La red virtual no requiere una subred de puerta de enlace y no puede tener ninguna puerta de enlace de red virtual. Si no tiene una red virtual, siga los pasos de este artículo para crear una.
* Obtenga un intervalo de direcciones IP para la región del concentrador. El centro de conectividad es una red virtual, y el intervalo de direcciones que especifique para la región del concentrador no se puede superponer con ninguna de las redes virtuales existentes a las que esté conectado. Tampoco se puede superponer con los intervalos de direcciones a los que se conecte en el entorno local. Si no está familiarizado con los intervalos de direcciones IP ubicados en la configuración de red local, consulte a alguien que pueda proporcionarle estos detalles.
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.