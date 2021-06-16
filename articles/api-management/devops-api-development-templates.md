---
title: CI/CD para Azure API Management mediante plantillas de Resource Manager
description: Introducción a DevOps de API con Azure API Management mediante plantillas de Azure Resource Manager para administrar implementaciones de API en una canalización de CI/CD
services: api-management
author: miaojiang
ms.service: api-management
ms.topic: conceptual
ms.date: 10/09/2020
ms.author: apimpm
ms.openlocfilehash: 49dc71dac6d246907de3498ed0a39541a3caf522
ms.sourcegitcommit: 34feb2a5bdba1351d9fc375c46e62aa40bbd5a1f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111888435"
---
# <a name="cicd-for-api-management-using-azure-resource-manager-templates"></a>CI/CD para API Management mediante plantillas de Azure Resource Manager

En este artículo se muestra cómo usar DevOps de API con Azure API Management mediante plantillas de Azure Resource Manager. Dado el valor estratégico de las API, una canalización de integración continua e implementación continua (CI/CD) se ha convertido en un aspecto importante del desarrollo de las API. Permite a las organizaciones automatizar la implementación de los cambios de API sin pasos manuales propensos a errores, detectar problemas más pronto y, en última instancia, ofrecer valor a los usuarios con mayor rapidez. 

Para obtener más información, herramientas y ejemplos de código para implementar el enfoque de DevOps que se describe en este artículo, consulte el [kit de recursos de DevOps de Azure API Management](https://github.com/Azure/azure-api-management-devops-resource-kit) de código abierto en GitHub. Dado que los clientes presentan una amplia gama de culturas de ingeniería y de soluciones de automatización, el enfoque no es una solución única para todo el mundo.

## <a name="the-problem"></a>El problema.

Hoy en día, las organizaciones suelen tener varios entornos de implementación (como desarrollo, pruebas y producción) y usan instancias de API Management independientes para cada entorno. Varios equipos de desarrollo comparten algunas instancias y son responsables de distintas API con cadencias de liberación diferentes.

Como resultado, los clientes se enfrentan a los siguientes desafíos:

* Cómo se automatiza la implementación de las API en API Management
* Cómo se migran las configuraciones de un entorno a otro
* Cómo se evitan las interferencias entre distintos equipos de desarrollo que comparten la misma instancia de API Management

## <a name="manage-configurations-in-resource-manager-templates"></a>Administración de configuraciones en plantillas de Resource Manager

En la imagen siguiente se ilustra el enfoque propuesto. 

:::image type="content" source="media/devops-api-development-templates/apim-devops.png" alt-text="Diagrama que muestra DevOps con API Management.":::

En este ejemplo, hay dos entornos de implementación: *Desarrollo* y *Producción*. Cada uno tiene su propia instancia de API Management. 

* Los desarrolladores de API tienen acceso a la instancia de desarrollo y pueden usarla para desarrollar y probar sus API. 
* Un equipo designado denominado *Publicadores de API* administra la instancia de producción.

La clave de este enfoque propuesto es conservar todas las configuraciones de API Management en [plantillas de Azure Resource Manager](../azure-resource-manager/templates/syntax.md). La organización debe mantener estas plantillas en un sistema de control de código fuente como Git. Como se muestra en la imagen, un repositorio del publicador contiene todas las configuraciones de la instancia de API Management de producción en una colección de plantillas:

|Plantilla  |Descripción  |
|---------|---------|
|Plantilla de servicio     | Configuraciones de nivel de servicio de la instancia de API Management, como el plan de tarifa y los dominios personalizados.         |
|Plantillas compartidas     |  Recursos compartidos en toda una instancia de API Management, como grupos, productos y registradores.    |
|Plantillas de API     |  Configuraciones de API y sus subrecursos: operaciones, directivas y configuración de diagnóstico.        |
|Plantilla maestra (principal)     |   Conecta todo mediante la [vinculación](../azure-resource-manager/templates/linked-templates.md) de todas las plantillas y su implementación por orden. Para implementar todas las configuraciones en una instancia de API Management, implemente la plantilla principal. También puede implementar cada plantilla de manera individual.       |

Los desarrolladores de API bifurcarán el repositorio del publicador a un repositorio del desarrollador y trabajarán en los cambios de sus API. En la mayoría de los casos, se centran en las plantillas de API para sus API y no es necesario cambiar las plantillas de servicio o compartidas.

## <a name="migrate-configurations-to-templates"></a>Migración de configuraciones a plantillas
Los desarrolladores de API se enfrentan a algunos retos al trabajar con plantillas de Resource Manager:

* Los desarrolladores de API suelen trabajar con la [especificación OpenAPI](https://github.com/OAI/OpenAPI-Specification) y es posible que no estén familiarizados con los esquemas de Resource Manager. La creación manual de plantillas puede ser propensa a errores. 

   Una herramienta llamada [Creator](https://github.com/Azure/azure-api-management-devops-resource-kit/blob/master/src/APIM_ARMTemplate/README.md#Creator) del kit de recursos puede ayudarle a automatizar la creación de plantillas de API basadas en un archivo de especificación OpenAPI. Además, los desarrolladores pueden proporcionar directivas de API Management para una API en formato XML. 

* En el caso de los clientes que ya usan API Management, otro reto es extraer las configuraciones existentes a plantillas de Resource Manager. Para estos clientes, una herramienta llamada [Extractor](https://github.com/Azure/azure-api-management-devops-resource-kit/blob/master/src/APIM_ARMTemplate/README.md#extractor) del kit de recursos puede ayudar a generar plantillas mediante la extracción de configuraciones de sus instancias de API Management.  

## <a name="workflow"></a>Flujo de trabajo

* Una vez que los desarrolladores de API han terminado de desarrollar y probar una API y han generado las plantillas de API, pueden enviar una solicitud de incorporación de cambios para combinar los cambios en el repositorio del publicador. 

* Los publicadores de API pueden validar la solicitud de incorporación de cambios y asegurarse de que los cambios sean seguros y compatibles. Por ejemplo, pueden comprobar si solo HTTPS tiene permitido comunicarse con la API. La mayoría de las validaciones se pueden automatizar como un paso en la canalización de CI/CD.

* Una vez que los cambios se aprueban y se combinan correctamente, los publicadores de API pueden optar por implementarlos en la instancia de producción según una programación o a petición. La implementación de las plantillas se puede automatizar con [Acciones de GitHub](https://github.com/Azure/apimanagement-devops-samples), [Azure Pipelines](/azure/devops/pipelines), [Azure PowerShell](../azure-resource-manager/templates/deploy-powershell.md), la [CLI de Azure](../azure-resource-manager/templates/deploy-cli.md) u otras herramientas.


Con este enfoque, una organización puede automatizar la implementación de los cambios de API en instancias de API Management y es fácil promocionar los cambios de un entorno a otro. Dado que los distintos equipos de desarrollo de API van a trabajar en diferentes conjuntos de archivos y plantillas de API, evita las interferencias entre los distintos equipos.

## <a name="video"></a>Vídeo

> [!VIDEO https://www.youtube.com/embed/4Sp2Qvmg6j8]

## <a name="next-steps"></a>Pasos siguientes

- Para ver información adicional, herramientas y plantillas de ejemplo, consulte el [kit de recursos de DevOps de Azure API Management](https://github.com/Azure/azure-api-management-devops-resource-kit) de código abierto.