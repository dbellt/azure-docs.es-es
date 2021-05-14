---
title: Índice de ejemplos de planos técnicos
description: Índice de ejemplos de cumplimiento y estándar para implementar entornos, directivas y las bases de Cloud Adoption Framework con Azure Blueprints.
ms.date: 05/04/2021
ms.topic: sample
ms.openlocfilehash: a4e94e3f5a274fc30cb4d64d5c31846a4ddac03b
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108757176"
---
# <a name="azure-blueprints-samples"></a>Ejemplos de Azure Blueprints

En la tabla siguiente se incluyen vínculos a ejemplos de Azure Blueprints. Cada ejemplo tiene calidad de producción y está listo para su implementación inmediata para ayudarle a satisfacer sus requisitos de cumplimiento normativo.

## <a name="standards-based-blueprint-samples"></a>Ejemplos de planos técnicos basados en estándares

| Muestra | Descripción |
|---------|---------|
| [Australian Government ISM PROTECTED](./ism-protected/index.md) | Proporciona límites de protección para el cumplimiento de Australian Government ISM PROTECTED. |
| [Azure Security Benchmark](./azure-security-benchmark.md) | Proporciona protección para garantizar el cumplimiento de [Azure Security Benchmark](../../../security/benchmarks/overview.md). |
| [Azure Security Benchmark Foundation](./azure-security-benchmark-foundation/index.md) | Implementa y configura Azure Security Benchmark Foundation. |
| [Canada Federal PBMM](./canada-federal-pbmm.md) | Proporciona barreras de seguridad para garantizar el cumplimiento con la norma Canada Federal Protected B, Medium Integrity, Medium Availability (PBMM). |
| [CIS Microsoft Azure Foundations Benchmark v1.3.0](./cis-azure-1-3-0.md) | Proporciona un conjunto de directivas para ayudar a cumplir con las recomendaciones de la versión 1.3.0 de CIS Microsoft Azure Foundations Benchmark. |
| [CIS Microsoft Azure Foundations Benchmark v1.1.0](./cis-azure-1-1-0.md) | Proporciona un conjunto de directivas para ayudar a cumplir con las recomendaciones de la versión 1.1.0 de CIS Microsoft Azure Foundations Benchmark. |
| [CMMC nivel 3](./cmmc-l3.md) | Proporciona límites de protección que permiten cumplir la norma CMMC nivel 3. |
| [Nivel 4 de impacto de DoD](./dod-impact-level-4/index.md) | Proporciona un conjunto de directivas que ayudan a cumplir los requisitos del nivel 4 de impacto del DoD. |
| [Nivel de impacto 5 de DoD5](./dod-impact-level-5/index.md) | Proporciona un conjunto de directivas que ayudan a cumplir los requisitos del nivel de impacto 5 de DoD. |
| [FedRAMP Moderate](./fedramp-m/index.md) | Proporciona un conjunto de directivas para ayudar a cumplir con FedRAMP Moderate. |
| [Nivel de impacto alto de FedRAMP](./fedramp-h/index.md) | Proporciona un conjunto de directivas que ayudar a cumplir los requisitos del nivel de impacto alto de FedRAMP. |
| [HIPAA/HITRUST 9.2](./hipaa-hitrust-9-2.md) | Proporciona un conjunto de directivas para ayudar a cumplir con HIPAA/HITRUST. |
| [IRS 1075, septiembre de 2016](./irs-1075-sept2016.md) | Proporciona límites de protección que permiten cumplir la norma IRS 1075.|
| [ISO 27001](./iso-27001-2013.md) | Proporciona barreras de seguridad para garantizar el cumplimiento con la norma ISO 27001. |
| [ISO 27001: servicios compartidos](./iso27001-shared/index.md) | Proporciona un conjunto de patrones de infraestructura compatibles y una límites de protección de directivas que ayudan a obtener la atestación ISO 27001. |
| [Cargas de trabajo de App Service Environment y SQL Database compatibles con ISO 27001](./iso27001-ase-sql-workload/index.md) | Proporciona más infraestructura para el ejemplo de plano técnico de la [norma ISO 27001 sobre servicios compartidos](./iso27001-shared/index.md). |
| [Elementos multimedia](./media/index.md) | Proporciona un conjunto de directivas que ayudan a cumplir los requisitos del nivel de impacto alto de Media MPAA. |
| [ISM restringido en Nueva Zelanda](./new-zealand-ism.md) | Asigna directivas para abordar los controles específicos del manual de seguridad de información de Nueva Zelanda. |
| [NIST SP 800-53 R4](./nist-sp-800-53-r4.md) | Proporciona límites de protección para garantizar el cumplimiento de la norma NIST SP 800-53 R4. |
| [NIST SP 800-171 R2](./nist-sp-800-171-r2.md) | Proporciona límites de protección para garantizar el cumplimiento de la norma NIST SP 800-171 R2. |
| [PCI-DSS v3.2.1](./pci-dss-3.2.1/index.md) | Proporciona un conjunto de directivas que ayuda a lograr la conformidad con PCI DSS v3.2.1. |
| [SWIFT CSP-CSCF v2020](./swift-2020/index.md) | Ayuda en el cumplimiento de la norma SWIFT CSP-CSCF v2020. |
| [Gobernanza de UK OFFICIAL y UK NHS](./ukofficial-uknhs.md) | Proporciona un conjunto de patrones de infraestructura compatibles y una límites de protección de directivas que ayudan a obtener las atestaciones de UK OFFICIAL y UK NHS. |
| [Fundamentos de CAF](./caf-foundation/index.md) | Proporciona un conjunto de controles para ayudarle a administrar su propia nube en consonancia con [Microsoft Cloud Adoption Framework para Azure (CAF)](/azure/architecture/cloud-adoption/governance/journeys/index). |
| [Zona de aterrizaje de migración de CAF](./caf-migrate-landing-zone/index.md) | Proporciona un conjunto de controles para ayudarle a configurar la migración de la primera carga de trabajo y para administrar su propia nube en consonancia con [Microsoft Cloud Adoption Framework para Azure (CAF)](/azure/architecture/cloud-adoption/migrate/index). |

## <a name="samples-strategy"></a>Estrategia de ejemplos

:::image type="complex" source="../media/blueprint-samples-strategy.png" alt-text="Diagrama de dónde encajan los ejemplos de planos técnicos en la complejidad arquitectónica frente a los requisitos de cumplimiento." border="false":::
   Describe un sistema de coordenadas en el que la complejidad arquitectónica está en el eje X y los requisitos de cumplimiento se encuentran en el eje Y. Cuando aumenten tanto la complejidad arquitectónica como los requisitos de cumplimiento, adopte ejemplos de planos técnicos estándar del portal designado en la región E. En el caso de aquellos clientes que sea la primera vez que usan Azure, use los planos técnicos de base y zona de aterrizaje de Cloud Adoption Framework (CAF) designados por las regiones A y B. El espacio restante se atribuye a los planos técnicos personalizados creados por los clientes que son asociados para las regiones C, D y F. :::image-end:::

Los planos técnicos de la zona de aterrizaje de migración de CAF y de la fundación CAF asumen que el cliente prepara una suscripción individual limpia existente para migrar los recursos y cargas de trabajo locales a Azure
(regiones A y B en la ilustración).

Hay una oportunidad de iterar en los planos técnicos de ejemplo y buscar patrones de las personalizaciones que solicita el cliente. También existe la oportunidad de abordar de forma proactiva los planos técnicos específicos del sector, como los servicios financieros y el comercio electrónico (extremo superior de la región B). Del mismo modo, se prevé la creación de planos técnicos para consideraciones de arquitectura complejas, tales como suscripciones múltiples, alta disponibilidad, recursos entre regiones y clientes que implementan controles sobre las suscripciones y recursos existentes (regiones C y D).

Hay planos técnicos de ejemplo que abordan un escenario de cliente en el que los requisitos de cumplimiento son elevados y las complejidades de la arquitectura son altas (región E en la ilustración). La región F de la ilustración es a la que se dirigen los clientes y asociados que van a aplicar los planos técnicos de ejemplo y a personalizarlos en función de sus necesidades únicas.

## <a name="next-steps"></a>Pasos siguientes

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
- Puede consultar la información de [solución de problemas generales](../troubleshoot/general.md) para resolver los problemas durante la asignación de un plano técnico.
