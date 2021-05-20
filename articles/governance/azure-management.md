---
title: 'Introducción a la administración de Azure: Gobernanza de Azure'
description: Información general de las áreas de administración para las aplicaciones y los recursos de Azure con vínculos a contenido sobre las herramientas de administración de Azure.
ms.date: 05/01/2021
ms.topic: overview
ms.openlocfilehash: cc6758ffa98f14b12e25e7fdb443a29cb5d788a5
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747564"
---
# <a name="what-are-the-azure-management-areas"></a>¿Qué son las áreas de administración de Azure?

La gobernanza de Azure es uno de los aspectos de la administración de Azure. En este artículo se describen las distintas áreas de administración para implementar y mantener los recursos de Azure.

Administración hace referencia a las tareas y procesos necesarios para mantener las aplicaciones empresariales y los recursos que las dan soporte. Azure tiene muchos servicios y herramientas que funcionan en conjunto para proporcionar una administración completa. Estos servicios no solo son para los recursos de Azure, sino también de otras nubes y en entornos locales. El primer paso para diseñar un entorno de administración completo es entender las distintas herramientas y cómo funcionan en conjunto.

El siguiente diagrama muestra las diferentes áreas de administración necesarias para mantener cualquier aplicación o recurso. Estas áreas distintas pueden considerarse como un ciclo de vida. Cada área es necesaria en una sucesión continua a lo largo de toda la duración de un recurso. El ciclo de vida de este recurso empieza con la implementación inicial, sigue con la operación continua y, finalmente, termina cuando se le retira.

:::image type="complex" source="../monitoring/media/management-overview/management-capabilities.png" alt-text="Diagrama de las materias de administración en Azure." border="false":::
   Diagrama que muestra los elementos Migración, Seguridad, Protección, Supervisión, Configuración y Gobernanza del conjunto de servicios que posibilitan la administración y la gobernanza en Azure. Seguridad contiene los elementos secundarios Administración de seguridad y Protección contra amenazas. Protección contiene los elementos secundarios Copia de seguridad y Recuperación ante desastres. Supervisión contiene los elementos secundarios Aplicación, Supervisión de infraestructura y red, Log Analytics y Diagnósticos. Configuración contiene los elementos secundarios Configuración, Update Management, Automation y Scripting. Y Gobernanza contiene los elementos secundarios Administración de directivas y Cost Management.
:::image-end:::

No hay ningún servicio único de Azure que cumpla íntegramente con los requisitos de un área de administración determinada. En vez de eso, son varios los servicios que los satisfacen, trabajando en conjunto. Algunos servicios, como Application Insights, proporcionan una funcionalidad de supervisión dirigida para las aplicaciones web. Otros, como los registros de Azure Monitor, almacenan datos de administración para otros servicios. Esta característica permite analizar datos de tipos diferentes recopilados por distintos servicios.

En las secciones siguientes se describen sucintamente las distintas áreas de administración y se proporcionan vínculos a información detallada acerca de los principales servicios de Azure diseñados para trabajar en ellas.

## <a name="monitor"></a>Supervisión

La supervisión es el acto de recopilar y analizar datos para auditar el rendimiento, el mantenimiento y la disponibilidad de los recursos. Una estrategia de supervisión eficaz le permite entender el funcionamiento de los componentes y aumentar el tiempo de actividad con las notificaciones. Lea la información general de la supervisión, donde se analizan los distintos servicios usados, en [Supervisión de aplicaciones y recursos de Azure](../azure-monitor/overview.md).

## <a name="configure"></a>Configuración

La configuración se refiere a la implementación inicial y a la configuración de los recursos y el mantenimiento continuo.
La automatización de estas tareas permite eliminar la redundancia, reducir el tiempo y esfuerzo que debe invertir, y aumentar la precisión y eficiencia. [Azure Automation](../automation/automation-intro.md) proporciona la mayor parte de los servicios para automatizar las tareas de configuración. Mientras los runbooks controlan la automatización de los procesos, la administración de las actualizaciones y la configuración ayudan a administrar la configuración.

## <a name="govern"></a>Control

La gobernanza proporciona mecanismos y procesos para mantener el control de las aplicaciones y recursos de Azure. Conlleva la planificación de las iniciativas y el establecimiento de prioridades estratégicas.
En Azure, la gobernanza se implementa principalmente con dos servicios. [Azure Policy](./policy/overview.md) permite crear, asignar y administrar definiciones de directivas para aplicar las reglas en los recursos.
Con esta característica, esos recursos se mantienen en cumplimiento con los estándares corporativos.
[Azure Cost Management](../cost-management-billing/cost-management-billing-overview.md) permite realizar un seguimiento del uso y de los gastos de la nube de los recursos no solo de Azure, sino también de otros proveedores de servicios en la nube.

## <a name="secure"></a>Seguridad

Administre la seguridad de los recursos y los datos. Un programa de seguridad implica evaluar amenazas, recopilar y analizar datos y que las aplicaciones y los recursos cumplan con los requisitos. La supervisión de la seguridad y el análisis de amenazas los proporciona [Azure Security Center](../security-center/security-center-introduction.md), que incluye una administración unificada de la seguridad y una protección avanzada contra amenazas para cargas de trabajo en la nube híbrida. Consulte [Introducción a la seguridad de Azure](../security/fundamentals/overview.md) para información y guías detalladas sobre cómo proteger los recursos de Azure.

## <a name="protect"></a>Protección

La protección se refiere a mantener disponibles aplicaciones y datos, incluso si se producen interrupciones que escapan de todo control. En Azure, la protección se proporciona mediante dos servicios. [Azure Backup](../backup/backup-overview.md) proporciona copias de seguridad y recuperación de los datos, tanto los de la nube como los de un entorno local. [Azure Site Recovery](../site-recovery/site-recovery-overview.md) ofrece continuidad empresarial y la recuperación inmediata durante un desastre.

## <a name="migrate"></a>Migrar

La migración hace referencia a la transición de las cargas de trabajo que se ejecutan de forma local a la nube de Azure.
[Azure Migrate](../migrate/migrate-services-overview.md) es un servicio que ayuda a evaluar la idoneidad de la migración de máquinas virtuales locales a Azure. Azure Site Recovery migra máquinas virtuales [desde el entorno local](../site-recovery/migrate-tutorial-on-premises-azure.md) o [desde Amazon Web Services](../site-recovery/migrate-tutorial-aws-azure.md). [Azure Database Migration](../dms/dms-overview.md) lo ayuda a realizar la migración de orígenes de bases de datos a plataformas de datos de Azure.

## <a name="next-steps"></a>Pasos siguientes

Para más información acerca de la gobernanza en Azure, consulte estos artículos:

- Vea el [centro de gobernanza en Azure](./index.yml).
- Consulte [Gobernanza en Cloud Adoption Framework para Azure](/azure/cloud-adoption-framework/govern/)
