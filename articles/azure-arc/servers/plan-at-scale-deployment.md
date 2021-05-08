---
title: Planeación e implementación de servidores habilitados para Azure Arc
description: Aprenda a habilitar un gran número de máquinas en servidores habilitados para Azure Arc para simplificar la configuración de las funcionalidades esenciales de seguridad, administración y supervisión de Azure.
ms.date: 04/21/2021
ms.topic: conceptual
ms.openlocfilehash: e3f8fe410da56f627ceab5f17c980f2daa1a262c
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107831985"
---
# <a name="plan-and-deploy-arc-enabled-servers"></a>Planeación e implementación de servidores habilitados para Arc

La implementación de un servicio de infraestructura de TI o de una aplicación empresarial es un reto para cualquier empresa. Para llevarla a cabo correctamente y evitar sorpresas imprevistas y costos no planeados, debe planearla minuciosamente para asegurarse de que está lo más preparado posible. Para planear la implementación de servidores habilitados para Azure Arc a cualquier escala, debe abarcar los criterios de diseño e implementación que deben cumplirse para completar correctamente las tareas.

Para que la implementación se lleve a cabo sin problemas, el plan debe incluir una comprensión clara de los siguientes aspectos:

* Roles y responsabilidades.
* Inventario de servidores físicos o máquinas virtuales para comprobar que cumplen los requisitos de red y del sistema.
* El conjunto de habilidades y la formación requeridos para habilitar la implementación correcta y la administración continua.
* Criterios de aceptación y seguimiento de su grado de cumplimiento.
* Herramientas o métodos que se van a usar para automatizar las implementaciones.
* Riesgos identificados y planes de mitigación para evitar retrasos, interrupciones, etc.
* Cómo evitar la interrupción durante la implementación.
* ¿Cuál es la ruta de escalación cuando se produce un problema importante?

El propósito de este artículo es asegurarse de que está preparado para una implementación correcta de servidores habilitados para Azure Arc en varios servidores físicos o máquinas virtuales de producción en su entorno.

## <a name="prerequisites"></a>Requisitos previos

* Las máquinas ejecutan un [sistema operativo compatible](agent-overview.md#supported-operating-systems) para el agente de Connected Machine.
* Los equipos tienen conectividad desde la red local u otro entorno de nube a recursos de Azure, ya sea directamente o a través de un servidor proxy.
* Para instalar y configurar el agente de Connected Machine de los servidores habilitados para Arc, una cuenta con privilegios elevados (es decir, un administrador o como raíz) en las máquinas.
* Para incorporar máquinas, debe ser miembro del rol **Incorporación de Azure Connected Machine**.
* Para leer, modificar y eliminar una máquina, debe ser miembro del rol **Administrador de recursos de Azure Connected Machine**.

## <a name="pilot"></a>Piloto

Antes de realizar la implementación en todos los equipos de producción, empiece por evaluar este proceso de implementación antes de adoptarlo ampliamente en su entorno. Para una prueba piloto, identifique un muestreo representativo de máquinas que no sean críticas para la capacidad de su empresa de llevar a cabo sus actividades. Asegúrese de dejar tiempo suficiente para ejecutar la prueba piloto y evaluar su efecto: se recomienda un mínimo de 30 días.

Establezca un plan formal que describa el ámbito y los detalles de la prueba piloto. A continuación se muestra un ejemplo de lo que debe incluir un plan para ayudarle a comenzar.

* Objetivos: describe los factores técnicos y empresariales que llevaron a la decisión de que es necesaria una prueba piloto.
* Criterios de selección: especifica los criterios que se usan para seleccionar los aspectos de la solución que se mostrarán a través de una prueba piloto.
* Ámbito: describe el ámbito de la prueba piloto, que incluye, entre otros, los componentes de la solución, la programación prevista, la duración de la prueba piloto y el número de máquinas de destino.
* Criterios y métricas de éxito: defina los criterios de éxito de la prueba piloto y las medidas específicas empleadas para determinar el nivel de éxito.
* Plan de formación: describe el plan de formación de los ingenieros de sistemas, administradores, etc. que son nuevos en Azure y sus servicios durante la prueba piloto.
* Plan de transición: describe la estrategia y los criterios que se usan para guiar la transición de la prueba piloto a producción.
* Reversión: describe los procedimientos para revertir una prueba piloto al estado anterior a la implementación.
* Riesgos: enumera todos los riesgos identificados para la realización de la prueba piloto y los asociados a la implementación de producción.

## <a name="phase-1-build-a-foundation"></a>Fase 1: Creación de una base

En esta fase, los ingenieros de sistemas o los administradores habilitan las características principales en la suscripción de Azure de sus organizaciones para iniciar la base antes de habilitar sus máquinas para la administración por parte de los servidores habilitados para Arc y otros servicios de Azure.

|Tarea |Detail |Duration |
|-----|-------|---------|
| [Cree un grupo de recursos](../../azure-resource-manager/management/manage-resource-groups-portal.md#create-resource-groups) | Un grupo de recursos dedicado para incluir solo servidores habilitados para Arc y centralizar la administración y supervisión de estos recursos. | Una hora |
| Aplique [etiquetas](../../azure-resource-manager/management/tag-resources.md) para ayudar a organizar las máquinas. | Evalúe y desarrolle una [estrategia de etiquetado](/azure/cloud-adoption-framework/decision-guides/resource-tagging/) alineada con el equipo de TI que pueda ayudar a reducir la complejidad de la administración de los servidores habilitados para Arc y a simplificar la toma de decisiones de administración. | Un día |
| Diseño e implementación de [registros de Azure Monitor](../../azure-monitor/logs/data-platform-logs.md) | Evalúe las [consideraciones de diseño e implementación](../../azure-monitor/logs/design-logs-deployment.md) para determinar si su organización debe usar un área de trabajo de Log Analytics existente o implementar otro para almacenar los datos de registro recopilados de máquinas y servidores híbridos.<sup>1</sup> | Un día |
| Desarrollo de un plan de gobernanza de [Azure Policy](../../governance/policy/overview.md) | Determine cómo implementará la gobernanza de las máquinas y los servidores híbridos en el ámbito de la suscripción o del grupo de recursos con Azure Policy. | Un día |
| Configuración del [control de acceso basado en rol](../../role-based-access-control/overview.md) (RBAC) | Desarrolle un plan de acceso para controlar quién tiene acceso a la administración de los servidores habilitados para Arc y la capacidad de ver sus datos desde otros servicios y soluciones de Azure. | Un día |
| Identificación de las máquinas con el agente de Log Analytics ya instalado | Ejecute la siguiente consulta de registro en [Log Analytics](../../azure-monitor/logs/log-analytics-overview.md) para admitir la conversión de las implementaciones del agente de Log Analytics existentes en un agente administrado por extensiones:<br> Latido <br> &#124; where TimeGenerated > ago(30d) <br> &#124; where ResourceType == "machines" and (ComputerEnvironment == "Non-Azure") <br> &#124; summarize by Computer, ResourceProvider, ResourceType, ComputerEnvironment | Una hora |

<sup>1</sup> Una consideración importante en el marco de la evaluación del diseño del área de trabajo de Log Analytics es la integración con Azure Automation para la compatibilidad con sus características Update Management y Seguimiento de cambios e inventario, así como con Azure Security Center y Azure Sentinel. Si su organización ya tiene una cuenta de Automation y ha habilitado sus características de administración vinculadas a un área de trabajo Log Analytics, evalúe si puede centralizar y optimizar las operaciones de administración, así como minimizar el costo, mediante el uso de los recursos existentes en lugar de duplicar una cuenta, un área de trabajo, etc.

## <a name="phase-2-deploy-arc-enabled-servers"></a>Fase 2: Implementación de servidores habilitados para Arc

A continuación, utilizaremos la base establecida en la fase 1 para preparar e implementar los servidores habilitados para Arc con el agente de Connected Machine.

|Tarea |Detail |Duration |
|-----|-------|---------|
| Descarga del script de instalación predefinido | Revise y personalice el script de instalación predefinido para la implementación a escala del agente de Connected Machine para admitir los requisitos de implementación automatizada.<br><br> Ejemplos de recursos de incorporación a escala:<br><br> <ul><li> [Script de implementación básica a escala](onboard-service-principal.md)</ul></li> <ul><li>[Máquinas virtuales de Windows Server de VMware vSphere de incorporación a escala](https://github.com/microsoft/azure_arc/blob/main/docs/azure_arc_jumpstart/azure_arc_servers/scaled_deployment/vmware_scaled_powercli_win/_index.md)</ul></li> <ul><li>[Máquinas virtuales de Linux de VMware vSphere de incorporación a escala](https://github.com/microsoft/azure_arc/blob/main/docs/azure_arc_jumpstart/azure_arc_servers/scaled_deployment/vmware_scaled_powercli_linux/_index.md)</ul></li> <ul><li>[Instancias EC2 de AWS de incorporación a escala mediante Ansible](https://github.com/microsoft/azure_arc/blob/main/docs/azure_arc_jumpstart/azure_arc_servers/scaled_deployment/aws_scaled_ansible/_index.md)</ul></li> <ul><li>[Implementación a escala con comunicación remota de PowerShell](./onboard-powershell.md) (solo Windows)</ul></li>| Uno o varios días en función de los requisitos, los procesos de la organización (por ejemplo, administración de versiones y de cambios) y el método de automatización que se usan. |
| [Creación de una entidad de servicio](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale) |Cree una entidad de servicio para conectar máquinas de forma no interactiva mediante Azure PowerShell o desde el portal.| Una hora |
| Implementación del agente de Connected Machine en los equipos y servidores de destino |Use la herramienta de automatización para implementar los scripts en los servidores y conectarlos a Azure.| Uno o varios días según el plan de lanzamiento y si sigue una implementación por fases. |

## <a name="phase-3-manage-and-operate"></a>Fase 3: Administración y funcionamiento

En la fase 3, los administradores o ingenieros de sistemas permiten la automatización de las tareas manuales para administrar el agente de Connected Machine y la máquina durante su ciclo de vida y trabajar con ellos.

|Tarea |Detail |Duration |
|-----|-------|---------|
|Creación de una alerta de Resource Health |Si un servidor deja de enviar latidos a Azure durante más de 15 minutos, puede significar que está sin conexión, que la conexión de red se ha bloqueado o que el agente no se está ejecutando. Desarrolle un plan sobre cómo responderá a estos incidentes y cómo los investigará, así como sobre el uso de [alertas de Resource Health](../..//service-health/resource-health-alert-monitor-guide.md) para recibir notificaciones cuando se inicien.<br><br> Especifique lo siguiente al configurar la alerta:<br> **Tipo de recurso** = **Servidores habilitados para Azure Arc**<br> **Estado de recurso actual** = **No disponible**<br> **Estado de recurso previo** = **No disponible** | Una hora |
|Creación de una alerta de Azure Advisor | Para obtener la mejor experiencia y las correcciones de seguridad y errores más recientes, recomendamos mantener actualizado el agente de los servidores habilitados para Azure Arc. Los agentes no actualizados se identificarán con una [alerta de Azure Advisor](../../advisor/advisor-alerts-portal.md).<br><br> Especifique lo siguiente al configurar la alerta:<br> **Tipo de recomendación** = **Actualización a la versión más reciente del agente de Azure Connected Machine** | Una hora |
|[Asignación de las directivas de Azure](../../governance/policy/assign-policy-portal.md) al ámbito de la suscripción o del grupo de recursos |Asigne la [directiva](../../azure-monitor/vm/vminsights-enable-policy.md) **Habilitar Azure Monitor para VM** (y otras que satisfagan sus necesidades) al ámbito de suscripción o grupo de recursos. Azure Policy permite asignar definiciones de directivas que instalan los agentes necesarios para Información de máquinas virtuales en el entorno.| Varía |
|[Habilitación de Update Management para los servidores habilitados para Arc](../../automation/update-management/enable-from-automation-account.md) |Configure Update Management en Azure Automation para administrar las actualizaciones del sistema operativo de las máquinas virtuales de Windows y Linux registradas con servidores habilitados para Arc. | 15 minutos |

## <a name="next-steps"></a>Pasos siguientes

* Puede encontrar información sobre la solución de problemas en la guía [Solución de problemas de conexión del agente de Connected Machine](troubleshoot-agent-onboard.md).

* Aprenda a simplificar la implementación con otros servicios de Azure, como [State Configuration](../../automation/automation-dsc-overview.md) de Azure Automation y otras [extensiones de máquina virtual de Azure](manage-vm-extensions.md) admitidas.