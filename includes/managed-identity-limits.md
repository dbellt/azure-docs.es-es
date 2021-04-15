---
title: archivo de inclusión
description: archivo de inclusión
services: active-directory
author: daveba
ms.service: active-directory
ms.subservice: msi
ms.topic: include
ms.date: 05/31/2018
ms.author: daveba
ms.custom: include file
ms.openlocfilehash: 6e5885e076222cd23ba127f3be41c1218f327ca0
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "92379709"
---
- Cada identidad administrada cuenta para el límite de cuota de objetos de un inquilino de Azure AD, como se indica en [Restricciones y límites del servicio Azure AD](../articles/active-directory/enterprise-users/directory-service-limits-restrictions.md).
-   La velocidad a la que se pueden crear identidades administradas tiene los siguientes límites:

    1. Por inquilino de Azure AD por región de Azure: 200 operaciones de creación en 20 segundos.
    2. Por suscripción de Azure por región de Azure: 40 operaciones de creación en 20 segundos.

- Al crear identidades administradas asignadas por el usuario, solo se admiten caracteres alfanuméricos (0-9, a-z, A-Z) y el guion (-). Para que la asignación a una máquina virtual o un conjunto de escalado de máquinas virtuales funcione correctamente, el nombre está limitado a 24 caracteres.
