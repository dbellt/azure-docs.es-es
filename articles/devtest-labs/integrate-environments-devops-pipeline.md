---
title: Integración de entornos en Azure Pipelines en Azure DevTest Labs
description: Aprenda a integrar entornos de Azure DevTest Labs en las canalizaciones de integración continua (CI) y entrega continua (CD) de Azure DevOps.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 18f334fe85b6a38c38fc41d55c711ee6d6629760
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112006138"
---
# <a name="integrate-environments-into-your-azure-devops-cicd-pipelines"></a>Integración de entornos en las canalizaciones CI/CD de Azure DevOps
Puede usar la extensión Azure DevTest Labs Tasks que se instala en Azure DevOps Services (anteriormente conocido como Visual Studio Team Services) para integrar fácilmente la canalización de compilación y versión de integración continua (CI) y entrega continua (CD) con Azure DevTest Labs. Estas extensiones permiten que sea más fácil implementar rápidamente un [entorno](devtest-lab-test-env.md) para una determinada tarea de prueba y, luego, eliminarlo cuando finalice la prueba. 

En este artículo se muestra cómo crear e implementar un entorno para luego eliminarlo, todo ello en una canalización completa. Lo habitual sería realizar cada tarea de forma individual en su propia canalización de compilación, prueba e implementación. Las extensiones usadas en este artículo son un complemento a estas [tareas de creación o eliminación de máquinas virtuales de DTL](devtest-lab-integrate-ci-cd.md):

- Creación de un entorno
- Eliminación de un entorno

## <a name="before-you-begin"></a>Antes de empezar
Antes de integrar la canalización de CI/CD con Azure DevTest Labs, instale la extensión [Azure DevTest Labs Tasks](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks) desde Visual Studio Marketplace. 

## <a name="create-and-configure-the-lab-for-environments"></a>Creación y configuración del laboratorio para los entornos
En esta sección se describe cómo crear y configurar un laboratorio donde se va a implementar el entorno de Azure.

1. [Cree un laboratorio](devtest-lab-create-lab.md) si aún no tiene uno. 
2. Configure el laboratorio y cree una plantilla de entorno siguiendo las instrucciones de este artículo: [Creación de entornos de varias máquinas virtuales y recursos de PaaS con plantillas de Azure Resource Manager](devtest-lab-create-environment-from-arm.md).
3. En este ejemplo, use una plantilla de inicio rápido de Azure existente [https://azure.microsoft.com/resources/templates/web-app-redis-cache-sql-database](https://azure.microsoft.com/resources/templates/web-app-redis-cache-sql-database).
4. Copie la carpeta **web-app-redis-cache-sql-database** en la carpeta **ArmTemplate** del repositorio configurado en el paso 2.

## <a name="create-a-release-definition"></a>Creación de una definición de versión
Para crear la definición de la versión, siga estos pasos:

1.  En la pestaña **Versiones** del concentrador **Compilación y versión**, seleccione el botón con el **signo de la suma (+)**.
2.  En la ventana **Crear definición de versión**, seleccione la plantilla **Vacía** y, luego, seleccione **Siguiente**.
3.  Seleccione **Elegir más tarde** y, luego, seleccione **Crear** para crear una nueva definición de versión con un entorno predeterminado y ningún artefacto vinculado.
4.  Para abrir el menú contextual, en la nueva definición de versión, seleccione los **puntos suspensivos (...)** junto al nombre del entorno y, luego, seleccione **Configurar variables**.
5.  En la ventana de **configuración del entorno**, escriba los siguientes valores para las variables que se usan en las tareas de definición de versión:
1.  En **administratorLogin**, escriba el nombre de inicio de sesión del administrador de SQL.
2.  En **administratorLoginPassword**, escriba la contraseña que va a usar con el inicio de sesión del administrador de SQL. Use el icono de "candado" para ocultar y proteger la contraseña.
3.  En **databaseName**, escriba el nombre de la base de datos SQL.
4.  Estas variables son específicas de los entornos de ejemplo, entornos diferentes pueden tener distintas variables.

## <a name="create-an-environment"></a>Creación de un entorno
La siguiente fase de la implementación es crear el entorno que se usará con fines de desarrollo o prueba.

1. En la definición de versión, seleccione **Agregar tareas**.
2. En la pestaña **Tareas**, agregue una tarea de creación de entorno de Azure DevTest Labs. Configure la tarea de la siguiente manera:
    1. En **Suscripción de Azure Resource Manager**, seleccione una conexión en la lista **Conexiones a servicios de Azure disponibles**, o cree una conexión con permisos más restrictivos a la suscripción de Azure. Para más información, consulte [Azure Resource Manager service endpoint](/azure/devops/pipelines/library/service-endpoints) (Punto de conexión de servicio de Azure Resource Manager).
2. En **Nombre del laboratorio**, seleccione el nombre de la instancia que creó anteriormente*.
3. En **Nombre del repositorio**, seleccione el repositorio donde se ha insertado la plantilla de Resource Manager (201)*.
4. En **Nombre de la plantilla**, seleccione el nombre del entorno en el que guardó el repositorio de código fuente*. 
5. El **nombre del laboratorio**, el **nombre del repositorio** y el **nombre de la plantilla** son las representaciones descriptivas de los identificadores de recursos de Azure. Escribir manualmente el nombre descriptivo provoca errores, así que use las listas desplegables para seleccionar la información.
6. En **Nombre del entorno**, escriba un nombre para identificar de forma única la instancia del entorno en el laboratorio.  Debe ser único dentro del laboratorio.
7. Las secciones **Parameter File** (Archivo de parámetro) y **Parameters** (Parámetros) permiten pasar parámetros personalizados al entorno. Se pueden usar una o ambas secciones para establecer los valores de parámetro. En este ejemplo, se usará la sección de parámetros. Use los nombres de las variables que definió en el entorno, por ejemplo: `-administratorLogin "$(administratorLogin)" -administratorLoginPassword "$(administratorLoginPassword)" -databaseName "$(databaseName)" -cacheSKUCapacity 1`
8. La información dentro de la plantilla de entorno se puede recorrer en la sección de salida de la plantilla. Active la casilla **Create output variables based on the environment template output** (Crear variables de salida según la salida de la plantilla de entorno) para que otras tareas puedan usar los datos. `$(Reference name.Output Name)` es el patrón a seguir. Por ejemplo, si el nombre de referencia era DTL y el nombre de salida de plantilla era ubicación, la variable sería `$(DTL.location)`.

## <a name="delete-the-environment"></a>Eliminación del entorno
La fase final consiste en eliminar el entorno que ha implementado en su instancia de Azure DevTest Labs. Lo habitual es eliminar el entorno después de ejecutar las tareas de desarrollo o de ejecutar las pruebas que necesita en los recursos implementados.

En la definición de versión, seleccione **Agregar tareas** y, luego, en la pestaña **Implementar**, agregue una tarea de **eliminación de entorno de Azure DevTest Labs**. Configúrelo de la siguiente forma:

1. Para eliminar la máquina virtual, consulte [Azure DevTest Labs Tasks](https://marketplace.visualstudio.com/items?itemName=ms-azuredevtestlabs.tasks).
    1. En **Suscripción de Azure Resource Manager**, seleccione una conexión en la lista **Conexiones a servicios de Azure disponibles**, o cree una conexión con permisos más restrictivos a la suscripción de Azure. Para más información, consulte [Azure Resource Manager service endpoint](/azure/devops/pipelines/library/service-endpoints) (Punto de conexión de servicio de Azure Resource Manager).
    2. En **Nombre del laboratorio**, seleccione aquel donde existe el entorno.
    3. En **Nombre del entorno**, escriba el nombre del entorno que se va a quitar.
2. Escriba un nombre para la definición de versión y, a continuación, guárdelo.

## <a name="next-steps"></a>Pasos siguientes
Vea los artículos siguientes: 
- [Creación de entornos de varias máquinas virtuales con plantillas de Resource Manager](devtest-lab-create-environment-from-arm.md)
- Plantillas de inicio rápido de Resource Manager para la automatización de DevTest Labs desde el [repositorio de GitHub de DevTest Labs](https://github.com/Azure/azure-quickstart-templates).
- [Página de solución de problemas de VSTS](/azure/devops/pipelines/troubleshooting)

