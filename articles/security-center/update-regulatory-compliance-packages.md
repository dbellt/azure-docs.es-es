---
title: Uso del panel de cumplimiento normativo de Azure Security Center
description: Aprenda a agregar y quitar estándares normativos desde el panel de cumplimiento normativo de Security Center
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: c42d02e4-201d-4a95-8527-253af903a5c6
ms.service: security-center
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/04/2021
ms.author: memildin
ms.openlocfilehash: dab6b504fe026324251b7284fc3abdb52bb9911a
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107738961"
---
# <a name="customize-the-set-of-standards-in-your-regulatory-compliance-dashboard"></a>Personalización del conjunto de estándares en el panel de cumplimiento normativo

Azure Security Center compara continuamente la configuración de los recursos con los requisitos de los estándares del sector, las regulaciones y los bancos de pruebas. En el **panel de cumplimiento normativo** se proporciona información sobre su postura de cumplimiento en función de cómo cumple los requisitos de cumplimiento específicos.


## <a name="how-are-regulatory-compliance-standards-represented-in-security-center"></a>¿Cómo se representan los estándares de cumplimiento normativo en Security Center?

Los estándares del sector, los estándares normativos y las pruebas comparativas se representan en el panel de cumplimiento normativo de Security Center. Cada estándar es una iniciativa definida en Azure Policy.

Para ver los datos de cumplimiento asignados como evaluaciones en el panel, agregue un estándar de cumplimiento a la suscripción o al grupo de administración desde la página **Directiva de seguridad**. Para obtener más información sobre Azure Policy y las iniciativas, consulte [Uso de directivas de seguridad](tutorial-security-policy.md).

Cuando haya asignado un estándar o una prueba comparativa al ámbito seleccionado, el estándar aparece en el panel de cumplimiento normativo con todos los datos de cumplimiento asociados asignados como evaluaciones. También puede descargar informes de resumen para cualquiera de los estándares que se hayan asignado.

Microsoft realiza un seguimiento de los estándares normativos y mejora automáticamente su cobertura en algunos de los paquetes a lo largo del tiempo. Cuando Microsoft publica contenido nuevo para la iniciativa, aparece automáticamente en el panel a medida que se asignan directivas a los controles en el estándar.


## <a name="what-regulatory-compliance-standards-are-available-in-security-center"></a>¿Qué estándares de cumplimiento normativo están disponibles en Security Center?

De manera predeterminada, cada suscripción tiene la **Azure Security Benchmark** asignado. Son las directrices específicas de Azure creadas por Microsoft para ofrecer los procedimientos recomendados de seguridad y cumplimiento basados en marcos de cumplimiento comunes. [Mas información sobre Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/introduction).

También puede agregar estándares como:

- NIST SP 800-53 R4
- SWIFT CSP CSCF-v2020
- UK Official y UK NHS
- Canada Federal PBMM
- Azure CIS 1.1.0

Los estándares se agregan al panel a medida que están disponibles.


## <a name="add-a-regulatory-standard-to-your-dashboard"></a>Adición de un estándar de cumplimiento al panel

En los pasos siguientes se explica cómo agregar un paquete para supervisar el cumplimiento de uno de los estándares normativos admitidas.

> [!NOTE]
> Para agregar estándares al panel, la suscripción debe tener habilitado Azure Defender. Además, solo los usuarios que son propietario o colaborador de directivas tienen los permisos necesarios para agregar estándares de cumplimiento. 

1. En la barra lateral de Security Center, seleccione **Cumplimiento normativo** para abrir el panel de cumplimiento normativo. Aquí puede ver los estándares de cumplimiento asignados actualmente a las suscripciones que están seleccionadas.   

1. En la parte superior de la página, seleccione **Administrar directivas de cumplimiento**. Se mostrará la página Administración de directivas.

1. Seleccione la suscripción o el grupo de administración para el que desea administrar la postura de cumplimiento normativo. 

    > [!TIP]
    > Se recomienda seleccionar el ámbito más alto para el que se aplica el estándar y así poder agregar y realizar un seguimiento de los datos de cumplimiento para todos los recursos anidados. 

1. Para agregar los estándares pertinentes para su organización, haga clic en **Agregar más estándares**. 

1. En la página **Adición de estándares de cumplimiento normativo**, puede buscar cualquiera de los estándares disponibles, incluidos:

    - **NIST SP 800-53 R4**
    - **NIST SP 800 171 R2**
    - **SWIFT CSP CSCF v2020**
    - **UKO y UK NHS**
    - **Canada Federal PBMM**
    - **HIPAA/HITRUST**
    - **Azure CIS 1.1.0**
    
    ![Adición de estándares normativos al panel de cumplimiento normativo de Azure Security Center](./media/update-regulatory-compliance-packages/dynamic-regulatory-compliance-additional-standards.png)

1. Seleccione **Agregar** y escriba todos los detalles necesarios para la iniciativa específica, como el ámbito, los parámetros y la corrección.

1. En la barra lateral de Security Center, vuelva a seleccionar **Cumplimiento normativo** para volver al panel de cumplimiento normativo.

    El nuevo estándar aparece ahora en la lista de estándares normativos y del sector. 

    > [!NOTE]
    > Asimismo, un estándar recién agregado en el panel de cumplimiento puede tardar unas horas en aparecer en el mismo.

    :::image type="content" source="./media/security-center-compliance-dashboard/compliance-dashboard.png" alt-text="Panel de cumplimiento normativo" lightbox="./media/security-center-compliance-dashboard/compliance-dashboard.png":::

## <a name="remove-a-standard-from-your-dashboard"></a>Eliminación de un estándar del panel

Si alguno de los estándares normativos proporcionados no es relevante para su organización, quitarlo de la interfaz de usuario es un proceso sencillo. Esto le permite personalizar aún más el panel de cumplimiento normativo y centrarse solo en los estándares que se apliquen en su caso.

Para quitar un estándar:

1. En el menú de Security Center, seleccione **Directiva de seguridad**.

1. Seleccione la suscripción pertinente de la que desea quitar un estándar.

    > [!NOTE]
    > Puede quitar un estándar de una suscripción, pero no de un grupo de administración. 

    Se abrirá la página de directivas de seguridad. En la suscripción seleccionada, se muestra la directiva predeterminada, el sector y los estándares normativos, así como las iniciativas personalizadas que haya creado.

    :::image type="content" source="./media/update-regulatory-compliance-packages/remove-standard.png" alt-text="Eliminación de un estándar normativo del panel de cumplimiento normativo en Azure Security Center":::

1. Para el estándar que desea quitar, seleccione **Deshabilitar**. Se abrirá una ventana de confirmación.

    :::image type="content" source="./media/update-regulatory-compliance-packages/remove-standard-confirm.png" alt-text="Confirme que desea quitar el estándar normativo que ha seleccionado":::.

1. Seleccione **Sí**. Se quitará el estándar. 


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido cómo **agregar estándares de cumplimiento** para supervisar el cumplimiento de estándares normativos y del sector.

Para obtener material relacionado, vea las páginas siguientes:

- [Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/introduction)
- [Panel de cumplimiento normativo de Security Center](security-center-compliance-dashboard.md): obtenga información sobre cómo hacer seguimiento de los datos de cumplimiento normativo y cómo exportarlos con Security Center y herramientas externas.
- [Uso de las directivas de seguridad](tutorial-security-policy.md)