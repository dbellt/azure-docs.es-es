---
title: Acceso híbrido seguro de Azure AD con la guía de implementación de F5 | Microsoft Docs
description: Tutorial para implementar la máquina virtual de F5 BIG-IP Virtual Edition (VE) en IaaS de Azure para el acceso híbrido seguro
services: active-directory
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.subservice: app-mgmt
ms.topic: how-to
ms.workload: identity
ms.date: 10/12/2020
ms.author: gasinh
ms.collection: M365-identity-device-management
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 139009c55573e1e115a22069671f66e93695a635
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107783330"
---
# <a name="tutorial-to-deploy-f5-big-ip-virtual-edition-vm-in-azure-iaas-for-secure-hybrid-access"></a>Tutorial para implementar la máquina virtual de F5 BIG-IP Virtual Edition en IaaS de Azure para el acceso híbrido seguro

Este tutorial le guía a través del proceso completo de implementación de BIG-IP Vitural Edition (VE) en IaaS de Azure. Al final de este tutorial tendrá:

- Una máquina virtual de BIG-IP totalmente preparada para modelar una prueba de concepto de acceso híbrido seguro (SHA)

- Una instancia de almacenamiento provisional que se usará para probar nuevas revisiones y actualizaciones del sistema BIG-IP.

## <a name="prerequisites"></a>Requisitos previos

No se necesita experiencia ni conocimientos previos de F5 BIG-IP, aunque es recomendable que se familiarice con la [terminología de este](https://www.f5.com/services/resources/glossary). La implementación de una instancia de BIG-IP en Azure para el acceso híbrido seguro requiere:

- Una suscripción de pago de Azure o una [suscripción de evaluación](https://azure.microsoft.com/free/) gratuita de 12 meses.

- Cualquiera de las siguientes SKU de licencias de F5 BIG-IP

  - F5 BIG-IP® Best bundle

  - Licencia independiente de F5 BIG-IP Access Policy Manager™ (APM).

  - Licencia del complemento F5 BIG-IP Access Policy Manager™ (APM) en una instalación de F5 BIG-IP® Local Traffic Manager™ (LTM) ya existente.
  
  - [Licencia de evaluación gratuita](https://www.f5.com/trial/big-ip-trial.php) completa de 90 días de BIG-IP.

- Un certificado comodín o de nombre alternativo del firmante (SAN), para publicar aplicaciones web a través de la capa de sockets seguros (SSL). [Let's Encrypt](https://letsencrypt.org/) ofrece un certificado gratis de 90 días para las pruebas.

- Un certificado SSL para proteger la interfaz de administración de BIG-IPs. Se puede utilizar un certificado que se usa para publicar aplicaciones web, si su asunto se corresponde con el nombre de dominio completo de BIG-IP. Por ejemplo, un certificado comodín definido con un asunto *.contoso.com sería adecuado para `https://big-ip-vm.contoso.com:8443`

La implementación de la máquina virtual y las configuraciones del sistema base tardan aproximadamente 30 minutos, momento en el cual la plataforma BIG-IP estará lista para implementar cualquiera de los escenarios de acceso híbrido seguro que se muestran [aquí](f5-aad-integration.md).

Para probar los escenarios, en este tutorial se da por supuesto que BIG-IP se implementará en un grupo de recursos de Azure que contiene un entorno de Active Directory (AD). El entorno debe constar de una máquina virtual que actúe de controlador de dominio (DC) y otra de host web (IIS). Tener estos servidores en otras ubicaciones de la máquina virtual de BIG-IP también es correcto siempre que BIG-IP tenga una línea de visión sobre cada uno de los roles necesarios para admitir un escenario determinado. También se admiten los escenarios en los que la máquina virtual de BIG-IP está conectada a otro entorno a través de una conexión VPN.

Si no tiene los elementos mencionados aquí para realizar pruebas, puede implementar un entorno de dominio de AD completo en Azure mediante este [script](https://github.com/Rainier-MSFT/Cloud_Identity_Lab). Una colección de aplicaciones de prueba de ejemplo también se puede implementar mediante programación en un host web de IIS mediante esta [automatización con scripts](https://github.com/jeevanbisht/DemoSuite).

>[!NOTE]
>[Azure Portal](https://portal.azure.com/#home) está en continua evolución, por lo que algunos de los pasos de este tutorial pueden diferir del diseño real que se observa en Azure Portal.

## <a name="azure-deployment"></a>Implementación en Azure

Se puede implementar una instancia de BIG-IP en topologías diferentes. Esta guía se centra en una implementación en una sola interfaz de red (NIC). Sin embargo, si la implementación de BIG-IP requiere varias interfaces de red para lograr una alta disponibilidad, segregación de red o un rendimiento de más de 1 GB, considere la posibilidad de usar las [plantillas precompiladas de Azure Resource Manager](https://clouddocs.f5.com/cloud/public/v1/azure/Azure_multiNIC.html) de F5.

Realice las siguientes tareas para implementar BIG-IP Virtual Edition desde [Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps).

1. Inicie sesión en [Azure Portal](https://portal.azure.com/#home) con una cuenta que disponga de permisos para crear máquinas virtuales. Por ejemplo, una cuenta de colaborador.

2. En el cuadro de búsqueda de la cinta de opciones superior, escriba **Marketplace**, y pulse **ENTRAR**.

3. Escriba **F5** en el filtro de Marketplace y pulse **ENTRAR**.

4. Seleccione **+ Agregar** en la cinta de opciones superior y escriba **F5** en el filtro de Marketplace y pulse **ENTRAR**

5. Seleccione **F5 BIG-IP Virtual Edition (BYOL)**  > **Seleccionar un plan de software** > **F5 BIG-IP VE- ALL (BYOL, 2 ubicaciones de arranque)**

6. Seleccione **Crear**.

![La imagen muestra los pasos para seleccionar un plan de software.](./media/f5ve-deployment-plan/software-plan.png)

7. Siga los pasos del menú **Aspectos básicos** y use la siguiente configuración

 |  Detalles del proyecto     |  Valor     |
 |:-------|:--------|
 |Subscription|Suscripción de destino para la implementación de la máquina virtual de BIG-IP|
 |Resource group | Grupo de recursos de Azure existente en el que se implementará la máquina virtual de BIG-IP o se creará una. Debe ser el mismo grupo de recursos de las máquinas virtuales de DC y de IIS.|
 | **Detalles de instancia**|  |
 |Nombre de la máquina virtual| Máquina virtual de ejemplo de BIG-IP |
 |Region | Geoárea de Azure de destino para la máquina virtual de BIG-IP |
 |Opciones de disponibilidad| Habilite solo si se va a usar la máquina virtual en producción.|
 |Imagen| F5 BIG-IP VE - ALL (BYOL, 2 ubicaciones de arranque)|
 |Instancia de Azure Spot| No, pero no dude en habilitarla si es necesario |
 |Size| La especificación mínima deberían ser 2 vCPU y 8 GB de memoria|
 |**Cuenta de administrador**|  |
 |Tipo de autenticación|Por ahora, seleccione contraseña. Puede cambiar a "par de claves" más adelante. |
 |Nombre de usuario|La identidad que se creará como una cuenta local de BIG-IP para acceder a sus interfaces de administración. El nombre de usuario distingue mayúsculas de minúsculas.|
 |Contraseña|Proteja el acceso de administrador con una contraseña segura|
 |**Reglas de puerto de entrada**|  |
 |Puertos de entrada públicos|Ninguno|

8. Seleccione **Siguiente: Discos**, deje todos los valores predeterminados y seleccione **Siguiente: Redes**.

9. En el menú **Redes**, complete esta configuración.

 |interfaz de red|      Value |
 |:--------------|:----------------|
 |Virtual network|La misma red virtual de Azure que usan las máquinas virtuales de DC e IIS, o bien cree otra.|
 |Subnet| La misma subred interna de Azure que las máquinas virtuales de DC e IIS, o bien cree otra.|
 |Dirección IP pública |  Ninguno|
 |Grupo de seguridad de red de NIC| Seleccione Ninguno si la subred de Azure que seleccionó en los pasos anteriores ya está asociada a un grupo de seguridad de red. De lo contrario, seleccione Básica.|
 |Aceleración de redes| Desactivado |
 |**Equilibrio de carga**|     |
 |Equilibrio de carga de la máquina virtual| No|

10. Seleccione **Siguiente: Administración** y complete esta configuración.

 |Supervisión|    Valor |
 |:---------|:-----|
 |Supervisión detallada| Desactivado|
 |Diagnósticos de arranque|Habilite con la cuenta de almacenamiento personalizada. Permite conectarse a la interfaz de Secure Shell (SSH) para BIG-IP a través de la opción Consola serie de Azure Portal. Seleccione cualquier cuenta de Azure Storage que esté disponible|
 |**Identidad**|  |
 |Identidad administrada asignada por el sistema|Desactivado|
 |Azure Active Directory|BIG-IP no admite actualmente esta opción.|
 |**Apagado automático**|    |
 |Habilitar el apagado automático| Si va a realizar pruebas, considere la posibilidad de establecer la máquina virtual de BIG-IP para que se apague diariamente.|

11. Seleccione **Siguiente: Avanzado**, deje todos los valores predeterminados y seleccione **Siguiente: Etiquetas**.

12. Seleccione **Siguiente: Revisar y crear** para revisar las configuraciones de la máquina virtual de BIG-IP, antes de seleccionar **Crear** para iniciar la implementación.

13. El tiempo para implementar por completo una máquina virtual de BIG-IP suele ser de 5 minutos. Cuando haya terminado no seleccione **Ir al recurso**. En lugar de eso, expanda el menú de la izquierda de Azure Portal y seleccione **Grupos de recursos** para ir a la nueva máquina virtual de BIG-IP. Si se produce un error en la creación de la máquina virtual, seleccione **Atrás** y **Siguiente**.

## <a name="network-configuration"></a>Configuración de red

Cuando se arranca la máquina virtual de BIG-IP por primera vez, su NIC se aprovisiona con una IP privada **principal** emitida por el servicio Protocolo de configuración dinámica de host (DHCP) de la subred de Azure a la que está conectada. El sistema operativo de administración de tráfico (TMOS) de BIG-IP usará esta dirección IP para comunicarse con:

- Otros hosts y servicios

- Acceso de salida a Internet público

- Acceso de entrada a las interfaces de administración de la configuración web y SSH de BIG-IP

Exponer cualquiera de estas interfaces de administración a Internet aumenta la superficie de ataque de BIG-IP, y este es el motivo por el que la dirección IP principal de BIG-IP no se aprovisionó con una dirección IP pública durante la implementación. En su lugar, se aprovisionará una IP interna secundaria y una IP pública asociada para los servicios de publicación.
Esta asignación de 1 a 1 entre una dirección IP pública y una IP privada de máquina virtual permite que el tráfico externo llegue a una máquina virtual. Sin embargo, también se requiere una regla de grupo de seguridad de red de Azure para permitir el tráfico, así como un firewall.

En el diagrama se muestra una implementación con una sola NIC de una instancia de BIG-IP VE en Azure, configurada con una dirección IP principal para operaciones generales y administración, y una IP de servidor virtual independiente para los servicios de publicación.
En esta disposición, una regla de grupo de seguridad de red permite que el tráfico remoto con destino a `intranet.contoso.com` se enrute a la dirección IP pública del servicio publicado antes de que se reenvíe al servidor virtual de BIG-IP.

![La imagen muestra la implementación con una sola NIC](./media/f5ve-deployment-plan/single-nic-deployment.png) De forma predeterminada, las direcciones IP públicas y privadas emitidas para las máquinas virtuales de Azure son siempre dinámicas, por lo que es probable que cambien en cada reinicio de una máquina virtual. Evite problemas de conectividad imprevistos cambiando la IP de administración de BIG-IP a estática y haga lo mismo con las IP secundarias que se usan para los servicios de publicación.

1. En el menú de la máquina virtual de BIG-IP, vaya a **Configuración** > **Redes**.

2. En la vista Redes, seleccione el vínculo situado a la derecha de **Interfaz de red**.

![La imagen muestra la configuración de red](./media/f5ve-deployment-plan/network-config.png)

>[!NOTE]
>Los nombres de las máquinas virtuales se generan aleatoriamente durante la implementación.

3. En el panel izquierdo, seleccione **IP Configurations** (Configuraciones de IP) y, a continuación, seleccione la fila **ipconfig1**.

4. Establezca la opción **IP Assignment** (Asignación de IP) en estática y, si es necesario, cambie la dirección IP principal de las máquinas virtuales de BIG-IP. Después, seleccione **Guardar** y cierre el menú ipconfig1.

>[!NOTE]
>Usará esta dirección IP principal para conectarse y administrar la máquina virtual de BIG-IP.

5. Seleccione **+ Agregar** en la cinta de opciones superior y proporcione un nombre para una IP privada secundaria, por ejemplo, ipconfig2.

6. Establezca la opción **Asignación** de la configuración de la dirección IP privada en **Estática**. Proporcionar la siguiente dirección IP consecutiva mayor o menor ayuda a mantener todo en orden.

7. Establezca la dirección IP pública en **Asociar** y seleccione **Crear**

8. Proporcione un nombre para la nueva dirección IP pública, por ejemplo, BIG-IP-VM_ipconfig2_Public.

9. Si se le solicita, establezca la **SKU** en Estándar.

10. Si se le solicita, establezca el **Nivel** en **Global** y

11. Establezca la opción **Asignación** en **Estática** y, a continuación, seleccione **Aceptar** dos veces.

La máquina virtual de BIG-IP ya está lista para configurarse con:

- **IP privada principal**: dedicada a la administración de la máquina virtual de BIG-IP a través de su utilidad de configuración web y SSH. La usará el sistema de BIG-IP como su **dirección IP propia** para conectarse a los servicios de back-end publicados. Además, se conecta a servicios externos como NTP, AD y LDAP.

- **IP privada secundaria**: se usa al crear un servidor virtual de BIG-IP APM para escuchar la solicitud de entrada a servicios publicados.

- **Dirección IP pública**: asociada a la IP privada secundaria, permite que el tráfico de cliente desde la red pública de Internet llegue al servidor virtual de BIG-IP para los servicios publicados.

En el ejemplo se muestra la relación 1 a 1 entre las direcciones IP públicas y privadas de una máquina virtual. Una NIC de máquina virtual de Azure solo puede tener una dirección IP principal, y cualquier otra dirección IP creada de manera adicional se considerará siempre como secundaria.

>[!NOTE]
>Necesitará las asignaciones de IP secundarias para publicar servicios de BIG-IP.

![En esta imagen se muestran todas las IP secundarias](./media/f5ve-deployment-plan/secondary-ips.png)

Para implementar el acceso híbrido seguro mediante la **configuración del acceso guiado** de BIG-IP, repita los pasos 5 al 11 para crear pares adicionales de IP privadas y públicas para cada servicio adicional que tenga pensado publicar mediante la instancia de BIG-IP APM. También se puede usar el mismo enfoque si se publican servicios mediante la **configuración avanzada** de BIG-IP. Sin embargo, en ese escenario tiene la opción de evitar sobrecargas de IP públicas mediante el uso de una configuración de [Indicación de nombre de servidor (SNI)](https://support.f5.com/csp/#/article/K13452). En esta configuración, un servidor virtual de BIG-IP aceptará todo el tráfico de cliente que reciba y lo enrutará hacia su destino.

## <a name="dns-configuration"></a>Configuración de DNS

Es fundamental tener DNS configurado para que los clientes resuelvan los servicios de acceso híbrido seguro publicados en las IP públicas de la máquina virtual de BIG-IP.

En los pasos siguientes se da por supuesto que la zona DNS del dominio público que se usa para los servicios del acceso híbrido seguro se administra en Azure. Sin embargo, los mismos principios de DNS de creación de registros de localizador se siguen aplicando sin importar dónde se administra la zona DNS.

1. Si aún no está abierto, expanda el menú de la izquierda del portal y vaya a la máquina virtual de BIG-IP a través de la opción **Grupos de recursos**.

2. En el menú de la máquina virtual de BIG-IP, vaya a **Configuración** > **Redes**.

3. En la vista Redes de la máquina virtual de BIG-IP, seleccione la primera IP secundaria en la lista desplegable IP configuration (Configuración de IP) y seleccione el vínculo de su **dirección IP pública de NIC**.

![Captura de pantalla para mostrar la dirección IP pública de NIC](./media/f5ve-deployment-plan/networking.png)

4. Debajo de la sección **Configuración** en el panel izquierdo, seleccione **Configuración** para abrir el menú de IP pública y propiedades de DNS de la dirección IP secundaria seleccionada.

5. Seleccione y **cree** un registro de alias y seleccione la **zona DNS** en la lista desplegable. Si no existe una zona DNS, se puede administrar fuera de Azure, o puede crear una para el sufijo de dominio que habría comprobado en Azure AD.

6. Utilice la siguiente información para crear el primer registro de alias del DNS:

 | Campo | Value |
 |:-------|:-----------|
 |Subscription| La misma suscripción que la máquina virtual de BIG-IP|
 |Zona DNS| Zona DNS que tiene autorización para el sufijo de dominio comprobado que usarán los sitios web publicados, por ejemplo, www.contoso.com |
 |Nombre | El nombre de host que especifique se resolverá en la dirección IP pública asociada a la IP secundaria seleccionada. Asegúrese de definir el DNS correcto para las asignaciones de IP. Vea la última imagen de la sección de configuraciones de Redes, por ejemplo, intranet.contoso.com > 13.77.148.215|
 | TTL | 1 |
 |Unidades de TTL | Horas |

7. Seleccione **Crear** para que Azure guarde esos valores en el DNS público.

8. Deje el valor **Etiqueta de nombre DNS (opcional)** y seleccione **Guardar** antes de cerrar el menú de IP pública.

9. Repita los pasos del 1 al 6 para crear registros DNS adicionales para cada servicio que tenga planeado publicar mediante la configuración guiada de BIG-IP.

  Con los registros DNS vigentes, puede usar cualquiera de las herramientas en línea como [DNS Checker](https://dnschecker.org/) para comprobar que un registro creado se ha propagado correctamente en todos los servidores DNS públicos globales.

  Si administra el espacio de nombres de dominio del DNS con un proveedor externo como [GoDaddy](https://www.godaddy.com/), deberá crear los registros mediante su propia utilidad de administración de DNS.

>[!NOTE]
>También puede usar el archivo localhosts de un equipo si prueba y cambia con frecuencia los registros de DNS. Para acceder al archivo localhosts en un equipo con Windows presione Win + R en el teclado y escriba la palabra **drivers** en el cuadro Ejecutar. Tenga en cuenta que un registro de localhost solo proporcionará la resolución DNS para el equipo local, no para otros clientes.

## <a name="client-traffic"></a>Tráfico de cliente

De forma predeterminada, las redes virtuales de Azure y las subredes asociadas son redes privadas que no pueden recibir tráfico de Internet. Se debe asociar la NIC de la máquina virtual de BIG-IP al grupo de seguridad de red que especificó durante la implementación. Para que el tráfico web externo llegue a la máquina virtual de BIG-IP, debe definir una regla de grupo de seguridad de red de entrada para permitir el acceso a los puertos 443 (HTTPS) y 80 (HTTP) desde la red pública de Internet.

1. En el menú principal de **información general** de la máquina virtual de BIG-IP, seleccione **Redes**.

2. Seleccione la regla de entrada **Agregar** para especificar las siguientes propiedades de la regla del grupo de seguridad de red:

 |     Campo   |   Value        |
 |:------------|:------------|
 |Source| Any|
 |Source port ranges| *|
 |Direcciones IP de destino|Lista separada por comas de todas las IP privadas secundarias de la máquina virtual de BIG-IP|
 |Puertos de destino| 80, 443|
 |Protocolo| TCP |
 |Acción| Allow|
 |Priority|Menor valor disponible entre 100 y 4096|
 |Nombre | Un nombre descriptivo, como: `BIG-IP-VM_Web_Services_80_443`|

3. Seleccione **Agregar** para confirmar los cambios y cierre el menú **Redes**.

Ahora se permitirá que el tráfico HTTP y HTTPS procedente de cualquier ubicación llegue a todas las interfaces secundarias de la máquina virtual de BIG-IP. Autorizar el puerto 80 es útil para permitir que BIG-IP APM redirija automáticamente a los usuarios de HTTP a HTTPS. Esta regla se puede editar para agregar o quitar direcciones IP de destino siempre que sea necesario.

## <a name="manage-big-ip"></a>Administración de BIG-IP

Un sistema de BIG-IP se administra a través de su interfaz de usuario de la configuración web, a la que se puede acceder mediante cualquiera de los siguientes métodos recomendados:

- Desde una máquina de la red interna de BIG-IP

- Desde un cliente VPN conectado a la red interna de la máquina virtual de BIG-IP

- Publicado mediante [Azure AD Application Proxy](./application-proxy-add-on-premises-application.md)

Tendrá que decidir cuál es el método más adecuado para poder continuar con las configuraciones restantes. Si es necesario, puede conectarse directamente a la configuración web desde Internet mediante la configuración de la dirección IP principal de BIG-IP con una IP pública. Después, agregue una regla de grupo de seguridad de red para permitir el tráfico del puerto 8443 a esa dirección IP principal. Asegúrese de restringir el origen a su propia dirección IP de confianza; de lo contrario, cualquiera podrá conectarse.

Una vez que esté listo, confirme que puede conectarse a la configuración web de la máquina virtual de BIG-IP e inicie sesión con las credenciales especificadas durante la implementación de la máquina virtual:

- Si se va a conectar desde una máquina virtual de su red interna o a través de VPN, conéctese directamente a la dirección IP principal de BIG-IP y al puerto de la configuración web. Por ejemplo, `https://<BIG-IP-VM_Primary_IP:8443`. El explorador le avisará de que la conexión no es segura, pero puede pasar por alto el aviso hasta que la instancia de BIG-IP esté configurada. Si el explorador insiste en bloquear el acceso, borre la memoria caché y vuelva a intentarlo.

- Si ha publicado el archivo de configuración web mediante Application Proxy, use la dirección URL definida para acceder a la configuración web externamente, sin necesidad de anexar el puerto, por ejemplo, `https://big-ip-vm.contoso.com`. La dirección URL interna debe definirse mediante el puerto de configuración web, por ejemplo, `https://big-ip-vm.contoso.com:8443` 

Un sistema de BIG-IP también se puede administrar mediante su entorno SSH subyacente, el cual se usa normalmente para las tareas de la línea de comandos (CLI) y el acceso de nivel raíz. Existen varias opciones para conectarse a la CLI, entre las que se incluyen:

- [El servicio Azure Bastion](../../bastion/bastion-overview.md): permite conexiones rápidas y seguras a cualquier máquina virtual de una red virtual, desde cualquier ubicación.

- Conexión directa a través de un cliente SSH como PuTTy a través del enfoque JIT

- Consola serie: se ofrece en la parte inferior de la sección Soporte técnico y solución de problemas del menú de máquinas virtuales del portal. No admite transferencias de archivos.

- Al igual que con la configuración web, también puede conectarse directamente a la CLI desde Internet configurando la dirección IP principal de BIG-IP con una dirección IP pública y agregando una regla de grupo de seguridad de red para permitir el tráfico SSH. De nuevo, no olvide restringir el origen a su propia dirección IP de confianza, si usa este método.  

## <a name="big-ip-license"></a>Licencia de BIG-IP

Se debe activar y aprovisionar un sistema de BIG-IP con el módulo APM antes de que se pueda configurar para los servicios de publicación y acceso híbrido seguro.

1. Vuelva a iniciar sesión en la configuración web y, en la página **Propiedades generales** seleccione **Activar.**

2. En el campo **Base Registration key** (Clave de registro base) escriba la clave que distingue mayúsculas de minúsculas que proporciona F5.

3. Deje el valor **Activation Method** (Método de activación) establecido en **Automático** y seleccione **Siguiente**, y el sistema de BIG-IP validará la licencia y mostrará el contrato de licencia para el usuario final (EULA).

4. Seleccione **Aceptar** y espere a que se complete la activación antes de seleccionar **Continuar**.

5. Vuelva a iniciar sesión y, en la parte inferior de la página Resumen de licencias, seleccione **Siguiente**. BIG-IP mostrará ahora una lista de los módulos que proporcionan las distintas características que son necesarias para el acceso híbrido seguro. Si no aparece, en la pestaña principal, vaya a **Sistema** > **Resource Provisioning** (Aprovisionamiento de recursos).

6. Compruebe la columna de aprovisionamiento de la directiva de acceso (APM).

![La imagen muestra el aprovisionamiento de acceso](./media/f5ve-deployment-plan/access-provisioning.png)

7. Seleccione **Enviar** y acepte la advertencia.

8. Sea paciente y espere a que BIG-IP termine de destacar las nuevas características. Se puede repetir varias veces antes de que se inicialice completamente. 

9. Cuando esté listo, seleccione **Continuar** y, en la pestaña **Acerca de**, seleccione **Run the setup utility** (Ejecutar la utilidad de configuración).

>[!IMPORTANT]
>Las licencias de F5 están restringidas para su uso por una sola instancia de BIG-IP VE en cualquier momento. Puede haber razones por las que desee migrar una licencia de una instancia a otra y, si lo hace, asegúrese de [revocar](https://support.f5.com/csp/article/K41458656) la licencia de prueba en la instancia activa antes de retirarla; de lo contrario, la licencia se perderá de forma permanente.

## <a name="provision-big-ip"></a>Aprovisionamiento de BIG-IP

Es fundamental proteger el tráfico de administración hacia la configuración web de BIG-IP y desde esta. Configure un certificado de administración de dispositivos para ayudar a proteger el canal de configuración web.

1. En la barra de navegación izquierda, vaya a **System** > **Certificate Management** > **Traffic Certificate Management** > **SSL Certificate List** > **Import** (Sistema > Administración de certificados > Administración de certificados de tráfico > Lista de certificados de SSL > Importar).

2. En la lista desplegable **Tipo de importación**, seleccione **PKCS 12(IIS)** y **Elegir archivo**. Busque un certificado web de SSL que tenga un nombre de asunto o SAN que abarque el nombre de dominio completo que asignará a la máquina virtual de BIG-IP en los pasos siguientes.

3. Proporcione la contraseña para el certificado y, a continuación, seleccione **Importar**.

4. En la barra de navegación izquierda, vaya a **System** > **Platform** (Sistema > Plataforma)

5. Configure la máquina virtual de BIG-IP con un nombre de host completo y la zona horaria para su entorno, y seleccione **Update** (Actualizar).

![La imagen muestra las propiedades generales](./media/f5ve-deployment-plan/general-properties.png)

6. En la barra de navegación izquierda, vaya a **System** > **Configuration** > **Device** > **NTP** (Sistema > Configuración > Dispositivo > NTP).

7. Especifique un origen NTP confiable y seleccione **Add** (Agregar) y, después, seleccione **Update** (Actualizar). Por ejemplo: `time.windows.com`

Ahora necesita un registro de DNS para resolver el nombre de dominio completo de BIG-IP especificado en los pasos anteriores, en su dirección IP privada principal. Se debe agregar un registro al DNS interno del entorno o al archivo localhost de un equipo que se usará para conectarse a la configuración web de BIG-IP. En cualquier caso, la advertencia del explorador ya no debe aparecer cuando se conecta directamente a la configuración web. Es decir, no a través de Application Proxy ni de ningún otro proxy inverso.

## <a name="ssl-profile"></a>Perfil de SSL

Como proxy inverso, un sistema de BIG-IP puede actuar como un servicio de reenvío sencillo, también conocido como proxy transparente, o como un proxy completo que participa activamente en intercambios entre clientes y servidores.
Un proxy completo crea dos conexiones distintas: una conexión de cliente TCP de front-end junto con una conexión de servidor TCP de back-end independiente, unida por un hueco flexible en el medio. Los clientes se conectan al agente de escucha del proxy en un extremo, el cual se conoce como **servidor virtual**, y el proxy establece una conexión independiente con el servidor back-end. Esto es bidireccional en ambos lados.
En este modo de proxy completo, el sistema de F5 BIG-IP es capaz de inspeccionar el tráfico y, por lo tanto, también es posible interactuar con las solicitudes y respuestas. Ciertas funciones como el equilibrio de carga y la optimización del rendimiento web, así como servicios de administración de tráfico más avanzados, como la seguridad de la capa de aplicación, la aceleración web, el enrutamiento de páginas y el acceso remoto seguro, dependen de esta funcionalidad.
Al publicar servicios basados en SSL, el proceso de descifrar y cifrar el tráfico entre los clientes y los servicios back-end se controla mediante perfiles de SSL de BIG-IP.

Existen dos tipos de perfiles:

- **SSL de cliente**: la forma más común de configurar un sistema de BIG-IP para publicar servicios internos con SSL es crear un perfil de SSL de cliente. Con un perfil de SSL de cliente, un sistema de BIG-IP puede descifrar las solicitudes de cliente de entrada antes de enviarlas a un servicio de nivel inferior. A continuación, cifra las respuestas de back-end salientes antes de enviarlas a los clientes.

- **SSL de servidor**: en el caso de los servicios de back-end configurados para HTTPS, puede configurar BIG-IP para que use un perfil de SSL de servidor. Con un perfil de SSL de servidor, BIG-IP vuelve a cifrar la solicitud del cliente antes de enviarla al servicio back-end de destino. Cuando el servidor devuelve una respuesta cifrada, el sistema de BIG-IP la descifra y la vuelve a cifrar antes de enviarla al cliente a través del perfil de SSL de cliente configurado.

El aprovisionamiento de perfiles de SSL de cliente y de servidor tendrá la instancia de BIG-IP preconfigurada y lista para todos los escenarios de acceso seguro híbrido.

1. En la barra de navegación izquierda, vaya a **System** > **Certificate Management** > **Traffic Certificate Management** > **SSL Certificate List** > **Import** (Sistema > Administración de certificados > Administración de certificados de tráfico > Lista de certificados de SSL > Importar).

2. En la lista desplegable **Tipo de importación**, seleccione **PKCS 12(IIS)** .

3. Proporcione un nombre para el certificado importado, por ejemplo `ContosoWilcardCert`.

4. Seleccione **Elegir archivo** para ir al certificado web de SSL cuyo nombre de asunto se corresponde con el sufijo de dominio que se va a usar para los servicios publicados.

5. Proporcione la **contraseña** del certificado importado y, a continuación, seleccione **Importar**.

6. En la barra de navegación izquierda, vaya a **Local Traffic** > **Profiles** > **SSL** > **Client** (Tráfico local > Perfiles > SSL > Cliente) y, después, seleccione **Create** (Crear).

7. En la página **New Client SSL Profile** (Nuevo perfil de SSL de cliente), proporcione un nombre descriptivo único para el nuevo perfil de SSL de cliente y asegúrese de que el perfil primario esté establecido en **clientssl**.

![En la imagen se muestra la opción Update (Actualizar) de BIG-IP](./media/f5ve-deployment-plan/client-ssl.png)

8. Active la casilla de la derecha en la fila **Certificate Key Chain** (Cadena de claves de certificado) y seleccione **Add** (Agregar).

9. En las tres listas desplegables, seleccione el certificado comodín que importó sin una frase de contraseña y, a continuación, seleccione **Add** > **Finished** (Agregar > Finalizado).

![En la imagen se muestra la actualización del certificado comodín de BIG-IP](./media/f5ve-deployment-plan/contoso-wildcard.png)

10. Repita los pasos 6 a 9 para crear un **perfil de certificado SSL de servidor**. En la cinta de opciones superior, seleccione **SSL** > **Server** > **Create** (SSL > Servidor > Crear).

11. En la página **New Server SSL Profile** (Nuevo perfil de SSL de servidor), proporcione un nombre descriptivo único para el nuevo perfil de SSL de servidor y asegúrese de que el perfil primario esté establecido en **serverssl**.

12. Active la casilla de la derecha para las filas Certificado y Clave y, en la lista desplegable, seleccione el certificado importado y, después, seleccione **Finished** (Finalizado).

![En la imagen se muestra la actualización de todos los perfiles de servidor de BIG-IP](./media/f5ve-deployment-plan/server-ssl-profile.png)

>[!NOTE]
>No se desanime si no puede adquirir un certificado SSL, puede usar los certificados SSL de cliente y servidor de BIG-IP autofirmados integrados. Solo verá un error de certificado en el explorador.

Un paso final en la preparación de BIG-IP para el acceso seguro híbrido es asegurarse de que puede localizar los recursos que publican y también el servicio de directorio en el que se basa para el inicio de sesión único. Una instancia de BIG-IP tiene dos orígenes de resolución de nombres que empiezan por su archivo local/.../hosts, o si no se encuentra un registro, el sistema de BIG-IP usa cualquier servicio DNS con el que se haya configurado. El método de archivo de hosts no se aplica a los grupos y nodos de APM que usan un nombre de dominio completo.

1. En la configuración web, vaya a **System** > **Configuration** > **Device** > **DNS** (Sistema > Configuración > Dispositivo > DNS).

2. En **DNS Lookup Server List** (Lista de servidores de búsqueda de DNS), escriba la dirección IP del servidor DNS del entorno.

3. Seleccione **Add** > **Update** (Agregar > Actualizar).

Como paso independiente y opcional, puede considerar la posibilidad de una [configuración de LDAP](https://somoit.net/f5-big-ip/authentication-using-active-directory) para autenticar sysadmins de BIG-IP en AD en lugar de administrar cuentas locales de BIG-IP.

## <a name="update-big-ip"></a>Actualización de BIG-IP

La máquina virtual de BIG-IP debe actualizarse para desbloquear todas las nuevas funcionalidades que se describen en las [instrucciones basadas en escenarios](f5-aad-integration.md). Puede comprobar la versión de TMOS de los sistemas si mantiene el puntero del mouse sobre el nombre de host de BIG-IP en la parte superior izquierda de la página principal. Se recomienda ejecutar la versión v15.x y versiones posteriores, las cuales se pueden obtener en la página de [descarga de F5](https://downloads.f5.com/esd/productlines.jsp). Use la [guía](https://support.f5.com/csp/article/K34745165) para actualizar el sistema operativo principal del sistema (TMOS).

Si no es posible actualizar este sistema, considere la posibilidad de, al menos, actualizar solo la configuración guiada mediante estos pasos.

1. En la pestaña principal de la configuración web de BIG-IP, vaya a **Access** > **Guided Configuration** (Acceso > Configuración guiada).

2. En la página **Guided Configuration** (Configuración guiada), seleccione **Upgrade Guided Configuration** (Actualizar configuración guiada).

![En la imagen se muestra cómo actualizar BIG-IP](./media/f5ve-deployment-plan/update-big-ip.png)

3. En el cuadro de diálogo **Upgrade Guided Configuration** (Actualizar configuración guiada), seleccione **Choose File** (Elegir archivo) para cargar el paquete de casos de uso descargado y seleccione **Upload and Install** (Cargar e instalar).

4. Una vez completada la actualización, seleccione **Continue** (Continuar).

## <a name="backup-big-ip"></a>Copia de seguridad de BIG-IP

Con el sistema de BIG-IP ya aprovisionado totalmente, se recomienda realizar una copia de seguridad completa de su configuración:

1. Vaya a **System** > **Archives** > **Create** (Sistema > Archivos > Crear).

2. Proporcione un **nombre de archivo** único y habilite el **cifrado** con una frase de contraseña.

3. Establezca la opción **Private Keys** (Claves privadas) en **Include** (Incluir) también para realizar copias de seguridad de los certificados SSL y de dispositivo.

4. Seleccione **Finished** (Finalizado) y espere a que se complete el proceso.

5. Se mostrará un estado de la operación que indicará un estado con el resultado de la copia de seguridad. Seleccione **Aceptar**.

6. Guarde el archivo del conjunto de configuración del usuario (UCS) localmente; para ello, elija el vínculo de la copia de seguridad y seleccione **Download** (Descargar).

Como paso opcional, también puede realizar una copia de seguridad de todo el disco del sistema con [instantáneas de Azure](../../virtual-machines/windows/snapshot-copy-managed-disk.md) que, a diferencia de la copia de seguridad de la configuración web, proporcionaría cierta contingencia para realizar pruebas entre versiones de TMOS o revertir a un sistema nuevo.

```PowerShell
# Install modules
Install-module Az
Install-module AzureVMSnapshots

# Authenticate to Azure
Connect-azAccount

# Set subscription by Id
Set-AzContext -SubscriptionId ‘<Azure_Subscription_ID>’

#Create Snapshot
New-AzVmSnapshot -ResourceGroupName '<E.g.contoso-RG>' -VmName '<E.g.BIG-IP-VM>'

#List Snapshots
#Get-AzVmSnapshot -ResourceGroupName ‘<E.g.contoso-RG>'

#Get-AzVmSnapshot -ResourceGroupName ‘<E.g.contoso-RG>' -VmName '<E.g.BIG-IP-VM>' | Restore-AzVmSnapshot -RemoveOriginalDisk 

```

## <a name="restore-big-ip"></a>Restauración de BIG-IP

La restauración de BIG-IP sigue un procedimiento similar a la copia de seguridad y también puede usarse para migrar configuraciones entre las máquinas virtuales de BIG-IP. Deben tenerse en cuenta los detalles de las rutas de actualización admitidas antes de importar una copia de seguridad.

1. Vaya a **System** > **Archives** (Sistema > Archivos).

2. Elija el vínculo de una copia de seguridad que desea restaurar o seleccione el botón **Upload** (Cargar) para ir a un archivo UCS previamente guardado que no esté en la lista.

3. Proporcione la frase de contraseña para la copia de seguridad y seleccione **Restore** (Restaurar).

```PowerShell
# Authenticate to Azure
Connect-azAccount

# Set subscription by Id
Set-AzContext -SubscriptionId ‘<Azure_Subscription_ID>’

#Restore Snapshot
Get-AzVmSnapshot -ResourceGroupName '<E.g.contoso-RG>' -VmName '<E.g.BIG-IP-VM>' | Restore-AzVmSnapshot

```

>[!NOTE]
>En el momento de escribir este documento, el cmdlet AzVmSnapshot se limita a la restauración de la instantánea más reciente, según la fecha. Las instantáneas se almacenan en la raíz del grupo de recursos de la máquina virtual. Tenga en cuenta que al restaurar las instantáneas se reinicia una máquina virtual de Azure, por lo que debe hacerlo con cuidado.

## <a name="additional-resources"></a>Recursos adicionales

-   [Restablecimiento de la contraseña de BIG-IP VE en Azure](https://clouddocs.f5.com/cloud/public/v1/shared/azure_passwordreset.html)
    -   [Restablecimiento de la contraseña sin usar el portal](https://clouddocs.f5.com/cloud/public/v1/shared/azure_passwordreset.html#reset-the-password-without-using-the-portal)

-   [Cambio de la NIC usada para la administración de BIG-IP VE](https://clouddocs.f5.com/cloud/public/v1/shared/change_mgmt_nic.html)

-   [Acerca de las rutas en una sola configuración de NIC](https://clouddocs.f5.com/cloud/public/v1/shared/routes.html)

-   [Microsoft Azure: Waagent](https://clouddocs.f5.com/cloud/public/v1/azure/Azure_waagent.html)

## <a name="next-steps"></a>Pasos siguientes

Seleccione un [escenario de implementación](f5-aad-integration.md) e inicie la implementación.
