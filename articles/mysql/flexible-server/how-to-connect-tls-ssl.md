---
title: 'Conectividad cifrada mediante TLS/SSL en Azure Database for MySQL: servidor flexible'
description: 'Instrucciones e información sobre cómo conectarse mediante TLS/SSL en Azure Database for MySQL: servidor flexible.'
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 09/21/2020
ms.openlocfilehash: 399cf8087d39f78184cfdae4b9f0e34efecaea66
ms.sourcegitcommit: bfa7d6ac93afe5f039d68c0ac389f06257223b42
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2021
ms.locfileid: "106491625"
---
# <a name="connect-to-azure-database-for-mysql---flexible-server-with-encrypted-connections"></a>Conexión al servidor flexible de Azure Database for MySQL con conexiones cifradas

> [!IMPORTANT]
> Azure Database for MySQL: servidor flexible está actualmente en versión preliminar pública.

El servidor flexible de Azure Database for MySQL admite la conexión de las aplicaciones cliente al servidor MySQL mediante el cifrado de la Capa de sockets seguros (SSL) con la Seguridad de la capa de transporte (TLS). TLS es un protocolo estándar del sector que garantiza conexiones de red cifradas entre el servidor de bases de datos y las aplicaciones cliente, lo que le permite ajustarse a los requisitos de cumplimiento.

De forma predeterminada, el servidor flexible de Azure Database for MySQL solo admite conexiones cifradas mediante la Seguridad de la capa de transporte (TLS 1.2), y todas las conexiones entrantes con TLS 1.0 y TLS 1.1 se denegarán de forma predeterminada. La obligatoriedad de conexiones cifradas o la configuración de versiones de TLS en el servidor flexible se pueden cambiar como se indica en este artículo. 

A continuación, se muestran las distintas configuraciones de SSL y TLS que puede tener para el servidor flexible:

| Escenario   | Configuración de parámetros del servidor      | Descripción                                    |
|------------|--------------------------------|------------------------------------------------|
|Deshabilitar SSL (conexiones cifradas) | require_secure_transport = OFF |Si la aplicación heredada no admite conexiones cifradas al servidor MySQL, puede deshabilitar la obligatoriedad de conexiones cifradas en el servidor flexible estableciendo require_secure_transport=OFF.|
|Exigir SSL con la versión de TLS < 1.2 | require_secure_transport = ON y tls_version = TLSV1 o TLSV1.1| Si la aplicación heredada admite conexiones cifradas, pero necesita la versión de TLS > 1, puede permitir conexiones cifradas, pero configurar el servidor flexible para permitir conexiones con la versión de TLS (v1.0 o v1.1) admitida por la aplicación.|
|Exigir SSL con la versión de TLS = 1.2 (configuración predeterminada)|require_secure_transport = ON y tls_version = TLSV1.2| Esta es la configuración predeterminada y recomendada para el servidor flexible.|
|Exigir SSL con la versión de TLS = 1.3 (compatible con MySQL v8.0 y versiones posteriores)| require_secure_transport = ON y tls_version = TLSV1.3| Esta configuración es útil y recomendada para el desarrollo de nuevas aplicaciones|

> [!Note]
> No se admiten cambios en el cifrado SSL del servidor flexible. Los conjuntos de cifrado FIPS se aplican de forma predeterminada cuando tls_version está establecido en la versión de TLS 1.2. En el caso de versiones de TLS distintas de la versión 1.2, el cifrado SSL se establece en la configuración predeterminada que se incluye con la instalación de la comunidad de MySQL.

En este artículo, aprenderá a:
* Configurar el servidor flexible 
  * Con SSL deshabilitado 
  * Con SSL exigido con la versión de TLS < 1.2
* Conectarse al servidor mediante la línea de comandos de MySQL 
  * Con conexiones cifradas deshabilitadas
  * Con conexiones cifradas habilitadas
* Comprobar el estado de cifrado de la conexión
* Conectarse al servidor flexible con conexiones cifradas mediante varios marcos de trabajo de aplicaciones

## <a name="disable-ssl-on-your-flexible-server"></a>Deshabilitación de SSL en el servidor flexible
Si la aplicación cliente no admite conexiones cifradas, tendrá que deshabilitar la obligatoriedad de conexiones cifradas en el servidor flexible. Para deshabilitar la obligatoriedad de conexiones cifradas, debe establecer el parámetro de servidor require_secure_transport en OFF, tal y como se muestra en la captura de pantalla, y guardar la configuración para que surta efecto. require_secure_transport es un **parámetro de servidor dinámico** que surte efecto inmediatamente sin necesidad de reiniciar el servidor.

> :::image type="content" source="./media/how-to-connect-tls-ssl/disable-ssl.png" alt-text="Captura de pantalla que muestra cómo deshabilitar SSL con el servidor flexible de Azure Database for MySQL.":::

### <a name="connect-using-mysql-command-line-client-with-ssl-disabled"></a>Conexión mediante el cliente de la línea de comandos de MySQL con SSL deshabilitado

En el ejemplo siguiente se muestra cómo conectarse al servidor mediante la interfaz de la línea de comandos mysql. Use la configuración de cadena de conexión `--ssl-mode=DISABLED` para deshabilitar la conexión TLS/SSL del cliente de MySQL. Reemplace los valores por el nombre real del servidor y la contraseña. 

```bash
 mysql.exe -h mydemoserver.mysql.database.azure.com -u myadmin -p --ssl-mode=DISABLED 
```
Es importante tener en cuenta que establecer require_secure_transport en OFF no significa que las conexiones cifradas no se admitan en el lado servidor. Si establece require_secure_transport en OFF en el servidor flexible, pero el cliente se conecta con una conexión cifrada, se seguirá aceptando. La siguiente conexión con el cliente de MySQL a un servidor flexible configurado con require_secure_transport=OFF también funcionará como se muestra a continuación.

```bash
 mysql.exe -h mydemoserver.mysql.database.azure.com -u myadmin -p --ssl-mode=REQUIRED
```
```output
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 17
Server version: 5.7.29-log MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> show global variables like '%require_secure_transport%';
+--------------------------+-------+
| Variable_name            | Value |
+--------------------------+-------+
| require_secure_transport | OFF   |
+--------------------------+-------+
1 row in set (0.02 sec)
```

En resumen, la configuración de require_secure_transport=OFF reduce la obligatoriedad de conexiones cifradas en el servidor flexible y permite conexiones no cifradas al servidor desde el cliente además de conexiones cifradas.

## <a name="enforce-ssl-with-tls-version--12"></a>Exigir SSL con la versión de TLS < 1.2

Si la aplicación admite conexiones al servidor MySQL con SSL, pero acepta la versión de TLS > 1.2, necesitará establecer el parámetro de servidor de versiones de TLS en el servidor flexible. Para establecer las versiones de TLS que quiere que admita el servidor flexible, debe establecer el parámetro de servidor tls_version en TLSV1, TLSV 1.1 o TLSV1 y TLSV 1.1, como se muestra en la captura de pantalla, y guardar esta configuración para que surta efecto. tls_version es un **parámetro de servidor estático** que requerirá un reinicio del servidor para que surta efecto.

> :::image type="content" source="./media/how-to-connect-tls-ssl/tls-version.png" alt-text="Captura de pantalla que muestra cómo establecer la versión de TLS para un servidor flexible de Azure Database for MySQL.":::

## <a name="connect-using-mysql-command-line-client-with-tlsssl"></a>Conexión mediante el cliente de línea de comandos mysql con TLS/SSL

### <a name="download-the-public-ssl-certificate"></a>Descarga del certificado SSL público
Para usar conexiones cifradas con las aplicaciones cliente, deberá descargar el [certificado SSL público](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem), que también está disponible en la hoja Redes de Azure Portal, como se muestra en la captura de pantalla siguiente.

> :::image type="content" source="./media/how-to-connect-tls-ssl/download-ssl.png" alt-text="Captura de pantalla que muestra cómo descargar un certificado SSL público de Azure Portal.":::

Guarde el archivo de certificado en la ubicación que prefiera. Por ejemplo, en este tutorial se usa `c:\ssl` o `\var\www\html\bin` en el entorno local o en el entorno cliente donde se hospeda la aplicación. Esto permitirá que las aplicaciones se conecten de forma segura a la base de datos a través de SSL.

Si ha creado el servidor flexible con *acceso privado (integración con VNet)* , deberá conectarse a su servidor desde un recurso en misma red virtual que el servidor. Puede crear una máquina virtual y agregarla a la red virtual creada con el servidor flexible.

Si ha creado el servidor flexible con *acceso público (direcciones IP permitidas)* , puede agregar la dirección IP local a la lista de reglas de firewall del servidor.

Puede elegir [mysql.exe](https://dev.mysql.com/doc/refman/8.0/en/mysql.html) o [MySQL Workbench](./connect-workbench.md) para conectarse al servidor desde su entorno local. 

En el ejemplo siguiente se muestra cómo conectarse al servidor mediante la interfaz de la línea de comandos mysql. Use la configuración de la cadena de conexión `--ssl-mode=REQUIRED` para aplicar la comprobación del certificado TLS/SSL. Pase la ruta de acceso al archivo del certificado local al parámetro `--ssl-ca`. Reemplace los valores por el nombre real del servidor y la contraseña. 

```bash
sudo apt-get install mysql-client
wget --no-check-certificate https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem
mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p --ssl-mode=REQUIRED --ssl-ca=DigiCertGlobalRootCA.crt.pem
```
> [!Note]
> Confirme que el valor pasado a `--ssl-ca` coincide con la ruta de acceso al archivo del certificado que guardó.

Si intenta conectarse al servidor con conexiones no cifradas, verá un error parecido al siguiente, que indica que las conexiones que usan transporte no seguro están prohibidas:

```output
ERROR 3159 (HY000): Connections using insecure transport are prohibited while --require_secure_transport=ON.
```

## <a name="verify-the-tlsssl-connection"></a>Comprobación de la conexión TLS/SSL

Ejecute el comando **status** de MySQL para comprobar que se ha conectado al servidor de MySQL mediante TLS/SSL:

```dos
mysql> status
```
Confirme que la conexión está cifrada mediante la revisión de la salida, que debe mostrar:  **SSL: Cipher in use is** (SSL: el cifrado en uso es). Este conjunto de cifrado muestra un ejemplo; en función del cliente, puede ver otro conjunto de cifrado.

## <a name="connect-to-your-flexible-server-with-encrypted-connections-using-various-application-frameworks"></a>Conectarse al servidor flexible con conexiones cifradas mediante varios marcos de trabajo de aplicaciones

Las cadenas de conexión que están predefinidas en la página "Cadenas de conexión", disponible para el servidor en Azure Portal, incluyen los parámetros necesarios de lenguajes comunes para conectarse a su servidor de bases de datos mediante TLS/SSL. El parámetro TLS/SSL varía en función del conector. Por ejemplo, puede ser "useSSL=true", "sslmode=required" o "ssl_verify_cert=true", entre otras variables.

Para establecer una conexión cifrada a su servidor flexible a través de TLS/SSL desde la aplicación, consulte los siguientes ejemplos de código:

### <a name="wordpress"></a>WordPress
Descargue el [certificado SSL público](https://dl.cacerts.digicert.com/DigiCertGlobalRootCA.crt.pem) y agregue las siguientes líneas en wp-config.php después de la línea ```// ** MySQL settings - You can get this info from your web host ** //```.

```php
//** Connect with SSL** //
define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
//** SSL CERT **//
define('MYSQL_SSL_CERT','/FULLPATH/on-client/to/DigiCertGlobalRootCA.crt.pem');
```

### <a name="php"></a>PHP

```php
$conn = mysqli_init();
mysqli_ssl_set($conn,NULL,NULL, "/var/www/html/DigiCertGlobalRootCA.crt.pem", NULL, NULL);
mysqli_real_connect($conn, 'mydemoserver.mysql.database.azure.com', 'myadmin', 'yourpassword', 'quickstartdb', 3306, MYSQLI_CLIENT_SSL);
if (mysqli_connect_errno($conn)) {
die('Failed to connect to MySQL: '.mysqli_connect_error());
}
```

### <a name="php-using-pdo"></a>PHP (mediante PDO)

```phppdo
$options = array(
    PDO::MYSQL_ATTR_SSL_CA => '/var/www/html/DigiCertGlobalRootCA.crt.pem'
);
$db = new PDO('mysql:host=mydemoserver.mysql.database.azure.com;port=3306;dbname=databasename', 'myadmin', 'yourpassword', $options);
```

### <a name="python-mysqlconnector-python"></a>Python (MySQLConnector Python)

```python
try:
    conn = mysql.connector.connect(user='myadmin',
                                   password='yourpassword',
                                   database='quickstartdb',
                                   host='mydemoserver.mysql.database.azure.com',
                                   ssl_ca='/var/www/html/DigiCertGlobalRootCA.crt.pem')
except mysql.connector.Error as err:
    print(err)
```

### <a name="python-pymysql"></a>Python (PyMySQL)

```python
conn = pymysql.connect(user='myadmin',
                       password='yourpassword',
                       database='quickstartdb',
                       host='mydemoserver.mysql.database.azure.com',
                       ssl={'ca': '/var/www/html/DigiCertGlobalRootCA.crt.pem'})
```

### <a name="django-pymysql"></a>Django (PyMySQL)

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'quickstartdb',
        'USER': 'myadmin',
        'PASSWORD': 'yourpassword',
        'HOST': 'mydemoserver.mysql.database.azure.com',
        'PORT': '3306',
        'OPTIONS': {
            'ssl': {'ca': '/var/www/html/DigiCertGlobalRootCA.crt.pem'}
        }
    }
}
```

### <a name="ruby"></a>Ruby

```ruby
client = Mysql2::Client.new(
        :host     => 'mydemoserver.mysql.database.azure.com',
        :username => 'myadmin',
        :password => 'yourpassword',
        :database => 'quickstartdb',
        :sslca => '/var/www/html/DigiCertGlobalRootCA.crt.pem'
    )
```

### <a name="golang"></a>Golang

```go
rootCertPool := x509.NewCertPool()
pem, _ := ioutil.ReadFile("/var/www/html/DigiCertGlobalRootCA.crt.pem")
if ok := rootCertPool.AppendCertsFromPEM(pem); !ok {
    log.Fatal("Failed to append PEM.")
}
mysql.RegisterTLSConfig("custom", &tls.Config{RootCAs: rootCertPool})
var connectionString string
connectionString = fmt.Sprintf("%s:%s@tcp(%s:3306)/%s?allowNativePasswords=true&tls=custom",'myadmin' , 'yourpassword', 'mydemoserver.mysql.database.azure.com', 'quickstartdb')
db, _ := sql.Open("mysql", connectionString)
```

### <a name="java-mysql-connector-for-java"></a>Java (conector de MySQL para Java)

```java
# generate truststore and keystore in code
String importCert = " -import "+
    " -alias mysqlServerCACert "+
    " -file " + ssl_ca +
    " -keystore truststore "+
    " -trustcacerts " +
    " -storepass password -noprompt ";
String genKey = " -genkey -keyalg rsa " +
    " -alias mysqlClientCertificate -keystore keystore " +
    " -storepass password123 -keypass password " +
    " -dname CN=MS ";
sun.security.tools.keytool.Main.main(importCert.trim().split("\\s+"));
sun.security.tools.keytool.Main.main(genKey.trim().split("\\s+"));

# use the generated keystore and truststore
System.setProperty("javax.net.ssl.keyStore","path_to_keystore_file");
System.setProperty("javax.net.ssl.keyStorePassword","password");
System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file");
System.setProperty("javax.net.ssl.trustStorePassword","password");

url = String.format("jdbc:mysql://%s/%s?serverTimezone=UTC&useSSL=true", 'mydemoserver.mysql.database.azure.com', 'quickstartdb');
properties.setProperty("user", 'myadmin');
properties.setProperty("password", 'yourpassword');
conn = DriverManager.getConnection(url, properties);
```

### <a name="java-mariadb-connector-for-java"></a>Java (conector de MariaDB para Java)

```java
# generate truststore and keystore in code
String importCert = " -import "+
    " -alias mysqlServerCACert "+
    " -file " + ssl_ca +
    " -keystore truststore "+
    " -trustcacerts " +
    " -storepass password -noprompt ";
String genKey = " -genkey -keyalg rsa " +
    " -alias mysqlClientCertificate -keystore keystore " +
    " -storepass password123 -keypass password " +
    " -dname CN=MS ";
sun.security.tools.keytool.Main.main(importCert.trim().split("\\s+"));
sun.security.tools.keytool.Main.main(genKey.trim().split("\\s+"));

# use the generated keystore and truststore
System.setProperty("javax.net.ssl.keyStore","path_to_keystore_file");
System.setProperty("javax.net.ssl.keyStorePassword","password");
System.setProperty("javax.net.ssl.trustStore","path_to_truststore_file");
System.setProperty("javax.net.ssl.trustStorePassword","password");

url = String.format("jdbc:mariadb://%s/%s?useSSL=true&trustServerCertificate=true", 'mydemoserver.mysql.database.azure.com', 'quickstartdb');
properties.setProperty("user", 'myadmin');
properties.setProperty("password", 'yourpassword');
conn = DriverManager.getConnection(url, properties);
```

### <a name="net-mysqlconnector"></a>.NET (MySqlConnector)

```csharp
var builder = new MySqlConnectionStringBuilder
{
    Server = "mydemoserver.mysql.database.azure.com",
    UserID = "myadmin",
    Password = "yourpassword",
    Database = "quickstartdb",
    SslMode = MySqlSslMode.VerifyCA,
    SslCa = "DigiCertGlobalRootCA.crt.pem",
};
using (var connection = new MySqlConnection(builder.ConnectionString))
{
    connection.Open();
}
```

## <a name="next-steps"></a>Pasos siguientes
- [Use MySQL Workbench para conectarse y consultar datos en Azure Database for MySQL: servidor flexible](./connect-workbench.md).
- [Use PHP para conectarse y consultar datos en Azure Database for MySQL: servidor flexible](./connect-php.md).
- [Cree y administre redes virtuales de Azure Database for MySQL: servidor flexible mediante la CLI de Azure](./how-to-manage-virtual-network-cli.md).
- Obtenga más información sobre las [redes de Azure Database for MySQL: servidor flexible](./concepts-networking.md).
- Obtenga más información sobre las [reglas de firewall de Azure Database for MySQL: servidor flexible](./concepts-networking.md#public-access-allowed-ip-addresses).
