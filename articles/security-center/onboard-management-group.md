---
title: Incorporación de un grupo de administración en Security Center
description: Aprenda a usar una definición de Azure Policy suministrada para habilitar Azure Security Center en todas las suscripciones de un grupo de administración.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 03/11/2021
ms.author: memildin
ms.openlocfilehash: ea5522cfd8f407ffd7259335706cc69555ce2329
ms.sourcegitcommit: f3b930eeacdaebe5a5f25471bc10014a36e52e5e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2021
ms.locfileid: "112238251"
---
# <a name="enable-security-center-on-all-subscriptions-in-a-management-group"></a>Habilitación de Security Center en todas las suscripciones de un grupo de administración

Puede usar Azure Policy para habilitar Azure Security Center en todas las suscripciones de Azure del mismo grupo de administración (MG). Esto resulta más cómodo que acceder a ellas de forma individual desde el portal y funciona aunque las suscripciones pertenezcan a distintos propietarios. 

Para incorporar un grupo de administración y todas sus suscripciones:

1. Como usuario con permisos de **administrador de seguridad**, abra Azure Policy y busque la definición **Habilitar Azure Security Center en la suscripción**.

    :::image type="content" source="./media/security-center-get-started/enable-security-center-policy.png" alt-text="Definición de Azure Policy Habilitar Azure Security Center en la suscripción.":::

1. Seleccione **Asignar** y asegúrese de establecer el ámbito en el nivel de grupo de administración.

    :::image type="content" source="./media/security-center-get-started/assign-policy.png" alt-text="Asignación de la definición Habilitar Azure Security Center en la suscripción.":::

    > [!TIP]
    > Aparte del ámbito, no hay parámetros obligatorios.

1. Seleccione **Crear una tarea de corrección** para asegurarse de que se incorporarán todas las suscripciones existentes que no tienen Security Center habilitado.

    :::image type="content" source="./media/security-center-get-started/remediation-task.png" alt-text="Creación de una tarea de corrección para la definición de Azure Policy Habilitar Azure Security Center en la suscripción.":::

1. Cuando se asigne la definición:

    1. Detecte todas las suscripciones en el grupo de administración que aún no estén registradas en Security Center.
    1. Marque esas suscripciones como "no compatibles".
    1. Marque como "compatibles" todas las suscripciones registradas (independientemente de que tengan Azure Defender activado o no).

    Luego, la tarea de corrección habilitará Security Center, de forma gratuita, en las suscripciones no compatibles.

> [!IMPORTANT]
> La definición de la directiva solo habilitará Security Center en las suscripciones **existentes**. Para registrar las suscripciones recién creadas, abra la pestaña de cumplimiento, seleccione las suscripciones no compatibles pertinentes y cree una tarea de corrección. Repita este paso cuando tenga una o varias suscripciones nuevas que desee supervisar con Security Center.

## <a name="optional-modifications"></a>Modificaciones opcionales

Hay varias formas de modificar la definición de Azure Policy: 

- **Definir el cumplimiento de forma diferente**: la directiva suministrada clasifica todas las suscripciones del grupo de administración que aún no están registradas en Security Center como "no compatibles". Puede elegir establecerla en todas las suscripciones sin Azure Defender.

    La definición suministrada, puede definir *cualquiera* de los valores de "precio" siguientes como compatibles. Esto significa que una suscripción establecida como "estándar" o "gratis" es compatible.

    > [!TIP]
    > Cuando se habilita un plan de Azure Defender, se describe en una definición de directiva con el valor "Estándar". Cuando se deshabilita es "Gratis". Para más información acerca de las diferencias entre estos planes, consulte [Azure Security Center gratuito frente a Azure Defender habilitado](security-center-pricing.md). 

    ```
    "existenceCondition": {
        "anyof": [
            {
                "field": "microsoft.security/pricings/pricingTier",
                "equals": "standard"
            },
            {
                "field": "microsoft.security/pricings/pricingTier",
                "equals": "free"
            }
        ]
    },
    ```

    Si lo cambia a lo siguiente, solo se clasificarían como compatibles las suscripciones establecidas como "estándar":

    ```
    "existenceCondition": {
          {
            "field": "microsoft.security/pricings/pricingTier",
            "equals": "standard"
          },
    },
    ```

- **Definir los planes de Azure Defender que se aplican al habilitar Security Center**: la directiva suministrada habilita Security Center sin ninguno de los planes de Azure Defender opcionales. Puede elegir habilitar uno o varios de ellos.

    La sección `deployment` de la definición suministrada incluye el parámetro `pricingTier`. De forma predeterminada, se establece en `free`, pero se puede modificar. 


## <a name="next-steps"></a>Pasos siguientes:

Ahora que ha incorporado todo un grupo de administración, habilite las protecciones avanzadas de Azure Defender. 

> [!div class="nextstepaction"]
> [Habilitación de Azure Defender](enable-azure-defender.md)