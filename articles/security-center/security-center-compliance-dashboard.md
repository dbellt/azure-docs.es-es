---
title: 'Tutorial: Comprobaciones de cumplimiento normativo: Azure Security Center'
description: 'Tutorial: Aprenda a mejorar el cumplimiento de reglamentaciones mediante Azure Security Center.'
services: security-center
documentationcenter: na
author: memildin
manager: rkarlin
ms.assetid: 5f50c4dc-ea42-418d-9ea8-158ffeb93706
ms.service: security-center
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 02/10/2021
ms.author: memildin
ms.openlocfilehash: 284a7f532ed918397fe1cfcf3458bbc6fb0bdd32
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107739015"
---
# <a name="tutorial-improve-your-regulatory-compliance"></a>Tutorial: Mejora del cumplimiento normativo

Azure Security Center resulta de gran ayuda para simplificar el proceso necesario para cumplir los requisitos de cumplimiento normativo, para lo que se usa el **panel de cumplimiento normativo.** . 

Security Center evalúa continuamente el entorno de nube híbrida para analizar los factores de riesgo con arreglo a los controles y los procedimientos recomendados establecidos en los estándares que se aplican a las suscripciones. El panel refleja el estado de cumplimiento con respecto a estos estándares. 

Cuando habilite Security Center en una suscripción de Azure, se asignará automáticamente a esta suscripción el estándar [Azure Security Benchmark](https://docs.microsoft.com/security/benchmark/azure/introduction). Este punto de referencia ampliamente respetado está basado en los controles del [Centro de seguridad de Internet (CIS)](https://www.cisecurity.org/benchmark/azure/) y del [National Institute of Standards and Technology (NIST)](https://www.nist.gov/) y hace hincapié en la seguridad centrada en la nube.

En el panel de cumplimiento normativo se indica el estado de todas estas valoraciones realizadas en el entorno conforme a las normativas y los estándares elegidos. Al actuar sobre las recomendaciones y reducir los factores de riesgo en su entorno, su estado de cumplimiento normativo mejora.

En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Evaluar su cumplimiento normativo mediante el panel de cumplimiento normativo.
> * Mejorar su estado de cumplimiento normativo mediante la realización de acciones sobre las recomendaciones
> * Configurar alertas sobre los cambios en la estado de cumplimiento
> * Exportar los datos de cumplimiento como una secuencia continua y como instantáneas semanales

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>Prerrequisitos

Para recorrer las características descritas en este tutorial:

- [Azure Defender](azure-defender.md) debe estar habilitado. Puede probar Azure Defender gratis durante 30 días.
- Debe haber iniciado sesión con una cuenta que tenga acceso de lectura a los datos de cumplimiento de directivas (**Lector de seguridad** no es suficiente). El rol **Lector global** para la suscripción funcionará. Como mínimo, necesitará tener asignados los roles **Colaborador de directivas de recursos** y **Administrador de seguridad**.

##  <a name="assess-your-regulatory-compliance"></a>Mejora del cumplimiento de reglamentaciones

El panel de cumplimiento normativo muestra los estándares normativos seleccionados, con todos sus requisitos. Los requisitos compatibles se asignarán a las evaluaciones de seguridad correspondientes. El estado de estas evaluaciones refleja el estado de cumplimiento con respecto a los estándares.

Utilice el panel de cumplimiento normativo para centrarse en las deficiencias relativas al cumplimiento de las normativas y estándares elegidos. Esta vista focalizada también le permite supervisar continuamente su puntuación de cumplimiento a lo largo del tiempo para entornos híbrido y de nube dinámicos.

1. En el menú de Security Center, seleccione **Cumplimiento normativo**.

    En la parte superior de la pantalla hay un panel con información general sobre su estado de cumplimiento con respecto a las normativas compatibles. Allí se indica la puntuación global de cumplimiento y el número de valoraciones aprobadas y suspendidas asociadas a cada estándar.

    :::image type="content" source="./media/security-center-compliance-dashboard/compliance-dashboard.png" alt-text="Panel de cumplimiento normativo" lightbox="./media/security-center-compliance-dashboard/compliance-dashboard.png":::

1. Seleccione la pestaña de la norma que le interese (1). Verá las suscripciones en las que se aplica la norma (2) y la lista de todos los controles de esa norma (3). Para saber qué controles se pueden aplicar, puede ver los detalles de las evaluaciones superadas y no superadas asociadas a ese control (4) y el número de recursos afectados (5). Algunos controles aparecen atenuados, no tienen valoraciones de Security Center asociadas. Compruebe sus requisitos y evalúelos en el entorno. Es posible que algunos de ellos estén relacionados con los procesos y no sean técnicos.

    :::image type="content" source="./media/security-center-compliance-dashboard/compliance-drilldown.png" alt-text="Exploración de los detalles de cumplimiento con una norma específica":::

1. Para generar un informe PDF con un resumen de su estado de cumplimiento actual en relación con un estándar concreto, seleccione **Descargar informe**.

    El informe proporciona un resumen general sobre el estado de cumplimiento del estándar seleccionado en función de los datos de las evaluaciones de Security Center. El informe se organiza según los controles de ese estándar determinado. El informe se puede compartir con las partes interesadas competentes y puede proporcionar evidencia a los auditores internos y externos.

    :::image type="content" source="./media/security-center-compliance-dashboard/download-report.png" alt-text="Descarga del informe de cumplimiento":::

## <a name="improve-your-compliance-posture"></a>Mejora de su estado de cumplimiento de la norma

El panel contiene información que le ayudará a mejorar el cumplimiento normativo y le permitirá resolver las recomendaciones directamente en él.

1.  Seleccione cualquiera de las evaluaciones no superadas que aparecen en el panel para ver los detalles de dicha recomendación. Cada recomendación incluye un conjunto de pasos de corrección para resolver el problema.

1.  Seleccione un recurso concreto para ver más detalles y resolver la recomendación relacionada. <br>Por ejemplo, en el estándar **Azure CIS 1.1.0**, seleccione la recomendación **El cifrado de disco se debe aplicar en las máquinas virtuales**.

    :::image type="content" source="./media/security-center-compliance-dashboard/sample-recommendation.png" alt-text="Al seleccionar una recomendación de una norma se va directamente a la página de detalles de la recomendación.":::

1. En este ejemplo, si selecciona **Realizar acción** en la página de detalles de la recomendación, accederá Azure Portal, a las páginas de la máquina virtual de Azure, donde podrá abrir la pestaña **Seguridad** y habilitar el cifrado:

    :::image type="content" source="./media/security-center-compliance-dashboard/encrypting-vm-disks.png" alt-text="Botón Realizar acción en la página de detalles de la recomendación que conduce a las opciones de corrección":::

    Para más información sobre cómo aplicar las recomendaciones, consulte [Implementación de recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md).

1.  Una vez realizadas las acciones necesarias para resolver las recomendaciones, podrá ver el resultado en el informe del panel de cumplimiento, ya que la puntuación de cumplimiento mejora.

    > [!NOTE]
    > Las valoraciones se ejecutan aproximadamente cada 12 horas, por lo que el efecto sobre los datos de cumplimiento solo se constatará tras la ejecución siguiente de la valoración en cuestión.


## <a name="export-your-compliance-status-data"></a>Exportación de los datos de estado de cumplimiento

Si desea hacer un seguimiento del estado de cumplimiento con otras herramientas de supervisión de su entorno, Security Center dispone de un mecanismo de exportación que facilita esta tarea. Configure la **exportación continua** para enviar una selección de los datos a un centro de eventos de Azure o un área de trabajo de Log Analytics.

Utilice los datos de exportación continua con un centro de eventos de Azure o un área de trabajo de Log Analytics:

- Exporte todos los datos de cumplimiento normativo en un **flujo continuo**:

    :::image type="content" source="media/security-center-compliance-dashboard/export-compliance-data-stream.png" alt-text="Exportar ininterrumpidamente un flujo de datos de cumplimiento normativo" lightbox="media/security-center-compliance-dashboard/export-compliance-data-stream.png":::

- Exporte **instantáneas semanales** de los datos de cumplimiento normativo:

    :::image type="content" source="media/security-center-compliance-dashboard/export-compliance-data-snapshot.png" alt-text="Exportar ininterrumpidamente una instantánea semanal de los datos de cumplimiento normativo" lightbox="media/security-center-compliance-dashboard/export-compliance-data-snapshot.png":::

También puede exportar un **informe PDF/CSV** de los datos de cumplimiento directamente desde el panel:

:::image type="content" source="media/security-center-compliance-dashboard/export-compliance-data-report.png" alt-text="Exportar los datos de cumplimiento normativo como un informe PDF o CSV" lightbox="media/security-center-compliance-dashboard/export-compliance-data-report.png":::

Para más información, consulte este artículo sobre la [exportación continua de datos de Security Center](continuous-export.md).


## <a name="run-workflow-automations-when-there-are-changes-to-your-compliance"></a>Ejecución de automatizaciones de flujos de trabajo cuando se producen cambios en el cumplimiento

La característica de automatización de flujos de trabajo de Security Center puede desencadenar Logic Apps cada vez que una de las evaluaciones de cumplimiento normativo cambie el estado.

Por ejemplo, si quiere que Security Center envíe un correo electrónico a un usuario específico cuando no se supere una valoración de cumplimiento, primero tendrá que crear la aplicación lógica (mediante [Azure Logic Apps](../logic-apps/logic-apps-overview.md)) y después tendrá que configurar el desencadenador en una nueva automatización de flujos de trabajo, tal y como se explica en [Automatización de respuestas a desencadenadores de Security Center](workflow-automation.md).

:::image type="content" source="media/release-notes/regulatory-compliance-triggers-workflow-automation.png" alt-text="Uso de los cambios en las valoraciones de cumplimiento normativo para desencadenar la automatización de un flujo de trabajo" lightbox="media/release-notes/regulatory-compliance-triggers-workflow-automation.png":::




## <a name="faq---regulatory-compliance-dashboard"></a>Preguntas frecuentes sobre el panel de cumplimiento normativo

- [¿Qué estándares se admiten en el panel de cumplimiento?](#what-standards-are-supported-in-the-compliance-dashboard)
- [¿Por qué algunos controles aparecen atenuados?](#why-do-some-controls-appear-grayed-out)
- [¿Cómo se puede quitar del panel un estándar integrado, como PCI-DSS, ISO 27001 o SOC2 TSP?](#how-can-i-remove-a-built-in-standard-like-pci-dss-iso-27001-or-soc2-tsp-from-the-dashboard)
- [He realizado el cambio sugerido según la recomendación, pero no aparece reflejado en el panel](#i-made-the-suggested-changed-based-on-the-recommendation-yet-it-isnt-being-reflected-in-the-dashboard)
- [¿Qué permisos necesito para acceder al panel de cumplimiento?](#what-permissions-do-i-need-to-access-the-compliance-dashboard)
- [No se carga el panel de cumplimiento normativo](#the-regulatory-compliance-dashboard-isnt-loading-for-me)
- [¿Cómo puedo ver un informe de los controles superados y no superados por estándar en mi panel?](#how-can-i-view-a-report-of-passing-and-failing-controls-per-standard-in-my-dashboard)
- [¿Cómo puedo descargar un informe con datos de cumplimiento en un formato distinto de PDF?](#how-can-i-download-a-report-with-compliance-data-in-a-format-other-than-pdf)
- [¿Cómo puedo crear excepciones para algunas de las directivas en el panel de cumplimiento normativo?](#how-can-i-create-exceptions-for-some-of-the-policies-in-the-regulatory-compliance-dashboard)
- [¿Qué planes o licencias de Azure Defender necesito para usar el panel de cumplimiento normativo?](#what-azure-defender-plans-or-licenses-do-i-need-to-use-the-regulatory-compliance-dashboard)

### <a name="what-standards-are-supported-in-the-compliance-dashboard"></a>¿Qué estándares se admiten en el panel de cumplimiento?
De forma predeterminada, el panel de cumplimiento normativo muestra Azure Security Benchmark. Azure Security Benchmark es el conjunto de directrices específico de Azure creado por Microsoft para ofrecer los procedimientos recomendados de seguridad y cumplimiento basados en marcos de cumplimiento comunes. Puede obtener más información en [Introducción a las pruebas comparativas de seguridad de Azure](../security/benchmarks/introduction.md).

Para hacer un seguimiento del cumplimiento con cualquier otro estándar, deberá agregarlo explícitamente al panel.
 
Puede agregar estándares como Azure CIS 1.1.0 (nuevo), NIST SP 800-53 R4, NIST SP 800-171 R2, SWIFT CSP CSCF-v2020, UK Official y UK NHS, HIPAA HITRUST, Canada Federal PBMM, ISO 27001, SOC2-TSP y PCI-DSS 3.2.1.  
 
Se agregarán más estándares al panel y se incluirán en la información sobre cómo [personalizar el conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md).

### <a name="why-do-some-controls-appear-grayed-out"></a>¿Por qué algunos controles aparecen atenuados?
Cada estándar de cumplimiento incluido en el panel incluye una lista de los controles del estándar. Para saber qué controles se pueden aplicar, puede ver los detalles de las evaluaciones superadas y no superadas.

Algunos controles aparecen atenuados, no tienen valoraciones de Security Center asociadas. Algunos pueden estar relacionados con el procedimiento o el proceso y, por lo tanto, no se pueden comprobar mediante Security Center. Otros todavía no tienen ninguna directiva o evaluación automatizada implementada, pero la tendrán en el futuro. Y otros controles pueden ser responsabilidad de la plataforma, como se explica en [Responsabilidad compartida en la nube](../security/fundamentals/shared-responsibility.md). 

### <a name="how-can-i-remove-a-built-in-standard-like-pci-dss-iso-27001-or-soc2-tsp-from-the-dashboard"></a>¿Cómo se puede quitar del panel un estándar integrado, como PCI-DSS, ISO 27001 o SOC2 TSP? 
Para personalizar el panel de cumplimiento normativo y centrarse solo en los estándares aplicables, puede quitar cualquiera de los estándares normativos incluidos que no sean aplicables para su organización. Para quitar un estándar, siga las instrucciones de [Eliminación de un estándar del panel](update-regulatory-compliance-packages.md#remove-a-standard-from-your-dashboard).

### <a name="i-made-the-suggested-changed-based-on-the-recommendation-yet-it-isnt-being-reflected-in-the-dashboard"></a>He realizado el cambio sugerido según la recomendación, pero no aparece reflejado en el panel
Después de tomar medidas para resolver las recomendaciones, espere 12 horas para ver los cambios en los datos de cumplimiento. Las evaluaciones se ejecutan aproximadamente cada 12 horas, por lo que este es el tiempo que tardará en verse el efecto en los datos de cumplimiento.
 
### <a name="what-permissions-do-i-need-to-access-the-compliance-dashboard"></a>¿Qué permisos necesito para acceder al panel de cumplimiento?
Para ver los datos de cumplimiento, debe tener también al menos acceso de **lectura** a los datos de cumplimiento de directivas; por lo tanto, el acceso de lector de seguridad solo no será suficiente. Si es lector global de la suscripción, también será suficiente.

El conjunto mínimo de roles para acceder al panel y administrar los estándares es **Colaborador de la directiva de recursos** y **Administrador de seguridad**.


### <a name="the-regulatory-compliance-dashboard-isnt-loading-for-me"></a>No se carga el panel de cumplimiento normativo
Para usar el panel de cumplimiento normativo, Azure Security Center debe tener Azure Defender habilitado en el nivel de suscripción. Si el panel no se carga correctamente, pruebe los pasos siguientes:

1. Borre la caché del explorador.
1. Pruebe con otro explorador.
1. Intente abrir el panel desde otra ubicación de red.


### <a name="how-can-i-view-a-report-of-passing-and-failing-controls-per-standard-in-my-dashboard"></a>¿Cómo puedo ver un informe de los controles superados y no superados por estándar en mi panel?
En el panel principal, puede ver un informe de los controles superados y no superados de (1) los 4 principales estándares de cumplimiento con el resultado más bajo en el panel. Para ver todo el estado de todos los controles superados y no superados, seleccione (2) **Mostrar todo *x*** (donde x es el número de estándares de los que está realizando el seguimiento). Un plano de contexto muestra el estado de cumplimiento de cada uno de los estándares de los que se realiza el seguimiento.

:::image type="content" source="media/security-center-compliance-dashboard/summaries-of-compliance-standards.png" alt-text="Sección de resumen del panel de cumplimiento normativo":::


### <a name="how-can-i-download-a-report-with-compliance-data-in-a-format-other-than-pdf"></a>¿Cómo puedo descargar un informe con datos de cumplimiento en un formato distinto de PDF?
Al seleccionar **Descargar Informe**, puede seleccionar el estándar y el formato (PDF o CSV). El informe resultante reflejará el conjunto actual de suscripciones que ha seleccionado en el filtro del portal.

- El informe PDF muestra el estado resumido del estándar seleccionado.
- El informe CSV proporciona resultados detallados por recurso, ya que los relaciona con las directivas asociadas a cada control.

Actualmente, no se admite la descarga del informe de una directiva personalizada, solo se pueden descargar los estándares normativos proporcionados.


### <a name="how-can-i-create-exceptions-for-some-of-the-policies-in-the-regulatory-compliance-dashboard"></a>¿Cómo puedo crear excepciones para algunas de las directivas en el panel de cumplimiento normativo?
En el caso de las directivas integradas en Security Center y que se incluyen en la puntuación de seguridad, puede crear exenciones para uno o más recursos directamente en el portal, tal como se explica en [Exención de recursos y recomendaciones de la puntuación de seguridad](exempt-resource.md).

En el caso de otras directivas, puede crear una exención directamente en la propia directiva, siguiendo las instrucciones de [Estructura de exención de Azure Policy](../governance/policy/concepts/exemption-structure.md).


### <a name="what-azure-defender-plans-or-licenses-do-i-need-to-use-the-regulatory-compliance-dashboard"></a>¿Qué planes o licencias de Azure Defender necesito para usar el panel de cumplimiento normativo?
Si tiene cualquiera de los paquetes de Azure Defender habilitado en cualquiera de los tipos de recursos de Azure, tiene acceso al panel de cumplimiento normativo, con todos sus datos, en Security Center.





## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido a utilizar el panel de cumplimiento normativo de Security Center para:

> [!div class="checklist"]
> * Vea y supervise su estado de cumplimiento con arreglo a los estándares y regulaciones que le interesen.
> * Mejorar el estado de cumplimiento mediante la resolución de las recomendaciones pertinentes y la observación de la mejora en la puntuación del cumplimiento.

El panel de cumplimiento normativo puede simplificar considerablemente el proceso de cumplimiento y reducir en gran medida el tiempo necesario para recopilar las pruebas de cumplimiento en un entorno de Azure, un entorno híbrido y un entorno con varias nubes.

Para más información, consulte los artículos relacionados:

- [Personalización del conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md): aprenda a seleccionar los estándares que aparecen en el panel de cumplimiento normativo. 
- [Recomendaciones de seguridad en Azure Security Center](security-center-recommendations.md): aprenda a usar las recomendaciones de Security Center como ayuda para la protección de los recursos de Azure.