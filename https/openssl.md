# openssl

## 1. openssl genrsa <a href="#openssl--genrsa" id="openssl--genrsa"></a>

```
功能：
用于生成RSA私钥，不会生成公钥，因为公钥提取自私钥
使用参数：
openssl genrsa [-out filename] [-passout arg] [-des] [-des3] [-idea] [numbits]

选项说明：
-out filename     ：将生成的私钥保存至filename文件，若未指定输出文件，则为标准输出。

-numbits            ：指定要生成的私钥的长度，默认为1024。该项必须为命令行的最后一项参数。

-des|-des3|-idea：指定加密私钥文件用的算法，这样每次使用私钥文件都将输入密码，太麻烦所以很少使用。

-passout args    ：加密私钥文件时，传递密码的格式，如果要加密私钥文件时单未指定该项，则提示输入密码。传递密码的args的格式见一下格式。
a.pass:password   ：password表示传递的明文密码
b.env:var               ：从环境变量var获取密码值
c.file:filename        ：filename文件中的第一行为要传递的密码。若filename同时传递给"-passin"和"-passout"选项，则filename的第一行为"-passin"的值，第二行为"-passout"的值
d.stdin                   ：从标准输入中获取要传递的密码
```

## 2. openssl req <a href="#openssl-req" id="openssl-req"></a>

```
用途：
生成证书请求文件、验证证书请求文件和创建根CA。

语法：
openssl req 
[-new] [-newkey rsa:bits] [-verify] [-x509] [-in filename] [-out filename]
[-key filename] [-passin arg] [-passout arg] [-keyout filename] [-pubkey] 
[-nodes] [-[dgst]] [-config filename] [-subj arg] [-days n] [-set_serial n]
[-extensions section] [-reqexts section] [-utf8] [-nameopt] [-reqopt] [-subject] 
[-subj arg] [-text] [-noout] [-batch] [-verbose]


参数说明：
-new ：创建一个证书请求文件，会交互式提醒输入一些信息，这些交互选项以及交互选项信息的长度值以及其他一些扩展属性在配置文件(默认为：openssl.cnf，还有些辅助配置文件)中指定了默认值。如果没有指定"-key"选项，则会自动生成一个RSA私钥，该私钥的生成位置也在openssl.cnf中指定了。如果指定了-x509选项，则表示创建的是自签署证书文件，而非证书请求文件

-newkey args    ：类似于"-new"选项，创建一个新的证书请求，并创建私钥。args的格式是"rsa:bits"(其他加密算法请查看man)，其中bits是rsa密钥的长度，如果bits省略了(即-newkey rsa)，则长度根据配置文件中default_bits指令的值作为默认长度，默认该值为2048，如果指定了-x509选项，则表示创建的是自签署证书文件，而非证书请求文件

-nodes ：默认情况下，openssl req自动创建私钥时都要求加密并提示输入加密密码，指定该选项后则禁止对私钥文件加密

-key filename ：指定私钥的输入文件，创建证书请求时需要

-keyout filename ：指定自动创建私钥时私钥的存放位置，若未指定该选项，则使用配置文件中default_keyfile指定的值，默认该值为privkey.pem

-[dgst] ：指定对创建请求时提供的申请者信息进行数字签名时的单向加密算法，如-md5/-sha1/-sha512等，若未指定则默认使用配置文件中default_md指定的值 -verify ：对证书请求文件进行数字签名验证

-x509    ：指定该选项时，将生成一个自签署证书，而不是创建证书请求。一般用于测试或者为根CA创建自签名证书

-days n ：指定自签名证书的有效期限，默认30天，需要和"-x509"一起使用。注意是自签名证书期限，而非请求的证书期限，因为证书的有效期是颁发者指定的，证书请求者指定有效期是没有意义的，配置文件中的default_days指定了请求证书的有效期限，默认365天

-set_serial n    ：指定生成自签名证书时的证书序列号，该序列号将写入配置文件中serial指定的文件中，这样就不需要手动更新该序列号文件，支持数值和16进制值(0x开头)，虽然也支持负数，但不建议

-in filename ：指定证书请求文件filename。注意，创建证书请求文件时是不需要指定该选项的

-out filename ：证书请求或自签署证书的输出文件，也可以是其他内容的输出文件，不指定时默认stdout

-subj args ：替换或自定义证书请求时需要输入的信息，并输出修改后的请求信息。args的格式为"/type0=value0/type1=value1..."，如果value为空，则表示使用配置文件中指定的默认值，如果value值为"."，则表示该项留空。其中可识别type(man req)有：C是Country、ST是state、L是localcity、O是Organization、OU是Organization Unit、CN是common name等 

【输出内容选项：】

-text ：以文本格式打印证书请求

-noout ：不输出部分信息

 -subject ：输出证书请求文件中的subject(如果指定了x509，则打印证书中的subject)

-pubkey ：输出证书请求文件中的公钥 

【配置文件项和杂项：】

-passin arg ：传递解密密码

-passout arg ：指定加密输出文件时的密码

-config filename ：指定req的配置文件，指定后将忽略所有的其他配置文件。如果不指定则默认使用/etc/pki/tls/openssl.cnf中req段落的值

-batch ：非交互模式，直接从配置文件(默认/etc/pki/tls/openssl.cnf)中读取证书请求所需字段信息。但若不指定"-key"时，仍会询问key

-verbose ：显示操作执行的详细信息

input_password ：密码输入文件，和命令行的"-passin"选项对应， 

output_password    ：密码的输出文件，与命令行的"-passout"选项对应，

default_bits ：openssl req自动生成RSA私钥时的长度，不写时默认是512，命令行的"-new"和"-newkey"可能会用到它

default_keyfile：默认的私钥输出文件，与命令行的"-keyout"选项对应

encrypt_key ：当设置为no时，自动创建私钥时不会加密该私钥。设置为no时与命令行的"-nodes"等价。还有等价的兼容性写法：encry_rsa_key

default_md ：指定创建证书请求时对申请者信息进行数字签名的单向加密算法，与命令行的"-[dgst]"对应

prompt ：当指定为no时，则不提示输入证书请求的字段信息，而是直接从openssl.cnf中读取 ：请小心设置该选项，很可能请求文件创建失败就是因为该选项设置为no

distinguished_name：(DN)是一个扩展属性段落，用于指定证书请求时可被识别的字段名称。


示例：
a.根据私钥pri_key.pem生成一个新的证书请求文件。其中"-new"表示新生成一个新的证书请求文件，"-key"指定私钥文件，"-out"指定输出文件，此处输出文件即为证书请求文件。
openssl genrsa -out pri_key.pem
openssl req -new -key pri_key.pem -out req1.csr

b.查看证书请求文件内容。
openssl req -in req1.csr或cat req1.csr或openssl req -in req1.csr -text

c.指定证书请求文件中的签名算法。
 openssl req -new -key pri_key.pem -out req2.csr -md5

d.验证请求文件的数字签名,这样可以验证出证书请求文件是否被篡改过。
openssl req -verify -in req2.csr

e.自签署证书，可用于自建根CA时。
 openssl req -x509 -key pri_key.pem -in req1.csr -out CA1.crt -days 365

f.让openssl req自动创建所需的私钥文件。
openssl req -new -out req3.csr 或
openssl req -new -out req3.csr -nodes -keyout myprivkey.pem

g.使用"-newkey"选项。
openssl req -newkey rsa:2048 -out req3.csr -nodes -keyout myprivkey.pem
```

## 3. openssl x509 <a href="#openssl-x509" id="openssl-x509"></a>

```
openssl x509命令具以下的一些功能，例如输出证书信息，签署证书请求文件、生成自签名证书、转换证书格式等。openssl x509工具不会使用openssl配置文件中的设定，而是完全需要自行设定或者使用该伪命令的默认值，它就像是一个完整的小型的CA工具箱。

主要选项：
-in filename ： #指定证书输入文件，若同时指定了"-req"选项，则表示输入文件为证书请求文件。
-out filename ： #指定输出文件
-md2|-md5|-sha1|-mdc2： #指定单向加密的算法。

查看证书选项：
-text ：以text格式输出证书内容，即以最全格式输出， 包括public key,signature algorithms,issuer和subject names,serial number以及any trust settings.
-certopt option：自定义要输出的项
-noout ：禁止输出证书请求文件中的编码部分
-pubkey ：输出证书中的公钥
-modulus ：输出证书中公钥模块部分
-serial ：输出证书的序列号
-subject ：输出证书中的subject
-issuer ：输出证书中的issuer，即颁发者的subject
-subject_hash：输出证书中subject的hash码
-issuer_hash ：输出证书中issuer(即颁发者的subject)的hash码
-hash ：等价于"-subject_hash"，但此项是为了向后兼容才提供的选项
-email ：输出证书中的email地址，如果有email的话
-startdate ：输出证书有效期的起始日期
-enddate ：输出证书有效期的终止日期
-dates ：输出证书有效期，等价于"startdate+enddate"
-fingerprint ：输出指纹摘要信息

签署选项：
*****************************************************************************************
* 伪命令x509可以像openssl ca一样对证书或请求执行签名动作。注意，openssl x509 *
* 不读取配置文件，所有的一切配置都由x509自行提供，所以openssl x509像是一个"mini CA" *
*****************************************************************************************
-signkey filename ：该选项指定签名秘钥（秘钥必须是输入文件的私钥），并使得由-in 提供的输入文件转成自签名的证书。
                  ：自签署的输入文件"-in file"的file可以是证书请求文件，也可以是已签署过的证书。会使用文件中的subject name 作为issuer name.
-x509toreq ：将已签署的证书转换回证书请求文件。需要使用"-signkey"选项来传递需要的私钥。
-req ：x509工具默认以证书文件做为inputfile(-in file)，指定该选项将使得input file的file为证书请求文件。
-set_serial n ：指定证书序列号。该选项可以和"-singkey"或"-CA"选项一起使用。
                  ：如果和"-CA"一起使用，则"-CAserial"或"-CAcreateserial"选项指定的serial值将失效。
                  ：序列号可以使用数值或16进制值(0x开头)。也接受负值，但是不建议。
-CA filename ：指定签署时所使用的CA证书。该选项一般和"-req"选项一起使用，用于为证书请求文件签署。
-CAkey filename ：设置CA签署时使用的私钥文件。如果该选项没有指定，将假定CA私钥已经存在于CA自签名的证书文件中。
-CAserial filename：设置CA使用的序列号文件。当使用"-CA"选项来签名时，它将会使用某个文件中指定的序列号来唯一标识此次签名后的证书文件。
                  ：这个序列号文件的内容仅只有一行，这一行的值为16进制的数字。当某个序列号被使用后，该文件中的序列号将自动增加。
                  ：默认序列号文件以CA证书文件基名加".srl"为后缀命名。如CA证书为"mycert.pem"，则默认寻找的序列号文件为"mycert.srl"
-CAcreateserial ：当使用该选项时，如果CA使用的序列号文件不存在将自动创建：该文件将包含序列号值"02"并且此次签名后证书文件序列号为1。
                  ：一般如果使用了"-CA"选项而序列号文件不存在将会产生错误"找不到srl文件"。

证书扩展选项：
-purpose：选项检查证书的扩展项并决定该证书允许用于哪些方面，即证书使用目的范围。

#例如，使用x509工具自建CA。由于x509无法建立证书请求文件，所以只能使用openssl req来生成请求文件，然后使用x509来自签署。自签署时，
#使用"-req"选项明确表示输入文件为证书请求文件，否则将默认以为是证书文件，再使用"-signkey"提供自签署时使用的私钥。
openssl req -new -keyout key.pem -out req.csr
openssl x509 -req -in req.csr -signkey key.pem -out x509.crt

# x509也可以用来签署他人的证书请求，即为他人颁发证书。注意，为他人颁发证书时，确保serial文件存在，建议使用自动创建的选项"-CAcreateserial"。
openssl x509 -req -in req.csr -CA ca.crt -CAkey ca.key -out x509.crt -CAcreateserial



```

## References



{% embed url="http://www.cnblogs.com/aixiaoxiaoyu/p/8650180.html" %}



