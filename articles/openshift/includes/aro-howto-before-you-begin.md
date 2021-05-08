---
author: sabbour
ms.author: asabbour
ms.date: 4/5/2020
ms.openlocfilehash: d888d4df3f578bf88f8f38cb979ed70f681c64e8
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108157633"
---
### <a name="create-the-cluster"></a>Creación del clúster

Siga el tutorial para [crear un clúster de Red Hat OpenShift en Azure](../tutorial-create-cluster.md). Si decide instalar y usar la CLI localmente, en este tutorial es preciso que ejecute la versión 2.6.0 u otra posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

### <a name="connect-to-the-cluster"></a>Conectarse al clúster

Para administrar un clúster de Red Hat OpenShift en Azure, use [oc](https://docs.openshift.com/container-platform/4.7/cli_reference/openshift_cli/getting-started-cli.html), el cliente de línea de comandos de OpenShift.

> [!NOTE]
> Se recomienda [instalar la línea de comandos de OpenShift](../tutorial-connect-cluster.md) en [Azure Cloud Shell](https://shell.azure.com/) y usarla para todas las operaciones de línea de comandos siguientes. Inicie Shell desde shell.azure.com o haga clic en el vínculo:
>
> [![Insertar inicio](/azure/includes/media/cloud-shell-try-it/hdi-launch-cloud-shell.png "Inicio de Azure Cloud Shell")](https://shell.azure.com/bash)

Siga el tutorial para instalar la CLI, recuperar las credenciales del clúster y [conectarse al clúster](../tutorial-connect-cluster.md) mediante la consola web y la CLI de OpenShift.

Una vez que haya iniciado sesión, debería ver un mensaje que indica que está usando el proyecto `default`.

```output
Login successful.

You have access to 61 projects, the list has been suppressed. You can list all projects with 'oc projects'

Using project "default".
```