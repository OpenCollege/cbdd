# Consulta em Banco de Dados
Consulta em Banco de Dados - Cursado em 2019/2


## Conceitos gerais
Hash Join - Índice temporário



1 -
O produto cartesiano, combina tudo com tudo. Ou seja, em uma operação de produto cartesiano de uma tabela A com uma tabela B,  cada linha da tabela A é combinada com uma linha da tabela B. Neste caso, o grau e a cardinalidade do resultado é diferente do grau e cardinalidade das tabelas iniciais.
No caso de operações de conjunto, as tabelas precisam ser compatíveis. Ou seja, devem possuir o mesmo número de colunas, com o mesmo domínio. E diferentemente das junções, não é uma combinação de colunas para criar um resultado, apenas uma manipulação de linhas.
No caso das junções, há uma combinação de colunas. Pode-se dizer que é uma aplicação de um produto cartesiano, juntamente com uma seleção. As colunas de A e B são combinadas em um único resultado.

2 -
Tanto o conhecimento de álgebra relacional quanto o plano de execução nos ajudam a entender a eficiência de uma consulta. Ao analisar o plano de execução é possível checar possíveis melhorias, e utilizando conceitos de álgebra relacional podemos melhorar uma consulta. Por exemplo, ao analisar uma consulta e averiguar que o que se quer encontrar são resultados presentes em uma tabela mas ausentes em outra, podemos utilizar os conhecimentos de álgebra relacional e aplicar a diferença entre as tabelas, que no SQL tem a sintaxe de EXCEPT.

3 -
O where é um filtro que filtra linha por linha, checando a condição assim que é chamado. O having é aplicado na agregação. Por exemplo, uma chamada ao banco de dados pode ter um where e ter um having. Primeiro é feito o filtro where, ele pode inclusive ser agrupado, e depois disso, será feito o filtro descrito utilizando o having.

4 -
O plano de execução faz um inner join, ou possivelmente um natural join, e depois uma ordenação. Inicialmente é feito um scan sequencial em ambos. Ambulatórios, como tem uma primary key em nroa, passa um processo de hashing. Ambas as tabelas então são unidas por um hash join, e então ordenadas através de um sort.

5 -  
```sql
SELECT a.nome, count(*) livros_count, max(e.numero) total_edicoes

FROM autor a
INNER JOIN livroautor la
ON la.codigoautor = a.codigo
INNER JOIN livros l
ON l.codigo = la.codigolivro
INNER JOIN edicao e
ON e.codigolivro = l.codigo
GROUP BY a.nome
ORDER BY livros_count DESC
```

6 -
```sql
DROP TYPE IF EXISTS DadosAutor;

CREATE TYPE DadosAutor AS (
nome varchar(100),
num_livros int,
num_edicoes int
);

CREATE OR REPLACE FUNCTION relatorio()
RETURNS INT AS $$
DECLARE
	curs1 refcursor;
	dados_autor DadosAutor;
BEGIN

	OPEN curs1 FOR  
		SELECT a.nome, count(*) livros_count, max(e.numero) total_edicoes
		FROM autor a
		INNER JOIN livroautor la
		ON la.codigoautor = a.codigo
		INNER JOIN livros l
		ON l.codigo = la.codigolivro
		INNER JOIN edicao e
		ON e.codigolivro = l.codigo
		GROUP BY a.nome
		ORDER BY livros_count DESC;
	LOOP
		FETCH NEXT FROM curs1 INTO dados_autor;
IF NOT FOUND  THEN
EXIT;
END IF;
		RAISE NOTICE 'Autor %, Livros %, Edicoes %',
		dados_autor.nome, dados_autor.num_livros, dados_autor.num_edicoes;
	END LOOP;


	RETURN 1;
END;
$$ LANGUAGE plpgsql;

select relatorio();
```
7 -
```sql
drop view relatorioLivro;

CREATE OR REPLACE VIEW relatorioLivro AS
SELECT e.ano, count(l) livros_publicados, max(e.numero) edicoes_publicadas
FROM livros l
INNER JOIN edicao e
ON e.codigolivro = l.codigo
GROUP BY e.ano;

select * from relatorioLivro limit(10);
```
8 -
```sql
CREATE OR REPLACE FUNCTION validaNumeros()
RETURNS TRIGGER AS $$
DECLARE
numero int;
BEGIN
	numero = CAST(NEW.numero as INTEGER);
	IF (numero BETWEEN 1 AND 9) THEN
		RAISE NOTICE 'Numero OK';
	ELSE
		RAISE EXCEPTION 'Numero deve ser entre 1 e 9';
	END IF;

END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER validaNumerosTrigger
BEFORE INSERT ON edicao
FOR EACH ROW EXECUTE PROCEDURE  validaNumeros();

INSERT INTO edicao (codigolivro,numero,ano) VALUES (0,1,0);
INSERT INTO edicao (codigolivro,numero,ano) VALUES (0,0,0);
```
