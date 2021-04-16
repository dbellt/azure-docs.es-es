---
title: No participar en las credenciales verificables de Azure Active Directory (versión preliminar)
description: Obtenga información sobre cómo no participar en la versión preliminar de las credenciales verificables.
documentationCenter: ''
author: barclayn
manager: daveba
ms.service: identity
ms.topic: how-to
ms.subservice: verifiable-credentials
ms.date: 04/01/2021
ms.author: barclayn
ms.openlocfilehash: d6e72b6d6f566fcf3f52e1c48ab6824c0e9a968e
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222799"
---
# <a name="opt-out-of-the-verifiable-credentials-preview"></a>No participación en las credenciales verificables (versión preliminar)

En este artículo:

- Motivo por el que es posible que tenga que dejar de participar.
- Pasos necesarios.
- ¿Qué ocurre con los datos?
- Efecto sobre las credenciales verificables existentes.

> [!IMPORTANT]
> Las credenciales verificables de Azure Active Directory se encuentran actualmente en versión preliminar pública.
> Esta versión preliminar se ofrece sin Acuerdo de Nivel de Servicio y no se recomienda para cargas de trabajo de producción. Es posible que algunas características no sean compatibles o que tengan sus funcionalidades limitadas. Para más información, consulte [Términos de uso complementarios de las Versiones Preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Requisitos previos

- Completar la incorporación de credenciales verificables.

## <a name="potential-reasons-for-opting-out"></a>Posibles motivos para no participar

En este momento, no tenemos la capacidad de realizar modificaciones en la información del dominio. Como resultado, si comete un error o decide que desea realizar un cambio, no hay ninguna otra opción disponible, aparte de no participar y volver a iniciar.

## <a name="the-steps-required"></a>Pasos necesarios

1. En Azure Portal busque credenciales verificables.
2. Elija **Configuración** en el menú de la izquierda.
3. En la sección, **Reset your organization** (Restablecer la organización), seleccione **Delete all credentials, and opt out of preview** (Eliminar todas las credenciales y no participar en la versión preliminar).

   ![Configuración de restablecer la organización](media/how-to-opt-out/settings-reset.png)

4. Lea el mensaje de advertencia y, para continuar, seleccione **Delete and opt out** (Eliminar y no participar).

   ![Configuración de eliminar y no participar](media/how-to-opt-out/delete-and-opt-out.png)

Ahora no participa en la versión preliminar de las credenciales verificables. Siga leyendo para comprender lo que subyace.

## <a name="what-happens-to-your-data"></a>¿Qué ocurre con los datos?

Cuando complete la operación de no participar en el servicio de credenciales verificables de Azure Active Directory, se realizarán las siguientes acciones:

- Las claves DID de Key Vault se [eliminarán temporalmente](../../key-vault/general/soft-delete-overview.md).
- El objeto emisor se eliminará de nuestra base de datos.
- El identificador del inquilino se eliminará de nuestra base de datos. 
- Todos los objetos de contratos se eliminarán de nuestra base de datos.

Una vez que se produce no participación, no podrá recuperar el DID ni realizar ninguna operación en su DID. Este paso es una operación unidireccional y debe volver a participar, lo que hace que se cree un nuevo DID.  

## <a name="effect-on-existing-verifiable-credentials"></a>Efecto sobre las credenciales verificables existentes

Todas las credenciales verificables ya emitidas seguirán existiendo. No se invalidarán criptográficamente, ya que el DID se seguirá pudiendo resolver a través de ION.
Sin embargo, cuando las partes de confianza llaman a la API de estado, siempre recibirán un mensaje de error.

## <a name="next-steps"></a>Pasos siguientes

- Configuración de credenciales verificables en el [inquilino de Azure](get-started-verifiable-credentials.md)