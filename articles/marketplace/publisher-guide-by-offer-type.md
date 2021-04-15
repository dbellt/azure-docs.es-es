---
title: 'Guía de publicación por tipo de oferta: marketplace comercial de Microsoft'
description: En este artículo se describen los tipos de ofertas que están disponibles en marketplace comercial de Microsoft.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: conceptual
author: trkeya
ms.author: trkeya
ms.date: 04/06/2021
ms.openlocfilehash: 381ee45bd678ef22222483675ddf001e16970628
ms.sourcegitcommit: b0557848d0ad9b74bf293217862525d08fe0fc1d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "106552060"
---
# <a name="publishing-guide-by-offer-type"></a>Guía de publicación por tipo de oferta

En este artículo se describen los tipos de ofertas que están disponibles en marketplace comercial. El *tipo de oferta* define la estructura de la oferta, incluidos los metadatos, artefactos y otros contenidos usados para presentar la oferta en Marketplace.

Después de [decidir una opción de publicación](determine-your-listing-type.md), debe elegir un tipo de oferta antes de empezar a crear la oferta en el Centro de partners. El tipo de oferta se corresponderá con el tipo de oferta de solución, aplicación o servicio que quiera publicar, así como con su alineación con los servicios y productos de Microsoft.

> [!NOTE]
> Después de seleccionar un tipo de oferta, no se puede cambiar la oferta a otro tipo. Para crear otro tipo de oferta, debe crear una nueva oferta.

Un tipo de oferta se puede configurar de modos diferentes para habilitar distintas opciones de publicación, llamadas a la acción, aprovisionamiento o precios. La opción de publicación y la configuración del tipo de oferta también se alinea con la idoneidad de la oferta y los requisitos técnicos.

Asegúrese de revisar los requisitos de idoneidad del tipo de oferta y la tienda en línea, así como los requisitos técnicos de publicación antes de crear la oferta.

## <a name="list-of-offer-types"></a>Lista de tipos de oferta

En la tabla siguiente se muestran los tipos de oferta de Marketplace comercial en el Centro de partners.

| **Tipo de oferta**    | **Descripción**  |
| :------------------- | :-------------------|
| [**Aplicación de Azure**](plan-azure-application-offer.md) | Hay dos tipos de planes de la Aplicación de Azure: _plantilla de solución_ y _aplicación administrada_. Ambos tipos de planes admiten la automatización de la implementación y la configuración de una solución más allá de una sola máquina virtual (VM). Puede automatizar el proceso de suministrar varios recursos, incluidas VM, redes y recursos de almacenamiento, para proporcionar soluciones complejas, como soluciones IaaS. Ambos tipos de planes pueden emplear muchos tipos diferentes de recursos de Azure, incluidas, entre otras, VM.<ul><li>Los planes de **plantilla de solución** son uno de los principales mecanismos para publicar una solución en el marketplace comercial. Los planes de plantilla de solución no se pueden comercializar en el marketplace comercial, pero se pueden usar para implementar ofertas de VM de pago facturadas mediante el marketplace comercial. Use el tipo de plan de plantilla de solución cuando el cliente administre la solución y las transacciones se facturan a través de otro plan.</li><br><li>Los planes de **aplicación administrada** le permiten crear y entregar con facilidad a los clientes aplicaciones llave en mano totalmente administradas. Tienen las mismas funcionalidades que los planes de plantilla de solución, con algunas diferencias clave:</li><ul><li> Los recursos se implementan en un grupo de recursos y están administrados por el editor de la aplicación. El grupo de recursos está presente en la suscripción del consumidor, pero una identidad en el inquilino del editor tiene acceso al grupo de recursos.</li><li>Como editor, especifica el costo del soporte técnico continuo de la solución, y las transacciones se admiten a través del marketplace comercial.</li></ul>Use el tipo de plan de aplicación administrada cuando usted o su cliente requieran que un partner administre la solución, o cuando vayan a implementar una solución basada en suscripciones.</ul> |
| [**Contenedor de Azure**](marketplace-containers.md) | Use el tipo de oferta Contenedor de Azure cuando la solución sea una imagen de contenedor de Docker aprovisionada como un servicio de contenedor de Azure basado en Kubernetes. |
| [**Máquina virtual de Azure**](marketplace-virtual-machines.md) | Use el tipo de oferta de máquina virtual cuando implemente un dispositivo virtual para la suscripción asociada con el cliente. |
| [**Servicio de consultoría**](./plan-consulting-service-offer.md) | Los servicios de consultoría ayudan a conectar a los clientes con servicios que les permitan ser compatibles y extender el uso que hacen de Azure, Dynamics 365 o Power Suite.|
| [**Dynamics 365**](appsource-offer-publishing-guide.md) | Puede publicar ofertas de AppSource que se basan en Dynamics 365 Business central, Dynamics 365 Customer Engagement, Power Apps y aplicaciones de Finance and Operations o que los amplían.|
| [**Módulo IoT Edge**](iot-edge-module.md) | Los módulos de Azure IoT Edge son las unidades de cálculo más pequeñas que administra IoT Edge y pueden contener servicios de Microsoft (por ejemplo, Azure Stream Analytics), servicios de terceros o su propio código específico de la solución. |
| [**Servicio administrado**](./plan-managed-service-offer.md) | Puede crear ofertas de servicios administrados y administrar suscripciones o grupos de recursos delegados al cliente a través de [Azure Lighthouse](../lighthouse/overview.md).|
| [**Aplicación Power BI**<br/>**Microsoft 365**](appsource-offer-publishing-guide.md) | Puede publicar ofertas de AppSource que se basan en Power BI y Microsoft 365 o que los amplían.|
| [**Software como servicio**](plan-saas-offer.md) | Use el tipo de oferta de software como servicio (SaaS) para permitir que el cliente compre su solución técnica basada en SaaS como suscripción. Para más información sobre los requisitos de inicio de sesión único de las ofertas de SaaS, consulte [Azure AD y ofertas de SaaS comercializable en el marketplace comercial](azure-ad-saas.md). |


## <a name="next-steps"></a>Pasos siguientes

- Revise los requisitos de idoneidad en el artículo correspondientes para el tipo de oferta para finalizar la selección y la configuración de la oferta.
- Revise los patrones de publicación por tienda en línea para ver ejemplos de cómo la solución se asigna a un tipo de oferta y configuración.