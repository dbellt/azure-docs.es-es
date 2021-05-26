---
title: Instalación de Event Grid en el clúster de Kubernetes habilitado para Azure Arc
description: En este artículo se proporcionan los pasos para instalar Event Grid en el clúster de Kubernetes habilitado para Azure Arc.
author: jfggdl
ms.author: jafernan
ms.subservice: kubernetes
ms.date: 05/11/2021
ms.topic: how-to
ms.openlocfilehash: 43879b5124c49ba3d0c4494e2d3a1eba95e617ff
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386560"
---
# <a name="install-event-grid-extension-on-azure-arc-enabled-kubernetes-cluster"></a>Instalación de la extensión de Event Grid en el clúster de Kubernetes habilitado para Azure Arc
En este artículo se presentan los pasos para instalar Event Grid en el clúster de [Kubernetes habilitado para Azure Arc](../../azure-arc/kubernetes/overview.md).

Por razones de brevedad, este artículo hace referencia a "Event Grid en la extensión de Kubernetes" como "Event Grid en Kubernetes" o simplemente "Event Grid".

[!INCLUDE [event-grid-preview-feature-note.md](../../../includes/event-grid-preview-feature-note.md)]


## <a name="supported-kubernetes-distributions"></a>Distribuciones de Kubernetes admitidas
A continuación se presentan las distribuciones de Kubernetes admitidas en las que Event Grid se puede implementar y ejecutar.

1. [Distribuciones de Kubernetes admitidas](../../aks/supported-kubernetes-versions.md) en AKS de Azure.
1. RedHat [OpenShift Container Platform](https://www.openshift.com/products/container-platform).

Se incorporarán más distribuciones según los [comentarios del usuario](https://feedback.azure.com/forums/909934-azure-event-grid) y su [compatibilidad con Kubernetes habilitado para Azure Arc](../../azure-arc/kubernetes/validation-program.md).

## <a name="event-grid-extension"></a>Extensión de Event Grid
La operación que instala una instancia de servicio de Event Grid en un clúster de Kubernetes es la creación de una extensión de clúster de Azure Arc, que implementa un **agente de Event Grid** y un **operador de Event Grid**. Para obtener más información sobre la función del agente y el operador, consulte [Event Grid en componentes de Kubernetes](concepts.md#event-grid-on-kubernetes-components). La característica de [extensión de clúster de Azure Arc](../../azure-arc/kubernetes/conceptual-extensions.md) proporciona administración del ciclo de vida mediante operaciones de plano de control de Azure Resource Manager (ARM) para Event Grid implementado en clústeres de Kubernetes habilitados para Azure Arc.

> [!NOTE]
> La versión preliminar del servicio solo admite una única instancia de la extensión de Event Grid en un clúster de Kubernetes, ya que la extensión de Event Grid está definida actualmente como una extensión de ámbito de clúster. Todavía no se admiten implementaciones con ámbito de espacio de nombres para Event Grid que permitan implementar varias instancias en un clúster.  Para obtener más información sobre los ámbitos de extensión, consulte [Crear instancia de extensión](../../azure-arc/kubernetes/extensions.md#create-extensions-instance) y busque ``scope``.

## <a name="prerequisites"></a>Requisitos previos
Antes de continuar con la instalación de Event Grid, asegúrese de que se cumplen los siguientes requisitos previos. 

1. Un clúster que se ejecuta en una de las [distribuciones de Kubernetes admitidas](#supported-kubernetes-distributions).
1. [Una suscripción de Azure](https://azure.microsoft.com/en-us/free/).
1. [Certificados PKI](#pki-certificate-requirements) que se usarán para establecer una conexión HTTPS con el agente de Event Grid.
1. [Conexión del clúster a Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md).

## <a name="getting-support"></a>Obtención de soporte técnico
Si tiene un problema, consulte la sección [Solucionar problemas](#troubleshooting) para obtener ayuda con condiciones comunes. Si sigue teniendo problemas, [cree una solicitud de soporte técnico de Azure](get-support.md#how-to-create-a-support-request).

## <a name="pki-certificate-requirements"></a>Requisitos de certificados PKI
El agente de Event Grid (servidor) sirve dos tipos de clientes. La autenticación del servidor se realiza mediante certificados. La autenticación de cliente se realiza mediante certificados o claves de SAS basadas en el tipo de cliente.

- Los operadores de Event Grid que hacen solicitudes de plano de control al agente de Event Grid se autentican mediante certificados.
- Los editores de Event Grid que publican eventos en un tema de Event Grid se autentican con las claves de SAS del tema.

Para establecer una comunicación HTTPS segura con el agente de Event Grid y el operador de Event Grid, usamos certificados PKI durante la instalación de la extensión de Event Grid. Estos son los requisitos generales para estos certificados PKI:

1. Los certificados y las claves deben ser certificados [X.509](https://en.wikipedia.org/wiki/X.509) y codificados con [correo con privacidad mejorada](https://en.wikipedia.org/wiki/Privacy-Enhanced_Mail) (PEM).
1. Para configurar el certificado de agente de Event Grid (servidor) durante la instalación, deberá proporcionar lo siguiente:
    1. Certificado de CA
    1. Certificado público
    1. Una clave privada
1. Para configurar el certificado de operador de Event Grid (cliente) deberá proporcionar lo siguiente:
    1. Certificado de CA
    1. Certificado público
    1. Una clave privada

    Los clientes de publicación pueden usar el certificado de CA del agente de Event Grid para validar el servidor al publicar eventos en un tema.

    > [!IMPORTANT]
    > Si bien un dominio asociado al cliente puede tener más de un certificado público emitido por distintas entidades de certificación, Event Grid en Kubernetes solo permite cargar un único certificado de CA para clientes al instalar Event Grid. Como consecuencia, la misma entidad de certificación debe emitir (firmar) los certificados para el operador Event Grid con el fin de que la validación de la cadena de certificados se realice correctamente y se establezca una sesión de TLS de manera satisfactoria.
1. Al configurar el nombre común (CN) para los certificados de servidor y cliente, asegúrese de que son distintos del CN proporcionado para el certificado de entidad de certificación.

    > [!IMPORTANT] 
    > En las primeras fases de la prueba de concepto, los certificados autofirmados pueden ser una opción, pero en general, se deben adquirir y usar los certificados PKI adecuados firmados por una entidad de certificación (CA).

## <a name="install-event-grid-on-kubernetes-extension"></a>Instalación de Event Grid en la extensión de Kubernetes

1. En Azure Portal, busque (campo en la parte superior) **Azure Arc**
1. Seleccione **Clúster de Kubernetes** en el menú de la izquierda de la sección **Infraestructura**
1. En la lista de clústeres, localice aquel donde desea instalar Event Grid y selecciónelo. Se muestra la página de **información general** del clúster.
1. Seleccione **Extensiones** en el grupo **Configuración** en el menú izquierdo.
1. Seleccione **+Agregar**. Aparece una página que muestra las extensiones de Kubernetes de Azure Arc disponibles.
1. Seleccione **Event Grid**.
1. Seleccione **Crear** en el Event Grid en Kubernetes con página de Azure Arc.
1. Se muestra la pestaña **Aspectos básicos** de la página **Instalar Event Grid**. La sección **Detalles del proyecto** muestra los valores de suscripción de solo lectura y de grupo de recursos porque las extensiones de Azure Arc se implementan en la misma suscripción de Azure y el mismo grupo de recursos del clúster conectado en el que están instaladas.
1. Proporcione un nombre en el campo **nombre de extensión de Event Grid**. Este nombre debe ser único entre otras extensiones de Azure Arc implementadas en el mismo clúster conectado de Azure Arc.
1. En **Espacio de nombres de versión**, puede que quiera proporcionar el nombre de un espacio de nombres de Kubernetes donde se implementarán los componentes de Event Grid. El valor predeterminado es **eventgrid-system**. Si el espacio de nombres proporcionado no existe, se crea automáticamente.
1. En la sección de información del **agente de Event Grid**, se muestra el tipo de servicio. El agente de Event Grid, que es el componente que expone los puntos de conexión del tema a los que se envían los eventos, se expone como un tipo de servicio de Kubernetes **ClusterIP**. Por lo tanto, las direcciones IP asignadas a todos los temas utilizan el espacio de IP privado configurado para el clúster.
1. Proporcione el **nombre de clase de almacenamiento** que quiere usar para el agente y que es compatible con la distribución de Kubernetes. Por ejemplo, si usa AKS, podría usar `azurefile`, que utiliza Azure Standard Storage. Para obtener más información sobre las clases de almacenamiento definidas previamente compatibles con AKS, consulte [Clases de almacenamiento en AKS](../../aks/concepts-storage.md#storage-classes). Si usa otras distribuciones de Kubernetes, consulte la documentación de distribución de Kubernetes para conocer las clases de almacenamiento predefinidas que se admiten o cómo puede proporcionar las suyas propias.
1. **Tamaño de almacenamiento**. El valor predeterminado es 1 GiB. Tenga en cuenta la tasa de ingesta al determinar el tamaño del almacenamiento. La tasa de ingesta en MiB por segundo medida como el tamaño de los eventos multiplicado por la tasa de publicación (eventos por segundo) en todos los temas del agente de Event Grid es un factor clave a la hora de asignar almacenamiento. Los eventos son transitorios por naturaleza y, una vez entregados, esos eventos no consumen almacenamiento. Aunque la tasa de ingesta es un controlador principal del uso del almacenamiento, no es el único. La configuración de suscripciones a eventos y temas que contienen metadatos también consume espacio de almacenamiento, pero normalmente requiere una cantidad menor de espacio de almacenamiento que los eventos ingeridos y entregados por Event Grid.
1. **Límite de memoria**. El valor predeterminado es 1 GiB. 
1. **Solicitud de memoria**. El valor predeterminado es 200 MiB. Este campo no se puede editar.

    :::image type="content" source="./media/install-k8s-extension/basics-page.png" alt-text="Instalación de la extensión de Event Grid - página de aspectos básicos":::
1. Seleccione **Siguiente: Configuración** en la parte inferior de la página.
1. **Habilite la comunicación HTTP (no segura)** . Active esta casilla si quiere usar un canal no protegido cuando los clientes se comuniquen con el agente de Event Grid.

    > [!IMPORTANT]
    > Al habilitar esta opción, toda la comunicación con el agente de Event Grid usará HTTP como transporte. Por lo tanto, cualquier cliente de publicación y el operador de Event Grid no se comunicarán con el agente de Event Grid de forma segura. Se debe usar esta opción solo durante las primeras fases del desarrollo.
1. Si no ha habilitado la comunicación HTTP, seleccione cada uno de los archivos de certificado PKI que ha adquirido y cumpla los [requisitos de certificado PKI](#pki-certificate-requirements).

    :::image type="content" source="./media/install-k8s-extension/configuration-page.png" alt-text="Instalar la extensión de Event Grid - página de configuración":::
1. Seleccione **Siguiente: Supervisión** en la parte inferior de la página.
1. **Habilite las métricas** activando esta opción. Event Grid en Kubernetes expone métricas para suscripciones a evento y temas con el uso del [formato de exposición Prometheus](https://prometheus.io/docs/instrumenting/exposition_formats/).

    :::image type="content" source="./media/install-k8s-extension/monitoring-page.png" alt-text="Instalación de extensión de Event Grid - página de supervisión":::    
1. Seleccione **Siguiente: Etiquetas** para ir a la página **Etiquetas**. Defina [etiquetas](/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging), si es necesario.

    :::image type="content" source="./media/install-k8s-extension/tags-page.png" alt-text="Instalación de la extensión de Event Grid - página de etiquetas":::
1. En la parte inferior de la página, seleccione **Revisar y crear**.
1. En la pestaña **Revisar y crear**, seleccione **Crear**.
    
    :::image type="content" source="./media/install-k8s-extension/review-create-page.png" alt-text="Instalación de extensión de Event Grid - página Revisar y crear":::   
    
    > [!IMPORTANT]
    > La instalación de Event Grid es una operación asincrónica que puede ejecutarse durante más tiempo en el clúster de Kubernetes que cuando se ve una notificación en Azure Portal que informa de que la implementación se ha completado. Cuando vea la notificación "La implementación se ha completado", espere 5 minutos antes de intentar crear una ubicación personalizada (paso siguiente). Si tiene acceso al clúster de Kubernetes, en una sesión de Bash puede ejecutar el siguiente comando para validar si los pods del agente de Event Grid y del operador Event Grid están en estado "En ejecución", lo que indicaría que la instalación se ha completado:

    ```bash
    kubectl get pods -n \<release-namespace-name\>
    ```
    > [!IMPORTANT]
    > Es necesario crear una ubicación personalizada antes de intentar implementar temas de Event Grid. Para crear una ubicación personalizada, puede seleccionar la página **Contexto** en la parte inferior, 5 minutos después de que se muestre la notificación "La implementación se ha completado". Como alternativa, puede crear una ubicación personalizada mediante [Azure Portal](https://portal.azure.com/#blade/HubsExtension/BrowseResource/resourceType/Microsoft.ExtendedLocation%2FCustomLocations).
1. Una vez que la implementación se realiza correctamente, podrá ver una entrada en la página **Extensiones** con el nombre que proporcionó a la extensión de Event Grid.

## <a name="troubleshooting"></a>Solución de problemas

### <a name="azure-arc-connect-cluster-issues"></a>Problemas de conexión del clúster a Azure Arc

**Problema**: al ir a **Azure Arc** y hacer clic en **clúster de Kubernetes** en el menú de la izquierda, la página que aparece no muestra el clúster de Kubernetes donde quiero instalar Event Grid.

**Resolución**: el clúster de Kubernetes no está registrado en Azure. Siga los pasos descritos en el artículo [Conexión de un clúster de Kubernetes existente a Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md). Si tiene un problema durante este paso, presente una [solicitud de soporte técnico](#getting-support) al equipo de Kubernetes habilitado para Azure Arc.

### <a name="event-grid-extension-issues"></a>Problemas de extensión de Event Grid

**Problema**: al intentar instalar una "extensión de Event Grid", el usuario recibe el siguiente mensaje: "**Operación no válida**: ya se ha instalado una instancia de Event Grid en este clúster de Kubernetes conectado. La extensión de Event Grid se sitúa en el ámbito del clúster, lo que significa que solo se puede instalar una instancia en un clúster".
    
**Explicación**: Event Grid ya esta instalado. La versión preliminar de Event Grid solo admite que se implemente una instancia de extensión de Event Grid en un clúster.


## <a name="next-steps"></a>Pasos siguientes
Consulte el inicio rápido [Enrutar eventos en la nube a webhooks con Azure Event Grid en Kubernetes](create-topic-subscription.md).

