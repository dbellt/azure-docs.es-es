---
title: 'Tutorial: Expansión o reducción de clústeres en una nube privada'
description: En este tutorial, usará Azure Portal para escalar una nube privada de la versión preliminar de Azure VMware Solution.
ms.topic: tutorial
ms.date: 03/13/2021
ms.openlocfilehash: 23fa38f41a1effab212896e59118978202ef17a7
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108769298"
---
# <a name="tutorial-expand-or-shrink-clusters-in-a-private-cloud"></a>Tutorial: Expansión o reducción de clústeres en una nube privada

Para sacar el máximo partido de la experiencia de la nube privada de Azure VMware Solution, escale los clústeres y los hosts para que reflejen lo que necesita para las cargas de trabajo planeadas. Puede escalar los clústeres y los hosts en una nube privada, según sea necesario, para la carga de trabajo de la aplicación. Las limitaciones de rendimiento y disponibilidad de los servicios específicos deberán abordarse cada por caso. Los límites de hosts y clústeres se proporcionan en el artículo de [concepto de nube privada](concepts-private-clouds-clusters.md).

En este tutorial, usará Azure Portal para:

> [!div class="checklist"]
> * Agregar un clúster a una nube privada existente
> * Agregar nodos a un clúster existente

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará una nube privada existente. Si no ha creado una nube privada, use el tutorial para la [creación de una nube privada](tutorial-create-private-cloud.md) con el fin de crear una. 

## <a name="add-a-new-cluster"></a>Incorporación de un nuevo clúster

1. En la página de información general de una nube privada existente, en **Administrar**, seleccione **Scale private cloud** (Escalar la nube privada). A continuación, seleccione **+ Agregar un clúster**.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss2-select-add-cluster.png" alt-text="selección de Agregar un clúster" border="true":::

1. En la página **Add cluster** (Agregar clúster), use el control deslizante para seleccionar el número de hosts. Seleccione **Guardar**.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss3-configure-new-cluster.png" alt-text="En la página Add cluster (Agregar clúster), use el control deslizante para seleccionar el número de hosts. Seleccione Guardar." border="true":::

   Se iniciará la implementación del nuevo clúster.

## <a name="scale-a-cluster"></a>Escalado de un clúster 

1. En la página de información general de una nube privada existente, seleccione **Scale private cloud** (Escalar la nube privada) y seleccione el icono de lápiz para editar el clúster.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss4-select-scale-private-cloud-2.png" alt-text="Seleccionar Scale private cloud en información general" border="true":::

1. En la página **Edit Cluster** (Editar clúster), use el control deslizante para seleccionar el número de hosts. Seleccione **Guardar**.

   :::image type="content" source="./media/tutorial-scale-private-cloud/ss5-scale-cluster.png" alt-text="En la página Edit Cluster (Editar clúster), use el control deslizante para seleccionar el número de hosts. Seleccione Guardar." border="true":::

   Empiezan a agregarse hosts al clúster.

## <a name="next-steps"></a>Pasos siguientes

Si necesita otra nube privada de Azure VMware Solution, [créela](tutorial-create-private-cloud.md), con los mismos requisitos previos de red, clúster y límites de host.

<!-- LINKS - external-->

<!-- LINKS - internal -->
