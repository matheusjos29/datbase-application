# 1. Proponha um modelo em estrela ou floco de neve com o fato pedido, as seguintes métricas serão obrigatórias- valor total do pedido, valor unitario do prato e quantidade. As dimensões obrigatórias serão, cliente, ano mes e dia.


# 2. Entregar as queries que respondam:

# Qual o cliente que mais fez pedidos por ano
```
SELECT ano, cliente AS nome_cliente, num_pedidos
FROM (
  SELECT 
    YEAR(ms.data_hora_entrada) AS ano, 
    cl.nome_cliente AS cliente, 
    COUNT(*) AS num_pedidos,
    ROW_NUMBER() OVER (PARTITION BY YEAR(ms.data_hora_entrada) ORDER BY COUNT(*) DESC) AS rn
  FROM tb_pedido pd
  JOIN tb_mesa ms ON pd.codigo_mesa = ms.codigo_mesa
  JOIN tb_cliente cl ON ms.id_cliente = cl.id_cliente
  GROUP BY YEAR(ms.data_hora_entrada), cl.nome_cliente
) AS ranked
WHERE rn = 1;
```


# Qual o cliente que mais gastou em todos os anos

```
SELECT cliente, total_gasto
FROM (
  SELECT 
    cl.nome_cliente AS cliente, 
    SUM(pd.quantidade_pedido * p.preco_unitario_prato) AS total_gasto,
    ROW_NUMBER() OVER (ORDER BY SUM(pd.quantidade_pedido * p.preco_unitario_prato) DESC) AS rn
  FROM tb_pedido pd
  JOIN tb_prato p ON pd.codigo_prato = p.codigo_prato
  JOIN tb_mesa ms ON pd.codigo_mesa = ms.codigo_mesa
  JOIN tb_cliente cl ON ms.id_cliente = cl.id_cliente
  GROUP BY cl.nome_cliente
) AS ranked
WHERE rn = 1;
```


# Qual(is) o(s) cliente(s) que trouxe(ram) mais pessoas por ano

```
SELECT 
    ano,
    cliente,
    num_pessoas
FROM (
    SELECT
        YEAR(ms.data_hora_entrada) AS ano,
        cl.nome_cliente AS cliente,
        SUM(pd.quantidade_pedido) AS num_pessoas,
        ROW_NUMBER() OVER (PARTITION BY YEAR(ms.data_hora_entrada) ORDER BY SUM(pd.quantidade_pedido) DESC) AS rn
    FROM
        tb_pedido pd
        JOIN tb_mesa ms ON pd.codigo_mesa = ms.codigo_mesa
        JOIN tb_cliente cl ON ms.id_cliente = cl.id_cliente
    GROUP BY
        ano,
        cliente
) AS ranked
WHERE
    rn = 1;
```


# Qual a empresa que tem mais funcionarios como clientes do restaurante;
```
SELECT c.nome_empresa AS company_name, COUNT(DISTINCT e.codigo_funcionario) AS num_employees
FROM db_empresa.tb_beneficio e
JOIN db_empresa.tb_empresa c ON e.codigo_empresa = c.codigo_empresa
WHERE e.codigo_funcionario IN (
    SELECT DISTINCT codigo_funcionario
    FROM db_restaurante.tb_cliente
)
GROUP BY c.nome_empresa
ORDER BY COUNT(DISTINCT e.codigo_funcionario) DESC
LIMIT 1;
```

# Qual empresa que tem mais funcionarios que consomem sobremesas no restaurante por ano;

```
SELECT c.nome_empresa AS company_name, COUNT(DISTINCT b.codigo_funcionario) AS num_employees
FROM db_empresa.tb_beneficio b
JOIN db_empresa.tb_empresa c ON b.codigo_empresa = c.codigo_empresa
WHERE b.codigo_funcionario IN (
    SELECT DISTINCT codigo_funcionario
    FROM db_restaurante.tb_pedido p
    JOIN db_restaurante.tb_prato pr ON p.codigo_prato = pr.codigo_prato
    JOIN db_restaurante.tb_tipo_prato tp ON pr.codigo_tipo_prato = tp.codigo_tipo_prato
    WHERE tp.codigo_tipo_prato = 3
)
GROUP BY c.nome_empresa
ORDER BY COUNT(DISTINCT b.codigo_funcionario) DESC
LIMIT 1;

(creio que essa esteja errrada)
```
