---
title: Cifrados admitidos para Azure FXT Edge Filer
description: Lista de estándares de cifrado usados por los clústeres de FXT Edge Filer.
author: ekpgh
ms.author: v-erkel
ms.service: fxt-edge-filer
ms.topic: conceptual
ms.date: 05/20/2021
ms.openlocfilehash: b395e0cd2358ef22a223a58bcf8041fa8052b9cd
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111377480"
---
# <a name="supported-encryption-standards-for-azure-fxt-edge-filer"></a>Estándares de cifrado admitidos para Azure FXT Edge Filer

En este documento se describen los estándares de cifrado necesarios para Azure FXT Edge Filer. Estos estándares se implementan a partir de la versión 5.1.1.2 del sistema operativo.

Estos estándares se aplican a [Avere vFXT for Azure](../avere-vfxt/index.yml), así como a Azure FXT Edge Filer.

Cualquier sistema administrativo o de infraestructura que se conecte a la caché de Azure FXT Edge Filer o a nodos individuales debe cumplir estos estándares.

(Las máquinas cliente montan la caché mediante NFS, por lo que no se aplican estos requisitos de cifrado. Use otras medidas razonables para garantizar su seguridad).

## <a name="tls-standard"></a>Estándar TLS

* TLS 1.2 debe estar habilitado.
* SSL V2 y V3 deben estar deshabilitados.

TLS1.0 y TLS1.1 se pueden usar si es absolutamente necesario para la compatibilidad con versiones anteriores con almacenes de objetos privados, pero es mejor actualizar el almacenamiento privado a estándares de seguridad modernos. Póngase en contacto con el servicio al cliente y el soporte técnico de Microsoft para más información.

## <a name="permitted-cipher-suites"></a>Conjuntos de cifrado permitidos

Azure FXT Edge Filer permite negociar los siguientes conjuntos de cifrado TLS:

* ECDHE-ECDSA-AES128-GCM-SHA256
* ECDHE-ECDSA-AES256-GCM-SHA384
* ECDHE-RSA-AES128-GCM-SHA256
* ECDHE-RSA-AES256-GCM-SHA384
* ECDHE-ECDSA-AES128-SHA256
* ECDHE-ECDSA-AES256-SHA384
* ECDHE-RSA-AES128-SHA256
* ECDHE-RSA-AES256-SHA384

La interfaz HTTPS administrativa del clúster (que se usa para la GUI web del Panel de control y las conexiones RPC administrativas) solo admite los conjuntos de cifrado anteriores y TLS 1.2. No se admiten otros protocolos ni conjuntos de cifrado al conectarse a la interfaz administrativa.

## <a name="ssh-server-access"></a>Acceso al servidor SSH

Estos estándares se aplican al servidor SSH que está insertado en estos productos.

El servidor SSH no permite el inicio de sesión remoto con el superusuario "root". Si se requiere acceso remoto a SSH según las instrucciones del servicio de atención al cliente y soporte técnico de Microsoft, inicie sesión como usuario "admin" de SSH, que tiene un shell restringido.

Los siguientes conjuntos de cifrado SSH están disponibles en el servidor SSH del clúster. Asegúrese de que cualquier cliente que use SSH para conectarse al clúster tenga software actualizado que cumpla estos estándares.

### <a name="ssh-encryption-standards"></a>Estándares de cifrado SSH

| Tipo | Valores admitidos |
|--|--|
| Ciphers | aes256-gcm@openssh.com</br> aes128-gcm@openssh.com</br> aes256-ctr</br> aes128-ctr |
| MAC | hmac-sha2-512-etm@openssh.com</br> hmac-sha2-256-etm@openssh.com</br> hmac-sha2-512</br> hmac-sha2-256 |
| Algoritmos KEX | ecdh-sha2-nistp521</br> ecdh-sha2-nistp384</br> ecdh-sha2-nistp256</br> diffie-hellman-group-exchange-sha256 |
