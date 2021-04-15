---
title: Error de inicio de instancia de rol para Azure Cloud Services (soporte extendido)
description: Solución de problemas de inicio de instancia de rol para Azure Cloud Services (soporte extendido)
ms.topic: article
ms.service: cloud-services-extended-support
author: surbhijain
ms.author: surbhijain
ms.reviewer: gachadw
ms.date: 04/01/2021
ms.custom: ''
ms.openlocfilehash: ced7675350c0e0deadf77837cf0f13ba54fffab9
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106382362"
---
# <a name="troubleshoot-azure-cloud-service-extended-support-roles-that-fail-to-start"></a>Solución de problemas de roles de Azure Cloud Services (soporte extendido) que no se inician
Presentamos algunos problemas y soluciones comunes relacionados con los roles de Cloud Services (soporte extendido) que no se inician.

## <a name="cloud-service-operation-failed-with-roleinstancestartuptimeouterror"></a>Error en la operación de servicio en la nube con RoleInstanceStartupTimeoutError
Es posible que una o varias instancias de rol del servicio en la nube (soporte extendido) no se inicien en el tiempo estipulado. Es posible que estas instancias de rol tarden un tiempo en iniciarse o se estén reciclando, y la instancia de rol podría producir un error con un RoleInstanceStartupTimeoutError. Se trata de un error de aplicación de rol. La aplicación de roles contiene dos partes principales: "Tareas de inicio"' y "Código de rol (implementación de RoleEntryPoint)", y ambas pueden provocar el reciclaje de roles. Si se bloquea el rol, el agente PaaS siempre lo volverá a iniciar. 

Para obtener el estado actual y los detalles de las instancias de rol en caso de que se produzcan errores, utilice:

* PowerShell: use el cmdlet [Get-AzCloudServiceRoleInstanceView](https://docs.microsoft.com/powershell/module/az.cloudservice/get-azcloudserviceroleinstanceview) para recuperar información sobre el estado de tiempo de ejecución de una instancia de rol en un servicio en la nube.
```powershell
Get-AzCloudServiceRoleInstanceView -ResourceGroupName "ContosOrg" -CloudServiceName "ContosoCS" -RoleInstanceName "WebRole1_IN_0"
 
Statuses           PlatformFaultDomain PlatformUpdateDomain
--------           ------------------- --------------------
{RoleStateStarting} 0                   0
```

* Azure Portal: vaya a su servicio en la nube y seleccione la pestaña Roles e instancias. Haga clic en la instancia de rol para obtener los detalles de su estado. 
 
   :::image type="content" source="media/role-startup-failure-1.png" alt-text="La imagen muestra el error de inicio del rol en el portal.":::
   
Estos son algunos problemas comunes y soluciones relacionados con los roles de Azure Cloud Services (soporte extendido) que no se inician, o recorre los estados de inicialización, ocupado y detención.

## <a name="missing-dlls-or-dependencies"></a>DLL o dependencias que faltan
Tanto que los roles no respondan como que alternen entre los estados Inicializando, Ocupado y Deteniendo puede deberse a que faltan DLL o ensamblados.
Estos son los síntomas de que faltan DLL o ensamblados:

* Una instancia de rol alterna entre los estados **Inicializando**, **Ocupado** y **Deteniendo**.
* La instancia de rol se ha movido al estado **Listo** , pero si se navega a la aplicación web, la página no aparece.

Hay varios métodos recomendados para investigar estos problemas.

## <a name="diagnose-missing-dll-issues-in-a-web-role"></a>Diagnóstico del problema de DLL que faltan en un rol web
Cuando se navega a un sitio web que se implementa en un rol web y el explorador muestra un error de servidor similar al siguiente, puede significar que falta alguna DLL.

:::image type="content" source="media/role-startup-failure-2.png" alt-text="La imagen muestra un error en tiempo de ejecución en un error de inicio de rol.":::

## <a name="diagnose-issues-by-turning-off-custom-errors"></a>Diagnóstico de problemas mediante la desactivación de errores personalizados
Para ver una información más completa de los errores, configure el archivo web.config del rol web estableciendo el modo de error personalizado en desactivado y vuelva a implementar el servicio.
Para ver más errores completos sin usar Escritorio remoto:
1.  Abra la solución en Microsoft Visual Studio.
2.  En el Explorador de soluciones, busque el archivo web.config y ábralo.
3.  En el archivo web.config, busque la sección system.web y agregue la siguiente línea:
 ```xml
<customErrors mode="Off" />
```
4.  Guarde el archivo.
5.  Vuelva a empaquetar e implementar el servicio.
Una vez que se vuelva a implementar el servicio, verá un mensaje de error con el nombre del ensamblado o DLL que faltan.

## <a name="diagnose-issues-by-viewing-the-error-remotely"></a>Diagnóstico de problemas mediante la comprobación del error de forma remota
Puede usar Escritorio remoto para acceder al rol y ver información más completa sobre los errores de forma remota. Use los pasos siguientes para ver los errores desde Escritorio remoto:
1.  Habilite la extensión de Escritorio remoto para Cloud Services (soporte extendido). Para más información, consulte [Aplicación de la extensión de Escritorio remoto a Cloud Services (soporte extendido) mediante Azure Portal](enable-rdp.md)
2.  En Azure Portal, cuando la instancia muestre el estado Preparado, establezca una conexión remota a la instancia. Para más información sobre cómo usar el escritorio remoto con Cloud Services (soporte extendido), consulte [Conexión a instancias de rol con Escritorio remoto](https://docs.microsoft.com/azure/cloud-services-extended-support/enable-rdp#connect-to-role-instances-with-remote-desktop-enabled).
3.  Inicie sesión en la máquina virtual con las credenciales especificadas en la configuración de Escritorio remoto.
4.  Abra el símbolo del sistema.
5.  Escriba IPconfig.
6.  Anote el valor de dirección IPV4.
7.  Abra Internet Explorer.
8.  Escriba la dirección y el nombre de la aplicación web. Por ejemplo, http://<IPV4 Address>/default.aspx.
Si navega al sitio web, se devolverán mensajes de error más explícitos:
* Error del servidor en la aplicación '/'
* Descripción: Se produjo una excepción no controlada durante la ejecución de la solicitud web actual. Revise el seguimiento de la pila para obtener más información acerca del error y en donde se originó en el código.
* Detalles de la excepción: System.IO.FIleNotFoundException: No se puede cargar el archivo o ensamblado ‘Microsoft.WindowsAzure.StorageClient, Version=1.1.0.0, Culture=neutral, PublicKeyToken=31bf856ad364e35’ ni una de sus dependencias. El sistema no encuentra el archivo especificado.
Por ejemplo:

  :::image type="content" source="media/role-startup-failure-3.png" alt-text="La imagen muestra una excepción en un error de inicio de rol.":::
  
## <a name="diagnose-issues-by-using-the-compute-emulator"></a>Diagnóstico de problemas mediante el emulador de proceso
El emulador de proceso de Azure se puede usar para diagnosticar y solucionar problemas de errores en web.config y dependencias que faltan.
Para obtener unos mejores resultados con este método de diagnóstico, debe usar un equipo o máquina virtual que tenga una instalación limpia de Windows. 
1.  Instalación del [SDK de Azure](https://azure.microsoft.com/downloads/) 
2.  En la máquina de desarrollo, compile el proyecto de servicio en la nube.
3.  En el Explorador de Windows, desplácese a la carpeta bin\debug del proyecto del servicio en la nube.
4.  Copie la carpeta .csx y el archivo .cscfg en el equipo que se usa para depurar los problemas.
5.  En la máquina limpia, abra una ventana del símbolo del sistema de Azure SDK y escriba csrun.exe /devstore:start.
6.  En el símbolo del sistema escriba run csrun <ruta de la carpeta .csx> <ruta del archivo .cscfg> /launchBrowser.
7.  Cuando se inicie el rol, verá información detallada del error en Internet Explorer. También puede usar las herramientas de solución de problemas estándar de Windows para efectuar un diagnóstico exhaustivo del problema.

## <a name="diagnose-issues-by-using-intellitrace"></a>Diagnóstico de problemas mediante IntelliTrace
Para los roles web y de trabajo que usan .NET Framework 4, puede usar [IntelliTrace](https://docs.microsoft.com/visualstudio/debugger/intellitrace), que está disponible en Microsoft Visual Studio Enterprise.
Siga estos pasos para implementar el servicio con IntelliTrace habilitado:
1.  Confirme que está instalado el SDK 1.3 de Azure, o una versión posterior.
2.  Implemente la solución con Visual Studio. Durante la implementación, active la casilla Habilitar IntelliTrace para roles de .NET 4 .
3.  Una vez se inicia la instancia, abra el Explorador de servidores.
4.  Expanda el nodo Azure\Cloud Services y busque la implementación.
5.  Expanda la implementación hasta que vea las instancias de rol. Haga clic con el botón derecho en una de las instancias.
6.  Elija Ver registros de IntelliTrace. Se abrirá Resumen de IntelliTrace.
7.  Busque la sección de excepciones del resumen. Si hay excepciones, se denominará Datos de excepción.
8.  Expanda Datos de excepción y busque los errores de System.IO.FileNotFoundException, similares a lo siguiente:

    :::image type="content" source="media/role-startup-failure-4.png" alt-text="La imagen muestra datos de excepción en un error de inicio de rol." lightbox="media/role-startup-failure-4.png":::

## <a name="address-missing-dlls-and-assemblies"></a>Tratamiento de archivos DLL y ensamblados que faltan
Para tratar los errores de ensamblado y de DLL que faltan, siga estos pasos:
1.  Abra la solución en Visual Studio.
2.  En el Explorador de soluciones, abra la carpeta Referencias.
3.  Haga clic en el ensamblado identificado en el error.
4.  En el panel Propiedades, busque la propiedad Copy Local y establezca su valor en True.
5.  Vuelva a implementar el servicio en la nube.
Cuando haya comprobado que se han corregido todos los errores, se puede implementar el servicio sin activar la casilla Habilitar IntelliTrace para roles de .NET 4 .

## <a name="next-steps"></a>Pasos siguientes 
- Para más información acerca de cómo solucionar los problemas de los roles de los servicios en la nube mediante el uso de datos de diagnóstico de equipos de PaaS de Azure, consulte la [serie de blogs de Kevin Williamson](https://docs.microsoft.com/archive/blogs/kwill/windows-azure-paas-compute-diagnostics-data).
