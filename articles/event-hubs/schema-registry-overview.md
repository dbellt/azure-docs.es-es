---
title: Registro de esquema de Azure en Event Hubs (versión preliminar)
description: En este artículo se proporciona información general sobre la compatibilidad del registro de esquema de Azure Event Hubs (versión preliminar).
ms.topic: overview
ms.date: 05/10/2021
ms.custom: references_regions
ms.openlocfilehash: 43380ccd4f20616676f4eeb53fb2231b4f76f4e9
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110371212"
---
# <a name="azure-schema-registry-in-event-hubs-preview"></a>Registro de esquema de Azure en Event Hubs (versión preliminar)
En muchos escenarios de transmisión de eventos y de mensajería, la carga de eventos o mensajes contiene datos estructurados que se serializan o deserializan con un formato controlado por esquemas, como Apache Avro. Tal vez los remitentes y receptores quieran validar la integridad de los datos con un documento de esquema, como con el esquema JSON. En el caso de los formatos controlados por esquema, que el esquema esté disponible para el consumidor de mensajes es un requisito previo para que el consumidor pueda deserializar los datos. 

El **registro de esquema de Azure** es una característica de Event Hubs, que proporciona un repositorio central para los documentos de esquema de aplicaciones controladas por eventos y basadas en mensajería. Proporciona la flexibilidad para que las aplicaciones de productor y consumidor intercambien datos sin tener que administrar y compartir el esquema entre ellos, y también para evolucionar a diferentes velocidades. El registro de esquema también proporciona un marco de gobierno sencillo para los esquemas reutilizables y define la relación entre los esquemas a través de una construcción de agrupación (grupos de esquemas).

> [!NOTE]
> - La característica de **registro de esquema** se encuentra actualmente en **versión preliminar** y no se recomienda para las cargas de trabajo de producción.
> - La característica no está disponible en el nivel **Básico**.

Con marcos de serialización orientados a esquemas, como Apache Avro, la externalización de los metadatos de serialización en esquemas compartidos también puede ayudar a reducir drásticamente la sobrecarga por mensaje de información de tipo y nombres de campo incluidos con cada conjunto de datos, como es el caso de los formatos etiquetados como JSON. Al almacenar los esquemas junto con los eventos y dentro de la infraestructura de eventos se garantiza que los metadatos necesarios para la serialización o la deserialización estén siempre al alcance y los esquemas no se puedan perder. 

## <a name="event-hubs-namespace"></a>Espacio de nombres de Event Hubs
Un espacio de nombres de Event Hubs ahora puede hospedar grupos de esquemas junto con Event Hubs (o temas de Kafka). Hospeda un registro de esquema y puede tener varios grupos de esquemas. A pesar de estar hospedado en Azure Event Hubs, el registro de esquema se puede usar universalmente con todos los servicios de mensajería de Azure y cualquier otro agente de mensajes o eventos. Cada uno de estos grupos de esquemas es un repositorio protegible de forma independiente para un conjunto de esquemas. Los grupos se pueden alinear con una aplicación determinada o una unidad organizativa. 

## <a name="schema-groups"></a>Grupos de esquemas
El grupo de esquemas es un grupo lógico de esquemas similares basados en los criterios empresariales. Un grupo de esquemas puede contener varias versiones de un esquema. La configuración del cumplimiento de compatibilidad de un grupo de esquemas puede ayudar a garantizar que las versiones del esquema más recientes sean compatibles con versiones anteriores.

El límite de seguridad impuesto por el mecanismo de agrupación ayuda a garantizar que los secretos comerciales no se filtran accidentalmente a través de los metadatos en situaciones en las que el espacio de nombres se comparte entre varios asociados. También permite a los propietarios de aplicaciones administrar esquemas independientes de otras aplicaciones que comparten el mismo espacio de nombres.


## <a name="schemas"></a>Esquemas
Los esquemas definen el contrato entre los productores y los consumidores. Un esquema definido en un registro de esquema de Event Hubs ayuda a administrar el contrato fuera de los datos de eventos, lo que elimina la sobrecarga. Un esquema tiene un nombre, un tipo (por ejemplo, registro, matriz, etc.), el modo de compatibilidad (ninguna, con versiones anteriores o posteriores, completa) y el tipo de serialización (solo Avro por ahora). Puede crear varias versiones de un esquema y recuperar y usar una versión específica de un esquema. 

## <a name="client-sdks"></a>SDK de cliente
Puede usar una de las siguientes bibliotecas que incluyen un serializador de Avro, que puede usar para serializar y deserializar las cargas que contienen identificadores de esquema del registro de esquema y datos codificados por Avro.

- [.NET: Microsoft.Azure.Data.SchemaRegistry.ApacheAvro](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/schemaregistry/Microsoft.Azure.Data.SchemaRegistry.ApacheAvro)
- [Java: azure-data-schemaregistry-avro](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/schemaregistry/azure-data-schemaregistry-avro/)
- [Python: azure-schemaregistry-avroserializer](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/schemaregistry/azure-schemaregistry-avroserializer)
- [JavaScript: @azure/schema-registry-avro](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/schemaregistry/schema-registry-avro)
- [Apache Kafka](https://github.com/Azure/azure-schema-registry-for-kafka/): ejecute los serializadores y deserializadores de Apache Avro integrados en Kafka con el respaldo del registro de esquema de Azure. El serializador de cliente de Apache Kafka del cliente de Java para el registro de esquema de Azure se puede usar en cualquier escenario de Apache Kafka y con cualquier servicio en la nube o implementación basada en Apache Kafka®. 

La siguiente imagen muestra el flujo de información del registro de esquema con Event Hubs: 

:::image type="content" source="./media/schema-registry-overview/flow-diagram.png" alt-text="Diagrama de flujo":::

## <a name="limits"></a>Límites
Para conocer los límites (por ejemplo, el número de grupos de esquemas de un espacio de nombres) de Event Hubs, consulte [Cuotas y límites de Event Hubs](event-hubs-quotas.md).

## <a name="azure-role-based-access-control"></a>Control de acceso basado en roles de Azure
Al acceder al registro de esquema mediante programación, debe registrar una aplicación en Azure Active Directory (Azure AD) y agregar la entidad de seguridad de la aplicación a uno de los roles de control de acceso basado en rol de Azure (Azure RBAC):

| Role | Descripción | 
| ---- | ----------- | 
| Propietario | Leer, escribir y eliminar esquemas y grupos de registros de esquema. |
| Colaborador | Leer, escribir y eliminar esquemas y grupos de registros de esquema. |
| [Lector del registro de esquema (versión preliminar)](../role-based-access-control/built-in-roles.md#schema-registry-reader-preview) | Leer y enumerar grupos de registros de esquema y esquemas. |
| [Colaborador del registro de esquema (versión preliminar)](../role-based-access-control/built-in-roles.md#schema-registry-reader-preview) | Leer, escribir y eliminar esquemas y grupos de registros de esquema. |

Para obtener instrucciones sobre cómo crear el registro de una aplicación mediante Azure Portal, consulte [Registro de una aplicación con Azure AD](../active-directory/develop/quickstart-register-app.md). Anote el id. de cliente (id. de aplicación), el id de inquilino y el secreto que se usarán en el código. 

## <a name="next-steps"></a>Pasos siguientes

- Para información sobre cómo crear un registro de esquema mediante Azure Portal, vea [Creación de un registro de esquema de Event Hubs con Azure Portal](create-schema-registry.md).
- Vea los siguientes ejemplos de la **biblioteca de cliente Avro del registro de esquema**.
    - [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/schemaregistry/Microsoft.Azure.Data.SchemaRegistry.ApacheAvro/tests/Samples)
    - [Java](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/schemaregistry/azure-data-schemaregistry-avro/src/samples)
    - [JavaScript](https://github.com/Azure/azure-sdk-for-js/tree/master/sdk/schemaregistry/schema-registry-avro/samples )
    - [Python](https://github.com/Azure/azure-sdk-for-python/tree/master/sdk/schemaregistry/azure-schemaregistry-avroserializer/samples )
    - [Integración de Kafka Avro para el registro de esquema de Azure](https://github.com/Azure/azure-schema-registry-for-kafka/tree/master/csharp/avro/samples)
