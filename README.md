# Setup de Mac para estrategista de conteúdo

O caminho na ordem certa, do Mac recém-formatado até a operação rodando — escrito por alguém que não é desenvolvedora, para quem também não é.

Se você já tentou seguir um tutorial de programador e travou na terceira linha porque ele partia de coisas que "todo mundo sabe", este documento existe por causa disso.

Tempo: cerca de uma hora, quase toda de espera. Pré-requisito: nenhum.

---

## Antes de começar, três coisas que ninguém explica

**O Terminal não é perigoso.** É uma janela onde você digita o nome de um programa em vez de clicar num ícone. Abra com `Cmd + Espaço`, digite "Terminal", Enter. Nada do que está aqui apaga arquivo seu.

**Copiar e colar comando é normal.** Todo mundo faz isso, inclusive quem programa há vinte anos. O que importa é entender o que o comando resolve, e é isso que cada seção explica antes de mandar você colar.

**Se der erro, leia a última linha.** A mensagem costuma ser longa e assustadora, mas a informação útil quase sempre está no fim. Colar essa última linha no Claude resolve a maioria dos casos.

---

## Passo 1 — Homebrew, o instalador de tudo

Sem ele, cada programa vira uma caçada por site de download. Com ele, instalar é uma linha.

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Ao terminar, ele imprime duas ou três linhas pedindo para você rodá-las. **Rode.** É o passo que ensina o Mac a encontrar o que o Homebrew instalar. Pular isso é a causa número um de "instalei e o computador diz que não existe".

Conferir:

```bash
brew --version
```

---

## Passo 2 — Node, Python e os isoladores

O Node faz funcionar as ferramentas escritas em JavaScript. O pipx e o uv cuidam das ferramentas escritas em Python.

```bash
brew install node pipx uv
pipx ensurepath
```

O `pipx ensurepath` avisa o Mac sobre a pasta `~/.local/bin`, que é onde os programas de Python vão morar. **Feche o Terminal e abra de novo** para a mudança valer.

> **Por que não instalar Python direto?** Porque uma ferramenta pede uma versão de biblioteca, outra pede a versão anterior, e as duas brigam. O pipx dá uma caixa fechada para cada uma. É a diferença entre trabalhar em cópias e trabalhar no arquivo original.

---

## Passo 3 — Claude Code

O centro da operação.

```bash
npm install -g @anthropic-ai/claude-code
```

Se der erro de permissão, o motivo é que o npm está tentando escrever numa pasta do sistema. A correção limpa é dar a ele uma pasta sua:

```bash
mkdir -p ~/.npm-global
npm config set prefix ~/.npm-global
echo 'export PATH=~/.npm-global/bin:$PATH' >> ~/.zshrc
source ~/.zshrc
npm install -g @anthropic-ai/claude-code
```

Para usar: entre na pasta do projeto e chame.

```bash
cd ~/meu-projeto
claude
```

**A parte que muda o jogo.** Crie um arquivo chamado `CLAUDE.md` dentro da pasta do projeto e escreva ali quem é o cliente, qual é o tom de voz, o que nunca pode aparecer no texto. Ele lê esse arquivo sozinho em toda conversa. Deixa de ser assistente genérico e passa a trabalhar dentro das suas regras.

---

## Passo 4 — Mídia (a parte com pegadinha)

Aqui mora a única armadilha real deste documento, e ela me custou uma tarde inteira.

```bash
brew install ffmpeg ffmpeg@7 yt-dlp whisper-cpp tesseract poppler
pipx install whisperx
```

> ### A pegadinha do ffmpeg
>
> O ffmpeg 8, que é o que o Homebrew instala hoje como padrão, **quebra a separação de locutores** na transcrição. A biblioteca que faz esse trabalho só suporta as versões 4 a 7.
>
> Por isso a linha acima instala as duas: a 8 para uso geral e a **7 fixada em paralelo**, que fica guardada em `/opt/homebrew/opt/ffmpeg@7/`.
>
> Quem precisa da versão 7 aponta para ela explicitamente, no começo do script:
>
> ```bash
> export PATH="/opt/homebrew/opt/ffmpeg@7/bin:$PATH"
> export DYLD_FALLBACK_LIBRARY_PATH="/opt/homebrew/opt/ffmpeg@7/lib:$DYLD_FALLBACK_LIBRARY_PATH"
> ```
>
> O script do repositório [transcrever](https://github.com/ramonniellymorais/transcrever) já faz isso sozinho. Se você usar ele, não precisa pensar nisso nunca mais.

---

## Passo 5 — Publicar e versionar

```bash
brew install gh supabase deno
npm install -g vercel @railway/cli
gh auth login
```

O `gh auth login` abre o navegador para conectar sua conta do GitHub. Escolha **HTTPS** quando ele perguntar o protocolo — é o caminho mais simples.

---

## Passo 6 — Conectar o Claude ao resto do trabalho

Esta é a parte que separa "assistente que responde" de "assistente que trabalha".

**MCP** é a ponte entre o Claude e as ferramentas onde o trabalho já mora. Com o Notion conectado, ele lê e escreve no seu calendário editorial. Com o Drive, ele abre a transcrição. Sem isso, você vira o cabo de rede entre as duas coisas, copiando e colando o dia inteiro.

```bash
claude mcp add <nome> -- <comando do servidor>
claude mcp list
```

Cada serviço tem seu comando próprio, e a documentação de cada um está linkada em [stack-marketing-com-ia](https://github.com/ramonniellymorais/stack-marketing-com-ia#7-conectar-o-claude-ao-resto-do-trabalho).

Comece por **um** — o Notion, se o seu planejamento vive lá. Conectar cinco de uma vez no primeiro dia é receita para não entender qual deles quebrou.

---

## Passo 7 — Onde guardar senha e chave

Toda ferramenta desta lista uma hora pede uma chave de acesso.

**A regra que não se quebra:** chave nunca entra dentro de arquivo que vai para o GitHub. Um repositório público com chave dentro é chave vazada, e robôs varrem o GitHub procurando exatamente isso.

O caminho seguro é uma pasta fechada só sua:

```bash
mkdir -p ~/.credentials
chmod 700 ~/.credentials
```

Cada serviço vira um arquivo ali dentro, com permissão fechada:

```bash
chmod 600 ~/.credentials/nome-do-servico.md
```

Nos scripts, a chave entra por variável de ambiente, nunca escrita no meio do código. O [transcrever](https://github.com/ramonniellymorais/transcrever) mostra o padrão funcionando.

---

## Conferir se ficou tudo de pé

```bash
claude --version
gh --version
ffmpeg -version | head -1
yt-dlp --version
whisperx --version
vercel --version
```

Cada linha tem que devolver um número. Se alguma disser "command not found", quase sempre é o Passo 1 ou o Passo 2 pela metade — feche o Terminal, abra de novo e teste outra vez antes de reinstalar qualquer coisa.

---

## Depois daqui

- **[stack-marketing-com-ia](https://github.com/ramonniellymorais/stack-marketing-com-ia)** — o que cada ferramenta resolve no dia a dia de conteúdo
- **[transcrever](https://github.com/ramonniellymorais/transcrever)** — o primeiro uso prático que devolve tempo de verdade
- **[checar-copy](https://github.com/ramonniellymorais/checar-copy)** — verificador que reprova copy genérica

---

Feito por **[Ramonnielly Morais](https://ramonniellymorais.com.br)**, criadora do Método ELO Criativo.

Licença [CC BY 4.0](LICENSE) — use, adapte, cite a fonte.
