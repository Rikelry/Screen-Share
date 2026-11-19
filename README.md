# MLab

<p align="center">
  <img src="MLAB-logo.png" alt="MLab Logo" width="220" />
</p>

<p align="center">
  <strong>Transmissão de tela e áudio em rede local para salas de aula e laboratórios de informática</strong><br />
  Alta fidelidade em 1080p (30/60 fps), baixa latência, 100% offline e sem necessidade de internet.
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Go-1.26+-00ADD8?style=flat&logo=go&logoColor=white" alt="Go" />
  <img src="https://img.shields.io/badge/WebRTC-Pion-orange?style=flat&logo=webrtc&logoColor=white" alt="WebRTC" />
  <img src="https://img.shields.io/badge/Electron-Desktop-47848F?style=flat&logo=electron&logoColor=white" alt="Electron" />
  <img src="https://img.shields.io/badge/React-18-61DAFB?style=flat&logo=react&logoColor=black" alt="React" />
  <img src="https://img.shields.io/badge/Vite-Fast-646CFF?style=flat&logo=vite&logoColor=white" alt="Vite" />
  <img src="https://img.shields.io/badge/Methodology-SDD_%26_AI-7c3aed?style=flat" alt="SDD" />
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-green.svg" alt="License" /></a>
  <a href="https://github.com/MateusARJ/MLAB/releases/latest"><img src="https://img.shields.io/github/v/release/MateusARJ/MLAB?label=vers%C3%A3o&color=0f62fe" alt="Versão" /></a>
</p>

<p align="center">
  <a href="https://github.com/MateusARJ/MLAB/releases/latest"><strong>Baixar o MLab</strong></a> ·
  <a href="#instalação-para-uso-professor">Instalação</a> ·
  <a href="#primeira-aula-passo-a-passo">Primeira aula</a> ·
  <a href="#solução-de-problemas">Solução de problemas</a> ·
  <a href="#como-contribuir">Como contribuir</a>
</p>

---

## Sumário

- [O Problema](#o-problema)
- [O Objetivo](#o-objetivo)
- [Como Funciona](#como-funciona)
- [Funcionalidades](#funcionalidades)
- [Requisitos do Sistema](#requisitos-do-sistema)
- [Instalação para Uso (Professor)](#instalação-para-uso-professor)
- [Primeira Aula, Passo a Passo](#primeira-aula-passo-a-passo)
- [Atalhos do Professor](#atalhos-do-professor)
- [Ambiente de Desenvolvimento](#ambiente-de-desenvolvimento) — [Windows](#windows) · [macOS](#macos) · [Linux](#linux)
- [Gerando os Instaladores](#gerando-os-instaladores)
- [Versionamento e Releases](#versionamento-e-releases)
- [Rede e Firewall](#rede-e-firewall)
- [Segurança](#segurança)
- [Solução de Problemas](#solução-de-problemas)
- [Testes Automatizados](#testes-automatizados)
- [Desenvolvimento com IA e SDD](#desenvolvimento-com-ia-e-sdd)
- [Features e Decisões de Arquitetura](#features-e-decisões-de-arquitetura)
- [Como Contribuir](#como-contribuir)
- [Estrutura do Repositório](#estrutura-do-repositório)
- [Licença e Menções a Terceiros](#licença-e-menções-a-terceiros)

---

## O Problema

Nas aulas práticas e laboratoriais de cursos técnicos e superiores (como programação e informática), os professores frequentemente enfrentam graves limitações para compartilhar a sua tela com os estudantes:

1. **Limitações do Projetor Físico:** Os projetores multimídia instalados nas salas geralmente possuem baixa resolução nativa, lâmpadas com brilho desgastado e baixo contraste.
2. **Prejuízo à Visibilidade no Laboratório:** O projetor fica posicionado fixamente na parede frontal. Os alunos posicionados nas fileiras intermediárias e no fundo do laboratório têm enorme dificuldade em enxergar linhas finas de código-fonte no editor, saídas detalhadas de terminais e caracteres pequenos.
3. **Instabilidade e Queda da Internet:** A conexão de internet da instituição costuma oscilar ou cair durante os horários de pico, inviabilizando qualquer transmissão dependente de servidores externos.
4. **Bloqueio de Ferramentas em Nuvem pelo Firewall:** O firewall institucional bloqueia portas e protocolos de plataformas comerciais em nuvem (Google Meet, Zoom, Microsoft Teams e Discord), impedindo seu uso nas máquinas da instituição.

---

## O Objetivo

O **MLab** foi desenvolvido com o propósito de **acabar com a dependência do projetor físico** e proporcionar um salto substancial na qualidade das aulas.

O professor transmite a tela inteira ou uma janela específica diretamente para os monitores dos computadores dos alunos em **1080p Full HD** (a 30 ou 60 fps). A solução opera **100% em rede local (LAN)** — via cabo Ethernet ou Wi-Fi da sala —, funcionando de forma **offline**, **sem consumir dados da internet** e **sem exigir instalação de programas ou extensões por parte dos alunos**.

---

## Como Funciona

A arquitetura do MLab combina um aplicativo desktop para o professor, um servidor SFU em Go embarcado nesse aplicativo e uma página web leve para os alunos:

```text
       Computador do Professor (App Desktop Electron)
    ┌────────────────────────────────────────────────────────┐
    │  • Captura nativa de tela/janela em 1080p (30/60 fps)  │
    │  • Áudio do microfone e áudio interno do computador    │
    │  • Upload único (4 a 6 Mbps) via WebRTC                │
    │  • Painel da turma, gravação local e chat privado      │
    │  • Inicia o backend Go embarcado (porta :8080)         │
    └──────────────────────────┬─────────────────────────────┘
                               │ WebRTC (UDP 50000–50100) / WebSocket (:8080)
                               ▼
    ┌────────────────────────────────────────────────────────┐
    │  Backend SFU (Go / Pion WebRTC)                        │
    │  • Repasse em memória sem recompressão (SFU)           │
    │  • Controle de keyframes para novos alunos             │
    │  • Chat, fila de mãos levantadas e arquivos da aula    │
    │  • Serve a página web dos alunos                       │
    └───────────┬─────────────────────────────┬──────────────┘
                │ WebRTC Downstream           │ WebRTC Downstream
                ▼                             ▼
      Computadores dos Alunos       Dispositivos Móveis dos Alunos
    ┌────────────────────────┐    ┌──────────────────────────────┐
    │  Navegador Web (React) │    │  Navegador Mobile (Chrome/   │
    │  • Vídeo 1080p         │    │  Safari)                     │
    │  • Levantar a Mão      │    │  • Modo Tela Cheia e PiP     │
    │  • Chat público/privado│    │  • Sem instalar nada         │
    │  • Download de arquivos│    │  • 100% offline na rede local│
    └────────────────────────┘    └──────────────────────────────┘
```

- **Professor:** aplicativo desktop em Electron + React + TypeScript. O professor informa o nome (sem login), cria a sala, escolhe o monitor ou a janela, define a qualidade e transmite. O upload do professor é único: não cresce com o número de alunos.
- **Servidor:** backend em Go com Pion WebRTC, iniciado pelo próprio aplicativo do professor. Atua como SFU (Selective Forwarding Unit), repassando os pacotes de vídeo e áudio sem decodificar nem recomprimir. Também mantém o WebSocket da sala, o histórico da aula em memória e os arquivos compartilhados.
- **Alunos:** página web responsiva servida pelo backend na rede local. O aluno abre o link exibido no app do professor — no formato `http://<ip-do-professor>:8080/<sala>/<professor>/entrar` ou `http://mlab.local:8080/...` (`mlab-2.local`, `mlab-3.local`… quando há outros professores na mesma rede) — e se identifica com Nome e Matrícula.

---

## Funcionalidades

Resumo do que o MLab entrega hoje. Cada item vem de uma feature do Spec Kit; a lista completa, com o número de cada
uma, está em [Features e Decisões de Arquitetura](#features-e-decisões-de-arquitetura).

| Para o professor (app desktop) | Para o aluno (navegador) |
|---|---|
| Transmissão da tela inteira ou de uma janela em 1080p (30/60 fps) | Assiste sem instalar nada, no computador ou no celular |
| Barra flutuante de marcação: caneta, marcador, seta, retângulo, borracha, lupa e destaque | Entrada com Nome e Matrícula, sem cadastro |
| Fila de mãos levantadas, também visível na barra de marcação | Levantar e abaixar a mão |
| Chat público, chat privado e mensagens fixadas | Chat com histórico de quem entrou atrasado |
| Arquivos da aula por botão ou arrastando para a janela | Download dos arquivos pela própria página |
| Gravação local em WebM ou MP4 | Tela cheia e Picture-in-Picture |
| Áudio do microfone e do computador | Vídeo e áudio aparecem sozinhos, mesmo entrando antes do professor |

### Transmissão de Tela
- **Perfis Full HD:** **1080p @ 30 fps (4,0 Mbps)**, padrão para programação, slides e código-fonte (`contentHint = 'detail'` e `degradationPreference = 'maintain-resolution'`); e **1080p @ 60 fps (6,0 Mbps)** para animações, jogos, 3D e vídeos.
- **Troca de fonte sem quedas (`replaceTrack`):** o professor alterna entre "Tela Inteira" e "Janela Específica" durante a aula, e os alunos não precisam recarregar a página.
- **Sem tela preta:** quem entra recebe um keyframe logo de início, com cooldown no SFU e limite de pedidos de keyframe por aluno.
- **Aluno que chega antes do professor:** se a sala ainda está em espera, o vídeo e o áudio aparecem sozinhos quando a transmissão começa, sem apertar F5.
- **Upload único:** o professor envia uma só transmissão ao SFU, que repassa os pacotes a cada aluno sem recomprimir. O consumo de upload do professor não cresce com a turma. A meta é 40+ alunos por sala (limite padrão de 50).

### Cursor e Janela do MLab Fora da Transmissão
- A janela do próprio MLab usa proteção de conteúdo e não aparece no vídeo. O professor também pode copiar uma captura da janela transmitida para a área de transferência.
- **Janela específica:** o cursor só aparece quando a janela transmitida está em foco e sob o ponteiro. Sobre conteúdo em movimento ele continua visível, e o rastro de movimentos rápidos é limpo. Se a ocultação ameaçar o FPS, ela é suspensa por alguns instantes.
- A lupa só entra no vídeo quando está sobre a janela transmitida.

### Barra Flutuante de Marcação
- Overlay de anotação sobre qualquer janela, inclusive fora do MLab, com as ferramentas **cursor, caneta, marcador, seta, retângulo e borracha**, além de **lupa** com níveis de zoom e **destaque** (spotlight).
- Em tela cheia, as marcações e a lupa entram no vídeo, e a barra não aparece para os alunos.
- **Bolinha nos cantos:** recolhida, a barra vira uma bolinha que pode ficar em qualquer canto ou borda. Ao expandir, a barra cresce para dentro da tela e fica inteira, com o popover de cores. Ao minimizar, a bolinha volta ao ponto onde estava.
- **Mãos levantadas na barra:** quando um aluno levanta a mão, a barra (aberta ou recolhida) mostra uma borda de destaque e o número de mãos, sem texto. O professor percebe mesmo com outro programa em primeiro plano.
- Atalhos de teclado em [Atalhos do Professor](#atalhos-do-professor). No Linux (X11), a barra usa uma região de entrada própria para receber cliques sobre o overlay. Esse suporte ainda não foi validado num Ubuntu real (veja [Limitações conhecidas](#limitações-conhecidas)).

### Levantar a Mão com Fila de Chamada
- O aluno clica em **Levantar Mão** no player de vídeo ou na barra lateral de tela cheia.
- O professor vê a seção **Mãos Levantadas** na aba de Alunos, em ordem de chamada (`#1`, `#2`, `#3`).
- O professor pode responder em chat privado ou clicar em **Abaixar Mão**; o aluno também pode abaixar a própria mão.
- Com a barra lateral fechada, um aviso flutuante informa o professor. **Ver Turma** leva até a turma, **Dispensar aviso** fecha o aviso, e ele some sozinho quando a mão é abaixada. No celular do aluno, o aviso se ajusta à largura da tela.

### Entrada Identificada com Validação de Matrícula
- O aluno informa Nome Completo e Matrícula antes de entrar.
- A matrícula aceita siglas de curso de até 4 letras cercadas de dígitos (ex.: `2025116TADS0000`, `20241INFO1234`).
- **Matrícula única na transmissão** ([ADR 0010](spec-kit/docs/adrs/0010-sessao-unica-recusa-nova-entrada.md)): se a matrícula já está conectada, a nova entrada é recusada com o aviso "Esta matrícula já está conectada nesta transmissão." A sessão que já estava na sala não cai.
- **Sala encerrada não aceita entrada:** depois que o professor encerra a transmissão, o link antigo não recria a sala. O aluno que tenta entrar, ou estava conectando no momento do encerramento, recebe o aviso de transmissão encerrada.

### Chat com Máscara de Palavrões
- Chat público da turma e canal privado aluno-professor, separados.
- Filtro no backend que mascara palavras ofensivas (incluindo variações leetspeak e acentuação) com `****`.
- Histórico para quem entra atrasado (até 200 mensagens) e até 3 mensagens fixadas pelo professor.
- Limite de envio: o aluno é avisado quando manda mensagens rápido demais; o chat fica desabilitado até a conexão com a sala abrir.

### Compartilhamento de Arquivos na Rede Local
- O professor envia arquivos pelo botão ou **arrastando para a aba Arquivos**. Vários arquivos entram numa fila e sobem um por vez; se um falhar, os outros continuam. Se o arquivo for solto fora da zona de envio, o app não sai da sala.
- Limite de **50 MB** por arquivo. **Executáveis e instaladores** (`.exe`, `.msi`, `.bat` e outros) são recusados pela extensão e pela assinatura binária, mesmo se forem renomeados. Código-fonte (`.py`, `.js`, `.java`…) e arquivos compactados são aceitos.
- Os alunos baixam pela página, sem pen-drive nem internet. O download sempre vem como anexo, com o nome original (inclusive acentos) e o tipo definido pelo servidor.
- No celular, a lista de arquivos ocupa toda a altura da gaveta.

### Gravação Local em 1080p
- O professor salva a aula no próprio disco em WebM ou MP4, com a duração corrigida no arquivo para permitir avançar e voltar em qualquer player.

### Áudio do Professor
- Microfone com filtros contra microfonia e cancelamento de eco, e captura do áudio interno do computador.
- No aluno, se o navegador bloquear o som inicial, um banner permite ativá-lo com um clique.

### Operação 100% Offline e Link Fixo
- Funciona em salas sem conexão externa de internet. O estado da aula (salas, chat, arquivos) fica em memória e some quando o app fecha; não há banco de dados (ADR 0005).
- O backend descarta adaptadores virtuais (VMware, VirtualBox, WSL, Hyper-V, Docker, ZeroTier e VPNs) e usa o IPv4 físico da interface Ethernet ou Wi-Fi da sala.
- **Link `.local` por mDNS**, como alternativa ao IP: o primeiro professor da rede fica com `mlab.local`, o segundo com `mlab-2.local`, e assim por diante até `mlab-20.local`. O app sempre mostra o nome que o seu backend anuncia. O link também abre no celular (Android 12+ com Chrome e iOS com Safari). Se o mDNS não estiver disponível, o link por IP continua valendo.

### App do Professor
- Entrada só com o nome, sem login nem senha (ADR 0006).
- O rodapé das telas iniciais mostra a versão instalada, e a ação **Sobre** abre o nome, a versão, a descrição e os direitos autorais do MLab.
- A página dos alunos tem favicon próprio do MLab.

---

## Requisitos do Sistema

| Item | Requisito |
|---|---|
| Computador do professor | Windows 10/11 (64 bits), macOS 13+ (Ventura ou superior) ou Linux 64 bits (Ubuntu/Debian/Fedora) |
| Alunos | Qualquer computador ou smartphone com navegador moderno (Chrome, Edge, Firefox, Safari ou Opera) |
| Rede | Professor e alunos na mesma rede local (cabo ou Wi-Fi) |
| Desenvolvimento | Node.js 20+ (npm 10+), Go 1.26+ (`apps/backend/go.mod`), Git |

---

## Instalação para Uso (Professor)

Os instaladores ficam na página de [Releases](https://github.com/MateusARJ/MLAB/releases) do repositório (ou podem ser gerados conforme [Gerando os Instaladores](#gerando-os-instaladores)). O backend vem embutido no aplicativo: não é preciso instalar Go nem Node.js para usar.

### Windows
1. Baixe `MLab-Setup-<versão>.exe` e execute.
2. Se o SmartScreen avisar que o aplicativo não é reconhecido, clique em **Mais informações → Executar assim mesmo** (o instalador não é assinado).
3. Escolha a pasta de instalação; o atalho **MLab** é criado na Área de Trabalho e no Menu Iniciar.
4. Na primeira transmissão, permita o acesso do MLab às **redes privadas** quando o Firewall do Windows perguntar (veja [Rede e Firewall](#rede-e-firewall)).

### macOS
1. Baixe `MLab-Mac-<versão>.dmg`, abra e arraste o **MLab** para **Aplicativos**.
2. O aplicativo não é assinado pela Apple. Na primeira abertura, clique com o botão direito em **MLab → Abrir** e confirme. Se o macOS disser que o app "está danificado", remova a quarentena:
   ```bash
   xattr -cr /Applications/MLab.app
   ```
3. Em **Ajustes do Sistema → Privacidade e Segurança**, permita o MLab em **Gravação de Tela e Áudio do Sistema** e em **Microfone**. Reabra o app depois de conceder a permissão.
4. Quando o macOS perguntar sobre conexões de entrada ou acesso à rede local, permita.

### Linux
- **AppImage (qualquer distribuição):**
  ```bash
  chmod +x MLab-Linux-<versão>.AppImage
  ./MLab-Linux-<versão>.AppImage
  ```
  Em distribuições recentes pode ser necessário instalar o FUSE 2 (`sudo apt install libfuse2` ou `libfuse2t64` no Ubuntu 24.04+).
- **Debian/Ubuntu (.deb):**
  ```bash
  sudo apt install ./MLab-Linux-<versão>.deb
  ```
- **Outras distribuições (.tar.gz):** extraia o arquivo e execute o binário `mlab` (ou `MLab`) da pasta.
- **Wayland:** a captura de tela usa o portal do sistema (`xdg-desktop-portal` + PipeWire), que mostra a janela de escolha da tela. Se a captura falhar, verifique se esses pacotes estão instalados ou entre numa sessão Xorg.

### Código-fonte da versão
Cada release traz, além dos instaladores, o **`Source code (zip)`** e o **`Source code (tar.gz)`** do commit que gerou a versão.

---

## Primeira Aula, Passo a Passo

1. **Conecte o computador à rede da sala** (cabo ou Wi-Fi), a mesma rede que os alunos vão usar.
2. **Abra o MLab** e informe o seu nome. Não há cadastro nem senha.
3. **Crie a sala.** O app mostra o link dos alunos em dois formatos:
   - por nome: `http://mlab.local:8080/...` (ou `mlab-2.local`, `mlab-3.local`… se houver outro professor na rede);
   - por IP: `http://<ip-do-professor>:8080/...`, que funciona em qualquer rede.
4. **Passe o link aos alunos**, copiando, projetando ou escrevendo no quadro. Eles abrem no navegador e informam Nome e Matrícula. Quem entrar antes de você começar vê a sala em espera, e a transmissão aparece sozinha quando ela começa.
5. **Escolha o que transmitir** (tela inteira ou uma janela), a qualidade (30 ou 60 fps) e o áudio (microfone e/ou som do computador), e inicie a transmissão.
6. **Durante a aula:** use a barra flutuante para marcar a tela, acompanhe as mãos levantadas e o chat, compartilhe arquivos arrastando-os para a aba Arquivos e, se quiser, grave a aula no seu disco.
7. **Ao encerrar**, pare a transmissão e feche o app. O chat, a lista de alunos e os arquivos da aula ficam só na memória e somem quando o MLab fecha. Salve antes o que quiser guardar, como a gravação.

---

## Atalhos do Professor

Os atalhos funcionam com a barra de marcação ativa, mesmo com outro programa em primeiro plano. Todos têm uma variante
com `Alt+Shift+` (por exemplo `Alt+Shift+P`), para o caso de `Alt+<letra>` estar ocupado no sistema.

| Atalho | Ação |
|---|---|
| `Alt+H` | Recolher ou expandir a barra (bolinha) |
| `Alt+V` | Cursor (clicar nas janelas abaixo da marcação) |
| `Alt+P` | Caneta |
| `Alt+M` | Marcador |
| `Alt+A` | Seta |
| `Alt+R` | Retângulo |
| `Alt+E` | Borracha |
| `Alt+L` | Destaque (spotlight) |
| `Alt+Z` | Lupa: troca o nível de zoom |
| `Alt+Shift+U` / `Alt+Shift+Y` | Desfazer / refazer |
| `Alt+Shift+X` | Limpar as marcações |
| `Ctrl` duas vezes (até 600 ms) | Liga ou desliga a lupa (somente Windows) |

---

## Ambiente de Desenvolvimento

O repositório é um monorepo [Turborepo](https://turbo.build/) com `apps/desktop` (professor), `apps/web` (aluno), `apps/backend` (Go) e `packages/`.

### Windows

1. Instale as ferramentas (PowerShell):
   ```powershell
   winget install OpenJS.NodeJS.LTS
   winget install GoLang.Go
   winget install Git.Git
   ```
   Feche e reabra o terminal e confira: `node -v` (20+), `go version` (1.26+).
2. Clone e instale as dependências:
   ```powershell
   git clone https://github.com/MateusARJ/MLAB.git
   cd MLAB
   npm install
   ```
3. (Opcional) Se o antivírus bloquear a pasta temporária do Go, aponte-a para dentro do projeto:
   ```powershell
   $env:GOTMPDIR = "$PWD\.gotmp"
   ```
4. Inicie tudo:
   ```powershell
   npm run dev
   ```

### macOS

1. Instale o [Homebrew](https://brew.sh/) e as ferramentas:
   ```bash
   xcode-select --install
   brew install node go git
   ```
   Confira: `node -v` (20+), `go version` (1.26+).
2. Clone e instale as dependências:
   ```bash
   git clone https://github.com/MateusARJ/MLAB.git
   cd MLAB
   npm install
   ```
3. Inicie tudo:
   ```bash
   npm run dev
   ```
4. Na primeira captura, o macOS pede a permissão de **Gravação de Tela** para o Electron (ou para o Terminal/IDE de onde ele foi iniciado). Conceda em **Ajustes do Sistema → Privacidade e Segurança** e reinicie o `npm run dev`.

### Linux

1. Instale as ferramentas:
   - **Ubuntu/Debian:**
     ```bash
     sudo apt update
     sudo apt install -y git build-essential curl
     curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
     sudo apt install -y nodejs
     ```
   - **Fedora:**
     ```bash
     sudo dnf install -y git nodejs npm golang
     ```
   - **Go 1.26+:** se o pacote da distribuição for mais antigo, instale pelo site oficial:
     ```bash
     curl -LO https://go.dev/dl/go1.26.0.linux-amd64.tar.gz
     sudo rm -rf /usr/local/go && sudo tar -C /usr/local -xzf go1.26.0.linux-amd64.tar.gz
     echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc && source ~/.bashrc
     ```
   - Dependências de execução do Electron (Ubuntu/Debian): `sudo apt install -y libnss3 libatk-bridge2.0-0 libgtk-3-0 libgbm1 libasound2`
     (no Ubuntu 24.04+ o último pacote se chama `libasound2t64`).
2. Clone e instale as dependências:
   ```bash
   git clone https://github.com/MateusARJ/MLAB.git
   cd MLAB
   npm install
   ```
3. Inicie tudo:
   ```bash
   npm run dev
   ```

### O que o `npm run dev` faz

O script `scripts/dev.js` gera uma chave de host aleatória (`MLAB_HOST_KEY`), sobe o backend Go (porta `8080`), o web do aluno (Vite, porta `5173`) e o app desktop em modo de desenvolvimento. Não há login: o professor entra informando o nome (ADR 0006).

Para rodar as partes separadamente, use a mesma chave de host no backend e no desktop:

```bash
# Terminal 1 — backend (porta 8080)
cd apps/backend
MLAB_HOST_KEY=<chave> MLAB_DEV_MONOREPO=1 go run ./cmd/server

# Terminal 2 — web do aluno (porta 5173)
cd apps/web
npm run dev

# Terminal 3 — app do professor
cd apps/desktop
MLAB_HOST_KEY=<chave> npm run dev
```

No PowerShell, defina as variáveis com `$env:MLAB_HOST_KEY = "<chave>"` antes do comando. Sem `MLAB_HOST_KEY`, o backend recusa a sessão do professor. O segredo dos tokens (`JWT_SECRET`) é gerado aleatoriamente a cada execução quando não é informado.

Variáveis opcionais do backend: `PORT`, `WEB_PORT`, `PUBLIC_URL`, `BASE_URL`, `NGROK_URL`, `STUN_URL`, `TURN_URL`/`TURN_SECRET` (TURN opcional, ADR 0005).

---

## Gerando os Instaladores

O aplicativo do professor é empacotado com `electron-builder`. Cada comando compila o web do aluno, o backend Go para o sistema de destino e o app desktop. Os arquivos saem em `apps/desktop/dist/`.

> Os comandos `npm run make*` não alteram a versão: os instaladores saem com a versão atual do `package.json`. Para mudar de versão, siga [Versionamento](docs/versionamento.md).

| Sistema | Comando (na raiz) | Saída |
|---|---|---|
| Windows | `npm run make:win` | `MLab-Setup-<versão>.exe` (NSIS) e `win-unpacked/` |
| macOS (em um Mac) | `npm run make:mac` | `MLab-Mac-<versão>.dmg` e `.zip` |
| Linux (em Linux) | `npm run make:linux` | `MLab-Linux-<versão>.AppImage`, `.deb` e `.tar.gz` |
| Linux só AppImage / só .deb | `npm run make:linux:appimage` / `npm run make:linux:deb` | o pacote escolhido |
| Windows + Linux | `npm run make` | todos os acima, exceto macOS |

- **Linux a partir do Windows:** `make:linux` e `make` empacotam a parte Linux pelo **WSL** (`scripts/build-linux-wsl.js`). É preciso ter o WSL com Node.js instalado dentro dele.
- **macOS:** só pode ser gerado em um Mac. O app não é assinado nem notarizado (veja as instruções de instalação acima).

### Binário do backend isolado

```bash
cd apps/backend
go build -ldflags="-s -w" -o ./bin/mlab-server ./cmd/server
```

Nos instaladores o binário se chama `mlab-backend.exe` (Windows) ou `mlab-backend` (Linux/macOS), em `resources/bin/`.
O app inicia o backend e o encerra ao fechar; se o app morrer, o backend se encerra sozinho em poucos segundos. Até a
1.15.3 o binário se chamava `server.exe`/`server`.

---

## Versionamento e Releases

O MLab segue o **versionamento semântico** (`MAJOR.MINOR.PATCH`), com **uma versão única** para todo o monorepo, a
partir da 1.7.1 ([ADR 0007](spec-kit/docs/adrs/0007-versionamento-semantico.md)). O guia completo está em
[docs/versionamento.md](docs/versionamento.md).

| Nível | Quando usar | Exemplo |
|---|---|---|
| **MAJOR** | Quebra de compatibilidade: funcionalidade removida, contrato incompatível, requisito de sistema maior | `1.14.4 → 2.0.0` |
| **MINOR** | Funcionalidade nova compatível (US/RF novo, adição compatível em contrato) | `1.13.0 → 1.14.0` |
| **PATCH** | Correção, segurança, desempenho ou refatoração sem mudar capacidade nem contrato | `1.14.3 → 1.14.4` |
| sem incremento | Só documentação, testes, CI, estilo ou Spec Kit | — |

```bash
npm run version:check            # mostra a versão atual e confere se todos os pacotes estão alinhados
npm run bump -- patch            # ou minor | major: último commit do PR, "chore(release): versao X.Y.Z"
```

**Fluxo de release:**

1. Cada PR para a `main` que muda o app termina com o commit de versão. O CI valida o build de Windows, macOS e Linux em todo PR, e o merge é feito com merge commit, sem squash.
2. Para publicar, as novidades de cada versão são escritas em `docs/releases/vX.Y.Z.md`, num PR `docs` para a `main`.
3. O responsável cria a tag `vX.Y.Z` na `main`. O workflow publica a release com os instaladores, o código-fonte, o texto de `docs/releases/vX.Y.Z.md` e a lista automática de PRs.

---

## Rede e Firewall

| Porta | Protocolo | Uso |
|---|---|---|
| `8080` | TCP | API, WebSocket da sala e página web dos alunos |
| `50000–50100` | UDP | Mídia WebRTC (SFU) |
| `5353` | UDP | mDNS (`mlab.local`), opcional |

- **Windows:** permita o MLab em **redes privadas** no aviso do Firewall. Se a rede da sala estiver classificada como pública, mude para privada ou libere as portas acima.
- **macOS:** permita conexões de entrada quando o sistema perguntar (Firewall em **Ajustes do Sistema → Rede**).
- **Linux:** com `ufw`: `sudo ufw allow 8080/tcp && sudo ufw allow 50000:50100/udp && sudo ufw allow 5353/udp`. Com `firewalld`: `sudo firewall-cmd --add-port=8080/tcp --add-port=50000-50100/udp --add-port=5353/udp --permanent && sudo firewall-cmd --reload`.
- Redes com isolamento de clientes (comum em Wi-Fi de visitantes) impedem que os alunos alcancem o professor; use a rede interna da sala.
- **mDNS (`.local`):** usa a porta UDP 5353. Se ela estiver bloqueada ou ocupada por outro programa, o link `.local` não aparece no app, mas o link por IP continua funcionando.

---

## Segurança

Baseada na [ADR 0004](spec-kit/docs/adrs/0004-arquitetura-de-seguranca-e-testes-de-seguranca.md) e na feature
`002-security-hardening`:

- **Sem login nem senhas guardadas** ([ADR 0006](spec-kit/docs/adrs/0006-descontinuacao-do-login-e-sessao-local.md)): o professor entra pelo nome, e o app embarcado prova ao backend que é o dono da sessão com uma **chave de host** aleatória por execução.
- **Papéis distintos:** professor e aluno têm tokens com capacidades diferentes, e toda autorização é validada no servidor. O aluno não publica mídia nem modera a sala.
- **Tokens JWT** assinados com um segredo aleatório gerado a cada execução, sem dados pessoais além do necessário ([ADR 0009](spec-kit/docs/adrs/0009-remocao-da-claim-email-do-professor.md)).
- **WebSocket** com verificação de origem, limite de envio no chat e limite de pedidos de keyframe por aluno.
- **Arquivos:** limite de 50 MB, bloqueio de executáveis por extensão e assinatura, download sempre como anexo com `X-Content-Type-Options: nosniff`.
- **Electron endurecido:** renderer em sandbox, sem acesso direto ao Node.js (tudo via `contextBridge`) e com bloqueio de navegação para fora do app.

Encontrou uma vulnerabilidade? Abra uma issue sem detalhes de exploração e peça um canal privado ao mantenedor.

---

## Solução de Problemas

| Sintoma | O que fazer |
|---|---|
| O aluno não abre o link | Confirme que ele está na **mesma rede** do professor e que a rede não isola clientes. Libere a porta TCP `8080` no firewall ([Rede e Firewall](#rede-e-firewall)). |
| O link `mlab.local` não abre (mas o IP abre) | O mDNS pode estar bloqueado na rede ou no aparelho. Use o link por IP. No celular, prefira Chrome (Android 12+) ou Safari (iOS). |
| O aluno entra, mas o vídeo fica preto | Libere as portas UDP `50000–50100`. Peça ao aluno para recarregar uma vez; o SFU envia um keyframe novo a quem entra. |
| O aluno não ouve o áudio | O navegador bloqueou o som automático. O aluno clica em **Clique para Ativar Som** no aviso do topo. |
| "Esta matrícula já está conectada nesta transmissão." | A matrícula já está na sala, em outra aba ou dispositivo. Feche a sessão aberta antes de entrar de novo. Se for de outra pessoa, avise o professor. |
| "A transmissão foi encerrada pelo professor" | A sala foi encerrada e o link antigo não vale mais. Peça o link da nova transmissão ao professor. |
| Aviso de mensagens rápidas demais no chat | Limite de envio do chat. Espere alguns segundos. |
| Arquivo recusado no envio | Ele passa de 50 MB ou é executável/instalador. Compacte em `.zip` se for material didático. |
| macOS: tela preta ou captura vazia | Permita o MLab em **Gravação de Tela e Áudio do Sistema** e reabra o app. |
| macOS: "o app está danificado" | `xattr -cr /Applications/MLab.app` |
| Linux: captura falha no Wayland | Instale `xdg-desktop-portal` e PipeWire, ou entre numa sessão Xorg. |
| Windows: SmartScreen bloqueia o instalador | **Mais informações → Executar assim mesmo** (o instalador não é assinado). |
| O professor não entra depois de reabrir o app | Um backend de uma execução anterior pode ter ficado na porta `8080`. No Windows o MLab encerra sozinho o backend órfão ao abrir. Se a porta estiver com outro programa, o conflito fica no `mlab-main.log`: feche esse programa. |
| O MLab parece continuar rodando depois de fechado | Veja os logs na pasta de dados do app: Windows `%APPDATA%\@mlab\desktop\logs`, Linux `~/.config/@mlab/desktop/logs`, macOS `~/Library/Application Support/@mlab/desktop/logs`. O `mlab-main.log` registra a partida e cada etapa do encerramento; o `mlab-backend.log`, o motivo e a duração do encerramento do backend. Anexe os dois ao abrir uma issue. |

### Limitações conhecidas

- **Barra de marcação no Linux:** recolher, receber cliques e atalhos dependem de X11 e ainda não foram validados num Ubuntu real ([#30](https://github.com/MateusARJ/MLAB/issues/30), [#32](https://github.com/MateusARJ/MLAB/issues/32)).
- **Downloads em HTTP:** alguns navegadores avisam que o download é inseguro, porque a página da LAN é servida em `http` ([#47](https://github.com/MateusARJ/MLAB/issues/47)). Basta confirmar o download.
- **Lupa por duplo `Ctrl`:** só no Windows. Nos outros sistemas, use o botão ou `Alt+Z`.
- **Estado em memória:** nada da aula é guardado depois que o app fecha (por decisão, ADR 0005).
- **Instaladores sem assinatura:** Windows e macOS mostram avisos na primeira execução.

---

## Testes Automatizados

```bash
# Todos os testes do monorepo (Go, web e desktop)
npm test

# Backend Go
cd apps/backend && go test ./...

# Smoke do backend com servidor real (matriz de regressão)
npm run test:smoke -w @mlab/backend

# Web do aluno (Vitest) e app do professor (Vitest)
npm test -w @mlab/web
npm test -w @mlab/desktop

# E2E do aluno no navegador (Playwright)
npx playwright install chromium
npm run test:e2e -w @mlab/web

# Relatório consolidado da matriz de smoke (docs/metrics/smoke-2026-11.md)
npm run smoke

# Validação dos artefatos do Spec Kit
npm run spec:check

# Lint e checagem de tipos de todo o monorepo
npm run lint

# Versão alinhada em todos os pacotes e no package-lock
npm run version:check
```

| Camada | Ferramenta | O que cobre |
|---|---|---|
| Backend | `go test` | Unidade, segurança (auth, salas, sinalização, SFU), mDNS e teste de carga real do SFU com 40 e 50 alunos |
| Smoke de backend | `go test` com servidor real | Itens de servidor da matriz de regressão (`004-smoke-regression`) |
| Web do aluno | Vitest + Testing Library | Hooks, páginas e componentes |
| E2E do aluno | Playwright | Entrada, sessão única, chat, mãos e arquivos com professor simulado |
| Desktop | Vitest | Processo principal (overlay, atalhos, hit-test, guarda de navegação) e renderer |

A estratégia está na [ADR 0003](spec-kit/docs/adrs/0003-estrategia-de-testes-automatizados.md). As medições ficam em
[`docs/metrics/`](docs/metrics/). O CI (`.github/workflows/build-all.yml`) roda os testes e valida o build nos três
sistemas em todo PR para a `main`; os instaladores são publicados só na release.

---

## Desenvolvimento com IA e SDD

O MLab foi desenvolvido em **pair programming com inteligência artificial**, seguindo a metodologia **Spec-Driven Development (SDD)**, inspirada no [GitHub Spec Kit](https://github.com/github/spec-kit). Duas ferramentas de IA foram usadas ao longo do projeto:

- **[Google Antigravity](https://antigravity.google/):** usado nas fases iniciais e na maior parte da construção do produto — arquitetura do backend Go e do SFU Pion, captura e processamento de áudio no desktop, interface com o Carbon Design System e as primeiras suítes de testes.
- **[Claude Code](https://claude.com/claude-code) (Anthropic):** usado no realinhamento do Spec Kit ao código, no endurecimento de segurança (remoção do login, chave de host, proteção de arquivos e origens), no teste de carga real do SFU, na matriz de smoke de regressão (smoke de backend e E2E com Playwright) e nas correções que ela apontou.

### A metodologia SDD na prática

Nenhuma mudança de comportamento entra no código sem um artefato aprovado antes. O fluxo de cada feature, definido em [`AGENTS.md`](AGENTS.md) e na [constituição do projeto](spec-kit/.specify/memory/constitution.md), é:

```text
constitution → specify → clarify → plan → checklist → tasks → analyze → implement → converge
```

1. **Specify:** o que e por quê — user stories, requisitos (RF/RNF) e regras de negócio em `spec-kit/specs/<feature>/spec.md`.
2. **Clarify:** ambiguidades que mudariam comportamento, segurança ou arquitetura são resolvidas com o responsável e registradas.
3. **Plan:** como — arquitetura, riscos, contratos e um **Scope Lock** com os caminhos que a feature pode e não pode alterar.
4. **Checklist, tasks e analyze:** tarefas pequenas e testáveis, cada uma com paths permitidos e proibidos; a análise só libera a implementação quando termina em `PASS`.
5. **Implement:** uma tarefa por vez, com testes, em commits atômicos (`tipo(escopo): descrição em português`).
6. **Converge:** compara o código real com spec, plan e tasks e registra o que falta.

Decisões arquiteturais ficam em ADRs (`spec-kit/docs/adrs/`), e `npm run spec:check` valida os artefatos antes de qualquer merge que altere `apps/` ou `packages/`. As regras valem igualmente para as duas ferramentas de IA: o agente para e pergunta quando falta spec, quando uma tarefa sairia do Scope Lock ou quando teria de inventar comportamento.

### Divisão de papéis

- **Desenvolvedor:** identificação das dores reais das salas de aula e laboratórios, definição do escopo, decisões de produto nas etapas de clarify, aprovação de ADRs, validação da interface e testes práticos no ambiente real da instituição.
- **Agentes de IA (Antigravity e Claude Code):** redação e manutenção dos artefatos SDD, implementação das tarefas aprovadas, testes automatizados, medições e relatórios de convergência — sempre dentro do escopo aprovado pelo desenvolvedor.

---

## Features e Decisões de Arquitetura

Cada feature tem a sua pasta em [`spec-kit/specs/`](spec-kit/specs/), com `spec.md`, `clarify.md`, `plan.md`,
`tasks.md`, `analyze.md` e `converge.md`.

| # | Feature | O que entrega |
|---|---|---|
| 001 | [screen-share-core](spec-kit/specs/001-screen-share-core/) | Transmissão de tela professor → SFU → alunos |
| 002 | [security-hardening](spec-kit/specs/002-security-hardening/) | Fim do login, chave de host, proteção de arquivos e origens, teste de carga real |
| 003 | [classroom-interaction](spec-kit/specs/003-classroom-interaction/) | Mãos, chat, arquivos, matrícula, áudio, anotações, cursor e link `.local` |
| 004 | [smoke-regression](spec-kit/specs/004-smoke-regression/) | Matriz de smoke: backend com servidor real e E2E do aluno |
| 005 | [correcoes-smoke](spec-kit/specs/005-correcoes-smoke/) | Sessão única (4001), aviso de limite do chat e remoção de legado |
| 006 | [banner-mao-levantada](spec-kit/specs/006-banner-mao-levantada/) | Aviso de mão levantada responsivo no celular |
| 007 | [mascara-cursor-janela](spec-kit/specs/007-mascara-cursor-janela/) | Cursor e lupa ocultos fora da janela transmitida |
| 008 | [dev-entrada-unica-web](spec-kit/specs/008-dev-entrada-unica-web/) | Modo dev: a porta 8080 redireciona as páginas para o Vite |
| 009 | [marcacao-linux](spec-kit/specs/009-marcacao-linux/) | Barra de marcação no Linux (X11) |
| 010 | [mdns-celular](spec-kit/specs/010-mdns-celular/) | Link `mlab.local` abrindo no celular |
| 012 | [mdns-nome-unico](spec-kit/specs/012-mdns-nome-unico/) | Nome `.local` único por professor na mesma rede |
| 013 | [teste-ice-offline](spec-kit/specs/013-teste-ice-offline/) | Teste de entrada independente da rede |
| 014 | [gofmt-backend](spec-kit/specs/014-gofmt-backend/) | Backend formatado com `gofmt` |
| 015 | [remove-claim-email](spec-kit/specs/015-remove-claim-email/) | Remoção da claim `email` do JWT do professor |
| 016 | [modulo-go-mlab](spec-kit/specs/016-modulo-go-mlab/) | Nome do módulo Go do backend |
| 017 | [maos-na-barra-marcacao](spec-kit/specs/017-maos-na-barra-marcacao/) | Mãos levantadas na barra de marcação |
| 018 | [arquivos-arrastar](spec-kit/specs/018-arquivos-arrastar/) | Arrastar arquivos para a aba Arquivos |
| 019 | [validador-ids-tarefa](spec-kit/specs/019-validador-ids-tarefa/) | `spec:check` aceita IDs de tarefa com 3+ dígitos |
| 020 | [lint-tipos-desktop](spec-kit/specs/020-lint-tipos-desktop/) | Lint de tipos do desktop sem erros |
| 021 | [sobre-versao-desktop](spec-kit/specs/021-sobre-versao-desktop/) | Versão no rodapé e diálogo "Sobre" |
| 022 | [bloqueio-executaveis-upload](spec-kit/specs/022-bloqueio-executaveis-upload/) | Bloqueio de executáveis e headers de download |
| 023 | [tela-preta-entrada-antecipada](spec-kit/specs/023-tela-preta-entrada-antecipada/) | Aluno que entra antes vê a transmissão sem F5 |
| 024 | [lista-arquivos-celular](spec-kit/specs/024-lista-arquivos-celular/) | Lista de arquivos na altura da gaveta no celular |
| 025 | [favicon-web](spec-kit/specs/025-favicon-web/) | Favicon próprio do web dos alunos |
| 026 | [barra-flutuante-cantos](spec-kit/specs/026-barra-flutuante-cantos/) | Barra flutuante inteira nos cantos da tela |
| 027 | [matricula-unica-na-transmissao](spec-kit/specs/027-matricula-unica-na-transmissao/) | Matrícula conectada recusa a nova entrada (ADR 0010) |
| 029 | [join-sala-encerrada](spec-kit/specs/029-join-sala-encerrada/) | Join em sala encerrada responde `409` |
| 030 | [teste-mdns-sondagem](spec-kit/specs/030-teste-mdns-sondagem/) | Teste do mDNS mede só a sondagem |
| 031 | [aviso-mao-levantada](spec-kit/specs/031-aviso-mao-levantada/) | Aviso de mão levantada clicável e sincronizado com a fila |
| 032 | [fixar-janela-transmissao](spec-kit/specs/032-fixar-janela-transmissao/) | Fixar uma janela na transmissão de múltiplas janelas (especificada, ainda não implementada) |
| 033 | [encerramento-sala-pelo-hub](spec-kit/specs/033-encerramento-sala-pelo-hub/) | Encerramento da sala pelo hub reflete no store |
| 034 | [encerramento-backend-orfao](spec-kit/specs/034-encerramento-backend-orfao/) | App e backend embarcado encerram em qualquer fechamento |
| 035 | [status-sala-pelo-hub](spec-kit/specs/035-status-sala-pelo-hub/) | Status da sala acompanha a sessão do hub |
| 036 | [status-hub-sincronizado](spec-kit/specs/036-status-hub-sincronizado/) | Leitura sincronizada do status do hub |
| 037 | [registro-em-hub-encerrado](spec-kit/specs/037-registro-em-hub-encerrado/) | Conexão recusada em sala encerrada pelo hub |
| 038 | [envio-e-fechamento-do-cliente](spec-kit/specs/038-envio-e-fechamento-do-cliente/) | Envio e fechamento do canal do cliente sem corrida |
| 039 | [registro-antes-da-leitura](spec-kit/specs/039-registro-antes-da-leitura/) | Leitura do cliente só depois do registro no hub |
| 040 | [adulteracao-deterministica-do-jwt](spec-kit/specs/040-adulteracao-deterministica-do-jwt/) | Teste de adulteração do JWT determinístico |
| 041 | [termino-do-hub-encerrado](spec-kit/specs/041-termino-do-hub-encerrado/) | Goroutine do hub termina e libera a sala encerrada |

Os números 011 e 028 não foram usados.

**ADRs aprovadas** ([`spec-kit/docs/adrs/`](spec-kit/docs/adrs/)):

| ADR | Decisão |
|---|---|
| [0001](spec-kit/docs/adrs/0001-monorepo-turborepo.md) | Monorepo com Turborepo |
| [0002](spec-kit/docs/adrs/0002-desktop-app-framework.md) | Electron no app do professor |
| [0003](spec-kit/docs/adrs/0003-estrategia-de-testes-automatizados.md) | Estratégia de testes automatizados |
| [0004](spec-kit/docs/adrs/0004-arquitetura-de-seguranca-e-testes-de-seguranca.md) | Arquitetura e testes de segurança |
| [0005](spec-kit/docs/adrs/0005-operacao-lan-offline.md) | LAN offline como modo primário, TURN opcional, estado em memória |
| [0006](spec-kit/docs/adrs/0006-descontinuacao-do-login-e-sessao-local.md) | Fim do login: sessão por nome com chave de host |
| [0007](spec-kit/docs/adrs/0007-versionamento-semantico.md) | Versionamento semântico a partir da 1.7.1 |
| [0009](spec-kit/docs/adrs/0009-remocao-da-claim-email-do-professor.md) | Remoção da claim `email` do JWT do professor |
| [0010](spec-kit/docs/adrs/0010-sessao-unica-recusa-nova-entrada.md) | Matrícula já conectada recusa a nova entrada |
| [0011](spec-kit/docs/adrs/0011-fluxo-de-entrega-somente-main.md) | Fluxo de entrega somente com a `main` |

---

## Como Contribuir

As regras completas estão no [`AGENTS.md`](AGENTS.md) e valem para pessoas e agentes de IA.

1. **Comece pela spec.** Nenhuma mudança de comportamento entra sem um `tasks.md` aprovado (fluxo SDD acima). Se a ideia é nova, abra uma issue e crie a feature em `spec-kit/specs/`.
2. **Branch a partir da `main`** atualizada, com o nome `<tipo>/<descricao-em-kebab-case>` (ex.: `fix/lista-arquivos-celular`). Nunca commite direto na `main`.
3. **Commits atômicos**, um por entrega, no formato `tipo(escopo): descrição em português` (tipos em inglês: `feat`, `fix`, `docs`, `refactor`, `test`, `ci`, `chore`…), com `Ref: Txxx` da tarefa. O template fica em `.gitmessage` (`git config commit.template .gitmessage`).
4. **Versão:** se o PR muda `apps/` ou `packages/`, o último commit é o `npm run bump -- patch|minor|major` (veja [Versionamento e Releases](#versionamento-e-releases)).
5. **Antes do PR**, rode `npm run spec:check`, `npm run lint`, `npm test` e `npm run version:check`.
6. **Pull Request para a `main`**, preenchendo o [template](.github/pull_request_template.md): tarefa do Spec Kit, impacto de versão e desvios encontrados.
7. **Achou um bug fora do escopo?** Não corrija no mesmo PR. Abra uma issue com o template [Bug ou desvio](.github/ISSUE_TEMPLATE/bug-desvio.md), que segue o tratamento de desvios do `AGENTS.md` (§14).

---

## Estrutura do Repositório

```text
mlab/
├── apps/
│   ├── backend/               # Servidor Go: SFU Pion WebRTC, API REST e WebSocket
│   │   ├── cmd/server/        # Ponto de entrada, smoke e teste de carga
│   │   └── internal/
│   │       ├── auth/          # Sessão do professor (chave de host) e JWT
│   │       ├── chat/          # Máscara de palavrões
│   │       ├── room/          # Salas, links, arquivos (tipos bloqueados), origens, mDNS e IP da LAN
│   │       ├── signaling/     # Hub WebSocket: chat, mãos levantadas, sessão única, limite de keyframes
│   │       └── webrtc/        # SFU Pion, controle de keyframes e teste de carga
│   ├── desktop/               # App Electron + React (professor)
│   │   ├── src/main/          # Processo principal: backend embarcado, overlay de marcação, atalhos,
│   │   │                      # rastreio do cursor, guarda de navegação e preload (contextBridge)
│   │   ├── src/renderer/      # Interface React (design system 2.0): sala, barra flutuante, arquivos, gravação
│   │   └── src/renderer/media/ # Máscara do cursor (worker) e política de repasse dos quadros
│   └── web/                   # Web do aluno (React + Vite)
│       ├── e2e/               # Testes E2E (Playwright) com professor simulado
│       ├── src/hooks/         # Assinante WebRTC, reconexão e chat
│       └── src/pages/         # Entrada (Join) e sala ao vivo (Room)
├── packages/
│   ├── types/                 # DTOs e mensagens de sinalização compartilhados
│   └── ui/                    # Design system 2.0: tokens, fontes e componentes
├── docs/                      # Requisitos detalhados, métricas, teste de carga e design system
├── spec-kit/                  # Constituição, specs por feature e ADRs (SDD)
├── scripts/                   # dev, build do backend, empacotamento, smoke e spec:check
├── AGENTS.md                  # Regras para os agentes de IA (lido direto pelo Claude Code e outros agentes)
├── package.json               # Scripts da raiz
└── turbo.json                 # Pipelines do Turborepo
```

---

## Licença e Menções a Terceiros

O código-fonte do **MLab** é distribuído sob os termos da **[Licença MIT](LICENSE)**, © 2026 Mateus de Araujo.

### Atribuições e Licenças de Bibliotecas de Terceiros
O MLab é construído sobre tecnologias de código aberto. Agradecemos e reconhecemos os autores e mantenedores dos seguintes projetos:

| Projeto / Biblioteca | Licença | Detentores de Direitos / Autores | Finalidade no MLab |
| :--- | :--- | :--- | :--- |
| **[Pion WebRTC](https://github.com/pion/webrtc)** | MIT | Copyright (c) 2018 Pion | Motor WebRTC e SFU em Go puro |
| **[Gorilla WebSocket](https://github.com/gorilla/websocket)** | BSD-2-Clause | Copyright (c) 2013 The Gorilla WebSocket Authors | Servidor WebSocket e canal de sinalização |
| **[Golang-JWT](https://github.com/golang-jwt/jwt)** | MIT | Copyright (c) 2021 Dave Grijalva, Form3, Golang-JWT Authors | Assinatura e validação de tokens JWT |
| **[Go Cryptography](https://golang.org/x/crypto)** | BSD-3-Clause | Copyright (c) 2009 The Go Authors | Primitivas criptográficas usadas pelo Pion WebRTC (DTLS) |
| **[Electron](https://github.com/electron/electron)** | MIT | Copyright (c) OpenJS Foundation and Electron contributors | Framework desktop multiplataforma |
| **[React](https://github.com/facebook/react)** | MIT | Copyright (c) Meta Platforms, Inc. and affiliates | Biblioteca de interfaces |
| **[Lucide](https://lucide.dev)** | ISC | Copyright (c) Lucide Contributors | Ícones da interface |
| **[Atkinson Hyperlegible Next / Mono](https://www.brailleinstitute.org/freefont/)** | SIL OFL 1.1 | Copyright (c) Braille Institute of America | Fontes da interface (embutidas) |
| **[Vite](https://github.com/vitejs/vite)** | MIT | Copyright (c) 2019-present Evan You & Vite Contributors | Build e servidor de desenvolvimento do web |
| **[Turborepo](https://github.com/vercel/turbo)** | MIT | Copyright (c) 2021 Vercel, Inc. | Orquestrador do monorepo |
| **[TypeScript](https://github.com/microsoft/TypeScript)** | Apache-2.0 | Copyright (c) Microsoft Corporation | Tipagem estática em todo o monorepo |
| **[Vitest](https://github.com/vitest-dev/vitest)** | MIT | Copyright (c) 2021-present Anthony Fu & Vitest contributors | Testes unitários |
| **[Testing Library](https://github.com/testing-library)** | MIT | Copyright (c) 2017 Kent C. Dodds | Testes de componentes React |
| **[Playwright](https://github.com/microsoft/playwright)** | Apache-2.0 | Copyright (c) Microsoft Corporation | Testes E2E do web do aluno |

Para mais detalhes sobre os termos de uso de cada dependência, consulte os arquivos de licença originais de cada pacote.

# LICENCE

```
Copyright 2026 Rikelry

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```