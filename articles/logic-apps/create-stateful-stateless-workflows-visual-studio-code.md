---
title: Creación de flujos de trabajo de la versión preliminar de Logic Apps en Visual Studio Code
description: Cree y ejecute flujos de trabajo para los escenarios de automatización e integración en Visual Studio Code con la extensión Azure Logic Apps (versión preliminar).
services: logic-apps
ms.suite: integration
ms.reviewer: estfan, logicappspm, az-logic-apps-dev
ms.topic: conceptual
ms.date: 03/30/2021
ms.openlocfilehash: 491d5f14cc8f456d228a5bc6efaa6686575979c1
ms.sourcegitcommit: 3ee3045f6106175e59d1bd279130f4933456d5ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106078747"
---
# <a name="create-stateful-and-stateless-workflows-in-visual-studio-code-with-the-azure-logic-apps-preview-extension"></a>Creación de flujos de trabajo con o sin estado en Visual Studio Code con la extensión Azure Logic Apps (versión preliminar)

> [!IMPORTANT]
> Esta funcionalidad se encuentra en versión preliminar pública, se ofrece sin contrato de nivel de servicio y no se recomienda usarla para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Con la [versión preliminar de Azure Logic Apps](logic-apps-overview-preview.md), puede crear soluciones de automatización e integración en aplicaciones, datos, servicios en la nube y sistemas mediante la creación y ejecución de aplicaciones lógicas que incluyen [flujos de trabajo *con estado* y *sin estado*](logic-apps-overview-preview.md#stateful-stateless) en Visual Studio Code con la extensión Azure Logic Apps (versión preliminar). Con este nuevo tipo de aplicación lógica, puede compilar varios flujos de trabajo con la tecnología del runtime rediseñado de la versión preliminar de Azure Logic Apps, que proporciona portabilidad, mejor rendimiento y flexibilidad a la hora de implementar y ejecutar en diversos entornos de hospedaje, no solo Azure, sino también contenedores de Docker. Para obtener más información sobre el nuevo tipo de aplicación lógica, consulte [Introducción a la versión preliminar de Azure Logic Apps](logic-apps-overview-preview.md).

![Captura de pantalla que muestra Visual Studio Code, el proyecto de aplicación lógica y el flujo de trabajo.](./media/create-stateful-stateless-workflows-visual-studio-code/visual-studio-code-logic-apps-overview.png)

En Visual Studio Code, puede empezar por crear un proyecto en el que pueda compilar y ejecutar *localmente* los flujos de trabajo de la aplicación lógica en el entorno de desarrollo mediante la extensión Azure Logic Apps (versión preliminar). Aunque también puede empezar por [crear un recurso **Logic App (versión preliminar)** en Azure Portal](create-stateful-stateless-workflows-azure-portal.md), ambos enfoques proporcionan la capacidad de implementar y ejecutar la aplicación lógica en los mismos tipos de entornos de hospedaje.

Mientras tanto, todavía puede crear el tipo de aplicación lógica original. Aunque las experiencias de desarrollo en Visual Studio Code difieren entre los tipos de aplicaciones lógicas originales y nuevas, la suscripción de Azure puede incluir ambos tipos. Puede ver todas las aplicaciones lógicas implementadas en la suscripción de Azure, y acceder a ellas, pero se organizan en sus propias categorías y secciones.

En este artículo se muestra cómo crear la aplicación lógica y un flujo de trabajo en Visual Studio Code mediante la extensión Azure Logic Apps (versión preliminar) y realizar las siguientes tareas de alto nivel:

* Crear un proyecto para la aplicación lógica y el flujo de trabajo.

* Agregar un desencadenador y una acción.

* Ejecutar, probar, depurar y revisar el historial de ejecución de forma local.

* Buscar los detalles del nombre de dominio para el acceso al firewall.

* Implementar en Azure, que incluye opcionalmente habilitar Application Insights.

* Administrar la aplicación lógica implementada en Visual Studio Code y Azure Portal.

* Habilitar el historial de ejecución para flujos de trabajo sin estado.

* Habilitar o abrir Application Insights después de la implementación.

* Implementar en un contenedor de Docker que pueda ejecutar en cualquier lugar.

> [!NOTE]
> Para obtener información sobre los problemas conocidos actuales, vea la [página de problemas conocidos de la versión preliminar pública de Logic Apps en GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md).

## <a name="prerequisites"></a>Requisitos previos

### <a name="access-and-connectivity"></a>Acceso y conectividad

* Acceda a Internet para poder descargar los requisitos, conectarse desde Visual Studio Code a su cuenta de Azure y publicar desde Visual Studio Code en Azure, un contenedor de Docker u otro entorno.

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Para compilar la misma aplicación lógica de ejemplo en este artículo, necesita una cuenta de correo electrónico de Office 365 Outlook que use una cuenta profesional o educativa de Microsoft para iniciar sesión.

  Si decide usar otro [conector de correo electrónico que sea compatible con Azure Logic Apps](/connectors/), como Outlook.com o [Gmail](../connectors/connectors-google-data-security-privacy-policy.md), aún puede seguir el ejemplo, y los pasos generales son los mismos, pero la interfaz de usuario y las opciones pueden diferir de alguna manera. Por ejemplo, si usa el conector de Outlook.com, en su lugar use su cuenta personal de Microsoft para iniciar sesión.

<a name="storage-requirements"></a>

### <a name="storage-requirements"></a>Requisitos de almacenamiento

#### <a name="windows"></a>Windows

Para compilar y ejecutar localmente el proyecto de aplicación lógica en Visual Studio Code cuando use Windows, siga estos pasos para configurar el Emulador de Azure Storage:

1. Descargue e instale el [Emulador de Azure Storage 5.10](https://go.microsoft.com/fwlink/p/?linkid=717179).

1. Para que el emulador pueda ejecutarse, es preciso tener una instalación de base de SQL local, como [SQL Server 2019 Express Edition](https://go.microsoft.com/fwlink/p/?linkid=866658), que es gratuita.

   Para más información, consulte [Uso del emulador de Azure Storage para desarrollo y pruebas](../storage/common/storage-use-emulator.md).

1. Para poder ejecutar el proyecto, asegúrese de iniciar el emulador.

   ![Captura de pantalla que muestra el emulador de Azure Storage en ejecución.](./media/create-stateful-stateless-workflows-visual-studio-code/start-storage-emulator.png)

#### <a name="macos-and-linux"></a>macOS y Linux.

Para compilar y ejecutar localmente el proyecto de aplicación lógica en Visual Studio Code cuando se usan macOS o Linux, siga estos pasos para crear y configurar una cuenta de Azure Storage.

> [!NOTE]
> Actualmente, el diseñador de Visual Studio Code no funciona en el sistema operativo Linux, pero a pesar de ello puede compilar, ejecutar e implementar aplicaciones lógicas que usan el entorno en tiempo de ejecución de la versión preliminar de Logic Apps en máquinas virtuales basadas en Linux. Por ahora, puede compilar las aplicaciones lógicas en Visual Studio Code en Windows o macOS y, luego, implementarlas en una máquina virtual basada en Linux.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y [cree una cuenta de Azure Storage](../storage/common/storage-account-create.md?tabs=azure-portal), ya que es un [requisito previo para Azure Functions](../azure-functions/storage-considerations.md).

1. En el menú de la cuenta de almacenamiento, en **Configuración**, seleccione **Claves de acceso**.

1. En el panel **Claves de acceso**, busque y copie la cadena de conexión de la cuenta de almacenamiento, que es similar a la de este ejemplo:

   `DefaultEndpointsProtocol=https;AccountName=fabrikamstorageacct;AccountKey=<access-key>;EndpointSuffix=core.windows.net`

   ![Captura de pantalla que muestra Azure Portal con las claves de acceso de la cuenta de almacenamiento y la cadena de conexión copiada.](./media/create-stateful-stateless-workflows-visual-studio-code/find-storage-account-connection-string.png)

   Para más información, consulte [Administración de claves de cuenta de almacenamiento](../storage/common/storage-account-keys-manage.md?tabs=azure-portal#view-account-access-keys).

1. Guarde la cadena de conexión en un lugar seguro. Después de crear el proyecto de aplicación lógica en Visual Studio Code, tendrá que agregar la cadena al archivo **local.settings.json** de la carpeta de nivel raíz del proyecto.

   > [!IMPORTANT]
   > Si planea realizar la implementación en un contenedor de Docker, también debe usar esta cadena de conexión con el archivo de Docker que se usa para la implementación. En escenarios de producción, asegúrese de proteger estos secretos e información confidencial, por ejemplo, mediante el uso de un almacén de claves.
  
### <a name="tools"></a>Herramientas

* [Visual Studio Code 1.30.1 (enero de 2019) o superior](https://code.visualstudio.com/), que es gratis. Además, descargue e instale estas herramientas para Visual Studio Code, en caso de que aún no las tenga:

  * [Extensión de cuenta de Azure](https://marketplace.visualstudio.com/items?itemName=ms-vscode.azure-account), que proporciona una experiencia común única de filtrado de suscripción e inicio de sesión de Azure para todas las demás extensiones de Azure en Visual Studio Code.

  * [Extensión de C# para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-vscode.csharp), que habilita la funcionalidad F5 para ejecutar la aplicación lógica.

  * [Azure Functions Core Tools 3.0.3245, o cualquier versión posterior](https://github.com/Azure/azure-functions-core-tools/releases/tag/3.0.3245), mediante la versión de Microsoft Installer (MSI), que es la `func-cli-3.0.3245-x*.msi`.

    Estas herramientas incluyen una versión del mismo runtime en el que se basa el runtime de Azure Functions, que la extensión de versión preliminar usa en Visual Studio Code.

    > [!IMPORTANT]
    > Si tiene una instalación anterior a estas versiones, desinstale primero esa versión o asegúrese de que la variable de entorno PATH apunta a la versión que descargue e instale.

  * [Extensión Azure Logic Apps (versión preliminar) para Visual Studio Code](https://go.microsoft.com/fwlink/p/?linkid=2143167). Esta extensión proporciona la capacidad de crear aplicaciones lógicas en las que puede compilar flujos de trabajo con estado y sin estado que se ejecutan localmente en Visual Studio Code y, a continuación, implementar esas aplicaciones lógicas directamente en Azure o en contenedores de Docker.

    Actualmente, puede tener ambas extensiones, Azure Logic Apps original y la versión preliminar pública, instaladas en Visual Studio Code. Aunque las experiencias de desarrollo difieren en algunos métodos entre las extensiones, la suscripción de Azure puede incluir los dos tipos de aplicación lógica que se crean con las extensiones. Visual Studio Code muestra todas las aplicaciones lógicas implementadas en la suscripción de Azure, pero las organiza en secciones diferentes por nombres de extensión, **Logic Apps** y **Azure Logic Apps (versión preliminar)** .

    > [!IMPORTANT]
    > Si creó proyectos de aplicaciones lógicas con la extensión de versión preliminar privada, dichos proyectos no funcionarán con la extensión de versión preliminar pública. Sin embargo, se pueden migrar después de desinstalar la extensión de versión preliminar privada, eliminar los archivos asociados e instalar la extensión de versión preliminar pública. A continuación, puede crear un nuevo proyecto en Visual Studio Code y copiar en el nuevo proyecto el archivo **workflow.definition** de la aplicación lógica creada anteriormente. Para más información, consulte [Migración desde la extensión de versión preliminar privada](#migrate-private-preview).
    > 
    > Si creó proyectos de aplicaciones lógicas con la extensión de versión preliminar pública anterior, puede seguir usándolos sin hacer nada más.

    **Para instalar la extensión **Azure Logic Apps (versión preliminar)** , siga estos pasos:**

    1. En Visual Studio Code, en la barra de herramientas izquierda, seleccione **Extensiones**.

    1. En el cuadro de búsqueda de extensiones, escriba `azure logic apps preview`. En la lista de resultados, seleccione **Azure Logic Apps (Preview)** (Azure Logic Apps [versión preliminar]) **>** **Instalar**.

       Una vez finalizada la instalación, la extensión de versión preliminar aparece en la lista **Extensiones: Instaladas**.

       ![Captura de pantalla que muestra la lista de extensiones instaladas de Visual Studio Code con la extensión "Azure Logic Apps (versión preliminar)" subrayada.](./media/create-stateful-stateless-workflows-visual-studio-code/azure-logic-apps-extension-installed.png)

       > [!TIP]
       > Si la extensión no aparece en la lista de instaladas, pruebe a reiniciar Visual Studio Code.

* Para usar la [acción de Operaciones de código en línea](../logic-apps/logic-apps-add-run-inline-code.md) que ejecuta JavaScript, instale las [versiones 10.x.x, 11.x.x o 12.x.x de Node.js](https://nodejs.org/en/download/releases/).

  > [!TIP] 
  > Para Windows, descargue la versión de MSI. Si usa la versión ZIP, tendrá que hacer que Node.js esté disponible manualmente. Para ello, será preciso que use una variable de entorno PATH para su sistema operativo.

* Para ejecutar localmente las acciones y los desencadenadores basados en webhook, como el [desencadenador de webhook integrado de HTTP](../connectors/connectors-native-webhook.md), en Visual Studio Code, debe [configurar el reenvío de la dirección URL de devolución de llamada](#webhook-setup).

* Para probar la aplicación lógica de ejemplo que se crea en este artículo, necesita una herramienta que pueda enviar llamadas al desencadenador Solicitud, que es el primer paso de la aplicación lógica de ejemplo. Si no dispone de esta herramienta, puede descargar, instalar y usar [Postman](https://www.postman.com/downloads/).

* Si crea una aplicación lógica y la implementa con una configuración que admite el uso de [Application Insights](../azure-monitor/app/app-insights-overview.md), puede habilitar opcionalmente el registro de diagnóstico y el seguimiento para la aplicación lógica. Puede hacerlo al implementar la aplicación lógica desde Visual Studio Code o después de la implementación. Debe tener una instancia de Application Insights, pero puede crear este recurso [con antelación](../azure-monitor/app/create-workspace-resource.md), al implementar la aplicación lógica o después de la implementación.

<a name="migrate-private-preview"></a>

## <a name="migrate-from-private-preview-extension"></a>Migración desde la extensión de versión preliminar privada

Los proyectos de aplicación lógica creados con la extensión **Azure Logic Apps (versión preliminar privada)** no funcionarán con la extensión de versión preliminar pública. Sin embargo, se pueden migrar a proyectos nuevos, Para ello, debe seguir estos pasos:

1. Desinstale la extensión de versión preliminar privada.

1. Elimine las carpetas asociadas de agrupaciones de extensiones y de paquetes NuGet en estas ubicaciones:

   * La carpeta **Microsoft.Azure.Functions.ExtensionBundle.Workflows**, que contiene los conjuntos de extensiones anteriores y se encuentra en una de estas rutas de acceso:

     * `C:\Users\{userName}\AppData\Local\Temp\Functions\ExtensionBundles`

     * `C:\Users\{userName}\.azure-functions-core-tools\Functions\ExtensionBundles`

   * La carpeta **microsoft.azure.workflows.webjobs.extension**, que es la memoria caché de [NuGet](/nuget/what-is-nuget) para la extensión de versión preliminar privada y se encuentra en esta ruta de acceso:

     `C:\Users\{userName}\.nuget\packages`

1. Instale la extensión **Azure Logic Apps (versión preliminar)** .

1. Cree un nuevo proyecto en Visual Studio Code.

1. Copie en el nuevo proyecto el archivo **workflow.definition** de la aplicación lógica que creó anteriormente.

<a name="set-up"></a>

## <a name="set-up-visual-studio-code"></a>Configurar Visual Studio Code

1. Para asegurarse de que todas las extensiones estén instaladas correctamente, vuelva a cargar o reinicie Visual Studio Code.

1. Confirme que Visual Studio Code debe encontrar e instalar automáticamente las actualizaciones de extensión para que la extensión de versión preliminar obtenga las actualizaciones más recientes. De lo contrario, tendrá que desinstalar manualmente la versión obsoleta e instalar la versión más reciente.

   1. En el menú **Archivo**, vaya a **Preferencias** **>** **Configuración**.

   1. En la pestaña **Usuario**, vaya a **Características** **>** **Extensiones**.

   1. Confirme que las opciones **Auto Check Updates** (Comprobar actualizaciones automáticamente) y **Actualización automática** estén seleccionadas.

Además, de forma predeterminada, la siguiente configuración está habilitada y configurada para la extensión de versión preliminar de Logic Apps:

* **Azure Logic Apps V2: Project Runtime**, que está establecido en la versión  **~3**

  > [!NOTE]
  > Esta versión es necesaria para utilizar las [acciones de Operaciones de código en línea](../logic-apps/logic-apps-add-run-inline-code.md).

* **Azure Logic Apps V2: Experimental View Manager**, que habilita el diseñador más reciente en Visual Studio Code. Si tiene problemas en el diseñador, como realizar las operaciones de arrastrar y colocar elementos, desactive este valor.

Para buscar y confirmar esta configuración, siga estos pasos:

1. En el menú **Archivo**, vaya a **Preferencias** **>** **Configuración**.

1. En la pestaña **Usuario**, vaya a **>** **Extensiones** **>** **Azure Logic Apps (versión preliminar)** .

   Por ejemplo, el valor **Azure Logic Apps V2: Project Runtime** se puede buscar aquí, pero también se puede usar el cuadro de búsqueda para buscar otros valores:

   ![Captura de pantalla que muestra la configuración de Visual Studio Code para la extensión "Azure Logic Apps (versión preliminar)".](./media/create-stateful-stateless-workflows-visual-studio-code/azure-logic-apps-preview-settings.png)

<a name="connect-azure-account"></a>

## <a name="connect-to-your-azure-account"></a>Conexión a la cuenta de Azure

1. En la barra de actividad de Visual Studio Code, seleccione el icono de Azure.

   ![Captura de pantalla que muestra la barra de actividad de Visual Studio Code y el icono de Azure seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/visual-studio-code-azure-icon.png)

1. En el panel de Azure, en **Azure: Logic Apps (versión preliminar)** , seleccione **Iniciar sesión en Azure**. Cuando aparezca la página de autenticación de Visual Studio Code, inicie sesión con su cuenta de Azure.

   ![Captura de pantalla que muestra el panel de Azure y el vínculo seleccionado para el inicio de sesión de Azure.](./media/create-stateful-stateless-workflows-visual-studio-code/sign-in-azure-subscription.png)

   Después de iniciar sesión, el panel de Azure muestra las suscripciones de la cuenta de Azure. Si también tiene la extensión pública, puede encontrar cualquier aplicación lógica que haya creado con esa extensión en la sección **Logic Apps**, no en la sección **Logic Apps (versión preliminar)** .
   
   Si no aparecen las suscripciones esperadas o si desea que el panel muestre solo suscripciones específicas, siga estos pasos:

   1. En la lista de suscripciones, mueva el puntero situado junto a la primera suscripción hasta que aparezca el botón **Seleccionar suscripciones** (icono de filtro). Seleccione el icono de filtro.

      ![Captura de pantalla que muestra el panel de Azure y el icono de filtro seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/filter-subscription-list.png)

      O bien, en la barra de estado de Visual Studio Code, seleccione la cuenta de Azure. 

   1. Cuando aparezca otra lista suscripciones, seleccione las suscripciones que desee y, a continuación, asegúrese de seleccionar **Aceptar**.

<a name="create-project"></a>

## <a name="create-a-local-project"></a>Creación de un proyecto local

Para poder crear una aplicación lógica, antes debe crear un proyecto local, con el fin de que pueda administrar, ejecutar e implementar la aplicación lógica desde Visual Studio Code. El proyecto subyacente es similar a un proyecto de Azure Functions, también conocido como proyecto de aplicación de funciones; pero estos tipos de proyecto son independientes entre sí, por lo que las aplicaciones lógicas y las aplicaciones de función no pueden existir en el mismo proyecto.

1. En el equipo, cree una carpeta local *vacía* que se usará para el proyecto que se creará posteriormente en Visual Studio Code.

1. En Visual Studio Code, cierre todas las carpetas abiertas.

1. En el panel de Azure, junto a **Azure: Logic Apps (versión preliminar)** , seleccione **Crear proyecto** (icono que muestra una carpeta y un rayo).

   ![Captura de pantalla que muestra la barra de herramientas del panel de Azure con la opción "Crear proyecto" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/create-new-project-folder.png)

1. Si el Firewall de Windows Defender le pide que conceda acceso a la red para `Code.exe`, que es Visual Studio Code y, para `func.exe`, que es el Azure Functions Core Tools, seleccione **Redes privadas, como una red doméstica o del trabajo** **>** **Permitir acceso**.

1. Vaya a la ubicación en la que creó la carpeta del proyecto, seleccione esa carpeta y continúe.

   ![Captura de pantalla que muestra el cuadro de diálogo "Seleccionar carpeta" con una carpeta de proyecto recién creada y el botón "Seleccionar" seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/select-project-folder.png)

1. En la lista de plantillas que aparece, seleccione **Stateful Workflow** (Flujo de trabajo con estado) o **Stateless Workflow** (Flujo de trabajo sin estado). En este ejemplo se selecciona **Stateful Workflow** (Flujo de trabajo con estado).

   ![Captura de pantalla que muestra la lista de plantillas de flujo de trabajo con "Flujo de trabajo con estado" seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/select-stateful-stateless-workflow.png)

1. Proporcione un nombre para el flujo de trabajo y presione ENTRAR. En este ejemplo se usa `Fabrikam-Stateful-Workflow` como nombre.

   ![Captura de pantalla que muestra el cuadro "Create a new Stateful Workflow (3/4)" (Crear nuevo flujo de trabajo con estado [3/4]) y "Fabrikam-Stateful-Workflow" como nombre del flujo de trabajo.](./media/create-stateful-stateless-workflows-visual-studio-code/name-your-workflow.png)

   Visual Studio Code termina de crear el proyecto y abre el archivo **workflow.json** del flujo de trabajo en el editor de código.

   > [!NOTE]
   > Si se le pide que seleccione cómo se debe abrir el proyecto, seleccione **Open in current window** (Abrir en la ventana actual) si desea abrir el proyecto en la ventana actual de Visual Studio Code. Para abrir una nueva instancia de Visual Studio Code, seleccione **Open in new window** (Abrir en ventana nueva).

1. En la barra de herramientas de Visual Studio, abra el panel del explorador, si aún no está abierto.

   En el panel del explorador se muestra el proyecto, que ahora incluye los archivos de proyecto generados automáticamente. Por ejemplo, el proyecto tiene una carpeta que muestra el nombre del flujo de trabajo. Dentro de esta carpeta, el archivo **workflow.json** contiene la definición JSON subyacente del flujo de trabajo.

   ![Captura de pantalla que muestra el panel del explorador con la carpeta del proyecto, la carpeta del flujo de trabajo y el archivo "workflow.json".](./media/create-stateful-stateless-workflows-visual-studio-code/local-project-created.png)

1. Si usa macOS o Linux, configure el acceso a la cuenta de almacenamiento siguiendo estos pasos, que son necesarios para ejecutar el proyecto en un entorno local:

   1. En la carpeta raíz del proyecto, abra el archivo **local.settings.json**.

      ![Captura de pantalla que muestra el panel del explorador y el archivo "local.settings.json" en el proyecto.](./media/create-stateful-stateless-workflows-visual-studio-code/local-settings-json-files.png)

   1. Reemplace el valor de la propiedad `AzureWebJobsStorage` por la cadena de conexión de la cuenta de almacenamiento que guardó anteriormente, por ejemplo:

      Antes:

      ```json
      {
         "IsEncrypted": false,
         "Values": {
            "AzureWebJobsStorage": "UseDevelopmentStorage=true",
            "FUNCTIONS_WORKER_RUNTIME": "dotnet"
          }
      }
      ```

      Después:

      ```json
      {
         "IsEncrypted": false,
         "Values": {
            "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=fabrikamstorageacct;AccountKey=<access-key>;EndpointSuffix=core.windows.net",
           "FUNCTIONS_WORKER_RUNTIME": "dotnet"
         }
      }
      ```

      > [!IMPORTANT]
      > En escenarios de producción, asegúrese de proteger estos secretos e información confidencial, por ejemplo, mediante el uso de un almacén de claves.

   1. Cuando haya terminado, asegúrese de guardar los cambios.

<a name="enable-built-in-connector-authoring"></a>

## <a name="enable-built-in-connector-authoring"></a>Habilitación de la creación de conectores integrados

Puede crear sus propios conectores integrados para cualquier servicio que necesite mediante el [marco de extensibilidad de la versión preliminar](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-built-in-connector/ba-p/1921272). De forma similar a los conectores integrados, como Azure Service Bus y SQL Server, estos conectores proporcionan mayor rendimiento, baja latencia, conectividad local y se ejecutan de forma nativa en el mismo proceso que el entorno de ejecución de la versión preliminar.

La funcionalidad de creación solo está disponible actualmente en Visual Studio Code, pero no está habilitada de manera predeterminada. Para crear estos conectores, antes debe convertir el proyecto de una extensión basada en agrupación (Node.js) a una basada en paquetes NuGet (.NET).

> [!IMPORTANT]
> Esta acción es una operación unidireccional que no se puede deshacer.

1. En el panel Explorador, en la raíz del proyecto, mueva el puntero del ratón sobre cualquier área en blanco debajo de todos los demás archivos y carpetas, abra el menú contextual y seleccione **Convert to Nuget-based Logic App project** (Convertir en proyecto de aplicación lógica basada en Nuget).

   ![Captura de pantalla que muestra el panel Explorador con el menú contextual del proyecto abierto desde un área en blanco en la ventana de proyecto.](./media/create-stateful-stateless-workflows-visual-studio-code/convert-logic-app-project.png)

1. Cuando se le pida, confirme la conversión del proyecto.

1. Para continuar, revise y siga los pasos descritos en el artículo [Azure Logic Apps en ejecución en cualquier ubicación: extensibilidad de conectores integrada](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-built-in-connector/ba-p/1921272).

<a name="open-workflow-definition-designer"></a>

## <a name="open-the-workflow-definition-file-in-the-designer"></a>Apertura del archivo de definición del flujo de trabajo en el diseñador

1. Compruebe las versiones que ha instalado en el equipo ejecutando este comando:

   `..\Users\{yourUserName}\dotnet --list-sdks`

   Si tiene el SDK de .NET Core 5.x, es posible que esta versión le impida abrir la definición de flujo de trabajo subyacente de la aplicación lógica en el diseñador. En lugar de desinstalar esta versión, en la carpeta raíz del proyecto, cree un archivo **global.json** que haga referencia a la versión 3.x del runtime de .NET Core que tiene, que sea superior a la 3.1.201, por ejemplo:

   ```json
   {
      "sdk": {
         "version": "3.1.8",
         "rollForward": "disable"
      }
   }
   ```

   > [!IMPORTANT]
   > Asegúrese de que agrega de forma explícita el archivo **global.json** en la carpeta raíz del proyecto desde Visual Studio Code, ya que, de lo contrario, el diseñador no se abrirá.

1. Expanda la carpeta del proyecto para el flujo de trabajo. Abra el menú contextual del archivo **workflow.json** y seleccione **Open in Designer** (Abrir en el diseñador).

   ![Captura de pantalla que muestra el panel del Explorador y la ventana contextual del archivo workflow.json con la opción "Abrir en el diseñador" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/open-definition-file-in-designer.png)

1. En la lista **Enable connectors in Azure** (Habilitar conectores en Azure), seleccione la opción **Use connectors from Azure** (Usar conectores de Azure) que se aplica a todos los conectores administrados que están disponibles e implementados en Azure, no solo a los conectores de los servicios de Azure.

   ![Captura de pantalla que muestra el panel del Explorador con la lista "Habilitar conectores en Azure" abierta y  la opción "Usar conectores de Azure" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/use-connectors-from-azure.png)

   > [!NOTE]
   > Actualmente, los flujos de trabajo sin estado solo admiten *acciones* de [conectores administrados](../connectors/apis-list.md#managed-api-connectors) que se implementan en Azure, y no desencadenadores. Aunque tiene la opción de habilitar conectores en Azure para el flujo de trabajo sin estado, el diseñador no muestra ningún desencadenador de conector administrado para que pueda seleccionarlo.

1. En la lista **Seleccionar suscripción**, seleccione la suscripción de Azure que se usará para el proyecto de aplicación lógica.

   ![Captura de pantalla que muestra el panel del explorador con el cuadro "Seleccionar suscripción" y la suscripción seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/select-azure-subscription.png)

1. En la lista del grupo de recursos, seleccione **Crear un nuevo grupo de recursos**.

   ![Captura de pantalla que muestra el panel del explorador con la lista de grupos de recursos y la opción "Crear un nuevo grupo de recursos" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/create-select-resource-group.png)

1. Proporcione un nombre para el grupo de recursos y presione ENTRAR. En este ejemplo se usa `Fabrikam-Workflows-RG`.

   ![Captura de pantalla que muestra el panel del Explorador y el cuadro de nombre del grupo de recursos.](./media/create-stateful-stateless-workflows-visual-studio-code/enter-name-for-resource-group.png)

1. En la lista de ubicaciones, busque y seleccione la región de Azure que se usará al crear el grupo de recursos y los recursos. En este ejemplo se usa **Centro-oeste de EE. UU.**

   ![Captura de pantalla que muestra el panel del Explorador con la lista de ubicaciones y la opción "Centro-oeste de EE. UU." seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/select-azure-region.png)

   Después de realizar este paso, Visual Studio Code abre el diseñador de flujos de trabajo.

   > [!NOTE]
   > Cuando Visual Studio Code inicia la API en tiempo de diseño del flujo de trabajo, puede que reciba un mensaje que indica que el inicio puede tardar unos segundos. Puede pasar por alto este mensaje o seleccionar **Aceptar**.
   >
   > Si el diseñador no se abre, consulte en la sección de solución de problemas, el apartado en que se indica qué hacer si el [diseñador no se abre](#designer-fails-to-open).

   Después de que aparezca el diseñador, aparece la solicitud **Elija una operación** en el diseñador y está seleccionada de manera predeterminada, lo que muestra el panel **Agregar una acción**.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo.](./media/create-stateful-stateless-workflows-visual-studio-code/workflow-app-designer.png)

1. A continuación, [agregue un desencadenador y acciones](#add-trigger-actions) al flujo de trabajo.

<a name="add-trigger-actions"></a>

## <a name="add-a-trigger-and-actions"></a>Adición de un desencadenador y acciones

Después de abrir el diseñador, aparece la solicitud **Elija una operación** en el diseñador y está seleccionada de manera predeterminada. Ahora puede empezar a crear el flujo de trabajo agregando un desencadenador y acciones.

El flujo de trabajo de este ejemplo usa este desencadenador y estas acciones:

* El [desencadenar Solicitud](../connectors/connectors-native-reqres.md) integrado, **Cuando se recibe una solicitud HTTP**, que recibe llamadas o solicitudes entrantes y crea un punto de conexión al que pueden llamar otros servicios o aplicaciones lógicas.

* La [acción de Office 365 Outlook](../connectors/connectors-create-api-office365-outlook.md), **Enviar correo electrónico**.

* La [acción Respuesta](../connectors/connectors-native-reqres.md) integrada, que se usa para enviar una respuesta y devolver los datos al autor de la llamada.

### <a name="add-the-request-trigger"></a>Adición del desencadenador Request

1. Junto al diseñador, en el panel **Agregar un desencadenador**, en el cuadro de búsqueda **Elegir una operación**, asegúrese de que **Integrado** esté seleccionado para que pueda seleccionar un desencadenador que se ejecute de forma nativa.

1. En el cuadro de búsqueda **Elegir una operación**, escriba `when a http request` y seleccione el desencadenador Solicitud integrado denominado **Cuando se recibe una solicitud HTTP**.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo y el panel **Agregar un desencadenador** con el desencadenador "Cuando se recibe una solicitud HTTP" seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/add-request-trigger.png)

   Cuando el desencadenador aparece en el diseñador, se abre el panel de detalles del desencadenador para mostrar las propiedades, la configuración y otras acciones del desencadenador.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo con el desencadenador "Cuando se recibe una solicitud HTTP" seleccionado y el panel de detalles del desencadenador abierto.](./media/create-stateful-stateless-workflows-visual-studio-code/request-trigger-added-to-designer.png)

   > [!TIP]
   > Si el panel de detalles no aparece, asegúrese de que el desencadenador esté seleccionado en el diseñador.

1. Si necesita eliminar un elemento del diseñador, [siga estos pasos para eliminar elementos del diseñador](#delete-from-designer).

### <a name="add-the-office-365-outlook-action"></a>Adición de la acción Office 365 Outlook

1. En el diseñador, en el desencadenador que agregó, seleccione **Nuevo paso**.

   Aparece la solicitud **Elija una operación** en el diseñador, y se vuelve a abrir el panel **Agregar una acción** para que pueda seleccionar la acción siguiente.

1. En el panel **Agregar una acción**, en el cuadro de búsqueda **Elegir una operación**, seleccione **Azure** para poder buscar y seleccionar una acción para un conector administrado que esté implementado en Azure.

   En este ejemplo se selecciona y usa la acción de Office 365 Outlook, **Enviar correo electrónico (V2)** .

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo y el panel **Agregar una acción** con la acción "Enviar correo electrónico" de Office 365 Outlook seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/add-send-email-action.png)

1. En el panel de detalles de la acción, seleccione **Iniciar sesión** para que pueda crear una conexión a su cuenta de correo electrónico.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo y el panel **Enviar correo electrónico (V2)** con "Iniciar sesión" seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/send-email-action-sign-in.png)

1. Cuando Visual Studio Code le pida consentimiento para acceder a su cuenta de correo electrónico, seleccione **Abrir**.

   ![Captura de pantalla que muestra la solicitud de Visual Studio Code para permitir el acceso.](./media/create-stateful-stateless-workflows-visual-studio-code/visual-studio-code-open-external-website.png)

   > [!TIP]
   > Para evitar futuras solicitudes, seleccione **Configure Trusted Domains** (Configurar dominios de confianza) para que pueda agregar la página de autenticación como dominio de confianza.

1. Siga las indicaciones posteriores para iniciar sesión, permitir el acceso y permitir que se vuelva a Visual Studio Code.

   > [!NOTE]
   > Si transcurre demasiado tiempo antes de completar las solicitudes, el proceso de autenticación agota el tiempo de espera y se produce un error. En este caso, vuelva al diseñador e intente iniciar sesión de nuevo para crear la conexión.

1. Cuando la extensión Azure Logic Apps (versión preliminar) le solicite consentimiento para acceder a su cuenta de correo electrónico, seleccione **Abrir**. Siga la solicitud a continuación para permitir el acceso.

   ![Captura de pantalla que muestra la solicitud de la extensión de versión preliminar para permitir el acceso.](./media/create-stateful-stateless-workflows-visual-studio-code/allow-preview-extension-open-uri.png)

   > [!TIP]
   > Para evitar futuras solicitudes, seleccione **No preguntar de nuevo sobre esta extensión**.

   Una vez que Visual Studio Code crea la conexión, algunos conectores muestran el mensaje `The connection will be valid for {n} days only`. Este límite de tiempo se aplica solo mientras se crea la aplicación lógica en Visual Studio Code. Después de la implementación, ya no se aplica este límite porque la aplicación lógica puede autenticarse durante el tiempo de ejecución mediante su [identidad administrada asignada por el sistema](../logic-apps/create-managed-service-identity.md) habilitada automáticamente. Esta identidad administrada se diferencia de las credenciales de autenticación o de la cadena de conexión que se usan al crear una conexión. Si deshabilita esta identidad administrada asignada por el sistema, las conexiones no funcionarán durante el tiempo de ejecución.

1. En el diseñador, si la acción **Enviar correo electrónico** no aparece seleccionada, seleccione esa acción.

1. En el panel de detalles de la acción, en la pestaña **Parámetros**, proporcione la información necesaria para la acción, por ejemplo:

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo con detalles para la acción "Enviar un correo electrónico" de Office 365 Outlook.](./media/create-stateful-stateless-workflows-visual-studio-code/send-email-action-details.png)

   | Propiedad | Obligatorio | Value | Descripción |
   |----------|----------|-------|-------------|
   | **To** | Sí | <*your-email-address*> | El destinatario del correo electrónico, que puede ser su dirección de correo electrónico con fines de prueba. En este ejemplo se usa la dirección de correo electrónico ficticia `sophiaowen@fabrikam.com`. |
   | **Subject** | Sí | `An email from your example workflow` | El asunto del correo electrónico |
   | **Cuerpo** | Sí | `Hello from your example workflow!` | El contenido del cuerpo del correo electrónico. |
   ||||

   > [!NOTE]
   > Si quiere hacer cambios en el panel de detalles de la pestaña **Configuración**, **Resultado estático** o **Ejecutar después de**, asegúrese de seleccionar **Listo** para confirmar dichos cambios antes de cambiar de pestaña o cambiar el foco al diseñador. De lo contrario, Visual Studio Code no conservará los cambios. Para obtener información, vea la [página de problemas conocidos de la versión preliminar pública de Logic Apps en GitHub](https://github.com/Azure/logicapps/blob/master/articles/logic-apps-public-preview-known-issues.md).

1. En el diseñador, seleccione **Guardar**.

> [!IMPORTANT]
> Para ejecutar localmente un flujo de trabajo que usa un desencadenador o acciones basados en webhook, como el [desencadenador o acción de webhook de HTTP integrado](../connectors/connectors-native-webhook.md), debe habilitar esta funcionalidad mediante la [configuración del reenvío de la dirección URL de devolución de llamada del webhook](#webhook-setup).

<a name="webhook-setup"></a>

## <a name="enable-locally-running-webhooks"></a>Habilitación de webhooks que se ejecutan localmente

Cuando se usa un desencadenador o una acción basados en webhook, como **Webhook de HTTP**, con una aplicación lógica que se ejecuta en Azure, el runtime de Logic Apps se suscribe al punto de conexión de servicio mediante la generación y el registro de una dirección URL de devolución de llamada con ese punto de conexión. Después, el desencadenador o la acción espera a que el punto de conexión de servicio llame a la dirección URL. Sin embargo, cuando se trabaja en Visual Studio Code, la dirección URL de devolución de llamada generada comienza por `http://localhost:7071/...`. Esta dirección URL es para el servidor localhost, que es privado, por lo que el punto de conexión de servicio no puede llamar a esta dirección URL.

Para ejecutar localmente las acciones y los desencadenadores basados en webhook en Visual Studio Code, debe configurar una dirección URL pública que exponga el servidor localhost y reenvíe de forma segura las llamadas desde el punto de conexión de servicio a la dirección URL de devolución de llamada del webhook. Puede usar un servicio de reenvío y una herramienta como [**ngrok**](https://ngrok.com/), que abre un túnel HTTP en el puerto localhost, o puede usar su propia herramienta.

#### <a name="set-up-call-forwarding-using-ngrok"></a>Configuración del reenvío de llamadas con **ngrok**

1. [Regístrese para obtener una cuenta de **ngrok**](https://dashboard.ngrok.com/signup) si no tiene una. En otro caso, [inicie sesión en su cuenta](https://dashboard.ngrok.com/login).

1. Obtenga el token de autenticación personal, que el cliente de **ngrok** necesita para conectarse y autentique el acceso a su cuenta.

   1. Para encontrar la [página de tokens de autenticación](https://dashboard.ngrok.com/auth/your-authtoken), en el menú del panel de la cuenta, expanda **Autenticación** y seleccione **Your Authtoken** (Su Authtoken).

   1. En el cuadro **Your Authtoken** (Su Authtoken), copie el token en una ubicación segura.

1. En la [página de descargas de **ngrok**](https://ngrok.com/download) o [el panel de la cuenta](https://dashboard.ngrok.com/get-started/setup), descargue la versión de **ngrok** que desee y extraiga el archivo .zip. Para obtener más información, consulte [Paso 1: Descomprima para la instalación](https://ngrok.com/download).

1. En el equipo, abra la herramienta del símbolo del sistema. Desplácese hasta la ubicación en que se encuentra el archivo **ngrok.exe**.

1. Conecte el cliente de **ngrok** a su cuenta de **ngrok** ejecutando el siguiente comando. Para más información, consulte [Paso 2: Conecte su cuenta](https://ngrok.com/download).

   `ngrok authtoken <your_auth_token>`

1. Abra el túnel HTTP en el puerto localhost 7071 ejecutando el siguiente comando. Para obtener más información, consulte el [Paso 3: Actívelo](https://ngrok.com/download).

   `ngrok http 7071`

1. En la salida, busque la línea siguiente:

   `http://<domain>.ngrok.io -> http://localhost:7071`

1. Copie y guarde la dirección URL que tiene este formato: `http://<domain>.ngrok.io`

#### <a name="set-up-the-forwarding-url-in-your-app-settings"></a>Configuración de la dirección URL de reenvío en la configuración de la aplicación

1. En Visual Studio Code, en el diseñador, agregue el desencadenador o la acción **HTTP + webhook**.

1. Cuando aparezca el mensaje en que se solicita la ubicación del punto de conexión del host, escriba la dirección URL de reenvío (redireccionamiento) que creó anteriormente.

   > [!NOTE]
   > Si se ignora el mensaje, aparecerá una advertencia, en la que se indica que se debe especificar la dirección URL de reenvío, así que seleccione **Configurar** y escriba la dirección URL. Después de este paso, el mensaje no volverá a aparecer en las acciones o desencadenadores de webhook que pueda agregar posteriormente.
   >
   > Para que el mensaje vuelva a aparecer, en el nivel raíz del proyecto, abra el menú contextual del archivo **local.settings.json** y seleccione **Configure Webhook Redirect Endpoint** (Configurar punto de conexión de redireccionamiento de webhook). Ahora aparece el símbolo del sistema para que pueda especificar la dirección URL de reenvío.

   Visual Studio Code agrega la dirección URL de reenvío al archivo **local.settings.json** de la carpeta raíz del proyecto. En el objeto `Values`, ahora aparece la propiedad denominada `Workflows.WebhookRedirectHostUri` y se establece en la dirección URL de reenvío, por ejemplo:
   
   ```json
   {
      "IsEncrypted": false,
      "Values": {
         "AzureWebJobsStorage": "UseDevelopmentStorage=true",
         "FUNCTIONS_WORKER_RUNTIME": "node",
         "FUNCTIONS_V2_COMPATIBILITY_MODE": "true",
         <...>
         "Workflows.WebhookRedirectHostUri": "http://xxxXXXXxxxXXX.ngrok.io",
         <...>
      }
   }
   ```

La primera vez que se inicia una sesión de depuración local o se ejecuta el flujo de trabajo sin depurar, el runtime de Logic Apps registra el flujo de trabajo con el punto de conexión de servicio y se suscribe a ese punto de conexión para notificar las operaciones de webhook. La próxima vez que se ejecute el flujo de trabajo, el tiempo de ejecución no se registrará ni se volverá a suscribir porque el registro de la suscripción ya existe en el almacenamiento local.

Al detener la sesión de depuración para una ejecución de flujo de trabajo que usa desencadenadores o acciones basados en webhook de ejecución local, los registros de suscripción existentes no se eliminan. Para anular el registro, tiene que quitar o eliminar manualmente los registros de suscripciones.

> [!NOTE]
> Después de que el flujo de trabajo empiece a ejecutarse, la ventana de terminal podría mostrar errores como el de este ejemplo:
>
> `message='Http request failed with unhandled exception of type 'InvalidOperationException' and message: 'System.InvalidOperationException: Synchronous operations are disallowed. Call ReadAsync or set AllowSynchronousIO to true instead.`
>
> En este caso, abra el archivo **local.settings.json** en la carpeta raíz del proyecto y asegúrese de que la propiedad está establecida en `true`:
>
> `"FUNCTIONS_V2_COMPATIBILITY_MODE": "true"`

<a name="manage-breakpoints"></a>

## <a name="manage-breakpoints-for-debugging"></a>Administración de puntos de interrupción para la depuración

Antes de ejecutar y probar el flujo de trabajo de la aplicación lógica mediante el inicio de una sesión de depuración, puede establecer [puntos de interrupción](https://code.visualstudio.com/docs/editor/debugging#_breakpoints) dentro del archivo **workflow.json** de cada flujo de trabajo. No es necesario realizar ninguna otra configuración. 

En la actualidad, los puntos de interrupción solo se admiten en las acciones, no en los desencadenadores. Cada definición de acción tiene estas ubicaciones de puntos de interrupción:

* Establezca el punto de interrupción inicial en la línea que muestra el nombre de la acción. Cuando este punto de interrupción llega durante la sesión de depuración, puede revisar las entradas de la acción antes de que se evalúen.

* Establezca el punto de interrupción inicial en la línea que muestra la llave de cierre ( **}** ) de la acción. Cuando este punto de interrupción llega durante la sesión de depuración, puede revisar los resultados de la acción antes de que esta termine su ejecución.

Para agregar un punto de interrupción, siga estos pasos:

1. Abra el archivo **workflow.json** para el flujo de trabajo que quiere depurar.

1. En la línea donde quiere establecer el punto de interrupción, en la columna izquierda, seleccione dentro de esa columna. Para quitar el punto de interrupción, seleccione ese punto de interrupción.

   Al iniciar la sesión de depuración, la vista de ejecución aparece en el lado izquierdo de la ventana de código, mientras que la barra de herramientas de depuración aparece cerca de la parte superior.

   > [!NOTE]
   > Si la vista de ejecución no aparece automáticamente, presione Ctrl+Mayús+D.

1. Para revisar la información disponible cuando se alcanza un punto de interrupción, en la vista de ejecución, examine el panel **Variables**.

1. Para continuar con la ejecución del flujo de trabajo, en la barra de herramientas de depuración, seleccione **Continuar** (botón de reproducción).

Puede agregar y quitar puntos de interrupción en cualquier momento durante la ejecución del flujo de trabajo. Sin embargo, si actualiza el archivo **workflow.json** después de que se inicie la ejecución, los puntos de interrupción no se actualizan automáticamente. Para actualizar los puntos de interrupción, reinicie la aplicación lógica.

Para obtener información general, consulte [Puntos de interrupción: Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging#_breakpoints).

<a name="run-test-debug-locally"></a>

## <a name="run-test-and-debug-locally"></a>Ejecución, prueba y depuración locales

Para probar la aplicación lógica, siga estos pasos para iniciar una sesión de depuración y encontrar la dirección URL del punto de conexión que se crea mediante el desencadenador de solicitud. Necesitará esta dirección URL para poder enviar una solicitud a ese punto de conexión más adelante.

1. Para depurar un flujo de trabajo sin estado con mayor facilidad, puede [habilitar el historial de ejecución para ese flujo de trabajo](#enable-run-history-stateless).

1. En la barra de actividad de Visual Studio Code, abra el menú **Ejecutar** y seleccione **Iniciar depuración** (F5).

   Se abre la ventana **Terminal** para que pueda revisar la sesión de depuración.

   > [!NOTE]
   > Si aparece el error **"Error exists after running preLaunchTask 'generateDebugSymbols'"** (Se produce un error tras ejecutar preLaunchTask 'generateDebugSymbols), consulte la sección [La sesión de depuración no se inicia](#debugging-fails-to-start).

1. Ahora, busque la URL de devolución de llamada para el punto de conexión en el desencadenador Solicitud.

   1. Vuelva a abrir el panel del Explorador para que pueda ver el proyecto.

   1. En el menú contextual del archivo **workflow.json**, seleccione **Información general**.

      ![Captura de pantalla que muestra el panel del Explorador y la ventana contextual del archivo workflow.json con la opción "Información general" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/open-workflow-overview.png)

   1. Busque el valor **URL de devolución de llamadas**, que tiene un aspecto similar a esta dirección URL para el desencadenador Solicitud de ejemplo:

      `http://localhost:7071/api/<workflow-name>/triggers/manual/invoke?api-version=2020-05-01-preview&sp=%2Ftriggers%2Fmanual%2Frun&sv=1.0&sig=<shared-access-signature>`

      ![Captura de pantalla que muestra la página de información general del flujo de trabajo con la URL de devolución de llamada](./media/create-stateful-stateless-workflows-visual-studio-code/find-callback-url.png)

1. Para probar la dirección URL de devolución de llamada al desencadenar el flujo de trabajo de la aplicación lógica, abra [Postman](https://www.postman.com/downloads/) o su herramienta preferida para crear y enviar solicitudes.

   Este ejemplo continúa con Postman. Para obtener más información, consulte la [Introducción a Postman](https://learning.postman.com/docs/getting-started/introduction/).

   1. En la barra de herramientas de Postman, seleccione **New** (Nuevo).

      ![Captura de pantalla que muestra Postman con el botón Nuevo seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/postman-create-request.png)

   1. En el panel **Create New** (Crear nuevo), en **Building Blocks** (Bloques de creación), seleccione **Request** (Solicitud).

   1. En la ventana **Save Request** (Guardar solicitud), en **Request name** (Nombre de solicitud), escriba un nombre para la solicitud, por ejemplo, `Test workflow trigger`.

   1. En **Select a collection or folder to save to** (Seleccionar una colección o carpeta donde guardar), seleccione **Create Collection** (Crear colección).

   1. En **All Collections** (Todas las colecciones), escriba un nombre para la colección que se va a crear para organizar las solicitudes, presione ENTRAR y, a continuación, seleccione **Save to <*nombre-de-la-colección*>** (Guardar en <nombre de la colección>). En este ejemplo se usa `Logic Apps requests` como nombre de la colección.

      Se abre el panel de solicitudes de Postman para que pueda enviar una solicitud a la URL de devolución de llamada para el desencadenador Solicitud.

      ![Captura de pantalla que muestra Postman con el panel de solicitudes abierto.](./media/create-stateful-stateless-workflows-visual-studio-code/postman-request-pane.png)

   1. Vuelva a Visual Studio Code. Desde la página de información general del flujo de trabajo, copie el valor de la propiedad **URL de devolución de llamadas**.

   1. Vuelva a Postman. En el panel de solicitudes, junto a la lista de métodos, que actualmente muestra **GET** como método de solicitud predeterminado, pegue la URL de devolución de llamada que copió anteriormente en el cuadro de dirección y seleccione **Send** (Enviar).

      ![Captura de pantalla que muestra Postman y la URL de devolución de llamada en el cuadro de dirección con el botón Enviar seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/postman-test-call-back-url.png)

      El flujo de trabajo de aplicación lógica de ejemplo envía un correo electrónico similar a este ejemplo:

      ![Captura de pantalla que muestra el correo electrónico de Outlook tal como se describe en el ejemplo.](./media/create-stateful-stateless-workflows-visual-studio-code/workflow-app-result-email.png)

1. En Visual Studio Code, vuelva a la página de información general del flujo de trabajo.

   Si creó un flujo de trabajo con estado, después de que la solicitud que envió desencadene el flujo de trabajo, la página de información general mostrará el estado y el historial de ejecución del flujo de trabajo.

   > [!TIP]
   > Si el estado de ejecución no aparece, intente actualizar la página de información general seleccionando **Actualizar**. No se produce ninguna ejecución para un desencadenador que se omite debido a criterios que no se cumplen o si no se encuentra ningún dato.

   ![Captura de pantalla que muestra la página de información general del flujo de trabajo con el estado y el historial de ejecución.](./media/create-stateful-stateless-workflows-visual-studio-code/post-trigger-call.png)

   | Estado de la ejecución | Descripción |
   |------------|-------------|
   | **Anulado** | La ejecución se ha detenido o no ha finalizado debido a problemas externos; por ejemplo, una interrupción del sistema o una suscripción de Azure vencida. |
   | **Cancelado** | La ejecución se ha desencadenado y se ha iniciado, pero ha recibido una solicitud de cancelación. |
   | **Erróneo** | Se ha producido un error en al menos en una acción de la ejecución. No se ha configurado ninguna acción posterior en el flujo de trabajo para controlar el error. |
   | **Ejecución** | La ejecución se ha desencadenado y está en curso, pero este estado también puede aparecer para una ejecución que está limitada debido a los [límites de acción](logic-apps-limits-and-config.md) o al [plan de precios actual](https://azure.microsoft.com/pricing/details/logic-apps/). <p><p>**Sugerencia**: Si se configura un [registro de diagnóstico](monitor-logic-apps-log-analytics.md), se puede obtener información sobre los eventos de limitación que se produzcan. |
   | **Correcto** | La ejecución se ha completado correctamente. Si se ha producido un error en alguna acción, se ha controlado mediante una acción posterior en el flujo de trabajo. |
   | **Tiempo de espera agotado** | Se ha agotado el tiempo de espera de la ejecución porque la duración actual ha superado el límite de duración de la ejecución, que se controla mediante el valor [**Retención del historial de ejecución en días**](logic-apps-limits-and-config.md#run-duration-retention-limits). La duración de una ejecución se calcula mediante su hora de inicio y su límite de duración en esa hora de inicio. <p><p>**Nota**: Si la duración de la ejecución también supera el *límite de retención del historial de ejecución actual*, que también se controla mediante la opción [**Retención del historial de ejecución en días**](logic-apps-limits-and-config.md#run-duration-retention-limits), la ejecución se borra del historial de ejecuciones mediante un trabajo de limpieza diaria. Con independencia de que la ejecución agote el tiempo de espera o se complete, el período de retención siempre se calcula mediante la hora de inicio de la ejecución y el límite de retención *actual*. Por tanto, si reduce el límite de duración de una ejecución en curso, se agota su tiempo de espera. Sin embargo, la ejecución se mantiene o se borra del historial de ejecución en función de si su duración supera el límite de retención. |
   | **En espera** | La ejecución no se ha iniciado o está en pausa, por ejemplo, debido a un flujo de trabajo anterior que sigue en ejecución. |
   |||

1. Para revisar los estados de cada paso de una ejecución específica, y las entradas y salidas del paso, seleccione el botón de puntos suspensivos ( **…** ) para esa ejecución y seleccione **Mostrar la ejecución**.

   ![Captura de pantalla que muestra la fila del historial de ejecución del flujo de trabajo con el botón de puntos suspensivos y la opción "Mostrar la ejecución" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/show-run-history.png)

   Visual Studio Code abre la vista de supervisión y muestra el estado de cada paso de la ejecución.

   ![Captura de pantalla que muestra cada paso de la ejecución del flujo de trabajo y su estado.](./media/create-stateful-stateless-workflows-visual-studio-code/run-history-action-status.png)

   > [!NOTE]
   > Si se produce un error en una ejecución y un paso en la vista de supervisión muestra el error `400 Bad Request`, este problema puede deberse a un nombre de acción o nombre de desencadenador más largo que hace que el identificador uniforme de recursos (URI) subyacente supere el límite de caracteres predeterminado. Para más información, vea ["400 Solicitud errónea"](#400-bad-request).

   Estos son los posibles estados que puede tener cada paso del flujo de trabajo:

   | Estado de la acción | Icono | Descripción |
   |---------------|------|-------------|
   | **Anulado** | ![Icono de estado de la acción "Anulado"][aborted-icon] | La acción se ha detenido o no ha finalizado debido a problemas externos; por ejemplo, una interrupción del sistema o una suscripción de Azure vencida. |
   | **Cancelado** | ![Icono de estado de la acción "Cancelado"][cancelled-icon] | La acción se estaba ejecutando, pero recibió una solicitud de cancelación. |
   | **Erróneo** | ![Icono de estado de la acción "Con error"][failed-icon] | Se produjo un error en la acción. |
   | **Ejecución** | ![Icono de estado de la acción "En ejecución"][running-icon] | La acción se está ejecutando actualmente. |
   | **Omitido** | ![Icono de estado de la acción "Omitido"][skipped-icon] | La acción se omitió porque se produjo un error en la acción inmediatamente anterior. Una acción tiene una condición `runAfter` que requiere que la acción anterior finalice correctamente antes de que se pueda ejecutar la acción actual. |
   | **Correcto** | ![Icono de estado de la acción "Correcto"][succeeded-icon] | La acción se realizó correctamente. |
   | **Se realizó correctamente con reintentos** | ![Icono de estado de la acción "Correcto con reintentos"][succeeded-with-retries-icon] | La acción se realizó correctamente, pero solo después de uno o varios reintentos. Para revisar el historial de reintentos en la vista de detalles del historial de ejecución, seleccione esa acción para que pueda ver las entradas y salidas. |
   | **Tiempo de espera agotado** | ![Icono de estado de la acción "Tiempo de espera agotado"][timed-out-icon] | La acción se detuvo debido al límite de tiempo de espera que especificó la configuración de la acción. |
   | **En espera** | ![Icono de estado de la acción "En espera"][waiting-icon] | Se aplica a una acción de webhook que está esperando una solicitud entrante de un autor de llamada. |
   ||||

   [aborted-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/aborted.png
   [cancelled-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/cancelled.png
   [failed-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/failed.png
   [running-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/running.png
   [skipped-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/skipped.png
   [succeeded-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/succeeded.png
   [succeeded-with-retries-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/succeeded-with-retries.png
   [timed-out-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/timed-out.png
   [waiting-icon]: ./media/create-stateful-stateless-workflows-visual-studio-code/waiting.png

1. Para revisar las entradas y salidas de cada paso, seleccione el paso que quiere inspeccionar.

   ![Captura de pantalla que muestra el estado de cada paso del flujo de trabajo más las entradas y salidas de la acción "Enviar correo electrónico" expandida.](./media/create-stateful-stateless-workflows-visual-studio-code/run-history-details.png)

1. Para revisar aún más las entradas y salidas sin procesar para ese paso, seleccione **Mostrar entradas sin procesar** o **Mostrar salidas sin procesar**.

1. Para detener la sesión de depuración, en el menú **Ejecutar**, seleccione **Detener depuración** (Mayús+F5).

<a name="return-response"></a>

## <a name="return-a-response"></a>Devolución de respuesta

Para devolver una respuesta al autor de la llamada que envió una solicitud a la aplicación lógica, puede usar la [acción Respuesta](../connectors/connectors-native-reqres.md) integrada para un flujo de trabajo que se inicie con el desencadenador de solicitud.

1. En el diseñador de flujos de trabajo, en la acción **Enviar un correo electrónico**, seleccione **Nuevo paso**.

   Aparece la solicitud **Elegir una operación** en el diseñador, y se vuelve a abrir el **panel Agregar una acción** para que pueda seleccionar la acción siguiente.

1. En el panel **Agregar una acción**, en el cuadro de búsqueda **Elegir una acción**, asegúrese de que **Integrado** esté seleccionado. En el cuadro de búsqueda, escriba `response` y seleccione la acción **Respuesta**.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo con la acción Respuesta seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/add-response-action.png)

   Cuando aparece la acción **Respuesta** en el diseñador, se abre automáticamente el panel de detalles de la acción.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo con el panel de detalles de la acción "Respuesta" abierto y la propiedad "Cuerpo" establecida en el valor de la propiedad "Cuerpo" de la acción "Enviar un correo electrónico".](./media/create-stateful-stateless-workflows-visual-studio-code/response-action-details.png)

1. En la pestaña **Parámetros**, proporcione la información necesaria para la función a la que quiere llamar.

   En este ejemplo se devuelve el valor de propiedad **Cuerpo** que se obtiene de la acción **Enviar correo electrónico**.

   1. Haga clic dentro del cuadro de propiedad **Cuerpo** para que aparezca la lista de contenido dinámico y muestre los valores de salida disponibles del desencadenador y las acciones anteriores en el flujo de trabajo.

      ![Captura de pantalla que muestra el panel de detalles de la acción "Respuesta" con el puntero del mouse dentro de la propiedad "Cuerpo" para que aparezca la lista de contenido dinámico.](./media/create-stateful-stateless-workflows-visual-studio-code/open-dynamic-content-list.png)

   1. En la lista de contenido dinámico, en **Enviar correo electrónico**, seleccione **Cuerpo**.

      ![Captura de pantalla que muestra la lista de contenido dinámico abierta. En la lista, en el encabezado "Enviar correo electrónico", el valor de salida "Cuerpo" está seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/select-send-email-action-body-output-value.png)

      Cuando haya terminado, la propiedad **Cuerpo** de la acción Respuesta estará establecida en el valor de salida **Cuerpo** de la acción **Enviar correo electrónico**.

      ![Captura de pantalla que muestra el estado de cada paso del flujo de trabajo más las entradas y salidas de la acción "Respuesta" expandida.](./media/create-stateful-stateless-workflows-visual-studio-code/response-action-details-body-property.png)

1. En el diseñador, seleccione **Guardar**.

<a name="retest-workflow"></a>

## <a name="retest-your-logic-app"></a>Nueva comprobación de la aplicación lógica

Después de realizar actualizaciones en la aplicación lógica, puede ejecutar otra prueba volviendo a ejecutar el depurador en Visual Studio y enviando otra solicitud para desencadenar la aplicación lógica actualizada, de forma similar a los pasos descritos en [Ejecución, prueba y depuración locales](#run-test-debug-locally).

1. En la barra de actividad de Visual Studio Code, abra el menú **Ejecutar** y seleccione **Iniciar depuración** (F5).

1. En Postman o en la herramienta para crear y enviar solicitudes, envíe otra solicitud para desencadenar el flujo de trabajo.

1. Si ha creado un flujo de trabajo con estado, en la página información general del flujo de trabajo, compruebe el estado de la ejecución más reciente. Para ver los estados, las entradas y las salidas de cada paso en esa ejecución, seleccione el botón de puntos suspensivos ( **…** ) para esa ejecución y seleccione **Mostrar la ejecución**.

   Por ejemplo, este es el estado paso a paso de una ejecución después de que se haya actualizado el flujo de trabajo de ejemplo con la acción Respuesta.

   ![Captura de pantalla que muestra el estado de cada paso del flujo de trabajo actualizado más las entradas y salidas de la acción "Respuesta" expandida.](./media/create-stateful-stateless-workflows-visual-studio-code/run-history-details-rerun.png)

1. Para detener la sesión de depuración, en el menú **Ejecutar**, seleccione **Detener depuración** (Mayús+F5).

<a name="firewall-setup"></a>

##  <a name="find-domain-names-for-firewall-access"></a>Búsqueda de nombres de dominio para el acceso al firewall

Antes de implementar y ejecutar el flujo de trabajo de la aplicación lógica en Azure Portal, si su entorno tiene estrictos requisitos de red o firewalls que limitan el tráfico, debe configurar los permisos para las conexiones de desencadenador o acción que existan en el flujo de trabajo.

Para buscar los nombres de dominio completos (FQDN) de estas conexiones, siga estos pasos:

1. En el proyecto de la aplicación lógica, abra el archivo **connections.json**, que se crea después de agregar el primer desencadenador o acción basado en conexión al flujo de trabajo, y busque el objeto `managedApiConnections`.

1. Para cada conexión que ha creado, busque, copie y guarde el valor de la propiedad `connectionRuntimeUrl` en un lugar seguro para que pueda configurar el firewall con esta información.

   Este archivo **connections.json** de ejemplo contiene dos conexiones, una conexión AS2 y una conexión de Office 365, con estos valores de `connectionRuntimeUrl`:

   * AS2: `"connectionRuntimeUrl": https://9d51d1ffc9f77572.00.common.logic-{Azure-region}.azure-apihub.net/apim/as2/11d3fec26c87435a80737460c85f42ba`

   * Office 365: `"connectionRuntimeUrl": https://9d51d1ffc9f77572.00.common.logic-{Azure-region}.azure-apihub.net/apim/office365/668073340efe481192096ac27e7d467f`

   ```json
   {
      "managedApiConnections": {
         "as2": {
            "api": {
               "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/as2"
            },
            "connection": {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Web/connections/{connection-resource-name}"
            },
            "connectionRuntimeUrl": https://9d51d1ffc9f77572.00.common.logic-{Azure-region}.azure-apihub.net/apim/as2/11d3fec26c87435a80737460c85f42ba,
            "authentication": {
               "type":"ManagedServiceIdentity"
            }
         },
         "office365": {
            "api": {
               "id": "/subscriptions/{Azure-subscription-ID}/providers/Microsoft.Web/locations/{Azure-region}/managedApis/office365"
            },
            "connection": {
               "id": "/subscriptions/{Azure-subscription-ID}/resourceGroups/{Azure-resource-group}/providers/Microsoft.Web/connections/{connection-resource-name}"
            },
            "connectionRuntimeUrl": https://9d51d1ffc9f77572.00.common.logic-{Azure-region}.azure-apihub.net/apim/office365/668073340efe481192096ac27e7d467f,
            "authentication": {
               "type":"ManagedServiceIdentity"
            }
         }
      }
   }
   ```

<a name="deploy-azure"></a>

## <a name="deploy-to-azure"></a>Implementar en Azure

Desde Visual Studio Code, puede publicar directamente el proyecto en Azure, que implementa la aplicación lógica con el nuevo tipo de recurso **Logic Apps (versión preliminar**). De forma similar al recurso de la aplicación de funciones en Azure Functions, la implementación de este nuevo tipo de recurso requiere que seleccione un [plan de hospedaje y plan de tarifa](../app-service/overview-hosting-plans.md), que puede configurar durante la implementación. Para obtener más información sobre los planes de hospedaje y los precios, revise estos temas:

* [Escalado vertical de una aplicación en Azure App Service](../app-service/manage-scale-up.md)
* [Escalado y hospedaje de Azure Functions](../azure-functions/functions-scale.md)

La aplicación lógica se puede publicar como un nuevo recurso, lo que crea automáticamente los recursos necesarios, como una [cuenta de Azure Storage, similar a los requisitos de la aplicación de funciones](../azure-functions/storage-considerations.md). O bien, puede publicar la aplicación lógica en un recurso **Logic Apps (versión preliminar)** implementado anteriormente, que sobrescribe dicha aplicación lógica.

### <a name="publish-to-a-new-logic-app-preview-resource"></a>Publicación en un nuevo recurso de Logic Apps (versión preliminar)

1. En la barra de actividad de Visual Studio Code, seleccione el icono de Azure.

1. En la barra de herramientas del panel **Azure: Logic Apps (versión preliminar)** , seleccione **Deploy to Logic App** (Implementar en la aplicación lógica).

   ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar)" y la barra de herramientas del panel con la opción "Implementar en la aplicación lógica" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/deploy-to-logic-app.png)

1. Si se le solicita, seleccione la suscripción de Azure que se va a usar para la implementación de la aplicación lógica.

1. En la lista que abre Visual Studio Code, seleccione una de estas opciones:

   * **Create new Logic App (Preview) in Azure** (Crear nueva instancia de Logic Apps [versión preliminar] en Azure) (rápido)
   * **Create new Logic App (Preview) in Azure Advanced** (Crear nueva instancia de Logic Apps [versión preliminar] en Azure avanzado)
   * Un recurso **Logic Apps (versión preliminar)** implementado previamente, si existe.

   Este ejemplo continúa con **Create new Logic App (Preview) in Azure Advanced** (Crear nueva Logic Apps [versión preliminar] en Azure avanzado).

   ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar)" con una lista con "Crear nueva Logic Apps (versión preliminar) en Azure" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/select-create-logic-app-options.png)

1. Para crear el nuevo recurso **Logic Apps (versión preliminar)** , siga estos pasos:

   1. Proporcione un nombre único globalmente para la nueva aplicación lógica, que es el nombre que se usará para el recurso **Logic Apps (versión preliminar)** . En este ejemplo se usa `Fabrikam-Workflows-App`.

      ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar)" y una solicitud para proporcionar un nombre para la nueva aplicación lógica que se va a crear.](./media/create-stateful-stateless-workflows-visual-studio-code/enter-logic-app-name.png)

   1. Seleccione un [plan de hospedaje](../app-service/overview-hosting-plans.md) para la nueva aplicación lógica: [**Plan de App Service** (dedicado)](../azure-functions/dedicated-plan.md) o [**Prémium**](../azure-functions/functions-premium-plan.md).

      > [!IMPORTANT]
      > Los planes de consumo no son compatibles ni están disponibles para este tipo de recurso. El plan que haya seleccionado determinará las funcionalidades y los planes de tarifa que más tarde estarán disponibles. Para obtener más información, consulte estos temas: 
      >
      > * [Escalado y hospedaje de Azure Functions](../azure-functions/functions-scale.md)
      > * [Detalles de precios de App Service](https://azure.microsoft.com/pricing/details/app-service/)
      >
      > Por ejemplo, el plan Prémium proporciona acceso a las funcionalidades de red, como la conexión e integración de forma privada con redes virtuales de Azure, de manera similar a Azure Functions al crear e implementar las aplicaciones lógicas. 
      > Para obtener más información, consulte estos temas:
      > 
      > * [Opciones de redes de Azure Functions](../azure-functions/functions-networking-options.md)
      > * [Azure Logic Apps en ejecución en cualquier ubicación: posibilidades de conexión de red con Azure Logic Apps (versión preliminar)](https://techcommunity.microsoft.com/t5/integrations-on-azure/logic-apps-anywhere-networking-possibilities-with-logic-app/ba-p/2105047)

      En este ejemplo se usa el **plan de App Service**.

      ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar)" y una solicitud para seleccionar "Plan de App Service" o "Premium".](./media/create-stateful-stateless-workflows-visual-studio-code/select-hosting-plan.png)

   1. Cree un nuevo plan de App Service o seleccione un plan existente. En este ejemplo se selecciona **Crear nuevo plan de App Service**.

      ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar) y una solicitud para "Crear nuevo plan de App Service" o seleccionar un plan de App Service existente.](./media/create-stateful-stateless-workflows-visual-studio-code/create-app-service-plan.png)

   1. Proporcione un nombre para el plan de App Service y, a continuación, seleccione un [plan de tarifa](../app-service/overview-hosting-plans.md) para el plan. En este ejemplo se selecciona el plan **F1 Gratis**.

      ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar)" y una solicitud para seleccionar un plan de tarifa.](./media/create-stateful-stateless-workflows-visual-studio-code/select-pricing-tier.png)

   1. Para obtener un rendimiento óptimo, busque y seleccione el mismo grupo de recursos que el proyecto para la implementación.

      > [!NOTE]
      > Aunque puede crear o usar un grupo de recursos diferente, esto podría afectar al rendimiento. Si crea o elige otro grupo de recursos, pero cancela después de que aparezca el mensaje de confirmación, la implementación también se cancelará.

   1. Para flujos de trabajo con estado, seleccione **Crear una nueva cuenta de almacenamiento** o una cuenta de almacenamiento existente.

      ![Captura de pantalla que muestra el panel "Azure: Logic Apps (versión preliminar)" y una solicitud para crear o seleccionar una cuenta de almacenamiento.](./media/create-stateful-stateless-workflows-visual-studio-code/create-storage-account.png)

   1. Si la configuración de la creación e implementación de la aplicación lógica admite el uso de [Application Insights](../azure-monitor/app/app-insights-overview.md), puede habilitar opcionalmente el registro de diagnóstico y el seguimiento para la aplicación lógica. Puede hacerlo al implementar la aplicación lógica desde Visual Studio Code o después de la implementación. Debe tener una instancia de Application Insights, pero puede crear este recurso [con antelación](../azure-monitor/app/create-workspace-resource.md), al implementar la aplicación lógica o después de la implementación.

      Para habilitar el registro y el seguimiento ahora, siga estos pasos:

      1. Seleccione un recurso de Application Insights existente o **Cree un nuevo recurso de Application Insights**.

      1. En [Azure Portal](https://portal.azure.com), vaya al recurso de Application Insights.

      1. En el menú del recurso, seleccione **Información general**. Busque y copie el valor de **Clave de instrumentación**.

      1. En Visual Studio Code, en la carpeta raíz del proyecto, abra el archivo **local.settings.json**.

      1. En el objeto `Values`, agregue la propiedad `APPINSIGHTS_INSTRUMENTATIONKEY` y establezca el valor en la clave de instrumentación, por ejemplo:

         ```json
         {
            "IsEncrypted": false,
            "Values": {
               "AzureWebJobsStorage": "UseDevelopmentStorage=true",
               "FUNCTIONS_WORKER_RUNTIME": "dotnet",
               "APPINSIGHTS_INSTRUMENTATIONKEY": <instrumentation-key>
            }
         }
         ```

         > [!TIP]
         > Puede comprobar si los nombres de los desencadenadores y las acciones aparecen correctamente en la instancia de Application Insights.
         >
         > 1. En Azure Portal, vaya al recurso de Application Insights.
         >
         > 2. En el menú de recursos, en **Investigar**, seleccione **Asignación de aplicaciones**.
         >
         > 3. Revise los nombres de las operaciones que aparecen en el mapa.
         >
         > Algunas solicitudes entrantes de los desencadenadores integrados pueden aparecer como duplicadas en la asignación de aplicaciones. 
         > En lugar de usar el formato `WorkflowName.ActionName`, estos duplicados usan el nombre del flujo de trabajo como el nombre de la operación y se originan desde el host de Azure Functions.

      1. A continuación, si quiere puede ajustar el nivel de gravedad de los datos de seguimiento que la aplicación lógica recopila y envía a la instancia de Application Insights.

         Cada vez que se produce un evento relacionado con el flujo de trabajo, como cuando se desencadena un flujo de trabajo o cuando se ejecuta una acción, el tiempo de ejecución emite varios seguimientos. Estos seguimientos cubren la duración del flujo de trabajo e incluyen, entre otros, los siguientes tipos de evento:

         * Actividad del servicio, como inicio, detención y errores.
         * Actividad de los trabajos y del distribuidor.
         * Actividad de flujo de trabajo, como el desencadenador, la acción y la ejecución.
         * Actividad de solicitud de almacenamiento, como correcto o error.
         * Actividad de solicitud HTTP, como de entrada, salida, correcta y error.
         * Todos los seguimientos de desarrollo, como los mensajes de depuración.

         Cada tipo de evento se asigna a un nivel de gravedad. Por ejemplo, el nivel `Trace` captura los mensajes más detallados, mientras que el nivel `Information` captura la actividad general en el flujo de trabajo, como cuando la aplicación lógica, el flujo de trabajo, el desencadenador y las acciones se inician y detienen. En esta tabla se describen los niveles de gravedad y sus tipos de seguimiento:

         | Nivel de gravedad | Tipo de seguimiento |
         |----------------|------------|
         | Crítico | Registros que describen un error irrecuperable en la aplicación lógica. |
         | Depurar | Registros que puede usar para la investigación durante el desarrollo, por ejemplo, llamadas HTTP de entrada y salida. |
         | Error | Registros que indican un error en la ejecución del flujo de trabajo, pero no un error general en la aplicación lógica. |
         | Información | Registros que realizan el seguimiento de la actividad general en la aplicación lógica o el flujo de trabajo, por ejemplo: <p><p>- Cuando se inicia y finaliza un desencadenador, una acción o una ejecución. <br>- Cuando se inicia o finaliza la aplicación lógica. |
         | Seguimiento | Registros que contienen los mensajes más detallados, por ejemplo, las solicitudes de almacenamiento o la actividad del distribuidor, además de todos los mensajes relacionados con la actividad de ejecución del flujo de trabajo. |
         | Advertencia | Registros que resaltan un estado anómalo en la aplicación lógica, pero no impiden su ejecución. |
         |||

         Para establecer el nivel de gravedad, en el nivel raíz del proyecto, abra el archivo **host.json** y busque el objeto `logging`. Este objeto controla el filtrado de registros para todos los flujos de trabajo de la aplicación lógica y sigue el [diseño de ASP.NET Core para el filtrado de tipo de registro](/aspnet/core/fundamentals/logging/?view=aspnetcore-2.1&preserve-view=true#log-filtering).

         ```json
         {
            "version": "2.0",
            "logging": {
               "applicationInsights": {
                  "samplingExcludedTypes": "Request",
                  "samplingSettings": {
                     "isEnabled": true
                  }
               }
            }
         }
         ```

         Si el objeto `logging` no contiene un objeto `logLevel` que incluya la propiedad `Host.Triggers.Workflow`, agregue esos elementos. Establezca la propiedad en el nivel de gravedad para el tipo de seguimiento que desee, por ejemplo:

         ```json
         {
            "version": "2.0",
            "logging": {
               "applicationInsights": {
                  "samplingExcludedTypes": "Request",
                  "samplingSettings": {
                     "isEnabled": true
                  }
               },
               "logLevel": {
                  "Host.Triggers.Workflow": "Information"
               }
            }
         }
         ```

   Cuando haya terminado con los pasos de implementación, Visual Studio Code comienza a crear e implementar los recursos necesarios para publicar la aplicación lógica.

1. Para revisar y supervisar el proceso de implementación, en el menú **Ver**, seleccione **Salida**. En la lista de la barra de herramientas de la ventana Salida, seleccione **Azure Logic Apps**.

   ![Captura de pantalla que muestra la ventana Salida con "Azure Logic Apps" seleccionado en la lista de la barra de herramientas junto con el progreso y los estados de la implementación.](./media/create-stateful-stateless-workflows-visual-studio-code/logic-app-deployment-output-window.png)

   Cuando Visual Studio Code termina de implementar la aplicación lógica en Azure, aparece el siguiente mensaje:

   ![Captura de pantalla que muestra un mensaje que indica que la implementación en Azure se ha completado correctamente.](./media/create-stateful-stateless-workflows-visual-studio-code/deployment-to-azure-completed.png)

   Enhorabuena, la aplicación lógica ahora está activa en Azure y habilitada de manera predeterminada.

Ahora puede obtener información sobre cómo realizar estas tareas:

* [Agregar un flujo de trabajo en blanco al proyecto](#add-workflow-existing-project).

* [Administrar la aplicación lógica implementada en Visual Studio Code](#manage-deployed-apps-vs-code) o mediante [Azure Portal](#manage-deployed-apps-portal).

* [Habilitar el historial de ejecución para flujos de trabajo sin estado](#enable-run-history-stateless).

* [Habilitar la vista de supervisión en Azure Portal para una aplicación lógica implementada](#enable-monitoring).

<a name="add-workflow-existing-project"></a>

## <a name="add-blank-workflow-to-project"></a>Adición de un flujo de trabajo en blanco al proyecto

Puede tener varios flujos de trabajo en el proyecto de aplicación lógica. Para agregar un flujo de trabajo en blanco al proyecto, siga estos pasos:

1. En la barra de actividad de Visual Studio Code, seleccione el icono de Azure.

1. En el panel de Azure, junto a **Azure: Logic Apps (versión preliminar)** , seleccione **Crear flujo de trabajo** (icono de Azure Logic Apps).

1. Seleccione el tipo de flujo de trabajo que desea agregar: **Con estado** o **Sin estado**.

1. Escriba un nombre para el flujo de trabajo.

Cuando haya terminado, aparecerá una nueva carpeta de flujo de trabajo en el proyecto junto con un archivo **workflow.json** para la definición de flujo de trabajo.

<a name="manage-deployed-apps-vs-code"></a>

## <a name="manage-deployed-logic-apps-in-visual-studio-code"></a>Administración de aplicaciones lógicas implementadas en Visual Studio Code

En Visual Studio Code, puede ver todas las aplicaciones lógicas implementadas en la suscripción a Azure, independientemente de si son del tipo de recurso **Logic Apps** original o **Logic Apps (versión preliminar)** , y seleccionar tareas que le ayuden a administrar esas aplicaciones lógicas. Sin embargo, para acceder a ambos tipos de recursos, necesita las extensiones **Azure Logic Apps** y **Azure Logic Apps (versión preliminar)** para Visual Studio Code.

1. En la barra de herramientas izquierda, seleccione el icono de Azure. En el área **Azure: Logic Apps (versión preliminar)** , expanda su suscripción, que muestra todas las aplicaciones lógicas implementadas para esa suscripción.

1. Abra la aplicación lógica que desea administrar. En el menú contextual de la aplicación lógica, seleccione la tarea que desea realizar.

   Por ejemplo, puede seleccionar tareas como detener, iniciar, reiniciar o eliminar la aplicación lógica implementada.

   ![Captura de pantalla que muestra Visual Studio Code con el panel de la extensión "Azure Logic Apps (versión preliminar)" abierto y el flujo de trabajo implementado.](./media/create-stateful-stateless-workflows-visual-studio-code/find-deployed-workflow-visual-studio-code.png)

1. Para ver todos los flujos de trabajo en la aplicación lógica, expándala y, después, expanda el nodo **Flujos de trabajo**.

1. Para ver un flujo de trabajo específico, abra el menú contextual del flujo de trabajo y seleccione **Open in Designer** (Abrir en el diseñador), que abre el flujo de trabajo en modo de solo lectura.

   Para editar el flujo de trabajo, tiene estas opciones:

   * En Visual Studio Code, abra el archivo **workflow.json** del proyecto en el diseñador de flujos de trabajo, realice los cambios y vuelva a implementar la aplicación lógica en Azure.

   * En Azure Portal, [busque y abra la aplicación lógica](#manage-deployed-apps-portal). Busque, edite y guarde el flujo de trabajo.

1. Para abrir la aplicación lógica implementada en Azure Portal, abra el menú contextual de la aplicación lógica y seleccione **Abrir en el portal**.

   Azure Portal se abre en el explorador, inicia sesión automáticamente en el portal si ha iniciado sesión en Visual Studio Code y muestra la aplicación lógica.

   ![Captura de pantalla que muestra la página de Azure Portal para la aplicación lógica en Visual Studio Code.](./media/create-stateful-stateless-workflows-visual-studio-code/deployed-workflow-azure-portal.png)

   También puede iniciar sesión por separado en Azure Portal, usar el cuadro de búsqueda del portal para buscar la aplicación lógica y, a continuación, seleccionar la aplicación lógica en la lista de resultados.

   ![Captura de pantalla que muestra Azure Portal y la barra de búsqueda con los resultados de la búsqueda para la aplicación lógica implementada, que aparece seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/find-deployed-workflow-azure-portal.png)

<a name="manage-deployed-apps-portal"></a>

## <a name="manage-deployed-logic-apps-in-the-portal"></a>Administración de aplicaciones lógicas implementadas en el portal

En Azure Portal, puede ver todas las aplicaciones lógicas implementadas en la suscripción a Azure, independientemente de si son del tipo de recurso **Logic Apps** original o del tipo de recurso **Logic Apps (versión preliminar)** . Actualmente, cada tipo de recurso se organiza y administra como categoría independiente en Azure. Para buscar aplicaciones lógicas que tengan el tipo de recurso **Logic Apps (versión preliminar)** , siga estos pasos:

1. En el cuadro de búsqueda de Azure Portal, escriba `logic app preview`. Cuando aparezca la lista de resultados, en **Servicios**, seleccione **Logic Apps (versión preliminar)** .

   ![Captura de pantalla que muestra el cuadro de búsqueda de Azure Portal con el texto de búsqueda "logic app preview".](./media/create-stateful-stateless-workflows-visual-studio-code/portal-find-logic-app-preview-resource.png)

1. En el panel **Logic Apps (versión preliminar)** , busque y seleccione la aplicación lógica que implementó en Visual Studio Code.

   ![Captura de pantalla que muestra Azure Portal y el recurso Logic Apps (versión preliminar) implementado en Azure.](./media/create-stateful-stateless-workflows-visual-studio-code/logic-app-preview-resources-pane.png)

   Azure Portal abre la página individual de recursos para la aplicación lógica seleccionada.

   ![Captura de pantalla que muestra la página de recursos del flujo de trabajo de la aplicación lógica en Azure Portal.](./media/create-stateful-stateless-workflows-visual-studio-code/deployed-workflow-azure-portal.png)

1. Para ver los flujos de trabajo en esta aplicación lógica, en el menú de la aplicación lógica, seleccione **Flujos de trabajo**.

   En el panel **Flujos de trabajo** se muestran todos los flujos de trabajo de la aplicación lógica actual. En este ejemplo se muestra el flujo de trabajo que creó en Visual Studio Code.

   ![Captura de pantalla que muestra una página de recurso "Logic Apps (versión preliminar)" con el panel "Flujos de trabajo" abierto y el flujo de trabajo implementado.](./media/create-stateful-stateless-workflows-visual-studio-code/deployed-logic-app-workflows-pane.png)

1. Para ver un flujo de trabajo, en el panel **Flujos de trabajo**, seleccione ese flujo de trabajo.

   Se abre el panel del flujo de trabajo y muestra más información y tareas que puede realizar en ese flujo de trabajo.

   Por ejemplo, para ver los pasos del flujo de trabajo, seleccione **Diseñador**.

   ![Captura de pantalla que muestra el panel "Información general" del flujo de trabajo seleccionado, mientras que el menú flujo de trabajo muestra el comando "Diseñador" seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/workflow-overview-pane-select-designer.png)

   El diseñador de flujos de trabajo se abre y muestra el flujo de trabajo que creó en Visual Studio Code. Ahora puede realizar cambios en este flujo de trabajo en Azure Portal.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo y el flujo de trabajo implementado desde Visual Studio Code.](./media/create-stateful-stateless-workflows-visual-studio-code/opened-workflow-designer.png)

<a name="add-workflow-portal"></a>

## <a name="add-another-workflow-in-the-portal"></a>Adición de otro flujo de trabajo en el portal

A través de Azure Portal, puede agregar flujos de trabajo vacíos a un recurso **Logic Apps (versión preliminar)** implementado desde Visual Studio Code y compilar esos flujos de trabajo en Azure Portal.

1. En [Azure Portal](https://portal.azure.com), busque y seleccione el recurso **Logic Apps (versión preliminar)** implementado.

1. En el menú de la aplicación lógica, seleccione **Flujos de trabajo**. En el panel **Flujos de trabajo**, seleccione **Agregar**.

   ![Captura de pantalla que muestra el panel "Flujos de trabajo" de la aplicación lógica seleccionada y la barra de herramientas con el comando "Agregar" seleccionado.](./media/create-stateful-stateless-workflows-visual-studio-code/add-new-workflow.png)

1. En el panel **Nuevo flujo de trabajo**, indique el nombre del flujo de trabajo. Seleccione **Con estado** o **Sin estado** **>** **Crear**.

   Una vez que Azure implemente el nuevo flujo de trabajo, que aparece en el panel **Flujos de trabajo**, seleccione ese flujo de trabajo para que pueda administrar y realizar otras tareas, como abrir el diseñador o la vista de código.

   ![Captura de pantalla que muestra el flujo de trabajo seleccionado con opciones de administración y revisión.](./media/create-stateful-stateless-workflows-visual-studio-code/view-new-workflow.png)

   Por ejemplo, al abrir el diseñador para un nuevo flujo de trabajo, se muestra un lienzo en blanco. Ahora puede compilar este flujo de trabajo en Azure Portal.

   ![Captura de pantalla que muestra el diseñador de flujos de trabajo y un flujo de trabajo en blanco.](./media/create-stateful-stateless-workflows-visual-studio-code/opened-blank-workflow-designer.png)

<a name="enable-run-history-stateless"></a>

## <a name="enable-run-history-for-stateless-workflows"></a>Habilitación del historial de ejecución para flujos de trabajo sin estado

Para depurar un flujo de trabajo sin estado con más facilidad, puede habilitar el historial de ejecución para ese flujo de trabajo y luego deshabilitar el historial de ejecución cuando haya terminado. Siga estos pasos para Visual Studio Code, o bien, si está trabajando en Azure Portal, consulte [Creación de flujos de trabajo con y sin estado en Azure Portal](create-stateful-stateless-workflows-azure-portal.md#enable-run-history-stateless).

1. En el proyecto de Visual Studio Code, expanda la carpeta **workflow-designtime** y abra el archivo **local.settings.json**.

1. Agregue la propiedad `Workflows.{yourWorkflowName}.operationOptions` y establezca el valor en `WithStatelessRunHistory`, por ejemplo:

   **Windows**

   ```json
   {
      "IsEncrypted": false,
      "Values": {
         "AzureWebJobsStorage": "UseDevelopmentStorage=true",
         "FUNCTIONS_WORKER_RUNTIME": "dotnet",
         "Workflows.{yourWorkflowName}.OperationOptions": "WithStatelessRunHistory"
      }
   }
   ```

   **macOS o Linux**

   ```json
   {
      "IsEncrypted": false,
      "Values": {
         "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=fabrikamstorageacct; \
             AccountKey=<access-key>;EndpointSuffix=core.windows.net",
         "FUNCTIONS_WORKER_RUNTIME": "dotnet",
         "Workflows.{yourWorkflowName}.OperationOptions": "WithStatelessRunHistory"
      }
   }
   ```

1. Para deshabilitar el historial de ejecución cuando haya terminado, establezca la propiedad `Workflows.{yourWorkflowName}.OperationOptions`en `None` o elimine la propiedad y su valor.

<a name="enable-monitoring"></a>

## <a name="enable-monitoring-view-in-the-azure-portal"></a>Habilitación de la vista de supervisión en Azure Portal

Después de implementar un recurso de **Logic App (versión preliminar)** de Visual Studio Code en Azure, puede revisar cualquier historial de ejecución disponible y los detalles de un flujo de trabajo en ese recurso mediante Azure Portal y la experiencia de **Supervisión** para ese flujo de trabajo. Sin embargo, en primer lugar, debe habilitar la funcionalidad de vista **Supervisión** en ese recurso de aplicación lógica.

1. En [Azure Portal](https://portal.azure.com), busque y seleccione el recurso **Logic Apps (versión preliminar)** implementado.

1. En el menú de ese recurso, en **API**, seleccione **CORS**.

1. En el panel **CORS**, en **Orígenes permitidos**, agregue el carácter comodín (*).

1. Cuando esté listo, en la barra de herramientas de **CORS**, seleccione **Guardar**.

   ![Captura de pantalla que muestra Azure Portal con un recurso Logic App (versión preliminar) implementado. En el menú de recursos, "CORS" está seleccionado con una nueva entrada para "Orígenes permitidos" establecida en el carácter comodín "*".](./media/create-stateful-stateless-workflows-visual-studio-code/enable-run-history-deployed-logic-app.png)

<a name="enable-open-application-insights"></a>

## <a name="enable-or-open-application-insights-after-deployment"></a>Habilitación o apertura de Application Insights después de la implementación

Durante la ejecución del flujo de trabajo, la aplicación lógica emite telemetría junto con otros eventos. Puede usar esta telemetría para obtener una mejor visibilidad de si el flujo de trabajo se ejecuta correctamente y cómo funciona el runtime de Logic Apps de varias maneras. Puede supervisar el flujo de trabajo mediante [Application Insights](../azure-monitor/app/app-insights-overview.md), que proporciona telemetría casi en tiempo real (métricas en directo). Esta funcionalidad puede ayudarle a investigar los errores y problemas de rendimiento con más facilidad si usa estos datos para diagnosticar problemas, configurar alertas y crear gráficos.

Si la configuración de la creación e implementación de la aplicación lógica admite el uso de [Application Insights](../azure-monitor/app/app-insights-overview.md), puede habilitar opcionalmente el registro de diagnóstico y el seguimiento para la aplicación lógica. Puede hacerlo al implementar la aplicación lógica desde Visual Studio Code o después de la implementación. Debe tener una instancia de Application Insights, pero puede crear este recurso [con antelación](../azure-monitor/app/create-workspace-resource.md), al implementar la aplicación lógica o después de la implementación.

Para habilitar Application Insights en una aplicación lógica implementada o para revisar los datos de Application Insights si ya está habilitado, siga estos pasos:

1. En Azure Portal, busque la aplicación lógica implementada.

1. En el menú de la aplicación lógica, en **Configuración**, seleccione **Application Insights**.

1. Si Application Insights no se ha habilitado, en el panel **Application Insights**, seleccione **Activar Application Insights**. Una vez que se actualice el panel, en la parte inferior, seleccione **Aplicar**.

   Si se ha habilitado Application Insights, en el panel **Application Insights**, seleccione **Ver datos de Application Insights**.

Después de que se abra Application Insights, puede revisar varias métricas de la aplicación lógica. Para obtener más información, consulte estos temas:

* [Azure Logic Apps en ejecución en cualquier ubicación: supervisión con Application Insights: parte 1](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/1877849)
* [Azure Logic Apps en ejecución en cualquier ubicación: supervisión con Application Insights: parte 2](https://techcommunity.microsoft.com/t5/integrations-on-azure/azure-logic-apps-running-anywhere-monitor-with-application/ba-p/2003332)

<a name="deploy-docker"></a>

## <a name="deploy-to-docker"></a>Implementación en Docker

Puede implementar la aplicación lógica en un [contenedor de Docker](/visualstudio/docker/tutorials/docker-tutorial#what-is-a-container) como el entorno de hospedaje mediante el uso de la [CLI de .NET](/dotnet/core/tools/). Con estos comandos, puede compilar y publicar el proyecto de la aplicación lógica. Después, puede compilar y ejecutar un contenedor de Docker como destino para implementar la aplicación lógica.

Si no está familiarizado con Docker, consulte estos temas:

* [¿Qué es Docker?](/dotnet/architecture/microservices/container-docker-introduction/docker-defined)
* [Introducción a los contenedores y Docker](/dotnet/architecture/microservices/container-docker-introduction/)
* [Introducción a .NET y Docker](/dotnet/core/docker/introduction)
* [Contenedores, imágenes y registros de Docker](/dotnet/architecture/microservices/container-docker-introduction/docker-containers-images-registries)
* [Tutorial: Introducción a Docker (Visual Studio Code)](/visualstudio/docker/tutorials/docker-tutorial)

### <a name="requirements"></a>Requisitos

* La cuenta de Azure Storage que usa la aplicación lógica para la implementación

* Un archivo de Docker para el flujo de trabajo que se usa al compilar el contenedor de Docker

  Por ejemplo, este archivo de Docker de ejemplo implementa una aplicación lógica y especifica la cadena de conexión que contiene la clave de acceso de la cuenta de Azure Storage que se usó para publicar la aplicación lógica en Azure Portal. Para encontrar esta cadena, consulte [Obtención de la cadena de conexión de la cuenta de almacenamiento](#find-storage-account-connection-string). Para obtener más información, revise [Procedimientos recomendados para escribir archivos de Docker](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/).
  
  > [!IMPORTANT]
  > En escenarios de producción, asegúrese de proteger estos secretos e información confidencial, por ejemplo, mediante el uso de un almacén de claves. Para los archivos de Docker específicamente, revise [Creación de imágenes con BuildKit](https://docs.docker.com/develop/develop-images/build_enhancements/) y [Administración de los datos confidenciales con secretos de Docker](https://docs.docker.com/engine/swarm/secrets/).

   ```text
   FROM mcr.microsoft.com/azure-functions/node:3.0

   ENV AzureWebJobsStorage <storage-account-connection-string>
   ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
       AzureFunctionsJobHost__Logging__Console__IsEnabled=true \
       FUNCTIONS_V2_COMPATIBILITY_MODE=true

   COPY . /home/site/wwwroot

   RUN cd /home/site/wwwroot
   ```

<a name="find-storage-account-connection-string"></a>

### <a name="get-storage-account-connection-string"></a>Cadena de conexión de la cuenta de almacenamiento

Para poder compilar y ejecutar una imagen de contenedor de Docker, es preciso que obtenga la cadena de conexión que contiene la clave de acceso a su cuenta de almacenamiento. Anteriormente, esta cuenta de almacenamiento se creó para usar la extensión en macOS o Linux, o cuando implementó la aplicación lógica en Azure Portal.

Para buscar esta cadena de conexión y copiarla, siga estos pasos:

1. En Azure Portal, en el menú de la cuenta de almacenamiento, en **Configuración**, seleccione **Claves de acceso**. 

1. En el panel **Claves de acceso**, busque y copie la cadena de conexión de la cuenta de almacenamiento, que es similar a la de este ejemplo:

   `DefaultEndpointsProtocol=https;AccountName=fabrikamstorageacct;AccountKey=<access-key>;EndpointSuffix=core.windows.net`

   ![Captura de pantalla que muestra Azure Portal con las claves de acceso de la cuenta de almacenamiento y la cadena de conexión copiada.](./media/create-stateful-stateless-workflows-visual-studio-code/find-storage-account-connection-string.png)

   Para más información, consulte [Administración de claves de cuenta de almacenamiento](../storage/common/storage-account-keys-manage.md?tabs=azure-portal#view-account-access-keys).

1. Guarde la cadena de conexión en un lugar seguro para que pueda agregarla al archivo de Docker que usará para la implementación. 

<a name="find-storage-account-master-key"></a>

### <a name="find-master-key-for-storage-account"></a>Búsqueda de la clave maestra de la cuenta de almacenamiento

Cuando el flujo de trabajo contiene un desencadenador de solicitud, es preciso [obtener la dirección URL de devolución de llamada del desencadenador](#get-callback-url-request-trigger) después de compilar y ejecutar la imagen de contenedor de Docker. Para esta tarea, también debe especificar el valor de la clave maestra de la cuenta de almacenamiento que usa para la implementación.

1. Para encontrar esta clave maestra, en el proyecto, abra el archivo **azure-webjobs-secrets/{nombre-de-implementación}/host.json**.

1. Busque la `AzureWebJobsStorage` propiedad y copie el valor de clave de esta sección:

   ```json
   {
      <...>
      "masterKey": {
         "name": "master",
         "value": "<master-key>",
         "encrypted": false
      },
      <...>
   }
   ```

1. Guarde este valor de clave en un lugar seguro, ya que lo usará más adelante.

<a name="build-run-docker-container-image"></a>

### <a name="build-and-run-your-docker-container-image"></a>Compilación y ejecución de la imagen de contenedor de Docker

1. Cree la imagen de contenedor de Docker con el archivo de Docker y ejecute este comando:

   `docker build --tag local/workflowcontainer .`

   Para obtener más información, consulte [docker build](https://docs.docker.com/engine/reference/commandline/build/).

1. Ejecute el contenedor localmente mediante este comando:

   `docker run -e WEBSITE_HOSTNAME=localhost -p 8080:80 local/workflowcontainer`

   Para obtener más información, consulte [docker run](https://docs.docker.com/engine/reference/commandline/run/).

<a name="get-callback-url-request-trigger"></a>

### <a name="get-callback-url-for-request-trigger"></a>Obtención de la dirección URL de devolución de llamada para el desencadenador de solicitud

En el caso de cualquier flujo de trabajo que use el desencadenador de solicitud, obtenga la dirección URL de devolución de llamada del desencadenador, para lo que debe enviar esta solicitud:

`POST /runtime/webhooks/workflow/api/management/workflows/{workflow-name}/triggers/{trigger-name}/listCallbackUrl?api-version=2020-05-01-preview&code={master-key}`

El valor `{trigger-name}` es el nombre del desencadenador de solicitud que aparece en la definición de JSON del flujo de trabajo. El valor `{master-key}` se define en la cuenta de Azure Storage que se establece para la propiedad `AzureWebJobsStorage` en el archivo, **azure-webjobs-secrets/{nombre-de-implementación}/host.json**. Para más información, consulte la sección [Búsqueda de la clave maestra de una cuenta de almacenamiento](#find-storage-account-master-key).

<a name="delete-from-designer"></a>

## <a name="delete-items-from-the-designer"></a>Eliminación de elementos del diseñador

Para eliminar un elemento del flujo de trabajo del diseñador, siga cualquiera de estos pasos:

* Seleccione el elemento, abra el menú contextual del elemento (Mayús+F10) y seleccione **Eliminar**. Para confirmar, seleccione **Aceptar**.

* Seleccione el elemento y presione la tecla Supr. Para confirmar, seleccione **Aceptar**.

* Seleccione el elemento para que se abra el panel de detalles de ese elemento. En la esquina superior derecha del panel, abra el menú de puntos suspensivos ( **...** ) y seleccione **Eliminar**. Para confirmar, seleccione **Aceptar**.

  ![Captura de pantalla que muestra el elemento seleccionado en el diseñador con el panel de detalles abierto y con el botón de puntos suspensivos seleccionado, así como el comando "Eliminar".](./media/create-stateful-stateless-workflows-visual-studio-code/delete-item-from-designer.png)

  > [!TIP]
  > Si no está visible el menú de puntos suspensivos, expanda la ventana de Visual Studio Code el ancho suficiente para que en el panel de detalles se muestre el botón de puntos suspensivos ( **...** ) en la esquina superior derecha.

<a name="troubleshooting"></a>

## <a name="troubleshoot-errors-and-problems"></a>Solución de errores y problemas

<a name="designer-fails-to-open"></a>

### <a name="designer-fails-to-open"></a>El diseñador no se abre

Al intentar abrir el diseñador, aparece este error, **Workflow design time could not be started** (No se pudo iniciar el tiempo de diseño del flujo de trabajo). Si anteriormente intentó abrir el diseñador y, después, dejó de usarlo o eliminó el proyecto, es posible que el conjunto de extensiones no se descargue correctamente. Para comprobar si esta es la causa del problema, siga estos pasos:

  1. En Visual Studio Code, abra la ventana Salida. En el menú **Ver**, seleccione **Salida**.

  1. En la lista de la barra de título de la ventana Salida, seleccione **Azure Logic Apps (versión preliminar)** para que pueda consultar la salida de la extensión, por ejemplo:

     ![Captura de pantalla que muestra la ventana Salida con la opción "Azure Logic Apps" seleccionada.](./media/create-stateful-stateless-workflows-visual-studio-code/check-outout-window-azure-logic-apps.png)

  1. Revise la salida y compruebe si aparece este mensaje de error:

     ```text
     A host error has occurred during startup operation '{operationID}'.
     System.Private.CoreLib: The file 'C:\Users\{userName}\AppData\Local\Temp\Functions\
     ExtensionBundles\Microsoft.Azure.Functions.ExtensionBundle.Workflows\1.1.7\bin\
     DurableTask.AzureStorage.dll' already exists.
     Value cannot be null. (Parameter 'provider')
     Application is shutting down...
     Initialization cancellation requested by runtime.
     Stopping host...
     Host shutdown completed.
     ```

   Para resolver este error, elimine la carpeta **ExtensionBundles** en esta ubicación **…\Users\{su-nombre-de-usuario}\AppData\Local\Temp\Functions\ExtensionBundles** y vuelva a intentar abrir el archivo **workflow.json** en el diseñador.

<a name="missing-triggers-actions"></a>

### <a name="new-triggers-and-actions-are-missing-from-the-designer-picker-for-previously-created-workflows"></a>Faltan nuevos desencadenadores y acciones en el selector del diseñador para flujos de trabajo creados previamente

La versión preliminar de Azure Logic Apps admite acciones integradas para las operaciones de Azure Functions, las operaciones de Liquid y las operaciones de XML, como **Validación XML** y **Transformar XML**. Sin embargo, en el caso de las aplicaciones lógicas creadas con anterioridad, es posible que estas acciones no aparezcan en el selector del diseñador para que las pueda seleccionar si Visual Studio Code usa una versión obsoleta del paquete de extensión `Microsoft.Azure.Functions.ExtensionBundle.Workflows`.

Además, las acciones y el conector **Azure Function Operations** (Operaciones de función de Azure) no aparecen en el selector del diseñador a menos que haya habilitado o seleccionado **Use connectors from Azure** (Usar conectores de Azure) al crear la aplicación lógica. Si no ha habilitado los conectores implementados por Azure en el momento de la creación de la aplicación, puede habilitarlos desde el proyecto en Visual Studio Code. Abra el menú contextual **workflow.json** y seleccione **Use connectors from Azure** (Usar conectores de Azure).

Para corregir el paquete obsoleto, siga estos pasos a fin de eliminar el paquete obsoleto, lo que hace que Visual Studio Code actualice automáticamente el paquete de extensión a la versión más reciente.

> [!NOTE]
> Esta solución solo se aplica a las aplicaciones lógicas que se crean e implementan con Visual Studio Code con Azure Logic Apps (versión preliminar), no a las aplicaciones lógicas que se han creado con Azure Portal. Vea [Faltan desencadenadores y acciones en el diseñador en Azure Portal](create-stateful-stateless-workflows-azure-portal.md#missing-triggers-actions).

1. Guarde cualquier trabajo que no quiera perder y cierre Visual Studio.

1. En el equipo, busque la carpeta siguiente, que contiene las carpetas con versión para el paquete existente:

   `...\Users\{your-username}\.azure-functions-core-tools\Functions\ExtensionBundles\Microsoft.Azure.Functions.ExtensionBundle.Workflows`

1. Elimine la carpeta de la versión del paquete anterior; por ejemplo, si tiene una carpeta para la versión 1.1.3, elimine esa carpeta.

1. Ahora, busque la carpeta siguiente, que contiene las carpetas con versión para el paquete NuGet necesario:

   `...\Users\{your-username}\.nuget\packages\microsoft.azure.workflows.webjobs.extension`

1. Elimine la carpeta de la versión del paquete anterior; por ejemplo, si tiene una carpeta para la versión 1.0.0.8-preview, elimine esa carpeta.

1. Vuelva a abrir Visual Studio Code, el proyecto y el archivo **workflow.json** en el diseñador.

Los desencadenadores y las acciones que faltan ahora aparecen en el diseñador.

<a name="400-bad-request"></a>

### <a name="400-bad-request-appears-on-a-trigger-or-action"></a>"400 Solicitud incorrecta" aparece en un desencadenador o una acción

Cuando se produce un error en una ejecución y se inspecciona la ejecución en la vista de supervisión, este error puede aparecer en un desencadenador o una acción que tienen un nombre más largo, lo que hace que el identificador uniforme de recursos (URI) subyacente supere el límite de caracteres predeterminado.

Para resolver este problema y ajustar el URI más largo, edite las claves del Registro `UrlSegmentMaxCount` y `UrlSegmentMaxLength` en el equipo siguiendo estos pasos. Estos valores predeterminados de la clave se describen en el tema [Configuración del Registro de Http.sys para Windows](/troubleshoot/iis/httpsys-registry-windows).

> [!IMPORTANT]
> Antes de empezar, asegúrese de guardar el trabajo. Esta solución requiere que reinicie el equipo una vez que haya terminado para que los cambios surtan efecto.

1. En el equipo, abra la ventana **Ejecutar** y ejecute el comando `regedit`, que abre el Editor del Registro.

1. En el cuadro **Control de cuentas de usuario**, seleccione **Sí** para permitir los cambios en el equipo.

1. En el panel izquierdo, en **Equipo**, expanda los nodos a lo largo de la ruta de acceso **HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\HTTP\Parameters** y, a continuación, seleccione **Parámetros**.

1. En el panel derecho, busque las claves del Registro `UrlSegmentMaxCount` y `UrlSegmentMaxLength`.

1. Aumente estos valores de clave lo suficiente para que los identificadores URI puedan contener los nombres que desea utilizar. Si estas claves no existen, agréguelas a la carpeta **Parámetros** siguiendo estos pasos:

   1. En el menú contextual **Parámetros**, seleccione **Nuevo** > **Valor de DWORD (32 bits)** .

   1. En el cuadro de edición que aparece, escriba `UrlSegmentMaxCount` como el nuevo nombre de la clave.

   1. Abra el menú contextual de la nueva clave y seleccione **Modificar**.

   1. En el cuadro **Editar cadena** que aparece, escriba el valor de clave de **Información del valor** que desee en formato hexadecimal o decimal. Por ejemplo, `400` en formato hexadecimal es equivalente a `1024` en formato decimal.

   1. Para agregar el valor de clave `UrlSegmentMaxLength`, repita estos pasos.

   Después de aumentar o agregar estos valores de clave, el Editor del Registro presenta un aspecto similar al de este ejemplo:

   ![Captura de pantalla en la que se muestra el Editor del Registro.](media/create-stateful-stateless-workflows-visual-studio-code/edit-registry-settings-uri-length.png)

1. Cuando haya terminado, reinicie el equipo para que los cambios surtan efecto.

<a name="debugging-fails-to-start"></a>

### <a name="debugging-session-fails-to-start"></a>La sesión de depuración no se inicia

Al intentar iniciar una sesión de depuración, aparece el error **"Error exists after running preLaunchTask 'generateDebugSymbols'"** (Aparece un error al ejecutar preLaunchTask 'generateDebugSymbols). Para resolver este problema, edite el archivo **tasks.json** del proyecto para omitir la generación de símbolos.

1. En el proyecto, expanda la carpeta **.vscode** y abra el archivo **tasks.json**.

1. En la siguiente tarea, elimine la línea, `"dependsOn: "generateDebugSymbols"`, junto con la coma del final de la línea anterior, por ejemplo:

   Antes:

   ```json
    {
      "type": "func",
      "command": "host start",
      "problemMatcher": "$func-watch",
      "isBackground": true,
      "dependsOn": "generateDebugSymbols"
    }
   ```

   Después:

   ```json
    {
      "type": "func",
      "command": "host start",
      "problemMatcher": "$func-watch",
      "isBackground": true
    }
   ```

## <a name="next-steps"></a>Pasos siguientes

Nos gustaría conocer sus experiencias con la extensión Azure Logic Apps (versión preliminar).

* En caso de errores o problemas, [dirija los problemas a GitHub](https://github.com/Azure/logicapps/issues).
* Si tiene preguntas, solicitudes, ideas y comentarios, [use este formulario de comentarios](https://aka.ms/lafeedback).
