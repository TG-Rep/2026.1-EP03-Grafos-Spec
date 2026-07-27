# EP03

O objetivo desta atividade é explorar **consultas e algoritmos de análise de grafos utilizando Neo4j**, aplicando conceitos avançados de Teoria dos Grafos no contexto de **cadeias de suprimento e logística**.

Diferentemente dos exercícios anteriores, neste EP os algoritmos clássicos não serão implementados diretamente em Python. Em vez disso, o objetivo é modelar consultas utilizando **Cypher** e utilizar algoritmos da biblioteca **Neo4j Graph Data Science (GDS)** para analisar a estrutura da rede logística.

---

# Orientações Gerais

* Utilize estritamente a estrutura disponibilizada no repositório. Não altere nomes de arquivos nem sua localização, pois isso poderá inviabilizar a execução automática dos testes durante o *push* do repositório;

* Todas as consultas devem ser escritas em **Cypher**, utilizando o banco de dados Neo4j disponibilizado para a disciplina;

* Quando solicitado, utilize algoritmos da biblioteca **Graph Data Science (GDS)**. Não implemente esses algoritmos manualmente;

* Organize as consultas de forma consistente e utilize nomes significativos para variáveis, facilitando sua leitura e manutenção;

* Cada grupo deverá desenvolver sua própria solução. As respostas serão inspecionadas visualmente e mecanicamente. Caso seja detectada cópia entre grupos, poderão ser aplicadas penalidades, incluindo nota zero na(s) questão(ões) envolvida(s);

* É responsabilidade do grupo manter sua solução privada durante todo o período da atividade;

* O trabalho deve ser realizado em grupo. Casos excepcionais deverão ser comunicados previamente à equipe docente.

---

# Entrega

* Realize *push* da versão final do repositório até o prazo definido no Google Classroom;

* Durante a correção será considerada a execução mais recente dos *workflows* do GitHub Actions. A equipe poderá reexecutar os *workflows*, caso necessário;

* Caso o projeto contenha erros que impeçam sua execução ou a execução dos testes automáticos, estes não poderão ser corrigidos após o prazo;

* A participação individual será avaliada através do histórico de *commits* e edições do repositório;

* Alterações realizadas após o prazo serão tratadas conforme as regras da disciplina para atividades de reposição.

---

# Organização do Repositório

O repositório possui a seguinte organização.

```
.
├── cypher/
│   ├── Q01.cypher
│   ├── Q02.cypher
│   ├── Q03.cypher
│   └── Q04.cypher
├── images/
│
└── README.md
```

* **cypher** - Contém as consultas Cypher que deverão ser desenvolvidas para cada questão.

* **images** - Contém figuras utilizadas neste documento.

# Domínio do Problema

Neste exercício continuaremos utilizando o mesmo domínio dos EP01 e EP02.

Uma **cadeia de suprimento** é composta por fornecedores, centros de distribuição e clientes, conectados por rotas de transporte que representam o fluxo de mercadorias. O objetivo da empresa é compreender melhor sua rede logística, respondendo perguntas estratégicas por meio de consultas ao banco de dados e algoritmos de análise de grafos.

# Modelo de Dados no Neo4j

A rede logística é representada por um grafo direcionado contendo três tipos de vértices.

## Supplier

Representa um fornecedor de produtos. Principais propriedades: 

* `name`
* `city`
* `capacity`

## Warehouse

Representa um centro de distribuição. Principais propriedades:

* `name`
* `city`
* `capacity`

## Retailer

Representa o cliente final. Principais propriedades:

* `name`
* `city`
* `retailer_type`

## Relacionamentos

Todos os relacionamentos possuem tipo `:ROUTE` e representam rotas de transporte entre entidades da cadeia logística.
Cada relacionamento possui, entre outras, as propriedades:

* `capacity`
* `distance`
* `cost`
* `name`

## Estrutura da Rede

O modelo admite apenas as seguintes conexões:

```
Supplier  ──ROUTE──► Warehouse

Warehouse ──ROUTE──► Warehouse

Warehouse ──ROUTE──► Retailer
```

Não existem relacionamentos diretos:

* Supplier → Retailer
* Retailer → qualquer outro vértice=

# Importação do Grafo

Após criar um sandbox Neo4j vazio, importe o grafo utilizando o procedimento APOC.

```cypher
CALL apoc.import.graphml(
  "https://raw.githubusercontent.com/TG-Rep/2026.1-EP03-Grafos-Spec/refs/heads/main/lognet.graphml",
  {readLabels: true}
);
```

Após a importação, os vértices possuirão uma propriedade denominada `type`. Essa propriedade deve ser convertida em labels do Neo4j.

```cypher
MATCH (n)
WHERE n.type IS NOT NULL
WITH n, toUpper(n.type) AS label
SET n:$(label)
REMOVE n.type
RETURN count(*) AS updatedNodes;
```
Como funciona?
* `WITH n, toUpper(n.type) AS label` - converte o atributo `type` (`supplier, warehouse, retailer`) para `SUPPLIER`, `WAREHOUSE` e `RETAILER`.
* `SET n:$(label)` - adiciona dinamicamente a label cujo nome está armazenado na variável label.
Essa é a sintaxe recomendada nas versões recentes do Neo4j.
* `REMOVE n.type` - Remove a propriedade `type`, já que ela passa a ser representada pela label.
  
Em seguida, substitua o relacionamento `RELATED` pelo relacionamento `ROUTE`.

```cypher
MATCH (a)-[r:RELATED]->(b)
WITH a, b, r, 'ROUTE' AS relType
CREATE (a)-[newRel:$(relType)]->(b)
SET newRel = properties(r)
DELETE r
RETURN count(*) AS updatedRelationships;
```

---

# Verificação da Importação

Antes de iniciar o exercício, recomenda-se verificar se a importação foi realizada corretamente.

Visualize o esquema da base de dados:

```cypher
CALL db.schema.visualization();
```

Liste as labels existentes:

```cypher
SHOW LABELS;
```

Liste os tipos de relacionamento:

```cypher
SHOW RELATIONSHIP TYPES;
```

O esquema esperado é equivalente ao seguinte:

```
(:Supplier)-[:ROUTE]->(:Warehouse)

(:Warehouse)-[:ROUTE]->(:Warehouse)

(:Warehouse)-[:ROUTE]->(:Retailer)
```

---

# Objetivos do EP03

Ao final desta atividade, espera-se que o aluno seja capaz de:

* escrever consultas utilizando a linguagem Cypher;
* utilizar agregações e agrupamentos sobre grafos;
* executar algoritmos da biblioteca Graph Data Science;
* interpretar métricas de centralidade;
* identificar comunidades em redes logísticas;
* modelar problemas clássicos de Teoria dos Grafos em Neo4j;
* interpretar resultados obtidos por algoritmos de análise de grafos no contexto de cadeias de suprimento.
