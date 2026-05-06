# Ambientes

#DML
#Insira um novo registro na tabela categoria com os seguintes dados: idcategoria = 1,
#Descricao = 'Eletrônicos'.
insert into categoria (idcategoria, Descricao)
values (1, 'Eletrônicos');
select * from categoria;

#Insira um novo produto chamado "Notebook", pertencente à categoria com id 1, com
#um preço de 2500.00 e quantidade em estoque de 10.
insert into produto (idproduto, Nome, Descricao, Preco, QuantEstoque, categoria_idcategoria) 
values (70010, 'Notebook', 'Descrição do Notebook', 2500.00, 10, 1);

#Atualize o preço do produto com id 1 para 2700.00.
update produto
set preco = 2700.00
where idproduto = 1;

#Liste o nome e preço de todos os produtos que estão na categoria com id 1.
select nome, preco
from produto
where idcategoria = 1;

#Exclua o produto cujo id é 1.
delete from produto
where idproduto = 1;

#Insira dois novos tipos de clientes: 'Regular' e 'VIP'.
insert into tipo_cliente (descricao)
values ('Regular'), ('VIP');

#Liste o nome dos clientes e os tipos de clientes correspondentes.
select c.nome, t.descricao AS tipo_cliente
from cliente c
inner join tipo_cliente t 
on c.idtipo_cliente = t.idtipo_cliente;

#Atualize a quantidade em estoque de todos os produtos na categoria 1, adicionando
#5 unidades.
update produto
set quantidade_estoque = quantidade_estoque + 5
where idcategoria = 1;

#Agregacao-------------------------------------------------------------------------
#Qual é a quantidade total de produtos em estoque?
select sum(quantidade_estoque) as total_estoque

#Qual é o preço médio dos produtos cadastrados?
select avg(preco) as preco_medio
from produto;

#Quantos clientes estão cadastrados no banco de dados?
select count(*) as total_clientes
from cliente;

#Qual é o produto mais caro no banco de dados?
select preco,nome from produto
order by preco desc
limit 1;

#Qual é a média de quantidade de produtos por pedido?
select avg(sub.qnt)from(
select pedido_idpedido, sum(Quantidade) as qnt from pedido_has_produto
group by pedido_idpedido)
as sub;

#Juncao----------------------------------------------------------------------------

#Crie uma consulta que retorne o nome do cliente e a descrição do status do pedido
#para todos os pedidos realizados. A consulta deve incluir apenas os pedidos que
#têm status definido.
select c.nome, s.descricao as status_pedido
from pedido p
inner join cliente c on p.idcliente = c.idcliente
inner join status_pedido s on p.idstatus = s.idstatus;

#Faça uma consulta que liste todos os produtos junto com o nome da categoria a que
#pertencem. Inclua produtos que não estejam associados a nenhuma categoria.
select p.nome as produto, c.descricao as categoria
from produto p
left join categoria c on p.categoria_idcategoria = c.idcategoria;

#Elabore uma consulta que mostre todos os tipos de cliente e seus respectivos
#clientes. Mesmo que um tipo de cliente não tenha clientes associados, ele deve aparecer.
select t.descricao as tipo_cliente, c.nome
from tipo_cliente t
left join cliente c on c.idtipo_cliente = t.idtipo_cliente;

#Crie uma consulta que retorne todas as combinações possíveis entre clientes e tipos
#de endereço. A consulta deve incluir todos os clientes e todos os tipos de endereço,
#mesmo que não haja correspondência.
select c.nome, te.descricao as tipo_endereco
from cliente c
cross join tipo_endereco te;

#Crie uma consulta que traga o nome do cliente, o produto que ele comprou e a
#quantidade comprada, considerando apenas os pedidos que têm produtos associados.
select c.nome as cliente, p.nome as produto, php.quantidade
from pedido_has_produto php
inner join pedido pe on php.pedido_idpedido = pe.idpedido
inner join cliente c on pe.idcliente = c.idcliente
inner join produto p on php.produto_idproduto = p.idproduto;

#Construa uma consulta que retorne todos os endereços de clientes, juntamente com o
#nome do cliente, mas apenas para aqueles endereços que são padrão. Inclua também os
#clientes que não têm endereços.
select c.nome, e.endereco
from cliente c
left join endereco e 
on c.idcliente = e.idcliente and e.padrao = 1;

#Faça uma consulta que retorne a quantidade total de produtos vendidos por cada
#produto, mesmo que alguns produtos não tenham sido vendidos. Mostre o nome do
#produto e a quantidade vendida.
select p.nome as produto, coalesce(sum(php.quantidade),0) as total_vendido
from produto p
left join pedido_has_produto php on p.idproduto = php.produto_idproduto
group by p.idproduto, p.nome;

#Crie uma consulta que mostre todos os pedidos realizados por um cliente específico
#(por exemplo, idcliente = 1), incluindo o nome do cliente, a data do pedido e o
#valor total do pedido.
select c.nome, p.data_pedido, p.valor_total
from pedido p
inner join cliente c on p.idcliente = c.idcliente
where c.idcliente = 1;

#Agregacao e Juncao-------------------------------------------------------------

#1. Quantos produtos há em cada categoria?
select c.descricao, count(p.idproduto) as total_produtos
from categoria c
left join produto p on p.categoria_idcategoria = c.idcategoria
group by c.idcategoria, c.descricao;

#2. Qual é a quantidade total de produtos em estoque para cada categoria?
select c.descricao, sum(p.QuantEstoque) as total_estoque
from categoria c
left join produto p on p.categoria_idcategoria = c.idcategoria
group by c.idcategoria, c.descricao;

#3. Quantos pedidos existem para cada status?
select s.descricao, count(p.idpedido) as total_pedidos
from status_pedido s
left join pedido p on p.idstatus = s.idstatus
group by s.idstatus, s.descricao;

#4. Quantos clientes pertencem a cada tipo de cliente?
select t.descricao, count(c.idcliente) as total_clientes
from tipo_cliente t
left join cliente c on c.idtipo_cliente = t.idtipo_cliente
group by t.idtipo_cliente, t.descricao;


#Subconsulta---------------------------------------------------------------------

#1. Quais produtos têm um preço acima da média de todos os produtos cadastrados?
select nome, preco
from produto
where preco > (select avg(preco) from produto);

#2. Liste o nome dos clientes que fizeram pelo menos um pedido.
select nome
from cliente
where idcliente in (select idcliente from pedido);

#3. Liste o número de cada pedido e o nome do produto mais caro vendido nesse pedido.
select p.idpedido, pr.nome
from pedido p
join pedido_has_produto php on p.idpedido = php.pedido_idpedido
join produto pr on pr.idproduto = php.produto_idproduto
where pr.preco = (
    select max(pr2.preco)
    from pedido_has_produto php2
    join produto pr2 on pr2.idproduto = php2.produto_idproduto
    where php2.pedido_idpedido = p.idpedido
);

#4. Quais clientes fizeram pedidos cujo valor total é maior que a média de todos os pedidos?
select distinct c.nome
from cliente c
join pedido p on c.idcliente = p.idcliente
where p.valor_total > (select avg(valor_total) from pedido);


#Desafio-------------------------------------------------------------------------

#1. Qual cliente realizou o maior valor total em pedidos?
select c.nome, sum(p.valor_total) as total_gasto
from cliente c
join pedido p on c.idcliente = p.idcliente
group by c.idcliente, c.nome
order by total_gasto desc
limit 1;

#2. Liste o produto mais vendido de cada categoria (baseado na quantidade total vendida).
select c.descricao as categoria, pr.nome, sum(php.quantidade) as total_vendido
from produto pr
join categoria c on pr.categoria_idcategoria = c.idcategoria
join pedido_has_produto php on pr.idproduto = php.produto_idproduto
group by c.idcategoria, pr.idproduto
having total_vendido = (
    select max(sub.total)
    from (
        select sum(php2.quantidade) as total
        from produto pr2
        join pedido_has_produto php2 on pr2.idproduto = php2.produto_idproduto
        where pr2.categoria_idcategoria = c.idcategoria
        group by pr2.idproduto
    ) as sub
);

#3. Quais clientes fizeram mais de 2 pedidos e o valor total desses pedidos é superior a 5000?
select c.nome, count(p.idpedido) as total_pedidos, sum(p.valor_total) as total_gasto
from cliente c
join pedido p on c.idcliente = p.idcliente
group by c.idcliente, c.nome
having count(p.idpedido) > 2 and sum(p.valor_total) > 5000;

#4. Qual categoria gerou o maior valor total em vendas?
select c.descricao, sum(php.quantidade * pr.preco) as total_vendas
from categoria c
join produto pr on pr.categoria_idcategoria = c.idcategoria
join pedido_has_produto php on pr.idproduto = php.produto_idproduto
group by c.idcategoria, c.descricao
order by total_vendas desc
limit 1;

#5. Qual é a média de pedidos por cliente, agrupada pelo tipo de cliente?
select t.descricao,
       avg(sub.total_pedidos) as media_pedidos
from tipo_cliente t
join (
    select c.idtipo_cliente, c.idcliente, count(p.idpedido) as total_pedidos
    from cliente c
    left join pedido p on c.idcliente = p.idcliente
    group by c.idcliente
) as sub on sub.idtipo_cliente = t.idtipo_cliente
group by t.idtipo_cliente, t.descricao;
