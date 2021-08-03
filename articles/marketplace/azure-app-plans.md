---
title: Creación de planes para una oferta de aplicación de Azure
description: Cree planes para una oferta de aplicación de Azure en el Centro de partners (Azure Marketplace).
author: aarathin
ms.author: aarathin
ms.reviewer: dannyevers
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: how-to
ms.date: 06/01/2021
ms.openlocfilehash: cd771baabad58cba4beca02f56e428730051aa96
ms.sourcegitcommit: bd65925eb409d0c516c48494c5b97960949aee05
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2021
ms.locfileid: "111542893"
---
# <a name="create-plans-for-an-azure-application-offer"></a>Creación de planes para una oferta de aplicación de Azure

Las ofertas vendidas mediante el marketplace comercial de Microsoft deben tener al menos un plan para publicar su oferta en el marketplace comercial. Puede crear una variedad de planes con diferentes opciones dentro de la misma oferta. Estos planes (denominados a veces SKU) pueden diferir en términos de tipo de plan (_plantilla de solución_ o _aplicación administrada_), monetización o público. Para obtener instrucciones generales sobre los planes, vea [Planes y precios de las ofertas del marketplace comercial](plans-pricing.md).

## <a name="create-a-plan"></a>Creación de un plan

1. Cerca de la parte superior de la pestaña **Información general del plan**, seleccione **+ Crear nuevo plan**.
1. En el cuadro de diálogo que aparece, en el cuadro **Id. de plan**, escriba un identificador de plan único. Este identificador será visible para los clientes en la dirección URL del producto. Use solo hasta 50 caracteres alfanuméricos en minúscula, guiones o caracteres de subrayado. No se puede modificar el identificador del plan después de seleccionar **Crear**.
1. En el cuadro **Nombre del plan**, escriba un nombre para este plan. Los clientes verán este nombre al decidir que plan van a seleccionar en su oferta. Use un máximo de 50 caracteres.
1. Seleccione **Crear**.

## <a name="define-the-plan-setup"></a>Definición de la configuración del plan

La pestaña **Configuración del plan** permite establecer el tipo de plan, si reutiliza la configuración técnica de otro plan y en qué regiones de Azure debe estar disponible el plan. Sus respuestas de esta pestaña afectarán a los campos mostrados en otras pestañas para este plan.

### <a name="select-the-plan-type"></a>Selección del tipo de plan

- En la lista **Tipo de plan**, seleccione **Plantilla de solución** o **Aplicación administrada**.

El cliente administra completamente los planes de **plantillas de soluciones**. Un plan de **aplicación administrada** permite que los anunciantes administren la aplicación en nombre del cliente. Para obtener más información sobre estos dos tipos de planes, vea [Tipos de planes](plan-azure-application-offer.md#types-of-plans).

#### <a name="to-re-use-a-technical-configuration"></a>Para reutilizar una configuración técnica:

1. Seleccione la casilla **This plan reuses the technical configuration from another plan of the same type** (Este plan reutiliza la configuración técnica de otro plan del mismo tipo).
1. En la lista que aparece, seleccione el plan base que desee.

> [!NOTE]
> Cuando reutilice paquetes de otro plan, toda la pestaña **Configuración técnica** desaparece de este plan. Los detalles de configuración técnica del otro plan, incluidas las actualizaciones que realice en el futuro, se usan también para este plan.

### <a name="select-azure-regions-clouds"></a>Selección de regiones de Azure (nubes)

El plan debe estar disponible al menos en una región de Azure. Una vez que el plan esté publicado y disponible en una región de Azure específica, no se puede quitar esa región de la oferta.

#### <a name="azure-global-region"></a>Región Azure global

La casilla **Azure global** está activada de forma predeterminada. Esto permite que el plan esté disponible para los clientes de todas las regiones globales de Azure que tengan la integración de marketplace comercial. En el caso de los planes de aplicaciones administradas, puede seleccionar en qué mercados desea que esté disponible el plan.

Para quitar su oferta de esta región, desactive la casilla **Azure global**.

#### <a name="azure-government-region"></a>Región de Azure Government

Esta región proporciona acceso controlado para los clientes de entidades tribales, locales, estatales o federales de Estados Unidos, así como para los asociados aptos para abastecerlas. Como publicador, es responsable de los controles de cumplimiento, las medidas de seguridad y los procedimientos recomendados. Azure Government usa redes y centros de datos aislados físicamente (ubicados solo en Estados Unidos).

Los servicios de Azure Government controlan datos que están sujetos a determinados reglamentos y requisitos gubernamentales. Por ejemplo, FedRAMP, NIST 800.171 (DIB), ITAR, IRS 1075, DoD L4 y CJIS. Para dar a conocer sus certificaciones de estos programas, puede proporcionar hasta 100 vínculos que las describan. Pueden ser vínculos a su anuncio en el programa directamente o a descripciones de su cumplimiento de estos en sus propios sitios web. Estos vínculos solo son visibles para los clientes de Azure Government.

##### <a name="to-select-the-azure-government-region"></a>Para seleccionar la región Azure Government

1. Seleccione la casilla **Azure Government**.
1. En **Certificaciones de Azure Government**, seleccione **+ Agregar certificación (máx. 100)** .
1. En los cuadros que aparecen, proporcione un nombre y un vínculo a una certificación.
1. Para agregar otra certificación, repita los pasos 2 y 3.

Seleccione **Guardar borrador** antes de pasar a la siguiente pestaña: Lista del plan.

## <a name="define-the-plan-listing"></a>Definición de la lista de planes

En la pestaña **Lista del plan** es donde se configuran los detalles de la lista del plan. Esta pestaña muestra información específica que muestra la diferencia entre los planes de la misma oferta. Puede definir el nombre, el resumen y la descripción del plan como desee que aparezcan en el marketplace comercial.

1. En el cuadro **Nombre del plan**, se muestra el nombre que proporcionó anteriormente para este plan. Puede cambiarlo en cualquier momento. Este nombre aparecerá en el marketplace comercial como el título del plan de software de la oferta, con un límite de 100 caracteres.
1. En el cuadro **Resumen del plan**, proporcione un breve resumen del plan, no de la oferta. Este resumen se limita a 100 caracteres.
1. En el cuadro **Descripción del plan**, explique lo que hace que este plan de software sea único y las diferencias con respecto a otros planes de software de su oferta. No describa la oferta, solo el plan. Esta descripción puede contener hasta 2000 caracteres.
1. Seleccione **Guardar borrador** antes de continuar.

## <a name="next-steps"></a>Pasos siguientes

Realice una de las siguientes acciones:

- [Configuración de un plan de plantilla de solución](azure-app-solution.md)
- [Configuración de un plan de una aplicación administrada](azure-app-managed.md)
