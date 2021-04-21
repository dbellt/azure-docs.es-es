---
title: 'Procedimientos de seguridad para fabricantes: Azure IoT Device Provisioning Service'
description: Información general sobre los procedimientos de seguridad comunes para los OEM y los fabricantes de dispositivos que preparan dispositivos para su inscripción en Azure IoT Device Provisioning Service (DPS).
author: timlt
ms.author: timlt
ms.date: 3/02/2020
ms.topic: conceptual
ms.service: iot-dps
ms.custom: iot-p0-scenario, iot-devices-deviceOEM
ms.reviewer: nberdy
ms.openlocfilehash: dad46e21b35225855d787c367d112362fc95a463
ms.sourcegitcommit: 6ed3928efe4734513bad388737dd6d27c4c602fd
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/07/2021
ms.locfileid: "107010548"
---
# <a name="security-practices-for-azure-iot-device-manufacturers"></a>Procedimientos de seguridad para los fabricantes de dispositivos Azure IoT
A medida que más fabricantes lanzan dispositivos IoT, resulta útil identificar una guía en torno a los procedimientos comunes. En este artículo se resumen los procedimientos de seguridad recomendados que se deben tener en cuenta al fabricar dispositivos para su uso con Azure IoT Device Provisioning Service (DPS).  

> [!div class="checklist"]
> * Selección de las opciones de autenticación de los dispositivos
> * Instalación de certificados en dispositivos IoT
> * Integración de un Módulo de plataforma segura (TPM) en el proceso de fabricación

## <a name="selecting-device-authentication-options"></a>Selección de las opciones de autenticación de los dispositivos
El objetivo final de cualquier medida de seguridad de dispositivos IoT es crear una solución IoT segura. Aunque problemas como las limitaciones de hardware, el costo y el nivel de conocimientos de seguridad afectan a las opciones que elija. Además, su enfoque en materia de seguridad afecta al modo en que los dispositivos IoT se conectan a la nube. Aunque hay [varios elementos de seguridad de IoT](https://www.microsoft.com/research/publication/seven-properties-highly-secure-devices/) que se deben tener en cuenta, un elemento clave con el que todos los clientes se encuentran es el tipo de autenticación que se va a usar. 

Los certificados X.509, los Módulos de plataforma segura (TPM) y las claves simétricas son tres tipos de autenticación ampliamente utilizados. Aunque existen otros tipos de autenticación, la mayoría de los clientes que crean soluciones IoT en Azure usan uno de estos tres tipos. En el resto de este artículo se analizan las ventajas y desventajas del uso de cada tipo de autenticación.

### <a name="x509-certificate"></a>Certificado X.509
Los certificados X.509 son un tipo de identidad digital que puede usar para la autenticación. El certificado X.509 estándar está documentado en [IETF RFC 5280](https://tools.ietf.org/html/rfc5280). En Azure IoT, hay dos maneras de autenticar los certificados:
- Huella digital. Se ejecuta un algoritmo de huella digital en un certificado para generar una cadena hexadecimal. La cadena generada es un identificador único del certificado. 
- Autenticación de CA basada en una cadena completa. Una cadena de certificados es una lista jerárquica de todos los certificados necesarios para autenticar un certificado de entidad final (EE). Para autenticar un certificado de entidad final, es necesario autenticar cada certificado de la cadena, incluida una CA raíz de confianza. 

Ventajas de X.509:
- X.509 es el tipo de autenticación más seguro admitido en Azure IoT.
- X.509 permite un alto nivel de control para la administración de certificados.
- Hay muchos proveedores disponibles para proporcionar soluciones de autenticación basadas en X.509.

Desventajas de X.509:
- Muchos clientes pueden necesitar confiar en proveedores externos por sus certificados.
- La administración de certificados puede ser costosa y se suma al costo total de la solución.
- La administración del ciclo de vida de los certificados puede ser difícil si la logística no está bien diseñada. 

### <a name="trusted-platform-module-tpm"></a>Módulo de plataforma segura (TPM)
TPM, también conocido como [ISO/IEC 11889](https://www.iso.org/standard/66510.html), es un estándar para generar y almacenar claves criptográficas de forma segura. TPM también hace referencia a un dispositivo de E/S virtual o físico que interactúa con los módulos que implementan el estándar. Un dispositivo TPM puede ser hardware discreto, hardware integrado, un módulo basado en firmware o un módulo basado en software. 

Hay dos diferencias principales entre los TPM y las claves simétricas: 
- Los chips TPM también pueden almacenar certificados X.509.
- La atestación de TPM en DPS usa la clave de aprobación (EK) de TPM, una forma de autenticación asimétrica. Con la autenticación asimétrica, se usa una clave pública para el cifrado y una clave privada independiente para el descifrado. En cambio, las claves simétricas utilizan la autenticación simétrica, en la que la clave privada se usa para el cifrado y el descifrado. 

Ventajas de TPM:
- Los TPM se incluyen como hardware estándar en muchos dispositivos Windows, con compatibilidad integrada para el sistema operativo. 
- La atestación de TPM es más fácil de proteger que la atestación de clave simétrica basada en tokens de firma de acceso compartido (SAS).
- Puede expirar y renovar fácilmente las credenciales del dispositivo. DPS revierte automáticamente las credenciales de IoT Hub cada vez que un dispositivo TPM tiene un reaprovisionamiento.

Desventajas de TPM: 
- Los TPM son complejos y pueden resultar difíciles de usar. 
- El desarrollo de aplicaciones con TPM es difícil a menos que disponga de un TPM físico o un emulador de calidad.
- Es posible que tenga que volver a diseñar la placa del dispositivo para incluir un TPM en el hardware. 
- Si revierte la EK de un TPM, se destruye la identidad del TPM y se crea una nueva. Aunque el chip físico permanece igual, tiene una nueva identidad en la solución IoT.

### <a name="symmetric-key"></a>Clave simétrica
Con las claves simétricas, se usa la misma clave para cifrar y descifrar los mensajes. Como resultado, el dispositivo y el servicio que lo autentica deben conocer la misma clave. Azure IoT admite conexiones de clave simétrica basadas en tokens de SAS. La autenticación de clave simétrica requiere una importante responsabilidad por parte del propietario para proteger las claves y lograr un nivel igual de seguridad que con la autenticación X.509. Si usa claves simétricas, el procedimiento recomendado es proteger las claves mediante un módulo de seguridad de hardware (HSM).

Ventajas de la clave simétrica:
- El uso de claves simétricas es la forma más sencilla y de menor costo para empezar a trabajar con la autenticación.
- El uso de claves simétricas simplifica el proceso, ya que no es necesario generar nada más. 

Desventajas de la clave simétrica: 
- Las claves simétricas suponen un importante grado de esfuerzo para proteger las claves. La misma clave se comparte entre el dispositivo y la nube, lo que significa que la clave debe estar protegida en dos lugares. Por el contrario, el desafío con TPM y los certificados X.509 demuestra la posesión de la clave pública sin revelar la clave privada.
- Las claves simétricas facilitan el seguimiento de procedimientos de seguridad deficientes. Una tendencia común con las claves simétricas es codificar de forma rígida las claves sin cifrar en los dispositivos. Aunque esta práctica es cómoda, deja las claves vulnerables. Puede mitigar algunos riesgos si almacena de forma segura la clave simétrica en el dispositivo. Sin embargo, si la prioridad es la seguridad en última instancia en lugar de la comodidad, use certificados X.509 o TPM para la autenticación. 

### <a name="shared-symmetric-key"></a>Clave simétrica compartida
Hay una variación de la autenticación de clave simétrica conocida como clave simétrica compartida. Este enfoque implica el uso de la misma clave simétrica en todos los dispositivos. La recomendación es evitar el uso de claves simétricas compartidas en los dispositivos. 

Ventajas de la clave simétrica compartida:
- Fácil de implementar y económico para producir a escala. 

Desventajas de la clave simétrica compartida: 
- Muy vulnerable a ataques. La ventaja de una fácil implementación es superada por el riesgo. 
- Cualquiera puede suplantar a los dispositivos si obtiene la clave compartida.
- Si confía en una clave simétrica compartida que se ve comprometida, es probable que pierda el control de los dispositivos. 

### <a name="making-the-right-choice-for-your-devices"></a>Elección de la opción adecuada para los dispositivos
Para elegir un método de autenticación, asegúrese de tener en cuenta las ventajas y los costos de cada enfoque para su proceso de fabricación específico.  En el caso de la autenticación de dispositivos, normalmente hay una relación inversa entre la seguridad de un enfoque determinado y su comodidad.  

## <a name="installing-certificates-on-iot-devices"></a>Instalación de certificados en dispositivos IoT
Si usa certificados X.509 para autenticar los dispositivos IoT, en esta sección se ofrecen instrucciones sobre cómo integrar los certificados en el proceso de fabricación. Deberá tomar varias decisiones.  Entre ellas se incluyen las decisiones sobre las variables de certificado comunes, cuándo generar certificados y cuándo instalarlos. 

Si está acostumbrado a usar contraseñas, es posible que se pregunte por qué no se puede usar el mismo certificado en todos los dispositivos, de la misma manera que se podría usar la misma contraseña en todos los dispositivos. En primer lugar, el uso de la misma contraseña en todos los sitios es peligroso. Ese procedimiento ha expuesto a las empresas a ataques de DDoS importantes, incluido el que desactivo los servicios de DNS en la costa este de EE. UU. hace varios años. Nunca use la misma contraseña en todas partes, incluso con cuentas personales. En segundo lugar, un certificado no es una contraseña, es una identidad única. Una contraseña es como un código secreto que cualquiera puede usar para abrir una puerta de un edificio protegido.  Es algo que conoce y puede dar la contraseña a cualquiera para obtener acceso.  Un certificado es como una licencia de conducción, con su foto y otros detalles, que puede mostrar a un guarda para entrar en un edificio protegido. Está ligado a quién es.  Siempre que el guarda compruebe con precisión la licencia de conducción de cada persona, solo usted puede usar su licencia (identidad) para obtener acceso. 

### <a name="variables-involved-in-certificate-decisions"></a>Variables implicadas en las decisiones sobre certificados
Tenga en cuenta las siguientes variables y cómo afecta cada una en los procesos de fabricación generales. 

#### <a name="where-the-certificate-root-of-trust-comes-from"></a>Procedencia de la raíz de confianza del certificado
Puede resultar costoso y complejo administrar una infraestructura de clave pública (PKI).  Especialmente si su empresa no tiene experiencia en la administración de una PKI. Tendrá las siguientes opciones:
- Usar una PKI de terceros. Puede adquirir certificados de firma intermedios de un proveedor de certificados de terceros. También puede usar una entidad de certificación (CA) privada. 
- Usar una PKI autoadministrada. Puede mantener su propio sistema de PKI y generar sus propios certificados.
- Usar el servicio de seguridad de [Azure Sphere](https://azure.microsoft.com/services/azure-sphere/). Esta opción solo se aplica a dispositivos Azure Sphere. 

#### <a name="where-certificates-are-stored"></a>Almacenamiento de los certificados
Hay algunos factores que afectan a la decisión de dónde se almacenan los certificados. Estos factores incluyen el tipo de dispositivo, los márgenes de beneficios esperados (si puede permitirse un almacenamiento seguro), las funcionalidades del dispositivo y la tecnología de seguridad existente en el dispositivo que puede usar. Considere las opciones siguientes:
- En un módulo de seguridad de hardware (HSM). Se recomienda usar un HSM. Compruebe si la placa de control del dispositivo ya tiene un HSM instalado. Si sabe que no tiene un HSM, colabore con el fabricante de hardware para identificar un HSM que satisfaga sus necesidades.
- En un lugar seguro en disco, como un entorno de ejecución de confianza (TEE).
- En el sistema de archivos local o en un almacén de certificados. Por ejemplo, el almacén de certificados de Windows. 

#### <a name="connectivity-at-the-factory"></a>Conectividad en la fábrica
La conectividad en la fábrica determina cómo y cuándo se van a obtener los certificados que se van a instalar en los dispositivos. Las opciones de conectividad son las siguientes:
- Conectividad. Disponer de conectividad es la opción óptima, optimiza el proceso de generación local de certificados. 
- Sin conectividad. En este caso, se usa un certificado firmado de una entidad de certificación para generar certificados de dispositivo localmente y sin conexión. 
- Sin conectividad. En este caso, puede obtener certificados que se generaron con anterioridad. También puede usar una PKI sin conexión para generar certificados de forma local.

#### <a name="audit-requirement"></a>Requisitos de auditoría
Según el tipo de dispositivos que fabrique, es posible que tenga un requisito de regulación para crear una traza de auditoría sobre cómo se instalan las identidades de dispositivo en los dispositivos. La auditoría supone un costo de producción significativo. Por lo tanto, en la mayoría de los casos, hágala únicamente si es necesario. Si no está seguro de si se requiere una auditoría, consulte al departamento legal de la empresa. Las opciones de auditoría son: 
- No es un sector confidencial. No se requiere auditoría.
- Sector confidencial. Los certificados se deben instalar en una sala segura según los requisitos de certificación de cumplimiento. Si necesita una sala segura para instalar los certificados, es probable que ya sepa cómo se instalan los certificados en los dispositivos. Y es probable que ya tenga un sistema de auditoría implementado. 

#### <a name="length-of-certificate-validity"></a>Longitud de la validez del certificado
Al igual que la licencia de conducción, los certificados tienen una fecha de expiración que se establece cuando se crean. Estas son las opciones de longitud de la validez del certificado:
- No se requiere renovación.  Este enfoque utiliza un período de renovación largo, por lo que nunca tendrá que renovar el certificado durante la vigencia del dispositivo. Aunque este enfoque es práctico, también es arriesgado.  Puede reducir el riesgo mediante el uso de un almacenamiento seguro, como un HSM en los dispositivos. Sin embargo, el procedimiento recomendado es evitar el uso de certificados de larga duración.
- Se requiere renovación.  Deberá renovar el certificado durante la vigencia del dispositivo. La longitud de la validez del certificado depende del contexto, por lo que necesitará una estrategia para la renovación.  La estrategia debe incluir el lugar en el que se obtienen los certificados y el tipo de funcionalidad inalámbrica que los dispositivos tienen que usar en el proceso de renovación. 

### <a name="when-to-generate-certificates"></a>Cuándo generar los certificados
Las funcionalidades de conectividad a Internet en la fábrica afectarán al proceso de generación de certificados. Tiene varias opciones en relación a cuándo generar los certificados: 

- Certificados cargados previamente.  Algunos proveedores de HSM ofrecen un servicio Premium en el que el proveedor de HSM instala los certificados para el cliente. En primer lugar, los clientes proporcionan al proveedor de HSM acceso a un certificado de firma. A continuación, el proveedor de HSM instala los certificados firmados por ese certificado de firma en cada HSM que compra el cliente. Todo lo que tiene que hacer el cliente es instalar el HSM en el dispositivo. Aunque este servicio agrega costos, ayuda a simplificar el proceso de fabricación.  Y resuelve la cuestión de cuándo instalar los certificados.
- Certificados generados por el dispositivo.  Si los dispositivos generan certificados internamente, debe extraer el certificado X.509 público del dispositivo para inscribirlo en DPS. 
- Fábrica conectada.  Si la fábrica tiene conectividad, puede generar certificados de dispositivo siempre que los necesite.
- Fábrica sin conexión con su propia PKI. Si la fábrica no tiene conectividad y usa su propia PKI con compatibilidad sin conexión, puede generar los certificados cuando los necesite.
- Fábrica sin conexión con PKI de terceros. Si la fábrica no tiene conectividad y usa una PKI de terceros, debe generar los certificados de antemano. Además, será necesario generar los certificados desde una ubicación que tenga conectividad. 

### <a name="when-to-install-certificates"></a>Cuándo instalar los certificados
Después de generar certificados para los dispositivos IoT, puede instalarlos en los dispositivos. 

Si usa certificados cargados previamente con un HSM, el proceso se simplifica. Después de instalar el HSM en el dispositivo, el código del dispositivo puede acceder a él. Después, llamará a las API del HSM para acceder al certificado almacenado en el HSM. Este enfoque es el más conveniente para el proceso de fabricación. 

Si no usa un certificado cargado previamente, debe instalar el certificado como parte del proceso de producción. El enfoque más sencillo consiste en instalar el certificado en el HSM al mismo tiempo que se escribe la imagen de firmware inicial. El proceso debe agregar un paso para instalar la imagen en cada dispositivo. Después de este paso, puede ejecutar las comprobaciones de calidad finales y cualquier otro paso antes de empaquetar y enviar el dispositivo. 

Hay herramientas de software disponibles que permiten ejecutar el proceso de instalación y la comprobación de calidad final en un solo paso. Puede modificar estas herramientas para generar un certificado o para extraer un certificado de un almacén de certificados generado previamente. Después, el software puede instalar el certificado donde sea necesario. Las herramientas de software de este tipo permiten ejecutar una fabricación con calidad de producción a escala. 

Una vez instalados los certificados en los dispositivos, el siguiente paso es aprender a inscribir los dispositivos en [DPS](about-iot-dps.md). 

## <a name="integrating-a-tpm-into-the-manufacturing-process"></a>Integración de un TPM en el proceso de fabricación
Si usa un TPM para autenticar los dispositivos IoT, en esta sección se ofrecen instrucciones. En la guía se describen los ampliamente utilizados dispositivos TPM 2.0, que admiten la clave de código de autenticación de mensajes basado en hash (HMAC). La especificación de TPM para los chips TPM es un estándar ISO que mantiene Trusted Computing Group. Para más información sobre TPM, consulte las especificaciones de [TPM 2.0](https://trustedcomputinggroup.org/tpm-library-specification/) e [ISO/IEC 11889](https://www.iso.org/standard/66510.html). 

### <a name="taking-ownership-of-the-tpm"></a>Toma de propiedad de un TPM
Un paso crítico en la fabricación de un dispositivo con un chip TPM es la toma de propiedad del TPM. Este paso es necesario para que pueda proporcionar una clave al propietario del dispositivo. El primer paso consiste en extraer la clave de aprobación (EK) del dispositivo. El paso siguiente es reclamar realmente la propiedad. La forma de hacerlo depende del TPM y del sistema operativo que use. Si es necesario, póngase en contacto con el fabricante del TPM o con el desarrollador del sistema operativo del dispositivo para determinar cómo reclamar la propiedad. 

En el proceso de fabricación, puede extraer la EK y reclamar la propiedad en momentos diferentes, lo que aporta flexibilidad. Muchos fabricantes aprovechan esta flexibilidad y agregan un módulo de seguridad de hardware (HSM) para mejorar la seguridad de sus dispositivos. En esta sección se proporcionan instrucciones sobre cuándo extraer la EK, cuándo reclamar la propiedad del TPM y consideraciones para la integración de estos pasos en una escala de tiempo de fabricación. 

> [!IMPORTANT]
> En esta guía se supone que usa un TPM discreto, de firmware o integrado. En los lugares donde es aplicable, la guía agrega notas sobre el uso de un TPM no discreto o de software. Si usa un TPM de software, puede haber pasos adicionales que no se incluyen en esta guía. Los TPM de software tienen una variedad de implementaciones que se encuentran fuera del ámbito de este artículo.  En general, es posible integrar un TPM de software en la siguiente escala de tiempo de fabricación general. Sin embargo, aunque un TPM emulado por software es adecuado para prototipos y pruebas, no proporciona el mismo nivel de seguridad que un TPM discreto, de firmware o integrado. Como procedimiento general, evite el uso de un TPM de software en producción.

### <a name="general-manufacturing-timeline"></a>Escala de tiempo de fabricación general
La escala de tiempo siguiente muestra cómo un TPM pasa por un proceso de producción y termina en un dispositivo. Cada proceso de fabricación es único y esta escala de tiempo muestra los patrones más comunes. La escala de tiempo ofrece instrucciones sobre cuándo realizar ciertas acciones con las claves. 

#### <a name="step-1-tpm-is-manufactured"></a>Paso 1: El TPM está fabricado
- Si compra un TPM de un fabricante para su uso en sus dispositivos, consulte si se van a extraer claves de aprobación públicas (EK_pubs). Resulta útil si el fabricante proporciona la lista de EK_pubs con los dispositivos enviados. 
    > [!NOTE]
    > Puede conceder al fabricante del TPM acceso de escritura a la lista de inscripción mediante directivas de acceso compartido en el servicio de aprovisionamiento.  Este enfoque le permite agregar los TPM a su lista de inscripción en su nombre.  Pero eso es una etapa temprana en el proceso de fabricación y requiere confianza en el fabricante del TPM. Hágalo bajo su responsabilidad. 

- Si fabrica TPM para vender a los fabricantes de dispositivos, considere la posibilidad de ofrecer a los clientes una lista de EK_pubs junto con los TPM físicos.  Al proporcionar a los clientes las EK_pubs, se ahorra un paso en el proceso. 
- Si fabrica TPM para usarlos con sus propios dispositivos, identifique qué punto del proceso es el más conveniente para extraer las EK_pub. Puede extraer la EK_pub en cualquiera de los puntos restantes de la escala de tiempo. 

#### <a name="step-2-tpm-is-installed-into-a-device"></a>Paso 2: El TPM está instalado en un dispositivo
En este punto del proceso de producción, debe saber qué instancia de DPS se usará con el dispositivo. Como resultado, podrá agregar dispositivos a la lista de inscripción para el aprovisionamiento automatizado. Para más información sobre el aprovisionamiento automático de dispositivos, consulte la [documentación de DPS](about-iot-dps.md).
- Si no ha extraído la EK_pub, ahora es un buen momento para hacerlo. 
- En función del proceso de instalación del TPM, este paso también es un buen momento para la toma de propiedad del TPM. 

#### <a name="step-3-device-has-firmware-and-software-installed"></a>Paso 3: El dispositivo tiene instalado el firmware y el software
En este punto del proceso, instale el cliente de DPS junto con el ámbito del identificador y la dirección URL global para el aprovisionamiento.
- Ahora es la última oportunidad para extraer la EK_pub. Si un tercero va a instalar el software en el dispositivo, se recomienda extraer primero la EK_pub.
- Este punto del proceso de fabricación es idóneo para la toma de propiedad del TPM.  
    > [!NOTE]
    > Si utiliza un TPM de software, puede instalarlo ahora.  Extraiga la EK_pub al mismo tiempo.

#### <a name="step-4-device-is-packaged-and-sent-to-the-warehouse"></a>Paso 4: El dispositivo se empaqueta y se envía al almacén
A veces, un dispositivo puede estar en un almacén hasta un año antes de su implementación y aprovisionamiento con DPS.  Si un dispositivo pasa en un almacén mucho tiempo antes de la implementación, los clientes que implementen el dispositivo quizá necesiten actualizar el firmware, el software o las credenciales expiradas.

#### <a name="step-5-device-is-installed-into-the-location"></a>Paso 5: El dispositivo está instalado en la ubicación
Una vez que el dispositivo llega a su ubicación final, pasa por el aprovisionamiento automatizado con DPS.

Para más información, vea [Aprovisionamiento](about-iot-dps.md#provisioning-process) y [Atestación de TPM](concepts-tpm-attestation.md). 

## <a name="resources"></a>Recursos

Además de los procedimientos de seguridad recomendados en este artículo, Azure IoT proporciona recursos para ayudarle a seleccionar hardware seguro y crear implementaciones de IoT seguras: 
- [Recomendaciones de seguridad](../iot-fundamentals/security-recommendations.md) de Azure IoT para guiar el proceso de implementación. 
- [Azure Security Center](https://azure.microsoft.com/services/security-center/) ofrece un servicio que ayuda a crear implementaciones de IoT seguras. 
- Para obtener ayuda con la evaluación del entorno de hardware, consulte el documento de notas del producto [Evaluación de la seguridad de IoT](https://download.microsoft.com/download/D/3/9/D3948E3C-D5DC-474E-B22F-81BA8ED7A446/Evaluating_Your_IOT_Security_whitepaper_EN_US.pdf). 
- Para obtener ayuda con la selección de hardware seguro, consulte [Hardware seguro adecuado para su implementación de IoT](https://download.microsoft.com/download/C/0/5/C05276D6-E602-4BB1-98A4-C29C88E57566/The_right_secure_hardware_for_your_IoT_deployment_EN_US.pdf). 