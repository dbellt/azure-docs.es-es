---
title: 'Preguntas más frecuentes sobre los aceleradores de soluciones de IoT: Azure | Microsoft Docs'
description: Este artículo responde a las preguntas más frecuentes sobre los aceleradores de soluciones de IoT. Incluye vínculos a los repositorios de GitHub.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 02/15/2018
ms.author: dobett
ms.openlocfilehash: 61993df77b0831926f16339a741a2553e80c2a0d
ms.sourcegitcommit: 3ed0f0b1b66a741399dc59df2285546c66d1df38
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107713847"
---
# <a name="frequently-asked-questions-for-iot-solution-accelerators"></a>Preguntas más frecuentes acerca de los aceleradores de soluciones de IoT

Consulte también la sección [Preguntas frecuentes específicas de la factoría conectada](iot-accelerators-faq-cf.md).

### <a name="where-can-i-find-the-source-code-for-the-solution-accelerators"></a>¿Dónde se puede encontrar el código fuente del acelerador de la solución?

El código fuente se almacena en los siguientes repositorios de GitHub:

* [Acelerador de la solución de factoría conectada](https://github.com/Azure/azure-iot-connected-factory)
* [Acelerador de soluciones de simulación de dispositivos](https://github.com/Azure/azure-iot-pcs-device-simulation)

### <a name="where-can-i-find-the-remote-monitoring-and-predictive-maintenance-solution-accelerators"></a>¿Dónde puedo encontrar los aceleradores de soluciones de supervisión remota y mantenimiento predictivo?

A partir del 10 de diciembre de 2020, los aceleradores de supervisión remota y mantenimiento predictivo se han quitado del sitio de [aceleradores de soluciones de Azure IoT](https://www.azureiotsolutions.com/Accelerators) y ya no están disponibles para nuevas implementaciones. Los repositorios de GitHub de ambos aceleradores se han archivado. El código sigue estando disponible para todo aquel que quiera acceder a él, pero ya no se aceptan nuevas contribuciones a los repositorios.

### <a name="what-happens-to-my-existing-remote-monitoring-and-predictive-maintenance-deployments"></a>¿Qué ocurre con mis implementaciones existentes de supervisión remota y mantenimiento predictivo?

Las implementaciones existentes no se verán afectadas por la eliminación de los aceleradores de soluciones de supervisión remota y mantenimiento predictivo y seguirán funcionando. Los repositorios bifurcados tampoco se verán afectados. Los repositorios maestros de GitHub se han archivado.

### <a name="how-do-i-deploy-device-simulation-solution-accelerator"></a>¿Cómo implemento el acelerador de soluciones de simulación de dispositivos?

Para implementar el acelerador de soluciones de simulación de dispositivos, consulte el repositorio de [simulación de dispositivos](https://github.com/Azure/azure-iot-pcs-device-simulation/blob/master/README.md) de GitHub.

### <a name="where-can-i-find-information-about-the-removed-solution-accelerators"></a>¿Dónde puedo encontrar información sobre los aceleradores de soluciones quitados?

Consulte las siguientes páginas en el sitio de versiones anterior:

* [Supervisión remota](/previous-versions/azure/iot-accelerators/about-iot-accelerators)
* [Mantenimiento predictivo](/previous-versions/azure/iot-accelerators/about-iot-accelerators)

### <a name="what-sdks-can-i-use-to-develop-device-clients-for-the-solution-accelerators"></a>¿Qué SDK puedo usar para desarrollar clientes de dispositivo para los aceleradores de soluciones?

En los repositorios de GitHub de los [SDK de IoT de Microsoft Azure](https://github.com/Azure/azure-iot-sdks) encontrará vínculos a los SDK de dispositivo IoT de los distintos lenguajes (C, .NET, Java, Node.js, Python).

Si utiliza el dispositivo DevKit, encontrará recursos y ejemplos en el repositorio de GitHub del [SDK de IoT DevKit](https://github.com/Microsoft/devkit-sdk).

### <a name="im-a-service-administrator-and-id-like-to-change-the-directory-mapping-between-my-subscription-and-a-specific-azure-ad-tenant-how-do-i-complete-this-task"></a>Soy administrador de servicios y quiero cambiar la asignación de directorios entre mi suscripción y un inquilino de Azure AD específico. ¿Cómo completo esta tarea?

Vea [Adición de una suscripción existente al directorio de Azure AD](../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md#to-associate-an-existing-subscription-to-your-azure-ad-directory).

### <a name="i-want-to-change-a-service-administrator-or-co-administrator-when-logged-in-with-an-organizational-account"></a>Quiero cambiar un administrador de servicios o coadministrador cuando haya iniciado sesión con una cuenta de la organización.

Consulte el artículo de asistencia [Changing Service Administrator and Co-Administrator when logged in with an organizational account](https://azure.microsoft.com/support/changing-service-admin-and-co-admin) (Cambiar el administrador de servicios y el coadministrador cuando ha iniciado sesión con una cuenta organizativa).

### <a name="why-am-i-seeing-this-error-your-account-does-not-have-the-proper-permissions-to-create-a-solution-please-check-with-your-account-administrator-or-try-with-a-different-account"></a>¿Por qué veo este error? "Su cuenta no tiene los permisos adecuados para crear una solución. Póngase en contacto con el administrador de cuentas o inténtelo con otra cuenta."

Observe el diagrama siguiente para obtener instrucciones:

![Diagrama de flujo de permisos](media/iot-accelerators-faq/flowchart.png)

> [!NOTE]
> Si sigue viendo el error después de validar que usted es el administrador global del inquilino de Azure AD y el coadministrador de la suscripción, haga que el administrador de cuenta quite el usuario y vuelva a asignar los permisos necesarios en este orden. En primer lugar, agregue al usuario como administrador global y luego agregue a un usuario como coadministrador de la suscripción de Azure. Si los problemas persisten, póngase en contacto con [Ayuda y soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade).

### <a name="why-am-i-seeing-this-error-when-i-have-an-azure-subscription-an-azure-subscription-is-required-to-create-pre-configured-solutions-you-can-create-a-free-trial-account-in-just-a-couple-of-minutes"></a>¿Por qué veo este error cuando tengo una suscripción de Azure? "Para crear soluciones preconfiguradas se requiere una suscripción de Azure. Puede crear una cuenta de evaluación gratuita en pocos minutos".

Si está seguro de que tiene una suscripción a Azure, valide la asignación del inquilino de la suscripción y compruebe si se ha seleccionado el inquilino correcto en la lista desplegable. Si ha validado que el inquilino es correcto, siga el diagrama anterior y valide la asignación de la suscripción y de este inquilino de Azure AD.

### <a name="whats-the-difference-between-deleting-a-resource-group-in-the-azure-portal-and-clicking-delete-on-a-solution-accelerator-in-azureiotsolutionscom"></a>¿Cuál es la diferencia entre eliminar un grupo de recursos en Azure Portal y hacer clic en Eliminar en un acelerador de soluciones en azureiotsolutions.com?

* Si elimina el acelerador de soluciones en [azureiotsolutions.com](https://www.azureiotsolutions.com/), elimina todos los recursos implementados cuando se creó el acelerador de soluciones. Si agrega recursos adicionales al grupo de recursos, dichos recursos también se eliminan.
* Si elimina el grupo de recursos en [Azure Portal](https://portal.azure.com), solo se eliminan los recursos de ese grupo de recursos. También debe eliminar la aplicación de Azure Active Directory asociada al acelerador de la solución.

### <a name="can-i-continue-to-leverage-my-existing-investments-in-azure-iot-solution-accelerators"></a>¿Puedo seguir aprovechando mi inversión en aceleradores de soluciones de Azure IoT?

Sí. Cualquier solución que exista en la actualidad sigue funcionando en la suscripción a Azure y el código fuente sigue estando disponible en GitHub.

### <a name="how-many-iot-hub-instances-can-i-provision-in-a-subscription"></a>¿Cuántas instancias de IoT Hub se pueden aprovisionar en una suscripción?

De forma predeterminada, puede aprovisionar [diez centros de IoT Hub por suscripción](../azure-resource-manager/management/azure-subscription-service-limits.md#iot-hub-limits). Puede crear un [vale de soporte de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para aumentar este límite. Como consecuencia, dado que cada acelerador de soluciones aprovisiona una nueva instancia de IoT Hub, solo puede aprovisionar hasta diez aceleradores de soluciones en una suscripción determinada.

### <a name="how-many-azure-cosmos-db-instances-can-i-provision-in-a-subscription"></a>¿Cuántas instancias de Azure Cosmos DB se pueden aprovisionar en una suscripción?

Cincuenta. Puede crear un [vale de soporte de Azure](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade) para aumentar este límite, pero de forma predeterminada solo se pueden aprovisionar 50 instancias de Cosmos DB por suscripción.

### <a name="can-i-create-a-solution-accelerator-if-i-have-microsoft-azure-for-dreamspark"></a>¿Puedo crear un acelerador de soluciones si tengo Microsoft Azure para DreamSpark?

> [!NOTE]
> Microsoft Azure para DreamSpark ahora se conoce como Microsoft Imagine para estudiantes.

En este momento, no se puede crear un acelerador de soluciones con una cuenta de [Microsoft Azure para DreamSpark](https://azure.microsoft.com/pricing/member-offers/imagine/). Sin embargo, puede crear una [cuenta de evaluación gratuita de Azure](https://azure.microsoft.com/free/) en un par de minutos, lo que le permite crear un acelerador de soluciones.

### <a name="how-do-i-delete-an-azure-ad-tenant"></a>¿Cómo puedo eliminar un inquilino de Azure AD?

Consulte la entrada del blog de Eric Golpe [Walkthrough of Deleting an Azure AD Tenant](/archive/blogs/ericgolpe/walkthrough-of-deleting-an-azure-ad-tenant) (Tutorial para la eliminación de inquilinos de Azure AD).

### <a name="next-steps"></a>Pasos siguientes

También puede explorar algunas de las demás características y funcionalidades de los aceleradores de soluciones de IoT:

* [Implementación del acelerador de soluciones de Connected Factory](quickstart-connected-factory-deploy.md)
* [Seguridad de Internet de las cosas desde el principio](../iot-fundamentals/iot-security-ground-up.md)