---
title: Uso de Herramientas de Azure Data Lake para Visual Studio Code
description: Obtenga información sobre cómo usar Herramientas de Azure Data Lake para Visual Studio Code para crear, probar y ejecutar scripts U-SQL.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 02/09/2018
ms.openlocfilehash: e1d74795fd25019e205f4b7b1d2bac1b67107e2d
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107878585"
---
# <a name="use-azure-data-lake-tools-for-visual-studio-code"></a>Uso de Herramientas de Azure Data Lake para Visual Studio Code

En este artículo, obtendrá información sobre cómo usar Herramientas de Azure Data Lake para Visual Studio Code (VS Code) para crear, probar y ejecutar scripts U-SQL. La información también se explica en el vídeo siguiente:

[![Reproductor de vídeo: Herramientas de Azure Data Lake para VS Code](media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-video.png)](https://channel9.msdn.com/Series/AzureDataLake/Azure-Data-Lake-Tools-for-VSCode?term=ADL%20Tools%20for%20VSCode")

## <a name="prerequisites"></a>Requisitos previos

Herramientas de Azure Data Lake para VS Code es compatible con Windows, Linux y macOS.  La ejecución y depuración locales de U-SQL solo funcionan en Windows.

- [Visual Studio Code](https://www.visualstudio.com/products/code-vs.aspx)

En macOS y Linux:

- [SDK de .NET 5.0](https://dotnet.microsoft.com/download)
- [Mono 5.2.x](https://www.mono-project.com/download/)

## <a name="install-azure-data-lake-tools"></a>Instalación de las herramientas de Azure Data Lake

Después de instalar los requisitos previos, puede instalar Herramientas de Azure Data Lake para VS Code.

### <a name="to-install-azure-data-lake-tools"></a>Para instalar Herramientas de Azure Data Lake

1. Abra Visual Studio Code.
2. En el panel izquierdo, seleccione **Extensiones**. Escriba **Herramientas de Azure Data Lake** en el cuadro de búsqueda.
3. Seleccione **Instalar** junto a **Herramientas de Azure Data Lake**.

   ![Selecciones para la instalación de Herramientas de Data Lake](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extensions.png)

   Después de unos segundos, el botón **Instalar** cambia a **Recargar**.
4. Seleccione **Recargar** para activar la extensión **Herramientas de Azure Data Lake**.
5. Haga clic en **Recargar ventana** para confirmar. Puede ver **Herramientas de Azure Data Lake** en el panel **Extensiones**.

## <a name="activate-azure-data-lake-tools"></a>Activación de Azure Data Lake Tools

Cree un archivo .usql o abra uno ya existente para activar la extensión.

## <a name="work-with-u-sql"></a>Trabajo con U-SQL

Para trabajar con U-SQL, es preciso abrir un archivo U-SQL o una carpeta.

### <a name="to-open-the-sample-script"></a>Para abrir el script de ejemplo

Abra la paleta de comandos (Ctrl+Mayús+P) y escriba **ADL: Open Sample Script**. Se abre otra instancia de este ejemplo. En esta instancia también puede editar, configurar y enviar un script.

### <a name="to-open-a-folder-for-your-u-sql-project"></a>Para abrir una carpeta para el proyecto U-SQL

1. En Visual Studio Code, seleccione el menú **Archivo** y, luego, seleccione **Abrir carpeta**.
2. Especifique una carpeta y, luego, seleccione **Seleccionar carpeta**.
3. Seleccione el menú **Archivo** y, luego, seleccione **Nuevo**. Se agregará un archivo Sin título-1 al proyecto.
4. Escriba el código siguiente en el archivo Sin título-1:

   ```usql
   @departments  =
       SELECT * FROM
           (VALUES
               (31,    "Sales"),
               (33,    "Engineering"),
               (34,    "Clerical"),
               (35,    "Marketing")
           ) AS
                 D( DepID, DepName );
   ```

   OUTPUT @departments     TO "/Output/departments.csv" USING Outputters.Csv();

    El script crea un archivo departments.csv con algunos datos incluidos en la carpeta /output.

5. Guarde el archivo como **myUSQL.usql** en la carpeta abierta.

### <a name="to-compile-a-u-sql-script"></a>Para compilar un script U-SQL

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Compile Script**. Los resultados de la compilación aparecen en la ventana **Salida**. También puede hacer clic con el botón derecho en un archivo de script y, luego, seleccionar **ADL: Compile Script** para compilar un trabajo de U-SQL. El resultado de la compilación aparece en el panel **Salida**.

### <a name="to-submit-a-u-sql-script"></a>Para enviar un script de U-SQL

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Submit Job**. También puede hacer clic con el botón derecho en un archivo de script y, luego, seleccionar **ADL: Submit Job**.

Después de enviar un trabajo de U-SQL, los registros de envío aparecen en la ventana **Salida** de VS Code. La vista del trabajo aparece en el panel derecho. Si el envío se realiza correctamente, la dirección URL del trabajo también se muestra. Puede abrir la dirección URL del trabajo en un explorador web para realizar el seguimiento de estado del trabajo en tiempo real.

En la pestaña **Resumen** de la vista del trabajo puede ver los detalles del trabajo. Las funciones principales incluyen la opción de reenviar un script, duplicarlo o abrirlo en el portal. En la pestaña **Datos** de la vista del trabajo puede consultar los archivos de entrada, los archivos de salida y los archivos de recursos. Los archivos se pueden descargar en el equipo local.

![Pestaña Resumen en la vista del trabajo](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-summary.png)

![Pestaña Datos en la vista del trabajo](./media/data-lake-analytics-data-lake-tools-for-vscode/job-view-data.png)

### <a name="to-set-the-default-context"></a>Para establecer el contexto predeterminado

Si no estableció los parámetros en sus archivos individualmente, puede establecer el contexto predeterminado para aplicar esta configuración a todos los archivos de script.

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Set Default Context**. Haga clic con el botón derecho en el editor de scripts y seleccione **ADL: Set Default Context**.
3. Elija la cuenta, la base de datos y el esquema que desee. La configuración se guarda en el archivo de configuración xxx_settings.json.

   ![Cuenta, base de datos y esquema establecidos como el contexto predeterminado](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-sequence.png)

### <a name="to-set-script-parameters"></a>Para establecer los parámetros del script

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Set Script Parameters**.
3. Se abre el archivo xxx_settings.json con las propiedades siguientes:

   - **account**: una cuenta de Azure Data Lake Analytics en la suscripción de Azure que se necesita para compilar y ejecutar los trabajos de U-SQL. Debe configurar la cuenta del equipo antes de compilar y ejecutar trabajos de U-SQL.
   - **database**: una base de datos de la cuenta. El valor predeterminado es **master**.
   - **schema**: un esquema de la base de datos. El valor predeterminado es **dbo**.
   - **optionalSettings**:
        - **priority**: el intervalo de prioridades va de 1 a 1000, siendo 1 la prioridad más alta. El valor predeterminado es **1000**.
        - **degreeOfParallelism**: el intervalo de paralelismo va de 1 a 150. El valor predeterminado es el paralelismo máximo que se permite en su cuenta de Azure Data Lake Analytics.

   ![Contenido del archivo JSON](./media/data-lake-analytics-data-lake-tools-for-vscode/default-context-setting.png)

> [!NOTE]
> Una vez guardada la configuración, la información de la cuenta, de la base de datos y del esquema aparece en la barra de estado que se encuentra en la esquina inferior izquierda del archivo .usql correspondiente, si no tiene configurado un contexto predeterminado.

### <a name="to-set-git-ignore"></a>Para establecer Git ignore

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Set Git Ignore**.

   - Si el archivo **.gitIgnore** no existe en la carpeta de trabajo de VS Code, se creará un archivo llamado **.gitIgnore** en la carpeta. De forma predeterminada, se agregarán al archivo cuatro elementos (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache** y **obj**). Puede realizar más actualizaciones si es necesario.
   - Si ya hay un archivo **.gitIgnore** en la carpeta de trabajo de VS Code, la herramienta agregará cuatro elementos (**usqlCodeBehindReference**, **usqlCodeBehindGenerated**, **.cache** y **obj**) al archivo **.gitIgnore** si los cuatro elementos no se encuentran en el archivo.

   ![Elementos del archivo .gitIgnore](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-gitignore.png)

## <a name="work-with-code-behind-files-c-sharp-python-and-r"></a>Trabajo con archivos de código subyacente: C#, Python y R

Herramientas de Azure Data Lake admite varios códigos personalizados. Para obtener instrucciones, consulte [Desarrollo de trabajos U-SQL con Python, R y CSharp para Azure Data Lake Analytics en VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md).

## <a name="work-with-assemblies"></a>Trabajo con ensamblados

Para información sobre el desarrollo de ensamblados, consulte [Desarrollo de ensamblados U-SQL para trabajos de Azure Data Lake Analytics]().

Puede usar Herramientas de Data Lake para registrar ensamblados de código personalizados en el catálogo de Data Lake Analytics.

### <a name="to-register-an-assembly"></a>Para registrar un ensamblado

Puede registrar el ensamblado mediante el comando **ADL: Register Assembly** o **ADL: Register Assembly (Advanced)** .

### <a name="to-register-through-the-adl-register-assembly-command"></a>Para realizar el registro mediante el comando ADL: Register Assembly

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Register Assembly**.
3. Especifique la ruta de acceso de ensamblado local.
4. Seleccione una cuenta de Data Lake Analytics.
5. Seleccione una base de datos.

El portal se abre en un explorador y muestra el proceso de registro del ensamblado.  

Una manera más cómoda de desencadenar el comando **ADL: Register Assembly** es hacer clic con el botón derecho en el archivo .dll en el Explorador de archivos.

### <a name="to-register-through-the-adl-register-assembly-advanced-command"></a>Para realizar el registro mediante el comando ADL: Register Assembly (Advanced)

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.
2. Escriba **ADL: Register Assembly (Advanced)**
3. Especifique la ruta de acceso de ensamblado local.
4. Se muestra el archivo JSON. Revise y edite las dependencias de ensamblado y los parámetros de recurso si es necesario. Las instrucciones se muestran en la ventana **Salida**. Para continuar con el registro del ensamblado, guarde (Ctrl+S) el archivo JSON.

   ![Archivo JSON con las dependencias de ensamblado y los parámetros de recursos](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-register-assembly-advance.png)

>[!NOTE]
>
>- Herramientas de Azure Data Lake detecta automáticamente si el archivo DLL tiene alguna dependencia de ensamblado. Las dependencias se muestran en el archivo JSON una vez que se detectan.
>- Puede cargar los recursos DLL (por ejemplo, .txt, .png y .csv) como parte del registro del ensamblado.

Otra manera de desencadenar el comando **ADL: Register Assembly (Advanced)** es hacer clic con el botón derecho en el archivo .dll en el Explorador de archivos.

El siguiente código de U-SQL muestra cómo llamar a un ensamblado. En el ejemplo, el nombre del ensamblado es *test*.

```usql
REFERENCE ASSEMBLY [test];
@a =
    EXTRACT
        Iid int,
    Starts DateTime,
    Region string,
    Query string,
    DwellTime int,
    Results string,
    ClickedUrls string
    FROM @"Sample/SearchLog.txt"
    USING Extractors.Tsv();
@d =
    SELECT DISTINCT Region
    FROM @a;
@d1 =
    PROCESS @d
    PRODUCE
        Region string,
    Mkt string
    USING new USQLApplication_codebehind.MyProcessor();
OUTPUT @d1
    TO @"Sample/SearchLogtest.txt"
    USING Outputters.Tsv();
```

## <a name="use-u-sql-local-run-and-local-debug-for-windows-users"></a>Uso de la ejecución y depuración locales de U-SQL para usuarios de Windows

La ejecución local de U-SQL prueba los datos locales y valida el script de manera local antes de publicar el código en Data Lake Analytics. Puede usar la característica de depuración local para completar las tareas siguientes antes de enviar el código a Data Lake Analytics:

- Depure el código subyacente de C#.
- Recorra el código.
- Valide localmente el script.

Las características de ejecución y depuración locales solo funcionan en entornos Windows y no se admiten en macOS y sistemas operativos basados en Linux.

Para instrucciones sobre la ejecución y la depuración locales, consulte [Ejecución y depuración locales de U-SQL con Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md).

## <a name="connect-to-azure"></a>Conexión con Azure

Para poder compilar y ejecutar scripts de U-SQL en Data Lake Analytics, debe conectarse a su cuenta de Azure.

<a id="sign-in-by-command"></a>

### <a name="to-connect-to-azure-by-using-a-command"></a>Para conectarse a Azure mediante un comando

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos.

2. Escriba **ADL: Login** (HDInsight: inicio de sesión). La información de inicio de sesión aparece en la esquina inferior derecha.

   ![Entrada del comando de inicio de sesión](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login.png)

   ![Notificaciones sobre el inicio de sesión y la autenticación](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-login-info.png)

3. Seleccione **Copiar y abrir** para abrir la [página de inicio de sesión](https://aka.ms/devicelogin). Pegue el código en el cuadro de texto y seleccione **Continuar**.

    ![Página de inicio de sesión](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-extension-login-paste-code.png)  

4. Siga las instrucciones para iniciar sesión desde la página web. Cuando se conecte, el nombre de la cuenta de Azure aparecerá en la barra de estado que se encuentra en la esquina inferior izquierda de la ventana de VS Code.

> [!NOTE]
>
> - Herramientas de Data Lake iniciará sesión automáticamente la próxima vez si no cierra la sesión.
> - Si la cuenta tiene habilitada la autenticación de dos pasos, se recomienda usar la autenticación por teléfono en lugar de usar un PIN.

Para cerrar la sesión, escriba el comando **ADL: Logout**.

### <a name="to-connect-to-azure-from-the-explorer"></a>Para conectarse a Azure desde el explorador

Expanda **Azure Data Lake**, seleccione **Iniciar sesión en Azure** y, a continuación, siga los pasos 3 y 4 de [Para conectarse a Azure mediante un comando](#sign-in-by-command).

![Selección de "Iniciar sesión en Azure" en el explorador](./media/data-lake-analytics-data-lake-tools-for-vscode/data-lake-tools-for-vscode-sign-in-from-explorer.png )  

No se puede cerrar la sesión desde el explorador. Para cerrar sesión, consulte [Para conectarse a Azure mediante un comando](#sign-in-by-command).

## <a name="create-an-extraction-script"></a>Creación de un script de extracción

Puede crear un script de extracción para los archivos .csv, .tsv y .txt mediante el comando **ADL: Create EXTRACT Script** o desde el explorador de Azure Data Lake.

### <a name="to-create-an-extraction-script-by-using-a-command"></a>Para crear un script de extracción mediante un comando

1. Seleccione Ctrl+Mayús+P para abrir la paleta de comandos y escriba **ADL: Create EXTRACT Script**.
2. Especifique la ruta de acceso completa de un archivo de Azure Storage y presione Entrar.
3. Seleccione una cuenta.
4. Para un archivo .txt, seleccione un delimitador para extraer el archivo.

![Proceso de creación de un script de extracción](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-process.png)

El script de extracción se genera en función de sus entradas. Para un script que no puede detectar las columnas, elija una de las dos opciones. Si no lo hace así, se generará un solo script.

![Resultado de la creación de un script de extracción](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-result.png)

### <a name="to-create-an-extraction-script-from-the-explorer"></a>Para crear un script de extracción desde el explorador

Otra forma de crear un script de extracción es haciendo clic con el botón derecho (menú contextual) en el archivo .csv, .tsv o .txt en Azure Data Lake Store o Azure Blob Storage.

![Comando "Create EXTRACT Script" en el menú contextual](./media/data-lake-analytics-data-lake-tools-for-vscode/create-extract-script-from-context-menu.png)

## <a name="next-steps"></a>Pasos siguientes

- [Desarrollo de trabajos U-SQL con Python, R y CSharp para Azure Data Lake Analytics en VS Code](data-lake-analytics-u-sql-develop-with-python-r-csharp-in-vscode.md)
- [Ejecución y depuración locales de U-SQL con Visual Studio Code](data-lake-tools-for-vscode-local-run-and-debug.md)
- [Tutorial: Introducción a Azure Data Lake Analytics](data-lake-analytics-get-started-portal.md)
- [Tutorial: Desarrollo de scripts U-SQL mediante Data Lake Tools para Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
