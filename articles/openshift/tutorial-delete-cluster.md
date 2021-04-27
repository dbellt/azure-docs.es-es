---
title: 'Tutorial: Eliminación de un clúster de Red Hat OpenShift en Azure'
description: En este tutorial, aprenderá a eliminar un clúster de Red Hat OpenShift en Azure mediante la CLI de Azure
author: sakthi-vetrivel
ms.custom: fasttrack-edit, devx-track-azurecli
ms.author: suvetriv
ms.topic: tutorial
ms.service: azure-redhat-openshift
ms.date: 04/24/2020
ms.openlocfilehash: 65bb27f1f85b7a26e35074da84cfc27b2a5761a1
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484768"
---
# <a name="tutorial-delete-an-azure-red-hat-openshift-4-cluster"></a>Tutorial: Eliminación de un clúster de la versión 4 de Red Hat OpenShift en Azure

En este tutorial, la tercera parte de tres, se elimina un clúster de Red Hat OpenShift en Azure que ejecuta OpenShift 4. Aprenderá a:

> [!div class="checklist"]
> * Eliminación de un clúster de Red Hat OpenShift en Azure


## <a name="before-you-begin"></a>Antes de comenzar

En los tutoriales anteriores, se creó un clúster de Red Hat OpenShift en Azure y se estableció una conexión a él mediante la consola web de OpenShift. Si no ha realizado estos pasos, pero desea continuar, comience con el [Tutorial 1: Creación de un clúster de la versión 4 de Red Hat Openshift en Azure](tutorial-create-cluster.md).

Si decide instalar y usar la CLI de forma local, en este tutorial es preciso que ejecute la CLI de Azure, versión 2.6.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, vea [Instalación de la CLI de Azure](/cli/azure/install-azure-cli).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Si ejecuta la CLI de Azure de manera local, ejecute `az login` para iniciar sesión en Azure.

```bash
az login
```

Si tiene acceso a varias suscripciones, ejecute `az account set -s {subscription ID}` y reemplace `{subscription ID}` por la suscripción que quiere usar.

## <a name="delete-the-cluster"></a>Eliminación del clúster

En los tutoriales anteriores, se establecieron las siguientes variables.

```bash
CLUSTER=yourclustername
RESOURCEGROUP=yourresourcegroup
```

Con estos valores, elimine el clúster:

```bash
az aro delete --resource-group $RESOURCEGROUP --name $CLUSTER
```

Después, se le pedirá que confirme si desea eliminar el clúster. Después de confirmar con `y`, la eliminación del clúster tardará varios minutos. Cuando finalice el comando, se eliminará todo el grupo de recursos y los recursos dentro de él (incluido el clúster).

## <a name="next-steps"></a>Pasos siguientes

En esta parte del tutorial, ha aprendido a:
> [!div class="checklist"]
> * Eliminación de un clúster de Red Hat OpenShift en Azure 4

Más información sobre cómo usar OpenShift con la [documentación oficial de Red Hat OpenShift](https://docs.openshift.com/container-platform/4.6/welcome/index.html)
