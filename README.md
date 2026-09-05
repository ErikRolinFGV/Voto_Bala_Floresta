# Site — *O Voto, A Bala e a Floresta*

Site estático com duas páginas:

| Arquivo | O que é |
| --- | --- |
| `index.html` | apresentação do projeto e painel de configuração do recorte inicial |
| `explorador.html` | a ferramenta: mapa de posts, mapa de clusters, rede, filtros facetados, linha do tempo |
| `dados/` | os dados gerados pelo pipeline (ver abaixo) |

Não há build, framework nem dependência instalada. As duas páginas são HTML com CSS e JS
embutidos; a única coisa que vem de fora são as fontes Archivo e Archivo Narrow, do Google Fonts,
e a ferramenta é usável antes de elas chegarem.

## Gerar os dados

Os arquivos de `dados/` **não** são editados à mão: saem do pipeline da IC.

```bash
python gerar_site.py      # na raiz do repositório, um nível acima desta pasta
```

Ele lê `data/embeddings_minilm.npy`, `data/Clusters_p10_posts.csv`, `data/Clusters_p10_temas.csv`
e `data/Clusters_claude_s1_reduce_outliers.csv`, e grava:

| Arquivo | Tamanho | Quando é baixado |
| --- | --- | --- |
| `dados/meta.json` | ~42 KB | primeiro, antes de tudo |
| `dados/posts.bin.gz` | ~1,0 MB | na carga inicial, com barra de progresso |
| `dados/detalhe.bin.gz` | ~0,9 MB | depois que o mapa aparece (vizinhos semânticos) |
| `dados/textos.bin.gz` | ~6,3 MB | depois que o mapa aparece (textos, links, busca textual) |

O mapa pinta com pouco mais de 1 MB. Textos e vizinhos chegam em segundo plano: até lá, o painel
de detalhe diz que está carregando e a busca funciona só por macro-tema e estado.

**A ordem das colunas em `posts.bin.gz` é um contrato.** Se mudar em `gerar_site.py`, tem de mudar
junto em `lerColunas()` no `explorador.html`.

## Rodar localmente

Precisa de um servidor HTTP — abrir com duplo clique não funciona, porque `fetch` não lê `file://`:

```bash
cd site
python -m http.server 8000
# abra http://localhost:8000
```

## Publicar na Vercel

**Esta pasta é o seu próprio repositório Git**, separado do repositório da IC. Isso é
deliberado: o repositório da IC carrega, no histórico, um CSV de 152 MB e um `.npy` de 115 MB,
e o GitHub recusa qualquer arquivo acima de 100 MB. O site pesa ~8 MB e sobe sem problema.

Primeira vez:

```bash
cd site
git init -b main
git add .
git commit -m "Explorador O Voto, A Bala e a Floresta"
git remote add origin https://github.com/<usuario>/<repositorio>.git
git push -u origin main
```

Na Vercel: **Add New → Project → Import** este repositório → Framework Preset **Other** →
Root Directory fica no padrão (`./`, porque a raiz do repositório já é esta pasta) → **Deploy**.
Sem build command e sem output directory.

O `vercel.json` define o cache dos dados (10 minutos no navegador, revalidação em segundo plano
por um dia) e dois cabeçalhos de segurança.

### Quando os dados mudarem

Rode `python gerar_site.py` na pasta da IC — ele grava direto em `site/dados/`. Depois:

```bash
cd site
git add dados
git commit -m "atualiza os dados"
git push
```

A Vercel republica sozinha. Se um dia os dados passarem de umas dezenas de MB, vale mover
`dados/` para Git LFS ou para um bucket externo.

## Limites conhecidos

- O texto de cada publicação vai truncado em 220 caracteres, para o download caber. O link para o
  post original está no painel de detalhe; a análise foi feita sobre o texto inteiro.
- A rede é calculada uma vez sobre a base inteira e não responde aos filtros de recorte — os
  controles inertes somem da aba em vez de ficarem lá enganando.
- `DecompressionStream` é obrigatório: Chrome, Edge, Firefox e Safari recentes têm; navegadores
  muito antigos veem uma mensagem explicando, em vez de uma tela branca.
