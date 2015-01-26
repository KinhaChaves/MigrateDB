[![Build Status](https://travis-ci.org/malukenho/MigrateDB.png?branch=master)](https://travis-ci.org/malukenho/MigrateDB) [![Latest Stable Version](https://poser.pugx.org/malukenho/migratedb/v/stable.png)](https://packagist.org/packages/malukenho/migratedb) [![Total Downloads](https://poser.pugx.org/malukenho/migratedb/downloads.png)](https://packagist.org/packages/malukenho/migratedb) [![Latest Unstable Version](https://poser.pugx.org/malukenho/migratedb/v/unstable.png)](https://packagist.org/packages/malukenho/migratedb) [![License](https://poser.pugx.org/malukenho/migratedb/license.png)](https://packagist.org/packages/malukenho/migratedb)
# MigrateDB




**MigrateDB** é uma ferramenta simples para migrar dados entre bancos de dados.

#### 1ª Etapa:

Instalando o  **MigrateDB** fica muito simples usar o composer :3

Crie *composer.json* acompanhando o roteiro

```javascript
{
    "require": {
        "malukenho/migratedb": "dev-master"
    }
}
```
Execute o  **composer install** e tudo certo!

Bem, nós temos que seguir a tabela

```sql
mysql> SELECT * FROM user;
+------+------------+---------+
| id   | name       | passwd  |
+------+------------+---------+
|    1 | Kika Pimpo | 123@456 |
|    2 | RamStrYou  | 1!#@$%6 |
+------+------------+---------+
1 row in set (0.00 sec)
```
E gostraia de migrar esses dados para outra tabela.

```sql
mysql> SELECT * FROM member;
+-----------+-----------------+------------+
| user_id   | user_name       | user_pass  |
+-----------+-----------------+------------+
|           Nothing to see here            |
+-----------+-----------------+------------+
1 row in set (0.00 sec)
```
Vamos lá!

Crie a classe para obter dados da tabela. 

Você pode usar a anotação ***@from_table*** para defini-la, e ***@complement*** para aumentar sua consulta.

```php
<?php
/**
 * @Configurations(
 *     from_table="user",
 *     to_table="member",
 *     complement="WHERE status = '1'",
 *     type="select"
 * )
 */
class User implements EnumTablesRelation
{
	const user_id = 'id';
	const user_name = 'name'
	const user_pass = 'passwd';
}
```

O código acima gera a seguinte consulta:

```sql
SELECT id, name, passwd FROM user WHERE status = 1
```

#### 2ª Etapa:

Crie a classe para relacionar com a classe anterior.

Irá inserir os dados selecionados no *1ª Etapa* nesse banco de dados.

```php
<?php
/**
 * @Configurations(
 *     from_table="user",
 *     to_table="member"
 * )
 */
class InsertUser implements EnumTablesRelation
{
	const IDENTIFY = 'user_id';
	const USERNAME = 'user_name'
	const PASSWORD = 'user_pass';
}
```
#### Etapa Final

Crie um arquivo de configuração e execute a migração :3

```php
<?php
$loader = require __DIR__.'/vendor/autoload.php';

$mySql = new PDO('...');
$mySql2 = new PDO('...');

$router = new MigrateDB(new User);
 
$result = $router->setConnection($mySql, $mySql2)
    ->MapperDatas('1');
 
$router->replyTo(new InsertUser)
    ->with($result);
```
Você também pode configurá-lo assim:

```php
<?php
$routerClient = new MigrateDB(new ClientData);

$routerClient->registerFilter(new ClientFilter)
    ->replyTo(new ClientDataReply)
    ->with(
        $routerClient->setConnection(
            $mySqlConnection, 
            $fireBirdConnection
        )->MapperDatas(
            rand(0, 9)
        )
    );
```


## Avançado

#### Tipos

Usamos estruturas avançadas para a migração de informações entre o banco de dados, isto é decidido de acordo com o tipo de anotação, é o tipo definada na classe EnumTablesRelation. 

Existem 3 tipos válidos até agora. Eles são:

- select
- join
- as

O tipo **select** temos visto em exemplos anteriores.

#### join

Aqui está um exemplo de `join`:

```php
<?php
/**
 * @Configurations(
 *     from_table="UserList",
 *     to_table="NewUserList",
 *     complement="WHERE UserList.iduser = $1",
 *     type="join"
 * )
 */
class UserRelation implements EnumTablesRelation
{
	const user_id = 'userid';
	const user_name = 'name.user_detail ON id = 1';
	const user_pass = 'passwd';
}
```

O código anterior gera a seguinte consulta:

```sql
SELECT 
    `UserList`.`userid` AS user_id, 
    `UserList`.`passwd` AS user_pass, 
    `user_detail`.`name` AS user_name 
        FROM UserList 
            INNER JOIN 
                `user_detail` ON user_detail.id = 1 
WHERE UserList.iduser = 1
```

#### as

Este é um exemplo de `as`:

```php
<?php
/**
 * @Configurations(
 *     from_table="UserList",
 *     to_table="NewUserList",
 *     complement="WHERE iduser = $1",
 *     type="as"
 * )
 */
class UserRelation implements EnumTablesRelation
{
	const user_id = 'userid';
	const user_name = 'name';
	const user_pass = 'passwd';
}
```

O código anterior gera a seguinte consulta:

```sql
SELECT 
    `user_id` AS `userid`, 
    `user_name` AS `name`, 
    `user_pass` AS `passwd` 
         FROM UserList 
WHERE iduser = 1
```
