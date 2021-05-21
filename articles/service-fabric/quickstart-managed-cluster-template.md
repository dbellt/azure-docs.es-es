---
title: Implementación de un clúster administrado de Service Fabric con Azure Resource Manager
description: Aprenda a crear un clúster administrado Service Fabric con una plantilla de Azure Resource Manager.
ms.topic: quickstart
ms.date: 5/10/2021
ms.openlocfilehash: 0775d5f6874de8fafb1d9a32cf19071f4a7a99b4
ms.sourcegitcommit: b35c7f3e7f0e30d337db382abb7c11a69723997e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/10/2021
ms.locfileid: "109685214"
---
# <a name="quickstart-deploy-a-service-fabric-managed-cluster-with-an-azure-resource-manager-template"></a>Inicio rápido: Implementación de un clúster administrado de Service Fabric con una plantilla de Azure Resource Manager

Los clústeres administrados de Service Fabric son una evolución del modelo de recursos de clúster de Azure Service Fabric que agiliza la experiencia de implementación y administración de clústeres. Los clústeres administrados de Service Fabric son un recurso totalmente encapsulado que le permite implementar un único recurso de clúster de Service Fabric en lugar de tener que implementar todos los recursos subyacentes que componen un clúster de Service Fabric. En este artículo se describe cómo implementar un clúster administrado de Service Fabric para probar en Azure mediante una plantilla de Azure Resource Manager (plantilla de ARM).

El clúster de SKU básico de tres nodos implementado en este tutorial solo está pensado para usarse con fines informativos (en lugar de cargas de trabajo de producción). Para obtener más información, consulte [SKU de clúster administrado de Service Fabric](overview-managed-cluster.md#service-fabric-managed-cluster-skus).

## <a name="prerequisites"></a>Requisitos previos

Antes de empezar este inicio rápido:

* Si no tiene ninguna suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de [Ejemplos de Azure: plantillas de clúster de Service Fabric](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/SF-Managed-Basic-SKU-1-NT).

## <a name="create-a-client-certificate"></a>Creación de un certificado de cliente

Los clústeres administrados de Service Fabric usan un certificado de cliente como clave para el control de acceso. Si ya tiene un certificado de cliente que le gustaría usar para el control de acceso al clúster, puede omitir este paso.

Si necesita crear un nuevo certificado de cliente, siga los pasos descritos en [Establecimiento y recuperación de un certificado de Azure Key Vault](../key-vault/certificates/quick-create-portal.md).

Tome nota de la huella digital del certificado, ya que será necesaria para implementar la plantilla en el paso siguiente.

## <a name="deploy-the-template"></a>Implementación de la plantilla

1. Seleccione la imagen siguiente para iniciar sesión en Azure y abrir una plantilla.

      [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fservice-fabric-cluster-templates%2Fmaster%2FSF-Managed-Basic-SKU-1-NT%2Fazuredeploy.json)

2. Seleccione o escriba los siguientes valores.

    Para este inicio rápido, proporcione sus propios valores para los siguientes parámetros de plantilla:

    * **Suscripción**: Seleccione una suscripción de Azure.
    * **Grupo de recursos**: Seleccione **Crear nuevo**. Escriba un nombre único para el grupo de recursos, como *myResourceGroup*, y elija **Aceptar**.
    * **Ubicación**: Seleccione una ubicación.
    * **Nombre del clúster**: escriba un nombre único para el clúster, como *mysfcluster*.
    * **Nombre de usuario administrador**: escriba un nombre para el administrador que se va a usar para RDP en las máquinas virtuales subyacentes del clúster.
    * **Contraseña de administrador**: escriba una contraseña para el administrador que se va a usar para RDP en las máquinas virtuales subyacentes del clúster.
    * **Huella digital de certificado de cliente**: Proporcione la huella digital del certificado de cliente que quiere usar para obtener acceso al clúster. Si no tiene un certificado, siga los pasos que se indican en [Establecimiento y recuperación de un certificado](../key-vault/certificates/quick-create-portal.md) para crear un certificado autofirmado.
    * **Nombre del tipo de nodo**: Escriba un nombre único para el tipo de nodo, como *nt1*.
    * **Acepto los términos y condiciones indicados anteriormente**: active esta casilla para indicar su conformidad. 

3. Seleccione **Comprar**.

4. El clúster administrado de Service Fabric tarda unos minutos en implementarse. Espere a que la implementación se complete correctamente antes de continuar con los pasos siguientes.

## <a name="validate-the-deployment"></a>Validación de la implementación

### <a name="review-deployed-resources"></a>Revisión de los recursos implementados

Una vez finalizada la implementación, busque el valor de Service Fabric Explorer en la salida y abra la dirección en un explorador web para ver el clúster en Service Fabric Explorer. Cuando se le solicite para un certificado, use el certificado para el que se proporcionó la huella digital del cliente en la plantilla.

> [!NOTE]
> Puede encontrar la salida de la implementación en Azure Portal en la pestaña de implementaciones del grupo de recursos.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no lo necesite, elimine el grupo de recursos del clúster administrado de Service Fabric. Para eliminar el grupo de recursos mediante el portal:

1. Escriba el nombre del grupo de recursos en el cuadro de *búsqueda* de la parte superior del portal. Cuando vea el grupo de recursos que se utiliza en esta guía de inicio rápido en los resultados de búsqueda, selecciónelo.
2. Seleccione **Eliminar grupo de recursos**.
3. En el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** escriba el nombre del grupo de recursos y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha implementado un clúster administrado de Service Fabric. Para más información sobre cómo escalar un clúster, consulte:

> [!div class="nextstepaction"]
> [Escalado horizontal de un clúster administrado de Service Fabric](tutorial-managed-cluster-scale.md)
