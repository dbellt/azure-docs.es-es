---
title: archivo de inclusión
description: archivo de inclusión
services: cognitive-services
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.date: 10/30/2020
ms.topic: include
ms.openlocfilehash: e006f804b8ab6411f4949424147acf567dc2ed24
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "97371325"
---
## <a name="sign-in-to-luis-portal"></a>Inicio de sesión en el portal de LUIS

[!INCLUDE [Note about portal deprecation](luis-portal-note.md)]

Un nuevo usuario de LUIS debe seguir este procedimiento:

1. Inicie sesión en el [portal de LUIS](https://www.luis.ai), seleccione su país o región, y acepte las condiciones de uso. Si ve **Mis aplicaciones**, significa que ya existe un recurso de LUIS y que debe omitirlo para crear una aplicación. Tendrá dos opciones de registro:

    * Uso de un recurso de Azure (recomendado): permite vincular una cuenta de LUIS con un recurso de creación de Azure Authoring nuevo o existente. Esto equivale a una suscripción que ya se ha migrado. No necesitará pasar por el [proceso de migración](../luis-migration-authoring.md#what-is-migration) más adelante.

    * Use una clave de prueba. Esto le permite iniciar sesión en LUIS con un recurso de prueba que no necesita configurar. Si elige esta opción, se le acabará pidiendo que [migre su cuenta](../luis-migration-authoring.md#migration-steps) y vincule las aplicaciones a un recurso de creación.

1. En la ventana **Choose an authoring** (Elección de una creación) que aparece, busque una suscripción de Azure y un recurso de creación de LUIS. Si no tiene un recursos, puede crearlo.

    :::image type="content" source="../media/luis-how-to-azure-subscription/choose-authoring-resource.png" alt-text="Elija un tipo de recurso de creación de Language Understanding.":::
    
    Al crear un recurso de creación, especifique la siguiente información:
    * **Nombre de inquilino**: el inquilino con el que está asociada la suscripción de Azure.
    * **Nombre de la suscripción**: la suscripción que se facturará para el recurso.
    * **Nombre del grupo de recursos de Azure**: nombre personalizado del grupo de recursos que elija o cree. Los grupos de recursos le permiten agrupar los recursos de Azure para el acceso y la administración.
    * **Nombre del recurso de Azure**: un nombre personalizado que elija y que se use como parte de la dirección URL para las consultas de punto de conexión de creación y predicción.
    * **Plan de tarifa**: el plan de tarifa determina el número máximo de transacciones por segundo y mes.


