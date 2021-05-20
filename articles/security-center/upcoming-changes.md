---
title: Próximos cambios importantes en Azure Security Center
description: Próximos cambios en Azure Security Center que debe tener en cuenta y para los que puede que necesite un plan
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: overview
ms.date: 05/09/2021
ms.author: memildin
ms.openlocfilehash: ed9ea3abf984f537ab693ccbadb90e2ba091f52d
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109683576"
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
| [El prefijo de las alertas de Kubernetes cambia de "AKS_" a "K8s_"](#prefix-for-kubernetes-alerts-changing-from-aks_-to-k8s_)                                               | Junio de 2021                 |
| [La implementación heredada de ISO 27001 se está reemplazando por la nueva ISO 27001:2013](#legacy-implementation-of-iso-27001-is-being-replaced-with-new-iso-270012013)          | Junio de 2021                 |
| [Las recomendaciones de AWS se lanzarán para disponibilidad general (GA)](#recommendations-from-aws-will-be-released-for-general-availability-ga)                     | **Agosto** de 2021           |
| [Mejoras en la recomendación de clasificación de datos de SQL](#enhancements-to-sql-data-classification-recommendation)                                                     | Segundo trimestre de 2021                   |
|                                                                                                                                                                       |                           |


### <a name="two-recommendations-from-apply-system-updates-security-control-being-deprecated"></a>Dos recomendaciones del control de seguridad "Aplicar actualizaciones del sistema" entran en desuso

**Fecha estimada del cambio:** abril de 2021

Las dos recomendaciones siguientes están en desuso:

- **La versión del sistema operativo debe actualizarse en los roles del servicio en la nube**: de manera predeterminada, Azure actualiza periódicamente el sistema operativo invitado a la imagen compatible más reciente dentro de la familia de sistema operativo que ha especificado en la configuración del servicio (.cscfg), como Windows Server 2016.
- Los **servicios de Kubernetes deben actualizarse a una versión de Kubernetes no vulnerable**: las evaluaciones de esta recomendación no son tan amplias como nos gustaría. La versión actual de esta recomendación se reemplazará finalmente por una versión mejorada que se ajuste más a las necesidades de seguridad de los clientes.


### <a name="prefix-for-kubernetes-alerts-changing-from-aks_-to-k8s_"></a>El prefijo de las alertas de Kubernetes cambia de "AKS_" a "K8s_"

**Fecha estimada del cambio:** junio de 2021

Azure Defender para Kubernetes se amplió recientemente para proteger los clústeres de Kubernetes hospedados en entornos locales y de varias nubes. Puede encontrar más información en [Uso de Azure Defender para Kubernetes para proteger implementaciones de Kubernetes híbridas y de varias nubes (en versión preliminar)](release-notes.md#use-azure-defender-for-kubernetes-to-protect-hybrid-and-multi-cloud-kubernetes-deployments-in-preview).

Para reflejar el hecho de que las alertas de seguridad proporcionadas por Azure Defender para Kubernetes ya no están restringidas a los clústeres de Azure Kubernetes Service, el prefijo de los tipos de alerta va a cambiar de "AKS_" a "K8s_". Cuando sea necesario, los nombres y las descripciones también se actualizarán. Por ejemplo, esta alerta:

|Alerta (tipo de alerta)|Descripción|
|----|----|
|Se ha detectado la herramienta de pruebas de penetración de Kubernetes.<br>(**AKS** _PenTestToolsKubeHunter)|El análisis del registro de auditoría de Kubernetes ha detectado el uso de la herramienta de pruebas de penetración de Kubernetes en el clúster de **AKS**. Aunque este comportamiento puede ser legítimo, los atacantes podrían usar estas herramientas públicas con fines malintencionados.
|||

se convertirá en:

|Alerta (tipo de alerta)|Descripción|
|----|----|
|Se ha detectado la herramienta de pruebas de penetración de Kubernetes.<br>(**K8s** _PenTestToolsKubeHunter)|El análisis del registro de auditoría de Kubernetes ha detectado el uso de la herramienta de pruebas de penetración de Kubernetes en el clúster de **Kubernetes**. Aunque este comportamiento puede ser legítimo, los atacantes podrían usar estas herramientas públicas con fines malintencionados.|
|||

Las reglas de supresión que hacen referencia a las alertas que comienzan por "AKS_" se convertirán automáticamente. Si ha configurado exportaciones de SIEM o scripts de automatización personalizados que hacen referencia a las alertas de Kubernetes por tipo de alerta, deberá actualizarlos con los nuevos tipos de alerta.

Para ver una lista completa de las alertas de Kubernetes, consulte [Alertas de clústeres de Kubernetes](alerts-reference.md#alerts-akscluster).

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