---
title: Identificación de amenazas avanzadas con el Análisis de comportamiento de usuarios y entidades (UEBA) en Azure Sentinel | Microsoft Docs
description: Cree líneas de base de comportamiento para entidades (usuarios, nombres de host y direcciones IP) y úselas para detectar comportamientos anómalos e identificar amenazas persistentes avanzadas (APT) de día cero.
services: sentinel
documentationcenter: na
author: yelevin
manager: rkarlin
editor: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 04/19/2021
ms.author: yelevin
ms.openlocfilehash: 64c921f1ad401baaf12b1545bfc2bdda54ab3444
ms.sourcegitcommit: 19dfdfa85e92c6a34933bdd54a7c94e8b00eacfd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109664391"
---
# <a name="identify-advanced-threats-with-user-and-entity-behavior-analytics-ueba-in-azure-sentinel"></a>Identificación de amenazas avanzadas con el Análisis de comportamiento de usuarios y entidades (UEBA) en Azure Sentinel

> [!IMPORTANT]
>
> - Las características de UEBA y de las páginas de entidad ahora están en **disponibilidad general** en **_todas_** las zonas geográficas y regiones de Azure Sentinel.

## <a name="what-is-user-and-entity-behavior-analytics-ueba"></a>¿Qué es el Análisis de comportamiento de usuarios y entidades (UEBA)?

La identificación de amenazas dentro de su organización y de su posible impacto, tanto si se trata de una entidad en peligro como de un usuario interno malintencionado, siempre ha sido un proceso que requiere mucho tiempo y trabajo. Examinar las alertas, unir los puntos y buscar de manera activa, en su conjunto, obligan a invertir una gran cantidad de tiempo y esfuerzo para obtener un retorno mínimo, y la posibilidad de que las amenazas sofisticadas simplemente eludan la detección. En particular, las amenazas esquivas, como las amenazas persistentes avanzadas, dirigidas y de día cero, pueden ser las más peligrosas para su organización, lo que hace que su detección sea sumamente importante.

La funcionalidad UEBA en Azure Sentinel elimina el trabajo monótono y aburrido de las cargas de trabajo de los analistas y la incertidumbre de sus esfuerzos, y ofrece inteligencia de alta fidelidad y accionable, para que puedan centrarse en la investigación y la corrección.

A medida que Azure Sentinel recopila registros y alertas de todos sus orígenes de datos conectados, los analiza y genera perfiles de comportamiento de línea de base de las entidades de la organización (como usuarios, hosts, direcciones IP y aplicaciones) a lo largo del tiempo y del horizonte del grupo de homólogos. A través de diversas técnicas y funcionalidades de aprendizaje automático, Azure Sentinel puede identificar actividades anómalas y ayudarle a determinar si un recurso se ha puesto en peligro. No solo eso, sino que también puede averiguar la sensibilidad relativa de los recursos concretos, identificar los grupos de homólogos y evaluar el impacto potencial de cualquier recurso en peligro determinado (su "radio de la explosión"). Gracias a esta información, puede priorizar la investigación y el tratamiento de incidentes de manera eficaz. 

### <a name="ueba-analytics-architecture"></a>Arquitectura de análisis de UEBA

:::image type="content" source="media/identify-threats-with-entity-behavior-analytics/entity-behavior-analytics-architecture.png" alt-text="Arquitectura de análisis de comportamiento de entidades":::

### <a name="security-driven-analytics"></a>Análisis controlado por seguridad

Basado en el paradigma de Gartner para las soluciones UEBA, Azure Sentinel proporciona un enfoque "outside-in", que se basa en tres marcos de referencia:

- **Casos de uso:** al priorizar los escenarios y vectores de ataque pertinentes según la investigación de seguridad, alineada con el marco de tácticas, técnicas y subtécnicas de MITRE ATT&CK, que coloca varias entidades como víctimas, autores o puntos de pivote en la cadena de eliminación; Azure Sentinel se centra específicamente en los registros más valiosos que puede proporcionar cada origen de datos.

- **Orígenes de datos:** Si bien ante todo se admiten orígenes de datos de Azure, Azure Sentinel selecciona cuidadosamente los orígenes de datos de terceros para proporcionar datos que coincidan con nuestros escenarios de amenazas.

- **Análisis:** a través de varios algoritmos de aprendizaje automático (ML), Azure Sentinel identifica las actividades anómalas y presenta las pruebas de manera clara y concisa en forma de enriquecimientos contextuales, algunos ejemplos de los cuales aparecen a continuación.

    :::image type="content" source="media/identify-threats-with-entity-behavior-analytics/behavior-analytics-top-down.png" alt-text="Enfoque &quot;outside-in&quot; para el análisis del comportamiento":::

Azure Sentinel presenta artefactos que ayudan a los analistas de seguridad a comprender claramente las actividades anómalas en su contexto y en comparación con el perfil de línea base del usuario. Las acciones realizadas por un usuario (o un host o una dirección) se evalúan en contexto, donde un resultado "verdadero" indica una anomalía identificada:
- entre ubicaciones geográficas, dispositivos y entornos;
- entre los horizontes de tiempo y frecuencia (en comparación con el propio historial del usuario);
- en comparación con el comportamiento de homólogos;
- en comparación con el comportamiento de la organización.

    :::image type="content" source="media/identify-threats-with-entity-behavior-analytics/context.png" alt-text="Contexto de la entidad":::


### <a name="scoring"></a>Puntuaciones

Cada actividad se puntúa con una "puntuación de prioridad de la investigación", que determina la probabilidad de que un usuario específico realice una actividad específica, en función del aprendizaje del comportamiento del usuario y de sus homólogos. Las actividades identificadas como las más anómalas reciben las puntuaciones más altas (en una escala del 0 al 10).

Consulte cómo se usa el análisis de comportamiento en [Microsoft Cloud App Security](https://techcommunity.microsoft.com/t5/microsoft-security-and/prioritize-user-investigations-in-cloud-app-security/ba-p/700136) para ver un ejemplo de cómo funciona.

## <a name="entity-pages"></a>Páginas de entidad

Descubra más sobre las [entidades en Azure Sentinel](entities-in-azure-sentinel.md) y vea la lista completa de las [entidades e identificadores admitidos](entities-reference.md).

Cuando se encuentra cualquier entidad (actualmente limitada a usuarios y hosts) en una búsqueda, una alerta o una investigación, puede seleccionar la entidad y llevarla a una **página de entidad**, una hoja de datos llena de información útil sobre esa entidad. Entre los tipos de información que encontrará en esta página se incluyen datos básicos sobre la entidad, una escala de tiempo de eventos importantes relacionados con esta entidad y conclusiones sobre el comportamiento de la entidad.
 
Las páginas de entidad constan de tres partes:
- El panel de la izquierda contiene la información de identificación de la entidad, recopilada de los orígenes de datos, como Azure Active Directory, Azure Monitor, Azure Security Center y Microsoft Defender.

- En el panel central se muestra una escala de tiempo gráfica y textual de eventos importantes relacionados con la entidad, como alertas, marcadores y actividades. Las actividades son agregaciones de eventos importantes desde Log Analytics. Los equipos de investigación de seguridad de Microsoft desarrollan las consultas que detectan esas actividades.

- En el panel derecho se muestran conclusiones de comportamiento sobre la entidad. Estas conclusiones ayudan a detectar rápidamente las anomalías y las amenazas de seguridad. Los equipos de investigación de seguridad de Microsoft desarrollan las conclusiones, que se basan en los modelos de detección de anomalías.

### <a name="the-timeline"></a>Escala de tiempo

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/entity-pages-timeline.png" alt-text="Escala de tiempo de las páginas de entidad":::

La escala de tiempo es una parte importante de la contribución de la página de entidad al análisis del comportamiento en Azure Sentinel. Presenta una historia sobre eventos relacionados con la entidad, lo que le ayuda a comprender la actividad de la entidad dentro de un período de tiempo específico.

Puede elegir el **intervalo de tiempo** entre varias opciones preestablecidas (como *últimas 24 horas*) o establecerlo en cualquier período de tiempo definido de forma personalizada. Además, puede establecer filtros que limiten la información de la escala de tiempo a tipos específicos de eventos o alertas.

En la escala de tiempo se incluyen los siguientes tipos de elementos:

- Alertas: alertas en las que la entidad se define como **entidad asignada**. Tenga en cuenta que si su organización ha creado [alertas personalizadas mediante reglas de análisis](./tutorial-detect-threats-custom.md), debe asegurarse de que la asignación de entidades de las reglas se realice correctamente.

- Marcadores: marcadores que incluyen la entidad específica que se muestra en la página.

- Actividades: agregación de eventos importantes relacionados con la entidad. 
 
### <a name="entity-insights"></a>Conclusiones sobre la entidad
 
Las conclusiones sobre la entidad son consultas definidas por los investigadores de seguridad de Microsoft para ayudar a los analistas a investigar de manera más eficiente y eficaz. Las conclusiones se presentan como parte de la página de entidad y proporcionan información de seguridad valiosa sobre los hosts y los usuarios, en forma de gráficos y datos tabulares. Disponer aquí de la información significa que no tiene que desplazarse a Log Analytics. Las conclusiones incluyen datos sobre inicios de sesión, adiciones a grupos, eventos anómalos, etc., así como algoritmos avanzados de Azure Machine Learning para detectar comportamientos anómalos. 

Las conclusiones se basan en los siguientes orígenes de datos:
- Syslog (Linux)
- SecurityEvent (Windows)
- AuditLogs (Azure AD)
- SigninLogs (Azure AD)
- OfficeActivity (Office 365)
- BehaviorAnalytics (Azure Sentinel UEBA)
- Heartbeat (agente de Azure Monitor)
- CommonSecurityLog (Azure Sentinel)

### <a name="how-to-use-entity-pages"></a>Cómo usar páginas de entidad

Las páginas de entidad están diseñadas para formar parte de varios escenarios de uso y se puede acceder a ellas desde la administración de incidentes, el gráfico de investigación, los marcadores o directamente desde la página de búsqueda de entidades en **Análisis de comportamiento de entidades** en el menú principal de Azure Sentinel.

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/entity-pages-use-cases.png" alt-text="Casos de uso de páginas de entidad":::

Para más información sobre los datos que se muestran en la tabla **Análisis de comportamiento de entidades**, consulte [Referencia de características enriquecidas de UEBA de Azure Sentinel](ueba-enrichments.md).

## <a name="querying-behavior-analytics-data"></a>Consulta de datos de análisis de comportamiento

Mediante [KQL](/azure/data-explorer/kusto/query/), se puede consultar la tabla de análisis de comportamiento.

Por ejemplo, si queremos encontrar todos los casos en los que un usuario no pudo iniciar sesión en un recurso de Azure, donde era el primer intento de conexión del usuario desde un país determinado, y las conexiones desde ese país son poco frecuentes incluso para los homólogos de ese usuario, podemos usar la siguiente consulta:

```Kusto
BehaviorAnalytics
| where ActivityType == "FailedLogOn"
| where FirstTimeUserConnectedFromCountry == True
| where CountryUncommonlyConnectedFromAmongPeers == True
```

### <a name="user-peers-metadata---table-and-notebook"></a>Metadatos de homólogos del usuario: tabla y cuaderno

Los metadatos de los homólogos del usuario proporcionan un contexto importante en las detecciones de amenazas, en la investigación de un incidente y en la búsqueda de una amenaza potencial. Los analistas de seguridad pueden observar las actividades normales de los homólogos del usuario para determinar si las actividades del usuario son inusuales en comparación con las de sus homólogos.

Azure Sentinel calcula y clasifica los homólogos de un usuario en función de la pertenencia a grupos de seguridad de Azure AD del usuario, la lista de distribución de correo, etc., y almacena los homólogos clasificados de 1 a 20 en la tabla **UserPeerAnalytics**. En la captura de pantalla siguiente se muestra el esquema de la tabla UserPeerAnalytics, y se muestran los ocho homólogos mejor clasificados del usuario Kendall Collins. Azure Sentinel usa el algoritmo *TF-IDF* ( frecuencia de término – frecuencia inversa de documento) para normalizar la ponderación para calcular la clasificación: cuanto menor sea el grupo, mayor será la ponderación. 

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/user-peers-metadata.png" alt-text="Captura de pantalla de la tabla de metadatos de homólogos del usuario":::

Puede usar el cuaderno de [Jupyter Notebook](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/BehaviorAnalytics/UserSecurityMetadata) proporcionado en el repositorio de GitHub de Azure Sentinel para visualizar los metadatos de los homólogos del usuario. Para obtener instrucciones detalladas sobre cómo usar el cuaderno, consulte el cuaderno [Análisis guiado: metadatos de seguridad de usuario](https://github.com/Azure/Azure-Sentinel-Notebooks/blob/master/BehaviorAnalytics/UserSecurityMetadata/Guided%20Analysis%20-%20User%20Security%20Metadata.ipynb).

### <a name="permission-analytics---table-and-notebook"></a>Análisis de permisos: tabla y cuaderno

El análisis de permisos ayuda a determinar el impacto potencial de que un atacante ponga en peligro un recurso de la organización. Este impacto también se conoce como "radio de explosión" del recurso. Los analistas de seguridad pueden usar esta información para clasificar las investigaciones y el control de incidentes.

Azure Sentinel determina los derechos de acceso directo y transitivo que mantiene un usuario determinado en los recursos de Azure, mediante la evaluación de las suscripciones de Azure a las que el usuario puede tener acceso directamente o a través de grupos o entidades de servicio. Esta información, así como la lista completa de la pertenencia a grupos de seguridad Azure AD del usuario, se almacena en la tabla **UserAccessAnalytics**. En la captura de pantalla siguiente se muestra una fila de ejemplo de la tabla UserAccessAnalytics para el usuario Alex Johnson. La **entidad de origen** es la cuenta de usuario o de entidad de servicio, y la **entidad de destino** es el recurso al que tiene acceso la entidad de origen. Los valores de **nivel de acceso** y **tipo de acceso** dependen del modelo de control de acceso de la entidad de destino. Puede ver que Alex tiene acceso de colaborador a la suscripción de Azure *Contoso Hotels Tenant*. El modelo de control de acceso de la suscripción es Azure RBAC.

:::image type="content" source="./media/identify-threats-with-entity-behavior-analytics/user-access-analytics.png" alt-text="Captura de pantalla de la tabla de análisis de acceso de usuario":::

Puede usar el cuaderno de [Jupyter Notebook](https://github.com/Azure/Azure-Sentinel-Notebooks/tree/master/BehaviorAnalytics/UserSecurityMetadata) (el mismo cuaderno mencionado anteriormente) del repositorio de GitHub de Azure Sentinel para visualizar los datos de análisis de permisos. Para obtener instrucciones detalladas sobre cómo usar el cuaderno, consulte el cuaderno [Análisis guiado: metadatos de seguridad de usuario](https://github.com/Azure/Azure-Sentinel-Notebooks/blob/master/BehaviorAnalytics/UserSecurityMetadata/Guided%20Analysis%20-%20User%20Security%20Metadata.ipynb).

### <a name="hunting-queries-and-exploration-queries"></a>Consultas de búsqueda y consultas de exploración

Azure Sentinel proporciona de serie un conjunto de consultas de búsqueda, consultas de exploración y el libro **Análisis de comportamiento de usuarios y entidades**, que se basa en la tabla **BehaviorAnalytics**. Estas herramientas presentan datos enriquecidos, centrados en casos de uso específicos, que indican un comportamiento anómalo.

Para obtener más información, consulte:

- [Búsqueda de amenazas con Azure Sentinel](hunting.md)
- [Visualizar y supervisar los datos](tutorial-monitor-your-data.md)

A medida que las herramientas de defensa heredadas se quedan obsoletas, las organizaciones pueden tener un patrimonio digital tan amplio y poroso que resulta imposible obtener una perspectiva completa del riesgo y la posición a los que su entorno puede estar haciendo frente. Confiar principalmente en esfuerzos reactivos, como el análisis y las reglas, permite a los actores malintencionados aprender a eludir esos esfuerzos. Aquí es donde UEBA entra en juego, ya que proporciona metodologías de puntuación de riesgos y algoritmos para averiguar lo que sucede realmente.


## <a name="next-steps"></a>Pasos siguientes
En este documento, ha aprendido acerca de las funcionalidades de análisis de comportamiento de entidades de Azure Sentinel. Para obtener instrucciones prácticas sobre la implementación y para usar las conclusiones obtenidas, consulte los siguientes artículos:

- [Habilitación del análisis de comportamiento de entidades](./enable-entity-behavior-analytics.md) en Azure Sentinel
- [Investigación de incidentes con datos de UEBA](investigate-with-ueba.md).
- [Búsqueda de amenazas de seguridad](./hunting.md)

Para más información, consulte también [Referencia de características enriquecidas de UEBA de Azure Sentinel](ueba-enrichments.md).
