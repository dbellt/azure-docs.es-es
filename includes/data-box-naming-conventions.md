---
author: alkohli
ms.service: databox
ms.subservice: heavy
ms.topic: include
ms.date: 05/21/2019
ms.author: alkohli
ms.openlocfilehash: d965b89659a9e3dc01035221a729f094c336eb5b
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "66244604"
---
| Entidad                                       | Convenciones   |
|----------------------------------------------|-----------------------------------------------------|
| Nombres de contenedor de blob en bloques y blob en páginas | Debe ser un nombre DNS válido que tenga entre 3 y 63 caracteres. <br>  Debe empezar por una letra o un número. <br> Solo puede contener letras minúsculas, números y el guion (-). <br> Los caracteres de guión (-) deben estar inmediatamente precedidos y seguidos por una letra o un número. <br> No se permiten guiones consecutivos en nombres. |
| Nombres de recursos compartidos de archivos de Azure                  | Lo mismo que antes.                                          |
| Nombres de archivos y directorios para archivos de Azure     |<li> No distingue mayúsculas y minúsculas, mantiene las mayúsculas y minúsculas, y no debe superar los 255 caracteres. </li><li> No puede terminar en una barra diagonal (/). </li><li>Si se proporciona, se quitará automáticamente. </li><li> No se permiten los caracteres siguientes: <code>" \\ / : \| < > * ?</code></li><li> Los caracteres de URL reservadas deben convertirse correspondientemente. </li><li> No se permiten caracteres no válidos en la ruta de acceso de la dirección URL. Los puntos de código como \\uE000 no son caracteres Unicode válidos. Tampoco se permiten algunos caracteres ASCII o Unicode, como los caracteres de control (0x00 a 0x1F, \\u0081, etc.). Para conocer las reglas que rigen las cadenas Unicode en HTTP/1.1, consulte RFC 2616, sección 2.2: Basic Rules y RFC 3987. </li><li> No se permiten los siguientes nombres de archivo: LPT1, LPT2, LPT3, LPT4, LPT5, LPT6, LPT7, LPT8, LPT9, COM1, COM2, COM3, COM4, COM5, COM6, COM7, COM8, COM9, PRN, AUX, NUL, CON, CLOCK$, carácter de punto (.) y caracteres de dos puntos (..).</li>|
| Nombres de blob para blob en bloques y blob en páginas      | </li><li>Los nombres de blob distinguen mayúsculas de minúsculas y pueden contener cualquier combinación de caracteres. </li><li>Un nombre de blob debe tener entre 1 y 1024 caracteres. </li><li>Los caracteres de URL reservadas deben convertirse correspondientemente. </li><li>El número de segmentos de ruta de acceso que componen el nombre del blob no puede superar los 254. Un segmento de ruta de acceso es la cadena entre caracteres delimitadores consecutivos (por ejemplo, la barra diagonal "/") que se corresponden con el nombre de un directorio virtual.</li> |
