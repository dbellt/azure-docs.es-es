---
title: 'Inicio rápido: Creación de una aplicación web HTML estática'
description: Implementación de su primera aplicación Hola mundo de HTML en Azure App Service en cuestión de minutos. Realice la implementación mediante Git, que es una de las distintas maneras de realizar implementaciones en App Service.
author: msangapu-msft
ms.assetid: 60495cc5-6963-4bf0-8174-52786d226c26
ms.topic: quickstart
ms.date: 08/23/2019
ms.author: msangapu
ms.custom: mvc, cli-validate, seodec18
ROBOTS: NOINDEX,NOFOLLOW
ms.openlocfilehash: be555039375e2d71d00bcf8ee58898f54cc3645a
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379577"
---
# <a name="create-a-static-html-web-app-in-azure"></a>Creación de una aplicación web HTML estática en Azure

Esta guía de inicio rápido muestra cómo implementar un sitio HTML+CSS básico en <abbr title="Un servicio basado en HTTP para hospedar aplicaciones web, API REST y aplicaciones de back-end para dispositivos móviles.">Azure App Service</abbr>. Completará esta guía de inicio rápido en [Cloud Shell](../cloud-shell/overview.md), pero también puede ejecutar estos comandos localmente con la [CLI de Azure](/cli/azure/install-azure-cli).

## <a name="1-prepare-your-environment"></a>1. Preparación del entorno

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

En [Cloud Shell](../cloud-shell/overview.md), cree un directorio de inicio rápido y luego cámbiese a él.

```bash
mkdir quickstart

cd $HOME/quickstart
```

A continuación, ejecute el comando siguiente para clonar el repositorio de la aplicación de ejemplo en el directorio de inicio rápido.

```bash
git clone https://github.com/Azure-Samples/html-docs-hello-world.git
```
<hr/>

## <a name="2-create-a-web-app"></a>2. Creación de una aplicación web

Cambie al directorio que contiene el código de ejemplo y ejecute el comando `az webapp up`. **Reemplace** `<app-name>` por un nombre único global.

```bash
cd html-docs-hello-world

az webapp up --location westeurope --name <app_name> --html
```

<details>
<summary>Solución de problemas</summary>
<ul>
<li>Si no se reconoce el comando <code>az</code>, asegúrese de tener instalada la CLI de Azure, tal y como se describe en <a href="#1-prepare-your-environment">Preparación del entorno inicial</a>.</li>
<li>Reemplace <code>&lt;app-name&gt;</code> por un nombre que sea único en todo Azure (<em>los caracteres válidos son <code>a-z</code>, <code>0-9</code> y <code>-</code></em>). Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.</li>
<li>El argumento <code>--sku F1</code> crea la aplicación web en el plan de tarifa Gratuito. Omita este argumento para usar un nivel Premium más rápido, lo que supondrá un costo por hora.</li>
<li>El <code>--html</code> argumento señala que todo el contenido de la carpeta debe tratarse como contenido estático y que hay que deshabilitar la automatización de compilaciones.</li>
<li>Opcionalmente, incluya el argumento <code>--location &lt;location-name&gt;</code>, donde <code>&lt;location-name&gt;</code> es una región de Azure disponible. Puede recuperar una lista de las regiones permitidas para su cuenta de Azure mediante la ejecución del comando <a href="/cli/azure/appservice#az-appservice-list-locations"><code>az account list-locations</code></a>.</li>
</ul>
</details>

El comando puede tardar varios minutos en completarse. 

<details>
<summary>¿Qué hace <code>az webapp up</code>?</summary>
<p>El comando <code>az webapp up</code> realiza las acciones siguientes:</p>
<ul>
<li>Crear un grupo de recursos predeterminado.</li>
<li>Crear un plan de App Service predeterminado.</li>
<li><a href="/cli/azure/webapp#az-webapp-create">Cree una aplicación de App Service</a> con el nombre especificado.</li>
<li><a href="/azure/app-service/deploy-zip">Implementar con ZIP</a> archivos desde el directorio de trabajo actual a la aplicación.</li>
<li>Mientras se ejecuta, proporciona mensajes sobre la creación de recursos, el registro y la implementación con zip.</li>
</ul>

Al terminar, muestra información similar a la del ejemplo siguiente:

```output
{
  "app_url": "https://&lt;app_name&gt;.azurewebsites.net",
  "location": "westeurope",
  "name": "&lt;app_name&gt;",
  "os": "Windows",
  "resourcegroup": "appsvc_rg_Windows_westeurope",
  "serverfarm": "appsvc_asp_Windows_westeurope",
  "sku": "FREE",
  "src_path": "/home/&lt;username&gt;/quickstart/html-docs-hello-world ",
  &lt; JSON data removed for brevity. &gt;
}
```

</details>

Más adelante, necesitará que el valor `resourceGroup` para [limpiar los recursos](#6-clean-up-resources).

<hr/>

## <a name="3-browse-to-the-app"></a>3. Navegación hasta la aplicación

En un explorador, vaya a la dirección URL de la aplicación: `http://<app_name>.azurewebsites.net`.

La página se ejecuta como una aplicación web de Azure App Service.

![Página de inicio de la aplicación de ejemplo](media/quickstart-html/hello-world-in-browser-az.png)

<hr/>

## <a name="4-update-and-redeploy-the-app"></a>4. Actualización de la aplicación y nueva implementación

En Cloud Shell, **escriba** `nano index.html` para abrir el editor de texto nano. 

En la etiqueta del encabezado `<h1>`, cambie "Azure App Service - Sample Static HTML Site" por "Azure App Service".

![Nano index.html](media/quickstart-html/nano-index-html.png)

**Guarde** los cambios mediante el comando `^O`.

**Salga** del editor nano mediante el comando `^X`.

Vuelva a implementar la aplicación mediante el comando `az webapp up`.

```bash
az webapp up --html
```

Vuelva a la ventana del explorador que se abrió en el paso **Navegación hasta la aplicación**.

**Actualice** la página.

![Página de inicio de la aplicación de ejemplo actualizada](media/quickstart-html/hello-azure-in-browser-az.png)

<hr/>

## <a name="5-manage-your-new-azure-app"></a>5. Administración de la nueva aplicación de Azure

**Vaya** a [Azure Portal](https://portal.azure.com). 

**Busque** y **seleccione** **App Services**.

![Selección de App Services en Azure Portal](./media/quickstart-html/portal0.png)

**Seleccione** el nombre de la aplicación de Azure.

![Navegación en el portal a la aplicación de Azure](./media/quickstart-html/portal1.png)

Podrá ver la página de información general de la aplicación web. En este caso, puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar.

![Hoja de App Service en Azure Portal](./media/quickstart-html/portal2.png)

El menú izquierdo proporciona distintas páginas para configurar la aplicación.

<hr/>

## <a name="6-clean-up-resources"></a>6. Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. Si prevé que no necesitará estos recursos en el futuro, elimine el grupo de recursos ejecutando el siguiente comando en Cloud Shell. Recuerde que el nombre del grupo de recursos se generó automáticamente para el usuario en el paso [Creación de una aplicación web](#2-create-a-web-app).

```bash
az group delete --name appsvc_rg_Windows_westeurope
```

Este comando puede tardar varios segundos en ejecutarse.

<hr/>

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Asignación de un dominio personalizado](app-service-web-tutorial-custom-domain-uiex.md)
