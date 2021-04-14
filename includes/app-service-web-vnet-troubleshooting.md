---
author: ccompy
ms.service: app-service-web
ms.topic: include
ms.date: 02/27/2020
ms.author: ccompy
ms.openlocfilehash: cec44bbabdb7d528c30a8d3396b819f2eb3c5386
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95999467"
---
La característica es fácil de configurar, aunque eso no quiere decir que no presente problemas con el uso. Si encuentra problemas para acceder al punto de conexión que quiere, existen varias utilidades que sirven para probar la conectividad desde la consola de la aplicación. Dispone de dos consolas que puede usar. Una es la consola Kudu y la otra es la consola a la que se accede en Azure Portal. Para acceder a la consola Kudu desde la aplicación, vaya a **Herramientas** > **Kudu**. También puede tener acceso a la consola de Kudo en [sitename].scm.azurewebsites.net. Después de que se cargue el sitio web, vaya a la pestaña **Consola de depuración**. Para llegar a la consola hospedada en Azure Portal desde su aplicación, vaya a **Herramientas** > **Consola**.

#### <a name="tools"></a>Herramientas
En las aplicaciones nativas de Windows, las herramientas **ping**, **nslookup** y **tracert** no funcionarán a través de la consola debido a las restricciones de seguridad (funcionan en [contenedores de Windows personalizados](../articles/app-service/quickstart-custom-container.md)). Para suplir esta carencia, se agregaron dos herramientas diferentes. Para probar la funcionalidad de DNS, se agregó una herramienta denominada **nameresolver.exe**. La sintaxis es:

```console
nameresolver.exe hostname [optional: DNS Server]
```

Puede usar nameresolver para comprobar los nombres de host de los que depende la aplicación. De este modo, puede probar si hay algo mal configurado en el DNS o si no tiene acceso al servidor DNS. Para ver el servidor DNS que la aplicación usa en la consola, examine las variables de entorno WEBSITE_DNS_SERVER y WEBSITE_DNS_ALT_SERVER.

> [!NOTE]
> nameresolver.exe no funciona actualmente en contenedores de Windows personalizados.
>

Puede usar la siguiente herramienta para probar la conectividad de TCP en una combinación de host y puerto. Esta herramienta se llama **tcpping** y la sintaxis es:

```console
tcpping.exe hostname [optional: port]
```

La utilidad **tcpping** indica si se puede llegar a un host y puerto específicos. Puede mostrar un resultado correcto solo si hay una aplicación que escucha en la combinación de host y puerto, y existe acceso a la red desde la aplicación hacia el host y puerto especificados.

#### <a name="debug-access-to-virtual-network-hosted-resources"></a>Depuración del acceso a recursos hospedados en la red virtual
Varias situaciones pueden impedir que la aplicación se comunique con un host y un puerto específicos. La mayoría de las veces se debe a una de estas causas:

* **Existe un firewall que lo impide.** Si tiene un firewall que lo impide, se agota el tiempo de espera de TCP. El tiempo de espera de TCP es de 21 segundos en este caso. Utilice la herramienta **tcpping** para probar la conectividad. Los tiempos de espera agotados de TCP pueden tener muchas causas además de los firewalls, pero comience por comprobar los firewalls.
* **El DNS no es accesible.** El tiempo de espera de DNS es de 3 segundos por cada servidor DNS. Si tiene dos servidores DNS, el tiempo de espera es de seis segundos. Utilice nameresolver para ver si funciona el DNS. No puede usar nslookup, ya que este no usa el DNS con el que se ha configurado la red virtual. Si no puede acceder, podría haber un firewall o NSG bloqueando el acceso a DNS, o este puede estar inactivo.

Si estos elementos no resuelven el problema, plantéese cuestiones como las siguientes:

**Integración con red virtual regional**
* ¿El destino no es una dirección RFC 1918 y el valor WEBSITE_VNET_ROUTE_ALL no está establecido en 1?
* ¿Hay un NSG que bloquea la salida de la subred de integración?
* Si está usando una VPN o Azure ExpressRoute, ¿la puerta de enlace local está configurada para enrutar el tráfico de vuelta a Azure? Si puede acceder a los puntos de conexión de la red virtual pero no a los del entorno local, compruebe las rutas.
* ¿Tiene permisos suficientes para configurar la delegación en la subred de integración? Al configurar la versión de Integración con red virtual que necesita una puerta de enlace, la subred de integración se delega en Microsoft.Web/serverFarms. La interfaz de usuario de integración de la red virtual delega la subred en Microsoft.Web/serverFarms automáticamente. Si la cuenta no tiene suficientes permisos de red para establecer la delegación, necesitará que un usuario que pueda configurar atributos en la subred de integración delegue la subred. Para delegar manualmente la subred de integración, vaya a la interfaz de usuario de la subred de Azure Virtual Network y establezca la delegación para Microsoft.Web/serverFarms.

**Integración con red virtual con requisito de puerta de enlace**
* ¿El intervalo de direcciones de punto a sitio se encuentra en los intervalos de RFC 1918 (10.0.0.0-10.255.255.255/172.16.0.0-172.31.255.255/192.168.0.0-192.168.255.255)?
* ¿La puerta de enlace aparece como activa en el portal? Si la puerta de enlace está inactiva, vuelva a activarla.
* ¿Los certificados aparecen como sincronizados o sospecha que la configuración de red ha cambiado?  Si los certificados no están sincronizados o sospecha que se produjo un cambio en la configuración de red virtual que no se ha sincronizado con sus planes de App Service, seleccione **Sincronizar red**.
* Si está usando una VPN, ¿la puerta de enlace local está configurada para enrutar el tráfico de vuelta a Azure? Si puede acceder a los puntos de conexión de la red virtual pero no del entorno local, compruebe las rutas.
* ¿Está intentando usar una puerta de enlace de coexistencia que admite tanto una conexión de punto a sitio como ExpressRoute? Las puertas de enlace de coexistencia no son compatibles con la Integración con red virtual.

Depurar problemas de red es todo un reto porque no se puede ver lo que está bloqueando el acceso en una combinación de host y puerto específica. Entre las causas se incluyen las siguientes:

* Tiene un firewall en el host que bloquea el acceso al puerto de la aplicación desde el intervalo IP de punto a sitio. A menudo se requiere acceso público para establecer conexiones entre subredes.
* El host de destino está fuera de servicio.
* La aplicación está fuera de servicio.
* La dirección IP o el nombre de host son incorrectos.
* La aplicación escucha en un puerto diferente al esperado. Puede hacer coincidir el id. de proceso con el puerto de escucha usando "netstat -aon" en el host del punto de conexión.
* Los grupos de seguridad de red están configurados de tal manera que evitan el acceso al host y al puerto de la aplicación desde el intervalo IP de punto a sitio.

No se sabe qué dirección usa realmente la aplicación. Podría ser cualquier dirección en el intervalo de direcciones de punto a sitio o de subred de integración, por lo que será necesario permitir el acceso desde el intervalo de direcciones completo.

Otros pasos de depuración son:

* Conectarse a una máquina virtual de la red virtual e intentar acceder al recurso host:puerto desde allí. Para probar el acceso de TCP, use el comando **test-netconnection** de PowerShell. La sintaxis es:

```powershell
test-netconnection hostname [optional: -Port]
```

* Abra una aplicación en una máquina virtual y pruebe el acceso a ese host y ese puerto desde la consola de la aplicación mediante **tcpping**.

#### <a name="on-premises-resources"></a>Recursos locales ####

Si la aplicación no puede acceder a un recurso local, compruebe si puede hacerlo desde su red virtual. Use el comando **test-netconnection** de PowerShell para comprobar el acceso de TCP. Si la máquina virtual no puede acceder al recurso local, puede que la conexión de VPN o ExpressRoute no esté configurada correctamente.

Si la máquina virtual hospedada en la red virtual puede acceder a su sistema local, pero la aplicación no, la causa es probablemente una de las razones siguientes:

* Las rutas no están configuradas con los intervalos de direcciones de punto a sitio o subred en la puerta de enlace local.
* Los grupos de seguridad de red están bloqueando el acceso del intervalo IP de punto a sitio.
* Los firewalls locales están bloqueando el tráfico del intervalo IP de punto a sitio.
* Está intentando acceder a una dirección que no es de RFC 1918 con la característica Integración con red virtual regional.
