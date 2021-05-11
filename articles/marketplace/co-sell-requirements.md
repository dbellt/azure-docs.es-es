---
title: Requisitos de venta conjunta | Azure Marketplace
description: Más información sobre los requisitos que una oferta del marketplace comercial de Microsoft debe cumplir para ser apta para el estado Listo para la venta conjunta o Venta conjunta incentivada.
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
author: vamahtan
ms.author: vamahtan
ms.reviewer: stmummer
ms.date: 04/16/2021
ms.openlocfilehash: 7a5e4375ebe289bb5ae012f94ba9dd050563bbbb
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209866"
---
# <a name="co-sell-requirements"></a>Requisitos de venta conjunta

En este artículo se proporcionan los requisitos para los distintos niveles de estado de la venta conjunta. Para obtener la lista más reciente de tipos de oferta que admiten la venta conjunta, consulte [Configuración de la venta conjunta para una oferta del marketplace comercial](co-sell-configure.md). Para obtener información general sobre la venta conjunta, consulte [Información general sobre la venta conjunta de los equipos de ventas de Microsoft y los asociados](co-sell-overview.md).

En esta tabla se muestran todos los posibles estados de la venta conjunta:

| Status | Comentario |
| ------------ | ------------- |
| No listo para la venta conjunta | No se cumplen los [requisitos mínimos para el estado Listo para la venta conjunta](#requirements-for-co-sell-ready-status). |
| Listo para la venta conjunta | Se cumplen todos los [requisitos para el estado Listo para la venta conjunta](#requirements-for-co-sell-ready-status). |
| Venta conjunta con incentivos de IP de Azure | Se cumplen los requisitos de venta conjunta, además de [estos requisitos adicionales](#requirements-for-azure-ip-co-sell-incentivized-status). |
| Venta conjunta con incentivos de Business Applications | Este estado se aplica a las ofertas de Dynamics 365 y Power Apps del [Programa Connect para ISV de Microsoft Business Applications](business-applications-isv-program.md) e indica que se cumplen todos los [requisitos de este estado](#requirements-for-business-applications-co-sell-incentivized-status). |
|||

## <a name="requirements-for-co-sell-ready-status"></a>Requisitos del estado Listo para la venta conjunta

> [!NOTE]
> Cualquier oferta publicada mediante el programa para desarrolladores del marketplace comercial del Centro de partners es válida para alcanzar el estado preparado para la venta cruzada, siempre que se cumplan los requisitos. Las aplicaciones y complementos de Office no son aptos (por ejemplo, Teams, soluciones de SharePoint, Outlook, Excel)

Para que una oferta obtenga el estado Listo para la venta conjunta, debe cumplir los siguientes requisitos:

**Todos los partners**:

- Deben tener un identificador de MPN y una [cuenta de marketplace comercial activa en el centro de Partners](create-account.md).
- Asegúrese de que tiene un [perfil de negocio](/partner-center/create-a-marketing-profile) completo en el Centro de partners. Como partner de Microsoft calificado, puede crear un perfil de negocio para presentar su negocio a los clientes que buscan sus soluciones únicas y la experiencia para satisfacer sus necesidades empresariales, lo cual luego dará como resultado la obtención de [referencias](/partner-center/referrals) para su negocio.
- Complete la pestaña **Venta conjunta con Microsoft** y publique la oferta en el marketplace comercial.
- Proporcione un contacto de ventas para cada zona geográfica que cumpla los requisitos de venta conjunta y la lista de materiales necesaria.

**Partners de servicios**:

- En el caso de las ofertas del tipo _Solución de servicio_, debe tener una competencia Gold activa en cualquier área de competencias.

**Fabricantes independientes de software para aplicaciones empresariales**:

- Las soluciones Dynamics 365 & PowerApps (excepto Dynamics 365 Business Central) requieren la inscripción a Connect para ISV.

### <a name="complete-the-co-sell-with-microsoft-tab"></a>Completar la pestaña Venta conjunta con Microsoft

Al publicar o actualizar la oferta, proporcione toda la información necesaria en la pestaña **Venta conjunta con Microsoft** como se indica en [Configuración de la venta conjunta para una oferta del marketplace comercial](./co-sell-configure.md). Esto incluye proporcionar los siguientes documentos:

- Página de descripción de la solución u oferta
- Presentación de la solución u oferta

Se proporcionan plantillas para ayudarle a crear estos documentos. Si desea conocer más detalles sobre la información obligatoria y opcional de la pestaña Venta conjunta con Microsoft, consulte [Configuración de la venta conjunta para una oferta del marketplace comercial](./co-sell-configure.md).

### <a name="publish-your-offer-live"></a>Publicación de la oferta

Para participar en el estado Listo para la venta conjunta, su oferta o solución debe publicarse en directo al menos en una de las tiendas de marketplace comercial en línea: Azure Marketplace o Microsoft AppSource. Para más información sobre la publicación de ofertas en el marketplace comercial, consulte la [Guía de publicación por tipo de oferta](publisher-guide-by-offer-type.md). Si no ha publicado anteriormente ninguna oferta en el marketplace comercial, asegúrese de que tiene una [cuenta en el marketplace comercial](create-account.md).

## <a name="requirements-for-azure-ip-co-sell-incentivized-status"></a>Requisitos del estado de incentivos de venta conjunta de direcciones IP de Azure

El estado Venta conjunta incentivada de Azure IP se aplica a los siguientes tipos de ofertas:

- Aplicación de Azure
- Contenedor de Azure
- Máquina virtual de Azure
- Módulo IoT Edge
- Software como servicio (SaaS)

Después de alcanzar el estado Listo para la venta conjunta, hay tres requisitos adicionales para lograr el estado Venta conjunta incentivada de Azure IP:

Requisito 1, lograr lo siguiente:

- En el _nivel de organización_, alcanzar como mínimo el umbral de 100 000 USD de ingresos por consumo de Azure durante los últimos 12 meses. Este umbral se puede conseguir a través de una combinación de soluciones de Azure. Si la oferta es comercializable en el marketplace comercial, puede cumplir este requisito si alcanza un umbral de ingresos facturados de 100 000 USD.

Requisito 2: aprobar la validación técnica de Microsoft para una solución basada en Azure:
- La validación técnica debe confirmar que más del 50% de la infraestructura de la oferta usa código IP repetible en Azure. Tenga en cuenta que las máquinas virtuales de Azure y las soluciones de Aplicación de Azure comercializables en el marketplace comercial deben cumplir este requisito de forma predeterminada.

Requisito 3: proporcionar un diagrama de la arquitectura de referencia:
- Cargue un diagrama de la arquitectura de referencia con los documentos sobre venta conjunta en el Centro de partners para su revisión. Para obtener instrucciones sobre cómo crear este diagrama, consulte [Diagrama de arquitectura de referencia](reference-architecture-diagram.md). Para más información sobre cómo cargar el diagrama, consulte [Configuración de la venta conjunta para una oferta del marketplace comercial](./co-sell-configure.md).

## <a name="requirements-for-business-applications-co-sell-incentivized-status"></a>Requisitos para el estado incentivado de venta conjunta para Business Applications

Este estado se aplica a las soluciones basadas en IP integradas en Dynamics 365 o Power Apps (excepto Dynamics 365 Business Central) que están inscritas en el programa Connect para ISV. Sin embargo, las ofertas también deben completar los requisitos para el estado de preparación para la venta conjunta (descrito anteriormente) para que los vendedores de Microsoft puedan vender la oferta con usted.

## <a name="next-steps"></a>Pasos siguientes

- [Configuración de la venta conjunta para una oferta del marketplace comercial](./co-sell-configure.md)