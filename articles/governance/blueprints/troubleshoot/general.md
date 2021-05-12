---
title: Solución de errores comunes
description: Obtenga información acerca de cómo solucionar problemas al crear, asignar y eliminar planos técnicos, como infracciones de directivas y funciones de parámetros del plano técnico.
ms.date: 05/01/2021
ms.topic: troubleshooting
ms.openlocfilehash: ca86b81f846465c996d05f08d3a721829f8e496f
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108733902"
---
# <a name="troubleshoot-errors-using-azure-blueprints"></a>Solución de problemas de errores con instancias de Azure Blueprint

Es posible que, al crear, asignar o quitar planos técnicos, se tropiece con errores. En este artículo se describen diversos errores que pueden producirse y cómo resolverlos.

## <a name="finding-error-details"></a>Búsqueda de detalles del error

Muchos errores serán el resultado de asignar un plano técnico a un ámbito. Cuando se produce un error en una asignación, el plano técnico proporciona detalles sobre la implementación errónea. Esta información indica el problema, de modo que se pueda corregir y la siguiente implementación se realice correctamente.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página de la izquierda seleccione **Planos técnicos asignados** y use el cuadro de búsqueda para filtrar las asignaciones de plano técnico a fin de encontrar las asignaciones con error. También puede ordenar la tabla de asignaciones por la columna **Estado de aprovisionamiento** para ver todas las asignaciones con error agrupadas conjuntamente.

1. Seleccione el plano técnico con el estado _Error_ o haga clic con el botón derecho y seleccione **Ver los detalles de la asignación**.

1. En la parte superior de la página de asignación de planos técnicos, un banner rojo de advertencia indica que la asignación dio un error. Seleccione cualquier lugar del banner para obtener más detalles.

Es habitual que el error se deba a un artefacto y no al plano técnico en su conjunto. Si un artefacto crea un almacén de claves y Azure Policy impide la creación, se producirá un error en toda la asignación.

## <a name="general-errors"></a>Errores generales

### <a name="scenario-policy-violation"></a><a name="policy-violation"></a>Escenario: Infracción de la directiva

#### <a name="issue"></a>Problema

Se produjo un error en la implementación de la plantilla debido a la infracción de la directiva.

#### <a name="cause"></a>Causa

Una directiva puede entrar en conflicto con la implementación por los motivos siguientes:

- El recurso que se crea está restringido por la directiva (normalmente restricciones de SKU o ubicación)
- La implementación establece campos que están configurados por la directiva (normalmente con etiquetas)

#### <a name="resolution"></a>Solución

Cambie el plano técnico para que no entre en conflicto con las directivas en los detalles del error. Si este cambio no es posible, una opción alternativa es hacer que el ámbito de la asignación de directivas cambie para que el plano técnico deje de estar en conflicto con la directiva.

### <a name="scenario-blueprint-parameter-is-a-function"></a><a name="escape-function-parameter"></a>Escenario: El parámetro de plano técnico es una función

#### <a name="issue"></a>Problema

Los parámetros de plano técnico que son funciones se procesan antes de que se pasen a los artefactos.

#### <a name="cause"></a>Causa

Pasar un parámetro de plano técnico que usa una función, como `[resourceGroup().tags.myTag]`, a un artefacto genera el resultado procesado de la función que se va a establecer en el artefacto, en lugar de la función dinámica.

#### <a name="resolution"></a>Solución

Para pasar una función como parámetro, escape la cadena completa con `[`, de modo que el parámetro de plano técnico sea similar a `[[resourceGroup().tags.myTag]`. El carácter de escape hace que los planos técnicos traten el valor como una cadena al procesar el plano técnico. El servicio de planos técnicos luego coloca la función en el artefacto para que pueda ser dinámico, según lo previsto. Para obtener más información, consulte [Sintaxis y expresiones de las plantillas de Azure Resource Manager](../../../azure-resource-manager/templates/template-expressions.md).

## <a name="delete-errors"></a>Eliminación de errores

### <a name="scenario-assignment-deletion-timeout"></a><a name="assign-delete-timeout"></a>Escenario: Tiempo de espera de eliminación de asignaciones

#### <a name="issue"></a>Problema

No se ha completado la eliminación de una asignación de plano técnico.

#### <a name="cause"></a>Causa

Una asignación de plano técnico puede quedar atrapada en un estado no terminal cuando se elimina. Este estado se produce cuando los recursos creados por la asignación de plano técnico todavía están pendientes de eliminación o no devuelven un código de estado a Azure Blueprints.

#### <a name="resolution"></a>Solución

Las asignaciones de plano técnico en un estado no terminal se marcan automáticamente como **Error** después de un tiempo de espera de _seis horas_. Una vez que el tiempo de espera ha ajustado el estado de la asignación de plano técnico, se puede volver a intentar la eliminación.

## <a name="next-steps"></a>Pasos siguientes

Si su problema no aparece o es incapaz de resolverlo, visite uno de nuestros canales para obtener ayuda adicional:

- Obtenga respuestas de expertos de Azure en los [foros de Azure](https://azure.microsoft.com/support/forums/).
- Póngase en contacto con [@AzureSupport](https://twitter.com/azuresupport), la cuenta oficial de Microsoft Azure para mejorar la experiencia del cliente, que pone en contacto a la comunidad de Azure con los recursos adecuados: respuestas, soporte técnico y expertos.
- Si necesita más ayuda, puede registrar un incidente de soporte técnico de Azure. Vaya al [sitio de Soporte técnico de Azure](https://azure.microsoft.com/support/options/) y seleccione **Obtener soporte**.
