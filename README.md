<img width="886" height="157" alt="image" src="https://github.com/user-attachments/assets/269fadf2-90db-4cc5-ab67-8283f3e63379" />



Vaga: Analista de Integração de Dados

Nome: Daniel Dias Ramos


1-	Recuperar a base de dados. Para isso instalei o Debeaver na minha máquina junto do PostgreSQL já que a conexão deveria ser rodada nesse modelo de banco

<img width="473" height="102" alt="image" src="https://github.com/user-attachments/assets/b32f56af-4271-4b50-b34b-82f114a19757" />



<img width="672" height="722" alt="image" src="https://github.com/user-attachments/assets/54160f77-146e-4c6f-96d1-53642e1187db" />







 
2-	Criar as tabelas para importar os dados do Excel. Nesse ponto basta colar os scripts enviados na estrutura de como os dados devem ser inseridos
O exemplo abaixo faço com o primeiro script, os demais são feitos da mesma forma
 
<img width="886" height="965" alt="image" src="https://github.com/user-attachments/assets/9fd6966c-a38f-4bc4-b1f6-74abe0c928db" />



Para o Scrip de Ordem de Compra faltava definir a chave primaria ordem_compra defini como not null por se tratar de chave. 

Para a tabela de produtos o nome produto_id estava escrito como idproduto, corrigi isso e adicionei uma virgula no final da declaração dos campos (antes de constrain) e o campo de chave primaria de filial_id não pode ser nulo então troquei o NULL por NOT NULL



Na tabela de fornecedor faltava corrigir idfornecedor, e descrição escrito errado e mudar idfornecedor o tipo para VARCHAR já que os códigos são alfanumericos

No final as tabelas ficaram dessa forma:

<img width="628" height="313" alt="image" src="https://github.com/user-attachments/assets/cfff85e2-b83b-4cec-966b-4a5991335158" />

 

3-	Para importar eu separei cada aba dessas tabelas em arquivos CSV (separados por ponto e virgula) diferentes e importei no debeaver:
Ex da tabela de fornecedores:

 <img width="630" height="668" alt="image" src="https://github.com/user-attachments/assets/eecd0283-2e4e-4b63-89a9-280f785c3d11" />



<img width="886" height="752" alt="image" src="https://github.com/user-attachments/assets/f76a8d3b-bfbc-4f64-8376-0367cb292e01" />

 

Obs: as importações estavam dando erro pois as chaves foram definidas como NOT NULL e o excel entendia como ultima linha usada linhas em branco, usando o notepad++  pude identificar as linhas e tratar direto no excel as linhas em branco ao converter pra csv a importação deu certo


Parte 2 – Consultas SQL Básicas

--Monte uma consulta que traga o total de vendas, 
--em quantidade e em valores (R$), de cada produto,
--no mês de fevereiro de 2025.

select produto_id ,qtde_vendida ,'R$ '||TRUNC((qtde_vendida * valor_unitario)::numeric,2) as TOTAL_VENDA 
from VENDA
where data_emissao between '01/02/2025' and '28/02/2025';

<img width="886" height="225" alt="image" src="https://github.com/user-attachments/assets/60ddc3d6-c191-42c1-aede-741200c794b0" />

 

obs: no banco postgre o resultado da busca se tornou do tipo double por devido a declaração dos dados, para não alterar a declaração adicionei a função ::numeric para na hora de truncar ele já entender que os dados que estou buscando são numéricos e conseguir executar a pesquisa.
 	Optei por não fazer o Join com a tabela de produtos_filial pois ela só traz 20 produtos e a tabela de venda tem 28, mesmo que fizesse Left Join ficariam produtos sem descrição o que não seria o ideal.

--Crie uma consulta para listar os produtos que foram requisitados, mas não recebidos.

select produto_id,descricao_produto   from pedido_compra 
where qtde_entregue  = 0 or qtde_entregue is null;

<img width="886" height="613" alt="image" src="https://github.com/user-attachments/assets/479bda64-77e1-49cc-a31e-865ddd246ec0" />

 
-- Concatenar os campos produto_id e descricao_produto (onde houver) no formato
select idproduto || ' - ' || descricao  from produtos_filial ;
 
<img width="716" height="784" alt="image" src="https://github.com/user-attachments/assets/d905c6ed-e69c-4f8f-95d3-b449b840baea" />




-- Transformar o campo de datas para o formato DD/MM/YYYY
select to_char(data_emissao,'DD/MM/YYYY')  from venda;
 
<img width="658" height="714" alt="image" src="https://github.com/user-attachments/assets/7a009f9b-1ce3-48da-bb63-ebf1e83e0f6d" />




--Retornar os dados filtrando apenas os produtos requisitados mais de 10 vezes no período.
Obs: como não tinha nenhum período especificado eu fiz uma pesquisa genérica para que caso seja um mês ou data específica eu possa colocar no ‘2025-01’ e se precisasse de uma recorrência menor substituiria no having que é quem faz a analise da existência da ocorrencia
 

<img width="863" height="441" alt="image" src="https://github.com/user-attachments/assets/4f858ad3-37e7-43d8-a257-d6531e72177a" />



Criar uma trigger que gere automaticamente um novo 
idfornecedor
numérico na tabela de produtos que se relacione com a tabela de fornecedor.
Primeiramente corrigi minha importação que estava vindo linhas em branco e em seguida defini a chave primaria da tabela fornecedor que não foi aplicada no script inicial de criação da tabela mas que pode ser adicionada no próprio debeaver
 <img width="886" height="608" alt="image" src="https://github.com/user-attachments/assets/1112d299-35b2-4442-8054-fa90fcc4c51a" />


Em seguida criei a function que a trigger irá chamar, ela irá avaliar o que foi escrito no campo idfornecedor da tabela de produtos_filial, independente da forma como for escrito ele avalia de fora alfanumérico. Em seguida ele irá avaliar na tabela fornecedor qual foi o ultimo idfornecedor criado (segu o padrão F + ultimo numerocriado) e com isso ele irá criar na tabela de fornecedor o novo idfornecedor e com o campo que foi informado na tabela produto_filial ele irá preencher o  campo razão_social da tabela fornecedor. E com isso na tabela de produtos_filial será preenchido corretamente o campo de idfornecedor.






Segue exemplo com o insert:

INSERT INTO produtos_filial 
(filial_id, idproduto, descricao, estoque, preco_unitario, preco_compra, preco_venda,idfornecedor) 
VALUES 
(1,'P21', 'Produto 21 teste', 10, 50, 80, 100,'SYSTOCK');

PRINT DA TABELA DE FORNECEDOR

 <img width="886" height="1059" alt="image" src="https://github.com/user-attachments/assets/edffd9fd-4842-4765-85e2-615165ac53aa" />


PRINT DA TABELA DE PRDUTOS_FILIAL
 
<img width="886" height="1050" alt="image" src="https://github.com/user-attachments/assets/3a7b53f8-d30b-41a8-82eb-d27ef9515fa9" />




O script usado para a function foi:

CREATE OR REPLACE FUNCTION fnc_gera_idfornecedor()
RETURNS TRIGGER AS $$
DECLARE
    v_proximo_id VARCHAR(10);
    v_ultimo_numero INT;
    v_nome_fornecedor VARCHAR;
BEGIN
    
    v_nome_fornecedor := NEW.idfornecedor;

    
    SELECT idfornecedor INTO v_proximo_id 
    FROM fornecedor 
    WHERE razao_social = v_nome_fornecedor; 

    
    IF v_proximo_id IS NULL THEN
        
        SELECT COALESCE(MAX(CAST(SUBSTRING(idfornecedor FROM 2) AS INT)), 0) + 1 
        INTO v_ultimo_numero 
        FROM fornecedor
        WHERE idfornecedor ~ '^F[0-9]+$';
        
        v_proximo_id := 'F' || v_ultimo_numero;

        
        INSERT INTO fornecedor (idfornecedor, razao_social) 
        VALUES (v_proximo_id, v_nome_fornecedor);
    END IF;

    
    NEW.idfornecedor := v_proximo_id;

    RETURN NEW;
END;
$$ LANGUAGE plpgsql;


A trigger criada que chama a function:

CREATE TRIGGER trg_insere_id_fornecedor
BEFORE INSERT ON produtos_filial
FOR EACH ROW
EXECUTE FUNCTION fnc_gera_idfornecedor();






Parte 4 – Estratégia de Validação com o Cliente
Imagine que você precisa validar os dados do mês de Fevereiro de 2025 com o cliente.
Monte um roteiro descritivo:

1.	Quais seriam os principais pontos que você validaria com o cliente?

Primeiramente iria entender o as principais dúvidas e levantar os requisitos dos dados que me apresentar e que estiverem com a divergência apontada. Iria encontrar a divergência para o cliente e apontar onde devem ser as correções para que ela seja diminuída ou mesmo resolvida.
Levando em conta o estoque iria levantar os principais produtos que estão saindo e quanto eles têm em estoque para que não falte nas saídas e ocorra corte no pedido e consequentemente a perda da venda. E também os produtos que menos saem se seria relevante realizar menos compras ou a diminuição do valor do produto com base na validade (na base de dados que foi disponibilizada não temos a validade, mas seria interessante esse ponto)
Também seria um ponto de atenção as ordens de compra que foram solicitadas mas não foram entregues.

3.	Quais técnicas utilizaria para garantir a exatidão e a precisão dos dados?

Levando em conta que os dados foram importados de uma base em excel seria interessante o tratamento desde a importação como espaços em branco denifir com clareza as chaves primarias, números com virgula ou mesmo números negativos quando estamos trabalhando com estoque não seriam interessantes também. Definir se os parâmetros vão receber maiúsculo ou minúsculo ou ambos. Em seguida validar direto com o cliente nas rotinas que ele usa para puxar as informações que ele está confrontando

4.	Quais consultas você deixaria prontas para usar na reunião de validação?

As padrões seriam pra fazer filtros simples como: 

Select f.* from fornecedor f Where f.dfornecedor =  ;

Select f.* from produtos_filial f where f.produto_id = ; 

Select f.* from pedido_compra f where f.pedido_id =  ; 

Select f.* from venda f where f.venda_id = ; 


