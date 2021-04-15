---
title: Creación de una clasificación personalizada y una regla de clasificación (versión preliminar)
description: Descubra cómo crear clasificaciones personalizadas para definir en Azure Purview los tipos de datos únicos del patrimonio de datos de su organización.
author: animukherjee
ms.author: anmuk
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 3/24/2021
ms.openlocfilehash: 7d6baee49250509e50cdeeea8cf8ca6cec5b362d
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106222177"
---
# <a name="custom-classifications-in-azure-purview"></a>Clasificaciones personalizadas en Azure Purview

En este artículo se describe cómo puede crear clasificaciones personalizadas para definir los tipos de datos únicos para su organización en el patrimonio de datos. También se detalla la creación de reglas de clasificación personalizadas que le permitirán buscar los datos especificados en todo el patrimonio de datos.

## <a name="default-classifications"></a>Clasificaciones predeterminadas

Azure Purview Data Catalog proporciona un gran conjunto de clasificaciones predeterminadas que representan los tipos de datos personales típicos que puede tener en su patrimonio de datos.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/classification.png" alt-text="seleccionar clasificación" border="true":::

También tiene la posibilidad de crear clasificaciones personalizadas, si alguna de las clasificaciones predeterminadas no satisface sus necesidades.

## <a name="steps-to-create-a-custom-classification"></a>Pasos para crear una clasificación personalizada

Para crear una clasificación personalizada, siga estos pasos:

1. En el catálogo, seleccione **Centro de administración** en el menú de la izquierda.

2. Seleccione **Clasificaciones** en **Metadata management** (Administración de metadatos).

3. Seleccione **+ Nuevo**

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/new-classification.png" alt-text="Nueva clasificación" border="true":::

Se abrirá el panel **Add new classification** (Agregar nueva clasificación), donde puede asignar un nombre y una descripción a la clasificación. Se recomienda usar una convención de espacio de nombres, como `your company name.classification name`.

Las clasificaciones del sistema de Microsoft se agrupan en el espacio de nombres reservado `MICROSOFT.`. Por ejemplo, **MICROSOFT.GOVERNMENT.US.SOCIAL\_SECURITY\_NUMBER**.

El nombre de la clasificación debe empezar con una letra seguida de una secuencia de letras, números y caracteres de punto (.) o de guion bajo. No se permiten espacios. A medida que escribe, la experiencia de usuario genera automáticamente un nombre descriptivo. Este nombre descriptivo es lo que ven los usuarios cuando se aplica a un recurso en el catálogo.

Para que el nombre sea corto, el sistema crea el nombre descriptivo en función de la siguiente lógica:

- Menos los dos últimos, todos los segmentos del espacio de nombres se recortan.

- El uso de mayúsculas y minúsculas se ajusta para que la primera letra de cada palabra esté en mayúsculas. Todas las demás letras se convierten a minúsculas.

- Todos los guiones bajos (\_) se reemplazan por espacios.

Por ejemplo, si ha llamado a su clasificación **CONTOSO.HR.EMPLOYEE\_ID**, el nombre descriptivo se almacena en el sistema como **Hr.Employee ID**.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/contoso-hr-employee-id.png" alt-text="Contoso.hr.employee_id" border="true":::

Seleccione **Aceptar** y la nueva clasificación se agregará a la lista de clasificación.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/custom-classification.png" alt-text="Clasificación personalizada" border="true":::

Al seleccionar la clasificación en la lista, se abre la página de detalles de la clasificación. Aquí encontrará todos los detalles sobre la clasificación.

Estos detalles incluyen información sobre cuántas instancias hay, el nombre formal, las reglas de clasificación asociadas (si existen) y el nombre del propietario.

:::image type="content" source="media/create-a-custom-classification-and-classification-rule/select-classification.png" alt-text="Seleccionar clasificación" border="true":::

## <a name="custom-classification-rules"></a>Reglas de clasificación personalizadas

El servicio de catálogo proporciona un conjunto de reglas de clasificación predeterminadas que el analizador usa para detectar automáticamente determinados tipos de datos. También puede agregar sus propias reglas de clasificación personalizadas para detectar otros tipos de datos que puede que le interese encontrar en su patrimonio de datos. Esta capacidad puede ser eficaz cuando quiere buscar datos en su patrimonio de datos.

Por ejemplo, supongamos que una empresa llamada Contoso tiene id. de empleados que están normalizados en toda la empresa con la palabra \"Employee\" seguido de un GUID para crear EMPLOYEE{GUID}. Por ejemplo, una instancia de un ID de empleado tiene el siguiente aspecto `EMPLOYEE9c55c474-9996-420c-a285-0d0fc23f1f55`.

Contoso puede configurar el sistema de análisis para buscar instancias de estos id. mediante la creación de una regla de clasificación personalizada. Gracias a ello, estas reglas pueden proporcionar una expresión regular que coincida con el patrón de datos, en este caso `\^Employee\[A-Za-z0-9\]{8}-\[A-Za-z0-9\]{4}-\[A-Za-z0-9\]{4}-\[A-Za-z0-9\]{4}-\[A-Za-z0-9\]{12}\$`. Opcionalmente, si los datos suelen estar en una columna de la cual se conoce el nombre (como Employee\_ID o EmployeeID), se puede agregar una expresión regular de patrón de columna para que el análisis sea aún más preciso. Un ejemplo de expresión regular es Employee\_ID\|EmployeeID.

A continuación, el sistema de análisis puede usar esta regla para examinar los datos reales de la columna y el nombre de la columna para intentar identificar cada instancia en la que se encuentra el patrón del id. de empleado.

## <a name="steps-to-create-a-custom-classification-rule"></a>Pasos para crear una regla de clasificación personalizada

Para crear una regla de clasificación personalizada:

1. Cree una clasificación personalizada siguiendo las instrucciones de la sección anterior. Una vez hecho esto, debe agregar esta clasificación personalizada a la configuración de la regla de clasificación para que el sistema la aplique cuando encuentre una coincidencia en la columna.

2. Seleccione el icono del **Centro de administración**.

3. Seleccione la sección **Classifications rules** (Reglas de clasificaciones).

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/classificationrules.png" alt-text="Icono de reglas de clasificación" border="true":::

4. Seleccione **Nueva**.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/newclassificationrule.png" alt-text="Agregar una nueva regla de clasificación" border="true":::

5. Se abre el cuadro de diálogo **New classification rule** (Nueva regla de clasificación). Rellene los campos y decida si va a crear una **regla de expresión regular** o una **regla de diccionario**.

   |Campo     |Descripción  |
   |---------|---------|
   |Nombre   |    Necesario. El máximo es de 100 caracteres.    |
   |Descripción      |Opcional. El máximo es de 256 caracteres.    |
   |Nombre de clasificación    | Obligatorio. Seleccione el nombre de la clasificación en la lista desplegable para indicar al escáner que lo aplique si se encuentra una coincidencia.        |
   |State   |  Obligatorio. Las opciones están habilitadas o deshabilitadas. "Habilitado" es el valor predeterminado.    |

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/create-new-classification-rule.png" alt-text="Crear una nueva regla de clasificación" border="true":::

### <a name="creating-a-regular-expression-rule"></a>Creación de una regla de expresión regular

1. Si crea una regla de expresión regular, verá la pantalla siguiente. Si lo desea, puede cargar un archivo que se usará para **generar sugerencias de patrones de regex** para la regla.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/create-new-regex-rule.png" alt-text="Creación de nueva regla de regex" border="true":::

1. Si decide generar un patrón de regex sugerido, después de cargar un archivo, seleccione uno de los patrones sugeridos y seleccione **Add to Patterns** (Agregar a patrones) para usar los patrones de datos y columnas sugeridos. Puede retocar los patrones sugeridos o también puede escribir sus propios patrones sin cargar un archivo.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/suggested-regex.png" alt-text="Generación del patrón sugerido" border="true":::

   |Campo     |Descripción  |
   |---------|---------|
   |Patrón de datos    |Opcional. Expresión regular que representa los datos que se almacenan en el campo de datos. El límite es muy amplio. En el ejemplo anterior, los patrones de datos prueban un id. de empleado que es literalmente la palabra `Employee{GUID}`.  |
   |Patrón de columna    |Opcional. Expresión regular que representa los nombres de columna que quiere buscar. El límite es muy amplio. |

1. En **Patrón de datos**, hay dos umbrales que se pueden establecer:

   - **Umbral de coincidencia único**: es el número total de valores de datos únicos que deben encontrarse en una columna antes de que el escáner ejecute el patrón de datos en ella. El valor sugerido es 8. Este valor se puede ajustar manualmente en un rango de 2 a 32. Asimismo, el sistema necesita este valor para asegurarse de que la columna contiene suficientes datos para que el escáner pueda clasificarla con precisión. Por ejemplo, una columna que contenga varias filas con el valor 1 no se clasificará. Tampoco se clasificarán las columnas que contengan una fila con un valor y el resto de las filas con valores NULL. Recuerde que si especifica varios patrones, este valor se aplica a cada uno de ellos.

   - **Umbral de coincidencia mínimo**: puede usar esta opción para establecer el porcentaje mínimo de coincidencias de valores de datos únicos que debe encontrar el escáner en una columna para que se aplique la clasificación. El valor sugerido es el 60 %. Debe tener cuidado con esta configuración. Si reduce el nivel por debajo del 60 %, podría introducir clasificaciones de falsos positivos en el catálogo. Si especifica varios patrones de datos, esta configuración se deshabilitará y el valor se fijará en el 60 %.

1. Ahora puede comprobar la regla y **crearla**.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/verify-rule.png" alt-text="Comprobación de la regla antes de crearla" border="true":::

1. Pruebe la regla de clasificación antes de completar el proceso de creación para validar que se aplicarán etiquetas a los recursos. Las clasificaciones de la regla se aplicarán a los datos de ejemplo cargados, tal como lo haría en un examen. Esto significa que todas las clasificaciones del sistema y su clasificación personalizada coincidirán con los datos del archivo.

   Los archivos de entrada pueden incluir contenido de archivos delimitados (CSV, PSV, SSV, TSV), JSON o XML. El contenido se analizará en función de la extensión del archivo de entrada. Los datos delimitados pueden tener una extensión de archivo que coincida con cualquiera de los tipos mencionados. Por ejemplo, los datos TSV pueden existir en un archivo denominado MySampleData.csv. El contenido delimitado también debe tener un mínimo de 3 columnas.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/test-rule-screen.png" alt-text="Comprobación de la regla antes de crearla" border="true":::

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/test-rule-uploaded-file-result-screen.png" alt-text="Visualización de las clasificaciones aplicadas después de cargar un archivo de prueba" border="true":::

### <a name="creating-a-dictionary-rule"></a>Creación de una regla de diccionario

1. Si crea una regla de diccionario, verá la siguiente pantalla. Cargue un archivo que contenga todos los valores posibles para la clasificación que va a crear en una sola columna.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/dictionary-rule.png" alt-text="Creación de regla de diccionario" border="true":::

1. Una vez generado el diccionario, puede ajustar la coincidencia de valores únicos y los umbrales de coincidencia mínimos, y enviar la regla.

- **Umbral de coincidencia único**: es el número total de valores de datos únicos que deben encontrarse en una columna antes de que el escáner ejecute el patrón de datos en ella. Un umbral de coincidencia único no tiene nada que ver con la coincidencia de patrones, pero es un requisito previo de la coincidencia de patrones. El valor sugerido es 8. Este valor se puede ajustar manualmente en un rango de 2 a 32. Asimismo, el sistema necesita este valor para asegurarse de que la columna contiene suficientes datos para que el escáner pueda clasificarla con precisión. Por ejemplo, una columna que contenga varias filas con el valor 1 no se clasificará. Tampoco se clasificarán las columnas que contengan una fila con un valor y el resto de las filas con valores NULL. Recuerde que si especifica varios patrones, este valor se aplica a cada uno de ellos.

   :::image type="content" source="media/create-a-custom-classification-and-classification-rule/dictionary-generated.png" alt-text="Cree una regla de diccionario, con marca de verificación generada por el diccionario." border="true":::

## <a name="next-steps"></a>Pasos siguientes

Ahora que ya ha creado la regla de clasificación, ya la puede agregar a un conjunto de reglas de examen para que el proceso use esa regla al comenzar el examen. Para obtener más información, consulte [Creación de un conjunto de reglas de examen](create-a-scan-rule-set.md).
