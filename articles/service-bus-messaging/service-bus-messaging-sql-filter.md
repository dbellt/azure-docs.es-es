---
title: Sintaxis de filtros de SQL de regla de suscripción de Azure Service Bus | Microsoft Docs
description: En este artículo se ofrece información sobre la gramática de filtros de SQL. Un filtro de SQL admite un subconjunto del estándar SQL-92.
ms.topic: article
ms.date: 04/30/2021
ms.openlocfilehash: 6b8190cf2a57b47fdce416fbe087fa8fa0485bda
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743244"
---
# <a name="subscription-rule-sql-filter-syntax"></a>Sintaxis de filtros de SQL de regla de suscripción

Un *filtro de SQL* es uno de los tipos de filtro disponibles para las suscripciones a tema de Service Bus. Es una expresión de texto que depende de un subconjunto del estándar SQL-92. Las expresiones de filtro se usan con el elemento `sqlExpression` de la propiedad "sqlFilter" de una `Rule` de Service Bus en una [plantilla de Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md), o con el argumento [`--filter-sql-expression`](/cli/azure/servicebus/topic/subscription/rule#az_servicebus_topic_subscription_rule_create) del comando `az servicebus topic subscription rule create` de la CLI de Azure y varias funciones de SDK que permiten administrar reglas de suscripción.

Service Bus Premium también es compatible con la [sintaxis de selector de mensajes SQL de JMS](https://docs.oracle.com/javaee/7/api/javax/jms/Message.html) a través de la API 2.0 de JMS.

  
``` 
<predicate ::=  
      { NOT <predicate> }  
      | <predicate> AND <predicate>  
      | <predicate> OR <predicate>  
      | <expression> { = | <> | != | > | >= | < | <= } <expression>  
      | <property> IS [NOT] NULL  
      | <expression> [NOT] IN ( <expression> [, ...n] )  
      | <expression> [NOT] LIKE <pattern> [ESCAPE <escape_char>]  
      | EXISTS ( <property> )  
      | ( <predicate> )  
  
```  
  
```  
<expression> ::=  
      <constant>   
      | <function>  
      | <property>  
      | <expression> { + | - | * | / | % } <expression>  
      | { + | - } <expression>  
      | ( <expression> )  
  
```  
  
```  
<property> :=   
       [<scope> .] <property_name>  
  
```  
  
## <a name="arguments"></a>Argumentos  
  
-   `<scope>` es una cadena opcional que indica el ámbito de `<property_name>`. Los valores válidos son `sys` y `user`. 
    - El valor `sys` indica el ámbito del sistema, donde `<property_name>` es cualquiera de las propiedades del mensaje de Service Bus, como se describe en [Mensajes, cargas y serialización](service-bus-messages-payloads.md).
    - El valor `user` indica el ámbito de usuario, donde `<property_name>` es una clave de las propiedades personalizadas que puede establecer en el mensaje al enviarlo a Service Bus.
    - El ámbito `user` es el predeterminado si no se especifica `<scope>`.  
  
## <a name="remarks"></a>Comentarios

Un intento de acceso a una propiedad de sistema que no existe es un error, mientras que uno a una propiedad de usuario inexistente, no lo es. En su lugar, una propiedad de usuario inexistente internamente se evalúa como un valor desconocido. Un valor desconocido se trata de una forma especial durante la evaluación de operador.  
  
## <a name="property_name"></a>property_name  
  
```  
<property_name> ::=  
     <identifier>  
     | <delimited_identifier>  
  
<identifier> ::=  
     <regular_identifier> | <quoted_identifier> | <delimited_identifier>  
  
```  
  
### <a name="arguments"></a>Argumentos  

 `<regular_identifier>` es una cadena que se representa mediante la siguiente expresión regular:  
  
```  
[[:IsLetter:]][_[:IsLetter:][:IsDigit:]]*  
```  
  
Esta gramática significa cualquier cadena que empiece por una letra y vaya seguida de uno o varios dígitos, letras o guiones bajo.  
  
`[:IsLetter:]` significa cualquier carácter Unicode que se clasifica como una letra Unicode. `System.Char.IsLetter(c)` devuelve `true` si `c` es una letra Unicode.  
  
`[:IsDigit:]` significa cualquier carácter Unicode que se clasifica como un dígito Unicode. `System.Char.IsDigit(c)` devuelve `true` si `c` es un dígito Unicode.  
  
`<regular_identifier>` no puede ser una palabra clave reservada.  
  
`<delimited_identifier>` es cualquier cadena que se incluye con corchetes izquierdos y derechos ([]). Un corchete derecho se representan como dos corchetes derechos. A continuación, se muestran ejemplos de `<delimited_identifier>`:  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
`<quoted_identifier>` es cualquier cadena que se incluye entre comillas dobles. Las comillas dobles en el identificador se representan como dos comillas dobles. No se recomienda usar identificadores entre comillas, ya que pueden confundirse fácilmente con una constante de cadena. Si es posible, utilice un identificador delimitado. Este es un ejemplo de `<quoted_identifier>`:  
  
```  
"Contoso & Northwind"  
```  
  
## <a name="pattern"></a>pattern  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Comentarios
  
`<pattern>` debe ser una expresión que se evalúa como una cadena. Se usa como patrón para el operador LIKE.      Puede contener los siguientes caracteres comodín:  
  
-   `%`: cualquier cadena de cero o más caracteres.  
  
-   `_`: cualquier carácter individual.  
  
## <a name="escape_char"></a>escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Comentarios  

`<escape_char>` debe ser una expresión que se evalúa como una cadena de longitud 1. Se usa como carácter de escape para el operador LIKE.  
  
 Por ejemplo, `property LIKE 'ABC\%' ESCAPE '\'` coincide con `ABC%`, en lugar de con una cadena que comienza con `ABC`.  
  
## <a name="constant"></a>constant  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### <a name="arguments"></a>Argumentos  
  
-   `<integer_constant>` es una cadena de números que no se incluye entre comillas y no contiene decimales. Los valores se almacenan como `System.Int64` internamente y siguen el mismo intervalo.  
  
     Los siguientes son ejemplos de constantes largas:  
  
    ```  
    1894  
    2  
    ```  
  
-   `<decimal_constant>` es una cadena de números que no se incluye entre comillas y contiene un separador decimal. Los valores se almacenan como `System.Double` internamente y siguen el mismo intervalo o la misma precisión.  
  
     En una versión futura, este número podría almacenarse en un tipo de datos diferente para admitir la semántica de número exacto. Por lo tanto, no debe confiar en el hecho de que el tipo de datos subyacente es `System.Double` en `<decimal_constant>`.  
  
     A continuación, se muestran ejemplos de constantes decimales:  
  
    ```  
    1894.1204  
    2.0  
    ```  
  
-   `<approximate_number_constant>` es un número escrito en la notación científica. Los valores se almacenan como `System.Double` internamente y siguen el mismo intervalo o la misma precisión. A continuación, se muestran ejemplos de constantes de número aproximado:  
  
    ```  
    101.5E5  
    0.5E-2  
    ```  
  
## <a name="boolean_constant"></a>boolean_constant  
  
```  
<boolean_constant> :=  
      TRUE | FALSE  
```  
  
### <a name="remarks"></a>Comentarios  

Las constantes booleanas se representan mediante las palabras clave **TRUE** o **FALSE**. Los valores se almacenan como `System.Boolean`.  
  
## <a name="string_constant"></a>string_constant  
  
```  
<string_constant>  
```  
  
### <a name="remarks"></a>Comentarios  

Las constantes de cadena se incluyen entre comillas simples y contienen caracteres Unicode válidos. Una comilla simple incrustada en una constante de cadena se representan como dos comillas simples.  
  
## <a name="function"></a>function  
  
```  
<function> :=  
      newid() |  
      property(name) | p(name)  
```  
  
### <a name="remarks"></a>Comentarios
  
La función `newid()` devuelve un elemento `System.Guid` generado por el método `System.Guid.NewGuid()`.  
  
La función `property(name)` devuelve el valor de la propiedad a la que hace referencia `name`. El valor `name` puede ser cualquier expresión válida que devuelve un valor de cadena.  
  
## <a name="considerations"></a>Consideraciones
  
Tenga en cuenta la siguiente semántica de Sql Filter:  
  
-   Los nombres de propiedad distinguen entre mayúsculas y minúsculas.  
  
-   Los operadores siguen la semántica de conversión implícita de C# siempre que sea posible.  
  
-   Las propiedades del sistema son cualquiera de las propiedades del mensaje de Service Bus, como se describe en [Mensajes, cargas y serialización](service-bus-messages-payloads.md).
  
    Tenga en cuenta la siguiente semántica de `IS [NOT] NULL`:  
  
    -   `property IS NULL` se evalúa como `true` si no existe la propiedad o valor de la propiedad es `null`.  
  
### <a name="property-evaluation-semantics"></a>Semántica de evaluación de propiedades  
  
- Un intento de evaluar una propiedad no existente del sistema genera una excepción `FilterException`.  
  
- Una propiedad que no existe se evalúa internamente como **valor desconocido**.  
  
  Evaluación desconocida en operadores aritméticos:  
  
- Para los operadores binarios, si la parte izquierda o derecha de los operandos se evalúa como **valor desconocido**, el resultado será **desconocido**.  
  
- Para los operadores unarios, si un operando se evalúa como **desconocido**, el resultado será **desconocido**.  
  
  Evaluación desconocida en los operadores de comparación binaria:  
  
- Si la parte izquierda o derecha de los operandos se evalúa como **valor desconocido**, el resultado será **desconocido**.  
  
  Evaluación desconocida en `[NOT] LIKE`:  
  
- Si cualquier operando se evalúa como **desconocido**, el resultado es **desconocido**.  
  
  Evaluación desconocida en `[NOT] IN`:  
  
- Si el operando izquierdo se evalúa como **desconocido**, el resultado es **desconocido**.  
  
  Evaluación desconocida en el operador **AND**:  
  
```  
+---+---+---+---+  
|AND| T | F | U |  
+---+---+---+---+  
| T | T | F | U |  
+---+---+---+---+  
| F | F | F | F |  
+---+---+---+---+  
| U | U | F | U |  
+---+---+---+---+  
```  
  
 Evaluación desconocida en el operador **OR**:  
  
```  
+---+---+---+---+  
|OR | T | F | U |  
+---+---+---+---+  
| T | T | T | T |  
+---+---+---+---+  
| F | T | F | U |  
+---+---+---+---+  
| U | T | U | U |  
+---+---+---+---+  
```  
  
### <a name="operator-binding-semantics"></a>Semántica de enlace de operadores
  
-   Los operadores de comparación, como `>`, `>=`, `<`, `<=`, `!=` y `=`, siguen la misma semántica que el enlace de operadores de C# en promociones de tipo de datos y conversiones implícitas.  
  
-   Los operadores aritméticos, como `+`, `-`, `*`, `/` y `%`, siguen la misma semántica que el enlace de operadores de C# en promociones de tipo de datos y conversiones implícitas.

## <a name="examples"></a>Ejemplos
Para obtener ejemplos, consulte [Ejemplos de filtros de Service Bus](service-bus-filter-examples.md).

## <a name="next-steps"></a>Pasos siguientes

- [Clase SQLFilter (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlfilter)
- [Clase SQLFilter (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlfilter)
- [Clase SqlFilter (Java)](/java/api/com.microsoft.azure.servicebus.rules.SqlFilter)
- [SqlRuleFilter (JavaScript)](/javascript/api/@azure/service-bus/sqlrulefilter)
- [`az servicebus topic subscription rule`](/cli/azure/servicebus/topic/subscription/rule)
- [New-AzServiceBusRule](/powershell/module/az.servicebus/new-azservicebusrule)
