---
author: baanders
description: Archivo de inclusión del proceso de publicación de una función de Azure desde Visual Studio
ms.service: digital-twins
ms.topic: include
ms.date: 1/21/2021
ms.author: baanders
ms.openlocfilehash: 9f8ca9ab6dc184abb8d9dcd25ebe87cbd7761722
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "102630278"
---
Para publicar el proyecto en una aplicación de funciones en Azure, haga clic con el botón derecho en el proyecto en el *Explorador de soluciones* y elija **Publicar**.

> [!IMPORTANT] 
> La publicación de una aplicación de funciones en Azure incurrirá en cargos adicionales por su suscripción, independientemente de Azure Digital Twins.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-1.png" alt-text="Captura de pantalla de Visual Studio que muestra el menú de la solución de la selección con el botón derecho. La publicación se resalta en el menú.":::

En la página *Publicar* que aparece a continuación, deje la selección de destino predeterminada de **Azure** y seleccione *Siguiente*. 

Para seleccionar un destino concreto, elija **Azure Function App (Windows)** y seleccione *Siguiente*.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-2.png" alt-text="Captura de pantalla de Visual Studio: cuadro de diálogo Publicar una función de Azure. Azure Function App (Windows) está seleccionado en la página Destino específico.":::

En la página *Functions instance* (Instancia de Functions), elija su suscripción. Después, seleccione el icono *+* para crear una función de Azure.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-3.png" alt-text="Captura de pantalla de Visual Studio: cuadro de diálogo Publicar una función de Azure. El botón + para crear una nueva función aparece resaltado en la página Functions instance (Instancia de Functions).":::

En la ventana *Function App (Windows) - Create new* [Aplicación de funciones (Windows): Crear nueva], rellene los siguientes campos:
* **Name** (Nombre) es el nombre del plan de consumo que Azure va a usar para hospedar la aplicación de Azure Functions. También será el nombre de la aplicación de funciones que contiene la función real. Puede elegir su propio valor o dejar el que se sugiere, que es el predeterminado.
* Asegúrese de que el valor del campo **Subscription** (Suscripción) coincida con la suscripción que desea usar. 
* Asegúrese de que el valor del campo **Resource group** (Grupos de recursos) coincida con el grupo de recursos que desea utilizar.
* En **Plan type** (Tipo de plan), deje *Consumption* (Consumo).
* En el campo **Location** (Ubicación), elija la ubicación del grupo de recursos.
* Cree un recurso de **Azure Storage** mediante el vínculo *New...* (Nuevo...). Establezca la ubicación que coincida con la del grupo de recursos, use los restantes valores predeterminados y seleccione "Ok" (Aceptar).

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-4.png" alt-text="Captura de pantalla de Visual Studio: cuadro de diálogo Publicar una función de Azure. Se están rellenando los detalles de una nueva aplicación de funciones, incluidos el nombre, suscripción, grupo de recursos, tipo de plan, ubicación e instancia de Azure Storage.":::

Seleccione **Crear**.

Después de una breve espera mientras se crea App Service, el cuadro de diálogo debe volver a la página *Functions instance* (Instancia de Functions), con la nueva aplicación de funciones que aparece en el área **Function Apps** (Aplicaciones de funciones) anidada debajo del grupo de recursos. Seleccione *Finalizar*.

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-5.png" alt-text="Publicar una función de Azure en Visual Studio: instancia de Functions (después de la aplicación de funciones)":::

En el panel *Publish* (Publicar) que se abre en la ventana principal de Visual Studio, compruebe que toda la información parece correcta y seleccione **Publish** (Publicar).

:::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-6.png" alt-text="Captura de pantalla de Visual Studio: cuadro de diálogo Publicar una función de Azure. La nueva aplicación de funciones aparece en la lista de aplicaciones de funciones y hay un botón Finalizar.":::

> [!NOTE]
> Si ve un elemento emergente similar al siguiente: :::image type="content" source="../articles/digital-twins/media/tutorial-end-to-end/publish-azure-function-7.png" alt-text="Captura de pantalla de la ventana emergente de Visual Studio denominada Credenciales de publicación. Contiene campos para un nombre de usuario y una contraseña, y un botón para intentar recuperar las credenciales de Azure." border="false":::
> Seleccione **Attempt to retrieve credentials from Azure** (Intentar recuperar credenciales de Azure) y **Save** (Guardar).
>
> Si ve una advertencia en la que se indica que *actualice la versión de Functions en Azure* o que *la versión del entorno de ejecución de Functions no coincide con la versión que se ejecuta en Azure*:
>
> siga las indicaciones para actualizar a la versión más reciente del entorno de ejecución de Azure Functions. Este problema puede producirse si utiliza una versión anterior de Visual Studio.

La aplicación de funciones ya está publicada en Azure.