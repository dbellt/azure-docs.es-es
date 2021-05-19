---
title: Referencia para la escritura de expresiones para la asignación de atributos en el aprovisionamiento de aplicaciones de Azure Active Directory
description: Obtenga información sobre cómo usar asignaciones de expresiones para transformar valores de atributos en un formato aceptable durante el aprovisionamiento automático de objetos de aplicaciones SaaS en Azure Active Directory. Incluye una lista de referencias de funciones.
services: active-directory
author: mtillman
manager: daveba
ms.service: active-directory
ms.subservice: app-provisioning
ms.workload: identity
ms.topic: reference
ms.date: 05/11/2021
ms.author: kenwith
ms.reviewer: arvinh
ms.openlocfilehash: 3c5d2023ebc26aca0e6d1a14f43a94d2d3e3dd87
ms.sourcegitcommit: 32ee8da1440a2d81c49ff25c5922f786e85109b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109782962"
---
# <a name="reference-for-writing-expressions-for-attribute-mappings-in-azure-active-directory"></a>Referencia para la escritura de expresiones para la asignación de atributos en Azure Active Directory

Al configurar el aprovisionamiento para una aplicación SaaS, uno de los tipos de asignaciones de atributos que puede especificar es una asignación de expresiones. En estos casos, debe escribir una expresión similar a un script que permita transformar los datos de los usuarios en formatos más aceptables para la aplicación SaaS.

## <a name="syntax-overview"></a>Información general sobre la sintaxis

La sintaxis de expresiones para asignaciones de atributos recuerda a las funciones de Visual Basic para Aplicaciones (VBA).

* Toda la expresión se tiene que definir en términos de funciones, que constan de un nombre seguido de argumentos entre paréntesis:  *FunctionName(`<<argument 1>>`,`<<argument N>>`)*
* Es posible anidar funciones dentro de otras. Por ejemplo:  *FunctionOne(FunctionTwo(`<<argument1>>`))*
* Puede transformar tres tipos diferentes de argumentos en funciones:
  
  1. Atributos, que deben ir entre corchetes. Por ejemplo: [NombreAtributo]
  2. Constantes de cadena, que deben ir entre comillas. Por ejemplo: "Estados Unidos"
  3. Otras funciones. Por ejemplo: FunctionOne(`<<argument1>>`, FunctionTwo(`<<argument2>>`))
* Para las constantes de cadena, si necesita una barra diagonal inversa (\) o comillas dobles (") en la cadena, se deben convertirse con el símbolo de barra diagonal inversa (\). Por ejemplo: "Nombre de la empresa: \\"Contoso\\""
* La sintaxis distingue entre mayúsculas y minúsculas, lo que se debe tener en cuenta al escribirlas como cadenas en una función, por oposición a copiarlas y pegarlas directamente desde aquí. 

## <a name="list-of-functions"></a>Lista de funciones

[Append](#append) &nbsp;&nbsp;&nbsp;&nbsp; [BitAnd](#bitand) &nbsp;&nbsp;&nbsp;&nbsp; [CBool](#cbool) &nbsp;&nbsp;&nbsp;&nbsp; [Coalesce](#coalesce) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToBase64](#converttobase64) &nbsp;&nbsp;&nbsp;&nbsp; [ConvertToUTF8Hex](#converttoutf8hex) &nbsp;&nbsp;&nbsp;&nbsp; [Count](#count) &nbsp;&nbsp;&nbsp;&nbsp; [CStr](#cstr) &nbsp;&nbsp;&nbsp;&nbsp; [DateFromNum](#datefromnum) &nbsp;[FormatDateTime](#formatdatetime) &nbsp;&nbsp;&nbsp;&nbsp; [Guid](#guid) &nbsp;&nbsp;&nbsp;&nbsp; [IIF](#iif) &nbsp;&nbsp;&nbsp;&nbsp;[InStr](#instr) &nbsp;&nbsp;&nbsp;&nbsp; [IsNull](#isnull) &nbsp;&nbsp;&nbsp;&nbsp; [IsNullOrEmpty](#isnullorempty) &nbsp;&nbsp;&nbsp;&nbsp; [IsPresent](#ispresent) &nbsp;&nbsp;&nbsp;&nbsp; [IsString](#isstring) &nbsp;&nbsp;&nbsp;&nbsp; [Item](#item) &nbsp;&nbsp;&nbsp;&nbsp; [Join](#join) &nbsp;&nbsp;&nbsp;&nbsp; [Left](#left) &nbsp;&nbsp;&nbsp;&nbsp; [Mid](#mid) &nbsp;&nbsp; &nbsp;&nbsp;&nbsp;&nbsp; [NormalizeDiacritics](#normalizediacritics) [Not](#not) &nbsp;&nbsp;&nbsp;&nbsp; [NumFromDate](#numfromdate) &nbsp;&nbsp;&nbsp;&nbsp; [RemoveDuplicates](#removeduplicates) &nbsp;&nbsp;&nbsp;&nbsp; [Replace](#replace) &nbsp;&nbsp;&nbsp;&nbsp; [SelectUniqueValue](#selectuniquevalue)&nbsp;&nbsp;&nbsp;&nbsp; [SingleAppRoleAssignment](#singleapproleassignment)&nbsp;&nbsp;&nbsp;&nbsp; [Split](#split)&nbsp;&nbsp;&nbsp;&nbsp;[StripSpaces](#stripspaces) &nbsp;&nbsp;&nbsp;&nbsp; [Switch](#switch)&nbsp;&nbsp;&nbsp;&nbsp; [ToLower](#tolower)&nbsp;&nbsp;&nbsp;&nbsp; [ToUpper](#toupper)&nbsp;&nbsp;&nbsp;&nbsp; [Word](#word)

---
### <a name="append"></a>Append

**Función:**  Append(source, suffix)

**Descripción:**  adopta un valor de la cadena de origen y anexa el sufijo al final de la misma.

**Parámetros:**

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen. |
| **suffix** |Obligatorio |String |La cadena que se va a anexar al final del valor de origen. |


### <a name="append-constant-suffix-to-user-name"></a>Anexar sufijos constantes a nombres de usuario
Ejemplo: si está utilizando un espacio aislado de Salesforce, deberá anexar un sufijo adicional a todos los nombres de usuario antes de sincronizarlas.

**Expresión:**  
`Append([userPrincipalName], ".test")`

**Entrada/salida de ejemplo:** 

* **ENTRADA**: (userPrincipalName): "John.Doe@contoso.com"
* **ENTRADA**:  "John.Doe@contoso.com.test"


---
### <a name="bitand"></a>BitAnd
**Función:** BitAnd(value1, value2)

**Descripción:** : esta función convierte ambos parámetros en la representación binaria y establece un bit en los siguientes valores:

- 0: si uno o ambos de los bits correspondientes en value1 y value2 son 0.
- 1: si ambos de los bits correspondientes son 1.

En otras palabras, devuelve 0 en todos los casos excepto cuando los bits correspondientes de los dos parámetros son 1.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **value1** |Obligatorio |num |Valor numérico que se debe agregar con AND a value2|
| **value2** |Obligatorio |num |Valor numérico que se debe agregar con AND a value1|

**Ejemplo:** 
`BitAnd(&HF, &HF7)`

11110111 AND 00000111 = 00000111 de modo que `BitAnd` devuelve 7, el valor binario de 00000111.

---
### <a name="cbool"></a>CBool
**Función:**  
`CBool(Expression)`

**Descripción:**  
`CBool` devuelve un valor booleano basado en la expresión evaluada. Si la expresión se evalúa en un valor distinto de cero, `CBool` devuelve *True*. En caso contrario, devuelve *False*.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **expression** |Obligatorio | expresión | Cualquier expresión válida |

**Ejemplo:** 
`CBool([attribute1] = [attribute2])`                                                                    
Devuelve True si ambos atributos tienen el mismo valor.

---
### <a name="coalesce"></a>Coalesce
**Función:** Coalesce(source1, source2, ..., defaultValue)

**Descripción:** Devuelve el primer valor de origen que no es NULL. Si todos los argumentos son NULL y defaultValue está presente, se devolverá defaultValue. Si todos los argumentos son NULL y defaultValue no está presente, Coalesce devuelve NULL.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **source1  … sourceN** | Obligatorio | String |Obligatorio, número variable de veces. Normalmente el nombre del atributo del objeto de origen. |
| **defaultValue** | Opcional | String | Valor predeterminado que usará cuando todos los valores de origen sean NULL. Puede tratarse de una cadena vacía ("").

### <a name="flow-mail-value-if-not-null-otherwise-flow-userprincipalname"></a>Proporciona el valor de correo si no es NULL; de lo contrario, proporciona userPrincipalName
Ejemplo: Desea proporcionar el atributo de correo si está presente. Si no es así, desea proporcionar el valor de userPrincipalName en su lugar.

**Expresión:**  
`Coalesce([mail],[userPrincipalName])`

**Entrada/salida de ejemplo:** 

* **ENTRADA** (mail): NULL
* **ENTRADA**: (userPrincipalName): "John.Doe@contoso.com"
* **ENTRADA**:  "John.Doe@contoso.com"


---
### <a name="converttobase64"></a>ConvertToBase64
**Función:** ConvertToBase64(source)

**Descripción:** La función ConvertToBase64 convierte una cadena en una en base64 de Unicode.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Cadena que se va a convertir a base 64.|

**Ejemplo:** 
`ConvertToBase64("Hello world!")`

devuelve "SABlAGwAbABvACAAdwBvAHIAbABkACEA".

---
### <a name="converttoutf8hex"></a>ConvertToUTF8Hex
**Función:** ConvertToUTF8Hex(source)

**Descripción:** : la función ConvertToUTF8Hex convierte una cadena en un valor codificado hexadecimal UTF8.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Cadena que se va a convertir a UTF8 Hex.|

**Ejemplo:** 
`ConvertToUTF8Hex("Hello world!")`

devuelve 48656C6C6F20776F726C6421.

---
### <a name="count"></a>Count
**Función:** Count(attribute)

**Descripción:** : la función Count devuelve el número de elementos en un atributo multivalor.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **attribute** |Obligatorio |atributo |Atributo con varios valores que tendrá elementos contados.|

---
### <a name="cstr"></a>CStr
**Función:** CStr(value)

**Descripción:** La función CStr convierte un valor en un tipo de datos de cadena.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **value** |Obligatorio | Numérico, referencia o booleano | puede ser un valor numérico, un atributo de referencia o un valor booleano. |

**Ejemplo:** 
`CStr([dn])`

Devuelve "cn=Joe,dc=contoso,dc=com".

---
### <a name="datefromnum"></a>DateFromNum
**Función:** DateFromNum(value)

**Descripción:** la función DateFromNum convierte un valor con formato de fecha de AD en un tipo DateTime.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **value** |Obligatorio | Date | Fecha de AD que se va a convertir al tipo DateTime. |

**Ejemplo:** 
`DateFromNum([lastLogonTimestamp])`

`DateFromNum(129699324000000000)`

Devuelve un valor DateTime que representa el 1 de enero de 2012 a las 23 h.

---
### <a name="formatdatetime"></a>FormatDateTime
**Función:** FormatDateTime(source, dateTimeStyles, inputFormat, outputFormat)

**Descripción:**  adopta una cadena de fecha en un formato y la convierte a un formato distinto.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen. |
| **dateTimeStyles** | Opcional | String | Utilice este parámetro para especificar las opciones de formato que personalizan el análisis de cadenas en algunos métodos de análisis de fecha y hora. Para conocer los valores admitidos, consulte la [documentación de DateTimeStyles](/dotnet/api/system.globalization.datetimestyles). Si se deja vacío, se utiliza el valor predeterminado: DateTimeStyles.RoundtripKind, DateTimeStyles.AllowLeadingWhite, DateTimeStyles.AllowTrailingWhite.  |
| **inputFormat** |Obligatorio |String |Formato esperado del valor de origen. Para conocer los formatos admitidos, consulte [/dotnet/standard/base-types/custom-date-and-time-format-strings](/dotnet/standard/base-types/custom-date-and-time-format-strings). |
| **outputFormat** |Obligatorio |String |Formato de la fecha de salida. |



### <a name="output-date-as-a-string-in-a-certain-format"></a>Fecha de resultado como una cadena en un formato determinado
Ejemplo: desea enviar fechas a una aplicación SaaS como ServiceNow en un formato determinado. Puede usar la siguiente expresión. 

**Expresión:** 

`FormatDateTime([extensionAttribute1], , "yyyyMMddHHmmss.fZ", "yyyy-MM-dd")`

**Entrada/salida de ejemplo:**

* **INPUT** (extensionAttribute1): "20150123105347.1Z"
* **OUTPUT**:  "2015-01-23"


---
### <a name="guid"></a>Guid
**Función:** Guid()

**Descripción:** La función GUID genera un nuevo GUID aleatorio.

---
### <a name="iif"></a>IIF
**Función:** IIF(condition,valueIfTrue,valueIfFalse)

**Descripción:** : la función IIF devuelve uno de un conjunto de valores posibles según una condición especificada.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **condition** |Obligatorio |Variable o expresión |Cualquier valor o expresión que pueda evaluarse como true o false. |
| **valueIfTrue** |Obligatorio |Variable o cadena | si la condición se evalúa como true, el valor devuelto. |
| **valueIfFalse** |Obligatorio |Variable o cadena |si la condición se evalúa como false, el valor devuelto.|

**Ejemplo:** 
`IIF([country]="USA",[country],[department])`

---
### <a name="instr"></a>InStr
**Función:** InStr(value1, value2, start, compareType)

**Descripción:** : la función InStr busca la primera repetición de una subcadena en una cadena.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **value1** |Obligatorio |String |Cadena que se va a buscar. |
| **value2** |Obligatorio |String |Cadena que se va a encontrar. |
| **start** |Opcional |Entero |Posición inicial para buscar la subcadena.|
| **compareType** |Opcional |Enum |Puede ser vbTextCompare o vbBinaryCompare. |

**Ejemplo:** 
`InStr("The quick brown fox","quick")`

Se evalúa en 5

`InStr("repEated","e",3,vbBinaryCompare)`

Se evalúa en 7.

---
### <a name="isnull"></a>IsNull
**Función:** IsNull(Expression)

**Descripción:**  si la expresión se evalúa como Null, la función IsNull devuelve True. para un atributo, un valor Null se expresa mediante la ausencia del atributo.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **expression** |Obligatorio |expresión |Expresión que se va a evaluar. |

**Ejemplo:** 
`IsNull([displayName])`

Devuelve True si el atributo no está presente.

---
### <a name="isnullorempty"></a>IsNullOrEmpty
**Función:** IsNullOrEmpty(Expression)

**Descripción:**  si la expresión es Null o una cadena vacía, la función IsNullOrEmpty devuelve True. para un atributo, esto se evaluaría como True si el atributo no está presente o está presente, pero es una cadena vacía.
La función contraria a esta es IsPresent.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **expression** |Obligatorio |expresión |Expresión que se va a evaluar. |

**Ejemplo:** 
`IsNullOrEmpty([displayName])`

Devuelve True si el atributo no está presente o si es una cadena vacía.

---
### <a name="ispresent"></a>IsPresent
**Función:** IsPresent(Expression)

**Descripción:**  si la expresión se evalúa como una cadena que no es Null y no está vacía, la función IsPresent devuelve True. la función contraria a esta es IsNullOrEmpty.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **expression** |Obligatorio |expresión |Expresión que se va a evaluar. |

**Ejemplo:** 
`Switch(IsPresent([directManager]),[directManager], IsPresent([skiplevelManager]),[skiplevelManager], IsPresent([director]),[director])`

---
### <a name="isstring"></a>IsString
**Función:** IsString(Expression)

**Descripción:**  si la expresión se puede evaluar en un tipo de cadena, la función IsString se evalúa en True.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **expression** |Obligatorio |expresión |Expresión que se va a evaluar. |

---
### <a name="item"></a>Elemento
**Función:** Item(attribute, index)

**Descripción:**  la función Item devuelve un elemento de un atributo o una cadena de varios valores.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **attribute** |Obligatorio |Atributo |Atributo multivalor en el que se va a buscar. |
| **índice** |Obligatorio |Entero | Índice de un elemento en la cadena multivalor.|

**Ejemplo:** 
`Item([proxyAddresses], 1)` devuelve el segundo elemento en el atributo de varios valores.

---
### <a name="join"></a>Join
**Función:**  Join(separator, source1, source2, …)

**Descripción:** Join() es similar a Append(), excepto en que puede combinar varios valores de cadena de **source** en una sola cadena, y cada valor estará separado por una cadena de **separator**.

Si uno de los valores de origen es un atributo multivalor, cada valor de ese atributo se unirá, separado por el valor del separador.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **separator** |Obligatorio |String |Cadena utilizada para separar los valores de origen cuando se concatenan en una sola cadena. Puede ser "" si no es necesario ningún separador. |
| **source1  … sourceN** |Obligatorio, número variable de veces |String |Valores de cadena que se van a agrupar. |

---
### <a name="left"></a>Left
**Función:** Left(String,NumChars)

**Descripción:**  la función Left devuelve un número especificado de caracteres desde la izquierda de una cadena. Con numChars = 0, se devuelve una cadena vacía.
Con numChars < 0, se devuelve una cadena de entrada.
Si la cadena es null, se devuelve una cadena vacía.
Si la cadena contiene menos caracteres que el número especificado en numChars, se devuelve una cadena idéntica a la cadena (es decir, que contiene todos los caracteres en el parámetro 1).

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **String** |Obligatorio |Atributo | Cadena de la que se van a devolver caracteres. |
| **NumChars** |Obligatorio |Entero | Número que identifica el número de caracteres que se va a devolver desde el principio (la izquierda) de la cadena.|

**Ejemplo:** 
`Left("John Doe", 3)`

devuelve “Joh”.

---
### <a name="mid"></a>Mid
**Función:**  Mid(source, start, length)

**Descripción:**  devuelve una subcadena del valor de origen. Una subcadena es una cadena que contiene sólo algunos de los caracteres de la cadena de origen.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo. |
| **start** |Obligatorio |integer |Índice de la cadena de **source** donde debe empezar la subcadena. El primer carácter de la cadena tendrá el índice de 1, el segundo carácter tendrá el índice de 2, y así sucesivamente. |
| **length** |Obligatorio |integer |Longitud de la subcadena. Si length acaba fuera de la cadena de **source**, la función devolverá una subcadena desde el índice de **start** hasta el final de la cadena de **source**. |

---
### <a name="normalizediacritics"></a>NormalizeDiacritics
**Función:** NormalizeDiacritics(source)

**Descripción:** Requiere un argumento de cadena. Devuelve la cadena, pero se reemplazan todos los caracteres diacríticos por sus equivalentes no diacríticos. Normalmente se usa para convertir nombres y apellidos que contienen caracteres diacríticos (acentos) en valores legales que se pueden emplear en diversos identificadores de usuario como, nombres principales de usuario, nombres de cuenta de SAM y direcciones de correo electrónico.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String | Normalmente un atributo de nombre o de apellido. |


### <a name="remove-diacritics-from-a-string"></a>Quitar los signos diacríticos de una cadena
Ejemplo: necesita reemplazar caracteres que contienen acentos por otros equivalentes que no los contengan.

**Expresión:** NormalizeDiacritics([givenName])

**Entrada/salida de ejemplo:** 

* **INPUT** (givenName): "Zoë"
* **OUTPUT**:  "Zoe"


---
### <a name="not"></a>Not
**Función:**  Not(source)

**Descripción:** Invierte el valor booleano de **source**. Si el valor de **source** es True, devuelve False. De lo contrario, devuelve True.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |Cadena booleana |Los valores de **source** esperados son "True" o "False". |

---
### <a name="numfromdate"></a>NumFromDate
**Función:** NumFromDate(value)

**Descripción:** La función NumFromDate convierte un valor DateTime a un formato de Active Directory necesario para establecer atributos como [accountExpires](/windows/win32/adschema/a-accountexpires). Use esta función para convertir valores DateTime recibidos de aplicaciones de recursos humanos en la nube, como WorkDay y SuccessFactors, a su representación de AD equivalente. 

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **value** |Obligatorio | String | Cadena de fecha y hora en el formato admitido. Para conocer los formatos admitidos, consulte https://msdn.microsoft.com/library/8kb3ddd4%28v=vs.110%29.aspx. |

**Ejemplo**:
* Ejemplo de Workday. Suponiendo que desee asignar el atributo *ContractEndDate* desde WorkDay, que tiene el formato *2020-12-31-08:00* al campo *accountExpires* en AD, aquí se muestra cómo puede usar esta función y cambiar el desplazamiento de zona horaria para que coincida con la configuración regional. 
  `NumFromDate(Join("", FormatDateTime([ContractEndDate], ,"yyyy-MM-ddzzz", "yyyy-MM-dd"), "T23:59:59-08:00"))`

* Ejemplo de SuccessFactors Suponiendo que desea asignar el atributo *endDate* de SuccessFactors, que tiene el formato *M/d/yyyy hh:mm:ss tt* para el campo *accountExpires* en AD, aquí se muestra cómo puede usar esta función y cambiar el ajuste de zona horaria para que coincida con la configuración regional.
  `NumFromDate(Join("",FormatDateTime([endDate], ,"M/d/yyyy hh:mm:ss tt","yyyy-MM-dd"),"T23:59:59-08:00"))`


---
### <a name="removeduplicates"></a>RemoveDuplicates
**Función:** RemoveDuplicates(attribute)

**Descripción:**  la función RemoveDuplicates toma una cadena multivalor y garantiza que cada valor es único.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **attribute** |Obligatorio |Atributo multivalor |Atributo multivalor en el que se habrán quitado los duplicados.|

**Ejemplo:** 
`RemoveDuplicates([proxyAddresses])`  devuelve un atributo proxyAddress saneado donde se han quitado todos los valores duplicados.

---
### <a name="replace"></a>Replace
**Función:** Replace(source, oldValue, regexPattern, regexGroupName, replacementValue, replacementAttributeName, template)

**Descripción:** Reemplaza los valores de una cadena de manera que distinga entre mayúsculas y minúsculas. La función se comporta de forma diferente según los parámetros proporcionados:

* Cuando se proporcionan **oldValue** y **replacementValue**:
  
  * Reemplaza todas las ocurrencias de **oldValue** en el **origen** por **replacementValue**
* Cuando se proporcionan **oldValue** y **template**:
  
  * Reemplaza todas las ocurrencias de **oldValue** en **template** por el valor de **source**
* Cuando se proporcionan **regexPattern** y **replacementValue**:

  * La función aplica **regexPattern** a la cadena de **source**, y usted puede usar los nombres de grupo regex para construir la cadena para **replacementValue**
* Cuando se proporcionan **regexPattern**, **regexGroupName** y **replacementValue**:
  
  * La función aplica **regexPattern** a la cadena de **source** y reemplaza todos los valores que coinciden con **regexGroupName** por **replacementValue**
* Cuando se proporcionan **regexPattern**, **regexGroupName** y **replacementAttributeName**:
  
  * Si **source** no tiene un valor, se devuelve **source**.
  * Si **source** tiene un valor, la función aplica **regexPattern** a la cadena de **source** y reemplaza todos los valores coincidentes de **regexGroupName** por el valor asociado con **replacementAttributeName**

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto **source**. |
| **oldValue** |Opcional |String |Valor que se va a reemplazar en **source** o **template**. |
| **regexPattern** |Opcional |String |Patrón Regex del valor que se va a reemplazar en **source**. O bien, cuando se usa **replacementPropertyName**, patrón para extraer el valor de la propiedad **replacementPropertyName**. |
| **regexGroupName** |Opcional |String |Nombre del grupo dentro de **regexPattern**. Solo cuando se usa **replacementPropertyName**, extraeremos el valor de este grupo como **replacementValue** de **replacementPropertyName**. |
| **replacementValue** |Opcional |String |Nuevo valor para  reemplazar uno anterior. |
| **replacementAttributeName** |Opcional |String |Nombre del atributo que se usará para el valor de reemplazo |
| **template** |Opcional |String |Cuando se proporcione el valor de **template**, buscaremos **oldValue** dentro de la plantilla y lo reemplazaremos por el valor de **source**. |

### <a name="replace-characters-using-a-regular-expression"></a>Reemplazar los caracteres con una expresión regular
Ejemplo: debe buscar los caracteres que coincidan con un valor de expresión regular y quitarlos.

**Expresión:** 

Replace([mailNickname], , "[a-zA-Z_]*", , "", , )

**Entrada/salida de ejemplo:**

* **INPUT** (mailNickname: "john_doe72"
* **OUTPUT**: "72"


---
### <a name="selectuniquevalue"></a>SelectUniqueValue
**Función:** SelectUniqueValue(uniqueValueRule1, uniqueValueRule2, uniqueValueRule3, …)

**Descripción:** Requiere dos argumentos como mínimo, que son las reglas de generación de valor único definidas con expresiones. La función evalúa cada regla y, a continuación, comprueba la unicidad del valor generado en el directorio o la aplicación de destino. Se devolverá el primer valor único encontrado. Si todos los valores ya existen en el destino, la entrada se depositará y el motivo se anota en los registros de auditoría. No hay ningún límite superior para el número de argumentos que se pueden proporcionar.


 - Esta es una función de nivel superior, no se puede anidar.
 - Esta función no se puede aplicar a los atributos que tienen una precedencia de coincidencia.     
 - Esta función solo está destinada a usarse para creaciones de entradas. Al usarla con un atributo, establezca la propiedad **Apply Mapping** (Aplicar asignación) en **Solo durante la creación del objeto**.
 - Actualmente, esta función solo se admite en el aprovisionamiento de usuarios de Workday en Active Directory y en el de SuccessFactors en Active Directory. No se puede usar con otras aplicaciones de aprovisionamiento. 


**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **uniqueValueRule1  … uniqueValueRuleN** |Al menos se requieren dos, sin límite superior |String | Lista de reglas de generación de valor único para realizar la evaluación. |

### <a name="generate-unique-value-for-userprincipalname-upn-attribute"></a>Generación de un valor único para el atributo userPrincipalName (UPN)
Ejemplo: según el nombre del usuario, el segundo nombre y los apellidos, deberá generar un valor para el atributo UPN y comprobar si es único en el directorio de AD de destino antes de asignarlo al atributo UPN.

**Expresión:** 

```ad-attr-mapping-expr
    SelectUniqueValue( 
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  [PreferredFirstName], [PreferredLastName]))), "contoso.com"), 
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  Mid([PreferredFirstName], 1, 1), [PreferredLastName]))), "contoso.com"),
        Join("@", NormalizeDiacritics(StripSpaces(Join(".",  Mid([PreferredFirstName], 1, 2), [PreferredLastName]))), "contoso.com")
    )
```

**Entrada/salida de ejemplo:**

* **INPUT** (PreferredFirstName): "John"
* **INPUT** (PreferredLastName): "Smith"
* **OUTPUT**: "John.Smith@contoso.com" si el valor de UPN John.Smith@contoso.com no existe aún en el directorio
* **OUTPUT**: "J.Smith@contoso.com" si el valor de UPN John.Smith@contoso.com ya existe en el directorio
* **OUTPUT**: "Jo.Smith@contoso.com" si los dos valores de UPN anteriores ya existen en el directorio



---
### <a name="singleapproleassignment"></a>SingleAppRoleAssignment
**Función:** SingleAppRoleAssignment([appRoleAssignments])

**Descripción:** Devuelve una única función appRoleAssignment de la lista de todas las funciones appRoleAssignments asignadas a un usuario para una aplicación determinada. Esta función es necesaria para convertir el objeto appRoleAssignments en una cadena de nombre de rol único. Tenga en cuenta que el procedimiento recomendado consiste en asegurarse de que solo una función appRoleAssignment esté asignada a un usuario a la vez. Si se asignan varios roles, la cadena del rol devuelta podría no ser predecible. 

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **[appRoleAssignments]** |Obligatorio |String |Objeto **[appRoleAssignments]** . |

---
### <a name="split"></a>Dividir
**Función:** Split(source, delimiter)

**Descripción:** divide una cadena en una matriz de varios valores, usando el carácter delimitador especificado.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |**de origen** que se actualiza. |
| **delimitador** |Obligatorio |String |Especifica el carácter que se usará para dividir la cadena (ejemplo: ","). |

### <a name="split-a-string-into-a-multi-valued-array"></a>Dividir una cadena en una matriz con varios valores
Ejemplo: debe tomar una lista de cadenas delimitadas con comas y dividir esas cadenas en una matriz que se pueda incluir en un atributo de varios valores, como el atributo PermissionSets de Salesforce. En este ejemplo, una lista de conjuntos de permisos se ha rellenado en extensionAttribute5 en Azure AD.

**Expresión:** Split([extensionAttribute5], ",")

**Entrada/salida de ejemplo:** 

* **INPUT** (extensionAttribute5):
* **OUTPUT**:  ["PermissionSetOne", "PermissionSetTwo"]


---
### <a name="stripspaces"></a>StripSpaces
**Función:**  StripSpaces(source)

**Descripción:**  quita todos los caracteres de espacio (" ") de la cadena de origen.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |**de origen** que se actualiza. |

---
### <a name="switch"></a>Switch
**Función:**  Switch(source, defaultValue, key1, value1, key2, value2, …)

**Descripción:** Cuando el valor de **source** coincide con una **key**, devuelve el **value** de dicha **key**. Si el valor de **source** no coincide con ninguna clave, devuelve **defaultValue**.  Los parámetros **key** y **value** siempre deben estar emparejados. La función espera siempre un número par de parámetros. La función no se debe usar para atributos referenciales, como manager. 

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |**Source** que se actualiza. |
| **defaultValue** |Opcional |String |Valor predeterminado que se usará si el origen no coincide con ninguna clave. Puede tratarse de una cadena vacía (""). |
| **key** |Obligatorio |String |**Key** con que se compara el valor de **source**. |
| **value** |Obligatorio |String |Valor de reemplazo para el **source** que coincide con la clave. |

### <a name="replace-a-value-based-on-predefined-set-of-options"></a>Reemplazar un valor basado en un conjunto predefinido de opciones
Ejemplo: debe definir la zona horaria del usuario según el código de estado almacenado en Azure AD. Si el código de estado no coincide con ninguna de las opciones predefinidas, use el valor predeterminado de "Australia/Sídney".

**Expresión:**  
`Switch([state], "Australia/Sydney", "NSW", "Australia/Sydney","QLD", "Australia/Brisbane", "SA", "Australia/Adelaide")`

**Entrada/salida de ejemplo:**

* **INPUT** (state): "QLD"
* **OUTPUT**: "Australia/Brisbane"


---
### <a name="tolower"></a>ToLower
**Función:** ToLower (origen, referencia cultural)

**Descripción:** Toma un valor de cadena de *origen* y se convierte a minúsculas mediante las reglas de referencia cultural que se hayan especificado. Si no hay ninguna información de *referencia cultural* especificada, se usará la referencia cultural invariable.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen |
| **referencia cultural** |Opcional |String |El formato para el nombre de la referencia cultural según RFC 4646 es *languagecode2-country/regioncode2*, donde *languagecode2* es el código de idioma de dos letras y *country/regioncode2* es el código de referencia de subcultura de dos letras. Algunos ejemplos son a ja-JP para japonés (Japón) y en-US para inglés (Estados Unidos). En casos donde un código de idioma de dos letras no está disponible, se usa un código de tres letras derivado de ISO 639-2.|

### <a name="convert-generated-userprincipalname-upn-value-to-lower-case"></a>Conversión del valor generado de userPrincipalName (UPN) a minúsculas
Ejemplo: desea generar el valor de UPN concatenando los campos de origen PreferredFirstName y PreferredLastName, y convirtiendo todos los caracteres a minúsculas. 

`ToLower(Join("@", NormalizeDiacritics(StripSpaces(Join(".",  [PreferredFirstName], [PreferredLastName]))), "contoso.com"))`

**Entrada/salida de ejemplo:**

* **INPUT** (PreferredFirstName): "John"
* **INPUT** (PreferredLastName): "Smith"
* **OUTPUT**: "john.smith@contoso.com"


---
### <a name="toupper"></a>ToUpper
**Función:** ToUpper (origen, referencia cultural)

**Descripción:** Toma un valor de cadena de *origen* y se convierte a mayúsculas mediante las reglas de referencia cultural que se hayan especificado. Si no hay ninguna información de *referencia cultural* especificada, se usará la referencia cultural invariable.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **de origen** |Obligatorio |String |Normalmente el nombre del atributo del objeto de origen. |
| **referencia cultural** |Opcional |String |El formato para el nombre de la referencia cultural según RFC 4646 es *languagecode2-country/regioncode2*, donde *languagecode2* es el código de idioma de dos letras y *country/regioncode2* es el código de referencia de subcultura de dos letras. Algunos ejemplos son a ja-JP para japonés (Japón) y en-US para inglés (Estados Unidos). En casos donde un código de idioma de dos letras no está disponible, se usa un código de tres letras derivado de ISO 639-2.|

---
### <a name="word"></a>Word
**Función:** Word(String,WordNumber,Delimiters)

**Descripción:** : la función Word devuelve una palabra incluida en una cadena, según los parámetros que describen los delimitadores que se usarán y el número de palabras que se devolverán. : cada cadena de caracteres en la cadena separada por uno de los caracteres en los delimitadores se identifica como palabras:

Si el número es < 1, se devuelve una cadena vacía.
Si la cadena es Null, se devuelve una cadena vacía.
Si la cadena contiene menos palabras o si la cadena no contiene palabras identificadas por los delimitadores, se devuelve una cadena vacía.

**Parámetros:** 

| Nombre | Obligatorio/Repetición | Tipo | Notas |
| --- | --- | --- | --- |
| **String** |Obligatorio |Atributo multivalor |Cadena de la que se va a devolver una palabra.|
| **WordNumber** |Obligatorio | Entero | Número que identifica qué número de palabras debe devolver.|
| **delimiters** |Obligatorio |String| Cadena que representa los delimitadores que deben usarse para identificar palabras.|

**Ejemplo:** 
`Word("The quick brown fox",3," ")`

Devuelve "brown".

`Word("This,string!has&many separators",3,",!&#")`

Devuelve "has".

---

## <a name="examples"></a>Ejemplos
En esta sección se proporcionan más ejemplos de uso de las funciones de expresión. 

### <a name="strip-known-domain-name"></a>Seccionar un nombre de dominio conocido
Debe seccionar un nombre de dominio conocido del correo electrónico de un usuario para obtener el nombre de usuario. Por ejemplo, si el dominio es "contoso.com", puede usar la expresión siguiente:

**Expresión:**  
`Replace([mail], "@contoso.com", , ,"", ,)`

**Ejemplo de entrada y salida:** 

* **ENTRADA** (mail): "john.doe@contoso.com"
* **SALIDA**: "john.doe"


### <a name="generate-user-alias-by-concatenating-parts-of-first-and-last-name"></a>Generar el alias de usuario concatenando partes de nombre y apellidos
Debe generar un alias de usuario con las tres primeras letras del nombre del usuario y las cinco primeras letras del apellido del usuario.

**Expresión:**  
`Append(Mid([givenName], 1, 3), Mid([surname], 1, 5))`

**Entrada/salida de ejemplo:** 

* **INPUT** (givenName): "John"
* **INPUT** (surname): "Doe"
* **OUTPUT**:  "JohnDoe"


## <a name="related-articles"></a>Artículos relacionados
* [Automatización del aprovisionamiento y desaprovisionamiento de usuarios para aplicaciones SaaS con Azure Active Directory](../app-provisioning/user-provisioning.md)
* [Personalización de asignaciones de atributos para el aprovisionamiento de usuarios](../app-provisioning/customize-application-attributes.md)
* [Filtros de ámbito para el aprovisionamiento de usuario](define-conditional-rules-for-provisioning-user-accounts.md)
* [Uso de SCIM para habilitar el aprovisionamiento automático de usuarios y grupos de Azure Active Directory a aplicaciones](../app-provisioning/use-scim-to-provision-users-and-groups.md)
* [Notificaciones de aprovisionamiento de cuentas](../app-provisioning/user-provisioning.md)
* [Lista de tutoriales sobre cómo integrar aplicaciones SaaS](../saas-apps/tutorial-list.md)
