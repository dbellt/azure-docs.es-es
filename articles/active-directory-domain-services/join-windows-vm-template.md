---
title: Uso de una plantilla para unir una máquina virtual con Windows a Azure AD DS | Microsoft Docs
description: Aprenda a usar plantillas de Azure Resource Manager para unir una máquina virtual con Windows Server nueva o existente a un dominio administrado de Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 4eabfd8e-5509-4acd-86b5-1318147fddb5
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 07/09/2020
ms.author: justinha
ms.openlocfilehash: fcfe5fb48a6eef0b7185fe8bba5a8f1e80fb4f1f
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112030546"
---
# <a name="join-a-windows-server-virtual-machine-to-an-azure-active-directory-domain-services-managed-domain-using-a-resource-manager-template"></a>Una una máquina virtual con Windows Server a un dominio administrado de Azure Active Directory Domain Services mediante una plantilla de Resource Manager

Para automatizar la implementación y la configuración de las máquinas virtuales (VM) de Azure, puede usar una plantilla de Resource Manager. Estas plantillas le permiten crear implementaciones coherentes cada vez. También se pueden incluir extensiones en las plantillas para configurar automáticamente una máquina virtual como parte de la implementación. Una extensión útil une máquinas virtuales a un dominio, que puede usarse con dominios administrados de Azure Active Directory Domain Services (Azure AD DS).

En este artículo se muestra cómo crear y unir una máquina virtual con Windows Server a un dominio administrado con Azure AD DS mediante plantillas de Resource Manager. También aprenderá a unir una máquina virtual con Windows Server existente a un dominio de Azure AD DS.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará los siguientes recursos y privilegios:

* Una suscripción de Azure activa.
    * Si no tiene una suscripción a Azure, [cree una cuenta](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Un inquilino de Azure Active Directory asociado a su suscripción, ya sea sincronizado con un directorio en el entorno local o con un directorio solo en la nube.
    * Si es necesario, [cree un inquilino de Azure Active Directory][create-azure-ad-tenant] o [asocie una suscripción a Azure con su cuenta][associate-azure-ad-tenant].
* Un dominio administrado de Azure Active Directory Domain Services habilitado y configurado en su inquilino de Azure AD.
    * Si es necesario, el primer tutorial [crea y configura un dominio administrado de Azure Active Directory Domain Services][create-azure-ad-ds-instance].
* Una cuenta de usuario que forme parte del dominio administrado.

## <a name="azure-resource-manager-template-overview"></a>Información general de plantillas de Azure Resource Manager

Las plantillas de Resource Manager le permiten definir la infraestructura de Azure en el código. Los recursos necesarios, las conexiones de red o la configuración de máquinas virtuales se pueden definir en una plantilla. Estas plantillas crean implementaciones coherentes y reproducibles cada vez, y pueden tener varias versiones a medida que realiza cambios. Para más información, consulte [Información general de plantillas de Azure Resource Manager][template-overview].

Cada recurso se define en una plantilla mediante la notación de objetos JavaScript (JSON). En el siguiente ejemplo de JSON se usa el tipo de recurso *Microsoft.Compute/virtualMachines/extensions* para instalar la extensión de unión a un dominio de Active Directory. Se especifican parámetros que usa en el momento de la implementación. Al implementarse la extensión, la VM se une al dominio administrado.

```json
 {
      "apiVersion": "2015-06-15",
      "type": "Microsoft.Compute/virtualMachines/extensions",
      "name": "[concat(parameters('dnsLabelPrefix'),'/joindomain')]",
      "location": "[parameters('location')]",
      "dependsOn": [
        "[concat('Microsoft.Compute/virtualMachines/', parameters('dnsLabelPrefix'))]"
      ],
      "properties": {
        "publisher": "Microsoft.Compute",
        "type": "JsonADDomainExtension",
        "typeHandlerVersion": "1.3",
        "autoUpgradeMinorVersion": true,
        "settings": {
          "Name": "[parameters('domainToJoin')]",
          "OUPath": "[parameters('ouPath')]",
          "User": "[concat(parameters('domainToJoin'), '\\', parameters('domainUsername'))]",
          "Restart": "true",
          "Options": "[parameters('domainJoinOptions')]"
        },
        "protectedSettings": {
          "Password": "[parameters('domainPassword')]"
        }
      }
    }
```

Esta extensión de máquina virtual se puede implementar incluso si no crea una máquina virtual en la misma plantilla. En los ejemplos de este artículo se muestran estos dos enfoques:

* [Creación de una máquina virtual con Windows Server y unión a un dominio administrado](#create-a-windows-server-vm-and-join-to-a-managed-domain)
* [Unión de una máquina virtual con Windows Server existente a un dominio administrado](#join-an-existing-windows-server-vm-to-a-managed-domain)

## <a name="create-a-windows-server-vm-and-join-to-a-managed-domain"></a>Creación de una máquina virtual con Windows Server y unión a un dominio administrado

Si necesita una máquina virtual con Windows Server, puede crear y configurar una mediante una plantilla de Resource Manager. Al implementarse la VM, se instala una extensión para unir la VM a un dominio administrado. Si ya tiene una VM que quiere unir a un dominio administrado, vaya a [Unión de una VM con Windows Server existente a un dominio administrado](#join-an-existing-windows-server-vm-to-a-managed-domain).

Para crear una VM con Windows Server y, a continuación, unirla a un dominio administrado, complete los siguientes pasos:

1. Vaya a la [plantilla de inicio rápido](https://azure.microsoft.com/resources/templates/vm-domain-join/). Seleccione la opción para **Implementar en Azure**.
1. En la página **Implementación personalizada**, escriba la siguiente información para crear y unir una VM con Windows Server al dominio administrado:

    | Configuración                   | Valor |
    |---------------------------|-------|
    | Subscription              | Elija la misma suscripción de Azure en la que ha habilitado Azure AD Domain Services. |
    | Resource group            | Elija el grupo de recursos para su máquina virtual. |
    | Location                  | Seleccione la ubicación de la máquina virtual. |
    | Nombre de red virtual existente        | El nombre de la red virtual existente a la que conectar la máquina virtual, como *myVnet*. |
    | Nombre de subred existente      | El nombre de la subred de red virtual existente, como *Cargas de trabajo*. |
    | Prefijo de etiqueta DNS          | Escriba un nombre DNS para usarlo para la máquina virtual, por ejemplo, *myvm*. |
    | Tamaño de VM                   | Especifique un tamaño de VM, como *Standard_DS2_v2*. |
    | Dominio al que unirse            | El nombre DNS de dominio administrado como, por ejemplo, *aaddscontoso.com*. |
    | Nombre de usuario de dominio           | La cuenta de usuario del dominio administrado que debe usarse para unir la VM al dominio administrado, como `contosoadmin@aaddscontoso.com`. Esta cuenta debe formar parte del dominio administrado. |
    | Contraseña de dominio           | La contraseña de la cuenta de usuario especificada en la configuración anterior. |
    | Ruta de acceso de unidad organizativa opcional          | La unidad organizativa personalizada en la que agregar la máquina virtual. Si no especifica un valor para este parámetro, la máquina virtual se agrega a la unidad organizativa *Equipos de DC de AAD* predeterminada. |
    | Nombre de usuario administrador de máquina virtual         | Especifique una cuenta de administrador local para su creación en la máquina virtual. |
    | Contraseña de administrador de máquina virtual         | Especifique una contraseña de administrador local para la máquina virtual. Cree una contraseña de administrador local segura a fin de ofrecer protección frente a ataques de fuerza bruta a contraseñas. |

1. Revise los términos y condiciones, y active la casilla **Acepto los términos y condiciones indicados anteriormente**. Cuando esté listo, seleccione **Comprar** para crear y unir la VM al dominio administrado.

> [!WARNING]
> **Controle las contraseñas con precaución.**
> El archivo de parámetros de plantilla solicita la contraseña de una cuenta de usuario que forme parte del dominio administrado. No especifique valores manualmente en este archivo y déjelo accesible en recursos compartidos de archivos u otras ubicaciones compartidas.

La implementación tarda unos minutos en finalizar correctamente. Una vez finalizada, la VM Windows se crea y une al dominio administrado. La máquina virtual se puede administrar o se puede iniciar sesión en ella mediante cuentas de dominio.

## <a name="join-an-existing-windows-server-vm-to-a-managed-domain"></a>unión de una máquina virtual con Windows Server existente a un dominio administrado

Si tiene una VM existente o un grupo de VM que quiere unir a un dominio administrado, puede usar una plantilla de Resource Manager para implementar simplemente la extensión de VM.

Para unir una VM con Windows Server existente a un dominio administrado, complete los siguientes pasos:

1. Vaya a la [plantilla de inicio rápido](https://azure.microsoft.com/resources/templates/vm-domain-join-existing/). Seleccione la opción para **Implementar en Azure**.
1. En la página **Implementación personalizada**, escriba la siguiente información para unir la VM al dominio administrado:

    | Configuración                   | Valor |
    |---------------------------|-------|
    | Subscription              | Elija la misma suscripción de Azure en la que ha habilitado Azure AD Domain Services. |
    | Resource group            | Elija el grupo de recursos con su máquina virtual existente. |
    | Location                  | Seleccione la ubicación de su máquina virtual existente. |
    | Lista de máquinas virtuales                   | Escriba la lista separada por comas de las VM existentes que unir al dominio administrado, como *myVM1,myVM2*. |
    | Nombre de usuario de unión a un dominio     | La cuenta de usuario del dominio administrado que debe usarse para unir la VM al dominio administrado, como `contosoadmin@aaddscontoso.com`. Esta cuenta debe formar parte del dominio administrado. |
    | Contraseña de usuario de unión a un dominio | La contraseña de la cuenta de usuario especificada en la configuración anterior. |
    | Ruta de acceso de unidad organizativa opcional          | La unidad organizativa personalizada en la que agregar la máquina virtual. Si no especifica un valor para este parámetro, la máquina virtual se agrega a la unidad organizativa *Equipos de DC de AAD* predeterminada. |

1. Revise los términos y condiciones, y active la casilla **Acepto los términos y condiciones indicados anteriormente**. Cuando esté listo, seleccione **Comprar** para unir la VM al dominio administrado.

> [!WARNING]
> **Controle las contraseñas con precaución.**
> El archivo de parámetros de plantilla solicita la contraseña de una cuenta de usuario que forme parte del dominio administrado. No especifique valores manualmente en este archivo y déjelo accesible en recursos compartidos de archivos u otras ubicaciones compartidas.

La implementación tarda unos minutos en finalizar correctamente. Una vez finalizada, las VM Windows especificadas se unen al dominio administrado y se pueden administrar o se puede iniciar sesión en ellas mediante cuentas de dominio.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, usó Azure Portal para configurar e implementar recursos mediante plantillas. También puede implementar recursos con plantillas de Resource Manager mediante [Azure PowerShell][deploy-powershell] o la [CLI de Azure][deploy-cli].

<!-- INTERNAL LINKS -->
[create-azure-ad-tenant]: ../active-directory/fundamentals/sign-up-organization.md
[associate-azure-ad-tenant]: ../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md
[create-azure-ad-ds-instance]: tutorial-create-instance.md
[template-overview]: ../azure-resource-manager/templates/overview.md
[deploy-powershell]: ../azure-resource-manager/templates/deploy-powershell.md
[deploy-cli]: ../azure-resource-manager/templates/deploy-cli.md
