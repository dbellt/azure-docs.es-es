---
title: Uso de Azure Portal para configurar las claves administradas por el cliente
titleSuffix: Cognitive Services
description: Aprenda a usar Azure Portal para configurar las claves administradas por el cliente con Azure Key Vault. Las claves administradas por el cliente le permiten crear, rotar, deshabilitar y revocar los controles de acceso.
services: cognitive-services
author: erindormier
ms.service: cognitive-services
ms.topic: include
ms.date: 05/28/2020
ms.author: egeaney
ms.openlocfilehash: c79846b0a5b675c34e4e7919e9ecd9d591bfefe5
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96356365"
---
## <a name="customer-managed-keys-with-azure-key-vault"></a>Claves administradas por el cliente con Azure Key Vault

Debe usar Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar las API de Azure Key Vault para generarlas. El recurso de Cognitive Services y el almacén de claves deben estar en la misma región y en el mismo inquilino de Azure Active Directory (Azure AD), pero pueden estar en distintas suscripciones. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](../../key-vault/general/overview.md).

Cuando se crea un nuevo recurso de Cognitive Services, siempre se cifra mediante claves administradas por Microsoft. No es posible habilitar claves administradas por el cliente en el momento en que se crea el recurso. Las claves administradas por el cliente se almacenan en Azure Key Vault y el almacén de claves se debe aprovisionar con directivas de acceso que concedan permisos de clave a la identidad administrada que está asociada al recurso de Cognitive Services. La identidad administrada solo está disponible después de crear el recurso con el plan de tarifa requerido para CMK.

Al habilitar las claves administradas por el cliente, también se habilitará una [identidad administrada](../../active-directory/managed-identities-azure-resources/overview.md) asignada por el sistema, una característica de Azure AD. Una vez habilitada la identidad administrada asignada por el sistema, este recurso se registrará con Azure Active Directory. Tras su registro, se concederá a la identidad administrada acceso a la instancia de Key Vault seleccionada durante la configuración de la clave administrada por el cliente. 

> [!IMPORTANT]
> Si deshabilita las identidades administradas asignadas por el sistema, se quitará el acceso al almacén de claves y los datos cifrados con las claves de cliente dejarán de estar disponibles. Las características dependientes de estos datos dejarán de funcionar.

> [!IMPORTANT]
> Las identidades administradas no admiten actualmente escenarios entre directorios. Al configurar las claves administradas por el cliente en Azure Portal, se asigna automáticamente una identidad administrada en segundo plano. Si posteriormente mueve la suscripción, el grupo de recursos o el recurso de un directorio de Azure AD a otro, la identidad administrada asociada al recurso no se transfiere al nuevo inquilino, por lo que es posible que las claves administradas por el cliente dejen de funcionar. Para más información, vea **Transferencia de una suscripción entre directorios de Azure AD** en [Preguntas frecuentes y problemas conocidos con identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/known-issues.md#transferring-a-subscription-between-azure-ad-directories).  

## <a name="configure-azure-key-vault"></a>Configuración de Azure Key Vault

Para usar las claves administradas por el cliente, es necesario establecer dos propiedades en el almacén de claves, **Eliminación temporal** y **No purgar**. Estas propiedades no están habilitadas de forma predeterminada, pero se pueden habilitar mediante PowerShell o la CLI de Azure tanto en un almacén de claves nuevo como en uno existente.

> [!IMPORTANT]
> Si no tiene habilitadas las propiedades **Eliminación temporal** y **No purgar** y elimina la clave, no podrá recuperar los datos en el recurso de Cognitive Services.

Para aprender a habilitar estas propiedades en un almacén de claves existente, consulte las secciones **Habilitación de la eliminación temporal** y **Habilitación de la protección de purgas** en cualquiera de los siguientes artículos:

- [Uso de la eliminación temporal con PowerShell](../../key-vault/general/key-vault-recovery.md).
- [Uso de la eliminación temporal con la CLI](../../key-vault/general/key-vault-recovery.md).

Las claves RSA de tamaño 2048 son las únicas que admite el cifrado de Azure Storage. Para más información acerca de las claves, consulte la sección **Claves en Key Vault** en [Información acerca de claves, secretos y certificados de Azure Key Vault](../../key-vault/general/about-keys-secrets-certificates.md).

## <a name="enable-customer-managed-keys-for-your-resource"></a>Habilitación de claves administradas por el cliente para el recurso

Para habilitar claves administradas del cliente en Azure Portal, siga estos pasos:

1. Vaya al recurso de Cognitive Services.
1. En la hoja **Configuración** del recurso de Cognitive Services, haga clic en **Cifrado**. Seleccione la opción **Claves administradas de cliente**, como se muestra en la ilustración siguiente.

    ![Captura de pantalla que muestra cómo seleccionar Claves administradas por el cliente](../media/cognitive-services-encryption/selectcmk.png)

## <a name="specify-a-key"></a>Especificar una clave

Después de habilitar las claves administradas por el cliente, tendrá la oportunidad de especificar una clave para asociarla con el recurso de Cognitive Services.

### <a name="specify-a-key-as-a-uri"></a>Especificación de una clave como URI

Para especificar una clave como URI, siga estos pasos:

1. Para buscar el URI de la clave en Azure Portal, vaya al almacén de claves y seleccione la opción de configuración **Claves**. Seleccione la clave que desee y luego haga clic en ella para ver sus versiones. Seleccione cualquiera de las versiones de clave para ver su configuración.
1. Copie el valor del campo **Identificador de clave**, que proporciona el URI.

    ![Captura de pantalla en que se muestra el URI de la clave del almacén de claves](../media/cognitive-services-encryption/key-uri-portal.png)

1. En las opciones de configuración de **cifrado** de la cuenta de almacenamiento, elija la opción **Introducir URI de la clave**.
1. Pegue el identificador URI que ha copiado en el campo **URI de clave**.

   ![Captura de pantalla en que se muestra cómo introducir el URI de la clave](../media/cognitive-services-encryption/ssecmk2.png)

1. Especifique el identificador de la suscripción que contiene el almacén de claves.
1. Guarde los cambios.

### <a name="specify-a-key-from-a-key-vault"></a>Especificación de una clave a partir de un almacén de claves

Para especificar una clave a partir de un almacén de claves, asegúrese de tener un almacén de claves que contenga una clave. Para especificar una clave a partir de un almacén de claves, siga estos pasos:

1. Elija la opción **Select from Key Vault** (Seleccionar desde almacén de claves).
1. Seleccione el almacén de claves que contiene la clave que desea usar.
1. Seleccione la clave en el almacén de claves.

   ![Captura de pantalla en que se muestra la opción de clave administrada del cliente](../media/cognitive-services-encryption/ssecmk3.png)

1. Guarde los cambios.

## <a name="update-the-key-version"></a>Actualización de la versión de la clave

Al crear una versión de una clave, actualice el recurso de Cognitive Services para usar la nueva versión. Siga estos pasos:

1. Vaya al recurso de Cognitive Services y muestre la configuración de **Cifrado**.
1. Escriba el identificador URI de la nueva versión de la clave. Como alternativa, puede volver a seleccionar el almacén de claves y la clave para actualizar la versión.
1. Guarde los cambios.

## <a name="use-a-different-key"></a>Uso de una clave distinta

Para cambiar la clave que se usa para el cifrado, siga estos pasos:

1. Vaya al recurso de Cognitive Services y muestre la configuración de **Cifrado**.
1. Escriba el identificador URI de la nueva clave. Otra opción es seleccionar el almacén de claves y elegir una clave nueva.
1. Guarde los cambios.

## <a name="rotate-customer-managed-keys"></a>Rotación de claves administradas por el cliente

Las claves administradas por el cliente se pueden rotar en Azure Key Vault según las directivas de cumplimiento. Si se rota la clave, es preciso actualizar el recurso de Cognitive Services para que use el identificador URI de la clave nueva. Para obtener información sobre cómo actualizar el recurso para usar una nueva versión de la clave en Azure Portal, consulte la sección [Actualización de la versión de la clave](#update-the-key-version).

La rotación de la clave no desencadena un nuevo cifrado de los datos del recurso. No es preciso que el usuario realice ninguna otra acción.

## <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Para revocar el acceso a las claves administradas por el cliente, use PowerShell o la CLI de Azure. Para más información, consulte la referencia de [PowerShell para Azure Key Vault](/powershell/module/az.keyvault//) o la referencia de la [CLI para Azure Key Vault](/cli/azure/keyvault). La revocación del acceso bloquea de manera eficaz el acceso a todos los datos del recurso de Cognitive Services, ya que el servicio no puede acceder a la clave de cifrado.

## <a name="disable-customer-managed-keys"></a>Deshabilitación de claves administradas por el cliente

Cuando se deshabilitan las claves administradas por el cliente, el recurso de Cognitive Services se cifra entonces con claves administradas por Microsoft. Para deshabilitar las claves administradas por el cliente, siga estos pasos:

1. Vaya al recurso de Cognitive Services y muestre la configuración de **Cifrado**.
1. Anule la selección de la casilla que se encuentra junto al valor **Usar su propia clave**.