---
title: Uso de secretos de Azure Key Vault en actividades de canalización
description: Obtenga información sobre cómo capturar credenciales almacenadas desde Azure Key Vault y usarlas durante las ejecuciones de canalización de factoría de datos.
author: nabhishek
ms.author: abnarain
ms.service: data-factory
ms.topic: conceptual
ms.date: 10/31/2019
ms.openlocfilehash: b6e3ac551c2660cef7437ee069d3c8e4846cbd00
ms.sourcegitcommit: b4032c9266effb0bf7eb87379f011c36d7340c2d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107906613"
---
# <a name="use-azure-key-vault-secrets-in-pipeline-activities"></a>Uso de secretos de Azure Key Vault en actividades de canalización

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Puede almacenar credenciales o valores de secreto en una instancia de Azure Key Vault y usarlos durante la ejecución de la canalización para pasarlos a las actividades.

## <a name="prerequisites"></a>Prerequisites

Esta característica se basa en la identidad administrada de Data Factory.  Obtenga información sobre cómo funciona en [Identidad administrada de Data Factory](./data-factory-service-identity.md) y asegúrese de que la instancia de Data Factory tenga una asociada.

## <a name="steps"></a>Pasos

1. Abra las propiedades de la factoría de datos y copie el valor Id. de aplicación de identidad administrada.

    ![Valor de identidad administrada](media/how-to-use-azure-key-vault-secrets-pipeline-activities/managedidentity.png)

2. Abra las directivas de acceso del almacén de claves y agregue los permisos de identidad administrada para obtener y enumerar los secretos.

    ![Captura de pantalla que muestra la página "Directivas de acceso" con la acción "Agregar directiva de acceso" resaltada.](media/how-to-use-azure-key-vault-secrets-pipeline-activities/akvaccesspolicies.png)

    ![Directivas de acceso de Key Vault](media/how-to-use-azure-key-vault-secrets-pipeline-activities/akvaccesspolicies-2.png)

    Haga clic en **Agregar** y luego en **Guardar**.

3. Navegue hasta el secreto de Key Vault y copie el identificador secreto.

    ![Identificador secreto](media/how-to-use-azure-key-vault-secrets-pipeline-activities/secretidentifier.png)

    Anote el URI del secreto que quiera obtener durante la ejecución de la canalización de factoría de datos.

4. En la canalización de Data Factory, agregue una nueva actividad web y configúrela como se indica a continuación.  

    |Propiedad  |Value  |
    |---------|---------|
    |Salida segura     |True         |
    |URL     |[El valor de URI del secreto]?api-version=7.0         |
    |Método     |GET         |
    |Authentication     |MSI         |
    |Resource        |https://vault.azure.net       |

    ![Actividad web](media/how-to-use-azure-key-vault-secrets-pipeline-activities/webactivity.png)

    > [!IMPORTANT]
    > Debe agregar **?api-version=7.0** al final del URI del secreto.  

    > [!CAUTION]
    > Establezca la opción Salida segura en true para evitar que el valor de secreto se registre en texto sin formato.  Todas las actividades adicionales que consuman este valor deben tener la opción Entrada segura establecida en true.

5. Para usar el valor en otra actividad, utilice la expresión de código siguiente **@activity("Web1").output.value**.

    ![Expresión de código](media/how-to-use-azure-key-vault-secrets-pipeline-activities/usewebactivity.png)

## <a name="next-steps"></a>Pasos siguientes

Para obtener información sobre cómo usar Azure Key Vault para almacenar credenciales para almacenes de datos y procesos, vea [Almacenamiento de credenciales en Azure Key Vault](./store-credentials-in-key-vault.md).