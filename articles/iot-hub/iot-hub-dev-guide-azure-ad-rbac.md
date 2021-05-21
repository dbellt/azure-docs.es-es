---
title: Control del acceso a IoT Hub mediante Azure Active Directory | Microsoft Docs
description: 'Guía para desarrolladores: cómo controlar el acceso a IoT Hub para aplicaciones de back-end mediante Azure AD y RBAC de Azure.'
author: jlian
manager: briz
ms.author: jlian
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/21/2021
ms.custom:
- 'Role: Cloud Development'
ms.openlocfilehash: 196afc38c24254c4628173180205a858d1085eeb
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109490100"
---
# <a name="control-access-to-iot-hub-using-azure-active-directory"></a>Control del acceso a IoT Hub mediante Azure Active Directory

Azure IoT Hub admite el uso de Azure Active Directory (AAD) para autenticar las solicitudes a sus API de servicio, como crear una identidad de dispositivo o invocar un método directo. Además, IoT Hub admite la autorización de las mismas API de servicio con el control de acceso basado en rol de Azure (RBAC de Azure). Usados juntos, puede conceder permisos para acceder a las API de servicio de IoT Hub a una entidad de seguridad de AAD, que podría ser un usuario, un grupo o una entidad de servicio de aplicación.

La autenticación del acceso con Azure AD y el control de permisos con RBAC de Azure proporciona una mayor seguridad y facilidad de uso respecto a los [tokens de seguridad.](iot-hub-dev-guide-sas.md) Para minimizar posibles vulnerabilidades de seguridad inherentes a los tokens de seguridad, Microsoft recomienda usar Azure AD con IoT Hub siempre que sea posible.

> [!NOTE]
> La autenticación con Azure AD no es compatible con las API de dispositivo de *IoT Hub* (como mensajes del dispositivo a la nube y la actualización de las propiedades notificadas). Use [claves simétricas](iot-hub-dev-guide-sas.md#use-a-symmetric-key-in-the-identity-registry) o [X.509](iot-hub-x509ca-overview.md) para autenticar dispositivos en IoT Hub.

## <a name="authentication-and-authorization"></a>Autenticación y autorización

Cuando una entidad de seguridad de Azure AD solicita el acceso a una API de servicio de IoT Hub, la identidad de la entidad de seguridad primero *se autentica*. Este paso requiere que la solicitud contenga un token de acceso de OAuth 2.0 en el entorno de ejecución. El nombre del recurso para solicitar el token es `https://iothubs.azure.net`. Si la aplicación se ejecuta dentro de un recurso de Azure como una máquina virtual de Azure, una aplicación de función de Azure Functions o una aplicación de App Service, se puede representar como una [identidad administrada](../active-directory/managed-identities-azure-resources/how-managed-identities-work-vm.md). 

Una vez autenticada la entidad de Azure AD, el segundo paso es la *autorización*. En este paso, IoT Hub consulta el servicio de asignación de roles de Azure AD para comprobar que permisos tiene la entidad de seguridad. Si los permisos de la entidad de seguridad coinciden con el recurso o la API solicitados, IoT Hub autoriza la solicitud. Así, este paso exige que se asignen uno o varios roles de Azure a la entidad de seguridad. IoT Hub proporciona algunos roles integrados que tienen grupos comunes de permisos.

## <a name="manage-access-to-iot-hub-using-azure-rbac-role-assignment"></a>Administración del acceso a IoT Hub mediante la asignación de roles de RBAC de Azure

Con Azure AD y RBAC, IoT Hub requiere que la entidad de seguridad que solicita la API tenga el nivel de permiso adecuado para la autorización. Para conceder permiso a la entidad de seguridad, asígnele también una asignación de roles. 

- Si la entidad de seguridad es un usuario, grupo o entidad de servicio de aplicación, siga [Asignación de roles de Azure mediante Azure Portal](../role-based-access-control/role-assignments-portal.md).
- Si la entidad de seguridad es una identidad administrada, siga [Asignación de acceso de una identidad administrada a un recurso mediante Azure Portal](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md).

Para garantizar los privilegios mínimos, asigne siempre el rol adecuado en el [ámbito de recursos](#resource-scope) más bajo posible,que es probable que sea el ámbito de IoT Hub.

Azure proporciona los siguientes roles integrados de Azure para autorizar el acceso a la API de servicio de IoT Hub mediante Azure AD y RBAC:

| Role | Descripción | 
| ---- | ----------- | 
| [Colaborador de datos de IoT Hub](../role-based-access-control/built-in-roles.md#iot-hub-data-contributor) | Permite el acceso total a las operaciones del plano de datos de IoT Hub. |
| [Lector de datos de IoT Hub](../role-based-access-control/built-in-roles.md#iot-hub-data-reader) | Permite el acceso de lectura total a las propiedades del plano de datos de IoT Hub. |
| [Colaborador del registro de IoT Hub](../role-based-access-control/built-in-roles.md#iot-hub-registry-contributor) | Permite el acceso completo al registro de dispositivos de IoT Hub. |
| [Colaborador de gemelos de IoT Hub](../role-based-access-control/built-in-roles.md#iot-hub-twin-contributor) | Permite el acceso de lectura y escritura a todos los dispositivos y módulos gemelos de IoT Hub. |

También puede definir roles personalizados para su uso con IoT Hub mediante la combinación de los [permisos](#permissions-for-iot-hub-service-apis) que necesita. Para más información, consulte [Creación de roles personalizados para el control de acceso basado en roles de Azure](../role-based-access-control/custom-roles.md).

### <a name="resource-scope"></a>Ámbito de recursos

Antes de asignar un rol de Azure RBAC a una entidad de seguridad, determine el ámbito de acceso que debería tener la entidad de seguridad. Los procedimientos recomendados dictan que siempre es mejor conceder únicamente el ámbito más restringido posible. Los roles de Azure RBAC definidos en un ámbito más amplio los heredan los recursos que están debajo de ellos.

En la lista siguiente se describen los niveles en los que puede definir el ámbito de acceso a IoT Hub, empezando por el ámbito más restringido:

- **IoT Hub.** En este ámbito, se aplica una asignación de roles a IoT Hub. No hay ningún ámbito menor que un centro de IoT individual. No se admite la asignación de roles en ámbitos más pequeños, como la sección de la identidad del dispositivo individual o del gemelo.
- **El grupo de recursos.** En este ámbito, se aplica una asignación de roles a todos los centros de IoT del grupo de recursos.
- **La suscripción.** En este ámbito, se aplica una asignación de roles a todos los centros de IoT de todos los grupos de recursos de la suscripción.
- **Un grupo de administración.** En este ámbito, se aplica una asignación de roles a todos los centros de IoT de todos los grupos de recursos de todas las suscripciones del grupo de administración.

## <a name="permissions-for-iot-hub-service-apis"></a>Permisos para las API del servicio IoT Hub

En las tablas siguientes se describen los permisos disponibles para las API del servicio IoT Hub. Para permitir que un cliente llame a una operación determinada, asegúrese de que el rol RBAC asignado del cliente ofrezca permisos suficientes para esa operación.

| Acción de RBAC | Descripción |
|-|-|
| Microsoft.Devices/IotHubs/devices/read | Lee cualquier identidad de dispositivo o módulo. |
| Microsoft.Devices/IotHubs/devices/write  | Crea o actualiza cualquier identidad de dispositivo o módulo.  |
| Microsoft.Devices/IotHubs/devices/delete | Elimina cualquier identidad del dispositivo o módulo. |
| Microsoft.Devices/IotHubs/twins/read | Lee cualquier dispositivo o módulo gemelo. |
| Microsoft.Devices/IotHubs/twins/write | Escribe en cualquier dispositivo o módulo gemelo. |
| Microsoft.Devices/IotHubs/jobs/read | Obtiene una lista de trabajos. |
| Microsoft.Devices/IotHubs/jobs/write | Crea o actualiza cualquier trabajo. |
| Microsoft.Devices/IotHubs/jobs/delete | Elimina cualquier trabajo. |
| Microsoft.Devices/IotHubs/cloudToDeviceMessages/send/action | Envía un mensaje de la nube al dispositivo a cualquier dispositivo.  |
| Microsoft.Devices/IotHubs/cloudToDeviceMessages/feedback/action | Recibe, completa o abandona la notificación de comentarios de mensajes de la nube al dispositivo. |
| Microsoft.Devices/IotHubs/cloudToDeviceMessages/queue/purge/action | Elimina todos los comandos pendientes de un dispositivo.  |
| Microsoft.Devices/IotHubs/directMethods/invoke/action | Invoca un método directo en un dispositivo. |
| Microsoft.Devices/IotHubs/fileUpload/notifications/action  | Recibe, completa o abandona notificaciones de carga de archivos. |
| Microsoft.Devices/IotHubs/statistics/read | Lee las estadísticas de dispositivos y servicios. |
| Microsoft.Devices/IotHubs/configurations/read | Lee las configuraciones de administración de dispositivos. |
| Microsoft.Devices/IotHubs/configurations/write | Crea o actualiza las configuraciones de administración de dispositivos. |
| Microsoft.Devices/IotHubs/configurations/delete | Elimina cualquier configuración de administración de dispositivos. |
| Microsoft.Devices/IotHubs/configurations/applyToEdgeDevice/action  | Aplica el contenido de configuración a un dispositivo perimetral. |
| Microsoft.Devices/IotHubs/configurations/testQueries/action | Valida la condición de destino y las consultas de métricas personalizadas para una configuración. |

> [!TIP]
> - La operación de [actualización del Registro en masa](/rest/api/iothub/service/bulkregistry/updateregistry) requiere *los dos*: `Microsoft.Devices/IotHubs/devices/write` *y* `Microsoft.Devices/IotHubs/devices/delete`.
> - La operación de [consulta de gemelo](/rest/api/iothub/service/query/gettwins) requiere `Microsoft.Devices/IotHubs/twins/read`.
> - La operación de [obtener gemelo digital](/rest/api/iothub/service/digitaltwin/getdigitaltwin) requiere `Microsoft.Devices/IotHubs/twins/read`, mientras que la de [actualizar gemelo digital](/rest/api/iothub/service/digitaltwin/updatedigitaltwin) requiere `Microsoft.Devices/IotHubs/twins/write`.
> - Para el [comando de invocar componente](/rest/api/iothub/service/digitaltwin/invokecomponentcommand) como el [comando invocar nivel de raíz](/rest/api/iothub/service/digitaltwin/invokerootlevelcommand) requieren `Microsoft.Devices/IotHubs/directMethods/invoke/action`.

## <a name="azure-ad-access-from-azure-portal"></a>Acceso de Azure AD desde Azure Portal

Cuando intenta acceder a IoT Hub, Azure Portal primero comprueba si se le ha asignado un rol de Azure con **Microsoft.Devices/iotHubs/listkeys/action**. Si es así, Azure Portal usa las claves de las directivas de acceso compartido para acceder a IoT Hub. De lo contrario, Azure Portal trata de acceder a los datos mediante su cuenta de Azure AD. 

Para acceder a IoT Hub desde Azure Portal mediante su cuenta de Azure AD, necesita permisos para acceder a los recursos de datos de IoT Hub (como dispositivos y gemelos), así como permisos para navegar al recurso de IoT Hub en Azure Portal. Los roles integrados proporcionados por IoT Hub conceden acceso a recursos como dispositivos y gemelos, pero no conceden acceso al recurso de IoT Hub. Por lo tanto, el acceso al portal también requiere la asignación de un rol de Azure Resource Manager (ARM), como [Lector](../role-based-access-control/built-in-roles.md#reader). El rol Lector es una buena opción porque es el rol más restringido que le permite navegar por el portal y no incluye el permiso **Microsoft.Devices/iotHubs/listkeys/action** (que proporciona acceso a todos los recursos de datos de IoT Hub a través de directivas de acceso compartido). 

Para asegurarse de que una cuenta no tiene acceso fuera de los permisos asignados, *no* incluya el permiso **Microsoft.Devices/iotHubs/listkeys/action** al crear un rol personalizado. Por ejemplo, para crear un rol personalizado que pueda leer identidades de dispositivo, pero que no pueda crear ni eliminar dispositivos, cree un rol personalizado que:
- Tenga la acción de datos **Microsoft.Devices/IotHubs/devices/read**.
- No tenga la acción de datos **Microsoft.Devices/IotHubs/devices/write**.
- No tenga la acción de datos **Microsoft.Devices/IotHubs/devices/delete**.
- No tenga la acción **Microsoft.Devices/iotHubs/listkeys/action**.

A continuación, asegúrese de que la cuenta no tiene ningún otro rol que tenga el permiso **Microsoft.Devices/iotHubs/listkeys/action**, como [Propietario](../role-based-access-control/built-in-roles.md#owner) o [Colaborador](../role-based-access-control/built-in-roles.md#contributor). Para permitir que la cuenta tenga acceso a los recursos y pueda navegar por el portal, asigne [Lector](../role-based-access-control/built-in-roles.md#reader).

## <a name="built-in-event-hub-compatible-endpoint-doesnt-support-azure-ad-authentication"></a>El punto de conexión compatible con el centro de eventos integrado no admite la autenticación con Azure AD

El [punto de conexión integrado](iot-hub-devguide-messages-read-builtin.md) no admite la integración con Azure AD. No es posible acceder a él con una entidad de seguridad o una identidad administrada. Para acceder al punto de conexión integrado, use el método de la cadena de conexión (clave de acceso compartido) como antes.

## <a name="sdk-samples"></a>Ejemplos del SDK

- [Ejemplo del SDK de .NET Microsoft.Azure.Devices](https://aka.ms/iothubaadcsharpsample)
- [Ejemplo de SDK de Java](https://aka.ms/iothubaadjavasample)

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información sobre las ventajas de usar Azure AD en la aplicación, vea [Integración con Azure Active Directory](../active-directory/develop/active-directory-how-to-integrate.md).
- Para obtener más información sobre la solicitud de tokens de acceso de Azure AD para usuarios y entidades de servicio, consulte el artículo sobre [escenarios de autenticación de Azure AD](../active-directory/develop/authentication-vs-authorization.md).