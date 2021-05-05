---
title: Definiciones de directivas integradas en Azure Security Center
description: Aquí se enumeran las definiciones de directivas integradas de Azure Policy en Azure Security Center. Estas definiciones de directivas integradas proporcionan enfoques comunes para administrar los recursos de Azure.
ms.date: 04/28/2021
ms.topic: reference
author: memildin
ms.author: memildin
ms.service: security-center
ms.custom: subject-policy-reference
ms.openlocfilehash: fb23fb24ca0376d84cb498049d60e1859769943a
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108179055"
---
# <a name="azure-policy-built-in-definitions-for-azure-security-center"></a>Definiciones integradas de Azure Policy en Azure Security Center

Esta página es un índice de las definiciones de directivas integradas de [Azure Policy](../governance/policy/overview.md) relacionadas con Azure Security Center. Están disponibles las siguientes agrupaciones de definiciones de directivas:

- El grupo [iniciativas](#azure-security-center-initiatives) muestra las definiciones de iniciativas de Azure Policy en la categoría "Security Center".
- En el grupo [iniciativa predeterminada](#azure-security-center-initiatives) se muestran todas las definiciones de Azure Policy que forman parte de la iniciativa predeterminada de Azure Security Center, [Azure Security Benchmark](/security/benchmark/azure/introduction). Este punto de referencia ampliamente respetado creado por Microsoft está basado en los controles del [Centro de seguridad de Internet (CIS)](https://www.cisecurity.org/benchmark/azure/) y del [National Institute of Standards and Technology (NIST)](https://www.nist.gov/) y hace hincapié en la seguridad centrada en la nube.
- El grupo [categoría](#azure-security-center-category) muestra todas las definiciones de Azure Policy en la categoría "Security Center".

Para más información acerca de las directivas de seguridad, consulte [Uso de directivas de seguridad](./tutorial-security-policy.md). Puede encontrar elementos integrados adicionales de Azure Policy para otros servicios en [Definiciones de elementos integrados de Azure Policy](../governance/policy/samples/built-in-policies.md).

El nombre de cada definición de directiva integrada se vincula a la definición de directiva en Azure Portal. Use el vínculo de la columna **Versión** para ver el origen en el [repositorio de GitHub de Azure Policy](https://github.com/Azure/azure-policy).

## <a name="azure-security-center-initiatives"></a>Iniciativas de Azure Security Center

Para comprender las directivas integradas que Security Center supervisa, consulte la tabla siguiente:

[!INCLUDE [azure-policy-reference-policyset-security-center](../../includes/policy/reference/bycat/policysets-security-center.md)]

## <a name="security-centers-default-initiative-azure-security-benchmark"></a>Iniciativa predeterminada de Security Center (Azure Security Benchmark)

Para comprender las directivas integradas que Security Center supervisa, consulte la tabla siguiente:

[!INCLUDE [azure-policy-reference-init-asc](../../includes/policy/reference/custom/init-asc.md)]

## <a name="azure-security-center-category"></a>Categoría Azure Security Center

[!INCLUDE [azure-policy-reference-category-securitycenter](../../includes/policy/reference/bycat/policies-security-center.md)]

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido sobre las definiciones de directiva de seguridad de Azure Policy en Security Center. Para obtener más información sobre las iniciativas, las directivas y cómo se relacionan con las recomendaciones de Security Center, vea [¿Qué son las directivas de seguridad, las iniciativas y las recomendaciones?](security-policy-concept.md)
