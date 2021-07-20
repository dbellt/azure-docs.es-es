---
title: ¿Qué es Azure Firewall?
description: Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network.
author: vhorne
ms.service: firewall
services: firewall
ms.topic: overview
ms.custom: mvc, contperf-fy21q1
ms.date: 07/01/2021
ms.author: victorh
ms.openlocfilehash: e87953217fe44fedc8d693a40a1f3ae942a06a39
ms.sourcegitcommit: 6bd31ec35ac44d79debfe98a3ef32fb3522e3934
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 07/02/2021
ms.locfileid: "113216376"
---
# <a name="what-is-azure-firewall"></a>¿Qué es Azure Firewall?

![Certificación ICSA](media/overview/icsa-cert-firewall-small.png)

Azure Firewall es un servicio de seguridad de red administrado y basado en la nube que protege los recursos de Azure Virtual Network. Se trata de un firewall como servicio con estado completo que incorpora alta disponibilidad y escalabilidad a la nube sin restricciones.

![Información general de firewalls](media/overview/firewall-threat.png)

Puede crear, aplicar y registrar directivas de aplicaciones y de conectividad de red a nivel central en suscripciones y redes virtuales. Azure Firewall usa una dirección IP pública estática para los recursos de red virtual, que permite que los firewall externos identifiquen el tráfico procedente de la red virtual.  El servicio está totalmente integrado con Azure Monitor para los registros y análisis.

Para obtener información sobre las características de Azure Firewall, consulte [Características de Azure Firewall](features.md).

## <a name="azure-firewall-premium-preview"></a>Versión preliminar de Azure Firewall Prémium

La versión preliminar de Azure Firewall Prémium es un firewall de próxima generación con funcionalidades que son necesarias en entornos de alta confidencialidad y regulados. Estas funcionalidades incluyen la inspección de TLS, IDPS, filtrado de direcciones URL y categorías web.

Para más información sobre las características de la versión preliminar de Azure Firewall Prémium, consulte [Características de la versión preliminar de Azure Firewall Prémium](premium-features.md).


Para ver cómo se configura la vista previa de Firewall Prémium en Azure Portal, consulte [Versión preliminar de Azure Firewall Prémium en Azure Portal](premium-portal.md).


## <a name="pricing-and-sla"></a>Precios y contrato de nivel de servicio

Para obtener información de los precios de Azure Firewall, consulte [Precios de Azure Firewall](https://azure.microsoft.com/pricing/details/azure-firewall/).

Para obtener información del Acuerdo de Nivel de Servicio de Azure Firewall, consulte [SLA para Azure Firewall](https://azure.microsoft.com/support/legal/sla/azure-firewall/).

## <a name="whats-new"></a>Novedades

Para obtener información sobre las novedades de Azure Firewall, consulte [Actualizaciones de Azure](https://azure.microsoft.com/updates/?category=networking&query=Azure%20Firewall).


## <a name="known-issues"></a>Problemas conocidos

Azure Firewall presenta los siguientes problemas conocidos:

|Problema  |Descripción  |Mitigación  |
|---------|---------|---------|
|Las reglas de filtrado de red para protocolos que no son TCP/UDP (por ejemplo, ICMP) no funcionan con el tráfico enlazado a Internet|Las reglas de filtrado de red de protocolos que no son TCP/UDP no funcionan con la traducción SNAT a la dirección IP pública. Los protocolos que no son TCP/UDP no se admiten entre subredes de radio y redes virtuales.|Azure Firewall usa Standard Load Balancer, [que actualmente no admite SNAT para los protocolos IP](../load-balancer/load-balancer-overview.md). Se están examinando opciones para admitir este escenario en una versión futura.|
|Falta de compatibilidad entre PowerShell y CLI con ICMP|Azure PowerShell y la CLI no admiten ICMP como protocolo válido en las reglas de red.|Aun así se puede usar ICMP como protocolo a través del portal y la API REST. Estamos trabajando para agregar pronto ICMP a PowerShell y la CLI.|
|Las etiquetas FQDN requieren que se establezca una combinación protocolo: puerto|Las reglas de aplicaciones con las etiquetas FQDN requieren la definición de puerto: protocolo.|Puede usar **https** como valor de puerto: protocolo. Estamos trabajando para que este campo sea opcional cuando se usen etiquetas FQDN.|
|No se admite la posibilidad de mover un firewall a otro grupo de recursos o suscripción.|No se admite la posibilidad de mover un firewall a otro grupo de recursos o suscripción.|La compatibilidad con esta funcionalidad está en nuestro mapa de ruta. Para mover un firewall a otro un grupo de recursos o suscripción, debe eliminar la instancia actual y volver a crearla en el nuevo grupo de recursos o suscripción.|
|Las alertas de inteligencia sobre amenazas pueden estar enmascaradas|Reglas de red con destino 80/443 para las alertas de inteligencia sobre amenazas de las máscaras de filtrado de salida cuando la configuración está establecida en el modo de solo alerta.|Cree filtrado de salida para 80/443 mediante reglas de aplicaciones. También puede cambiar el modo de inteligencia sobre amenazas a **Alertar y denegar**.|
|DNAT de Azure Firewall no funciona para destinos de IP privadas|La compatibilidad con DNAT de Azure Firewall se limita a la entrada/salida de Internet. DNAT no funciona actualmente con destinos de IP privada. Por ejemplo, radio a radio.|Se trata de una limitación actual.|
|No se puede eliminar la primera configuración de IP pública|Cada IP pública de Azure Firewall se asigna a una *configuración de IP*.  La primera configuración de IP se asigna durante la implementación del firewall y, normalmente, también contiene una referencia a la subred del firewall (a menos que se configure explícitamente de forma diferente mediante una implementación de plantilla). No se puede eliminar esta configuración de IP porque podría anular la asignación del firewall. Aún podrá cambiar o eliminar la dirección IP pública asociada con esta configuración de IP si el firewall tiene al menos otra dirección IP pública disponible para su uso.|es así por diseño.|
|Availability Zones solo se puede configurar durante la implementación.|Availability Zones solo se puede configurar durante la implementación. No se puede configurar una vez implementado un firewall.|es así por diseño.|
|SNAT en conexiones entrantes|Además de DNAT, en las conexiones mediante la dirección IP pública del firewall (entrante) se aplica SNAT en una de las direcciones IP privadas del firewall. Este requisito hoy en día (también para aplicaciones virtuales de red activa/activa) es para garantizar el enrutamiento simétrico.|Para conservar el código fuente original para HTTP/S, considere el uso de encabezados [XFF](https://en.wikipedia.org/wiki/X-Forwarded-For). Por ejemplo, use un servicio como [Azure Front Door](../frontdoor/front-door-http-headers-protocol.md#front-door-to-backend) o [Azure Application Gateway](../application-gateway/rewrite-http-headers-url.md) delante del firewall. También puede agregar WAF como parte de Azure Front Door Service y la cadena al firewall.
|El filtrado por nombre de dominio completo de SQL se admite solo en modo de proxy (puerto 1433)|Para Azure SQL Database, Azure Synapse Analytics y Azure SQL Managed Instance:<br><br>El filtrado por nombre de dominio completo de SQL se admite solo en modo de proxy (puerto 1433).<br><br>Para IaaS de Azure SQL:<br><br>Si va a usar puertos que no son los estándar, puede especificar esos puertos en las reglas de la aplicación.|Para SQL en modo de redirección (el valor predeterminado si se conecta desde dentro de Azure), puede filtrar en su lugar el acceso mediante la etiqueta de servicio de SQL como parte de las reglas de red de Azure Firewall.
|No se permite tráfico de salida en el puerto TCP 25.| Las conexiones SMTP salientes que usan el puerto TCP 25 se bloquearon. El puerto 25 se usa principalmente para la entrega de correo electrónico sin autenticación. Éste es el comportamiento predeterminado de la plataforma para las máquinas virtuales. Para más información, consulte [Solución de problemas de conectividad SMTP saliente en Azure](../virtual-network/troubleshoot-outbound-smtp-connectivity.md). Sin embargo, a diferencia de las máquinas virtuales, actualmente no es posible habilitar esta funcionalidad en Azure Firewall. Nota: para permitir SMTP autenticado (puerto 587) o SMTP a través de un puerto que no sea el 25, asegúrese de configurar una regla de red, no una regla de aplicación, ya que actualmente no se admite la inspección de SMTP.|Siga el método recomendado para enviar correo electrónico, como se documenta en el artículo de solución de problemas de SMTP. O bien, excluya la máquina virtual que necesita acceso SMTP saliente desde la ruta predeterminada hasta el firewall. En su lugar, configure el acceso saliente directamente a Internet.
|Agotamiento de puertos SNAT|Azure Firewall admite actualmente 1024 puertos por cada dirección IP pública por cada instancia de conjunto de escalado de máquinas virtuales de back-end. De manera predeterminada, hay dos instancias de conjunto de escalado de máquinas virtuales.|Se trata de una limitación de SLB y estamos buscando constantemente oportunidades para aumentar los límites. Mientras tanto, se recomienda configurar las implementaciones de Azure Firewall con un mínimo de cinco direcciones IP públicas para implementaciones susceptibles de agotamiento de SNAT. Esto aumenta cinco veces los puertos SNAT disponibles. Realice la asignación desde un prefijo de dirección IP para simplificar los permisos del flujo descendente.|
|DNAT no es compatible con la tunelización forzada habilitada|Los firewalls implementados con la tunelización forzada habilitada no admiten el acceso entrante desde Internet debido al enrutamiento asimétrico.|Esto es así por diseño. La ruta de acceso de retorno de las conexiones entrantes pasa por el firewall local, que no ha visto la conexión establecida.
|Es posible que el FTP pasivo saliente no funcione en firewalls con varias direcciones IP públicas, en función de la configuración del servidor FTP.|FTP pasivo establece distintas conexiones para los canales de control y datos. Cuando un firewall con varias direcciones IP públicas envía datos de salida, selecciona de manera aleatoria una de sus direcciones IP públicas como dirección IP de origen. FTP puede generar un error cuando los canales de control y de datos usan direcciones IP de origen diferentes, en función de la configuración del servidor FTP.|Está previsto una configuración de SNAT explícita. Entretanto, puede configurar el servidor FTP para que acepte los canales de control y de datos de distintas direcciones IP de origen (vea [un ejemplo de IIS](/iis/configuration/system.applicationhost/sites/sitedefaults/ftpserver/security/datachannelsecurity)). Otra opción es usar una única dirección IP en esta situación.|
|Es posible que el FTP pasivo entrante no funcione en función de la configuración del servidor FTP |FTP pasivo establece distintas conexiones para los canales de control y datos. En las conexiones entrantes de Azure Firewall se aplica SNAT a una de las direcciones IP privadas del firewall para garantizar el enrutamiento simétrico. FTP puede generar un error cuando los canales de control y de datos usan direcciones IP de origen diferentes, en función de la configuración del servidor FTP.|Se está investigando la conservación de la dirección IP de origen original. Entretanto, puede configurar el servidor FTP para que acepte los canales de control y de datos de distintas direcciones IP de origen.|
|Falta una dimensión de protocolo en la métrica NetworkRuleHit|La métrica ApplicationRuleHit permite el filtrado basado en el protocolo, pero esta funcionalidad falta en la métrica NetworkRuleHit correspondiente.|Se está investigando una solución.|
|No se admiten las reglas NAT con puertos entre el 64000 y el 65535|Azure Firewall permite cualquier puerto en el rango 1-65535 en reglas de red y de aplicación; sin embargo, las reglas NAT solo admiten los puertos del rango 1-63999.|Se trata de una limitación actual.
|Las actualizaciones de configuración pueden tardar cinco minutos por término medio.|Una actualización de la configuración de Azure Firewall puede tardar entre tres y cinco minutos por término medio, y no se admiten actualizaciones en paralelo.|Se está investigando una solución.|
|Azure Firewall usa encabezados TLS SNI para filtrar el tráfico HTTPS y MSSQL|Si el software del explorador o servidor no admite la extensión Server Name Indicator (SNI), no será posible conectarse mediante Azure Firewall.|Si el software de explorador o servidor no admite SNI, es posible que pueda controlar la conexión mediante una regla de red, en lugar de una regla de aplicación. Consulte [Indicación de nombre de servidor](https://wikipedia.org/wiki/Server_Name_Indication) para conocer el software que admite SNI.|
|DNS personalizado (versión preliminar) no funciona con la tunelización forzada.|Si está habilitada la tunelización forzada, DNS personalizado no funciona.|Se está investigando una solución.|
|El inicio o la detención no funcionan con los firewalls configurados en modo de túnel forzado|El inicio o la detención no funcionan con Azure Firewall configurado en modo de túnel forzado. Si se intenta iniciar Azure Firewall con la tunelización forzada configurada, aparece el siguiente error:<br><br>*Set-AzFirewall: AzureFirewall FW-xx management IP configuration cannot be added to an existing firewall. Redeploy with a management IP configuration if you want to use forced tunneling support.<br>StatusCode: 400<br>ReasonPhrase: Solicitud incorrecta* (Set-AzFirewall: la configuración de la IP de administración de AzureFirewall FW-xx no se puede agregar a un firewall existente. Vuelva a realizar la implementación de desea usar el soporte técnico de la tunelización forzada.
StatusCode: 400
ReasonPhrase: solicitud incorrecta).|Bajo investigación.<br><br>Como solución alternativa, puede eliminar el firewall existente y crear otro con los mismos parámetros.|
|No se pueden agregar etiquetas de directiva de firewall mediante el portal ni mediante plantillas de Azure Resource Manager (ARM)|La directiva de Azure Firewall tiene una limitación de compatibilidad de revisión que impide agregar una etiqueta mediante Azure Portal ni mediante plantillas de ARM. Se genera el siguiente error: *No se pudieron guardar las etiquetas del recurso*.|Se está investigando una solución. O bien, puede usar el cmdlet de Azure PowerShell `Set-AzFirewallPolicy` para actualizar las etiquetas.|
|Actualmente, no se admite IPv6.|Si agrega una dirección IPv6 a una regla, se produce un error en el firewall.|Use solo direcciones IPv4. La compatibilidad con IPv6 está en proceso de investigación.|
|La actualización de varios grupos de IP produce un error de conflicto.|Cuando se actualizan dos o más grupos de IP conectados al mismo firewall, uno de los recursos entra en estado de error.|Es un problema conocido, una limitación. <br><br>Al actualizar un grupo de IP, se desencadena una actualización en todos los firewalls a los que está conectado el grupo de IP. Si se inicia una actualización en un segundo grupo de IP con el firewall aún en estado *Actualizando*, se produce un error en la actualización del grupo de IP.<br><br>Para evitar el error, los grupos de IP conectados al mismo firewall se deben actualizar de uno en uno. Deje tiempo suficiente entre las actualizaciones para que el firewall pueda salir del estado *Actualizando*.|
|No se admite la eliminación de elementos RuleCollectionGroup mediante plantillas de ARM.|No se admite la eliminación de elementos RuleCollectionGroup mediante plantillas de ARM y se produce un error.|No es un operación admitida.|
|La regla DNAT para permitir *cualquiera* (*) permitirá el tráfico SNAT.|Si una regla DNAT permite *cualquiera* (*) como dirección IP de origen, una regla de red implícita coincidirá con el tráfico de red virtual a red virtual, y siempre transmitirá el tráfico mediante SNAT.|Se trata de una limitación actual.|
|No se admite la adición de una regla DNAT a un centro virtual protegido con un proveedor de seguridad.|Esto da como resultado una ruta asincrónica para el tráfico DNAT de vuelta, que va al proveedor de seguridad.|No compatible.|


## <a name="next-steps"></a>Pasos siguientes

- [Inicio rápido: Creación de una instancia de Azure Firewall y una directiva de firewall: plantilla de Azure Resource Manager](../firewall-manager/quick-firewall-policy.md)
- [Inicio rápido: Implementación de Azure Firewall con Availability Zones: plantilla de Resource Manager](deploy-template.md)
- [Tutorial: Implementación y configuración de Azure Firewall mediante Azure Portal](tutorial-firewall-deploy-portal.md)
