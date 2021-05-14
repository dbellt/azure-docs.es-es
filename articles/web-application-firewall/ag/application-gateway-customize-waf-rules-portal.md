---
title: 'Personalización de reglas mediante el portal: Firewall de aplicaciones Web de Azure'
description: En este artículo se proporciona información acerca de cómo personalizar reglas de Firewall de aplicaciones web en Application Gateway con Azure Portal.
services: web-application-firewall
author: vhorne
ms.service: web-application-firewall
ms.date: 04/21/2021
ms.author: victorh
ms.topic: article
ms.openlocfilehash: 0ab122d178e5390a53e5a3a39f1b7763b298dc6d
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107878333"
---
# <a name="customize-web-application-firewall-rules-using-the-azure-portal"></a>Personalización de reglas de Firewall de aplicaciones web con Azure Portal

El Firewall de aplicaciones web (WAF) de Azure Application Gateway proporciona protección a las aplicaciones web. Dicha protección la proporciona Open Web Application Security Project (OWASP) Core Rule Set (CRS). Algunas reglas pueden producir falsos positivos y bloquear el tráfico real. Por este motivo, Application Gateway ofrece la posibilidad de personalizar reglas y grupos de reglas. Para más información sobre las reglas y grupos de reglas específicos, consulte [Lista de reglas y grupos de reglas de CRS de Firewall de aplicaciones web](application-gateway-crs-rulegroups-rules.md).

>[!NOTE]
> Si la puerta de enlace de aplicaciones no usa el nivel WAF, la opción para actualizar la puerta de enlace de aplicaciones al nivel WAF aparece en el panel derecho. 

:::image type="content" source="../media/application-gateway-customize-waf-rules-portal/1.png" alt-text="Habilitar WAF"::: 

## <a name="view-rule-groups-and-rules"></a>Visualización de reglas y grupos de reglas

**Para ver reglas y grupos de reglas**
1. Navegue hasta la puerta de enlace de aplicaciones y seleccione **Firewall de aplicaciones web**.  
2. Seleccione su **Directiva WAF**.
2. Seleccione **Reglas administradas**.

   Esta vista muestra una tabla en la página de todos los grupos de reglas proporcionados con el conjunto de reglas elegido. Todas las casillas de la regla están seleccionadas.

## <a name="disable-rule-groups-and-rules"></a>Deshabilitación de reglas y grupos de reglas

> [!IMPORTANT]
> Tenga cuidado al deshabilitar los grupos de reglas o reglas. Esta acción puede exponerlo a mayores riesgos de seguridad.

**Para deshabilitar grupos de reglas o reglas concretas**

   1. Busque las reglas o los grupos de reglas que desea deshabilitar.
   2. Active las casillas de las reglas que quiere deshabilitar. 
   3. Seleccione la acción que se encuentra en la parte superior de la página (habilitar o deshabilitar) para las reglas seleccionadas.
   2. Seleccione **Guardar**.
    :::image type="content" source="../media/application-gateway-customize-waf-rules-portal/figure3.png" alt-text="Almacenamiento de reglas deshabilitadas"::: 

## <a name="mandatory-rules"></a>Reglas obligatorias

La lista siguiente contiene las condiciones que provocan que el firewall de aplicaciones web bloquee la solicitud durante el modo de prevención. En modo de detección, se registran como excepciones.

Estas condiciones no se pueden configurar o deshabilitar:

* La imposibilidad de analizar el cuerpo de la solicitud tiene como resultado que la solicitud se bloquea, a menos que la inspección del cuerpo esté desactivada (XML, JSON, datos de formulario).
* La longitud de los datos del cuerpo de la solicitud (sin archivos) es mayor que el límite configurado.
* El cuerpo de la solicitud (archivos incluidos) es mayor que el límite.
* Se produjo un error interno en el motor de WAF.

Específico de CRS 3.x:

* La puntuación de anomalías de entrada superó el umbral.

## <a name="next-steps"></a>Pasos siguientes

Después de configurar las reglas deshabilitadas, puede aprender a ver los registros de WAFS. Para más información, consulte [Diagnósticos de Application Gateway](../../application-gateway/application-gateway-diagnostics.md#diagnostic-logging).