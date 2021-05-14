---
title: Contenedorización de Azure App en ASP.NET. Contenedorización y migración de aplicaciones de ASP.NET a Azure Kubernetes.
description: 'Tutorial: Inclusión en contenedores y migración de aplicaciones de ASP.NET a Azure Kubernetes Service.'
services: ''
author: rahugup
manager: bsiva
ms.topic: tutorial
ms.date: 3/2/2021
ms.author: rahugup
ms.openlocfilehash: 24fd5cb0abbf36df74d6e2300c570e7e39b875ad
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108180579"
---
# <a name="aspnet-app-containerization-and-migration-to-azure-kubernetes-service"></a>Contenedorización de aplicaciones de ASP.NET y migración a Azure Kubernetes Service

En este artículo, aprenderá a incluir en contenedores las aplicaciones ASP.NET y a migrarlas a [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/services/kubernetes-service/) mediante la herramienta Azure Migrate: App Containerization. El proceso de contenedorización no requiere acceso a su código base y le proporciona una manera sencilla de incluir en contenedores las aplicaciones existentes. La herramienta usa el estado de ejecución de las aplicaciones en un servidor para determinar los componentes de la aplicación y ayuda a empaquetarlos en una imagen de contenedor. A continuación, la aplicación contenedorizada se puede implementar en Azure Kubernetes Service (AKS).

La herramienta de contenedorización de aplicaciones de Azure Migrate admite actualmente las acciones siguientes:

- Inclusión en contenedores de las aplicaciones ASP.NET e implementación en contenedores de Windows en AKS.
- Inclusión en contenedores de aplicaciones web de Java en Apache Tomcat (en servidores Linux) e implementación en contenedores de Linux en AKS. [Más información](./tutorial-containerize-java-kubernetes.md)


La herramienta Azure Migrate: App Containerization le ayuda a hacer lo siguiente:

- **Detectar la aplicación**: la herramienta se conecta de forma remota a los servidores de aplicaciones que ejecutan la aplicación ASP.NET y detecta los componentes de la aplicación. La herramienta crea un archivo Dockerfile que se puede usar para crear una imagen de contenedor para la aplicación.
- **Compilar la imagen de contenedor**: puede inspeccionar y personalizar aún más el archivo Dockerfile según los requisitos de la aplicación y usarlo para compilar la imagen de contenedor de la aplicación. La imagen de contenedor de la aplicación se inserta en la instancia de Azure Container Registry que especifique.
- **Implementar en Azure Kubernetes Service**: la herramienta genera los archivos YAML de definición de recursos de Kubernetes necesarios para implementar la aplicación contenedorizada en el clúster de Azure Kubernetes Service. Puede personalizar los archivos YAML y usarlos para implementar la aplicación en AKS.

> [!NOTE]
> La herramienta de contenedorización de Azure Migrate le ayuda a detectar tipos de aplicación específicos (ASP.NET y aplicaciones web de Java en Apache Tomcat), así como sus componentes, en un servidor de aplicaciones. Para detectar servidores y el inventario de las aplicaciones, roles y características que se ejecutan en máquinas locales, use la funcionalidad de detección y valoración de Azure Migrate. [Más información](./tutorial-discover-vmware.md)

Aunque todas las aplicaciones no se beneficiarán de un cambio directo a los contenedores sin realizar un rediseño significativo, estas son algunas de las ventajas de migrar las aplicaciones existentes a los contenedores sin rescribirlas:

- **Uso mejorado de la infraestructura:** con los contenedores, varias aplicaciones pueden compartir recursos y hospedarse en la misma infraestructura. Esto puede ayudarlo a consolidar la infraestructura y mejorar su utilización.
- **Administración simplificada:** al hospedar las aplicaciones en una plataforma moderna de infraestructura administrada como AKS, puede simplificar las prácticas de administración a la vez que mantiene el control sobre la infraestructura. Puede conseguirlo al retirar o reducir los procesos de administración y mantenimiento de la infraestructura que tradicionalmente realizaría con una infraestructura de su propiedad.
- **Portabilidad de la aplicación:** gracias a la mayor adopción y estandarización de los formatos de especificación de contenedor y las plataformas de orquestación, la portabilidad de la aplicación ya no es un problema.
- **Adopción de la administración moderna con DevOps:** le ayuda a adoptar y normalizar las prácticas modernas de administración y seguridad con la infraestructura como código y la transición a DevOps.


En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Configurar una cuenta de Azure.
> * Instalar la herramienta Azure Migrate: App Containerization.
> * Detectar la aplicación ASP.NET.
> * Compile la imagen de contenedor.
> * Ejecutar la aplicación contenedorizada en AKS.

> [!NOTE]
> En los tutoriales se muestra la ruta de implementación más sencilla para un escenario, de modo que pueda configurar rápidamente una prueba de concepto. En ellos se usan las opciones predeterminadas siempre que es posible y no muestran todos los valores y rutas de acceso posibles.

## <a name="prerequisites"></a>Requisitos previos

Antes de comenzar este tutorial, debe:

**Requisito** | **Detalles**
--- | ---
**Identificar una máquina en la que instalar la herramienta** | Una máquina Windows para instalar y ejecutar la herramienta de contenedorización de aplicaciones de Azure Migrate. La máquina Windows puede ser un sistema operativo de servidor (Windows Server 2016 o posterior) o de cliente (Windows 10), lo que significa que la herramienta también puede ejecutarse en el escritorio. <br/><br/> La máquina Windows que ejecuta la herramienta debe tener conectividad de red con los servidores o las máquinas virtuales que hospedan las aplicaciones ASP.NET que se incluirán en contenedores.<br/><br/> Asegúrese de que haya disponible 6 GB de espacio en la máquina Windows que ejecuta la herramienta de contenedorización de aplicaciones de Azure Migrate a fin de almacenar los artefactos de la aplicación. <br/><br/> La máquina Windows debe tener acceso a Internet, ya sea directamente o a través de un proxy. <br/> <br/>Instale la herramienta Microsoft Web Deploy en el equipo que ejecuta la herramienta auxiliar de contenedorización de aplicaciones y el servidor de aplicaciones si aún no está instalado. Puede descargar la herramienta desde [aquí](https://aka.ms/webdeploy3.6).
**Servidores de aplicaciones** | Habilite la comunicación remota de PowerShell en los servidores de aplicaciones: inicie sesión en el servidor de aplicaciones y siga [estas](/powershell/module/microsoft.powershell.core/enable-psremoting) instrucciones para activar la comunicación remota de PowerShell. <br/><br/> Si el servidor de aplicaciones ejecuta Windows Server 2008 R2, asegúrese de que esté instalada la versión 5.1 de PowerShell en el servidor de aplicaciones. Siga las instrucciones que se indican [aquí](/powershell/scripting/windows-powershell/wmf/setup/install-configure) para descargar e instalar PowerShell 5.1 en el servidor de aplicaciones. <br/><br/> Instale la herramienta Microsoft Web Deploy en el equipo que ejecuta la herramienta auxiliar de contenedorización de aplicaciones y el servidor de aplicaciones si aún no está instalado. Puede descargar la herramienta desde [aquí](https://aka.ms/webdeploy3.6).
**Aplicación ASP.NET** | La herramienta admite actualmente <br/><br/> - aplicaciones ASP.NET que usan Microsoft .NET Framework 3.5 o posterior.<br/> - servidores de aplicaciones que ejecutan Windows Server 2008 R2 o posterior (los servidores de aplicaciones deben ejecutar la versión 5.1 de PowerShell). <br/> - aplicaciones que se ejecutan en Internet Information Services (IIS) versión 7.5 o posterior. <br/><br/> La herramienta no admite actualmente <br/><br/> - aplicaciones que requieren la autenticación de Windows (AKS actualmente no admite gMSA). <br/> - aplicaciones que dependen de otros servicios de Windows hospedados fuera de IIS.


## <a name="prepare-an-azure-user-account"></a>Preparación de una cuenta de usuario de Azure

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/pricing/free-trial/) antes de empezar.

Una vez configurada la suscripción, necesitará una cuenta de usuario de Azure que tenga lo siguiente:
- Permisos de Propietario en la suscripción a Azure.
- Permisos para registrar aplicaciones de Azure Active Directory.

Si acaba de crear una cuenta de Azure gratuita, es el propietario de la suscripción. Si no es el propietario, trabaje con él para asignar los permisos, como se indica a continuación:

1. En Azure Portal, busque "suscripciones" y, en **Servicios**, seleccione **Suscripciones**.

    ![Cuadro para buscar la suscripción a Azure.](./media/tutorial-discover-vmware/search-subscription.png)

2. En la página **Suscripciones**, seleccione aquella en la que desee crear un proyecto de Azure Migrate.
3. En la suscripción, seleccione **Access control (IAM)**  > **Comprobar acceso**.
4. En **Comprobar acceso**, busque la cuenta de usuario correspondiente.
5. En **Agregar una asignación de roles**, haga clic en **Agregar**.

    ![Búsqueda de una cuenta de usuario para comprobar el acceso y asignar un rol.](./media/tutorial-discover-vmware/azure-account-access.png)

6. En **Agregar asignación de roles**, seleccione el rol Propietario y seleccione la cuenta (azmigrateuser en el ejemplo). A continuación, haga clic en **Save**(Guardar).

    ![Se abre la página Agregar asignación de roles para asignar un rol a la cuenta.](./media/tutorial-discover-vmware/assign-role.png)

7. Su cuenta de Azure también necesita **permisos para registrar aplicaciones de Azure Active Directory**.
8. En Azure Portal, vaya a **Azure Active Directory** > **Usuarios** > **Configuración de usuario**.
9. En **Configuración de usuario**, compruebe que los usuarios de Azure AD puedan registrar aplicaciones (establecido en **Sí** de forma predeterminada).

      ![Comprobación en la configuración de usuario de que los usuarios puedan registrar aplicaciones de Active Directory.](./media/tutorial-discover-vmware/register-apps.png)

10. Si la configuración de "Registros de aplicaciones" se ha establecido en "No", solicite al administrador global o de inquilinos que asigne el permiso necesario. Como alternativa, el administrador global o de inquilinos puede asignar el rol **Desarrollador de aplicaciones** a una cuenta para permitir el registro de aplicaciones de Azure Active Directory. [Más información](../active-directory/fundamentals/active-directory-users-assign-role-azure-portal.md).


## <a name="download-and-install-azure-migrate-app-containerization-tool"></a>Descarga e instalación de la herramienta de contenedorización de aplicaciones de Azure Migrate

1. [Descargue](https://go.microsoft.com/fwlink/?linkid=2134571) el instalador de la contenedorización de aplicaciones de Azure Migrate en una máquina Windows.
2. Inicie PowerShell en modo de administrador y cambie el directorio de PowerShell a la carpeta que contiene el instalador.
3. Ejecute el script de instalación mediante el comando siguiente.

   ```powershell
   .\AppContainerizationInstaller.ps1
   ```

## <a name="launch-the-app-containerization-tool"></a>Inicio de la herramienta de contenedorización de aplicaciones

1. Abra un explorador en cualquier máquina que pueda conectarse a la máquina Windows que ejecuta la herramienta de contenedorización de aplicaciones y abra su dirección URL: **https://*nombre de la máquina o dirección IP*: 44369**.

   También puede abrir la aplicación desde el escritorio, para lo que debe seleccionar el acceso directo de la aplicación.

2. Si ve una advertencia que indica que la conexión no es privada, haga clic en Opciones avanzadas y elija Continuar al sitio web. Se muestra esta advertencia, ya que la interfaz web usa un certificado TLS/SSL autofirmado.
3. En la pantalla de inicio de sesión, use la cuenta de administrador local en la máquina para iniciar sesión.
4. Para especificar el tipo de aplicación, seleccione **Aplicaciones web ASP.NET** como el tipo de aplicación que quiere incluir en un contenedor.

    ![Carga predeterminada de la herramienta de contenedorización de aplicaciones.](./media/tutorial-containerize-apps-aks/tool-home.png)


### <a name="complete-tool-pre-requisites"></a>Requisitos previos completos de la herramienta
1. Acepte los **términos de licencia** y lea la información de terceros.
6. En la aplicación web de la herramienta > **Configurar los requisitos previos**, complete los pasos siguientes:
   - **Conectividad**: la herramienta comprueba que la máquina Windows tenga acceso a Internet. Si la máquina usa un proxy:
     - Haga clic en **Configurar el proxy** y especifique la dirección del proxy (con los formatos de dirección IP o FQDN) y el puerto de escucha.
     - Especifique las credenciales si el proxy requiere autenticación.
     - Solo se admite un proxy HTTP.
     - Si agregó detalles del proxy o deshabilitó el proxy o la autenticación, haga clic en **Guardar** para desencadenar nuevamente la comprobación de conectividad.
   - **Instalación de actualizaciones**: la herramienta buscará automáticamente las actualizaciones más recientes y las instalará. También puede instalar manualmente la versión más reciente de la herramienta desde [aquí](https://go.microsoft.com/fwlink/?linkid=2134571).
   - **Instale la herramienta Microsoft Web Deploy**: se comprobará que la herramienta Microsoft Web Deploy está instalada en la máquina Windows que ejecuta la herramienta Azure Migrate: App Containerization.
   - **Habilite la comunicación remota de PowerShell**: la herramienta le informará de que está habilitada la comunicación remota de PowerShell en los servidores de aplicaciones que ejecutan las aplicaciones ASP.NET que se incluirán en contenedores.


## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Haga clic en **Iniciar sesión** para iniciar sesión en la cuenta de Azure.

1. Necesitará un código de dispositivo para autenticarse con Azure. Al hacer clic en Iniciar sesión, se abrirá una ventana modal con el código del dispositivo.
2. Haga clic en **Copiar código e Iniciar sesión** para copiar el código del dispositivo y abrir un símbolo del sistema de inicio de sesión de Azure en una nueva pestaña del explorador. Si no aparece, asegúrese de que ha deshabilitado el bloqueador de elementos emergentes en el explorador.

    ![Ventana modal que muestra el código del dispositivo.](./media/tutorial-containerize-apps-aks/login-modal.png)

3. En la pestaña nueva, pegue el código del dispositivo y finalice el inicio de sesión con sus credenciales de la cuenta de Azure. Puede cerrar la pestaña del explorador una vez completado el inicio de sesión y volver a la interfaz web de la herramienta de contenedorización de aplicaciones.
4. Seleccione el **inquilino de Azure** que quiere usar.
5. Especifique la **suscripción a Azure** que quiera usar.

## <a name="discover-aspnet-applications"></a>Supervisión de aplicaciones ASP.NET

La herramienta auxiliar de contenedorización de aplicaciones se conecta de forma remota a los servidores de aplicaciones con las credenciales proporcionadas e intenta detectar las aplicaciones ASP.NET hospedadas en dichos servidores de aplicaciones.

1. Especifique la **dirección IP/FQDN y las credenciales** del servidor que ejecuta la aplicación ASP.NET que se deben usar para conectarse de forma remota al servidor para la detección de las aplicaciones.
    - Las credenciales especificadas deben corresponder a un administrador local (Windows) en el servidor de aplicaciones.
    - En el caso de las cuentas de dominio (el usuario debe ser un administrador en el servidor de aplicaciones), anteponga el nombre de dominio con el formato *<dominio\nombre de usuario>* .
    - Puede ejecutar la detección de aplicaciones para hasta cinco servidores de manera simultánea.

2. Haga clic en **Validar** para comprobar que puede conectarse al servidor de aplicaciones desde la máquina que ejecuta la herramienta y que las credenciales son válidas. Una vez validada correctamente, la columna Estado tendrá el valor **Asignado**.  

    ![Captura de pantalla de la dirección IP y las credenciales del servidor.](./media/tutorial-containerize-apps-aks/discovery-credentials.png)

3. Haga clic en **Continuar** para iniciar la detección de aplicaciones en los servidores de aplicaciones seleccionados.

4. Después de completar correctamente la detección de la aplicación, puede seleccionar la lista de aplicaciones para incluir en contenedores.

    ![Captura de pantalla de la aplicación ASP.NET detectada.](./media/tutorial-containerize-apps-aks/discovered-app.png)


4. Use la casilla para seleccionar las aplicaciones que se incluirán en contenedores.
5. **Especifique el nombre del contenedor**: especifique un nombre para el contenedor de destino de cada aplicación seleccionada. El nombre del contenedor debe especificarse como <*nombre:etiqueta*>, donde la etiqueta se use para la imagen del contenedor. Por ejemplo, puede especificar el nombre del contenedor de destino como *appname:v1*.   

### <a name="parameterize-application-configurations"></a>Parametrización de las configuraciones de la aplicación
Al parametrizar la configuración, esta pasa a estar disponible como parámetro en el momento de la implementación. Esto le permite configurar este valor al implementar la aplicación, en lugar de codificar dicho valor de forma rígida en un valor específico en la imagen del contenedor. Por ejemplo, esta opción es útil para parámetros como las cadenas de conexión de base de datos.
1. Haga clic en **Configuraciones de aplicaciones** para revisar las configuraciones detectadas.
2. Active la casilla para parametrizar las configuraciones de aplicaciones detectadas.
3. Haga clic en **Aplicar** después de seleccionar las configuraciones que parametrizarán.

   ![Captura de pantalla de la parametrización de la configuración de una aplicación ASP.NET.](./media/tutorial-containerize-apps-aks/discovered-app-configs.png)

### <a name="externalize-file-system-dependencies"></a>Externalización de las dependencias del sistema de archivos

 Puede agregar otras carpetas que use la aplicación. Especifique si deben formar parte de la imagen de contenedor o se externalizarán a través de volúmenes persistentes en el recurso compartido de archivos de Azure. El uso de volúmenes persistentes funciona bien con las aplicaciones con estado que almacenan dicho estado fuera del contenedor o almacenan otro contenido estático en el sistema de archivos. [Más información](../aks/concepts-storage.md)

1. Haga clic en **Editar** en Carpetas de aplicaciones para revisar las carpetas de aplicaciones detectadas. Las carpetas de la aplicación detectadas se han identificado como artefactos obligatorios que la aplicación necesita y se copiarán en la imagen del contenedor.

2. Haga clic en **Agregar carpetas** y especifique las rutas de acceso de las carpetas que se agregarán.
3. Para agregar varias carpetas al mismo volumen, especifique valores separados por comas (`,`).
4. Seleccione **Volumen persistente** como opción de almacenamiento si quiere que las carpetas se almacenen fuera del contenedor en un volumen persistente.
5. Haga clic en **Guardar** después de revisar las carpetas de la aplicación.
   ![Captura de pantalla de la selección del almacenamiento de volúmenes de las aplicaciones.](./media/tutorial-containerize-apps-aks/discovered-app-volumes.png)

6. Haga clic en **Continuar** para continuar con la fase de compilación de la imagen de contenedor.

## <a name="build-container-image"></a>Compilar la imagen del contenedor


1. **Seleccione una instancia de Azure Container Registry**: use la lista desplegable para seleccionar una instancia de [Azure Container Registry](../container-registry/index.yml) que se usará para compilar y almacenar las imágenes de contenedor de las aplicaciones. Puede usar una instancia de Azure Container Registry existente o crear una mediante la opción Crear nuevo registro.

    ![Captura de pantalla de la selección de una instancia de ACR de la aplicación.](./media/tutorial-containerize-apps-aks/build-aspnet-app.png)


2. **Revise el archivo Dockerfile**: el archivo Dockerfile necesario para compilar las imágenes de contenedor de cada aplicación seleccionada se genera al principio del paso de compilación. Haga clic en **Revisar** para revisar el archivo Dockerfile. También puede agregar cualquier personalización necesaria al archivo Dockerfile en el paso de revisión y guardar los cambios antes de iniciar el proceso de compilación.

3. **Desencadene el proceso de compilación**: seleccione las aplicaciones para las que se van a compilar imágenes y haga clic en **Compilar**. Al hacer clic en Compilar, se iniciará la compilación de la imagen de contenedor para cada aplicación. La herramienta supervisa continuamente el estado de la compilación y le permitirá continuar con el paso siguiente tras completar la compilación correctamente.

4. **Haga un seguimiento del estado de la compilación**: también puede supervisar el progreso del paso de compilación si hace clic en el vínculo **Compilación en curso** en la columna Estado. El vínculo tarda un par de minutos en activarse después de haber desencadenado el proceso de compilación.  

5. Una vez completada la compilación, haga clic en **Continuar** para especificar la configuración de implementación.

    ![Captura de pantalla de la finalización de compilación de la imagen de contenedor de la aplicación.](./media/tutorial-containerize-apps-aks/build-aspnet-app-completed.png)

## <a name="deploy-the-containerized-app-on-aks"></a>Implementación de la aplicación contenedorizada en AKS

Una vez compilada la imagen de contenedor, el paso siguiente es implementar la aplicación como un contenedor en [Azure Kubernetes Service (AKS)](https://azure.microsoft.com/services/kubernetes-service/).

1. **Seleccione el clúster de Azure Kubernetes Service**: especifique el clúster de AKS en el que se debe implementar la aplicación.

     - El clúster de AKS seleccionado debe tener un grupo de nodos de Windows.
     - El clúster debe estar configurado para permitir la extracción de imágenes desde la instancia de Azure Container Registry que se seleccionó para almacenar las imágenes.
         - Ejecute el comando siguiente en la CLI de Azure para conectar el clúster de AKS a la instancia de ACR.
           ``` Azure CLI
           az aks update -n <cluster-name> -g <cluster-resource-group> --attach-acr <acr-name>
           ```  
     - Si no tiene un clúster de AKS o quiere crear un nuevo clúster de AKS para implementar la aplicación en él, puede optar por crear uno desde la herramienta. Para ello, haga clic en **Crear un nuevo clúster de AKS**.      
          - El clúster de AKS creado mediante la herramienta se creará con un grupo de nodos de Windows. El clúster se configurará para que pueda extraer imágenes de la instancia de Azure Container Registry que se creó anteriormente (si se eligió la opción Crear un nuevo registro).
     - Haga clic en **Continuar** después de seleccionar el clúster de AKS.

2. **Especifique el recurso compartido de archivos de Azure**: si agregó más carpetas y seleccionó la opción de volumen persistente, especifique el recurso compartido de archivos de Azure que debe usar la herramienta de contenedorización de aplicaciones de Azure Migrate durante el proceso de implementación. La herramienta creará directorios nuevos en este recurso compartido de archivos de Azure para copiarlos a las carpetas de la aplicación que están configuradas para el almacenamiento de volumen persistente. Una vez completada la implementación de la aplicación, la herramienta limpiará el recurso compartido de archivos de Azure mediante la eliminación de los directorios que se crearon.

     - Si no tiene un recurso compartido de archivos de Azure o quiere crear uno nuevo, puede optar por crear uno desde la herramienta. Para ello, haga clic en **Crear nueva cuenta de almacenamiento y recurso compartido de archivos**.  

3. **Configuración de la implementación de aplicaciones**: una vez completados los pasos anteriores, deberá especificar la configuración de implementación de la aplicación. Haga clic en **Configurar** para personalizar la implementación de la aplicación. En el paso de configuración puede especificar las siguientes personalizaciones:
     - **Cadena de prefijo**: especifique una cadena de prefijo que se usará en el nombre de todos los recursos que se creen para la aplicación contenedorizada en el clúster de AKS.
     - **Certificado SSL**: si la aplicación requiere un enlace de sitio HTTPS, especifique el archivo PFX que contiene el certificado que se usará para el enlace. El archivo PFX no debe estar protegido por contraseña y el sitio original no debe tener varios enlaces.
     - **Conjuntos de réplicas**: especifique el número de instancias de aplicación (pods) que deben ejecutarse dentro de los contenedores.
     - **Tipo de equilibrador de carga**: seleccione *Externo* si la aplicación contenedorizada debe ser accesible desde redes públicas.
     - **Configuración de la aplicación**: para cualquier configuración de aplicación que se haya parametrizado, especifique los valores que se usarán para la implementación actual.
     - **Almacenamiento**: para todas las carpetas de aplicación que se hayan configurado para el almacenamiento de volumen persistente, especifique si el volumen debe compartirse entre instancias de la aplicación, o bien si debe inicializarse individualmente con cada instancia del contenedor. De manera predeterminada, todas las carpetas de aplicación en volúmenes persistentes están configuradas como compartidas.  
     - Haga clic en **Aplicar** para guardar la configuración de implementación.
     - Haga clic en **Continuar** para implementar la aplicación.

    ![Captura de pantalla de la configuración de la implementación de la aplicación.](./media/tutorial-containerize-apps-aks/deploy-aspnet-app-config.png)

4. **Implemente la aplicación**: una vez guardada la configuración de implementación de la aplicación, la herramienta generará el archivo YAML de implementación de Kubernetes para la aplicación.
     - Haga clic en **Editar** para revisar y personalizar el archivo YAML de implementación de Kubernetes para las aplicaciones.
     - Seleccione la aplicación que quiere implementar.
     - Haga clic en **Implementar** para iniciar la implementación de las aplicaciones seleccionadas.

         ![Captura de pantalla de la configuración de implementación de las aplicaciones.](./media/tutorial-containerize-apps-aks/deploy-aspnet-app-deploy.png)

     - Una vez implementada la aplicación, puede hacer clic en la columna *Estado de implementación* para realizar el seguimiento de los recursos que se implementaron para la aplicación.

## <a name="download-generated-artifacts"></a>Descarga de artefactos generados

Todos los artefactos que se usan para compilar e implementar la aplicación en AKS, incluidos los archivos de especificación YAML de Dockerfile y Kubernetes, se almacenan en el equipo que ejecuta la herramienta. Los artefactos se encuentran en *C:\ProgramData\Microsoft Azure Migrate App Containerization*.

Se crea una sola carpeta para cada servidor de aplicaciones. Puede ver y descargar todos los artefactos intermedios que se usan en el proceso de contenedorización si se desplaza a esta carpeta. La carpeta correspondiente al servidor de aplicaciones se limpiará al principio de cada ejecución de la herramienta para un servidor determinado.

## <a name="troubleshoot-issues"></a>Solución de problemas

Para solucionar cualquier problema con la herramienta, puede ver los archivos de registro en la máquina Windows que ejecuta la herramienta de contenedorización de aplicaciones. Los archivos de registro de la herramienta se encuentran en *C:\ProgramData\Microsoft Azure Migrate App Containerization\Logs*.

## <a name="next-steps"></a>Pasos siguientes

- Inclusión en contenedores de aplicaciones web de Java en Apache Tomcat (en servidores Linux) e implementación en contenedores de Linux en AKS. [Más información](./tutorial-containerize-java-kubernetes.md)
