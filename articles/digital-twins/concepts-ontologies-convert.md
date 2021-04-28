---
title: Conversión de ontologías
titleSuffix: Azure Digital Twins
description: Descripción del proceso de conversión de modelos estándar del sector en DTDL para Azure Digital Twins
author: baanders
ms.author: baanders
ms.date: 2/12/2021
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 22b41fce59bf7dbe9db1186036c5ed44f07a4aad
ms.sourcegitcommit: afb79a35e687a91270973990ff111ef90634f142
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/14/2021
ms.locfileid: "107484492"
---
# <a name="convert-industry-standard-ontologies-to-dtdl-for-azure-digital-twins"></a>Conversión de ontologías estándar del sector a DTDL para Azure Digital Twins

La mayoría de las [ontologías](concepts-ontologies.md) se basan en estándares web semánticos como [OWL](https://www.w3.org/OWL/), [RDF](https://www.w3.org/2001/sw/wiki/RDF) y [RDFS](https://www.w3.org/2001/sw/wiki/RDFS). 

Para usar un modelo con Azure Digital Twins, debe tener el formato DTDL. En este artículo se describen las instrucciones de diseño generales en forma de **patrón de conversión** para convertir modelos basados en RDF al lenguaje DTDL, con el fin de que se puedan usar con Azure Digital Twins. 

Este artículo también contiene un [**código de ejemplo de convertidor**](#converter-samples) para los convertidores RDF y OWL, que se puede extender para otros esquemas del sector de la construcción.

## <a name="conversion-pattern"></a>Patrón de conversión

Hay varias bibliotecas de terceros que se pueden usar al convertir modelos basados en RDF a DTDL. Algunas de estas bibliotecas le permiten cargar el archivo del modelo en un grafo. Puede recorrer el grafo en bucle en busca de construcciones específicas de RDFS y OWL, y realizar la conversión a DTDL.   

La tabla siguiente es un ejemplo de cómo se pueden asignar las construcciones RDFS y OWL a DTDL. 

| Concepto RDFS/OWL | Construcción RDFS/OWL | Concepto DTDL | Construcción DTDL |
| --- | --- | --- | --- |
| Clases | `owl:Class`<br>Sufijo IRI<br>``rdfs:label``<br>``rdfs:comment`` | Interfaz | `@type:Interface`<br>`@id`<br>`displayName`<br>`comment` 
| Subclases  | `owl:Class`<br>Sufijo IRI<br>`rdfs:label`<br>`rdfs:comment`<br>`rdfs:subClassOf` | Interfaz | `@type:Interface`<br>`@id`<br>`displayName`<br>`comment`<br>`extends` 
| Propiedades de tipos de datos | `owl:DatatypeProperty`<br>`rdfs:label` o `INode`<br>`rdfs:label`<br>`rdfs:range` | Propiedades de interfaz | `@type:Property`<br>`name`<br>`displayName`<br>`schema` 
| Propiedades de objeto | `owl:ObjectProperty`<br>`rdfs:label` o `INode`<br>`rdfs:range`<br>`rdfs:comment`<br>`rdfs:label` | Relación | `type:Relationship`<br>`name`<br>`target` (o se omite si no hay `rdfs:range`)<br>`comment`<br>`displayName`<br>

En el siguiente fragmento de código en C# se muestra cómo se carga un archivo de modelo de RDF en un grafo y se convierte a DTDL, mediante la biblioteca [**dotNetRDF**](https://www.dotnetrdf.org/). 

:::code language="csharp" source="~/digital-twins-docs-samples/sdks/csharp/convertRDF.cs":::

## <a name="converter-samples"></a>Ejemplos de convertidor

### <a name="rdf-converter-application"></a>Aplicación del convertidor RDF 

Hay una aplicación de ejemplo disponible que convierte un archivo de modelo basado en RDF a [DTDL (versión 2)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) para su uso por parte del servicio Azure Digital Twins. Se ha validado para el esquema de [Brick](https://brickschema.org/ontology/) y se puede extender a otros esquemas del sector de la construcción (como [Building Topology Ontology (BOT)](https://w3c-lbd-cg.github.io/bot/), [Semantic Sensor Network](https://www.w3.org/TR/vocab-ssn/) o [buildingSmart Industry Foundation Classes (IFC)](https://technical.buildingsmart.org/standards/ifc/ifc-schema-specifications/)).

El ejemplo es una aplicación de línea de comandos de .NET Core denominada **RdfToDtdlConverter**.

Aquí puede obtener el ejemplo: [**RdfToDtdlConverter**](/samples/azure-samples/rdftodtdlconverter/digital-twins-model-conversion-samples/). 

Para descargar el código en la máquina, seleccione el botón **Examinar código** debajo del título de la página de ejemplo, que le llevará al repositorio de GitHub para el ejemplo. Seleccione el botón **Code** (Código) y **Download ZIP** (Descargar ZIP) para descargar el ejemplo como un archivo *.ZIP* denominado *RdfToDtdlConverter-main.zip*. A continuación, puede descomprimir el archivo y explorar el código.

:::image type="content" source="media/concepts-ontologies-convert/download-repo-zip.png" alt-text="Captura de pantalla del repositorio RdfToDtdlConverter en GitHub. El botón Code (Código) está seleccionado y se muestra un pequeño cuadro de diálogo en el que el botón Download ZIP (Descargar ZIP) está resaltado" lightbox="media/concepts-ontologies-convert/download-repo-zip.png":::

Puede usar este ejemplo para ver los patrones de conversión en contexto y tener como bloque de creación de sus propias aplicaciones la realización de conversiones de modelos de acuerdo con sus propias necesidades específicas.

### <a name="owl2dtdl-converter"></a>Convertidor OWL2DTDL 

El [**convertidor OWL2DTDL**](https://github.com/Azure/opendigitaltwins-building-tools/tree/master/OWL2DTDL) es un ejemplo que permite trasladar una ontología de OWL a un conjunto de declaraciones de interfaz DTDL que se puede usar con el servicio Azure Digital Twins. Esto también funciona para redes de ontologías que están constituidas por una ontología raíz que reutiliza otras ontologías a través de declaraciones `owl:imports`.

Este convertidor se usó para trasladar la [ontología RealStateCore](https://doc.realestatecore.io/3.1/full.html) a DTDL y se puede utilizar con cualquier ontología basada en OWL.

## <a name="next-steps"></a>Pasos siguientes 

* Obtenga más información sobre cómo extender las ontologías estándar del sector para que satisfagan sus especificaciones: [*Conceptos: Extensión de las ontologías del sector*](concepts-ontologies-extend.md).

* O bien, continúe el proceso de aprendizaje para desarrollar modelos basados en ontologías: [*Uso de estrategias de ontología en un procedimiento de desarrollo de modelos*](concepts-ontologies.md#using-ontology-strategies-in-a-model-development-path).