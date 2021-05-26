---
title: Información general de Microsoft Azure Confidential Ledger
description: Información general de Azure Confidential Ledger, un servicio muy seguro para administrar registros de datos confidenciales.
services: confidential-ledger
author: msmbaldwin
ms.service: confidential-ledger
ms.topic: overview
ms.date: 04/15/2021
ms.author: mbaldwin
ms.openlocfilehash: 7ea55c3f21664d504366657a653b5e3598666710
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110388491"
---
# <a name="microsoft-azure-confidential-ledger-preview"></a>Microsoft Azure Confidential Ledger (versión preliminar)

Microsoft Azure Confidential Ledger (ACL) o, para abreviar, Confidential Ledger (versión preliminar) es un servicio nuevo y altamente seguro para administrar registros de datos confidenciales. En función de un modelo de cadena de bloques con permisos, Confidential Ledger ofrece ventajas únicas para la integridad de datos. Esto incluye inmutabilidad, hacer que el libro de contabilidad solo se pueda anexar, y la protección frente a manipulaciones, para asegurar que todos los registros se mantengan intactos.

Confidential Ledger se ejecuta exclusivamente en enclaves seguros con respaldo de hardware, un entorno en tiempo de ejecución muy supervisado y aislado que mantiene a raya los posibles ataques. Además, nadie está "por encima" de Ledger, ni siquiera Microsoft. Al diseñar manteniéndonos fuera de la solución, el Confidential Ledger se ejecuta en una base de computación de confianza (TCB) minimalista que impide el acceso a los desarrolladores del servicio Ledger, los técnicos del centro de datos y los administradores de la nube.

Confidential Ledger recurre a los casos de uso en los que no se deben modificar los registros de metadatos críticos, incluso a perpetuidad, con fines de cumplimiento normativo y archivo. Estos son algunos ejemplos de cosas que puede almacenar en Ledger:

- Registros relacionados con las transacciones empresariales (por ejemplo, transferencias de dinero o ediciones a documentos confidenciales).
- Actualizaciones de recursos de confianza (por ejemplo, aplicaciones principales o contratos).
- Cambios administrativos y de control (por ejemplo, concesiones de permisos de acceso).
- Eventos operativos de TI y seguridad (por ejemplo, alertas de Azure Security Center).

Para obtener más información, puede ver la [demostración de Confidential Ledger en Microsoft Ignite 2020](https://mediusprodstatic.studios.ms/asset-b88de19d-4187-40c4-98f2-a65efc419e2a/OD221_1920x1080_AACAudio_1461.mp4?sv=2018-03-28&sr=b&sig=k5roi6WXnlqK1zP0fs5KYlJd4FD3Nuaf97z%2B2gV0aTs%3D&st=2020-09-22T08%3A05%3A01Z&se=2025-09-22T08%3A10%3A01Z&sp=r&rscd=filename%3DIG20-OD221-Inside%2BAzure%2BDatacenter%2BArchitecture%2Bwith%2BMark%2BRu.mp4).

## <a name="key-features"></a>Principales características

Confidential Ledger se expone a través de las API REST que se pueden integrar en aplicaciones nuevas o existentes. Los administradores pueden administrar Confidential Ledger mediante API administrativas (plano de control). También se puede llamar directamente mediante código de aplicación a través de las API funcionales (plano de datos). Las API administrativas admiten operaciones básicas, como crear, actualizar, obtener y eliminar. Las API funcionales permiten la interacción directa con Ledger con instancias, e incluyen operaciones como colocar y obtener datos.

## <a name="ledger-security"></a>Seguridad de Ledger

En esta sección se definen las protecciones de seguridad para Ledger. Las API de Ledger usan la autenticación basada en certificados de cliente. Actualmente, Ledger admite el proceso de autenticación basado en certificados con roles de propietario. Vamos a agregar compatibilidad con la autenticación basada en Azure Active Directory (AAD) y también el acceso basado en roles (por ejemplo, propietario, lector y colaborador).

Los datos para Ledger se envían a través de la conexión TLS 1.2, y la conexión TLS 1.2 finaliza dentro de los enclaves de seguridad con respaldo de hardware (enclaves Intel® SGX). Esto garantiza que nadie pueda interceptar la conexión entre el cliente de un cliente y los nodos de servidor de Confidential Ledger.

### <a name="ledger-storage"></a>Almacenamiento de Ledger

Las instancias de Confidential Ledger se crean como bloques en contenedores de almacenamiento en blobs que pertenecen a una cuenta de Azure Storage. Los datos de transacción se pueden almacenar cifrados o en texto no cifrado en función de sus necesidades. Al crear una instancia de Ledger, asociará una cuenta de almacenamiento siguiendo los pasos descritos en [Registro de una entidad de servicio de Confidential Ledger](register-ledger-service-principal.md).

Los administradores pueden administrar Confidential Ledger mediante API administrativas (plano de control), y se puede llamar directamente mediante código de aplicación a través de las API funcionales (plano de datos). Las API administrativas admiten operaciones básicas, como crear, actualizar, obtener y eliminar.

Las API funcionales permiten la interacción directa con Confidential Ledger con instancias, e incluyen operaciones como colocar y obtener datos.

## <a name="preview-limitations"></a>Limitaciones de versión preliminar

- Una vez creada una instancia de Confidential Ledger, no puede cambiar el tipo de libro de contabilidad.
- En este momento, Confidential Ledger no admite la recuperación ante desastres de Azure estándar. Sin embargo, Azure Confidential Ledger ofrece redundancia integrada dentro de la región de Azure, ya que Confidential Ledger se ejecuta en varios nodos independientes.
- La eliminación de Confidential Ledger conduce a una "eliminación permanente", por lo que los datos no se podrán recuperar después de la eliminación.
- Los nombres de Confidential Ledger deben ser únicos globalmente. No se permiten libros de contabilidad con el mismo nombre, independientemente de su tipo.

## <a name="terminology"></a>Terminología

| Término | Definición |
|--|--|
| ACL | Azure Confidential Ledger |
| Libro de contabilidad | Un registro de anexo inmutable de transacciones (también conocido como cadena de bloques) |
| Commit | Confirmación de que una transacción se ha confirmado localmente en un nodo. Una confirmación local por sí misma no garantiza que una transacción forme parte de Ledger. |
| Confirmación global | Confirmación de que la transacción se ha confirmado globalmente y forma parte de Ledger. |
| Recibo | Prueba de que Ledger procesó la transacción. |

## <a name="next-steps"></a>Pasos siguientes

- [Información general de Microsoft Azure Confidential Ledger](overview.md)
