---
title: Solución de errores comunes
description: Aprenda a solucionar problemas relacionados con la creación de definiciones de directivas, los diversos SDK y el complemento para Kubernetes.
ms.date: 04/19/2021
ms.topic: troubleshooting
ms.openlocfilehash: c4feae11c6d8d78a43bae9882405e292a18e90bd
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107725069"
---
# <a name="troubleshoot-errors-with-using-azure-policy"></a>Solución de errores mediante Azure Policy

Al crear definiciones de directivas, trabajar con los SDK o configurar el complemento [Azure Policy para Kubernetes](../concepts/policy-for-kubernetes.md), pueden producirse errores. En este artículo se describen los distintos errores generales que se pueden producir y se sugieren maneras de resolverlos.

## <a name="find-error-details"></a>Búsqueda de detalles del error

La ubicación de los detalles del error depende del aspecto de Azure Policy con el que trabaje.

- Si trabaja con una directiva personalizada, vaya a Azure Portal para obtener comentarios de linting o revisar los [datos de cumplimiento](../how-to/get-compliance-data.md) resultantes para ver cómo se evaluaron los recursos.
- Si trabaja con cualquiera de los diversos SDK, el SDK proporciona detalles sobre el motivo de que se produjera un error en la función.
- Si trabaja con el complemento para Kubernetes, empiece por el [registro](../concepts/policy-for-kubernetes.md#logging) en el clúster.

## <a name="general-errors"></a>Errores generales

### <a name="scenario-alias-not-found"></a>Escenario: Alias no encontrado

#### <a name="issue"></a>Incidencia

Se ha utilizado un alias incorrecto o inexistente en una definición de directiva. Azure Policy usa [alias](../concepts/definition-structure.md#aliases) para asignarlos a las propiedades de Azure Resource Manager.

#### <a name="cause"></a>Causa

Se ha utilizado un alias incorrecto o inexistente en una definición de directiva.

#### <a name="resolution"></a>Solución

En primer lugar, compruebe que la propiedad de Resource Manager tiene un alias. Para buscar los alias disponibles, vaya a [Extensión de Azure Policy para Visual Studio Code](../how-to/extension-for-vscode.md) o el SDK.
Si el alias de una propiedad de Resource Manager no existe, cree una incidencia de soporte técnico.

### <a name="scenario-evaluation-details-arent-up-to-date"></a>Escenario: Los detalles de evaluación no están actualizados

#### <a name="issue"></a>Problema

Un recurso tiene el estado _No iniciado_ o los detalles de cumplimiento no son actuales.

#### <a name="cause"></a>Causa

Cualquier nueva asignación de directiva o iniciativa tarda unos 30 minutos en aplicarse. Los recursos nuevos o actualizados dentro del ámbito de una asignación existente estarán disponibles unos 15 minutos más tarde. Un examen de cumplimiento estándar se produce cada 24 horas. Para más información, consulte los [desencadenadores de evaluación](../how-to/get-compliance-data.md#evaluation-triggers).

#### <a name="resolution"></a>Solución

En primer lugar, espere la cantidad de tiempo que haga falta a que finalice una evaluación y a que los resultados de cumplimiento estén disponibles en Azure Portal o el SDK. Para iniciar un nuevo examen de evaluación con Azure PowerShell o la API REST, consulte [Examen de evaluación a petición](../how-to/get-compliance-data.md#on-demand-evaluation-scan).

### <a name="scenario-compliance-isnt-as-expected"></a>Escenario: La compatibilidad no es la esperada

#### <a name="issue"></a>Problema

Un recurso no está en el estado de evaluación _Compatible_ o _No compatible_ que se espera para él.

#### <a name="cause"></a>Causa

El recurso no está en el ámbito correcto de la asignación de directiva o la definición de directiva no funciona según lo previsto.

#### <a name="resolution"></a>Solución

Para solucionar los problemas de la definición de directiva, haga lo siguiente:

1. En primer lugar, espere la cantidad de tiempo que haga falta a que finalice una evaluación y a que los resultados de cumplimiento estén disponibles en Azure Portal o el SDK. 

1. Para iniciar un nuevo examen de evaluación con Azure PowerShell o la API REST, consulte [Examen de evaluación a petición](../how-to/get-compliance-data.md#on-demand-evaluation-scan).
1. Compruebe que los parámetros de asignación y el ámbito de asignación estén configurados correctamente.
1. Compruebe el [modo de definición de directiva](../concepts/definition-structure.md#mode):
   - El modo debe ser `all` para todos los tipos de recursos.
   - El modo debe ser `indexed` si la definición de directiva comprueba las etiquetas o la ubicación.
1. Asegúrese de que el ámbito del recurso no sea [excluido](../concepts/assignment-structure.md#excluded-scopes) ni [exento](../concepts/exemption-structure.md).
1. Si la compatibilidad de una asignación de directiva muestra `0/0` recursos, se determinó que ningún recurso era aplicable en el ámbito de la asignación. Compruebe la definición de la directiva y el ámbito de la asignación.
1. En el caso de los recursos no compatibles que se espera que sean compatibles, consulte [Determinación de las razones del no cumplimiento](../how-to/determine-non-compliance.md). La comparación de la definición con el valor de propiedad evaluado indica por qué un recurso no era compatible.
   - Si el **valor de destino** es incorrecto, revise la definición de la directiva.
   - Si el **valor actual** es incorrecto, valide la carga de recursos a través de `resources.azure.com`.
1. Para otros problemas y soluciones comunes, consulte [Solución de problemas: La aplicación no es la esperada](#scenario-enforcement-not-as-expected).

Si todavía tiene un problema con la definición de la directiva integrada duplicada y personalizada, o la definición personalizada, cree una incidencia de soporte técnico en **Creación de una directiva** para dirigir el problema correctamente.

### <a name="scenario-enforcement-not-as-expected"></a>Escenario: La aplicación no es la esperada

#### <a name="issue"></a>Problema

Azure Policy no está realizando ninguna acción sobre el recurso esperado y no hay ninguna entrada en el [registro de actividad de Azure](../../../azure-monitor/essentials/platform-logs-overview.md).

#### <a name="cause"></a>Causa

La asignación de directivas se ha configurado para un valor de [**enforcementMode**](../concepts/assignment-structure.md#enforcement-mode) de _Deshabilitado_.
Aunque **enforcementMode** esté deshabilitado, el efecto de la directiva no se aplica y no hay ninguna entrada en el registro de actividad.

#### <a name="resolution"></a>Solución

Para solucionar los problemas de cumplimiento de la asignación de directivas, haga lo siguiente:

1. En primer lugar, espere la cantidad de tiempo que haga falta a que finalice una evaluación y a que los resultados de cumplimiento estén disponibles en Azure Portal o el SDK. 

1. Para iniciar un nuevo examen de evaluación con Azure PowerShell o la API REST, consulte [Examen de evaluación a petición](../how-to/get-compliance-data.md#on-demand-evaluation-scan).
1. Compruebe que los parámetros de asignación y el ámbito de asignación estén configurados correctamente y que **enforcementMode** esté _Habilitado_.
1. Compruebe el [modo de definición de directiva](../concepts/definition-structure.md#mode):
   - El modo debe ser `all` para todos los tipos de recursos.
   - El modo debe ser `indexed` si la definición de directiva comprueba las etiquetas o la ubicación.
1. Asegúrese de que el ámbito del recurso no sea [excluido](../concepts/assignment-structure.md#excluded-scopes) ni [exento](../concepts/exemption-structure.md).
1. Compruebe que la carga de recursos coincida con la lógica de la directiva. Para ello, [capture un seguimiento de archivo HTTP (HAR)](../../../azure-portal/capture-browser-trace.md) o revise las propiedades de la plantilla de Azure Resource Manager.
1. Para otros problemas y soluciones comunes, consulte [Solución de problemas: El cumplimiento no es el esperado](#scenario-compliance-isnt-as-expected).

Si todavía tiene un problema con la definición de la directiva integrada duplicada y personalizada, o la definición personalizada, cree una incidencia de soporte técnico en **Creación de una directiva** para dirigir el problema correctamente.

### <a name="scenario-denied-by-azure-policy"></a>Escenario: Denegado por Azure Policy

#### <a name="issue"></a>Problema

Se ha denegado la creación o actualización de un recurso.

#### <a name="cause"></a>Causa

Una asignación de directivas al ámbito en el que se encuentra el recurso nuevo o actualizado cumple los criterios de una definición de directiva con un efecto [Denegar](../concepts/effects.md#deny). Se impide que los recursos que reúnan estas condiciones se creen o actualicen.

#### <a name="resolution"></a>Solución

El mensaje de error de una asignación de directiva de denegación incluye los identificadores de definición de directiva y de asignación de directivas. Si se pierde la información de error en el mensaje, también está disponible en el [registro de actividad](../../../azure-monitor/essentials/activity-log.md#view-the-activity-log). Use esta información para obtener más detalles para comprender las restricciones de recursos y ajustar las propiedades de los recursos de la solicitud para que coincidan con los valores permitidos.

### <a name="scenario-definition-targets-multiple-resource-types"></a>Escenario: La definición incluye varios tipos de recursos

#### <a name="issue"></a>Problema

Una definición de directiva que incluye varios tipos de recursos produce el siguiente error de validación durante la creación o la actualización:

```error
The policy definition '{0}' targets multiple resource types, but the policy rule is authored in a way that makes the policy not applicable to the target resource types '{1}'.
```

#### <a name="cause"></a>Causa

La regla de definición de directiva tiene una o varias condiciones que los tipos de recursos de destino no evalúan.

#### <a name="resolution"></a>Solución

Si se usa un alias, asegúrese de que este se evalúe solo con respecto al tipo de recurso al que pertenece agregando una condición de tipo delante. Una alternativa es dividir la definición de directiva en varias definiciones para evitar incluir varios tipos de recursos.

## <a name="template-errors"></a>Errores de plantilla

### <a name="scenario-policy-supported-functions-processed-by-template"></a>Escenario: Funciones admitidas por la directiva que procesa una plantilla

#### <a name="issue"></a>Problema

Azure Policy admite varias funciones de plantilla de Resource Manager y funciones que solo están disponibles en una definición de directiva. Resource Manager procesa estas funciones como parte de una implementación en lugar de como parte de una definición de directiva.

#### <a name="cause"></a>Causa

El uso de funciones admitidas, como `parameter()` o `resourceGroup()` da lugar al resultado procesado de la función en el momento de la implementación en lugar de permitir que la definición de directiva y el motor de Azure Policy la procesen.

#### <a name="resolution"></a>Solución

Para pasar una función para que forme parte de una definición de directiva, escape la cadena completa con `[` de forma que la propiedad se parezca a `[[resourceGroup().tags.myTag]`. El carácter de escape hace que Resource Manager trate el valor como una cadena al procesar la plantilla. Azure Policy coloca entonces la función en la definición de directiva, lo que permite que sea dinámica como se esperaba. Para obtener más información, consulte [Sintaxis y expresiones de las plantillas de Azure Resource Manager](../../../azure-resource-manager/templates/template-expressions.md).

## <a name="add-on-for-kubernetes-installation-errors"></a>Complemento para errores de instalación de Kubernetes

### <a name="scenario-installation-by-using-a-helm-chart-fails-because-of-a-password-error"></a>Escenario: No se puede realizar la instalación mediante un gráfico de Helm debido a un error de contraseña

#### <a name="issue"></a>Problema

Se produce un error en el comando `helm install azure-policy-addon` y se devuelve uno de los siguientes errores:

- `!: event not found`
- `Error: failed parsing --set data: key "<key>" has no value (cannot end with ,)`

#### <a name="cause"></a>Causa

La contraseña generada incluye una coma (`,`) que hace que el gráfico de Helm se divida.

#### <a name="resolution"></a>Solución

Al ejecutar `helm install azure-policy-addon`, escape la coma (`,`) del valor de contraseña con una barra diagonal inversa (`\`).

### <a name="scenario-installation-by-using-a-helm-chart-fails-because-the-name-already-exists"></a>Escenario: Se produce un error en la instalación mediante un gráfico de Helm porque el nombre ya existe

#### <a name="issue"></a>Problema

El comando `helm install azure-policy-addon` produce un error y devuelve el siguiente error:

- `Error: cannot re-use a name that is still in use`

#### <a name="cause"></a>Causa

El gráfico de Helm llamado `azure-policy-addon` ya se ha instalado completa o parcialmente.

#### <a name="resolution"></a>Solución

Siga las instrucciones para [quitar el complemento Azure Policy para Kubernetes](../concepts/policy-for-kubernetes.md#remove-the-add-on) y, luego, vuelva a ejecutar el comando `helm install azure-policy-addon`.

### <a name="scenario-azure-virtual-machine-user-assigned-identities-are-replaced-by-system-assigned-managed-identities"></a>Escenario: Las identidades asignadas por el usuario de una máquina virtual de Azure se sustituyen por identidades administradas asignadas por el sistema

#### <a name="issue"></a>Incidencia

Después de asignar las iniciativas de directiva de configuración de invitado a la configuración de auditoría dentro de una máquina, las identidades administradas asignadas por el usuario que se asignaron a la máquina ya no se asignan. Solo se asigna una identidad administrada asignada por el sistema.

#### <a name="cause"></a>Causa

Las definiciones de directiva usadas anteriormente en las definiciones de DeployIfNotExists de la configuración de invitado garantizaban que una identidad asignada por el sistema estuviese asignada a la máquina, pero también quitaban las asignaciones de identidad asignadas por el usuario.

#### <a name="resolution"></a>Solución

Las definiciones que anteriormente provocaban este problema aparecen como _\[En desuso\]_ y se han reemplazado por definiciones de directivas que administran los requisitos previos sin quitar las identidades administradas asignadas por el usuario. Se requiere un paso manual. Elimine las asignaciones de directiva existentes marcadas como _\[En desuso\]_ y reemplácelas por la iniciativa de directiva de requisitos previos actualizada y las definiciones de directivas que tengan el mismo nombre que el original.

Para encontrar una descripción detallada, consulte la entrada de blog [Cambio importante publicado para las directivas de auditoría de configuración de invitado](https://techcommunity.microsoft.com/t5/azure-governance-and-management/important-change-released-for-guest-configuration-audit-policies/ba-p/1655316).

## <a name="add-on-for-kubernetes-general-errors"></a>Complemento para errores generales de Kubernetes

### <a name="scenario-the-add-on-is-unable-to-reach-the-azure-policy-service-endpoint-because-of-egress-restrictions"></a>Escenario: El complemento no puede tener acceso al punto de conexión de servicio de Azure Policy debido a las restricciones de salida.

#### <a name="issue"></a>Problema

El complemento no puede tener acceso al punto de conexión de servicio de Azure Policy y devuelve uno de los siguientes errores:

- `failed to fetch token, service not reachable`
- `Error getting file "Get https://raw.githubusercontent.com/Azure/azure-policy/master/built-in-references/Kubernetes/container-allowed-images/template.yaml: dial tcp 151.101.228.133.443: connect: connection refused`

#### <a name="cause"></a>Causa

Este problema se produce cuando se bloquea una salida del clúster.

#### <a name="resolution"></a>Solución

Asegúrese de que los dominios y puertos que se mencionan en los siguientes artículos están abiertos:

- [Reglas de red de salida y FQDN necesarios para clústeres de AKS](../../../aks/limit-egress-traffic.md#required-outbound-network-rules-and-fqdns-for-aks-clusters)
- [Instalación del complemento de Azure Policy de Kubernetes habilitado para Azure Arc (versión preliminar)](../concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-azure-arc-enabled-kubernetes)

### <a name="scenario-the-add-on-is-unable-to-reach-the-azure-policy-service-endpoint-because-of-the-aad-pod-identity-configuration"></a>Escenario: El complemento no puede acceder al punto de conexión de servicio de Azure Policy debido a la configuración de aad-pod-identity

#### <a name="issue"></a>Problema

El complemento no puede tener acceso al punto de conexión de servicio de Azure Policy y devuelve uno de los siguientes errores:

- `azure.BearerAuthorizer#WithAuthorization: Failed to refresh the Token for request to https://gov-prod-policy-data.trafficmanager.net/checkDataPolicyCompliance?api-version=2019-01-01-preview: StatusCode=404`
- `adal: Refresh request failed. Status Code = '404'. Response body: getting assigned identities for pod kube-system/azure-policy-8c785548f-r882p in CREATED state failed after 16 attempts, retry duration [5]s, error: <nil>`

#### <a name="cause"></a>Causa

Este error se produce cuando _add-pod-identity_ está instalado en el clúster y los pods de _kube-system_ no se excluyen en _aad-pod-identity_.

Los pods de Identidad administrada del nodo componente (NMI) de _aad-pod-identity_ modifican las tablas de IP de los nodos para interceptar las llamadas que se realizan al punto de conexión de metadatos de instancia de Azure. Esta configuración implica que NMI intercepta toda solicitud realizada al punto de conexión de metadatos, incluso aunque el pod no utilice _aad-pod-identity_.
CustomResourceDefinition (CRD) de _AzurePodIdentityException_ se puede configurar para informar a _aad-pod-identity_ de que las solicitudes a un punto de conexión de metadatos que se originan desde un pod que coincide con las etiquetas definidas en la CRD deben pasar por un servidor proxy sin ningún procesamiento en NMI.

#### <a name="resolution"></a>Solución

Excluya los pods del sistema que tengan la etiqueta `kubernetes.azure.com/managedby: aks` en el espacio de nombres _kube-system_ en _aad-pod-identity_ mediante la configuración de la CRD de _AzurePodIdentityException_.

Para más información, consulte [Deshabilitación de la identidad del pod de Azure Active Directory (Azure AD) para un pod o una aplicación específicos](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).

Para configurar una excepción, siga este ejemplo:

```yaml
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: mic-exception
  namespace: default
spec:
  podLabels:
    app: mic
    component: mic
---
apiVersion: "aadpodidentity.k8s.io/v1"
kind: AzurePodIdentityException
metadata:
  name: aks-addon-exception
  namespace: kube-system
spec:
  podLabels:
    kubernetes.azure.com/managedby: aks
```

### <a name="scenario-the-resource-provider-isnt-registered"></a>Escenario: El proveedor de recursos no está registrado

#### <a name="issue"></a>Problema

El complemento puede acceder al punto de conexión de servicio de Azure Policy, pero los registros del complemento muestran uno de los siguientes errores:

- `The resource provider 'Microsoft.PolicyInsights' is not registered in subscription '{subId}'. See
  https://aka.ms/policy-register-subscription for how to register subscriptions.`

- `policyinsightsdataplane.BaseClient#CheckDataPolicyCompliance: Failure responding to request:
  StatusCode=500 -- Original Error: autorest/azure: Service returned an error. Status=500
  Code="InternalServerError" Message="Encountered an internal server error.`

#### <a name="cause"></a>Causa

El proveedor de recursos "Microsoft.PolicyInsights" no está registrado. Se debe registrar para que el complemento obtenga las definiciones de directiva y devuelva datos de cumplimiento.

#### <a name="resolution"></a>Solución

Registre el proveedor de recursos "Microsoft.PolicyInsights" en la suscripción del clúster. Puede encontrar instrucciones en [Registro de un proveedor de recursos](../../../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider).

### <a name="scenario-the-subscription-is-disabled"></a>Escenario: La suscripción está deshabilitada

#### <a name="issue"></a>Problema

El complemento puede acceder al punto de conexión de servicio de Azure Policy, pero se muestra el siguiente error:

`The subscription '{subId}' has been disabled for azure data-plane policy. Please contact support.`

#### <a name="cause"></a>Causa

Este error significa que la suscripción se determinó como problemática y se agregó la marca de características `Microsoft.PolicyInsights/DataPlaneBlocked` para bloquear la suscripción.

#### <a name="resolution"></a>Solución

Para investigar y resolver este problema, [póngase en contacto con el equipo de características](mailto:azuredg@microsoft.com).

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece en este artículo o no lo puede resolver, visite uno de los siguientes canales para obtener ayuda:

- Obtenga respuestas de expertos a través de [Preguntas y respuestas de Microsoft](/answers/topics/azure-policy.html).
- Conectar con [@AzureSupport](https://twitter.com/azuresupport). Este recurso oficial de Microsoft Azure en Twitter ayuda a mejorar la experiencia del cliente al conectar la comunidad de Azure a las respuestas, el soporte técnico y los expertos adecuados.
- Si sigue necesitando ayuda, vaya al [sitio de soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Enviar una solicitud de soporte técnico**.
