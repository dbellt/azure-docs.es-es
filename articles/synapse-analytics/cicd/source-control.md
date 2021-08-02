---
title: Control de código fuente en Synapse Studio
description: Obtenga información sobre cómo configurar el control de código fuente en Azure Synapse Studio
author: liud
ms.service: synapse-analytics
ms.subservice: cicd
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: liud
ms.reviewer: pimorano
ms.openlocfilehash: 7371bc023cc13278c0d1382ac1ac60ee42a39f0b
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109809147"
---
# <a name="source-control-in-synapse-studio"></a>Control de código fuente en Synapse Studio

De forma predeterminada, Synapse Studio crea directamente en el servicio Synapse. Si necesita colaboración al usar Git para el control de código fuente, Synapse Studio permite asociar el área de trabajo a un repositorio de Git, Azure DevOps o GitHub. 

En este artículo se describe cómo configurar un área de trabajo de Synapse y trabajar con él con el repositorio de Git habilitado. Además, destacamos algunos procedimientos recomendados y una guía de solución de problemas.

> [!NOTE]
> La integración de Git de Synapse Studio no está disponible en la nube de Azure Government.

## <a name="configure-git-repository-in-synapse-studio"></a>Configuración del repositorio de Git en Synapse Studio 

Después de iniciar Synapse Studio, puede configurar un repositorio de Git en el área de trabajo. Un área de trabajo de Synapse Studio solo se puede asociar a un repositorio de Git a la vez. 

### <a name="configuration-method-1-global-bar"></a>Método de configuración 1: barra global

En la barra global de Synapse Studio, seleccione el menú desplegable **Synapse Live** (Modo activo de Synapse) y luego seleccione **Set up code repository** (Configurar el repositorio de código).

![Configuración de los parámetros del repositorio de código desde la creación](media/configure-repo-1.png)

### <a name="configuration-method-2-manage-hub"></a>Método de configuración 2: Centro de administración

Vaya al centro de administración de Synapse Studio. Seleccione **Configuración de Git** en la sección **Control de código fuente**. Si no tiene ningún repositorio conectado, haga clic en **Configurar**.

![Configuración del repositorio de código en el centro de administración](media/configure-repo-2.png)

> [!NOTE]
> Los usuarios a los que se les ha concedido el rol de colaborador del área de trabajo, propietario o roles de nivel superior, pueden configurar, editar la configuración y desconectar el repositorio de Git en Azure Synapse Studio. 

Puede conectar el repositorio de Git de Azure DevOps o GitHub en el área de trabajo.

## <a name="connect-with-azure-devops-git"></a>Conexión con Git de Azure DevOps 

Puede asociar un área de trabajo de Synapse con un repositorio de Azure DevOps para el control del código fuente, la colaboración, el control de versiones, etc. Si no tiene un repositorio de Azure DevOps, siga [estas instrucciones](/azure/devops/organizations/accounts/create-organization-msa-or-work-student) para crear los recursos de repositorio.

### <a name="azure-devops-git-repository-settings"></a>Configuración del repositorio de Git de Azure DevOps

Al conectarse al repositorio de Git, seleccione primero el tipo de repositorio como Git de Azure DevOps y luego seleccione un inquilino de Azure AD en la lista desplegable y haga clic en **Continuar**.

![Configuración del repositorio de código](media/connect-with-azuredevops-repo-selected.png)

En el panel configuración se muestra la siguiente configuración de Git de Azure DevOps:

| Configuración | Descripción | Value |
|:--- |:--- |:--- |
| **Tipo de repositorio** | Tipo de repositorio de código de Azure Repos.<br/> | Git o GitHub de Azure DevOps |
| **Azure Active Directory** | El nombre de su inquilino de Azure AD. | `<your tenant name>` |
| **Cuenta de Azure DevOps** | Nombre de la organización de Azure Repos. Puede buscar el nombre de la organización de Azure Repos en `https://{organization name}.visualstudio.com`. Puede [iniciar sesión en la organización de Azure Repos](https://www.visualstudio.com/team-services/git/) para acceder a su perfil de Visual Studio y ver sus proyectos y repositorios. | `<your organization name>` |
| **ProjectName** | Nombre de proyecto de Azure Repos. Puede buscar el nombre de proyecto de Azure Repos en `https://{organization name}.visualstudio.com/{project name}`. | `<your Azure Repos project name>` |
| **RepositoryName** | Nombre del repositorio de código de Azure Repos. Los proyectos de Azure Repos contienen repositorios de Git para administrar el código fuente a medida que crece el proyecto. Puede crear un repositorio nuevo o usar uno existente en el proyecto. | `<your Azure Repos code repository name>` |
| **Rama de colaboración** | Rama de colaboración de Azure Repos que se usa para la publicación. De forma predeterminada, es `master`. Cámbiela en caso de que desee publicar recursos de otra rama. Puede seleccionar ramas existentes o crear nuevas. | `<your collaboration branch name>` |
| **Carpeta raíz** | Carpeta raíz de la rama de colaboración de Azure Repos. | `<your root folder name>` |
| **Import existing resources to repository** (Importar recursos existentes en el repositorio). | Especifica si se deben importar los recursos existentes de Synapse Studio en un repositorio de Git de Azure Repos. Active la casilla para importar los recursos del área de trabajo (excepto los grupos) en el repositorio de Git asociado en formato JSON. Esta acción exporta cada recurso individualmente. Cuando esta casilla no está activada, no se importan los recursos existentes. | Activada (de forma predeterminada) |
| **Import resource into this branch** (Importar recurso en esta rama) | Seleccione la rama en la que se importan los recursos (script de SQL, cuaderno, definición de trabajo de Spark, conjunto de datos, flujo de datos, etc.). 

También puede usar el vínculo de repositorio para apuntar rápidamente al repositorio de Git con el que desea conectarse. 

### <a name="use-a-different-azure-active-directory-tenant"></a>Uso de un inquilino de Azure Active Directory distinto

El repositorio Git de Azure Repos puede estar en un inquilino de Azure Active Directory distinto. Para especificar otro inquilino de Azure AD, deberá tener permisos de administrador para la suscripción de Azure que utilice. Para más información, consulte [Agregar o cambiar los administradores de la suscripción de Azure](../../cost-management-billing/manage/add-change-subscription-administrator.md#assign-a-subscription-administrator).

> [!IMPORTANT]
> Para conectarse a otro directorio de Azure Active Directory, el usuario que ha iniciado sesión debe formar parte de ese directorio de Active Directory. 

### <a name="use-your-personal-microsoft-account"></a>Uso de la cuenta de Microsoft personal

Para usar una cuenta de Microsoft personal para la integración de Git, puede vincular el repositorio de Azure personal al entorno de Active Directory de su organización.

1. Agregue la cuenta de Microsoft personal al entorno de Active Directory de su organización como invitado. Para más información, vea [Incorporación de usuarios de colaboración B2B de Azure Active Directory en Azure Portal](../../active-directory/external-identities/add-users-administrator.md).

2. Inicie sesión en Azure Portal con la cuenta de Microsoft personal. A continuación, cambie al entorno de Active Directory de su organización.

3. Vaya a la sección de Azure DevOps, donde verá ahora el repositorio personal. Seleccione el repositorio y establezca la conexión con Active Directory.

Después de estos pasos de configuración, su repositorio personal está disponible al configurar la integración de Git en la interfaz de usuario de Synapse Studio.

Para más información sobre cómo conectar Azure Repos al entorno de Active Directory de su organización, vea [Conexión de la organización a Azure Active Directory](/azure/devops/organizations/accounts/connect-organization-to-azure-ad).

## <a name="connect-with-github"></a>Conexión con GitHub 

 Puede asociar un área de trabajo con un repositorio de GitHub para el control del código fuente, la colaboración y el control de versiones. Si no tiene un repositorio o una cuenta de GitHub, siga [estas instrucciones](https://github.com/join) para crear los recursos.

La integración de GitHub con Synapse Studio admite tanto GitHub público (es decir, [https://github.com](https://github.com)) como GitHub Enterprise. Puede usar los repositorios de GitHub públicos y privados siempre que tenga permisos de lectura y escritura para el repositorio en GitHub.

### <a name="github-settings"></a>Configuración de GitHub

Al conectarse al repositorio de Git, seleccione primero el tipo de repositorio como GitHub y luego proporcione la cuenta de GitHub o la dirección URL del servidor de GitHub Enterprise (si usa dicho servidor) y haga clic en **Continuar**.

![Configuración del repositorio de GitHub](media/connect-with-github-repo-1.png)

En el panel configuración se muestra la siguiente configuración del repositorio de GitHub:

| **Configuración** | **Descripción**  | **Valor**  |
|:--- |:--- |:--- |
| **Tipo de repositorio** | Tipo de repositorio de código de Azure Repos. | GitHub |
| **Use GitHub Enterprise** (Usar GitHub Enterprise) | Casilla para seleccionar GitHub Enterprise | no seleccionado (valor predeterminado) |
| **GitHub Enterprise URL** (URL de GitHub Enterprise) | La dirección URL de GitHub Enterprise (debe ser HTTPS para el servidor local de GitHub Enterprise). Por ejemplo: `https://github.mydomain.com`. Solo es necesario si se selecciona **Use GitHub Enterprise** (Usar GitHub Enterprise). | `<your GitHub enterprise url>` |                                                           
| **Cuenta de GitHub** | El nombre de la cuenta de GitHub. Este nombre se puede encontrar en https:\//github.com/{nombre de cuenta}/{nombre de repositorio}. Para ir a esta página se le pide que introduzca credenciales de OAuth de GitHub en la cuenta de GitHub. | `<your GitHub account name>` |
| **Nombre del repositorio**  | El nombre del repositorio de código de GitHub. Las cuentas de GitHub contienen repositorios de Git para administrar el código fuente. Puede crear un repositorio o usar uno existente que ya se encuentre en su cuenta. | `<your repository name>` |
| **Rama de colaboración** | La rama de colaboración de GitHub que se usa para la publicación. De forma predeterminada, es master. Cámbiela en caso de que desee publicar recursos de otra rama. | `<your collaboration branch>` |
| **Carpeta raíz** | La carpeta raíz de la rama de colaboración de GitHub. |`<your root folder name>` |
| **Import existing resources to repository** (Importar recursos existentes en el repositorio). | Especifica si se deben importar los recursos existentes de Synapse Studio en un repositorio de Git. Active la casilla para importar los recursos del área de trabajo (excepto los grupos) en el repositorio de Git asociado en formato JSON. Esta acción exporta cada recurso individualmente. Cuando esta casilla no está activada, no se importan los recursos existentes. | Activada (valor predeterminado) |
| **Import resource into this branch** (Importar recurso en esta rama) | Seleccione la rama en la que se importan los recursos (script de SQL, cuaderno, definición de trabajo de Spark, conjunto de datos, flujo de datos, etc.). 

### <a name="github-organizations"></a>Organizaciones de GitHub

La conexión a una organización de GitHub requiere que la organización conceda permiso a Synapse Studio. Un usuario con permisos de administrador en la organización debe realizar los pasos siguientes.

#### <a name="connecting-to-github-for-the-first-time"></a>Conexión a GitHub por primera vez

Si se va a conectar a GitHub desde Synapse Studio por primera vez, siga estos pasos para conectarse a una organización de GitHub.

1. En el panel de configuración de Git, escriba el nombre de la organización en el campo *Cuenta de GitHub*. Aparecerá una solicitud para iniciar sesión en GitHub. 

1. Inicie sesión con sus credenciales de usuario.

1. Se le pedirá que autorice a Synapse como una aplicación llamada *Azure Synapse*. En esta pantalla, verá una opción para conceder permiso a Synapse para acceder a la organización. Si no ve la opción para conceder el permiso, pida a un administrador que conceda el permiso manualmente a través de GitHub.

Una vez que siga estos pasos, su área de trabajo podrá conectarse a repositorios públicos y privados dentro de la organización. Si no puede conectarse, pruebe a borrar la memoria caché del explorador y vuelva a intentarlo.

#### <a name="already-connected-to-github-using-a-personal-account"></a>Ya está conectado a GitHub con una cuenta personal

Si ya se ha conectado a GitHub y solo se le ha concedido permiso para obtener acceso a una cuenta personal, siga estos pasos para conceder permisos a una organización.

1. Vaya a GitHub y abra **Configuración**.

    ![Apertura de la configuración de GitHub](media/github-settings.png)

1. Seleccione **Aplicaciones**. En la pestaña **Aplicaciones de OAuth autorizadas**, verá *Azure Synapse*.

    ![Aplicaciones de OAuth autorizadas](media/authorize-app.png)

1. Seleccione *Azure Synapse* y conceda el acceso a su organización.

    ![Concesión de permiso de organización](media/grant-organization-permission.png)

Una vez completados estos pasos, su área de trabajo podrá conectarse a repositorios públicos y privados dentro de la organización.

## <a name="version-control"></a>Control de versiones

Los sistemas de control de versiones (también conocidos como _control de código fuente_) permiten a los desarrolladores colaborar en el código y realizar el seguimiento de los cambios. El control de código fuente es una herramienta esencial para proyectos de varios desarrolladores.

### <a name="creating-feature-branches"></a>Creación de ramas de características

Tan pronto como cada repositorio de Git se asocia a una instancia de Synapse Studio, tiene una rama de colaboración. (`main` o `master` es la rama de colaboración predeterminada). Los usuarios también pueden crear ramas de características; para ello, pueden hacer clic en **+ New Branch** (+ Nueva rama) en la lista desplegable de ramas. Cuando aparezca el panel de nueva rama, escriba el nombre de la rama de características.

![Creación de una rama](media/create-new-branch.png)

Cuando esté listo para combinar los cambios de su rama de características con la rama de colaboración, haga clic en la lista desplegable de ramas y seleccione **Create pull request** (Crear solicitud de incorporación de cambios). Esto le llevará al proveedor de Git, donde podrá generar la solicitud de incorporación de cambios, revisar el código y combinar los cambios en su rama de colaboración. Solo tiene permitido publicar en el servicio Synapse de la rama de colaboración. 

![Creación de una nueva solicitud de incorporación de cambios](media/create-pull-request.png)

### <a name="configure-publishing-settings"></a>Configurar los valores de publicación

De forma predeterminada, Synapse Studio genera las plantillas del área de trabajo y las guarda en una rama llamada `workspace_publish`. Para configurar una rama de publicación personalizada, agregue un archivo `publish_config.json` a la carpeta raíz de la rama de colaboración. Durante la publicación, Synapse Studio lee este archivo, busca el campo `publishBranch` y guarda los archivos de plantilla de área de trabajo en la ubicación especificada. Si no existe la rama, Synapse Studio la crea automáticamente. A continuación, se muestra un ejemplo de cuál es el aspecto de este archivo:

```json
{
    "publishBranch": "workspace_publish"
}
```

Synapse Studio no puede tener varias ramas de publicación simultaneas. Al especificar una nueva rama de publicación, la rama de publicación anterior no se elimina. Si quiere quitar la rama de publicación anterior, tiene que hacerlo manualmente.


### <a name="publish-code-changes"></a>Cambios de código publicando

Después de combinar los cambios en la rama de colaboración, haga clic en **Publicar** para publicar manualmente los cambios de código realizados en la rama de colaboración en el servicio Synapse.

![Publicación de cambios](media/gitmode-publish.png)

Se abrirá un panel lateral en el que se confirma que la rama de publicación y los cambios pendientes son correctos. Cuando haya comprobado los cambios, haga clic en **OK** (Aceptar) para confirmar la publicación.

![Confirmar la rama de publicación correcta](media/publish-change.png)

> [!IMPORTANT]
> La rama de colaboración no es representativa de lo que se implementa en el servicio. Los cambios en la rama de colaboración *deben* publicarse manualmente en el servicio.

## <a name="switch-to-a-different-git-repository"></a>Cambio a un repositorio de Git diferente

Para cambiar a un repositorio de Git diferente, vaya a la página de configuración de Git en el centro de administración en **Control de código fuente**. Seleccione **Desconectar**. 

![Icono de Git](media/remove-repository.png)

Escriba el nombre del área de trabajo y haga clic en **Desconectar** para quitar el repositorio de Git asociado al área de trabajo.

Después de quitar la asociación con el repositorio actual, puede configurar las opciones de Git para usar un repositorio diferente y, luego, importar los recursos existentes en el nuevo repositorio.

> [!IMPORTANT]
> La eliminación de la configuración de Git de un área de trabajo no elimina nada del repositorio. El área de trabajo de Synapse contendrá todos los recursos publicados. Puede seguir editando el área de trabajo directamente en el servicio.

## <a name="best-practices-for-git-integration"></a>Procedimientos recomendados para la integración de Git

-   **Permisos**. Una vez que tenga un repositorio de Git conectado al área de trabajo, cualquiera que pueda acceder al repositorio de Git con cualquier rol en el área de trabajo podrá actualizar artefactos, como el script de SQL, el cuaderno, la definición del trabajo de Spark, el conjunto de datos, el flujo de datos y la canalización en modo Git. Lo normal es que no quiera que todos los miembros del equipo tengan permisos para actualizar el área de trabajo. Conceda solo el permiso de repositorio de Git a los autores de artefactos del área de trabajo de Synapse. 
-   **Colaboración**. Se recomienda no permitir inserciones directas en el repositorio en la rama de colaboración. Esta restricción puede ayudarle a evitar errores, ya que cada inserción en el repositorio pasará por un proceso de solicitud de incorporación de cambios, tal como se describe en [Crear una rama de características](source-control.md#creating-feature-branches).
-   **Modo activo de Synapse**. Después de la publicación en el modo Git, todos los cambios se reflejarán en el modo activo de Synapse. En el modo activo de Synapse, la publicación está deshabilitada. Y puede ver y ejecutar artefactos en modo activo si se le ha concedido el permiso adecuado. 
-   **Edite los artefactos en Studio**. Synapse Studio es el único lugar en el que puede habilitar el control de código fuente del área de trabajo y sincronizar los cambios en Git automáticamente. Cualquier cambio a través del SDK, PowerShell, no se sincronizará con Git. Se recomienda editar siempre el artefacto en Studio cuando Git esté habilitado.

## <a name="troubleshooting-git-integration"></a>Solución de problemas de la integración de Git

### <a name="access-to-git-mode"></a>Acceso al modo Git 

Si se le ha concedido el permiso al repositorio de Git de GitHub vinculado con el área de trabajo, pero no puede acceder al modo Git: 

1. Borre la memoria caché y actualice la página. 

1. Inicie sesión en la cuenta de GitHub.

### <a name="stale-publish-branch"></a>Rama de publicación obsoleta

Si la rama de publicación no está sincronizada con la rama de colaboración y contiene recursos no actualizados a pesar de una publicación reciente, pruebe lo siguiente:

1. Quite el repositorio de Git actual.

1. Vuelva a configurar Git con los mismos valores, pero asegúrese de que la opción **Import existing resources to repository** (Importar recursos existentes en el repositorio) esté activada y elija la misma rama.  

1. Cree una solicitud de incorporación de cambios para combinar los cambios con la rama de colaboración. 

## <a name="unsupported-features"></a>Características no admitidas

- Synapse Studio no permite la selección exclusiva de confirmaciones ni la publicación selectiva de recursos. 
- Synapse Studio no admite personalizar el mensaje de confirmación.
- Por diseño, la acción de eliminación en Studio se confirmará en Git directamente.

## <a name="next-steps"></a>Pasos siguientes

* Para implementar la integración e implementación continuas, consulte [Integración y entrega continuas (CI/CD)](continuous-integration-deployment.md).
