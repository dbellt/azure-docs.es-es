---
title: 'Inicio rápido: Creación de una aplicación de Python en Linux'
description: Para empezar a trabajar con Azure App Service, implemente una aplicación de Python en un contenedor de Linux en App Service.
ms.topic: quickstart
ms.date: 09/22/2020
ms.custom: seo-python-october2019, cli-validate, devx-track-python
zone_pivot_groups: python-frameworks-01
robots: noindex
ms.openlocfilehash: 464583f9e20d55fa351a6ae0df31b608b18dc428
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107379543"
---
# <a name="quickstart-create-a-python-app-in-azure-app-service-on-linux"></a>Inicio rápido: Creación de una aplicación de Python en Azure App Service en Linux

En este inicio rápido se implementa una aplicación web de Python en [App Service en Linux](overview.md#app-service-on-linux), un servicio de Azure de hospedaje de sitios web muy escalable y con aplicación automática de revisiones. La [interfaz de la línea de comandos (CLI) de Azure](/cli/azure/install-azure-cli) local se puede usar en un equipo Mac, Linux o Windows para implementar un ejemplo con los marcos Flask o Django. La aplicación web que va a configurar usa un nivel de App Service gratuito, por lo que no incurrirá en ningún gasto en el transcurso de este artículo.

> [!TIP]
> Si prefiere implementar aplicaciones mediante un entorno de desarrollo integrado, consulte **[Implementación de aplicaciones de Python en App Service desde Visual Studio Code](/azure/developer/python/tutorial-deploy-app-service-on-linux-01)** .

## <a name="set-up-your-initial-environment"></a>Configuración del entorno inicial

1. Disponga de una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
1. Instale <a href="https://www.python.org/downloads/" target="_blank">Python 3.6 o una versión posterior</a>.
1. Instale la <a href="/cli/azure/install-azure-cli" target="_blank">CLI de Azure</a> 2.0.80 o posterior, con la que se ejecutan comandos en cualquier shell para aprovisionar y configurar los recursos de Azure.

Abra una ventana del terminal y compruebe que la versión de Python es la 3.6 o superior:

# <a name="bash"></a>[Bash](#tab/bash)

```bash
python3 --version
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```cmd
py -3 --version
```

# <a name="cmd"></a>[Cmd](#tab/cmd)

```cmd
py -3 --version
```

---

Compruebe que la versión de la CLI de Azure es la 2.0.80 o posterior:

```azurecli
az --version
```

Después, inicie sesión en Azure mediante la CLI:

```azurecli
az login
```

Este comando abre un explorador que recopila las credenciales. Cuando el comando finaliza, muestra una salida JSON que contiene información sobre las suscripciones.

Una vez que haya iniciado sesión, puede ejecutar comandos de Azure con la CLI de Azure para trabajar con los recursos de su suscripción.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="clone-the-sample"></a>Clonación del ejemplo

Ejecute el comando siguiente para clonar el repositorio de ejemplo y vaya a la carpeta de ejemplo. ([Instale git](https://git-scm.com/downloads) si aún no lo tiene).

::: zone pivot="python-framework-flask"
```terminal
git clone https://github.com/Azure-Samples/python-docs-hello-world
```

Después, vaya a esa carpeta:

```terminal
cd python-docs-hello-world
```
::: zone-end

::: zone pivot="python-framework-django"
```terminal
git clone https://github.com/Azure-Samples/python-docs-hello-django
```

Después, vaya a esa carpeta:

```terminal
cd python-docs-hello-django
```
::: zone-end

El ejemplo contiene código específico del marco que Azure App Service reconoce al iniciar la aplicación. Para más información, consulte [Proceso de inicio del contenedor](configure-language-python.md#container-startup-process).

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="deploy-the-sample"></a>Implementación del ejemplo

Implemente el código en la carpeta local (*python-docs-hello-world*) mediante el comando `az webapp up`:

```azurecli
az webapp up --sku B1 --name <app-name>
```

- Si no se reconoce el comando `az`, asegúrese de que tiene instalada la CLI de Azure tal y como se describe en [Configuración del entorno inicial](#set-up-your-initial-environment).
- Si no se reconoce el comando `webapp`, es porque la versión de la CLI de Azure es 2.0.80 o posterior. De lo contrario, [instale la versión más reciente](/cli/azure/install-azure-cli).
- Reemplace `<app_name>` por un nombre que sea único en todo Azure (*los caracteres válidos son `a-z`, `0-9` y `-`* ). Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.
- El argumento `--sku B1` crea la aplicación web en el plan de tarifa básico, que genera un pequeño costo por hora. Omita este argumento para usar un nivel Premium más rápido.
- Opcionalmente, incluya el argumento `--location <location-name>`, donde `<location_name>` es una región de Azure disponible. Puede recuperar una lista de las regiones permitidas para su cuenta de Azure mediante la ejecución del comando [`az account list-locations`](/cli/azure/appservice#az-appservice-list-locations).
- Si ve el error "No se pudo detectar automáticamente la pila en tiempo de ejecución de la aplicación", asegúrese de que ejecuta el comando en la carpeta *python-docs-hello-world* (Flask) o en la carpeta *python-docs-hello-django* (Django) que contiene el archivo *requirements.txt*. (Consulte [Solución de problemas de detección automática con az webapp up](https://github.com/Azure/app-service-linux-docs/blob/master/AzWebAppUP/runtime_detection.md) (GitHub)).

El comando puede tardar varios minutos en completarse. Mientras se ejecuta, proporciona mensajes sobre cómo crear el grupo de recursos, el plan de App Service y la aplicación de hospedaje, configurar el registro y, a continuación, realizar la implementación del archivo ZIP. A continuación, se muestra el mensaje "Puede iniciar la aplicación en http://&lt;nombre_de_la_aplicación&gt;.azurewebsites.net", que es la dirección URL de la aplicación en Azure.

![Salida de ejemplo del comando az webapp up](./media/quickstart-python/az-webapp-up-output.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

[!include [az webapp up command note](../../includes/app-service-web-az-webapp-up-note.md)]

## <a name="browse-to-the-app"></a>Navegación hasta la aplicación

Vaya a la aplicación implementada mediante el explorador web en la dirección URL `http://<app-name>.azurewebsites.net`. Tardará unos instantes en iniciar la aplicación la primera vez.

El código de ejemplo de Python se ejecuta en un contenedor de Linux en App Service con una imagen integrada.

![Ejecución de una aplicación de Python de ejemplo en Azure](./media/quickstart-python/run-hello-world-sample-python-app-in-browser.png)

**¡Enhorabuena!** Ha implementado su primera aplicación Python en App Service en Linux.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="run-the-sample"></a>Ejecución del ejemplo 

::: zone pivot="python-framework-flask"
1. Asegúrese de que se encuentra en la carpeta *python-docs-hello-world*. 

1. Cree un entorno virtual e instale las dependencias:

    [!include [virtual environment setup](../../includes/app-service-quickstart-python-venv.md)]

    Si encuentra "[Errno 2] No se encontró el archivo o directorio: 'requirements.txt'", asegúrese de que se encuentra en la carpeta *python-docs-hello-world*.

1. Ejecute el servidor de desarrollo.

    ```terminal  
    flask run
    ```
    
    De forma predeterminada, el servidor supone que el módulo de entrada de la aplicación está en *app.py*, como se usa en el ejemplo. (Si usa un nombre de módulo diferente, establezca la variable de entorno `FLASK_APP` en ese nombre).

1. Abra un explorador web y vaya a la aplicación de ejemplo en `http://localhost:5000/`. La aplicación muestra el mensaje **Hola mundo**.

    ![Ejecución de una aplicación de Python de ejemplo en el entorno local](./media/quickstart-python/run-hello-world-sample-python-app-in-browser-localhost.png)
    
1. En la ventana de terminal, presione **Ctrl**+**C** para salir del servidor de desarrollo.
::: zone-end

::: zone pivot="python-framework-django"
1. Asegúrese de que se encuentra en la carpeta *python-docs-hello-django*. 

1. Cree un entorno virtual e instale las dependencias:

    [!include [virtual environment setup](../../includes/app-service-quickstart-python-venv.md)]

    Si encuentra "[Errno 2]: No se encontró el archivo o directorio: 'requirements.txt'", asegúrese de que se encuentra en la carpeta *python-docs-hello-django*.
    
1. Ejecute el servidor de desarrollo.

    # <a name="bash"></a>[Bash](#tab/bash)

    ```bash
    python3 manage.py runserver
    ```

    # <a name="powershell"></a>[PowerShell](#tab/powershell)

    ```powershell
    py -3 manage.py runserver
    ```

    # <a name="cmd"></a>[Cmd](#tab/cmd)

    ```cmd
    py -3 manage.py runserver
    ```

    ---    

1. Abra un explorador web y vaya a la aplicación de ejemplo en `http://localhost:8000/`. La aplicación muestra el mensaje **Hola mundo**.

    ![Ejecución de una aplicación de Python de ejemplo en el entorno local](./media/quickstart-python/run-hello-world-sample-python-app-in-browser-localhost.png)
    
1. En la ventana de terminal, presione **Ctrl**+**C** para salir del servidor de desarrollo.
::: zone-end

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="redeploy-updates"></a>Reimplementación de actualizaciones

En esta sección, realizará un pequeño cambio de código y, a continuación, volverá a implementar el código en Azure. Este cambio de código incluye una instrucción `print` para generar la salida de registro con la que trabajará en la sección siguiente.

::: zone pivot="python-framework-flask"
Abra *app.py* en un editor y actualice la función `hello` para que coincida con el código siguiente. 

```python
def hello():
    print("Handling request to home page.")
    return "Hello, Azure!"
```
::: zone-end
::: zone pivot="python-framework-django"
Abra *hello/views.py* en un editor y actualice la función `hello` para que coincida con el código siguiente.

```python
def hello(request):
    print("Handling request to home page.")
    return HttpResponse("Hello, Azure!")
```
::: zone-end
    
Guarde los cambios y vuelva a implementar la aplicación con el comando `az webapp up`:

```azurecli
az webapp up
```

Este comando utiliza valores que se almacenan en caché de forma local en el archivo *.azure/config*, incluidos el nombre de la aplicación, el grupo de recursos y el plan de App Service.

Una vez completada la implementación, vuelva a la ventana del explorador abierta en `http://<app-name>.azurewebsites.net`. Actualice la página, lo que debería mostrar el mensaje modificado:

![Ejecución de una aplicación de Python de ejemplo actualizada en Azure](./media/quickstart-python/run-updated-hello-world-sample-python-app-in-browser.png)

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

> [!TIP]
> Visual Studio Code proporciona extensiones eficaces para Python y Azure App Service que simplifican el proceso de implementación de aplicaciones web de Python en App Service. Para más información, consulte [Deploy Python apps to App Service from Visual Studio Code](/azure/python/tutorial-deploy-app-service-on-linux-01) (Implementación de aplicaciones de Python en App Service desde Visual Studio Code).

## <a name="stream-logs"></a>Transmisión de registros

Puede acceder a los registros de consola generados desde dentro de la aplicación y del contenedor en que esta se ejecuta. Los registros incluyen todas las salidas generadas mediante instrucciones `print`.

Para transmitir registros, ejecute el comando [az webapp log tail](/cli/azure/webapp/log#az_webapp_log_tail):

```azurecli
az webapp log tail
```

También puede incluir el parámetro `--logs` con el comando `az webapp up` para abrir automáticamente la secuencia de registro en la implementación.

Actualice la aplicación en el explorador para que se generen registros de la consola, que incluyen mensajes que describen las solicitudes HTTP a la aplicación. Si no aparece la salida inmediatamente, inténtelo de nuevo en 30 segundos.

También puede inspeccionar los archivos de registro desde el explorador en `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.

Para detener el streaming de registros en cualquier momento, presione **Ctrl**+**C** en el terminal.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="manage-the-azure-app"></a>Administración de la aplicación de Azure

Vaya a <a href="https://portal.azure.com" target="_blank">Azure Portal</a> para administrar la aplicación que ha creado. Busque y seleccione **App Services**.

![Ir a App Services en Azure Portal](./media/quickstart-python/navigate-to-app-services-in-the-azure-portal.png)

Seleccione el nombre de la aplicación de Azure.

![Aplicación de Python en App Services desde Azure Portal](./media/quickstart-python/navigate-to-app-in-app-services-in-the-azure-portal.png)

Al seleccionar la aplicación, se abre la página **Información general**, donde puede realizar tareas de administración básicas como examinar, detener, iniciar, reiniciar y eliminar.

![Administración de la aplicación de Python en la página de información general de Azure Portal](./media/quickstart-python/manage-an-app-in-app-services-in-the-azure-portal.png)

El menú de App Service proporciona distintas páginas para configurar la aplicación.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="clean-up-resources"></a>Limpieza de recursos

En los pasos anteriores, creó recursos de Azure en un grupo de recursos. El grupo de recursos tiene un nombre como "appsvc_rg_Linux_CentralUS", en función de su ubicación. Si mantiene la aplicación web en ejecución, se generarán costos continuos (consulte [Precios de App Service](https://azure.microsoft.com/pricing/details/app-service/linux/)).

Si prevé que no necesitará estos recursos en el futuro, ejecute el siguiente comando para eliminar el grupo de recursos:

```azurecli
az group delete --no-wait
```

Este comando usa el nombre del grupo de recursos almacenado en la memoria caché del archivo *.azure/config*.

El argumento `--no-wait` permite la devolución del comando antes de que se complete la operación.

[¿Tiene problemas? Háganoslo saber.](https://aka.ms/FlaskCLIQuickstartHelp)

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación web Python (Django) con PostgreSQL](tutorial-python-postgresql-app.md)

> [!div class="nextstepaction"]
> [Configuración de una aplicación de Python](configure-language-python.md)

> [!div class="nextstepaction"]
> [Incorporación del inicio de sesión de un usuario a una aplicación web de Python](../active-directory/develop/quickstart-v2-python-webapp.md)

> [!div class="nextstepaction"]
> [Tutorial: Ejecución de la aplicación Python en un contenedor personalizado](tutorial-custom-container.md)
