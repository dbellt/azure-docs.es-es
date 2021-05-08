---
title: Error de inicio de la instancia de rol para Azure Cloud Services (soporte extendido)
description: Solución de problemas de inicio de instancia de rol en Azure Cloud Services (soporte extendido).
ms.topic: article
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachadw
ms.date: 04/01/2021
ms.custom: ''
ms.openlocfilehash: 9c6fdee8dbb28e86e5084c9c0cfca97f5ac5ea05
ms.sourcegitcommit: fc9fd6e72297de6e87c9cf0d58edd632a8fb2552
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/30/2021
ms.locfileid: "108287165"
---
# <a name="troubleshoot-azure-cloud-services-extended-support-roles-that-fail-to-start"></a>Solución de problemas de roles de Azure Cloud Services (soporte extendido) que no se inician

Presentamos algunos problemas y soluciones comunes relacionados con los roles de Azure Cloud Services (soporte extendido) que no se inician.

## <a name="cloud-service-operation-fails-with-roleinstancestartuptimeouterror"></a>Error en la operación de servicio en la nube con RoleInstanceStartupTimeoutError

Una o varias de las instancias de rol en Azure Cloud Services (soporte extendido) pueden tardar en iniciarse; también es posible que la instancia de rol devuelva un error mientras el resto se reciclan. Aparece el error `RoleInstanceStartupTimeoutError` de la aplicación de rol.

La aplicación de rol contiene dos partes que pueden provocar el reciclaje de roles: las *Tareas de inicio* y el *Código de rol (implementación de RoleEntryPoint)* . 

Si el rol se detiene, el agente de plataforma como servicio (PaaS) reinicia el rol.

Puede obtener el estado actual y los detalles de una instancia de rol para diagnosticar errores mediante PowerShell o Azure Portal:

* **PowerShell**: use el cmdlet [Get-AzCloudServiceRoleInstanceView](/powershell/module/az.cloudservice/get-azcloudserviceroleinstanceview) para recuperar información sobre el estado del entorno de ejecución de la instancia de rol:

    ```powershell
    Get-AzCloudServiceRoleInstanceView -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstanceName "WebRole1_IN_0"
     
    Statuses           PlatformFaultDomain PlatformUpdateDomain
    --------           ------------------- --------------------
    {RoleStateStarting} 0                   0
    ```

* **Azure Portal**: en el portal, vaya a la instancia del servicio en la nube. Para ver los detalles de estado, seleccione **Roles e instancias** y, a continuación, seleccione la instancia de rol.

  :::image type="content" source="media/role-startup-failure-portal.png" alt-text="Captura de pantalla que muestra un error de inicio de rol en Azure Portal.":::

## <a name="missing-dlls-or-dependencies"></a>DLL o dependencias que faltan

Es posible que la falta de DLL o los ensamblados sean la causa de que los roles que no respondan o que cambien sus estados.

Estos son los síntomas de que faltan DLL o ensamblados:

* Una instancia de rol alterna entre los estados **Inicializando**, **Ocupado** y **Deteniendo**.
* La instancia de rol se ha movido al estado **Listo**, pero si va a la aplicación web, la página no aparece.


Un sitio web que se ha implementado en un rol web y un archivo DLL que falta podrían mostrar este error en el entorno de ejecución del servidor:

  :::image type="content" source="media/role-startup-failure-runtime-error.png" alt-text="Captura de pantalla que muestra un error en el entorno de ejecución después de un error de inicio de rol.":::

### <a name="resolve-missing-dlls-and-assemblies"></a>Solución para los archivos DLL y ensamblados que faltan

Para resolver los errores de los archivos DLL y ensamblados que faltan:

1. En Visual Studio, abra la solución.
2. En el Explorador de soluciones, abra la carpeta *Referencias*.
3. Seleccione el ensamblado que se identifica en el error.
4. En **Propiedades**, establezca la propiedad **Copiar de forma local** en **Verdadero**.
5. Vuelva a implementar el servicio en la nube.

Después de comprobar que los errores ya no aparecen, vuelva a implementar el servicio. Al configurar la implementación, no active la casilla **Habilitar IntelliTrace para roles de .NET 4**.

## <a name="diagnose-role-instance-errors"></a>Diagnóstico de errores de instancia de rol

Elija entre las siguientes opciones para diagnosticar problemas con las instancias de rol.

### <a name="turn-off-custom-errors"></a>Desactivación de errores personalizados

Para ver la información de error completa, en el archivo *web.config* del rol web, configure el modo de error personalizado en `off` y, a continuación, vuelva a implementar el servicio:

1. En Visual Studio, abra la solución.
2. En el Explorador de soluciones, abra el archivo *web.config*.
3. En la sección `system.web`, agregue el siguiente código:

   ```xml
   <customErrors mode="Off" />
   ```

4. Guarde el archivo.
5. Vuelva a empaquetar e implementar el servicio.

Cuando se vuelve a implementar el servicio, un mensaje de error incluye el nombre de los ensamblados o archivos DLL que faltan.

### <a name="use-remote-desktop"></a>Uso de Escritorio Remoto

Puede usar el Escritorio remoto para acceder al rol y ver información completa sobre el error:

1. [Adición de la extensión del Escritorio remoto en Azure Cloud Services (soporte extendido)](enable-rdp.md).
2. En Azure Portal, cuando la instancia del servicio en la nube muestre el estado **Listo**, use el Escritorio remoto para iniciar sesión en el servicio en la nube. Para obtener más información, consulte [Conectarse a las instancias de rol mediante el Escritorio remoto](enable-rdp.md#connect-to-role-instances-with-remote-desktop-enabled).
3. Inicie sesión en la máquina virtual mediante las credenciales que usó para configurar el Escritorio remoto.
4. Abra una ventana de símbolo del sistema.
5. En el símbolo del sistema, escriba **ipconfig**. Anote el valor devuelto para la dirección IPv4.
6. Abra Microsoft Internet Explorer.
7. En la barra de direcciones, escriba la dirección IPv4 seguida de una barra diagonal y el nombre del archivo predeterminado de la aplicación web. Por ejemplo, `http://<IPv4 address>/default.aspx`.

Si va al sitio web, verá mensajes de error que contienen más información. Este es un ejemplo:

:::image type="content" source="media/role-startup-failure-error-message.png" alt-text="Captura de pantalla que muestra un ejemplo de un mensaje de error.":::
  
### <a name="use-the-compute-emulator"></a>Uso del emulador de proceso

El emulador de proceso de Azure se puede usar para diagnosticar y solucionar problemas de errores en *web.config* y las dependencias que faltan. Cuando use este método para diagnosticar problemas, si quiere obtener los mejores resultados, use un equipo o una máquina virtual que tenga una instalación limpia de Windows.

Para diagnosticar problemas mediante el emulador de proceso de Azure:

1. Instale el [SDK de Azure](https://azure.microsoft.com/downloads/).
2. En el equipo de desarrollo, compile el proyecto de servicio en la nube.
3. En el Explorador de archivos, en el proyecto de servicio en la nube, vaya a la carpeta *bin\debug*.
4. Copie la carpeta *.csx* y el archivo *.cscfg* en el equipo que se usa para depurar los problemas.
5. En la máquina limpia, abra una ventana del símbolo del sistema del SDK de Azure.
6. En el símbolo del sistema, escriba **csrun.exe /devstore:start**.
7. A continuación, escriba **run csrun \<path to .csx folder\> \<path to .cscfg file\> /launchBrowser**.
8. Cuando se inicia el rol, Internet Explorer muestra información detallada del error.

Si necesita más información de diagnóstico, puede usar las herramientas estándar de solución de problemas de Windows.

### <a name="use-intellitrace"></a>Uso de IntelliTrace

Para los roles de trabajo y web que usan .NET Framework 4, puede usar [IntelliTrace](/visualstudio/debugger/intellitrace). IntelliTrace está disponible en Visual Studio Enterprise.

Para implementar el servicio en la nube con IntelliTrace activado:

1. Confirme que está instalado el SDK 1.3 de Azure, o una versión posterior.
2. En Visual Studio, implemente la solución. Al configurar la implementación, seleccione la casilla **Habilitar IntelliTrace para roles de .NET 4**.
3. Una vez que se inicie la instancia de rol, abra el Explorador de servidores.
4. Expanda el nodo **Azure\Cloud Services**.
5. Expanda la implementación para enumerar las instancias de rol. Haga clic con el botón derecho en una instancia de rol.
6. Seleccione **Ver registros de IntelliTrace**.
7. En **Resumen de IntelliTrace**, vaya a **Datos de excepción**.
8. Expanda la opción de los **Datos de excepción** y busque un error `System.IO.FileNotFoundException`:

   :::image type="content" source="media/role-startup-failure-exception-data.png" alt-text="Captura de pantalla de los datos de excepción de un error de inicio de rol." lightbox="media/role-startup-failure-exception-data.png":::

## <a name="next-steps"></a>Pasos siguientes

- Para obtener más información, consulte [Cómo solucionar los problemas de los roles de los servicios en la nube mediante el uso de datos de diagnóstico de equipos de PaaS de Azure](/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).