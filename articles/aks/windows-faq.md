---
title: Preguntas frecuentes sobre los grupos de nodos de Windows Server
titleSuffix: Azure Kubernetes Service
description: Vea las preguntas frecuentes cuando ejecute cargas de trabajo de aplicaciones y grupos de nodos de Windows Server en Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 10/12/2020
ms.openlocfilehash: 5d3d78eb20a9ca8b663fa0cf381fcce1bd528345
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110463310"
---
# <a name="frequently-asked-questions-for-windows-server-node-pools-in-aks"></a>Preguntas frecuentes sobre los grupos de nodos de Windows Server en AKS

En Azure Kubernetes Service (AKS) puede crear un grupo de nodos que ejecuta Windows Server como sistema operativo invitado en los nodos. Estos nodos pueden ejecutar aplicaciones de contenedor nativas de Windows, como las integradas en .NET Framework. Hay diferencias con respecto al modo en que los sistemas operativos Linux y Windows proporcionan compatibilidad de contenedor. Algunas características comunes de Linux relacionadas con Kubernetes y los pods no están disponibles en este momento para los grupos de nodos de Windows.

En este artículo se esbozan algunas de las preguntas frecuentes y los conceptos de sistema operativo sobre los nodos de Windows Server en AKS.

## <a name="which-windows-operating-systems-are-supported"></a>¿Qué sistemas operativos de Windows se admiten?

AKS usa Windows Server 2019 como la versión del sistema operativo del host y solo admite el aislamiento de procesos. No se admiten las imágenes de contenedor compiladas con otras versiones de Windows Server. Para obtener más información, vea [Compatibilidad con versiones de contenedores de Windows][windows-container-compat].

## <a name="is-kubernetes-different-on-windows-and-linux"></a>¿Es Kubernetes diferente en Windows y Linux?

La compatibilidad del grupo de nodos de Windows Server incluye algunas limitaciones que forman parte de Windows Server ascendente en el proyecto de Kubernetes. Estas limitaciones no son específicas de AKS. Para obtener más información sobre esta compatibilidad ascendente con Windows Server en Kubernetes, consulte la sección de [funcionalidades y limitaciones admitidas][upstream-limitations] del documento [Información sobre la compatibilidad de Windows en Kubernetes][intro-windows], desde el proyecto Kubernetes.

Kubernetes se ha centrado siempre en Linux. Muchos ejemplos usados en el sitio web [Kubernetes.io][kubernetes] ascendente están diseñados para su uso en nodos de Linux. Al crear implementaciones que usan contenedores de Windows Server, se aplican las consideraciones siguientes en el nivel de sistema operativo:

- **Identidad**: Linux identifica a un usuario mediante un identificador de usuario numérico de enteros (UID). Los usuarios también tienen un nombre de usuario alfanumérico para iniciar sesión, que se convierte en Linux en su UID. De igual forma, Linux identifica a los grupos de usuarios mediante un identificador de grupo numérico de enteros (GID) y traduce el nombre de grupo en su GID correspondiente.
    - Windows Server usa un identificador de seguridad binario de mayor tamaño (SID) que se almacena en la base de datos de Windows Security Access Manager (SAM). Esta base de datos no se comparte entre el host y los contenedores ni entre contenedores.
- **Permisos de archivos**: Windows Server utiliza una lista de control de acceso basada en SID, en lugar de una máscara de bits de permisos y UID+GID.
- **Rutas de archivo**: la convención en Windows Server consiste en usar \ en lugar de /.
    - En las especificaciones de pod que montan los volúmenes, especifique la ruta correctamente para los contenedores de Windows Server. Por ejemplo, en lugar de un punto de montaje de */mnt/volume* en un contenedor de Linux, especifique la letra de unidad y la ubicación como */K/Volume* para montarlo como la unidad *K:* .

## <a name="what-kind-of-disks-are-supported-for-windows"></a>¿Qué tipo de discos se admiten en Windows?

Azure Disks y Azure Files son los tipos de volúmenes admitidos. Se accede a ellos como volúmenes NTFS del contenedor de Windows Server.

## <a name="can-i-run-windows-only-clusters-in-aks"></a>¿Puedo ejecutar clústeres que sean solo de Windows en AKS?

AKS hospeda los nodos maestros (esto es, el plano de control) de un clúster de AKS en el servicio, y usted no verá el sistema operativo de los nodos que hospedan los componentes maestros. Todos los clústeres de AKS se crean con un primer grupo de nodos predeterminado basado en Linux. Este grupo de nodos contiene los servicios del sistema necesarios para que el clúster funcione. Se recomienda ejecutar al menos dos nodos en el primer grupo de nodos para garantizar la confiabilidad del clúster y que tenga la capacidad de realizar operaciones de clúster. No se puede eliminar este primer grupo de nodos basado en Linux, a menos que se elimine el propio clúster de AKS.

## <a name="how-do-i-patch-my-windows-nodes"></a>¿Cómo se aplica una revisión a los nodos de Windows?

Para obtener las revisiones más recientes de los nodos de Windows, puede [actualizar el grupo de nodos][nodepool-upgrade] o [actualizar la imagen de los nodos][upgrade-node-image]. Las actualizaciones de Windows no están habilitadas en los nodos de AKS. AKS publica nuevas imágenes de los grupos de nodos en cuanto hay revisiones disponibles; es responsabilidad de los usuarios actualizar los grupos de nodos para mantenerse al día de las revisiones y correcciones. Esto también se aplica a la versión de Kubernetes que se esté usando. Las [notas de la versión de AKS][aks-release-notes] indican cuándo hay nuevas versiones disponibles. Para obtener más información sobre la actualización del grupo de nodos completo de Windows Server, vea [Actualización de un grupo de nodos en AKS][nodepool-upgrade]. Si solo está interesado en actualizar la imagen de los nodos, vea [Actualizaciones de imágenes de nodo de AKS][upgrade-node-image].

> [!NOTE]
> La imagen de Windows Server actualizada solo se usa si se ha realizado una actualización del clúster (actualización del plano de control) antes de actualizar el grupo de nodos.
>

## <a name="what-network-plug-ins-are-supported"></a>¿Qué complementos de red se admiten?

Los clústeres de AKS con grupos de nodos de Windows deben usar el modelo de redes (avanzado) de Azure CNI. No se admiten las redes Kubenet (básico). Para obtener más información sobre las diferencias en los modelos de redes, consulte [Conceptos de redes para las aplicaciones en AKS][azure-network-models]. El modelo de redes de Azure CNI requiere una planeación y consideraciones adicionales para la administración de direcciones IP. Para obtener más información sobre cómo planear e implementar Azure CNI, consulte [Configuración de redes de Azure CNI en AKS][configure-azure-cni].

Los nodos de Windows en los clústeres de AKS también tienen habilitado [Direct Server Return (DSR)][dsr] de forma predeterminada cuando Calico está habilitado.

## <a name="is-preserving-the-client-source-ip-supported"></a>¿Se admite la conservación de la IP de origen del cliente?

En este momento, la [conservación de la IP de origen del cliente][client-source-ip] no es compatible con los nodos de Windows.

## <a name="can-i-change-the-max--of-pods-per-node"></a>¿Puedo cambiar el número máximo de pods por nodo?

Sí. Para conocer las implicaciones y las opciones disponibles, consulte [Pods máximos por nodo][maximum-number-of-pods].

## <a name="why-am-i-seeing-an-error-when-i-try-to-create-a-new-windows-agent-pool"></a>¿Por qué se muestra un error al tratar de crear un nuevo grupo de agentes de Windows?

Si ha creado el clúster antes de febrero de 2020 y nunca ha realizado ninguna operación de actualización, este sigue usando una imagen de Windows antigua. Es posible que se haya mostrado un error similar al siguiente:

"No se encontró la siguiente lista de imágenes a las que se hace referencia en la plantilla de implementación: Publicador: MicrosoftWindowsServer, Oferta: WindowsServer, SKU: 2019-datacenter-core-smalldisk-2004, Versión: más reciente. Consulte https://docs.microsoft.com/azure/virtual-machines/windows/cli-ps-findimage para obtener instrucciones sobre cómo buscar las imágenes disponibles".

Para corregir este error:

1. Actualice el [plano de control del clúster][upgrade-cluster-cp] a fin de actualizar la oferta y el publicador de la imagen.
1. Cree nuevos grupos de agentes de Windows.
1. Mueva los pods de Windows de los grupos de agentes de Windows existentes a los nuevos.
1. Elimine los grupos de agentes de Windows anteriores.

## <a name="how-do-i-rotate-the-service-principal-for-my-windows-node-pool"></a>¿Cómo se realiza la rotación de la entidad de servicio para el grupo de nodos de Windows?

Los grupos de nodos de Windows no admiten la rotación de la entidad de servicio. Para actualizar la entidad de servicio, cree un nuevo grupo de nodos de Windows y migre los pods del grupo anterior al nuevo. Cuando termine, elimine el grupo de nodos antiguo.

En su lugar, use identidades administradas que, básicamente, son contenedores en torno a entidades de servicio. Para obtener más información, consulte [Uso de identidades administradas en Azure Kubernetes Service][managed-identity].

## <a name="how-do-i-change-the-administrator-password-for-windows-server-nodes-on-my-cluster"></a>¿Cómo cambio la contraseña de administrador de los nodos de Windows Server en el clúster?

Al crear el clúster de AKS, se especifican los parámetros `--windows-admin-password` y `--windows-admin-username` para establecer las credenciales de administrador de los nodos de Windows Server del clúster. Si no especificó la credenciales de administrador como, por ejemplo, al crear un clúster mediante Azure Portal o al establecer `--vm-set-type VirtualMachineScaleSets` y `--network-plugin azure` mediante la CLI de Azure, el nombre de usuario tiene como valor predeterminado *azureuser* y una contraseña aleatoria.

Para cambiar la contraseña de administrador, use el comando `az aks update`:

```azurecli
az aks update \
    --resource-group $RESOURCE_GROUP \
    --name $CLUSTER_NAME \
    --windows-admin-password $NEW_PW
```

> [!IMPORTANT]
> Al realizar esta operación, se actualizarán todos los grupos de nodos de Windows Server. Los grupos de nodos de Linux no se ven afectados.
> 
> Al cambiar `--windows-admin-password`, la nueva contraseña debe tener al menos 14 caracteres y cumplir los [requisitos de contraseña de Windows Server][windows-server-password].

## <a name="how-many-node-pools-can-i-create"></a>¿Cuántos grupos de nodos puedo crear?

El clúster de AKS puede tener un máximo de 10 grupos de nodos. Puede tener un máximo de 1000 nodos distribuidos entre esos grupos de nodos. [Limitaciones del grupo de nodos][nodepool-limitations].

## <a name="what-can-i-name-my-windows-node-pools"></a>¿Qué nombre puedo usar para mis grupos de nodos de Windows?

El nombre debe tener un máximo de 6 (seis) caracteres. Esta es una limitación de AKS.

## <a name="are-all-features-supported-with-windows-nodes"></a>¿Son todas las características compatibles con los nodos de Windows?

Kubenet actualmente no es compatible con los nodos de Windows.

## <a name="can-i-run-ingress-controllers-on-windows-nodes"></a>¿Puedo ejecutar controladores de entrada en los nodos de Windows?

Sí, un controlador de entrada que admita contenedores de Windows Server se puede ejecutar en nodos de Windows en AKS.

## <a name="can-my-windows-server-containers-use-gmsa"></a>¿Pueden usar gMSA los contenedores de Windows Server?

La compatibilidad con las cuentas de servicio administradas de grupo (gMSA) no está disponible actualmente en AKS.

## <a name="can-i-use-azure-monitor-for-containers-with-windows-nodes-and-containers"></a>¿Puedo usar Azure Monitor para contenedores con nodos y contenedores de Windows?

Sí puede; no obstante, Azure Monitor se encuentra en versión preliminar pública para la recopilación de registros (stdout, stderr) y métricas de los contenedores de Windows. También puede conectarse al streaming en vivo de registros de stdout desde un contenedor de Windows.

## <a name="are-there-any-limitations-on-the-number-of-services-on-a-cluster-with-windows-nodes"></a>¿Existe alguna limitación en el número de servicios que puede haber en un clúster con nodos de Windows?

Un clúster con nodos de Windows puede tener aproximadamente 500 servicios antes de que se agote el puerto.

## <a name="can-i-use-azure-hybrid-benefit-with-windows-nodes"></a>¿Puedo usar Ventaja híbrida de Azure con los nodos de Windows?

Sí. Ventaja híbrida de Azure para Windows Server reduce los costes operativos, al permitirle llevar su licencia de Windows Server local a los nodos de Windows de AKS.

Ventaja híbrida de Azure puede usarse en todo el clúster de AKS o en nodos individuales. Para los nodos individuales, debe navegar hasta el [grupo de recursos de nodos][resource-groups] y aplicar Ventaja híbrida de Azure directamente a los nodos. Para obtener más información sobre cómo aplicar Ventaja híbrida de Azure a nodos individuales, consulte [Ventaja híbrida de Azure para Windows Server][hybrid-vms]. 

Para usar Ventaja híbrida de Azure en un nuevo clúster de AKS, use el argumento `--enable-ahub`.

```azurecli
az aks create \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --load-balancer-sku Standard \
    --windows-admin-password 'Password1234$' \
    --windows-admin-username azure \
    --network-plugin azure
    --enable-ahub
```

Para usar Ventaja híbrida de Azure en un clúster de AKS existente, actualice el clúster con el argumento `--enable-ahub`.

```azurecli
az aks update \
    --resource-group myResourceGroup
    --name myAKSCluster
    --enable-ahub
```

Para comprobar si se ha establecido Ventaja híbrida de Azure en el clúster, use el siguiente comando:

```azurecli
az vmss show --name myAKSCluster --resource-group MC_CLUSTERNAME
```

Si el clúster tiene Ventaja híbrida de Azure habilitada, la salida de `az vmss show` será similar a la siguiente:

```console
"platformFaultDomainCount": 1,
  "provisioningState": "Succeeded",
  "proximityPlacementGroup": null,
  "resourceGroup": "MC_CLUSTERNAME"
```

## <a name="can-i-use-the-kubernetes-web-dashboard-with-windows-containers"></a>¿Puedo usar el panel web de Kubernetes con los contenedores de Windows?

Sí, puede usar el [panel web de Kubernetes][kubernetes-dashboard] para acceder a la información sobre los contenedores de Windows, pero en este momento no puede ejecutar *kubectl exec* en un contenedor de Windows en ejecución directamente desde el panel web de Kubernetes. Para obtener más detalles sobre la conexión al contenedor de Windows en ejecución, vea [Conexión con RDP a los nodos de Windows Server de clúster de Azure Kubernetes Service (AKS) para el mantenimiento o la solución de problemas][windows-rdp].

## <a name="how-do-i-change-the-time-zone-of-a-running-container"></a>¿Cómo se cambia la zona horaria de un contenedor en ejecución?

Para cambiar la zona horaria de un contenedor de Windows Server en ejecución, conéctese al contenedor en ejecución con una sesión de PowerShell. Por ejemplo:
    
```azurecli-interactive
kubectl exec -it CONTAINER-NAME -- powershell
```

En el contenedor en ejecución, use [Set-TimeZone](/powershell/module/microsoft.powershell.management/set-timezone) para establecer la zona horaria del contenedor en ejecución. Por ejemplo:

```powershell
Set-TimeZone -Id "Russian Standard Time"
```

Para ver la zona horaria actual del contenedor en ejecución o una lista disponible de zonas horarias, use [Get-TimeZone](/powershell/module/microsoft.powershell.management/get-timezone).

## <a name="what-if-i-need-a-feature-thats-not-supported"></a>¿Qué ocurre si necesito una característica que no se admite?

Estamos trabajando día y noche para poner a su disposición todas las características que necesite para Windows en AKS, pero si encuentra errores, el proyecto ascendente de código abierto [aks-engine][aks-engine], proporciona una manera fácil y totalmente personalizable de ejecutar Kubernetes en Azure e incluye el soporte técnico de Windows. Asegúrese de consultar el plan de desarrollo de próximas características en el [Plan de desarrollo de AKS][aks-roadmap].

## <a name="next-steps"></a>Pasos siguientes

Para comenzar con los contenedores de Windows Server en AKS, [cree un grupo de nodos que ejecute Windows Server en AKS][windows-node-cli].

<!-- LINKS - external -->
[kubernetes]: https://kubernetes.io
[aks-engine]: https://github.com/azure/aks-engine
[upstream-limitations]: https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/#supported-functionality-and-limitations
[intro-windows]: https://kubernetes.io/docs/setup/production-environment/windows/intro-windows-in-kubernetes/
[aks-roadmap]: https://github.com/Azure/AKS/projects/1
[aks-release-notes]: https://github.com/Azure/AKS/releases

<!-- LINKS - internal -->
[azure-network-models]: concepts-network.md#azure-virtual-networks
[configure-azure-cni]: configure-azure-cni.md
[nodepool-upgrade]: use-multiple-node-pools.md#upgrade-a-node-pool
[windows-node-cli]: windows-container-cli.md
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[upgrade-cluster]: upgrade-cluster.md
[upgrade-cluster-cp]: use-multiple-node-pools.md#upgrade-a-cluster-control-plane-with-multiple-node-pools
[azure-outbound-traffic]: ../load-balancer/load-balancer-outbound-connections.md#defaultsnat
[nodepool-limitations]: use-multiple-node-pools.md#limitations
[windows-container-compat]: /virtualization/windowscontainers/deploy-containers/version-compatibility?tabs=windows-server-2019%2Cwindows-10-1909
[maximum-number-of-pods]: configure-azure-cni.md#maximum-pods-per-node
[azure-monitor]: ../azure-monitor/containers/container-insights-overview.md#what-does-azure-monitor-for-containers-provide
[client-source-ip]: concepts-network.md#ingress-controllers
[kubernetes-dashboard]: kubernetes-dashboard.md
[windows-rdp]: rdp.md
[upgrade-node-image]: node-image-upgrade.md
[managed-identity]: use-managed-identity.md
[hybrid-vms]: ../virtual-machines/windows/hybrid-use-benefit-licensing.md
[resource-groups]: faq.md#why-are-two-resource-groups-created-with-aks
[dsr]: ../load-balancer/load-balancer-multivip-overview.md#rule-type-2-backend-port-reuse-by-using-floating-ip
[windows-server-password]: /windows/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements#reference