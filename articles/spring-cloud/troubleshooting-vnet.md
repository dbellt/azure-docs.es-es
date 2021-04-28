---
title: Solución de problemas de Azure Spring Cloud en la red virtual
description: Guía para la solución de problemas de Azure Spring Cloud en la red virtual.
author: mikedodaro
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/19/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 6a8be63c134c25373f85a146ed3df7316e1eb7dc
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108132412"
---
# <a name="troubleshooting-azure-spring-cloud-in-virtual-networks"></a>Solución de problemas de Azure Spring Cloud en redes virtuales

Este documento le ayudará a resolver diversos problemas que pueden surgir al usar Azure Spring Cloud en redes virtuales.

## <a name="i-encountered-a-problem-with-creating-an-azure-spring-cloud-service-instance"></a>Ha aparecido un problema al crear una instancia de servicio de Azure Spring Cloud

Para crear una instancia de Azure Spring Cloud, debe tener permisos suficientes para implementar la instancia en la red virtual.  La instancia de servicio de Spring Cloud debe [concederse a sí misma el permiso de servicio de Azure Spring Cloud para la red virtual](./how-to-deploy-in-azure-virtual-network.md#grant-service-permission-to-the-virtual-network).

Si usa Azure Portal para configurar la instancia de servicio en Azure Spring Cloud, Azure Portal validará los permisos.

Para configurar la instancia de servicio de Azure Spring Cloud mediante la [CLI de Azure](/cli/azure/get-started-with-azure-cli), compruebe lo siguiente:

- Que la suscripción está activa.
- Que Azure Spring Cloud admite la ubicación.
- Que se ha creado el grupo de recursos de la instancia.
- Que el nombre del recurso se ajusta a la regla de nomenclatura Solo debe contener minúsculas, números y guiones. El primer carácter debe ser una letra. El último carácter debe ser una letra o un número. El valor tiene que contener entre 2 y 32 caracteres.

Para configurar la instancia de servicio de Azure Spring Cloud mediante la plantilla de Resource Manager, consulte [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md).

### <a name="common-creation-issues"></a>Problemas habituales de creación

| Mensaje de error | Solución |
|------|------|
| Resources created by Azure Spring Cloud were disallowed by policy (La directiva no permitió los recursos creados por Azure Spring Cloud). | Los recursos de red se crearán al implementar Azure Spring Cloud en su propia red virtual. Compruebe si se ha definido [Azure Policy](../governance/policy/overview.md) para bloquear esa creación. Los recursos que no se pudieron crear se encuentran en el mensaje de error. |
| Required traffic is not allowlisted (El tráfico necesario no está en la lista de permitidos). | Vea [Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual](./vnet-customer-responsibilities.md) para asegurarse de que el tráfico necesario esté en la lista de permitidos. |

## <a name="my-application-cant-be-registered"></a>No se puede registrar mi aplicación

Este problema se produce si la red virtual está configurada con la configuración de DNS personalizada. En este caso, la zona DNS privada que usa Azure Spring Cloud es ineficaz. Agregue la IP 168.63.129.16 de Azure DNS como servidor DNS ascendente en el servidor DNS personalizado.

## <a name="other-issues"></a>Otros problemas

[Solución de problemas comunes de Azure Spring Cloud](./troubleshoot.md).