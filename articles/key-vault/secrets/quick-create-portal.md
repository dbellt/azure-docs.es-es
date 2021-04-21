---
title: 'Inicio rápido de Azure: Establecimiento y recuperación de un secreto de Key Vault mediante Azure Portal | Microsoft Docs'
description: Guía de inicio rápido que muestra cómo establecer y recuperar un secreto de Azure Key Vault mediante Azure Portal
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc
ms.date: 09/03/2019
ms.author: mbaldwin
ms.openlocfilehash: ba5b375f75d2655045e62583679839c8dd2a5720
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107752451"
---
# <a name="quickstart-set-and-retrieve-a-secret-from-azure-key-vault-using-the-azure-portal"></a>Inicio rápido: Establecimiento y recuperación de un secreto de Azure Key Vault mediante Azure Portal

Azure Key Vault es un servicio de almacenamiento seguro de secretos en la nube. Puede almacenar de forma segura claves, contraseñas, certificados y otros secretos. Las instancias de Azure Key Vault se pueden crear y administrar a través de Azure Portal. En esta guía de inicio rápido se crea un almacén de claves y se usa para almacenar un secreto. 

Para más información, consulte 
- [Introducción a Azure Key Vault](../general/overview.md)
- [Información general de los secretos](about-secrets.md).

## <a name="prerequisites"></a>Requisitos previos

Para acceder a Azure Key Vault, necesitará una suscripción de Azure. Si todavía no tiene una suscripción, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

Todo acceso a los secretos tiene lugar a través de Azure Key Vault. Para este inicio rápido, cree un almacén de claves mediante [Azure Portal](../general/quick-create-portal.md), la [CLI de Azure](../general/quick-create-cli.md) o [Azure PowerShell](../general/quick-create-powershell.md).

## <a name="sign-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="add-a-secret-to-key-vault"></a>Incorporación de un secreto a Key Vault

Para agregar un secreto al almacén, siga estos pasos:

1. Vaya al nuevo almacén de claves en Azure Portal.
1. En las páginas de configuración de Key Vault, seleccione **Secretos**.
1. Haga clic en **Generar o Importar**.
1. En la pantalla **Crear un secreto**, elija los siguientes valores:
    - **Opciones de carga**: Manual.
    - **Name**: Escriba un nombre para el secreto. El nombre del secreto debe ser único en Key Vault. El nombre debe ser una cadena de entre 1 y 127 caracteres que solo contenga 0-9, a-z, A-Z y -. Para más información sobre la nomenclatura, consulte [Objetos, identificadores y control de versiones de Key Vault](../general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning).
    - **Valor**: Escriba un valor para el secreto. Las API de Key Vault aceptan y devuelven los valores de secreto como cadenas. 
    - Deje las restantes opciones con sus valores predeterminados. Haga clic en **Crear**.

Una vez recibido el mensaje de que el secreto se ha creado correctamente, puede hacer clic en él en la lista. 

Para más información sobre los atributos de secretos, consulte [Acerca de los secretos de Azure Key Vault](./about-secrets.md).

## <a name="retrieve-a-secret-from-key-vault"></a>Recuperar un secreto del almacén de claves

Si hace clic en la versión actual puede ver el valor especificado en el paso anterior.

![Propiedades del secreto](../media/quick-create-portal/current-version-hidden.png)

Si hace clic en el botón "Mostrar valor secreto" en el panel de la derecha, puede usar el valor oculto. 

![Valor secreto visible](../media/quick-create-portal/current-version-shown.png)

También puede usar la [CLI de Azure]() o [Azure PowerShell]() para recuperar el secreto creado previamente.

## <a name="clean-up-resources"></a>Limpieza de recursos

Otras guías de inicio rápido y tutoriales de Key Vault se basan en esta. Si tiene pensado seguir trabajando en otras guías de inicio rápido y tutoriales, considere la posibilidad de dejar estos recursos activos.
Cuando ya no lo necesite, elimine el grupo de recursos; de este modo se eliminarán también Key Vault y los recursos relacionados. Para eliminar el grupo de recursos mediante el portal:

1. Escriba el nombre del grupo de recursos en el cuadro de búsqueda de la parte superior del portal. Cuando vea el grupo de recursos que se utiliza en esta guía de inicio rápido en los resultados de búsqueda, selecciónelo.
2. Seleccione **Eliminar grupo de recursos**.
3. En el cuadro **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** escriba el nombre del grupo de recursos y seleccione **Eliminar**.

> [!NOTE]
> Es importante tener en cuenta que una vez que se elimina un secreto, una clave, un certificado o un almacén de claves, no se podrá recuperar durante un período configurable de 7 a 90 días naturales. Si no se especifica ninguna configuración, el período de recuperación predeterminado se establecerá en 90 días. De esta forma, los usuarios tendrán tiempo suficiente para darse cuenta de la eliminación accidental de un secreto y responder a ella. Para más información sobre cómo eliminar y recuperar los almacenes de claves y los objetos de Key Vault, consulte [Información general sobre la eliminación temporal de Azure Key Vault](../general/soft-delete-overview.md).

## <a name="next-steps"></a>Pasos siguientes

En este inicio rápido ha creado una instancia de Key Vault y ha almacenado un secreto en ella. Para más información sobre Key Vault y cómo integrarlo con las aplicaciones, continúe con los artículos siguientes.

- Lea una [introducción a Azure Key Vault](../general/overview.md).
- Lea [Protección del acceso a un almacén de claves](../general/security-overview.md)
- [Uso de Key Vault con una aplicación web de App Service](../general/tutorial-net-create-vault-azure-web-app.md)
- Consulte [Uso de Key Vault con una aplicación implementada en una máquina virtual](../general/tutorial-net-virtual-machine.md)
- Consulte la [guía del desarrollador de Azure Key Vault](../general/developers-guide.md).
- Consulte [Introducción a la seguridad de Azure Key Vault](../general/security-overview.md)