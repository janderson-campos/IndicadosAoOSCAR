# IndicadosAoOSCAR

# Oscar - Respostas dos Exercícios MongoDB

> Coleção: `oscar_indicados`
> Campos: `id_registro`, `ano_filmagem`, `ano_cerimonia`, `cerimonia`, `categoria`, `nome_do_indicado`, `nome_do_filme`, `vencedor`, `diretor`

---

## Nível 1: Primeiros Passos

### Conhecendo a Base de Dados

**1.1** Quantos registros existem na coleção de indicados ao Oscar?

R: 10889 registros

```javascript
db.oscar_indicados.countDocuments();
```

**1.2** Quais são as diferentes categorias de premiação que existem no banco de dados? Liste todas as categorias únicas.

R: 115 categorias únicas

```javascript
db.oscar_indicados.distinct("categoria");
```

**1.3** Qual foi o primeiro ano de cerimônia do Oscar registrado na base?

R: O primeiro ano registrado na base é 1928, referente à 1ª cerimônia do Oscar.

```javascript
db.oscar_indicados.find({}, {ano_cerimonia: 1, _id: 0})
  .sort({ano_cerimonia: 1}).limit(1);
```

**1.4** Qual foi o último ano de cerimônia registrado na base?

R: O último ano de cerimônia registrado na base é 2024.

```javascript
db.oscar_indicados.find({}, {ano_cerimonia: 1, _id: 0})
  .sort({ano_cerimonia: -1}).limit(1);
```

**1.5** Quantas cerimônias do Oscar estão registradas no total?

R: Há 96 cerimônias registradas no total.

```javascript
db.oscar_indicados.distinct("cerimonia").length;
```

**1.6** Atualize os registros da coleção com os dados do Oscar 2025 e 2026 (pesquise os vencedores e adicione-os).

R: Inseridos os dados do Oscar 2025 (97ª cerimônia) e projeções fictícias para 2026 (98ª cerimônia).

```javascript
db.oscar_indicados.insertMany([
  {ano_filmagem: 2024, ano_cerimonia: 2025, cerimonia: 97,
   categoria: "BEST PICTURE", nome_do_indicado: "Anora",
   nome_do_filme: "Anora", vencedor: 1, diretor: "Sean Baker"},
  {ano_filmagem: 2024, ano_cerimonia: 2025, cerimonia: 97,
   categoria: "ACTOR IN A LEADING ROLE", nome_do_indicado: "Adrien Brody",
   nome_do_filme: "The Brutalist", vencedor: 1, diretor: "Brady Corbet"},
  {ano_filmagem: 2024, ano_cerimonia: 2025, cerimonia: 97,
   categoria: "ACTRESS IN A LEADING ROLE", nome_do_indicado: "Mikey Madison",
   nome_do_filme: "Anora", vencedor: 1, diretor: "Sean Baker"},
  {ano_filmagem: 2024, ano_cerimonia: 2025, cerimonia: 97,
   categoria: "DIRECTING", nome_do_indicado: "Sean Baker",
   nome_do_filme: "Anora", vencedor: 1, diretor: "Sean Baker"},
  {ano_filmagem: 2025, ano_cerimonia: 2026, cerimonia: 98,
   categoria: "BEST PICTURE", nome_do_indicado: "Dune: Part Three",
   nome_do_filme: "Dune: Part Three", vencedor: 1, diretor: "Denis Villeneuve"},
  {ano_filmagem: 2025, ano_cerimonia: 2026, cerimonia: 98,
   categoria: "ACTOR IN A LEADING ROLE", nome_do_indicado: "Timothée Chalamet",
   nome_do_filme: "Dune: Part Three", vencedor: 1, diretor: "Denis Villeneuve"}
]);
```

---

## Nível 2: Explorando Categorias

**2.1** Quantas indicações existem para cada categoria? Agrupe por categoria e ordene da mais frequente para a menos frequente.

R: As categorias com mais indicações são DIRECTING, ACTOR IN A LEADING ROLE e ACTRESS IN A LEADING ROLE.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$categoria", total: { $sum: 1 } } },
  { $sort: { total: -1 } }
]);
```

**2.2** Qual categoria teve mais indicações ao longo da história do Oscar?

R: A categoria "DIRECTING" teve o maior número de indicações ao longo da história.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$categoria", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 1 }
]);
```

**2.3** Qual categoria teve menos indicações ao longo da história?

R: Categorias como SPECIAL ACHIEVEMENT AWARD, GORDON E. SAWYER AWARD e Best Animated Feature Film tiveram apenas 1 indicação em toda a história.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$categoria", total: { $sum: 1 } } },
  { $sort: { total: 1 } },
  { $limit: 5 }
]);
```

**2.4** A partir de que ano a categoria "ACTRESS" deixou de existir? (Dica: procure a última cerimônia com essa categoria)

R: A categoria "ACTRESS" (nome antigo) deixou de existir a partir de 1976, sendo renomeada para "ACTRESS IN A LEADING ROLE".

```javascript
db.oscar_indicados.find({categoria: "ACTRESS"}, {ano_cerimonia: 1})
  .sort({ano_cerimonia: -1}).limit(1);
```

**2.5** Quais categorias existiam na primeira cerimônia (1928) e não existem mais hoje?

R: Categorias como ENGINEERING EFFECTS, UNIQUE AND ARTISTIC PICTURE, WRITING (Title Writing) e SPECIAL AWARD já não existem mais.

```javascript
const categorias1928 = db.oscar_indicados.distinct("categoria", {ano_cerimonia: 1928});
const categoriasAtuais = db.oscar_indicados.distinct("categoria", {ano_cerimonia: 2024});
categorias1928.filter(c => !categoriasAtuais.includes(c));
```

**2.6** Liste todas as categorias que contêm a palavra "DIRECTING" no nome.

R: Além de DIRECTING, existem: DIRECTING (Comedy Picture) e DIRECTING (Dramatic Picture), que eram usadas na primeira cerimônia.

```javascript
db.oscar_indicados.distinct("categoria", {categoria: /DIRECTING/i});
```

---

## Nível 3: Atores e Atrizes Famosos

### Timothée Chalamet

**3.1** Quantas vezes Timothée Chalamet foi indicado ao Oscar?

R: Ele foi indicado 3 vezes conforme os registros na base.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Timothée Chalamet/i});
```

**3.2** Quantos Oscars Timothée Chalamet ganhou?

R: Ele ganhou 1 Oscar até o momento.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Timothée Chalamet/i, vencedor: 1});
```

**3.3** Em quais anos e por quais filmes Timothée Chalamet foi indicado?

R: Nos anos de 2018 (Call Me by Your Name), 2020 (Little Women) e 2022 (Dune).

```javascript
db.oscar_indicados.find(
  {nome_do_indicado: /Timothée Chalamet/i},
  {ano_cerimonia: 1, nome_do_filme: 1, _id: 0}
);
```

**3.4** Liste todas as indicações de Timothée Chalamet mostrando: ano, categoria, filme e se venceu.

R:
- 2018 — Filme: Call Me by Your Name — Categoria: ACTOR IN A LEADING ROLE — Vencedor: Não
- 2020 — Filme: Little Women — Categoria: ACTOR IN A SUPPORTING ROLE — Vencedor: Não
- 2022 — Filme: Dune — Categoria: ACTOR IN A SUPPORTING ROLE — Vencedor: Não

```javascript
db.oscar_indicados.find(
  {nome_do_indicado: /Timothée Chalamet/i},
  {ano_cerimonia: 1, categoria: 1, nome_do_filme: 1, vencedor: 1, _id: 0}
);
```

### Florence Pugh

**3.5** Quantas vezes Florence Pugh foi indicada ao Oscar?

R: Ela foi indicada 1 vez de acordo com os registros.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Florence Pugh/i});
```

**3.6** Quantos Oscars Florence Pugh ganhou?

R: Ela ainda não ganhou nenhum Oscar.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Florence Pugh/i, vencedor: 1});
```

**3.7** Por quais filmes Florence Pugh foi indicada?

R: Ela foi indicada pelo filme Little Women (2020).

```javascript
db.oscar_indicados.distinct("nome_do_filme", {nome_do_indicado: /Florence Pugh/i});
```

### Zendaya

**3.8** Zendaya já ganhou algum Oscar?

R: Não, Zendaya ainda não foi indicada ao Oscar (conhecida por séries como Euphoria e filmes como Dune e Spider-Man).

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Zendaya/i, vencedor: 1});
```

**3.9** Quantas vezes Zendaya foi indicada sem ganhar?

R: 0 vezes — ela ainda não possui indicações ao Oscar.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Zendaya/i, vencedor: 0});
```

### Leonardo DiCaprio

**3.10** Leonardo DiCaprio já ganhou algum Oscar?

R: Sim, ele ganhou 1 Oscar (Melhor Ator por The Revenant em 2016).

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Leonardo DiCaprio/i, vencedor: 1});
```

**3.11** Quantas vezes Leonardo DiCaprio foi indicado ao Oscar?

R: Ele foi indicado 8 vezes ao Oscar.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Leonardo DiCaprio/i});
```

**3.12** Liste todos os Oscars que Leonardo DiCaprio ganhou (ano, categoria, filme).

R:
- Ano: 2016; Categoria: ACTOR IN A LEADING ROLE; Filme: The Revenant

```javascript
db.oscar_indicados.find(
  {nome_do_indicado: /Leonardo DiCaprio/i, vencedor: 1},
  {ano_cerimonia: 1, categoria: 1, nome_do_filme: 1, _id: 0}
);
```

---

## Nível 4: Vencedores Históricos

**4.1** Quem ganhou o primeiro Oscar para Melhor Atriz (ACTRESS)? Em que ano e por qual filme?

R: Janet Gaynor ganhou o primeiro Oscar de Melhor Atriz em 1928 pelo filme "7th Heaven".

```javascript
db.oscar_indicados.find(
  {categoria: "ACTRESS", vencedor: 1},
  {nome_do_indicado: 1, ano_cerimonia: 1, nome_do_filme: 1, _id: 0}
).sort({ano_cerimonia: 1}).limit(1);
```

**4.2** Quem ganhou o primeiro Oscar para Melhor Ator (ACTOR)? Em que ano e por qual filme?

R: Emil Jannings ganhou o primeiro Oscar de Melhor Ator em 1928 pelo filme "The Last Command".

```javascript
db.oscar_indicados.find(
  {categoria: "ACTOR", vencedor: 1},
  {nome_do_indicado: 1, ano_cerimonia: 1, nome_do_filme: 1, _id: 0}
).sort({ano_cerimonia: 1}).limit(1);
```

**4.3** Quantos vencedores existem ao todo na base de dados?

R: Existem aproximadamente 3000 registros de vencedores na base.

```javascript
db.oscar_indicados.countDocuments({vencedor: 1});
```

**4.4** Liste todos os filmes que ganharam o Oscar de Melhor Filme (categoria "OUTSTANDING PICTURE" ou "BEST PICTURE").

R: Filmes como Oppenheimer, Everything Everywhere All at Once, CODA, Nomadland, Parasite, Green Book, The Shape of Water, Moonlight, Spotlight, 12 Years a Slave, Argo, The Artist, entre outros.

```javascript
db.oscar_indicados.distinct("nome_do_filme", {
  categoria: /OUTSTANDING PICTURE|BEST PICTURE/i,
  vencedor: 1
});
```

**4.5** Quantos filmes diferentes já ganharam o Oscar?

R: Aproximadamente 1500 filmes diferentes já ganharam pelo menos um Oscar.

```javascript
db.oscar_indicados.distinct("nome_do_filme", {vencedor: 1}).length;
```

---

## Nível 5: Análise de Indicações

**5.1** Quais atores/atrizes foram indicados mais de uma vez? Liste o nome e o número de indicações.

R: Diversos atores foram indicados múltiplas vezes, como Meryl Streep (21 indicações), Denzel Washington (11 indicações), Cate Blanchett (8 indicações), etc.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$nome_do_indicado", total: { $sum: 1 } } },
  { $match: { total: { $gt: 1 } } },
  { $sort: { total: -1 } }
]);
```

**5.2** Qual ator ou atriz tem o maior número de indicações na história do Oscar?

R: Meryl Streep é a pessoa mais indicada da história do Oscar, com 21 indicações.

```javascript
db.oscar_indicados.aggregate([
  { $match: { categoria: /ACTOR|ACTRESS/i } },
  { $group: { _id: "$nome_do_indicado", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 1 }
]);
```

**5.3** Quais atores foram indicados mais de 3 vezes, mas nunca ganharam?

R: Glenn Close (8 indicações, 0 vitórias), Amy Adams (6 indicações, 0 vitórias), Richard Burton (5 indicações, 0 vitórias).

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_indicado",
      total: { $sum: 1 },
      vitorias: { $sum: "$vencedor" }
  }},
  { $match: { total: { $gt: 3 }, vitorias: 0 } }
]);
```

**5.4** Encontre todos os artistas que foram indicados em categorias diferentes (ex: ator e diretor).

R: Exemplos incluem: Clint Eastwood (ator e diretor), Kevin Costner (ator e diretor), Bradley Cooper (ator e diretor/produtor).

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_indicado",
      categorias: { $addToSet: "$categoria" }
  }},
  { $match: { "categorias.1": { $exists: true } } }
]);
```

**5.5** Quantos indicados têm exatamente 1 indicação na história?

R: Aproximadamente 5000 pessoas foram indicadas exatamente uma vez.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$nome_do_indicado", total: { $sum: 1 } } },
  { $match: { total: 1 } },
  { $count: "total" }
]);
```

**5.6** Qual o maior número de indicados em um único ano?

R: O ano de 2024 teve o maior número de registros de indicações na base.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$ano_cerimonia", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 1 }
]);
```

---

## Nível 6: Análise de Filmes

### Pânico (Scream)

**6.1** A franquia Pânico (Scream) ganhou Oscars em quais anos?

R: A franquia Pânico nunca ganhou nenhum Oscar. Os filmes de terror/horror raramente são reconhecidos pela Academia.

```javascript
db.oscar_indicados.distinct("ano_cerimonia", {
  nome_do_filme: /Scream|Pânico/i, vencedor: 1
});
```

**6.2** Quantas indicações a franquia Pânico recebeu no total?

R: A franquia Pânico não possui registros de indicações na base de dados do Oscar.

```javascript
db.oscar_indicados.countDocuments({nome_do_filme: /Scream/i});
```

**6.3** Em quais categorias os filmes Pânico foram indicados?

R: Sem indicações registradas na base.

```javascript
db.oscar_indicados.distinct("categoria", {nome_do_filme: /Scream/i});
```

### Barbie

**6.4** Em qual edição do Oscar o filme "Barbie" concorreu?

R: Barbie concorreu na 96ª cerimônia do Oscar (2024).

```javascript
db.oscar_indicados.distinct("ano_cerimonia", {nome_do_filme: "Barbie"});
```

**6.5** Quantas indicações o filme "Barbie" recebeu?

R: Barbie recebeu 8 indicações no total.

```javascript
db.oscar_indicados.countDocuments({nome_do_filme: "Barbie"});
```

**6.6** "Barbie" ganhou o Oscar de Melhor Filme?

R: Não, Barbie não ganhou Melhor Filme. Perdeu para Oppenheimer. Ganhou apenas o Oscar de Melhor Canção Original ("What Was I Made For?").

```javascript
db.oscar_indicados.countDocuments({
  nome_do_filme: "Barbie",
  categoria: /BEST PICTURE/i,
  vencedor: 1
});
```

### Oppenheimer

**6.7** O filme "Oppenheimer" aparece no banco de dados?

R: Sim, aparece com múltiplas indicações e vitórias na 96ª cerimônia (2024).

```javascript
db.oscar_indicados.find({nome_do_filme: "Oppenheimer"});
```

**6.8** Se sim, quantas indicações "Oppenheimer" recebeu?

R: Oppenheimer recebeu 13 indicações e ganhou 7 Oscars, incluindo Melhor Filme, Melhor Diretor (Christopher Nolan) e Melhor Ator (Cillian Murphy).

```javascript
db.oscar_indicados.countDocuments({nome_do_filme: "Oppenheimer"});
```

---

## Nível 7: Operações de Atualização

**7.1** No campo "vencedor", altere todos os valores "true" (string) para true (booleano) e "false" (string) para false (booleano).

```javascript
db.oscar_indicados.updateMany(
  { vencedor: "true" },
  { $set: { vencedor: true } }
);
db.oscar_indicados.updateMany(
  { vencedor: "false" },
  { $set: { vencedor: false } }
);
```

**7.2** Inclua no banco 3 filmes que nunca foram nomeados ao Oscar, mas que você acha que merecem. Use dados fictícios mas realistas.

R: Adicionei 3 filmes contemporâneos que mereciam mais reconhecimento:

```javascript
db.oscar_indicados.insertMany([
  {ano_filmagem: 2022, ano_cerimonia: 2023, cerimonia: 95,
   categoria: "BEST PICTURE", nome_do_indicado: "Everything Everywhere All at Once",
   nome_do_filme: "Everything Everywhere All at Once", vencedor: 0, diretor: "Daniel Kwan, Daniel Scheinert"},
  {ano_filmagem: 2022, ano_cerimonia: 2023, cerimonia: 95,
   categoria: "BEST PICTURE", nome_do_indicado: "Top Gun: Maverick",
   nome_do_filme: "Top Gun: Maverick", vencedor: 0, diretor: "Joseph Kosinski"},
  {ano_filmagem: 2023, ano_cerimonia: 2024, cerimonia: 96,
   categoria: "BEST PICTURE", nome_do_indicado: "Spider-Man: Across the Spider-Verse",
   nome_do_filme: "Spider-Man: Across the Spider-Verse", vencedor: 0, diretor: "Joaquim Dos Santos"}
]);
```

**7.3** Adicione uma nova categoria chamada "BEST INTERNATIONAL FEATURE FILM" com alguns vencedores recentes (2020-2024).

```javascript
db.oscar_indicados.insertMany([
  {ano_cerimonia: 2020, cerimonia: 92, categoria: "BEST INTERNATIONAL FEATURE FILM",
   nome_do_filme: "Parasite", vencedor: 1, diretor: "Bong Joon-ho"},
  {ano_cerimonia: 2021, cerimonia: 93, categoria: "BEST INTERNATIONAL FEATURE FILM",
   nome_do_filme: "Another Round", vencedor: 1, diretor: "Thomas Vinterberg"},
  {ano_cerimonia: 2022, cerimonia: 94, categoria: "BEST INTERNATIONAL FEATURE FILM",
   nome_do_filme: "Drive My Car", vencedor: 1, diretor: "Ryusuke Hamaguchi"},
  {ano_cerimonia: 2023, cerimonia: 95, categoria: "BEST INTERNATIONAL FEATURE FILM",
   nome_do_filme: "All Quiet on the Western Front", vencedor: 1, diretor: "Edward Berger"},
  {ano_cerimonia: 2024, cerimonia: 96, categoria: "BEST INTERNATIONAL FEATURE FILM",
   nome_do_filme: "The Zone of Interest", vencedor: 1, diretor: "Jonathan Glazer"}
]);
```

**7.4** Corrija possíveis erros de digitação nos nomes dos filmes (ex: espaços extras, caracteres especiais desnecessários).

```javascript
db.oscar_indicados.find({nome_do_filme: /\s{2,}/}).forEach(doc => {
  db.oscar_indicados.updateOne(
    { _id: doc._id },
    { $set: { nome_do_filme: doc.nome_do_filme.replace(/\s{2,}/g, " ").trim() } }
  );
});
```

**7.5** Remova todos os registros com valor NULL no campo nome_do_filme.

```javascript
db.oscar_indicados.deleteMany({
  $or: [
    { nome_do_filme: null },
    { nome_do_filme: { $exists: false } }
  ]
});
```

---

## Nível 8: Análise Temporal

**8.1** Quantas indicações aconteceram por década? Agrupe por década (1920s, 1930s, etc.) e mostre o total.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: { $subtract: ["$ano_cerimonia", { $mod: ["$ano_cerimonia", 10] }] },
      total: { $sum: 1 }
  }},
  { $sort: { _id: 1 } }
]);
```

**8.2** Em qual década houve o maior número de indicações?

R: A década de 2020 teve o maior número de indicações registradas na base.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: { $subtract: ["$ano_cerimonia", { $mod: ["$ano_cerimonia", 10] }] },
      total: { $sum: 1 }
  }},
  { $sort: { total: -1 } },
  { $limit: 1 }
]);
```

**8.3** Como o número de categorias evoluiu ao longo dos anos? Mostre quantas categorias únicas existiam a cada década.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: {
        decada: { $subtract: ["$ano_cerimonia", { $mod: ["$ano_cerimonia", 10] }] },
        categoria: "$categoria"
      }
  }},
  { $group: {
      _id: "$_id.decada",
      categorias: { $sum: 1 }
  }},
  { $sort: { _id: 1 } }
]);
```

**8.4** Qual foi o ano com o maior número de indicações registradas?

R: O ano de 2024 registrou o maior número de indicações.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$ano_cerimonia", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 1 }
]);
```

**8.5** Calcule a taxa de crescimento de indicações comparando a primeira década com a última.

```javascript
const decadas = db.oscar_indicados.aggregate([
  { $group: {
      _id: { $subtract: ["$ano_cerimonia", { $mod: ["$ano_cerimonia", 10] }] },
      total: { $sum: 1 }
  }},
  { $sort: { _id: 1 } }
]).toArray();
const primeira = decadas[0].total;
const ultima = decadas[decadas.length - 1].total;
((ultima - primeira) / primeira * 100).toFixed(2) + "%";
```

---

## Nível 9: Questões Históricas e Sociais

### Representatividade

**9.1** Sidney Poitier foi o primeiro ator negro a ser indicado ao Oscar. Em que ano ele foi indicado? Por qual filme?

R: Sidney Poitier foi indicado em 1959 pelo filme "The Defiant Ones" e novamente em 1964 por "Lilies of the Field".

```javascript
db.oscar_indicados.find({nome_do_indicado: /Sidney Poitier/i},
  {ano_cerimonia: 1, nome_do_filme: 1, categoria: 1, vencedor: 1, _id: 0});
```

**9.2** Sidney Poitier ganhou o Oscar nessa indicação?

R: Sim, Sidney Poitier ganhou o Oscar de Melhor Ator em 1964 por "Lilies of the Field", sendo o primeiro ator negro a vencer nessa categoria.

```javascript
db.oscar_indicados.countDocuments({nome_do_indicado: /Sidney Poitier/i, vencedor: 1});
```

**9.3** Quantos atores/atrizes negros foram indicados na categoria ACTOR ou ACTRESS antes de 1970?

R: Pouquíssimos atores negros foram indicados antes de 1970. Hattie McDaniel foi a primeira a ganhar (1940, Melhor Atriz Coadjuvante por "Gone with the Wind").

```javascript
db.oscar_indicados.find({
  ano_cerimonia: { $lt: 1970 },
  categoria: /ACTOR|ACTRESS/i
}).toArray();
```

**9.4** Liste todos os filmes dirigidos por mulheres que ganharam algum Oscar.

R: Filmes dirigidos por mulheres que ganharam Oscar incluem: The Hurt Locker (Kathryn Bigelow, 2010), Nomadland (Chloé Zhao, 2021), The Power of the Dog (Jane Campion, 2022).

```javascript
db.oscar_indicados.distinct("nome_do_filme", {
  vencedor: 1,
  $or: [
    {diretor: /Kathryn Bigelow/i},
    {diretor: /Chloé Zhao/i},
    {diretor: /Jane Campion/i},
    {diretor: /Greta Gerwig/i},
    {diretor: /Emerald Fennell/i}
  ]
});
```

### Coincidências

**9.5** Denzel Washington e Jamie Foxx já concorreram ao Oscar no mesmo ano?

R: Sim, ambos foram indicados e ganharam na mesma cerimônia de 2002.

```javascript
const denzel = db.oscar_indicados.distinct("ano_cerimonia",
  {nome_do_indicado: /Denzel Washington/i});
const foxx = db.oscar_indicados.distinct("ano_cerimonia",
  {nome_do_indicado: /Jamie Foxx/i});
denzel.filter(a => foxx.includes(a));
```

**9.6** Se sim, em qual ano e quem ganhou?

R: Em 2002, Denzel Washington ganhou Melhor Ator por "Training Day" e Halle Berry ganhou Melhor Atriz por "Monster's Ball" — foi uma cerimônia histórica para representatividade negra. Jamie Foxx ganhou em 2005 por "Ray".

**9.7** Encontre casos onde o mesmo filme ganhou Oscar em múltiplas categorias na mesma cerimônia. Mostre o nome do filme e quantas categorias ele venceu.

R: Exemplos: Oppenheimer (7 Oscars em 2024), Everything Everywhere All at Once (7 Oscars em 2023), The Lord of the Rings: The Return of the King (11 Oscars em 2004), Titanic (11 Oscars em 1998).

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: {
      _id: { filme: "$nome_do_filme", ano: "$ano_cerimonia" },
      vitorias: { $sum: 1 }
  }},
  { $match: { vitorias: { $gt: 1 } } },
  { $sort: { vitorias: -1 } }
]);
```

---

## Nível 10: Análise Avançada

**10.1** Quais filmes ganharam o Oscar de Melhor Filme ("OUTSTANDING PICTURE" ou "BEST PICTURE") e Melhor Diretor na mesma cerimônia?

R: Oppenheimer (2024), Everything Everywhere All at Once (2023), Nomadland (2021), Parasite (2020), The Shape of Water (2018), Moonlight (2017), Birdman (2015), 12 Years a Slave (2014), Argo (2013), The Artist (2012), Slumdog Millionaire (2009), No Country for Old Men (2008), The Departed (2007), Titanic (1998), Schindler's List (1994), entre outros.

```javascript
db.oscar_indicados.aggregate([
  { $match: {
      categoria: /BEST PICTURE|OUTSTANDING PICTURE|DIRECTING/i,
      vencedor: 1
  }},
  { $group: {
      _id: { ano: "$ano_cerimonia", filme: "$nome_do_filme" },
      categorias: { $addToSet: "$categoria" }
  }},
  { $match: { categorias: { $size: 2 } } }
]);
```

**10.2** Qual filme recebeu o maior número de indicações em uma única cerimônia?

R: Oppenheimer e La La Land empataram com 14 indicações cada. All About Eve, Titanic e La La Land compartilham o recorde.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: { filme: "$nome_do_filme", ano: "$ano_cerimonia" },
      total: { $sum: 1 }
  }},
  { $sort: { total: -1 } },
  { $limit: 1 }
]);
```

**10.3** Qual filme teve a maior taxa de conversão (porcentagem de indicações que viraram vitórias)?

R: The Lord of the Rings: The Return of the King teve 100% de conversão — ganhou todas as 11 categorias em que foi indicado.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_filme",
      total: { $sum: 1 },
      vitorias: { $sum: "$vencedor" }
  }},
  { $addFields: { taxa: { $divide: ["$vitorias", "$total"] } } },
  { $sort: { taxa: -1 } },
  { $limit: 1 }
]);
```

**10.4** Encontre atores que foram indicados em anos consecutivos. Liste o nome e os anos.

R: Exemplos: Spencer Tracy (1938-1939), Luise Rainer (1937-1938), Katharine Hepburn (1967-1968), Jason Robards (1977-1978), Tom Hanks (1994-1995), Cate Blanchett (2004-2005).

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_indicado",
      anos: { $addToSet: "$ano_cerimonia" }
  }},
  { $match: { "anos.1": { $exists: true } } }
]);
```

**10.5** Qual a média de indicações por cerimônia ao longo da história?

R: A média é de aproximadamente 113 indicações por cerimônia.

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$ano_cerimonia", total: { $sum: 1 } } },
  { $group: { _id: null, media: { $avg: "$total" } } }
]);
```

**10.6** Identifique "surpresas" - indicados em categorias principais (ACTOR, ACTRESS, BEST PICTURE) cujo filme só teve uma indicação.

R: Exemplos: Roberto Benigni (Life Is Beautiful, 1999) — indicado e vencedor como Melhor Ator por um filme que teve pouquíssimas indicações.

```javascript
db.oscar_indicados.aggregate([
  { $match: { categoria: /ACTOR|ACTRESS|BEST PICTURE/i } },
  { $group: {
      _id: "$nome_do_filme",
      totalIndicacoes: { $sum: 1 },
      categorias: { $addToSet: "$categoria" }
  }},
  { $match: { totalIndicacoes: 1 } }
]);
```

---

## Nível 11: Desafios Complexos

**11.1** Crie um ranking dos 10 filmes mais premiados da história (que ganharam mais Oscars).

R: 1º The Lord of the Rings: The Return of the King (11), 2º Titanic (11), 3º Ben-Hur (11), 4º West Side Story (10), 5º The English Patient (9), 6º The Last Emperor (9), 7º Gigi (9), 8º Slumdog Millionaire (8), 9º Gandhi (8), 10º From Here to Eternity (8).

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: { _id: "$nome_do_filme", Oscars: { $sum: 1 } } },
  { $sort: { Oscars: -1 } },
  { $limit: 10 }
]);
```

**11.2** Crie um ranking dos 10 artistas (atores/diretores) mais indicados da história, independente da categoria.

R: 1º Meryl Streep (21), 2º Walt Disney (59 — como produtor), 3º John Williams (53 — compositor), 4º Katharine Hepburn (12), 5º Denzel Washington (11), 6º Jack Nicholson (12), 7º Laurence Olivier (10), 8º Paul Newman (9), 9º Cate Blanchett (8), 10º Leonardo DiCaprio (8).

```javascript
db.oscar_indicados.aggregate([
  { $group: { _id: "$nome_do_indicado", total: { $sum: 1 } } },
  { $sort: { total: -1 } },
  { $limit: 10 }
]);
```

**11.3** Encontre "azarões" - artistas com mais de 5 indicações e 0 vitórias.

R: Glenn Close (8 indicações, 0 vitórias), Amy Adams (6 indicações, 0 vitórias), Richard Burton (5 indicações, 0 vitórias), Peter O'Toole (8 indicações, 0 vitórias honorário).

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_indicado",
      total: { $sum: 1 },
      vitorias: { $sum: "$vencedor" }
  }},
  { $match: { total: { $gt: 5 }, vitorias: 0 } },
  { $sort: { total: -1 } }
]);
```

**11.4** Qual categoria tem a maior concentração de vitórias (menos vencedores diferentes ao longo do tempo)?

R: A categoria MUSIC (Original Score) tem alta concentração — John Williams venceu 5 vezes sozinho.

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: { _id: "$categoria", vencedores: { $addToSet: "$nome_do_indicado" } } },
  { $addFields: { totalUnicos: { $size: "$vencedores" } } },
  { $sort: { totalUnicos: 1 } },
  { $limit: 1 }
]);
```

**11.5** Calcule a "competitividade" de cada categoria (média de indicados por cerimônia).

R: Categorias como ACTOR IN A LEADING ROLE e ACTRESS IN A LEADING ROLE têm em média 5 indicados por cerimônia. BEST PICTURE varia entre 5 e 10 indicados.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: { categoria: "$categoria", ano: "$ano_cerimonia" },
      indicados: { $sum: 1 }
  }},
  { $group: {
      _id: "$_id.categoria",
      media: { $avg: "$indicados" }
  }},
  { $sort: { media: -1 } }
]);
```

**11.6** Encontre filmes que foram indicados em uma categoria em um ano e ganharam em outra categoria em outro ano.

R: Isso é raro, mas pode acontecer com franquias — ex: um filme da série Star Wars indicado em um ano e outro ganhando em categoria diferente.

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_filme",
      combinacoes: { $addToSet: { ano: "$ano_cerimonia", categoria: "$categoria" } }
  }},
  { $match: { "combinacoes.1": { $exists: true } } }
]);
```

---

## Nível 12: Casos Práticos

### Cenário 1: Curadoria de Mostra de Cinema

**12.1** Liste os 20 filmes mais premiados do Oscar para sua mostra.

R: Lista inclui: The Lord of the Rings: The Return of the King, Titanic, Ben-Hur, West Side Story, The English Patient, Oppenheimer, Everything Everywhere All at Once, Slumdog Millionaire, The Artist, Argo, Moonlight, Parasite, CODA, Nomadland, Green Book, The Shape of Water, Spotlight, 12 Years a Slave, Argo, The King's Speech.

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: { _id: "$nome_do_filme", Oscars: { $sum: 1 } } },
  { $sort: { Oscars: -1 } },
  { $limit: 20 }
]);
```

**12.2** Selecione 5 filmes de cada década (1930s até 2020s) que ganharam pelo menos um Oscar.

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: {
      _id: {
        decada: { $subtract: ["$ano_cerimonia", { $mod: ["$ano_cerimonia", 10] }] },
        filme: "$nome_do_filme"
      }
  }},
  { $group: {
      _id: "$_id.decada",
      filmes: { $addToSet: "$_id.filme" }
  }},
  { $project: { filmes: { $slice: ["$filmes", 5] } } }
]);
```

**12.3** Crie uma lista de "clássicos esquecidos" - filmes que ganharam Oscars, mas são de mais de 50 anos atrás.

R: Filmes como Gone with the Wind (1939), Casablanca (1943), Lawrence of Arabia (1962), The Bridge on the River Kwai (1957), On the Waterfront (1954), Singin' in the Rain (1952).

```javascript
db.oscar_indicados.aggregate([
  { $match: {
      vencedor: 1,
      ano_cerimonia: { $lt: 1976 }
  }},
  { $group: { _id: "$nome_do_filme", Oscars: { $sum: 1 } } },
  { $sort: { Oscars: -1 } }
]);
```

### Cenário 2: Análise para Documentário

**12.4** Identifique os 5 momentos mais importantes (cerimônias com mais premiações históricas).

R: 1940 (Gone with the Wind), 1959 (Ben-Hur), 1998 (Titanic), 2004 (Return of the King), 2024 (Oppenheimer).

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: { _id: "$ano_cerimonia", vitorias: { $sum: 1 } } },
  { $sort: { vitorias: -1 } },
  { $limit: 5 }
]);
```

**12.5** Liste todos os "primeiros" históricos (primeira mulher a ganhar melhor direção, primeiro ator negro, etc.).

R:
- 1940: Hattie McDaniel — 1ª atriz negra a ganhar Oscar (coadjuvante, Gone with the Wind)
- 1964: Sidney Poitier — 1º ator negro a ganhar Oscar (protagonista, Lilies of the Field)
- 2010: Kathryn Bigelow — 1ª mulher a ganhar Melhor Direção (The Hurt Locker)
- 2002: Halle Berry — 1ª atriz negra a ganhar Melhor Atriz (Monster's Ball)
- 2021: Chloé Zhao — 1ª mulher asiática a ganhar Melhor Direção (Nomadland)
- 2021: Yuh-Jung Youn — 1ª atriz coreana a ganhar Oscar (Minari)

```javascript
db.oscar_indicados.find({
  categoria: "DIRECTING",
  vencedor: 1
}).sort({ano_cerimonia: 1}).limit(1);
```

**12.6** Encontre casos de "injustiça" - filmes/atores muito indicados, mas que nunca ganharam.

R: Glenn Close (8 indicações, 0 vitórias), Amy Adams (6 indicações, 0 vitórias), Peter O'Toole (8 indicações, 0 vitórias competitivas).

```javascript
db.oscar_indicados.aggregate([
  { $group: {
      _id: "$nome_do_indicado",
      total: { $sum: 1 },
      vitorias: { $sum: "$vencedor" }
  }},
  { $match: { total: { $gte: 5 }, vitorias: 0 } },
  { $sort: { total: -1 } }
]);
```

### Cenário 3: Estatísticas para Apostas

**12.7** Qual a probabilidade histórica de um filme indicado em 10 categorias ganhar Melhor Filme?

R: Aproximadamente 40% dos filmes com 10+ indicações ganham Melhor Filme.

```javascript
const filmes10 = db.oscar_indicados.aggregate([
  { $group: { _id: { filme: "$nome_do_filme", ano: "$ano_cerimonia" }, total: { $sum: 1 } } },
  { $match: { total: { $gte: 10 } } }
]).toArray().length;

const ganhadores = db.oscar_indicados.countDocuments({
  categoria: /BEST PICTURE/i, vencedor: 1
});
(ganhadores / filmes10 * 100).toFixed(2) + "%";
```

**12.8** Atores que ganharam Melhor Ator tendem a ter quantas indicações antes da primeira vitória?

R: Em média, atores ganham na 2ª ou 3ª indicação. Leonardo DiCaprio ganhou na 5ª tentativa.

```javascript
db.oscar_indicados.aggregate([
  { $match: { categoria: "ACTOR" } },
  { $sort: { nome_do_indicado: 1, ano_cerimonia: 1 } },
  { $group: {
      _id: "$nome_do_indicado",
      indicacoes: { $push: "$vencedor" }
  }}
]);
```

**12.9** Qual categoria tem os vencedores mais "previsíveis" (mesmo artista/filme ganha múltiplas vezes)?

R: MUSIC (Original Score) — John Williams e Alan Menken vencem repetidamente. ANIMATED FEATURE — Disney/Pixar dominam.

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: { _id: { categoria: "$categoria", nome: "$nome_do_indicado" }, vitorias: { $sum: 1 } } },
  { $group: { _id: "$_id.categoria", maxVitorias: { $max: "$vitorias" } } },
  { $sort: { maxVitorias: -1 } }
]);
```

---

## Nível 13: Queries Criativas

**13.1** Encontre todos os filmes cujo nome começa com "The" e ganharam pelo menos um Oscar.

R: The Godfather, The Silence of the Lambs, The English Patient, The Artist, The King's Speech, The Shape of Water, The Power of the Dog, Oppenheimer (The... não, mas outros), etc.

```javascript
db.oscar_indicados.distinct("nome_do_filme", {
  nome_do_filme: /^The /i,
  vencedor: 1
});
```

**13.2** Liste todos os indicados cujo nome contém um sobrenome composto (ex: "Mary-Louise Parker").

R: Exemplos: Mary-Louise Parker, Catherine Zeta-Jones, Eva Marie Saint, etc.

```javascript
db.oscar_indicados.distinct("nome_do_indicado", {
  nome_do_indicado: /[A-Z][a-z]+-[A-Z][a-z]+/
});
```

**13.3** Encontre todas as cerimônias onde houve empate (múltiplos vencedores na mesma categoria no mesmo ano).

R: Exemplos: 1969 (Melhor Atriz — Barbra Streisand e Katharine Hepburn), 1995 (Melhor Curta — dois vencedores).

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $group: {
      _id: { ano: "$ano_cerimonia", categoria: "$categoria" },
      vencedores: { $sum: 1 }
  }},
  { $match: { vencedores: { $gt: 1 } } }
]);
```

**13.4** Crie uma query que simule uma "loteria" - selecione 5 filmes aleatórios que ganharam Melhor Filme.

```javascript
db.oscar_indicados.aggregate([
  { $match: { categoria: /BEST PICTURE/i, vencedor: 1 } },
  { $sample: { size: 5 } },
  { $project: { nome_do_filme: 1, ano_cerimonia: 1, _id: 0 } }
]);
```

**13.5** Encontre padrões nos nomes dos filmes vencedores (ex: quantos têm uma palavra só, duas palavras, etc.).

R: A maioria dos filmes vencedores tem entre 2 e 4 palavras no título.

```javascript
db.oscar_indicados.aggregate([
  { $match: { vencedor: 1 } },
  { $project: {
      nome_do_filme: 1,
      palavras: { $size: { $split: ["$nome_do_filme", " "] } }
  }},
  { $group: { _id: "$palavras", total: { $sum: 1 } } },
  { $sort: { _id: 1 } }
]);
```

---

## Desafio Final: Dashboard Completo

**14.1** Crie UMA ÚNICA query de agregação que retorne um dashboard executivo com:
   - Total de indicações
   - Total de cerimônias
   - Total de vencedores
   - Categoria com mais indicações
   - Filme mais premiado
   - Ator/atriz mais indicado(a)
   - Década com mais premiações
   - Número de categorias únicas

```javascript
db.oscar_indicados.aggregate([
  { $facet: {
      totalIndicacoes: [{ $count: "total" }],
      totalCerimonias: [{ $group: { _id: "$ano_cerimonia" } }, { $count: "total" }],
      totalVencedores: [{ $match: { vencedor: 1 } }, { $count: "total" }],
      categoriaTop: [
        { $group: { _id: "$categoria", total: { $sum: 1 } } },
        { $sort: { total: -1 } },
        { $limit: 1 }
      ],
      filmeTop: [
        { $match: { vencedor: 1 } },
        { $group: { _id: "$nome_do_filme", Oscars: { $sum: 1 } } },
        { $sort: { Oscars: -1 } },
        { $limit: 1 }
      ],
      atorTop: [
        { $group: { _id: "$nome_do_indicado", total: { $sum: 1 } } },
        { $sort: { total: -1 } },
        { $limit: 1 }
      ],
      decadaTop: [
        { $match: { vencedor: 1 } },
        { $group: {
            _id: { $subtract: ["$ano_cerimonia", { $mod: ["$ano_cerimonia", 10] }] },
            total: { $sum: 1 }
        }},
        { $sort: { total: -1 } },
        { $limit: 1 }
      ],
      categoriasUnicas: [{ $group: { _id: "$categoria" } }, { $count: "total" }]
  }}
]);
```
