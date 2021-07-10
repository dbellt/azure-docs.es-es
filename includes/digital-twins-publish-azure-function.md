---
author: baanders
description: Archivo de inclusión del proceso de publicación de una función de Azure desde Visual Studio
ms.service: digital-twins
ms.topic: include
ms.date: 1/21/2021
ms.author: baanders
ms.openlocfilehash: 49f57f39ba59575417f3336d395b1cfa6cf97eb6
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110451114"
---
Para publicar el proyecto en una aplicación de funciones en Azure, empiece por el Explorador de soluciones. Haga clic con el botón derecho en el proyecto y, después, elija **Publicar**.

> [!IMPORTANT] 
> La publicación de una aplicación de funciones en Azure incurrirá en cargos adicionales por su suscripción, independientemente de Azure Digital Twins.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-1.png" alt-text="Captura de pantalla de Visual Studio que muestra el menú de botón derecho de la solución. En el menú aparece resaltado &quot;Publicar&quot;.":::

En la página **Publicar** que se abre, mantenga la selección destino predeterminada de **Azure**. Luego, seleccione **Siguiente**. 

Para seleccionar un destino concreto, elija **Azure Function App (Windows)** y seleccione **Siguiente**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-2.png" alt-text="Captura de pantalla Visual Studio, que muestra el cuadro de diálogo de publicación de una función de Azure. En la página de destino específico, la selección es Azure Function App (Windows).":::

En la página **Functions instance** (Instancia de Functions), elija su suscripción. Seleccione el icono del signo más (+) para crear una nueva función.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-3.png" alt-text="Captura de pantalla Visual Studio que muestra el cuadro de diálogo de publicación de una función de Azure. El icono del signo más aparece resaltado.":::

En la ventana **Function App (Windows) - Create new** [Aplicación de funciones (Windows): Crear nueva], rellene los siguientes campos:
* **Name** (Nombre) es el nombre del plan de consumo que Azure va a usar para hospedar la aplicación de Azure Functions. Este también será el nombre de la aplicación de funciones que contiene la función real. Puede elegir un valor único o dejar el sugerido de forma predeterminada.
* Asegúrese de que el valor del campo **Suscripción** coincida con la suscripción que desea usar. 
* Asegúrese de que el valor del campo **Grupos de recursos** es el que desea utilizar.
* En **Plan type** (Tipo de plan), deje **Consumo**.
* Seleccione la **ubicación** del grupo de recursos.
* Cree un nuevo recurso de **Azure Storage** mediante el vínculo **New** (Nuevo). Establezca la ubicación para que coincida con la del grupo de recursos, use los demás valores predeterminados y seleccione **OK** (Aceptar).

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-4.png" alt-text="Captura de pantalla de Visual Studio que muestra la página del cuadro de diálogo de publicación de una función de Azure, con los detalles de una nueva aplicación de funciones rellenados.":::

Seleccione **Crear**.

Una vez creada la instancia de App Service, se abrirá la pestana **Functions instance** (Instancia de Functions). La nueva aplicación de funciones aparece en el área **Function Apps** debajo del grupo de recursos. Seleccione **Finalizar**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-5.png" alt-text="Captura de pantalla de Visual Studio, en la que se muestra el cuadro de diálogo de publicación de una función de Azure con la pestaña Instancia de Functions seleccionada.":::

En el panel **Publish** (Publicar) que se abre en la ventana principal de Visual Studio, compruebe que toda la información sea correcta. Después, seleccione **Publicar**.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-6.png" alt-text="Captura de pantalla Visual Studio que muestra el panel Publish (Publicar). El botón Publicar aparece resaltado.":::

> [!NOTE]
> Si ve una ventana emergente como la del ejemplo siguiente, seleccione **Attempt to retrieve credentials from Azure** (Intentar recuperar credenciales de Azure) y, a continuación, seleccione **Save** (Guardar).
> :::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-7.png" alt-text="Captura de pantalla de Visual Studio, que muestra una ventana emergente denominada Credenciales de publicación." border="false":::
>
> Si ve una de las siguientes advertencias, siga las indicaciones para actualizar el producto a la versión en tiempo de ejecución más reciente Azure Functions:
> * "Upgrade Functions version on Azure" (Actualizar la versión de Functions en Azure).
> * "Your version of the functions runtime does not match the version running in Azure." (La versión del entorno de ejecución de Functions no coincide con la versión que se ejecuta en Azure).
>
> Estas advertencias pueden aparecer si usa una versión anterior de Visual Studio.

La aplicación de funciones ya está publicada en Azure.
