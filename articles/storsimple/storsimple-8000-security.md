---
title: Seguridad de la serie 8000 de StorSimple | Microsoft Docs
description: Describe las características de seguridad y privacidad que protegen el servicio y el dispositivo StorSimple y los datos locales y en la nube.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 05/18/2018
ms.author: alkohli
ms.openlocfilehash: ecfeffb490c54d24a6254961cf17b13f0b6a34bc
ms.sourcegitcommit: 2e123f00b9bbfebe1a3f6e42196f328b50233fc5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108076678"
---
# <a name="storsimple-security-and-data-protection"></a>Protección de datos y seguridad de StorSimple

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Información general

La seguridad es una preocupación clave para cualquiera que adopte una tecnología nueva, en especial cuando se utiliza la tecnología con información confidencial o de su propiedad. A medida que evalúa distintas tecnologías, debe considerar mayores riesgos y costes para la protección de datos. Microsoft Azure StorSimple proporciona una solución de seguridad y privacidad para la protección de datos, lo que le ayuda a garantizar:

* **Confidencialidad**: solo las entidades autorizadas pueden ver sus datos.
* **Integridad**: solo las entidades autorizadas modificar o eliminar sus datos.

La solución de Microsoft Azure StorSimple consta de cuatro componentes principales que interactúan entre sí:

* **Servicio StorSimple Device Manager hospedado en Microsoft Azure**: el servicio de administración que se usa para configurar y aprovisionar el dispositivo de StorSimple.
* **Dispositivo de StorSimple** : un dispositivo físico instalado en el centro de datos. Todos los hosts y los clientes que generan datos se conectan al dispositivo de StorSimple y el dispositivo administra los datos y los mueve a la nube de Azure según corresponda.
* **Clientes/hosts conectados al dispositivo** : los clientes en la infraestructura que se conectan al dispositivo de StorSimple y generan datos que se deben proteger.
* **Almacenamiento en la nube** : la ubicación en la nube de Azure donde se almacenan los datos.

Las secciones siguientes describen las características de seguridad de StorSimple que ayudan a proteger cada uno de estos componentes y los datos que se almacenan en ellos. También se incluye una lista de las preguntas que probablemente tenga acerca de la seguridad de Microsoft Azure StorSimple y las respuestas correspondientes.

## <a name="storsimple-device-manager-service-protection"></a>Protección del servicio StorSimple Device Manager

El servicio StorSimple Device Manager es un servicio de administración hospedado en Microsoft Azure y se usa para administrar todos los dispositivos de StorSimple que ha adquirido su organización. Puede tener acceso al servicio StorSimple Device Manager con sus credenciales de organización para iniciar sesión en Azure Portal mediante un explorador web.

El acceso al servicio StorSimple Device Manager requiere que la organización tenga una suscripción de Azure que incluya StorSimple. Su suscripción regula las características a las que puede tener acceso en Azure Portal. Si su organización no tiene una suscripción de Azure y desea obtener más información sobre ellas, vea [Inicio de sesión en Azure como una organización](../active-directory/fundamentals/sign-up-organization.md).

Debido a que el servicio StorSimple Device Manager está hospedado en Azure, se encuentra protegido por las características de seguridad de Azure. Para obtener más información acerca de las características de seguridad que proporciona Microsoft Azure, visite el [Centro de confianza de Microsoft Azure](https://azure.microsoft.com/support/trust-center/security/).

## <a name="storsimple-device-protection"></a>Protección del dispositivo de StorSimple

El dispositivo de StorSimple es un dispositivo de almacenamiento híbrido local que contiene unidades de estado sólido (SSD) y discos duros (HDD), en conjunto con controladores redundantes y funcionalidades de conmutación automática por error. Los controladores administran la organización en niveles del almacenamiento y ponen los datos actualmente utilizados (o activos) en el almacenamiento local (en el dispositivo de StorSimple o en servidores locales), mientras que mueven los datos usados con menos frecuencia a la nube.

Solo los dispositivos de StorSimple autorizados tienen permitido unirse al servicio StorSimple Device Manager que creó en su suscripción de Azure. Para autorizar un dispositivo, debe registrarlo con el servicio StorSimple Device Manager mediante la clave de registro del servicio. La clave de registro del servicio es una clave aleatoria de 128 bits generada en Azure Portal.

![Clave de registro del servicio](./media/storsimple-security/ServiceRegistrationKey.png)

Para obtener información sobre cómo obtener una clave de registro del servicio, vaya al [Paso 2: Obtención de la clave de registro del servicio](storsimple-8000-deployment-walkthrough-u2.md#step-2-get-the-service-registration-key).

La clave de registro del servicio es una clave larga que contiene más de 100 caracteres. Puede copiar la clave y guardarla en un archivo de texto en una ubicación segura para que pueda utilizarla para autorizar dispositivos adicionales según sea necesario. Si se pierde la clave de registro del servicio una vez que registra el primer dispositivo, puede generar una clave nueva desde el servicio StorSimple Device Manager. Esto no afectará la operación de los dispositivos existentes.

Una vez que se registra un dispositivo, usa tokens para comunicarse con Microsoft Azure. La clave de registro del servicio no se usa después del registro del dispositivo.

> [!NOTE]
> Se recomienda que regenere la clave de registro del servicio después de cada uso.


## <a name="protect-your-storsimple-solution-via-passwords"></a>Protección de la solución de StorSimple mediante contraseñas

Las contraseñas son un aspecto importante de la seguridad informática y se usan ampliamente en la solución de StorSimple para garantizar que solo usuarios autorizados puedan tener acceso a sus datos. StorSimple permite configurar las siguientes contraseñas:

* Contraseña para el administrador del dispositivo de StorSimple
* Contraseñas para iniciador y destino del Protocolo de autenticación por desafío mutuo (CHAP)
* Contraseña para StorSimple Snapshot Manager

### <a name="windows-powershell-for-storsimple-and-the-storsimple-device-administrator-password"></a>Contraseña para el administrador del dispositivo de StorSimple y Windows Powershell para StorSimple

Windows PowerShell para StorSimple es una interfaz de línea de comandos que puede utilizar para administrar el dispositivo de StorSimple. Windows PowerShell para StorSimple tiene características que le permite registrar su dispositivo, configurar la interfaz de red en su dispositivo, instalar ciertos tipos de actualizaciones, solucionar problemas con su dispositivo mediante el acceso a la sesión de soporte y cambiar el estado del dispositivo. Puede tener acceso a Windows PowerShell para StorSimple si se conecta a la consola en serie en el dispositivo o si usa la comunicación remota de Windows PowerShell.

La comunicación remota de PowerShell se puede realizar a través de HTTPS o HTTP. Si la administración remota a través de HTTPS está habilitada, deberá descargar el certificado de administración remota desde el dispositivo e instalarlo en el cliente remoto. Para obtener más información sobre la comunicación remota de PowerShell, vaya a [Conexión remota al dispositivo de StorSimple](storsimple-8000-remote-connect.md).

Después de que use Windows PowerShell para StorSimple para conectarse al dispositivo, necesitará proporcionar la contraseña de administrador del dispositivo para iniciar sesión en el dispositivo.

![Contraseña de administrador de dispositivo](./media/storsimple-security/DeviceAdminPW.png)

Tenga presente los siguientes procedimientos recomendados:

* La administración remota está desactivada de manera predeterminada. Puede usar el servicio StorSimple Device Manager para habilitarla. Como procedimiento de seguridad recomendado, el acceso remoto solo debe estar habilitado durante el periodo que realmente se necesita.
* Si cambia la contraseña, asegúrese de notificar a todos los usuarios de acceso remoto para que no experimenten una pérdida de conectividad inesperada.
* El servicio StorSimple Device Manager no puede recuperar contraseñas existentes: solo puede restablecerlas. Se recomienda almacenar todas las contraseñas en un lugar seguro para que no deba restablecer una contraseña si se le olvida. En caso de sí necesitar restablecer una contraseña, asegúrese de notificar a todos los usuarios antes de hacerlo.

Puede tener acceso a la interfaz de Windows PowerShell mediante el uso de una conexión serie con el dispositivo. También puede tener acceso remoto si utiliza HTTP o HTTPS, lo que proporciona mayor seguridad. HTTPS proporciona un nivel de seguridad más alto que una conexión serie o HTTP. Sin embargo, para usar HTTPS, primero debe instalar un certificado en el equipo cliente que tendrá acceso al dispositivo. Puede descargar el certificado de acceso remoto desde la página de configuración del dispositivo en el servicio StorSimple Device Manager. Si se pierde el certificado del acceso remoto, debe descargar un certificado nuevo y propagarlo a todos los clientes autorizados a usar administración remota.

### <a name="challenge-handshake-authentication-protocol-chap-initiator-and-target-passwords"></a>Contraseñas para iniciador y destino del Protocolo de autenticación por desafío mutuo (CHAP)

CHAP es un esquema de autenticación que el dispositivo de StorSimple utiliza para validar la identidad de los clientes remotos. La comprobación se basa en una contraseña compartida. CHAP puede ser unidireccional o mutuo (bidireccional). Con CHAP unidireccional, el destino (el dispositivo de StorSimple) autentica un iniciador (host). El protocolo CHAP mutuo o inverso requiere que el destino autentique al iniciador y después que este autentique a aquel. StorSimple se puede configurar para que utilice cualquiera de estos métodos.

Tenga en cuenta lo siguiente al configurar CHAP:

* El nombre de usuario de CHAP no puede contener más de 233 caracteres.
* La contraseña de CHAP debe contener entre 12 y 16 caracteres. Si se intenta usar un nombre de usuario o una contraseña con más caracteres, se producirá un error de autenticación en el host de Windows.
* No puede usar la misma contraseña para el iniciador de CHAP y el destino de CHAP.
* Después de establecer la contraseña, es posible cambiarla pero no se puede recuperar. Si se cambia la contraseña, asegúrese de notificar a todos los usuarios de acceso remoto para que puedan conectarse correctamente al dispositivo de StorSimple.

Para obtener más información sobre CHAP y cómo configurarlo para su solución de StorSimple, vaya a [Configuración de CHAP para el dispositivo de StorSimple](storsimple-8000-configure-chap.md).

### <a name="storsimple-snapshot-manager-password"></a>Contraseña para StorSimple Snapshot Manager

StorSimple Snapshot Manager es un complemento de Microsoft Management Console (MMC) que utiliza grupos de volúmenes y el Servicio de instantáneas de volumen de Windows para generar copias de seguridad coherentes con la aplicación. Además, puede usar StorSimple Snapshot Manager para crear programaciones de copias de seguridad y clonar o restaurar volúmenes.

Cuando configure un dispositivo para que use Snapshot Manager de StorSimple, deberá proporcionar la contraseña de dicha aplicación. Esta contraseña se establece primero en Windows PowerShell para StorSimple durante el registro. También es posible establecer y cambiar la contraseña desde el servicio StorSimple Device Manager. Esta contraseña autentica el dispositivo con StorSimple Snapshot Manager.

![Contraseña para StorSimple Snapshot Manager](./media/storsimple-security/SnapshotMgrPassword.png)

Debe tener entre 14 y 15 caracteres y debe contener tres o más combinaciones de mayúsculas, minúsculas, caracteres numéricos y caracteres especiales. Después de establecer la contraseña de StorSimple Snapshot Manager, es posible cambiarla pero no se puede recuperar. Si se cambia la contraseña, asegúrese de notificar a todos los usuarios remotos.

Para obtener más información sobre Snapshot Manager de StorSimple, vaya a [¿Qué es Snapshot Manager de StorSimple?](storsimple-what-is-snapshot-manager.md)

### <a name="password-best-practices"></a>Procedimientos recomendados sobre las contraseñas

Se recomienda usar las siguientes instrucciones para asegurarse de que las contraseñas de StorSimple sean seguras y estén bien protegidas:

* Cambie las contraseñas cada tres meses. Es obligatorio cambiar de contraseña todos los años.
* Use contraseñas seguras. Para obtener más información, vaya a [Creación de contraseñas más seguras y cómo protegerlas](https://cloudblogs.microsoft.com/microsoftsecure/2014/08/25/create-stronger-passwords-and-protect-them/).
* Utilice siempre contraseñas distintas para los diferentes mecanismos de acceso; cada una de las contraseñas que especifique debe ser única.
* No comparta las contraseñas con nadie que no esté autorizado a tener acceso al dispositivo de StorSimple.
* No hable de una contraseña frente a otras personas ni revele ninguna indicación respecto del formato de una contraseña.
* Si sospecha que se ha comprometido la seguridad de una cuenta o una contraseña, informe el incidente a su departamento de seguridad de la información.
* Considere todas las contraseñas como información confidencial y sensible. 

## <a name="storsimple-data-protection"></a>Protección de datos de StorSimple

En esta sección se describen las características de seguridad de StorSimple que protegen los datos en tránsito y los datos almacenados.

Según lo descrito en otras secciones, las contraseñas se usan para autorizar y autenticar a los usuarios antes de que puedan obtener acceso a su solución de StorSimple. Otra consideración de seguridad es la protección de los datos ante usuarios no autorizados mientras se transfieren entre sistemas de almacenamiento y mientras se almacenan. En las secciones siguientes se describen las características de protección de datos que se proporcionan con StorSimple.

> [!NOTE]
> La desduplicación proporciona protección adicional para datos almacenados en el dispositivo de StorSimple y en almacenamiento de Microsoft Azure. Cuando se desduplican los datos, los objetos de datos se almacenan de manera independiente de los metadatos usados para asignarlos y tener acceso a ellos: no hay un contexto de nivel de almacenamiento disponible para reconstruir los datos basados en la estructura del volumen, el sistema de archivos o el nombre del archivo.


## <a name="protect-data-flowing-through-the-service"></a>Protección del flujo de datos a través del servicio

El propósito principal del servicio StorSimple Device Manager es administrar y configurar el dispositivo de StorSimple. El servicio StorSimple Device Manager se ejecuta en Microsoft Azure. Azure Portal se usa para escribir los datos de configuración del dispositivo. Una vez hecho esto, Microsoft Azure usa el servicio StorSimple Device Manager para enviar los datos al dispositivo. StorSimple usa un sistema de pares de claves asimétricas para ayudar a garantizar que un compromiso del servicio de Azure no generará un compromiso de la seguridad de la información almacenada.

![Cifrado de datos a bordo](./media/storsimple-security/DataEncryption.png)

El sistema de claves asimétricas ayuda a proteger los datos que fluyen a través del servicio de la siguiente manera:

1. Un certificado de cifrado de datos que utiliza un par de claves asimétricas públicas y privadas se genera en el dispositivo y se utiliza para proteger los datos. Las claves se generar cuando se registra el primer dispositivo.
2. Las claves de certificado de cifrado de datos se exportan a un archivo de intercambio de información personal (.pfx) que está protegido por la clave de cifrado de datos del servicio, que es una clave segura de 128 bits que el primer dispositivo genera de manera aleatoria durante el registro.
3. La clave pública del certificado queda disponible de manera segura para el servicio StorSimple Device Manager y la clave privada queda dentro del dispositivo.
4. Los datos que ingresan al servicio se cifran usando la clave pública y se descifran usando la clave privada almacenada en el dispositivo, lo que garantiza que el servicio de Azure no puede descifrar los datos que fluyen al dispositivo.

La clave de cifrado de datos de servicio se genera solo en el primer dispositivo registrado con el servicio. Todos los dispositivos subsiguientes que se registran con el servicio deben usar la misma clave de cifrado de datos de servicio.

> [!IMPORTANT]
> Es muy importante crear una copia de esta clave y guardarla en una ubicación segura. Una copia de la clave de cifrado de datos de servicio se debe almacenar de manera tal para que una persona autorizada pueda tener acceso a ella y se pueda comunicar fácilmente al administrador del dispositivo.
> 
> Si se pierde la clave de cifrado de datos de servicio, una persona del soporte técnico de Microsoft puede ayudarle a recuperarla siempre que tenga al menos un dispositivo en línea. Se recomienda cambiar la clave de cifrado de datos de servicio después de recuperarla.

Para cambiar la clave de cifrado de datos de servicio y el certificado de cifrado de datos correspondiente, siga los pasos de la opción [Cambiar la clave de cifrado de datos de servicio](storsimple-8000-manage-service.md#change-the-service-data-encryption-key) para el servicio StorSimple Device Manager. Cambiar las claves de cifrado requiere que todos los dispositivos se actualicen con la clave nueva. Por lo tanto, se recomienda cambiar la clave cuando todos los dispositivos estén en línea. Si los dispositivos están sin conexión, es posible que las claves cambien en momentos distintos. Los dispositivos con claves obsoletas de todos modos podrán ejecutar copias de seguridad, pero no podrán restaurar datos hasta que se actualice la clave.

La clave de cifrado de datos de servicio y el certificado de cifrado de datos no expiran. Sin embargo, se recomienda cambiar la clave de cifrado de datos de servicio anualmente para evitar comprometer la seguridad de la clave.

## <a name="protect-data-at-rest"></a>Protección de los datos en reposo

El dispositivo de StorSimple administra los datos a través de su almacenamiento en niveles de manera local y en la nube, según la frecuencia de uso. Todos los equipos host que están conectados al dispositivo envían datos al dispositivo, el que luego mueve los datos a la nube, según corresponda. Los datos se transfieren desde el dispositivo a la nube de forma segura a través de Internet. Cada dispositivo tiene un destino de iSCSI que expone todos los volúmenes compartidos en ese dispositivo. Todos los datos se cifran antes de enviarlos al almacenamiento en la nube. 

![Clave de cifrado de almacenamiento en la nube](./media/storsimple-security/CloudStorageEncryption.png)

Para ayudar a garantizar la seguridad y la integridad de los datos migrados a la nube, StorSimple le permite definir claves de cifrado de almacenamiento en la nube de la siguiente manera:

* La clave de cifrado de almacenamiento en la nube se especifica cuando se crea un contenedor de volúmenes. No es posible modificar o agregar más adelante la clave.
* Todos los volúmenes de un contenedor de volúmenes comparten la misma clave de cifrado. Si desea una forma distinta de cifrado para un volumen específico, se recomienda crear un contenedor de volúmenes nuevo para hospedar ese volumen.
* Cuando escriba la clave de cifrado de almacenamiento en la nube en el servicio StorSimple Device Manager, esta se cifra con la parte pública de la clave de cifrado de datos de servicio y luego se envía al dispositivo.
* La clave de cifrado de almacenamiento en la nube no se almacena en ningún lugar del servicio y solo la conoce el dispositivo.
* Especificar una clave de cifrado de almacenamiento en la nube es opcional. Puede enviar datos cifrados en el host al dispositivo.

### <a name="additional-security-best-practices"></a>Prácticas recomendadas de seguridad adicionales

* División de tráfico: aísle la SAN iSCSI del tráfico de usuario en una LAN corporativa mediante la implementación de una red totalmente separada y a través de VLAN donde el aislamiento físico no es opción. Una red dedicada para almacenamiento de iSCSI garantizará la seguridad y el rendimiento de los datos críticos para la empresa. No se recomienda combinar el tráfico de usuario y almacenamiento en una LAN corporativa y puede aumentar la latencia y provocar errores en la red.
* Para la seguridad de la red en el lado host, utilice interfaces de red que admitan el motor de descarga de TCP/IP (TOE). TOE reduce la carga de CPU a través del procesamiento de TCP en el adaptador de red.

## <a name="protect-data-via-storage-accounts"></a>Protección de datos a través de cuentas de almacenamiento

Cada suscripción de Microsoft Azure puede crear una o más cuentas de almacenamiento. (Una cuenta de almacenamiento proporciona un espacio de nombres único para trabajar con datos almacenados en la nube de Azure). El acceso a la cuenta de almacenamiento se controla mediante la suscripción y las claves de acceso asociadas a esa cuenta de almacenamiento.

Cuando crea una cuenta de almacenamiento, Microsoft Azure genera dos claves de acceso de almacenamiento de 512 bits, una de las cuales se usa para autenticación cuando el dispositivo de StorSimple tiene acceso a la cuenta de almacenamiento. Observe que solo se usa una de estas claves. La otra clave se mantiene en reserva, lo que le permite rotar periódicamente las claves. Para rotar las claves, active la clave secundaria y, a continuación, elimine la principal. Luego puede crear una clave nueva para usarla durante la siguiente rotación. (Por motivos de seguridad, muchos centros de datos requieren que las claves se roten).

Se recomienda seguir estos procedimientos recomendados para la rotación de claves:

* Debe rotar las claves de cuenta de almacenamiento de manera regular para garantizar que usuarios no autorizados no tengan acceso a la cuenta de almacenamiento.
* De forma regular, el administrador de Azure debe cambiar o volver a generar la clave principal o secundaria mediante la sección Storage de Azure Portal para acceder directamente a la cuenta de almacenamiento.

## <a name="protect-data-via-encryption"></a>Protección de datos mediante cifrado

StorSimple usa los siguientes algoritmos de cifrado para proteger datos almacenados o que se mueven entre los componentes de su solución de StorSimple.

| Algoritmo | Longitud de la clave | Protocolos/aplicaciones/comentarios |
| --- | --- | --- |
| RSA |2048 |Azure Portal usa el servicio RSA PKCS 1 v1.5 para cifrar datos de configuración que se envían al dispositivo, como las credenciales de la cuenta de almacenamiento, la configuración del dispositivo StorSimple y las claves de cifrado de almacenamiento en la nube. |
| AES |256 |AES con CBC se usa para cifrar la parte pública de la clave de cifrado de datos de servicio antes de enviarla a Azure Portal desde el dispositivo StorSimple. El dispositivo de StorSimple también lo usa para cifrar datos antes de que se envíen a la cuenta de almacenamiento en la nube. |

## <a name="storsimple-cloud-appliance-security"></a>Seguridad de StorSimple Cloud Appliance

[!INCLUDE [storsimple Cloud Appliance security](../../includes/storsimple-virtual-device-security.md)]

## <a name="managing-personal-information"></a>Administración de información personal

StorSimple Device Manager para las series física y virtual recopila información personal en las instancias principales siguientes:

- La configuración del usuario de alerta en la que se configuran las direcciones de correo electrónico de los usuarios. El administrador puede ver y borrar esta información. Esto se aplica a los dispositivos de la serie StorSimple 8000 y a StorSimple Virtual Array.
  * Para ver y borrar la configuración de la serie StorSimple 8000, siga los pasos que se indican en [Visualización y administración de alertas de StorSimple](storsimple-8000-manage-alerts.md#configure-alert-settings).
  * Para ver y borrar la configuración de StorSimple Virtual Array, siga los pasos que se indican en [Visualización y administración de alertas de StorSimple](storsimple-virtual-array-manage-alerts.md#configure-alert-settings).
- Los usuarios que pueden acceder a los datos que residen en los recursos compartidos. Se muestra y se puede ver una lista de los usuarios que pueden acceder a los datos del recurso compartido. Esta lista también se elimina al eliminar los recursos compartidos. Esto se aplica solo a StorSimple Virtual Array.
  * Para ver la lista de usuarios que pueden obtener acceso a un recurso compartido o eliminarlo, siga los pasos que se indican en [Administración de recursos compartidos en StorSimple Virtual Array](storsimple-virtual-array-manage-shares.md).

Para obtener más información, revise la directiva de privacidad de Microsoft en el [Centro de confianza](https://www.microsoft.com/trustcenter).

## <a name="frequently-asked-questions-faq"></a>Preguntas más frecuentes

Las siguientes son algunas preguntas y respuestas acerca de la seguridad y de Microsoft Azure StorSimple.

**P:** Mi servicio se ha puesto en peligro. ¿Cuáles deben ser mis siguientes pasos?

**R:** Debe cambiar inmediatamente la clave de cifrado de datos de servicio y las claves de la cuenta de almacenamiento que usa para la organización en niveles de los datos. Para obtener instrucciones, vaya a:

* [Cambiar la clave de cifrado de datos de servicio](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)
* [Rotación de claves de cuentas de almacenamiento](storsimple-8000-manage-storage-accounts.md#key-rotation-of-storage-accounts)

**P:** Tengo un dispositivo de StorSimple nuevo que pide la clave de registro de servicio. ¿Cómo la recupero?

**R:** Esta clave se creó cuando creó por primera vez el servicio StorSimple Device Manager. Cuando usa el servicio de StorSimple Device Manager para conectarse al dispositivo, puede usar la página de inicio rápido del servicio para ver o volver a generar la clave de registro del servicio. La generación de una nueva clave de registro del servicio no afectará a los dispositivos registrados existentes. Para obtener instrucciones, vaya a:

* [Ver o volver a generar la clave de registro de servicio](storsimple-8000-manage-service.md#regenerate-the-service-registration-key)

**P:** Perdí mi clave de cifrado de datos de servicio. ¿Qué puedo hacer?

**R:** Póngase en contacto con el soporte técnico de Microsoft. Puede iniciar sesión en una sesión de asistencia en el dispositivo y ayudarle a recuperar la clave (siempre que haya al menos un dispositivo en línea). Inmediatamente después de obtener la clave de cifrado de datos de servicio, deberá cambiarla para asegurarse de que solo usted conozca la nueva clave. Para obtener instrucciones, vaya a:

* [Cambiar la clave de cifrado de datos de servicio](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)

**P:** Autoricé a un dispositivo para que realizara un cambio de clave de cifrado de datos de servicio, pero no inicié el proceso de cambio de clave. ¿Cuál debo hacer?

**R:** Si el periodo de espera expiró, deberá volver a autorizar al dispositivo para que realice el cambio de clave de cifrado de datos de servicio y que así pueda volver a comenzar el proceso.

**P:** Cambié la clave de cifrado de datos de servicio, pero no me fue posible actualizar los demás dispositivos en un plazo de 4 horas. ¿Es necesario empezar de nuevo?

**R:** El período de tiempo de 4 horas solo es para iniciar el cambio. Una vez que inicie el proceso de actualización en el dispositivo de StorSimple autorizado, la autorización es válida hasta que se actualicen todos los dispositivos.

**P:** Nuestro administrador de StorSimple ya no trabaja en la empresa. ¿Cuál debo hacer?

**R:** Cambie y restablezca las contraseñas que permiten el acceso al dispositivo de StorSimple y cambie la clave de cifrado de datos de servicio para asegurarse de que el personal no autorizado no conozca la nueva información. Para obtener instrucciones, vaya a:

* [Use the StorSimple Device Manager service to change your storsimple passwords](storsimple-8000-change-passwords.md) (Uso del servicio StorSimple Device Manager para cambiar las contraseñas de StorSimple)
* [Cambiar la clave de cifrado de datos de servicio](storsimple-8000-manage-service.md#change-the-service-data-encryption-key)
* [Configurar CHAP para el dispositivo StorSimple](storsimple-8000-configure-chap.md)

**P:** Deseo proporcionar la contraseña de StorSimple Snapshot Manager a un host que se está conectando con un dispositivo StorSimple, pero la contraseña no se encuentra disponible. ¿Qué puedo hacer?

**R:** Si olvidó la contraseña, debería crear una nueva. A continuación, asegúrese de informar a todos los usuarios existentes que la contraseña cambió y que deberán actualizar sus clientes para que usen la contraseña nueva. Para obtener instrucciones, vaya a:

* [Cambio de la contraseña de StorSimple Snapshot Manager](storsimple-8000-change-passwords.md#set-the-storsimple-snapshot-manager-password)
* [Autenticar un dispositivo](storsimple-snapshot-manager-manage-devices.md#authenticate-a-device)

**P:** El certificado para acceso remoto a Windows PowerShell para StorSimple cambió en el dispositivo. ¿Cómo actualizo mis clientes de acceso remoto?

**R:** Puede descargar el nuevo certificado desde el servicio StorSimple Device Manager y, a continuación, proporcionarlo para instalarlo en el almacén de certificados de sus clientes de acceso remoto. Para obtener instrucciones, vaya a:

* [Cmdlet Import-Certificate](/powershell/module/pki/import-certificate)

**P:** ¿Mis datos están protegidos si el servicio StorSimple Device Manager se pone en peligro?

**P:** Los datos de configuración del servicio se cifran siempre con su clave pública cuando accede a ella desde un explorador web. Dado que el servicio no tiene acceso a la clave privada, el servicio no podrá ver ningún dato. Si el servicio StorSimple Device Manager se pone en peligro, no habrá ningún efecto, ya que no hay ninguna clave almacenada en dicho servicio.

**P:** Si alguna persona obtiene acceso al certificado de cifrado de datos, ¿se verán mis datos comprometidos?

**R:** Microsoft Azure almacena la clave de cifrado de datos del cliente (archivo .pfx) en un formato cifrado. Como el archivo .pfx está cifrado y el servicio StorSimple no cuenta con la clave de cifrado de datos de servicio para descifrar el archivo .pfx, el hecho de obtener acceso solo al archivo .pfx no expondrá ningún secreto.

**P:** ¿Qué ocurre si una entidad gubernamental pide mis datos a Microsoft?

**R:** Como todos los datos están cifrados en el servicio y la clave privada se mantiene en el dispositivo, la entidad gubernamental deberá pedir los datos al cliente.



## <a name="next-steps"></a>Pasos siguientes

[Implementación del dispositivo de StorSimple](storsimple-8000-deployment-walkthrough-u2.md)
