---
title: Implementación de soluciones de Azure Sentinel | Microsoft Docs
description: En este artículo se muestra cómo los clientes pueden encontrar e implementar fácilmente herramientas de análisis de datos empaquetadas junto con conectores de datos.
services: sentinel
cloud: na
documentationcenter: na
author: yelevin
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/05/2021
ms.author: yelevin
ms.openlocfilehash: ce1620982aac833472102dce8a80b0c4195eb61d
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109811908"
---
# <a name="discover-and-deploy-azure-sentinel-solutions"></a>Detección e implementación de soluciones de Azure Sentinel

> [!IMPORTANT]
>
> La experiencia de las soluciones de Azure Sentinel está actualmente en **VERSIÓN PRELIMINAR**, al igual que todos los paquetes de soluciones individuales. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

Las soluciones de Azure Sentinel proporcionan detectabilidad en el producto, implementación en un solo paso y habilitación de escenarios de productos, dominios o verticales de un extremo a otro en Azure Sentinel. Esta experiencia cuenta con la tecnología [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace) para la detectabilidad, implementación y habilitación de soluciones, y del [Centro de partners de Microsoft](/partner-center/overview) para la creación y publicación de soluciones.

Las soluciones pueden constar de cualquiera de los componentes siguientes (o todos ellos):

- **Conectores de datos**, algunos con **analizadores** adjuntos
- **Libros**
- **Reglas de análisis**
- **Consultas de búsqueda**
- **Playbooks**

## <a name="find-your-solution"></a>Búsqueda de la solución

1. En el menú de navegación de Azure Sentinel, seleccione **Soluciones (versión preliminar)** .

1. La hoja **Soluciones** muestra una lista de soluciones en la que se pueden realizar búsquedas.

    :::image type="content" source="./media/sentinel-solutions-deploy/solutions-list.png" alt-text="Lista de soluciones":::

    - Si se desplaza hasta la parte inferior de la lista pero no encuentra lo que busca, haga clic en el vínculo **Cargar más** en la parte inferior para expandir la lista.

        :::image type="content" source="./media/sentinel-solutions-deploy/load-more.png" alt-text="Carga de más soluciones":::

1. Para restringir las opciones y encontrar la solución que quiere más fácilmente, escriba cualquier parte del nombre de la solución en el campo **Buscar** de la parte superior de la lista. (El motor de búsqueda solo reconocerá palabras enteras).

    :::image type="content" source="./media/sentinel-solutions-deploy/solutions-search-1.png" alt-text="Búsqueda de soluciones":::

1. Seleccione la solución deseada de la lista para implementarla. La página de detalles de la solución se abrirá en la pestaña **Información general**, que muestra información esencial e importante sobre la solución.

    :::image type="content" source="./media/sentinel-solutions-deploy/proofpoint-tap-solution.png" alt-text="Solución Proofpoint Tap":::

1. Puede ver otra información útil sobre la solución en las pestañas **Planes** e **Información de uso y soporte técnico**, y puede obtener las opiniones de otros clientes en la pestaña **Revisiones**.

## <a name="deploy-your-solution"></a>Implementación de la solución

1. Seleccione el botón **Crear** para iniciar el Asistente para la implementación de soluciones, que se abrirá en la pestaña **Aspectos básicos**.

    :::image type="content" source="./media/sentinel-solutions-deploy/wizard-basics.png" alt-text="pestaña de aspectos básicos del Asistente para la implementación":::

1. Escriba la suscripción, el grupo de recursos y el área de trabajo en la que desea implementar la solución. 

1. Haga clic en **Siguiente** para recorrer las pestañas restantes (correspondientes a los componentes incluidos en la solución), donde puede obtener información sobre cada uno de los componentes y, en algunos casos, configurarlos.

    > [!NOTE]
    > Las pestañas que se enumeran a continuación se corresponden con los componentes que ofrece la solución que se muestra en las capturas de pantalla adjuntas. Las diferentes soluciones pueden tener distintos tipos de componentes, por lo que es posible que no vea todas las mismas pestañas en todas las soluciones, y es posible que vea pestañas que no se muestran a continuación.

    1. Pestaña **Análisis** :::image type="content" source="./media/sentinel-solutions-deploy/wizard-analytics.png" alt-text="pestaña Análisis del Asistente para la implementación":::

    1. Pestaña **Libros** :::image type="content" source="./media/sentinel-solutions-deploy/wizard-workbooks.png" alt-text="pestaña Libros del Asistente para la implementación":::

    1. Pestaña **Cuadernos de estrategias**: aquí deberá escribir las credenciales válidas de Proofpoint TAP, para que el cuaderno de estrategias pueda autenticarse en su sistema Proofpoint para realizar las acciones de respuesta prescritas.
        :::image type="content" source="./media/sentinel-solutions-deploy/wizard-playbooks.png" alt-text="pestaña Cuadernos de estrategias del Asistente para implementación":::

1. Por último, en la pestaña **Revisar y crear**, espere el mensaje "Validación completada" y, a continuación, haga clic en **Crear** para implementar la solución. También puede seleccionar el vínculo **Descargar una plantilla para la automatización** para implementar la solución como código.

    :::image type="content" source="./media/sentinel-solutions-deploy/wizard-create.png" alt-text="pestaña Revisar y crear del Asistente para implementación":::

## <a name="next-steps"></a>Pasos siguientes

En este documento, ha aprendido sobre las soluciones de Azure Sentinel, así como a buscarlas e implementarlas.

- Obtenga más información sobre las [soluciones de Azure Sentinel](sentinel-solutions.md).
- Consulte el [catálogo completo de soluciones de Sentinel](sentinel-solutions-catalog.md).
