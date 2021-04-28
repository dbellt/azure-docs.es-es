---
title: 'Implementación del servicio de administración de certificados OPC Vault: Azure | Microsoft Docs'
description: En este artículo se explica cómo implementar el servicio de administración de certificados OPC Vault desde cero.
author: mregen
ms.author: mregen
ms.date: 08/16/2019
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: b20f6318c2e6be701446e29ab93598752e93d287
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107870290"
---
# <a name="build-and-deploy-the-opc-vault-certificate-management-service"></a>Compilación e implementación del servicio de administración de certificados de OPC Vault

> [!IMPORTANT]
> Mientras actualizamos este artículo, consulte el artículo sobre [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/), donde encontrará el contenido más actualizado.

En este artículo se explica cómo implementar el servicio de administración de certificados de OPC Vault en Azure.

> [!NOTE]
> Para más información, consulte el [repositorio de OPC Vault](https://github.com/Azure/azure-iiot-opc-vault-service) de GitHub.

## <a name="prerequisites"></a>Requisitos previos

### <a name="install-required-software"></a>Instalación del software necesario

Actualmente, la operación de compilación e implementación está limitada a Windows.
Todos los ejemplos están escritos para C# .NET Standard, que se necesita para compilar el servicio y los ejemplos para la implementación.
Todas las herramientas que necesita para .NET Standard se incluyen con las herramientas de .NET Core. Consulte [Introducción a .NET Core](/dotnet/articles/core/getting-started).

1. [Instale .NET Core 2.1+][dotnet-install].
2. [Instale Docker][docker-url] (opcional, solo si se necesita la compilación de Docker local).
4. Instale las [herramientas de línea de comandos de Azure para PowerShell][powershell-install].
5. Regístrese para obtener una [suscripción de Azure][azure-free].

### <a name="clone-the-repository"></a>Clonación del repositorio

Si todavía no lo ha hecho, clone este repositorio de GitHub. Abra un símbolo del sistema o un terminal y ejecute lo siguiente:

```bash
git clone https://github.com/Azure/azure-iiot-opc-vault-service
cd azure-iiot-opc-vault-service 
```

También, puede clonar el repositorio directamente en Visual Studio 2017.

### <a name="build-and-deploy-the-azure-service-on-windows"></a>Compilación e implementación del servicio de Azure en Windows

Un script de PowerShell proporciona una manera sencilla de implementar la aplicación y el microservicio de OPC Vault.

1. Abra una ventana de PowerShell en la raíz del repositorio. 
3. Vaya a la carpeta de implementación `cd deploy`.
3. Elija un nombre para `myResourceGroup` que no sea probable que provoque un conflicto con otras páginas web implementadas. Consulte la sección "El nombre del sitio web ya está en uso" más adelante en este artículo.
5. Inicie la implementación con `.\deploy.ps1` para la instalación interactiva, o escriba una línea de comandos completa:  
`.\deploy.ps1  -subscriptionName "MySubscriptionName" -resourceGroupLocation "East US" -tenantId "myTenantId" -resourceGroupName "myResourceGroup"`
7. Si planea desarrollar con esta implementación, agregue `-development 1` para habilitar la interfaz de usuario de Swagger e implementar las compilaciones de depuración.
6. Siga las instrucciones del script para iniciar sesión en la suscripción y proporcionar información adicional.
9. Después de que una operación de compilación e implementación se realice correctamente, debería ver el mensaje siguiente:
   ```
   To access the web client go to:
   https://myResourceGroup.azurewebsites.net

   To access the web service go to:
   https://myResourceGroup-service.azurewebsites.net

   To start the local docker GDS server:
   .\myResourceGroup-dockergds.cmd

   To start the local dotnet GDS server:
   .\myResourceGroup-gds.cmd
   ```

   > [!NOTE]
   > En caso de problemas, consulte la sección "Solución de problemas con errores de implementación" más adelante en el artículo.

8. Abra su explorador favorito y abra la página de la aplicación: `https://myResourceGroup.azurewebsites.net`.
8. Dé a la aplicación web y al microservicio OPC Vault unos minutos para prepararse después de la implementación. Es posible que la página web principal deje de responder en el primer uso, durante hasta un minuto, hasta que obtenga las primeras respuestas.
11. Para echar un vistazo a la API de Swagger, abra `https://myResourceGroup-service.azurewebsites.net`.
13. Para iniciar un servidor GDS local con dotnet, inicie `.\myResourceGroup-gds.cmd`. Con Docker, inicie `.\myResourceGroup-dockergds.cmd`.

Es posible volver a implementar una compilación con exactamente la misma configuración. Tenga en cuenta que una operación de este tipo renueva todos los secretos de la aplicación y puede restablecer algunos valores de configuración de los registros de aplicación de Azure Active Directory (Azure AD).

También es posible volver a implementar solo los archivos binarios de la aplicación web. Con el parámetro `-onlyBuild 1`, se implementan nuevos paquetes zip del servicio y la aplicación en las aplicaciones web.

Después de una implementación correcta, puede empezar a usar los servicios. Consulte [Administración del servicio de administración de certificados de OPC Vault](howto-opc-vault-manage.md).

## <a name="delete-the-services-from-the-subscription"></a>Eliminación de los servicios desde la suscripción

A continuación, se indica cómo puede hacerlo.

1. Inicie sesión en [Azure Portal](https://portal.azure.com).
2. Vaya al grupo de recursos en el que se implementó el servicio.
3. Seleccione **Eliminar grupo de recursos** y confirme.
4. Después de unos minutos, se eliminan todos los componentes de servicio implementados.
5. Seleccione **Azure Active Directory** > **Registros de aplicaciones**.
6. Debe haber tres registros enumerados para cada grupo de recursos implementado. Los registros tienen los nombres siguientes: `resourcegroup-client`, `resourcegroup-module`, `resourcegroup-service`. Elimine cada registro por separado.

Ahora se quitan todos los componentes implementados.

## <a name="troubleshooting-deployment-failures"></a>Solución de problemas con errores de implementación

### <a name="resource-group-name"></a>Definición de un nombre de grupo de recursos

Utilice un nombre de grupo de recursos corto y sencillo. El nombre también se usa para asignar un nombre a los recursos y al prefijo de la dirección URL del servicio. Como tal, debe cumplir los requisitos de nomenclatura de los recursos.  

### <a name="website-name-already-in-use"></a>El nombre del sitio web ya está en uso

Es posible que el nombre del sitio web ya esté en uso. Debe usar un nombre de grupo de recursos diferente. Los nombres de host que usa el script de implementación son: https:\//resourcegroupname.azurewebsites.net and https:\//resourgroupname-service.azurewebsites.net.
Otros nombres de los servicios se compilan mediante la combinación de hashes de nombre corto y no es probable que entren en conflicto con otros servicios.

### <a name="azure-ad-registration"></a>Registro de Azure AD 

El script de implementación intenta registrar tres aplicaciones de Azure AD en Azure AD. En función de los permisos del inquilino de Azure AD seleccionado, es posible que se produzca un error en esta operación. Hay dos opciones:

- Si ha elegido un inquilino de Azure AD de una lista de inquilinos, reinicie el script y elija otro de la lista.
- Como alternativa, implemente un inquilino de Azure AD privado en otra suscripción. Reinicie el script y seleccione usarlo.

## <a name="deployment-script-options"></a>Opciones del script de implementación

El script usa los parámetros siguientes:


```
-resourceGroupName
```

Puede ser el nombre de un grupo de recursos nuevo o existente.

```
-subscriptionId
```


Es el identificador de la suscripción en la que se implementarán los recursos. Es opcional.

```
-subscriptionName
```


Como alternativa, puede usar el nombre de la suscripción.

```
-resourceGroupLocation
```


Es una ubicación del grupo de recursos. Si se especifica, este parámetro intenta crear un nuevo grupo de recursos en esta ubicación. Este parámetro también es opcional.


```
-tenantId
```


Este es el identificador del inquilino de Azure AD que se va a usar. 

```
-development 0|1
```

Esto se va a implementar para el desarrollo. Use la compilación de depuración y establezca el entorno de ASP.NET en Desarrollo. Cree `.publishsettings` para la importación en Visual Studio 2017 con el fin de permitir que se implementen directamente la aplicación y el servicio. Este parámetro también es opcional.

```
-onlyBuild 0|1
```

Esto es para recompilar y volver a implementar solo las aplicaciones web y para recompilar los contenedores de Docker. Este parámetro también es opcional.

[azure-free]:https://azure.microsoft.com/free/
[powershell-install]:https://azure.microsoft.com/downloads/#powershell
[docker-url]: https://www.docker.com/
[dotnet-install]: https://dotnet.microsoft.com/download

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a implementar OPC Vault desde cero, puede seguir este paso:

> [!div class="nextstepaction"]
> [Administración de OPC Vault](howto-opc-vault-manage.md)