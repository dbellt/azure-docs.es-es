---
title: Lista de clasificaciones admitidas
description: En esta página se enumeran las clasificaciones del sistema admitidas en Azure Purview.
author: anmuk601
ms.author: anmuk
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: reference
ms.date: 4/1/2021
ms.openlocfilehash: e1d3d495d958465e966701aa7ce91bc2706b48e0
ms.sourcegitcommit: 3f684a803cd0ccd6f0fb1b87744644a45ace750d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/02/2021
ms.locfileid: "106219593"
---
# <a name="supported-classifications-in-azure-purview"></a>Clasificaciones admitidas en Azure Purview

En este artículo se enumeran las clasificaciones del sistema admitidas y definidas en Azure Purview (versión preliminar).


- **Umbral de coincidencia único**: es el número total de valores de datos únicos que deben encontrarse en una columna antes de que el escáner ejecute el patrón de datos en ella. Un umbral de coincidencia único no tiene nada que ver con la coincidencia de patrones, pero es un requisito previo de la coincidencia de patrones. Nuestras reglas de clasificación del sistema requieren que haya al menos 8 valores distintos en cada columna para someterlos a clasificación. Asimismo, el sistema necesita este valor para asegurarse de que la columna contiene suficientes datos para que el escáner pueda clasificarla con precisión. Por ejemplo, una columna que contenga varias filas con el valor 1 no se clasificará. Tampoco se clasificarán las columnas que contengan una fila con un valor y el resto de las filas con valores NULL. Recuerde que si especifica varios patrones, este valor se aplica a cada uno de ellos.

- **Umbral de coincidencia mínimo**: es el porcentaje mínimo de coincidencias de valores de datos que debe encontrar el escáner en una columna para que se aplique la clasificación. El valor de clasificación del sistema se establece en 60 %.


## <a name="defined-system-classifications"></a>Clasificaciones del sistema definidas

Azure Purview clasifica los datos por [expresiones regulares](https://wikipedia.org/wiki/Regular_expression) y [filtro de Bloom](https://wikipedia.org/wiki/Bloom_filter). En las siguientes listas se describen el formato, el patrón y las palabras clave de las clasificaciones del sistema que define Azure Purview.

Cada nombre de clasificación tiene el prefijo MICROSOFT.

## <a name="bloom-filter-classifications"></a>Clasificaciones del filtro de Bloom

## <a name="city-country-and-place"></a>Ciudad, País o región y Ubicación

Los filtros de Ciudad, País o región y Ubicación se han creado con los mejores conjuntos de datos disponibles para preparar los datos.

## <a name="person"></a>Person

El filtro de Bloom de tipo Person se ha preparado con los dos conjuntos de datos siguientes.

- [Datos del censo de 2010de EE. UU. para los apellidos (entradas 162-K)](https://www.census.gov/topics/population/genealogy/data/2010_surnames.html)
- [Nombres de bebé populares (de SSN), teniendo en cuenta todos los años de 1880 a 2019 (entradas 98-K)](https://www.ssa.gov/oact/babynames/limits.html)

## <a name="regex-classifications"></a>Clasificaciones RegEx

## <a name="aba-routing"></a>Enrutamiento ABA

### <a name="format"></a>Formato

Nueve dígitos, que pueden estar en un patrón con formato o sin formato.

### <a name="pattern"></a>Patrón

Con formato:

- cuatro dígitos empezando con 0, 1, 2, 3, 6, 7 u 8
- un guion
- cuatro dígitos
- un guion
- un dígito

Sin formato: nueve dígitos consecutivos que empiezan por 0, 1, 2, 3, 6, 7 u 8

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_aba_routing"></a>Keyword_aba_routing

```
amba number
aba #
aba
abarouting#
abaroutingnumber
americanbankassociationrouting#
americanbankassociationroutingnumber
bank routing#
bankroutingnumber
routing #
routing no
routing number
routing transit number
routing#
RTN
```

## <a name="argentina-national-identity-dni-number"></a>Número de identidad nacional de Argentina (DNI)

### <a name="format"></a>Formato

Ocho dígitos con o sin puntos.

### <a name="pattern"></a>Patrón

Ocho dígitos:

- dos dígitos
- un punto opcional
- tres dígitos
- un punto opcional
- tres dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_argentina_national_id"></a>Keyword_argentina_national_id

```
Argentina National Identity number
cedula
cédula
dni
documento nacional de identidad
documento número
documento numero
registro nacional de las personas
rnp
```

## <a name="australia-bank-account-number"></a>Número de cuenta bancaria de Australia

### <a name="format"></a>Formato

De seis a 10 dígitos con o sin un número sucursal bancaria.

### <a name="pattern"></a>Patrón

El número de cuenta tiene entre seis y 10 dígitos.

Número de sucursal bancaria de Australia:

- tres dígitos
- un guion
- tres dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_australia_bank_account_number"></a>Keyword_australia_bank_account_number

```
swift bank code
correspondent bank
base currency
usa account
holder address
bank address
information account
fund transfers
bank charges
bank details
banking information
full names
iaea
```

## <a name="australia-drivers-license-number"></a>Número de permiso de conducir de Australia

### <a name="format"></a>Formato
Nueve letras y dígitos

### <a name="pattern"></a>Patrón
nueve letras y dígitos:

- dos dígitos o letras (no distingue entre mayúsculas y minúsculas)
- dos dígitos
- cinco dígitos o letras (no distingue entre mayúsculas y minúsculas)

O BIEN

- una o dos letras opcionales (no distingue entre mayúsculas y minúsculas)
- de cuatro a nueve dígitos

O BIEN

- nueve dígitos o letras (no distingue entre mayúsculas y minúsculas)

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_australia_drivers_license_number"></a>Keyword_australia_drivers_license_number

```
international driving permits
australian automobile association
international driving permit
DriverLicence
DriverLicences
Driver Lic
Driver License
Driver Licenses
DriversLic
DriversLicence
DriversLicences
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicence
Driver'sLicences
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
DriverLic#
DriverLics#
DriverLicence#
DriverLicences#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicence#
DriversLicences#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicence#
Driver'sLicences#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
```

#### <a name="keyword_australia_drivers_license_number_exclusions"></a>Keyword_australia_drivers_license_number_exclusions

```
aaa
DriverLicense
DriverLicenses
Driver License
Driver Licenses
DriversLicense
DriversLicenses
Drivers License
Drivers Licenses
Driver'License
Driver'Licenses
Driver' License
Driver' Licenses
Driver'sLicense
Driver'sLicenses
Driver's License
Driver's Licenses
DriverLicense#
DriverLicenses#
Driver License#
Driver Licenses#
DriversLicense#
DriversLicenses#
Drivers License#
Drivers Licenses#
Driver'License#
Driver'Licenses#
Driver' License#
Driver' Licenses#
Driver'sLicense#
Driver'sLicenses#
Driver's License#
Driver's Licenses#
```

## <a name="australian-medicare-number"></a>Número de Medicare en Australia

### <a name="format"></a>Formato

10-11 dígitos

### <a name="pattern"></a>Patrón

10-11 dígitos:

- el primer dígito está en el intervalo 2-6
- el noveno dígito es un dígito de control
- el décimo dígito es el dígito de emisión
- el undécimo dígito (opcional) es el número individual

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_australia_medicare_number"></a>Keyword_Australia_Medicare_Number

```
bank account details
medicare payments
mortgage account
bank payments
information branch
credit card loan
department of human services
local service
medicare
```

## <a name="australia-passport-number"></a>Número de pasaporte de Australia

### <a name="format"></a>Formato

Una letra seguida de siete dígitos.

### <a name="pattern"></a>Patrón

Una letra (no distingue entre mayúsculas y minúsculas) seguida de siete dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

#### <a name="keyword_australia_passport_number"></a>Keyword\_australia\_passport\_number

```
passport
passport details
immigration and citizenship
commonwealth of australia
department of immigration
residential address
department of immigration and citizenship
visa
national identity card
passport number
travel document
issuing authority
```

## <a name="australia-tax-file-number"></a>Número de expediente fiscal de Australia

### <a name="format"></a>Formato

de ocho a nueve dígitos

### <a name="pattern"></a>Patrón

entre ocho y nueve dígitos que normalmente se presentan con espacios tal como se indica a continuación:

- tres dígitos
- un espacio opcional
- tres dígitos
- un espacio opcional
- de dos a tres dígitos en los que el último dígito es un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_australia_tax_file_number"></a>Keyword\_australia\_tax\_file\_number

```
australian business number
marginal tax rate
medicare levy
portfolio number
service veterans
withholding tax
individual tax return
tax file number
tfn
```

## <a name="belgium-national-number"></a>Número nacional de Bélgica

### <a name="format"></a>Formato

11 dígitos más delimitadores opcionales

### <a name="pattern"></a>Patrón

11 dígitos más delimitadores:

- seis dígitos y dos puntos opcionales con el formato AA.MM.DD para la fecha de nacimiento
- Un delimitador opcional de punto, guion o espacio
- tres dígitos secuenciales (impares para masculino, pares para femenino)
- Un delimitador opcional de punto, guion o espacio
- dos dígitos de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_belgium_national_number"></a>Keyword\_belgium\_national\_number

```
be lasting aantal
bnn#
bnn
carte d'identité
identifiant national
identifiantnational#
identificatie
identification
identifikation
identifikationsnummer
identifizierung
identité
identiteit
identiteitskaart
identity
inscription
national number
national register
national number#
national number
nif#
nif
numéro d'assuré
numéro de registre national
numéro de sécurité
numéro d'identification
numéro d'immatriculation
numéro national
numéronational#
personal ID number
personalausweis
personalidnumber#
registratie
registration
registrationsnumme
registrierung
social security number
ssn#
ssn
steuernummer
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="brazil-cpf-number"></a>Número de CPF de Brasil

### <a name="format"></a>Formato

11 dígitos que incluyen un dígito de control y pueden tener formato o no.

### <a name="pattern"></a>Patrón

Con formato:

- tres dígitos
- un punto
- tres dígitos
- un punto
- tres dígitos
- un guion
- dos dígitos, que son dígitos de control

Sin formato:

- 11 dígitos en los que los dos últimos dígitos son dígitos de control.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_brazil_cpf"></a>Keyword\_brazil\_cpf

```
CPF
Identification
Registration
Revenue
Cadastro de Pessoas Físicas
Imposto
Identificação
Inscrição
Receita
```

## <a name="brazil-legal-entity-number-cnpj"></a>Número de entidad jurídica de Brasil (CNPJ)

### <a name="format"></a>Formato

14 dígitos que incluyen un número de registro, un número de sucursal y los dígitos de control, más los delimitadores.

### <a name="pattern"></a>Patrón

14 dígitos más delimitadores:

- dos dígitos
- un punto
- tres dígitos
- un punto
- tres dígitos (estos primeros ocho dígitos son el número de registro)
- una barra diagonal
- número de sucursal de cuatro dígitos
- un guion
- dos dígitos, que son dígitos de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_brazil_cnpj"></a>Keyword\_brazil\_cnpj

```
CNPJ
CNPJ/MF
CNPJ-MF
National Registry of Legal Entities
Taxpayers Registry
Legal entity
Legal entities
Registration Status
Business
Company
CNPJ
Cadastro Nacional da Pessoa Jurídica
Cadastro Geral de Contribuintes
CGC
Pessoa jurídica
Pessoas jurídicas
Situação cadastral
Inscrição
Empresa
```

## <a name="brazil-national-identification-card-rg"></a>Número de identificación nacional de Brasil (RG)

### <a name="format"></a>Formato

:::no-loc text="Registro Geral (old format): Nine digits":::

:::no-loc text="Registro de Identidade (RIC) (new format): 11 digits":::

### <a name="pattern"></a>Patrón

:::no-loc text="Registro Geral (old format):":::

- dos dígitos
- un punto
- tres dígitos
- un punto
- tres dígitos
- un guion
- un dígito, que es un dígito de control

:::no-loc text="Registro de Identidade (RIC) (new format):":::

- 10 dígitos
- un guion
- un dígito, que es un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_brazil_rg"></a>Keyword\_brazil\_rg

```
Cédula de identidade
identity card
national ID
número de rregistro
registro de Iidentidade
registro geral
RG (this keyword is case-sensitive)
RIC (this keyword is case-sensitive)
```

## <a name="canada-bank-account-number"></a>Número de cuenta bancaria de Canadá

### <a name="format"></a>Formato

7 o 12 dígitos

### <a name="pattern"></a>Patrón

Un número de cuenta bancaria de Canadá tiene entre 7 o 12 dígitos.

Un número de tránsito de cuenta bancaria de Canadá tiene:

- cinco dígitos
- un guion
- tres dígitos O
- un cero &quot;0&quot;
- ocho dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_canada_bank_account_number"></a>Keyword\_canada\_bank\_account\_number

```
canada savings bonds
canada revenue agency
canadian financial institution
direct deposit form
canadian citizen
legal representative
notary public
commissioner for oaths
child care benefit
universal child care
canada child tax benefit
income tax benefit
harmonized sales tax
social insurance number
income tax refund
child tax benefit
territorial payments
institution number
deposit request
banking information
direct deposit
```

## <a name="canada-drivers-license-number"></a>Número de permiso de conducir de Canadá

### <a name="format"></a>Formato

Varía según la provincia.

### <a name="pattern"></a>Patrón

Varios patrones que abarcan Alberta, Columbia Británica, Manitoba, Nuevo Brunswick, Terranova/Labrador, Nueva Escocia, Ontario, Isla del Príncipe Eduardo, Quebec y Saskatchewan.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_province_name_drivers_license_name"></a>Keyword\_[province\_name]\_drivers\_license\_name

- La abreviatura de la provincia; por ejemplo, AB.
- El nombre de la provincia, por ejemplo, Alberta.

#### <a name="keyword_canada_drivers_license"></a>Keyword\_canada\_drivers\_license

```
DL
DLS
CDL
CDLS
DriverLic
DriverLics
DriverLicense
DriverLicenses
DriverLicence
DriverLicences
Driver Lic
Driver Lics
Driver License
Driver Licenses
Driver License
Driver Licenses
DriversLic
DriversLics
DriversLicence
DriversLicences
DriversLicense
DriversLicenses
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicense
Driver'sLicenses
Driver'sLicence
Driver'sLicences
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
Driver's License
Driver's Licenses
Permis de Conduire
ID
IDs
ID card number
ID card numbers
ID card #
ID card #s
ID card
ID cards
ID card
identification number
identification numbers
identification #
identification #s
identification card
identification cards
identification
DL#
DLS#
CDL#
CDLS#
DriverLic#
DriverLics#
DriverLicense#
DriverLicenses#
DriverLicence#
DriverLicences#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicense#
DriversLicenses#
DriversLicence#
DriversLicences#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicense#
Driver'sLicenses#
Driver'sLicence#
Driver'sLicences#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
Driver's License#
Driver's Licenses#
Permis de Conduire#
ID#
IDs#
ID card#
ID cards#
ID card#
identification card#
identification cards#
identification#
```

## <a name="canada-health-service-number"></a>Número del servicio de salud de Canadá

### <a name="format"></a>Formato

10 dígitos

### <a name="pattern"></a>Patrón

10 dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_canada_health_service_number"></a>Keyword\_canada\_health\_service\_number

```
personal health number
patient information
health services
specialty services
automobile accident
patient hospital
psychiatrist
workers compensation
disability
```

## <a name="canada-passport-number"></a>Número de pasaporte de Canadá

### <a name="format"></a>Formato

dos letras mayúsculas seguidas de seis dígitos

### <a name="pattern"></a>Patrón

dos letras mayúsculas seguidas de seis dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_canada_passport_number"></a>Keyword\_canada\_passport\_number

```
canadian citizenship
canadian passport
passport application
passport photos
certified translator
canadian citizens
processing times
renewal application
```

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="canada-personal-health-identification-number-phin"></a>Número de identificación sanitaria personal de Canadá (PHIN)

### <a name="format"></a>Formato

nueve dígitos

### <a name="pattern"></a>Patrón

nueve dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_canada_phin"></a>Keyword\_canada\_phin

```
social insurance number
health information act
income tax information
manitoba health
health registration
prescription purchases
benefit eligibility
personal health
power of attorney
registration number
personal health number
practitioner referral
wellness professional
patient referral
health and wellness
```

#### <a name="keyword_canada_provinces"></a>Keyword\_canada\_provinces

```
Nunavut
Quebec
Northwest Territories
Ontario
British Columbia
Alberta
Saskatchewan
Manitoba
Yukon
Newfoundland and Labrador
New Brunswick
Nova Scotia
Prince Edward Island
Canada
```

## <a name="canada-social-insurance-number"></a>Número de la seguridad social de Canadá

### <a name="format"></a>Formato

nueve dígitos con guiones o espacios opcionales

### <a name="pattern"></a>Patrón

Con formato:

- tres dígitos
- un guion o un espacio
- tres dígitos
- un guion o un espacio
- tres dígitos

Sin formato: nueve dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_sin"></a>Keyword\_sin

```
sin
social insurance
numero d'assurance sociale
sins
ssn
ssns
social security
numero d'assurance social
national identification number
national ID
sin#
soc ins
social ins
```

#### <a name="keyword_sin_collaborative"></a>Keyword\_sin\_collaborative

```
driver's license
drivers license
driver's license
drivers license
DOB
Birthdate
Birthday
Date of Birth
```

## <a name="chile-identity-card-number"></a>Número del carné de identidad de Chile

### <a name="format"></a>Formato

de siete a ocho dígitos más delimitadores de un dígito o letra de control.

### <a name="pattern"></a>Patrón

de siete a ocho dígitos más delimitadores:

- uno o dos dígitos
- un punto opcional
- tres dígitos
- un punto opcional
- tres dígitos
- un guión
- un dígito o letra (no distingue entre mayúsculas y minúsculas), que es un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_chile_id_card"></a>Keyword\_chile\_id\_card

```
cédula de identidad
identificación
national identification
national identification number
national ID
número de identificación nacional
rol único nacional
rol único tributario
RUN
RUT
tarjeta de identificación
Rol Unico Nacional
Rol Unico Tributario
RUN#
RUT#
nationaluniqueroleID#
nacional identidad
número identificación
identidad número
numero identificacion
identidad numero
Chilean identity no.
Chilean identity number
Chilean identity #
Unique Tax Registry
Unique Tributary Role
Unique Tax Role
Unique Tributary Number
Unique National Number
Unique National Role
National unique role
Chile identity no.
Chile identity number
Chile identity #
```

## <a name="china-resident-identity-card-prc-number"></a>Número del carné de identidad de residente de China (PRC)

### <a name="format"></a>Formato

18 dígitos

### <a name="pattern"></a>Patrón

18 dígitos:

- seis dígitos, que son un código de dirección
- ocho dígitos con el formato AAAAMMDD, que son la fecha de nacimiento
- tres dígitos, que son un código de orden
- un dígito, que es un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_china_resident_id"></a>Keyword\_china\_resident\_id

```
Resident Identity Card
PRC
National Identification Card
身份证
居民身份证
居民身份证
鉴定
身分證
居民身份證
鑑定
```

## <a name="credit-card-number"></a>Número de tarjeta de crédito

### <a name="format"></a>Formato

de 14 a 16 dígitos que pueden tener formato o no (dddddddddddddddd) y que deben pasar la prueba Luhn.

### <a name="pattern"></a>Patrón

Patrón complejo y sólido que detecta tarjetas de todas las marcas principales en todo el mundo, como Visa, MasterCard, Discover Card, JCB, American Express, tarjetas regalo y tarjetas Diner.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_cc_verification"></a>Keyword\_cc\_verification

```
card verification
card identification number
cvn
cid
cvc2
cvv2
pin block
security code
security number
security no
issue number
issue no
cryptogramme
numéro de sécurité
numero de securite
kreditkartenprüfnummer
kreditkartenprufnummer
prüfziffer
prufziffer
sicherheits Kode
sicherheitscode
sicherheitsnummer
verfalldatum
codice di verifica
cod. sicurezza
cod sicurezza
n autorizzazione
código
codigo
cod. seg
cod seg
código de segurança
codigo de seguranca
codigo de segurança
código de seguranca
cód. segurança
cod. seguranca
cod. segurança
cód. seguranca
cód segurança
cod seguranca
cod segurança
cód seguranca
número de verificação
numero de verificacao
ablauf
gültig bis
gültigkeitsdatum
gultig bis
gultigkeitsdatum
scadenza
data scad
fecha de expiracion
fecha de venc
vencimiento
válido hasta
valido hasta
vto
data de expiração
data de expiracao
data em que expira
validade
valor
vencimento
transaction
transaction number
reference number
セキュリティコード
セキュリティ コード
セキュリティナンバー
セキュリティ ナンバー
セキュリティ番号
```

#### <a name="keyword_cc_name"></a>Keyword\_cc\_name

```
amex
Americans express
Americans express
americano espresso
Visa
Mastercard
master card
mc
master cards
master cards
diner's Club
diners club
diners club
discover
discover card
discover card
discover cards
JCB
BrandSmart
japanese card bureau
carte blanche
carteblanche
credit card
cc#
cc#:
expiration date
exp date
expiry date
date d'expiration
date d'exp
date expiration
bank card
bank card
card number
card num
card number
card numbers
card numbers
credit card
credit cards
credit cards
ccn
card holder
cardholder
card holders
cardholders
check card
check card
check cards
check cards
debit card
debit card
debit cards
debit cards
atm card
atmcard
atm cards
atmcards
enroute
en route
card type
Cardmember Acct
cardmember account
Card no
Corporate Card
Corporate cards
Type of card
card account number
card member account
Cardmember Acct.
card no.
card no
card number
carte bancaire
carte de crédit
carte de credit
numéro de carte
numero de carte
nº de la carte
nº de carte
kreditkarte
karte
karteninhaber
karteninhabers
kreditkarteninhaber
kreditkarteninstitut
kreditkartentyp
eigentümername
kartennr
kartennummer
kreditkartennummer
kreditkarten-nummer
carta di credito
carta credito
n. carta
n carta
nr. carta
nr carta
numero carta
numero della carta
numero di carta
tarjeta credito
tarjeta de credito
tarjeta crédito
tarjeta de crédito
tarjeta de atm
tarjeta atm
tarjeta debito
tarjeta de debito
tarjeta débito
tarjeta de débito
nº de tarjeta
no. de tarjeta
no de tarjeta
numero de tarjeta
número de tarjeta
tarjeta no
tarjetahabiente
cartão de crédito
cartão de credito
cartao de crédito
cartao de credito
cartão de débito
cartao de débito
cartão de debito
cartao de debito
débito automático
debito automatico
número do cartão
numero do cartão
número do cartao
numero do cartao
número de cartão
numero de cartão
número de cartao
numero de cartao
nº do cartão
nº do cartao
nº. do cartão
no do cartão
no do cartao
no. do cartão
no. do cartao
クレジットカード番号
クレジットカードナンバー
クレジットカード＃
クレジットカード
クレジット
クレカ
カード番号
カードナンバー
カード＃
アメックス
アメリカンエクスプレス
アメリカン エクスプレス
Visaカード
Visa カード
マスターカード
マスター カード
マスター
ダイナースクラブ
ダイナース クラブ
ダイナース
有効期限
期限
キャッシュカード
キャッシュ カード
カード名義人
カードの名義人
カードの名義
デビット カード
デビットカード
```

## <a name="croatia-drivers-license-number"></a>Número de permiso de conducir de Croacia

Esta entidad de tipo de información confidencial solo está disponible en el tipo de información confidencial del número de permiso de conducir de la UE.

### <a name="format"></a>Formato

ocho dígitos sin espacios ni delimitadores

### <a name="pattern"></a>Patrón

ocho dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_eu_drivers_license_number"></a>Keywords\_eu\_driver's\_license\_number

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
```

#### <a name="keywords_croatia_eu_drivers_license_number"></a>Keywords\_croatia\_eu\_driver's\_license\_number

```
vozačka dozvola
vozačke dozvole
```

## <a name="croatia-identity-card-number"></a>Número del carné de identidad de Croacia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de identificación nacional de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

nueve dígitos

### <a name="pattern"></a>Patrón

nueve dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_croatia_id_card"></a>Keyword\_croatia\_id\_card

```
majstorski broj građana
master citizen number
nacionalni identifikacijski broj
national identification number
oib#
oib
osobna iskaznica
osobni ID
osobni identifikacijski broj
personal identification number
porezni broj
porezni identifikacijski broj
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="croatia-personal-identification-oib-number"></a>Número de identificación personal de Croacia (OIB)

### <a name="format"></a>Formato

11 dígitos

### <a name="pattern"></a>Patrón

11 dígitos:

- 10 dígitos
- el dígito final es un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_croatia_oib_number"></a>Keyword\_croatia\_oib\_number

```
majstorski broj građana
master citizen number
nacionalni identifikacijski broj
national identification number
oib#
oib
osobna iskaznica
osobni ID
osobni identifikacijski broj
personal identification number
porezni broj
porezni identifikacijski broj
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax Id number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="denmark-personal-identification-number"></a>Número de identificación personal de Dinamarca

### <a name="format"></a>Formato

10 dígitos que contienen un guion

### <a name="pattern"></a>Patrón

10 dígitos:

- seis dígitos con el formato DDMMAA, que son la fecha de nacimiento
- un guion
- cuatro dígitos en los que el último dígito es un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_denmark_id"></a>Keyword\_denmark\_id

```
centrale person register
civilt registreringssystem
cpr
cpr#
gesundheitskarte nummer
gesundheitsversicherungkarte nummer
health card
health insurance card number
health insurance number
identification number
identifikationsnummer
identifikationsnummer#
identity number
krankenkassennummer
national ID#
national number#
national number
personalidnumber#
personalidentityno#
personal ID number
personnummer
personnummer#
reisekrankenversicherungskartenummer
rejsesygesikringskort
ssn
ssn#
skat ID
skat kode
skat nummer
skattenummer
social security number
sundhedsforsikringskort
sundhedsforsikringsnummer
sundhedskort
sundhedskortnummer
sygesikring
sygesikringkortnummer
tax code
travel health insurance card
uniqueidentityno#
tax number
tax registration number
tax ID
tax identification number
tax ID#
tax number#
tax no
tax no#
tax number
tax identification no
tin#
tax ID no#
tax ID number#
tax no#
tin ID
tin no
cpr.nr
cprnr
cprnummer
personnr
person register
sygesikringsbevis
sygesikringsbevisnr
sygesikringsbevisnummer
sygesikringskort
sygesikringskortnr
sygesikringskortnummer
sygesikringsnr
sygesikringsnummer
```

## <a name="eu-debit-card-number"></a>Número de tarjeta de débito de la UE

### <a name="format"></a>Formato

16 dígitos

### <a name="pattern"></a>Patrón

Patrón complejo y sólido.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_eu_debit_card"></a>Keyword\_eu\_debit\_card

```
account number
card number
card no.
security number
cc#
```

#### <a name="keyword_card_terms_dict"></a>Keyword\_card\_terms\_dict

```
acct nbr
acct num
acct no
Americans express
Americans express
americano espresso
amex
atm card
atm cards
atm kaart
atmcard
atmcards
atmkaart
atmkaarten
ban contact
bank card
bankkaart
card holder
card holders
card num
card number
card numbers
card type
cardano numerico
cardholder
cardholders
card number
card numbers
carta bianca
carta credito
carta di credito
cartao de credito
cartao de crédito
cartao de debito
cartao de débito
carte bancaire
carte blanche
carte bleue
carte de credit
carte de crédit
carte di credito
carteblanche
cartão de credito
cartão de crédito
cartão de debito
cartão de débito
cb
ccn
check card
check cards
check card
check cards
chequekaart
cirrus
cirrus-edc-maestro
controlekaart
controlekaarten
credit card
credit cards
credit card
credit cards
debetkaart
debetkaarten
debit card
debit cards
debit card
debit cards
debito automatico
diners club
diners club
discover
discover card
discover cards
discover card
discover cards
débito automático
edc
eigentümername
european debit card
hoofdkaart
hoofdkaarten
in viaggio
japanese card bureau
japanse kaartdienst
jcb
kaart
kaart num
kaartaantal
kaartaantallen
kaarthouder
kaarthouders
karte
karteninhaber
karteninhabers
kartennr
kartennummer
kreditkarte
kreditkarten-nummer
kreditkarteninhaber
kreditkarteninstitut
kreditkartennummer
kreditkartentyp
maestro
master card
master cards
Mastercard
master cards
mc
mister cash
n carta
carta
no de tarjeta
no do cartao
no do cartão
no. de tarjeta
no. do cartao
no. do cartão
nr carta
nr. carta
numeri di scheda
numero carta
numero de cartao
numero de carte
numero de cartão
numero de tarjeta
numero della carta
numero di carta
numero di scheda
numero do cartao
numero do cartão
numéro de carte
nº carta
nº de carte
nº de la carte
nº de tarjeta
nº do cartao
nº do cartão
nº. do cartão
número de cartao
número de cartão
número de tarjeta
número do cartao
scheda dell'assegno
scheda dell'atmosfera
scheda dell'atmosfera
scheda della banca
scheda di controllo
scheda di debito
scheda matrice
schede dell'atmosfera
schede di controllo
schede di debito
schede matrici
scoprono la scheda
scoprono le schede
solo
supporti di scheda
supporto di scheda
switch
tarjeta atm
tarjeta credito
tarjeta de atm
tarjeta de credito
tarjeta de debito
tarjeta debito
tarjeta no
tarjetahabiente
tipo della scheda
ufficio giapponese della
scheda
v pay
v-pay
visa
visa plus
visa electron
visto
visum
vpay
```

#### <a name="keyword_card_security_terms_dict"></a>Keyword\_card\_security\_terms\_dict

```
card identification number
card verification
cardi la verifica
cid
cod seg
cod seguranca
cod segurança
cod sicurezza
cod. seg
cod. seguranca
cod. segurança
cod. sicurezza
codice di sicurezza
codice di verifica
codigo
codigo de seguranca
codigo de segurança
crittogramma
cryptogram
cryptogramme
cv2
cvc
cvc2
cvn
cvv
cvv2
cód seguranca
cód segurança
cód. seguranca
cód. segurança
código
código de seguranca
código de segurança
de kaart controle
geeft nr uit
issue no
issue number
kaartidentificatienummer
kreditkartenprufnummer
kreditkartenprüfnummer
kwestieaantal
no. dell'edizione
no. di sicurezza
numero de securite
numero de verificacao
numero dell'edizione
numero di identificazione della
scheda
numero di sicurezza
numero van veiligheid
numéro de sécurité
nº autorizzazione
número de verificação
perno il blocco
pin block
prufziffer
prüfziffer
security code
security no
security number
sicherheits kode
sicherheitscode
sicherheitsnummer
speldblok
veiligheid nr
veiligheidsaantal
veiligheidscode
veiligheidsnummer
verfalldatum
```

#### <a name="keyword_card_expiration_terms_dict"></a>Keyword\_card\_expiration\_terms\_dict

```
ablauf
data de expiracao
data de expiração
data del exp
data di exp
data di scadenza
data em que expira
data scad
data scadenza
date de validité
datum afloop
datum van exp
de afloop
espira
espira
exp date
exp datum
expiration
expire
expires
expiry
fecha de expiracion
fecha de venc
gultig bis
gultigkeitsdatum
gültig bis
gültigkeitsdatum
la scadenza
scadenza
valable
validade
valido hasta
valor
venc
vencimento
vencimiento
verloopt
vervaldag
vervaldatum
vto
válido hasta
```

## <a name="eu-drivers-license-number"></a>Número de permiso de conducir de la UE

Estas son las entidades del tipo de información confidencial del Número de permiso de conducir de la UE.

- Austria
- Bélgica
- Bulgaria
- Croacia
- Chipre
- Checo
- Dinamarca
- Estonia
- Finlandia
- Francia
- Alemania
- Grecia
- Hungría
- Irlanda
- Italia
- Letonia
- Lituania
- Luxemburgo
- Malta
- Países Bajos
- Polonia
- Portugal
- Rumanía
- Eslovaquia
- Eslovenia
- España
- Suecia
- Reino Unido

## <a name="eu-national-identification-number"></a>Número de identificación nacional de la UE

Estas son las entidades del tipo de información confidencial del Número de identificación nacional de la UE.

- Austria
- Bélgica
- Bulgaria
- Croacia
- Chipre
- Checo
- Dinamarca
- Estonia
- Finlandia
- Francia
- Alemania
- Grecia
- Hungría
- Irlanda
- Italia
- Letonia
- Lituania
- Luxemburgo
- Malta
- Países Bajos
- Polonia
- Portugal
- Rumanía
- Eslovaquia
- Eslovenia
- España
- Reino Unido

## <a name="eu-passport-number"></a>Número de pasaporte de la UE

Estas son las entidades del tipo de información confidencial de los Números de pasaporte de la UE, y las entidades del conjunto de números de pasaporte de la UE.

- Austria
- Bélgica
- Bulgaria
- Croacia
- Chipre
- Checo
- Dinamarca
- Estonia
- Finlandia
- Francia
- Alemania
- Grecia
- Hungría
- Irlanda
- Italia
- Letonia
- Lituania
- Luxemburgo
- Malta
- Países Bajos
- Polonia
- Portugal
- Rumanía
- Eslovaquia
- Eslovenia
- España
- Suecia
- Reino Unido

## <a name="eu-social-security-number-or-equivalent-identification"></a>Número del seguro social (SSN) o identificación equivalente de la UE

Estas son las entidades que se encuentran en el tipo de información confidencial del Número del seguro social de la UE o de una identificación equivalente.

- Austria
- Bélgica
- Croacia
- Checo
- Dinamarca
- Finlandia
- Francia
- Alemania
- Grecia
- Hungría
- Portugal
- España
- Suecia

## <a name="eu-tax-identification-number"></a>Número de identificación fiscal de la UE

Estas son las entidades del tipo de información confidencial del Número de identificación fiscal de la UE.

- Austria
- Bélgica
- Bulgaria
- Croacia
- Chipre
- Checo
- Dinamarca
- Estonia
- Finlandia
- Francia
- Alemania
- Grecia
- Hungría
- Irlanda
- Italia
- Letonia
- Lituania
- Luxemburgo
- Malta
- Países Bajos
- Polonia
- Portugal
- Rumanía
- Eslovaquia
- Eslovenia
- España
- Suecia
- Reino Unido



## <a name="finland-national-id"></a>Documento nacional de identidad de Finlandia

### <a name="format"></a>Formato

seis dígitos más un carácter que indica un siglo, más tres dígitos, más un dígito de control

### <a name="pattern"></a>Patrón

El patrón debe incluir lo siguiente:

- seis dígitos con el formato DDMMAA, que son la fecha de nacimiento
- marcador de siglo ("-", "+" o "a")
- número de identificación personal de tres dígitos
- un dígito o letra (no distingue entre mayúsculas y minúsculas), que es un dígito de control

### <a name="keywords"></a>Palabras clave

```
ainutlaatuinen henkilökohtainen tunnus
henkilökohtainen tunnus
henkilötunnus
henkilötunnusnumero#
henkilötunnusnumero
hetu
ID no
ID number
identification number
identiteetti numero
identity number
ID number
kansallinen henkilötunnus
kansallisen henkilökortin
national ID card
national ID no.
personal ID
personal identity code
personalidnumber#
personbeteckning
personnummer
social security number
sosiaaliturvatunnus
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
tunnistenumero
tunnus numero
tunnusluku
tunnusnumero
verokortti
veronumero
verotunniste
verotunnus
```

## <a name="finland-passport-number"></a>Número de pasaporte de Finlandia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de pasaporte de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

combinación de nueve letras y dígitos.

### <a name="pattern"></a>Patrón

combinación de nueve letras y dígitos:

- dos letras (no se distingue entre mayúsculas y minúsculas)
- siete dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_eu_passport_number_common"></a>Keywords\_eu\_passport\_number\_common

```
passport#
passport #
passport ID
passports
passport no
passport no
passport number
passport number
passport numbers
passport numbers
```

#### <a name="keyword_finland_passport_number"></a>Keyword\_finland\_passport\_number

```
suomalainen passi
passin numero
passin numero.#
passin numero#
passin numero.
passi#
passi number
```

## <a name="france-drivers-license-number"></a>Número de permiso de conducir de Francia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de permiso de conducir de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

12 dígitos

### <a name="pattern"></a>Patrón

12 dígitos con validación para descontar patrones similares, como números de teléfono franceses.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_french_drivers_license"></a>Keyword\_french\_drivers\_license

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
permis de conduire
license number
license number
license numbers
license numbers
numéros de license
```

## <a name="france-national-id-card-cni"></a>Carné de identidad nacional de Francia (CNI)

### <a name="format"></a>Formato

12 dígitos

### <a name="pattern"></a>Patrón

12 dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_france_eu_national_id_card"></a>Keywords\_france\_eu\_national\_id\_card

```
card number
carte nationale d'identité
carte nationale d'idenite no
cni#
cni
compte bancaire
national identification number
national identity
nationalidno#
numéro d'assurance maladie
numéro de carte vitale
```

## <a name="france-passport-number"></a>Número de pasaporte de Francia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de pasaporte de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

nueve dígitos y letras

### <a name="pattern"></a>Patrón

nueve dígitos y letras:

- dos dígitos
- dos letras (no se distingue entre mayúsculas y minúsculas)
- cinco dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="france-social-security-number-insee-or-equivalent-identification"></a>Número del seguro social (INSEE) o identificación equivalente de Francia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número del seguro social de la UE y el id. equivalente, y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

15 dígitos

### <a name="pattern"></a>Patrón

Debe coincidir con uno de dos patrones:

- 13 dígitos seguidos de un espacio, seguido de dos dígitos, o
- 15 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_fr_insee"></a>Keyword\_fr\_insee

```
insee
securité sociale
securite sociale
national ID
national identification
numéro d'identité
no d'identité
no. d'identité
numero d'identite
no d'identite
no. d'identite
social security number
social security code
social insurance number
le numéro d'identification nationale
d'identité nationale
numéro de sécurité sociale
le code de la sécurité sociale
numéro d'assurance sociale
numéro de sécu
code sécu
```

## <a name="germany-drivers-license-number"></a>Número de permiso de conducir de Alemania

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de permiso de conducir de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

combinación de 11 dígitos y letras

### <a name="pattern"></a>Patrón

11 dígitos y letras (no se distingue entre mayúsculas y minúsculas):

- un dígito o una letra
- dos dígitos
- seis dígitos o letras
- un dígito
- un dígito o una letra

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_german_drivers_license_number"></a>Keyword\_german\_drivers\_license\_number

```
ausstellungsdatum
ausstellungsort
ausstellende behöde
ausstellende behorde
ausstellende behoerde
führerschein
fuhrerschein
fuehrerschein
führerscheinnummer
fuhrerscheinnummer
fuehrerscheinnummer
führerschein-
fuhrerschein-
fuehrerschein-
führerscheinnummernr
fuhrerscheinnummernr
fuehrerscheinnummernr
führerscheinnummerklasse
fuhrerscheinnummerklasse
fuehrerscheinnummerklasse
nr-führerschein
nr-fuhrerschein
nr-fuehrerschein
no-führerschein
no-fuhrerschein
no-fuehrerschein
n-führerschein
n-fuhrerschein
n-fuehrerschein
permis de conduire
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dlno
```

## <a name="germany-identity-card-number"></a>Número del carné de identidad de Alemania

### <a name="format"></a>Formato

desde el 1 de noviembre de 2010: Nueve letras y dígitos

desde el 1 de abril de 1987 hasta el 31 de octubre de 2010: 10 dígitos

### <a name="pattern"></a>Patrón

desde el 1 de noviembre de 2010:

- una letra (no distingue entre mayúsculas y minúsculas)
- ocho dígitos

desde el 1 de abril de 1987 hasta el 31 de octubre de 2010:

- 10 dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_germany_id_card"></a>Keyword\_germany\_id\_card

```
ausweis
gpid
identification
identifikation
identifizierungsnummer
identity card
identity number
id-nummer
personal ID
personalausweis
persönliche ID nummer
persönliche identifikationsnummer
persönliche-id-nummer
```

## <a name="germany-passport-number"></a>Número de pasaporte de Alemania

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de pasaporte de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

10 dígitos o letras

### <a name="pattern"></a>Patrón

El patrón debe incluir lo siguiente:

- el primer carácter es un dígito o una letra de este conjunto (C, F, G, H, J, K)
- tres dígitos
- cinco dígitos o letras de este conjunto (C, -H, J-N, P, R, T, V-Z)
- un dígito

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_german_passport"></a>Keyword\_german\_passport

```
reisepasse
reisepassnummer
No-Reisepass
Nr-Reisepass
Reisepass-Nr
Passnummer
reisepässe
passeport no.
passeport no
```

## <a name="greece-national-id-card"></a>Número de carné de identificación nacional de Grecia

### <a name="format"></a>Formato

Combinación de 7-8 letras y números más un guion

### <a name="pattern"></a>Patrón

Siete letras y números (formato anterior):

- Una letra (cualquier letra del alfabeto griego)
- Un guión
- Seis dígitos

Ocho letras y números (formato nuevo):

- Dos letras cuyo carácter en mayúsculas aparece en los alfabetos griego y latino (ABEZHIKMNOPTYX)
- Un guión
- Seis dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_greece_id_card"></a>Keyword\_greece\_id\_card

```
greek ID
greek national ID
greek personal ID card
greek police ID
identity card
tautotita
ταυτότητα
ταυτότητας
```

## <a name="hong-kong-identity-card-hkid-number"></a>Número del carné de identidad de Hong Kong (HKID)

### <a name="format"></a>Formato

Combinación de 8-9 letras y números más paréntesis opcionales alrededor del carácter final.

### <a name="pattern"></a>Patrón

Combinación de 8-9 letras:

- 1-2 letras (no distingue entre mayúsculas y minúsculas)
- Seis dígitos
- El carácter final (cualquier dígito o la letra A), es el dígito de control y se incluye opcionalmente entre paréntesis.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_hong_kong_id_card"></a>Keyword\_hong\_kong\_id\_card

```
hkid
Hong Kong identity card
HKIDC
ID card
identity card
hk identity card
Hong Kong ID
香港身份證
香港永久性居民身份證
身份證
身份証
身分證
身分証
香港身份証
香港身分證
香港身分証
香港身份證
香港居民身份證
香港居民身份証
香港居民身分證
香港居民身分証
香港永久性居民身份証
香港永久性居民身分證
香港永久性居民身分証
香港永久性居民身份證
香港非永久性居民身份證
香港非永久性居民身份証
香港非永久性居民身分證
香港非永久性居民身分証
香港特別行政區永久性居民身份證
香港特別行政區永久性居民身份証
香港特別行政區永久性居民身分證
香港特別行政區永久性居民身分証
香港特別行政區非永久性居民身份證
香港特別行政區非永久性居民身份証
香港特別行政區非永久性居民身分證
香港特別行政區非永久性居民身分証
```

## <a name="ip-address"></a>Dirección IP

### <a name="format"></a>Formato

#### <a name="ipv4"></a>IPv4:

patrón complejo que tiene en cuenta las versiones con formato (puntos) y sin formato (sin puntos) de las direcciones IPv4.

#### <a name="ipv6"></a>IPv6:

patrón complejo que tiene en cuenta los números de las direcciones IPv6 con formato (que incluyen dos puntos).

### <a name="pattern"></a>Patrón

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_ipaddress"></a>Keyword\_ipaddress

```
IP (this keyword is case-sensitive)
ip address
ip addresses
internet protocol
IP-כתובת ה
```

## <a name="india-permanent-account-number-pan"></a>Número de cuenta permanente de India (PAN)

### <a name="format"></a>Formato

10 letras o dígitos

### <a name="pattern"></a>Patrón

10 letras o dígitos:

- Tres letras (no se distingue entre mayúsculas y minúsculas)
- Una letra C, P, H, F, A, T, B, L, J, G (no distingue entre mayúsculas y minúsculas)
- Una letra
- Cuatro dígitos
- Una letra (no distingue entre mayúsculas y minúsculas)

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_india_permanent_account_number"></a>Keyword\_india\_permanent\_account\_number

```
Permanent Account Number
PAN
```

## <a name="india-unique-identification-aadhaar-number"></a>Número de identificación único de India (Aadhaar)

### <a name="format"></a>Formato

12 dígitos que contienen espacios o guiones opcionales

### <a name="pattern"></a>Patrón

12 dígitos:

- Un dígito, que no es 0 ni 1
- Tres dígitos
- Un espacio o un guion opcional
- Cuatro dígitos
- Un espacio o un guion opcional
- El dígito final es el dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_india_aadhar"></a>Keyword\_india\_aadhar

```
aadhaar
aadhar
aadhar#
uid
आधार
uidai
```

## <a name="indonesia-identity-card-ktp-number"></a>Número del carné de identidad de Indonesia (KTP)

### <a name="format"></a>Formato

16 dígitos que contienen puntos opcionales

### <a name="pattern"></a>Patrón

16 dígitos:

- Código de provincia de dos dígitos
- Un punto (opcional)
- Código de regencia de ciudad de dos dígitos
- Código de subdistrito de dos dígitos
- Un punto (opcional)
- Seis dígitos con el formato DDMMAA, que son la fecha de nacimiento
- Un punto (opcional)
- Cuatro dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_indonesia_id_card"></a>Keyword\_indonesia\_id\_card

```
KTP
Kartu Tanda Penduduk
Nomor Induk Kependudukan
```

## <a name="international-banking-account-number-iban"></a>Número de cuenta bancaria internacional (IBAN)

### <a name="format"></a>Formato

Código del país o región (dos letras), más los dígitos de comprobación (dos dígitos), más el número :::no-loc text="bban"::: (hasta 30 caracteres).

### <a name="pattern"></a>Patrón

El patrón debe incluir lo siguiente:

- Código de país o región de dos letras
- Dos dígitos de control (seguidos de un espacio opcional)
- 1-7 grupos de cuatro letras o dígitos (pueden separarse con espacios)
- 1-3 letras o dígitos

El formato de cada país o región es ligeramente diferente. El tipo de información confidencial de IBAN abarca estos 60 países o regiones:

```
ad, ae, al, at, az, ba, be, bg, bh, ch, cr, cy, cz, de, dk, do, ee, es, fi, fo, fr, gb, ge, gi, gl, gr, hr, hu, that is, il, is, it, kw, kz, lb, li, lt, lu, lv, mc, md, me, mk, mr, mt, mu, nl, no, pl, pt, ro, rs, sa, se, si, sk, sm, tn, tr, vg
```

### <a name="keywords"></a>Palabras clave

None

## <a name="ireland-personal-public-service-pps-number"></a>Número de servicio público personal de Irlanda (PPS)

### <a name="format"></a>Formato

Formato antiguo (hasta el 31 de diciembre de 2012):

- siete dígitos seguidos de una o dos letras

Nuevo formato (a partir del 1 de enero de 2013):

- siete dígitos seguidos de dos letras

### <a name="pattern"></a>Patrón

Formato antiguo (hasta el 31 de diciembre de 2012):

- siete dígitos
- una o dos letras (no se distingue entre mayúsculas y minúsculas)

Nuevo formato (a partir del 1 de enero de 2013):

- siete dígitos
- una letra (no distingue entre mayúsculas y minúsculas), que es un dígito de control alfabético
- Una letra opcional en el intervalo A-I o &quot;W&quot;

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_ireland_eu_national_id_card"></a>Keywords\_ireland\_eu\_national\_id\_card

```
client identity service
identification number
personal ID number
personal public service number
personal service no
phearsanta seirbhíse poiblí
pps no
pps number
pps num
pps service no
ppsn
ppsno#
ppsno
psp
public service no
publicserviceno#
publicserviceno
revenue and social insurance number
rsi no
rsi number
rsin
seirbhís aitheantais cliant
uimh
uimhir aitheantais chánach
uimhir aitheantais phearsanta
uimhir phearsanta seirbhíse poiblí
tax ID
tax identification no
tax identification number
tax no#
tax no
tax number
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="israel-bank-account-number"></a>Número de cuenta bancaria de Israel

### <a name="format"></a>Formato

13 dígitos

### <a name="pattern"></a>Patrón

Con formato:

- dos dígitos
- un guión
- tres dígitos
- un guión
- ocho dígitos

Sin formato:

- 13 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_israel_bank_account_number"></a>Keyword\_israel\_bank\_account\_number

```
Bank Account Number
Bank Account
Account Number
מספר חשבון בנק
```

## <a name="israel-national-identification-number"></a>Número de identificación nacional de Israel

### <a name="format"></a>Formato

nueve dígitos

### <a name="pattern"></a>Patrón

nueve dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_israel_national_id"></a>Keyword\_Israel\_National\_ID

```
מספר זהות
מספר זיה וי
מספר זיהוי ישר אלי
זהותישר אלית
هو ية اسرائيل ية عدد
هوية إسرائ يلية
رقم الهوية
عدد هوية فريدة من نوعها
ID number#
ID number
identity no
identity number#
identity number
israeliidentitynumber
personal ID
unique ID
```

## <a name="italy-drivers-license-number"></a>Número de permiso de conducir de Italia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de permiso de conducir de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

combinación de 10 letras y dígitos.

### <a name="pattern"></a>Patrón

combinación de 10 letras y dígitos:

- una letra (no distingue entre mayúsculas y minúsculas)
- la letra &quot;A&quot; o &quot;V&quot; (no distingue entre mayúsculas y minúsculas)
- siete dígitos
- una letra (no distingue entre mayúsculas y minúsculas)

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_italy_drivers_license_number"></a>Keyword\_italy\_drivers\_license\_number

```
numero di patente
patente di guida
patente guida
patenti di guida
patenti guida
```

## <a name="japan-bank-account-number"></a>Número de cuenta bancaria de Japón

### <a name="format"></a>Formato

siete u ocho dígitos

### <a name="pattern"></a>Patrón

número de cuenta bancaria:

- siete u ocho dígitos
- código de la sucursal de la cuenta bancaria:
- cuatro dígitos
- un espacio o guion (opcional)
- tres dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_jp_bank_account"></a>Keyword\_jp\_bank\_account

```
Checking Account Number
Checking Account
Checking Account #
Checking Acct Number
Checking Acct #
Checking Acct No.
Checking Account No.
Bank Account Number
Bank Account
Bank Account #
Bank Acct Number
Bank Acct #
Bank Acct No.
Bank Account No.
Savings Account Number
Savings Account
Savings Account #
Savings Acct Number
Savings Acct #
Savings Acct No.
Savings Account No.
Debit Account Number
Debit Account
Debit Account #
Debit Acct Number
Debit Acct #
Debit Acct No.
Debit Account No.
口座番号
銀行口座
銀行口座番号
総合口座
普通預金口座
普通口座
当座預金口座
当座口座
預金口座
振替口座
銀行
バンク
```

#### <a name="keyword_jp_bank_branch_code"></a>Keyword\_jp\_bank\_branch\_code

```
支店番号
支店コード
店番号
```

## <a name="japan-drivers-license-number"></a>Número de permiso de conducir de Japón

### <a name="format"></a>Formato

12 dígitos

### <a name="pattern"></a>Patrón

12 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_jp_drivers_license_number"></a>Keyword\_jp\_drivers\_license\_number

```
driver license
drivers license
driver'license
drivers licenses
driver'licenses
driver licenses
dl#
dls#
lic#
lics#
運転免許証
運転免許
免許証
免許
運転免許証番号
運転免許番号
免許証番号
免許番号
運転免許証ナンバー
運転免許ナンバー
免許証ナンバー
運転免許証no
運転免許no
免許証no
免許no
運転経歴証明書番号
運転経歴証明書
運転免許証No.
運転免許No.
免許証No.
免許No.
運転免許証#
運転免許#
免許証#
免許#
```

## <a name="japan-passport-number"></a>Número de pasaporte de Japón

### <a name="format"></a>Formato

dos letras seguidas por siete dígitos

### <a name="pattern"></a>Patrón

dos letras (no distingue entre mayúsculas y minúsculas) seguidas de siete dígitos

#### <a name="keyword_jp_passport"></a>Keyword\_jp\_passport

```
Passport
Passport Number
Passport No.
Passport #
パスポート
パスポート番号
パスポートナンバー
パスポート＃
パスポート#
パスポートNo.
旅券番号
旅券番号＃
旅券番号♯
旅券ナンバー
```

## <a name="japan-residence-card-number"></a>Número de la tarjeta de residencia de Japón

### <a name="format"></a>Formato

12 letras y dígitos

### <a name="pattern"></a>Patrón

12 letras y dígitos:

- dos letras (no se distingue entre mayúsculas y minúsculas)
- ocho dígitos
- dos letras (no se distingue entre mayúsculas y minúsculas)

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_jp_residence_card_number"></a>Keyword\_jp\_residence\_card\_number

```
Residence card number
Residence card no
Residence card #
在留カード番号
在留カード
在留番号
```

## <a name="japan-resident-registration-number"></a>Número de registro de residente de Japón

### <a name="format"></a>Formato

11 dígitos

### <a name="pattern"></a>Patrón

11 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_jp_resident_registration_number"></a>Keyword\_jp\_resident\_registration\_number

```
Resident Registration Number
Residents Basic Registry Number
Resident Registration No.
Resident Register No.
Residents Basic Registry No.
Basic Resident Register No.
外国人登録証明書番号
証明書番号
登録番号
外国人登録証
```

## <a name="japan-social-insurance-number-sin"></a>Número del seguro social (SIN) de Japón

### <a name="format"></a>Formato

7-12 dígitos

### <a name="pattern"></a>Patrón

7-12 dígitos:

- cuatro dígitos
- un guion (opcional)
- seis dígitos O
- 7-12 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_jp_sin"></a>Keyword\_jp\_sin

```
Social Insurance No.
Social Insurance Num
Social Insurance Number
健康保険被保険者番号
健保番号
基礎年金番号
雇用保険被保険者番号
雇用保険番号
保険証番号
社会保険番号
社会保険No.
社会保険
介護保険
介護保険被保険者番号
健康保険被保険者整理番号
雇用保険被保険者整理番号
厚生年金
厚生年金被保険者整理番号
```

## <a name="malaysia-identification-card-number"></a>Número del carné de identidad de Malasia

### <a name="format"></a>Formato

12 dígitos que contienen guiones opcionales

### <a name="pattern"></a>Patrón

12 dígitos:

- seis dígitos con el formato AAMMDD, que son la fecha de nacimiento
- un guión (opcional)
- un código del lugar de nacimiento de dos letras
- un guión (opcional)
- tres dígitos aleatorios
- un código de género de un dígito

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_malaysia_id_card_number"></a>Keyword\_malaysia\_id\_card\_number

```
digital application card
i/c
i/c no
ic
ic no
ID card
identification Card
identity card
k/p
k/p no
kad akuan diri
kad aplikasi digital
kad pengenalan malaysia
kp
kp no
mykad
mykas
mykid
mypr
mytentera
malaysia identity card
malaysian identity card
nric
personal identification card
```

## <a name="netherlands-citizens-service-bsn-number"></a>Número de servicio de ciudadano de Países Bajos (BSN)

### <a name="format"></a>Formato

entre ocho y nueve dígitos que contienen espacios opcionales

### <a name="pattern"></a>Patrón

entre ocho y nueve dígitos:

- tres dígitos
- un espacio (opcional)
- tres dígitos
- un espacio (opcional)
- entre dos y tres dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_netherlands_eu_national_id_card"></a>Keywords\_netherlands\_eu\_national\_id\_card

```
bsn#
bsn
burgerservicenummer
citizen service number
person number
personal number
personal numeric code
person-related number
persoonlijk nummer
persoonlijke numerieke code
persoonsgebonden
persoonsnummer
sociaal-fiscaal nummer
social-fiscal number
sofi
sofinummer
uniek identificatienummer
uniek identiteitsnummer
unique identification number
unique identity number
uniqueidentityno#
```

## <a name="new-zealand-ministry-of-health-number"></a>Número del Ministerio de Sanidad de Nueva Zelanda

### <a name="format"></a>Formato

tres letras, un espacio (opcional) y cuatro dígitos

### <a name="pattern"></a>Patrón

- tres letras (no se distingue entre mayúsculas y minúsculas), excepto "I" y "O"
- un espacio (opcional)
- cuatro dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_nz_terms"></a>Keyword\_nz\_terms

```
NHI
New Zealand
Health
treatment
National Health Index Number
nhi number
nhi no.
NHI#
National Health Index No.
National Health Index Id
National Health Index #
```

## <a name="norway-identification-number"></a>Número de identificación de Noruega

### <a name="format"></a>Formato

11 dígitos

### <a name="pattern"></a>Patrón

11 dígitos:

- seis dígitos con el formato DDMMAA, que son la fecha de nacimiento
- número individual de tres dígitos
- dos dígitos de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_norway_id_number"></a>Keyword\_norway\_id\_number

```
Personal identification number
Norwegian ID Number
ID Number
Identification
Personnummer
Fødselsnummer
```

## <a name="philippines-unified-multi-purpose-identification-number"></a>Número de identificación multiuso unificado de Filipinas

### <a name="format"></a>Formato

12 dígitos separados por guiones

### <a name="pattern"></a>Patrón

12 dígitos:

- cuatro dígitos
- un guion
- siete dígitos
- un guion
- un dígito

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_philippines_id"></a>Keyword\_philippines\_id

```
Unified Multi-Purpose ID
UMID
Identity Card
Pinag-isang Multi-Layunin ID
```

## <a name="poland-identity-card"></a>Carné de identidad de Polonia

### <a name="format"></a>Formato

tres letras y seis dígitos

### <a name="pattern"></a>Patrón

tres letras (no distingue entre mayúsculas y minúsculas) seguidas de seis dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_poland_national_id_passport_number"></a>Keyword\_poland\_national\_id\_passport\_number

```
Dowód osobisty
Numer dowodu osobistego
Nazwa i numer dowodu osobistego
Nazwa i nr dowodu osobistego
Nazwa i nr dowodu tożsamości
Dowód Tożsamości
dow. os.
```

## <a name="poland-national-id-pesel"></a>Identificador nacional de Polonia (PESEL)

### <a name="format"></a>Formato

11 dígitos

### <a name="pattern"></a>Patrón

- 6 dígitos que representan la fecha de nacimiento en el formato AAMMDD
- 4 dígitos
- 1 dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_pesel_identification_number"></a>Keyword\_pesel\_identification\_number

```
dowód osobisty
dowódosobisty
niepowtarzalny numer
niepowtarzalnynumer
nr.-pesel
nr-pesel
numer identyfikacyjny
pesel
tożsamości narodowej
```

## <a name="poland-passport-number"></a>Número de pasaporte de Polonia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de pasaporte de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

dos letras y siete dígitos

### <a name="pattern"></a>Patrón

Dos letras (no distingue entre mayúsculas y minúsculas) seguidas de siete dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_poland_national_id_passport_number"></a>Keyword\_poland\_national\_id\_passport\_number

```
Numer paszportu
Nr. Paszportu
Paszport
```

## <a name="portugal-citizen-card-number"></a>Número de la tarjeta de ciudadano de Portugal

### <a name="format"></a>Formato

ocho dígitos

### <a name="pattern"></a>Patrón

ocho dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_portugal_citizen_card"></a>Keyword\_portugal\_citizen\_card

```
bilhete de identidade
cartão de cidadão
citizen card
document number
documento de identificação
ID number
identification no
identification number
identity card no
identity card number
national ID card
nic
número bi de portugal
número de identificação civil
número de identificação fiscal
número do documento
portugal bi number
```

## <a name="portugal-drivers-license-number"></a>Número de permiso de conducir de Portugal

Esta entidad de tipo de información confidencial solo está disponible en el tipo de información confidencial del número de permiso de conducir de la UE.

### <a name="format"></a>Formato

dos patrones: dos letras seguidas de 5-8 dígitos con caracteres especiales

### <a name="pattern"></a>Patrón

Patrón 1: Dos letras seguidas de 5/6 con caracteres especiales:

- Dos letras (no se distingue entre mayúsculas y minúsculas)
- Un guión
- Cinco o seis dígitos
- Un espacio
- Un dígito

Patrón 2: Una letra seguida de 6/8 dígitos con caracteres especiales:

- Una letra (no distingue entre mayúsculas y minúsculas)
- Un guión
- Seis u ocho dígitos
- Un espacio
- Un dígito

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_eu_drivers_license_number"></a>Keywords\_eu\_driver's\_license\_number

```
driverlic
driverlics
driver license
driver licenses
driverlicence
driverlicences
driver lic
driver lics
driver license
driver licenses
driver license
driver licenses
driverslic
driverslics
driverslicence
driverslicences
drivers license
drivers licenses
drivers lic
drivers lics
drivers license
drivers licenses
drivers license
drivers licenses
driver'lic
driver'lics
driver'license
driver'licenses
driver'license
driver'licenses
driver' lic
driver' lics
driver' license
driver' licenses
driver' license
driver' licenses
driver'slic
driver'slics
driver'license
driver'licenses
driver'license
driver'licenses
driver's lic
driver's lics
driver's license
driver's licenses
driver's license
driver's licenses
dl#
dls#
driverlic#
driverlics#
driver license#
driver licenses#
driverlicence#
driverlicences#
driver lic#
driver lics#
driver license#
driver licenses#
driver licenses#
driverslic#
driverslics#
drivers license#
drivers licenses#
driverslicence#
driverslicences#
drivers lic#
drivers lics#
drivers license#
drivers licenses#
drivers license#
drivers licenses#
driver'lic#
driver'lics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver' lic#
driver' lics#
driver' license#
driver' licenses#
driver' license#
driver' licenses#
driver'slic#
driver'slics#
driver'license#
driver'licenses#
driver'license#
driver'licenses#
driver's lic#
driver's lics#
driver's license#
driver's licenses#
driver's license#
driver's licenses#
driving license
driving license
dlno#
driv lic
driv licen
driv license
driv licenses
driv license
driv licenses
driver licen
drivers licen
driver's licen
driving lic
driving licen
driving licenses
driving license
driving licenses
driving permit
dl no
dlno
dl number
```

#### <a name="keywords_portugal_eu_drivers_license_number"></a>Keywords\_portugal\_eu\_driver's\_license\_number

```
carteira de motorista
carteira motorista
carteira de habilitação
carteira habilitação
número de licença
número licença
permissão de condução
permissão condução
Licença condução Portugal
carta de condução
```

## <a name="saudi-arabia-national-id"></a>Identificador nacional de Arabia Saudí

### <a name="format"></a>Formato

10 dígitos

### <a name="pattern"></a>Patrón

10 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_saudi_arabia_national_id"></a>Keyword\_saudi\_arabia\_national\_id

```
Identification Card
I card number
ID number
الوطنية الهوية بطاقة رقم
```

## <a name="singapore-national-registration-identity-card-nric-number"></a>Número del carné de identidad del registro nacional de Singapur (NRIC)

### <a name="format"></a>Formato

nueve letras y dígitos

### <a name="pattern"></a>Patrón

- nueve letras y dígitos:
- la letra &quot;F&quot;, &quot;G&quot;, &quot;S&quot; o &quot;T&quot; (no distingue entre mayúsculas y minúsculas)
- siete dígitos
- un dígito de control alfabético

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_singapore_nric"></a>Keyword\_singapore\_nric

```
National Registration Identity Card
Identity Card Number
NRIC
IC
Foreign Identification Number
FIN
身份证
身份證
```

## <a name="south-africa-identification-number"></a>Número de identificación de Sudáfrica

### <a name="format"></a>Formato

13 dígitos que pueden contener espacios

### <a name="pattern"></a>Patrón

13 dígitos:

- seis dígitos con el formato AAMMDD, que son la fecha de nacimiento
- cuatro dígitos
- indicador de ciudadanía de un solo dígito
- el dígito &quot;8&quot; o &quot;9&quot;
- un dígito, que es un dígito de suma de comprobación

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_south_africa_identification_number"></a>Keyword\_south\_africa\_identification\_number

```
Identity card
ID
Identification
```

## <a name="south-korea-resident-registration-number"></a>Número de registro de residente de Corea del Sur

### <a name="format"></a>Formato

13 dígitos que contienen un guion

### <a name="pattern"></a>Patrón

13 dígitos:

- seis dígitos con el formato AAMMDD, que son la fecha de nacimiento
- un guion
- un dígito determinado por el siglo y el sexo
- un código de la región de nacimiento de cuatro dígitos
- un dígito que se usa para diferenciar a las personas cuyos números anteriores son idénticos
- un dígito de control

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_south_korea_resident_number"></a>Keyword\_south\_korea\_resident\_number

```
National ID card
Citizen's Registration Number
Jumin deungnok beonho
RRN
주민등록번호
```

## <a name="spain-social-security-number-ssn"></a>Número de la seguridad social de España (SSN)

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número del seguro social de la UE o el id. equivalente, y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

11-12 dígitos

### <a name="pattern"></a>Patrón

11-12 dígitos:

- dos dígitos
- una barra diagonal (opcional)
- de siete a ocho dígitos
- una barra diagonal (opcional)
- dos dígitos

### <a name="keywords"></a>Palabras clave

None

## <a name="sweden-national-id"></a>Identificación nacional de Suecia

### <a name="format"></a>Formato

10 o 12 dígitos y un delimitador opcional

### <a name="pattern"></a>Patrón

10 o 12 dígitos y un delimitador opcional:

- dos dígitos (opcional)
- Seis dígitos en formato de fecha AAMMDD
- delimitador de &quot;-&quot; o &quot;+&quot; (opcional)
- cuatro dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_swedish_national_identifier"></a>Keywords\_swedish\_national\_identifier

```
ID no
ID number
ID#
identification no
identification number
identifikationsnumret#
identifikationsnumret
identitetshandling
identity document
identity no
identity number
id-nummer
personal ID
personnummer#
personnummer
skatteidentifikationsnummer
```

## <a name="sweden-passport-number"></a>Número de pasaporte de Suecia

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de pasaporte de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

ocho dígitos

### <a name="pattern"></a>Patrón

ocho dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_sweden_passport"></a>Keyword\_sweden\_passport

```
visa requirements
Alien Registration Card
Schengen visas
Schengen visa
Visa Processing
Visa Type
Single Entry
Multiple Entries
G3 Processing Fees
```

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```

## <a name="swift-code"></a>Código SWIFT

### <a name="format"></a>Formato

cuatro letras seguidas de 5-31 letras o dígitos

### <a name="pattern"></a>Patrón

cuatro letras seguidas de 5-31 letras o dígitos:

- código bancario de cuatro letras (no distingue mayúsculas de minúsculas)
- un espacio opcional
- 4-28 letras o dígitos (el número de cuenta bancaria básico [BBAN])
- un espacio opcional
- de una a tres letras o dígitos (resto del BBAN)

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_swift"></a>Keyword\_swift

```
international organization for standardization 9362
iso 9362
iso9362
swift#
swift code
swift number
swiftroutingnumber
swift code
swift number #
swift routing number
bic number
bic code
bic #
bic#
bank identifier code
Organization internationale de normalization 9362
rapide #
code SWIFT
le numéro de swift
swift numéro d'acheminement
le numéro BIC
#BIC
code identificateur de banque
SWIFTコード
SWIFT番号
BIC番号
BICコード
SWIFT コード
SWIFT 番号
BIC 番号
BIC コード
金融機関識別コード
金融機関コード
銀行コード
```

## <a name="taiwan-national-identification-number"></a>Número de identificación nacional de Taiwán

### <a name="format"></a>Formato

una letra (en inglés) seguida de nueve dígitos

### <a name="pattern"></a>Patrón

una letra (en inglés) seguida de nueve dígitos:

- una letra (en inglés y que no distingue entre mayúsculas y minúsculas)
- el dígito &quot;1&quot; o &quot;2&quot;
- ocho dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_taiwan_national_id"></a>Keyword\_taiwan\_national\_id

```
身份證字號
身份證
身份證號碼
身份證號
身分證字號
身分證
身分證號碼
身份證號
身分證統一編號
國民身分證統一編號
簽名
蓋章
簽名或蓋章
簽章
```

## <a name="taiwan-passport-number"></a>Número de pasaporte de Taiwán

### <a name="format"></a>Formato

- número de pasaporte biométrico: nueve dígitos
- número de pasaporte no biométrico: nueve dígitos

### <a name="pattern"></a>Patrón

número de pasaporte biométrico:

- el carácter &quot;3&quot;
- ocho dígitos

número de pasaporte no biométrico:

- nueve dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_taiwan_passport"></a>Keyword\_taiwan\_passport

```
ROC passport number
Passport number
Passport no
Passport Num
Passport #
护照
中華民國護照
Zhōnghuá Mínguó hùzhào
```

## <a name="taiwan-resident-certificate-arctarc-number"></a>Certificado de residente de Taiwán (ARC/TARC)

### <a name="format"></a>Formato

10 letras y dígitos

### <a name="pattern"></a>Patrón

10 letras y dígitos:

- dos letras (no se distingue entre mayúsculas y minúsculas)
- ocho dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_taiwan_resident_certificate"></a>Keyword\_taiwan\_resident\_certificate

```
Resident Certificate
Resident Cert
Resident Cert.
Identification card
Alien Resident Certificate
ARC
Taiwan Area Resident Certificate
TARC
居留證
外僑居留證
台灣地區居留證
```

## <a name="uk-drivers-license-number"></a>Reino Unido Número de permiso de conducir

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de permiso de conducir de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

Combinación de 18 letras y dígitos en el formato especificado

### <a name="pattern"></a>Patrón

18 letras y dígitos:

- cinco letras (no se distingue entre mayúsculas y minúsculas) o el dígito &quot;9&quot; en lugar de una letra
- un dígito
- cinco dígitos con el formato de fecha MMDDA para la fecha de nacimiento (el séptimo carácter se incrementa en 50 si el conductor es una mujer; es decir, de 51 a 62 en lugar de 01 a 12)
- dos letras (no se distingue entre mayúsculas y minúsculas) o el dígito &quot;9&quot; en lugar de una letra
- cinco dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_uk_drivers_license"></a>Keyword\_uk\_drivers\_license

```
DVLA
light vans
quad bikes
motor cars
125cc
sidecar
tricycles
motorcycles
photo card license
learner drivers
license holder
license holders
driving licenses
driving license
dual control car
```

## <a name="uk-electoral-roll-number"></a>Reino Unido número de lista electoral

### <a name="format"></a>Formato

dos letras seguidas por 1-4 dígitos

### <a name="pattern"></a>Patrón

dos letras (no se distingue entre mayúsculas y minúsculas) seguidas de los números 1 al 4

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_uk_electoral"></a>Keyword\_uk\_electoral

- nominación del consejo
- formulario de nominación
- registro electoral
- lista electoral

## <a name="uk-national-health-service-number"></a>Reino Unido número del servicio de salud nacional

### <a name="format"></a>Formato

10-17 dígitos separados por espacios

### <a name="pattern"></a>Patrón

10-17 dígitos:

- tres o 10 dígitos
- un espacio
- tres dígitos
- un espacio
- cuatro dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_uk_nhs_number"></a>Keyword\_uk\_nhs\_number

```
national health service
nhs
health services authority
health authority
```

#### <a name="keyword_uk_nhs_number1"></a>Keyword\_uk\_nhs\_number1

```
patient ID
patient identification
patient no
patient number
```

#### <a name="keyword_uk_nhs_number_dob"></a>Keyword\_uk\_nhs\_number\_dob

```
GP
DOB
D.O.B
Date of Birth
Birth Date
```

## <a name="uk-national-insurance-number-nino"></a>Reino Unido número del seguro nacional (NINO)

Esta entidad de tipo de información confidencial se incluye en el tipo de información confidencial del Número de identificación nacional de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

siete o nueve caracteres separados por espacios o guiones

### <a name="pattern"></a>Patrón

dos posibles patrones:

- dos letras (los números NINO válidos usan solo ciertos caracteres en este prefijo, que el patrón se encarga de validar; no se distingue entre mayúsculas y minúsculas)
- seis dígitos
- "A", "B", "C" o "D" (como en el prefijo, solo se permiten determinados caracteres en el sufijo; no se distingue entre mayúsculas y minúsculas)

O BIEN

- dos letras
- un espacio o guion
- dos dígitos
- un espacio o guion
- dos dígitos
- un espacio o guion
- dos dígitos
- un espacio o guion
- "A", "B", "C" o "D"


### <a name="keywords"></a>Palabras clave

#### <a name="keyword_uk_nino"></a>Keyword\_uk\_nino

```
national insurance number
national insurance contributions
protection act
insurance
social security number
insurance application
medical application
social insurance
medical attention
social security
Great Britain
NI Number
NI No.
NI #
NI#
insurance#
insurance number
national insurance#
nationalinsurancenumber
```

## <a name="uk-unique-taxpayer-reference-number"></a>Reino Unido Número único de referencia del contribuyente

Este tipo de información confidencial solo está disponible para su uso en:

- directivas para la prevención de pérdida de datos
- directivas de cumplimiento de la comunicación
- gobernanza de la información
- administración de registros
- Microsoft Cloud App Security

### <a name="format"></a>Formato

10 dígitos sin espacios ni delimitadores

### <a name="pattern"></a>Patrón

10 dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keywords_uk_eu_tax_file_number"></a>Keywords\_uk\_eu\_tax\_file\_number

```
tax number
tax file
tax ID
tax identification no
tax identification number
tax no#
tax no
tax registration number
tax ID#
tax ID no#
tax ID number#
tax no#
tax number#
tax number
tin ID
tin no
tin#
```

## <a name="us-bank-account-number"></a>Número de cuenta bancaria de EE. UU.

### <a name="format"></a>Formato

6-17 dígitos

### <a name="pattern"></a>Patrón

6-17 dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_usa_bank_account"></a>Keyword\_usa\_Bank\_Account

```
Checking Account Number
Checking Account
Checking Account #
Checking Acct Number
Checking Acct #
Checking Acct No.
Checking Account No.
Bank Account Number
Bank Account #
Bank Acct Number
Bank Acct #
Bank Acct No.
Bank Account No.
Savings Account Number
Savings Account.
Savings Account #
Savings Acct Number
Savings Acct #
Savings Acct No.
Savings Account No.
Debit Account Number
Debit Account
Debit Account #
Debit Acct Number
Debit Acct #
Debit Acct No.
Debit Account No.
```

## <a name="us-drivers-license-number"></a>Número de permiso de conducir de EE. UU.

### <a name="format"></a>Formato

Depende del estado

### <a name="pattern"></a>Patrón

depende del estado; por ejemplo, Nueva York:

- nueve dígitos con formato DDD coincidirán.
- nueve dígitos, con formato ddddddddd, no coincidirán.

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_us_drivers_license_abbreviations"></a>Keyword\_us\_drivers\_license\_abbreviations

```
DL
DLS
CDL
CDLS
ID
IDs
DL#
DLS#
CDL#
CDLS#
ID#
IDs#
ID number
ID numbers
LIC
LIC#
```

#### <a name="keyword_us_drivers_license"></a>Keyword\_us\_drivers\_license

```
DriverLic
DriverLics
DriverLicense
DriverLicenses
Driver Lic
Driver Lics
Driver License
Driver Licenses
DriversLic
DriversLics
DriversLicense
DriversLicenses
Drivers Lic
Drivers Lics
Drivers License
Drivers Licenses
Driver'Lic
Driver'Lics
Driver'License
Driver'Licenses
Driver' Lic
Driver' Lics
Driver' License
Driver' Licenses
Driver'sLic
Driver'sLics
Driver'sLicense
Driver'sLicenses
Driver's Lic
Driver's Lics
Driver's License
Driver's Licenses
identification number
identification numbers
identification #
ID card
ID cards
identification card
identification cards
DriverLic#
DriverLics#
DriverLicense#
DriverLicenses#
Driver Lic#
Driver Lics#
Driver License#
Driver Licenses#
DriversLic#
DriversLics#
DriversLicense#
DriversLicenses#
Drivers Lic#
Drivers Lics#
Drivers License#
Drivers Licenses#
Driver'Lic#
Driver'Lics#
Driver'License#
Driver'Licenses#
Driver' Lic#
Driver' Lics#
Driver' License#
Driver' Licenses#
Driver'sLic#
Driver'sLics#
Driver'sLicense#
Driver'sLicenses#
Driver's Lic#
Driver's Lics#
Driver's License#
Driver's Licenses#
ID card#
ID cards#
identification card#
identification cards#
```

#### <a name="keyword_state_name_drivers_license_name"></a>Keyword\_[state\_name]\_drivers\_license\_name

- abreviatura del estado (por ejemplo, &quot;NY&quot;)
- nombre del estado (por ejemplo, &quot;Nueva York&quot;)

## <a name="us-individual-taxpayer-identification-number-itin"></a>Número de identificación fiscal (NIF) individual de EE. UU. (ITIN)

### <a name="format"></a>Formato

nueve dígitos que comienzan con un &quot;9&quot; y contienen un &quot;7&quot; u &quot;8&quot; como cuarto dígito; asimismo, cuentan con un formato opcional de espacios o guiones.

### <a name="pattern"></a>Patrón

con formato:

- el dígito &quot;9&quot;
- dos dígitos
- un espacio o guion
- un &quot;7&quot; u &quot;8&quot;
- un dígito
- un espacio o guion
- cuatro dígitos

sin formato:

- el dígito &quot;9&quot;
- dos dígitos
- un &quot;7&quot; u &quot;8&quot;
- cinco dígitos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_itin"></a>Keyword\_itin

```
taxpayer
tax ID
tax identification
itin
i.t.i.n.
ssn
tin
social security
tax payer
itins
tax ID
individual taxpayer
```

## <a name="us-social-security-number-ssn"></a>Número del seguro social de EE. UU. (SSN)

### <a name="format"></a>Formato

nueve dígitos que pueden estar en un patrón con formato o sin formato.

>[!Note]
> Si se emite antes de mediados de 2011, el SSN tiene un formato seguro en el que algunas partes del número deben estar dentro de determinados intervalos para ser válidas (pero no hay ninguna suma de comprobación).
>

### <a name="pattern"></a>Patrón

cuatro funciones buscan el número de SSN en cuatro patrones diferentes:

- Func\_ssn busca números de SSN con un formato seguro anterior a 2011, a los que se agregan guiones o espacios (ddd-dd-dddd O ddd dd dddd).
- Func\_unformatted\_ssn busca números de SSN con un formato seguro anterior a 2011 que no tengan el formato de nueve dígitos consecutivos (ddddddddd).
- Func\_randomized\_formatted\_ssn busca números de SSN anteriores a 2011, que tengan un formato que incluye guiones o espacios (ddd-dd-dddd O ddd dd dddd).
- Func\_randomized\_unformatted\_ssn busca números de SSN anterior a 2011 que no tenga el formato de nueve dígitos consecutivos (ddddddddd).

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_ssn"></a>Keyword\_ssn

```
SSA Number
social security number
social security #
social security#
social security no
Social Security#
Soc Sec
SSN
SSNS
SSN#
SS#
SSID
```

## <a name="us--uk-passport-number"></a>EE. UU./Reino Unido número de pasaporte

Esta entidad de tipo de información confidencial del número de pasaporte del Reino Unido. se incluye en el tipo de información confidencial del Número de pasaporte de la UE y está disponible como entidad independiente del tipo de información confidencial.

### <a name="format"></a>Formato

nueve dígitos

### <a name="pattern"></a>Patrón

nueve dígitos consecutivos

### <a name="keywords"></a>Palabras clave

#### <a name="keyword_passport"></a>Keyword\_passport

```
Passport Number
Passport No
Passport #
Passport#
PassportID
Passport no
passport number
パスポート
パスポート番号
パスポートのNum
パスポート＃
Numéro de passeport
Passeport n °
Passeport Non
Passeport #
Passeport#
PasseportNon
Passeportn °
```
