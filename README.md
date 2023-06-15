# TRANSACT SQL
O **TRANSACT SQL**, ou também **T-SQL**, é uma linguagem interpretada utilizada pelo SQL Server para a manipulação dos bancos de dados. O T-SQL engloba desde comandos básicos SQL (como `select, insert, drop, ...`) até condicionais e laços de repetição. Assim, os comandos SQL ANSI tradicionais estão contidos no conjunto da linguagem T-SQL.

É com a linguagem T-SQL que é possível realizar procedimentos mais complexos no banco de dados, tais quais criar funções e os próprios procedimentos armazenados.

#### OBSERVAÇÃO: Para este repositório, usaremos um banco de dados chamado *Sucos_Vendas*.
* Tal *database* consiste nas tabelas:
    * *clientes*: clientes da marca fictícia; 
    * *vendedores*: vendedores da marca fictícia;
    * *produtos*: produtos da marca fictícia;
    * *notas_fiscais*: registros que marcam as transações e vendas da empresa;
    * *itens_notas_fiscais*: registros que guardam as informações do preço e quantidade vendidas do produto por cada nota fiscal.

## 1. VARIÁVEIS
Variáveis são, assim como nas demais linguagens de programação, um espaço na memória onde atribuiremos um valor que será utilizado diversas vezes.

Para declarar uma variável no T-SQL, utilizamos `DECLARE @[Nome da Variável] [Tipo da Variável]`. Toda variável precisa ter um tipo, e uma característica do T-SQL é que qualquer tipo pode ser atribuido a uma variável, até mesmo o tipo *tabela*.

Para atribuir um valor a uma variável, utilizamos `SET @[Nome da Variável] = [Valor]`.
> Podemos realizar uma declaração e atribuição de valor simultaneamente, como em: `DECLARE @NAME VARCHAR(100) = 'Behemoth'`

Para imprimirmos uma variável, utilizamos o comando `PRINT @[Nome da Variável]`.

Exemplo: *crie duas variáveis numéricas e as multiplique*.

O código gerado foi este:
```sql
DECLARE @num1 INT;
SET @num1 = 18
DECLARE @num2 INT;
SET @num2 = 2
DECLARE @multi INT
SET @multi = @num1 * @num2
PRINT (@multi)
```

**Desafio: crie 4 variáveis: nome, idade, data de nascimento e valor de compra; atribua valores a elas e as imprima em uma única mensagem de texto.**

Solução:
```sql
DECLARE @cliente varchar(10);
DECLARE @idade int;
DECLARE @data_nasc date;
DECLARE @compra float;

SET @cliente = 'João'
SET @idade = 10
SET @data_nasc = '10-01-2007'
SET @compra = 10.23

PRINT (CONCAT(' O cliente ', @cliente, ', nascido em ', YEAR(@data_nasc), ' e com ', @idade, ' anos, comprou ', @compra, ' reais em produtos!'))
-- saída:  O cliente João, nascido em 2007 e com 10 anos, comprou 10.23 reais em produtos!
```

## 2. LIGANDO VARIÁVEIS AO BANCO DE DADOS
É possível declarar variáveis e utilizá-las em consultas, inserções e mais procedimentos no banco de dados.
Por exemplo, em um caso de múltiplas inserções em um banco de dados, ao invés de inserirmos múltiplas vezes os valores dentro das instruções `INSERT INTO`, podemos fazer:
```sql
DECLARE @matricula varchar(5), @nome varchar(100), @percentual float, @data_admissao date, @de_ferias bit, @bairro varchar(50);
SET @matricula = '00239'
SET @nome = 'George Harrison'
SET @percentual = 0.10
SET @data_admissao = '2012-03-12'
SET @de_ferias = 1
SET @bairro = 'Jardins'

INSERT INTO TABELA_DE_VENDEDORES 
VALUES
(@matricula, @nome, @percentual, @data_admissao, @de_ferias, @bairro)

SET @matricula = '00240'
SET @nome = 'Roger Waters'
SET @percentual = 0.12
SET @data_admissao = '2015-06-17'
-- reutilizamos o comando insert. note que ele irá incluir dois vendedores distintos!
INSERT INTO TABELA_DE_VENDEDORES 
VALUES
(@matricula, @nome, @percentual, @data_admissao, @de_ferias, @bairro)

PRINT 'Dois vendedores incluídos!'
```

Outra maneira de uso é coletar valores de campos das tabelas e atribuí-los a uma variável. Por exemplo, aqui vamos coletar a data de nascimento de um cliente e modificar a sua idade:
```sql
DECLARE @CPF VARCHAR(11), @DATA_NASC DATE, @IDADE INT

SELECT @CPF = CPF FROM TABELA_DE_CLIENTES TC WHERE DATA_DE_NASCIMENTO = '2000-02-12'
SELECT @DATA_NASC = DATA_DE_NASCIMENTO FROM TABELA_DE_CLIENTES WHERE CPF = @CPF
SET @IDADE = DATEDIFF(YEAR, @DATA_NASC, GETDATE())

PRINT (@DATA_NASC) -- 2000-02-12
PRINT (@IDADE) -- 23

UPDATE TABELA_DE_CLIENTES 
SET IDADE = @IDADE
WHERE DATA_DE_NASCIMENTO = @DATA_NASC
```
Como resultado, modificamos a idade do cliente *Fernando Cavalcante* para 23.

Em outro caso, vamos selecionar a quantidade de notas fiscais geradas no dia 01/01/2017:
```sql
DECLARE @NUMNOTAS INT, @DATA_VENDA DATE
SET @DATA_VENDA = '2017-01-01'
SELECT @NUMNOTAS = COUNT(*) FROM NOTAS_FISCAIS WHERE DATA_VENDA = @DATA_VENDA
PRINT @NUMNOTAS -- 74
```

## 3. CONTROLE DE FLUXO - IF
A linguagem T-SQL, ao contrário do SQL ANSI padrão, possui formas de lidar com situações em que deve haver controle de fluxo do programa. No T-SQL, assim como em inúmeras linguagens de programação, utiliza-se o comando **IF-ELSE**.
Sua sintaxe é:
```sql
IF <Expressão>
    <comandos SQL/controle de bloco>
ELSE
    <comandos SQL/controle de bloco>
```

* A expressão a ser passada no If, também chamada de *expressão lógica*, retorna verdadeiro ou falso. Então, se a expressão for verdadeira, os comandos dentro do If serão executados; se não, os comandos dentro do Else que serão.

* Controle de bloco nada mais é que alguns comandos SQL encadeados e limitados por uma instrução `BEGIN` e `END`. Mesmo que haja somente um comando dentro do If, por vezes é uma boa prática limitá-lo com esses blocos.

Para exemplificar sua sintaxe, tenhamos o seguinte caso:
```sql
if 4^2 = 4 * 2
    -- executará este comando, pois 16 não é igual a 8.
	print 'vdd!'
else
	print 'falso'
```

Se quisermos verificar se daqui a *x* número de dias será dia de semana ou fim de semana, podemos fazer:
```sql
DECLARE @DIA_SEMANA VARCHAR(20), @NUMDIAS INT

SET @NUMDIAS = 999

SET @DIA_SEMANA = DATENAME (WEEKDAY, DATEADD(DAY, @NUMDIAS, GETDATE()))

PRINT @DIA_SEMANA -- sábado

IF @DIA_SEMANA = 'Sábado' OR @DIA_SEMANA = 'Domingo'
	PRINT 'fim de semana!'
ELSE
	PRINT 'dia normal'
```

Em outro caso, verifiquemos se o limite de crédito dos clientes do bairro de Jardins ultrapassa o valor de 90 mil:
```sql
DECLARE @bairro VARCHAR(50);
DECLARE @limiteAtual FLOAT;
DECLARE @limiteMax FLOAT;

SET @bairro = 'jardins';
SET @limiteMax = 90000;

SELECT @limiteAtual = sum(limite_de_credito) FROM TABELA_DE_CLIENTES WHERE bairro = @bairro
-- comparar o maximo com o atual
IF @limiteAtual > @limiteMax
	BEGIN
        -- executará esse.
		PRINT 'passou do limite'
	END
ELSE
	BEGIN
		PRINT 'está dentro do limite'
	END
```

## 4. CONTROLE DE FLUXO - WHILE
O comando **while** é responsável pelos laços de repetição do T-SQL, que nada mais são que trechos de código SQL que podem ser executados repetidas vezes. Ele expressa uma condição: *se* a condição for verdadeira, tudo dentro do `WHILE` será executado; se não, não será executado.

Sua sintaxe é:
```sql
WHILE <Expressão Lógica>
    <comandos SQL/controle de bloco>
        <BREAK ou CONTINUE>
```
* O comando `BREAK` faz com que o loop se encerre, ou seja, ele força o fim do loop; em contrapartida, o comando `CONTINUE` é responsável por reiniciar o loop, até que a condição lógica seja verdadeira.

Vejamos um exemplo simples: contar todos os números de 1 até 1000. 
```sql
DECLARE @min int, @max int;
SET @min = 1;
SET @max = 1000;

PRINT 'Início'
WHILE @min <= @max
	BEGIN
		PRINT @min
		SET @min = @min + 1
	CONTINUE
	END
PRINT 'Fim'
```

Em um outro exemplo, vamos verificar o número total de vendas feitas a cada dia do intervalo 01/01/2017 e 30/01/2017:
```sql
DECLARE @dateMin date, @dateMax date, @numNotas int
SET @dateMin = '2017-01-01'
SET @dateMax = '2017-01-30'

WHILE @dateMin <= @dateMax	
	BEGIN
		SELECT @numNotas = COUNT(*) FROM NOTAS_FISCAIS WHERE DATA_VENDA = @dateMin
		PRINT CONVERT(VARCHAR, @dateMin) + ' ---> ' + CONVERT(VARCHAR, @numNotas) + ' notas!'
		SELECT @dateMin = (DATEADD(DAY, 1, @dateMin))
		CONTINUE
	END
```

Mas e se quisermos verificar somente os 10 primeiros registros dessa consulta? Podemos fazer o seguinte com o while:
```sql
DECLARE @dateMin date, @dateMax date, @numNotas int, @numLinhas int, @numMaxLinhas int
SET @dateMin = '2017-01-01'
SET @dateMax = '2017-01-30'
SET @numMaxLinhas = 10
SET @numLinhas = 0

WHILE @dateMin <= @dateMax	
	BEGIN
		SELECT @numNotas = COUNT(*) FROM NOTAS_FISCAIS WHERE DATA_VENDA = @dateMin
		PRINT CONVERT(VARCHAR, @dateMin) + ' ---> ' + CONVERT(VARCHAR, @numNotas) + ' notas!'
		SELECT @dateMin = (DATEADD(DAY, 1, @dateMin))
		SET @numLinhas = @numLinhas + 1
		IF @numLinhas = @numMaxLinhas
			BEGIN
				PRINT 'Foram impressas 10 linhas.'
				BREAK
			END
		CONTINUE
	END
```

### 4.1 Inserindo dados em loop
Presuma que queremos verificar cada nota fiscal por seu número e decidir se essa nota existe ou não. Para isso, vamos criar uma tabela adicional chamada *STATUS_NOTA* com dois atributos: 
```sql
CREATE TABLE STATUS_NOTA (
	NUMERO INT, -- número da nota
	STATUS VARCHAR(30)
);
```
Vamos agora verificar alguns registros. Primeiro, devemos determinar o intervalo de números a ser verificado e um número que será usado para teste. Depois, executaremos o loop com uma condição dentro e inserções. Ao final, serão inseridos novos 99.999 registros na tabela *STATUS_NOTA*.
```sql
DECLARE @min INT, @max INT, @teste int
SET @min = 1
SET @max = 100000

DELETE FROM STATUS_NOTA -- exclusão necessária para não duplicar os valores a cada execução

WHILE @min < @max
	BEGIN
        -- essa variável terá valor 1 se existir um registro com o mesmo número do momento, ou 0 caso não exista.
		SELECT @teste = count(*) FROM NOTAS_FISCAIS WHERE NUMERO = @min
		IF @teste = 1
			INSERT INTO STATUS_NOTA VALUES (@min, 'É nota fiscal')
		ELSE
			INSERT INTO STATUS_NOTA VALUES (@min, 'Não é nota fiscal')
		SET @min = @min + 1
		CONTINUE
	END
```

## 5. TABELAS TEMPORÁRIAS

Tabelas temporárias são tabelas que existem somente durante o tempo de consulta e execução, não ocupando espaço na memória do computador - mas ocupa na memória RAM. Uma diferença notável de uma tabela temporária para uma tradicional é que ela **não pode possuir chaves estrangeiras**. Existem alguns tipos específicos:
* Tabelas que iniciam com `#`: são tabelas que está *somente* na própria instância de conexão local. Na prática, significa que em uma janela de script diferente da que está sendo executada a consulta esta tabela não será exibida.
* Tabelas que iniciam com `##`: são tabelas que aparecem em *todas* as conexões do SQL Server, e só desaparecerão quando o serviço do SQL Server for interrompido.
* Tabelas que iniciam com `@`: são tabelas que só existem *enquanto os comandos T-SQL estão sendo executados*. Ou seja, quando a mensagem *(Comandos concluídos com êxito)* for exibida a tabela será apagada. Elas são geralmente associadas às variáveis.

Analisemos alguns casos:

### 5.1 Tabela com "#"
A princípio, vamos criar uma tabela temporária de linguagens de programação com 3 atributos: ID (com autoincremento), nome e ranking.
```sql
CREATE TABLE #LANGS (
	ID INT IDENTITY(1,1),
	NAME VARCHAR(30),
	RANKING NVARCHAR(100)
);
```
Em seguida, vamos inserir alguns valores:
```sql
INSERT INTO #LANGS VALUES 
('C++', '1'), 
('Lua', '3'), 
('Bash Script', '2')
```

É possível notar que quando criamos uma nova conexão não é possível visualizar esta tabela de linguagens, visto que, como comentado, tabelas que iniciam com `#` só são visíveis e acessíveis dentro da própria conexão onde foram criadas.

### 5.2 Tabelas com "##"

Vamos criar novamente a mesma tabela:
```sql
CREATE TABLE ##LANGS (
	ID INT IDENTITY(1,1),
	NAME VARCHAR(30),
	RANKING NVARCHAR(100)
);
```
E inserimos novos valores:
```sql
INSERT INTO ##LANGS VALUES 
('Python', '3'), 
('Typescript', '1'), 
('Java', '2')
```

Note que, se for criada uma nova janela de script, ainda será possível consultar os dados da tabela `##LANGS`, pois tabelas que iniciam com ## são acessíveis em todas as instâncias de conexão do SQL Server.

### 5.3 Tabelas com "@"
Criaremos a mesma tabela anterior, mas de outra forma:
```sql
DECLARE @LANGS TABLE (
    ID INT IDENTITY(1,1), NAME VARCHAR(30), RANKING INT
)

INSERT INTO @LANGS VALUES ('Assembly', '2'), ('BASIC', '1')

SELECT * FROM @LANGS
```
Como é possível notar, a tabela com @ é, na verdade, uma variável que armazena uma tabela. Por isso, fora do ambiente de execução, não é possível executar o `SELECT * FROM @LANGS`, já que a variável só é declarada no momento de execução.

## 6. FUNÇÕES UDF
Uma função é um conjunto de instruções T-SQL armazenadas no banco de dados com o objetivo de executar uma ação e retornar resultados (tabelas ou valores escalares, como strings, números, etc.). 
Uma função UDF (*User-Defined-Function*) é, por sua vez, uma função desenvolvida pelo usuário, e não pré-definida pelo banco de dados.
Para criar uma função UDF, existem algumas obrigações:
- Uso de `begin` e `end` para delimitar o início e fim da função;
- Uso do `return` para sempre retornar um valor ao fim;
- Declaração dos *parâmetros* logo após a definição do nome da função.

Sua sintaxe é:
```sql
CREATE FUNCTION <Nome da Função> (<Argumentos>)
RETURNS <Tipo de Retorno>
AS
BEGIN
	<Instruções T-SQL>
	RETURN <Variável de retorno>
END
```
> Funções podem retornar tabelas ao invés de valores escalares. Para isso, declara-se o tipo de RETURNS como "TABLE" e o RETURN como a operação que retorna a tabela.

Para exemplificar, vamos criar uma função que calcula o faturamento total de uma determinada nota fiscal e ver como podemos utilizar de maneira prática a nossa função:
* Seja a consulta abaixo a base:
```sql
-- realiza a contagem da nota fiscal de número 120
SELECT SUM(QUANTIDADE * [PREÇO]) FROM [dbo].[ITENS NOTAS FISCAIS] WHERE NUMERO = 120;
``` 

* Adaptando para uma função:
```sql 
CREATE FUNCTION CalcularFaturamento (@num AS INT)
RETURNS FLOAT
AS
BEGIN
	-- armazena o resultado da consulta em uma variável e a retorna
	DECLARE @RESULTADO FLOAT
	SELECT @RESULTADO = SUM(QUANTIDADE * [PREÇO]) FROM [dbo].[ITENS NOTAS FISCAIS] WHERE NUMERO = @num
	RETURN @RESULTADO
END
```
> Ao executar este comando, a função será guardada no banco de dados na pasta de "Programação".

* Para executar:
```sql
SELECT NUMERO, dbo.CalcularFaturamento(NUMERO) FROM [dbo].[NOTAS FISCAIS]
```
> O que essa consulta faz é selecionar o número da nota fiscal, passar este número para a função como parâmetro e a executa, gerando uma tabela com duas colunas: uma com os números das notas fiscais e a outra com o faturamento de cada uma.

Em outro exemplo, quero agora calcular o faturamento total do bairro que o usuário inserir. Para isso, temos a primeira consulta:

```sql
SELECT TOP 100 SUM(INF.QUANTIDADE * INF.[PREÇO]) FROM [dbo].[TABELA DE VENDEDORES] TV
INNER JOIN [dbo].[NOTAS FISCAIS] NF
	ON NF.MATRICULA = TV.MATRICULA
INNER JOIN [dbo].[ITENS NOTAS FISCAIS] INF
	ON INF.NUMERO = NF.NUMERO
WHERE BAIRRO = 'Jardins' -- exemplo de bairro
```

Adaptando para função e adicionando um comportamento dinâmico:

```sql
CREATE FUNCTION FaturamentoBairro (@bairro AS VARCHAR(100))
RETURNS FLOAT
AS 
BEGIN
	DECLARE @TOTAL FLOAT
	SELECT @TOTAL = SUM(INF.QUANTIDADE * INF.[PREÇO]) FROM [dbo].[TABELA DE VENDEDORES] TV
		INNER JOIN [dbo].[NOTAS FISCAIS] NF
			ON NF.MATRICULA = TV.MATRICULA
		INNER JOIN [dbo].[ITENS NOTAS FISCAIS] INF
			ON INF.NUMERO = NF.NUMERO
		WHERE BAIRRO = @bairro
	RETURN @TOTAL
END
```

Executando o comando `SELECT dbo.FaturamentoBairro('Jardins')`, temos como retorno o número *46050253,8327901*, ou seja, o total de vendas naquele bairro durante todo o período.

Agora, um exemplo que retorna uma tabela: função que retorne informações de notas fiscais onde um determinado produto de código qualquer aparece.

```sql
CREATE FUNCTION ListaProdutosPorNota (@codigo AS INT)
RETURNS TABLE
AS
RETURN SELECT * FROM [dbo].[ITENS NOTAS FISCAIS] WHERE [CODIGO DO PRODUTO] = @codigo

-- Consultando
SELECT * FROM dbo.ListaProdutosPorNota(479745)
```

### 6.1 Modificando uma função
Para modificar uma função usa-se a palavra-chave `ALTER FUNCTION`, similar a outros comandos de alteração do SQL Server. Veja o exemplo abaixo:

Suponha que desejo criar uma função que retorne o endereço completo de todos os meus clientes. Seja esta a função:
```sql
CREATE FUNCTION MontarEnderecoCompleto (
@endereco AS VARCHAR(100),
@bairro AS VARCHAR(100),
@cidade AS VARCHAR(50),
@estado AS VARCHAR(2),
@cep AS VARCHAR(8)
)
RETURNS VARCHAR(200)
AS
BEGIN
	DECLARE @ENDERECOCOMPLETO VARCHAR(200) = CONCAT(@endereco, ' no bairro ', @bairro, ', em ', @cidade, '(', @estado, '), CEP ', @cep)
	RETURN @ENDERECOCOMPLETO
END
```

No entanto, decidi que desejo que não apareçam mais os trechos "no bairro" e "em". Se eu tentar alterar diretamente na instrução `CREATE FUNCTION`, dará um erro pois a função já existe e não pode ser "criada novamente". Assim, deve-se utilizar a instrução `ALTER FUNCTION` da seguinte maneira:
```sql
ALTER FUNCTION MontarEnderecoCompleto (
@endereco AS VARCHAR(100),
@bairro AS VARCHAR(100),
@cidade AS VARCHAR(50),
@estado AS VARCHAR(2),
@cep AS VARCHAR(8)
)
RETURNS VARCHAR(200)
AS
BEGIN
	DECLARE @ENDERECOCOMPLETO VARCHAR(200) = CONCAT(@endereco, ', ', @bairro, ', ', @cidade, '(', @estado, '), CEP ', @cep)
	RETURN @ENDERECOCOMPLETO
END

-- Consultando
SELECT NOME, [dbo].MontarEnderecoCompleto([ENDERECO 1], BAIRRO, CIDADE, ESTADO, CEP) AS ENDEREÇO FROM [TABELA DE CLIENTES]
-- Exemplo de saída:
-- R. Dois de Fevereiro, Água Santa, Rio de Janeiro(RJ), CEP 22000000
```

### 6.2 Deletando uma função
Para apagar uma função, basta apenas utilizar o comando `DROP FUNCTION <Nome da Função>`.

### 6.3 DESAFIO: Criar uma nova nota fiscal com um cliente, vendedor e produtos aleatórios!

1. *Criar uma função que gere números aleatórios entre um limite mínimo e máximo*

```sql
-- Cálculo: (MAXIMO - MINIMO - 1) * NUMERO ALEATORIO ENTRE 0 E 1 + MINIMO = NÚMERO ENTRE MÍNIMO E MÁXIMO

-- Mínimo = 100; Máximo = 1000
SELECT ROUND((1000 - 100 - 1 ) * RAND() + 100, 0)

-- A função rand() não pode ser "parcela de cálculo", por isso urge a necessidade de criar essa view para consumir seu resultado de fora
CREATE VIEW VW_ALEATORIO AS SELECT RAND() AS ALEATORIO

-- Transportando para função
CREATE FUNCTION GerarNumAleatorio (@min AS INT, @max as INT)
RETURNS INT
AS
BEGIN
	DECLARE @NUM INT
	DECLARE @ALEATORIO FLOAT
	SELECT @ALEATORIO = ALEATORIO FROM VW_ALEATORIO
	SET @NUM = ROUND((@max - @min - 1 ) * @ALEATORIO + @min, 0)
	RETURN @NUM
END

-- Utilizando
SELECT [dbo].GerarNumAleatorio(100, 1000)
```

2. *Criar uma função que percorra essa consulta e verifique se o número gerado corresponde a uma nota fiscal no banco de dados*

```sql
DECLARE @NUM_ALEATORIO INT, @NOTA INT
SET @NUM_ALEATORIO = [dbo].GerarNumAleatorio(1, 1000)
SELECT @NOTA = COUNT(*) FROM [dbo].[NOTAS FISCAIS] WHERE NUMERO = @NUM_ALEATORIO
IF @NOTA = 0
	BEGIN
		PRINT CONCAT('Não existe nota de número ', @NUM_ALEATORIO)
	END
ELSE
	BEGIN
		PRINT CONCAT('A nota ', @NUM_ALEATORIO, ' existe')
	END

-- Consulta
DECLARE @RESULTADO INT
SELECT @RESULTADO = [dbo].IsNotaFiscal([dbo].GerarNumAleatorio(10, 1000))
IF @RESULTADO = 0
	PRINT 'Não existe nota fiscal.'
ELSE
	PRINT 'Existe nota fiscal.'
```

3. *Faça com que a função seja executada diversas vezes e percorra toda a tabela*

```sql
-- Criação de tabela temporária para visualização dos resultados
DECLARE @STATUS_NOTAS TABLE ([NUMERO] INT, [STATUS] VARCHAR(200))
DECLARE @MIN INT, @MAX INT, @RESULTADO INT, @CONDICAO INT

-- Intervalo
SET @MIN = 0
SET @MAX = 100000

WHILE @MIN <= @MAX
BEGIN
	SELECT @RESULTADO = [dbo].GerarNumAleatorio(@MIN, @MAX)
	SELECT @CONDICAO = [dbo].IsNotaFiscal(@RESULTADO)
	IF @CONDICAO > 0
		BEGIN
			INSERT INTO @STATUS_NOTAS VALUES (@RESULTADO, 'É nota fiscal.')
		END
	ELSE
		BEGIN
			INSERT INTO @STATUS_NOTAS VALUES (@RESULTADO, 'Não é nota fiscal.')
		END
	SET @MIN = @MIN + 1
	CONTINUE
END
-- Como os números aleatórios podem se repetir, usamos a cláusula `distinct` para reduzir o número de linhas
SELECT DISTINCT * FROM @STATUS_NOTAS ORDER BY NUMERO
```

