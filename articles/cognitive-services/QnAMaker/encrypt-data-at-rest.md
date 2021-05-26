---
title: Cifrado de datos en reposo de QnA Maker
titleSuffix: Azure Cognitive Services
description: Microsoft ofrece claves de cifrado administradas por Microsoft y también le permite administrar las suscripciones de Cognitive Services con sus propias claves, llamadas claves administradas por el cliente (CMK). En este artículo se trata el cifrado de datos en reposo de QnA Maker y cómo habilitar y administrar las CMK.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.author: egeaney
ms.openlocfilehash: 84f95e4c1d24ee100efbd0572dfe0dc88aca22d3
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369493"
---
# <a name="qna-maker-encryption-of-data-at-rest"></a>Cifrado de datos en reposo de QnA Maker

QnA Maker cifra automáticamente sus datos cuando se guardan en la nube, lo que ayuda a cumplir los objetivos de seguridad y cumplimiento normativo de la organización.

## <a name="about-encryption-key-management"></a>Información sobre la administración de claves de cifrado

De forma predeterminada, su suscripción usa claves de cifrado administradas por Microsoft. También puede administrar la suscripción con sus propias claves, que se denominan claves administradas por el cliente (CMK). Estas claves ofrecen mayor flexibilidad para crear, rotar, deshabilitar y revocar los controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos. Si la opción CMK está configurada en su suscripción, dispondrá de un cifrado doble, lo que constituye un segundo nivel de protección, y al mismo tiempo podrá controlar la clave de cifrado mediante Azure Key Vault.

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

QnA Maker usa la compatibilidad con CMK de Azure Search. Configure [CMK en Azure Search mediante Azure Key Vault](../../search/search-security-manage-encryption-keys.md). Esta instancia de Azure debe estar asociada con el servicio QnA Maker para que esté habilitada para CMK.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

La característica de respuesta a preguntas personalizada usa la [compatibilidad con CMK de Azure Search](../../search/search-security-manage-encryption-keys.md) y asocia automáticamente la clave administrada por el cliente proporcionada para cifrar los datos almacenados en el índice de Azure Search.

---

> [!IMPORTANT]
> El recurso de servicio Azure Search debe haberse creado después de enero de 2019 de enero y no puede estar en el nivel gratis (compartido). No se admite la configuración de claves administradas por el cliente en Azure Portal.

## <a name="enable-customer-managed-keys"></a>Habilitar claves administradas del cliente

El servicio QnA Maker usa CMK del servicio Azure Search. Siga estos pasos para habilitar CMK:

# <a name="qna-maker-ga-stable-release"></a>[Disponibilidad general de QnA Maker (versión estable)](#tab/v1)

1. Cree una nueva instancia de Azure Search y habilite los requisitos previos que se mencionan en los [requisitos previos de la clave administrada por el cliente de Azure Cognitive Search](../../search/search-security-manage-encryption-keys.md#prerequisites).

   ![Vista de opciones de cifrado 1](../media/cognitive-services-encryption/qna-encryption-1.png)

2. Cuando se crea un recurso de QnA Maker, se asocia automáticamente con una instancia de Azure Search. Esta instancia no se puede usar con la clave administrada por el cliente. Para usarlas, debe asociar la instancia recién creada de Azure Search que se creó en el paso 1. En concreto, deberá actualizar `AzureSearchAdminKey` y `AzureSearchName` en el recurso de QnA Maker.

   ![Vista de opciones de cifrado 2](../media/cognitive-services-encryption/qna-encryption-2.png)

3. A continuación, cree una nueva configuración de la aplicación:
   * **Name**: Establézcala en `CustomerManagedEncryptionKeyUrl`
   * **Valor**: Utilice el valor que ha recibido en el paso 1 al crear la instancia de Azure Search.

   ![Vista de opciones de cifrado 3](../media/cognitive-services-encryption/qna-encryption-3.png)

4. Cuando termine, reinicie el entorno de ejecución. Ahora el servicio QnA Maker está habilitado para CMK.

# <a name="custom-question-answering-preview-release"></a>[Respuesta a preguntas personalizada (versión preliminar)](#tab/v2)

1.  Vaya a la pestaña **Cifrado** del servicio Text Analytics con la característica de respuesta a preguntas personalizada (versión preliminar) habilitada.
2.  Seleccione la opción **Claves administradas de cliente**. Especifique los detalles de sus [claves administradas por el cliente](../../storage/common/customer-managed-keys-configure-key-vault.md?tabs=portal) y haga clic en **Guardar**.

> [!div class="mx-imgBorder"]
> ![Clave administrada por el cliente de respuesta a preguntas](media/question-answering-cmk.png)
   
3.  Si se guarda correctamente, la clave administrada por el cliente se usará para cifrar los datos almacenados en el índice de Azure Search.

> [!IMPORTANT]
> Se recomienda establecer la clave administrada por el cliente en un servicio Azure Cognitive Search nuevo antes de crear knowledge bases. Si establece la clave administrada por el cliente en un servicio de Text Analytics con knowledge bases existentes, puede perder el acceso a ellas. En Azure Cognitive Search puede encontrar más información sobre cómo [trabajar con contenido cifrado](../../search/search-security-manage-encryption-keys.md#work-with-encrypted-content).

> [!NOTE]
> Para solicitar la capacidad de usar claves administradas por el cliente, rellene y envíe el [formulario de solicitud de claves administradas por el cliente de Cognitive Services](https://aka.ms/cogsvc-cmk).

---

## <a name="regional-availability"></a>Disponibilidad regional

Las claves administradas por el cliente están disponibles en todas las regiones de Azure Search.

## <a name="encryption-of-data-in-transit"></a>Cifrado de datos en tránsito

El portal de QnA Maker se ejecuta en el explorador del usuario. Cada acción desencadena una llamada directa a la API del servicio cognitivo correspondiente. Por tanto, QnA Maker es compatible con los datos en tránsito.
Pero como el servicio del portal de QnA Maker se hospeda en el oeste de EE. UU., todavía no es idóneo para clientes de fuera de este país. 

## <a name="next-steps"></a>Pasos siguientes

* [Cifrado en Azure Search mediante CMK en Azure Key Vault](../../search/search-security-manage-encryption-keys.md)
* [Cifrado de datos en reposo](../../security/fundamentals/encryption-atrest.md)
* [Más información sobre Azure Key Vault](../../key-vault/general/overview.md)
