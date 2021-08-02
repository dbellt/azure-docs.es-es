---
title: Introducción a las plantillas
description: Describe las ventajas del uso de plantillas de Azure Resource Manager (plantillas de ARM) para la implementación de recursos.
ms.topic: conceptual
ms.date: 03/12/2021
ms.openlocfilehash: 62d25d2bd87f9335ac6442a7038f9326b04b36b0
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111960366"
---
# <a name="what-are-arm-templates"></a>¿Qué son las plantillas de Resource Manager?

Con el traslado a la nube, muchos equipos han adoptado métodos de desarrollo más ágiles. Estos equipos se iteran rápidamente. Necesitan implementar repetidamente sus soluciones en la nube y saber que su infraestructura se encuentra en un estado confiable. Dado que la infraestructura se ha convertido en parte del proceso iterativo, ha desaparecido la división entre las operaciones y el desarrollo. Por ello, los equipos necesitan administrar la infraestructura y el código de aplicación a través de un proceso unificado.

Para cumplir estos desafíos, puede automatizar las implementaciones y usar la práctica de infraestructura como código. En el código, debe definir la infraestructura que va a implementar. Así pues, el código de infraestructura se convierte en parte del proyecto. Al igual que el código de la aplicación, puede almacenar el código de infraestructura en un repositorio de origen y agregarle un número de versión. Cualquier miembro del equipo podrá ejecutar el código e implementar entornos similares.

Para implementar la infraestructura como código para las soluciones de Azure, use las plantillas de Azure Resource Manager (plantillas de ARM). La plantilla es un archivo de notación de objetos JavaScript (JSON) que contiene la infraestructura y la configuración del proyecto. La plantilla usa sintaxis declarativa, lo que permite establecer lo que pretende implementar sin tener que escribir la secuencia de comandos de programación para crearla. En la plantilla se especifican los recursos que se van a implementar y las propiedades de esos recursos.

Hemos introducido un nuevo lenguaje denominado Bicep que se usa para desarrollar archivos JSON para plantillas de ARM. Los archivos de Bicep y las plantillas JSON ofrecen las mismas funcionalidades. Puede convertir las plantillas entre los dos lenguajes. Bicep proporciona una sintaxis que es más fácil de usar para crear plantillas. Para más información, vea [¿Qué es Bicep?](../bicep/overview.md)

Para obtener información sobre cómo puede empezar a trabajar con las plantillas de ARM, consulte el vídeo siguiente.

> [!VIDEO https://channel9.msdn.com/Shows/Azure-Enablement/How-and-why-to-learn-about-ARM-templates/player]

## <a name="why-choose-arm-templates"></a>¿Por qué elegir plantillas de Resource Manager?

Si intenta decidir entre usar plantillas de Resource Manager o una de las demás infraestructuras como servicios de código, tenga en cuenta las ventajas siguientes del uso de plantillas:

* **Sintaxis declarativa**: las plantillas de Resource Manager permiten crear e implementar una infraestructura de Azure completa de forma declarativa. Por ejemplo, puede implementar no solo máquinas virtuales, sino también la infraestructura de red, los sistemas de almacenamiento y cualquier otro recurso que pueda necesitar.

* **Resultados repetibles**: Implemente repetidamente la infraestructura a lo largo del ciclo de vida del desarrollo y tenga la seguridad de que los recursos se implementan de forma coherente. Las plantillas son idempotentes, lo que significa que puede implementar la misma plantilla varias veces y obtener los mismos tipos de recursos en el mismo estado. Puede desarrollar una plantilla que represente el estado deseado, en lugar de desarrollar muchas plantillas independientes para representar las actualizaciones.

* **Orquestación**: No tiene que preocuparse por la complejidad de las operaciones de ordenación. Resource Manager se encarga de gestionar la implementación de recursos interdependientes para que se creen en el orden correcto. Cuando es posible, Resource Manager implementa los recursos en paralelo para que las implementaciones finalicen más rápido que las implementaciones en serie. La plantilla se implementa mediante un comando, en lugar de hacerlo con varios comandos imperativos.

   ![Comparación de la implementación de plantillas](./media/overview/template-processing.png)

* **Archivos modulares**: Puede dividir las plantillas en componentes más pequeños y reutilizables y vincularlos en el momento de la implementación. También puede anidar una plantilla dentro de otras plantillas.

* **Cree cualquier recurso de Azure**: Puede usar inmediatamente los nuevos servicios y características de Azure en las plantillas. En cuanto un proveedor de recursos introduce nuevos recursos, puede implementarlos a través de plantillas. No tiene que esperar a que se actualicen las herramientas o los módulos antes de usar los nuevos servicios.

* **Extensibilidad**: con los [scripts de implementación](deployment-script-template.md), puede agregar scripts de PowerShell o Bash a las plantillas. Los scripts de implementación amplían su capacidad para configurar recursos durante la implementación. Un script se puede incluir en la plantilla o almacenar en un origen externo y hacer referencia a él en la plantilla. Los scripts de implementación le ofrecen la posibilidad de completar la configuración del entorno integral en una sola plantilla de ARM.

* **Pruebas**: puede asegurarse de que la plantilla sigue las instrucciones recomendadas si la prueba con el kit de herramientas de la plantilla ARM (arm-ttk). Este kit de pruebas es un script de PowerShell que puede descargar de [GitHub](https://github.com/Azure/arm-ttk). El kit de herramientas facilita el desarrollo de conocimientos con el lenguaje de plantilla.

* **Vista previa de los cambios**: puede usar la [operación hipotética](./deploy-what-if.md) para obtener una vista previa de los cambios antes de implementar la plantilla. Con la operación hipotética puede ver qué recursos se crearán, actualizarán o eliminarán, así como las propiedades de los recursos que se cambiarán. La operación hipotética comprueba el estado actual del entorno y elimina la necesidad de administrar el estado.

* **Validación integrada**: La plantilla solo se implementa después de pasar la validación. Resource Manager se encarga de comprobar la plantilla antes de iniciar la implementación para asegurarse de que esta se realizará correctamente. Es menos probable que la implementación se detenga a medio acabar.

* **Implementaciones con seguimiento**: En Azure Portal puede revisar el historial de implementación y obtener información sobre la implementación de la plantilla. También puede ver la plantilla que se implementó, los valores de parámetro agregados y los valores de salida. Recuerde que no se realiza el seguimiento de otras infraestructuras como servicios de código a través del portal.

   ![Historial de implementación](./media/overview/deployment-history.png)

* **Directiva como código**: [Azure Policy](../../governance/policy/overview.md) es un marco de directiva como código para automatizar la gobernanza. Si usa directivas de Azure, la corrección de la directiva se realiza en recursos no compatibles cuando se implementa mediante plantillas.

* **Planos técnicos de implementación**: Puede aprovechar las ventajas de los [Planos técnicos](../../governance/blueprints/overview.md) proporcionados por Microsoft para cumplir los estándares de cumplimiento normativo. Estos planos técnicos incluyen plantillas precompiladas para distintas arquitecturas.

* **Integración de CI/CD**: Puede integrar plantillas en sus herramientas de integración e implementación continuas (CI/CD), que pueden automatizar las canalizaciones de versión para llevar a cabo actualizaciones de infraestructura y aplicaciones rápidas y confiables. Mediante la tarea de plantilla de Resource Manager y Azure DevOps puede usar Azure Pipelines para compilar e implementar proyectos de plantillas de Resource Manager de manera continua. Para más información, consulte [Proyecto de VS con canalizaciones](add-template-to-azure-pipelines.md) y el [Tutorial: Integración continua de plantillas de Azure Resource Manager en Azure Pipelines](./deployment-tutorial-pipeline.md).

* **Código exportable**: Puede recuperar una plantilla de un grupo de recursos existente mediante la exportación del estado actual del grupo de recursos o la visualización de la plantilla de una implementación determinada. Una buena estrategia para aprender sobre la sintaxis de una plantilla es consultar la [plantilla exportada](export-template-portal.md).

* **Herramientas de creación**: Puede crear plantillas con [Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md) y la extensión de la herramienta de plantillas. Podrá utilizar IntelliSense, el resaltado de sintaxis, la ayuda en línea y muchas otras funciones de lenguaje. Además de Visual Studio Code, también puede usar [Visual Studio](create-visual-studio-deployment-project.md).

## <a name="template-file"></a>Archivo de plantilla

Dentro de la plantilla, puede escribir [expresiones de plantilla](template-expressions.md) que aumentan las capacidades de JSON. Estas expresiones hacen uso de las [funciones](template-functions.md) que proporciona Resource Manager.

La plantilla contiene las secciones siguientes:

* [Parámetros](./parameters.md): proporcione valores durante la implementación que permitan usar la misma plantilla con entornos diferentes.

* [Variables](./variables.md): defina los valores que se reutilizan en las plantillas. Se pueden crear a partir de valores de parámetro.

* [Funciones definidas por el usuario](./user-defined-functions.md): cree funciones personalizadas que simplifiquen la plantilla.

* [Recursos](resource-declaration.md): especifique los recursos que se van a implementar.

* [Salidas](./outputs.md): devuelva valores de los recursos implementados.

## <a name="template-deployment-process"></a>Proceso de implementación de plantillas

Cuando se implementa una plantilla, Resource Manager la convierte en operaciones de la API de REST. Por ejemplo, cuando Resource Manager recibe una plantilla con la siguiente definición de recursos:

```json
"resources": [
  {
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2019-04-01",
    "name": "mystorageaccount",
    "location": "westus",
    "sku": {
      "name": "Standard_LRS"
    },
    "kind": "StorageV2",
    "properties": {}
  }
]
```

Convierte la definición en la siguiente operación de API de REST, que se envía al proveedor de recursos Microsoft.Storage:

```HTTP
PUT
https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Storage/storageAccounts/mystorageaccount?api-version=2019-04-01
REQUEST BODY
{
  "location": "westus",
  "sku": {
    "name": "Standard_LRS"
  },
  "kind": "StorageV2",
  "properties": {}
}
```

Tenga en cuenta que el valor de **apiVersion** establecido en la plantilla para el recurso se usa como la versión de API para la operación REST. Puede implementar la plantilla varias veces y confiar en que seguirá funcionando. Al usar la misma versión de la API, no tiene que preocuparse de los cambios importantes que se pueden introducir en versiones posteriores.

Para implementar una plantilla, use cualquiera de las siguientes opciones:

* [Azure Portal](deploy-portal.md)
* [CLI de Azure](deploy-cli.md)
* [PowerShell](deploy-powershell.md)
* [REST API](deploy-rest.md)
* [Button del repositorio de GitHub](deploy-to-azure-button.md)
* [Azure Cloud Shell](deploy-cloud-shell.md)

## <a name="template-design"></a>Diseño de plantilla

La definición de plantillas y grupos de recursos depende únicamente de usted, al igual que la administración de la solución. Por ejemplo, puede implementar su aplicación de tres niveles a través de una única plantilla en un único grupo de recursos.

![plantilla de tres niveles](./media/overview/3-tier-template.png)

No obstante, no es necesario que defina toda la infraestructura en una sola plantilla. A menudo, tiene sentido dividir los requisitos de implementación en un conjunto de plantillas seleccionadas, específicas para un propósito. Estas plantillas se pueden reutilizar fácilmente para distintas soluciones. Para implementar una solución concreta, cree una plantilla principal que vincule todas las plantillas necesarias. La imagen siguiente muestra cómo implementar una solución de tres niveles mediante una plantilla principal que incluye tres plantillas anidadas.

![plantilla de niveles anidada](./media/overview/nested-tiers-template.png)

Si desea que sus niveles tengan ciclos de vida independientes, puede implementar los tres niveles en grupos de recursos independientes. Observe que todavía se pueden vincular los recursos a los recursos de otros grupos.

![plantilla de niveles](./media/overview/tier-templates.png)

Para más información acerca de las plantillas anidadas, consulte [Uso de plantillas vinculadas con Azure Resource Manager](linked-templates.md).

## <a name="share-templates"></a>Uso compartido de plantillas

Después de crear la plantilla, puede que desee compartirla con otros usuarios de la organización. Las [especificaciones de plantilla](template-specs.md) permiten almacenar una plantilla como un tipo de recurso. Usará el control de acceso basado en rol para conceder acceso a la especificación de plantilla. Los usuarios con acceso de lectura a la especificación de plantilla pueden implementarla, pero no cambiar la plantilla.

Este enfoque significa que puede compartir de forma segura plantillas que satisfagan los estándares de su organización.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener un tutorial paso a paso que le guíe en el proceso de creación de una plantilla, consulte [Tutorial: Creación e implementación de su primera plantilla de Resource Manager](template-tutorial-create-first-template.md).
* Para una guía en módulos con información sobre las plantillas de ARM en Microsoft Learn, consulte [Implementación y administración de recursos en Azure mediante plantillas de ARM](/learn/paths/deploy-manage-resource-manager-templates/).
* Para obtener información sobre las propiedades de los archivos de plantilla, vea [Nociones sobre la estructura y la sintaxis de las plantillas de Azure Resource Manager](./syntax.md).
* Para obtener información sobre la exportación de plantillas, consulte [Guía de inicio rápido: Creación e implementación de plantillas de Resource Manager mediante Azure Portal](quickstart-create-templates-use-the-portal.md).
* Para obtener respuestas a preguntas comunes, consulte [Preguntas más frecuentes sobre las plantillas de ARM](frequently-asked-questions.md).