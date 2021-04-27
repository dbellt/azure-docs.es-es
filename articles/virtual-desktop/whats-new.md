---
title: Novedades de Windows Virtual Desktop - Azure
description: Nuevas características y actualizaciones de productos para Windows Virtual Desktop.
author: Heidilohr
ms.topic: overview
ms.date: 04/08/2021
ms.author: helohr
ms.reviewer: thhickli; darank
manager: femila
ms.custom: references_regions
ms.openlocfilehash: 242dd2476783a3d1db6b74452d35e87c5cd189b6
ms.sourcegitcommit: db925ea0af071d2c81b7f0ae89464214f8167505
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2021
ms.locfileid: "107516559"
---
# <a name="whats-new-in-windows-virtual-desktop"></a>Novedades de Windows Virtual Desktop

Windows Virtual Desktop se actualiza periódicamente. En este artículo encontrará información sobre:

- Las actualizaciones más recientes
- Nuevas características
- Mejoras de las características existentes
- Corrección de errores

Este artículo se actualiza mensualmente. Asegúrese de volver aquí con frecuencia para mantenerse al día con las nuevas actualizaciones.

## <a name="client-updates"></a>Actualizaciones de clientes

Consulte estos artículos para obtener información acerca de las actualizaciones de nuestros clientes para Windows Virtual Desktop y Servicios de Escritorio remoto:

- [Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-whatsnew)
- [macOS](/windows-server/remote/remote-desktop-services/clients/mac-whatsnew)
- [iOS](/windows-server/remote/remote-desktop-services/clients/ios-whatsnew)
- [Android](/windows-server/remote/remote-desktop-services/clients/android-whatsnew)
- [Web](/windows-server/remote/remote-desktop-services/clients/web-client-whatsnew)

## <a name="windows-virtual-desktop-agent-updates"></a>Actualizaciones del agente de Windows Virtual Desktop

El agente de Windows Virtual Desktop se actualiza al menos una vez al mes.

Estos son los cambios que se han realizado en el agente Windows Virtual Desktop:

- Versión 1.0.2990.800. Esta actualización se publicó el 13 de abril de 2021 e incorpora los cambios siguientes:
    - Los mensajes de error del agente se han actualizado.
    - Se ha agregado una excepción que impide instalar agentes que no son de Windows 7 en máquinas virtuales Windows 7.
    - Se ha actualizado la lógica del servicio de latido.
- Versión 1.0.2944.1400. Esta actualización se publicó el 7 de abril de 2021 e incorpora los cambios siguientes:
    - Se han incluido vínculos a la guía de solución de problemas del Agente de Windows Virtual Desktop en los registros del visor de eventos para los errores del agente.
    - Se ha agregado una nueva excepción para tener un mejor control de los errores.
    - Se ha agregado WVDAgentUrlTool.exe, que permite a los clientes comprobar las direcciones URL necesarias a las pueden acceder.
- Versión 1.0.2866.1500. Esta actualización se publicó el 26 de marzo de 2021 y corrige un problema relacionado con la comprobación del estado de la pila.
- Versión 1.0.2800.2802. Esta actualización se publicó el 10 de marzo de 2021 e incorpora correcciones de errores y mejoras generales.
- Versión 1.0.2800.2800. Esta actualización se publicó el 2 marzo de 2021 y corrige un problema de conexión inversa.
- Versión 1.0.2800.2700. Esta actualización se publicó el 10 de febrero de 2021 e incorpora correcciones de errores y mejoras generales.
- Versión 1.0.2800.2700. Esta actualización se publicó el 4 de febrero de 2021 y corrige un problema de orquestación de acceso denegado.

## <a name="fslogix-updates"></a>Actualizaciones de FSLogix

¿Desea conocer las últimas actualizaciones de FSLogix? Consulte las [novedades de FSLogix](/fslogix/whats-new).

## <a name="march-2021"></a>Marzo de 2021

A continuación, se incluyen los cambios de marzo de 2021.

### <a name="updates-to-the-azure-portal-ui-for-windows-virtual-desktop"></a>Cambios en la interfaz de usuario de Azure Portal para Windows Virtual Desktop

Hemos efectuado los siguientes cambios en Windows Virtual Desktop para Azure Portal:

- Hemos habilitado nuevas opciones de disponibilidad (zonas y conjunto de disponibilidad) para que los flujos de trabajo creen grupos host y agreguen máquinas virtuales.
- Hemos corregido un problema por el que un host cuyo estado indica que "necesita asistencia" aparecía como no disponible. Ahora aparecerá un icono de advertencia junto al host.
- Hemos habilitado la ordenación de sesiones activas.
- Ahora es posible enviar mensajes a usuarios específicos o cerrar la sesión en la pestaña de detalles del host.
- Hemos cambiado el campo de límite máximo de sesión.
- Hemos agregado una ruta de validación de unidades organizativas (UO) al flujo de trabajo para crear un grupo host.
- Ahora se puede usar la versión más reciente de la imagen Windows 10 al crear un grupo host personal.

### <a name="generation-2-images-and-trusted-launch"></a>Imágenes de generación 2 e inicio seguro

Azure Marketplace ahora tiene imágenes de generación 2 para Windows 10 Enterprise y la sesión múltiple de Windows 10 Enterprise. Estas imágenes le permitirán usar máquinas virtuales con inicio seguro. Obtenga más información sobre las máquinas virtuales de generación 2 en [¿Debería crear una máquina virtual de generación 1 o 2?](../virtual-machines/generation-2.md) Para obtener información sobre el aprovisionamiento de máquinas virtuales de con inicio seguro de Windows Virtual Desktop, consulte [nuestra publicación de TechCommunity](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/windows-virtual-desktop-support-for-trusted-launch/m-p/2206170).

### <a name="fslogix-is-now-preinstalled-on-windows-10-enterprise-multi-session-images"></a>FSLogix se ha preinstalado en imágenes multisesión de Windows 10 Enterprise

Teniendo en cuenta los comentarios de los clientes, hemos incorporado una nueva versión de la imagen multisesión de Windows 10 Enterprise que incluye una versión preinstalada y sin configurar de FSLogix. De este modo, esperamos que la implementación de Windows Virtual Desktop se más sencilla.

### <a name="azure-monitor-for-windows-virtual-desktop-is-now-in-general-availability"></a>Azure Monitor para Windows Virtual Desktop se encuentra ahora disponible con carácter general

Azure Monitor para Windows Virtual Desktop se encuentra ahora disponible con carácter general para el público. Esta característica es un servicio automatizado que supervisa las implementaciones y permite ver eventos y sugerencias de solución de problemas y mantenimiento en un mismo lugar. Para obtener más información, consulte [nuestra documentación](azure-monitor.md) o la [publicación de TechCommunity](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/azure-monitor-for-windows-virtual-desktop-is-generally-available/m-p/2242861).

### <a name="march-2021-updates-for-teams-on-windows-virtual-desktop"></a>Cambios de marzo de 2021 para Teams en Windows Virtual Desktop

Hemos efectuado los siguientes cambios para Teams en Windows Virtual Desktop:

- Hemos mejorado la calidad de vídeo para las llamadas y el modo 2x2.
- Hemos reducido el uso de CPU entre un 5 % y un 10 % (dependiendo de la generación de CPU) mediante la descarga de hardware del procesamiento de vídeo (XVP).
- Las máquinas más antiguas ahora pueden usar XVP y la decodificación de hardware para mostrar eficazmente más secuencias de vídeo entrantes en el modo 2x2.
- Hemos actualizado la pila de WebRTC de M74 a M88 para mejorar la sincronización de AV y reducir los problemas de transición.
- Hemos reemplazado nuestro codificador de software H264 por OpenH264 (OSS utilizado por Teams en Internet), que ha aumentado la calidad del vídeo para la cámara de salida.
- Hemos habilitado el modo 2x2 para Teams Server para el público general desde el 30 de marzo. El modo 2x2 muestra hasta cuatro secuencias de vídeo entrantes al mismo tiempo.

### <a name="start-vm-on-connect-public-preview"></a>Iniciar VM al establecer la conexión (versión preliminar pública)

La nueva configuración del grupo host, Iniciar VM al establecer la conexión, ya está disponible en versión preliminar pública. Esta configuración permite activar las máquinas virtuales siempre que las necesite. Si desea ahorrar costos, desasigne las máquinas virtuales mediante la configuración de Azure Compute. Para más información, consulte [nuestra entrada de blog](https://aka.ms/wvdstartvmonconnect) y la [documentación](start-virtual-machine-connect.md).

### <a name="windows-virtual-desktop-specialty-certification"></a>Certificación especializada de Windows Virtual Desktop

Hemos publicado una versión beta del examen AZ-140 que le permitirá acreditar sus conocimientos sobre Windows Virtual Desktop en Azure. Para más información, consulte [nuestra publicación de TechCommunity](https://techcommunity.microsoft.com/t5/microsoft-learn-blog/beta-exam-prove-your-expertise-in-windows-virtual-desktop-on/ba-p/2147107).

## <a name="february-2021"></a>Febrero de 2021

Estos son los cambios que han tenido lugar en febrero de 2021.

### <a name="portal-experience"></a>Experiencia del portal

Hemos mejorado la experiencia de Azure Portal de las siguientes maneras:

- Modo de purga masiva en hosts en la pestaña de cuadrícula del host de sesión. 
- La asociación de aplicaciones en formato MSIX está ahora disponible en versión preliminar pública.
- Información general del grupo de hosts fijo para el modo oscuro.

### <a name="eu-metadata-storage-now-in-public-preview"></a>Almacenamiento de metadatos de EU ahora en versión preliminar pública

Ahora hospedamos una versión preliminar pública de la geografía de Europa (EU) como opción de almacenamiento para los metadatos de servicio en Windows Virtual Desktop. Los clientes pueden elegir entre Oeste o Norte de Europa al crear sus objetos de servicio. Los objetos de servicio y los metadatos de los grupos de hosts se almacenarán en la geografía de Azure asociada a cada región. Para obtener más información, lea [la entrada de blog que anuncia la versión preliminar pública](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/announcing-public-preview-of-windows-virtual-desktop-service/m-p/2143939).

### <a name="teams-on-windows-virtual-desktop-plugin-updates"></a>Teams en la actualización del complemento Windows Virtual Desktop

Ha mejorado la calidad de las llamadas de vídeo en el complemento Windows Virtual Desktop gracias a haber solucionado los problemas que se notifican con más frecuencia, como el hecho de que la pantalla se quede oscura de repente o que la imagen y el sonido dejen de estar sincronizados. Estas mejoras deberían aumentar el rendimiento de la vista de un solo vídeo con cambio de altavoz activo. También se ha corregido un problema que hacía que los dispositivos de hardware con caracteres especiales no estuvieran disponibles en Teams.

## <a name="january-2021"></a>Enero de 2021

Estos son los cambios que han tenido lugar en enero de 2021:

### <a name="new-windows-virtual-desktop-offer"></a>Nueva oferta de Windows Virtual Desktop

Los clientes nuevos se ahorrarán un 30 % en los costos de Windows Virtual Desktop para las máquinas virtuales de las series D y B hasta un máximo de 90 días al usar la solución nativa de Microsoft. Puede canjear esta oferta en Azure Portal antes del 31 de marzo de 2021. Obtenga más información en nuestra [página de la oferta de Windows Virtual Desktop](https://azure.microsoft.com/services/virtual-desktop/offer/).

### <a name="networksecuritygrouprules-value-change"></a>Cambio del valor de networkSecurityGroupRules 

En la plantilla anidada de Azure Resource Manager, hemos cambiado el valor predeterminado de networkSecurityGroupRules de un objeto a una matriz. Esto evitará que se produzcan errores al usar managedDisks-customimagevm.json sin especificar un valor para networkSecurityGroupRules. No se trata de un cambio significativo y es compatible con versiones anteriores.

### <a name="fslogix-hotfix-update"></a>Actualización de revisión de FSLogix

Hemos publicado FSLogix, versión 2009 HF_01 (2.9.7654.46150) para resolver problemas de la versión anterior (2.9.7621.30127). Se recomienda dejar de usar la versión anterior y actualizar FSLogix lo antes posible.

Para obtener más información, consulte las notas de la versión en las [novedades de FSLogix](/fslogix/whats-new#fslogix-apps-2009-hf_01-29765446150).

### <a name="azure-portal-experience-improvements"></a>Mejoras de la experiencia en Azure Portal

Hemos efectuado las siguientes mejoras de la experiencia en Azure Portal:

- Ahora se pueden agregar directamente credenciales de administrador local de máquina virtual, en lugar de tener que agregar una cuenta local creada con las credenciales de la cuenta de unión de dominio de Active Directory.
- Los usuarios ahora pueden mostrar asignaciones individuales y de grupo en pestañas separadas para usuarios y grupos individuales.
- El número de versión del agente de Windows Virtual Desktop ahora es visible en la información general de la máquina virtual para los grupos host.
- Eliminación masiva agregada para grupos de hosts y grupos de aplicaciones.
- Ahora es posible habilitar o deshabilitar el modo de purga para múltiples hosts de sesión en un grupo de hosts.
- El campo de IP pública se ha eliminado de la página de detalles de máquina virtual.

### <a name="windows-virtual-desktop-agent-troubleshooting"></a>Solución de problemas del agente de Windows Virtual Desktop

Recientemente se ha redactado la [guía de solución de problemas del agente de Windows Virtual Desktop](troubleshoot-agent.md) para ayudar a los clientes que han experimentado problemas comunes.

### <a name="microsoft-defender-for-endpoint-integration"></a>Integración de Microsoft Defender for Endpoint

La integración de Microsoft Defender for Endpoint ya está disponible con carácter general. Esta característica permite que las máquinas virtuales de Windows Virtual Desktop proporcionen la misma experiencia de investigación que una máquina local de Windows 10. Si usa Windows 10 Enterprise multisesión, Microsoft Defender for Endpoint admitirá hasta 50 conexiones de usuario simultáneas. De este modo, ahorrará en el costo de Windows 10 multisesión y disfrutará de la seguridad de Microsoft Defender for Endpoint. Para más información, consulte [nuestra entrada en el blog](https://techcommunity.microsoft.com/t5/microsoft-defender-for-endpoint/windows-virtual-desktop-support-is-now-generally-available/ba-p/2103712).

### <a name="azure-security-baseline-for-windows-virtual-desktop"></a>Línea de base de seguridad de Azure para Windows Virtual Desktop

Recientemente hemos publicado un [artículo sobre la línea de base de seguridad de Azure](security-baseline.md) para Windows Virtual Desktop que nos gustaría que consultara. Estas directrices incluyen información sobre cómo aplicar Azure Security Benchmark 2.0 en Windows Virtual Desktop. Azure Security Benchmark describe la configuración y las procedimientos que recomendamos para usar las soluciones en la nube de forma segura en Azure.

## <a name="december-2020"></a>Diciembre de 2020

Estos son los cambios que han tenido lugar en diciembre de 2020: 

### <a name="azure-monitor-for-windows-virtual-desktop"></a>Azure Monitor para Windows Virtual Desktop

Ya está disponible la versión preliminar pública de Azure Monitor para Windows Virtual Desktop. Esta nueva característica incluye un sólido panel basado en los libros de Azure Monitor que ayuda a los profesionales de TI a conocer sus entornos de Windows Virtual Desktop. Para más información, consulte [el anuncio en nuestro blog](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/azure-monitor-for-windows-virtual-desktop-public-preview/m-p/1946587). 

### <a name="azure-resource-manager-template-change"></a>Cambio de plantilla de Azure Resource Manager 

En la actualización más reciente, hemos quitado todos los parámetros de dirección IP pública de la plantilla de Azure Resource Manager para crear y aprovisionar grupos de hosts. Se recomienda encarecidamente evitar el uso de IP públicas para Windows Virtual Desktop, con el fin de mantener la implementación segura. Si para la implementación se han usado IP públicas, será preciso cambiar la configuración para que use direcciones IP privadas; de lo contrario, la implementación no funcionará correctamente.

### <a name="msix-app-attach-public-preview"></a>Versión preliminar pública de conexión de aplicaciones MSIX 

Conexión de aplicaciones MSIX es otro servicio que ha comenzado su versión preliminar pública este mes. Conexión de aplicaciones MSIX es un servicio que presenta dinámicamente aplicaciones MSIX en las máquinas virtuales de host de sesión de Windows Virtual Desktop. Para más información, consulte [el anuncio en nuestro blog](https://techcommunity.microsoft.com/t5/windows-virtual-desktop/msix-app-attach-azure-portal-integration-public-preview/m-p/1986231). 

### <a name="screen-capture-protection"></a>Protección de la captura de pantalla 

Este mes también ha marcado el comienzo de la versión preliminar de la protección de la captura de pantalla. Esta característica se puede usar para evitar que se capture información confidencial en los puntos de conexión de cliente. Para probar la protección de la captura de pantalla, vaya a [esta página](https://aka.ms/WVDScreenCaptureProtection).  

### <a name="built-in-roles"></a>Roles integrados

Hemos agregado nuevos roles integrados a Windows Virtual Desktop para permisos de administrador. Para más información, consulte [Roles integrados de Windows Virtual Desktop](rbac.md). 

### <a name="application-group-limit-increase"></a>Aumento del límite de grupos de aplicaciones

Hemos aumentado el límite predeterminado de grupos de aplicaciones por inquilino de Azure Active Directory a 200 grupos.

### <a name="client-updates-for-december-2020"></a>Actualizaciones del cliente de diciembre de 2020

Hemos lanzado nuevas versiones de los siguientes clientes: 

- Android
- macOS
- Windows

Para más información sobre las actualizaciones de los clientes, consulte [Actualizaciones de clientes](whats-new.md#client-updates).

## <a name="november-2020"></a>Noviembre de 2020

### <a name="azure-portal-experience"></a>Experiencia de Azure Portal

Hemos corregido dos errores en Azure Portal:

- El nombre descriptivo de la aplicación de escritorio ya no se sobrescribe en el flujo de trabajo "Agregar máquina virtual".
- La pestaña del host de sesión se cargará si los hosts de sesión forman parte de conjuntos de escalado.

### <a name="fslogix-client-version-2009"></a>Cliente de FSLogix, versión 2009 

Se ha publicado una nueva versión del cliente FSLogix con muchas correcciones y mejoras. En [nuestra entrada de blog](https://social.msdn.microsoft.com/Forums/en-US/defe5828-fba4-4715-a68c-0e4d83eefa6b/release-notes-for-fslogix-apps-release-2009-29762130127?forum=FSLogix) encontrará más información.

### <a name="rdp-shortpath-public-preview"></a>Versión preliminar pública de RDP Shortpath

RDP Shortpath introduce una conectividad directa con el host de sesión de Windows Virtual Desktop tanto mediante redes virtuales privadas de punto a sitio y de sitio a sitio, como mediante ExpressRoute. También presenta el protocolo de transporte URCP. RDP Shortpath está diseñado para reducir la latencia y los saltos en la red, y así mejorar la experiencia del usuario. Obtenga más información al respecto en [RDP Shortpath de Windows Virtual Desktop](shortpath.md).

### <a name="azdesktopvirtualization-version-201"></a>Az.DesktopVirtualization, versión 2.0.1

Se ha publicado la versión 2.0.1 de los cmdlets de Windows Virtual Desktop. Esta actualización incluye cmdlets que permitirán administrar la asociación de aplicaciones en formato .MSIX. La nueva versión se puede descargar en [la galería de PowerShell](https://www.powershellgallery.com/packages/Az.DesktopVirtualization/2.0.1).

### <a name="azure-advisor-updates"></a>Actualizaciones de Azure Advisor

Azure Advisor ya tiene una nueva recomendación para la guía de proximidad en Windows Virtual Desktop y una nueva recomendación para optimizar el rendimiento en los grupos de hosts con equilibrio de carga con equilibrio de carga en profundidad. Obtenga más información en [el sitio web de Azure](https://azure.microsoft.com/updates/new-recommendations-from-azure-advisor/).

## <a name="october-2020"></a>Octubre de 2020

Estos son los cambios que han tenido lugar en octubre de 2020:

### <a name="improved-performance"></a>rendimiento mejorado.

- Hemos optimizado el rendimiento mediante la reducción de la latencia de conexión en las siguientes ubicaciones geográficas de Azure:
    - Suiza
    - Canadá

Ahora puede usar el [estimador de experiencia](https://azure.microsoft.com/services/virtual-desktop/assessment/) para estimar la calidad de la experiencia del usuario en estas áreas.

### <a name="azure-government-cloud-availability"></a>Disponibilidad de Azure Government Cloud

Azure Government Cloud ya está disponible con carácter general. En [nuestra entrada de blog](https://azure.microsoft.com/updates/windows-virtual-desktop-is-now-generally-available-in-the-azure-government-cloud/) encontrará más información.

### <a name="windows-virtual-desktop-azure-portal-updates"></a>Actualizaciones de Windows Virtual Desktop en Azure Portal

Se han realizado varias actualizaciones de Windows Virtual Desktop en Azure Portal:

- Se corrigió un error de resourceID que impedía a los usuarios abrir la pestaña "Sesiones".
- Se ha simplificado la interfaz de usuario en la pestaña "Hosts de sesión".
- Se han corregido los valores "Valores predeterminados", "Facilidad de uso" y "Restaurar valores predeterminados" en las propiedades del RDP.
- Las funciones "Quitar" y "Eliminar" aparecen en todas las pestañas.
- El portal ahora valida los nombres de las aplicaciones en el flujo de trabajo "Agregar una aplicación".
- Se corrigió el problema de que los datos exportados del host de sesión no estaban alineados en las columnas.
- Se corrigió el problema de que el portal no puede recuperar las sesiones del usuario.
- Se corrigió un problema en la recuperación del host de sesión que se producía cuando la máquina virtual se creaba en otro grupo de recursos.
- Se actualizó la pestaña "Host de sesión", ahora muestra tanto las sesiones activas como las desconectadas.
- La pestaña "Aplicaciones" ahora tiene páginas.
- Se corrigió el problema de que el texto "requiere línea de comandos" no se mostraba correctamente en la pestaña "Lista de aplicaciones".
- Se corrigió el problema de que el portal no podía implementar grupos de hosts ni máquinas virtuales si se usaba la versión en alemán de Shared Image Gallery.

### <a name="client-updates-for-october-2020"></a>Actualizaciones del cliente de octubre de 2020

Hemos lanzado nuevas versiones de los clientes. Para obtener más información, vea estos artículos:

- [Windows](/windows-server/remote/remote-desktop-services/clients/windowsdesktop-whatsnew)
- [iOS](/windows-server/remote/remote-desktop-services/clients/ios-whatsnew)

Para más información sobre los otros clientes, consulte [Actualizaciones de clientes](#client-updates).

## <a name="september-2020"></a>Septiembre de 2020

Estos son los cambios que se han producido en septiembre de 2020:

- Hemos optimizado el rendimiento mediante la reducción de la latencia de conexión en las siguientes ubicaciones geográficas de Azure:
    - Alemania
    - Sudáfrica (solo para entornos de validación)

Ahora puede usar el [estimador de experiencia](https://azure.microsoft.com/services/virtual-desktop/assessment/) para estimar la calidad de la experiencia del usuario en estas áreas.

- Se ha publicado la versión 1.2.1364 del cliente de escritorio de Windows para Windows Virtual Desktop. En esta actualización hemos realizado los cambios siguientes:
    - Se corrigió un problema por el que el inicio de sesión único (SSO) no funcionaba en Windows 7.
    - Se corrigió un problema que hacía que el cliente se desconectara cuando un usuario que habilitaba la optimización de medios para los equipos intentaba llamar o unirse a una reunión de Teams mientras otra aplicación tenía una secuencia de audio abierta en modo exclusivo.
    - Se corrigió un problema por el que Teams no enumeraba dispositivos de audio o vídeo cuando se habilitaba la optimización de medios para Teams.
    - A agregó un vínculo "¿Necesita ayuda con la configuración?" a la página de configuración del escritorio.
    - Se corrigió un problema con el botón "Suscribirse" que se producía al usar los temas oscuros de contraste alto.
    
- Gracias a la enorme ayuda de nuestros usuarios, hemos corregido dos problemas críticos para el Cliente de Escritorio remoto de Microsoft Store. Seguiremos revisando los comentarios y solucionaremos los problemas a medida que vayamos ampliando nuestra versión en fases del cliente a más usuarios por todo el mundo.
    
- Hemos agregado una nueva característica que le permite cambiar la ubicación de la máquina virtual, la imagen, el grupo de recursos, el nombre del prefijo y la configuración de red como parte del flujo de trabajo de incorporación de una máquina virtual a la implementación de Azure Portal.

- Los profesionales de TI ahora pueden administrar máquinas virtuales híbridas de Windows 10 Enterprise conectadas a Azure Active Directory con Microsoft Endpoint Manager. Para más información, consulte nuestra [entrada del blog](https://techcommunity.microsoft.com/t5/microsoft-endpoint-manager-blog/microsoft-endpoint-manager-announces-support-for-windows-virtual/ba-p/1681048).

## <a name="august-2020"></a>Agosto de 2020

Estos son los cambios que se han producido en agosto de 2020:

- Hemos mejorado el rendimiento para reducir la latencia de conexión en las siguientes regiones de Azure: 

    - Reino Unido
    - Francia
    - Noruega
    - Corea del Sur

   Puede usar el [Estimador de experiencia](https://azure.microsoft.com/services/virtual-desktop/assessment/) para obtener una idea general de la forma en que estos cambios afectarán a sus usuarios.

- El cliente de Escritorio remoto de Microsoft Store (v 10.2.1522 +) ya está disponible con carácter general. Esta versión del cliente de Escritorio remoto de Microsoft Store es compatible con Windows Virtual Desktop. También hemos incorporado flujos de interfaz de usuario actualizados para mejorar las experiencias de los usuarios. Esta actualización incluye un diseño fluido, modos claros y oscuros, y muchos otros cambios atractivos. También hemos reescrito el cliente para que use el mismo motor de protocolo de escritorio remoto (RDP) subyacente que los clientes iOS, macOS y Android, lo que nos permite ofrecer nuevas características a mayor velocidad en todas las plataformas. [Descargue el cliente](https://www.microsoft.com/p/microsoft-remote-desktop/9wzdncrfj3ps?rtc=1&activetab=pivot:overviewtab) y pruébelo.

- Hemos solucionado un problema en el cliente de escritorio de Teams (versión 1.3.00.21759) en el que el cliente solo mostraba la zona horaria UTC en el chat, los canales y el calendario. El cliente actualizado muestra ahora la zona horaria de la sesión remota.

- Ahora, Azure Advisor forma parte de Windows Virtual Desktop. Al acceder a Windows Virtual Desktop a través de Azure Portal, puede ver recomendaciones para optimizar el entorno de Windows Virtual Desktop. Más información en [Azure Advisor](azure-advisor.md).

- La CLI de Azure ahora admite Windows Virtual Desktop (`az desktopvirtualization`) para ayudarle a automatizar las implementaciones de Windows Virtual Desktop. Consulte [desktopvirtualization](/cli/azure/ext/desktopvirtualization/) para ver una lista de los comandos de la extensión.

- Hemos actualizado nuestras plantillas de implementación para que sean totalmente compatibles con las interfaces de Azure Resource Manager de Windows Virtual Desktop. Puede encontrar las plantillas en [GitHub](https://github.com/Azure/RDS-Templates/tree/master/ARM-wvd-templates).

- El portal de US Gov de Windows Virtual Desktop se encuentra ahora en versión preliminar pública. Para más información, consulte [nuestro anuncio](https://azure.microsoft.com/updates/windows-virtual-desktop-is-now-available-in-the-azure-government-cloud-in-preview/).

## <a name="july-2020"></a>Julio de 2020  

En julio se ha iniciado la disponibilidad con carácter general de Windows Virtual Desktop con integración de Azure Resource Management.

Esto es lo que ha cambiado con esta nueva versión: 

- La "versión Otoño 2019" se conoce ahora como "Windows Virtual Desktop (clásico)", mientras que la "versión de primavera de 2020" es ahora simplemente "Windows Virtual Desktop". Para más información, consulte [esta entrada de blog](https://azure.microsoft.com/blog/new-windows-virtual-desktop-capabilities-now-generally-available/). 

Para más información acerca de las nuevas características, consulte [esta entrada de blog](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-virtual-desktop-spring-update-enters-public-preview/ba-p/1340245). 

### <a name="autoscaling-tool-update"></a>Actualización de la herramienta de escalado automático

La versión más reciente de la herramienta de escalado automático que estaba en versión preliminar ya está disponible con carácter general. Esta herramienta usa una cuenta de Azure Automation y la aplicación lógica de Azure para apagar y reiniciar automáticamente las máquinas virtuales del host de sesión dentro de un grupo host, lo que reduce costos de infraestructura. Más información en [Escalado de hosts de sesión con Azure Automation](set-up-scaling-script.md).

### <a name="azure-portal"></a>Azure Portal

Ahora puede hacer lo siguiente con Azure Portal en Windows Virtual Desktop: 

- Asignación directa de usuarios a hosts de sesión de escritorio personal.  
- Cambio de la configuración del entorno de validación para grupos host. 

### <a name="diagnostics"></a>Diagnóstico

Hemos publicado algunas nuevas consultas precompiladas para el área de trabajo de Log Analytics. Para obtener acceso a las consultas, vaya a **Registros** y, en **Categoría**, seleccione **Windows Virtual Desktop**. Más información en [Uso de Log Analytics para la característica de diagnóstico](diagnostics-log-analytics.md).

### <a name="update-for-remote-desktop-client-for-android"></a>Actualización para el cliente de Escritorio remoto para Android

El [cliente de Escritorio remoto para Android](https://play.google.com/store/apps/details?id=com.microsoft.rdc.androidx) ahora admite las conexiones de Windows Virtual Desktop. A partir de la versión 10.0.7, el cliente de Android ofrece una nueva interfaz de usuario para mejorar la experiencia del usuario. El cliente también se integra con Microsoft Authenticator en los dispositivos Android para habilitar el acceso condicional cuando se realiza una suscripción a áreas de trabajo de Windows Virtual Desktop.  

La versión anterior del cliente de Escritorio remoto se denomina ahora "Escritorio remoto 8". Las conexiones que ya tenga en la versión anterior del cliente se transferirán sin problemas al nuevo cliente. El nuevo cliente se ha vuelto a escribir en el mismo motor de RDP Core subyacente que los clientes de iOS y macOS, lo que agiliza la versión de las nuevas características en todas las plataformas. 

### <a name="teams-update"></a>Actualización de Teams

Hemos realizado algunas mejoras en Microsoft Teams para Windows Virtual Desktop. Lo más importante es que Windows Virtual Desktop ahora admite la optimización de audio y vídeo para el cliente del escritorio de Windows. El redireccionamiento mejora la latencia, con la creación de rutas de acceso directas entre usuarios cuando estos realizan llamadas de audio o vídeo y conferencias. Una Menor distancia significa menos saltos, lo que mejora tanto la imagen como el sonido en las llamadas. Más información en [Uso de Teams en Windows Virtual Desktop](teams-on-wvd.md).

## <a name="june-2020"></a>Junio de 2020

El mes pasado, presentamos la versión preliminar de la integración de Windows Virtual Desktop con Azure Resource Manager. Esta actualización tiene muchas características nuevas y atractivas de las que nos encantaría informarle. Estas son las novedades de esta versión de Windows Virtual Desktop.

### <a name="windows-virtual-desktop-is-now-integrated-with-azure-resource-manager"></a>Windows Virtual Desktop ahora se integra con Azure Resource Manager

Windows Virtual Desktop ahora se integra en Azure Resource Manager. En la última actualización, todos los objetos de Windows Virtual Desktop pasan a ser recursos de Azure Resource Manager. Esta actualización también se integra con los controles de acceso basado en rol (Azure RBAC) de Azure. Para más información, consulte [¿Qué es Azure Resource Manager?](../azure-resource-manager/management/overview.md).

Esto es lo que hace este cambio:

- Windows Virtual Desktop ahora está integrado en Azure Portal, lo que significa que puede administrar todo directamente en el portal, no se requieren PowerShell, aplicaciones web o herramientas de terceros. Para empezar, consulte nuestro tutorial en [Creación de un grupo de hosts con Azure Portal](create-host-pools-azure-marketplace.md).

- Antes de esta actualización, solo podía publicar RemoteApps y Desktops para usuarios individuales. Con Azure Resource Manager, ahora puede publicar recursos en grupos de Azure Active Directory.

- La versión anterior de Windows Virtual Desktop tenía cuatro roles de administrador integrados que podría asignar a un inquilino o a un grupo de hosts. Estos roles están ahora en el [Control de acceso basado en rol de Azure (Azure RBAC)](../role-based-access-control/overview.md). Estos roles se pueden a todos los objetos de Azure Resource Manager de Windows Virtual Desktop, lo que le permite tener un modelo completo de delegación enriquecida.

- En esta actualización, ya no preciso ejecutar Azure Marketplace o la plantilla de GitHub varias veces para expandir un grupo de hosts. Lo único que se necesita para expandir un grupo de hosts es ir a él en Azure Portal y seleccionar **+ Agregar** para implementar hosts de sesión adicionales.

- Ahora, la implementación de los grupos de hosts está completamente integrada en [Azure Shared Image Gallery](../virtual-machines/shared-image-galleries.md). Shared Image Gallery es un servicio de Azure independiente que almacena definiciones de imágenes de máquina virtual (VM), incluidas las versiones de las imágenes. También puede usar la replicación global para copiar y enviar las imágenes a otras regiones de Azure para su implementación local.

- Las funciones de supervisión que solían realizarse a través de PowerShell o de la aplicación web del servicio de diagnóstico se han desplazado a Log Analytics en Azure Portal. Ahora también tiene dos opciones para visualizar los informes. Puede ejecutar consultas Kusto y usar Workbooks para crear informes visuales.

- Ya no es preciso completar el consentimiento de Azure Active Directory (Azure AD) para usar Windows Virtual Desktop. En esta actualización, el inquilino de Azure AD de su suscripción de Azure autentica a los usuarios y proporciona controles de Azure RBAC a los administradores.

### <a name="powershell-support"></a>Compatibilidad con PowerShell

Hemos agregado nuevos cmdlets AzWvd al módulo Az de Azure PowerShell con esta actualización. Este nuevo módulo es compatible con PowerShell Core, que se ejecuta en .NET Core.

Para instalar el módulo, siga las instrucciones de [Configuración del módulo de PowerShell para Windows Virtual Desktop](powershell-module.md).

También puede ver una lista de los comandos disponibles en la [referencia de AzWvd PowerShell](/powershell/module/az.desktopvirtualization/#desktopvirtualization).

Para más información acerca de las nuevas características, consulte [nuestra entrada del blog](https://techcommunity.microsoft.com/t5/itops-talk-blog/windows-virtual-desktop-spring-update-enters-public-preview/ba-p/1340245).

### <a name="additional-gateways"></a>Puertas de enlace adicionales

Hemos agregado un nuevo clúster de puerta de enlace en Sudáfrica para reducir la latencia de conexión.

### <a name="microsoft-teams-on-windows-virtual-desktop-preview"></a>Microsoft Teams en Windows Virtual Desktop (versión preliminar)

Hemos realizado algunas mejoras en Microsoft Teams para Windows Virtual Desktop. Y lo que es más importante, Windows Virtual Desktop ahora admite el redireccionamiento de audio y visual para las llamadas. El redireccionamiento mejora la latencia, ya que crea rutas de acceso directas entre usuarios cuando llaman mediante audio o vídeo. Una Menor distancia significa menos saltos, lo que mejora tanto la imagen como el sonido en las llamadas.

Para más información, consulte nuestra [entrada del blog](https://azure.microsoft.com/updates/windows-virtual-desktop-media-optimization-for-microsoft-teams-is-now-available-in-public-preview/).

## <a name="next-steps"></a>Pasos siguientes

Obtenga información sobre el futuro de [Windows Virtual Desktop en el plan de desarrollo de Microsoft 365](https://www.microsoft.com/microsoft-365/roadmap?filters=Windows%20Virtual%20Desktop).
