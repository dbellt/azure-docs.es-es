---
title: Exención de una recomendación de Azure Security Center de un recurso, una suscripción, un grupo de administración y una puntuación de seguridad
description: Obtenga información sobre cómo crear reglas para excluir las recomendaciones de seguridad de las suscripciones o los grupos de administración y evitar que afecten a la puntuación de seguridad.
author: memildin
ms.author: memildin
ms.date: 03/11/2021
ms.topic: how-to
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: d545610b8fa13d0c0f8b91bf92107d53b81454c3
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107313851"
---
# <a name="exempting-resources-and-recommendations-from-your-secure-score"></a>Exención de recursos y recomendaciones de la puntuación de seguridad 

Una prioridad básica de cada equipo de seguridad es asegurarse de que los analistas puedan centrarse en las tareas y los incidentes que importan en la organización. Security Center presenta muchas características para personalizar la experiencia y garantizar que la puntuación de seguridad refleje las prioridades de seguridad de la organización. La opción **Exención** es una característica de este tipo.

Cuando se investigan recomendaciones de seguridad en Azure Security Center, uno de los primeros datos que se revisan es la lista de recursos afectados.

En ocasiones, aparecerá un recurso que no debe estar incluido. O se mostrará una recomendación en un ámbito en el que cree que no pertenece. Es posible que el recurso lo haya corregido un proceso del que Security Center no ha realizado un seguimiento. La recomendación podría no ser adecuada para una suscripción específica. O quizás la organización ha decidido simplemente aceptar los riesgos relacionados con la recomendación o el recurso específico.

En tales casos, puede crear una exención para una recomendación para:

- **Excluir un recurso** para asegurarse de que no aparezca con los recursos en mal estado en el futuro y no afecte a la puntuación de seguridad. El recurso se mostrará como no aplicable y el motivo aparecerá como "exento" con la justificación específica que seleccione.

- **Excluir una suscripción o un grupo de administración** para asegurarse de que la recomendación no afecte a la puntuación de seguridad y no que no se muestre para la suscripción o el grupo de administración en el futuro. Esto se refiere a los recursos existentes y cualquiera que cree en el futuro. La recomendación se marcará con la justificación específica que seleccione para el ámbito que ha seleccionado.

## <a name="availability"></a>Disponibilidad

| Aspecto                          | Detalles                                                                                                                                                                                                                                                                                                                            |
|---------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Estado de la versión:                  | Versión preliminar<br>[!INCLUDE [Legalese](../../includes/security-center-preview-legal-text.md)]                                                                                                                                                                                                                                             |
| Precios:                        | Se trata de una funcionalidad prémium de Azure Policy que se ofrece a los clientes de Azure Defender sin costo adicional. En el caso de otros usuarios, pueden aplicarse cargos en el futuro.                                                                                                                                                                 |
| Roles y permisos necesarios: | **Propietario** o **Colaborador de la directiva de recursos** para crear una exención<br>Para crear una regla, necesita permisos para editar directivas en Azure Policy.<br>Obtenga más información en [Permisos de Azure RBAC en Azure Policy](../governance/policy/overview.md#azure-rbac-permissions-in-azure-policy).                                            |
| Limitaciones:                    | Solo se pueden crear exenciones para las recomendaciones incluidas en la iniciativa predeterminada de Azure Security Center, Azure Security Benchmark o cualquiera de las iniciativas normativas estándar proporcionadas. No se pueden crear exenciones para las recomendaciones que se generan a partir de iniciativas personalizadas. Obtenga más información sobre las relaciones entre [directivas, iniciativas y recomendaciones](security-policy-concept.md). |
| Nubes:                         | ![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)                                                                                                                                                                                         |
|                                 |                                                                                                                                                                                                                                                                                                                                    |

## <a name="define-an-exemption"></a>Definición de una exención

Para ajustar las recomendaciones de seguridad que Security Center realiza para las suscripciones, el grupo de administración o los recursos, puede crear una regla de exención para:

- Marcar una **recomendación** específica o como "Mitigada" o "Riesgo aceptado". Puede crear exenciones de recomendación para una suscripción, varias suscripciones o un grupo de administración completo.
- Marcar **uno o más recursos** como "Mitigado" o "Riesgo aceptado" para una recomendación concreta.

> [!NOTE]
> Solo se pueden crear exenciones para las recomendaciones incluidas en la iniciativa predeterminada de Azure Security Center, Azure Security Benchmark o cualquiera de las iniciativas normativas estándar proporcionadas. No se pueden crear exenciones para las recomendaciones que se generan a partir de iniciativas personalizadas asignadas a las suscripciones. Obtenga más información sobre las relaciones entre [directivas, iniciativas y recomendaciones](security-policy-concept.md).

> [!TIP]
> También puede crear exenciones mediante la API. Para obtener un ejemplo de JSON y una explicación de las estructuras pertinentes, consulte [Estructura de exención de Azure Policy](../governance/policy/concepts/exemption-structure.md).

Para crear una regla de exención:

1. Abra la página de detalles de recomendaciones de la recomendación específica.

1. En la barra de herramientas de la parte superior de la página, haga clic en **Configuración**.

    :::image type="content" source="media/exempt-resource/exempting-recommendation.png" alt-text="Cree una regla de exención para que una recomendación se excluya de una suscripción o un grupo de administración.":::

1. En el panel **Exención**:
    1. Seleccione el ámbito de esta regla de exención:
        - Si selecciona un grupo de administración, la recomendación se excluirá de todas las suscripciones de ese grupo.
        - Si va a crear esta regla para excluir uno o varios recursos de la recomendación, elija "Recursos seleccionados" y seleccione los correspondientes en la lista.

    1. Escriba un nombre para esta regla de exención.
    1. Opcionalmente, puede establecer una fecha de expiración.
    1. Seleccione la categoría de la exención:
        - **Se resuelve a través de terceros (mitigada)** : si está usando un servicio de terceros que Security Center no ha identificado. 

            > [!NOTE]
            > Cuando se excluye de una recomendación como mitigada, no se proporcionan puntos hacia la puntuación de seguridad. No obstante, dado que los puntos no se *quitan* para los recursos incorrectos, el resultado es que aumentará la puntuación.

        - **Riesgo aceptado (renuncia)** : si ha decidido aceptar el riesgo de no mitigar esta recomendación.
    1. Si lo desea, escriba una descripción.
    1. Seleccione **Crear**.

    :::image type="content" source="media/exempt-resource/defining-recommendation-exemption.png" alt-text="Pasos para crear una regla de exención para que una recomendación se excluya de una suscripción o un grupo de administración.":::

    Cuando la exención surte efecto (puede tardar hasta 30 minutos):
    - La recomendación o los recursos no afectarán a la puntuación de seguridad.
    - Si ha excluido recursos específicos, se mostrarán en la pestaña **No aplicable** de la página de detalles de la recomendación.
    - Si ha excluido una recomendación, se ocultará de manera predeterminada en la página de recomendaciones de Security Center. Esto se debe a que las opciones predeterminadas del filtro de **estado de recomendación** de la página indican que se excluyan las recomendaciones **No aplicables**. Lo mismo sucede si se excluyen todas las recomendaciones en un control de seguridad.

        :::image type="content" source="media/exempt-resource/recommendations-filters-hiding-not-applicable.png" alt-text="Filtros predeterminados en la página de recomendaciones de Azure Security Center Ocultar las recomendaciones y los controles de seguridad no aplicables":::

    - La franja de información en la parte superior de la página de detalles de la recomendación actualiza el número de recursos exentos:
        
        :::image type="content" source="./media/exempt-resource/info-banner.png" alt-text="Número de recursos exentos":::

1. Para revisar los recursos exentos, abra la pestaña **No aplicable**:

    :::image type="content" source="./media/exempt-resource/modifying-exemption.png" alt-text="Modificación de una exención":::

    La razón de cada exención se incluye en la tabla (1).

    Para modificar o eliminar una exención, seleccione el menú de puntos suspensivos ("...") como se muestra (2).

1. Para revisar todas las reglas de exención de la suscripción, seleccione **Ver exenciones** desde la franja de información:

    > [!IMPORTANT]
    > Para ver las exenciones específicas relacionadas con una recomendación, filtre la lista según el ámbito pertinente y el nombre de la recomendación.

    :::image type="content" source="./media/exempt-resource/policy-page-exemption.png" alt-text="Página de exenciones de Azure Policy":::

    > [!TIP]
    > Como alternativa, [use Azure Resource Graph para encontrar recomendaciones con exenciones](#find-recommendations-with-exemptions-using-azure-resource-graph).

## <a name="monitor-exemptions-created-in-your-subscriptions"></a>Supervisión de las exenciones creadas en las suscripciones

Como se explicó anteriormente en esta página, las reglas de exención son una herramienta eficaz que proporciona un control pormenorizado sobre las recomendaciones que afectan a los recursos de las suscripciones y los grupos de administración. 

Para realizar un seguimiento de cómo los usuarios están ejerciendo esta funcionalidad, hemos creado una plantilla de Azure Resource Manager (ARM) que implementa un cuaderno de estrategias de aplicaciones lógicas y todas las conexiones de API necesarias para recibir una notificación cuando se ha creado una exención.

- Para más información sobre los cuadernos de estrategias, vea la entrada de blog de la comunidad tecnológica [Seguimiento de las exenciones de recursos en Azure Security Center](https://techcommunity.microsoft.com/t5/azure-security-center/how-to-keep-track-of-resource-exemptions-in-azure-security/ba-p/1770580).
- Encontrará la plantilla de ARM en el [repositorio de GitHub de Azure Security Center](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation/Notify-ResourceExemption).
- Para implementar todos los componentes necesarios, [use este proceso automatizado](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2FAzure-Security-Center%2Fmaster%2FWorkflow%2520automation%2FNotify-ResourceExemption%2Fazuredeploy.json).


## <a name="find-recommendations-with-exemptions-using-azure-resource-graph"></a>Uso de Azure Resource Graph para encontrar recomendaciones con exenciones

Azure Resource Graph (ARG) proporciona acceso instantáneo a la información de los recursos en los entornos en la nube con funcionalidades sólidas para filtrar, agrupar y ordenar. Es una forma rápida y eficaz de consultar información en las suscripciones de Azure mediante programación o desde Azure Portal.

Para ver todas las recomendaciones que tienen reglas de exención:

1. Abra **Azure Resource Graph Explorer**.

    :::image type="content" source="./media/security-center-identity-access/opening-resource-graph-explorer.png" alt-text="Inicio de la página de recomendaciones de Azure Resource Graph Explorer**" :::

1. Escriba la siguiente consulta y seleccione **Ejecutar consulta**.

    ```kusto
    securityresources
    | where type == "microsoft.security/assessments"
    // Get recommendations in useful format
    | project
    ['TenantID'] = tenantId,
    ['SubscriptionID'] = subscriptionId,
    ['AssessmentID'] = name,
    ['DisplayName'] = properties.displayName,
    ['ResourceType'] = tolower(split(properties.resourceDetails.Id,"/").[7]),
    ['ResourceName'] = tolower(split(properties.resourceDetails.Id,"/").[8]),
    ['ResourceGroup'] = resourceGroup,
    ['ContainsNestedRecom'] = tostring(properties.additionalData.subAssessmentsLink),
    ['StatusCode'] = properties.status.code,
    ['StatusDescription'] = properties.status.description,
    ['PolicyDefID'] = properties.metadata.policyDefinitionId,
    ['Description'] = properties.metadata.description,
    ['RecomType'] = properties.metadata.assessmentType,
    ['Remediation'] = properties.metadata.remediationDescription,
    ['Severity'] = properties.metadata.severity,
    ['Link'] = properties.links.azurePortal
    | where StatusDescription contains "Exempt"    
    ```


Más información en las siguientes páginas:
- [Más información sobre Azure Resource Graph](../governance/resource-graph/index.yml).
- [Creación de consultas con Azure Resource Graph Explorer](../governance/resource-graph/first-query-portal.md)
- [Lenguaje de consulta de Kusto (KQL)](/azure/data-explorer/kusto/query/)





## <a name="faq---exemption-rules"></a>Preguntas frecuentes: reglas de exención

### <a name="what-happens-when-one-recommendation-is-in-multiple-policy-initiatives"></a>¿Qué ocurre cuando una recomendación está en varias iniciativas de directivas?

A veces, se muestra una recomendación de seguridad en más de una iniciativa de directiva. Si tiene varias instancias de la misma recomendación asignadas a la misma suscripción y crea una exención para la recomendación, afectará a todas las iniciativas para las que tenga permiso de edición. 

Por ejemplo, la recomendación **** forma parte de la iniciativa de directivas predeterminada asignada a todas las suscripciones de Azure mediante Azure Security Center. También en XXXXX.

Si intenta crear una exención para esta recomendación, verá uno de los dos mensajes siguientes:

- Si tiene los permisos necesarios para editar ambas iniciativas, verá el siguiente mensaje:

    *Esta recomendación se incluye en varias iniciativas de directiva: [nombres de las iniciativas separados por comas]. Se crearán exenciones en todas ellas.*  

- Si no tiene permisos suficientes en ambas iniciativas, verá este mensaje en su lugar:

    *Tiene permisos limitados para aplicar la exención en todas las iniciativas de directiva. Las exenciones se crearán solo en las iniciativas con permisos suficientes.*


## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió a eximir un recurso de una recomendación para que no afecte a la puntuación segura. Para más información sobre la puntuación segura, consulte:

- [Puntuación de seguridad de Azure Security Center](secure-score-security-controls.md)
