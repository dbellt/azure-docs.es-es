---
title: Cuotas en Azure Static Web Apps
description: Más información sobre las cuotas asociadas a Azure Static Web Apps
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: overview
ms.date: 05/08/2020
ms.author: cshoe
ms.openlocfilehash: 12666d69f6dcab043e909dbb2b49276644a70d76
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110069569"
---
# <a name="quotas-in-azure-static-web-apps"></a>Cuotas en Azure Static Web Apps

Existen las siguientes cuotas para Azure Static Web Apps.

| Característica                     | Plan gratuito        | Plan Estándar |
|-----------------------------|------------------|---------------|
| Ancho de banda incluido          | 100 GB al mes, por suscripción | 100 GB al mes, por suscripción |
| Ancho de banda de uso por encima del límite           | No disponible      | 0,20 USD por GB |
| Aplicaciones por suscripción a Azure | 10               | Sin límite |
| Tamaño de la aplicación                    | 250 MB           | 500 MB |
| Entornos de preproducción | 3                | 10 |
| Dominios personalizados              | 2 por aplicación        | 5 por aplicación |
| Autorización (con roles personalizados y reglas de enrutamiento) | Máximo de 25 usuarios finales que pueden pertenecer a roles personalizados | Máximo de 25 usuarios finales que pueden pertenecer a roles personalizados |

## <a name="github-storage"></a>Almacenamiento de GitHub

Los paquetes y las Acciones de GitHub usan el almacenamiento de GitHub, que tiene su propio conjunto de cuotas. Cuando crea un sitio de Azure Static Web Apps, GitHub almacena los artefactos para el sitio incluso después de la implementación.

Para obtener más información, consulte los siguientes recursos.

- [Administración del espacio de almacenamiento de acciones](https://github.community/t5/GitHub-Actions/Managing-Actions-storage-space/td-p/38944)
- [Acerca de la facturación para las Acciones de GitHub](https://help.github.com/github/setting-up-and-managing-billing-and-payments-on-github/about-billing-for-github-actions#about-billing-for-github-actions)
- [Administración del límite de gasto para las acciones de GitHub](https://help.github.com/github/setting-up-and-managing-billing-and-payments-on-github/managing-your-spending-limit-for-github-actions)

## <a name="next-steps"></a>Pasos siguientes

- [Información general](overview.md)
