---
title: Descripción de las directivas de seguridad, las iniciativas y las recomendaciones en Azure Security Center
description: Conozca las directivas de seguridad, las iniciativas y las recomendaciones en Azure Security Center
author: memildin
ms.author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: conceptual
ms.date: 03/04/2021
ms.openlocfilehash: eb6369e89066451779d05a97057acbdd451de192
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108146678"
---
# <a name="what-are-security-policies-initiatives-and-recommendations"></a>¿Qué son las directivas de seguridad, las iniciativas y las recomendaciones?

Security Center aplica iniciativas de seguridad a las suscripciones. Estas iniciativas contienen una o varias directivas de seguridad. Cada una de esas directivas da como resultado una recomendación de seguridad para mejorar la posición de seguridad. En esta página se explica con detalle cada una de estas ideas.


## <a name="what-is-a-security-policy"></a>¿Qué es una directiva de seguridad?

Una definición de directiva de Azure, creada en Azure Policy, es una regla sobre condiciones de seguridad específicas que quiere controlar. Las definiciones integradas incluyen elementos como el control del tipo de recursos que se pueden implementar o la aplicación del uso de etiquetas en todos los recursos. También puede crear su propia definición de directiva personalizada.

Para implementar estas definiciones de directiva (tanto integradas como personalizadas), será preciso asignarlas. Puede asignar cualquiera de estas directivas a través de Azure Portal, PowerShell o la CLI de Azure.

Hay diferentes tipos de directivas en Azure Policy. Security Center utiliza principalmente directivas de "auditoría" que comprueban condiciones y configuraciones específicas y, luego, notifican el cumplimiento. También hay directivas de "aplicación" que se pueden usar para aplicar una configuración segura.

## <a name="what-is-a-security-initiative"></a>¿Qué es una iniciativa de seguridad?

Una iniciativa de Azure es una colección de definiciones de directivas de Azure, o reglas, agrupadas para satisfacer un objetivo o propósito concreto. Las iniciativas de Azure simplifican la administración de las directivas al agrupar conjuntos de directivas, de manera lógica, en un único elemento.

Una iniciativa de seguridad define la configuración deseada de las cargas de trabajo y le ayuda a garantizar el cumplimiento de los requisitos de seguridad normativos o corporativos.

Al igual que las directivas de seguridad, las iniciativas de Security Center también se crean en Azure Policy. Puede usar [Azure Policy](../governance/policy/overview.md) para administrar las directivas y crear iniciativas y asignarlas a varias suscripciones o a grupos de administración completos.

La iniciativa predeterminada asignada automáticamente a todas las suscripciones en Azure Security Center es Azure Security Benchmark. Este punto de referencia es el conjunto de directrices específico de Azure y creado por Microsoft relativo a los procedimientos recomendados de seguridad y cumplimiento basados en marcos de cumplimiento comunes. Esta prueba comparativa, que cuenta con amplísimo respaldo, se basa en los controles del [Centro de seguridad de Internet (CIS)](https://www.cisecurity.org/benchmark/azure/) y del [Instituto Nacional de Normas y Tecnología (NIST)](https://www.nist.gov/), con un enfoque en seguridad centrada en la nube. Mas información sobre [Azure Security Benchmark](/security/benchmark/azure/introduction).

Security Center le ofrece las siguientes opciones para trabajar con iniciativas y directivas de seguridad:

- **Ver y editar la iniciativa predeterminada integrada**: al habilitar Security Center, la iniciativa denominada "Azure Security Benchmark" se asigna automáticamente a todas las suscripciones registradas de Security Center. Para personalizar esta iniciativa, puede habilitar o deshabilitar directivas individuales en ella mediante la edición de los parámetros de una directiva. Consulte la lista de [directivas de seguridad integradas](./policy-reference.md) para comprender las opciones disponibles.

- **Agregar sus propias directivas personalizadas**: si quiere personalizar las iniciativas de seguridad que se aplican a su suscripción, puede hacerlo en Security Center. A continuación, recibirá recomendaciones si las máquinas no siguen las directivas que creó. Para instrucciones sobre la creación y asignación de directivas personalizadas, consulte [Uso de iniciativas y directivas de seguridad personalizadas](custom-security-policies.md).

- **Agregar estándares de cumplimiento normativo como iniciativas**: el panel de cumplimiento normativo de Security Center muestra el estado de todas las evaluaciones del entorno, en el contexto de una normativa o estándar determinado (por ejemplo, Azure CIS, NIST SP 800-53 R4 o SWIFT CSP CSCF-v2020). Para obtener más información, consulte [Mejora del cumplimiento normativo](security-center-compliance-dashboard.md).

## <a name="what-is-a-security-recommendation"></a>¿Qué es una recomendación de seguridad?

Mediante las directivas, Security Center analiza periódicamente el estado de cumplimiento de los recursos para identificar posibles debilidades o errores de configuración de seguridad. Luego, proporciona recomendaciones sobre cómo corregir esos problemas. Las recomendaciones son el resultado de la evaluación de los recursos con respecto a las directivas pertinentes y la identificación de los recursos que no cumplen los requisitos definidos.

Security Center realiza sus recomendaciones de seguridad en función de las iniciativas elegidas. Cuando una directiva de una iniciativa se compara con los recursos y se encuentra que uno o varios no se ajustan al cumplimiento normativo, se presenta como una recomendación en Security Center.

Las recomendaciones son acciones que se deben llevar a cabo para proteger y consolidar los recursos. Cada recomendación proporciona la siguiente información:

- Descripción breve del problema
- Pasos de corrección que se deben llevar a cabo para implementar la recomendación.
- Recursos afectados.

En la práctica, funciona de la siguiente manera:

1. Azure Security Benchmark es una ***iniciativa***.
1. Incluye una ***directiva*** que exige que todas las cuentas de Azure Storage restrinjan el acceso a la red para reducir la superficie expuesta a ataques. Esta directiva se denomina "Storage accounts should restrict network access using virtual network rules" (Las cuentas de almacenamiento deben restringir el acceso a la red mediante reglas de red virtual) y se puede deshabilitar o habilitar en Azure Policy.
1. Si Azure Security Center encuentra una cuenta de Azure Storage en cualquiera de las suscripciones protegidas, evalúa esas cuentas para ver si están protegidas con reglas de red virtual. Si no es así, se muestra una ***recomendación*** para corregir esa situación y reforzar la seguridad de esos recursos. 

Por lo tanto, una iniciativa (1) incluye directivas (2) que generan recomendaciones cuando corresponda (3). 

## <a name="viewing-the-relationship-between-a-recommendation-and-a-policy"></a>Visualización de la relación entre una recomendación y una directiva

Como se mencionó anteriormente, las recomendaciones integradas de Security Center se basan en la prueba comparativa de seguridad de Azure. Casi todas las recomendaciones tienen una directiva subyacente derivada de un requisito en la prueba comparativa.

Cuando se revisan los detalles de una recomendación, a menudo resulta útil poder ver la directiva subyacente. Para cada recomendación compatible con una directiva, use el vínculo **View policy definition** (Ver definición de directiva) de la página de detalles de la recomendación para ir directamente a la entrada Azure Policy de la directiva pertinente:

:::image type="content" source="media/release-notes/view-policy-definition.png" alt-text="Vínculo a la página de Azure Policy de la directiva específica que admite una recomendación":::

Use este vínculo para ver la definición de directiva y revisar la lógica de evaluación. 

Si va a revisar la lista de recomendaciones de la [guía de referencia de recomendaciones de seguridad](recommendations-reference.md), también verá vínculos a las páginas de definición de directiva:

:::image type="content" source="media/release-notes/view-policy-definition-from-documentation.png" alt-text="Acceso a la página de Azure Policy de una directiva específica directamente desde la página de referencia de recomendaciones de Azure Security Center":::


## <a name="next-steps"></a>Pasos siguientes

En esta página se explica, en líneas generales, los conceptos básicos y las relaciones entre las directivas, las iniciativas y las recomendaciones. Para obtener información relacionada, consulte:

- [Creación de iniciativas personalizadas](custom-security-policies.md)
- [Deshabilitación de las directivas de seguridad para desactivar las recomendaciones](tutorial-security-policy.md#disable-security-policies-and-disable-recommendations)
- [Obtenga información sobre cómo editar una directiva de seguridad en Azure Policy](../governance/policy/tutorials/create-and-manage.md).