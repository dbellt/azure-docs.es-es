---
title: Incorporación de réplicas a Azure Cache for Redis
description: Aprenda a agregar más réplicas a las instancias de Azure Cache for Redis de nivel Premium.
author: yegu-ms
ms.author: yegu
ms.service: cache
ms.topic: conceptual
ms.date: 08/11/2020
ms.openlocfilehash: aef8055d289ada8a62078185517c3ba129972488
ms.sourcegitcommit: 17345cc21e7b14e3e31cbf920f191875bf3c5914
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2021
ms.locfileid: "110099862"
---
# <a name="add-replicas-to-azure-cache-for-redis"></a>Incorporación de réplicas a Azure Cache for Redis
En este artículo, aprenderá a configurar una instancia de caché de Azure con réplicas adicionales mediante Azure Portal.

Los niveles Estándar y Premium de Azure Cache for Redis ofrecen redundancia mediante el hospedaje de cada caché en dos máquinas virtuales (VM) dedicadas. Estas máquinas virtuales están configuradas como principal y réplica. Cuando la máquina virtual principal deja de estar disponible, la réplica lo detecta y toma el relevo automáticamente como la nueva principal. Ahora puede aumentar el número de réplicas en una caché Premium hasta tres, lo que le dará un total de cuatro máquinas virtuales que respaldan una caché. Tener varias réplicas tiene como resultado una mayor resistencia que la que puede proporcionar una sola.

## <a name="prerequisites"></a>Requisitos previos
* Una suscripción a Azure: [cree una cuenta gratuita](https://azure.microsoft.com/free/)

## <a name="create-a-cache"></a>Creación de una caché
Para crear una instancia de caché, siga estos pasos:

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y después seleccione **Crear un recurso**.
  
1. En la página **Nuevo**, seleccione **Base de datos** y, a continuación, seleccione **Azure Cache for Redis**.

    :::image type="content" source="media/cache-create/new-cache-menu.png" alt-text="Selección de Azure Cache for Redis.":::
   
1. En la página **Basics** (Información básica), configure las opciones de la nueva instancia de caché.
   
    | Configuración      | Valor sugerido  | Descripción |
    | ------------ |  ------- | -------------------------------------------------- |
    | **Suscripción** | Seleccione su suscripción. | La suscripción en la que se creará esta nueva instancia de Azure Cache for Redis. | 
    | **Grupos de recursos** | Seleccione un grupo de recursos o **Crear nuevo**, y escriba un nombre nuevo para el grupo de recursos. | Nombre del grupo de recursos en el que se van a crear la caché y otros recursos. Al colocar todos los recursos de la aplicación en un grupo de recursos, puede administrarlos o eliminarlos fácilmente. | 
    | **Nombre DNS** | Escriba un nombre único global. | El nombre de la memoria caché debe ser una cadena de entre 1 y 63 caracteres, y solo puede contener números, letras o guiones. El nombre debe comenzar y terminar por un número o una letra y no puede contener guiones consecutivos. El *nombre de host* de la instancia de caché será *\<DNS name>.redis.cache.windows.net*. | 
    | **Ubicación** | Seleccione una ubicación. | Seleccione una [región](https://azure.microsoft.com/regions/) cerca de otros servicios que vayan a usar la memoria caché. |
    | **Tipo de caché** | Seleccione un plan de tarifa [Premium](https://azure.microsoft.com/pricing/details/cache/). |  El plan de tarifa determina el tamaño, el rendimiento y las características disponibles para la memoria caché. Para más información, consulte la [introducción a Azure Redis Cache](cache-overview.md). |
   
1. En la página **Advanced** (Avanzadas), elija **Recuento de réplicas**.
   
    :::image type="content" source="media/cache-how-to-multi-replicas/create-multi-replicas.png" alt-text="Recuento de réplicas.":::

1. Deje las demás opciones con los valores predeterminados. 

    > [!NOTE]
    > Actualmente, la compatibilidad con varias réplicas solo funciona con cachés no agrupadas.
    >

1. Haga clic en **Crear**.
   
    La caché tarda un tiempo en crearse. Puede supervisar el progreso en la página **Información general** de Azure Cache for Redis. Cuando **Estado** se muestra como **En ejecución**, la memoria caché está lista para su uso.

    > [!NOTE]
    > Una vez creado, el número de réplicas de una caché no se puede cambiar.
    >

## <a name="next-steps"></a>Pasos siguientes
Más información sobre las características de Azure Cache for Redis.

> [!div class="nextstepaction"]
> [Niveles de servicio de Azure Cache for Redis Premium](cache-overview.md#service-tiers)
