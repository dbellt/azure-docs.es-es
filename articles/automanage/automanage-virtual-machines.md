---
title: Azure Automanage para máquinas virtuales
description: Conozca Azure Automanage para máquinas virtuales.
author: deanwe
ms.service: virtual-machines
ms.subservice: automanage
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 02/23/2021
ms.author: deanwe
ms.custom: references_regions
ms.openlocfilehash: 514f1af2a1b120254840986fc5ceb803dfc24345
ms.sourcegitcommit: dddd1596fa368f68861856849fbbbb9ea55cb4c7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107363383"
---
# <a name="azure-automanage-for-virtual-machines"></a>Azure Automanage para máquinas virtuales

En este artículo se incluye información sobre Azure Automanage para máquinas virtuales. Este servicio presenta las siguientes ventajas:

- Incorpora máquinas virtuales de forma inteligente a servicios de Azure de procedimientos recomendados.
- Configura automáticamente cada servicio según los procedimientos recomendados de Azure.
- Supervisa las desviaciones y las corrige cuando se detectan.
- Proporciona una experiencia sencilla (punto, clic, establecer, olvidar)


## <a name="overview"></a>Información general

Azure Automanage para máquinas virtuales es un servicio que elimina la necesidad de detectar, saber incorporar y configurar determinados servicios en Azure que serían beneficiosos para una máquina virtual. Se considera que estos son servicios de procedimientos recomendados de Azure y que ayudan a mejorar la confiabilidad, seguridad y administración de máquinas virtuales. Algunos ejemplos de estos servicios son [Azure Update Management](../automation/update-management/overview.md) y [Azure Backup](../backup/backup-overview.md).

Después de incorporar las máquinas virtuales a Azure Automanage, se usa la configuración recomendada en todos los servicios de procedimientos recomendados. Los procedimientos recomendados son diferentes para cada uno de los servicios. Por ejemplo, en Azure Backup el procedimiento recomendado es realizar una copia de seguridad de la máquina virtual una vez al día y tener un período de retención de seis meses.

Azure Automanage también supervisa automáticamente las desviaciones y las corrige cuando se detectan. Esto significa que, si la máquina virtual se incorpora a Azure Automanage, no solo se configurará según los procedimientos recomendados de Azure, sino que se supervisará para garantizar que sigue cumpliendo esos procedimientos recomendados durante todo su ciclo de vida. Si la máquina virtual se desvía de esos procedimientos (por ejemplo, si se retira un servicio), se corregirá y se la devolverá al estado deseado.

## <a name="prerequisites"></a>Requisitos previos

Antes de intentar habilitar Azure Automanage en las máquinas virtuales, debe tener en cuenta varios requisitos previos.

- [Versiones de Windows Server](automanage-windows-server.md#supported-windows-server-versions) y [distribuciones de Linux](automanage-linux.md#supported-linux-distributions-and-versions) compatibles
- Las máquinas virtuales deben estar en una región admitida (véase a continuación)
- El usuario debe tener los permisos correctos (véase a continuación)
- Automanage no admite suscripciones de espacio aislado en este momento

### <a name="supported-regions"></a>Regiones admitidas
Automanage solo admite máquinas virtuales que se encuentran en las siguientes regiones:
* Oeste de Europa
* Norte de Europa
* Centro de EE. UU.
* Este de EE. UU.
* Este de EE. UU. 2
* Oeste de EE. UU.
* Oeste de EE. UU. 2
* Centro de Canadá
* Centro-Oeste de EE. UU.
* Centro-sur de EE. UU.
* Japón Oriental
* Sur de Reino Unido 2
* Este de Australia
* Sudeste de Australia
* Sudeste de Asia

### <a name="required-rbac-permissions"></a>Permisos de RBAC necesarios
Su cuenta requerirá roles de RBAC ligeramente diferentes en función de si va a habilitar Automanage con una cuenta de Automanage nueva.

Si va a habilitar Automanage con una cuenta de Automanage nueva:
* Rol de **Propietario** en las suscripciones que contienen las máquinas virtuales, _**o**_
* Roles de **Colaborador** y **Administrador de acceso de usuario** en las suscripciones que contienen las máquinas virtuales

Si va a habilitar Automanage con una cuenta de Automanage existente:
* Rol de **Colaborador** en el grupo de recursos que contiene las máquinas virtuales

Se concederá a la cuenta de Automanage permisos de **Colaborador** y **Colaborador de directivas de recursos** para realizar acciones en las máquinas de Automanage.

> [!NOTE]
> Si quiere usar Automanage en una máquina virtual que está conectada a un área de trabajo en una suscripción diferente, debe tener los permisos descritos anteriormente en cada suscripción.

## <a name="participating-services"></a>Servicios participantes

:::image type="content" source="media\automanage-virtual-machines\intelligently-onboard-services-1.png" alt-text="Incorporación inteligente de servicios.":::

Para obtener una lista completa de los servicios de Azure participantes, así como su entorno compatible, consulte estos temas:
- [Automanage para Linux](automanage-linux.md)
- [Automanage para Windows Server](automanage-windows-server.md)

 Le incorporaremos automáticamente a estos servicios participantes. Estos servicios son fundamentales para nuestra nota del producto de procedimientos recomendados, que puede encontrar en nuestro marco [Cloud Adoption Framework](/azure/cloud-adoption-framework/manage/azure-server-management).

Todos estos servicios se incorporarán y configurarán automáticamente, se supervisarán en busca de variaciones y, en caso de detectarse estas, se corregirán.


## <a name="enabling-automanage-for-vms-in-azure-portal"></a>Habilitación de Automanage para máquinas virtuales en Azure Portal

En Azure Portal, puede habilitar Automanage en máquinas virtuales nuevas o existentes. Para conocer los pasos resumidos de este proceso, consulte el [inicio rápido de Automanage para máquinas virtuales](quick-create-virtual-machines-portal.md).

Si es la primera vez que habilita Automanage en la máquina virtual, puede buscar **Automanage: procedimientos recomendados de máquinas virtuales de Azure** en Azure Portal. Haga clic en **Enable on existing VM** (Habilitar en máquina virtual existente), seleccione las máquinas virtuales que quiere incorporar, haga clic en **Seleccionar** y en **Habilitar**, y ya está.

El único caso en que podría tener que interactuar con esta máquina virtual para administrar estos servicios es si intentamos corregir la máquina virtual, pero no lo conseguimos. Si hemos corregido la máquina virtual correctamente, la devolveremos a su estado de conformidad sin avisarle. Para más información, consulte [Estado de las máquinas virtuales](#status-of-vms).

## <a name="enabling-automanage-for-vms-using-azure-policy"></a>Habilitación de Automanage para las máquinas virtuales mediante Azure Policy
También puede habilitar Automanage en las máquinas virtuales a escala mediante la instancia de Azure Policy integrada. La directiva tiene un efecto de DeployIfNotExists, lo que significa que todas las máquinas virtuales que se encuentren dentro del ámbito de la directiva se incorporarán automáticamente a los procedimientos recomendados de VM de Automanage.

[Este es un vínculo directo ](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F270610db-8c04-438a-a739-e8e6745b22d3) a la directiva.

### <a name="how-to-apply-the-policy"></a>Cómo aplicar la directiva
1. Haga clic en el botón **Asignar** mientras visualiza la definición de directiva.
1. Seleccione el ámbito en el que quiere aplicar la directiva (puede ser un grupo de administración, suscripción o grupo de recursos).
1. En **Parámetros**, especifique los parámetros de la cuenta de Automanage, el perfil de configuración y el efecto (normalmente, el efecto debe ser DeployIfNotExists)
    1. Si no tiene ninguna cuenta de Automanage, deberá [crear una](./automanage-account.md).
1. En **Corrección**, active la casilla "Click a remediation task" (Hacer clic en una tarea de corrección). Esto llevará a cabo la incorporación a Automanage.
1. Haga clic en **Revisar y crear** y asegúrese de que toda la configuración sea correcta.
1. Haga clic en **Crear**.

## <a name="environment-configuration"></a>Configuración del entorno

Para habilitar Automanage en una máquina virtual, se requiere un entorno. Los entornos son la base de este servicio. Definen qué servicios se incorporan a las máquinas y, hasta cierto punto, cuál será la configuración de esos servicios.

### <a name="default-environments"></a>Entornos predeterminados

Actualmente, hay dos entornos disponibles.

- El entorno **Desarrollo/pruebas** está diseñado para máquinas de desarrollo y pruebas.
- El entorno de **Producción** es para producción.

La razón de esta diferencia es que algunos servicios se recomiendan en función de la carga de trabajo en ejecución. Por ejemplo, en una máquina de producción le incorporaremos automáticamente a Azure Backup. Sin embargo, en el caso de una máquina de desarrollo/pruebas, un servicio de copia de seguridad sería un costo innecesario, ya que las máquinas de desarrollo/pruebas suelen tener un menor efecto para la empresa.

### <a name="customizing-an-environment-using-preferences"></a>Uso de las preferencias para personalizar entornos

Además de los servicios estándar a los que le incorporamos, le permitimos configurar un determinado subconjunto de preferencias. Estas preferencias se permiten dentro de una variedad de opciones de configuración. Por ejemplo, en el caso de Azure Backup, le permitiremos definir la frecuencia de la copia de seguridad y el día de la semana en el que se produce.

> [!NOTE]
> En el entorno de desarrollo/pruebas, no se realizará una copia de seguridad de la máquina virtual.

Se puede ajustar la configuración de un entorno predeterminado mediante las preferencias. Aprenda a crear una preferencia [aquí](virtual-machines-custom-preferences.md).

> [!NOTE]
> Mientras Automanage está habilitado, no se puede cambiar la configuración del entorno de la máquina virtual. Tendrá que deshabilitar Automanage en esa máquina virtual y habilitarlo de nuevo con el entorno y las preferencias que desee.

Aquí puede obtener una lista completa de los servicios de Azure que participan y consultar si admiten preferencias:
- [Automanage para Linux](automanage-windows-server.md)
- [Automanage para Windows Server](automanage-windows-server.md)


## <a name="automanage-account"></a>Cuenta de Automanage

La cuenta de Automanage es el contexto de seguridad o la identidad bajo los que tienen lugar las operaciones automatizadas. Por lo general, no es necesario seleccionar la opción de cuenta de Automanage, pero si hubiera un escenario de delegación en el que quisiera dividir la administración automatizada de sus recursos (quizás entre dos administradores del sistema), esta opción del flujo de habilitación le permite definir una identidad de Azure para cada uno de esos administradores.

Para más información sobre la cuenta de Automanage y cómo crear una, visite la [documentación de la cuenta de Automanage](./automanage-account.md).

## <a name="status-of-vms"></a>Estado de las máquinas virtuales

En Azure Portal, vaya a la página **Automanage – Azure virtual machine best practices** (Automanage: procedimientos recomendados de máquina virtual de Azure), que muestra todas las máquinas virtuales administradas automáticamente. Aquí verá el estado general de cada máquina virtual.

:::image type="content" source="media\automanage-virtual-machines\configured-status.png" alt-text="Lista de máquinas virtuales configuradas.":::

Se muestran los siguientes detalles de todas las máquinas virtuales enumeradas: nombre, entorno, preferencia de configuración, estado, sistema operativo, cuenta, suscripción y grupo de recursos.

La columna **Status** (Estado) puede mostrar los siguientes estados:
- *In-progress* (En curso). la máquina virtual se acaba de habilitar y se está configurando.
- *Configured* (Configurada): la máquina virtual está configurada y no se detecta ninguna desviación.
- *Failed*(Error): la máquina virtual presenta desviaciones y no somos capaces de corregirlas.
- *Pendiente*: la máquina virtual no se está ejecutando actualmente y Automanage intentará incorporar o corregir la máquina virtual en su próxima ejecución

Si en **Status** (Estado), aparece *Failed* (Error), puede solucionar el problema de la implementación mediante el grupo de recursos en el que se encuentra la máquina virtual. Vaya a **Resource groups** (Grupos de recursos), seleccione el grupo de recursos, haga clic en **Deployments** (Implementaciones) y vea ahí el estado *Failed* (Error) con los detalles del error.


## <a name="disabling-automanage-for-vms"></a>Deshabilitación de Automanage para máquinas virtuales

Puede decidir un día deshabilitar Automanage en determinadas máquinas virtuales. Por ejemplo, la máquina está ejecutando una carga de trabajo segura muy confidencial y debe bloquearla incluso por encima de lo que Azure haría de forma natural, por lo que debe configurar la máquina fuera de los procedimientos recomendados de Azure.

Para ello, en Azure Portal, vaya a la página **Automanage – Azure virtual machine best practices** (Automanage: procedimientos recomendados de máquina virtual de Azure), que muestra todas las máquinas virtuales administradas automáticamente. Active la casilla situada junto a la máquina virtual en la que quiere deshabilitar Automanage y, luego, haga clic en el botón **Disable automanagment** (Deshabilitar la administración automática).

:::image type="content" source="media\automanage-virtual-machines\disable-step-1.png" alt-text="Deshabilitación de Automanage en una máquina virtual.":::

Lea detenidamente los mensajes del elemento emergente resultante antes de aceptar **Disable** (Deshabilitar).

> [!NOTE]
> Al deshabilitar la administración en una VM, se produce el comportamiento siguiente:
>
> - La configuración de la VM y los servicios que se incorporan no cambian.
> - Los cargos en los que incurran esos servicios se seguirán facturando y se continuarán generando.
> - La supervisión del desfase de datos de Automanage se detiene inmediatamente.


En primer lugar, no se desactivará la máquina virtual de ninguno de los servicios que hayamos incorporado y configurado. Los cargos que generen esos servicios se seguirán facturando. Si es necesario, debe desactivarlos. Cualquier acción de Automanage se detendrá inmediatamente. Por ejemplo, ya no se supervisarán las desviaciones de la máquina virtual.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido que Automanage para máquinas virtuales proporciona un medio para eliminar la necesidad de conocer, incorporar y configurar servicios de Azure de procedimientos recomendados. Además, si una máquina que se ha incorporado a Automanage para máquinas virtuales se desvía del entorno Configurado, la devolveremos automáticamente a su estado de conformidad.

Intente habilitar Automanage para máquinas virtuales en Azure Portal.

> [!div class="nextstepaction"]
> [Habilitación de Automanage para máquinas virtuales en Azure Portal](quick-create-virtual-machines-portal.md)