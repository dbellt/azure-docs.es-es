---
title: 'Tutorial: Implementación de una aplicación de Django de Python con Postgres'
description: Cree una aplicación web de Python con una base de datos de PostgreSQL e impleméntela en Azure. En el tutorial se usa el marco de Django, y la aplicación se hospeda en Azure App Service en Linux.
ms.devlang: python
ms.topic: tutorial
ms.date: 02/02/2021
ms.custom:
- mvc
- seodec18
- seo-python-october2019
- cli-validate
- devx-track-python
- devx-track-azurecli
ms.openlocfilehash: be55a3fb07b35fccb0f71f9ca7bfd2c88a9d097c
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108017056"
---
# <a name="tutorial-deploy-a-django-web-app-with-postgresql-in-azure-app-service"></a>Tutorial: Implementación de una aplicación web de Django de Python con PostgreSQL en Azure App Service

En este tutorial se muestra cómo implementar una aplicación web de [Django](https://www.djangoproject.com/) de Python en [Azure App Service](overview.md) y cómo conectarla a una base de datos de Azure Database for Postgres. App Service proporciona un servicio de hospedaje web muy escalable y con aplicación de revisiones de un modo automático.

En este tutorial, se usa la CLI de Azure para completar las siguientes tareas:

> [!div class="checklist"]
> * Configurar el entorno inicial con Python y la CLI de Azure
> * Crear una base de datos de Azure Database for PostgreSQL
> * Implementar código en Azure App Service y conectarse a PostgreSQL
> * Actualizar el código y volver a implementarlo
> * Visualización de los registros de diagnóstico
> * Administrar la aplicación web en Azure Portal

También puede usar la [versión de Azure Portal de este tutorial](/azure/developer/python/tutorial-python-postgresql-app-portal).


## <a name="1-set-up-your-initial-environment"></a>1. Configuración del entorno inicial

1. Disponga de una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
1. Instale <a href="https://www.python.org/downloads/" target="_blank">Python 3.6 o una versión posterior</a>.
1. Instale la <a href="/cli/azure/install-azure-cli" target="_blank">CLI de Azure</a> 2.18.0 o posterior, con la que se ejecutan comandos en cualquier shell para aprovisionar y configurar los recursos de Azure.

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

Compruebe que la versión de la CLI de Azure es la 2.18.0 o posterior:

```azurecli
az --version
```

Si necesita actualizar, pruebe el comando `az upgrade` (requiere la versión 2.11 o superior), o consulte <a href="/cli/azure/install-azure-cli" target="_blank">Instalación de la CLI de Azure</a>.

Después, inicie sesión en Azure mediante la CLI:

```azurecli
az login
```

Este comando abre un explorador que recopila las credenciales. Cuando el comando finaliza, muestra una salida JSON que contiene información sobre las suscripciones.

Una vez que haya iniciado sesión, puede ejecutar comandos de Azure con la CLI de Azure para trabajar con los recursos de su suscripción.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

## <a name="2-clone-or-download-the-sample-app"></a>2. Clonación o descarga de la aplicación de ejemplo

# <a name="git-clone"></a>[Clonación de Git](#tab/clone)

Clone el repositorio de ejemplo:

```terminal
git clone https://github.com/Azure-Samples/djangoapp
```

Después, vaya a esa carpeta:

```terminal
cd djangoapp
```

# <a name="download"></a>[Descargar](#tab/download)

Visite [https://github.com/Azure-Samples/djangoapp](https://github.com/Azure-Samples/djangoapp), seleccione **Clonar** y, a continuación, seleccione **Descargar archivo ZIP**. 

Descomprima el archivo ZIP en una carpeta denominada *djangoapp*. 

A continuación, abra una ventana del terminal en la carpeta *djangoapp*.

---

El ejemplo djangoapp contiene la aplicación en Django de sondeos controlada por datos que se obtiene al seguir [Creación de la primera aplicación Django](https://docs.djangoproject.com/en/3.1/intro/tutorial01/) de la documentación de Django. La aplicación completa se proporciona aquí para su comodidad.

El ejemplo también se ha modificado para ejecutarse en un entorno de producción, como App Service:

- La configuración para producción está en el archivo *azuresite/production.py*. Los valores del desarrollo están en el archivo *azuresite/settings.py*.
- La aplicación usa la configuración de producción cuando la variable de entorno `WEBSITE_HOSTNAME` está establecida. Azure App Service establece automáticamente esta variable en la dirección URL de la aplicación web, como `msdocs-django.azurewebsites.net`.

Los valores de producción son específicos de la configuración de Django para ejecutarse en cualquier entorno de producción y no son específicos de App Service. Para obtener más información, consulte la [lista de implementación de Django](https://docs.djangoproject.com/en/3.1/howto/deployment/checklist/). Consulte también [Configuración de producción para Django en Azure](configure-language-python.md#production-settings-for-django-apps) para los detalles de algunos de los cambios.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

## <a name="3-create-postgres-database-in-azure"></a>3. Creación de una base de datos de Postgres en Azure

<!-- > [!NOTE]
> Before you create an Azure Database for PostgreSQL server, check which [compute generation](../postgresql/concepts-pricing-tiers.md#compute-generations-and-vcores) is available in your region. -->

Instale la extensión `db-up` para la CLI de Azure:

```azurecli
az extension add --name db-up
```

Si no se reconoce el comando `az`, asegúrese de que tiene instalada la CLI de Azure tal y como se describe en [Configuración del entorno inicial](#1-set-up-your-initial-environment).

A continuación, cree la base de datos de Postgres en Azure con el comando [`az postgres up`](/cli/azure/postgres#az_postgres_up):

```azurecli
az postgres up --resource-group DjangoPostgres-tutorial-rg --location westus2 --sku-name B_Gen5_1 --server-name <postgres-server-name> --database-name pollsdb --admin-user <admin-username> --admin-password <admin-password> --ssl-enforcement Enabled
```

- **Reemplace** *\<postgres-server-name>* por un nombre que sea **único en Azure** (el punto de conexión de servidor será `https://<postgres-server-name>.postgres.database.azure.com`). Un buen patrón es usar una combinación del nombre de la empresa y otro valor exclusivo.
- Para *\<admin-username>* y *\<admin-password>* , especifique las credenciales para crear un usuario administrador de este servidor Postgres. El nombre de inicio de sesión del administrador no puede ser *azure_superuser*, *azure_pg_admin*, *admin*, *administrator*, *root*, *guest* ni *public*. No puede empezar por *pg_* . La contraseña debe contener entre **8 y 128 caracteres** de tres de las categorías siguientes: letras del alfabeto inglés mayúsculas, letras del alfabeto inglés minúsculas, números (0 a 9) y caracteres no alfanuméricos (por ejemplo, !, $, #, %). La contraseña no puede contener el nombre de usuario.
- No use el carácter `$` en el nombre de usuario ni la contraseña. Más adelante creará variables de entorno con estos valores, donde el carácter `$` tiene un significado especial dentro del contenedor de Linux que se usa para ejecutar aplicaciones de Python.
- El [plan de tarifa](../postgresql/concepts-pricing-tiers.md) B_Gen5_1 (Básico, Gen5 y 1 núcleo) que se usa aquí es el menos costoso. En el caso de las bases de datos de producción, omita el argumento `--sku-name` para usar el nivel GP_Gen5_2 (De uso general, gen. 5, 2 núcleos) en su lugar.

Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

- Crea un [grupo de recursos](../azure-resource-manager/management/overview.md#terminology) denominado `DjangoPostgres-tutorial-rg`, si aún no existe.
- Cree un servidor Postgres y asigne el nombre con el argumento `--server-name`.
- Cree una cuenta de administrador con los argumentos `--admin-user` y `--admin-password`. Puede omitir estos argumentos para permitir que el comando genere automáticamente unas credenciales únicas.
- Cree la base de datos `pollsdb` y asigne el nombre con el argumento `--database-name`.
- Habilita el acceso desde la dirección IP local.
- Habilita el acceso desde servicios de Azure.
- Crea un usuario de base de datos con acceso a la base de datos `pollsdb`.

Puede realizar todos los pasos por separado con otros comandos `az postgres` y `psql`, pero `az postgres up` realiza todos los pasos juntos.

Cuando el comando se completa, genera un objeto JSON que contiene cadenas de conexión diferentes para la base de datos, junto con la dirección URL del servidor, un nombre de usuario generado (como "joyfulKoala@msdocs-djangodb-12345") y una contraseña de GUID. **Copie el nombre de usuario y la contraseña en un archivo de texto temporal**, ya que los necesitará más adelante en este tutorial.

<!-- not all locations support az postgres up -->
> [!TIP]
> `-l <location-name>`, se puede establecer en cualquiera de las [regiones de Azure](https://azure.microsoft.com/global-infrastructure/regions/). Puede obtener las regiones disponibles para su suscripción con el comando [`az account list-locations`](/cli/azure/account#az_account_list_locations). En el caso de las aplicaciones de producción, coloque la base de datos y la aplicación en la misma ubicación.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

## <a name="4-deploy-the-code-to-azure-app-service"></a>4. Implementación del código en Azure App Service

En esta sección, creará un host de aplicación en la aplicación de App Service, conectará esta aplicación a la base de datos de Postgres y, a continuación, implementará el código en ese host.

### <a name="41-create-the-app-service-app"></a>4.1 Creación de la aplicación de App Service

En el terminal, asegúrese de estar en la carpeta del repositorio *djangoapp* que contiene el código de la aplicación.

Cree una aplicación de App Service (el proceso de host) con el comando [`az webapp up`](/cli/azure/webapp#az_webapp_up):

```azurecli
az webapp up --resource-group DjangoPostgres-tutorial-rg --location westus2 --plan DjangoPostgres-tutorial-plan --sku B1 --name <app-name>
```
<!-- without --sku creates PremiumV2 plan -->

- Para el argumento `--location`, use la misma ubicación que usó para la base de datos en la sección anterior.
- **Reemplace** *\<app-name>* por un nombre que sea único en Azure (el punto de conexión del servidor es `https://<app-name>.azurewebsites.net`). Los caracteres permitidos para *\<app-name>* son `A`-`Z`, `0`-`9` y `-`. Un buen patrón es usar una combinación del nombre de la empresa y un identificador de la aplicación.

Este comando realiza las siguientes acciones, que pueden tardar algunos minutos:

<!-- - Create the resource group if it doesn't exist. `--resource-group` is optional. -->
<!-- No it doesn't. az webapp up doesn't respect --resource-group -->
- Cree el [grupo de recursos](../azure-resource-manager/management/overview.md#terminology) si todavía no existe. (En este comando se usa el mismo grupo de recursos en el que se creó la base de datos anterior).
- Cree el [plan de App Service](overview-hosting-plans.md) *DjangoPostgres-tutorial-plan* en el plan de tarifa Básico (B1), si no existe. `--plan` y `--sku` son opcionales.
- Cree la aplicación de App Service, si no existe.
- Habilite el registro predeterminado de la aplicación, si aún no está habilitado.
- Cargue el repositorio mediante la implementación del archivo ZIP con la automatización de compilación habilitada.
- Almacenar en caché los parámetros comunes, como el nombre del grupo de recursos y el plan de App Service, mediante el archivo *.azure/config*. Como consecuencia, no es necesario especificar el mismo parámetro con comandos posteriores. Por ejemplo, para volver a implementar la aplicación después de realizar cambios, puede ejecutar `az webapp up` de nuevo sin ningún parámetro. Sin embargo, los comandos que proceden de extensiones de la CLI, como `az postgres up`, no usan la memoria caché en este momento, por lo que es necesario especificar aquí el grupo de recursos y la ubicación, con el uso inicial de `az webapp up`.

Tras una implementación correcta, el comando genera la salida JSON como en el ejemplo siguiente:

![Ejemplo de la salida del comando az webapp up](./media/tutorial-python-postgresql-app/az-webapp-up-output.png)

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

### <a name="42-configure-environment-variables-to-connect-the-database"></a>4.2 Configuración de variables de entorno para conectar la base de datos

Con el código ahora implementado en App Service, el paso siguiente consiste en conectar la aplicación a la base de datos de Postgres en Azure.

El código de la aplicación espera encontrar la información de la base de datos en cuatro variables de entorno llamadas `DBHOST`, `DBNAME`, `DBUSER` y `DBPASS`.

Para establecer variables de entorno en App Service, cree "valores de aplicación" mediante el comando [az webapp config appsettings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) siguiente.

```azurecli
az webapp config appsettings set --settings DBHOST="<postgres-server-name>" DBNAME="pollsdb" DBUSER="<username>" DBPASS="<password>"
```

- Reemplace *\<postgres-server-name>* por el nombre que usó anteriormente con el comando `az postgres up`. El código de *azuresite/production.py* anexa automáticamente `.postgres.database.azure.com` para crear la dirección URL completa del servidor Postgres.
- Reemplace *\<username>* y *\<password>* por las credenciales de administrador que usó con el comando `az postgres up` anterior, o por las que generó automáticamente el comando `az postgres up`. El código de *azuresite/production.py* construye automáticamente el nombre de usuario de Postgres completo a partir de `DBUSER` y `DBHOST`, así que no incluya la parte `@server`. (Además, como se indicó anteriormente, no debe usar el carácter `$` en ninguno de los dos valores, ya que tiene un significado especial para las variables de entorno de Linux).
- El grupo de recursos y el nombre de la aplicación se extraen de los valores almacenados en caché en el archivo *.azure/config*.

En el código de Python, tiene acceso a esta configuración como variables de entorno con instrucciones como `os.environ.get('DBHOST')`. Para obtener más información, consulte [Acceso a variables de entorno](configure-language-python.md#access-environment-variables).

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

### <a name="43-run-django-database-migrations"></a>4.3 Ejecución de migraciones de la base de datos de Django

Las migraciones de bases de datos de Django aseguran que el esquema de la base de datos de PostgreSQL en Azure coincida con las descritas en el código.

1. Abra una sesión de SSH **en el explorador**. Para ello, vaya a la siguiente dirección URL e inicie sesión con las credenciales de su cuenta de Azure (no las credenciales del servidor de bases de datos).

    ```
    https://<app-name>.scm.azurewebsites.net/webssh/host
    ```

    Reemplace `<app-name>` por el nombre usado anteriormente en el comando `az webapp up`.

    Puede conectarse alternativamente a una sesión SSH mediante el comando [`az webapp ssh`](/cli/azure/webapp#az_webapp_ssh). En Windows, este comando requiere la CLI de Azure 2.18.0 o posterior.

    Si no puede conectarse a la sesión SSH, significa que no se pudo iniciar la propia aplicación. [Compruebe los registros de diagnóstico](#6-stream-diagnostic-logs) para más información. Por ejemplo, si no ha creado la configuración de la aplicación necesaria en la sección anterior, los registros indicarán `KeyError: 'DBNAME'`.

1. En la sesión de SSH, ejecute los siguientes comandos (puede pegar los comandos con **CTRL**+**Mayús**+**V**):

    ```bash
    # Change to the app folder
    cd $APP_PATH
    
    # Activate the venv
    source antenv/bin/activate

    # Install requirements
    pip install -r requirements.txt

    # Run database migrations
    python manage.py migrate

    # Create the super user (follow prompts)
    python manage.py createsuperuser
    ```

    Si encuentra algún error relacionado con la conexión a la base de datos, compruebe los valores de la configuración de la aplicación creada en la sección anterior.

1. El comando `createsuperuser` le pide las credenciales de superusuario. A los efectos de este tutorial, use el nombre de usuario predeterminado `root`, presione **ENTRAR** en la dirección de correo electrónico para que quede en blanco y escriba `Pollsdb1` como contraseña.

1. Si aparece un error que indica que la base de datos está bloqueada, asegúrese de haber ejecutado el comando `az webapp settings` en la sección anterior. Sin esa configuración, el comando Migrate no se puede comunicar con la base de datos, y esto provoca el error.

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).
    
### <a name="44-create-a-poll-question-in-the-app"></a>4.4 Creación de una pregunta de sondeo en la aplicación

1. En un explorador, abra la dirección URL `http://<app-name>.azurewebsites.net`. La aplicación debe mostrar los mensajes "Polls app" (Aplicación de sondeos) y "No polls are available" (No hay sondeos disponibles) porque todavía no hay sondeos específicos en la base de datos.

    Si ve el mensaje "Application Error" (Error de aplicación), es probable que no haya creado la configuración necesaria en el paso anterior, [Configuración de las variables de entorno para conectarse a la base de datos](#42-configure-environment-variables-to-connect-the-database), o que dichos valores contengan errores. Ejecute el comando `az webapp config appsettings list` para comprobar la configuración. También puede [comprobar los registros de diagnóstico](#6-stream-diagnostic-logs) para ver errores específicos durante el inicio de la aplicación. Por ejemplo, si no ha creado la configuración, los registros mostrarán el error `KeyError: 'DBNAME'`.

    Después de actualizar la configuración para corregir los errores, espere un minuto para que la aplicación se reinicie y, después, actualice el explorador.

1. Vaya a `http://<app-name>.azurewebsites.net/admin`. Inicie sesión con las credenciales de superusuario de Django de la sección anterior (`root` y `Pollsdb1`). En **Polls** (Sondeos), seleccione **Add** (Agregar) junto a **Questions** (Preguntas) y cree una pregunta de sondeo con algunas opciones.

1. Vaya de nuevo a `http://<app-name>.azurewebsites.net` para confirmar que las preguntas se presentan ahora al usuario. Responda a las preguntas como desee para generar algunos datos en la base de datos.

**¡Enhorabuena!** Está ejecutando una aplicación web de Django de Python en Azure App Service para Linux, con una base de datos de Postgres activa.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

> [!NOTE]
> App Service detecta un proyecto de Django mediante la búsqueda de un archivo *wsgi.py* en cada subcarpeta, que `manage.py startproject` crea de forma predeterminada. Cuando App Service encuentra ese archivo, carga la aplicación web de Django. Para obtener más información, consulte [Configuración de una imagen de Python integrada](configure-language-python.md).

## <a name="5-make-code-changes-and-redeploy"></a>5. Cambios en el código y reimplementación

En esta sección, realizará cambios locales en la aplicación y volverá a implementar el código en App Service. En el proceso, se configura un entorno virtual de Python que admite el trabajo en curso.

### <a name="51-run-the-app-locally"></a>5.1 Ejecución de la aplicación de forma local

Ejecute los siguientes comandos en una ventana de terminal. Asegúrese de seguir las indicaciones al crear el superusuario:

# <a name="bash"></a>[bash](#tab/bash)

```bash
# Configure the Python virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="powershell"></a>[PowerShell](#tab/powershell)

```powershell
# Configure the Python virtual environment
py -3 -m venv venv
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force
venv\scripts\activate

# Install dependencies
pip install -r requirements.txt
# Run Django migrations
python manage.py migrate
# Create Django superuser (follow prompts)
python manage.py createsuperuser
# Run the dev server
python manage.py runserver
```

# <a name="cmd"></a>[CMD](#tab/cmd)

```cmd
:: Configure the Python virtual environment
py -3 -m venv venv
venv\scripts\activate

:: Install dependencies
pip install -r requirements.txt
:: Run Django migrations
python manage.py migrate
:: Create Django superuser (follow prompts)
python manage.py createsuperuser
:: Run the dev server
python manage.py runserver
```
---

Una vez que la aplicación web esté completamente cargada, el servidor de desarrollo de Django proporcionará la dirección URL de la aplicación local en el mensaje "Starting development server at http://127.0.0.1:8000/. Quit the server with CTRL-BREAK" (Iniciando el servidor de desarrollo en https… Salga del servidor con Ctrl+Interrumpir).

![Ejemplo de salida del servidor de desarrollo de Django](./media/tutorial-python-postgresql-app/django-dev-server-output.png)

Pruebe la aplicación localmente con los pasos siguientes:

1. Vaya a `http://localhost:8000` en un explorador, que debería mostrar un mensaje que indica que no hay sondeos disponibles. 

1. Vaya a `http:///localhost:8000/admin` e inicie sesión con el usuario administrador que creó previamente. En **Sondeos**, vuelva a seleccionar **Agregar** junto a **Preguntas** y cree una pregunta de sondeo con opciones. 

1. Vaya a *http:\//localhost: 8000* de nuevo y responda la pregunta para probar la aplicación. 

1. Detenga el servidor de Django presionando **CTRL**+**C**.

Cuando se ejecute localmente, la aplicación usará una base de datos de Sqlite3 local y no interferirá con la base de datos de producción. También puede usar una base de datos de PostgreSQL local, si lo desea, para simular mejor el entorno de producción.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

### <a name="52-update-the-app"></a>5.2 Actualización de la aplicación

En `polls/models.py`, busque la línea que comienza con `choice_text` y cambie el parámetro `max_length` a 100:

```python
# Find this line of code and set max_length to 100 instead of 200
choice_text = models.CharField(max_length=100)
```

Dado que ha cambiado el modelo de datos, cree una nueva migración de Django y migre la base de datos:

```
python manage.py makemigrations
python manage.py migrate
```

Vuelva a ejecutar el servidor de desarrollo con `python manage.py runserver` y pruebe la aplicación en *http:\//localhost:8000/admin*:

Vuelva a detener el servidor web de Django con **CTRL**+**C**.

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

### <a name="53-redeploy-the-code-to-azure"></a>5.3 Nueva implementación del código en Azure

Ejecute el comando siguiente en la raíz del repositorio:

```azurecli
az webapp up
```

Este comando usa los parámetros almacenados en la memoria caché del archivo *.azure/config*. Dado que App Service detecta que la aplicación ya existe, simplemente vuelve a implementar el código.

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

### <a name="54-rerun-migrations-in-azure"></a>5.4 Nueva ejecución de migraciones en Azure

Dado que ha realizado cambios en el modelo de datos, debe volver a ejecutar las migraciones de base de datos en App Service.

Abra una sesión de SSH de nuevo en el explorador. Para ello, vaya a `https://<app-name>.scm.azurewebsites.net/webssh/host`. Luego, ejecute los siguientes comandos:

```
cd $APP_PATH
source antenv/bin/activate
pip install -r requirements.txt
python manage.py migrate
```

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

### <a name="55-review-app-in-production"></a>5.5 Revisión de la aplicación en producción

Vaya a `http://<app-name>.azurewebsites.net` y vuelva a probar la aplicación en el entorno de producción. (Dado que solo ha cambiado la longitud de un campo de la base de datos, el cambio solo se aprecia si intenta especificar una respuesta más larga al crear una pregunta).

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

## <a name="6-stream-diagnostic-logs"></a>6. Transmisión de registros de diagnóstico

Puede acceder a los registros de la consola generados desde dentro del contenedor que hospeda la aplicación en Azure.

Ejecute el siguiente comando de la CLI de Azure para ver la secuencia de registro. Este comando usa los parámetros almacenados en la memoria caché del archivo *.azure/config*.

```azurecli
az webapp log tail
```

Si no ve los registros de la consola de inmediato, vuelve a comprobarlo en 30 segundos.

Para detener el streaming de registro en cualquier momento, escriba **Ctrl**+**C**.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

> [!NOTE]
> También puede inspeccionar los archivos de registro desde el explorador en `https://<app-name>.scm.azurewebsites.net/api/logs/docker`.
>
> `az webapp up` activa el registro predeterminado. Por motivos de rendimiento, este registro se desactiva después de un tiempo, pero vuelve a activarse cada vez que se vuelve a ejecutar `az webapp up`. Para activarlo manualmente, ejecute el siguiente comando:
>
> ```azurecli
> az webapp log config --docker-container-logging filesystem
> ```

## <a name="7-manage-your-app-in-the-azure-portal"></a>7. Administración de una aplicación en Azure Portal

En [Azure Portal](https://portal.azure.com), busque el nombre de la aplicación y seleccione la aplicación en los resultados.

![Vaya a la aplicación Python Django en Azure Portal](./media/tutorial-python-postgresql-app/navigate-to-django-app-in-app-services-in-the-azure-portal.png)

De forma predeterminada, el portal muestra la página **Información general** de la aplicación, que proporciona una vista del rendimiento general. En ella puede también realizar tareas de administración básicas como examinar, detener, reiniciar y eliminar. Las pestañas del lado izquierdo de la página muestran las diferentes páginas de configuración que puede abrir.

![Administración de la aplicación Python Django en la página de información general de Azure Portal](./media/tutorial-python-postgresql-app/manage-django-app-in-app-services-in-the-azure-portal.png)

¿Tiene problemas? Consulte primero la [Guía de solución de problemas](configure-language-python.md#troubleshooting) y, si eso no funciona, [háganoslo saber](https://aka.ms/DjangoCLITutorialHelp).

## <a name="8-clean-up-resources"></a>8. Limpieza de recursos

Si desea mantener la aplicación o continuar con los tutoriales adicionales, vaya a [Pasos siguientes](#next-steps). De lo contrario, para evitar incurrir en cargos adicionales, puede eliminar la creación del grupo de recursos para este tutorial:

```azurecli
az group delete --no-wait
```

Este comando usa el nombre del grupo de recursos almacenado en la memoria caché del archivo *.azure/config*. Al eliminar el grupo de recursos, también se desasignan y eliminan todos los recursos que contiene.

La eliminación de todos los recursos puede llevar tiempo. El argumento `--no-wait` permite que el comando devuelva resultados inmediatamente.

¿Tiene problemas? [Póngase en contacto con nosotros](https://aka.ms/DjangoCLITutorialHelp).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre cómo asignar un nombre DNS personalizado a la aplicación:

> [!div class="nextstepaction"]
> [Tutorial: Asignar un nombre DNS personalizado a la aplicación](app-service-web-tutorial-custom-domain.md)

Obtenga información sobre cómo App Service ejecuta una aplicación de Python:

> [!div class="nextstepaction"]
> [Configuración de una aplicación de Python](configure-language-python.md)
