---
title: Notas de la versión de Azure Security Center
description: Una descripción de las novedades y los cambios en Azure Security Center.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: reference
ms.date: 05/25/2021
ms.author: memildin
ms.openlocfilehash: 2d18b777cb70ed129ba3067e999338a9c54390b5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110460024"
---
# <a name="whats-new-in-azure-security-center"></a>Novedades de Azure Security Center

Defender está en desarrollo activo y recibe mejoras continuas. Para mantenerse al día con los avances más recientes, esta página proporciona información sobre las nuevas características, las correcciones de errores y las funcionalidades en desuso.

Esta página se actualiza con frecuencia, por lo que se recomienda visitarla a menudo. 

Para obtener información sobre los cambios *planeados* que están próximos a materializarse en Security Center, consulte el artículo [Próximos cambios importantes en Azure Security Center](upcoming-changes.md). 

> [!TIP]
> Si busca elementos de más de 6 meses, puede encontrarlos en las [Novedades de Azure Security Center](release-notes-archive.md).


## <a name="may-2021"></a>Mayo de 2021

Las actualizaciones de mayo incluyen:

- [Lanzamiento de Azure Defender para DNS y Azure Defender para Resource Manager para disponibilidad general (GA)](#azure-defender-for-dns-and-azure-defender-for-resource-manager-released-for-general-availability-ga)
- [Lanzamiento de Azure Defender para bases de datos relacionales de código abierto para disponibilidad general (GA)](#azure-defender-for-open-source-relational-databases-released-for-general-availability-ga)
- [Más consultas de Resource Graph disponibles para algunas recomendaciones](#more-resource-graph-queries-available-for-some-recommendations)
- [Se ha cambiado la gravedad de la recomendación de clasificación de datos SQL](#sql-data-classification-recommendation-severity-changed)
- [Nuevas recomendaciones para habilitar las funcionalidades de inicio seguro (en versión preliminar)](#new-recommendations-to-enable-trusted-launch-capabilities-in-preview)
- [Nuevas recomendaciones para reforzar los clústeres de Kubernetes (en versión preliminar)](#new-recommendations-for-hardening-kubernetes-clusters-in-preview)
- [La API de evaluaciones se ha ampliado con dos nuevos campos](#assessments-api-expanded-with-two-new-fields)
- [El inventario de recursos obtiene un filtro de entorno de nube](#asset-inventory-gets-a-cloud-environment-filter)


### <a name="azure-defender-for-dns-and-azure-defender-for-resource-manager-released-for-general-availability-ga"></a>Lanzamiento de Azure Defender para DNS y Azure Defender para Resource Manager para disponibilidad general (GA)

Estos dos planes de protección contra amenazas con amplitud nativa de nube ahora están disponibles de forma general.

Estas nuevas protecciones mejoran enormemente la resistencia frente a los ataques de actores de amenazas y aumentan considerablemente el número de recursos de Azure protegidos por Azure Defender.

- **Azure Defender para Resource Manager**: supervisa automáticamente todas las operaciones de administración de recursos realizadas en la organización. Para más información, consulte:
    - [Introducción a Azure Defender para Resource Manager](defender-for-resource-manager-introduction.md)
    - [Respuesta a las alertas de Azure Defender para Resource Manager](defender-for-resource-manager-usage.md)
    - [Lista de alertas proporcionadas por Azure Defender para Resource Manager](alerts-reference.md#alerts-resourcemanager)

- **Azure Defender para DNS**: supervisa continuamente todas las consultas de DNS de los recursos de Azure. Para más información, consulte:
    - [Introducción a Azure Defender para DNS](defender-for-dns-introduction.md)
    - [Respuesta a las alertas de Azure Defender para DNS](defender-for-dns-usage.md)
    - [Lista de alertas proporcionadas por Azure Defender para DNS](alerts-reference.md#alerts-dns)

Para simplificar el proceso de habilitación de estos planes, use las recomendaciones siguientes:

- **Se debe habilitar Azure Defender para Resource Manager**
- **Se debe habilitar Azure Defender para DNS**

> [!NOTE]
> La habilitación de los planes de Azure Defender conlleva cargos. Obtenga información sobre los detalles de los precios por región en la página de precios de Security Center: https://aka.ms/pricing-security-center.


### <a name="azure-defender-for-open-source-relational-databases-released-for-general-availability-ga"></a>Lanzamiento de Azure Defender para bases de datos relacionales de código abierto para disponibilidad general (GA)

Azure Security Center amplía su oferta de protección de SQL con un nuevo conjunto para cubrir las bases de datos relacionales de código abierto:

- **Azure Defender para servidores de bases de datos Azure SQL**: defiende los servidores SQL Server nativos de Azure.
- **Azure Defender para servidores SQL Server en máquinas**: amplía las mismas protecciones a los servidores SQL Server en entornos híbridos, multinube y locales.
- **Azure Defender para bases de datos relacionales de código abierto**: protege los servidores únicos de Azure Database for MySQL, PostgreSQL y MariaDB.

Azure Defender para bases de datos relacionales de código abierto supervisa constantemente las amenazas de seguridad de los servidores y detecta actividades anómalas de base de datos que indican posibles amenazas para Azure Database for MySQL, PostgreSQL y MariaDB. Ejemplos:

- **Detección granular de ataques por fuerza bruta**: Azure Defender para bases de datos relacionales de código abierto proporciona información detallada sobre los ataques por fuerza bruta intentados y cometidos. Esto le permite investigar y responder con una comprensión más completa de la naturaleza y el estado del ataque en su entorno.
- **Detección de alertas de comportamiento**: Azure Defender para bases de datos relacionales de código abierto le alertan de comportamientos sospechosos e inesperados en los servidores, como cambios en el patrón de acceso a la base de datos.
- **Detección basada en inteligencia sobre amenazas**: Azure Defender aprovecha la inteligencia sobre amenazas de Microsoft y la amplia base de conocimientos para que pueda actuar contra ellas.

Obtenga más información en [Introducción a Azure Defender para bases de datos relacionales de código abierto](defender-for-databases-introduction.md).

### <a name="more-resource-graph-queries-available-for-some-recommendations"></a>Más consultas de Resource Graph disponibles para algunas recomendaciones

Todas las recomendaciones de Security Center ofrecen la opción de ver la información sobre el estado de los recursos afectados mediante Azure Resource Graph desde la opción **Abrir consulta**. Para obtener detalles completos sobre esta eficaz característica, consulte [Revisión de los datos de recomendación en Azure Resource Graph Explorer (ARG)](security-center-recommendations.md#review-recommendation-data-in-azure-resource-graph-explorer-arg).

Security Center incluye escáneres de vulnerabilidades integrados para examinar las VM, los servidores SQL Server y sus hosts, y los registros de contenedor en busca de vulnerabilidades de seguridad. Los resultados se devuelven como recomendaciones con todas las conclusiones individuales de cada tipo de recurso recopiladas en una sola vista. Las recomendaciones son:

- Es necesario corregir las vulnerabilidades de las imágenes de Azure Container Registry (con tecnología de Qualys)
- Es necesario corregir las vulnerabilidades de las máquinas virtuales
- Las bases de datos SQL deben tener resueltos los hallazgos de vulnerabilidades.
- Los servidores SQL de las máquinas deben tener resueltos los hallazgos de vulnerabilidades.

Con este cambio, puede usar el botón **Abrir consulta** para abrir también la consulta que muestra los hallazgos de seguridad.

:::image type="content" source="media/release-notes/open-query-menu-security-findings.png" alt-text="El botón Abrir consulta ahora ofrece opciones para una consulta más profunda que muestra las conclusiones sobre seguridad de las recomendaciones relacionadas con el detector de vulnerabilidades.":::

El botón **Abrir consulta** también ofrece opciones adicionales para algunas otras recomendaciones si procede.

Obtenga más información sobre los detectores de vulnerabilidades de Security Center:

- [Detector de evaluación de vulnerabilidades integrado en Azure Defender para Azure y máquinas híbridas](deploy-vulnerability-assessment-vm.md)
- [Detector de evaluación de vulnerabilidades integrado de Azure Defender para los servidores SQL Server](defender-for-sql-on-machines-vulnerability-assessment.md)
- [Detector de evaluación de vulnerabilidades integrado de Azure Defender para los registros de contenedor](defender-for-container-registries-usage.md)

### <a name="sql-data-classification-recommendation-severity-changed"></a>Se ha cambiado la gravedad de la recomendación de clasificación de datos SQL

La gravedad de la recomendación **Sensitive data in your SQL databases should be classified** (Se deben clasificar los datos confidenciales de las bases de datos SQL) ha cambiado de **Alta** a **Baja**.

Este cambio forma parte de los cambios en curso de esta recomendación anunciados en [Mejoras en la recomendación de clasificación de datos de SQL](upcoming-changes.md#enhancements-to-sql-data-classification-recommendation). 


### <a name="new-recommendations-to-enable-trusted-launch-capabilities-in-preview"></a>Nuevas recomendaciones para habilitar las funcionalidades de inicio seguro (en versión preliminar)

Azure ofrece el inicio seguro como una manera continua de mejorar la seguridad de las máquinas virtuales de [generación 2](../virtual-machines/generation-2.md). El inicio seguro protege frente a técnicas de ataque persistentes y avanzadas. El inicio seguro se compone de varias tecnologías de infraestructura coordinadas que se pueden habilitar de manera independiente. Cada tecnología proporciona otro nivel de defensa contra amenazas sofisticadas. Obtenga más información en [Inicio seguro para máquinas virtuales de Azure](../virtual-machines/trusted-launch.md).

> [!IMPORTANT]
> El inicio seguro requiere la creación de nuevas máquinas virtuales. No se puede habilitar el inicio seguro en las máquinas virtuales existentes que se crearon inicialmente sin él.
> 
> El inicio seguro está actualmente en versión preliminar pública. Esta versión preliminar se ofrece sin acuerdo de nivel de servicio y no es aconsejable usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas.

La recomendación de Security Center, **vTPM debe estar habilitado en las máquinas virtuales admitidas**, garantiza que las VM de Azure usen vTPM. Esta versión virtualizada del hardware Módulo de plataforma segura permite la atestación mediante la medición de toda la cadena de arranque de la VM (UEFI, sistema operativo, sistema y controladores).

Con vTPM habilitado, la **extensión de atestación de invitado** puede validar de forma remota el arranque seguro. Las siguientes recomendaciones garantizan la implementación de esta extensión:

- **El arranque seguro debe estar habilitado en las máquinas virtuales Windows admitidas**
- **La extensión de atestación de invitados debe estar instalada en las máquinas virtuales Windows**
- **La extensión de atestación de invitados debe estar instalada en conjuntos de escalado de máquinas virtuales Windows admitidos**
- **La extensión de atestación de invitados debe estar instalada en las máquinas virtuales Linux**
- **La extensión de atestación de invitados debe estar instalada en conjuntos de escalado de máquinas virtuales Linux admitidos**

Obtenga más información en [Inicio seguro para máquinas virtuales de Azure](../virtual-machines/trusted-launch.md). 

### <a name="new-recommendations-for-hardening-kubernetes-clusters-in-preview"></a>Nuevas recomendaciones para reforzar los clústeres de Kubernetes (en versión preliminar)

Las siguientes recomendaciones le permiten reforzar aún más los clústeres de Kubernetes.

- **Los clústeres de Kubernetes no deben utilizar el espacio de nombres predeterminado**: para proteger del acceso no autorizado los tipos de recurso ConfigMap, Pod, Secret, Service y ServiceAccount, evite utilizar el espacio de nombres predeterminado en los clústeres de Kubernetes.
- **Los clústeres de Kubernetes deben deshabilitar el montaje automático de credenciales de API**: para evitar que un recurso pod potencialmente comprometido ejecute comandos de API en clústeres de Kubernetes, deshabilite el montaje automático de credenciales de API.
- **Los clústeres de Kubernetes no deben otorgar funcionalidades de seguridad CAPSYSADMIN**.

Descubra qué puede hacer Security Center para proteger los entornos contenedorizados en [Seguridad de contenedor en Security Center](container-security.md).

### <a name="assessments-api-expanded-with-two-new-fields"></a>La API de evaluaciones se ha ampliado con dos nuevos campos

Se han agregado los dos campos siguientes a la [API REST de evaluaciones](https://docs.microsoft.com/rest/api/securitycenter/assessments):

- **FirstEvaluationDate**: hora a la que se creó y evaluó por primera vez la recomendación. Se devuelve en horario UTC en formato ISO 8601.
- **StatusChangeDate**: hora a la que cambió por última vez el estado de la recomendación. Se devuelve en horario UTC en formato ISO 8601.

El valor predeterminado inicial de estos campos (para todas las recomendaciones) es `2021-03-14T00:00:00+0000000Z`.

Para acceder a esta información, puede usar cualquiera de los métodos de la tabla siguiente.

| Herramienta                 | Detalles                                                                                                                                                                |
|----------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Llamada a API REST        | GET https://management.azure.com/subscriptions/<SUBSCRIPTION_ID>/providers/Microsoft.Security/assessments?api-version=2019-01-01-preview& $expand=statusEvaluationDates |
| Azure Resource Graph | `securityresources`<br>`where type == "microsoft.security/assessments"`                                                                                                |
| Automatización de flujos de trabajo  | Los dos campos dedicados estarán disponibles para los datos del área de trabajo de Log Analytics.                                                                                            |
| [Exportación de CSV](continuous-export.md#manual-one-time-export-of-alerts-and-recommendations) | Los dos campos se incluyen en los archivos CSV.                                                        |
|                      |                                                                                                                                                                        |


Más información sobre la [API REST de evaluaciones](https://docs.microsoft.com/rest/api/securitycenter/assessments).


### <a name="asset-inventory-gets-a-cloud-environment-filter"></a>El inventario de recursos obtiene un filtro de entorno de nube

La página del inventario de recursos de Security Center ofrece una serie de filtros para refinar rápidamente la lista de recursos que se muestran. Más información en [Exploración y administración de los recursos con Asset Inventory](asset-inventory.md).

Un nuevo filtro ofrece la opción de refinar la lista según las cuentas en la nube conectadas con las características de varias nubes de Security Center:

:::image type="content" source="media/asset-inventory/filter-environment.png" alt-text="Filtro de entorno del inventario":::

Obtenga más información sobre las funcionalidades de varias nubes:

- [Conexión de las cuentas de AWS a Azure Security Center](quickstart-onboard-aws.md)
- [Conexión de las cuentas de GCP a Azure Security Center](quickstart-onboard-gcp.md)


## <a name="april-2021"></a>Abril de 2021

Las actualizaciones de abril incluyen:
- [Actualización de la página de estado de recursos (en versión preliminar)](#refreshed-resource-health-page-in-preview)
- [Las imágenes del registro de contenedor que se han extraído recientemente ahora se vuelven a examinar semanalmente (lanzado en disponibilidad general)](#container-registry-images-that-have-been-recently-pulled-are-now-rescanned-weekly-released-for-general-availability-ga)
- [Use Azure Defender para Kubernetes para proteger implementaciones de Kubernetes híbridas y de varias nubes (versión preliminar)](#use-azure-defender-for-kubernetes-to-protect-hybrid-and-multi-cloud-kubernetes-deployments-in-preview)
- [La integración de Microsoft Defender para punto de conexión con Azure Defender ahora es compatible con Windows Server 2019 y Windows 10 Virtual Desktop (WVD) (lanzado en disponibilidad general)](#microsoft-defender-for-endpoint-integration-with-azure-defender-now-supports-windows-server-2019-and-windows-10-virtual-desktop-wvd-released-for-general-availability-ga)
- [Recomendaciones para habilitar Azure Defender para DNS y Resource Manager (en versión preliminar)](#recommendations-to-enable-azure-defender-for-dns-and-resource-manager-in-preview)
- [Se han agregado tres estándares de cumplimiento normativo: Azure CIS 1.3.0, CMMC Nivel 3 e ISM restringido de Nueva Zelanda](#three-regulatory-compliance-standards-added-azure-cis-130-cmmc-level-3-and-new-zealand-ism-restricted)
- [Cuatro nuevas recomendaciones relacionadas con la configuración de invitado (en versión preliminar)](#four-new-recommendations-related-to-guest-configuration-in-preview)
- [Recomendaciones de CMK trasladadas al control de seguridad de procedimientos recomendados](#cmk-recommendations-moved-to-best-practices-security-control)
- [11 alertas de Azure Defender se han puesto en desuso](#11-azure-defender-alerts-deprecated).
- [Dos recomendaciones del control de seguridad "Aplicar actualizaciones del sistema" entraron en desuso](#two-recommendations-from-apply-system-updates-security-control-were-deprecated).
- [Azure Defender para SQL en el icono de la máquina se ha eliminado del panel de Azure Defender](#azure-defender-for-sql-on-machine-tile-removed-from-azure-defender-dashboard)
- [Traslado de 21 recomendaciones entre controles de seguridad](#21-recommendations-moved-between-security-controls)

### <a name="refreshed-resource-health-page-in-preview"></a>Actualización de la página de estado de recursos (en versión preliminar)

El estado de los recursos de Security Center se ha ampliado y mejorado para proporcionar una vista de instantánea del estado general de un recurso individual. 

Puede revisar información detallada sobre el recurso y todas las recomendaciones que se aplican a ese recurso. Además, si usa [Azure Defender](azure-defender.md), también puede ver alertas de seguridad pendientes para ese recurso específico.

Para abrir la página de mantenimiento de un recurso, seleccione cualquier recurso en la [página de inventario de recursos](asset-inventory.md).

En esta página de vista previa del portal de Security Center se muestra lo siguiente:

1. **Información sobre los recursos**: el grupo de recursos y la suscripción a la que está asociado, la ubicación geográfica, etc.
1. **Característica de seguridad aplicada:** muestra si Azure Defender está habilitado para el recurso o no.
1. **Recuentos de recomendaciones y alertas pendientes:** el número de recomendaciones de seguridad y alertas pendientes de Azure Defender.
1. **Recomendaciones y alertas prácticas:** en dos pestañas se incluyen las recomendaciones y alertas que son aplicables al recurso.

:::image type="content" source="media/investigate-resource-health/resource-health-page-virtual-machine.gif" alt-text="Página de estado de los recursos de Azure Security Center que muestra la información de estado de una máquina virtual":::

Obtenga más información en [Tutorial: Investigación del estado de los recursos](investigate-resource-health.md).


### <a name="container-registry-images-that-have-been-recently-pulled-are-now-rescanned-weekly-released-for-general-availability-ga"></a>Las imágenes del registro de contenedor que se han extraído recientemente ahora se vuelven a examinar semanalmente (lanzado en disponibilidad general)

Azure Defender para registros de contenedor incluye un analizador de vulnerabilidades integrado. Este analizador examina inmediatamente cualquier imagen que se inserte en el registro y cualquier imagen que se haya extraído en los últimos 30 días.

Cada día se detectan nuevas vulnerabilidades. Con esta actualización, las imágenes de contenedor que fueron extraídas de los registros durante los últimos 30 días se **volverán a examinar** cada semana. Esto garantiza que se identifiquen las vulnerabilidades recién detectadas en las imágenes.

El examen se cobra por imagen, por lo que no hay ningún cargo adicional por estos nuevos exámenes.

Más información sobre este analizador en [Uso de Azure Defender para registros de contenedor para examinar las imágenes en busca de vulnerabilidades](defender-for-container-registries-usage.md).


### <a name="use-azure-defender-for-kubernetes-to-protect-hybrid-and-multi-cloud-kubernetes-deployments-in-preview"></a>Use Azure Defender para Kubernetes para proteger implementaciones de Kubernetes híbridas y de varias nubes (versión preliminar)

Azure Defender para Kubernetes está expandiendo sus funcionalidades de protección contra amenazas para defender los clústeres dondequiera que estén implementados. Esto se ha habilitado mediante la integración con [Kubernetes habilitado para Azure Arc](../azure-arc/kubernetes/overview.md) y sus nuevas [funcionalidades de extensiones](../azure-arc/kubernetes/extensions.md). 

Una vez habilitado Azure Arc en los clústeres de Kubernetes que no son de Azure, una nueva recomendación de Azure Security Center ofrece la implementación de la extensión de Azure Defender en ellos con solo unos clics.

Use la recomendación (los **clústeres de Kubernetes habilitados para Azure Arc deben tener instalada la extensión de Azure Defender**) y la extensión para proteger los clústeres de Kubernetes implementados en otros proveedores de nube, aunque no en los servicios administrados de Kubernetes.

Esta integración entre Azure Security Center, Azure Defender y Kubernetes habilitado para Azure Arc ofrece:

- Aprovisionamiento sencillo de la extensión de Azure Defender en clústeres de Kubernetes habilitado para Azure Arc sin protección (manualmente y a escala)
- Supervisión de la extensión de Azure Defender y su estado de aprovisionamiento desde el portal de Azure Arc
- Las recomendaciones de seguridad de Security Center se muestran en la nueva página de seguridad del portal de Azure Arc.
- Las amenazas de seguridad identificadas de Azure Defender se muestran en la nueva página de seguridad del portal de Azure Arc.
- Los clústeres de Kubernetes habilitados para Azure Arc están integrados en la plataforma y la experiencia de Azure Security Center

Puede encontrar más información en [Uso de Azure Defender para Kubernetes con los clústeres de Kubernetes locales y de varias nubes](defender-for-kubernetes-azure-arc.md).

:::image type="content" source="media/defender-for-kubernetes-azure-arc/extension-recommendation.png" alt-text="La recomendación de Azure Security Center para implementar la extensión de Azure Defender para clústeres de Kubernetes habilitados para Azure Arc." lightbox="media/defender-for-kubernetes-azure-arc/extension-recommendation.png":::


### <a name="microsoft-defender-for-endpoint-integration-with-azure-defender-now-supports-windows-server-2019-and-windows-10-virtual-desktop-wvd-released-for-general-availability-ga"></a>La integración de Microsoft Defender para punto de conexión con Azure Defender ahora es compatible con Windows Server 2019 y Windows 10 Virtual Desktop (WVD) (lanzado en disponibilidad general)

Microsoft Defender para punto de conexión es una solución integral de seguridad de punto de conexión que se entrega en la nube. Proporciona funciones de administración y evaluación de vulnerabilidades basadas en riesgos, así como de detección y respuesta de puntos de conexión (EDR). Para obtener una lista completa de las ventajas del uso de Defender para punto de conexión junto con Azure Security Center, consulte [Proteja los puntos de conexión con la solución EDR integrada de Security Center: Microsoft Defender para punto de conexión](security-center-wdatp.md).

Cuando habilita Azure Defender para servidores en un servidor de Windows, el plan incluye una licencia de Defender para punto de conexión. Si ya ha habilitado Azure Defender para servidores y tiene servidores de Windows 2019 en la suscripción, los servidores recibirán automáticamente Defender para punto de conexión con esta actualización. No se requiere ninguna acción manual. 

Ahora se ha ampliado la compatibilidad para incluir Windows Server 2019 y [Windows Virtual Desktop (WVD)](../virtual-desktop/overview.md).

> [!NOTE]
> Si va a habilitar Defender para punto de conexión en un equipo con Windows Server 2019, asegúrese de que cumple los requisitos previos descritos en [Habilitación de la integración de Microsoft Defender para punto de conexión](security-center-wdatp.md#enable-the-microsoft-defender-for-endpoint-integration).


### <a name="recommendations-to-enable-azure-defender-for-dns-and-resource-manager-in-preview"></a>Recomendaciones para habilitar Azure Defender para DNS y Resource Manager (en versión preliminar)

Se han agregado dos nuevas recomendaciones para simplificar el proceso de habilitación de [Azure Defender para Resource Manager](defender-for-resource-manager-introduction.md) y [Azure Defender para DNS](defender-for-dns-introduction.md):

- **Azure Defender para Resource Manager se debe habilitar**: Defender para Resource Manager supervisa automáticamente las operaciones de administración de recursos de la organización. Azure Defender detecta amenazas y alerta sobre actividades sospechosas.
- **Azure Defender para DNS se debe habilitar**: Defender para DNS proporciona una capa adicional de protección para los recursos de la nube, ya que supervisa de forma ininterrumpida todas las consultas de DNS de los recursos de Azure. Azure Defender alerta sobre las actividades sospechosas en la capa de DNS.

La habilitación de los planes de Azure Defender conlleva cargos. Obtenga información sobre los detalles de los precios por región en la página de precios de Security Center: https://aka.ms/pricing-security-center.

> [!TIP]
> Las recomendaciones de la versión preliminar no representan un recurso incorrecto y no se incluyen en los cálculos de una puntuación segura. Corríjalas siempre que sea posible, de tal forma que, cuando finalice el período de versión preliminar, contribuyan a la puntuación. Puede encontrar más información sobre cómo responder a estas recomendaciones en [Recomendaciones de corrección en Azure Security Center](security-center-remediate-recommendations.md).


### <a name="three-regulatory-compliance-standards-added-azure-cis-130-cmmc-level-3-and-new-zealand-ism-restricted"></a>Se han agregado tres estándares de cumplimiento normativo: Azure CIS 1.3.0, CMMC Nivel 3 e ISM restringido de Nueva Zelanda

Hemos agregado tres estándares para su uso con Azure Security Center. Con el panel de cumplimiento normativo, ahora puede realizar un seguimiento del cumplimiento con:

- [CIS Microsoft Azure Foundations Benchmark 1.3.0](../governance/policy/samples/cis-azure-1-3-0.md)
- [CMMC nivel 3](../governance/policy/samples/cmmc-l3.md)
- [ISM restringido en Nueva Zelanda](../governance/policy/samples/new-zealand-ism.md)

Puede asignarlos a las suscripciones como se describe en [Personalización del conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md).

:::image type="content" source="media/release-notes/additional-regulatory-compliance-standards.png" alt-text="Se han agregado tres estándares para su uso en el panel de cumplimiento normativo de Azure Security Center." lightbox="media/release-notes/additional-regulatory-compliance-standards.png":::

Puede encontrar más información en:
- [Personalización del conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md)
- [Tutorial: Mejora del cumplimiento normativo](security-center-compliance-dashboard.md)
- [Tutorial: Mejora del cumplimiento normativo](security-center-compliance-dashboard.md#faq---regulatory-compliance-dashboard)

### <a name="four-new-recommendations-related-to-guest-configuration-in-preview"></a>Cuatro nuevas recomendaciones relacionadas con la configuración de invitado (en versión preliminar)

La [extensión de configuración de invitado](../governance/policy/concepts/guest-configuration.md) de Azure informa a Security Center para ayudar a garantizar la protección de la configuración de invitado de las máquinas virtuales. La extensión no es necesaria para los servidores habilitados para Arc porque está incluida en el agente Connected Machine de Arc. La extensión requiere una identidad administrada por el sistema en la máquina.

Hemos agregado cuatro nuevas recomendaciones a Security Center para sacar el máximo partido a esta extensión.

- En dos recomendaciones se le pide que instale la extensión y su identidad administrada por el sistema requerida:
    - **La extensión "Configuración de invitado" debe estar instalada en las máquinas**
    - **La extensión "Configuración de invitado" de las máquinas virtuales debe implementarse con una identidad administrada asignada por el sistema**

- Cuando la extensión esté instalada y en ejecución, comenzará a auditar las máquinas y se le pedirá que refuerce la configuración del sistema operativo y la del entorno. Estas dos recomendaciones le pedirán que proteja sus máquinas Windows y Linux como se describe a continuación:
    - **La Protección contra vulnerabilidades de seguridad de Windows Defender debe estar habilitada en las máquinas**.
    - **La autenticación en máquinas Linux debe requerir claves SSH.**

Puede encontrar más información en [Información sobre Guest Configuration de Azure Policy](../governance/policy/concepts/guest-configuration.md).

### <a name="cmk-recommendations-moved-to-best-practices-security-control"></a>Recomendaciones de CMK trasladadas al control de seguridad de procedimientos recomendados

El programa de seguridad de cada organización incluye requisitos de cifrado de datos. De manera predeterminada, los datos de los clientes de Azure se cifran en reposo con claves administradas por el servicio. Sin embargo, las claves administradas por el cliente (CMK) suelen ser necesarias para cumplir estándares de cumplimiento normativo. Las CMK permiten cifrar los datos con una clave de [Azure Key Vault](../key-vault/general/overview.md) creada por el usuario y propiedad de este. Esto le da control y responsabilidad total sobre el ciclo de vida de la clave, incluidas la rotación y la administración.

Los controles de seguridad de Azure Security Center son grupos lógicos de recomendaciones de seguridad relacionadas y reflejan las superficies de ataque vulnerables. Cada control tiene un número máximo de puntos que puede sumar a la puntuación de seguridad si corrige todas las recomendaciones enumeradas en el control para todos los recursos. El control de seguridad **Implementar procedimientos recomendados de seguridad** vale cero puntos. Por lo tanto, las recomendaciones de este control no afectan a la puntuación de seguridad.

Las recomendaciones que se enumeran a continuación se trasladan al control de seguridad **Implementar procedimientos recomendados de seguridad** para reflejar mejor su naturaleza opcional. Este traslado garantiza que cada una de estas recomendaciones está en el control más adecuado para cumplir su objetivo.

- Las cuentas de Azure Cosmos DB deben usar claves administradas por el cliente para cifrar los datos en reposo
- Las áreas de trabajo de Azure Machine Learning deben cifrarse con una clave administrada por el cliente (CMK).
- Las cuentas de Cognitive Services deben habilitar el cifrado de datos con una clave administrada por el cliente (CMK)
- Las instancias de Container Registry se deben cifrar con una clave administrada por el cliente (CMK)
- Las instancias administradas de SQL deben usar claves administradas por el cliente para cifrar los datos en reposo
- Los servidores SQL deben usar claves administradas por el cliente para cifrar los datos en reposo
- Las cuentas de almacenamiento deben usar claves administradas por el cliente (CMK) para el cifrado

Obtenga información sobre qué recomendaciones se encuentran en cada control de seguridad en [Controles de seguridad y sus recomendaciones](secure-score-security-controls.md#security-controls-and-their-recommendations).


### <a name="11-azure-defender-alerts-deprecated"></a>11 alertas de Azure Defender se han puesto en desuso.

Las once alertas de Azure Defender que se enumeran a continuación han quedado en desuso.

- Las nuevas alertas reemplazarán estas dos y proporcionarán una mejor cobertura:

    | AlertType                | AlertDisplayName                                                         |
    |--------------------------|--------------------------------------------------------------------------|
    | ARM_MicroBurstDomainInfo | VERSIÓN PRELIMINAR: Se detectó la ejecución de la función "Get-AzureDomainInfo" del kit de herramientas de MicroBurst. |
    | ARM_MicroBurstRunbook    | VERSIÓN PRELIMINAR: Se detectó la ejecución de la función "Get-AzurePasswords" del kit de herramientas de MicroBurst.  |
    |                          |                                                                          |

- Estas nueve alertas se refieren a un conector de Azure Active Directory Identity Protection (IPC) que ya está en desuso:

    | AlertType           | AlertDisplayName              |
    |---------------------|-------------------------------|
    | UnfamiliarLocation  | Propiedades de inicio de sesión desconocidas |
    | AnonymousLogin      | Dirección IP anónima          |
    | InfectedDeviceLogin | Dirección IP vinculada al malware     |
    | ImpossibleTravel    | Viaje atípico               |
    | MaliciousIP         | Dirección IP malintencionada          |
    | LeakedCredentials   | Credenciales con fugas            |
    | PasswordSpray       | Difusión de contraseñas                |
    | LeakedCredentials   | Inteligencia de Azure AD sobre amenazas  |
    | AADAI               | IA de Azure AD                   |
    |                     |                               |
 
    > [!TIP]
    > Estas nueve alertas de IPC nunca fueron alertas de Security Center. Forman parte del conector de Azure Active Directory Identity Protection que las envía a Security Center. Durante los últimos dos años, los únicos clientes que han estado viendo esas alertas son las organizaciones que han configurado la exportación (del conector a ASC) en 2019 o versiones anteriores. El conector de Azure Active Directory Identity Protection ha continuado mostrando sus propios sistemas de alertas y ha continuado estando disponible en Azure Sentinel. El único cambio es que ya no aparecen en Security Center.

### <a name="two-recommendations-from-apply-system-updates-security-control-were-deprecated"></a>Dos recomendaciones del control de seguridad "Aplicar actualizaciones del sistema" entraron en desuso. 

Las dos recomendaciones siguientes estaban en desuso y los cambios pueden dar lugar a un leve impacto en la puntuación de seguridad:

- **Las máquinas deben reiniciarse para aplicar las actualizaciones del sistema**
- **El agente de supervisión debe instalarse en las máquinas**. Esta recomendación solo está relacionada con máquinas locales y parte de su lógica se transferirá a otra recomendación, **Se deben resolver los problemas de estado del agente de Log Analytics en las máquinas**.

Se recomienda comprobar las configuraciones de exportación continua y de automatización del flujo de trabajo para ver si estas recomendaciones están incluidas en ellas. Además, los paneles u otras herramientas de supervisión que puedan estar usándolas se deben actualizar en consecuencia.

Más información sobre estas recomendaciones en la [página de referencia de las recomendaciones de seguridad](recommendations-reference.md).

### <a name="azure-defender-for-sql-on-machine-tile-removed-from-azure-defender-dashboard"></a>Azure Defender para SQL en el icono de la máquina se ha eliminado del panel de Azure Defender

El área de cobertura del panel de Azure Defender incluye iconos para los planes de Azure Defender pertinentes para su entorno. Debido a un problema con los informes de los números de recursos protegidos y no protegidos, hemos decidido quitar temporalmente el estado de cobertura de recursos para **Azure Defender para SQL en las máquinas** hasta que se resuelva el problema.


### <a name="21-recommendations-moved-between-security-controls"></a>Traslado de 21 recomendaciones entre controles de seguridad 

Las siguientes recomendaciones se han trasladado a otros controles de seguridad. Los controles de seguridad son grupos lógicos de recomendaciones de seguridad relacionadas y reflejan las superficies de ataque vulnerables. Este traslado garantiza que cada una de estas recomendaciones está en el control más adecuado para cumplir su objetivo.

Obtenga información sobre qué recomendaciones se encuentran en cada control de seguridad en [Controles de seguridad y sus recomendaciones](secure-score-security-controls.md#security-controls-and-their-recommendations).

|Recomendación |Cambio e impacto  |
|---------|---------|
|La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.<br>La evaluación de vulnerabilidad debe estar habilitada en las instancias administradas de SQL.<br>Las vulnerabilidades de sus bases de datos SQL deben remediarse ahora<br>Las vulnerabilidades de las bases de datos SQL en máquinas virtuales deben corregirse     |Cambia de Corregir vulnerabilidades (con un valor de 6 puntos)<br>a Corregir configuraciones de seguridad (con un valor de 4 puntos)<br>En función del entorno, estas recomendaciones tendrán menor impacto en la puntuación.|
|Debe haber más de un propietario asignado a su suscripción<br>Las variables de cuenta de automatización deben cifrarse<br> Dispositivos IoT: el proceso auditado dejó de enviar eventos.<br> Dispositivos IoT: error de validación de línea base del sistema operativo<br> Dispositivos IoT: es preciso actualizar el conjunto de cifrado de TLS.<br>Dispositivos IoT: puertos abiertos en el dispositivo.<br>Dispositivos IoT: se encontró una directiva de firewall permisiva en una de las cadenas.<br> Dispositivos IoT: se encontró una regla de firewall permisiva en la cadena de entrada.<br> Dispositivos IoT: se encontró una regla de firewall permisiva en la cadena de salida.<br>Los registros de diagnóstico de IoT Hub deben estar habilitados<br>Dispositivos IoT: mensajes infrautilizados de envío del agente.<br>Dispositivos IoT: la directiva de filtro de IP predeterminada debe ser Denegar.<br>Dispositivos IoT: intervalo de IP amplio de la regla del filtro de IP.<br>Dispositivos IoT: se deben ajustar tanto el tamaño como los intervalos de los mensajes de los agentes<br>Dispositivos IoT: credenciales de autenticación idénticas<br>Dispositivos IoT: el proceso auditado dejó de enviar eventos.<br>Dispositivos IoT: la configuración de la línea base del sistema operativo (SO) debe corregirse|Cambia a **Implementación de procedimientos recomendados de seguridad**.<br>Cuando una recomendación pasa al control de seguridad Implementar prácticas recomendadas de seguridad, que no vale ningún punto, la recomendación deja de afectar a la puntuación segura.|
|||


## <a name="march-2021"></a>Marzo de 2021

Las actualizaciones de marzo incluyen:

- [Administración de Azure Firewall integrada en Security Center](#azure-firewall-management-integrated-into-security-center)
- [La evaluación de vulnerabilidades de SQL ahora incluye la experiencia "Deshabilitación de regla" (versión preliminar)](#sql-vulnerability-assessment-now-includes-the-disable-rule-experience-preview)
- [Se integran en Security Center los libros de Azure Monitor y se proporcionan tres plantillas](#azure-monitor-workbooks-integrated-into-security-center-and-three-templates-provided)
- [El panel de cumplimiento normativo ahora incluye informes de auditoría de Azure (versión preliminar)](#regulatory-compliance-dashboard-now-includes-azure-audit-reports-preview)
- [Los datos de recomendaciones se pueden ver en Azure Resource Graph con "Explore in ARG" (Explorar en ARG)](#recommendation-data-can-be-viewed-in-azure-resource-graph-with-explore-in-arg)
- [Actualizaciones en las directivas para implementar la automatización de flujos de trabajo](#updates-to-the-policies-for-deploying-workflow-automation)
- [Dos recomendaciones heredadas dejan de escribir datos directamente en el registro de actividad de Azure](#two-legacy-recommendations-no-longer-write-data-directly-to-azure-activity-log)
- [Mejoras de la página de recomendaciones](#recommendations-page-enhancements)


### <a name="azure-firewall-management-integrated-into-security-center"></a>Administración de Azure Firewall integrada en Security Center

Al abrir Azure Security Center, la primera página que aparece es la página de información general. 

Este panel interactivo proporciona una vista unificada de la posición de seguridad de las cargas de trabajo de la nube híbrida. Además, muestra alertas de seguridad, información de cobertura y mucho más.

Para ayudarle a ver el estado de seguridad desde una experiencia central, hemos integrado Azure Firewall Manager en este panel. Ahora puede comprobar el estado de cobertura del firewall en todas las redes y administrar de forma centralizada las directivas de Azure Firewall desde Security Center.

Obtenga más información sobre este panel en [Página de información general de Azure Security Center](overview-page.md).

:::image type="content" source="media/release-notes/overview-dashboard-firewall-manager.png" alt-text="Panel de información general de Security Center con un icono para Azure Firewall":::


### <a name="sql-vulnerability-assessment-now-includes-the-disable-rule-experience-preview"></a>La evaluación de vulnerabilidades de SQL ahora incluye la experiencia "Deshabilitación de regla" (versión preliminar)

Security Center incluye un detector de vulnerabilidades integrado para ayudarle a detectar, realizar un seguimiento y corregir posibles vulnerabilidades de la base de datos. Los resultados de los exámenes de evaluación proporcionan información general sobre el estado de seguridad de las máquinas SQL y detalles de las conclusiones sobre seguridad.

Si tiene una necesidad organizativa de omitir un resultado, en lugar de corregirlo, tiene la opción de deshabilitarlo. Los resultados deshabilitados no afectan a la puntuación de seguridad ni generan ruido no deseado.

Obtenga más información en [Deshabilitación de resultados específicos](defender-for-sql-on-machines-vulnerability-assessment.md#disable-specific-findings-preview).



### <a name="azure-monitor-workbooks-integrated-into-security-center-and-three-templates-provided"></a>Se integran en Security Center los libros de Azure Monitor y se proporcionan tres plantillas

En la conferencia Ignite de primavera de 2021, anunciamos la integración de los libros de Azure Monitor en Security Center.

Puede aprovechar la nueva integración para empezar a usar las plantillas predefinidas de la galería de Security Center. Con las plantillas de libro, puede acceder y crear informes visuales y dinámicos para realizar un seguimiento de la posición de seguridad de su organización. Además, puede crear nuevos libros basados en los datos de Security Center o en cualquier otro tipo de datos admitido e implementar rápidamente libros de comunidad desde la comunidad de GitHub de Security Center.

Se proporcionan tres plantillas de informe:

- **Puntuación de seguridad a lo largo del tiempo**: puede realizar el seguimiento de las puntuaciones de las suscripciones y de los cambios en las recomendaciones sobre sus recursos.
- **Actualizaciones del sistema**: puede ver las actualizaciones del sistema que faltan por recursos, sistema operativo, gravedad, etc.
- **Conclusiones de la evaluación de vulnerabilidad**: puede ver los resultados de los exámenes de vulnerabilidades de los recursos de Azure.

Obtenga información sobre el uso de estos informes o la creación de sus propios informes en [Creación de informes enriquecidos e interactivos con los datos de Security Center](custom-dashboards-azure-workbooks.md).

:::image type="content" source="media/custom-dashboards-azure-workbooks/secure-score-over-time-snip.png" alt-text="Puntuación de seguridad a lo largo del tiempo":::


### <a name="regulatory-compliance-dashboard-now-includes-azure-audit-reports-preview"></a>El panel de cumplimiento normativo ahora incluye informes de auditoría de Azure (versión preliminar)

Ahora puede descargar informes de certificación de Azure y Dynamics desde la barra de herramientas del panel de cumplimiento normativo. 

:::image type="content" source="media/release-notes/audit-reports-regulatory-compliance-dashboard.png" alt-text="Barra de herramientas del panel de cumplimiento normativo":::

Puede seleccionar la pestaña correspondiente a cada tipo de informe pertinente (PCI, SOC, ISO y otros) y usar filtros para buscar los informes específicos que necesita.

Más información sobre [cómo administrar los estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md).

:::image type="content" source="media/release-notes/audit-reports-list-regulatory-compliance-dashboard.png" alt-text="Filtrado de la lista de informes de auditoría de Azure disponibles":::



### <a name="recommendation-data-can-be-viewed-in-azure-resource-graph-with-explore-in-arg"></a>Los datos de recomendaciones se pueden ver en Azure Resource Graph con "Explore in ARG" (Explorar en ARG)

Ahora, las páginas de detalles de la recomendación incluyen el botón de la barra de herramientas "Explore in ARG" (Explorar en ARG). Utilice este botón para abrir una consulta de Azure Resource Graph y explorar, exportar y compartir los datos de la recomendación.

Azure Resource Graph (ARG) proporciona acceso instantáneo a la información de los recursos en los entornos en la nube con funcionalidades sólidas para filtrar, agrupar y ordenar. Es una forma rápida y eficaz de consultar información en las suscripciones de Azure mediante programación o desde Azure Portal.

Más información sobre [Azure Resource Graph (ARG)](../governance/resource-graph/index.yml).

:::image type="content" source="media/release-notes/explore-in-resource-graph.png" alt-text="Exploración de los datos de recomendaciones en Azure Resource Graph.":::


### <a name="updates-to-the-policies-for-deploying-workflow-automation"></a>Actualizaciones en las directivas para implementar la automatización de flujos de trabajo

La automatización de los procesos de supervisión y respuesta ante incidentes de la organización puede mejorar considerablemente el tiempo necesario para investigar y mitigar los incidentes de seguridad.

Proporcionamos tres directivas "DeployIfNotExist" de Azure Policy que crean y configuran procedimientos de automatización de flujos de trabajo para que pueda implementar sus automatizaciones en toda la organización:

|Objetivo  |Directiva  |Id. de directiva  |
|---------|---------|---------|
|Automatización de flujos de trabajo para alertas de seguridad|[Implementar la automatización del flujo de trabajo para las alertas de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2ff1525828-9a90-4fcf-be48-268cdd02361e)|f1525828-9a90-4fcf-be48-268cdd02361e|
|Automatización de flujos de trabajo para recomendaciones de seguridad|[Implementar la automatización del área de trabajo para las recomendaciones de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f73d6ab6c-2475-4850-afd6-43795f3492ef)|73d6ab6c-2475-4850-afd6-43795f3492ef|
|Automatización de flujos de trabajo para cambios de cumplimiento normativo|[Implementación de la automatización del flujo de trabajo para el cumplimiento normativo de Azure Security Center](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2fproviders%2fMicrosoft.Authorization%2fpolicyDefinitions%2f73d6ab6c-509122b9-ddd9-47ba-a5f1-d0dac20be63c)|509122b9-ddd9-47ba-a5f1-d0dac20be63c|
||||

Hay dos novedades en las características de estas directivas:

- Cuando se asignan, permanecen habilitadas por cumplimiento.
- Ahora puede personalizar estas directivas y actualizar cualquiera de sus parámetros incluso después de que se hayan implementado. Por ejemplo, si un usuario desea agregar otra clave de evaluación o editar una clave de evaluación existente, puede hacerlo.

Empiece a usar las [plantillas de automatización de flujos de trabajo](https://github.com/Azure/Azure-Security-Center/tree/master/Workflow%20automation).

Puede obtener más información en [Automatización de respuestas a desencadenadores de Security Center](workflow-automation.md).


### <a name="two-legacy-recommendations-no-longer-write-data-directly-to-azure-activity-log"></a>Dos recomendaciones heredadas dejan de escribir datos directamente en el registro de actividad de Azure 

Security Center pasa los datos de casi todas las recomendaciones de seguridad a Azure Advisor que, a su vez, los escribe en el [registro de actividad de Azure](../azure-monitor/essentials/activity-log.md).

Para dos recomendaciones, los datos se escriben de forma simultánea directamente en el registro de actividad de Azure. Con este cambio, Security Center deja de escribir datos para estas recomendaciones de seguridad heredadas directamente en el registro de actividad. En su lugar, exportamos los datos a Azure Advisor como hacemos para las restantes recomendaciones.

Las dos recomendaciones heredadas son:
- Los problemas de estado de protección de puntos de conexión se deben resolver en las máquinas
- Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas

Si ha accedido a la información de estas dos recomendaciones en la categoría "Recomendación de tipo TaskDiscovery" del registro de actividad, ya no estará disponible.


### <a name="recommendations-page-enhancements"></a>Mejoras de la página recomendaciones 

Hemos lanzado una versión mejorada de la lista de recomendaciones para presentar más información que se pueda consultar de un vistazo.

Ahora, en la página verá:

1. La puntuación máxima y la puntuación actual de cada control de seguridad.
1. Hay iconos que reemplazan etiquetas como **Corrección** y **Versión preliminar**.
1. Una columna nueva que muestra la [iniciativa de directiva](security-policy-concept.md) relacionada con cada recomendación: se puede ver cuando "Agrupar por controles" está deshabilitado.

:::image type="content" source="media/release-notes/recommendations-grid-enhancements.png" alt-text="Mejoras en la página de recomendaciones de Azure Security Center: marzo de 2021" lightbox="media/release-notes/recommendations-grid-enhancements.png":::

:::image type="content" source="media/release-notes/recommendations-grid-enhancements-initiatives.png" alt-text="Mejoras en la lista &quot;plana&quot; de recomendaciones de Azure Security Center: marzo de 2021" lightbox="media/release-notes/recommendations-grid-enhancements-initiatives.png":::

Más información en [Recomendaciones de seguridad en el Centro de seguridad de Azure](security-center-recommendations.md)


## <a name="february-2021"></a>Febrero de 2021

Las actualizaciones de febrero incluyen:

- [La nueva página de alertas de seguridad de Azure Portal se ha publicado con disponibilidad general](#new-security-alerts-page-in-the-azure-portal-released-for-general-availability-ga)
- [Publicación de las recomendaciones de protección de cargas de trabajo de Kubernetes para disponibilidad general (GA)](#kubernetes-workload-protection-recommendations-released-for-general-availability-ga)
- [La integración de Microsoft Defender para punto de conexión con Azure Defender ahora es compatible con Windows Server 2019 y Windows 10 Virtual Desktop (WVD) (en versión preliminar)](#microsoft-defender-for-endpoint-integration-with-azure-defender-now-supports-windows-server-2019-and-windows-10-virtual-desktop-wvd-in-preview)
- [Vínculo directo a la directiva desde la página de detalles de la recomendación](#direct-link-to-policy-from-recommendation-details-page)
- [La recomendación de clasificación de datos SQL ya no afecta a la puntuación segura](#sql-data-classification-recommendation-no-longer-affects-your-secure-score)
- [Las automatizaciones de los flujos de trabajo se pueden desencadenar mediante cambios en las evaluaciones de cumplimiento normativo (versión preliminar)](#workflow-automations-can-be-triggered-by-changes-to-regulatory-compliance-assessments-in-preview)
- [Mejoras en la página de inventario de recursos](#asset-inventory-page-enhancements)


### <a name="new-security-alerts-page-in-the-azure-portal-released-for-general-availability-ga"></a>La nueva página de alertas de seguridad de Azure Portal se ha publicado con disponibilidad general

La página de alertas de seguridad de Azure Security Center se ha rediseñado para proporcionar:

- **Mejora en la evaluación de prioridades en las alertas**: ahora es más fácil ayudar a reducir la fatiga de las alertas y centrarse en las amenazas más importantes, la lista incluye filtros personalizables y opciones de agrupación.
- **Más información en la lista de alertas**: por ejemplo las tácticas ATT y ACK de MITRE.
- **Botón para crear alertas de ejemplo**: para evaluar las funcionalidades de Azure Defender y probar la configuración de las alertas (para la integración de SIEM, las notificaciones por correo electrónico y las automatizaciones de los flujos de trabajo), puede crear alertas de ejemplo desde todos los planes de Azure Defender.
- **Alineación con la experiencia de incidentes de Azure Sentinel**: para los clientes que usan ambos productos, cambiar entre ellos ahora es más sencillo y es fácil que uno aprenda del otro.
- **Mejor rendimiento** de listas de alertas grandes.
- **Desplazamiento mediante el teclado** a través de la lista de alertas.
- **Alertas de Azure Resource Graph**: puede consultar las alertas en Azure Resource Graph, la API similar a Kusto para todos los recursos. Esto también resulta útil si va a crear sus propios paneles de alertas. [Más información sobre Azure Resource Graph](../governance/resource-graph/index.yml).
- **Característica Crear alertas de ejemplo**: para crear alertas de ejemplo en la nueva experiencia de alertas, consulte el apartado [Generación de alertas de ejemplo de Azure Defender](security-center-alert-validation.md#generate-sample-azure-defender-alerts).

:::image type="content" source="media/security-center-managing-and-responding-alerts/alerts-page.png" alt-text="Lista de alertas de seguridad de Security Center":::


### <a name="kubernetes-workload-protection-recommendations-released-for-general-availability-ga"></a>Publicación de las recomendaciones de protección de la carga de trabajo de Kubernetes para disponibilidad general (GA)

Nos complace anunciar la disponibilidad general (GA) del conjunto de recomendaciones para las protecciones de la carga de trabajo de Kubernetes.

Para que las cargas de trabajo de Kubernetes sean seguras de forma predeterminada, Security Center ha agregado recomendaciones de protección de nivel de Kubernetes, incluidas opciones de cumplimiento con el control de admisión de Kubernetes.

Cuando el complemento de Azure Policy para Kubernetes está instalado en el clúster de Azure Kubernetes Service (AKS), todas las solicitudes al servidor de la API de Kubernetes se supervisarán con el conjunto predefinido de procedimientos recomendados (que aparecen en forma de 13 recomendaciones de seguridad) antes de que se guarden en el clúster. Después, puede realizar la configurar para aplicar los procedimientos recomendados y exigirlos para futuras cargas de trabajo.

Por ejemplo, puede exigir que no se creen los contenedores con privilegios y que se bloqueen las solicitudes futuras para este fin.

Más información en [Procedimientos recomendados de protección de cargas de trabajo con el control de admisión de Kubernetes](container-security.md#workload-protection-best-practices-using-kubernetes-admission-control)

> [!NOTE]
> Aunque las recomendaciones estaban en versión preliminar, no representaban los recursos de clúster de AKS en mal estado y no se incluían en los cálculos de la puntuación segura. Con este anuncio de disponibilidad general, se incluirán en el cálculo de puntuaciones. Si aún no ha hecho nada al respecto, esto podría afectar negativamente a su puntuación segura. Corríjalas siempre que sea posible como se describe en [Recomendaciones de corrección en Azure Security Center](security-center-remediate-recommendations.md).


### <a name="microsoft-defender-for-endpoint-integration-with-azure-defender-now-supports-windows-server-2019-and-windows-10-virtual-desktop-wvd-in-preview"></a>La integración de Microsoft Defender para punto de conexión con Azure Defender ahora es compatible con Windows Server 2019 y Windows 10 Virtual Desktop (WVD) (en versión preliminar)

Microsoft Defender para punto de conexión es una solución integral de seguridad de punto de conexión que se entrega en la nube. Proporciona funciones de administración y evaluación de vulnerabilidades basadas en riesgos, así como de detección y respuesta de puntos de conexión (EDR). Para obtener una lista completa de las ventajas del uso de Defender para punto de conexión junto con Azure Security Center, consulte [Proteja los puntos de conexión con la solución EDR integrada de Security Center: Microsoft Defender para punto de conexión](security-center-wdatp.md).

Cuando habilita Azure Defender para servidores en un servidor de Windows, el plan incluye una licencia de Defender para punto de conexión. Si ya ha habilitado Azure Defender para servidores y tiene servidores de Windows 2019 en la suscripción, los servidores recibirán automáticamente Defender para punto de conexión con esta actualización. No se requiere ninguna acción manual. 

Ahora se ha ampliado la compatibilidad para incluir Windows Server 2019 y [Windows Virtual Desktop (WVD)](../virtual-desktop/overview.md).

> [!NOTE]
> Si va a habilitar Defender para punto de conexión en un equipo con Windows Server 2019, asegúrese de que cumple los requisitos previos descritos en [Habilitación de la integración de Microsoft Defender para punto de conexión](security-center-wdatp.md#enable-the-microsoft-defender-for-endpoint-integration).

### <a name="direct-link-to-policy-from-recommendation-details-page"></a>Vínculo directo a la directiva desde la página de detalles de la recomendación

Cuando se revisan los detalles de una recomendación, a menudo resulta útil poder ver la directiva subyacente. Para cada recomendación admitida en una directiva, hay un vínculo nuevo en la página de detalles de la recomendación:

:::image type="content" source="media/release-notes/view-policy-definition.png" alt-text="Vínculo a la página de Azure Policy de la directiva específica que admite una recomendación":::

Use este vínculo para ver la definición de directiva y revisar la lógica de evaluación. 

Si va a revisar la lista de recomendaciones de la [guía de referencia de recomendaciones de seguridad](recommendations-reference.md), también verá vínculos a las páginas de definición de directiva:

:::image type="content" source="media/release-notes/view-policy-definition-from-documentation.png" alt-text="Acceso a la página de Azure Policy de una directiva específica directamente desde la página de referencia de recomendaciones de Azure Security Center" lightbox="media/release-notes/view-policy-definition-from-documentation.png":::


### <a name="sql-data-classification-recommendation-no-longer-affects-your-secure-score"></a>La recomendación de clasificación de datos SQL ya no afecta a la puntuación segura
La recomendación **Los datos confidenciales de las bases de datos SQL deben clasificarse** ya no afecta a la puntuación segura. Esta es la única recomendación del control de seguridad **Apply data classification** (Aplicar clasificación de datos), de modo que el control ahora tiene un valor de puntuación segura de 0.

Para obtener una lista completa de todos los controles de seguridad de Security Center, junto con sus puntuaciones y una lista de las recomendaciones de cada uno, consulte [Controles de seguridad y sus recomendaciones](secure-score-security-controls.md#security-controls-and-their-recommendations).

### <a name="workflow-automations-can-be-triggered-by-changes-to-regulatory-compliance-assessments-in-preview"></a>Las automatizaciones de los flujos de trabajo se pueden desencadenar mediante cambios en las evaluaciones de cumplimiento normativo (versión preliminar)
Hemos agregado un tercer tipo de datos a las opciones del desencadenador para las automatizaciones del flujo de trabajo: cambios en las evaluaciones de cumplimiento normativo.

Aprenda a usar las herramientas de automatización del flujo de trabajo en [Automatización de respuestas a desencadenadores de Security Center](workflow-automation.md).

:::image type="content" source="media/release-notes/regulatory-compliance-triggers-workflow-automation.png" alt-text="Uso de los cambios en las evaluaciones de cumplimiento normativo para desencadenar la automatización de un flujo de trabajo" lightbox="media/release-notes/regulatory-compliance-triggers-workflow-automation.png":::


### <a name="asset-inventory-page-enhancements"></a>Mejoras en la página de inventario de recursos
La página de inventario de recursos de Security Center se ha mejorado como se indica a continuación:

- Los resúmenes de la parte superior de la página ahora incluyen **suscripciones no registradas**, que muestran el número de suscripciones sin Security Center habilitado.

    :::image type="content" source="media/release-notes/unregistered-subscriptions.png" alt-text="Recuento de suscripciones no registradas en los resúmenes de la parte superior de la página de inventario de activos":::

- Se han expandido y mejorado los filtros, que incluyen:
    - **Recuentos**: cada filtro presenta el número de recursos que cumplen los criterios de cada categoría.

        :::image type="content" source="media/release-notes/counts-in-inventory-filters.png" alt-text="Recuentos en los filtros en la página del inventario de recursos de Azure Security Center":::.

    - **Contiene filtro de exenciones** (opcional): restrinja los resultados a los recursos que tienen o no tienen exenciones. Este filtro no se muestra de forma predeterminada, pero se puede acceder a él el botón **Agregar filtro**.

        :::image type="content" source="media/release-notes/adding-contains-exemption-filter.gif" alt-text="Incorporación del filtro &quot;contiene exención&quot; en la página del inventario de recursos de Azure Security Center":::

Más información sobre los procedimientos para [explorar y administrar los recursos con el inventario de recursos](asset-inventory.md).

## <a name="january-2021"></a>Enero de 2021

Las actualizaciones de enero incluyen:

- [Azure Security Benchmark es ahora la iniciativa de directivas predeterminada para Azure Security Center](#azure-security-benchmark-is-now-the-default-policy-initiative-for-azure-security-center)
- [La valoración de vulnerabilidades tanto para máquinas locales como las que están en varias nubes se ha publicado en disponibilidad general (GA)](#vulnerability-assessment-for-on-premise-and-multi-cloud-machines-is-released-for-general-availability-ga)
- [Puntuación segura para grupos de administración ya está disponible en versión preliminar](#secure-score-for-management-groups-is-now-available-in-preview)
- [La API de Puntuación segura se ha publicado en disponibilidad general (GA)](#secure-score-api-is-released-for-general-availability-ga)
- [Las protecciones de DNS pendientes se han agregado a Azure Defender para App Service](#dangling-dns-protections-added-to-azure-defender-for-app-service)
- [Los conectores de nubes múltiples se publican en disponibilidad general (GA)](#multi-cloud-connectors-are-released-for-general-availability-ga)
- [Exclusión de recomendaciones completas de su puntuación segura para suscripciones y grupos de administración](#exempt-entire-recommendations-from-your-secure-score-for-subscriptions-and-management-groups)
- [Los usuarios ya pueden solicitar visibilidad en todo el inquilino de su administrador global](#users-can-now-request-tenant-wide-visibility-from-their-global-administrator)
- [Se han agregado 35 recomendaciones en versión preliminar para aumentar la cobertura de Azure Security Benchmark](#35-preview-recommendations-added-to-increase-coverage-of-azure-security-benchmark).
- [Exportación de CSV de una lista filtrada de recomendaciones](#csv-export-of-filtered-list-of-recommendations)
- [Los recursos en estado "No aplicable" ahora se notifican como "Compatible" en las valoraciones de Azure Policy](#not-applicable-resources-now-reported-as-compliant-in-azure-policy-assessments)
- [Exportación de instantáneas semanales de datos de cumplimiento normativo y puntuación segura con exportación continua (versión preliminar)](#export-weekly-snapshots-of-secure-score-and-regulatory-compliance-data-with-continuous-export-preview)


### <a name="azure-security-benchmark-is-now-the-default-policy-initiative-for-azure-security-center"></a>Azure Security Benchmark es ahora la iniciativa de directivas predeterminada para Azure Security Center

Azure Security Benchmark es el conjunto de directrices específico de Azure creado por Microsoft para ofrecer los procedimientos recomendados de seguridad y cumplimiento basados en marcos de cumplimiento comunes. Este punto de referencia, que cuenta con un amplísimo respaldo, se basa en los controles del [Centro de seguridad de Internet (CIS)](https://www.cisecurity.org/benchmark/azure/) y del [Instituto Nacional de Normas y Tecnología (NIST)](https://www.nist.gov/), con un enfoque en seguridad centrada en la nube.

En los últimos meses, la lista de recomendaciones de seguridad integradas de Security Center ha crecido considerablemente para ampliar la cobertura de este punto de referencia.

A partir de esta versión, el punto de referencia es la base de las recomendaciones de Security Center y está totalmente integrado como iniciativa de directivas predeterminada. 

Todos los servicios de Azure tienen una página de línea base de seguridad en su documentación. Por ejemplo, [esta es la línea base de Security Center](security-baseline.md). Estas líneas base se basan en el Azure Security Benchmark.

Si usa el panel de cumplimiento normativo de Security Center, verá dos instancias del punto de referencia durante un período de transición:

:::image type="content" source="media/release-notes/regulatory-compliance-with-azure-security-benchmark.png" alt-text="Panel de cumplimiento normativo de Security Center que muestra Azure Security Benchmark":::

Las recomendaciones existentes no se ven afectadas y a medida que crezca el punto de referencia, los cambios se reflejarán automáticamente en Security Center. 

Para más información, consulte las páginas siguientes:

- [Más información sobre Azure Security Benchmark](/security/benchmark/azure/introduction)
- [Personalización del conjunto de estándares en el panel de cumplimiento normativo](update-regulatory-compliance-packages.md)

### <a name="vulnerability-assessment-for-on-premise-and-multi-cloud-machines-is-released-for-general-availability-ga"></a>La valoración de vulnerabilidades tanto para máquinas locales como las que están en varias nubes se ha publicado en disponibilidad general (GA)

En octubre, se anunció una versión preliminar para examinar servidores con Azure Arc habilitado con el analizador de evaluación de vulnerabilidades integrado de [Azure Defender para servidores](defender-for-servers-introduction.md) (con tecnología de Qualys).

Ahora se ha publicado en disponibilidad general (GA).

Al habilitar Azure Arc en máquinas que no son de Azure, Security Center le ofrecerá la opción de implementar en ellas el analizador de vulnerabilidades integrado, manualmente y a escala.

Con esta actualización, puede aprovechar toda la capacidad de **Azure Defender para servidores** para consolidar su programa de administración de vulnerabilidades en todos sus recursos, tanto si son de Azure como si no.

Principales funcionalidades:

- Supervisión del estado de aprovisionamiento del analizador de evaluación de vulnerabilidades en máquinas de Azure Arc.
- Aprovisionamiento del agente de evaluación de vulnerabilidades integrado en máquinas de Azure Arc Windows y Linux (manualmente y a escala).
- Recepción y análisis de vulnerabilidades detectadas por agentes implementados (manualmente y a escala).
- Experiencia unificada para máquinas virtuales de Azure y máquinas de Azure Arc.

[Obtenga más información sobre la implementación del analizador de vulnerabilidades integrado en las máquinas híbridas](deploy-vulnerability-assessment-vm.md#deploy-the-integrated-scanner-to-your-azure-and-hybrid-machines).

[Obtenga más información sobre los servidores habilitados para Azure Arc](../azure-arc/servers/index.yml).


### <a name="secure-score-for-management-groups-is-now-available-in-preview"></a>Puntuación segura para grupos de administración ya está disponible en versión preliminar

La página de puntuación segura ahora muestra las puntuaciones seguras agregadas para los grupos de administración, además del nivel de suscripción. Por consiguiente, ahora puede ver la lista de grupos de administración de la organización y la puntuación de cada uno de ellos.

:::image type="content" source="media/secure-score-security-controls/secure-score-management-groups.png" alt-text="Visualización de las puntuaciones seguras de los grupos de administración.":::

Obtenga más información sobre la [puntuación segura y los controles de seguridad en Azure Security Center](secure-score-security-controls.md).

### <a name="secure-score-api-is-released-for-general-availability-ga"></a>La API de Puntuación segura se ha publicado en disponibilidad general (GA)

Ahora puede acceder a la puntuación a través de la [API de puntuación segura](/rest/api/securitycenter/securescores/). Los métodos de API proporcionan la flexibilidad necesaria para consultar los datos y crear su propio mecanismo de creación de informes de las puntuaciones seguras a lo largo del tiempo. Por ejemplo:

- use la API **Secure Scores** para obtener la puntuación de una suscripción específica.
- use la API **Secure Score Controls** para mostrar los controles de seguridad y la puntuación actual de las suscripciones.

Conozca las herramientas externas que son posibles gracias a la API de puntuación segura en el [área de puntuación segura de la comunidad de GitHub](https://github.com/Azure/Azure-Security-Center/tree/master/Secure%20Score).

Obtenga más información sobre la [puntuación segura y los controles de seguridad en Azure Security Center](secure-score-security-controls.md).


### <a name="dangling-dns-protections-added-to-azure-defender-for-app-service"></a>Las protecciones de DNS pendientes se han agregado a Azure Defender para App Service

Las adquisiciones de subdominios son una amenaza muy grave que se producen de forma común en las organizaciones. Una adquisición de subdominio puede producirse cuando se tiene un registro DNS que apunta a un sitio web desaprovisionado. Estos registros DNS también se conocen como entradas "DNS pendientes". Los registros CNAME son especialmente vulnerables a esta amenaza. 

Las adquisiciones de subdominios permiten a los actores de las amenazas redirigir el tráfico destinado al dominio de una organización a un sitio que realiza una actividad malintencionada.

Azure Defender para App Service ahora detecta las entradas DNS pendientes cuando se retira un sitio web de App Service. Este es el momento en el que la entrada DNS apunta a un recurso que no existe y su sitio web es vulnerable a la adquisición del subdominio. Estas protecciones están disponibles tanto si los dominios se administran con Azure DNS como con un registrador de dominios externo y se aplican a App Service en Windows y en Linux.

Más información:

- [Tabla de referencia de alertas de App Service](alerts-reference.md#alerts-azureappserv): incluye dos nuevas alertas de Azure Defender que se desencadenan cuando se detecta una entrada DNS pendiente
- [Evitar las entradas DNS pendientes y evitar la adquisición de subdominios](../security/fundamentals/subdomain-takeover.md): obtenga información sobre la amenaza de la adquisición de subdominios y el aspecto de los DNS pendientes
- [Introducción a Azure Defender para App Service](defender-for-app-service-introduction.md)


### <a name="multi-cloud-connectors-are-released-for-general-availability-ga"></a>Los conectores de nubes múltiples se publican en disponibilidad general (GA)

Las cargas de trabajo de nube abarcan normalmente varias plataformas de nube, por lo que los servicios de seguridad de la nube deben hacer lo mismo.

Azure Security Center protege las cargas de trabajo de Azure, Amazon Web Services (AWS) y Google Cloud Platform (GCP).

La conexión de las cuentas de AWS o GCP integra sus herramientas de seguridad nativas, como AWS Security Hub y GCP Security Center, en Azure Security Center.

Esta capacidad significa que Security Center proporciona visibilidad y protección en los principales entornos de nube. Algunas de las ventajas de esta integración son:

- Aprovisionamiento automático de agentes (Security Center usa Azure Arc para implementar el agente de Log Analytics en las instancias de AWS)
- Administración de directivas
- Administración de vulnerabilidades
- Detección y respuesta de puntos de conexión (EDR) integrados
- Detección de errores de configuración de seguridad
- Una vista única que muestra recomendaciones de seguridad de todos los proveedores de nube
- Incorporación de todos los recursos a los cálculos de puntuación segura de Security Center
- Evaluaciones de cumplimiento normativo de los recursos de AWS y GCP

En el menú de Security Center, seleccione **Multi cloud connectors** (Conectores en varias nubes) y verá las opciones para crear conectores:

:::image type="content" source="./media/quickstart-onboard-aws/add-aws-account.png" alt-text="Botón Add AWS account (Agregar cuenta de AWS) de la página Conectores multinube de Security Center":::

Puede encontrar más información en:
- [Conexión de las cuentas de AWS a Azure Security Center](quickstart-onboard-aws.md)
- [Conexión de las cuentas de GCP a Azure Security Center](quickstart-onboard-gcp.md)


### <a name="exempt-entire-recommendations-from-your-secure-score-for-subscriptions-and-management-groups"></a>Exclusión de recomendaciones completas de su puntuación segura para suscripciones y grupos de administración

Estamos ampliando la capacidad de exención para incluir recomendaciones completas. Proporcionar más opciones para ajustar las recomendaciones de seguridad que Security Center realiza para las suscripciones, el grupo de administración o los recursos.

En ocasiones, un recurso se mostrará como incorrecto cuando se sepa que el problema se ha resuelto mediante una herramienta de terceros que Security Center no ha detectado. O bien se mostrará una recomendación en un ámbito al que cree que no pertenece. La recomendación podría no ser adecuada para una suscripción concreta. O quizás la organización ha decidido aceptar los riesgos relacionados con la recomendación o el recurso específicos.

Con esta característica en vista previa (GB), ahora puede crear una exención para que una recomendación:

- **Excluya un recurso** para asegurarse de que no aparezca con los recursos incorrectos en el futuro y no afecte a la puntuación de seguridad. El recurso se mostrará como no aplicable y el motivo aparecerá como "exento" con la justificación específica que seleccione.

- **Excluir una suscripción o un grupo de administración** para asegurarse de que la recomendación no afecte a la puntuación de seguridad y no que no se muestre para la suscripción o el grupo de administración en el futuro. Esto se refiere a los recursos existentes y cualquiera que cree en el futuro. La recomendación se marcará con la justificación específica que seleccione para el ámbito que ha seleccionado.

Puede obtener más información en [Exención de recursos y recomendaciones de la puntuación de seguridad](exempt-resource.md).



### <a name="users-can-now-request-tenant-wide-visibility-from-their-global-administrator"></a>Los usuarios ya pueden solicitar visibilidad en todo el inquilino de su administrador global

Si algún usuario no tiene los permisos necesarios para ver los datos de Security Center, ahora verá un vínculo para solicitar permisos al administrador global de su organización. La solicitud incluye el rol que desean y la justificación de por qué es necesario.

:::image type="content" source="media/security-center-management-groups/request-tenant-permissions.png" alt-text="Banner que informa a los usuarios de que pueden solicitar permisos para todo el inquilino.":::

Para más información, consulte [Solicitud de permisos para todo el inquilino cuando el suyo sea insuficiente](tenant-wide-permissions-management.md#request-tenant-wide-permissions-when-yours-are-insufficient).


### <a name="35-preview-recommendations-added-to-increase-coverage-of-azure-security-benchmark"></a>Se han agregado 35 recomendaciones en versión preliminar para aumentar la cobertura de Azure Security Benchmark.

[Azure Security Benchmark](/security/benchmark/azure/introduction) es la iniciativa de directivas predeterminada de Azure Security Center. 

Para aumentar la cobertura del punto de referencia se han agregado a Security Center las siguientes 35 recomendaciones en versión preliminar.

> [!TIP]
> Las recomendaciones de la versión preliminar no representan un recurso incorrecto y no se incluyen en los cálculos de una puntuación segura. Corríjalas siempre que sea posible, de tal forma que, cuando finalice el período de versión preliminar, contribuyan a la puntuación. Puede encontrar más información sobre cómo responder a estas recomendaciones en [Recomendaciones de corrección en Azure Security Center](security-center-remediate-recommendations.md).

| Control de seguridad                     | Nuevas recomendaciones                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|--------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Habilitación del cifrado de datos en reposo            | - Las cuentas de Azure Cosmos DB deben usar claves administradas por el cliente para cifrar los datos en reposo.<br>- Las áreas de trabajo de Azure Machine Learning deben cifrarse con una clave administrada por el cliente (CMK).<br>- La protección de datos de Bring Your Own Key debe estar habilitada para los servidores MySQL.<br>- La protección de datos de Bring Your Own Key debe estar habilitada para los servidores PostgreSQL.<br>- Las cuentas de Cognitive Services deben habilitar el cifrado de datos con una clave administrada por el cliente (CMK).<br>- Las instancias de Container Registry se deben cifrar con una clave administrada por el cliente (CMK).<br>- Las instancias administradas de SQL deben usar claves administradas por el cliente para cifrar los datos en reposo.<br>- Los servidores SQL deben usar claves administradas por el cliente para cifrar los datos en reposo.<br>- Las cuentas de almacenamiento deben usar la clave administrada por el cliente (CMK) para el cifrado.                                                                                                                                                              |
| Implementación de procedimientos recomendados de seguridad    | - Las suscripciones deben tener una dirección de correo electrónico de contacto para los problemas de seguridad.<br> - El aprovisionamiento automático del agente de Log Analytics debe estar habilitado en su suscripción.<br> - La opción para enviar notificaciones por correo electrónico para alertas de gravedad alta debe estar habilitada.<br> - La opción para enviar notificaciones por correo electrónico al propietario de la suscripción en relación a alertas de gravedad alta debe estar habilitada.<br> - Los almacenes de claves deben tener habilitada la protección contra operaciones de purga.<br> - Los almacenes de claves deben tener habilitada la eliminación temporal. |
| Administración de acceso y permisos        | - Las aplicaciones de funciones deben tener la opción "Certificados de cliente (certificados de cliente entrantes)" habilitada. |
| Protección de aplicaciones contra ataques DDoS | - Web Application Firewall (WAF) debe estar habilitado para Application Gateway.<br> - Web Application Firewall (WAF) debe estar habilitado para Azure Front Door Service. |
| Restricción de los accesos de red no autorizados | - El firewall debe estar habilitado en Key Vault.<br> - Se debe configurar un punto de conexión privado para Key Vault.<br> - App Configuration debe usar un vínculo privado.<br> - Azure Cache for Redis debe residir en una red virtual.<br> - Los dominios de Azure Event Grid deben usar un vínculo privado.<br> - Los temas de Azure Event Grid deben usa un vínculo privado.<br> - Las áreas de trabajo de Azure Machine Learning deben usar un vínculo privado.<br> Azure SignalR Service debe usar un vínculo privado.<br> - Azure Spring Cloud debe usar la inserción de red.<br> -Las instancias de Container Registry no deben permitir el acceso de red sin restricciones.<br> - Las instancias de Container Registry deben usar vínculo privado.<br> -El acceso a redes públicas debe estar deshabilitado para los servidores de MariaDB.<br> El acceso a las redes públicas debe estar deshabilitado para los servidores de MySQL<br> - El acceso a redes públicas debe estar deshabilitado para los servidores de PostgreSQL.<br> - La cuenta de almacenamiento debería utilizar una conexión de vínculo privado<br> - Las cuentas de almacenamiento deben restringir el acceso a la red mediante el uso de reglas de red virtual.<br> - Las plantillas de VM Image Builder deben usar un vínculo privado.|
|                                      |                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |

Vínculos relacionados:

- [Más información sobre Azure Security Benchmark](/security/benchmark/azure/introduction)
- [Más información sobre Azure Database for MariaDB](../mariadb/overview.md)
- [Más información sobre Azure Database for MySQL](../mysql/overview.md)
- [Más información sobre Azure Database for PostgreSQL](../postgresql/overview.md)




### <a name="csv-export-of-filtered-list-of-recommendations"></a>Exportación de CSV de una lista filtrada de recomendaciones 

En noviembre de 2020, hemos agregado filtros a la página de recomendaciones (consulte [La lista de recomendaciones ahora incluye filtros](release-notes-archive.md#recommendations-list-now-includes-filters)). En diciembre, hemos expandido esos filtros ([La página Recomendaciones tiene nuevos filtros de entorno, gravedad y respuestas disponibles](#recommendations-page-has-new-filters-for-environment-severity-and-available-responses)). 

Con este anuncio, vamos a cambiar el comportamiento del botón **Descargar como CSV** para que la exportación de CSV solo incluya las recomendaciones que se muestran actualmente en la lista filtrada. 

Por ejemplo, en la imagen siguiente puede ver que la lista se ha filtrado a dos recomendaciones. El archivo CSV que se genera incluye los detalles de estado de cada recurso afectado por esas dos recomendaciones.   

:::image type="content" source="media/security-center-managing-and-responding-alerts/export-to-csv-with-filters.png" alt-text="Exportación de recomendaciones filtradas a un archivo CSV":::

Más información en [Recomendaciones de seguridad en el Centro de seguridad de Azure](security-center-recommendations.md)


### <a name="not-applicable-resources-now-reported-as-compliant-in-azure-policy-assessments"></a>Los recursos en estado "No aplicable" ahora se notifican como "Compatible" en las valoraciones de Azure Policy

Anteriormente, los recursos que se evaluaban para una recomendación y se detectaba que eran **no aplicables** aparecían en in Azure Policy como "No compatible". Ninguna acción del usuario podría cambiar su estado a "Compatible". Con este cambio, se notificarán como "Compatible" para aumentar la claridad.

El único impacto se verá en Azure Policy, donde el número de recursos compatibles aumentará. Esto no afectará a la puntuación de seguridad de Azure Security Center.


### <a name="export-weekly-snapshots-of-secure-score-and-regulatory-compliance-data-with-continuous-export-preview"></a>Exportación de instantáneas semanales de datos de cumplimiento normativo y puntuación segura con exportación continua (versión preliminar)

Hemos agregado una nueva característica en versión preliminar a las herramientas de [exportación continua](continuous-export.md) para exportar instantáneas semanales de datos de cumplimiento normativo y puntuación segura.

Al definir una exportación continua, establezca la frecuencia de exportación:

:::image type="content" source="media/release-notes/export-frequency.png" alt-text="Elección de la frecuencia de la exportación continua":::

- **Streaming**: las evaluaciones se enviarán en tiempo real cuando se actualice el estado de mantenimiento de un recurso (si no se produce ninguna actualización, no se enviarán datos).
- **Instantáneas**: se enviará una instantánea del estado actual de todas las evaluaciones de cumplimiento normativo cada semana (se trata de una característica en versión preliminar para las instantáneas semanales de datos de cumplimiento normativo y puntuaciones seguras).

Puede encontrar más información sobre las funcionalidades completas de esta característica en [Exportación continua de alertas y recomendaciones de seguridad](continuous-export.md).

## <a name="december-2020"></a>Diciembre de 2020

Las actualizaciones de diciembre incluyen:

- [Disponibilidad general de Azure Defender para servidores SQL Server en máquinas](#azure-defender-for-sql-servers-on-machines-is-generally-available)
- [Compatibilidad de Azure Defender para SQL para el grupo de SQL dedicado de Azure Synapse Analytics disponible con carácter general](#azure-defender-for-sql-support-for-azure-synapse-analytics-dedicated-sql-pool-is-generally-available)
- [Ahora, los administradores globales se pueden conceder a sí mismos permisos de nivel de inquilino](#global-administrators-can-now-grant-themselves-tenant-level-permissions)
- [Dos nuevos planes de Azure Defender: Azure Defender para DNS y Azure Defender para Resource Manager (en versión preliminar)](#two-new-azure-defender-plans-azure-defender-for-dns-and-azure-defender-for-resource-manager-in-preview)
- [Nueva página de alertas de seguridad en Azure Portal (versión preliminar)](#new-security-alerts-page-in-the-azure-portal-preview)
- [Experiencia de Security Center revitalizada en Azure SQL Database y SQL Managed Instance](#revitalized-security-center-experience-in-azure-sql-database--sql-managed-instance)
- [Se han actualizado las herramientas y filtros para el inventario de recursos](#asset-inventory-tools-and-filters-updated)
- [La recomendación acerca de las aplicaciones web que solicitan certificados SSL ya no forma parte de la puntuación segura](#recommendation-about-web-apps-requesting-ssl-certificates-no-longer-part-of-secure-score)
- [La página Recomendaciones tiene nuevos filtros de entorno, gravedad y respuestas disponibles](#recommendations-page-has-new-filters-for-environment-severity-and-available-responses)
- [La exportación continua obtiene nuevos tipos de datos y directivas deployifnotexist mejoradas](#continuous-export-gets-new-data-types-and-improved-deployifnotexist-policies)


### <a name="azure-defender-for-sql-servers-on-machines-is-generally-available"></a>Disponibilidad general de Azure Defender para servidores SQL Server en máquinas

Azure Security Center ofrece dos planes de Azure Defender para servidores SQL Server:

- **Azure Defender para servidores de bases de datos Azure SQL**: defiende los servidores SQL Server nativos de Azure. 
- **Azure Defender para servidores SQL Server en máquinas**: amplía las mismas protecciones a los servidores SQL Server en entornos híbridos, multinube y locales.

Con este anuncio, **Azure Defender para SQL** ahora protege las bases de datos y sus datos dondequiera que se encuentren.

Azure Defender para SQL incluye funcionalidades de evaluación de vulnerabilidades. La herramienta de evaluación de vulnerabilidades incluye las siguientes características avanzadas:

- La **configuración de línea de base** (nuevo) para limitar de forma inteligente los resultados de los análisis de vulnerabilidades a aquellas que podrían representar problemas de seguridad reales. Una vez establecido el estado de seguridad de la línea de base, la herramienta de evaluación de vulnerabilidades solo informa de las desviaciones con respecto a esa línea de base. Los resultados que coinciden con la línea de base se consideran como correctos en análisis posteriores. Esto le permite a usted y a sus analistas centrar la atención en lo que importa.
- **Información comparativa detallada** para ayudarle a *comprender* los hechos detectados y por qué se relacionan con los recursos.
- **Scripts de corrección** para ayudarle a mitigar los riesgos identificados.

Más información sobre [Azure Defender for SQL](defender-for-sql-introduction.md).


### <a name="azure-defender-for-sql-support-for-azure-synapse-analytics-dedicated-sql-pool-is-generally-available"></a>Compatibilidad de Azure Defender para SQL para Azure Synapse Analytics disponible con carácter general

Azure Synapse Analytics (anteriormente SQL Data Warehouse) es un servicio de análisis que combina almacenamiento de datos empresariales con análisis de macrodatos. Los grupos de SQL dedicados son las características de almacenamiento de datos empresariales de Azure Synapse. Más información en [¿Qué es Azure Synapse Analytics (anteriormente SQL Data Warehouse)?](../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md)

Azure Defender para SQL protege los grupos de SQL dedicados con:

- **Protección contra amenazas avanzada** para detectar amenazas y ataques. 
- **Funcionalidades de evaluación de vulnerabilidades** para identificar y corregir configuraciones erróneas de seguridad.

La compatibilidad de Azure Defender para SQL con los grupos de SQL de Azure Synapse Analytics se agrega automáticamente al paquete de bases de datos SQL de Azure en Azure Security Center. Encontrará una nueva pestaña "Azure Defender para SQL" en la página del área de trabajo de Synapse en Azure Portal.

Más información sobre [Azure Defender for SQL](defender-for-sql-introduction.md).


### <a name="global-administrators-can-now-grant-themselves-tenant-level-permissions"></a>Ahora, los administradores globales se pueden conceder a sí mismos permisos de nivel de inquilino.

Un usuario con el rol de **administrador global** de Azure Active Directory puede tener responsabilidades en todo el inquilino, pero carecer de los permisos de Azure para ver la información de toda la organización en Azure Security Center. 

Para asignarse a sí mismo permisos de nivel de inquilino, siga las instrucciones que se indican en [Concederse a uno mismo permisos para todo el inquilino](tenant-wide-permissions-management.md#grant-tenant-wide-permissions-to-yourself).


### <a name="two-new-azure-defender-plans-azure-defender-for-dns-and-azure-defender-for-resource-manager-in-preview"></a>Dos nuevos planes de Azure Defender: Azure Defender para DNS y Azure Defender para Resource Manager (en versión preliminar)

Se han agregado dos nuevas funcionalidades de protección contra amenazas con amplitud nativa de nube para el entorno de Azure.

Estas nuevas protecciones mejoran enormemente la resistencia frente a los ataques de actores de amenazas y aumentan considerablemente el número de recursos de Azure protegidos por Azure Defender.

- **Azure Defender para Resource Manager**: supervisa automáticamente todas las operaciones de administración de recursos realizadas en la organización. Para más información, consulte:
    - [Introducción a Azure Defender para Resource Manager](defender-for-resource-manager-introduction.md)
    - [Respuesta a las alertas de Azure Defender para Resource Manager](defender-for-resource-manager-usage.md)
    - [Lista de alertas proporcionadas por Azure Defender para Resource Manager](alerts-reference.md#alerts-resourcemanager)

- **Azure Defender para DNS**: supervisa continuamente todas las consultas de DNS de los recursos de Azure. Para más información, consulte:
    - [Introducción a Azure Defender para DNS](defender-for-dns-introduction.md)
    - [Respuesta a las alertas de Azure Defender para DNS](defender-for-dns-usage.md)
    - [Lista de alertas proporcionadas por Azure Defender para DNS](alerts-reference.md#alerts-dns)


### <a name="new-security-alerts-page-in-the-azure-portal-preview"></a>Nueva página de alertas de seguridad en Azure Portal (versión preliminar)

La página de alertas de seguridad de Azure Security Center se ha rediseñado para proporcionar:

- **Mejora en la evaluación de prioridades en las alertas**: ahora es más fácil ayudar a reducir la fatiga de las alertas y centrarse en las amenazas más importantes, la lista incluye filtros personalizables y opciones de agrupación.
- **Más información en la lista de alertas**: por ejemplo las tácticas ATT y ACK de MITRE
- **Botón para crear alertas de ejemplo**: para evaluar las funcionalidades de Azure Defender y probar la configuración de las alertas (para la integración de SIEM, las notificaciones por correo electrónico y las automatizaciones de los flujos de trabajo), puede crear alertas de ejemplo desde todos los planes de Azure Defender.
- **Alineación con la experiencia de incidentes de Azure Sentinel**: para los clientes que usan ambos productos, cambiar entre ellos ahora es más sencillo y es fácil que uno aprenda del otro.
- **Mejor rendimiento** de listas de alertas grandes.
- **Desplazamiento mediante el teclado** a través de la lista de alertas.
- **Alertas de Azure Resource Graph**: puede consultar las alertas en Azure Resource Graph, la API similar a Kusto para todos los recursos. Esto también resulta útil si va a crear sus propios paneles de alertas. [Más información sobre Azure Resource Graph](../governance/resource-graph/index.yml).

Para acceder a la nueva experiencia, use el vínculo "Intentarlo ahora" del banner de la parte superior de la página de alertas de seguridad.

:::image type="content" source="media/security-center-managing-and-responding-alerts/preview-alerts-experience-banner.png" alt-text="Banner con un vínculo a la versión preliminar de la nueva experiencia de alertas":::

Para crear alertas de ejemplo a partir de la nueva experiencia de alertas, consulte el apartado [Generación de alertas de ejemplo de Azure Defender](security-center-alert-validation.md#generate-sample-azure-defender-alerts).


### <a name="revitalized-security-center-experience-in-azure-sql-database--sql-managed-instance"></a>Experiencia de Security Center revitalizada en Azure SQL Database y SQL Managed Instance 

La experiencia de Security Center en SQL proporciona acceso a las siguientes características de Security Center y Azure Defender para SQL:

- **Recomendaciones de seguridad**: Azure Security Center analiza periódicamente el estado de seguridad de todos los recursos de Azure conectados para identificar posibles configuraciones erróneas de seguridad. A continuación, proporciona recomendaciones sobre cómo corregir esas vulnerabilidades y mejorar la postura de seguridad de las organizaciones.
- **Alertas de seguridad**: un servicio de detección que supervisa continuamente las actividades de Azure SQL para detectar amenazas tales como la inyección de código SQL, ataques por fuerza bruta y abuso de privilegios. Este servicio desencadena alertas de seguridad detalladas y orientadas a acciones en Security Center, y proporciona opciones para continuar las investigaciones con Azure Sentinel, la solución SIEM nativa de Microsoft Azure.
- **Conclusiones**: un servicio de evaluación de vulnerabilidades que supervisa continuamente las configuraciones de Azure SQL y ayuda a corregir las vulnerabilidades. Los análisis de evaluación proporcionan una visión general de los estados de seguridad de Azure SQL junto con conclusiones detalladas de seguridad.     

:::image type="content" source="media/release-notes/azure-security-center-experience-in-sql.png" alt-text="Las características de seguridad de Azure Security Center para SQL están disponibles en Azure SQL":::


### <a name="asset-inventory-tools-and-filters-updated"></a>Se han actualizado las herramientas y filtros para el inventario de recursos

La página Inventario de Azure Security Center se ha actualizado con los cambios siguientes:

- Se han agregado **guías y comentarios** a la barra de herramientas. Esto permite abrir un panel con vínculos a información y herramientas relacionados. 
- Se ha agregado un **filtro de suscripciones** a los filtros predeterminados disponibles para los recursos.
- Se ha agregado el enlace **Abrir consulta** para abrir las opciones de filtro actuales como una consulta de Azure Resource Graph (anteriormente se denominaba "Ver en Resource Graph Explorer").
- **Opciones de operador** para cada filtro. Ahora puede elegir entre más operadores lógicos adicionales distintos de "=". Por ejemplo, puede que desee buscar todos los recursos con recomendaciones activas cuyos títulos incluyan la cadena "Encrypt". 

    :::image type="content" source="media/release-notes/inventory-filter-operators.png" alt-text="Controles para la opción de operador en los filtros de Asset Inventory":::

Más información acerca del inventario en [Exploración y administración de los recursos con Asset Inventory](asset-inventory.md).


### <a name="recommendation-about-web-apps-requesting-ssl-certificates-no-longer-part-of-secure-score"></a>La recomendación acerca de las aplicaciones web que solicitan certificados SSL ya no forma parte de la puntuación segura

La recomendación "Las aplicaciones web deben solicitar un certificado SSL a todas las solicitudes entrantes" se ha trasladado desde el control de seguridad **Administración de acceso y permisos** (con una puntuación máxima de 4 puntos) a **Implementación de procedimientos recomendados de seguridad** (con ningún punto). 

Asegurarse de que una aplicación web solicita un certificado ciertamente hace que sea más segura. Sin embargo, para las aplicaciones web de acceso público, esto es irrelevante. Si accede a su sitio a través de HTTP y no de HTTPS, no recibirá ningún certificado de cliente. Por lo tanto, si la aplicación requiere certificados de cliente, no debe permitir solicitudes a la aplicación mediante HTTP. Puede encontrar más información en [Configuración de la autenticación mutua de TLS en Azure App Service](../app-service/app-service-web-configure-tls-mutual-auth.md).

Con este cambio, la recomendación es ahora un procedimiento recomendado que no influye en la puntuación. 

Obtenga información sobre qué recomendaciones se encuentran en cada control de seguridad en [Controles de seguridad y sus recomendaciones](secure-score-security-controls.md#security-controls-and-their-recommendations).


### <a name="recommendations-page-has-new-filters-for-environment-severity-and-available-responses"></a>La página Recomendaciones tiene nuevos filtros de entorno, gravedad y respuestas disponibles

Azure Security Center supervisa todos los recursos conectados y genera recomendaciones de seguridad. Use estas recomendaciones para fortalecer su postura en la nube híbrida y realizar un seguimiento del cumplimiento de las directivas y los estándares importantes para su organización, sector y país.

A medida que Security Center amplía su cobertura y características, la lista de recomendaciones de seguridad crece cada mes. Por ejemplo, consulte [Se han agregado 29 recomendaciones en versión preliminar para aumentar la cobertura de Azure Security Benchmark](release-notes-archive.md#29-preview-recommendations-added-to-increase-coverage-of-azure-security-benchmark).

Con una lista en continuo crecimiento, existe la necesidad de filtrar las recomendaciones para encontrar las de mayor interés. En noviembre, hemos agregado filtros a la página Recomendaciones (consulte [La lista de recomendaciones ahora incluye filtros](release-notes-archive.md#recommendations-list-now-includes-filters)).

Los filtros que se han agregado este mes proporcionan opciones para refinar la lista de recomendaciones según:

- El **entorno**: vea las recomendaciones para los recursos de AWS, GCP o Azure (o cualquier combinación).
- La **gravedad**: vea las recomendaciones según la clasificación de gravedad establecida por Security Center.
- **Acciones de respuesta**: consulte las recomendaciones según la disponibilidad de las opciones de respuesta de Security Center: corrección, denegación y aplicación

    > [!TIP]
    > El filtro de acciones de respuesta reemplaza al filtro **Solución rápida disponible (Sí/No)** . 
    > 
    > Puede aprender más información acerca de cada una de estas opciones de respuesta:
    > - [Botón Corregir](security-center-remediate-recommendations.md#fix-button)
    > - [Evitar errores de configuración con las recomendaciones Aplicar o Denegar](prevent-misconfigurations.md).

:::image type="content" source="./media/release-notes/added-recommendations-filters.png" alt-text="Recomendaciones agrupadas por control de seguridad" lightbox="./media/release-notes/added-recommendations-filters.png":::

### <a name="continuous-export-gets-new-data-types-and-improved-deployifnotexist-policies"></a>La exportación continua obtiene nuevos tipos de datos y directivas deployifnotexist mejoradas

Las herramientas de exportación continua de Azure Security Center permiten exportar las recomendaciones y alertas de Security Center para su uso con otras herramientas de supervisión de su entorno.

La exportación continua le permite personalizar completamente qué se exportará y a dónde irá. Para más información, consulte [Exportación continua de datos de Security Center](continuous-export.md).

Estas herramientas se han mejorado y ampliado de las siguientes maneras:

- **Se han mejorado las directivas deployifnotexist de la exportación continua**. Las directivas ahora:

    - **Comprueban si la configuración está habilitada.** Si no es así, la directiva aparecerá como no compatible y creará un recurso compatible. Obtenga más información acerca de las plantillas de Azure Policy proporcionadas en la pestaña "Implementación a gran escala con Azure Policy" de[Configuración de una exportación continua](continuous-export.md#set-up-a-continuous-export).

    - **Compatibilidad con la exportación de conclusiones de seguridad.** Cuando usa las plantillas de Azure Policy, puede configurar la exportación continua para que incluya conclusiones. Esto es importante cuando se exportan recomendaciones que incluyen "subrecomendaciones", como en el caso de los resultados de los escáneres de evaluación de vulnerabilidades o las actualizaciones específicas del sistema para la recomendación principal "Se deben instalar actualizaciones del sistema en las máquinas".
    
    - **Compatibilidad con la exportación de datos de puntuación de seguridad.**

- **Se han agregado datos de evaluación de cumplimiento normativo (en versión preliminar).** Ahora puede exportar continuamente las actualizaciones de las evaluaciones de cumplimiento normativo, incluidas las iniciativas personalizadas, a un área de trabajo Log Analytics o a un centro de eventos. Esta característica no está disponible en las nubes nacionales o soberanas.

    :::image type="content" source="media/release-notes/continuous-export-regulatory-compliance-option.png" alt-text="Opciones para incluir información de evaluación conforme a la normativa con los datos de exportación continua.":::
