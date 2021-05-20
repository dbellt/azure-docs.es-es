---
title: Sintaxis de acciones de SQL de regla de suscripción de Azure Service Bus | Microsoft Docs
description: En este artículo se ofrece una referencia para la sintaxis de acciones de reglas SQL. Las acciones se escriben con una sintaxis basada en lenguaje SQL que se realiza en un mensaje.
ms.topic: article
ms.date: 11/24/2020
ms.openlocfilehash: 7aceb1992aafbbf92718a6bf558a72060f3275df
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108770090"
---
# <a name="subscription-rule-sql-action-syntax"></a>Sintaxis de acciones de SQL de regla de suscripción

Una *acción de SQL* se usa para manipular metadatos de mensajes después de que un filtro de una regla de suscripción haya seleccionado un mensaje. Es una expresión de texto que depende de un subconjunto del estándar SQL-92. Las expresiones de acción se usan con el elemento `sqlExpression` de la propiedad "action" de una `Rule` de Service Bus en una [plantilla de Resource Manager](service-bus-resource-manager-namespace-topic-with-rule.md), o con el argumento [`--action-sql-expression`](/cli/azure/servicebus/topic/subscription/rule#az_servicebus_topic_subscription_rule_create) del comando `az servicebus topic subscription rule create` de la CLI de Azure y varias funciones de SDK que permiten administrar reglas de suscripción.
  
  
```  
<statements> ::=
    <statement> [, ...n]  
  
```  
  
```  
<statement> ::=
    <action> [;]
    Remarks
    -------
    Semicolon is optional.  
  
```  
  
```  
<action> ::=
    SET <property> = <expression>
    REMOVE <property>  
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
  
### <a name="remarks"></a>Comentarios  

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
  
 Es decir, cualquier cadena que empieza por una letra y va seguida de uno o varios dígitos, letras o guiones bajo.  
  
 `[:IsLetter:]` significa cualquier carácter Unicode que se clasifica como una letra Unicode. `System.Char.IsLetter(c)` devuelve `true` si `c` es una letra Unicode.  
  
 `[:IsDigit:]` significa cualquier carácter Unicode que se clasifica como un dígito Unicode. `System.Char.IsDigit(c)` devuelve `true` si `c` es un dígito Unicode.  
  
 `<regular_identifier>` no puede ser una palabra clave reservada.  
  
 `<delimited_identifier>` es cualquier cadena que se incluye con corchetes izquierdos y derechos ([]). Un corchete derecho se representan como dos corchetes derechos. A continuación, se muestran ejemplos de `<delimited_identifier>`:  
  
```  
[Property With Space]  
[HR-EmployeeID]  
  
```  
  
 `<quoted_identifier>` es cualquier cadena que se incluye entre comillas dobles. Las comillas dobles en el identificador se representan como dos comillas dobles. No se recomienda usar identificadores entre comillas, ya que pueden confundirse fácilmente con una constante de cadena. Si es posible, utilice un identificador delimitado. A continuación, se muestra un ejemplo de `<quoted_identifier>`:  
  
```  
"Contoso & Northwind"  
```  
  
## <a name="pattern"></a>pattern  
  
```  
<pattern> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Comentarios
  
 `<pattern>` debe ser una expresión que se evalúa como una cadena. Se utiliza como patrón para el operador LIKE.      Puede contener los siguientes caracteres comodín:  
  
-   `%`: cualquier cadena de cero o más caracteres.  
  
-   `_`: cualquier carácter individual.  
  
## <a name="escape_char"></a>escape_char  
  
```  
<escape_char> ::=  
      <expression>  
```  
  
### <a name="remarks"></a>Comentarios
  
 `<escape_char>` debe ser una expresión que se evalúa como una cadena de longitud 1. Se utiliza como carácter de escape para el operador LIKE.  
  
 Por ejemplo, `property LIKE 'ABC\%' ESCAPE '\'` coincide con `ABC%`, en lugar de con una cadena que comienza con `ABC`.  
  
## <a name="constant"></a>constant  
  
```  
<constant> ::=  
      <integer_constant> | <decimal_constant> | <approximate_number_constant> | <boolean_constant> | NULL  
```  
  
### <a name="arguments"></a>Argumentos  
  
-   `<integer_constant>` es una cadena de números que no se incluye entre comillas y no contiene decimales. Los valores se almacenan como `System.Int64` internamente y siguen el mismo intervalo.  
  
     A continuación, se muestran ejemplos de constantes largas:  
  
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
  
Las constantes booleanas se representan mediante las palabras clave `TRUE` o `FALSE`. Los valores se almacenan como `System.Boolean`.  
  
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

## <a name="examples"></a>Ejemplos
Para obtener ejemplos, consulte [Ejemplos de filtros de Service Bus](service-bus-filter-examples.md).
  
## <a name="considerations"></a>Consideraciones

- SET se usa para crear una nueva propiedad o actualizar el valor de una propiedad existente.
- REMOVE se utiliza para quitar una propiedad.
- SET realiza la conversión implícita si es posible cuando el tipo de expresión y el tipo de propiedad existentes son diferentes.
- La acción no se realiza correctamente si se hace referencia a propiedades del sistema inexistentes.
- La acción no se realiza correctamente si se hace referencia a propiedades del usuario inexistentes.
- Una propiedad de usuario inexistente se evalúa como desconocida internamente siguiendo la misma semántica que [SQLFilter](/dotnet/api/microsoft.servicebus.messaging.sqlfilter) al evaluar los operadores.

## <a name="next-steps"></a>Pasos siguientes

- [Clase SQLRuleAction (.NET Framework)](/dotnet/api/microsoft.servicebus.messaging.sqlruleaction)
- [Clase SQLRuleAction (.NET Standard)](/dotnet/api/microsoft.azure.servicebus.sqlruleaction)
- [Clase SqlRuleAction (Java)](/java/api/com.microsoft.azure.servicebus.rules.sqlruleaction)
- [SqlRuleAction (JavaScript)](/javascript/api/@azure/service-bus/sqlruleaction)
- [`az servicebus topic subscription rule`](/cli/azure/servicebus/topic/subscription/rule)
- [New-AzServiceBusRule](/powershell/module/az.servicebus/new-azservicebusrule)
