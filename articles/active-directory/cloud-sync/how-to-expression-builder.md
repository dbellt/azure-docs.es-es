---
title: Empleo del Generador de expresiones con Azure AD Connect Cloud Sync
description: En este artículo se explica cómo usar el Generador de expresiones con la sincronización en la nube.
services: active-directory
author: billmath
manager: mtillman
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 04/19/2021
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: e169c2cbabf9e9e37bb93b1e3ee26080105d501e
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108776944"
---
# <a name="expression-builder-with-cloud-sync"></a>Generador de expresiones con sincronización en la nube
El Generador de expresiones es una nueva hoja de Azure que se encuentra en la sección de sincronización en la nube. Ayuda a compilar expresiones complejas y permite probarlas antes de su aplicación al entorno de sincronización en la nube.

## <a name="use-the-expression-builder"></a>Empleo del Generador de expresiones
Para acceder al Generador de expresiones, siga estos pasos.

 1. En Azure Portal, seleccione **Azure Active Directory**.
 2. Seleccione **Azure AD Connect**.
 3. Seleccione **Manage cloud sync** (Administrar sincronización en la nube).
 4. En **Configuración**, seleccione su configuración.
 5. En **Administrar atributos**, seleccione **Haga clic para editar las asignaciones**.
 6. En la hoja **Editar asignaciones de atributos**, haga clic en **Agregar asignación de atributos**.
 7. En **Tipo de asignación**, seleccione **Expresión**.
 8. Seleccione **Probar Generador de expresiones (versión preliminar)** .
 ![Empleo del Generador de expresiones](media/how-to-expression-builder/expression-1.png)

## <a name="build-an-expression"></a>Compilación de una expresión
Esta sección permite usar el menú desplegable para seleccionar entre una lista de funciones admitidas.  Luego se proporcionan campos adicionales para rellenar según la función seleccionada.  Una vez que se selecciona **Aplicar expresión**, la sintaxis aparece en el cuadro **Entrada de la expresión**.

Por ejemplo, al seleccionar **Replace** en la lista desplegable, se proporcionan cuadros adicionales.  La sintaxis de la función se muestra en el cuadro azul claro.  Los cuadros que se muestran corresponden a la sintaxis de la función seleccionada.  Replace funciona de forma diferente según los parámetros proporcionados.  En este ejemplo se va a usar:

- Cuando se proporcionan valorAnterior y valorDeReemplazo:
    - Reemplaza todas las ocurrencias de oldValue en el origen con replacementValue

Para obtener más información, vea [Replace](reference-expressions.md#replace).

Lo primero que se debe hacer es seleccionar el atributo que constituye el origen de la función replace. En el ejemplo se ha seleccionado el atributo **mail**. 

Luego se rellena el valor de oldValue.  oldValue será **@fabrikam.com** .  Por último, el cuadro replacementValue se rellena con el valor **@contoso.com** .

Así, la expresión básicamente indica: reemplazar el atributo mail en los objetos user que tengan un valor @fabrikam.com por el valor @contoso.com.  Al hacer clic en el botón **Agregar expresión**, es posible ver la sintaxis de la **Entrada de la expresión**.


>[!NOTE]
>Asegúrese de colocar los valores en los cuadros que corresponderían a oldValue y replacementValue en función de la sintaxis que se genera cuando se ha seleccionado Replace.

Para obtener más información sobre las expresiones admitidas, vea [Escritura de expresiones para la asignación de atributos en Azure Active Directory](reference-expressions.md).

### <a name="information-on-expression-builder-input-boxes"></a>Información sobre los cuadros de entrada del Generador de expresiones
Los cuadros proporcionados por el Generador de expresiones aceptan varios valores según la función que se haya seleccionado.  Por ejemplo, la función JOIN acepta cadenas o el valor asociado a un atributo determinado.  Por ejemplo, se puede usar el valor del atributo [givenName] y combinarlo con un valor de cadena "@contoso.com" para crear una dirección de correo electrónico.

  ![Valores del cuadro de entrada](media/how-to-expression-builder/expression-8.png)

Para obtener más información sobre los valores aceptables y sobre cómo escribir expresiones, vea [Escritura de expresiones para la asignación de atributos en Azure Active Directory](reference-expressions.md).

## <a name="test-an-expression"></a>Prueba de una expresión
En esta sección se pueden probar las expresiones.  En el menú desplegable, seleccione el atributo **mail**.  Rellene el valor con **@fabrikam.com** y haga clic en **Prueba de la expresión**.  

Se ve el valor **@contoso.com** en el cuadro **Ver la salida de la expresión**.

 ![Prueba de la expresión](media/how-to-expression-builder/expression-4.png)

## <a name="deploy-the-expression"></a>Implementación de la expresión
Una vez que se está satisfecho con la expresión, solo hay que hacer clic en el botón **Aplicar expresión**.
![Incorporación de la expresión](media/how-to-expression-builder/expression-5.png)

Con esto se agrega la expresión a la configuración del agente.
![Configuración de agente](media/how-to-expression-builder/expression-6.png)

## <a name="setting-a-null-value-on-an-expression"></a>Establecimiento de un valor NULL en una expresión
Para establecer el valor de un atributo en NULL,  se puede usar una expresión con el valor `""`.  Esto envía el valor NULL al atributo de destino.

![Valor NULL](media/how-to-expression-builder/expression-7.png)



## <a name="next-steps"></a>Pasos siguientes 

- [Escritura de expresiones para la asignación de atributos en Azure Active Directory](reference-expressions.md)
- [Configuración de la sincronización en la nube](how-to-configure.md)
