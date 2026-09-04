# CutPic

Divida, baixe e imprima. Sem burocracia!

CutPic é uma ferramenta web que divide qualquer imagem em uma grade de linhas
e colunas (de 1 a 12) e gera automaticamente um PDF em A4 — uma parte da
imagem por página — pronto para imprimir e montar como pôster. Todo o
processamento acontece **localmente, no navegador**: não há upload para
servidor, não há cadastro e não há banco de dados.

## Estrutura do projeto

```
cutpic/
├── index.html              Página principal com a ferramenta
├── privacidade.html         Política de Privacidade
├── termos.html               Termos de Uso
├── assets/
│   ├── css/
│   │   └── style.css        Estilos (design system completo)
│   └── js/
│       └── script.js        Lógica de upload, grade, prévia e geração de PDF
└── README.md                 Este arquivo
```

## Como usar

1. Baixe/extraia a pasta `cutpic`.
2. Abra o arquivo `index.html` diretamente no navegador (duplo clique), ou
   sirva a pasta com qualquer servidor estático — por exemplo:
   ```bash
   cd cutpic
   python3 -m http.server 8080
   # depois acesse http://localhost:8080
   ```
3. Na seção "Ferramenta", arraste uma imagem (PNG, JPG ou WEBP) para a área
   de upload, ou clique para selecioná-la do seu computador.
4. Ajuste o número de **linhas** e **colunas** (1 a 12 cada). A prévia mostra
   a grade de corte sobreposta à imagem em tempo real, e o contador exibe
   quantas páginas serão geradas (linhas × colunas).
5. Clique em **"Gerar e baixar PDF"**. O navegador cria o PDF localmente e
   inicia o download automaticamente — cada página traz uma parte da imagem
   em tamanho A4, com marcas de corte, guias de alinhamento e uma referência
   de posição no canto (ex.: `L1 · C2 — 3/12`) para facilitar a montagem do
   pôster.

## Marcas de corte e guias de alinhamento

Tanto a prévia (no `<canvas>`) quanto o PDF final trazem o mesmo sistema de
marcas, pensado para dar precisão na hora de cortar e colar as partes:

- **Marcas de corte (crop marks):** pequenos traços em "L" próximos a cada
  vértice da grade — nos quatro cantos de cada página e em cada interseção
  interna —, com um pequeno vão até o ponto exato de corte, no padrão usado
  por gráficas profissionais. Isso evita que a marca "manche" o ponto exato
  onde a tesoura ou estilete deve passar.
- **Guia de trim:** uma linha tracejada fina contornando a área útil de cada
  página, servindo de referência contínua para cortar com régua.
- **Ticks de alinhamento:** pequenas marcas no meio de cada borda da área
  útil, que se repetem na mesma posição na página vizinha — úteis para
  alinhar duas partes com exatidão na hora de colar.
- **Margem técnica:** cada página do PDF reserva 8 mm de margem branca ao
  redor da imagem (área útil = 210×297 mm menos 8 mm de cada lado). Essa
  margem existe porque a maioria das impressoras não imprime até a borda
  exata do papel — sem ela, a imagem seria cortada de forma inconsistente
  entre as páginas. Por ser sempre a mesma margem, o tamanho da área
  impressa é idêntico em todas as páginas, o que garante o encaixe correto
  entre as partes.

Não é necessária conexão com servidores próprios: a única dependência externa
é o carregamento, via CDN, da fonte tipográfica (Google Fonts) e da
biblioteca [jsPDF](https://github.com/parallax/jsPDF), usada para montar o
arquivo PDF diretamente no navegador.

## Tecnologias utilizadas

- **HTML5** — estrutura semântica das três páginas.
- **CSS3** — design system próprio (variáveis CSS, grid, flexbox, media
  queries para responsividade em computador, tablet e celular).
- **JavaScript (vanilla, sem frameworks)** — upload por clique/arrastar,
  desenho da grade de prévia em `<canvas>`, corte da imagem via
  `CanvasRenderingContext2D.drawImage` e geração do PDF com jsPDF.

## Decisões de privacidade (resumo técnico)

- A imagem enviada é lida com a API `File`/`FileReader` do navegador e
  mantida apenas como um objeto `Image` em memória (RAM), referenciado por
  uma `Object URL` local (`URL.createObjectURL`).
- Nenhum `fetch`/`XMLHttpRequest` envia o conteúdo da imagem para qualquer
  endpoint — o projeto não possui back-end.
- A aplicação não usa `localStorage`, `sessionStorage`, cookies próprios ou
  qualquer banco de dados.
- Ao trocar de imagem, recarregar ou fechar a página, a `Object URL` é
  revogada (`URL.revokeObjectURL`), liberando o recurso da memória do
  navegador.
- Os detalhes completos estão descritos, em linguagem acessível, nas páginas
  `privacidade.html` e `termos.html`.

## Limites técnicos

- Tamanho máximo de arquivo: 30 MB.
- Maior lado da imagem: até 5000 px (limite de segurança para não travar o
  navegador durante o corte).
- Grade de corte: de 1 a 12 linhas e de 1 a 12 colunas.

## Personalização

Toda a identidade visual (cores, tipografia, espaçamentos) está centralizada
em variáveis CSS no topo de `assets/css/style.css`, o que facilita ajustar a
paleta ou a tipografia sem precisar alterar o restante do código.

## Responsividade

O layout foi construído mobile-first, com breakpoints em `assets/css/style.css`
para 860px, 760px, 640px, 460px e 360px, cobrindo computador, tablet e
celular. Em telas estreitas: a navegação do cabeçalho se reduz (o link "Como
funciona" é ocultado abaixo de 460px, já que a mesma seção é acessível pelo
botão principal do topo), o painel da ferramenta empilha a área de upload
sobre os controles, os campos de linhas/colunas ganham alvos de toque
maiores, e o rodapé passa a empilhar verticalmente. Os campos numéricos usam
fonte de 16px ou mais para evitar zoom automático no iOS ao tocar neles.

---

© 2026 CutPic. Todos os direitos reservados. Desenvolvido por Dadson Gomes.


## Vercel Web Analytics

O projeto inclui o script oficial do Vercel Web Analytics nas páginas HTML. Depois de publicar esta versão na Vercel, abra o projeto > Analytics e confirme a coleta. O painel pode levar alguns instantes para começar a exibir os dados.

O Analytics é usado para contar visitantes e visualizações de página. O conteúdo das imagens processadas pelo CutPic não é enviado para o Analytics.
