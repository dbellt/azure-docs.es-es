---
title: Integrar Azure Key Vault con Azure Policy
description: Aprenda a integrar Azure Key Vault con Azure Policy.
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/31/2021
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.openlocfilehash: cddc7b931bf59412d4a7ec8e6b0eecfe148f3d5e
ms.sourcegitcommit: 6686a3d8d8b7c8a582d6c40b60232a33798067be
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107749283"
---
# <a name="integrate-azure-key-vault-with-azure-policy"></a>Integrar Azure Key Vault con Azure Policy

[Azure Policy](../../governance/policy/index.yml) es una herramienta de gobierno que ofrece a los usuarios la capacidad de auditar y administrar su entorno de Azure a escala. Azure Policy proporciona la capacidad de colocar barreras en los recursos de Azure para asegurarse de que son compatibles con las reglas de directivas asignadas. Permite a los usuarios realizar tareas de auditoría, aplicación en tiempo real y corrección de su entorno de Azure. Los resultados de las auditorías realizadas por la directiva estarán disponibles para los usuarios en un panel de cumplimiento, donde podrán ver una exploración en profundidad de qué recursos y componentes son compatibles y cuáles no.  Para más información, consulte la [Información general del servicio Azure Policy](../../governance/policy/overview.md).

Escenarios de uso de ejemplo:

- Quiere mejorar la postura de seguridad de su empresa mediante la implementación de requisitos en torno a los tamaños de clave mínimos y los períodos de validez máximos de los certificados de los almacenes de claves de su empresa, pero no sabe qué equipos serán compatibles y cuáles no.
- Actualmente no tiene una solución para realizar una auditoría en toda la organización o lleva a cabo auditorías manuales de su entorno pidiendo a los equipos individuales de su organización que informen de su cumplimiento. Busca una manera de automatizar esta tarea, realizar auditorías en tiempo real y garantizar la precisión de la auditoría.
- Quiere aplicar las directivas de seguridad de la empresa y evitar que los usuarios creen certificados autofirmados, pero no dispone de una forma automatizada de bloquear su creación. 
- Quiere relajar algunos requisitos para los equipos de pruebas, pero desea mantener controles estrechos en el entorno de producción. Necesita una manera automatizada y sencilla de separar la aplicación de los recursos.
- Quiere asegurarse de que puede revertir la aplicación de nuevas directivas en caso de que se produzca un problema en el sitio activo. Necesita una solución de un solo clic para desactivar la aplicación de la directiva. 
- Se basa en una solución de terceros para la auditoría de su entorno y desea usar una oferta interna de Microsoft.

## <a name="types-of-policy-effects-and-guidance"></a>Tipos de instrucciones y efectos de directivas

**Auditoría**: Cuando el efecto de una directiva se establece en auditoría, la directiva no producirá ningún cambio importante en el entorno. Solo le avisará de los componentes, como los certificados que no cumplan las definiciones de directiva dentro de un ámbito especificado, marcando estos componentes como no compatibles en el panel de cumplimiento de directivas. La auditoría es predeterminada si no se selecciona ningún efecto de directiva.

**Denegar**: Cuando el efecto de una directiva se establece en denegar, la directiva bloqueará la creación de nuevos componentes, como los certificados, así como el bloqueo de nuevas versiones de componentes existentes que no cumplen con la definición de la directiva. Los recursos existentes no compatibles dentro de un almacén de claves no se ven afectados. Las capacidades de ' auditoría ' seguirán funcionando.

## <a name="available-built-in-policy-definitions"></a>Definiciones de Directiva "Integradas" disponibles

Key Vault ha creado un conjunto de directivas que se pueden usar para administrar los objetos de clave, certificado y secreto. Estas directivas están "Integradas", lo que significa que no requieren que se escriba ningún JSON personalizado para habilitarlas y están disponibles en el Azure Portal que se va a asignar. Todavía puede personalizar determinados parámetros para ajustarse a las necesidades de su organización.

# <a name="certificate-policies"></a>[Directivas de certificado](#tab/certificates)

### <a name="certificates-should-have-the-specified-maximum-validity-period-preview"></a>Los certificados deben tener el período de validez máximo especificado (versión preliminar)

Esta directiva permite administrar el período de validez máximo de los certificados almacenados en el almacén de claves. Es una buena práctica de seguridad limitar el período de validez máximo de los certificados. Si una clave privada del certificado va a estar en peligro sin detección, el uso de certificados de corta duración reduce el período de tiempo para los daños en curso y reduce el valor del certificado a un atacante.

### <a name="certificates-should-use-allowed-key-types-preview"></a>Los certificados deben usar tipos de clave permitidos (versión preliminar)

Esta directiva le permite restringir el tipo de certificados que pueden encontrarse en el almacén de claves. Puede usar esta directiva para asegurarse de que las claves privadas del certificado son RSA, ECC o están respaldadas por HSM. Puede elegir en la siguiente lista los tipos de certificados que se permiten.

- RSA
- RSA - HSM
- ECC
- ECC - HSM

### <a name="certificates-should-have-the-specified-lifetime-action-triggers-preview"></a>Los certificados deben tener los desencadenadores de acciones de vigencia especificados (versión preliminar)

Esta directiva permite administrar la acción de duración especificada para los certificados que están dentro de un número determinado de días de expiración o que han alcanzado un porcentaje determinado de su vida útil.

### <a name="certificates-should-be-issued-by-the-specified-integrated-certificate-authority-preview"></a>La entidad de certificación integrada especificada debe emitir los certificados (versión preliminar)

Si usa una Key Vault entidad de certificación integrada (DigiCert o GlobalSign) y quiere que los usuarios usen uno de estos proveedores o cualquiera de ellos, puede usar esta directiva para auditar o aplicar la selección. Esta directiva también se puede utilizar para auditar o denegar la creación de certificados autofirmados en el almacén de claves.

### <a name="certificates-should-be-issued-by-the-specified-non-integrated-certificate-authority-preview"></a>La entidad de certificación integrada especificada debe emitir los certificados (versión preliminar)

Si usa una entidad de certificación interna o una entidad de certificación que no está integrada con el almacén de claves y quiere que los usuarios usen una entidad de certificación de la lista que proporcione, puede usar esta directiva para crear una lista de entidades de certificación permitidas por nombre de emisor. Esta directiva también se puede utilizar para auditar o denegar la creación de certificados autofirmados en el almacén de claves.

### <a name="certificates-using-elliptic-curve-cryptography-should-have-allowed-curve-names-preview"></a>Los certificados que usan criptografía de curva elíptica deben tener nombres de curva permitidos (versión preliminar)

Si usa criptografía de curva elíptica o certificados ECC, puede personalizar una lista de nombres de curva permitidos en la siguiente lista. La opción predeterminada permite los siguientes nombres de curva.

- P-256
- P-256K
- P-384
- P-521

## <a name="certificates-using-rsa-cryptography-manage-minimum-key-size-for-rsa-certificates-preview"></a>Los certificados que usan la criptografía RSA deben tener el tamaño de clave mínimo especificado (versión preliminar)

Si usa certificados RSA, puede elegir un tamaño de clave mínimo que los certificados deben tener. Puede seleccionar una opción de la lista siguiente.

- bit 2048
- bit 3072
- bit 4096

## <a name="manage-certificates-that-are-within-a-specified-number-of-days-of-expiration-preview"></a>Administrar certificados que estén dentro de un número especificado de días de expiración (versión preliminar)

El servicio puede experimentar una interrupción si un certificado que no se está supervisando adecuadamente no se gira antes de la expiración. Esta directiva es fundamental para asegurarse de que los certificados almacenados en el almacén de claves se están supervisando. Se recomienda aplicar esta directiva varias veces con umbrales de expiración diferentes, por ejemplo, en los umbrales 180, 90, 60 y 30 días. Esta directiva se puede usar para supervisar y evaluar la expiración de certificados en la organización.

# <a name="key-policies"></a>[Directivas de claves](#tab/keys)

### <a name="keys-should-not-be-active-for-longer-than-the-specified-number-of-days-preview"></a>Las claves no deben estar activas durante más tiempo que el número especificado de días (versión preliminar)

Si desea asegurarse de que las claves no estén activas durante más tiempo que un número especificado de días, puede usar esta directiva para auditar cuánto tiempo ha estado activa la clave.

**Si la clave tiene establecida una fecha de activación**, esta directiva calculará el número de días transcurridos desde la **fecha de activación** de la clave hasta la fecha actual. Si el número de días supera el umbral establecido, la clave se marcará como no compatible con la directiva.

**Si la clave no tiene establecida una fecha de activación**, esta directiva calculará el número de días transcurridos desde la **fecha de creación** de la clave hasta la fecha actual. Si el número de días supera el umbral establecido, la clave se marcará como no compatible con la directiva.

### <a name="keys-should-be-the-specified-cryptographic-type-rsa-or-ec-preview"></a>Las claves deben ser del tipo criptográfico especificado, RSA o EC (versión preliminar)

Esta directiva le permite restringir el tipo de claves que pueden encontrarse en el almacén de claves. Puede usar esta directiva para asegurarse de que las claves son RSA, ECC o están respaldadas por HSM. Puede elegir en la siguiente lista los tipos de certificados que se permiten.

- RSA
- RSA - HSM
- ECC
- ECC - HSM

### <a name="keys-using-elliptic-curve-cryptography-should-have-the-specified-curve-names-preview"></a>Las claves que usan criptografía de curva elíptica deben tener especificados nombres de curva (versión preliminar)

Si usa criptografía de curva elíptica o claves ECC, puede personalizar una lista de nombres de curva permitidos en la siguiente lista. La opción predeterminada permite los siguientes nombres de curva.

- P-256
- P-256K
- P-384
- P-521

### <a name="keys-should-have-expirations-dates-set-preview"></a>Las claves deben tener establecida la fecha de expiración (versión preliminar)

Esta directiva audita todas las claves de los almacenes de claves y marca las claves que no tienen establecida una fecha de expiración como no compatibles. También puede usar esta directiva para bloquear la creación de claves que no tienen establecida una fecha de expiración.

### <a name="keys-should-have-more-than-the-specified-number-of-days-before-expiration-preview"></a>Deben faltar más días que el número especificado para la expiración de las claves (versión preliminar)

Si una clave está demasiado cerca de la expiración, un retraso de la organización para rotar la clave puede producir una interrupción. Las claves se deben rotar cuando falta un número especificado de días antes de la expiración, para proporcionar el tiempo suficiente para reaccionar ante un error. Esta directiva auditará las claves que estén demasiado cerca de su fecha de expiración y le permitirá establecer este umbral en días. También puede usar esta directiva para evitar la creación de nuevas claves que estén demasiado cerca de su fecha de expiración.

### <a name="keys-should-be-backed-by-a-hardware-security-module-preview"></a>Las claves deben estar respaldadas por un módulo de seguridad de hardware (versión preliminar)

Un HSM es un módulo de seguridad de hardware que almacena claves. Un HSM proporciona una capa física de protección para las claves criptográficas. La clave criptográfica no puede salir de un HSM físico que proporciona un mayor nivel de seguridad que una clave de software. Algunas organizaciones tienen requisitos de cumplimiento que exigen el uso de claves de HSM. Use esta directiva para auditar las claves almacenadas en el almacén de claves que no estén respaldadas por HSM. También puede usar esta directiva para bloquear la creación de claves que no están respaldadas por HSM. Esta directiva se aplicará a todos los tipos de clave, RSA y ECC.

### <a name="keys-using-rsa-cryptography-should-have-a-specified-minimum-key-size-preview"></a>Las claves que usan la criptografía RSA deben tener el tamaño de clave mínimo que se haya especificado (versión preliminar)

El uso de claves RSA con tamaños de clave más pequeños no es una práctica de diseño seguro. Puede estar sujeto a estándares de auditoría y certificación que exijan el uso de un tamaño de clave mínimo. La siguiente directiva le permite establecer un requisito mínimo de tamaño de clave en el almacén de claves. Puede auditar las claves que no cumplan este requisito mínimo. Esta directiva también se puede usar para bloquear la creación de nuevas claves que no cumplan el requisito de tamaño mínimo de clave.

### <a name="keys-should-have-the-specified-maximum-validity-period-preview"></a>Las claves deben tener el período de validez máximo especificado (versión preliminar)

Administre los requisitos de cumplimiento de su organización. Para ello, especifique la cantidad máxima de tiempo en días que una clave puede ser válido en el almacén de claves. Las claves que son válidas durante más tiempo que el umbral establecido se marcarán como no compatibles. También puede usar esta directiva para bloquear la creación de nuevas claves que tienen establecida una fecha de expiración mayor que el período de validez máximo que especifique.

# <a name="secret-policies"></a>[Directivas de secretos](#tab/secrets)

### <a name="secrets-should-not-be-active-for-longer-than-the-specified-number-of-days-preview"></a>Los secretos no deben estar activos durante más tiempo que el número especificado de días (versión preliminar)

Si desea asegurarse de que los secretos no estén activos durante más tiempo que un número especificado de días, puede usar esta directiva para auditar cuánto tiempo ha estado activo el secreto.

**Si el secreto tiene establecida una fecha de activación**, esta directiva calculará el número de días transcurridos desde la **fecha de activación** del secreto hasta la fecha actual. Si el número de días supera el umbral establecido, el secreto se marcará como no compatible con la directiva.

**Si el secreto no tiene establecida una fecha de activación**, esta directiva calculará el número de días transcurridos desde la **fecha de creación** del secreto hasta la fecha actual. Si el número de días supera el umbral establecido, el secreto se marcará como no compatible con la directiva.

### <a name="secrets-should-have-content-type-set-preview"></a>Los secretos deben tener establecido el tipo de contenido (versión preliminar)

Cualquier archivo de texto sin formato o codificado se puede almacenar como un secreto del almacén de claves. Sin embargo, puede que la organización quiera establecer diferentes directivas de rotación y restricciones en las contraseñas, las cadenas de conexión o los certificados almacenados como claves. Una etiqueta de tipo de contenido puede ayudar a un usuario a ver lo que se almacena en un objeto de secreto sin leer el valor del secreto. Puede usar esta directiva para auditar secretos que no tienen un conjunto de etiquetas de tipo de contenido. También puede usar esta directiva para evitar que se creen nuevos secretos si no tienen un conjunto de etiquetas de tipo de contenido.

### <a name="secrets-should-have-expiration-date-set-preview"></a>Los secretos deben tener establecida la fecha de expiración (versión preliminar)

Esta directiva audita todos los secretos de los almacenes de claves y marca los secretos que no tienen establecida una fecha de expiración como no compatibles. También puede usar esta directiva para bloquear la creación de secretos que no tienen establecida una fecha de expiración.

### <a name="secrets-should-have-more-than-the-specified-number-of-days-before-expiration-preview"></a>Deben faltar más días que el número especificado para la expiración de los secretos (versión preliminar)

Si un secreto está demasiado cerca de la expiración, un retraso de la organización para rotar el secreto puede producir una interrupción. Los secretos se deben rotar cuando falta un número especificado de días antes de la expiración, para proporcionar el tiempo suficiente para reaccionar ante un error. Esta directiva auditará los secretos que estén demasiado cerca de su fecha de expiración y le permitirá establecer este umbral en días. También puede usar esta directiva para evitar la creación de nuevos secretos que estén demasiado cerca de su fecha de expiración.

### <a name="secrets-should-have-the-specified-maximum-validity-period-preview"></a>Los secretos deben tener el período de validez máximo especificado (versión preliminar)

Administre los requisitos de cumplimiento de su organización. Para ello, especifique la cantidad máxima de tiempo en días que un secreto puede ser válido en el almacén de claves. Los secretos que son válidos durante más tiempo que el umbral establecido se marcarán como no compatibles. También puede usar esta directiva para bloquear la creación de nuevos secretos que tienen establecida una fecha de expiración mayor que el período de validez máximo que especifique.

---

## <a name="example-scenario"></a>Escenario de ejemplo

Puede administrar un almacén de claves usado por varios equipos que contengan 100 certificados y desea asegurarse de que ninguno de los certificados del almacén de claves sea válido durante más de 2 años.

1. Asigne la directiva **Los certificados deben tener el período de validez máximo especificado**, especifique que el período de validez máximo de un certificado es de 24 meses y establezca el efecto de la directiva en "auditoría". 
1. Puede ver el [informe de cumplimiento en el Azure Portal](#view-compliance-results) y detectar que 20 certificados son no compatibles y válidos durante > 2 años, y que los certificados restantes son conformes. 
1. Se pone en contacto con los propietarios de estos certificados y comunican el nuevo requisito de seguridad que los certificados no pueden ser válidos durante más de 2 años. Algunos equipos responden y 15 de los certificados se renovaron con un período de validez máximo de 2 años o menos. Otros equipos no responden y todavía tiene 5 certificados no compatibles en el almacén de claves.
1. Cambia el efecto de la directiva asignada a "denegar". Los 5 certificados no compatibles no se revocan y continúan funcionando. Sin embargo, no se pueden renovar con un período de validez superior a 2 años. 

## <a name="enabling-and-managing-a-key-vault-policy-through-the-azure-portal"></a>Habilitación y administración de una directiva de Key Vault a través del Azure Portal

### <a name="select-a-policy-definition"></a>Seleccione una definición de Directiva

1. Inicie sesión en el Portal de Azure. 
1. Busque "Directiva" en la barra de búsqueda y seleccione **Directiva**.

    ![Captura de pantalla que muestra la barra de búsqueda.](../media/policy-img1.png)

1. En la ventana Directiva, seleccione **Definiciones**.

    ![Captura de pantalla que resalta la opción Definiciones.](../media/policy-img2.png)

1. En el Filtro de categoría, anule la selección de **Seleccionar todos** y seleccione **Almacén de claves**. 

    ![Captura de pantalla que muestra el filtro de categoría y la categoría de Key Vault seleccionada.](../media/policy-img3.png)

1. Ahora debería poder ver todas las directivas disponibles para la Versión preliminar pública, por Azure Key Vault. Asegúrese de leer y comprender la sección de instrucciones de directiva anterior y seleccionar una directiva que desee asignar a un ámbito.  

    ![Captura de pantalla que muestra las directivas que están disponibles para la versión preliminar pública.](../media/policy-img4.png)

### <a name="assign-a-policy-to-a-scope"></a>Asignar una Directiva a un Ámbito 

1. Seleccione la directiva que quiere aplicar en este ejemplo, se muestra la directiva **Administrar el período de validez del certificado**. Haga clic en el botón asignar situado en la esquina superior izquierda.

    ![Captura de pantalla que muestra la directiva Administrar el período de validez del certificado.](../media/policy-img5.png)
  
1. Seleccione la suscripción en la que desea que se aplique la directiva. Puede optar por restringir el ámbito a un solo grupo de recursos dentro de una suscripción. Si desea aplicar la directiva a toda la suscripción y excluir algunos grupos de recursos, también puede configurar una lista de exclusión. Establezca el selector de cumplimiento de directivas en **Habilitado** si quiere que se produzca el efecto de la directiva (auditoría o denegación) o **Deshabilitado** para desactivar el efecto (auditoría o denegación). 

    ![Captura de pantalla que muestra dónde puede optar por restringir el ámbito a un solo grupo de recursos dentro de una suscripción.](../media/policy-img6.png)

1. Haga clic en la pestaña parámetros en la parte superior de la pantalla para especificar el período de validez máximo en meses que desee. Seleccione **auditoría** o **denegar** para el efecto de la directiva siguiendo las instrucciones de las secciones anteriores. Después, seleccione el botón revisar y crear. 

    ![Captura de pantalla que muestra la pestaña Parámetros en la que puede especificar el período de validez máximo en meses que desee.](../media/policy-img7.png)

### <a name="view-compliance-results"></a>Ver Resultados de Cumplimiento

1. Vuelva a la hoja Directiva y seleccione la pestaña cumplimiento. Haga clic en la asignación de directiva para la que quiere ver los resultados de cumplimiento.

    ![Captura de pantalla que muestra la pestaña Cumplimiento, donde puede seleccionar la asignación de directiva de la que desea ver los resultados de cumplimiento.](../media/policy-img8.png)

1. En esta página puede filtrar los resultados por almacenes compatibles o no compatibles. Aquí puede ver una lista de almacenes de claves no compatibles dentro del ámbito de la asignación de directiva. Un almacén se considera no compatible si alguno de los componentes (certificados) del almacén no es compatible. Puede seleccionar un almacén individual para ver los componentes individuales no compatibles (certificados). 


    ![Captura de pantalla que muestra una lista de almacenes de claves no compatibles dentro del ámbito de la asignación de directiva.](../media/policy-img9.png)

1. Ver el nombre de los componentes de un almacén que no son compatibles


    ![Captura de pantalla que muestra dónde puede ver el nombre de los componentes de un almacén que no son compatibles.](../media/policy-img10.png)

1. Si tiene que comprobar si se deniega a los usuarios la posibilidad de crear recursos en el almacén de claves, puede hacer clic en la pestaña **Eventos de componentes (versión preliminar)** para ver un resumen de las operaciones de certificado denegado con el solicitante y las marcas de tiempo de las solicitudes. 


    ![Introducción al funcionamiento de Azure Key Vault](../media/policy-img11.png)

## <a name="feature-limitations"></a>Limitaciones de característica

Asignar una directiva con un efecto de "denegación" puede tardar hasta 30 minutos (caso promedio) y 1 hora (peor de los casos) para empezar a denegar la creación de recursos no compatibles. La evaluación de la directiva de los componentes existentes en un almacén puede tardar hasta 1 hora (caso promedio) y 2 horas (peor de los casos) antes de que se puedan ver los resultados de cumplimiento en la interfaz de usuario del portal. Si los resultados de cumplimiento se muestran como "No iniciado", puede deberse a los siguientes motivos:
- La valoración de la directiva no se ha completado todavía. La latencia de evaluación inicial puede tardar hasta 2 horas en el peor de los casos. 
- No hay almacén de claves en el ámbito de la asignación de directiva.
- No hay almacén de claves con certificados dentro del ámbito de la asignación de directivas.

> [!NOTE]
> Los [modos de proveedor de recursos](../../governance/policy/concepts/definition-structure.md#resource-provider-modes) de Azure Policy, como los de Azure Key Vault, proporcionan información sobre el cumplimiento en la página [Compatibilidad de componentes](../../governance/policy/how-to/get-compliance-data.md#component-compliance).

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre el [servicio de Azure Policy](../../governance/policy/overview.md)
- Vea ejemplos de Key Vault: [Definiciones de directivas integradas en Key Vault](../../governance/policy/samples/built-in-policies.md#key-vault)
