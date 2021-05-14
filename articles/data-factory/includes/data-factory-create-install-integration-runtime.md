---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 11/09/2018
ms.author: jingwang
ms.openlocfilehash: c0a0b44b3de088539a070f1182248c4e6db7303f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108771192"
---
## <a name="create-a-self-hosted-integration-runtime"></a>Creación de una instancia de Integration Runtime autohospedada

En esta sección se crea una instancia de Integration Runtime autohospedada y se asocia con un equipo local con la base de datos de SQL Server. El entorno de ejecución de integración autohospedado es el componente que copia los datos de SQL Server de la máquina a Azure SQL Database. 

1. Cree una variable para el nombre de Integration Runtime. Utilice un nombre único y anótelo. Lo usará más adelante en este tutorial. 

    ```powershell
   $integrationRuntimeName = "ADFTutorialIR"
    ```
2. Cree una instancia de Integration Runtime autohospedada. 

   ```powershell
   Set-AzDataFactoryV2IntegrationRuntime -Name $integrationRuntimeName -Type SelfHosted -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName
   ```

   Este es la salida de ejemplo:

   ```console
    Name              : <Integration Runtime name>
    Type              : SelfHosted
    ResourceGroupName : <ResourceGroupName>
    DataFactoryName   : <DataFactoryName>
    Description       : 
    Id                : /subscriptions/<subscription ID>/resourceGroups/<ResourceGroupName>/providers/Microsoft.DataFactory/factories/<DataFactoryName>/integrationruntimes/ADFTutorialIR
    ```
  
3. Para recuperar el estado de la instancia de Integration Runtime creada, ejecute el siguiente comando. Confirme que el valor de la propiedad **Estado** está establecida en **NeedRegistration**. 

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntime -name $integrationRuntimeName -ResourceGroupName $resourceGroupName -DataFactoryName $dataFactoryName -Status
   ```

   Este es la salida de ejemplo:

   ```console  
   State                     : NeedRegistration
   Version                   : 
   CreateTime                : 9/24/2019 6:00:00 AM
   AutoUpdate                : On
   ScheduledUpdateDate       : 
   UpdateDelayOffset         : 
   LocalTimeZoneOffset       : 
   InternalChannelEncryption : 
   Capabilities              : {}
   ServiceUrls               : {eu.frontend.clouddatahub.net}
   Nodes                     : {}
   Links                     : {}
   Name                      : ADFTutorialIR
   Type                      : SelfHosted
   ResourceGroupName         : <ResourceGroup name>
   DataFactoryName           : <DataFactory name>
   Description               : 
   Id                        : /subscriptions/<subscription ID>/resourceGroups/<ResourceGroup name>/providers/Microsoft.DataFactory/factories/<DataFactory name>/integrationruntimes/<Integration Runtime name>
   ```

4. Para recuperar las claves de autenticación utilizadas para registrar la instancia de Integration Runtime autohospedado con el servicio Azure Data Factory en la nube, ejecute el siguiente comando: 

   ```powershell
   Get-AzDataFactoryV2IntegrationRuntimeKey -Name $integrationRuntimeName -DataFactoryName $dataFactoryName -ResourceGroupName $resourceGroupName | ConvertTo-Json
   ```

   Este es la salida de ejemplo:

   ```json
   {
    "AuthKey1": "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx=",
    "AuthKey2":  "IR@0000000000-0000-0000-0000-000000000000@xy0@xy@yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy="
   }
   ```    

5. Copie una de las claves (sin las comillas dobles) utilizadas para registrar la instancia de Integration Runtime autohospedado que instalará en el equipo en los pasos siguientes.  

## <a name="install-the-integration-runtime-tool"></a>Instalación de la herramienta Integration Runtime

1. Si ya tiene Integration Runtime en su equipo, desinstálelo utilizando **Agregar o quitar programas**. 

2. [Descargar](https://www.microsoft.com/download/details.aspx?id=39717) Integration Runtime autohospedado en un equipo de Windows local. Ejecución de la instalación.

3. En la página del **Asistente para la instalación de Microsoft Integration Runtime**, haga clic en **Siguiente**.

4. En la página del **Contrato de licencia para el usuario final**, acepte los términos del Contrato de licencia y seleccione **Siguiente**.

5. En la página **Carpeta de destino**, seleccione **Siguiente**.

6. En la página **Preparado para instalar Microsoft Integration Runtime**, seleccione **Instalar**.

7. En la página **Ha completado el Asistente para la instalación de Microsoft Integration Runtime**, seleccione **Finalizar**.

8. En la página **Registro de Integration Runtime (autohospedado)** , pegue la clave guardada en la sección anterior y seleccione en **Registrar**. 

    ![Registro de Integration Runtime](media/data-factory-create-install-integration-runtime/register-integration-runtime.png)

9. En la página **Nuevo nodo de Integration Runtime (autohospedado)** , seleccione **Finalizar**. 

10. Verá el siguiente mensaje cuando la instancia de Integration Runtime autohospedado se haya registrado correctamente:

    ![Se registró correctamente](media/data-factory-create-install-integration-runtime/registered-successfully.png)

14. En la página **Registro de Integration Runtime (autohospedado)** , seleccione **Iniciar Configuration Manager**.

15. Verá la siguiente página cuando el nodo esté conectado al servicio en la nube:

    ![Página El nodo está conectado](media/data-factory-create-install-integration-runtime/node-is-connected.png)

16. Ahora, pruebe la conectividad a la base de datos de SQL Server.

    ![Ficha Diagnóstico](media/data-factory-create-install-integration-runtime/config-manager-diagnostics-tab.png)   

    a. En la página **Configuration Manager**, vaya a la pestaña **Diagnósticos**.

    b. Seleccione **SqlServer** para el tipo de origen de datos.

    c. Escriba el nombre del servidor.

    d. Escriba el nombre de la base de datos.

    e. Seleccione el modo de autenticación.

    f. Escriba el nombre de usuario.

    g. Escriba la contraseña asociada con el nombre de usuario.

    h. Para confirmar que Integration Runtime puede conectarse a SQL Server, seleccione **Probar**. Verá una marca de verificación verde si la conexión es correcta. Verá un mensaje de error si la conexión no se realiza correctamente. Solucione los problemas y asegúrese de que Integration Runtime puede conectarse a SQL Server.    

    > [!NOTE]
    > Tome nota de los valores para la contraseña, el usuario, la base de datos, el servidor y el tipo de autenticación. Los usará más adelante en este tutorial.
