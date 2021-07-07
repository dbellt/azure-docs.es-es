---
title: 'Tutorial: Examen de datos con Azure Purview (versión preliminar)'
description: En este tutorial se ejecuta un Starter Kit para configurar el patrimonio de datos y después se examinan datos de orígenes de datos para el catálogo de Azure Purview.
author: viseshag
ms.author: viseshag
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: tutorial
ms.date: 12/01/2020
ms.openlocfilehash: fb11033e0f4fcff171825d579e62e071f66b1cea
ms.sourcegitcommit: 8bca2d622fdce67b07746a2fb5a40c0c644100c6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/09/2021
ms.locfileid: "111745896"
---
# <a name="tutorial-scan-data-with-azure-purview-preview"></a>Tutorial: Examen de datos con Azure Purview (versión preliminar)

> [!IMPORTANT]
> Azure Purview se encuentra actualmente en VERSIÓN PRELIMINAR. Los [Términos de uso complementarios para las versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en la versión beta, en versión preliminar o que todavía no se han publicado para que estén disponibles con carácter general.

En esta *serie de cinco tutoriales* aprenderá los aspectos básicos de cómo administrar la gobernanza de datos en un patrimonio de datos mediante Azure Purview (versión preliminar). El patrimonio de datos que se crea en esta parte del tutorial se usa para el resto de la serie.

En la parte 1 de esta serie de tutoriales, deberá realizar lo siguiente:

> [!div class="checklist"]
>
> * Crear un patrimonio de datos con varios recursos de datos de Azure.
> * Examinar los datos en un catálogo.

## <a name="prerequisites"></a>Prerrequisitos

* Suscripción a Azure. Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) antes de empezar.
* Una [cuenta de Azure Purview](create-catalog-portal.md).
* [El Starter Kit](https://github.com/Azure/Purview-Samples/blob/master/PurviewStarterKitV4.zip) que implementará el patrimonio de datos.

> [!NOTE]
> El Starter Kit solo está disponible para Windows.

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en [Azure Portal](https://portal.azure.com).

## <a name="create-a-data-estate"></a>Creación de un patrimonio de datos

En esta sección ejecutará los scripts del Starter Kit para crear un patrimonio de datos simulado. Un patrimonio de datos es una cartera de todos los datos que posee una empresa. Este script del Starter Kit realiza las acciones siguientes:

* Crea una cuenta de Azure Blob Storage y la rellena con datos.
* Crea una cuenta de Azure Data Lake Storage Gen2.
* Crea una instancia de Azure Data Factory y la asocia Azure Purview.
* Configura y desencadena una canalización de actividad de copia entre las cuentas de Azure Blob Storage y de Azure Data Lake Storage Gen2.
* Envía el linaje asociado de Azure Data Factory a Azure Purview.

### <a name="prepare-to-run-the-starter-kit"></a>Preparación para la ejecución del Starter Kit

Siga estos pasos para configurar el software cliente del Starter Kit en la máquina Windows:

1. [Descargue el Starter Kit](https://github.com/Azure/Purview-Samples/blob/master/PurviewStarterKitV4.zip) y extraiga su contenido en la ubicación que prefiera.


1. En el equipo, escriba **PowerShell** en el cuadro de búsqueda de la barra de tareas de Windows. En la lista de búsqueda, haga clic con el botón derecho en **Windows PowerShell** y seleccione **Ejecutar como administrador**.

1. En la ventana de PowerShell, escriba el comando siguiente, reemplazando `<path-to-starter-kit>` por la ruta de acceso a la carpeta de los archivos del Starter Kit extraídos.

   ```powershell
   dir -Path <path-to-starter-kit> -Recurse | Unblock-File
   ```

1. Escriba el siguiente comando para instalar los cmdlets de Azure.

   ```powershell
   Install-Module -Name Az -AllowClobber -Scope CurrentUser
   ```

1. Si ve el mensaje de advertencia *Se necesita el proveedor de NuGet para continuar*, escriba **Y** y presione Entrar.

1. Si ve el mensaje de advertencia *Repositorio que no es de confianza*, escriba **A** y presione Entrar.

PowerShell puede tardar hasta un minuto en instalar los módulos necesarios.

### <a name="collect-data-needed-to-run-the-scripts"></a>Recopilación de los datos necesarios para ejecutar los scripts

Antes de ejecutar los scripts de PowerShell para arrancar el catálogo, obtenga los valores de los siguientes argumentos para usarlos en los scripts:

* TenantID
   1. En [Azure Portal](https://portal.azure.com), seleccione **Azure Active Directory**.
   1. En la sección **Administrar** del panel de navegación izquierdo, seleccione **Propiedades**. A continuación, seleccione el icono de copia de **Id. de inquilino** para guardar el valor en el portapapeles. Pegue el valor en un editor de texto para usarlo posteriormente.

* SubscriptionID
   1. En Azure Portal, busque y seleccione el nombre de la instancia de Azure Purview que creó como requisito previo.
   1. Seleccione la sección **Información general** guarde el GUID para el **Id. de la suscripción**.

   > [!NOTE]
   > - Asegúrese de que está usando la misma suscripción donde creó la cuenta de Azure Purview. Se trata de la misma suscripción que se colocó en la lista de permitidos.
   > - El linaje de datos podría faltar a veces en Azure Purview después de ejecutar Starter Kit. Esto se debe a que faltan permisos en Purview para la instancia de Data Factory creada por Starter Kit. Seleccione [**este vínculo al documento**](how-to-link-azure-data-factory.md#view-existing-data-factory-connections) para asegurarse de que la instancia de Data Factory está configurada correctamente y se le ha asignado el rol adecuado en faltan permisos para Starter Kit en Purview.


* CatalogName: nombre de la cuenta de Azure Purview que creó en [Creación de una cuenta de Azure Purview](create-catalog-portal.md).

* CatalogResourceGroupName: nombre del grupo de recursos en el que creó la cuenta de Azure Purview.

### <a name="verify-permissions"></a>Comprobar los permisos

Siga estos pasos para agregar el usuario que ejecuta el script a la cuenta de Azure Purview que se creó como requisito previo. Los usuarios necesitan los roles *Purview Data Curator* (Administrador de datos de Purview) y *Purview Data Source Administrator* (Administrador de orígenes de datos de Purview). 

Si ha creado la cuenta de Azure Purview usted mismo, se le concederá acceso automáticamente y puede omitir esta sección.

1. Vaya a la página Purview accounts (Cuentas de Purview) en Azure Portal y seleccione la cuenta de Azure Purview que desee modificar.

1. En la página de la cuenta, seleccione la pestaña **Access control (IAM)** (Control de acceso [IAM]) y **+ Add** (+ Agregar).

1. Seleccione **Agregar asignación de roles**.

1. Escriba **Purview Data Curator Role** (Rol Administrador de datos de Purview) en *Role* (Rol).
 
1. Use el valor predeterminado de *Assign accesss to* (Asignar acceso a). El valor predeterminado debe ser **User, group, or service principal** (Usuario, grupo o entidad de servicio).

1. Escriba el nombre del usuario que ejecuta la script en **Select** (Seleccionar).

1. Seleccione **Guardar**.

1. Repita los pasos anteriores con *Role* (Rol) establecido en **Purview Data Source Administrator Role** (Rol Administrador de orígenes de datos de Purview).

Para más información, consulte [Permisos del catálogo](catalog-permissions.md).

### <a name="run-the-client-side-setup-scripts"></a>Ejecución de los scripts de instalación del lado cliente

Una vez completada la configuración del catálogo, ejecute los siguientes scripts en la ventana de PowerShell para crear los recursos, reemplazando los marcadores de posición por los valores que recopiló anteriormente.

1. Use el siguiente comando para ir al directorio del Starter Kit. Reemplace `path-to-starter-kit` por la ruta de acceso de la carpeta del archivo extraído.

   ```powershell
   cd <path-to-starter-kit>
   ```

1. El siguiente comando establece la directiva de ejecución para la máquina local. Escriba **A** para *Sí a todo* cuando se le pida que cambie la directiva de ejecución.

   ```powershell
   Set-ExecutionPolicy -ExecutionPolicy Unrestricted
   ```

1. Conéctese a Azure mediante el comando siguiente. De nuevo, reemplace los marcadores `TenantID` y `SubscriptionID` .

   ```powershell
   .\RunStarterKit.ps1 -ConnectToAzure -TenantId <TenantID> `
   -SubscriptionId <SubscriptionID>
   ```

   Al ejecutar el comando, puede aparecer una ventana emergente para que inicie sesión con sus credenciales de Azure Active Directory.


1. Use el siguiente comando para ejecutar el Starter Kit. Reemplace los marcadores de posición `CatalogName`, `TenantID`, `SubscriptionID`, `NewResourceGroupName` y `CatalogResourceGroupName`. Para `NewResourceGroupName`, use un nombre único (solo con caracteres alfanuméricos en minúscula) para el grupo de recursos que contendrá el patrimonio de datos.

   > [!IMPORTANT]
   > El **nuevonmbredegrupoderecursos** usa solo números y letras minúsculas y debe tener menos de 17 caracteres. **No se permiten alfabetos en mayúsculas ni caracteres especiales.** Esta restricción proviene de las reglas de nomenclatura de la cuenta de almacenamiento.

   ```powershell
   .\RunStarterKit.ps1 -CatalogName <CatalogName> -TenantId <TenantID>`
   -ResourceGroup <newresourcegroupname> `
   -SubscriptionId <SubscriptionID> `
   -CatalogResourceGroup <CatalogResourceGroupName>
   ```

El entorno puede tardar hasta 10 minutos en configurarse. Durante este tiempo, es posible que vea varias ventanas emergentes, que puede omitir. No cierre la ventana de **BlobDataCreator.exe**; se cierra automáticamente al finalizar.

Cuando vea el mensaje `Executing Copy pipeline xxxxxxxxxx-487e-4fc4-9628-92dd8c2c732b`, espere a que otra instancia de **BlobDataCreator.exe** inicie y termine de ejecutarse.

> [!IMPORTANT]
> Si el "número de tareas activas" deja de descender, puede salir de la ventana de creación de blobs y presionar Entrar en la ventana de Powershell.

Una vez finalizado el proceso, se crea un grupo de recursos con el nombre proporcionado. Las cuentas de Azure Data Factory, Azure Blob Storage y Azure Data Lake Storage Gen2 se incluyen en este grupo de recursos. El grupo de recursos está incluido en la suscripción especificada.

## <a name="scan-data-into-the-catalog"></a>Examen de los datos del catálogo

El examen es un proceso por el que el catálogo se conecta directamente a un origen de datos según una programación especificada por el usuario. El catálogo refleja el patrimonio de datos de una empresa mediante el examen, el linaje, el portal y la API. Entre los objetivos se incluyen el examen de lo que está dentro, la extracción de esquemas y el intento de comprender la semántica. En esta sección configurará un examen de los orígenes de datos que generó con el Starter Kit.

El script del Starter Kit que ejecutó creó dos orígenes de datos, Azure Blob Storage y Azure Data Lake Storage Gen2. Puede examinar estos orígenes de datos en el catálogo de uno en uno.

### <a name="authenticate-to-your-storage-with-managed-identity"></a>Autenticación en el almacenamiento con una identidad administrada

Cuando se crea la cuenta, se crea automáticamente una identidad administrada con el mismo nombre que la cuenta de Azure Purview. Antes de poder examinar los datos, debe conceder permisos de rol de Azure Purview en las cuentas de almacenamiento.

1. Vaya hasta el grupo de recursos creado por el Starter Kit y seleccione su cuenta de almacenamiento de blobs.

1. Seleccione **Access Control (IAM)** (Control de acceso [IAM]) en el menú de navegación izquierdo. Seguidamente, seleccione **+ Agregar**.

1. Establezca el rol en **Storage Blob Data Reader** (Lector de datos de blobs de almacenamiento) y escriba el nombre de la cuenta de Azure Purview en **Select** (Seleccionar). A continuación, seleccione **Save** (Guardar) para dar esta asignación de rol a su cuenta de Purview.

   :::image type="content" source="media/tutorial-scan-data/add-role-assignment.png" alt-text="Agregar asignación de roles":::

1. Repita los pasos anteriores para Azure Data Lake Storage Gen2.

### <a name="scan-your-data-sources"></a>Examen de los orígenes de datos

1. En [Azure Portal](https://portal.azure.com), busque y seleccione el nombre de la instancia de Azure Purview que creó como requisito previo y, después, seleccione *Open Purview Studio* (Abrir Purview Studio). Se le dirigirá automáticamente a la página principal de Purview Studio.

1. Seleccione **Sources** (Orígenes) en la página web del catálogo y seleccione **Register** (Registrar). Después, seleccione **Azure Blob Storage** y **Continue** (Continuar).

   :::image type="content" source="media/tutorial-scan-data/add-blob-storage.png" alt-text="Registro de recursos de Blob Storage.":::

1. En la página **Register sources** (Registrar orígenes), escriba un valor en **Name** (Nombre). Elija el valor para **Storage account name** (Nombre de la cuenta de almacenamiento) de la cuenta de Azure Blob Storage que creó antes con el Starter Kit. La cuenta tiene el nombre `<YourResourceGroupName>adcblob`. Seleccione **Finalizar**.

   :::image type="content" source="./media/tutorial-scan-data/register-azure-blob-storage.png" alt-text="Captura de pantalla que muestra la configuración para registrar un origen de datos de Azure Blob Storage." border="true":::

1. En la vista de mapa **Data sources** (Orígenes de datos), seleccione **New scan** (Nuevo examen) en el icono de Azure Blob Storage.

   :::image type="content" source="./media/tutorial-scan-data/select-setup-scan.png" alt-text="Captura de pantalla que muestra cómo seleccionar una configuración de examen de un origen de datos." border="true":::

1. En la página **Scan** (Examinar), escriba un nombre para el examen, seleccione su identidad administrada de la lista desplegable **Credential** (Credencial) y, después, **Continue**(Continuar).

   :::image type="content" source="media/tutorial-scan-data/scan-blob.png" alt-text="Examen de Blob Storage en Azure Purview.":::

1. Puede limitar el examen a blobs individuales. En este tutorial mantendremos todos los recursos marcados para examinar todo y haremos clic en **Continue** (Continuar).

   :::image type="content" source="media/tutorial-scan-data/scope-your-scan.png" alt-text="Ámbito del examen del almacenamiento.":::

1. Seleccione el conjunto de reglas de examen predeterminado y haga clic en **Continue** (Continuar).

1. Puede configurar un desencadenador de examen para exámenes periódicos. Para este tutorial, seleccione **Once** (Una vez) y **Continue** (Continuar).

1. Seleccione **Save and run** (Guardar y ejecutar) para completar el examen.

1. Repita los pasos anteriores para examinar la cuenta de Azure Data Lake Storage Gen2.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:
> [!div class="checklist"]
>
> * Ejecutar el script del Starter Kit para configurar un entorno de patrimonio de datos.
> * Examinar los datos en un catálogo de Azure Purview.

Avance al siguiente tutorial para aprender a desplazarse por la página principal y buscar un recurso.

> [!div class="nextstepaction"]
> [Desplazamiento por la página principal y búsqueda de un recurso](tutorial-asset-search.md)
