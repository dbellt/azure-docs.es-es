---
title: Actualización de un clúster para usar el nombre común del certificado
description: Obtenga información sobre cómo convertir en nombres comunes un certificado de clúster de Azure Service Fabric basado en huella digital.
ms.topic: conceptual
ms.date: 09/06/2019
ms.openlocfilehash: daf8d55e156f30b1f9e9ec5c50d60714e6f17884
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107308020"
---
# <a name="convert-cluster-certificates-from-thumbprint-based-declarations-to-common-names"></a>Conversión de certificados de Azure basados en huella digital en nombres comunes

La firma de un certificado (conocida comúnmente como huella digital) es única. Un certificado de clúster declarado mediante huella digital hace referencia a una instancia específica de un certificado. Esta especificidad hace que la sustitución de certificados y la administración en general sean difíciles y explícitas. Para cada cambio es necesario orquestar las actualizaciones del clúster y los hosts informáticos subyacentes.

La conversión de las declaraciones de certificado de un clúster de Azure Service Fabric de declaraciones basadas en huellas digitales a otras basadas en el nombre común (CN) del firmante del certificado simplifica considerablemente la administración. En concreto, para sustituir un certificado ya no es necesario actualizar el clúster. En este artículo se describe cómo convertir un clúster existente en declaraciones basadas en CN sin tiempo de inactividad.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="move-to-certificate-authority-signed-certificates"></a>Cambio a certificados firmados por una entidad de certificación

La seguridad de un clúster cuyo certificado se declara mediante la huella digital radica en que no es posible, o no es factible desde el punto de vista de los recursos, falsificar un certificado con la misma firma que el de otro. En este caso, la procedencia del certificado es menos importante, por lo que los certificados autofirmados son adecuados.

Por el contrario, la seguridad de un clúster cuyos certificados se declaran mediante CN se origina en la confianza implícita que el propietario del clúster tiene en su proveedor de certificados. El proveedor es el servicio de infraestructura de clave pública (PKI) que ha emitido el certificado. La confianza se basa, entre otros factores, en los procedimientos de certificación de la PKI, con independencia de que su seguridad operativa la auditen y aprueben otras partes de confianza, etc.

El propietario del clúster también debe tener un conocimiento detallado de las entidades de certificación (CA) que emiten sus certificados, ya que se trata de un aspecto fundamental de la validación de certificados por firmante. Esto también implica que los certificados autofirmados no son adecuados para este fin. Literalmente, cualquier usuario puede generar un certificado con un firmante concreto.

Normalmente, un certificado declarado por CN se considera válido si:

* Su cadena se puede crear de forma correcta.
* El firmante tiene el elemento de CN esperado.
* Su emisor (inmediato o superior en la cadena) es de confianza para el agente que realiza la validación.

Service Fabric admite la declaración de certificados mediante CN de dos maneras:

* Con emisores *implícitos*, lo que significa que la cadena debe terminar en un anclaje de veracidad.
* Con emisores declarados mediante huella digital, lo que se conoce como asignación del emisor.

Para obtener más información, vea [Declaraciones de validación de certificados basadas en nombres comunes](cluster-security-certificates.md#common-name-based-certificate-validation-declarations).

Para convertir a CN un clúster mediante un certificado autofirmado declarado por huella digital, primero se debe introducir el certificado firmado por la CA en el clúster mediante la huella digital. Solo después se podrá realizar la conversión de la huella digital a CN.

Con fines de prueba, un certificado autofirmado *podría* ser declarado mediante CN, pero solo si el emisor está asignado a su propia huella digital. Desde el punto de vista de la seguridad, esta acción es casi equivalente a declarar el mismo certificado mediante la huella digital. Una conversión correcta de este tipo no garantiza una conversión correcta de la huella digital a CN con un certificado firmado por una entidad de certificación. Se recomienda probar la conversión con un certificado adecuado firmado por una CA. Existen opciones gratuitas para realizar esta prueba.

## <a name="upload-the-certificate-and-install-it-in-the-scale-set"></a>Cargar el certificado e instalarlo en el conjunto de escalado

En Azure, el mecanismo recomendado para obtener y aprovisionar certificados implica Azure Key Vault y sus herramientas. En todos los nodos de los conjuntos de escalado de máquinas virtuales que componen el clúster se debe aprovisionar un certificado que coincida con la declaración de certificado de clúster. Para obtener más información, vea [Secretos de conjuntos de escalado de máquinas virtuales](../virtual-machine-scale-sets/virtual-machine-scale-sets-faq.yml#how-do-i-securely-ship-a-certificate-to-the-vm-).

Es importante instalar los certificados de clúster actuales y de destino en las máquinas virtuales de cada tipo de nodo del clúster antes de realizar cambios en las declaraciones de certificado del clúster. El recorrido de la emisión del certificado al aprovisionamiento en un nodo de Service Fabric se analiza en profundidad en [Recorrido de un certificado](cluster-security-certificate-management.md#the-journey-of-a-certificate).

## <a name="bring-the-cluster-to-an-optimal-starting-state"></a>Activación del clúster en un estado de inicio óptimo

La conversión de una declaración de certificado de basada en huella digital a basada en CN afecta a lo siguiente:

- La forma en que cada nodo del clúster busca sus credenciales y las presenta a otros nodos.
- La forma en que cada nodo valida las credenciales de su homólogo al establecer una conexión segura.

Revise las reglas de [presentación y validación de ambas configuraciones](cluster-security-certificates.md#certificate-configuration-rules) antes de continuar. La consideración más importante a la hora de realizar una conversión de huella digital a CN es que los nodos actualizados y no actualizados (es decir, los nodos que pertenecen a otros dominios de actualización) deben poder realizar la autenticación mutua correctamente en cualquier momento durante la actualización. La forma recomendada de lograr este comportamiento consiste en declarar el certificado de destino por huella digital en una actualización inicial. Después, complete la transición a CN en una segunda actualización. Si el clúster ya está en un estado de inicio recomendado, puede omitir esta sección.

Hay varios estados de inicio válidos para una conversión. Lo que no varía es que el clúster ya usa el certificado de destino (declarado mediante la huella digital) al inicio de la actualización a CN. En este artículo se consideran `GoalCert`, `OldCert1` y `OldCert2`.

#### <a name="valid-starting-states"></a>Estados de inicio válidos

- `Thumbprint: GoalCert, ThumbprintSecondary: None`
- `Thumbprint: GoalCert, ThumbprintSecondary: OldCert1`, donde `GoalCert` tiene una fecha `NotBefore` posterior a la de `OldCert1`
- `Thumbprint: OldCert1, ThumbprintSecondary: GoalCert`, donde `GoalCert` tiene una fecha `NotBefore` posterior a la de `OldCert1`

> [!NOTE]
> Antes de la versión 7.2.445 (7.2 CU4), Service Fabric seleccionaba el certificado que tardaba más en expirar (el certificado con la propiedad "NotAfter" más lejana), por lo que los estados iniciales anteriores a 7.2 CU4 requieren que GoalCert tenga una fecha de `NotAfter` posterior a `OldCert1`

Si el clúster no está en uno de los estados válidos descritos antes, vea la información sobre cómo lograr ese estado en la sección al final de este artículo.

## <a name="select-the-desired-cn-based-certificate-validation-scheme"></a>Selección del esquema deseado de validación de certificados basados en CN

Como se ha descrito antes, Service Fabric admite la declaración de certificados por CN con un anclaje de veracidad implícito o con la asignación explícita de las huellas digitales del emisor. Para obtener más información, vea [Declaraciones de validación de certificados basadas en nombres comunes](cluster-security-certificates.md#common-name-based-certificate-validation-declarations).

Asegúrese de que comprende bien las diferencias y las implicaciones de la elección de cualquiera de estos mecanismos. Sintácticamente, esta diferencia u opción viene determinada por el valor del parámetro `certificateIssuerThumbprintList`. Vacío significa que se confía en una CA raíz de confianza (anclaje de veracidad), mientras que un conjunto de huellas digitales restringe los emisores directos permitidos de los certificados de clúster.

   > [!NOTE]
   > El campo `certificateIssuerThumbprint` permite especificar los emisores directos esperados de los certificados declarados por el CN del firmante. Los valores aceptables son una o varias huellas digitales SHA1 separadas por comas. Esta acción refuerza la validación del certificado.
   >
   > Si no se especifica ningún emisor o la lista está vacía, se aceptará el certificado para la autenticación si su cadena se puede crear. Después, el certificado finaliza en una raíz de confianza para el validador. Si se especifica una o varias huellas digitales de emisor, el certificado se aceptará si la huella digital de su emisor directo, extraída de la cadena, coincide con cualquiera de los valores especificados en este campo. El certificado se aceptará con independencia de que la raíz sea de confianza o no.
   >
   > Es posible que una PKI use otras entidades de certificación (también conocidas como *emisores*) para firmar certificados con un firmante determinado. Por esta razón, es importante especificar todas las huellas digitales del emisor esperado para ese firmante. Es decir, no se garantiza que la renovación de un certificado esté firmada por el mismo emisor que el del certificado que se renueva.
   >
   > Un procedimiento recomendado consiste en especificar el emisor. La omisión del emisor seguirá funcionando para los certificados vinculados a una raíz de confianza, pero este comportamiento tiene limitaciones y es posible que desaparezca en un futuro próximo. Los clústeres implementados en Azure, protegidos con certificados X509 emitidos por una PKI privada y declarados por firmante, podrían no ser validados por Service Fabric (para la comunicación entre el clúster y el servicio). La validación requiere que la directiva de certificados de la PKI sea reconocible, disponible y accesible.

## <a name="update-the-clusters-azure-resource-manager-template-and-deploy"></a>Actualización e implementación de la plantilla de Azure Resource Manager del clúster

Administre los clústeres de Service Fabric con plantillas de Azure Resource Manager (ARM). Una alternativa, en la que también se usan artefactos JSON, es [Azure Resource Explorer (versión preliminar)](https://resources.azure.com). En este momento no hay disponible una experiencia equivalente en Azure Portal.

Si la plantilla original correspondiente a un clúster existente no está disponible, se puede obtener una plantilla equivalente en Azure Portal. Vaya al grupo de recursos que contiene el clúster y seleccione **Exportar plantilla** en el menú **Automation** de la izquierda. Después, seleccione los recursos que quiera. Como mínimo, se deben exportar los recursos de conjunto de escalado de máquinas virtuales y clúster, respectivamente. La plantilla generada también se puede descargar. Es posible que haya que modificar esta plantilla antes de que se pueda implementar completamente. También es posible que no coincida exactamente con la original. Es un reflejo del estado actual del recurso de clúster.

Los cambios necesarios son los siguientes:

- Actualización de la definición de la extensión de nodo de Service Fabric (en el recurso de máquina virtual). Si el clúster define varios tipos de nodo, tendrá que actualizar la definición de cada conjunto de escalado de máquinas virtuales correspondiente.
- Actualización de la definición del recurso de clúster.

Aquí se incluyen ejemplos detallados.

### <a name="update-the-virtual-machine-scale-set-resources"></a>Actualización de los recursos de conjunto de escalado de máquinas virtuales
De:
```json
"virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
                    "properties": {
                        "type": "ServiceFabricNode",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                            ...
                        },
                        "publisher": "Microsoft.Azure.ServiceFabric",
                        "settings": {
                            ...
                            "certificate": {
                                "thumbprint": "[parameters('certificateThumbprint')]",
                                "x509StoreName": "[parameters('certificateStoreValue')]"
                            }
                        },
                        ...
                    }
                },
```
A:
```json
"virtualMachineProfile": {
        "extensionProfile": {
            "extensions": [
                {
                    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
                    "properties": {
                        "type": "ServiceFabricNode",
                        "autoUpgradeMinorVersion": true,
                        "protectedSettings": {
                            ...
                        },
                        "publisher": "Microsoft.Azure.ServiceFabric",
                        "settings": {
                            ...
                            "certificate": {
                                "commonNames": [
                                    "[parameters('certificateCommonName')]"
                                ],
                                "x509StoreName": "[parameters('certificateStoreValue')]"
                            }
                        },
                        ...
                    }
                },
```

### <a name="update-the-cluster-resource"></a>Actualización del recurso de clúster

En el recurso **Microsoft.ServiceFabric/clusters**, agregue una propiedad **certificateCommonNames** con un valor **commonNames** y quite la propiedad **certificate** (toda su configuración).

De:
```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            ...
        ],
        "properties": {
            "addonFeatures": [
                ...
            ],
            "certificate": {
              "thumbprint": "[parameters('certificateThumbprint')]",
              "x509StoreName": "[parameters('certificateStoreValue')]"
            },
        ...
```
A:
```json
    {
        "apiVersion": "2018-02-01",
        "type": "Microsoft.ServiceFabric/clusters",
        "name": "[parameters('clusterName')]",
        "location": "[parameters('clusterLocation')]",
        "dependsOn": [
            ...
        ],
        "properties": {
            "addonFeatures": [
                ...
            ],
            "certificateCommonNames": {
                "commonNames": [
                    {
                        "certificateCommonName": "[parameters('certificateCommonName')]",
                        "certificateIssuerThumbprint": "[parameters('certificateIssuerThumbprintList')]"
                    }
                ],
                "x509StoreName": "[parameters('certificateStoreValue')]"
            },
        ...
```

Para obtener más información, vea [Implementación de un clúster de Service Fabric que usa un nombre común de certificado en lugar de una huella digital](./service-fabric-create-cluster-using-cert-cn.md).

## <a name="deploy-the-updated-template"></a>Implementar la plantilla actualizada

Vuelva a implementar la plantilla actualizada después de realizar los cambios.

```powershell
$groupname = "sfclustertutorialgroup"

New-AzResourceGroupDeployment -ResourceGroupName $groupname -Verbose `
    -TemplateParameterFile "C:\temp\cluster\parameters.json" -TemplateFile "C:\temp\cluster\template.json" 
```

## <a name="achieve-a-valid-starting-state-for-converting-a-cluster-to-cn-based-certificate-declarations"></a>Obtención de un estado inicial válido para convertir un clúster en declaraciones de certificado basadas en CN

| Estado inicial | Actualización 1 | Actualización 2 |
| :--- | :--- | :--- |
| `Thumbprint: OldCert1, ThumbprintSecondary: None` y `GoalCert` tiene una fecha `NotBefore` posterior a la de `OldCert1` | `Thumbprint: OldCert1, ThumbprintSecondary: GoalCert` | - |
| `Thumbprint: OldCert1, ThumbprintSecondary: None` y `OldCert1` tiene una fecha `NotBefore` posterior a la de `GoalCert` | `Thumbprint: GoalCert, ThumbprintSecondary: OldCert1` | `Thumbprint: GoalCert, ThumbprintSecondary: None` |
| `Thumbprint: OldCert1, ThumbprintSecondary: GoalCert`, donde `OldCert1` tiene una fecha `NotBefore` posterior a la de `GoalCert` | Actualice a `Thumbprint: GoalCert, ThumbprintSecondary: None` | - |
| `Thumbprint: GoalCert, ThumbprintSecondary: OldCert1`, donde `OldCert1` tiene una fecha `NotBefore` posterior a la de `GoalCert` | Actualice a `Thumbprint: GoalCert, ThumbprintSecondary: None` | - |
| `Thumbprint: OldCert1, ThumbprintSecondary: OldCert2` | Quite `OldCert1` o `OldCert2` para obtener el estado `Thumbprint: OldCertx, ThumbprintSecondary: None` | Continúe desde el nuevo estado inicial |

> [!NOTE]
> Para un clúster de una versión anterior a la 7.2.445 (7.2 CU4), reemplace `NotBefore` por `NotAfter` en los estados anteriores.

Para obtener instrucciones sobre cómo realizar cualquiera de estas actualizaciones, vea [Administración de certificados en un clúster de Azure Service Fabric](service-fabric-cluster-security-update-certs-azure.md).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre la [seguridad del clúster](service-fabric-cluster-security.md).
* Obtenga información sobre cómo [sustituir un certificado de clúster por nombre común](service-fabric-cluster-rollover-cert-cn.md).
* Obtenga información sobre cómo [configurar un clúster para la sustitución automática sin interacción](cluster-security-certificate-management.md).

[image1]: ./media/service-fabric-cluster-change-cert-thumbprint-to-cn/PortalViewTemplates.png
