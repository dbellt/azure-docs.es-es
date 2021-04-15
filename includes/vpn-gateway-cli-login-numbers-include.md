---
title: archivo de inclusión
description: archivo de inclusión
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file, devex-tag-azurecli
ms.openlocfilehash: 1a22c63eb01abc3775b2bf299d8464323e69c372
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107386920"
---
1. Inicie sesión en la suscripción de Azure con el comando [az login](/cli/azure/) y siga las instrucciones que aparecen en pantalla. Para más información sobre el inicio de sesión, consulte [Introducción a la CLI de Azure](/cli/azure/get-started-with-azure-cli).

   ```azurecli
   az login
   ```
2. Si tiene más de una suscripción de Azure, enumere las suscripciones de la cuenta.

   ```azurecli
   az account list --all
   ```
3. Especifique la suscripción que desea usar.

   ```azurecli
   az account set --subscription <replace_with_your_subscription_id>
   ```
