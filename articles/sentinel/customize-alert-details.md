---
title: Personalización de los detalles personalizados de las alertas de Azure Sentinel | Microsoft Docs
description: Personalice cómo se denominan y describen las alertas, junto con su gravedad y tácticas asignadas, en función del contenido de las alertas.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/10/2021
ms.author: yelevin
ms.openlocfilehash: 725711ed7d49b21303769ccd6959fc8b2083e9fa
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112063786"
---
# <a name="customize-alert-details-in-azure-sentinel"></a>Personalización de los detalles de las alertas de Azure Sentinel | Microsoft Docs 

> [!IMPORTANT]
>
> - La característica de detalles de la alerta está en **VERSIÓN PRELIMINAR**. Consulte [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) para conocer los términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

## <a name="introduction"></a>Introducción

Al definir un nombre y una descripción para las reglas de análisis programadas y asignarles gravedades y tácticas de MITRE ATT&CK, todas las alertas generadas por una regla determinada (y todos los incidentes creados como resultado) se mostrarán con el mismo nombre, descripción, y así sucesivamente, sin tener en cuenta el contenido concreto de una instancia específica de la alerta.

Con la característica de **detalles de alerta**, puede adaptar la apariencia de una alerta a su contenido. Aquí puede seleccionar parámetros de la alerta que se pueden representar en el nombre o la descripción de cada instancia de la alerta, o que pueden contener las tácticas y la gravedad asignadas a esa instancia de la alerta. Si el parámetro seleccionado no tiene ningún valor (o tiene un valor no válido en el caso de las tácticas y la gravedad), los detalles de la alerta se revertirán a los valores predeterminados especificados en la primera página del asistente.

El procedimiento que se detalla a continuación forma parte del Asistente para crear reglas de análisis. Aquí se trata de forma independiente para abordar el escenario de agregar o cambiar los detalles de la alerta en una regla de análisis existente.

## <a name="how-to-customize-alert-details"></a>Personalización de los detalles de la alerta

1. En el menú de navegación de Azure Sentinel, seleccione **Análisis**.

1. Seleccione una regla de consulta programada y haga clic en **Editar**. O bien, para crear una nueva regla, haga clic en **Crear > Regla de consulta programada** en la parte superior de la pantalla.

1. Haga clic en la pestaña **Establecer la lógica de la regla**.

1. En la sección **Alert enrichment (Preview)** (Enriquecimiento de alertas [versión preliminar]), expanda **Detalles de la alerta**.

    :::image type="content" source="media/customize-alert-details/alert-enrichment.png" alt-text="Personalización de detalles de la alerta":::

1. En la sección **Detalles de la alerta** ahora expandida, agregue texto libre que incluya los parámetros correspondientes a los detalles que quiere mostrar en la alerta:

    1. En el campo **Formato del nombre de la alerta**, escriba el texto que quiere que aparezca como nombre de la alerta (el texto de la alerta) e incluya, entre llaves, los parámetros que quiera que formen parte del texto de la alerta.

        Ejemplo: `Alert from {{ProviderName}}: {{AccountName}} failed to log on to computer {{ComputerName}} with IP address {{IPAddress}}.`

    1. Haga lo mismo con el campo **Formato de la descripción de la alerta**.
    
    1. Use los campos **Columna de táctica** y **Columna de gravedad** solo si los resultados de la consulta contienen columnas con esta información. Para cada uno de ellos, elija la columna que contiene la información correspondiente.

    Si cambia de opinión, o si comete un error, puede quitar un detalle de la alerta haciendo clic en el icono de la papelera junto a los campos **Tactic/Severity Column** (Columna de táctica o gravedad) o eliminar el texto libre de los campos **Alert Name/Description Format** (Nombre de alerta/Formato de la descripción).

1. Cuando haya terminado de personalizar los detalles de la alerta, continúe con la pestaña siguiente del asistente. Si está editando una regla existente, haga clic en la pestaña **Revisar y crear**. Una vez que la validación de la regla se haya realizado correctamente, haga clic en **Guardar**.

## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido a personalizar los detalles de la alertas en las reglas de análisis de Azure Sentinel. Para más información sobre Azure Sentinel, consulte los siguientes artículos:
- Obtenga una visión completa de las [reglas de análisis de consultas programadas](tutorial-detect-threats-custom.md).
- Obtenga más información sobre las [entidades en Azure Sentinel](entities-in-azure-sentinel.md).
