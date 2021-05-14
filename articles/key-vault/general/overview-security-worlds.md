---
title: Espacios de seguridad de Azure Key Vault | Microsoft Docs
description: Azure Key Vault es un servicio multiinquilino. Usa un grupo de HSM en cada región de Azure. Todas las regiones de una geografía comparten un límite criptográfico.
ms.service: key-vault
ms.subservice: general
ms.topic: conceptual
author: msmbaldwin
ms.author: mbaldwin
ms.date: 07/03/2017
ms.openlocfilehash: 0d82a3cb4c08d47b6827072378b9827037d32412
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107751821"
---
# <a name="azure-key-vault-security-worlds-and-geographic-boundaries"></a>Espacios de seguridad y límites geográficos de Azure Key Vault

Los productos de Azure están disponibles en varias [zonas geográficas de Azure](https://azure.microsoft.com/en-us/global-infrastructure/geographies/), y cada geografía de Azure contiene una o varias regiones. Por ejemplo, la geografía de Europa contiene dos regiones (Norte de Europa y Oeste de Europa), mientras que la única región de la geografía de Brasil es Sur de Brasil.

Azure Key Vault es un servicio multiinquilino que usa un grupo de módulos de seguridad de hardware (HSM). Todos los HSM de una geografía comparten el mismo límite criptográfico, denominado "entorno de seguridad". Cada geografía corresponde a un único entorno de seguridad y viceversa.

Por ejemplo, las regiones del Este de EE. UU. y Oeste de EE. UU. comparten el mismo entorno de seguridad porque pertenecen a la ubicación geográfica de Estados Unidos. Del mismo modo, todas las regiones de Azure en Japón comparten el mismo entorno de seguridad, así como todas las regiones de Azure de Australia, etc.

>[!NOTE]
> Una excepción es que el Departamento de Defensa del este y del centro de Estados Unidos tienen sus propios ámbitos de seguridad.

## <a name="backup-and-restore-behavior"></a>Tareas de copia de seguridad y restauración

Una copia de seguridad tomada de una clave en una instancia de Key Vault de una región de Azure puede restaurarse en una instancia de Key Vault de otra región de Azure, siempre que se den estas dos condiciones:

- Las dos regiones de Azure pertenecen a la misma geografía.
- Los dos almacenes de claves pertenecen a la misma suscripción de Azure.

Por ejemplo, una copia de seguridad de una clave realizada en un almacén de claves del Oeste de la India se puede restaurar en otro almacén de claves de la misma suscripción en la geografía de la India (esto es, en las regiones Oeste de la India, Centro de la India y Sur de la India).

## <a name="next-steps"></a>Pasos siguientes

- Consulte los [Productos de Microsoft por región](https://azure.microsoft.com/regions/services/).
