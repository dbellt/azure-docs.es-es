---
title: 'Inicio rápido: aprovisionamiento y activación de un HSM administrado de Azure'
description: Inicio rápido que muestra cómo aprovisionar y activar un HSM administrado mediante la CLI de Azure
services: key-vault
author: amitbapat
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: managed-hsm
ms.topic: quickstart
ms.date: 06/21/2021
ms.author: ambapat
ms.openlocfilehash: cbde53687c08208dc8fed3b69a6a98ed732cf57c
ms.sourcegitcommit: 351279883100285f935d3ca9562e9a99d3744cbd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/19/2021
ms.locfileid: "112380381"
---
# <a name="quickstart-provision-and-activate-a-managed-hsm-using-azure-cli"></a>Inicio rápido: Aprovisionamiento y activación de un HSM administrado mediante la CLI de Azure

Azure Key Vault Managed HSM es un servicio en la nube que cumple los estándares, totalmente administrado, de alta disponibilidad y de un solo inquilino, que le permite proteger las claves criptográficas de las aplicaciones en la nube mediante HSM validados de **FIPS 140-2 de nivel 3**. Para más información sobre Managed HSM, puede consultar esta [introducción](overview.md). 

En este inicio rápido, se crea y activa un HSM administrado con la CLI de Azure.

## <a name="prerequisites"></a>Requisitos previos

Para completar los pasos de este artículo, debe tener los siguientes elementos:

* Una suscripción a Microsoft Azure. Si no tiene una, puede registrarse para una [evaluación gratuita](https://azure.microsoft.com/pricing/free-trial).
* La CLI de Azure, versión 2.25.0 o posterior. Ejecute `az --version` para encontrar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure]( /cli/azure/install-azure-cli).
* Un HSM administrado en la suscripción. Consulte [Quickstart: Aprovisionamiento y activación de un HSM administrado mediante la CLI de Azure](quick-create-cli.md) para aprovisionar y activar un HSM administrado.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Para iniciar sesión en Azure mediante la CLI puede escribir:

```azurecli
az login
```

## <a name="create-a-resource-group"></a>Crear un grupo de recursos

Un grupo de recursos es un contenedor lógico en el que se implementan y se administran los recursos de Azure. En el ejemplo siguiente, se crea un grupo de recursos denominado *ContosoResourceGroup* en la ubicación *eastus2*.

```azurecli-interactive
az group create --name "ContosoResourceGroup" --location eastus2
```

## <a name="create-a-managed-hsm"></a>Creación de un HSM administrado

La creación de un HSM administrado es un proceso que se compone de dos pasos:
1. Aprovisionar un recurso de Managed HSM.
1. Activar el HSM administrado mediante la descarga del dominio de seguridad.

### <a name="provision-a-managed-hsm"></a>Aprovisionamiento de un HSM administrado

Use el comando `az keyvault create` para crear un HSM administrado. Este script tiene tres parámetros obligatorios: un nombre de grupo de recursos, un nombre de HSM y la ubicación geográfica.

Para crear un recurso de Managed HSM, es preciso que proporcione lo siguiente:
- Un grupo de recursos donde se colocará en su suscripción.
- Una ubicación de Azure.
- Una lista de administradores iniciales.

En el ejemplo siguiente se crea un HSM denominado **ContosoMHSM**, en el grupo de recursos **ContosoResourceGroup**, que reside en la ubicación **Este de EE. UU. 2**, con **el usuario actual con sesión iniciada** como único administrador y un **período de retención de 28 días** para la eliminación temporal. Más información sobre la [Eliminación temporal de HSM administrado](soft-delete-overview.md).

```azurecli-interactive
oid=$(az ad signed-in-user show --query objectId -o tsv)
az keyvault create --hsm-name "ContosoMHSM" --resource-group "ContosoResourceGroup" --location "East US 2" --administrators $oid --retention-days 28
```

> [!NOTE]
> La creación de un comando puede tardar unos minutos. Una vez que vuelva correctamente, está listo para activar el HSM.

La salida de este comando muestra las propiedades del HSM administrado que ha creado. Las dos propiedades más importantes son:

* **name**: en el ejemplo, el nombre es ContosoMHSM. Usará este nombre para otros comandos de Key Vault.
* **hsmUri**: en el ejemplo, el URI es "https://contosohsm.managedhsm.azure.net". Las aplicaciones que utilizan el HSM a través de su API REST deben usar este identificador URI.

Su cuenta de Azure ahora está autorizada para realizar operaciones en este HSM administrado. Hasta ahora, nadie más está autorizado.

### <a name="activate-your-managed-hsm"></a>Activación de un HSM administrado

Todos los comandos de plano de datos están deshabilitados hasta que se activa el HSM. No podrá crear claves ni asignar roles. El HSM solo pueden activarlo los administradores designados que se asignaron durante el comando create. Para activar el HSM, debe descargar el [dominio de seguridad](security-domain.md).

Para activar el HSM necesita:
- Tres pares de claves RSA como mínimo (diez como máximo).
- Especificar el número mínimo de claves necesarias para descifrar el dominio de seguridad (cuórum)

Para activar el HSM, envíe al menos tres claves públicas RSA (diez como máximo) al HSM. El HSM cifra el dominio de seguridad con estas claves y lo devuelve. Una vez que la descarga de este dominio de seguridad se haya completado correctamente, el HSM está listo para usarse. También debe especificar el cuórum, que es el número mínimo de claves privadas necesarias para descifrar el dominio de seguridad.

En el ejemplo siguiente se muestra cómo usar  `openssl` para generar tres certificados autofirmados.

```azurecli-interactive
openssl req -newkey rsa:2048 -nodes -keyout cert_0.key -x509 -days 365 -out cert_0.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_1.key -x509 -days 365 -out cert_1.cer
openssl req -newkey rsa:2048 -nodes -keyout cert_2.key -x509 -days 365 -out cert_2.cer
```

> [!IMPORTANT]
> Cree y almacene los pares de claves de RSA y el archivo de dominio de seguridad generados en este paso de forma segura.

Use el comando `az keyvault security-domain download` para descargar el dominio de seguridad y activar el HSM administrado. En el ejemplo siguiente, se usan tres pares de claves de RSA (para este comando solo se necesitan claves públicas) y se establece el cuórum en 2.

```azurecli-interactive
az keyvault security-domain download --hsm-name ContosoMHSM --sd-wrapping-keys ./certs/cert_0.cer ./certs/cert_1.cer ./certs/cert_2.cer --sd-quorum 2 --security-domain-file ContosoMHSM-SD.json
```

Almacene el archivo del dominio de seguridad y los pares de claves de RSA de forma segura. Los necesitará tanto para la recuperación ante desastres como para crear otro HSM administrado que comparta el mismo dominio de seguridad, con el fin de que puedan compartir claves.

Después de descargar correctamente el dominio de seguridad, el HSM estará en estado activo y listo para que lo use.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de esta colección se basan en los valores de esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.

Cuando ya no se necesiten, puede usar el comando [az group delete](/cli/azure/group) para quitar el grupo de recursos y todos los recursos relacionados. Puede eliminar los recursos como se indica a continuación:

```azurecli-interactive
az group delete --name ContosoResourceGroup
```

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido, ha creado una instancia de Key Vault y ha almacenado un secreto en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Managed HSM](overview.md)
- Obtenga información sobre la [administración de claves en un HSM administrado](key-management.md)
- Puede encontrar más información en [Administración de roles de Managed HSM](role-management.md).
- Consulte [Procedimientos recomendados de Managed HSM](best-practices.md)
