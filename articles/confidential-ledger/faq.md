---
title: Preguntas frecuentes de Azure Confidential Ledger
description: Preguntas frecuentes de Azure Confidential Ledger
services: confidential-ledger
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: overview
ms.date: 04/15/2021
ms.author: mbaldwin
ms.openlocfilehash: 1baa1470bcaef8e447b19eb37580b21b94c02df6
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386640"
---
# <a name="frequently-asked-questions-for-azure-confidential-ledger"></a>Preguntas frecuentes de Azure Confidential Ledger

## <a name="how-can-i-tell-if-the-acc-ledger-service-would-be-useful-to-my-organization"></a>¿Cómo puedo saber si el servicio Ledger de ACC sería útil para mi organización?

Confidential Ledger es ideal para organizaciones con registros lo suficientemente valiosos como para que un atacante motivado intente poner en peligro el sistema subyacente de registro o almacenamiento, incluidos los escenarios de "trabajos internos" en los que un empleado no autorizado podría intentar falsificar, modificar o quitar registros anteriores.

## <a name="what-makes-acc-ledger-much-more-secure"></a>¿Qué hace que Ledger de ACC sea mucho más seguro?

Como su nombre lo sugiere, Ledger usa la [plataforma de Computación confidencial de Azure](../confidential-computing/index.yml). Una instancia de Ledger abarca tres o más instancias idénticas, donde cada una se ejecuta en un enclave con respaldo de hardware dedicado y totalmente atestiguado. La integridad de Ledger se mantiene a través de una cadena de bloques basada en consenso.

## <a name="when-writing-to-the-acc-ledger-do-i-need-to-store-write-receipts"></a>Al escribir en Ledger de ACC, ¿es necesario almacenar recibos de escritura?

No necesariamente. En la actualidad, algunas soluciones requieren que los usuarios mantengan recibos de escritura para futuras validaciones de registros. Esto exige que los usuarios administren esos recibos en una instalación de almacenamiento seguro, lo que agrega una carga adicional. Ledger elimina este desafío mediante un enfoque basado en el árbol de Merkle, donde los recibos de escritura incluyen una ruta de acceso de árbol completa a una raíz de confianza firmada. Los usuarios pueden comprobar las transacciones sin almacenar ni administrar los datos de Ledger.

## <a name="how-do-i-verify-ledgers-authenticity"></a>¿Cómo compruebo la autenticidad de Ledger?

Puede comprobar que los nodos del servidor de Ledger con los que se comunica el cliente sean auténticos. Para obtener más información, consulte [Autenticación de nodos de Confidential Ledger](authenticate-ledger-nodes.md).



## <a name="next-steps"></a>Pasos siguientes

- [Información general de Microsoft Azure Confidential Ledger](overview.md)