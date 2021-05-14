---
title: Directivas de soporte técnico para Azure Kubernetes Service (AKS)
description: Aprenda sobre las directivas de soporte técnico de Azure Kubernetes Service (AKS), la responsabilidad compartida y las características en versión preliminar (o alfa o beta).
services: container-service
ms.topic: article
ms.date: 09/18/2020
ms.openlocfilehash: d30e2aa5ab13c5d7676ca7cb455a2372981e4a46
ms.sourcegitcommit: 5da0bf89a039290326033f2aff26249bcac1fe17
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109714483"
---
# <a name="support-policies-for-azure-kubernetes-service"></a>Directivas de soporte técnico para Azure Kubernetes Service

En este artículo se proporcionan detalles sobre las directivas de soporte técnico y las limitaciones de Azure Kubernetes Service (AKS). En el artículo también se describen la administración de los nodos de agente, los componentes del plano de control administrado, los componentes de código abierto de terceros y la administración de la seguridad o las revisiones.

## <a name="service-updates-and-releases"></a>Actualizaciones de servicio y versiones

* Para información sobre la versión, consulte las [notas de la versión de AKS](https://github.com/Azure/AKS/releases).
* Para información sobre las características en versión preliminar, consulte [AKS preview features and related projects](https://awesomeopensource.com/projects/aks?categoryPage=11) (Características en versión preliminar y proyectos relacionados de AKS).

## <a name="managed-features-in-aks"></a>Características administradas en AKS

Los componentes base de la nube de infraestructura como servicio (IaaS), como los de proceso o red, proporcionan a los usuarios acceso a opciones de personalización y controles de nivel bajo. Por el contrario, AKS proporciona una implementación de Kubernetes inmediata que le ofrece el conjunto habitual de configuraciones y funcionalidades que necesita para su clúster. Como usuario de AKS, tiene opciones de personalización e implementación limitadas. A cambio, no tiene que preocuparse de administrar directamente los clústeres de Kubernetes.

Con AKS, obtiene un de *plano de control* totalmente administrado. El plano de control contiene todos los componentes y servicios que debe ofrecer y proporciona clústeres de Kubernetes para los usuarios finales. Todos los componentes de Kubernetes los mantiene y opera Microsoft.

Microsoft administra y supervisa los siguientes componentes mediante el panel de control:

* Servidores de la API de Kubernetes o Kubelet
* Etcd o un almacén de pares clave-valor compatible, que proporciona calidad de servicio (QoS), escalabilidad y tiempo de ejecución
* Servicios DNS (por ejemplo, kube-dns o CoreDNS)
* Proxy o red de Kubernetes
* Cualquier complemento adicional o componente del sistema que se ejecute en el espacio de nombres kube-system

AKS no es una solución de plataforma como servicio (PaaS). Algunos componentes, como los nodos de agente, tienen *responsabilidad compartida* con los usuarios para ayudar a mantener el clúster de AKS. Se requiere la participación del usuario, por ejemplo, para aplicar una revisión de seguridad del sistema operativo en el nodo de agente.

Los servicios son *administrados* en el sentido en que Microsoft y el equipo de AKS implementan y operan el servicio, y son responsables de su disponibilidad y funcionalidad. Los clientes no pueden modificar estos componentes administrados. Microsoft limita la personalización para garantizar una experiencia de usuario coherente y escalable. Para una solución totalmente personalizable, consulte el [motor de AKS](https://github.com/Azure/aks-engine).

## <a name="shared-responsibility"></a>Responsabilidad compartida

Cuando se crea un clúster, se definen los nodos de agente de Kubernetes que crea AKS. Las cargas de trabajo se ejecutan en esos nodos.

Dado que los nodos de agente ejecutan código privado y almacenan información confidencial, el equipo de Soporte técnico de Microsoft solo puede acceder a ellos con restricciones. El equipo de Soporte técnico de Microsoft no puede iniciar sesión en estos nodos, ejecutar comandos en ellos ni ver sus registros sin el consentimiento expreso o la asistencia del cliente.

Cualquier modificación realizada directamente en los nodos de agente mediante cualquiera de las API de IaaS representa el clúster no compatible. Cualquier modificación realizada en los nodos del agente debe realizarse mediante mecanismos kubernetes-native, como `Daemon Sets`.

Del mismo modo, aunque puede agregar metadatos al clúster y a los nodos, como etiquetas, si cambia cualquiera de los metadatos creados por el sistema, el clúster no se admitirá.

## <a name="aks-support-coverage"></a>Cobertura de soporte de AKS

Microsoft proporciona soporte técnico para lo siguiente:

* Conectividad a todos los componentes de Kubernetes que el servicio Kubernetes proporciona y admite, como el servidor de API.
* La administración, el tiempo de actividad, la calidad de servicio y las operaciones de los servicios del plano de control de Kubernetes (por ejemplo, el plano de control de Kubernetes, el servidor de API, etcd y kube-dns).
* Almacén de datos etcd. La compatibilidad incluye las copias de seguridad automatizadas y transparentes de todos los datos de etcd cada 30 minutos para la restauración del estado del clúster y la planificación ante desastres. Estas copias de seguridad no están directamente disponibles para los clientes o los usuarios. Garantizan la coherencia y confiabilidad de los datos. Etcd. la reversión a petición o la restauración no se admiten como una característica.
* Los puntos de integración en el controlador del proveedor de nube de Azure para Kubernetes. Estos incluyen las integraciones en otros servicios de Azure, como los equilibradores de carga, los volúmenes persistentes o las redes (Kubernetes y Azure CNI).
* Preguntas o problemas sobre la personalización de los componentes del plano de control, como el servidor de API de Kubernetes, etcd y coreDNS.
* Problemas relacionados con las redes, como Azure CNI, kubenet, u otros problemas de acceso de red y funcionalidades. Podrían incluir los de resolución de DNS, pérdida de paquetes, enrutamiento, etc. Microsoft admite diversos escenarios de redes:
  * Kubenet y Azure CNI mediante redes virtuales administradas o con subredes personalizadas (modelo Traiga sus propias subredes).
  * Conectividad con otros servicios y aplicaciones de Azure
  * Controladores de entrada y las configuraciones del equilibrador de carga o de entrada
  * Rendimiento y latencia de la red
  * [Directivas de red](use-network-policies.md#differences-between-azure-and-calico-policies-and-their-capabilities)


> [!NOTE]
> Todas las acciones de clúster realizadas por Microsoft/AKS se realizan con el consentimiento del usuario en un rol de Kubernetes integrado `aks-service` y el enlace de roles integrado `aks-service-rolebinding`. Este rol permite a AKS solucionar problemas de clústeres, pero no puede modificar permisos ni crear roles, enlaces de roles ni otras acciones de privilegios elevados. El acceso a roles solo se habilita en incidencias de soporte técnico activos con acceso Just-in-Time (JIT).

Microsoft no proporciona soporte técnico para lo siguiente:

* Preguntas acerca de cómo usar Kubernetes. Por ejemplo, el Soporte técnico de Microsoft no aconseja sobre la creación controladores de entrada personalizados, el uso de cargas de trabajo de aplicación o la aplicación de herramientas o paquetes de software de código abierto de terceros.
  > [!NOTE]
  > El Soporte técnico de Microsoft puede aconsejar sobre la funcionalidad del clúster de AKS, la personalización y los ajustes (por ejemplo, los problemas y los procedimientos de las operaciones de Kubernetes).
* Proyectos de código abierto de terceros que no se proporcionan como parte del plano de control de Kubernetes o se implementan con clústeres de AKS. Estos proyectos pueden incluir Istio, Helm, Envoy u otros.
  > [!NOTE]
  > Microsoft puede hacer lo posible por ayudar con proyectos de código abierto de terceros como Helm. Cuando la herramienta de código abierto de terceros integre el proveedor de nube de Azure de Kubernetes u otros errores específicos de AKS, Microsoft es compatible con ejemplos y aplicaciones de la documentación de Microsoft.
* Software de código cerrado de terceros. Este software puede incluir herramientas de análisis de seguridad y dispositivos o software de red.
* Personalizaciones de red distintas de las que se enumeran en la [documentación de AKS](./index.yml).


## <a name="aks-support-coverage-for-agent-nodes"></a>Cobertura de soporte técnico de AKS para los nodos de agente

### <a name="microsoft-responsibilities-for-aks-agent-nodes"></a>Responsabilidades de Microsoft sobre los nodos de agente de AKS

Microsoft y los clientes comparten la responsabilidad sobre los nodos de agente de Kubernetes donde:

* La imagen del sistema operativo base ha requerido adiciones (por ejemplo, agentes de red y de supervisión).
* Los nodos de agente reciban automáticamente revisiones del sistema operativo.
* Los problemas con los componentes del plano de control de Kubernetes que se ejecuten en los nodos de agente se remedian automáticamente. Estos componentes incluyen lo siguiente:
  * `Kube-proxy`
  * Túneles de red que proporcionan rutas de acceso de comunicación para los componentes maestros de Kubernetes
  * `Kubelet`
  * `Moby` o `ContainerD`

> [!NOTE]
> Si un nodo de agente no está operativo, AKS podría reiniciar componentes individuales o todo el nodo de agente. Estas operaciones de reinicio se automatizan y proporcionan corrección automática de los problemas comunes. Si desea obtener más información sobre los mecanismos de corrección automática, consulte [Reparación automática de nodo](node-auto-repair.md).

### <a name="customer-responsibilities-for-aks-agent-nodes"></a>Responsabilidades del cliente sobre los nodos de agente de AKS

Microsoft proporciona revisiones y nuevas imágenes para los nodos de imagen semanalmente, pero no las revisará automáticamente de forma predeterminada. Para mantener los componentes del sistema operativo y del runtime del nodo de agente con las últimas revisiones, debe mantener una programación de [actualizaciones de imagen de nodo](node-image-upgrade.md) periódicas o automatizarlas.

De forma similar, AKS lanza periódicamente nuevas revisiones de Kubernetes y versiones secundarias. Estas actualizaciones pueden contener mejoras de seguridad o funcionalidad para Kubernetes. Usted es el responsable de mantener actualizada la versión de Kubernetes de los clústeres y de acuerdo con la [directiva de versión de compatibilidad con Kubernetes AKS](supported-kubernetes-versions.md).

#### <a name="user-customization-of-agent-nodes"></a>Personalización de usuarios de nodos de agente
> [!NOTE]
> Los nodos de agente de AKS aparecen en Azure Portal como recursos de Azure IaaS normales. No obstante, estas máquinas virtuales se implementan en un grupo de recursos de Azure personalizado (normalmente, con el prefijo MC_\*). No se puede cambiar la imagen de sistema operativo base ni realizar ninguna personalización directa en estos nodos mediante las API o los recursos de IaaS. Los cambios personalizados que no se realicen a través de la API de AKS no se conservarán a través de una actualización, una escala, una actualización o un reinicio. Evite realizar cambios en los nodos de agente a menos que el equipo de Soporte técnico de Microsoft le indique que realice cambios.

AKS administra el ciclo de vida y las operaciones de los nodos de agente en nombre de los clientes, por lo que **no admite** la modificación de los recursos de IaaS asociados a los nodos de agente. Un ejemplo de una operación no admitida es la personalización de un conjunto de escalado de máquinas virtuales del grupo de nodos mediante la modificación manual de las configuraciones a través del portal o la API del conjunto de escalado de máquinas virtuales.
 
En el caso de las configuraciones o paquetes específicos de la carga de trabajo, AKS recomienda usar [Kubernetes `daemon sets`](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

El uso de contenedores de tipo init y `daemon sets` con privilegios de Kubernetes permite a los clientes ajustar, modificar o instalar software de terceros en los nodos de agente del clúster. Algunos ejemplos de estas personalizaciones incluyen agregar software personalizado de examen de seguridad o actualizar la configuración de sysctl.

Aunque se trata de una ruta de acceso recomendada si se cumplen los requisitos anteriores, los servicios de ingeniería y soporte técnico de AKS no pueden ayudarle en la solución de problemas, en el diagnóstico de modificaciones que indiquen que el nodo no está disponible debido a un `daemon set` personalizado implementado.

### <a name="security-issues-and-patching"></a>Problemas de seguridad y aplicación de revisiones

Si se encuentra un error de seguridad en uno o más componentes administrados de AKS, el equipo de AKS revisará todos los clústeres afectados para mitigar el problema. Como alternativa, el equipo proporciona a los usuarios instrucciones de actualización.

En el caso de los nodos de agente afectados por un error de seguridad, Microsoft le notificará con detalles sobre el impacto y los pasos para corregir o mitigar el problema de seguridad (normalmente una actualización de imagen de nodo o una actualización de la revisión de clúster).

### <a name="node-maintenance-and-access"></a>Acceso a los nodos y mantenimiento

Aunque puede iniciar sesión en los nodos de agente y cambiarlos, esto no es recomendable, ya que los cambios pueden hacer que un clúster deje de ser compatible.

## <a name="network-ports-access-and-nsgs"></a>Puertos de red, acceso y grupos de seguridad de red

Solo puede personalizar NSG en subredes personalizadas. No puede personalizar NSG en subredes administradas o en el nivel de NIC de los nodos del agente. AKS tiene requisitos de salida para puntos de conexión específicos, para controlar la salida y garantizar la conectividad necesaria; consulte [Limitación del tráfico de salida](limit-egress-traffic.md).

## <a name="stopped-or-de-allocated-clusters"></a>Clústeres detenidos o con asignación anuladas

Según se indicó anteriormente, la anulación manual de la asignación de todos los nodos de clúster a través de las API de IaaS, la CLI o el portal representan el clúster fuera del soporte técnico. La única manera admitida de detener y anular la asignación de todos los nodos es [detener el clúster de AKS](start-stop-cluster.md#stop-an-aks-cluster), que conservará el estado del clúster durante 12 meses.

Los clústeres que se detienen durante más de 12 meses ya no conservarán el estado. 

Los clústeres que se desasignan fuera de las API de AKS no tienen ninguna garantía de preservación del estado. Los planes de control de los clústeres en este estado se archivarán después de 30 días y se eliminarán después de 12 meses.

AKS se reserva el derecho de archivar los planos de control que se han configurado fuera de las directrices de compatibilidad durante períodos prolongados iguales o superiores a 30 días. AKS mantiene copias de seguridad de los metadatos de etcd del clúster y puede reasignar fácilmente el clúster. Esta reasignación puede iniciarse con cualquier operación PUT haciendo que el clúster vuelva a ser compatible, como una actualización o un escalado a los nodos de agente activos.

Si su suscripción se suspende o se elimina, el estado y el plano de control del clúster se eliminarán después de 90 días.

## <a name="unsupported-alpha-and-beta-kubernetes-features"></a>Características de Kubernetes alfa y beta no admitidas

Dentro del proyecto de Kubernetes ascendente, AKS solo admite características estables. A menos que se documente lo contrario, AKS no admite características alfa que estén disponibles en el proyecto de Kubernetes ascendente.

## <a name="preview-features-or-feature-flags"></a>Características en versión preliminar o marcas de característica

Para las características y la funcionalidad que requieran pruebas ampliadas y comentarios de los usuarios, Microsoft publica nuevas características en versión preliminar o características con una marca de característica. Considere estas características como versión preliminar o características beta.

Las características en versión preliminar o con marca de característica no son para producción. Los cambios continuos en las API y el comportamiento, las soluciones de errores y otros cambios pueden dar lugar a inestabilidad en los clústeres y tiempo de inactividad.

Las características en versión preliminar pública suponen el mejor esfuerzo de soporte, por estar en versión preliminar y no estar previstas para producción; se ocupan de ellas los equipos de soporte técnico de AKS únicamente durante el horario comercial. Para más información, consulte:

* [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/)

## <a name="upstream-bugs-and-issues"></a>Problemas y errores ascendentes

Dada la velocidad de desarrollo del proyecto ascendente de Kubernetes, siempre se producen errores. Algunos de estos errores no se pueden revisar o solucionar dentro del sistema de AKS. En su lugar, las correcciones de errores requieren mayores revisiones en los proyectos ascendentes (como Kubernetes, los sistemas operativos del nodo o de agente y kernels). Para los componentes que pertenecen a Microsoft (por ejemplo, el proveedor de nube de Azure), el personal de AKS y Azure se comprometen a solucionar los problemas ascendentes en la comunidad.

Cuando un problema de soporte técnico lo causa originalmente uno o más errores ascendentes, los equipos de soporte técnico e ingeniería de AKS:

* Identificarán y vincularán los errores ascendentes con detalles que lo justifiquen para intentar explicar por qué afecta al clúster o a la carga de trabajo. Los clientes recibirán vínculos a los repositorios necesarios para que puedan consultar los problemas y cuándo una nueva versión proporciona correcciones.
* Proporcionarán posibles soluciones alternativas o mitigaciones. Si se puede mitigar el problema, se archivará un [problema conocido](https://github.com/Azure/AKS/issues?q=is%3Aissue+is%3Aopen+label%3Aknown-issue) en el repositorio de AKS. En el archivo de problemas conocidos se explica:
  * El problema, con vínculos a los errores ascendentes.
  * La solución y los detalles sobre una actualización u otra manera de que la solución persista.
  * Escalas de tiempo aproximadas de la inclusión del problema en función de la cadencia de la versión ascendente.
