---
title: Conversión configuraciones en recursos compuestos para Azure Automation State Configuration
description: En este artículo se explica cómo convertir configuraciones en recursos compuestos para Azure Automation State Configuration.
keywords: dsc,powershell,configuration,setup
services: automation
ms.service: automation
ms.subservice: dsc
author: mgreenegit
ms.author: migreene
ms.date: 08/08/2019
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: 3baf87cc1b14ca4d8d7d3630e777b03801398968
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108737556"
---
# <a name="convert-configurations-to-composite-resources"></a>Conversión de configuraciones en recursos compuestos

> Se aplica a: Windows PowerShell 5.1

Una vez que empiece a crear configuraciones, podrá crear rápidamente "escenarios" que administren grupos de valores.
Algunos ejemplos serían:

- crear un servidor web
- crear un servidor de nombres de dominio
- crear un servidor que ejecute SharePoint
- configurar un clúster de SQL
- administrar la configuración del firewall
- administrar la configuración de la contraseña

Si desea compartir este trabajo con otros usuarios, la mejor opción es empaquetar la configuración en forma de [recurso compuesto](/powershell/scripting/dsc/resources/authoringresourcecomposite).
La primera vez que se crean recursos compuestos la tarea puede ser sobrecogedora.

> [!NOTE]
> En este artículo se hace referencia a una solución que se mantiene en la comunidad de código abierto.
> La compatibilidad solo está disponible en forma de colaboración en GitHub, no con Microsoft.

## <a name="community-project-compositeresource"></a>Proyecto de la comunidad: CompositeResource

Para resolver este reto, se ha creado una solución mantenida por la comunidad llamada [CompositeResource](https://github.com/microsoft/compositeresource).

CompositeResource automatiza el proceso de creación de módulos a partir de su configuración.
Para empezar, [use el operador punto](https://devblogs.microsoft.com/scripting/how-to-reuse-windows-powershell-functions-in-scripts/) en el script de configuración de la estación de trabajo (o servidor de compilación) para que se cargue en memoria.
A continuación, en lugar de ejecutar la configuración para generar un archivo MOF, use la función que proporciona el módulo CompositeResource para automatizar una conversión.
El cmdlet cargará el contenido de la configuración, obtendrá la lista de parámetros y generará un módulo con todo lo necesario.

Una vez que haya generado un módulo, puede usar una versión superior y agregar notas de la versión cada vez que realice cambios y la publique en su [repositorio de PowerShellGet](https://powershellexplained.com/2018-03-03-Powershell-Using-a-NuGet-server-for-a-PSRepository/?utm_source=blog&utm_medium=blog&utm_content=psscriptrepo).

Una vez que haya creado un módulo de recursos compuestos que contenga la configuración (o varias configuraciones), puede usarlos en la [creación de composiciones](./compose-configurationwithcompositeresources.md) en Azure, o bien agregarlos a [los scripts de configuración de DSC](/powershell/scripting/dsc/configurations/configurations) para generar archivos MOF y [cargar los archivos MOF en Azure Automation](./tutorial-configure-servers-desired-state.md#create-and-upload-a-configuration-to-azure-automation).
A continuación, registre los servidores desde una [ubicación local](./automation-dsc-onboarding.md#enable-physicalvirtual-linux-machines) o [en Azure](./automation-dsc-onboarding.md#enable-azure-vms) para extraer las configuraciones.
La actualización más reciente del proyecto también ha publicado [runbooks](https://www.powershellgallery.com/packages?q=DscGallerySamples) para que Azure Automation automatice el proceso de importación de configuraciones desde la Galería de PowerShell.

Para probar la automatización de la creación de recursos compuestos para DSC, visite la [Galería de PowerShell](https://www.powershellgallery.com/packages/compositeresource/) y descargue la solución, o bien haga clic en "Project Site" (Sitio del proyecto) para ver la [documentación](https://github.com/microsoft/compositeresource).

## <a name="next-steps"></a>Pasos siguientes

- Para comprender DSC de PowerShell, vea [Información general sobre Desired State Configuration de PowerShell](/powershell/scripting/dsc/overview/overview).
- Obtenga información sobre los recursos de DSC de PowerShell en [Recursos de DSC](/powershell/scripting/dsc/resources/resources).
- Para obtener información detallada sobre la configuración de Configuration Manager, vea [Configuración de Configuration Manager local](/powershell/scripting/dsc/managing-nodes/metaconfig).
