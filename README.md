
CALL apoc.import.graphml(
  "https://raw.githubusercontent.com/TG-Rep/2026.1-EP03-Grafos-Spec/refs/heads/main/lognet.graphml",
  {readLabels: true}
);


MATCH (n)
WHERE n.type IS NOT NULL
WITH n, toUpper(n.type) AS label
SET n:$(label)
REMOVE n.type
RETURN count(*) AS updatedNodes;

Como funciona
WITH n, toUpper(n.type) AS label
Converte o atributo type (supplier, warehouse, retailer) para SUPPLIER, WAREHOUSE e RETAILER.
SET n:$(label)
Adiciona dinamicamente a label cujo nome está armazenado na variável label.
Essa é a sintaxe recomendada nas versões recentes do Neo4j.
REMOVE n.type
Remove a propriedade type, já que ela passa a ser representada pela label.


MATCH (a)-[r:RELATED]->(b)
WITH a, b, r, 'ROUTE' AS relType
CREATE (a)-[newRel:$(relType)]->(b)
SET newRel = properties(r)
DELETE r
RETURN count(*) AS updatedRelationships;
