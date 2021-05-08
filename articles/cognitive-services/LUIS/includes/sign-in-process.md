---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 04/23/2021
ms.topic: include
ms.openlocfilehash: 727178c2851e43c787c2b1fcf1e7bb349b3ea4f5
ms.sourcegitcommit: ad921e1cde8fb973f39c31d0b3f7f3c77495600f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/25/2021
ms.locfileid: "107948367"
---
## <a name="sign-in-to-luis-portal"></a>Inicio de sesión en el portal de LUIS

[!INCLUDE [Note about portal deprecation](luis-portal-note.md)]

Un nuevo usuario de LUIS debe seguir este procedimiento:

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su país o región, y acepte las condiciones de uso. Si ve **Mis aplicaciones**, significa que ya existe un recurso de LUIS y que debe omitirlo para crear una aplicación. De lo contrario, empiece por usar un recurso de Azure que le permite vincular una cuenta de LUIS con un recurso de creación de Azure Authoring nuevo o existente. <!---This is equivalent to signing up already migrated. You won't need to go through the [migration process](../luis-migration-authoring.md#what-is-migration) later on.-->
<!---
    * Using a trial key. This allows you to sign in to LUIS with a trial resource that you don't need to set up. If you choose this option, you will eventually be required to [migrate your account](../luis-migration-authoring.md#migration-steps) and link your applications to an authoring resource.
-->

2. En la ventana **Choose an authoring** (Elección de una creación) que aparece, busque una suscripción de Azure y un recurso de creación de LUIS. Si no tiene un recursos, puede crearlo.

    <!---:::image type="content" source="../media/luis-how-to-azure-subscription/choose-authoring-resource.png" alt-text="Choose a type of Language Understanding authoring resource.":::
    -->
    ![Elija un tipo de recurso de creación de Language Understanding.](../media/luis-how-to-azure-subscription/choose-authoring-resource.png)
    
    Al crear un recurso de creación, especifique la siguiente información:
    * **Nombre de inquilino**: el inquilino con el que está asociada la suscripción de Azure.
    * **Nombre de la suscripción**: la suscripción que se facturará para el recurso.
    * **Nombre del grupo de recursos de Azure**: nombre personalizado del grupo de recursos que elija o cree. Los grupos de recursos le permiten agrupar los recursos de Azure para el acceso y la administración.
    * **Nombre del recurso de Azure**: un nombre personalizado que elija y que se use como parte de la dirección URL para las consultas de punto de conexión de creación y predicción.
    * **Plan de tarifa**: el plan de tarifa determina el número máximo de transacciones por segundo y mes.


