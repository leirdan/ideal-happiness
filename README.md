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
* Note que não é possível declarar e simultaneamente atribuir um valor a uma variável no T-SQL!

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