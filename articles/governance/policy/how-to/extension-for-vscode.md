---
title: Extensión de Azure Policy para Visual Studio Code
description: Obtenga información acerca de cómo usar la extensión de Azure Policy para Visual Studio Code para buscar alias de Azure Resource Manager.
ms.date: 04/25/2021
ms.topic: how-to
ms.openlocfilehash: 8fe0d22aeb307f82034d697e8564f8fccab61a81
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108133842"
---
# <a name="use-azure-policy-extension-for-visual-studio-code"></a>Use la extensión de Azure Policy para Visual Studio Code

> Se aplica a la versión de extensión de Azure Policy **0.1.2** y a versiones más recientes.

Aprenda a usar la extensión de Azure Policy para Visual Studio Code y así poder buscar [alias](../concepts/definition-structure.md#aliases), revisar recursos y definiciones de directivas, exportar objetos y evaluar definiciones de directivas. Primero, describiremos cómo instalar la extensión de Azure Policy en Visual Studio Code. Después, veremos cómo buscar alias.

La extensión de Azure Policy para Visual Studio Code se puede instalar en Linux, Mac y Windows.

## <a name="prerequisites"></a>Prerrequisitos

Los elementos siguientes son necesarios para completar los pasos indicados en este artículo:

- Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.
- [Visual Studio Code](https://code.visualstudio.com).

## <a name="install-and-configure-the-azure-policy-extension"></a>Instalación y configuración de la extensión de Azure Policy

Después de completar los requisitos previos, puede instalar la extensión de Azure Policy para Visual Studio Code siguiendo estos pasos:

1. Abra Visual Studio Code.
1. En la barra de menús, vaya a **Vista** > **Extensiones**.
1. En el cuadro de búsqueda, escriba **Azure Policy**.
1. Seleccione **Azure Policy** en los resultados de la búsqueda, luego haga clic en **Instalar**.
1. Seleccione **Recargar** cuando sea necesario.

Para usuarios de la nube nacional, siga estos pasos para configurar el entorno de Azure en primer lugar:

1. Seleccione **Archivo\Preferencias\Configuración**.
1. Busque la siguiente cadena: _Azure: Cloud_ (Nube)
1. Seleccione la nube nacional de la lista:

   :::image type="content" source="../media/extension-for-vscode/set-default-azure-cloud-sign-in.png" alt-text="Captura de pantalla de la selección del inicio de sesión en la nube de Azure nacional para Visual Studio Code." border="false":::

## <a name="using-the-policy-extension"></a>Uso de la extensión de Policy

> [!NOTE]
> Los cambios realizados localmente en las definiciones de directivas que se ven en la extensión de Azure Policy para Visual Studio Code no se sincronizan con Azure.

### <a name="connect-to-an-azure-account"></a>Conexión a la cuenta de Azure

Para evaluar los recursos y los alias de búsqueda, debe conectarse a su cuenta de Azure. Siga estos pasos para conectarse a Azure desde Visual Studio Code:

1. Inicie sesión en Azure desde la extensión de Azure Policy o la paleta de comandos.

   - Extensión de Azure Policy

     En la extensión de Azure Policy, seleccione **Iniciar sesión en Azure**.

     :::image type="content" source="../media/extension-for-vscode/azure-cloud-sign-in-policy-extension.png" alt-text="Captura de pantalla de Visual Studio Code y el icono para la extensión de Azure Policy." border="false":::

   - Paleta de comandos

     En la barra de menús, vaya a **Vista** > **Paleta de comandos** y escriba **Azure: Sign In** (Iniciar sesión).

     :::image type="content" source="../media/extension-for-vscode/azure-cloud-sign-in-command-palette.png" alt-text="Captura de las opciones de inicio de sesión en la nube de Azure para Visual Studio Code desde la paleta de comandos." border="false":::

1. Siga las instrucciones para iniciar sesión en Azure. Una vez establecida la conexión, el nombre de la cuenta de Azure se muestra en la barra de estado en la parte inferior de la ventana de Visual Studio Code.

### <a name="select-subscriptions"></a>Selección de suscripciones

La primera vez que inicia sesión, la extensión de Azure Policy carga los recursos y las definiciones de directivas predeterminados de la suscripción. Para agregar o quitar suscripciones de los recursos y definiciones de directivas que se muestran, siga estos pasos:

1. Inicie el comando de suscripción desde la paleta de comandos o el pie de página de la ventana.

   - Paleta de comandos:

     En la barra de menús, vaya a **Vista** > **Paleta de comandos** y escriba **Azure: Selección de suscripciones**.

   - Pie de página de ventana

     En el pie de página de la ventana, en la parte inferior de la pantalla, seleccione el segmento que coincida con **Azure: \<your account\>** .

1. Utilice el cuadro de filtro para buscar rápidamente las suscripciones por nombre. Luego, active o desactive la casilla de cada suscripción para establecer las suscripciones que se muestran en la extensión de Azure Policy. Cuando haya terminado de agregar o quitar las suscripciones que se van a mostrar, seleccione **Aceptar**.

### <a name="search-for-and-view-resources"></a>Buscar y ver recursos

La extensión de Azure Policy enumera los recursos de las suscripciones seleccionadas por el proveedor de recursos y por el grupo de recursos en el panel **Recursos**. La vista de árbol incluye las siguientes agrupaciones de recursos dentro de la suscripción seleccionada o en el nivel de suscripción:

- **Proveedores de recursos**
  - Cada proveedor de recursos registrado con recursos y recursos secundarios relacionados que tienen alias de directiva
- **Grupos de recursos**
  - Todos los recursos por el grupo de recurso en el que se encuentran

De forma predeterminada, la extensión filtra la parte del proveedor de recursos de los recursos existentes y de los que tienen alias de directiva. Cambie este comportamiento en **Configuración** > **Extensiones** > **Azure Policy** para ver todos los proveedores de recursos sin filtrado.

Los clientes con cientos o miles de recursos en una sola suscripción pueden preferir una forma de búsqueda para localizar sus recursos. La extensión de Azure Policy permite buscar un recurso específico con los pasos siguientes:

1. Empiece la interfaz de búsqueda desde la extensión de Azure Policy o la paleta de comandos.

   - Extensión de Azure Policy

     En la extensión de Azure Policy, mantenga el mouse sobre el panel **Recursos** y seleccione los puntos suspensivos, luego elija **Buscar recursos**.

   - Paleta de comandos:

     En la barra de menús, vaya a **Vista** > **Paleta de comandos** y escriba **Azure Policy: Búsqueda de recursos**.

1. Si se selecciona más de una suscripción para mostrarla, use el filtro para elegir la suscripción que se va a buscar.

1. Use el filtro para elegir el grupo de recursos que se va a buscar que forma parte de la suscripción elegida anteriormente.

1. Use el filtro para seleccionar el recurso que se va a mostrar. El filtro funciona para el nombre del recurso y el tipo de recurso.

### <a name="discover-aliases-for-resource-properties"></a>Detección de alias para propiedades de recursos

Cuando se selecciona un recurso, ya sea mediante la interfaz de búsqueda o al seleccionarlo en la vista de árbol, la extensión de Azure Policy abre el archivo JSON que representa ese recurso y todos sus valores de propiedad de Azure Resource Manager.

Una vez que se abre un recurso, al mantener el mouse sobre el nombre o valor de la propiedad del administrador de recursos, se muestra el alias de Azure Policy, si existe uno. En este ejemplo, el recurso es un tipo de recurso `Microsoft.Compute/virtualMachines` y la propiedad **properties. storageProfile.imageReference.offer** se mantiene sobre ella. Al mantener el mouse se muestran los alias que coinciden.

:::image type="content" source="../media/extension-for-vscode/extension-hover-shows-property-alias.png" alt-text="Captura de pantalla de la extensión Azure Policy para Visual Studio Code al mantener el puntero sobre una propiedad para mostrar los nombres de alias." border="false":::

> [!NOTE]
> La extensión de VS Code solo admite la evaluación de propiedades del modo de Resource Manager. Para más información sobre los modos, consulte las [definiciones de modo](../concepts/definition-structure.md#mode).

### <a name="search-for-and-view-policy-definitions-and-assignments"></a>Búsqueda y visualización de definiciones y asignaciones de directivas

La extensión de Azure Policy enumera los tipos de directivas y las asignaciones de directivas como una vista de árbol para las suscripciones seleccionadas que se mostrarán en el panel **Directivas**. Los clientes con cientos o miles de definiciones de directivas o asignaciones a una sola suscripción pueden preferir una forma de búsqueda para localizar sus direcciones de directivas o asignaciones. La extensión de Azure Policy permite buscar una directiva o asignación específica con los pasos siguientes:

1. Empiece la interfaz de búsqueda desde la extensión de Azure Policy o la paleta de comandos.

   - Extensión de Azure Policy

     En la extensión de Azure Policy, mantenga el mouse sobre el panel **Directivas** y seleccione los puntos suspensivos, luego elija **Buscar directivas**.

   - Paleta de comandos:

     En la barra de menús, vaya a **Vista** > **Paleta de comandos** y escriba **Azure Policy: Búsqueda de directivas**.

1. Si se selecciona más de una suscripción para mostrarla, use el filtro para elegir la suscripción que se va a buscar.

1. Use el filtro para elegir el tipo de directiva o la asignación que se va a buscar y que forma parte de la suscripción elegida anteriormente.

1. Use el filtro para seleccionar la directiva o asignación que se va a mostrar. El filtro funciona para _displayName_ para la definición de directiva o la asignación de directiva.

Cuando se selecciona una asignación o directiva, ya sea a través de la interfaz de búsqueda o al seleccionarla en la vista de árbol, la extensión de Azure Policy abre el archivo JSON que representa esa asignación o directiva y todos sus valores de propiedad de administrador de recursos. La extensión puede validar el esquema JSON abierto de Azure Policy.

### <a name="export-objects"></a>Exportación de objetos

Los objetos de las suscripciones se pueden exportar a un archivo JSON local. En el panel de **Recursos** o **Directivas**, mantenga el mouse sobre el contenido o seleccione un objeto exportable. Al final de la fila resaltada, seleccione el icono de guardar y, a continuación, seleccione una carpeta para guardar los recursos JSON.

Los objetos siguientes se pueden exportar de forma local:

- Panel de recursos
  - Grupos de recursos
  - Recursos individuales (ya sea en un grupo de recursos o en un proveedor de recursos)
- Panel de directivas
  - Asignaciones de directivas
  - Definiciones de directiva integradas
  - Definiciones de directivas personalizadas
  - Iniciativas

### <a name="on-demand-evaluation-scan"></a>Examen de evaluación a petición

Un examen de evaluación se puede iniciar con la extensión de Azure Policy para Visual Studio Code. Para iniciar una evaluación, seleccione y ancle cada uno de los objetos siguientes: un recurso, una definición de directiva y una asignación de directiva.

1. Para anclar cada objeto, selecciónelo en el panel de **Recursos** o en el panel de **Directivas** y seleccione el icono "Pin to an edit tab" (Anclar a una pestaña de edición). Al anclar un objeto, éste se agrega al panel de **Evaluación** de la extensión.
1. En el panel de **Evaluación**, seleccione uno de cada objeto y use el icono "Select for evaluation" (Seleccionar para su evaluación) y así poder indicar que está incluido en la evaluación.
1. En la parte superior del panel **Evaluación**, seleccione el icono "Run evaluation" (Ejecutar evaluación). Se abrirá un nuevo panel en Visual Studio Code con los detalles de evaluación resultantes en formato JSON.

> [!NOTE]
> En el caso de las definiciones de directivas [AuditIfNotExists](../concepts/effects.md#auditifnotexists) o [DeployIfNotExists](../concepts/effects.md#deployifnotexists), use el icono con el signo "más" en el panel **Evaluación** o en **Azure Policy: Seleccionar un recurso para la comprobación de la existencia (solo se usa para directivas de tipo "if-not-exists")** de la Paleta de comandos para seleccionar un recurso _relacionado_ con la comprobación de la existencia.

Los resultados de la evaluación proporcionan información sobre la definición y asignación de directivas junto con la propiedad **policyEvaluations.evaluationResult**. El resultado es similar al ejemplo siguiente:

```json
{
    "policyEvaluations": [
        {
            "policyInfo": {
                ...
            },
            "evaluationResult": "Compliant",
            "effectDetails": {
                "policyEffect": "Audit",
                "existenceScope": "None"
            }
        }
    ]
}
```

> [!NOTE]
> La extensión de VS Code solo admite la evaluación de propiedades del modo de Resource Manager. Para más información sobre los modos, consulte las [definiciones de modo](../concepts/definition-structure.md#mode).
>
> La característica de evaluación no funciona en instalaciones de macOS y Linux de la extensión.

### <a name="sign-out"></a>Cerrar sesión

En la barra de menús, vaya a **Vista** > **Paleta de comandos** y escriba **Azure: Sign Out** (Cerrar sesión).

## <a name="next-steps"></a>Pasos siguientes

- Puede consultar ejemplos en [Ejemplos de Azure Policy](../samples/index.md).
- Revise la [estructura de definición de Azure Policy](../concepts/definition-structure.md).
- Vea la [Descripción de los efectos de directivas](../concepts/effects.md).
- Obtenga información acerca de [cómo se pueden crear definiciones de directivas mediante programación](programmatically-create.md).
- Obtenga información sobre cómo [corregir recursos no compatibles](remediate-resources.md).
- En [Organización de los recursos con grupos de administración de Azure](../../management-groups/overview.md), obtendrá información sobre lo que es un grupo de administración.
