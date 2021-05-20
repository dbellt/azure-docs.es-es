---
title: Ejemplo de plano técnico de ISM restringido de Nueva Zelanda
description: Introducción al ejemplo de plano técnico de ISM restringido de Nueva Zelanda. Este ejemplo de plano técnico ayuda a los clientes a evaluar determinados controles concretos.
ms.date: 03/22/2021
ms.topic: sample
ms.openlocfilehash: 1a7f50853c00336df584d210330860889e3f6881
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108757115"
---
# <a name="new-zealand-ism-restricted-blueprint-sample"></a>Ejemplo de plano técnico de ISM restringido de Nueva Zelanda

El ejemplo de plano técnico de ISM restringido de Nueva Zelanda proporciona guías de protección de gobernanza mediante [Azure Policy](../../policy/overview.md) que le ayudan a evaluar los controles específicos del [Manual de seguridad de la información de Nueva Zelanda](https://www.nzism.gcsb.govt.nz/). Este plano técnico ayuda a los clientes a implementar un conjunto básico de directivas para cualquier arquitectura implementada en Azure que deba implantar controles para ISM restringido de Nueva Zelanda.

## <a name="control-mapping"></a>Asignación de controles

La [asignación de controles de Azure Policy](../../policy/samples/new-zealand-ism.md) proporciona detalles sobre las definiciones de directiva incluidas en este plano técnico y cómo se asignan estas definiciones de directiva a los **controles** del Manual de seguridad de la información de Nueva Zelanda. Cuando se asignan a una arquitectura, Azure Policy evalúa los recursos para detectar posibles incumplimientos de las definiciones de directiva asignadas. Para más información, consulte [Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Implementación

Para implementar el ejemplo de plano técnico de ISM restringido de Nueva Zelanda de Azure Blueprints, debe realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

### <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico **ISM restringido de Nueva Zelanda** en _Otros ejemplos_ y seleccione **Usar este ejemplo**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: proporcione un nombre para su copia del ejemplo de plano técnico de ISM restringido de Nueva Zelanda.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos incluidos en el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

### <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla de la alineación con los controles de ISM restringido de Nueva Zelanda.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. Proporcione las **Notas de cambios**, por ejemplo, "Primera versión publicada del ejemplo de plano técnico de ISM restringido de Nueva Zelanda". A continuación, seleccione **Publicar** en la parte inferior de la página.

### <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción predeterminada de identidad administrada _asignada por el sistema_.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

### <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Lista de usuarios que deben incluirse en el grupo de administradores de VM de Windows|Lista separada por punto y coma de los usuarios que se deben incluir en el grupo Administradores local. P. ej.: Administrador; miUsuario1; miUsuario2|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Lista de usuarios que se deben excluir del grupo de administradores de VM Windows|Lista separada por punto y coma de los usuarios que se deben excluir en el grupo Administradores local; P. ej.: Administrador; miUsuario1; miUsuario2|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Lista de usuarios que solo debe incluir el grupo de administradores de VM Windows|Lista separada por punto y coma de todos los miembros esperados del grupo local de administradores. Por ejemplo: Administrador; miUsuario1; miUsuario2.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Id. del área de trabajo de Log Analytics para la creación de informes del agente de VM.|Id. (GUID) del área de trabajo de Log Analytics en que los agentes de VM deben realizar sus notificaciones.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Web Application Firewall (WAF) debe estar habilitado para Azure Front Door Service.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: la configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: las recomendaciones de protección de redes adaptables se deben aplicar en las máquinas virtuales con conexión a Internet.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Debe haber más de un propietario asignado a su suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: El cifrado de discos debe aplicarse en máquinas virtuales|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se debe desactivar la depuración remota para Function App|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Web Application Firewall (WAF) debe usar el modo especificado para Application Gateway.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Requisito del modo de WAF para Application Gateway|El modo de prevención o detección debe estar habilitado en el servicio Application Gateway.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se debe permitir el cifrado de datos transparente en bases de datos SQL|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: la evaluación de vulnerabilidades debe estar habilitada en SQL Managed Instance.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Opcional: lista de imágenes de máquina virtual personalizadas que han admitido el sistema operativo Windows para agregar a un ámbito adicional a las imágenes de la galería para la directiva: Implementación: configurar Dependency Agent para su habilitación en máquinas virtuales Windows|Para más información sobre la configuración de invitado, consulte [https://aka.ms/gcpol](../../policy/concepts/guest-configuration.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: solo se deben habilitar las conexiones seguras a la instancia de Azure Cache for Redis.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La solución de protección del punto de conexión debe instalarse en las máquinas virtuales|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores con conexión a Arc al evaluar la directiva: auditar las máquinas Windows en las que falta alguno de los miembros especificados del grupo de administradores|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Opcional: lista de imágenes de máquina virtual personalizadas que han admitido el sistema operativo Windows para agregar a un ámbito adicional a las imágenes de la galería para la directiva: [Versión preliminar]: el agente de Log Analytics debe estar habilitado para las imágenes de máquina virtual enumeradas|Para más información sobre la configuración de invitado, consulte [https://aka.ms/gcpol](../../policy/concepts/guest-configuration.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Opcional: lista de imágenes de máquina virtual personalizadas que han admitido el sistema operativo Linux para agregar a un ámbito adicional a las imágenes de la galería para la directiva: [Versión preliminar]: el agente de Log Analytics debe estar habilitado para las imágenes de máquina virtual enumeradas|Para más información sobre la configuración de invitado, consulte [https://aka.ms/gcpol](../../policy/concepts/guest-configuration.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: las cuentas de almacenamiento deben restringir el acceso a la red.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Opcional: lista de imágenes de máquina virtual personalizadas que han admitido el sistema operativo Windows para agregar a un ámbito adicional a las imágenes de la galería para la directiva: Implementación: configurar Dependency Agent para su habilitación en conjuntos de escalado de máquinas virtuales Windows|Para más información sobre la configuración de invitado, consulte [https://aka.ms/gcpol](../../policy/concepts/guest-configuration.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades en la configuración de seguridad de los conjuntos de escalado de máquinas virtuales|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores con conexión a Arc al evaluar la directiva: auditar las máquinas Windows con cuentas adicionales en el grupo de administradores|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se debe habilitar la transferencia segura a las cuentas de almacenamiento|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Web Application Firewall (WAF) debe usar el modo especificado para Azure Front Door Service.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Requisito del modo de WAF para Azure Front Door Service|El modo de prevención o detección debe estar habilitado en el servicio Azure Front Door.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: los controles de aplicaciones adaptables para definir aplicaciones seguras deben estar habilitados en las máquinas.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Debe designar un máximo de tres propietarios para la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: (versión preliminar) no se debe permitir el acceso público a la cuenta de almacenamiento.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: la solución de evaluación de vulnerabilidades debe estar habilitada en las máquinas virtuales.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Web Application Firewall (WAF) debe estar habilitado para Application Gateway.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: CORS no debe permitir que todos los recursos accedan a las aplicaciones web|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores conectados mediante Arc al evaluar la directiva: auditar los servidores web de Windows que no usen protocolos de comunicación segura.|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Versión de TLS mínima para servidores web con Windows|Los servidores web de Windows con versiones anteriores de TLS se evaluarán como no compatibles.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Opcional: lista de imágenes de máquina virtual personalizadas que han admitido el sistema operativo Linux para agregar a un ámbito adicional a las imágenes de la galería para la directiva: el agente de Log Analytics debe estar habilitado en los conjuntos de escalado de máquinas virtuales para las imágenes de máquina virtual enumeradas|Para más información sobre la configuración de invitado, consulte [https://aka.ms/gcpol](../../policy/concepts/guest-configuration.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Opcional: lista de imágenes de máquina virtual personalizadas que han admitido el sistema operativo Windows para agregar a un ámbito adicional a las imágenes de la galería para la directiva: el agente de Log Analytics debe estar habilitado en los conjuntos de escalado de máquinas virtuales para las imágenes de máquina virtual enumeradas|Para más información sobre la configuración de invitado, consulte [https://aka.ms/gcpol](../../policy/concepts/guest-configuration.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de escritura deben quitarse de la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores con conexión a Arc al evaluar la directiva: auditar las máquinas Windows con los miembros especificados del grupo de administradores|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Las cuentas en desuso deben quitarse de la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Acceso a Function App solo a través de HTTPS|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: las suscripciones de Azure deben tener un perfil de registro para el registro de actividad.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Lista de tipos de recursos que deben tener los registros de diagnóstico habilitados||
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se deben instalar actualizaciones del sistema en las máquinas|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación de API.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La extensión IaaSAntimalware de Microsoft debe implementarse en servidores Windows.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Acceso a la aplicación web solo a través de HTTPS|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Azure DDoS Protection Standard debe estar habilitado.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: se debe habilitar Advanced Data Security en la instancia administrada de SQL.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Supervisión de la falta de Endpoint Protection en Azure Security Center|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: el registro de actividad debe conservarse durante al menos un año.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: los puertos de administración de las máquinas virtuales deben protegerse con el control de acceso de red Just-In-Time.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Los clústeres de Service Fabric solo deben usar Azure Active Directory para la autenticación de cliente|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Acceso a API App solo a través de HTTPS|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: auditar las máquinas Windows en las que la Protección contra vulnerabilidades de seguridad de Microsoft Defender no está habilitada.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores conectados mediante Arc al evaluar la directiva: auditar las máquinas Windows en las que la Protección contra vulnerabilidades de seguridad de Microsoft Defender no está habilitada.|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Estado de cumplimiento para notificar máquinas Windows en que la Protección contra vulnerabilidades de seguridad de Microsoft Defender no está disponible|La Protección contra vulnerabilidades de seguridad de Windows Defender solo está disponible a partir de Windows 10 y Windows Server con la actualización 1709. Al establecer este valor en "No compatible" se muestran las máquinas con versiones anteriores en las que la Protección contra vulnerabilidades de seguridad de Windows Defender no está disponible (como Windows Server 2012 R2) como no compatible. Al establecer este valor en "Compatible" se muestran estas máquinas como compatibles.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se deben instalar las actualizaciones del sistema en los conjuntos de escalado de máquinas virtuales|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se debe desactivar la depuración remota para las aplicaciones web|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades en la configuración de seguridad en las máquinas|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades en las configuraciones de seguridad de contenedor|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se debe desactivar la depuración remota para API Apps|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: auditar las máquinas Linux que permiten conexiones remotas desde cuentas sin contraseña.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores con conexión a Arc al evaluar la directiva: auditar las máquinas Linux que permiten conexiones remotas desde cuentas sin contraseñas|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Las cuentas en desuso con permisos de propietario deben quitarse de la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación web.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: las máquinas Windows deben cumplir los requisitos de "Opciones de seguridad - Directivas de cuenta".|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Forzar el historial de contraseñas para las cuentas locales de la VM Windows|Especifica los límites de reutilización de contraseñas: el número de veces que se debe crear una contraseña nueva para una cuenta de usuario antes de que se pueda repetir la contraseña.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores con conexión a Arc al evaluar la directiva: las máquinas Windows deben cumplir los requisitos de "Configuración de seguridad - Directivas de cuenta".|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Duración máxima de la contraseña para cuentas locales de VM de Windows|Especifica el número máximo de días que pueden transcurrir antes de que se deba cambiar la contraseña de una cuenta de usuario; el formato del valor es de dos enteros separados por una coma, lo que denota un intervalo inclusivo.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Vigencia mínima de la contraseña para las cuentas locales de VM Windows|Especifica el número mínimo de días que deben transcurrir antes de que se pueda cambiar una contraseña de cuenta de usuario.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Longitud mínima de la contraseña para las cuentas locales de VM Windows|Especifica el número mínimo de caracteres que puede contener una contraseña de cuenta de usuario.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|La contraseña debe cumplir los requisitos de complejidad para las cuentas locales de la VM Windows|Especifica si la contraseña de una cuenta de usuario debe ser compleja. Si es así, una contraseña compleja no debe contener parte del nombre de la cuenta del usuario ni su nombre completo, debe tener una longitud mínima de 6 caracteres y debe incluir una combinación de mayúsculas, minúsculas, números y caracteres no alfabéticos.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: las máquinas virtuales con conexión a Internet deben protegerse con grupos de seguridad de red.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: auditar las máquinas Linux que tienen cuentas sin contraseña.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Incluir servidores con conexión a Arc al evaluar la directiva: auditar las máquinas Linux que tienen cuentas sin contraseñas|Al seleccionar "true", acepta el cargo mensual por máquina conectada a Arc.|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de propietario deben quitarse de la suscripción|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación de funciones.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: (versión preliminar) todo el tráfico de Internet debe enrutarse mediante la instancia de Azure Firewall implementada.|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|
|ISM restringido de Nueva Zelanda|Asignación de directiva|Efecto para la directiva: Se deben corregir las vulnerabilidades de las bases de datos SQL|Para más información sobre los efectos, consulte [https://aka.ms/policyeffects](../../policy/concepts/effects.md).|

## <a name="next-steps"></a>Pasos siguientes

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
