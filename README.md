# Colinha de votação — como usar

## Estrutura de pastas
```
colinha/
├── index.html
├── style.css
├── config.js
├── app.js
├── candidatos.csv   ← troque pela sua base real do TSE
└── fotos/           ← coloque aqui as fotos baixadas do TSE
```

## Passo a passo

1. **Troque `candidatos.csv`** pela sua base de dados real, exportada do TSE
   (o arquivo de exemplo tem só 4 linhas, pra você testar).
   As colunas que o app usa são: `CD_CARGO`, `SQ_CANDIDATO`, `NR_CANDIDATO`,
   `NM_URNA_CANDIDATO` (ou `NM_CANDIDATO`), `SG_PARTIDO`, `SG_UF`.
   Se o seu CSV usa vírgula em vez de ponto e vírgula como separador, troque
   `delimiter: ";"` por `delimiter: ","` em `app.js`.

2. **Coloque as fotos na pasta `fotos/`**, exatamente como baixadas do TSE
   (ex.: `FMG130002537202_div.jpg`). O app monta o nome do arquivo sozinho a
   partir de `SG_UF` + `SQ_CANDIDATO`, então não precisa renomear nada.

3. **Ajuste `config.js`** — é o único lugar que você deveria precisar editar:
   - `CARGOS`: quais cargos aparecem, em que ordem, quantos dígitos cada um
     tem na urna, e o código `CD_CARGO` correspondente (os códigos do TSE
     são: 1 = Presidente, 3 = Governador, 5 = Senador, 6 = Deputado Federal,
     7 = Deputado Estadual — confira se batem com os valores reais no seu CSV,
     porque podem variar um pouco entre eleições).
   - O campo `fixo` é opcional: preenche um número de cara (foi assim que o
     site original já deixa "456 — Aécio Neves" preenchido no senador).

4. **Rode com um servidor local** (não abra o `index.html` direto clicando
   duas vezes — o navegador bloqueia a leitura do CSV por segurança quando
   o arquivo é aberto via `file://`). Qualquer um destes serve:
   ```bash
   # opção 1, com Python já instalado
   python3 -m http.server 8000

   # opção 2, com Node instalado
   npx serve .
   ```
   Depois abra `http://localhost:8000` no navegador.

## Como funciona por dentro

- `app.js` lê o CSV inteiro uma vez, ao carregar a página, e monta um índice
  em memória: `INDEX[cargo][numero] → {nome, partido, foto}`. Isso deixa a
  busca instantânea enquanto a pessoa digita, sem precisar de backend.
- Cada cargo vira um grupo de `<input>` de um dígito, que avança o foco
  automaticamente e volta com backspace — igual ao site original.
- Quando o número digitado bate com o número de dígitos configurado, o app
  procura no índice e, se achar, preenche nome, partido e foto (via `<img
  onerror>` — se a foto não existir, o app só deixa em branco, não quebra).
- "Gerar colinha" usa a biblioteca `html2canvas` para tirar um "print" do
  cartão inteiro e baixar como PNG — é o mesmo truque que sites assim
  costumam usar pra gerar a imagem que a pessoa salva no celular.

## Publicando online

Como é tudo estático (HTML/CSS/JS + um CSV + imagens), dá pra hospedar de
graça em qualquer serviço de site estático: Cloudflare Pages, Netlify,
Vercel, GitHub Pages. Basta subir a pasta inteira — não precisa de servidor
próprio nem banco de dados.

## Um ponto de atenção

O CPF de candidatos (`NR_CPF_CANDIDATO`) e outras colunas sensíveis que
aparecem no CSV oficial do TSE não são usados aqui e não deveriam ir para o
site público — mantenha só as colunas necessárias (nome, partido, número,
foto) se for publicar o `candidatos.csv` em um site acessível a qualquer
pessoa.
