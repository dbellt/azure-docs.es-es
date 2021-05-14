---
title: Información general sobre los conjuntos de disponibilidad
description: Obtener más información sobre los conjuntos de disponibilidad en Azure
author: mimckitt
ms.author: mimckitt
ms.service: virtual-machines
ms.topic: conceptual
ms.date: 02/18/2021
ms.openlocfilehash: b4e9f106354915fe40a4bcf9b60bc35443345202
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530590"
---
# <a name="availability-sets-overview"></a>Información general sobre los conjuntos de disponibilidad

En este artículo proporciona una visión general de las características de disponibilidad de las máquinas virtuales (VM) de Azure.

## <a name="what-is-an-availability-set"></a>¿Qué es un conjunto de disponibilidad? 

Un conjunto de disponibilidad es una agrupación lógica de máquinas virtuales que permite a Azure conocer cómo se crea su aplicación para proporcionar redundancia y disponibilidad. Se recomienda la creación de dos, o más, máquinas virtuales en un conjunto de disponibilidad no solo para proporcionar una aplicación de alta disponibilidad sino también para cumplir el [99,95 % del Acuerdo de Nivel de Servicio de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/). No hay ningún costo asociado con el conjunto de disponibilidad propiamente dicho, solo se paga por cada instancia de máquina virtual que cree.

## <a name="how-do-availability-sets-work"></a>¿Cómo funcionan los conjuntos de disponibilidad?
La plataforma Azure subyacente asigna a cada máquina virtual del conjunto de disponibilidad un **dominio de actualización** y un **dominio de error**. Cada conjunto de disponibilidad se puede configurar con hasta tres dominios de error y 20 dominios de actualización. Los dominios de actualización indican grupos de máquinas virtuales y hardware físico subyacente que se pueden reiniciar al mismo tiempo. Cuando se configuran más de cinco máquinas virtuales en un único conjunto de disponibilidad, la sexta máquina virtual se coloca en el mismo dominio de actualización que la primera, la séptima en el mismo que la segunda, y así sucesivamente. Es posible que el orden en que se reinician los dominios de actualización no siga una secuencia durante un mantenimiento planeado, pero se reinician de uno en uno. Un dominio de actualización reiniciado tiene 30 minutos para recuperar antes de que el mantenimiento se inicie en un dominio de actualización diferente.

Los dominios de error definen un grupo de máquinas virtuales que comparten un origen de alimentación y un interruptor de red comunes. De manera predeterminada, las máquinas virtuales configuradas dentro de su conjunto de disponibilidad se separan en hasta tres dominios de error. Aunque colocar las máquinas virtuales en un conjunto de disponibilidad no protege su aplicación contra errores del sistema operativo ni específicos de aplicaciones, limita el impacto de posibles errores de hardware físico, interrupciones de red o cortes de alimentación.

:::image type="content" source="./media/virtual-machines-common-manage-availability/ud-fd-configuration.png" alt-text="Diagrama que muestra varios clústeres de proceso divididos en dominios de error y, dentro de esos dominios de error, hay varios dominios de actualización":::

Las máquinas virtuales también están alineadas con dominios de error de disco. Esta alineación garantiza que todos los discos administrados conectados a una máquina virtual se encuentren en los mismos dominios de error. 

Solo se pueden crear máquinas virtuales con discos administrados en un conjunto de disponibilidad administrada. El número de dominios de error de disco administrado varía según la región: dos o tres dominios de error de disco administrado por región. 

:::image type="content" source="./media/virtual-machines-common-manage-availability/md-fd-updated.png" alt-text="Diagrama que muestra cómo se alinean los dominios de error de los discos y las máquinas virtuales.":::

## <a name="next-steps"></a>Pasos siguientes
Ya puede empezar a usar estas características de disponibilidad y redundancia para crear un entorno de Azure. Para información sobre los procedimientos recomendados, consulte [Lista de comprobación de disponibilidad](/azure/architecture/checklist/resiliency-per-service).

