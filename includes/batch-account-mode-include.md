---
title: archivo de inclusión
description: archivo de inclusión
services: batch
author: laurenhughes
ms.service: batch
ms.topic: include
ms.date: 05/04/2018
ms.author: lahugh
ms.custom: include file
ms.openlocfilehash: 51b687dc2a6264eb12362616429746c9b182c3b1
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "68323885"
---
> [!NOTE]
> Al crear una cuenta de Batch, puede elegir entre dos modos de *asignación de grupo*: **suscripción de usuario** y **servicio Batch**. En la mayoría de los casos, debe usar el modo predeterminado servicio Batch, en el que los grupos se asignan en segundo plano en las suscripciones administradas por Azure. En el modo de suscripción de usuario alternativo, tanto las máquinas virtuales de Batch como otros recursos se crean directamente en su suscripción cuando se crea un grupo. El modo de suscripción de usuario es necesario si quiere crear grupos de [Azure Reserved Virtual Machine Instances](https://azure.microsoft.com/pricing/reserved-vm-instances/). Para crear una cuenta de Batch en modo de suscripción de usuario, también debe registrar su suscripción a Azure Batch y asociar la cuenta a una instancia de Azure Key Vault.