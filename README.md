
## **Sumário**

[Introdução](#introdução)

[Objetivo](#objetivo)

[Público-Alvo](#público-alvo)

[Requisitos mínimos](#requisitos-mínimos)

[m-SiTef](#m-sitef)

[Integração utilizando TLS](#integração-utilizando-tls)

[TLS Software Express](#tls-software-express)

[Integração TLS WNB Comnect](#integração-tls-wnb-comnect)

[Integração TLS Gsurf](#integração-tls-gsurf)

[Novo TLS Software Express (TLSGWP)](#Novo_TLS_Software_Express_TLSGWP)

[Integração com outro app](#integração-com-outro-app)

[Iniciando o m-SiTef através de outro app](#iniciando-o-m-sitef-através-de-outro-app)

[Exemplos de chamada do m-SiTef por outro app](#exemplos-de-chamada-do-m-sitef-por-outro-app)

[Exemplo de chamada de Pix](#exemplo-de-chamada-de-pix)

[Resposta do m-SiTef para outro app](#resposta-do-m-sitef-para-outro-app)

[Exemplo de tratamento do retorno do m-SiTef](#exemplo-de-tratamento-do-retorno-do-m-sitef)

[Mudando o CLSIT](#mudando-o-clsit)

[Utilizando o AndroidX](#utilizando-o-androidx)

[Histórico de Alterações](#histórico-de-alterações)

## Introdução

O m-SiTef é um aplicativo desenvolvido pela Software Express para a
plataforma Android, tanto em celular ou tablet como em POS, que realiza
transações (TEF) através do servidor SiTef.

Este aplicativo contém a biblioteca CliSiTef e possibilita ao integrador
a facilidade de apenas interagir com as funcionalidades da CliSiTef, sem
a necessidade de controlar o fluxo das transações.

## Objetivo

Ser um guia de referência para integração com o m-SiTef.

## Público-Alvo

Parceiros que desejam se integrar com o m-SiTef.

## Requisitos mínimos

O m-SiTef é compatível com o Android versão 4.4 ou superior.

## m-SiTef

Antes de apresentar as formas de integração, descreveremos neste
capítulo as características gerais do m-SiTef.

Ele disponibiliza para o integrador as mesmas funcionalidades da
CliSiTef que estão descritas no documento "SiTef - Interface
Simplificada com a aplicação" na Tabela de códigos de funções.

Para executar as funções, é obrigatório enviar 4 parâmetros ao m-SiTef:
**empresaSitef**, **enderecoSitef**, **modalidade** e
**CNPJ_CPF**. Para a modalidade 0, também é obrigatório o parâmetro
**valor**.

**Importante:** Todos esses parâmetros apresentados neste documento em
trechos de códigos são apenas ilustrativos e devem ser substituídos por
valores válidos.

A seguir, estão listados todos os parâmetros aceitos na aplicação.
<br/>

### **Tabela 1 - parâmetros de entrada do m-SiTef**

 **Parâmetro**                 | **Tipo**                           | **Descrição**
-------------------------------|------------------------------------|-----------------------------------------------
empresaSitef                   | Obrigatório                        | Empresa SiTef. <br/> O tamanho é de 8 dígitos alfanuméricos.
enderecoSitef                  | Obrigatório                        | Endereço dos servidores do SiTef. <br/> O campo pode ser composto de 1 a 3 endereços separados por ‘;’. <br/> O formato do endereço pode ser: <br/><br/>1. IP, ```IP:PORTA```; <br/>2. Nome do servidor, ```NOME:PORTA```. <br/><br/>Quando a porta não é especificada, é assumida a porta padrão (4096). <br/>Não informar URL nesse campo.
terminalSitef                  | Opcional                           | Número de terminal SiTef.<br/> Se não informado o m-SiTef irá usar o número de série do APOS ou o UUID do aparelho Android
modalidade                     | Obrigatório                        | Funcionalidade da CliSiTef que deseja executar. Por exemplo:<br/>0 – Pagamento<br/>200 – Cancelamento<br/>114 – Reimpressão<br/>A lista completa pode ser encontrada no documento “SiTef - Interface Simplificada com a aplicação” na Tabela de códigos de funções.
CNPJ\_CPF                      | Obrigatório                        | CNPJ ou CPF do estabelecimento.<br/>Este campo não deve conter caracteres especiais e pode ser utilizado nos dados de subadquirência e no ParmsClient.
valor                          | Obrigatório p/ pagamento           | Valor da venda.<br/>O campo é numérico com tamanho até 12 algarismos, onde os dois últimos dígitos são decimais.
restricoes                     | Opcional                           | Opções de pagamento que não aparecerão no fluxo de coleta. O campo deve ter o seguinte formato:<br/>```<Opção>```;```<Opção>```; ....<br/>Os valores das opções estão no documento “SiTef - Interface Simplificada com a aplicação”, na Tabela de códigos de funções. <br/> Observação: a opção ```TransacoesHabilitadas=T1,T2``` também pode ser passada nesse parametro e terá preferência sobre o parâmetro TransacoesHabilitadas
operador                       | Opcional                           | Código do operador.<br/>O campo é alfanumérico com tamanho até 20 caracteres.
Data                           | Obrigatório                           | Data fiscal no formato AAAAMMDD.
Hora                           | Obrigatório                           | Hora fiscal no formato HHMMSS.
numeroCupom                    | Obrigatório                           | Número do cupom fiscal correspondente à venda.<br/>O campo é alfanumérico com tamanho até 20 caracteres.
numParcelas                    | Opcional                           | Número de parcelas, em caso de compra parcelada. O campo é numérico e varia de acordo com o cartão utilizado.
Otp                            | Opcional                           | Código obrigatório quando é utilizada comunicação com TLS GSurf.
transacoesHabilitadas          | Opcional                           | Opções de pagamento que serão habilitadas.<br/> O campo deve ter o seguinte formato {```<Funcionalidade1>```;```<Funcionalidade2>```;...}. Os valores das opções estão no documento “SiTef - Interface Simplificada com a aplicação”, na Tabela de códigos de funções.
pinpadMac                      | Opcional                           | Endereço MAC Address Bluetooth do Pinpad para o m-SiTef se conectar diretamente com o pinpad. Caso esse parâmetro não seja passado, o m-SiTef apresenta uma lista de dispositivos Bluetooth pareados com o aparelho para o usuário selecionar qual utilizar.<br/>Formato: 00:00:00:00:00:00
comExterna                     | Obrigatório                        | Campo para definir qual serviço TLS será usado:<br/>0 –   Sem (apenas para SiTef dedicado)<br/>1 – TLS Software Express<br/>2 – TLS WNB Comnect<br/>3 – TLS Gsurf<br/>4 - TLS GWP (TLS Fiserv)
isDoubleValidation             | Obrigatório p/TLS Software Express | Campo para definir qual tipo de validação será usado:<br/>0 – Para validação simples<br/>1 – Para validação dupla
cnpj\_automacao                | Obrigatório                           | CNPJ da empresa que desenvolveu a automação comercial.<br/>Dado utilizado no ParmsClient.
cnpj\_facilitador              | Obrigatório                           | CNPJ do Facilitador (Van).<br/>Dado utilizado no ParmsClient.
timeoutColeta                  | Opcional                           | Define o tempo de timeout para coletas e interações do fluxo da transação.<br/>O valor deve ser preenchido em “segundos” e se não for preenchido, o valor padrão é de 60 segundos. Caso seja definido como 0 ou números negativos, o timeout de coleta será inativado.<br/>O timeout é aplicado se o usuário não interagir com a coleta pelo número de segundos especificado. Nesse caso, a transação é automaticamente cancelada e a aplicação recebe o valor RESULT\_CANCELED no parâmetro resultCode.<br/>Este timeout não se aplica à leitura de QRCode, à coleta do PIN e a retirada do cartão da leitora.
acessibilidadeVisual           | Opcional                           | Campo para definir se a acessibilidade visual deve ser habilitada:<br/>0 – Para desabilitar (valor padrão)<br/>1 – Para habilitar<br/><br/>A habilitação da acessibilidade visual consiste em algumas modificações visuais e sonoras. Caso este campo seja habilitado, o m-SiTef terá as seguintes alterações:<br/>·       Fontes maiores para melhor visualização<br/>·       Alto contraste nas cores para melhor visualização em caso de daltonismo<br/>·       Ativação do text-to-speech (leitura de tela) durante todo o fluxo<br/>·       Se for o caso, ativa a adaptação da tela para compatibilidade com a capa de acessibilidade na inserção da senha ou pin (aplicável em modelos pré-definidos).<br/><br/>Observação: para que a leitura de tela seja realizada pelo módulo de text-to-speech, o sistema operacional precisa ter previamente instalado o pacote da língua correspondente.<br/>No caso do GPOS700, o pacote de linguagem PT-BR será disponibilizado pela Gertec.<br/>
tipoPinpad                     | Obrigatório p/ dispositivos USB.   | ANDROID\_USB – Tenta obter conexão apenas com pinpad´s USB.<br/><br/>ANDROID\_BT – Tenta obter conexão apenas com pinpad´s Bluetooth.
dadosSubAdqui                  | Opcional                           | Conjunto de informações complementares que permite que o lojista personalize o que será impresso na fatura do comprador.<br/>Formato:<br/>```<id1><tam1><val1><id2><tam2></tam2><val2>...<idn><tamn></tamn><valn>```<br/>Onde:<br/>Id - identificador do campo com 2 bytes, conforme valores definidos na tabela da especificação ”**Informações de Sub-adquirência (Soft Descriptor) Bibliotecas CliSiTefI e CliSiTef**”<br/>tam - tamanho do campo com 2 bytes.<br/>Val – valor do campo.<br/>Exemplo de envio:<br/>_i.putExtra("dadosSubAdqui", "0022STL*SOFTWARE EXPRESS");_<br/>Para mais informações referente aos dados consultar a especificação: “**Informações de Sub-adquirência (Soft Descriptor) Bibliotecas CliSiTefI e CliSiTef**”
tipoCampos                     | Opcional                           | Permite informar valores pré-determinados para os campos que a CliSiTef solicita para a aplicação durante o fluxo da transação. Os campos e seus respectivos valores devem ser informados no formato JSON abaixo. Os valores sempre deverão ser informados com o valor do tipo String. E é importante observar que nesse formato, é possível definir apenas um valor por tipo de campo. <br/><br/>Formato JSON:<br/><br/> ```{"campo1":"val1", "campo2":"val2"..."campon":"valn"}```<br/><br/>Exemplos de envio: <br/> ```i.putExtra("tipoCampos","{\"147\":\"8000\", \"516\":\"99900021\"}"); ```<br/><br/>  ```i.putExtra("tipoCampos","{\"1025":\"Descricao do Produto\"}");```<br/>
habilitaColetaTaxaEmbarqueIATA | Opcional                           | Quando habilitado, indica que se trata de uma transação de venda a crédito IATA e será disponibilizado campo para coleta do valor da taxa de embarque.<p>0 – Para desabilitar (valor padrão) <br/>1 – Para habilitar</p>
habilitaColetaValorEntradaIATA | Opcional                           | <p>Quando habilitado, indica a coleta do valor de entrada em caso de parcelamento da venda a crédito IATA. Esta coleta será permitida apenas para venda IATA, isto é, a habilitação deste item só terá efeito se habilitaColetaTaxaEmbarqueIATA=1.<br/> 0 – Para desabilitar (valor padrão)<br/> 1 – Para habilitar </p>
clsit                          | Opcional                           | Para poder adicionar, atualizar e remover campos  do arquivo de configuração CLSIT do pacote. Olhar seção [Mudando o CLSIT](#mudando-o-clsit).


## Integração utilizando TLS
<br/>
O m-SiTef é compatível com 4 provedores TLS: Software Express, GSurf, WNB Comnect e TLSGWP.
Abaixo, será explicado como utilizar cada um deles.<br/>

## TLS Software Express
<br/>

```java
/**
Deve ser enviado o valor 1 com o parâmetro “comExterna” 
e o valor 0 com o parâmetro “isDoubleValidation”:

**Importante:** O endereço SiTef para esse caso será uma
 URL de domínio correspondente com o que está configurado
  no certificado.
**/
Intent i = new  Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
i.putExtra("comExterna", "1");
startActivityForResult(i,1234);
```

## Integração TLS WNB Comnect
<br/>

```java
//Deve ser enviado o valor 2 com o parâmetro “comExterna”:
Intent i = new  Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
i.putExtra("comExterna", "2");
startActivityForResult(i,1234);
```

## Integração TLS Gsurf
<br/>

```java
//Deve ser enviado o valor 3 com o parâmetro “comExterna” e um valor de otp, necessário solicitar com a Gsurf.
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
i.putExtra("comExterna", "3");
i.putExtra("otp", "1234567");
startActivityForResult(i,1234);
```

## Novo TLS Software Express (TLSGWP)
<br/>

```java
/**
Deve ser enviado o valor 4 no parâmetro “comExterna” 
e o token de registro no parâmetro “tokenRegistroTls”:

**Importante:** O endereço SiTef para esse caso será um IP ou
 URL de domínio correspondente com o token gerado.
**/
Intent i = new  Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "tls-prod.fiservapp.com:443");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("tokenRegistroTls", "0000-0000-0000-0000"); //substituir por um token válido
i.putExtra("comExterna", "4");
startActivityForResult(i,1234);
```

Alternativamente, o registro de terminal para usar o novo TLSGWP pode ser realizado através das modalidades 699 (Registro de terminal) ou 110 (menu administrativo). Neste caso, a CliSiTef irá abrir uma coleta para o token ser informado e a partir desse momento o terminal estará registrado e deverá informar nas próximas transações que o tipo de conexão será o TLSGWP informando o parâmetro comExterna = 4 como no exemplo a seguir.

### Registro Manual (TLSGWP)

```java
//primeiro deve ser realizado o registro
Intent i = new  Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "tls-prod.fiservapp.com:443");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("modalidade", "699"); //Ou 110
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
startActivityForResult(i,1234);

// Nas próximas transações deve ser informado o tipo de conexão TLSGWP (comExterna = 4)

Intent i = new  Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "tls-prod.fiservapp.com:443");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("comExterna", "4");
startActivityForResult(i,1234);
```

## Integração com outro app

### Iniciando o m-SiTef através de outro app

O primeiro passo é instanciar um objeto Intent passando o nome da aplicação como argumento – neste caso, o nome é br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF. Através desta informação, o Android buscará automaticamente o m-SiTef entre os aplicativos instalados no dispositivo móvel.
Em seguida, devem ser configurados quatro parâmetros obrigatórios através da função putExtra(String, String): empresaSitef, enderecoSitef, CNPJ_CPF e modalidade. Esses e outros parâmetros serão detalhados no item 5.
Por fim, é executada a função nativa do Android startActivityForResult(Intent, int) passando como parâmetros o objeto Intent e um número inteiro arbitrário. Este número será utilizado como um ID para a recuperação de informações que o m-SiTef enviará, após encerrar o seu processamento, ao aplicativo que o acionou. Neste documento, utilizaremos o ID = 1234.
<br/>

```java
// O exemplo abaixo mostra uma chamada simples do fluxo de venda.
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
startActivityForResult(i,1234); // ID = 1234
```
### Exemplos de chamada do m-SiTef por outro app
a)	Pagamento

```java
//Para efetuar o pagamento, o aplicativo do integrador 
//pode fazer a seguinte chamada:

Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
startActivityForResult(i,1234);
```

Ao ser acionado, o m-SiTef pode apresentar as seguintes telas:

![](./media/image11.png)

b)	Débito à vista

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("modalidade", "2");
i.putExtra("valor", "9000");
i.putExtra("restricoes", "TransacoesHabilitadas=16");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
startActivityForResult(i,1234);
```
c)	Crédito à vista

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "1");
i.putExtra("modalidade", "3");
i.putExtra("valor", "9000");
i.putExtra("restricoes", "TransacoesHabilitadas=26");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
startActivityForResult(i,1234);
```

d)	Crédito parcelado

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "3");
i.putExtra("valor", "9000");
i.putExtra("restricoes", "TransacoesHabilitadas=27");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");
startActivityForResult(i,1234);
```

e)	Cancelamento

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("modalidade", "200");
i.putExtra("CNPJ_CPF", "12345678912345");
startActivityForResult(i,1234);
```

f)	Reimpressão

>**Atenção**: A impressão de cupons **não é realizada pelo m-SiTef**. O serviço de reimpressão realiza uma consulta aos cupons previamente salvos no SiTef. As informações coletadas nesta consulta são passadas no retorno da Intent, conforme no exemplo abaixo. A impressão dos cupons retornados pela Intent fica sob responsabiliade da aplicação que realizou a chamada ao m-SiTef.

```java
    // Exemplo de código de chamada do m-SiTef via Intent.
    intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
    i.putExtra("empresaSitef", "Empresa SiTef");
    i.putExtra("enderecoSitef", "Endereço SiTef");
    i.putExtra("modalidade", "114");
    i.putExtra("transacoesHabilitadas", "725");
    startPagamentoIntents.lauch(i);


    // Exemplo de código para obter os informaçõs dos cupons para que a automação realize a impressão.
    // Os principais campos de retorno das funções 725 e 726 são: 
    //   - VIA_ESTABELECIMENTO: String com as informações da via do estabelecimento
    //   - VIA_CLIENTE: String com as informações da via do cliente
    ActivityResultLauncher<Intent> startPagamentoIntent = registerForActivityResult(
        new ActivityResultContracts.StartActivityForResult(), result -> {
            if (result.getResultCode() == RESULT_OK) {
            Intent data = result.getData();
            log.d("msitef", "CODRESP: " + data.getExtras().getString("CODRESP"));
            log.d("msitef", "VIA_ESTABELECIMENTO: " + data.getExtras().getString("VIA_ESTABELECIMENTO"));
            log.d("msitef", "VIA_CLIENTE: " + data.getExtras().getString("VIA_CLIENTE"));
            }
        });
```

g)	Menu administrativo

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("modalidade", "110");
i.putExtra("CNPJ_CPF", "12345678912345");
startActivityForResult(i,1234);
```

## Exemplo de chamada de Pix 

Para realizar o Pix a automação deve seguir o exemplo abaixo, escolhendo o menu de carteiras digitais. Importante: o campo “cnpj_automacao” é de extrema importância que seja enviado, pois é através dele que a Software Express será capaz de reconhecer o estabelecimento e assim fazer o repasse financeiro.
<br/>

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("modalidade", "0");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("cnpj_automacao", "12345678912345");
i.putExtra("transacoesHabilitadas", "7;8;");
startActivityForResult(i,1234);
```

Se desejar, a automação pode chamar o Pix diretamente [restringindo as opções de carteiras digitais](https://dev.softwareexpress.com.br/docs/clisitef-carteiras-digitais/habilitando_desabilitando_opcoes_dos_menus_de_carteiras_digitais/) e passando a modalidade 122 (Carteiras digitais), como mostra exemplo a seguir.

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("modalidade", "122");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("cnpj_automacao", "12345678912345");
i.putExtra("transacoesHabilitadas", "7;8;");
i.putExtra("restricoes", "CarteirasDigitaisHabilitadas=027160110024");
startActivityForResult(i,1234);
```

![](./media/image12.png)

## Resposta do m-SiTef para outro app

### Resposta do m-SiTef para outro app
<br/>
Após a execução do m-SiTef, o fluxo é retornado ao aplicativo que o acionou e são devolvidos alguns parâmetros. Através deles, o aplicativo conseguirá saber se o processamento da transação ocorreu com êxito e além de obter outras informações.
Para receber estes dados, a aplicação deve implementar a função protected void onActivityResult( int requestCode, int resultCode, Intent data), que é executada pois o método startActivityForResult(Intent i, int requestCode) foi chamado para iniciar o m-SiTef. 
O argumento requestCode é usado para verificar a correspondência da chamada ao m-SiTef com o seu retorno. O resultCode mostra o status da execução do fluxo do m-SiTef (os valores pertencem ao Android RESULT_OK (-1) RESULT_CANCELED (0)). O argumento data é utilizado para recuperar os dados enviados pelo m-SiTef.
A implementação do onActivityResult deve conter uma condição comparando se o requestCode é igual ao número passado no startActivityForResult e, dentro desta condição, é verificado se o m-SiTef foi executado com êxito e são recuperados os parâmetros enviados por ele através da função data.getExtras().getString().

O aplicativo integrado ao m-SiTef pode tratar apenas os parâmetros que precisar.
<br/>

### **Tabela 2 - parâmetros de saída do m-SiTef**
 **Parâmetro**       | **Descrição**
---------------------|--------------------------------
CODRESP              | Código de resposta da transação realizada com o SiTef.<br/>Veja a tabela a seguir para maiores detalhes<br/>
COMP\_DADOS\_CONF    | Dados que deverão ser passados para a CliSiTef, a fim de realizar a confirmação da transação realizada.<br/>
CODTRANS             | Indica código da transação realizada. Possíveis valores:<br/>·       ‘00’: Consulta Cheque<br/>·       ‘01’: Cartão Débito<br/>·       ‘02’: Cartão Crédito
TIPO\_PARC           | Valores possíveis:<br/>·       “00”: À vista<br/>·       “01”: Pré-Datado<br/>·       “02”: Parcelado Estabelecimento<br/>·       “03”: Parcelado Administradora
VLTROCO              | Contém o valor aprovado para o troco em dinheiro. (Utilizado em compra com Saque disponível para algumas Redes).
REDE\_AUT            | Campo contendo a rede autorizadora da transação TEF realizada.<br/>Os códigos estão no documento "Especificação Técnica – Interface com os meios de pagamento do SiTef", na Tabela de Código das Redes Autorizadoras.
BANDEIRA             | Campo contendo a bandeira da transação TEF realizada. Os códigos estão no documento "Especificação Técnica – Interface com os meios de pagamento do SiTef", na Tabela de Código da Bandeira.
NSU\_SITEF           | NSU do servidor SiTef.
NSU\_HOST            | NSU do Host Autorizador.
COD\_AUTORIZACAO     | Código de autorização da transação de crédito. (Presente somente em transações com cartão de crédito).
NUM\_PARC            | Campo contendo a quantidade de parcelas da transação. Caso ele esteja ausente, ou tiver valor “0” ou “1”, considerar venda à vista.
VIA\_ESTABELECIMENTO | Cupom referente à via do estabelecimento.
VIA\_CLIENTE         | Cupom referente à via do cliente.
TIPO\_CAMPOS         | JsonObject contendo todos os campos da transação com a CliSiTef, incluindo os outros campos dessa tabela. Formato:<br/>```{"Id TipoCampo N" : [ "valor 1", ....]}```<br/>Os valores são sempre passados em JsonArray, mesmo ocorrendo somente uma ocorrência do campo pela CliSiTef.<br/>Para descobrir o significado de cada campo contido nessa lista favor olhar a tabela “Tabela de valores para TipoCampo” no documento SiTef - Interface Simplificada com a aplicação seção 5.3.2.<br/>Nem todos os parâmetros da tabela 5.3.2 podem estar presentes em todas as transações, alguns parâmetros podem não ser enviados dependendo da forma de pagamento ou se a transação não for concluída.
<br/>

### **Tabela 3 - Valores do CODRESP**
***CODRESP*** | ***Descrição***
--------------|------------------------
0             | Sucesso na execução da função.
1             | Endereço IP inválido ou não resolvido
2             | Código da loja inválido
3             | Código de terminal inválido
6             | Erro na inicialização do Tcp/Ip
7             | Falta de memória
8             | Não encontrou a CliSiTef ou ela está com problemas
9             | Configuração de servidores SiTef foi excedida.
10            | Erro de acesso na pasta CliSiTef (possível falta de permissão para escrita)
11            | Dados inválidos passados pela automação.
12            | Modo seguro não ativo
13            | Caminho DLL inválido (o caminho completo das bibliotecas está muito grande).
Outro valor positivo|Negada pelo autorizador.
-1            | Módulo não inicializado. O PDV tentou chamar alguma rotina sem antes executar a função configura.
-2            | Operação cancelada pelo operador.
-3            | O parâmetro função / modalidade é inexistente/inválido.
-4            | Falta de memória no PDV.
-5            | Sem comunicação com o SiTef.
-6            | Operação cancelada pelo usuário (no pinpad).
-9            | A automação chamou a rotina ContinuaFuncaoSiTefInterativo sem antes iniciar uma função iterativa.
-10           | Algum parâmetro obrigatório não foi passado pela automação comercial.
-12           | Erro na execução da rotina iterativa. Provavelmente o processo iterativo anterior não foi executado até o final (enquanto o retorno for igual a 10000).
-13           | Documento fiscal não encontrado nos registros da CliSiTef. Retornado em funções de consulta tais como ObtemQuantidadeTransaçõesPendentes.
-15           | Operação cancelada pela automação comercial.
-20           | Parâmetro inválido passado para a função.
-25           | Erro no Correspondente Bancário: Deve realizar sangria.
-30           | Erro de acesso ao arquivo. Certifique-se que o usuário que roda a aplicação tem direitos de leitura/escrita.
-40           | Transação negada pelo servidor SiTef.
-41           | Dados inválidos.
-43           | Problema na execução de alguma das rotinas no pinpad.
-50           | Transação não segura.
-100          | Erro interno do módulo.
Outro valor negativo   | Erros detectados internamente pela rotina.

## Exemplo de tratamento do retorno do m-SiTef
<br/>

```java
protected void onActivityResult (int requestCode, int resultCode, Intent data) {
if (requestCode == 1234) { 
   if (resultCode == RESULT_OK){
      System.out.println("m-SiTef executado com sucesso!");
      System.out.println("Erro no m-SiTef!");
      System.out.println("CODRESP: " + data.getExtras().getString("CODRESP"));
      System.out.println("COMP_DADOS_CONF: " + data.getExtras().getString("COMP_DADOS_CONF"));
      System.out.println("CODTRANS: " + data.getExtras().getString("CODTRANS"));
      System.out.println("TIPO_PARC: " + data.getExtras().getString("TIPO_PARC"));
      System.out.println("VLTROCO: " + data.getExtras().getString("VLTROCO"));
      System.out.println("REDE_AUT: " + data.getExtras().getString("REDE_AUT"));
      System.out.println("BANDEIRA: " + data.getExtras().getString("BANDEIRA"));
      System.out.println("NSU_SITEF: " + data.getExtras().getString("NSU_SITEF"));
      System.out.println("NSU_HOST: " + data.getExtras().getString("NSU_HOST"));
      System.out.println("COD_AUTORIZACAO: " + data.getExtras().getString("COD_AUTORIZACAO"));
      System.out.println("NUM_PARC: " + data.getExtras().getString("NUM_PARC"));

      System.out.println("VIA_ESTABELECIMENTO: " + data.getExtras().getString("VIA_ESTABELECIMENTO"));
      System.out.println("VIA_CLIENT: " + data.getExtras().getString("VIA_CLIENTE"));

      String tipoCampos = data.getExtras().getString("TIPO_CAMPOS");
      JSONObject jsonCampos= null;
      jsonCampos = new JSONObject(tipoCampos);
      String campo = jsonCampos.getString("132");

    }
  }
}
```

## Mudando o CLSIT
<br/>
A partir da versão 3.167 é possível alterar o arquivo CLSIT. A alteração só pode ser feita uma vez, depois disso só será possível mudar o CLSIT mudando a empresa ou forçando a parada do aplicativo no menu de configuração do Android. 
O arquivo CLSIT possui o seguinte formato: 

```json
[seção]
Chaves=valor
//O arquivo pode ter n seções e cada seção pode ter n chaves
```
Veja abaixo exemplos de como usar essa funcionalidade e um modelo de CLSIT que será usado como base para os exemplos.

````json
[CliSiTef]
DiretorioTrace=./files
HabilitaTrace=1
TraceRotativo=20
TamanhoTraceRotativo=122880

[CliSiTefI]
DiretorioTrace=./files
HabilitaTrace=1

[Geral]
TransacoesHabilitadas=7;8;16;26;27;28;30;40;43;56;57;58;3203;3624;3627
````

### **Alterando campos**
````java
//Montando as seções
JSONObject clsitJson = new JSONObject();
JSONObject clisitefSectionJson = new JSONObject();
JSONObject geralSection = new JSONObject();
clisitefSectionJson.put("DiretorioTrace", "./files2");
geralSection.put("TransacoesHabilitadas", "7;8;20;29;42;3006;3014;3020;3021;3022;3027;3028;3029;3030;3031;3203;3321;3322;3323");

clsitJson.put("CliSiTef", clisitefSectionJson);
clsitJson.put("Geral", geralSection);

i.putExtra("clsit", clsitJson.toString());
````

Veja no CLSIT abaixo que as chaves *DiretorioTrace* e *TrancacoesHabilitadas* foram atualizadas com os valores enviados pelo exemplo acima, importante ressaltar que caso a chave não exista ela será adicionada na seção enviada e se já existir seu valor será mesclado com os valores já existentes.

````json
[CliSiTef]
DiretorioTrace=./files2
HabilitaTrace=1
TraceRotativo=20
TamanhoTraceRotativo=122880

[CliSiTefI]
DiretorioTrace=./files
HabilitaTrace=1

[Geral]
TransacoesHabilitadas=7;8;20;29;42;3006;3014;3020;3022;3021;3027;3028;3029;3030;3031;3221;3322;3323;
````

### **Adicionando campos**
````java
JSONObject geralSection = new JSONObject();
geralSection.put("HabilitaColetaTaxaEmbarque", "1");
geralSection.put("ColetaValorEntradaIATA", "1");

JSONObject novaSecao = new JSONObject();
novaSecao.put("NovaChave", "NovoValor");
clsitJson.put("NovaSecao", novaSecaoJson);
clsitJson.put("Geral", geralSection);

i.putExtra("clsit", clsitJson.toString());
````

Veja abaixo que as chaves *HabilitaColetaTaxaEmbarque* e *ColetaValorEntradaIATA* foram adicionadas a seção Geral e que a seção *NovaSecao* foi criada, pois ela ainda não existia.

````json
[CliSiTef]
DiretorioTrace=./files
HabilitaTrace=1
TraceRotativo=20
TamanhoTraceRotativo=122880

[CliSiTefI]
DiretorioTrace=./files
HabilitaTrace=1

[Geral]
TransacoesHabilitadas=7;8;16;26;27;28;30;40;43;56;57;58;3203;3624;3627
HabilitaColetaTaxaEmbarque=1

[NovaSecao]
NovaChave=NovoValor
````

### **Removendo campos**
````java
JSONObject clisitefSectionJson = new JSONObject();
clisitefSectionJson.put("TamanhoTraceRotativo", JSONObject.NULL);

clsitJson.put("CliSiTef", clisitefSectionJson);

i.putExtra("clsit", clsitJson.toString());
````

Veja abaixo que a chave *TamanhoTarceRotativo* foi removida do CLSIT. Caso uma seção não contenha mais chaves ela será excluída também.

````json
[CliSiTef]
DiretorioTrace=./files
HabilitaTrace=1
TraceRotativo=20

[CliSiTefI]
DiretorioTrace=./files
HabilitaTrace=1

[Geral]
TransacoesHabilitadas=7;8;16;26;27;28;30;40;43;56;57;58;3203;3624;3627
````

**Importante:**
Para saber exatamente quais seções, chaves e valores devem ser passados para montagem do seu arquivo CLSIT, por favor consultar o nosso suporte. 
O m-SiTef possui um CLSIT padrão que será adotado caso nenhuma alteração especificada nesse capítulo seja feita.
Segue a imagem dele:

````json
[CliSiTef]
DiretorioTrace=./files
HabilitaTrace=1
TraceRotativo=20
TamanhoTraceRotativo=122880

[CliSiTefI]
DiretorioTrace=./files
HabilitaTrace=1

[Geral]
TransacoesHabilitadas=7;8;16;26;27;28;30;40;43;56;57;58;3203;3624;3627
````

## Exemplo de integração utilizando o AndroidX
<br/>
Caso esteja realizando a integração com o m-SiTef utilizando as bibliotecas do AndroidX, os métodos startActivityForResult e onActivityResult estão depreciados.
Abaixo segue um exemplo da chamada do m-SiTef utilizando o AndroidX
<br/>

```java
Intent i = new Intent("br.com.softwareexpress.sitef.msitef.ACTIVITY_CLISITEF");
i.putExtra("empresaSitef", "00000001");
i.putExtra("enderecoSitef", "127.0.0.1;127.0.0.1:20036");
i.putExtra("operador", "0001");
i.putExtra("data", "20140312");
i.putExtra("hora", "150000");
i.putExtra("numeroCupom", "1");
i.putExtra("numParcelas", "3");
i.putExtra("modalidade", "0");
i.putExtra("valor", "9000");
i.putExtra("CNPJ_CPF", "12345678912345");
i.putExtra("timeoutColeta", "30");
i.putExtra("acessibilidadeVisual", "0");

startPagamentoIntent.launch(i);
```

Definição do ActivityResultLauncher para realizar o tratamento do retorno do m-SiTef

```java
ActivityResultLauncher<Intent> startPagamentoIntent = registerForActivityResult(
    new ActivityResultContracts.StartActivityForResult(),
    result -> {
        if (result.getResultCode() == RESULT_OK) {
          Intent data = result.getData();
          System.out.println("m-SiTef executado com sucesso!");
          System.out.println("Erro no m-SiTef!");
          System.out.println("CODRESP: " + data.getExtras().getString("CODRESP"));
          System.out.println("COMP_DADOS_CONF: " + data.getExtras().getString("COMP_DADOS_CONF"));
          System.out.println("CODTRANS: " + data.getExtras().getString("CODTRANS"));
          System.out.println("TIPO_PARC: " + data.getExtras().getString("TIPO_PARC"));
          System.out.println("VLTROCO: " + data.getExtras().getString("VLTROCO"));
          System.out.println("REDE_AUT: " + data.getExtras().getString("REDE_AUT"));
          System.out.println("BANDEIRA: " + data.getExtras().getString("BANDEIRA"));
          System.out.println("NSU_SITEF: " + data.getExtras().getString("NSU_SITEF"));
          System.out.println("NSU_HOST: " + data.getExtras().getString("NSU_HOST"));
          System.out.println("COD_AUTORIZACAO: " + data.getExtras().getString("COD_AUTORIZACAO"));
          System.out.println("NUM_PARC: " + data.getExtras().getString("NUM_PARC"));

          System.out.println("VIA_ESTABELECIMENTO: " + data.getExtras().getString("VIA_ESTABELECIMENTO"));
          System.out.println("VIA_CLIENT: " + data.getExtras().getString("VIA_CLIENTE"));

          String tipoCampos = data.getExtras().getString("TIPO_CAMPOS");
          JSONObject jsonCampos= null;
          jsonCampos = new JSONObject(tipoCampos);
          String campo = jsonCampos.getString("132");
        }
    });
```

