# Guia de integração para consulta de Qualificação Cadastral por Lote

A automação foi desenvolvida em cima do [portal oficial do eSocial](https://esociallote.dataprev.gov.br/EsocialLote/pages/index.xhtml) para consultas em lote.

A integração é composta pelo consumo de 2 endpoints e passa por 3 etapas:

- [Criptografia do Certificado Digital A1](#criptografia-do-certificado-digital-a1)

- [Envio do Lote](#envio-do-lote)

- [Retorno do Lote](#retorno-do-lote)

## Criptografia do Certificado Digital A1

O portal exige autenticação via Certificado Digital e, para a segurança do cliente, o mesmo deve ser trafegado de forma criptografada pelas requisições.

## Instruções de Criptografia

Para transmitir parâmetros encriptados é necessário usar o algoritmo AES com chave de 256 bits e modo CBC.

A **chave de criptografia** deve ser requerida a um representante da Exato Digital.

#### Exemplos de criptografia em diferentes linguagens de programação

> Exemplos baseados no projeto [AES Everywhere](https://github.com/mervick/aes-everywhere).



Shell

```shell
echo -n "EXEMPLO_DADO" | openssl enc -aes-256-cbc -md md5 -a -A -pass "pass:EXEMPLO_CHAVE"
```



C#

```C#
using AesEverywhere;

AES256 aes = new AES256();

string criptograma = aes.Encrypt("EXEMPLO_DADO", "EXEMPLO_CHAVE");
```



Java

```java
// Veja instruções de instalação do pacote aes-everywhere-java no link abaixo:
// https://github.com/mervick/aes-everywhere/tree/master/java

import com.github.mervick.aes_everywhere.Aes256;

String criptograma = Aes256.encrypt("EXEMPLO_DADO", "EXEMPLO_CHAVE");
```



Node.js

```js
// npm install aes-everywhere

const AES256  = require('aes-everywhere');

const criptograma = AES256.encrypt('EXEMPLO_DADO', 'EXEMPLO_CHAVE')
```



PHP

```php
// Veja instruções de instalação da biblioteca aes-everywhere no link abaixo:
// https://github.com/mervick/aes-everywhere/tree/master/php

$criptograma = \mervick\aesEverywhere\AES256::encrypt('EXEMPLO_DADO', 'EXEMPLO_CHAVE');

```



Python

```python
# pip install aes-everywhere

from AesEverywhere import aes256

criptograma = aes256.encrypt('EXEMPLO_DADO', 'EXEMPLO_CHAVE')
```



Ruby

```ruby
# gem install aes-everywhere

require 'aes-everywhere'

criptograma = AES256.encrypt('EXEMPLO_DADO', 'EXEMPLO_CHAVE')
```



Para trafegar o Certificado Digital e sua senha, siga os passos:

1. Realizar a leitura o arquivo binário do certificado (geralmente é um arquivo com extensão `.pfx`)
2. Converter o arquivo lido para `base64`
3. Encriptar a string `base64` obtida do arquivo de certificado utilizando **AES-256**, seguindo as instruções acima. A string gerada será usada nos campos com nome `pkcs12_certificate`.
4. Encriptar a senha do certificado utilizando **AES-256**, seguindo as instruções acima. A string gerada será usada nos campos com nome `pkcs12_pass`.
   
   

## Envio do Lote

**Endpoint:** POST https://api.exato.digital/previdencia/qualificacao-cadastral/bulk/upload?format=json&token={token}

É importante que seja armazenado o retorno do campo `FileName`, pois o valor dele será usado como identificador para recuperar a resposta do lote.



**Exemplo de corpo de requisição:**

```json
{
    "trabalhadores": [
        {
            "nome": "JESSICA CARVALHO LEMOS",
            "data_nascimento": "1989-09-25",
            "cpf": "12345678910",
            "nis": "32165498701"
        },
        {
            "nome": "GRAZIELLE DA SILVA SOUZA",
            "data_nascimento": "2005-01-12",
            "cpf": "09876543201",
            "nis": "89056723410"
        }
    ],
    "pkcs12_pass": "U2FsdGVkX18YyBE+xLqUY6H8X3HQkOGezAQtrDkgFhk=",
    "pkcs12_certificate": "U2FsdGVkX1/JdnH2hEFuzSgahseTvBcYxkLY7G2NhOITp4gxwDSSVhCogqtnBLkB0YJW+3FRS8OE5f7rEaZQtMJ7uM1zMyIg4AP4WPyMEFkExF5OFcTsY8Nvrn8g23adZoPFRZcqU+Ln2Ojunb6xsTuv5COaxsTKDUkxGEMHR0TThmEYIp8BSr+W0Q+pNeP+e8x/99AElQV17OBIvb7/w2Ggv/CbfKdL7beHCW9IyMM+HAzr0rIpr4JacMePdwBQx/3bR24qoWLt+yhDFp7TPznHAz64adigKMnhTNuTai1MfP7X2odFdKQdVn1lnxBArdKr9CkJ3ntHYNWS05TlGibOvXfArklv4KlDFRqjhJr0uRCNaOSn+v4wgZM28acwb/Q5+Q7LmyioVfH4CILgW1M5eFnSg8jWH9v6sB/eZsqjiPbAUe54hDpaY81N6Mxx5Z89kE42xmGdXfMdHf/5391SMLBXAE/ujU3wWry9QzFj3kN36GcB2mtLIG/EJWH6bwHFmMIeGkWtvZ5Pra78/usyaOqtU8C7KTLIecjUB5NkFzAkd5znFrx+nqRrm9HZQc41YnU/luEyl4b/PILePsCUEJKDZFGqvDb1sJGe19am7Oz1/BhF9Hhojr5kxApcPkOwbazLNZOMt2wzb7bUpxF1uVb75UmsOS2YX7wgRdJ4oOdMNg1cg5w+Hzwz9yqrpIGegMpA0VT0Z61aGha32YK/tUwg1uchVfOguUECTQsLJAcLCbgZm251yiizx9KC3WqwBOs2v/9qveubIyJs9040UscmpzR4YtGKHUMgXU+fOowpT7kaxLP6du/InMYauAn+NDCtsx1W6J6iwmFxoK4ulxvBFyN1Tb0s89lpYd1vvFPOKOQ+ViBH7+lSW/PpBqVBwvLOnzJ02uyES6rRktlmM161os3y1hnwuaExuwedTuw1r2CEDIzDIJyH/M5Z4Y6Z/hyzCa5szgtIyDoF0hY7sY5KokqccU1RsgLdfidbQhIMj/dyp1ZLumP08SNr5EM78O8m/9OuEkR1EbBI1UaE20PH8SYFSdCdb8UqpJZLSQnrFqwTH4OHgbSIFIA3vI8FUpIz1G6zqxcmf9vVzXcPT5uN..."
}
```



**Exemplo da chamada via cURL:**

```bash
curl --location 'https://api.exato.digital/previdencia/qualificacao-cadastral/bulk/upload?format=json&token={token}' --header 'Content-Type: application/json' --header 'Cookie: ss-pid=vZdX7LSTiMQfhsExGDIO' --data '{"trabalhadores":[{"nome":"JESSICA CARVALHO LEMOS","data_nascimento":"1989-09-25","cpf":"12345678910","nis":"32165498701"},{"nome":"GRAZIELLE DA SILVA SOUZA","data_nascimento":"2005-01-12","cpf":"09876543201","nis":"89056723410"}],"pkcs12_pass":"U2FsdGVkX18YyBE+xLqUY6H8X3HQkOGezAQtrDkgFhk=","pkcs12_certificate":"U2FsdGVkX1/JdnH2hEFuzSgahseTvBcYxkLY7G2NhOITp4gxwDSSVhCogqtnBLkB0YJW+3FRS8OE5f7rEaZQtMJ7uM1zMyIg4AP4WPyMEFkExF5OFcTsY8Nvrn8g23adZoPFRZcqU+Ln2Ojunb6xsTuv5COaxsTKDUkxGEMHR0TThmEYIp8BSr+W0Q+pNeP+e8x/99AElQV17OBIvb7/w2Ggv/CbfKdL7beHCW9IyMM+HAzr0rIpr4JacMePdwBQx/3bR24qoWLt+yhDFp7TPznHAz64adigKMnhTNuTai1MfP7X2odFdKQdVn1lnxBArdKr9CkJ3ntHYNWS05TlGibOvXfArklv4KlDFRqjhJr0uRCNaOSn+v4wgZM28acwb/Q5+Q7LmyioVfH4CILgW1M5eFnSg8jWH9v6sB/eZsqjiPbAUe54hDpaY81N6Mxx5Z89kE42xmGdXfMdHf/5391SMLBXAE/ujU3wWry9QzFj3kN36GcB2mtLIG/EJWH6bwHFmMIeGkWtvZ5Pra78/usyaOqtU8C7KTLIecjUB5NkFzAkd5znFrx+nqRrm9HZQc41YnU/luEyl4b/PILePsCUEJKDZFGqvDb1sJGe19am7Oz1/BhF9Hhojr5kxApcPkOwbazLNZOMt2wzb7bUpxF1uVb75UmsOS2YX7wgRdJ4oOdMNg1cg5w+Hzwz9yqrpIGegMpA0VT0Z61aGha32YK/tUwg1uchVfOguUECTQsLJAcLCbgZm251yiizx9KC3WqwBOs2v/9qveubIyJs9040UscmpzR4YtGKHUMgXU+fOowpT7kaxLP6du/InMYauAn+NDCtsx1W6J6iwmFxoK4ulxvBFyN1Tb0s89lpYd1vvFPOKOQ+ViBH7+lSW/PpBqVBwvLOnzJ02uyES6rRktlmM161os3y1hnwuaExuwedTuw1r2CEDIzDIJyH/M5Z4Y6Z/hyzCa5szgtIyDoF0hY7sY5KokqccU1RsgLdfidbQhIMj/dyp1ZLumP08SNr5EM78O8m/9OuEkR1EbBI1UaE20PH8SYFSdCdb8UqpJZLSQnrFqwTH4OHgbSIFIA3vI8FUpIz1G6zqxcmf9vVzXcPT5uN..."}'
```



**Exemplo do retorno da requisição:**

```json
{
    "UniqueIdentifier": "9kulgzu6w8sl9g02bp5qyxi98",
    "TransactionResultTypeCode": 1,
    "TransactionResultType": "Success",
    "Message": "Sucesso. Arquivo D.CNS.CPF.001.20250225130126.12387530000113.12387530000113.TXT enviado com sucesso no dia 25/02/2025 às 13:01:27 pelo empregador 12387530000113. Este arquivo será processado em até 48h.",
    "TotalCostInCredits": 0,
    "BalanceInCredits": 0,
    "ElapsedTimeInMilliseconds": 7896,
    "Reserved": null,
    "Date": "2025-02-25T13:01:22.9214798-03:00",
    "OutdatedResult": false,
    "HasPdf": false,
    "ValidationResultRiskIndicator": "undefined",
    "ValidationResults": [],
    "DataSourceHtml": null,
    "DateString": "2025-02-25T13:01:22.9214798-03:00",
    "OriginalFilesUrl": "http://localhost/services/original-files/9kulgzu6w8sl9g02bp5qyxi98",
    "PdfUrl": null,
    "ResultHtml": null,
    "TotalCost": 0,
    "ValidationResultRiskIndicatorN": "Undefined",
    "ValidationResultsN": [],
    "BalanceInBrl": null,
    "ResultHtmlHeader": null,
    "ResultHtmlDataSourceRiskIndicator": null,
    "DataSourceCategory": "Sem categoria",
    "Result": {
        "FileName": "D.CNS.CPF.001.20250225130126.12387530000113.12387530000113.TXT"
    }
}
```



## Retorno do Lote

**Endpoint:** POST https://api.exato.digital/previdencia/qualificacao-cadastral/bulk/download?format=json&token={{token}}

É recomendado que utilize esse endpoint com a técnica de _short polling_, pois a fonte oficial se compromete a processar o lote em até 48h. Portanto é necessário verificar periodicamente se o resultado já está disponível.



**Exemplo de corpo de requisição:**

```json
{
    "nome_arquivo": "D.CNS.CPF.001.20250225130126.123875",
    "pkcs12_pass": "U2FsdGVkX18YyBE+xLqUY6H8X3HQkOGezAQtrDkgFhk=",
    "pkcs12_certificate": "U2FsdGVkX1/JdnH2hEFuzSgahseTvBcYxkLY7G2NhOITp4gxwDSSVhCogqtnBLkB0YJW+3FRS8OE5f7rEaZQtMJ7uM1zMyIg4AP4WPyMEFkExF5OFcTsY8Nvrn8g23adZoPFRZcqU+Ln2Ojunb6xsTuv5COaxsTKDUkxGEMHR0TThmEYIp8BSr+W0Q+pNeP+e8x/99AElQV17OBIvb7/w2Ggv/CbfKdL7beHCW9IyMM+HAzr0rIpr4JacMePdwBQx/3bR24qoWLt+yhDFp7TPznHAz64adigKMnhTNuTai1MfP7X2odFdKQdVn1lnxBArdKr9CkJ3ntHYNWS05TlGibOvXfArklv4KlDFRqjhJr0uRCNaOSn+v4wgZM28acwb/Q5+Q7LmyioVfH4CILgW1M5eFnSg8jWH9v6sB/eZsqjiPbAUe54hDpaY81N6Mxx5Z89kE42xmGdXfMdHf/5391SMLBXAE/ujU3wWry9QzFj3kN36GcB2mtLIG/EJWH6bwHFmMIeGkWtvZ5Pra78/usyaOqtU8C7KTLIecjUB5NkFzAkd5znFrx+nqRrm9HZQc41YnU/luEyl4b/PILePsCUEJKDZFGqvDb1sJGe19am7Oz1/BhF9Hhojr5kxApcPkOwbazLNZOMt2wzb7bUpxF1uVb75UmsOS2YX7wgRdJ4oOdMNg1cg5w+Hzwz9yqrpIGegMpA0VT0Z61aGha32YK/tUwg1uchVfOguUECTQsLJAcLCbgZm251yiizx9KC3WqwBOs2v/9qveubIyJs9040UscmpzR4YtGKHUMgXU+fOowpT7kaxLP6du/InMYauAn+NDCtsx1W6J6iwmFxoK4ulxvBFyN1Tb0s89lpYd1vvFPOKOQ+ViBH7+lSW/PpBqVBwvLOnzJ02uyES6rRktlmM161os3y1hnwuaExuwedTuw1r2CEDIzDIJyH/M5Z4Y6Z/hyzCa5szgtIyDoF0hY7sY5KokqccU1RsgLdfidbQhIMj/dyp1ZLumP08SNr5EM78O8m/9OuEkR1EbBI1UaE20PH8SYFSdCdb8UqpJZLSQnrFqwTH4OHgbSIFIA3vI8FUpIz1G6zqxcmf9vVzXcPT5uN..."
}
```



**Exemplo da chamada via cURL:**

```bash
curl --location 'https://api.exato.digital//previdencia/qualificacao-cadastral/bulk/download?format=json&token={token}' --header 'Content-Type: application/json' --header 'Cookie: ss-pid=4BiEgFudQ0BOYiUMGsc5' --data '{"nome_arquivo":"D.CNS.CPF.001.20250225130126.123875","pkcs12_pass":"U2FsdGVkX18YyBE+xLqUY6H8X3HQkOGezAQtrDkgFhk=","pkcs12_certificate":"U2FsdGVkX1/JdnH2hEFuzSgahseTvBcYxkLY7G2NhOITp4gxwDSSVhCogqtnBLkB0YJW+3FRS8OE5f7rEaZQtMJ7uM1zMyIg4AP4WPyMEFkExF5OFcTsY8Nvrn8g23adZoPFRZcqU+Ln2Ojunb6xsTuv5COaxsTKDUkxGEMHR0TThmEYIp8BSr+W0Q+pNeP+e8x/99AElQV17OBIvb7/w2Ggv/CbfKdL7beHCW9IyMM+HAzr0rIpr4JacMePdwBQx/3bR24qoWLt+yhDFp7TPznHAz64adigKMnhTNuTai1MfP7X2odFdKQdVn1lnxBArdKr9CkJ3ntHYNWS05TlGibOvXfArklv4KlDFRqjhJr0uRCNaOSn+v4wgZM28acwb/Q5+Q7LmyioVfH4CILgW1M5eFnSg8jWH9v6sB/eZsqjiPbAUe54hDpaY81N6Mxx5Z89kE42xmGdXfMdHf/5391SMLBXAE/ujU3wWry9QzFj3kN36GcB2mtLIG/EJWH6bwHFmMIeGkWtvZ5Pra78/usyaOqtU8C7KTLIecjUB5NkFzAkd5znFrx+nqRrm9HZQc41YnU/luEyl4b/PILePsCUEJKDZFGqvDb1sJGe19am7Oz1/BhF9Hhojr5kxApcPkOwbazLNZOMt2wzb7bUpxF1uVb75UmsOS2YX7wgRdJ4oOdMNg1cg5w+Hzwz9yqrpIGegMpA0VT0Z61aGha32YK/tUwg1uchVfOguUECTQsLJAcLCbgZm251yiizx9KC3WqwBOs2v/9qveubIyJs9040UscmpzR4YtGKHUMgXU+fOowpT7kaxLP6du/InMYauAn+NDCtsx1W6J6iwmFxoK4ulxvBFyN1Tb0s89lpYd1vvFPOKOQ+ViBH7+lSW/PpBqVBwvLOnzJ02uyES6rRktlmM161os3y1hnwuaExuwedTuw1r2CEDIzDIJyH/M5Z4Y6Z/hyzCa5szgtIyDoF0hY7sY5KokqccU1RsgLdfidbQhIMj/dyp1ZLumP08SNr5EM78O8m/9OuEkR1EbBI1UaE20PH8SYFSdCdb8UqpJZLSQnrFqwTH4OHgbSIFIA3vI8FUpIz1G6zqxcmf9vVzXcPT5uN..."}'
```



**Exemplo do retorno da requisição (lote em processamento):**

```json
{
    "UniqueIdentifier": "2goo1393n13ismbdet9r7hw77",
    "TransactionResultTypeCode": 5,
    "TransactionResultType": "EntityNotFound",
    "Message": "Documento/entidade inexistente ou não encontrada. Documentos ainda não processados pela fonte, tente novamente mais tarde!",
    "TotalCostInCredits": 0,
    "BalanceInCredits": 0,
    "ElapsedTimeInMilliseconds": 4662,
    "Reserved": null,
    "Date": "2025-02-25T14:18:24.3948418-03:00",
    "OutdatedResult": false,
    "HasPdf": false,
    "ValidationResultRiskIndicator": "undefined",
    "ValidationResults": [],
    "DataSourceHtml": null,
    "DateString": "2025-02-25T14:18:24.3948418-03:00",
    "OriginalFilesUrl": "http://localhost/services/original-files/2goo1393n13ismbdet9r7hw77",
    "PdfUrl": null,
    "ResultHtml": null,
    "TotalCost": 0,
    "ValidationResultRiskIndicatorN": "Undefined",
    "ValidationResultsN": [],
    "BalanceInBrl": null,
    "ResultHtmlHeader": null,
    "ResultHtmlDataSourceRiskIndicator": null,
    "DataSourceCategory": "Sem categoria",
    "Result": null
}
```



**Exemplo do retorno da requisição (lote finalizado):**

```json
{
    "UniqueIdentifier": "bq79dep0kwht18zunkzy1oceo",
    "TransactionResultTypeCode": 2,
    "TransactionResultType": "SuccessWithRemarks",
    "Message": "Sucesso com ressalvas",
    "TotalCostInCredits": 0,
    "BalanceInCredits": 0,
    "ElapsedTimeInMilliseconds": 3178,
    "Reserved": null,
    "Date": "2025-02-25T14:30:20.6429178-03:00",
    "OutdatedResult": false,
    "HasPdf": false,
    "ValidationResultRiskIndicator": "undefined",
    "ValidationResults": [],
    "DataSourceHtml": null,
    "DateString": "2025-02-25T14:30:20.6429178-03:00",
    "OriginalFilesUrl": "http://localhost/services/original-files/bq79dep0kwht18zunkzy1oceo",
    "PdfUrl": null,
    "ResultHtml": null,
    "TotalCost": 0,
    "ValidationResultRiskIndicatorN": "Undefined",
    "ValidationResultsN": [],
    "BalanceInBrl": null,
    "ResultHtmlHeader": null,
    "ResultHtmlDataSourceRiskIndicator": null,
    "DataSourceCategory": "Sem categoria",
    "Result": {
        "Retornos": [
            {
                "NomeInformado": "JESSICA CARVALHO LEMOS",
                "DataNascimentoInformada": "1989-09-25T00:00:00.0000000",
                "CpfInformado": "12345678910",
                "NisInformado": "32165498701",
                "Mensagem": "JESSICA CARVALHO LEMOS SOUZA",
                "Orientação": "Procurar Conveniadas da RFB: Correios, Banco do Brasil ou CAIXA.",
                "DadosCorretos": false,
                "CpfEncontrado": true,
                "NisEncontrado": true,
                "NomeInconsistente": true,
                "DataNascimentoInconsistente": false,
                "CpfInconsistente": false
            },
            {
                "NomeInformado": "GRAZIELLE DA SILVA SOUZA",
                "DataNascimentoInformada": "2005-01-12T00:00:00.0000000",
                "CpfInformado": "09876543201",
                "NisInformado": "89056723410",
                "Mensagem": "",
                "Orientação": "",
                "DadosCorretos": true,
                "CpfEncontrado": true,
                "NisEncontrado": true,
                "NomeInconsistente": false,
                "DataNascimentoInconsistente": false,
                "CpfInconsistente": false
            }
        ]
    }
}
```
