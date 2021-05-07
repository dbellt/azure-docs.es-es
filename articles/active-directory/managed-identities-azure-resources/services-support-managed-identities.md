---
title: 'Servicios de Azure que admiten identidades administradas: Azure AD'
description: Lista de servicios que admiten identidades administradas para recursos de Azure y autenticación de Azure AD
services: active-directory
author: barclayn
ms.author: barclayn
ms.date: 01/28/2021
ms.topic: conceptual
ms.service: active-directory
ms.subservice: msi
manager: daveba
ms.collection: M365-identity-device-management
ms.custom: references_regions
ms.openlocfilehash: 96c534919e7279af61a86d24ae4c4ca984c02662
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108124242"
---
# <a name="services-that-support-managed-identities-for-azure-resources"></a>Servicios que admiten identidades administradas para recursos de Azure

Las identidades administradas para los recursos de Azure proporcionan a los servicios de Azure una identidad administrada automáticamente en Azure Active Directory. Mediante una identidad administrada puede autenticarse en cualquier servicio que admita la autenticación de Azure AD, sin necesidad de tener credenciales en el código. Estamos en el proceso de integración de identidades administradas para recursos de Azure y de la autenticación de Azure AD en Azure. Compruebe con frecuencia si existen actualizaciones.

> [!NOTE]
> Identidades administradas para recursos de Azure es el nombre con el que ahora se conoce al servicio Managed Service Identity (MSI).

## <a name="azure-services-that-support-managed-identities-for-azure-resources"></a>Servicios de Azure que admiten identidades administradas de recursos de Azure

Los siguientes servicios de Azure admiten identidades administradas para recursos de Azure:


### <a name="azure-api-management"></a>Azure API Management

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | Vista previa | Vista previa | No disponible | Vista previa |

Consulte la lista siguiente para configurar la identidad administrada para Azure API Management (en las regiones donde esté disponible):

- [Plantilla de Azure Resource Manager](../../api-management/api-management-howto-use-managed-service-identity.md)

### <a name="azure-app-configuration"></a>Azure App Configuration

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check]  | No disponible  | ![Disponible][check] |

Consulte la lista siguiente para configurar la identidad administrada para Azure App Configuration (en las regiones donde esté disponible):

- [CLI de Azure](../../azure-app-configuration/overview-managed-identity.md)

### <a name="azure-app-service"></a>Azure App Service

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check]  | ![Disponible][check]  | ![Disponible][check] |

Consulte la lista siguiente para configurar la identidad administrada para Azure App Service (en las regiones donde esté disponible):

- [Azure Portal](../../app-service/overview-managed-identity.md#using-the-azure-portal)
- [CLI de Azure](../../app-service/overview-managed-identity.md#using-the-azure-cli)
- [Azure PowerShell](../../app-service/overview-managed-identity.md#using-azure-powershell)
- [Plantilla de Azure Resource Manager](../../app-service/overview-managed-identity.md#using-an-azure-resource-manager-template)

### <a name="azure-arc-enabled-kubernetes"></a>Kubernetes habilitado para Azure Arc

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Versión preliminar | No disponible | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Kubernetes habilitado para Azure Arc [admite actualmente la identidad asignada por el sistema](../../azure-arc/kubernetes/quickstart-connect-cluster.md). El certificado de identidad de servicio administrado se usa en todos los agentes Kubernetes habilitados para Azure Arc para la comunicación con Azure.

### <a name="azure-arc-enabled-servers"></a>Servidores habilitados para Azure Arc

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Todos los servidores habilitados para Azure Arc tienen una identidad asignada por el sistema. No se puede deshabilitar ni cambiar la identidad asignada por el sistema en un servidor habilitado para Azure Arc. Consulte los siguientes recursos para obtener más información sobre cómo consumir identidades administradas en servidores habilitados para Azure Arc:

- [Autenticación en recursos de Azure con servidores habilitados para Arc](../../azure-arc/servers/managed-identity-authentication.md)
- [Uso de una identidad administrada con servidores habilitados para Arc](../../azure-arc/servers/security-overview.md#using-a-managed-identity-with-arc-enabled-servers)

### <a name="azure-automanage"></a>Azure Automanage

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Versión preliminar | No disponible | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte el documento siguiente para volver a configurar una identidad administrada si ha migrado la suscripción a un nuevo inquilino:

* [Reparación de una cuenta de Automanage estropeada](../../automanage/repair-automanage-account.md)

### <a name="azure-automation"></a>Azure Automation

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte los siguientes documentos para usar una identidad administrada con [Azure Automation](../../automation/automation-intro.md):

* [Introducción a la autenticación de cuentas de Automation: identidades administradas](../../automation/automation-security-overview.md#managed-identities-preview)
* [Habilitación y uso de la identidad administrada para Automation](../../automation/enable-managed-identity-for-automation.md)

### <a name="azure-blueprints"></a>Azure Blueprint

|Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | No disponible |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check] | No disponible | No disponible |

Consulte la siguiente lista para usar una identidad administrada con [Azure Blueprints](../../governance/blueprints/overview.md):

- [Azure Portal: asignación de plano técnico](../../governance/blueprints/create-blueprint-portal.md#assign-a-blueprint)
- [API de REST: asignación de plano técnico](../../governance/blueprints/create-blueprint-rest-api.md#assign-a-blueprint)


### <a name="azure-cognitive-search"></a>Azure Cognitive Search

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

### <a name="azure-cognitive-services"></a>Azure Cognitive Services

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |


### <a name="azure-communication-services"></a>Azure Communication Services

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | No disponible | No disponible | No disponible |
| Asignado por el usuario | ![Disponible][check] | No disponible | No disponible | No disponible |


### <a name="azure-container-instances"></a>Azure Container Instances

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Linux: Versión preliminar<br>Windows: No disponible | No disponible | No disponible | No disponible |
| Asignado por el usuario | Linux: Versión preliminar<br>Windows: No disponible | No disponible | No disponible | No disponible |

Consulte la lista siguiente para configurar la identidad administrada para Azure Container Instances (en las regiones donde esté disponible):

- [CLI de Azure](~/articles/container-instances/container-instances-managed-identity.md)
- [Plantilla de Azure Resource Manager](~/articles/container-instances/container-instances-managed-identity.md#enable-managed-identity-using-resource-manager-template)
- [YAML](~/articles/container-instances/container-instances-managed-identity.md#enable-managed-identity-using-yaml-file)


### <a name="azure-container-registry-tasks"></a>Tareas de Azure Container Registry

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | No disponible | No disponible | No disponible |
| Asignado por el usuario | Vista previa | No disponible | No disponible | No disponible |

Consulte la lista siguiente para configurar la identidad administrada para Azure Container Registry Tasks (en las regiones donde esté disponible):

- [CLI de Azure](~/articles/container-registry/container-registry-tasks-authentication-managed-identity.md)

### <a name="azure-data-explorer"></a>Explorador de datos de Azure

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

### <a name="azure-data-factory-v2"></a>Azure Data Factory V2

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte la lista siguiente para configurar la identidad administrada para Azure Data Factory V2 (en las regiones donde esté disponible):

- [Azure Portal](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity)
- [PowerShell](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity-using-powershell)
- [REST](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity-using-rest-api)
- [SDK](~/articles/data-factory/data-factory-service-identity.md#generate-managed-identity-using-sdk)

### <a name="azure-digital-twins"></a>Azure Digital Twins

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | No disponible | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte la lista siguiente para configurar la identidad administrada para Azure Digital Twins (en las regiones donde esté disponible):

- [Azure Portal](../../digital-twins/how-to-enable-managed-identities-portal.md)

### <a name="azure-event-grid"></a>Azure Event Grid

Tipo de identidad administrada |Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Versión preliminar | Vista previa | No disponible | Vista previa |
| Asignado por el usuario | No disponible | No disponible  | No disponible  | No disponible |

### <a name="azure-firewall-policy"></a>Directiva de Azure Firewall

Tipo de identidad administrada |Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | No disponible | No disponible | No disponible | No disponible |
| Asignado por el usuario | Vista previa | No disponible  | No disponible  | No disponible |

### <a name="azure-functions"></a>Azure Functions

Tipo de identidad administrada |Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check]  | ![Disponible][check]  | ![Disponible][check]  |

Consulte la lista siguiente para configurar la identidad administrada para Azure Functions (en las regiones donde esté disponible):

- [Azure Portal](../../app-service/overview-managed-identity.md#using-the-azure-portal)
- [CLI de Azure](../../app-service/overview-managed-identity.md#using-the-azure-cli)
- [Azure PowerShell](../../app-service/overview-managed-identity.md#using-azure-powershell)
- [Plantilla de Azure Resource Manager](../../app-service/overview-managed-identity.md#using-an-azure-resource-manager-template)

### <a name="azure-iot-hub"></a>Azure IoT Hub

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte la lista siguiente para configurar la identidad administrada para Azure IoT Hub (en las regiones donde esté disponible):

- [Azure Portal](../../iot-hub/virtual-network-support.md#turn-on-managed-identity-for-iot-hub)

### <a name="azure-importexport"></a>Azure Import/Export

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | --- | --- | --- | --- |
| Asignado por el sistema | Disponible en la región donde está disponible el servicio Azure Import/Export | Versión preliminar | Disponible | Disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

### <a name="azure-kubernetes-service-aks"></a>Azure Kubernetes Service (AKS)

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | No disponible |
| Asignado por el usuario | Vista previa | No disponible | No disponible | No disponible |


Para obtener más información, consulte [Uso de identidades administradas en Azure Kubernetes Service](../../aks/use-managed-identity.md).

### <a name="azure-log-analytics-cluster"></a>Clúster de Azure Log Analytics

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |

Para más información, consulte [cómo funciona la identidad en Azure Monitor](../../azure-monitor/logs/customer-managed-keys.md).

### <a name="azure-logic-apps"></a>Azure Logic Apps

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check] | No disponible | ![Disponible][check] |


Consulte la lista siguiente para configurar la identidad administrada para Azure Logic Apps (en las regiones donde esté disponible):

- [Azure Portal](../../logic-apps/create-managed-service-identity.md#enable-system-assigned-identity-in-azure-portal)
- [Plantilla de Azure Resource Manager](../../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)

### <a name="azure-machine-learning"></a>Azure Machine Learning

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Versión preliminar | No disponible | No disponible | No disponible |
| Asignado por el usuario | Vista previa | No disponible | No disponible | No disponible |

Para más información, consulte [Uso de identidades administradas con Azure Machine Learning](../../machine-learning/how-to-use-managed-identities.md).

### <a name="azure-policy"></a>Azure Policy

|Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte la lista siguiente para configurar la identidad administrada para Azure Policy (en las regiones donde esté disponible):

- [Azure Portal](../../governance/policy/tutorials/create-and-manage.md#assign-a-policy)
- [PowerShell](../../governance/policy/how-to/remediate-resources.md#create-managed-identity-with-powershell)
- [CLI de Azure](/cli/azure/policy/assignment#az_policy_assignment_create)
- [Plantillas del Administrador de recursos de Azure](/azure/templates/microsoft.authorization/policyassignments)
- [REST](/rest/api/policy/policyassignments/create)


### <a name="azure-service-fabric"></a>Azure Service Fabric

La [identidad administrada para aplicaciones de Service Fabric](../../service-fabric/concepts-managed-identity.md) está disponible en todas las regiones.

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | No disponible | No disponible | No disponible |
| Asignado por el usuario | ![Disponible][check] | No disponible | No disponible |No disponible |

Consulte la siguiente lista para configurar la identidad administrada para las aplicaciones de Azure Service Fabric en todas las regiones:

- [Plantilla de Azure Resource Manager](https://github.com/Azure-Samples/service-fabric-managed-identity/tree/anmenard-docs)

### <a name="azure-spring-cloud"></a>Azure Spring Cloud

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | No disponible | No disponible | ![Disponible][check] |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |


Para más información, consulte [Habilitación de la identidad administrada asignada por el sistema para una aplicación de Azure Spring Cloud](../../spring-cloud/how-to-enable-system-assigned-managed-identity.md).

### <a name="azure-stack-edge"></a>Azure Stack Edge

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | --- | --- | --- | --- |
| Asignado por el sistema | Disponible en la región en la que está disponible el servicio Azure Stack Edge | No disponible | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

### <a name="azure-virtual-machine-scale-sets"></a>Conjuntos de escalado de máquinas virtuales de Azure

|Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |

Consulte la lista siguiente para configurar la identidad administrada para Azure Virtual Machine Scale Sets (en las regiones donde esté disponible):

- [Azure Portal](qs-configure-portal-windows-vm.md)
- [PowerShell](qs-configure-powershell-windows-vm.md)
- [CLI de Azure](qs-configure-cli-windows-vm.md)
- [Plantillas del Administrador de recursos de Azure](qs-configure-template-windows-vm.md)
- [REST](qs-configure-rest-vm.md)



### <a name="azure-virtual-machines"></a>Azure Virtual Machines

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |
| Asignado por el usuario | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] | ![Disponible][check] |

Consulte la lista siguiente para configurar la identidad administrada para Azure Virtual Machines (en las regiones donde esté disponible):

- [Azure Portal](qs-configure-portal-windows-vm.md)
- [PowerShell](qs-configure-powershell-windows-vm.md)
- [CLI de Azure](qs-configure-cli-windows-vm.md)
- [Plantillas del Administrador de recursos de Azure](qs-configure-template-windows-vm.md)
- [REST](qs-configure-rest-vm.md)
- [SDK de Azure](qs-configure-sdk-windows-vm.md)


### <a name="azure-vm-image-builder"></a>Azure VM Image Builder

| Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | No disponible | No disponible | No disponible | No disponible |
| Asignado por el usuario | [Disponible en regiones admitidas](../../virtual-machines/image-builder-overview.md#regions) | No disponible | No disponible | No disponible |

Para más información sobre cómo configurar la identidad administrada para Azure VM Image Builder (en las regiones donde esté disponible), consulte la [introducción a Image Builder](../../virtual-machines/image-builder-overview.md#permissions).
### <a name="azure-signalr-service"></a>Servicio Azure SignalR

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Versión preliminar | Vista previa | No disponible | Vista previa |
| Asignado por el usuario | Vista previa | Vista previa | No disponible | Versión preliminar |

Consulte la lista siguiente para configurar la identidad administrada para Azure SignalR Service (en las regiones donde esté disponible):

- [Plantilla de Azure Resource Manager](../../azure-signalr/howto-use-managed-identity.md)

### <a name="azure-resource-mover"></a>Azure Resource Mover

Tipo de identidad administrada | Regiones globales de Azure<br>con disponibilidad general | Azure Government | Azure Alemania | Azure China 21Vianet |
| --- | :-: | :-: | :-: | :-: |
| Asignado por el sistema | Disponible en las regiones en las que está disponible el servicio Azure Resource Mover | No disponible | No disponible | No disponible |
| Asignado por el usuario | No disponible | No disponible | No disponible | No disponible |

Consulte el siguiente documento para usar Azure Resource Mover:

- [Azure Resource Mover](../../resource-mover/overview.md)

## <a name="azure-services-that-support-azure-ad-authentication"></a>Servicios de Azure que admiten la autenticación de Azure AD

Los siguientes servicios admiten la autenticación de Azure AD y se han probado con los servicios de cliente que usan identidades administradas para recursos de Azure.

### <a name="azure-resource-manager"></a>Azure Resource Manager

Consulte la siguiente lista para configurar el acceso a Azure Resource Manager:

- [Asignar el acceso a través de Azure Portal](howto-assign-access-portal.md)
- [Asignar el acceso a través de PowerShell](howto-assign-access-powershell.md)
- [Asignar el acceso a través de la CLI de Azure](howto-assign-access-CLI.md)
- [Asignar el acceso a través de una plantilla de Azure Resource Manager](../../role-based-access-control/role-assignments-template.md)

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://management.azure.com/`| ![Disponible][check] |
| Azure Government | `https://management.usgovcloudapi.net/` | ![Disponible][check] |
| Azure Alemania | `https://management.microsoftazure.de/` | ![Disponible][check] |
| Azure China 21Vianet | `https://management.chinacloudapi.cn` | ![Disponible][check] |

### <a name="azure-key-vault"></a>Azure Key Vault

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://vault.azure.net`| ![Disponible][check] |
| Azure Government | `https://vault.usgovcloudapi.net` | ![Disponible][check] |
| Azure Alemania |  `https://vault.microsoftazure.de` | ![Disponible][check] |
| Azure China 21Vianet | `https://vault.azure.cn` | ![Disponible][check] |

### <a name="azure-data-lake"></a>Azure Data Lake

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://datalake.azure.net/` | ![Disponible][check] |
| Azure Government |  | No disponible |
| Azure Alemania |   | No disponible |
| Azure China 21Vianet |  | No disponible |

### <a name="azure-sql"></a>Azure SQL

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://database.windows.net/` | ![Disponible][check] |
| Azure Government | `https://database.usgovcloudapi.net/` | ![Disponible][check] |
| Azure Alemania | `https://database.cloudapi.de/` | ![Disponible][check] |
| Azure China 21Vianet | `https://database.chinacloudapi.cn/` | ![Disponible][check] |

### <a name="azure-data-explorer"></a>Explorador de datos de Azure

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://<account>.<region>.kusto.windows.net` | ![Disponible][check] |
| Azure Government | `https://<account>.<region>.kusto.usgovcloudapi.net` | ![Disponible][check] |
| Azure Alemania | `https://<account>.<region>.kusto.cloudapi.de` | ![Disponible][check] |
| Azure China 21Vianet | `https://<account>.<region>.kusto.chinacloudapi.cn` | ![Disponible][check] |

### <a name="azure-event-hubs"></a>Azure Event Hubs

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://eventhubs.azure.net` | ![Disponible][check] |
| Azure Government |  | No disponible |
| Azure Alemania |   | No disponible |
| Azure China 21Vianet |  | No disponible |

### <a name="azure-service-bus"></a>Azure Service Bus

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://servicebus.azure.net`  | ![Disponible][check] |
| Azure Government |  | ![Disponible][check] |
| Azure Alemania |   | No disponible |
| Azure China 21Vianet |  | No disponible |









### <a name="azure-storage-blobs-and-queues"></a>Colas y blobs de Azure Storage

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://storage.azure.com/` <br /><br />`https://<account>.blob.core.windows.net` <br /><br />`https://<account>.queue.core.windows.net` | ![Disponible][check] |
| Azure Government | `https://storage.azure.com/`<br /><br />`https://<account>.blob.core.usgovcloudapi.net` <br /><br />`https://<account>.queue.core.usgovcloudapi.net` | ![Disponible][check] |
| Azure Alemania | `https://storage.azure.com/`<br /><br />`https://<account>.blob.core.cloudapi.de` <br /><br />`https://<account>.queue.core.cloudapi.de` | ![Disponible][check] |
| Azure China 21Vianet | `https://storage.azure.com/`<br /><br />`https://<account>.blob.core.chinacloudapi.cn` <br /><br />`https://<account>.queue.core.chinacloudapi.cn` | ![Disponible][check] |

### <a name="azure-analysis-services"></a>Azure Analysis Services

| Nube | Id. de recurso | Status |
|--------|------------|:-:|
| Azure Global | `https://*.asazure.windows.net` | ![Disponible][check] |
| Azure Government | `https://*.asazure.usgovcloudapi.net` | ![Disponible][check] |
| Azure Alemania | `https://*.asazure.cloudapi.de` | ![Disponible][check] |
| Azure China 21Vianet | `https://*.asazure.chinacloudapi.cn` | ![Disponible][check] |

> [!Note]
> Microsoft Power BI también [admite identidades administradas](../../stream-analytics/powerbi-output-managed-identity.md).


[check]: media/services-support-managed-identities/check.png "Disponible"