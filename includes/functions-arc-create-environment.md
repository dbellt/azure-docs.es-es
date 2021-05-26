---
author: cephalin
ms.service: app-service
ms.topic: include
ms.date: 05/12/2021
ms.author: cephalin
ms.openlocfilehash: 43eda9f0f5cdb108c2949205d72433c9d7176571
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371999"
---
## <a name="create-an-app-service-kubernetes-environment"></a>Creación de un entorno de Kubernetes de App Service

Antes de comenzar, debe [crear un entorno de Kubernetes de App Service](../articles/app-service/manage-create-arc-environment.md) para un clúster de Kubernetes habilitado para Azure Arc. 

> [!NOTE] 
> Al crear el entorno, asegúrese de anotar el nombre de la ubicación personalizada y el nombre del grupo de recursos que la contiene. Puede usarlos para encontrar el identificador de la ubicación personalizada, que necesitará al crear la aplicación de funciones en el entorno. 
>
> Si no ha creado el entorno, acuda al administrador del clúster.