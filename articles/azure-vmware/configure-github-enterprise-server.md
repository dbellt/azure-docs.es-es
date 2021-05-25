---
title: Configuración del servidor de GitHub Enterprise en Azure VMware Solution
description: Aprenda a configurar el servidor de GitHub Enterprise en la nube privada de Azure VMware Solution.
ms.topic: how-to
ms.date: 02/11/2021
ms.openlocfilehash: 50e061c319bdde10246433d9b31c28d8db2388a2
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108777008"
---
# <a name="configure-github-enterprise-server-on-azure-vmware-solution"></a>Configuración del servidor de GitHub Enterprise en Azure VMware Solution

En este artículo se recorren los pasos para configurar el servidor de GitHub Enterprise, la versión "local" de [GitHub.com](https://github.com/), en la nube privada de Azure VMware Solution. El escenario que trataremos es una instancia de servidor de GitHub Enterprise que puede atender hasta a 3000 desarrolladores que ejecutan hasta 25 trabajos por minuto en Acciones de GitHub. En el momento de su redacción incluye la configuración de características en *versión preliminar*, como Acciones de GitHub. Para personalizar la configuración conforme a sus necesidades particulares, revise los requisitos indicados en [Instalación del servidor de GitHub Enterprise en VMware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#hardware-considerations).

## <a name="before-you-begin"></a>Antes de empezar

El servidor de GitHub Enterprise requiere una clave de licencia válida. Puede registrarse para obtener una [licencia de prueba](https://enterprise.github.com/trial). Si busca ampliar las funcionalidades del servidor de GitHub Enterprise por medio de una integración, puede optar a una licencia gratuita de desarrollador para cinco usuarios. Aplique esta licencia por medio del [programa de asociados de GitHub](https://partner.github.com/).

## <a name="install-github-enterprise-server-on-vmware"></a>Instalación del servidor de GitHub Enterprise en VMware

Descargue [la versión actual del servidor de GitHub Enterprise](https://enterprise.github.com/releases/2.19.0/download) para VMware ESXi/vSphere (OVA) e [implemente la plantilla de OVA](https://docs.vmware.com/en/VMware-vSphere/6.5/com.vmware.vsphere.vm_admin.doc/GUID-17BEDA21-43F6-41F4-8FB2-E01D275FE9B4.html) que ha descargado.

:::image type="content" source="media/github-enterprise-server/github-options.png" alt-text="Selección de la ejecución de GitHub en el entorno local o en la nube.":::  

:::image type="content" source="media/github-enterprise-server/deploy-ova-template.png" alt-text="Implementación de la plantilla de OVA.":::  

Proporcione un nombre reconocible para la nueva máquina virtual, por ejemplo, GitHubEnterpriseServer. No es necesario incluir los detalles de la versión en el nombre de la máquina virtual, ya que estos detalles se vuelven obsoletos cuando la instancia se actualiza. Por ahora, seleccione todos los valores predeterminados (estos detalles se van a editar en breve) y espere a que se importe la plantilla de OVA.

Una vez importada, [ajuste la configuración de hardware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#creating-the-github-enterprise-server-instance) en función de sus necesidades. En este escenario de ejemplo se necesita la siguiente configuración.

| Resource | Configuración estándar | Configuración estándar + "Características beta" (Acciones) |
| --- | --- | --- |
| vCPU | 4 | 8 |
| Memoria | 32 GB | 61 GB |
| Almacenamiento acoplado | 250 GB | 300 GB |
| Almacenamiento raíz | 200 GB | 200 GB |

Pero las necesidades pueden variar. Vea la guía sobre consideraciones de hardware en [Instalación del servidor de GitHub Enterprise en VMware](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#hardware-considerations). Vea también [Incorporación de recursos de CPU o memoria para VMware](https://docs.github.com/en/enterprise/admin/enterprise-management/increasing-cpu-or-memory-resources#adding-cpu-or-memory-resources-for-vmware) para personalizar la configuración de hardware en función de la situación.

## <a name="configure-the-github-enterprise-server-instance"></a>Configuración de la instancia del servidor de GitHub Enterprise

:::image type="content" source="media/github-enterprise-server/install-github-enterprise.png" alt-text="Instalación de GitHub Enterprise.":::  

Una vez encendida la máquina virtual (VM) que se acaba de aprovisionar, [configúrela mediante el explorador](https://docs.github.com/en/enterprise/admin/installation/installing-github-enterprise-server-on-vmware#configuring-the-github-enterprise-server-instance). Tiene que cargar el archivo de licencia y establecer una contraseña para la consola de administración. Asegúrese de anotar esta contraseña en algún lugar seguro.

:::image type="content" source="media/github-enterprise-server/ssh-access.png" alt-text="Acceso al shell de administración a través de SSH.":::    

Se recomienda realizar al menos los siguientes pasos:

1. Cargue una clave SSH pública en la consola de administración para que pueda [acceder al shell administrativo a través de SSH](https://docs.github.com/en/enterprise/admin/configuration/accessing-the-administrative-shell-ssh). 

2. [Configure TLS en la instancia](https://docs.github.com/en/enterprise/admin/configuration/configuring-tls) para poder usar un certificado firmado por una entidad de certificación de confianza.

:::image type="content" source="media/github-enterprise-server/configuring-your-instance.png" alt-text="Configuración de la instancia.":::

Aplique la configuración.  Mientras se reinicia la instancia, puede ir al paso siguiente, **Configuración de Blob Storage para Acciones de GitHub**.

:::image type="content" source="media/github-enterprise-server/create-admin-account.png" alt-text="Creación de la cuenta de administrador.":::

Una vez que se reinicie la instancia, puede crear una cuenta de administrador en ella. Asegúrese de anotar también esta contraseña de usuario.

### <a name="other-configuration-steps"></a>Otros pasos de configuración

Para reforzar la instancia para su uso en producción, se recomienda seguir los siguientes pasos de configuración opcionales:

1. Configure [alta disponibilidad](https://help.github.com/enterprise/admin/guides/installation/configuring-github-enterprise-for-high-availability/) para la protección frente a:

    - Bloqueos de software (en el nivel de sistema operativo o aplicación)
    - Errores de hardware (almacenamiento, CPU, RAM, etc.)
    - Errores del sistema de host de virtualización
    - Red rota lógica o físicamente

2. [Configure](https://docs.github.com/en/enterprise/admin/configuration/configuring-backups-on-your-appliance) [herramientas de copia de seguridad](https://github.com/github/backup-utils) y proporcione instantáneas con versiones para la recuperación ante desastres hospedadas en una disponibilidad independiente de la instancia principal.
3. [Configure aislamiento de subdominios](https://docs.github.com/en/enterprise/admin/configuration/enabling-subdomain-isolation) mediante un certificado TLS válido para mitigar el scripting entre sitios y otras vulnerabilidades relacionadas.

## <a name="configure-blob-storage-for-github-actions"></a>Configuración de Blob Storage para Acciones de GitHub

> [!NOTE]
> Acciones de GitHub está [disponible actualmente como versión beta limitada en el servidor de GitHub Enterprise versión 2.22](https://docs.github.com/en/enterprise/admin/github-actions).

Se necesita una instancia externa de Blob Storage para habilitar Acciones de GitHub en el servidor de GitHub Enterprise (disponible actualmente como una característica "beta"). Acciones usa esta instancia externa de Blob Storage para almacenar artefactos y registros. Acciones en el servidor de GitHub Enterprise [admite Azure Blob Storage como proveedor de almacenamiento](https://docs.github.com/en/enterprise/admin/github-actions/enabling-github-actions-and-configuring-storage#about-external-storage-requirements) (y algunos otros). Por lo tanto, se va a aprovisionar una nueva cuenta de almacenamiento de Azure con un [tipo de cuenta de almacenamiento](../storage/common/storage-account-overview.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#types-of-storage-accounts) BlobStorage:

:::image type="content" source="media/github-enterprise-server/storage-account.png" alt-text="Aprovisionamiento de la cuenta de Azure Blob Storage.":::

Una vez completada la implementación del nuevo recurso BlobStorage, copie y anote la cadena de conexión (disponible en Claves de acceso). Va a necesitar esta cadena en breve.

## <a name="set-up-the-github-actions-runner"></a>Configuración del ejecutor de Acciones de GitHub

> [!NOTE]
> Acciones de GitHub está [disponible actualmente como versión beta limitada en el servidor de GitHub Enterprise versión 2.22](https://docs.github.com/en/enterprise/admin/github-actions).

En este punto, debe tener una instancia del servidor de GitHub Enterprise en ejecución con una cuenta de administrador creada. También debe tener una instancia externa de Blob Storage que Acciones de GitHub use para la persistencia.

Ahora se va a crear alguna ubicación para que se ejecute Acciones de GitHub; una vez más, se va a usar Azure VMware Solution.

En primer lugar, se va a aprovisionar una nueva máquina virtual en el clúster. La máquina virtual se va a basar en [una versión reciente del servidor Ubuntu](http://releases.ubuntu.com/20.04.1/).

:::image type="content" source="media/github-enterprise-server/provision-new-vm.png" alt-text="Aprovisionamiento de una nueva máquina virtual.":::

:::image type="content" source="media/github-enterprise-server/provision-new-vm-2.png" alt-text="Aprovisionamiento de una nueva máquina virtual, paso 2.":::

Una vez creada la máquina virtual, enciéndala y conéctese a ella a través de SSH.

A continuación, instale la aplicación [ejecutor de Acciones](https://github.com/actions/runner), que ejecuta un trabajo desde un flujo de trabajo de Acciones de GitHub. Identifique y descargue la versión de Linux x64 más reciente del ejecutor de Acciones, ya sea desde [la página de versiones](https://github.com/actions/runner/releases) o mediante la ejecución del siguiente script rápido. Este script requiere que curl y [jq](https://stedolan.github.io/jq/) estén presentes en la máquina virtual.

`LATEST\_RELEASE\_ASSET\_URL=$( curl https://api.github.com/repos/actions/runner/releases/latest | \`

`  jq -r '.assets | .[] | select(.name | match("actions-runner-linux-arm64")) | .url' )`

`DOWNLOAD\_URL=$( curl $LATEST\_RELEASE\_ASSET\_URL | \`

`  jq -r '.browser\_download\_url' )`

`curl -OL $DOWNLOAD\_URL`

Ahora debería tener un archivo local en la máquina virtual: actions-runner-linux-arm64-\*.tar.gz. Extraiga este tarball localmente:

`tar xzf actions-runner-linux-arm64-\*.tar.gz`

Esta extracción desempaqueta algunos archivos en local, incluido un script `config.sh` y `run.sh`, al que se va a volver en breve.

## <a name="enable-github-actions"></a>Habilitación de Acciones de GitHub

> [!NOTE]
> Acciones de GitHub está [disponible actualmente como versión beta limitada en el servidor de GitHub Enterprise versión 2.22](https://docs.github.com/en/enterprise/admin/github-actions).

Casi ha terminado. Vamos a configurar y habilitar Acciones de GitHub en la instancia del servidor de GitHub Enterprise. Es necesario [acceder al shell administrativo de la instancia del servidor de GitHub Enterprise a través de SSH](https://docs.github.com/en/enterprise/admin/configuration/accessing-the-administrative-shell-ssh) y, después, ejecutar los siguientes comandos:

`# set an environment variable containing your Blob storage connection string`

`export CONNECTION\_STRING="<your connection string from the blob storage step>"`

`# configure actions storage`

`ghe-config secrets.actions.storage.blob-provider azure`

`ghe-config secrets.actions.storage.azure.connection-string "$CONNECTION\_STRING"`

`# apply these settings`

`ghe-config-apply`

`# execute a precheck, this install additional software required by Actions on GitHub Enterprise Server`

`ghe-actions-precheck -p azure -cs "$CONNECTION\_STRING"`

`# enable actions, and re-apply the config`

`ghe-config app.actions.enabled true`

`ghe-config-apply`

Siguiente ejecución:

`ghe-actions-check -s blob`

La salida debería ser la siguiente: "Blob Storage is healthy" (Blob Storage correcto).

Ahora que Acciones de GitHub está configurado, habilítelo para los usuarios. Inicie sesión como administrador en la instancia del servidor de GitHub Enterprise y seleccione el ![icono de cohete](media/github-enterprise-server/rocket-icon.png) en la esquina superior derecha de cualquier página. En la barra lateral izquierda, seleccione **Enterprise overview** (Información general de Enterprise), **Directivas**, **Acciones** y seleccione la opción para **habilitar Acciones para todas las organizaciones**.

A continuación, configure el ejecutor desde la pestaña **Self-hosted runners** (Ejecutores autohospedados). Seleccione **Agregar nuevo** y **New runner** (Nuevo ejecutor) en la lista desplegable.

En la página siguiente se le presenta un conjunto de comandos para ejecutar, solo es necesario copiar el comando para **configurar** el ejecutor, por ejemplo:

`./config.sh --url https://10.1.1.26/enterprises/octo-org --token AAAAAA5RHF34QLYBDCHWLJC7L73MA`

Copie el comando `config.sh` y péguelo en una sesión en el ejecutor de Acciones (creado anteriormente).

:::image type="content" source="media/github-enterprise-server/actions-runner.png" alt-text="Ejecutor de Acciones.":::

Use el comando run.sh para *iniciar* el ejecutor:

:::image type="content" source="media/github-enterprise-server/run-runner.png" alt-text="Inicio del ejecutor.":::

Para que este ejecutor esté disponible para las organizaciones de la empresa, edite su acceso organizativo:

:::image type="content" source="media/github-enterprise-server/edit-runner-access.png" alt-text="Edición del acceso al ejecutor.":::

Aquí se va a poner a disposición de todas las organizaciones, pero puede limitar el acceso a un subconjunto de estas e incluso a repositorios específicos.

## <a name="optional-configure-github-connect"></a>(Opcional) Configuración de GitHub Connect

Aunque este paso es opcional, se recomienda si tiene previsto usar acciones de código abierto disponibles en GitHub.com. Le permite basarse en el trabajo de otras personas y usar estas acciones reutilizables como referencia en los flujos de trabajo.

Para habilitar GitHub Connect, siga los pasos de [Habilitación del acceso automático a acciones de GitHub.com mediante GitHub Connect](https://docs.github.com/en/enterprise/admin/github-actions/enabling-automatic-access-to-githubcom-actions-using-github-connect).

Una vez habilitado GitHub Connect, seleccione la opción **Server can use actions from GitHub.com in workflow runs** (El servidor puede usar acciones de GitHub.com en ejecuciones de flujo de trabajo).

:::image type="content" source="media/github-enterprise-server/enable-using-actions.png" alt-text="Habilitación del uso de acciones de GitHub.com en ejecuciones de flujo de trabajo.":::

## <a name="set-up-and-run-your-first-workflow"></a>Configuración y ejecución del primer flujo de trabajo

Ahora que Acciones y GitHub Connect están configurados, vamos a darle buen uso a todo este trabajo. Este es un flujo de trabajo de ejemplo que hace referencia a la excelente [octokit/request-action](https://github.com/octokit/request-action), lo que permite aplicar scripting a GitHub por medio de interacciones mediante la API de GitHub, con tecnología de Acciones de GitHub.

En este flujo de trabajo básico se va a usar `octokit/request-action` para abrir una incidencia en GitHub mediante la API.

:::image type="content" source="media/github-enterprise-server/workflow-example.png" alt-text="Flujo de trabajo de ejemplo.":::

>[!NOTE]
>GitHub.com hospeda la acción, pero cuando se ejecuta en el servidor de GitHub Enterprise, *automáticamente* usa la API del servidor de GitHub Enterprise.

Si decide no habilitar GitHub Connect, puede usar el siguiente flujo de trabajo alternativo.

:::image type="content" source="media/github-enterprise-server/workflow-example-2.png" alt-text="Ejemplo de flujo de trabajo alternativo.":::

Vaya a un repositorio de la instancia y agregue el flujo de trabajo anterior como: `.github/workflows/hello-world.yml`

:::image type="content" source="media/github-enterprise-server/workflow-example-3.png" alt-text="Otro flujo de trabajo de ejemplo.":::

En la pestaña **Acciones** del repositorio, espere a que se ejecute el flujo de trabajo.

:::image type="content" source="media/github-enterprise-server/executed-example-workflow.png" alt-text="Flujo de trabajo de ejemplo ejecutado.":::

También puede ver cómo lo está procesando el ejecutor.

:::image type="content" source="media/github-enterprise-server/workflow-processed-by-runner.png" alt-text="Flujo de trabajo procesado por el ejecutor.":::

Si todo se ha ejecutado correctamente, debería ver una nueva incidencia en el repositorio con el título "Hola mundo".

:::image type="content" source="media/github-enterprise-server/example-in-repo.png" alt-text="Ejemplo en el repositorio.":::

Enhorabuena. Acaba de completar el primer flujo de trabajo de Acciones en el servidor de GitHub Enterprise, que se ejecuta en la nube privada de Azure VMware Solution.

En este artículo se ha configurado una nueva instancia del servidor de GitHub Enterprise, el equivalente autohospedado de GitHub.com, en la nube privada de Azure VMware Solution. Esta instancia incluye compatibilidad con Acciones de GitHub y usa Azure Blob Storage para la persistencia de registros y artefactos. Sin embargo, esto es solo una pequeña muestra de lo que se puede hacer con Acciones de GitHub. Consulte la lista de Acciones en [Marketplace de GitHub](https://github.com/marketplace) o [cree las propias](https://docs.github.com/en/actions/creating-actions).

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha visto cómo configurar el servidor de GitHub Enterprise en la nube privada de Azure VMware Solution, quizás quiera aprender sobre lo siguiente: 

- [Introducción a Acciones de GitHub](https://docs.github.com/en/actions)
- [Unión al programa beta](https://resources.github.com/beta-signup/)
- [Administración del servidor de GitHub Enterprise](https://githubtraining.github.io/admin-training/#/00_getting_started).
