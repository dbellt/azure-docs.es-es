---
title: Configuración de aplicaciones de Python para Linux
description: Aprenda a configurar el contenedor de Python en el que se ejecutan las aplicaciones web, mediante Azure Portal y la CLI de Azure.
ms.topic: quickstart
ms.date: 03/16/2021
ms.reviewer: astay; kraigb
ms.custom: mvc, seodec18, devx-track-python, devx-track-azurecli
ms.openlocfilehash: 605d1e0f67ac959d2c7325e04e2fd10d9d2419be
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107829501"
---
# <a name="configure-a-linux-python-app-for-azure-app-service"></a>Configuración de una aplicación de Python en Linux para Azure App Service

En este artículo se describe la forma en que [Azure App Service](overview.md) ejecuta aplicaciones de Python, cómo migrar aplicaciones existentes a Azure y cómo personalizar el comportamiento de App Service cuando es necesario. Las aplicaciones de Python deben implementarse con todos los módulos [pip](https://pypi.org/project/pip/) requeridos.

El motor de implementación de App Service activa automáticamente un entorno virtual y ejecuta `pip install -r requirements.txt` automáticamente cuando se implementa un [repositorio de Git](deploy-local-git.md) o un [paquete ZIP](deploy-zip.md).

En esta guía se incluyen conceptos clave e instrucciones para los desarrolladores de Python que usan un contenedor Linux integrado en App Service. Si nunca ha usado Azure App Service, siga primero el [inicio rápido de Python](quickstart-python.md) y el [tutorial de Python con PostgreSQL](tutorial-python-postgresql-app.md).

Puede usar [Azure Portal](https://portal.azure.com) o la CLI de Azure para la configuración:

- **Azure Portal**, use la página **Configuración** > **Configuración** de la aplicación, como se describe en [Configurar una aplicación de App Service en Azure Portal](configure-common.md).

- **CLI de Azure**: tiene dos opciones.

    - Ejecute los comandos en [Azure Cloud Shell](../cloud-shell/overview.md).
    - Ejecute los comandos localmente instalando la versión más reciente de la [CLI de Azure](/cli/azure/install-azure-cli) y, luego, inicie sesión en Azure con [az login](/cli/azure/reference-index#az_login).
    
> [!NOTE]
> Linux es actualmente la opción recomendada para ejecutar aplicaciones de Python en App Service. Para obtener información acerca de la opción de Windows, consulte [Configuración de un entorno de Python en Azure App Service (Windows)](/visualstudio/python/managing-python-on-azure-app-service).

## <a name="configure-python-version"></a>Configuración de la versión de Python

- **Azure Portal**: use la pestaña **Configuración general** de la página **Configuración**, como se describe en [Configurar las opciones generales](configure-common.md#configure-general-settings) para contenedores Linux.

- **CLI de Azure**:

    -  Muestre la versión actual de Python con [az webapp config show](/cli/azure/webapp/config#az_webapp_config_show):
    
        ```azurecli
        az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
        ```
        
        Reemplace `<resource-group-name>` y `<app-name>` por los nombres adecuados para su aplicación web.
    
    - Establezca la versión de Python con [az webapp config set](/cli/azure/webapp/config#az_webapp_config_set).
        
        ```azurecli
        az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "PYTHON|3.7"
        ```
    
    - Muestre todas las versiones de Python que se admiten en Azure App Service con [az webapp list-runtimes](/cli/azure/webapp#az_webapp_list_runtimes):
    
        ```azurecli
        az webapp list-runtimes --linux | grep PYTHON
        ```
    
Para ejecutar una versión no compatible de Python, cree una imagen de su propio imagen de contenedor. Para más información, consulte [Uso de una imagen personalizada de Docker](tutorial-custom-container.md?pivots=container-linux).

<!-- <a> element here to preserve external links-->
<a name="access-environment-variables"></a>

## <a name="customize-build-automation"></a>Personalización de la automatización de compilaciones

El sistema de compilación de App Service, denominado Oryx, realiza los pasos siguientes al implementar la aplicación mediante paquetes Git o ZIP:

1. Ejecute un script anterior a la compilación personalizado si lo especifica el valor `PRE_BUILD_COMMAND` . (El script puede a su vez ejecutar otros scripts de Python y Node.js, comandos pip y npm, y herramientas basadas en nodos como Yarn; por ejemplo, `yarn install` y `yarn build`).

1. Ejecute `pip install -r requirements.txt`. El archivo *requirements.txt* debe estar dentro de la carpeta raíz del proyecto. De lo contrario, el proceso de compilación notifica el error: "Could not find setup.py or requirements.txt; Not running pip install" (No se pudo encontrar setup.py o requirements.txt; no se ejecutará la instalación de PIP).

1. Si *manage.py* se encuentra en la raíz del repositorio (lo que indica una aplicación de Django), ejecute *manage.py collectstatic*. Sin embargo, si el valor `DISABLE_COLLECTSTATIC` es `true`, se omitirá este paso.

1. Ejecute el script posterior a la compilación personalizado si lo especifica el valor `POST_BUILD_COMMAND`. (De nuevo, el script puede ejecutar otros scripts de Python y Node.js, comandos pip y npm, y herramientas basadas en nodos).

De forma predeterminada, los valores `PRE_BUILD_COMMAND`, `POST_BUILD_COMMAND` y `DISABLE_COLLECTSTATIC` están vacíos. 

- Para deshabilitar la ejecución de collectstatic al compilar aplicaciones de Django, establezca `DISABLE_COLLECTSTATIC` en true.

- Para ejecutar comandos anteriores a la compilación, establezca el valor `PRE_BUILD_COMMAND` para que contenga un comando, como `echo Pre-build command`, o una ruta de acceso a un archivo de script relativo a la raíz del proyecto, como `scripts/prebuild.sh`. Todos los comandos deben usar rutas de acceso relativas a la carpeta raíz del proyecto.

- Para ejecutar comandos posteriores a la compilación, establezca el valor `POST_BUILD_COMMAND` para que contenga un comando, como `echo Post-build command`, o una ruta de acceso a un archivo de script relativo a la raíz del proyecto, como `scripts/postbuild.sh`. Todos los comandos deben usar rutas de acceso relativas a la carpeta raíz del proyecto.

Puede encontrar otros valores que personalicen la automatización de la compilación en [Configuración de Oryx](https://github.com/microsoft/Oryx/blob/master/doc/configuration.md). 

Para acceder a los registros de compilación e implementación, consulte [Acceso a los registros de implementación](#access-deployment-logs).

Para más información sobre cómo App Service ejecuta y compila aplicaciones de Python en Linux, consulte [Cómo Oryx detecta y compila aplicaciones de Python](https://github.com/microsoft/Oryx/blob/master/doc/runtimes/python.md).

> [!NOTE]
> Los valores `PRE_BUILD_SCRIPT_PATH` y `POST_BUILD_SCRIPT_PATH` son idénticos a `PRE_BUILD_COMMAND` y `POST_BUILD_COMMAND`, que se pueden usar como valores heredados.
> 
> Un valor denominado `SCM_DO_BUILD_DURING_DEPLOYMENT`, si contiene `true` o 1, desencadena una compilación de Oryx durante la implementación. El valor es true cuando se implementa con Git, el comando `az webapp up` de la CLI de Azure y Visual Studio Code.

> [!NOTE]
> Use siempre rutas de acceso relativas en todos los scripts anteriores y posteriores a la compilación, ya que el contenedor de compilación en el que se ejecuta Oryx es diferente del contenedor en tiempo de ejecución en el que se ejecuta la aplicación. Nunca confíe en la ubicación exacta de la carpeta de proyecto de la aplicación dentro del contenedor (por ejemplo, si está ubicada en *site/wwwroot*).

## <a name="migrate-existing-applications-to-azure"></a>Migración de aplicaciones existentes a Azure

Las aplicaciones web existentes se pueden volver a implementar en Azure de la siguiente manera:

1. **Repositorio de origen**: mantenga el código fuente en un repositorio adecuado como GitHub, que permite configurar la implementación continua más adelante en este proceso.
    1. El archivo *requirements.txt* debe estar en la raíz del repositorio para que App Service instale automáticamente los paquetes necesarios.    

1. **Base de datos**: si la aplicación depende de una base de datos, aprovisione también los recursos necesarios en Azure. Consulte [Tutorial: Implementación de una aplicación web Django con PostgreSQL: creación de una base de datos](tutorial-python-postgresql-app.md#3-create-postgres-database-in-azure) para ver un ejemplo.

1. **Recursos de App Service**: cree un grupo de recursos, un plan de App Service y una aplicación web de App Service para hospedar la aplicación. Para ello, lo más sencillo es realizar una implementación inicial del código mediante el comando de la CLI de Azure `az webapp up`, tal como se muestra en [Tutorial: Implementación de una aplicación web Django con PostgreSQL: implementación del código](tutorial-python-postgresql-app.md#4-deploy-the-code-to-azure-app-service). Reemplace los nombres del grupo de recursos, el plan de App Service y la aplicación web para adaptarlos a su aplicación.

1. **Variables de entorno**: si la aplicación requiere alguna variable de entorno, cree una [configuración de aplicación de App Service](configure-common.md#configure-app-settings) equivalente. Esta configuración de App Service se muestra en el código como variables de entorno, como se describe en [Acceso a variables de entorno](#access-app-settings-as-environment-variables).
    - Por ejemplo, las conexiones de base de datos se administran a menudo mediante esta configuración, como se muestra en [Tutorial: Implementación de una aplicación web Django con PostgreSQL: configuración de variables para conectar la base de datos](tutorial-python-postgresql-app.md#42-configure-environment-variables-to-connect-the-database).
    - Consulte [Configuración de producción para aplicaciones de Django](#production-settings-for-django-apps) para ver la configuración específica de las aplicaciones de Django típicas.

1. **Inicio de la aplicación**: revise la sección [Proceso de inicio del contenedor](#container-startup-process) más adelante en este artículo para entender cómo App Service intenta ejecutar la aplicación. App Service usa el servidor web Gunicorn de forma predeterminada, que debe poder encontrar el objeto de aplicación o la carpeta *wsgi.py*. También puede [personalizar el comando de inicio](#customize-startup-command) si es necesario.

1. **Implementación continua**: configure la implementación continua, como se describe en [Implementación continua en Azure App Service](deploy-continuous-deployment.md) si usa la implementación de Azure Pipelines o Kudu, o [Implementación de App Service con Acciones de GitHub](./deploy-continuous-deployment.md) si usa acciones de GitHub.

1. **Acciones personalizadas**: para realizar acciones en el contenedor de App Service que hospeda la aplicación, como migraciones de base de datos de Django, puede [conectarse al contenedor mediante SSH](configure-linux-open-ssh-session.md). Para obtener un ejemplo de cómo ejecutar migraciones de base de datos de Django, consulte [Tutorial: Implementación de una aplicación web Django con PostgreSQL: ejecución de migraciones de base de datos](tutorial-python-postgresql-app.md#43-run-django-database-migrations).
    - Al usar la implementación continua, puede realizar esas acciones con comandos posteriores a la compilación, como se ha descrito anteriormente en [Personalización de la automatización de compilaciones](#customize-build-automation).

Una vez realizados estos pasos, debe poder confirmar los cambios en el repositorio de origen y hacer que dichas actualizaciones se implementen automáticamente en App Service.

### <a name="production-settings-for-django-apps"></a>Configuración de producción para aplicaciones de Django

En un entorno de producción como Azure App Service, las aplicaciones de Django deben seguir la [lista de comprobación de implementación](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/) de Django (djangoproject.com).

En la tabla siguiente se describe la configuración de producción que es pertinente para Azure. Esta configuración se define en el archivo *setting.py* de la aplicación.

| Valor de Django | Instrucciones para Azure |
| --- | --- |
| `SECRET_KEY` | Almacene el valor en un valor de configuración de App Service, como se describe en [Acceso a la configuración de la aplicación como variables de entorno](#access-app-settings-as-environment-variables). Como alternativa, es posible [almacenar el valor como un "secreto" en Azure Key Vault](../key-vault/secrets/quick-create-python.md). |
| `DEBUG` | Cree un valor `DEBUG` en App Service con el valor 0 (false) y, luego, cargue el valor como una variable de entorno. En el entorno de desarrollo, cree una variable de entorno `DEBUG` con el valor 1 (true). |
| `ALLOWED_HOSTS` | En producción, Django requiere que incluya la dirección URL de la aplicación en la matriz `ALLOWED_HOSTS` de *settings.py*. Puede recuperar esta dirección URL en tiempo de ejecución con el código `os.environ['WEBSITE_HOSTNAME']`. App Service establece automáticamente la variable de entorno `WEBSITE_HOSTNAME` en la dirección URL de la aplicación. |
| `DATABASES` | Defina la configuración en App Service para la conexión de base de datos y cárguela como variables de entorno para rellenar el diccionario [`DATABASES`](https://docs.djangoproject.com/en/3.1/ref/settings/#std:setting-DATABASES). También puede almacenar los valores (especialmente el nombre de usuario y la contraseña) como [secretos de Azure Key Vault](../key-vault/secrets/quick-create-python.md). |

## <a name="serve-static-files-for-django-apps"></a>Servicio de archivos estáticos para aplicaciones Django

Si la aplicación web de Django incluye archivos front-end estáticos, siga primero las instrucciones de [administración de archivos estáticos](https://docs.djangoproject.com/en/3.1/howto/static-files/) en la documentación de Django.

Para App Service, haga las siguientes modificaciones:

1. Considere la posibilidad de usar variables de entorno (para el entorno de desarrollo local) y la configuración de aplicaciones (para el entorno en la nube) para establecer dinámicamente las variables `STATIC_URL` y `STATIC_ROOT` de Django. Por ejemplo:    

    ```python
    STATIC_URL = os.environ.get("DJANGO_STATIC_URL", "/static/")
    STATIC_ROOT = os.environ.get("DJANGO_STATIC_ROOT", "./static/")    
    ```

    `DJANGO_STATIC_URL` y `DJANGO_STATIC_ROOT` pueden cambiar según sea necesario para los entornos local y en la nube. Por ejemplo, si el proceso de compilación de los archivos estáticos los coloca en una carpeta denominada `django-static`, puede establecer `DJANGO_STATIC_URL` en `/django-static/` para evitar usar el valor predeterminado.

1. Si tiene un script anterior a la compilación que genera archivos estáticos en una carpeta diferente, incluya esa carpeta en la variable `STATICFILES_DIRS` de Django para que el proceso `collectstatic` de Django los encuentre. Por ejemplo, si ejecuta `yarn build` en la carpeta front-end y Yarn genera una carpeta `build/static` que contiene archivos estáticos, incluya esa carpeta tal y como se indica a continuación:

    ```python
    FRONTEND_DIR = "path-to-frontend-folder" 
    STATICFILES_DIRS = [os.path.join(FRONTEND_DIR, 'build', 'static')]    
    ```

    En este caso, `FRONTEND_DIR`, para crear una ruta de acceso a la ubicación en la que se ejecuta una herramienta de compilación como Yarn. Puede volver a usar una variable de entorno y la configuración de la aplicación según sus necesidades.

1. Agregue `whitenoise` al archivo *requirements.txt*. [Whitenoise](http://whitenoise.evans.io/en/stable/) (whitenoise.evans.io) es un paquete de Python que facilita que una aplicación Django de producción pueda servir sus propios archivos estáticos. Whitenoise concretamente sirve los archivos que se encuentran en la carpeta especificada por la variable `STATIC_ROOT` de Django.

1. En el archivo *settings.py*, agregue la línea siguiente para Whitenoise:

    ```python
    STATICFILES_STORAGE = ('whitenoise.storage.CompressedManifestStaticFilesStorage')
    ```

1. Modifique también las listas de `MIDDLEWARE` y `INSTALLED_APPS` para incluir Whitenoise:

    ```python
    MIDDLEWARE = [                                                                   
        'django.middleware.security.SecurityMiddleware',
        # Add whitenoise middleware after the security middleware                             
        'whitenoise.middleware.WhiteNoiseMiddleware',
        # Other values follow
    ]

    INSTALLED_APPS = [
        "whitenoise.runserver_nostatic",
        # Other values follow
    ]
    ```

## <a name="container-characteristics"></a>Características del contenedor

Cuando se implementan en App Service, las aplicaciones de Python se ejecutan en un contenedor de Docker de Linux que se define en el [repositorio de GitHub App Service Python](https://github.com/Azure-App-Service/python). Puede encontrar las configuraciones de imagen dentro de los directorios específicos de la versión.

Este contenedor tiene las siguientes características:

- Las aplicaciones se ejecutan mediante el [servidor HTTP de WSGI de Gunicorn](https://gunicorn.org/), mediante los argumentos adicionales `--bind=0.0.0.0 --timeout 600`.
    - Puede proporcionar valores de configuración para Gunicorn mediante un archivo *gunicorn.conf.py* en la raíz del proyecto, como se describe en [Introducción a la configuración de Gunicorn](https://docs.gunicorn.org/en/stable/configure.html#configuration-file) (docs.gunicorn.org). También puede [personalizar el comando de inicio](#customize-startup-command).

    - Para proteger la aplicación web de ataques de DDOS accidentales o deliberados, Gunicorn se ejecuta detrás de un proxy inverso de Nginx, como se describe en [Implementación de Gunicorn](https://docs.gunicorn.org/en/latest/deploy.html) (docs.gunicorn.org).

- De forma predeterminada, la imagen base del contenedor incluye solo la plataforma web Flask, pero el contenedor admite otras plataformas que son compatibles con WSGI y Python 3.6, como Django.

- Para instalar otros paquetes, como Django, cree un archivo [*requirements.txt*](https://pip.pypa.io/en/stable/user_guide/#requirements-files) en la raíz del proyecto que especifique las dependencias directas. A continuación, App Service instala automáticamente esas dependencias al implementar el proyecto.

    Para que se instalen esas dependencias, el archivo *requirements.txt* *debe* estar en la raíz del proyecto. De lo contrario, el proceso de compilación notifica el error: "Could not find setup.py or requirements.txt; Not running pip install" (No se pudo encontrar setup.py o requirements.txt; no se ejecutará la instalación de PIP). Si se produce este error, compruebe la ubicación del archivo de requisitos.

- App Service define automáticamente una variable de entorno llamada `WEBSITE_HOSTNAME` con la dirección URL de la aplicación web, por ejemplo, `msdocs-hello-world.azurewebsites.net`. También define `WEBSITE_SITE_NAME` con el nombre de la aplicación, por ejemplo, `msdocs-hello-world`. 
   
- npm y Node.js se instalan en el contenedor para que pueda ejecutar herramientas de compilación basadas en nodos, como Yarn.

## <a name="container-startup-process"></a>Proceso de inicio del contenedor

Durante el inicio, la instancia de App Service en el contenedor de Linux ejecuta los siguientes pasos:

1. Use un [comando de inicio personalizado](#customize-startup-command), si se proporciona.
2. Compruebe la existencia de una [aplicación de Django](#django-app) e inicie Gunicorn para ella si se detecta.
3. Compruebe la existencia de una [aplicación de Flask](#flask-app) e inicie Gunicorn para ella si se detecta.
4. Si no se encuentra ninguna otra aplicación, inicia una aplicación predeterminada que está integrada en el contenedor.

En las secciones siguientes se proporcionan detalles adicionales para cada opción.

### <a name="django-app"></a>Aplicación de Django

Para las aplicaciones de Django, App Service busca un archivo denominado `wsgi.py` en el código de la aplicación y, a continuación, ejecuta Gunicorn mediante el comando siguiente:

```bash
# <module> is the name of the folder that contains wsgi.py
gunicorn --bind=0.0.0.0 --timeout 600 <module>.wsgi
```

Si quiere un control más específico sobre el comando de inicio, use un [comando de inicio personalizado](#customize-startup-command), reemplace `<module>` por el nombre de la carpeta que contiene *wsgi.py* y agregue un argumento `--chdir` si ese módulo no está en la raíz del proyecto. Por ejemplo, si *wsgi.py* se encuentra en *knboard/backend/config* de la raíz del proyecto, use los argumentos `--chdir knboard/backend config.wsgi`.

Para habilitar el registro de producción, agregue los parámetros `--access-logfile` y `--error-logfile` como se muestra en los ejemplos de los [comandos de inicio personalizados](#customize-startup-command).

### <a name="flask-app"></a>Aplicación de Flask

En el caso de Flask, App Service busca un archivo denominado *application.py* o *app.py* e inicia Gunicorn como se indica a continuación:

```bash
# If application.py
gunicorn --bind=0.0.0.0 --timeout 600 application:app

# If app.py
gunicorn --bind=0.0.0.0 --timeout 600 app:app
```

Si el módulo de la aplicación principal está contenido en un archivo diferente, use un nombre diferente para el objeto de la aplicación, o si desea proporcionar argumentos adicionales para Gunicorn, use un [comando de inicio personalizado](#customize-startup-command).

### <a name="default-behavior"></a>Comportamiento predeterminado

Si App Service no encuentra un comando personalizado, una aplicación de Django o una aplicación de Flask, se ejecutará una aplicación predeterminada de solo lectura, ubicada en la carpeta _opt/defaultsite_ como se muestra en la siguiente imagen.

Si ha implementado el código y sigue viendo la aplicación predeterminada, consulte [Solución de problemas: la aplicación no aparece](#app-doesnt-appear).

[![Instancia predeterminada de App Service en la página web de Linux](media/configure-language-python/default-python-app.png)](#app-doesnt-appear)

De nuevo, si espera ver una aplicación implementada en lugar de la aplicación predeterminada, consulte [Solución de problemas: la aplicación no aparece](#app-doesnt-appear).

## <a name="customize-startup-command"></a>Personalización del comando de inicio

Como se indicó anteriormente en este artículo, puede proporcionar valores de configuración para Gunicorn mediante un archivo *gunicorn.conf.py* en la raíz del proyecto, como se describe en [Introducción a la configuración de Gunicorn](https://docs.gunicorn.org/en/stable/configure.html#configuration-file).

Si dicha configuración no es suficiente, puede controlar el comportamiento de inicio del contenedor proporcionando un comando de inicio personalizado o varios comandos en un archivo de comandos de inicio. Un archivo de comandos de inicio puede usar cualquier nombre que elija, por ejemplo, *startup.sh*, *startup.cmd*, *startup. txt*, etc.

Todos los comandos deben usar rutas de acceso relativas a la carpeta raíz del proyecto.

Para especificar un comando de inicio o un archivo de comandos:

- **Azure Portal**: seleccione la página **Configuración** de la aplicación y, luego, elija **Configuración general**. En el campo **Comando de inicio**, coloque el texto completo del comando de inicio o el nombre del archivo de comandos de inicio. Luego, seleccione **Guardar** para aplicar los cambios. En el caso de contenedores Linux, consulte [Configuración general](configure-common.md#configure-general-settings).

- **CLI de Azure**: use el comando [az webapp config set](/cli/azure/webapp/config#az_webapp_config_set) con el parámetro `--startup-file` para establecer el comando de inicio o el archivo de comandos de inicio:

    ```azurecli
    az webapp config set --resource-group <resource-group-name> --name <app-name> --startup-file "<custom-command>"
    ```
        
    Reemplace `<custom-command>` por el texto completo del comando de inicio o por el nombre del archivo de comandos de inicio.
        
App Service omite los errores que se producen al procesar un comando de inicio personalizado o un archivo de comandos de inicio personalizados y continúa su proceso de inicio buscando aplicaciones de Django y Flask. Si no ve el comportamiento que espera, compruebe que el comando de inicio o el archivo de comandos de inicio no tengan errores y que el archivo de comandos de inicio esté implementado en App Service junto con el código de la aplicación. También puede consultar los [registros de diagnóstico](#access-diagnostic-logs) para más información. Consulte también la página **Diagnosticar y solucionar de problemas** en [Azure Portal](https://portal.azure.com).

### <a name="example-startup-commands"></a>Comandos de inicio de ejemplo

- **Argumentos de Gunicorn agregados**: en el ejemplo siguiente se agrega `--workers=4` a una línea de comandos de Gunicorn para iniciar una aplicación de Django: 

    ```bash
    # <module-path> is the relative path to the folder that contains the module
    # that contains wsgi.py; <module> is the name of the folder containing wsgi.py.
    gunicorn --bind=0.0.0.0 --timeout 600 --workers=4 --chdir <module_path> <module>.wsgi
    ```    

    Para más información, consulte [Running Gunicorn](https://docs.gunicorn.org/en/stable/run.html) (Ejecución de Gunicorn) (docs.gunicorn.org).

- **Habilitar el registro de producción para Django**: agregue los argumentos `--access-logfile '-'` y `--error-logfile '-'` a la línea de comandos:

    ```bash    
    # '-' for the log files means stdout for --access-logfile and stderr for --error-logfile.
    gunicorn --bind=0.0.0.0 --timeout 600 --workers=4 --chdir <module_path> <module>.wsgi --access-logfile '-' --error-logfile '-'
    ```    

    Estos registros aparecerán en la [secuencia de registro de App Service](#access-diagnostic-logs).

    Para más información, consulte [Registro de Gunicorn](https://docs.gunicorn.org/en/stable/settings.html#logging) (docs.gunicorn.org).
    
- **Personalizar el módulo principal de Flask**: de forma predeterminada, App Service supone que el módulo principal de una aplicación de Flask es *application.py* o *app.py*. Si el módulo principal utiliza un nombre diferente, debe personalizar el comando de inicio. Por ejemplo, si tiene una aplicación de Flask cuyo módulo principal es *hello.py* y el objeto de aplicación de Flask en ese archivo se denomina `myapp`, el comando es de esta manera:

    ```bash
    gunicorn --bind=0.0.0.0 --timeout 600 hello:myapp
    ```
    
    Si el módulo principal está en una subcarpeta como, por ejemplo, `website` especifique esa carpeta con el argumento `--chdir`:
    
    ```bash
    gunicorn --bind=0.0.0.0 --timeout 600 --chdir website hello:myapp
    ```
    
- **Usar un servidor que no sea de Gunicorn**: para usar un servidor web diferente, como [aiohttp](https://aiohttp.readthedocs.io/en/stable/web_quickstart.html), utilice el comando adecuado como comando de inicio o en el archivo de comandos de inicio:

    ```bash
    python3.7 -m aiohttp.web -H localhost -P 8080 package.module:init_func
    ```

## <a name="access-app-settings-as-environment-variables"></a>Acceso a la configuración de la aplicación como variables de entorno

La configuración de la aplicación son valores almacenados en la nube específicamente para la aplicación, como se describe en [Configuración de valores de la aplicación](configure-common.md#configure-app-settings). Esta configuración está disponible para el código de la aplicación como variables de entorno y se puede acceder a ella mediante el patrón estándar [os.environ](https://docs.python.org/3/library/os.html#os.environ).

Por ejemplo, si ha creado una configuración de aplicación llamada `DATABASE_SERVER`, el código siguiente recupera el valor de esta configuración:

```python
db_server = os.environ['DATABASE_SERVER']
```

## <a name="detect-https-session"></a>Detección de sesión de HTTPS

En App Service, la [terminación de SSL](https://wikipedia.org/wiki/TLS_termination_proxy) (wikipedia.org) se produce en los equilibradores de carga de red, por lo que todas las solicitudes HTTPS llegan a su aplicación en forma de solicitudes HTTP sin cifrar. Si su aplicación lógica necesita comprobar si las solicitudes de usuario están cifradas, inspeccione el encabezado `X-Forwarded-Proto`.

```python
if 'X-Forwarded-Proto' in request.headers and request.headers['X-Forwarded-Proto'] == 'https':
# Do something when HTTPS is used
```

Los marcos web más usados le permiten acceder a la información de `X-Forwarded-*` en el patrón de aplicación estándar. En [CodeIgniter](https://codeigniter.com/), [is_https()](https://github.com/bcit-ci/CodeIgniter/blob/master/system/core/Common.php#L338-L365) comprueba el valor de `X_FORWARDED_PROTO` de forma predeterminada.

## <a name="access-diagnostic-logs"></a>Acceso a los registros de diagnóstico

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

Para acceder a los registros mediante Azure Portal, seleccione **Supervisión** > **Secuencia de registro** en el menú izquierdo de la aplicación.

## <a name="access-deployment-logs"></a>Acceso a los registros de implementación

Al implementar el código, App Service realiza el proceso de compilación descrito anteriormente en la sección [Personalización de la automatización de compilaciones](#customize-build-automation). Dado que la compilación se ejecuta en su propio contenedor, los registros de compilación se almacenan de forma independiente de los registros de diagnóstico de la aplicación.

Siga estos pasos para acceder a los registros de implementación:

1. En Azure Portal, en la aplicación web, seleccione **Implementación** > **Centro de implementación (versión preliminar)** en el menú de la izquierda.
1. En la pestaña **Registros**, seleccione el **Id. de confirmación** de la confirmación más reciente.
1. En la página **Detalles del registro** que aparece, seleccione el vínculo **Mostrar registros...** que aparece junto a "Running Oryx Build..." (Ejecución de compilación de Oryx).

Los problemas de compilación, como las dependencias incorrectas del archivo *requirements.txt* y los errores en los scripts anteriores o posteriores a la compilación aparecerán en estos registros. También aparecen errores si el archivo de requisitos no se llama exactamente *requirements.txt* o no aparece en la carpeta raíz del proyecto.

## <a name="open-ssh-session-in-browser"></a>Abrir sesión SSH en el explorador

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

Cuando esté conectado correctamente a la sesión de SSH, debería ver el mensaje "Conexión SSH establecida" en la parte inferior de la ventana. Si ve errores como "SSH_CONNECTION_CLOSED" o un mensaje que indica que el contenedor se está reiniciando, es posible que se impida el inicio del contenedor de la aplicación. Consulte [Solución de problemas](#troubleshooting) para conocer los pasos para investigar los posibles problemas.

## <a name="troubleshooting"></a>Solución de problemas

En general, el primer paso en la solución de problemas es usar los diagnósticos de App Service:

1. En Azure Portal, en la aplicación web, seleccione **Diagnosticar y solucionar problemas** en el menú de la izquierda.
1. Seleccione **Rendimiento y disponibilidad**.
1. Examine la información de las opciones **Registros de aplicación**, **Bloqueo de contenedor** y **Problemas de contenedor**, donde se mostrarán los problemas más comunes.

A continuación, examine los [registros de implementación](#access-deployment-logs) y los [registros de aplicación](#access-diagnostic-logs) en busca de mensajes de error. Estos registros a menudo identifican problemas específicos que pueden impedir la implementación de la aplicación o el inicio de la aplicación. Por ejemplo, se puede producir un error en la compilación si el archivo *requirements.txt* tiene un nombre de archivo incorrecto o no está presente en la carpeta raíz del proyecto.

En las secciones siguientes se proporcionan instrucciones adicionales para problemas específicos.

- [La aplicación no aparece: se muestra la aplicación predeterminada](#app-doesnt-appear)
- [La aplicación no aparece: mensaje "servicio no disponible"](#service-unavailable)
- [No se pudo encontrar setup.py o requirements.txt](#could-not-find-setuppy-or-requirementstxt)
- [ModuleNotFoundError en el inicio](#modulenotfounderror-when-app-starts)
- [La base de datos está bloqueada](#database-is-locked)
- [Las contraseñas no aparecen en la sesión SSH cuando se escriben](#other-issues)
- [Parece que los comandos de la sesión SSH se han cortado](#other-issues)
- [Los recursos estáticos no aparecen en una aplicación de Django](#other-issues)
- [Error irrecuperable: se requiere una conexión SSL](#other-issues)

#### <a name="app-doesnt-appear"></a>La aplicación no aparece

- **Ve la aplicación predeterminada después de implementar su propio código de aplicación.** La [aplicación predeterminada](#default-behavior) aparece porque no ha implementado el código de la aplicación en App Service o porque App Service no ha encontrado el código de la aplicación y ha ejecutado la aplicación predeterminada en su lugar.

    - Reinicie App Service, espere 15-20 segundos y vuelva a comprobar la aplicación.
    
    - Asegúrese de que está utilizando App Service para Linux en lugar de una instancia basada en Windows. Desde la CLI de Azure, ejecute el comando `az webapp show --resource-group <resource-group-name> --name <app-name> --query kind`, reemplazando `<resource-group-name>` y `<app-name>` en consecuencia. Debería ver `app,linux` como salida; en caso contrario, vuelva a crear la instancia de App Service y elija Linux.
    
    - Use [SSH](#open-ssh-session-in-browser) para conectarse directamente al contenedor de App Service y compruebe que los archivos existen en *site/wwwroot*. Si los archivos no existen, siga estos pasos:
      1. Cree una configuración de aplicación llamada `SCM_DO_BUILD_DURING_DEPLOYMENT` con el valor 1, vuelva a implementar el código, espere unos minutos y, a continuación, intente acceder a la aplicación de nuevo. Para más información sobre cómo crear la configuración de la aplicación, consulte [Configurar una aplicación de App Service en Azure Portal](configure-common.md).
      1. Revise el proceso de implementación, [compruebe los registros de implementación](#access-deployment-logs), corrija los errores y vuelva a implementar la aplicación.
    
    - Si existen los archivos, App Service no fue capaz de identificar el archivo de inicio específico. Compruebe que la aplicación está estructurada como espera App Service para [Django](#django-app) o [Flask](#flask-app), o use un [comando de inicio personalizado](#customize-startup-command).

- <a name="service-unavailable"></a>**Aparece el mensaje "Servicio no disponible" en el explorador.** El explorador ha superado el tiempo de espera mientras esperaba una respuesta de App Service, lo cual indica que App Service ha iniciado el servidor Gunicorn, pero la propia aplicación no se ha iniciado. Esta condición podría indicar que los argumentos de Gunicorn son incorrectos o que hay un error en el código de la aplicación.

    - Actualice el explorador, especialmente si usa los planes de tarifa más bajos en su plan de App Service. Por ejemplo, la aplicación puede tardar más en iniciarse si usa niveles gratis, por ejemplo, y comienza a tener capacidad de respuesta después de actualizar el explorador.

    - Compruebe que la aplicación está estructurada como espera App Service para [Django](#django-app) o [Flask](#flask-app), o use un [comando de inicio personalizado](#customize-startup-command).

    - Examine la [secuencia de registro de la aplicación](#access-diagnostic-logs) para ver si hay mensajes de error. Los registros mostrarán los errores en el código de la aplicación.

#### <a name="could-not-find-setuppy-or-requirementstxt"></a>No se pudo encontrar setup.py o requirements.txt

- **La secuencia de registro muestra "Could not find setup.py or requirements.txt; Not running pip install" (No se pudo encontrar setup.py o requirements.txt; no se ejecutará la instalación de PIP).** : el proceso de compilación de Oryx no ha podido encontrar el archivo *requirements.txt*.

    - Conéctese al contenedor de la aplicación web mediante [SSH](#open-ssh-session-in-browser) y compruebe que el archivo *requirements.txt* tiene el nombre correcto y existe directamente en *site/wwwroot*. Si no existe, haga que el archivo exista en el repositorio y que esté incluido en la implementación. Si existe en una carpeta distinta, muévalo a la raíz.

#### <a name="modulenotfounderror-when-app-starts"></a>ModuleNotFoundError cuando se inicia la aplicación

Si ve un error como `ModuleNotFoundError: No module named 'example'`, significa que Python no ha podido encontrar uno o varios de los módulos al iniciarse la aplicación. Esto suele ocurrir si implementa el entorno virtual con el código. Los entornos virtuales no son portables, por lo que no se deben implementar con el código de la aplicación. En su lugar, permita que Oryx cree un entorno virtual e instale los paquetes en la aplicación web mediante la creación de una configuración de aplicación, `SCM_DO_BUILD_DURING_DEPLOYMENT`, que se establezca en `1`. Esto obligará a Oryx a instalar los paquetes cada vez que realice la implementación en App Service. Para obtener más información, vea [este artículo sobre portabilidad del entorno virtual](https://azure.github.io/AppService/2020/12/11/cicd-for-python-apps.html).

### <a name="database-is-locked"></a>La base de datos está bloqueada

Al intentar ejecutar migraciones de base de datos con una aplicación de Django, puede ver "sqlite3. OperationalError: database is locked." (sqlite3. OperationalError: la base de datos está bloqueada) El error indica que la aplicación usa una base de datos de SQLite para la que Django está configurado de forma predeterminada, en lugar de usar una base de datos en la nube como PostgreSQL para Azure.

Compruebe la variable `DATABASES` en el archivo *settings.py* de la aplicación para asegurarse de que la aplicación usa una base de datos en la nube, en lugar de SQLite.

Si encuentra este error con el ejemplo de [Tutorial: Implementación de una aplicación web de Django con PostgreSQL](tutorial-python-postgresql-app.md), compruebe que ha completado los pasos descritos en la sección [Configuración de variables de entorno para conectar la base de datos](tutorial-python-postgresql-app.md#42-configure-environment-variables-to-connect-the-database).

#### <a name="other-issues"></a>Otros problemas

- **Las contraseñas no aparecen en la sesión SSH cuando se escriben**: por motivos de seguridad, la sesión SSH mantiene la contraseña oculta mientras escribe. Sin embargo, los caracteres se graban; escriba la contraseña como de costumbre y pulse **Entrar** cuando haya terminado.

- **Parece que los comandos de la sesión SSH se han cortado**: es posible que el editor no ajuste el texto de los comandos, pero deberían ejecutarse correctamente.

- **Los recursos estáticos no aparecen en una aplicación de Django**: asegúrese de que ha habilitado el módulo [whitenoise](http://whitenoise.evans.io/en/stable/django.html).

- **Aparece el mensaje "Error irrecuperable: se requiere una conexión SSL"** : compruebe los nombres de usuario y las contraseñas usados para acceder a los recursos (como las bases de datos) desde dentro de la aplicación.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Tutorial: Aplicación de Python con PostgreSQL](tutorial-python-postgresql-app.md)

> [!div class="nextstepaction"]
> [Tutorial: Implementar desde el repositorio de contenedor privado](tutorial-custom-container.md?pivots=container-linux)

> [!div class="nextstepaction"]
> [P+F sobre App Service en Linux](faq-app-service-linux.md)
