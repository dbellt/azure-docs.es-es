---
title: Comprobación del archivo de definición de la interfaz de usuario
description: Describe cómo probar la experiencia del usuario para la creación de la aplicación administrada de Azure a través del portal.
author: tfitzmac
ms.topic: conceptual
ms.date: 06/04/2021
ms.author: tomfitz
ms.openlocfilehash: 55a966c17f3bcd51f354198e36e03071efd4834d
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111951466"
---
# <a name="test-your-portal-interface-for-azure-managed-applications"></a>Prueba de la interfaz de su portal para Azure Managed Applications

Después de [crear el archivo createUiDefinition.json](create-uidefinition-overview.md) para la aplicación administrada, debe probar la experiencia del usuario. Para simplificar las pruebas, use un entorno de espacio aislado que cargue el archivo en el portal. No es necesario implementar la aplicación administrada. El espacio aislado presenta la interfaz de usuario en la experiencia del portal actual y en pantalla completa. El espacio aislado es la manera recomendada de obtener una vista previa de la interfaz.

## <a name="prerequisites"></a>Prerrequisitos

* Un archivo **createUiDefinition.json**. Si no tiene este archivo, copie el [archivo de ejemplo](https://github.com/Azure/azure-quickstart-templates/blob/master/demos/100-marketplace-sample/createUiDefinition.json).

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="use-sandbox"></a>Uso de un espacio aislado

1. Abra el [espacio aislado de Crear definición de UI](https://portal.azure.com/?feature.customPortal=false&#blade/Microsoft_Azure_CreateUIDef/SandboxBlade).

   ![Mostrar espacio aislado](./media/test-createuidefinition/show-sandbox.png)

1. Reemplace la definición vacía con el contenido del archivo createUiDefinition.json. Seleccione **Vista previa**.

   ![Selección de vista previa](./media/test-createuidefinition/select-preview.png)

1. Se muestra el formulario creado. Puede recorrer la experiencia del usuario y rellenar los valores.

   ![Mostrar formulario](./media/test-createuidefinition/show-ui-form.png)

### <a name="troubleshooting"></a>Solución de problemas

Si no se muestra el formulario después de seleccionar **Versión preliminar**, es posible que haya un error de sintaxis. Busque el indicador rojo en la barra de desplazamiento a la derecha y navegue hasta él.

![Mostrar errores de sintaxis](./media/test-createuidefinition/show-syntax-error.png)

Si no se muestra el formulario y, en su lugar, ve un icono de una nube con una gota, esto significa que el formulario tiene un error, como una propiedad que falta. Abra Web Developer Tools en el explorador. La **consola** muestra mensajes importantes sobre la interfaz.

![Mostrar error](./media/test-createuidefinition/show-error.png)

## <a name="test-your-solution-files"></a>Prueba de los archivos de la solución

Ahora que ha comprobado que la interfaz del portal funciona como se espera, es el momento de validar que el archivo createUiDefinition está correctamente integrado con el archivo mainTemplate.json. Puede ejecutar una prueba del script de validación para probar el contenido de los archivos de la solución, incluido el archivo createUiDefinition. El script valida la sintaxis JSON, comprueba si hay expresiones regex en los campos de texto y se asegura de que los valores de salida de la interfaz del portal coincidan con los parámetros de la plantilla. Para más información sobre cómo ejecutar este script, consulte [Ejecución de comprobaciones de validación estática para plantillas](https://aka.ms/arm-ttk).

## <a name="next-steps"></a>Pasos siguientes

Después de validar la interfaz de su portal, infórmese sobre cómo hacer que la [aplicación administrada de Azure esté disponible en Marketplace](../../marketplace/azure-app-offer-setup.md).