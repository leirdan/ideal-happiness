# TRANSACT SQL
O **TRANSACT SQL**, ou também **T-SQL**, é uma linguagem interpretada utilizada pelo SQL Server para a manipulação dos bancos de dados. O T-SQL engloba desde comandos básicos SQL (como `select, insert, drop, ...`) até condicionais e laços de repetição. Assim, os comandos SQL ANSI tradicionais estão contidos no conjunto da linguagem T-SQL.

É com a linguagem T-SQL que é possível realizar procedimentos mais complexos no banco de dados, tais quais criar funções e os próprios procedimentos armazenados.

### 1. VARIÁVEIS
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