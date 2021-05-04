---
title: Inicio rápido para crear un proyecto de Azure Migrate mediante una plantilla de Azure Resource Manager.
description: En este inicio rápido, aprenderá a crear un proyecto de Azure Migrate mediante una plantilla de Azure Resource Manager (plantilla de ARM).
ms.date: 04/23/2021
author: rahulg1190
manager: bsiva
ms.author: rahugup
ms.topic: quickstart
ms.custom:
- subject-armqs
- mode-arm
ms.openlocfilehash: 095f929bcda569b918f69741f31b5286a2652942
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108069914"
---
# <a name="quickstart-create-an-azure-migrate-project-using-an-arm-template"></a>Inicio rápido: Creación de un proyecto de Azure Migrate mediante una plantilla de ARM

En este inicio rápido, se describe cómo configurar un proyecto de Azure Migrate mediante una plantilla de Azure Resource Manager (plantilla de ARM). Azure Migrate proporciona un centro centralizado para evaluar y migrar a Azure servidores, infraestructuras, aplicaciones y datos locales. Azure Migrate admite la valoración y migración de máquinas virtuales de VMware locales, máquinas virtuales de Hyper-V, servidores físicos, otras máquinas virtuales virtualizadas, bases de datos, aplicaciones web y escritorios virtuales.

Esta plantilla crea un proyecto de Azure Migrate que se usará más adelante para evaluar y migrar los servidores, la infraestructura, las aplicaciones y los datos locales de Azure.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si su entorno cumple los requisitos previos y está familiarizado con el uso de plantillas de Resource Manager, seleccione el botón **Implementar en Azure**. La plantilla se abrirá en Azure Portal.

[![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-migrate-project-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Requisitos previos

Si no tiene una suscripción activa a Azure, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="review-the-template"></a>Revisión de la plantilla

La plantilla usada en este inicio rápido forma parte de las [plantillas de inicio rápido de Azure](https://azure.microsoft.com/en-us/resources/templates/101-migrate-project-create/).

:::code language="json" source="~/quickstart-templates/101-migrate-project-create/azuredeploy.json":::



## <a name="deploy-the-template"></a>Implementación de la plantilla

Para implementar la plantilla, se requieren los valores de **Suscripción**, **Grupo de recursos**, **Nombre del proyecto** y **Ubicación**.

1. Para iniciar sesión en Azure y abrir la plantilla, seleccione la imagen **Implementar en Azure**.

   [![Implementación en Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-migrate-project-create%2Fazuredeploy.json)

2. Seleccione o escriba los siguientes valores:

   :::image type="content" source="media/quickstart-create-migrate-project-template/create-migrate-project.png" alt-text="Plantilla para crear un proyecto de Azure Migrate.":::

   - **Suscripción**: Seleccione su suscripción a Azure.
   - **Grupo de recursos**: seleccione un grupo existente, o bien seleccione **Crear nuevo** para agregar un grupo.
   - **Región**: el valor predeterminado es la ubicación del grupo de recursos y deja de estar disponible cuando se selecciona un grupo de recursos.
   - **Nombre del proyecto de Azure Migrate**: escriba el nombre del proyecto.
   - **Ubicación**: seleccione la ubicación donde desea implementar el proyecto de Azure Migrate y sus recursos.

3. Haga clic en el botón **Revisar y crear** para iniciar la implementación.

## <a name="validate-the-deployment"></a>Validación de la implementación

Para confirmar que se ha creado el proyecto de Azure Migrate, use Azure Portal.


1. Para ir a Azure Migrate, busque **Azure Migrate** en la barra de búsqueda de Azure Portal.
2. Haga clic en el botón **Detectar**, **Evaluar** y **Migrar** en el icono de Windows, Linux y SQL Server.
3. Seleccione la **suscripción de Azure** y el **proyecto** según los valores especificados en la implementación.


## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado un proyecto de Azure Migrate. Para más información sobre Azure Migrate y sus funcionalidades, continúe con la información general de Azure Migrate.

> [!div class="nextstepaction"]
> [Información general de Azure Migrate](migrate-services-overview.md).
>
