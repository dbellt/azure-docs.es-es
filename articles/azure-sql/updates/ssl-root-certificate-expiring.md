---
title: Rotación de certificados en Azure SQL Database y SQL Managed Instance
description: Conozca los cambios futuros de los certificados raíz que afectarán Azure SQL Database y Azure SQL Managed Instance.
author: srdan-bozovic-msft
ms.author: srbozovi
ms.service: sql-db-mi
ms.subservice: security
ms.topic: conceptual
ms.date: 09/13/2020
ms.openlocfilehash: d569065a022180f2b2f94131099e6eedc4c468fb
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110702086"
---
# <a name="understanding-the-changes-in-the-root-ca-change-for-azure-sql-database--sql-managed-instance"></a>Descripción de los cambios en la entidad de certificación raíz en Azure SQL Database y SQL Managed Instance

Azure SQL Database y SQL Managed Instance van a cambiar el certificado raíz de la aplicación o el controlador de cliente habilitados con SSL, que se usa para establecer una conexión TDS segura. Como parte de los procedimientos recomendados de seguridad y mantenimiento estándar, el [certificado raíz existente](https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem) está configurado para expirar el 26 de octubre de 2020. En este artículo se proporcionan más detalles sobre los próximos cambios, los recursos que se verán afectados y los pasos necesarios para asegurarse de que la aplicación mantenga la conectividad con el servidor de base de datos.

## <a name="what-update-is-going-to-happen"></a>¿Qué actualización va a producirse?

Recientemente se publicaron en [Browser Forum sobre la entidad de certificación (CA)](https://cabforum.org/) informes de varios certificados emitidos por proveedores de CA que no son compatibles.

Según los requisitos de cumplimiento del sector, los proveedores de CA comenzaron a revocar los certificados de CA para las CA no compatibles, lo que requiere que los servidores usen certificados emitidos por CA compatibles y firmados por certificados de CA de esas CA compatibles. Dado que actualmente Azure SQL Database y SQL Managed Instance usan uno de estos certificados no compatibles, que las aplicaciones cliente emplean para validar sus conexiones SSL, es necesario asegurarse de que se realicen las acciones adecuadas (descritas a continuación) para minimizar los posibles efectos en los servidores de Azure SQL.

El nuevo certificado se utilizará a partir del 26 de octubre de 2020. Si usa la validación completa del certificado de servidor al conectarse desde un cliente SQL (TrustServerCertificate=true), debe asegurarse de que este cliente pueda validar el nuevo certificado raíz antes del 26 de octubre de 2020.

## <a name="how-do-i-know-if-my-application-might-be-affected"></a>¿Cómo sé si este cambio afecta a mi aplicación?

Todas las aplicaciones que usan SSL/TLS y verifican el certificado raíz deben actualizarlo para conectarse a Azure SQL Database y SQL Managed Instance. 

Si actualmente no usa SSL/TLS, este cambio no afecta a la disponibilidad de aplicaciones. Puede confirmar si la aplicación cliente intenta verificar el certificado raíz examinando la cadena de conexión. Si TrustServerCertificate está establecido explícitamente en true, no se verá afectado por el cambio.

Si el controlador de cliente utiliza el almacén de certificados del sistema operativo, como es el caso de la mayoría de los controladores, y el sistema operativo se mantiene regularmente, es probable que este cambio no le afecte, ya que el certificado raíz al que se va a cambiar debe estar disponible en el almacén de certificados raíz de confianza. Busque Baltimore CyberDigiCert GlobalRoot G2 y valide que exista.

Si el controlador de cliente utiliza el almacén de certificados de archivo local, para evitar que se interrumpa la disponibilidad de la aplicación debido a la revocación inesperada de certificados, o para actualizar un certificado que se ha revocado, consulte la sección [ **¿Qué tengo que hacer para mantener la conectividad?**](./ssl-root-certificate-expiring.md#what-do-i-need-to-do-to-maintain-connectivity)

## <a name="what-do-i-need-to-do-to-maintain-connectivity"></a>¿Qué tengo que hacer para mantener la conectividad?

Para evitar que se interrumpa la disponibilidad de la aplicación debido a la revocación inesperada de certificados, o para actualizar un certificado que se ha revocado, siga estos pasos:

*   Descargue la CA raíz de Baltimore CyberTrust Root & DigiCert GlobalRoot G2 de los siguientes vínculos:
    *   https://www.digicert.com/CACerts/BaltimoreCyberTrustRoot.crt.pem
    *   https://cacerts.digicert.com/DigiCertGlobalRootG2.crt.pem

*   Genere un almacén de certificados de CA combinado que incluya los certificados **BaltimoreCyberTrustRoot** y **DigiCertGlobalRootG2**.

## <a name="what-can-be-the-impact"></a>¿Cuál puede ser el efecto?
Si va a validar los certificados de servidor como se documenta aquí, la disponibilidad de la aplicación puede verse interrumpida, ya que no se podrá acceder a la base de datos. Según la aplicación, puede recibir distintos mensajes de error, entre los que se incluyen los siguientes:
*   Certificado no válido o certificado revocado
*   Se agotó el tiempo de espera de la conexión
*   Error, si corresponde

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

### <a name="if-i-am-not-using-ssltls-do-i-still-need-to-update-the-root-ca"></a>Si no utilizo SSL/TLS, ¿aún tengo que actualizar la CA raíz?
Si no usa SSL/TLS, no es necesario hacer nada con respecto a este cambio. Sin embargo, debe establecer un plan para empezar a usar la versión más reciente de TLS mientras planeamos la aplicación de TLS en un futuro próximo.

### <a name="what-will-happen-if-i-do-not-update-the-root-certificate-before-october-26-2020"></a>¿Qué sucederá si no actualizo el certificado raíz antes del 26 de octubre de 2020?
Si no actualiza el certificado raíz antes del 30 de noviembre de 2020, las aplicaciones que se conecten a través de SSL/TLS y verifiquen dicho certificado no podrán comunicarse con Azure SQL Database y SQL Managed Instance, y la aplicación experimentará problemas de conectividad con estos servicios.

### <a name="do-i-need-to-plan-a-maintenance-downtime-for-this-changebr"></a>¿Es necesario planear un tiempo de inactividad de mantenimiento para este cambio?<BR>
No. Dado que el cambio aquí solo es en el lado cliente para la conexión al servidor, no hay necesidad de ningún tiempo de inactividad de mantenimiento.

### <a name="what-if-i-cannot-get-a-scheduled-downtime-for-this-change-before-october-26-2020"></a>¿Qué ocurre si no puedo obtener un tiempo de inactividad programado para este cambio antes del 26 de octubre de 2020?
Dado que los clientes usados para conectarse al servidor deben actualizar la información del certificado tal y como se describe en la sección de corrección [aquí](./ssl-root-certificate-expiring.md#what-do-i-need-to-do-to-maintain-connectivity), no es necesario un tiempo de inactividad para el servidor en este caso.

### <a name="if-i-create-a-new-server-after-november-30-2020-will-i-be-impacted"></a>Si creo un servidor después del 30 de noviembre de 2020, ¿me afectará este cambio?
En los servidores creados después del 26 de octubre de 2020, puede usar el certificado recién emitido para que las aplicaciones se conecten mediante SSL.

### <a name="how-often-does-microsoft-update-their-certificates-or-what-is-the-expiry-policy"></a>¿Con qué frecuencia actualiza Microsoft sus certificados o cuál es la directiva de expiración?
Estos certificados utilizados por Azure SQL Database y Azure SQL Managed Instance provienen de entidades de certificación (CA) de confianza. Por lo tanto, la compatibilidad de estos certificados en Azure SQL Database y SQL Managed Instance está ligada a la compatibilidad de estos certificados por parte de la CA. Sin embargo, como en este caso, puede haber errores imprevistos en estos certificados predefinidos, que deben corregirse lo antes posible.

### <a name="if-i-am-using-read-replicas-do-i-need-to-perform-this-update-only-on-primary-server-or-all-the-read-replicas"></a>Si utilizo réplicas de lectura, ¿tengo que realizar esta actualización solo en el servidor principal o en todas las réplicas de lectura?
Puesto que esta actualización es un cambio en el lado cliente, si el cliente leía los datos del servidor de réplicas, será necesario aplicar también los cambios para esos clientes. 

### <a name="do-we-have-server-side-query-to-verify-if-ssl-is-being-used"></a>¿Tenemos que consultar en el lado servidor para comprobar si se está utilizando SSL?
Dado que esta configuración es en el lado cliente, la información no está disponible en el lado del servidor.

### <a name="what-if-i-have-further-questions"></a>¿Qué hago si tengo más preguntas?
Si tiene un plan de soporte técnico y necesita ayuda técnica, cree una solicitud de soporte técnico de Azure. Para ello, consulte [Creación de una solicitud de soporte técnico de Azure](../../azure-portal/supportability/how-to-create-azure-support-request.md).