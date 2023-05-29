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

