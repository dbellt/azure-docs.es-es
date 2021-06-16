---
title: Cree directivas del Firewall de aplicaciones web (WAF) para Application Gateway
description: Más información sobre cómo crear directivas de Firewall de aplicaciones web para Application Gateway.
services: web-application-firewall
ms.topic: conceptual
author: vhorne
ms.service: web-application-firewall
ms.date: 02/08/2020
ms.author: victorh
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 478c2cbe35c43ffb0fdc507cc1b00585623013ea
ms.sourcegitcommit: df574710c692ba21b0467e3efeff9415d336a7e1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/28/2021
ms.locfileid: "110668548"
---
# <a name="create-web-application-firewall-policies-for-application-gateway"></a>Cree directivas del Firewall de aplicaciones web para Application Gateway

La Asociación de una directiva WAF con agentes de escucha permite la protección de varios sitios detrás de un único WAF con directivas diferentes. Por ejemplo, si hay cinco sitios detrás de su WAF, puede tener cinco directivas de WAF independientes (una para cada cliente de escucha) para personalizar las exclusiones, las reglas personalizadas y los conjuntos de reglas administradas de un sitio sin tener en cuenta los otros cuatro. Si desea que una sola directiva se aplique a todos los sitios, puede asociar simplemente la directiva con el Application Gateway, en lugar de los agentes de escucha individuales, para que se aplique globalmente. Las directivas también se pueden aplicar a una regla de enrutamiento basada en rutas de acceso. 

Puede hacer tantas directivas como desee. Una vez que se crea una directiva, se debe asociar a un Application Gateway para entrar en vigor, pero se puede asociar a cualquier combinación de Application Gateways y agentes de escucha. 

Si el Application Gateway tiene una directiva aplicada y después aplica una directiva diferente a un cliente de escucha en ese Application Gateway, la directiva del cliente de escucha surtirá efecto, pero solo para los clientes de escucha a los que están asignados. La directiva de Application Gateway se sigue aplicando a todos los demás agentes de escucha que no tienen una directiva específica asignada. 

   > [!NOTE]
   > Una vez que una directiva de firewall se asocia a un WAF, siempre debe haber una directiva asociada a ese WAF. Puede sobrescribir esa directiva, pero no se admite la desasociación completa de una directiva del WAF. 

Toda la nueva configuración de WAF del Firewall de aplicaciones web (reglas personalizadas, configuraciones de rulset administradas, exclusiones, etc.) se encuentra dentro de una Directiva de WAF. Si tiene un WAF existente, es posible que esta configuración siga existiendo en la configuración de WAF. Para conocer los pasos para cambiar a la nueva Directiva de WAF, consulte [Migrar la configuración de WAF a una directiva de WAF](#migrate) más adelante en este artículo. 

## <a name="create-a-policy"></a>Crear una directiva

En primer lugar, cree una directiva básica de WAF con un conjunto de reglas predeterminado (DRS) administrado con Azure Portal.

1. En la parte superior izquierda del portal, seleccione **Crear un recurso**. Busque **WAF**, seleccione **Firewall de aplicaciones web**, después seleccione **Crear**.
2. En página de **Crear directiva de WAF**, pestaña **Basics**, escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y, luego, seleccione **Revisar + crear**:

   |Configuración  |Value  |
   |---------|---------|
   |Directiva de     |WAF regional (Application Gateway)|
   |Subscription     |Seleccione el nombre de la suscripción|
   |Resource group     |Seleccionar el grupo de recursos|
   |Nombre de la directiva     |Escriba un nombre único para la directiva WAF.|
3. En la pestaña de **Asociación**, especifique una de las siguientes opciones y, después, seleccione **Agregar**:

   |Configuración  |Value  |
   |---------|---------|
   |Asociar Application Gateway     |Seleccione el nombre del perfil de Application Gateway.|
   |Agentes de escucha asociados     |Seleccione el nombre del agente de escucha de Application Gateway y, luego, seleccione **Agregar**.|

   > [!NOTE]
   > Si asigna una directiva a su Application Gateway (o agente de escucha) que ya tiene una directiva, la directiva original se sobrescribe y se reemplaza por la nueva directiva.
4. Seleccione **Revisar y crear** y, luego, **Crear**.

   ![Aspectos básicos de la Directiva de WAF](../media/create-waf-policy-ag/waf-policy-basics.png)

## <a name="configure-waf-rules-optional"></a>Configuración de reglas de WAF (opcional)

Cuando se crea una directiva WAF, de forma predeterminada, esta está en modo *Detección*. En el Modo de detección, WAF no bloquea ninguna solicitud. En su lugar, las reglas de WAF coincidentes se registran en los registros de WAF. Para ver WAF en acción, puede cambiar la configuración del modo de *Prevención*. En Modo de prevención, las reglas de coincidencia definidas en el conjunto de reglas de CRS que seleccionó se bloquean o registran en los registros de WAF.

## <a name="managed-rules"></a>Reglas administradas

Las reglas de OWASP administradas por Azure están habilitadas de forma predeterminada. Para deshabilitar una regla individual dentro de un grupo de reglas, expanda las reglas dentro de ese grupo de reglas, active la casilla delante del número de regla y seleccione **Deshabilitar** en la pestaña anterior.

[ ![Reglas administradas](../media/create-waf-policy-ag/managed-rules.png) ](../media/create-waf-policy-ag/managed-rules-lrg.png#lightbox)

## <a name="custom-rules"></a>Reglas personalizadas

Para crear una regla personalizada, seleccione **Agregar regla personalizada** debajo de la pestaña **Reglas personalizadas**. Se abrirá la página Configuración de reglas personalizadas. En la captura de pantalla siguiente se muestra una regla personalizada de ejemplo configurada para bloquear una solicitud si la cadena de consulta contiene el texto *blockme*.

[ ![Editar regla personalizada](../media/create-waf-policy-ag/edit-custom-rule.png) ](../media/create-waf-policy-ag/edit-custom-rule-lrg.png#lightbox)

## <a name="migrate-your-waf-config-to-a-waf-policy"></a><a name="migrate"></a>Migrar la configuración de WAF a una directiva de WAF

Si tiene un WAF existente, es posible que haya observado algunos cambios en el portal. En primer lugar, debe identificar el tipo de Directiva que ha habilitado en el WAF. Hay tres estados posibles:

- Sin directiva WAF
- Directiva de solo reglas personalizadas
- Directiva WAF

Puede saber en qué estado se encuentra el WAF al verlo el portal. Si los valores de WAF son visibles y se pueden cambiar desde la vista de Application Gateway, el valor de WAF se encuentra en el estado 1.

[ ![Configuración de WAF](../media/create-waf-policy-ag/waf-configure.png) ](../media/create-waf-policy-ag/waf-configure-lrg.png#lightbox)

Si selecciona **Firewall de aplicaciones web** y muestra una directiva asociada, el WAF se encuentra en el estado 2 o el estado 3. Después de desplazarse a la Directiva, si **solo** muestra reglas personalizadas y las Application Gateways asociadas, se trata de una Directiva solo de Reglas personalizadas.

![Reglas personalizadas del firewall de aplicaciones web](../media/create-waf-policy-ag/waf-custom-rules.png)

Si también muestra la Configuración de directiva y las Reglas administradas, es una directiva de Firewall de aplicación web completa. 

![Configuración de directiva de WAF](../media/create-waf-policy-ag/waf-policy-settings.png)

## <a name="migrate-to-waf-policy"></a>Migrar a la Directiva WAF

Si solo tiene una Política WAF de Reglas personalizadas, entonces es posible que quiera cambiar a la nueva Política WAF. En el futuro, la directiva de Firewall admitirá la configuración de directiva de WAF, los conjuntos de reglas administrados, las exclusiones y los grupos de reglas deshabilitados. En esencia, todas las configuraciones de WAF que se realizaron previamente en el Application Gateway se realizan ahora a través de la Directiva WAF. 

Las modificaciones en la regla personalizada solo se deshabilitan la Directiva WAF. Para editar cualquier configuración de WAF, como deshabilitar reglas, agregar exclusiones, etc., debe migrar a un nuevo recurso de directiva de Firewall de nivel superior.

Para ello, cree una *directiva de Firewall de aplicación Web* y asócielo a sus Application Gateway(s) y agente(s) de escucha que desee. Esta nueva directiva debe ser exactamente la misma que la configuración actual de WAF, lo que significa que cada regla personalizada, exclusión, regla deshabilitada, etc., debe copiarse en la nueva directiva que está creando. Una vez que tenga una Directiva asociada a su Application Gateway, podrá seguir realizando cambios en las reglas y la configuración de WAF. También puede hacer esto en Azure PowerShell. Para más información, consulte [Asociar una directiva de WAF a una Application Gateway existente](associate-waf-policy-existing-gateway.md).

Opcionalmente, puede usar un script de migración para migrar a una directiva de WAF. Para obtener más información, consulte [Migración de directivas de firewall de aplicaciones web mediante Azure PowerShell](migrate-policy.md).

## <a name="force-mode"></a>Modo de forzado

Si no desea copiar todo en una directiva que sea exactamente igual a la configuración actual, puede establecer WAF en el modo de "forzado". Ejecute el siguiente código de Azure PowerShell y WAF estará en modo de forzado. Después, puede asociar cualquier directiva de WAF a su WAF, aunque no tenga exactamente la misma configuración que su configuración. 

```azurepowershell-interactive
$appgw = Get-AzApplicationGateway -Name <your Application Gateway name> -ResourceGroupName <your Resource Group name>
$appgw.ForceFirewallPolicyAssociation = $true
```

A continuación, siga con los pasos para asociar una directiva de WAF a su puerta de enlace de aplicaciones. Para obtener más información, consulte [Asociar una directiva de WAF de Application Gateway existente](associate-waf-policy-existing-gateway.md).

## <a name="next-steps"></a>Pasos siguientes

Más información sobre las reglas y los grupos de reglas de [Firewall de aplicaciones web (CRS)](application-gateway-crs-rulegroups-rules.md).
