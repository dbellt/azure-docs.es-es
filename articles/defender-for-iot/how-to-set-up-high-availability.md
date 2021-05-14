---
title: Configuración de la alta disponibilidad
description: Aumente la resistencia de la implementación de Defender para IoT mediante la instalación de un dispositivo de alta disponibilidad de la consola de administración local. Las implementaciones de alta disponibilidad garantizan que los sensores administrados informen continuamente a una consola de administración local activa.
ms.date: 12/07/2020
ms.topic: how-to
ms.openlocfilehash: f341dee00574422ec325871ff97d6f3e8644ac48
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108209002"
---
# <a name="about-high-availability"></a>Acerca de la alta disponibilidad

Aumente la resistencia de la implementación de Defender para IoT mediante la instalación de un dispositivo de alta disponibilidad de la consola de administración local. Las implementaciones de alta disponibilidad garantizan que los sensores administrados informen continuamente a una consola de administración local activa.

Esta implementación se realiza con un par de la consola de administración local que incluye un dispositivo principal y otro secundario.

## <a name="about-primary-and-secondary-communication"></a>Acerca de la comunicación principal y secundaria

Cuando se emparejan una consola de administración local principal y secundaria:

- Se aplica un certificado SSL de la consola de administración local para crear una conexión segura entre el dispositivo principal y el secundario. El SSL puede ser el certificado autofirmado instalado de manera predeterminada o un certificado instalado por el cliente.

- Se realiza una copia de seguridad automática de los datos de la consola de administración local principal en la secundaria de forma automática cada 10 minutos. Se realiza una copia de seguridad de las configuraciones de la consola de administración local y de los datos del dispositivo. Los archivos PCAP y los registros no se incluyen en la copia de seguridad. Puede realizar copias de seguridad de los archivos PCAP y los registros y restaurarlos manualmente.

- La configuración principal de la consola de administración se duplica en la secundaria; por ejemplo, la configuración del sistema. Si esta configuración se actualiza en la consola principal, también lo hace en la secundaria.

- Antes de que expire la licencia de la consola secundaria, debe definirla como principal para poder actualizar la licencia.

## <a name="about-failover-and-failback"></a>Acerca de la conmutación por error y la conmutación por recuperación

Si un sensor no puede conectarse a la consola de administración local principal, se conecta automáticamente a la secundaria. El sistema se admite tanto en la principal como en la secundaria de manera simultánea siempre que menos de la mitad de los sensores se comuniquen con la secundaria. La secundaria toma el control cuando más de la mitad de los sensores se comunican con ella. La conmutación por error de la principal a la secundaria tarda aproximadamente tres minutos. Cuando se produce la conmutación por error, la consola de administración local principal se inmoviliza. Cuando ocurre esto, puede iniciar sesión en la secundaria con las mismas credenciales de inicio de sesión.

Durante la conmutación por error, los sensores continúan intentando comunicarse con el dispositivo principal. Si más de la mitad de los sensores administrados se comunican correctamente con la consola principal, esta se restaura. Cuando se restaura la principal, aparece el siguiente mensaje en la consola secundaria.

:::image type="content" source="media/how-to-set-up-high-availability/secondary-console-message.png" alt-text="Captura de pantalla de un mensaje que aparece en la consola secundaria cuando se restaura la principal.":::

Vuelva a iniciar sesión en el dispositivo principal después del redireccionamiento.

## <a name="high-availability-setup-overview"></a>Información general sobre la configuración de alta disponibilidad

Los procedimientos de instalación y configuración se realizan en cuatro fases principales:

1. Instalación de un dispositivo principal de consola de administración local. 

2. Configuración del dispositivo principal de consola de administración local. Por ejemplo, configuración de copia de seguridad programada, configuración de VLAN. Vea la guía de usuario sobre la consola de administración local para obtener más información. Todas las configuraciones se aplican automáticamente al dispositivo secundario después del emparejamiento.

3. Instalación de un dispositivo secundario de consola de administración local. Para obtener más información, vea [Acerca de la instalación de Defender para IoT](how-to-install-software.md).

4. Emparejamiento de los dispositivos primario y secundario de consola de administración local, como se describe [aquí](https://infrascale.secure.force.com/pkb/articles/Support_Article/How-to-access-your-Appliance-Management-Console). La consola de administración local principal debe administrar al menos dos sensores para poder llevar a cabo la configuración.

## <a name="high-availability-requirements"></a>Requisitos de la alta disponibilidad

Compruebe que se cumplen los siguientes requisitos de la alta disponibilidad:

- Requisitos de certificados

- Requisitos de software y hardware

- Requisitos de acceso a la red

### <a name="software-and-hardware-requirements"></a>Requisitos de software y hardware

- Tanto el dispositivo de consola de administración local principal como el secundario deben ejecutar versiones de software y modelos de hardware idénticos.

- Únicamente los usuarios de Defender para IoT pueden configurar el sistema de alta disponibilidad mediante las herramientas de la CLI.

### <a name="network-access-requirements"></a>Requisitos de acceso a la red

Debe comprobar que la directiva de seguridad de la organización permita el acceso a los siguientes servicios en la consola de administración local principal y secundaria. Estos servicios también permiten la conexión entre los sensores y la consola de administración local secundaria:

|Port|Servicio|Descripción|
|----|-------|-----------|
|**443 o TCP**|HTTPS|Concede acceso a la consola web de la consola de administración local.|
|**22 o TCP**|SSH|Sincroniza los datos entre los dispositivos principal y secundario de consola de administración local.|
|**123 o UDP**|NTP| Sincronización de hora de NTP de la consola de administración local. Compruebe que los dispositivos activos y pasivos están definidos con la misma zona horaria.|

## <a name="create-the-primary-and-secondary-pair"></a>Creación del par principal y secundario

Compruebe que tanto el dispositivo principal como el secundario de consola de administración local están encendidos antes de iniciar el procedimiento.  

### <a name="on-the-primary"></a>En el dispositivo principal

1. Inicie sesión en la CLI como usuario de Defender para IoT.

2. Ejecute el comando siguiente en el dispositivo principal:

```azurecli-interactive
sudo cyberx-management-trusted-hosts-add -ip <Secondary IP> -token <primary token>
```

>[!NOTE]
>En este documento, la consola de administración local principal se conoce como principal y el agente se conoce como secundario.

3. Escriba la dirección IP del dispositivo secundario en el campo ```<Secondary ip>``` y seleccione Entrar. A continuación, la dirección IP se valida y el certificado SSL se descarga en la consola principal. Al escribir la dirección IP también se asocian los sensores al dispositivo secundario.

4. Ejecute el siguiente comando en la consola principal para comprobar que el certificado está instalado correctamente:

```azurecli-interactive
sudo cyberx-management-trusted-hosts-apply
```

5. Ejecute el comando siguiente en la consola principal. **No ejecute con sudo.**

```azurecli-interactive
cyberx-management-deploy-ssh-key <Secondary IP>
```

Esto permite la conexión entre los dispositivos principal y secundario para fines de copia de seguridad y restauración entre ellos.

6. Escriba la dirección IP de la consola secundaria y seleccione Entrar.

### <a name="on-the-secondary"></a>En el dispositivo secundario

1. Inicie sesión en la CLI como usuario de Defender para IoT.

2. Ejecute el siguiente comando en el dispositivo secundario. **No ejecute con sudo**:

```azurecli-interactive
cyberx-management-deploy-ssh-key <Primary ip>
```

Esto permite la conexión entre los dispositivos principal y secundario para fines de copia de seguridad y restauración entre ellos.

3. Escriba la dirección IP de la consola principal y presione Entrar.

### <a name="track-high-availability-activity"></a>Seguimiento de la actividad de alta disponibilidad

Los registros de aplicaciones principales se pueden exportar al equipo de soporte técnico de Defender para IoT para tratar cualquier problema de alta disponibilidad.  

Para acceder a los registros principales:

1. Seleccione **Exportar** en la ventana **Configuración del sistema**.

## <a name="update-the-on-premises-management-console-with-high-availability"></a>Actualización de la consola de administración local con alta disponibilidad

Realice la actualización de alta disponibilidad en el orden siguiente. Asegúrese de completar cada paso antes de comenzar uno nuevo.

Para actualizar con alta disponibilidad:

1. Actualice la consola de administración local principal.

2. Actualice la consola de administración local secundaria.

3. Actualice los sensores.

## <a name="see-also"></a>Consulte también

[Activación y configuración de la consola de administración local](how-to-activate-and-set-up-your-on-premises-management-console.md)