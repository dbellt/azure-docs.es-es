---
title: Administración de proyectos de Azure Migrate a escala
description: Obtenga información sobre cómo usar Azure Migrate con eficacia en recursos delegados de clientes.
ms.date: 05/11/2021
ms.topic: how-to
ms.openlocfilehash: a449244de1a834b8b85396d716c49415ac32c6f3
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109784258"
---
# <a name="manage-azure-migrate-projects-at-scale-with-azure-lighthouse"></a>Administración de proyectos de Azure Migrate a gran escala con Azure Lighthouse

En este tema se proporciona información general sobre cómo [Azure Lighthouse](../overview.md) puede ayudarle a usar [Azure Migrate](../../migrate/migrate-services-overview.md) de forma escalable en varios inquilinos de Azure Active Directory (Azure AD).

Azure Lighthouse permite a los proveedores de servicios realizar operaciones a escala a través de varios inquilinos a la vez, lo que hace que las tareas de administración sean más eficaces.

Azure Migrate proporciona un centro centralizado para evaluar y migrar a Azure servidores, infraestructuras, aplicaciones y datos locales. Normalmente, los asociados que realizan evaluaciones y migración a gran escala para varios clientes deben acceder a cada suscripción de cliente individualmente mediante el [modelo de suscripción CSP (proveedor de soluciones en la nube)](/partner-center/customers-revoke-admin-privileges) o la [creación de un usuario invitado en el inquilino del cliente](../../active-directory/external-identities/what-is-b2b.md).

La integración de Azure Lighthouse con Azure Migrate permite a los proveedores de servicios detectar, evaluar y migrar cargas de trabajo para diferentes clientes a gran escala, a la vez que permite a los clientes tener una visibilidad y un control completos de sus entornos. Mediante la administración de recursos delegados de Azure, los proveedores de servicios tienen una vista única de todos los proyectos de Azure Migrate que administran en varios inquilinos de los clientes.

> [!NOTE]
> Mediante Azure Lighthouse, los asociados pueden detectar, evaluar y migrar máquinas virtuales de VMware locales, máquinas virtuales de Hyper-V, servidores físicos e instancias de AWS/GCP. Hay dos opciones para la [migración de máquinas virtuales de VMware](../../migrate/server-migrate-overview.md). Actualmente, solo se puede usar el método de migración basado en agentes cuando se trabaja en un proyecto de migración en una suscripción de cliente delegada. Actualmente no se admite la migración mediante la replicación sin agente con el acceso delegado al ámbito del cliente.

> [!TIP]
> Aunque en este tema hacemos referencia a proveedores de servicios y clientes, esta guía es aplicable también a [empresas que usan Azure Lighthouse para administrar varios inquilinos](../concepts/enterprise.md).

En función de su escenario, puede que desee crear el proyecto de Azure Migrate en el inquilino del cliente o en el inquilino de administración. Revise las consideraciones siguientes y determine qué modelo satisface mejor las necesidades de migración del cliente.

## <a name="create-an-azure-migrate-project-in-the-customer-tenant"></a>Creación de un proyecto de Azure Migrate en el inquilino del cliente

Una opción cuando se usa Azure Lighthouse es crear el proyecto de Azure Migrate en el inquilino del cliente. Los usuarios del inquilino de administración pueden seleccionar la suscripción del cliente al crear un proyecto de migración. En el inquilino de administración, el proveedor de servicios puede realizar las operaciones de migración necesarias. Esto puede incluir la implementación del dispositivo de Azure Migrate para detectar las cargas de trabajo, la evaluación de las cargas de trabajo mediante la agrupación de las máquinas virtuales y el cálculo de los costos relacionados con la nube, la revisión de la preparación de la máquina virtual y la realización de la migración.

En este escenario, no se creará ningún recurso y se almacenará en el inquilino de administración, aunque los pasos de detección y evaluación se pueden iniciar y ejecutar desde ese inquilino. Todos los recursos, como los proyectos de migración, los informes de evaluación de las cargas de trabajo locales y los recursos migrados en el destino específico, se implementarán en la suscripción del cliente. Sin embargo, el proveedor de servicios puede acceder a todos los proyectos del cliente desde su propia experiencia con el inquilino y el portal.

Este enfoque minimiza el cambio de contexto para los proveedores de servicios que trabajan en varios clientes, a la vez que permite que los clientes mantengan todos sus recursos en sus propios inquilinos.

El flujo de trabajo de este modelo será similar al siguiente:

1. El cliente se [incorpora a Azure Lighthouse](onboard-customer.md). El rol integrado de colaborador es necesario para la identidad que se usará con Azure Migrate. Para ver un ejemplo de uso de este rol, vea la plantilla de ejemplo [delegated-resource-management-azmigrate](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate).
1. El usuario designado inicia sesión en el inquilino de administración en Azure Portal y, después, pasa a Azure Migrate. Este usuario [crea un proyecto de Azure Migrate](../../migrate/create-manage-projects.md), mediante selección de la suscripción de cliente delegada adecuada.
1. A continuación, el usuario [realiza pasos de detección y evaluación](../../migrate/tutorial-discover-vmware.md).

   Si se trata de máquinas virtuales de VMware, antes de configurar el dispositivo, puede limitar la detección a centros de datos, clústeres, una carpeta de clústeres, hosts, una carpeta de hosts o VM concretas de vCenter Server. Para establecer el ámbito, asigne permisos en la cuenta que usa el dispositivo para acceder a vCenter Server. Esto resulta útil si se hospedan varias máquinas virtuales de clientes en el hipervisor. No se puede limitar el ámbito de detección de Hyper-V.

    > [!NOTE]
    > Puede detectar y evaluar máquinas virtuales de VMware para la migración mediante Azure Migrate con el acceso delegado que proporciona Azure Lighthouse. Para la migración de máquinas virtuales de VMware, solo se admite actualmente el método basado en agentes cuando se trabaja en un proyecto de migración en una suscripción de cliente delegada.

1. Cuando la suscripción de cliente de destino esté lista, continúe con la migración mediante el acceso concedido por Azure Lighthouse. El proyecto de migración que contiene los resultados de la evaluación y los recursos migrados se creará en el inquilino del cliente en la suscripción de destino.

> [!TIP]
> Antes de la migración, se debe implementar una zona de aterrizaje para aprovisionar los recursos de la infraestructura de base y preparar la suscripción a la que se migrarán las máquinas virtuales. Para acceder a algunos recursos o crearlos en esta zona de aterrizaje, puede ser necesario el rol integrado de propietario, que no se admite actualmente en Azure Lighthouse. En estos casos, es posible que el cliente tenga que proporcionar el rol de acceso de invitado o delegar el acceso de administrador mediante el modelo de suscripción de CSP. Para adoptar un enfoque para la creación de zonas de aterrizaje multiinquilino, vea la [solución de demostración Multi-tenant-Landing-Zones](https://github.com/Azure/Multi-tenant-Landing-Zones) en GitHub.

## <a name="create-an-azure-migrate-project-in-the-managing-tenant"></a>Creación de un proyecto de Azure Migrate en el inquilino de administración

En este escenario, las operaciones relacionadas con la migración, como la detección y la evaluación, todavía las realizan los usuarios en el inquilino de administración. Sin embargo, el proyecto de migración y todos los recursos pertinentes residirán en el inquilino del asociado, y el cliente no tendrá visibilidad directa del proyecto (a pesar de que las evaluaciones se pueden compartir con los clientes, si así lo desea). El destino de la migración de cada cliente será la suscripción del cliente.

Este enfoque permite a los proveedores de servicios iniciar rápidamente proyectos de detección y evaluación de la migración, lo que abstrae los pasos iniciales de las suscripciones y los inquilinos del cliente.

El flujo de trabajo de este modelo será similar al siguiente:

1. El cliente se [incorpora a Azure Lighthouse](onboard-customer.md). El rol integrado de colaborador es necesario para la identidad que se usará con Azure Migrate. Para ver un ejemplo de uso de este rol, vea la plantilla de ejemplo [delegated-resource-management-azmigrate](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate).
1. El usuario designado inicia sesión en el inquilino de administración en Azure Portal y, después, pasa a Azure Migrate. Este usuario [crea un proyecto de Azure Migrate](../../migrate/create-manage-projects.md) en una suscripción que pertenece al inquilino de administración.
1. A continuación, el usuario [realiza pasos de detección y evaluación](../../migrate/tutorial-discover-vmware.md). Las máquinas virtuales locales se detectarán y evaluarán dentro del proyecto de migración creado en el inquilino de administración y, después, se migrarán desde allí.

   Si administra varios clientes en el mismo host de Hyper-V, puede detectar todas las cargas de trabajo a la vez. Las máquinas virtuales específicas del cliente se pueden seleccionar en el mismo grupo, se puede crear una evaluación y se puede realizar la migración seleccionando la suscripción del cliente adecuada como destino específico. No es necesario limitar el ámbito de detección, y se puede mantener una visión general completa de todas las cargas de trabajo de cliente en un proyecto de migración.

1. Cuando esté listo, continúe con la migración mediante la selección de la suscripción de cliente delegada como destino específico para replicar y migrar las cargas de trabajo. Los recursos que se acaban de crear existirán en la suscripción del cliente, mientras que los datos de evaluación y los recursos pertenecientes al proyecto de migración permanecerán en el inquilino de administración.

NOTA: Debe modificar el archivo de parámetros para que refleje el entorno antes de implementar https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegated-resource-management-azmigrate.

## <a name="partner-recognition-for-customer-migrations"></a>Reconocimiento de asociados para migraciones de clientes

Como miembro de [Microsoft Partner Network](https://partner.microsoft.com), puede vincular su identificador de asociado con las credenciales utilizadas para administrar los recursos de cliente delegados. Esto permite que Microsoft pueda atribuir la influencia y los ingresos consumidos por Azure a su organización en función de las tareas que realice para los clientes, incluidos los proyectos de migración.

Para más información, consulte [Vinculación de un identificador de asociado para habilitar el crédito que ha obtenido un asociado en los recursos delegados](partner-earned-credit.md).

## <a name="next-steps"></a>Pasos siguientes

- Más información acerca de [Azure Migrate](../../migrate/migrate-services-overview.md).
- Obtenga información sobre otras [experiencias de administración entre inquilinos](../concepts/cross-tenant-management-experience.md) compatibles con Azure Lighthouse.
