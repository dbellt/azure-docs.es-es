---
title: Próximos cambios importantes en Azure Security Center
description: Próximos cambios en Azure Security Center que debe tener en cuenta y para los que puede que necesite un plan
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 04/27/2021
ms.author: memildin
ms.openlocfilehash: b8b8b15083711c868add7ac041514bcf1facc30d
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108076894"
---
# <a name="important-upcoming-changes-to-azure-security-center"></a>Próximos cambios importantes en Azure Security Center

> [!IMPORTANT]
> La información de esta página está relacionada con productos o características en versión preliminar que pueden modificarse sustancialmente antes de lanzarse comercialmente, si es que lo hacen finalmente. Microsoft no ofrece ningún compromiso o garantía, ni expresa ni implícita, con respecto a la información que se proporciona aquí.

En esta página, obtendrá información sobre los cambios planeados para Security Center. Describe las modificaciones planeadas en el producto que pueden afectar a aspectos como los flujos de trabajo o la puntuación segura.

Si busca las notas de la versión más recientes, puede encontrarlas en [Novedades de Azure Security Center](release-notes.md).


## <a name="planned-changes"></a>Cambios planeados

| Cambio planeado                                                                                                                                                        | Fecha estimada del cambio |
|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------|
| [Dos recomendaciones del control de seguridad "Aplicar actualizaciones del sistema" entran en desuso](#two-recommendations-from-apply-system-updates-security-control-being-deprecated) | Abril de 2021                |
| [La implementación heredada de ISO 27001 se está reemplazando por la nueva ISO 27001:2013](#legacy-implementation-of-iso-27001-is-being-replaced-with-new-iso-270012013)          | Junio de 2021                 |
| [Las recomendaciones de AWS se lanzarán para disponibilidad general (GA)](#recommendations-from-aws-will-be-released-for-general-availability-ga)                     | **Agosto** de 2021           |
| [Mejoras en la recomendación de clasificación de datos de SQL](#enhancements-to-sql-data-classification-recommendation)                                                     | Segundo trimestre de 2021                   |
|                                                                                                                                                                       |                           |


### <a name="two-recommendations-from-apply-system-updates-security-control-being-deprecated"></a>Dos recomendaciones del control de seguridad "Aplicar actualizaciones del sistema" entran en desuso

**Fecha estimada del cambio:** abril de 2021

Las dos recomendaciones siguientes están en desuso:

- **La versión del sistema operativo debe actualizarse en los roles del servicio en la nube**: de manera predeterminada, Azure actualiza periódicamente el sistema operativo invitado a la imagen compatible más reciente dentro de la familia de sistema operativo que ha especificado en la configuración del servicio (.cscfg), como Windows Server 2016.
- Los **servicios de Kubernetes deben actualizarse a una versión de Kubernetes no vulnerable**: las evaluaciones de esta recomendación no son tan amplias como nos gustaría. La versión actual de esta recomendación se reemplazará finalmente por una versión mejorada que se ajuste más a las necesidades de seguridad de los clientes.


### <a name="legacy-implementation-of-iso-27001-is-being-replaced-with-new-iso-270012013"></a>La implementación heredada de ISO 27001 se está reemplazando por la nueva ISO 27001:2013

La implementación heredada de ISO 27001 se eliminará del panel de cumplimiento normativo de Security Center. Si está haciendo un seguimiento del cumplimiento de la norma ISO 27001 mediante Security Center, incorpore el nuevo estándar ISO 27001:2013 para los grupos de administración o suscripciones pertinentes, y la norma ISO 27001 heredada actual pronto se eliminará del panel.

:::image type="content" source="media/upcoming-changes/removing-iso-27001-legacy-implementation.png" alt-text="Panel de cumplimiento normativo de Security Center que muestra el mensaje sobre la eliminación de la implementación heredada de ISO 27001." lightbox="media/upcoming-changes/removing-iso-27001-legacy-implementation.png":::

### <a name="recommendations-from-aws-will-be-released-for-general-availability-ga"></a>Las recomendaciones de AWS se lanzarán para disponibilidad general (GA)

**Fecha estimada del cambio:** agosto de 2021

Azure Security Center protege las cargas de trabajo de Azure, Amazon Web Services (AWS) y Google Cloud Platform (GCP).

Las recomendaciones que provienen de AWS Security Hub han estado en versión preliminar desde que se introdujeron los conectores en la nube. Las recomendaciones marcadas como **Versión preliminar** no se incluyen en los cálculos de la puntuación segura, sino que deben corregirse siempre que sea posible, de modo que cuando el período de versión preliminar finalice, contribuyan a la puntuación.

Con este cambio, dos conjuntos de recomendaciones de AWS pasarán a disponibilidad general:

- [Controles de PCI DSS de Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-pci-controls.html)
- [Controles del banco de pruebas de CIS AWS Foundations de Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/securityhub-cis-controls.html)

Cuando se encuentren en versión de disponibilidad general y las evaluaciones se ejecuten en los recursos de AWS, los resultados afectarán a la puntuación segura combinada para todos los recursos de nube híbrida y multinube.



### <a name="enhancements-to-sql-data-classification-recommendation"></a>Mejoras en la recomendación de clasificación de datos de SQL

**Fecha estimada del cambio:** Segundo trimestre de 2021

La recomendación **Los datos confidenciales de las bases de datos SQL deben clasificarse** del control de seguridad **Aplicación de la clasificación de datos** se reemplazará por una nueva versión que esté más alineada con la estrategia de clasificación de datos de Microsoft. Como consecuencia, el identificador de la recomendación también cambiará (actualmente es b0df6f56-862d-4730-8597-38c0fd4ebd59).



## <a name="next-steps"></a>Pasos siguientes

Para ver todos los cambios recientes realizados en el producto, consulte [Novedades de Azure Security Center](release-notes.md).