---
title: Autenticación de nodos de Azure Confidential Ledger
description: Autenticación de nodos de Azure Confidential Ledger
services: confidential-ledger
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: overview
ms.date: 04/15/2021
ms.author: mbaldwin
ms.openlocfilehash: 0a8f033aa540d2a469141c4c529981a0cbeef3ec
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386658"
---
# <a name="authenticating-azure-confidential-ledger-nodes"></a>Autenticación de nodos de Azure Confidential Ledger

Los nodos de Azure Confidential Ledger se pueden autenticar mediante ejemplos de código y usuarios.

## <a name="code-samples"></a>Ejemplos de código

Al realizar la reinicialización, los ejemplos de código obtienen el certificado de nodo consultando el servicio de identidad. Una vez recuperado el certificado de nodo, un ejemplo de código consultará la red de Ledger para obtener una cita, que luego se valida mediante los archivos binarios de comprobación de host. Si la comprobación se realiza correctamente, el ejemplo de código continúa con las operaciones de Ledger.

## <a name="users"></a>Usuarios

Los usuarios pueden validar la autenticidad de los nodos de Confidential Ledger para confirmar que realmente están interfiriendo con el enclave de su instancia de Ledger. Puede generar confianza en los nodos de Confidential Ledger de varias maneras, que se pueden apilar entre sí para aumentar el nivel general de confianza. Por lo tanto, los pasos 1 a 2 ayudan a generar confianza en ese enclave de Confidential Ledger como parte del protocolo de enlace TLS inicial y la autenticación dentro de flujos de trabajo funcionales. Más allá de eso, se mantiene una conexión de cliente persistente entre Confidential Ledger y el cliente del usuario.

- **Validar el nodo de Confidential Ledger**: esto se logra consultando el servicio de identidad hospedado por Microsoft, que proporciona un certificado de red y, por tanto, ayuda a comprobar que el nodo Ledger presenta un certificado aprobado o firmado por el certificado de red para esa instancia específica. De forma similar a HTTPS basado en PKI, el certificado de un servidor está firmado por una entidad de certificación (CA) conocida o una entidad de certificación intermedia. En el caso de Confidential Ledger, el servicio de identidad devuelve el certificado de la entidad de certificación en forma de un certificado de red. Se trata de una medida importante de creación de confianza para los usuarios de Confidential Ledger. Si este certificado de nodo no está firmado por el certificado de red devuelto, se producirá un error en la conexión de cliente (como se implementa en el código de ejemplo).
- **Validar el enclave de Confidential Ledger**: Confidential Ledger se ejecuta en un enclave de Intel® SGX representado por una Cita, un blob de datos generado dentro de ese enclave. Cualquier otra entidad puede usarlo para comprobar que la cita se ha producido a partir de una aplicación que se ejecuta con las protecciones de Intel® SGX. La cita está estructurada de forma que permite una comprobación sencilla. Contiene notificaciones que ayudan a identificar varias propiedades del enclave y la aplicación que está ejecutando. Se trata de un mecanismo de creación de confianza importante para los usuarios de Confidential Ledger. Esto se puede lograr mediante una llamada a una API de flujo de trabajo funcional para obtener una cita de enclave. La conexión de cliente debe producir un error si la cita no es válida. La cita recuperada se puede validar con la herramienta open_enclaves Host_Verify. Aquí puede encontrar más detalles al respecto.

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Microsoft Azure Confidential Ledger](overview.md)
- [Arquitectura de Azure Confidential Ledger](architecture.md)
