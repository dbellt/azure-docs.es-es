---
author: Blackmist
ms.service: machine-learning
ms.topic: include
ms.date: 04/21/2021
ms.author: larryfr
ms.openlocfilehash: 6d9b6fe18d7eb4ce532c6c2960944420eee952f0
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107903978"
---
> [!IMPORTANT]
> La instancia de Cosmos DB se crea en un grupo de recursos administrados por Microsoft en __su suscripción__. Los siguientes servicios también se crean en este grupo de recursos y se usan en la configuración de claves administradas por el cliente:
> * Cuenta de Azure Storage
> * Azure Search
>
> Puesto que estos servicios se crean en la suscripción de Azure, se le cobrará por estas instancias de servicio. 
>
> El grupo de recursos administrado se denomina con el formato `<AML Workspace Resource Group Name><GUID>`. Si el área de trabajo de Azure Machine Learning usa un punto de conexión privado, también se crea una red virtual en este grupo de recursos. Esta red virtual se usa para proteger la comunicación entre los servicios de este grupo de recursos y el área de trabajo de Azure Machine Learning.
> 
> * __No elimine el grupo de recursos__ que contiene esta instancia de Cosmos DB, ni ninguno de los recursos que se crean de forma automática en este grupo. Si necesita eliminar el grupo de recursos, la instancia de Cosmos DB, etc., primero debe eliminar el área de trabajo de Azure Machine Learning que la usa. El grupo de recursos, la instancia de Cosmos DB y los otros recursos que se crean automáticamente se eliminan cuando se elimina el área de trabajo asociada.
> * Las [__Unidades de solicitud__](../articles/cosmos-db/request-units.md) predeterminadas para esta cuenta de Cosmos DB son __8000__.
> * No __puede proporcionar una red virtual propia para usarla con la instancia de Cosmos DB__ que se crea. Tampoco __puede modificar la red virtual__. Por ejemplo, no puede cambiar el rango de direcciones IP que usa.