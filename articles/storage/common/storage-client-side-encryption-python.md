---
title: Cifrado en el lado de cliente con Python
titleSuffix: Azure Storage
description: Biblioteca de cliente de Azure Storage para Python admite el cifrado de lado cliente para obtener una seguridad máxima de las aplicaciones de Azure Storage.
services: storage
author: tamram
ms.service: storage
ms.devlang: python
ms.topic: how-to
ms.date: 02/18/2021
ms.author: tamram
ms.reviewer: ozgun
ms.subservice: common
ms.openlocfilehash: b76a1b8fa3a7d42f8b649adc225af89b4a40f15c
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110461817"
---
# <a name="client-side-encryption-with-python"></a>Cifrado en el lado de cliente con Python

[!INCLUDE [storage-selector-client-side-encryption-include](../../../includes/storage-selector-client-side-encryption-include.md)]

## <a name="overview"></a>Información general
La [Biblioteca de cliente de Azure Storage para Python](https://pypi.python.org/pypi/azure-storage) permite tanto el cifrado de datos dentro de las aplicaciones de cliente antes de cargarlos en Azure Storage, como el descifrado de datos mientras estos se descargan al cliente.

> [!NOTE]
> La biblioteca de Python para Azure Storage está en su versión preliminar.
> 
> 

## <a name="encryption-and-decryption-via-the-envelope-technique"></a>Cifrado y descifrado a través de la técnica de sobres
El proceso de cifrado y descifrado sigue la técnica de sobres.

### <a name="encryption-via-the-envelope-technique"></a>Cifrado a través de la técnica de sobres
El cifrado mediante la técnica de sobres funciona de la siguiente manera:

1. La biblioteca de cliente de almacenamiento de Azure genera una clave de cifrado de contenido (CEK), que es una clave simétrica de un solo uso.
2. Los datos de usuario se cifran mediante esta CEK.
3. Se encapsula la CEK (cifrada) con la clave de cifrado de clave (KEK). La KEK se identifica mediante un identificador de clave y puede ser un par de clave asimétrico o una clave simétrica que se administra de forma local.
   La propia biblioteca de cliente de almacenamiento no tiene nunca acceso a la KEK. La biblioteca invoca el algoritmo de encapsulado de clave proporcionado por la KEK. Los usuarios pueden elegir utilizar proveedores personalizados para el ajuste y desajuste clave si lo desean.
4. A continuación, se cargan los datos cifrados en el servicio Azure Storage. La clave encapsulada y algunos metadatos adicionales de cifrado se almacenan como metadatos (en un blob) o se interpolan con los datos cifrados (cola de mensajes y las entidades de tabla).

### <a name="decryption-via-the-envelope-technique"></a>Descifrado a través de la técnica de sobres
El descifrado mediante la técnica de sobres funciona de la siguiente manera:

1. La biblioteca de cliente asume que el usuario está administrando la clave de cifrado de claves (KEK) de manera local. El usuario no necesita conocer la clave específica que se usó para el cifrado. En su lugar, se puede configurar y usar una resolución de clave que resuelva distintos identificadores de clave para las claves.
2. La biblioteca de cliente descarga los datos cifrados junto con cualquier material de cifrado que esté almacenado en el servicio.
3. A continuación, la clave de cifrado de contenido encapsulado (CEK) se desencapsula (descifra) usando la clave de cifrado de claves (KEK). Aquí nuevamente, la biblioteca de cliente no tiene acceso a la KEK. Simplemente invoca el algoritmo de desencapsulado del proveedor personalizado.
4. La clave de cifrado de contenido (CEK) se usa entonces para descifrar los datos cifrados del usuario.

## <a name="encryption-mechanism"></a>Mecanismo de cifrado
La biblioteca de cliente de almacenamiento usa [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) para cifrar los datos del usuario. En concreto, emplea el modo [Cipher Block Chaining (CBC)](https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation#Cipher-block_chaining_.28CBC.29) con AES. Cada servicio funciona de forma ligeramente diferente, por lo que describiremos aquí cada uno de ellos.

### <a name="blobs"></a>Datos BLOB
La biblioteca de cliente solo admite actualmente el cifrado de blobs completos. En concreto, se admite el cifrado cuando los usuarios emplean los métodos **create**\*. En el caso de las descargas, se admiten tanto las descargas de intervalo como las completas, y está disponible la paralelización tanto de la carga como de la descarga.

Durante el cifrado, la biblioteca de cliente generará un vector de inicialización (IV) aleatorio de 16 bytes, junto con una clave de cifrado de contenido (CEK) aleatoria de 32 bytes, y realiza el cifrado de sobres de los datos de blob con esta información. Posteriormente, la CEK encapsulada y algunos metadatos de cifrado adicionales se almacenan como metadatos de blob junto con el objeto blob cifrado en el servicio.

> [!WARNING]
> Si está modificando o cargando sus propios metadatos para el blob, deberá asegurarse de que estos metadatos se conserven. Si carga nuevos metadatos sin estos metadatos, la CEK encapsulada, IV y otros metadatos se perderán y el contenido del blob nunca podrá recuperarse nuevamente.
> 
> 

Descargar un blob cifrado implica recuperar el contenido del blob completo mediante los cómodos métodos **get**\*. La CEK encapsulada se desencapsula y se utiliza junto con el vector de inicialización (que se almacena como metadatos de blob, en este caso) para devolver los datos descifrados a los usuarios.

Descargar un intervalo arbitrario (métodos **get**\* con parámetros de intervalo transferidos) en el blob cifrado implica ajustar el intervalo proporcionado por los usuarios para obtener una pequeña cantidad de datos adicionales que pueden usarse para descifrar correctamente el intervalo solicitado.

Los blobs en bloques y los blobs en páginas solo se pueden cifrar y descifrar usando este esquema. Actualmente, no se admite el cifrado de blobs en anexos.

### <a name="queues"></a>Colas
Dado que la cola de mensajes puede tener cualquier formato, la biblioteca de cliente define un formato personalizado que incluye el vector de inicialización (IV) y la clave de cifrado de contenido (CEK) cifrada en el texto del mensaje.

Durante el cifrado, la biblioteca de cliente genera un vector de inicialización aleatorio de 16 bytes junto con una CEK aleatoria de 32 bytes y realiza el cifrado de sobres del texto del mensaje de cola con esta información. La CEK encapsulada y algunos metadatos de cifrado adicionales se incluyen entonces en el mensaje de cola cifrado. Este mensaje modificado (que se muestra a continuación) se almacena en el servicio.

```
<MessageText>{"EncryptedMessageContents":"6kOu8Rq1C3+M1QO4alKLmWthWXSmHV3mEfxBAgP9QGTU++MKn2uPq3t2UjF1DO6w","EncryptionData":{…}}</MessageText>
```

Durante el descifrado, la clave encapsulada se extrae del mensaje de cola y se desencapsula. El vector de inicialización también se extrae del mensaje de cola y se utiliza junto con la clave desencapsulada para descifrar los datos de mensaje de cola. Tenga en cuenta que los metadatos de cifrado son pequeños (menos de 500 bytes), por lo que, aunque se tienen en cuenta para el límite de 64 KB para un mensaje de la cola, el impacto debería ser fácil de administrar.

### <a name="tables"></a>Tablas
La biblioteca de cliente admite el cifrado de propiedades de entidad para operaciones de insertar y reemplazar.

> [!NOTE]
> Las operaciones de combinación no se admiten actualmente. Puesto que un subconjunto de propiedades puede haberse cifrado previamente con una clave distinta, si simplemente se combinan las nuevas propiedades y se actualizan los metadatos, se producirá una pérdida de datos. Para realizar una combinación es necesario realizar llamadas de servicio adicionales para leer la entidad existente desde el servicio. También puede usar una nueva clave por propiedad. Ninguno de estos procedimientos es adecuado por motivos de rendimiento.
> 
> 

El cifrado de datos de tabla funciona de la siguiente forma:

1. Los usuarios especifican las propiedades que se van a cifrar.
2. La biblioteca de cliente genera un vector de inicialización (IV) aleatorio de 16 bytes junto con una clave de cifrado de contenido (CEK) aleatoria de 32 bytes para cada entidad. Después, realiza el cifrado de sobres en las propiedades individuales que se van a cifrar derivando un nuevo vector de inicialización por propiedad. La propiedad de cifrado se almacena como datos binarios.
3. La CEK encapsulada y algunos metadatos de cifrado adicional se almacenan posteriormente como dos propiedades reservadas adicionales. La primera propiedad reservada (\_ClientEncryptionMetadata1) es una propiedad de cadena que contiene la información sobre el vector de inicialización, la versión y la clave encapsulada. La segunda propiedad reservada (\_ClientEncryptionMetadata2) es una propiedad binaria que contiene la información sobre las propiedades que se cifran. La información de esta segunda propiedad (\_ClientEncryptionMetadata2) está ella misma cifrada.
4. A causa de estas propiedades reservadas adicionales necesarias para el cifrado, es posible que los usuarios ahora tengan solo 250 propiedades personalizadas en lugar de 252. El tamaño total de la entidad debe ser inferior a 1 MB.

   Tenga en cuenta que solo se pueden cifrar las propiedades de cadena. Si hay que cifrar otros tipos de propiedades, habrá que convertirlas en cadenas. Las cadenas cifradas se almacenan en el servicio como propiedades binarias y se convierten en cadenas (cadenas sin formato, no EntityProperties con tipo EdmType.STRING) después del descifrado.

   Para las tablas, además de la directiva de cifrado, los usuarios deben especificar las propiedades que se van a cifrar. Para ello, estas propiedades se pueden almacenar en objetos TableEntity con el tipo establecido en EdmType.STRING y el cifrado establecido en true, o bien se puede establecer el valor encryption_resolver_function en el objeto tableservice. Una resolución de cifrado es una función que toma una clave de partición, una clave de fila y un nombre de propiedad y devuelve un valor booleano que indica si se debe cifrar dicha propiedad. Durante el cifrado, la biblioteca de cliente usará esta información para decidir si se debe cifrar una propiedad mientras se escribe en la conexión. El delegado también proporciona la posibilidad de lógica con respecto a la forma de cifrar las propiedades. (Por ejemplo, si el valor es X, hay que cifrar la propiedad A; en caso contrario, hay que cifrar las propiedades A y B). Tenga en cuenta que no es necesario proporcionar esta información para leer o consultar entidades.

### <a name="batch-operations"></a>Operaciones por lotes
Una directiva de cifrado se aplica a todas las filas del lote. La biblioteca de cliente generará internamente un nuevo vector de inicialización aleatorio y una CEK aleatoria por cada fila del lote. Los usuarios también pueden optar por cifrar diferentes propiedades para cada operación del lote mediante la definición de este comportamiento en la resolución de cifrado.
Si se crea un lote como el administrador de contexto a través del método tableservice batch(), la directiva de cifrado de tableservice se aplicará automáticamente al lote. Si para crear explícitamente un lote se llama al constructor, la directiva de cifrado se debe transmitir como un parámetro y no se debe modificar durante todo el ciclo de vida del lote.
Tenga en cuenta que las entidades se cifran en cuanto se insertan en el lote con la directiva de cifrado del lote (las entidades NO se cifran en el momento de enviar el lote con la directiva de cifrado de tableservice).

### <a name="queries"></a>Consultas
> [!NOTE]
> Dado que las entidades están cifradas, no se pueden ejecutar consultas que filtran por una propiedad cifrada.  Si lo intenta, los resultados serán incorrectos, porque el servicio estaría intentando comparar los datos cifrados con los datos sin cifrar.
> 
> 
> Para realizar operaciones de consulta, debe especificar a una resolución de clave que sea capaz de resolver todas las claves en el conjunto de resultados. Si una entidad incluida en el resultado de la consulta no se puede resolver en un proveedor, la biblioteca de cliente producirá un error. Para cualquier consulta que realice proyecciones del lado servidor, la biblioteca de cliente agregará las propiedades de metadatos de cifrado especiales (\_ClientEncryptionMetadata1 y \_ClientEncryptionMetadata2) a las columnas seleccionadas de forma predeterminada.
> 
> [!IMPORTANT]
> Tenga en cuenta estos puntos importantes al usar el cifrado del lado del cliente:
> 
> * Al leer desde un blob cifrado o escribir en él, utilice comandos de carga completa del blob y comandos de descarga de blobs de intervalo/completos. Evite escribir en un blob cifrado mediante operaciones de protocolo, como Colocar bloque, Colocar lista de bloque, Escribir páginas o Borrar páginas; de lo contrario, puede dañar el objeto blob cifrado y que no sea legible.
> * Para las tablas, existe una restricción similar. Tenga cuidado de no actualizar propiedades cifradas sin actualizar los metadatos de cifrado.
> * Si establece los metadatos en el objeto blob cifrado, puede sobrescribir los metadatos relacionados con el cifrado necesarios para el descifrado, ya que el establecimiento de metadatos no es aditivo. Esto también se aplica a las instantáneas: evite la especificación de metadatos durante la creación de una instantánea de un blob cifrado. Si se deben establecer metadatos, asegúrese de llamar primero al método **get_blob_metadata**, para así obtener los metadatos de cifrado actuales y evitar las escrituras simultáneas mientras estos se establecen.
> * Habilite la marca **require_encryption** en el objeto de servicio para los usuarios que deban trabajar solo con datos cifrados. Vea a continuación para obtener más información.

La biblioteca de cliente de almacenamiento espera la KEK proporcionada y la resolución de claves para implementar la interfaz siguiente. [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) con la administración de KEK de Python está pendiente y se integrará a esta biblioteca cuando se complete.

## <a name="client-api--interface"></a>Interfaz/API de cliente
Una vez que se crea un objeto de servicio de almacenamiento (es decir, blockblobservice), el usuario puede asignar valores a los campos que constituyen una directiva de cifrado: key_encryption_key, key_resolver_function y require_encryption. Los usuarios pueden proporcionar solo una KEK, solo una resolución o ambos. key_encryption_key es el tipo de clave básico que se identifica mediante un identificador de claves y que proporciona la lógica para la encapsulación y desencapsulación. key_resolver_function se usa para resolver una clave durante el proceso de descifrado. Devuelve una KEK válida en función de un identificador de clave. Esto ofrece a los usuarios la posibilidad de elegir entre varias claves que se administran en varias ubicaciones.

La KEK debe implementar los métodos siguientes para cifrar correctamente los datos:

* wrap_key(cek): encapsula la CEK especificada (bytes) con un algoritmo a elección del usuario. Devuelve la clave encapsulada.
* get_key_wrap_algorithm(): devuelve el algoritmo que se usa para encapsular las claves.
* get_kid(): devuelve el id. de clave de cadena de esta KEK.
  La KEK debe implementar los métodos siguientes para descifrar correctamente los datos:
* unwrap_key(cek, algoritmo): devuelve el formato desencapsulado de la CEK especificada con el algoritmo especificado en la cadena.
* get_kid(): devuelve un id. de clave de cadena de esta KEK.

La resolución de claves debe, como mínimo, implementar un método que, dado un id. de clave, devuelve la KEK correspondiente que implementa la interfaz anterior. Solo este método se asignará a la propiedad key_resolver_function en el objeto de servicio.

* Para el cifrado, se utiliza siempre la clave. Si no hay clave, se producirá un error.
* Para el descifrado:

  * La resolución de claves se invoca si se especifica para obtener la clave. Si se especifica la resolución, pero no se proporciona una asignación para el identificador de clave, se produce un error.
  * Si no se especifica la resolución, pero sí se especifica una clave, la clave se usa si su identificador coincide con el identificador de clave necesario. Si el identificador no coincide, se genera un error.

    Los ejemplos de cifrado en azure.storage.samples muestran un escenario más detallado de un extremo a otro para blobs, colas y tablas.
      En los archivos de ejemplo, se proporcionan ejemplos de implementaciones de la KEK y de la resolución de claves, como KeyWrapper y KeyResolver, respectivamente.

### <a name="requireencryption-mode"></a>Modo RequireEncryption
Los usuarios pueden habilitar opcionalmente un modo de operación en el que se deben cifrar todas las cargas y descargas. En este modo, los intentos de cargar datos sin una directiva de cifrado o de descargar datos no cifrados en el servicio generarán un error en el cliente. La marca **require_encryption** en el objeto de servicio controla este comportamiento.

### <a name="blob-service-encryption"></a>Cifrado de Blob service
Establezca los campos de directiva de cifrado en el objeto blockblobservice. Todo lo demás lo controlará la biblioteca de cliente internamente.

# <a name="python-v12-sdk"></a>[SDK de Python v12](#tab/python)

Actualmente, estamos trabajando para crear fragmentos de código que reflejen la versión 12.x de las bibliotecas cliente de Azure Storage. Para más información, consulte el [anuncio de las bibliotecas cliente de Azure Storage v12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1')  # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_block_blob_service.key_encryption_key = kek
my_block_blob_service.key_resolver_funcion = key_resolver.resolve_key

# Upload the encrypted contents to the blob.
my_block_blob_service.create_blob_from_stream(
    container_name, blob_name, stream)

# Download and decrypt the encrypted contents from the blob.
blob = my_block_blob_service.get_blob_to_bytes(container_name, blob_name)
```
---

### <a name="queue-service-encryption"></a>Cifrado del servicio Cola
Establezca los campos de directiva de cifrado en el objeto queueservice. Todo lo demás lo controlará la biblioteca de cliente internamente.

# <a name="python-v12-sdk"></a>[SDK de Python v12](#tab/python)

Actualmente, estamos trabajando para crear fragmentos de código que reflejen la versión 12.x de las bibliotecas cliente de Azure Storage. Para más información, consulte el [anuncio de las bibliotecas cliente de Azure Storage v12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1')  # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Set the KEK and key resolver on the service object.
my_queue_service.key_encryption_key = kek
my_queue_service.key_resolver_funcion = key_resolver.resolve_key

# Add message
my_queue_service.put_message(queue_name, content)

# Retrieve message
retrieved_message_list = my_queue_service.get_messages(queue_name)
```
---

### <a name="table-service-encryption"></a>Cifrado de Table service
Además de crear una directiva de cifrado y configurarla en las opciones de solicitud, debe especificar un elemento **encryption_resolver_function** en **tableservice**, o bien establecer el atributo de cifrado en EntityProperty.

### <a name="using-the-resolver"></a>Uso de la resolución

# <a name="python-v12-sdk"></a>[SDK de Python v12](#tab/python)

Actualmente, estamos trabajando para crear fragmentos de código que reflejen la versión 12.x de las bibliotecas cliente de Azure Storage. Para más información, consulte el [anuncio de las bibliotecas cliente de Azure Storage v12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

```python
# Create the KEK used for encryption.
# KeyWrapper is the provided sample implementation, but the user may use their own object as long as it implements the interface above.
kek = KeyWrapper('local:key1')  # Key identifier

# Create the key resolver used for decryption.
# KeyResolver is the provided sample implementation, but the user may use whatever implementation they choose so long as the function set on the service object behaves appropriately.
key_resolver = KeyResolver()
key_resolver.put_key(kek)

# Define the encryption resolver_function.


def my_encryption_resolver(pk, rk, property_name):
    if property_name == 'foo':
        return True
    return False


# Set the KEK and key resolver on the service object.
my_table_service.key_encryption_key = kek
my_table_service.key_resolver_funcion = key_resolver.resolve_key
my_table_service.encryption_resolver_function = my_encryption_resolver

# Insert Entity
my_table_service.insert_entity(table_name, entity)

# Retrieve Entity
# Note: No need to specify an encryption resolver for retrieve, but it is harmless to leave the property set.
my_table_service.get_entity(
    table_name, entity['PartitionKey'], entity['RowKey'])
```
---

### <a name="using-attributes"></a>Uso de los atributos
Tal como se mencionó anteriormente, se puede marcar una propiedad para cifrado si se la almacena en un objeto EntityProperty y se establece el campo de cifrado.

# <a name="python-v12-sdk"></a>[SDK de Python v12](#tab/python)

Actualmente, estamos trabajando para crear fragmentos de código que reflejen la versión 12.x de las bibliotecas cliente de Azure Storage. Para más información, consulte el [anuncio de las bibliotecas cliente de Azure Storage v12](https://techcommunity.microsoft.com/t5/azure-storage/announcing-the-azure-storage-v12-client-libraries/ba-p/1482394).

# <a name="python-v21"></a>[Python v2.1](#tab/python2)

```python
encrypted_property_1 = EntityProperty(EdmType.STRING, value, encrypt=True)
```
---

## <a name="encryption-and-performance"></a>Cifrado y rendimiento
Tenga en cuenta que el cifrado de sus resultados de datos de almacenamiento da lugar a la sobrecarga de rendimiento adicional. Se deben generar la clave de contenido e IV, se debe cifrar el contenido mismo y se debe dar formato a metadatos adicionales, además de cargarlos. Esta sobrecarga variará según la cantidad de datos que se cifran. Se recomienda que los clientes prueben siempre sus aplicaciones para obtener un rendimiento durante el desarrollo.

## <a name="next-steps"></a>Pasos siguientes
* Descargue el [paquete PyPi de Biblioteca de cliente de Azure Storage para Java](https://pypi.python.org/pypi/azure-storage)
* Descargue el [Código fuente de la Biblioteca de cliente de Azure Storage para Python desde GitHub](https://github.com/Azure/azure-storage-python)
