---
title: Integración empresarial B2B
description: Aprenda a crear flujos de trabajo B2B para la integración empresarial mediante Azure Logic Apps y Enterprise Integration Pack
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, logicappspm
ms.topic: overview
ms.date: 03/24/2021
ms.openlocfilehash: 09d253aae4c054db5efdc252f62986044e1d366b
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107771864"
---
# <a name="b2b-enterprise-integration-solutions-with-azure-logic-apps-and-enterprise-integration-pack"></a>Soluciones de integración empresarial B2B con Azure Logic Apps y Enterprise Integration Pack

Si desea tener soluciones de negocio a negocio (B2B) y comunicación directa entre organizaciones, puede crear flujos de trabajo de integración empresarial escalable automatizados mediante el uso de Enterprise Integration Pack (EIP) con [Azure Logic Apps](../logic-apps/logic-apps-overview.md). Aunque las organizaciones usan distintos protocolos y formatos, pueden intercambiar mensajes electrónicamente. EIP transforma distintos formato en un formato que los sistemas de las organizaciones pueden procesar y admiten protocolos estándar del sector, como [AS2](../logic-apps/logic-apps-enterprise-integration-as2.md), [X12](logic-apps-enterprise-integration-x12.md) y [EDIFACT](../logic-apps/logic-apps-enterprise-integration-edifact.md). La seguridad de los mensajes también puede mejorar mediante el cifrado y las firmas digitales. EIP admite estos [conectores de integración empresarial](../connectors/managed.md#enterprise-connectors) y estos estándares del sector:

* Intercambio electrónico de datos (EDI)
* Enterprise Application Integration (EAI)

Si conoce Microsoft BizTalk Server o Azure BizTalk Services, EIP se rige por conceptos similares, lo que facilita el uso de las características. Sin embargo, una diferencia importante es que la arquitectura de EIP se basa en utiliza las "cuentas de integración" para simplificar la administración y el almacenamiento de los artefactos empleados en las comunicaciones B2B. Estas cuentas son contenedores basados en la nube que almacenan todos sus artefactos, como asociados, contratos, esquemas, mapas, y certificados. 

## <a name="why-use-the-enterprise-integration-pack"></a>Motivos para usar Enterprise Integration Pack

* Con EIP, puede almacenar todos los artefactos en un solo lugar, su cuenta de integración.

* Puede crear flujos de trabajo B2B e integrarlos en aplicaciones de software como servicio (SaaS) de terceros, aplicaciones locales y aplicaciones personalizadas mediante Azure Logic Apps y sus conectores.

* Puede crear código personalizado para las aplicaciones lógicas con Azure Functions.

## <a name="how-do-i-get-started"></a>¿Cómo empiezo?

Para poder empezar a crear flujos de trabajo de aplicaciones lógicas B2B con la EIP, necesita estos elementos:

* Suscripción a Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) con los artefactos que desea usar

* Para crear mapas y esquemas, puede usar [Herramientas de integración empresarial para Visual Studio 2015 2.0 de Microsoft Azure Logic Apps](https://aka.ms/vsmapsandschemas) y Visual Studio 2015.

Después de crear una cuenta de integración y agregar los artefactos, puede empezar a crear flujos de trabajo B2B con estos artefactos mediante la creación de una aplicación lógica en Azure Portal. Si no conoce las aplicaciones lógicas, pruebe a [crear una aplicación lógica básica](../logic-apps/quickstart-create-first-logic-app-workflow.md). Para trabajar con estos artefactos, asegúrese de que vincular antes la cuenta de integración a la aplicación lógica. Después, la aplicación lógica podrá acceder a la cuenta de integración. También puede crear, administrar e implementar aplicaciones lógica mediante Visual Studio o [PowerShell.](/powershell/module/az.logicapp)

Estos son los pasos de alto nivel para empezar a crear aplicaciones lógicas B2B:

![Requisitos previos para la creación de aplicaciones lógicas B2B](./media/logic-apps-enterprise-integration-overview/overview.png)  

## <a name="try-now"></a>Probar ahora

[Implementación de una aplicación lógica de ejemplo completamente operativa que envíe y reciba mensajes AS2](https://github.com/Azure/azure-quickstart-templates/tree/master/201-logic-app-as2-send-receive)

## <a name="next-steps"></a>Pasos siguientes

* [Crear socios comerciales](logic-apps-enterprise-integration-partners.md)
* [Crear acuerdos](../logic-apps/logic-apps-enterprise-integration-agreements.md)
* [Incorporación de esquemas](logic-apps-enterprise-integration-schemas.md)
* [Incorporación de asignaciones](../logic-apps/logic-apps-enterprise-integration-maps.md)
* [Migración desde BizTalk Services](../logic-apps/logic-apps-move-from-mabs.md)
