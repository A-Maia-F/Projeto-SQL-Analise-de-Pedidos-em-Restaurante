# Projeto SQL- Análise de Pedidos em Restaurante
Análise de banco de dados relacional de um restaurante utilizando MySQL. O projeto envolve a criação de views, funções e uso de JOINs, com foco em consolidar e analisar dados de pedidos, clientes, funcionários e produtos.

## Objetivo
Aplicar conhecimentos práticos em SQL através da criação de views, funções personalizadas, joins complexos e utilização de comandos de análise (como EXPLAIN) para explorar e analisar informações do banco de dados restaurante.

## Etapas
### 1. Criação da VIEW `resumo_pedido`
Consolidação de múltiplas tabelas em uma única view:
- Tabelas envolvidas: `pedidos`, `clientes`, `funcionarios`, `produtos`
- Campos selecionados: `id_pedido`, `quantidade`, `data_pedido`, `nome_cliente`, `email`, `nome_funcionário`, `nome_produto`, `preço`

```
USE restaurante
CREATE VIEW resumo_pedido AS
SELECT
    pe.id_pedido, pe.quantidade, pe.data_pedido,
    c.nome AS nome_cliente, c.email,
    f.nome AS nome_funcionário,
    pr.nome AS nome_produto, pr.preço
FROM pedidos pe
JOIN clientes c ON pe.id_cliente = c.id_cliente
JOIN funcionários f ON pe.id_funcionario = f.id_funcionario
JOIN produtos pr ON pe.id_produto = pr.id_produto;
```

### 2. Cálculo do total por pedido (quantidade × preço)
Consulta direta na view para extrair o total de cada pedido:
```
SELECT id_pedido, nome_cliente, (quantidade * preço) AS total
FROM resumo_pedido;
```

### 3. Atualização da View `resumo_pedido` com o campo `total`
```
DROP VIEW resumo_pedido;

CREATE VIEW resumo_pedido AS
SELECT
    pe.id_pedido, pe.quantidade, pe.data_pedido,
    c.nome AS nome_cliente, c.email,
    f.nome AS nome_funcionário,
    pr.nome AS nome_produto, pr.preço,
    (pe.quantidade * pr.preço) AS total
FROM pedidos pe
JOIN clientes c ON pe.id_cliente = c.id_cliente
JOIN funcionários f ON pe.id_funcionario = f.id_funcionario
JOIN produtos pr ON pe.id_produto = pr.id_produto;
```

### Verificação da performance da consulta com `EXPLAIN`
```
EXPLAIN SELECT id_pedido, nome_cliente, total FROM resumo_pedido;
```

### 5. Criação da função `BuscaIngredientesProduto`
Retorna os ingredientes de um produto a partir do ID:
```
DELIMITER //
CREATE FUNCTION BuscaIngredientesProduto(produto_id INT)
RETURNS VARCHAR(200)
READS SQL DATA
BEGIN
    DECLARE ingrediente VARCHAR(200);
    SELECT ingredientes INTO ingrediente
    FROM info_produtos
    WHERE id_produto = produto_id;
    RETURN ingrediente;
END //
DELIMITER ;
```

Exemplo de uso:
```
SELECT BuscaIngredientesProduto(10);
```

### 6. Criação da função mediaPedido
Compara o valor total de um pedido com a média de todos os pedidos e retorna uma mensagem descritiva:
```
DELIMITER //
CREATE FUNCTION mediaPedido(pedido_id INT)
RETURNS VARCHAR(100)
READS SQL DATA
BEGIN
    DECLARE total_pedido DECIMAL(10,2);
    DECLARE media_total DECIMAL(10,2);
    DECLARE resposta VARCHAR(100);

    SELECT (pe.quantidade * pr.preço)
    INTO total_pedido
    FROM pedidos pe 
    JOIN produtos pr ON pe.id_produto = pr.id_produto
    WHERE pe.id_pedido = pedido_id;

    SELECT AVG(pe.quantidade * pr.preço)
    INTO media_total
    FROM pedidos pe 
    JOIN produtos pr ON pe.id_produto = pr.id_produto;

    SET resposta =
    CASE
        WHEN total_pedido > media_total THEN "Acima da média"
        WHEN total_pedido = media_total THEN "Igual à média"
        ELSE "Abaixo da média"
    END;

    RETURN resposta;
END //
DELIMITER ;
```

Exemplo de uso:
```
SELECT mediaPedido(5);
SELECT mediaPedido(6);
```

## Tecnologias Ultilizadas
- MySQL - Sistema de gerenciamento de banco de dados
- Funções e Views - Criação e manutenção
- Joins - INNER JOIN para consolidar dados
- Funções personalizadas - Uso de `DELIMITER`, `DECLARE`, `SELECT INTO`, `CASE`
