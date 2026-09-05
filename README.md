# O Voto, A Bala e a Floresta

**Uma análise de enquadramento da cobertura eleitoral em veículos de mídia local da Amazônia Legal.**

🔎 **Explorador interativo: [voto-bala-floresta.vercel.app](https://voto-bala-floresta.vercel.app)**

Este repositório guarda a ferramenta de exploração de uma pesquisa de iniciação científica (PIBIC)
da Escola de Comunicação, Mídia e Informação da FGV. A pergunta não é apenas sobre o que a mídia
local amazônica falou durante as eleições municipais de 2024, mas sobre como ela delimitou o que
mostrou: quais assuntos ganharam contorno próprio, quais se dissolveram no ruído e o que essa
organização revela sobre as rotinas produtivas dessas redações.

O referencial é a Teoria do Enquadramento, na linhagem de Goffman, Entman, Gamson & Modigliani,
Gitlin e Tuchman. O método é computacional, mas a decisão interpretativa está declarada e separada
do que o modelo fez sozinho.

---

## O corpus

Todas as publicações de Instagram de veículos jornalísticos sediados nas nove capitais da Amazônia
Legal (Belém, Boa Vista, Cuiabá, Macapá, Manaus, Palmas, Porto Velho, Rio Branco e São Luís),
entre 16 de agosto e 30 de outubro de 2024, os 76 dias do período eleitoral. A coleta foi executada
pelo DAPP Lab da FGV; a amostragem partiu do cadastro do Atlas da Notícia, edições 2024 e 2025,
cruzado e verificado manualmente.

| Etapa | Publicações |
|---|---:|
| Coletadas | 78.497 |
| Duplicadas na entrega da coleta | −49 |
| Veículos da Paraíba, trazidos por engano e fora da Amazônia Legal | −3.085 |
| **Recorte analítico** | **75.363** |
| ↳ classificadas em algum cluster | 67.265 |
| ↳ não classificadas, declaradas | 8.098 |

São 195 perfis de veículos no recorte analítico, agrupados pelo modelo em 120 clusters e, por
decisão interpretativa do pesquisador, em 13 macro-temas.

É um território habitado, desigual e mediado: blogs de uma pessoa só ao lado de jornais centenários,
redações precárias ao lado de portais que reproduzem release de assessoria.

## Como foi feito

1. **Amostragem.** Cadastro do Atlas da Notícia recortado para as capitais dos nove estados,
   com verificação manual.
2. **Coleta.** DAPP Lab da FGV, 78.497 publicações de 211 perfis.
3. **Pré-processamento.** Correção de mojibake com dupla passagem de ftfy, normalização Unicode e
   três camadas de stopwords próprias: jargão de portal ("link na bio", "arraste"), os handles dos
   próprios veículos e termos genéricos de alta frequência. Sete rodadas de depuração.
4. **Clusterização.** BERTopic guiado por sementes temáticas: embeddings multilíngues
   (`paraphrase-multilingual-MiniLM-L12-v2`), UMAP, HDBSCAN, c-TF-IDF. Resultado: 120 clusters.
5. **Tipologia.** Os 120 clusters foram reunidos em 13 macro-temas. Esta é a única etapa em que a
   decisão é humana, e está declarada como tal.

### O tratamento do residual

O HDBSCAN deixou 35,3% das publicações como ruído. Descartar tudo perderia um terço da base;
reatribuir tudo forçaria publicações a tópicos com os quais têm pouca relação. A solução foi um
critério de referência interna: uma publicação só é reatribuída a um tópico se sua similaridade ao
centroide for maior ou igual ao percentil 10 dos documentos genuínos daquele mesmo tópico.

A decisão foi testada. A hierarquia dos temas foi recalculada em três cenários (só o núcleo do
HDBSCAN, o critério adotado e a reatribuição irrestrita) e a correlação de postos entre eles fica
entre 0,89 e 0,99. A conclusão da pesquisa não depende dessa escolha.

As 8.098 publicações que permaneceram sem cluster não são lixo, e o explorador permite trazê-las de
volta ao mapa. Medidas contra o restante da base, elas têm mediana de 450 caracteres: são matérias
jornalísticas completas, de pauta singular demais para formar um agrupamento. O que sobrou do
modelo é jornalismo disperso, não conteúdo descartável.

## O que o explorador mostra

Três visões da mesma base, lidas de distâncias diferentes.

**Mapa de posts.** As 75.363 publicações, uma por ponto. A posição vem dos embeddings de cada texto,
projetados em duas dimensões por MDS de referência sobre os 120 centroides. Clicar num ponto abre o
texto, o veículo, o cluster e as cinco publicações semanticamente mais próximas, com retas ligando
cada uma ao seu lugar no mapa.

**Mapa de clusters.** Os 120 agrupamentos, com área proporcional ao volume dentro do recorte em
vigor. O denominador é absoluto, o maior cluster da base inteira, de modo que filtrar encolhe as
bolhas de verdade em vez de apenas reembaralhar tamanhos.

**Rede.** Macro-temas ou clusters ligados quando a similaridade de cosseno entre seus centroides
supera um limiar ajustável. As posições vêm de stress majorization sobre a matriz inteira,
calculada uma vez: o desenho abre sempre igual e mudar o limiar não o reembaralha.

Sobre as três visões incidem filtros facetados por macro-tema, estado, suporte do veículo, perfil de
publicação, engajamento, período e busca textual. Cada contador mostra quantas publicações aquela
opção traz considerando os outros filtros já ativos. O recorte em vigor fica declarado
permanentemente na tela, e cada mudança entra num histórico numerado, com quantas publicações
entraram ou saíram. O recorte pode ser exportado em CSV, e o mapa em PNG.

## Alguns achados que a ferramenta deixa ver

A proximidade entre clusters foi medida no espaço completo de 384 dimensões, no nível de cluster e
não de tema, e cada agrupamento proposto passou por teste de permutação contra 4.000 conjuntos
aleatórios do mesmo tamanho. Ganho é a coesão interna do agrupamento menos sua similaridade média
com os clusters de fora.

| Campo | Clusters | Ganho | z |
|---|---:|---:|---:|
| Político-institucional (Eleitoral, Política, Educação) | 39 | +0,247 | +15,8 |
| Núcleo de ocorrência (*fait divers*) | 7 | +0,213 | +5,2 |
| Crime e Estado (Segurança, Violência) | 7 | +0,191 | +4,6 |
| Gestão material (Ambiente, Infra, Economia, Informação) | 30 | +0,100 | +5,8 |
| Campo da vivência | 41 | +0,058 | +3,7 |

Dois resultados que o mapa torna visíveis:

**O núcleo de ocorrência é um gênero, não um assunto.** Sete clusters (acidente de veículo, vítima
de crime, aeronave, corpo encontrado, tiros, motociclistas, bombeiros) somam 7.359 publicações e
formam o segundo agrupamento mais coeso da análise, acima do campo Crime e Estado. É o fato
consumado de registro policial e de trânsito, o *fait divers* da tradicional editoria de cidades.
A tipologia temática corta esse gênero ao meio, distribuindo-o entre *Sociedade e cotidiano* e
*Violência e criminalidade*. Fora dele ficam, significativamente, os clusters de violência de
gênero, enquadrados como pauta de direitos e não como ocorrência.

**Há uma assimetria de institucionalidade.** A cobertura ancorada em fontes oficiais (campanha,
política, gestão pública) é densamente estruturada, com coesão de 0,587. A cobertura
não-institucional é real, mas fragmentada: o "campo da vivência" reúne 41 clusters e 41% da base
classificada, e supera com folga conjuntos aleatórios de mesmo tamanho, porém se parte em dois
núcleos que se aproximam apenas 0,325 entre si, menos do que a região inteira se aproxima do campo
político-institucional. Não é um campo, são dois campos ocupando a mesma zona do mapa, a zona do
que não é institucional nem gerencial. O que chega por release se organiza; o resto se dispersa.

É desse desenho que vem o título. O Voto aparece colado à política institucional. A Bala forma um
par isolado entre segurança e violência, na periferia do mapa. A Floresta se aproxima menos da
ecologia do que da infraestrutura e da gestão material.

## O que a ferramenta não permite afirmar

Estes avisos não são rodapé legal, são parte do argumento. Dentro do explorador, cada um aparece
junto do controle a que se refere.

**Proximidade visual não é proximidade semântica.** A projeção retém 27,9% da variância, e a
distorção foi medida: apenas 7,5% dos vizinhos mais próximos no espaço original também são os mais
próximos na tela (24,0% considerando os seis primeiros; correlação de 0,623 entre distância no plano
e distância real). Dois clusters podem parecer colados sem estar entre os mais semelhantes. Para
afirmar proximidade, use o painel de detalhe ou a rede, que trabalham com as distâncias reais.

**Engajamento acompanha o tamanho da audiência.** No decil mais engajado, dois veículos concentram
48% das publicações e o Pará salta de 17% para 37% do total. Filtrar por engajamento é, em boa
medida, filtrar por porte de veículo.

**Televisão e rádio não sustentam comparação.** Restam 352 publicações de televisão e 1.520 de
rádio, e o segmento "rádio" é 85% um único veículo. Diferenças ali descrevem um veículo, não um
suporte.

**O segmento é autodeclarado** e vem do cadastro do Atlas, podendo estar desatualizado. Caso
conhecido: o Gazeta Digital, de Cuiabá, está cadastrado como rádio apesar do nome e do formato.

**A UF é a do veículo, não a do assunto.** Um veículo de Manaus cobrindo Brasília conta como AM.

**O recorte não é exatamente "nove capitais".** Cinco veículos do cadastro têm sede fora da capital
(Rondonópolis, Lucas do Rio Verde, Canarana, Ariquemes e Açailândia), somando 684 publicações. Há um
filtro para excluí-los.

**Sete por cento das publicações não têm veículo identificado**, sobretudo perfis pessoais de
jornalistas ausentes do cadastro. O vínculo com o Atlas cobre 180 dos 195 perfis, ou 93,0% das
publicações.

**O texto exibido é o início da legenda.** Para o site carregar em tempo razoável, cada publicação
traz os primeiros 220 caracteres. O link para o post original está no painel de detalhe, e a análise
foi feita sobre o texto inteiro.

## Ficha técnica

**Erik Rolin**, bolsista PIBIC, Comunicação Digital, FGV ECMI.
Orientação: **prof. Eurico Matos**.
Coleta: DAPP Lab da FGV. Amostragem a partir do Atlas da Notícia.
Iniciação Científica PIBIC, 2025 a 2026.

O design da interface foi desenvolvido com o Claude Design; a implementação é HTML, CSS e
JavaScript sem framework nem dependência instalada.

---

<details>
<summary><strong>Notas técnicas: como este repositório funciona</strong></summary>

### Estrutura

| Arquivo | O que é |
|---|---|
| `index.html` | apresentação do projeto e painel de configuração do recorte inicial |
| `explorador.html` | a ferramenta: as três visões, filtros, painel de detalhe, linha do tempo |
| `dados/` | os dados gerados pelo pipeline da pesquisa |
| `vercel.json` | cabeçalhos de cache e de segurança |

Não há build, framework nem dependência instalada. As duas páginas são HTML com CSS e JavaScript
embutidos. A única coisa que vem de fora são as fontes Archivo e Archivo Narrow, do Google Fonts, e
a ferramenta é usável antes de elas chegarem. O explorador precisa de `DecompressionStream`, presente
em Chrome, Edge, Firefox e Safari recentes.

### Os dados

Os arquivos de `dados/` não são editados à mão: saem do `gerar_site.py`, no repositório do pipeline
da pesquisa, que lê os embeddings e os CSVs de clusterização.

| Arquivo | Tamanho | Quando é baixado |
|---|---|---|
| `dados/meta.json` | ~42 KB | primeiro, antes de tudo |
| `dados/posts.bin.gz` | ~1,0 MB | na carga inicial, com barra de progresso |
| `dados/detalhe.bin.gz` | ~0,9 MB | depois que o mapa aparece (vizinhos semânticos) |
| `dados/textos.bin.gz` | ~6,3 MB | depois que o mapa aparece (textos, links, busca textual) |

O mapa pinta com pouco mais de 1 MB. Textos e vizinhos chegam em segundo plano: até lá, o painel de
detalhe avisa que está carregando e a busca funciona só por macro-tema e estado.

A ordem das colunas em `posts.bin.gz` é um contrato. Se mudar no `gerar_site.py`, tem de mudar junto
em `lerColunas()`, dentro do `explorador.html`.

### Rodar localmente

Precisa de um servidor HTTP. Abrir com duplo clique não funciona, porque `fetch` não lê `file://`:

```bash
python -m http.server 8000
# abra http://localhost:8000
```

### Publicar

Deploy estático na Vercel, sem build command e sem output directory, com o Root Directory no padrão.
Quando os dados mudarem, rode o gerador, faça commit de `dados/` e empurre: a Vercel republica
sozinha.

</details>
