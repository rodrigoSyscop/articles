# macOS like a Boss - High Sierra

Como tenho o costume de fazer uma instalação limpa sempre que há o lançamento de um novo sistema operacional da maçã, acabei compilando este guia com as principais ações realizadas logo após a instalação.




## Homebrew

Para quem vem do Linux como eu, o Homebrew (ou somente brew) funciona como um gerenciador de pacotes para o MacOS. Além do brew eu também utilizo o Homebrew Cask, projeto que surgiu depois, mas que hoje funciona integrado ao brew e nos permite instalar aplicativos gráficos - GUI.

Instala o brew:

```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```

Ref.: [https://brew.sh](https://brew.sh)

Instala o brew cask:

```bash
 brew tap caskroom/cask
```

 Ref.: [https://caskroom.github.io](https://caskroom.github.io)

Com o `brew` e o `brew cask` instalado, fica fácil procurar e instalar novos aplicativos sem precisar recorrer ao browser:

```bash
# procura e instala o wget, uma ferramenta de linha de comando
brew search wget
brew install wget

# procura e instala o vlc, uma aplicação gráfica
brew cask search vlc
brew cask install vlc

```

### Instalando aplicativos

A seguir eu instalo diversas ferramentas, aplicativos e fontes que eu costumo utilizar. Veja nos comentários o que é cada um e absorva o que for útil para você:

```bash
# dev tools
brew install git git-lfs
```

```bash

# ativa o "repositório" de fontes
brew tap caskroom/fonts

# fontes para terminal e editores de textos
brew cask install font-fira-code font-source-code-pro 

# editor de texto
brew cask install visual-studio-code

# dev tools
brew cask install github-desktop sequel-pro

# social, groupware, communication
brew cask install slack skype telegram whatsapp

# downloads, players
brew cask install transmission vlc

# browsers
brew cask install firefox google-chrome

# extras
brew cask install kindle

```


## Bash Completions

Você deve ter reparado que após instalar alguns aplicativos pelo `brew` ele exibe a seguinte mensagem:

```bash
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d
```

Isso significa que o programa instalado veio acompanhado de algumas macros que instruem o bash sobre os possíveis argumentos, opções ou parâmetros do comando em questão quando você precionar a tecla `[TAB]` no terminal. O famoso recurso de autocompletar do shell do Linux.

Para tirar proveito disso você deve instalar o seguinte utilitário:

```bash
brew install bash-completion
```

Conforme sugerido pelo brew, você deve adicionar o seguinte trecho ao seu `.bash_profile` que talvez nem exista ainda:

```bash
vim ~/.bash_profile

    [ -f /usr/local/etc/bash_completion ] && . /usr/local/etc/bash_completion

```

Feche seu terminal e abra-o novamente ou simplesmente digite `source ~/.bash_profile`.
Agora teste digitando `git[TAB][TAB]`. 

Você deve se deparar com todos os subcomandos possíveis do `git`.




## Chaves SSH

Eu utilizo criptografia de chave pública para conectar nos servidores que eu administro e recomendo fortemente caso você ainda esteja utilizando senha para acessar seus servidores, a migrar para chaves ssh como eu.

Minhas chaves privadas são protegidas por senha, então caso eu me descuide e tenha essas chaves comprometidas, ainda teriam que adivinhar minha passphrase para efetivamente utilizar minhas chaves. Até lá eu já teria revogado minhas chaves antigas e substituído por novas.

Para não ter que ficar digitando senha toda vez que eu preciso utilizar minha chave privada, eu uso o _Keychain_ do MacOS. Assim só preciso digitar a passphrase da minha chave privada uma única vez a cada login no meu Mac.

Caso você já tenha sua chave ssh protegida por passphrase, você só precisa adicionar a seguinte opção ao seu arquivo de configuração do agente ssh:

```bash
vim ~/.ssh/config

    AddKeysToAgent yes
```

Aproveite para comentar a seguinte opção no seu `/etc/ssh/ssh_config`:

```bash
Host *
#       SendEnv LANG LC_*  # <- comentar essa linha no final do arquivo
```

Caso contrário você começará a se deparar com os seguintes aviso quando fizer um login remoto:

```bash
-bash: warning: setlocale: LC_CTYPE: cannot change locale (UTF-8): No such file or directory

```

Caso precise gerar um novo par de chaves:

```bash
$ ssh-keygen -t rsa -b 3072 -C "$(whoami)@$(hostname)-$(date +'%Y-%m-%d')"
```


## Personalizações gerais


### Trackpad

Em _System Preferences_ -> _Trackpad_:

Aba _Point & Click_:
- Tap to click
- Click: light
- Tracking speed: 5

Aba _More Gestures_:
- App Exposé


### Keyboard

Em _System Preferences_ -> _Keyboard_:

Aba _Keyboard_:
- Key Repeat: no máximo (Fast)
- Delay Until Repeat: no máximo (Short).


### Dock

- Size: por volta dos 20%.
- Magnification: desligado.

Marcar os dois checkboxes abaixo, ficando todos marcados:
- _Minimize windows into application icon_.
- _Automatically hide and show the Dock_

Eu uso bastante um recurso de arrastar janelas com três dedos, o qual foi bem escondido pela maçã nas últimas versões do MacOS. Se você também usa, ou utilizava e não sabe onde foi parar o recurso:

_System Preferences_ -> _Accessibility_ -> _Mouse & Trackpad_:
- Clique no botão _Trackpad Options..._
- Marque a opção _Enable dragging_
- No menu dropdown escolha: _three finger drag_

**Obs**: Com essa opção ativa, alguns jestos que antes utilizavam três dedos, passarão a requerer quatro.


### Safari

Em _Safari_ -> _Preferences_ -> _Advanced_:
- Show full website address
- Show Develop menu in menu bar

Em _View_ -> _Show status bar_ (ou `CMD` + `/`)


### Tema para o Terminal

Convenhamos, terminal com fundo branco!? Eu costumo utilizar o tema [pappermint](https://noahfrederick.com/log/lion-terminal-theme-peppermint), disponível para o Terminal nativo do MacOS e também para o iTerm.

Basta fazer o download e dar um duplo clique sobre o arquivo. Se seu Mac reclamar que este não é um software assinado e reconhecido, vá em _System Preferences_ -> _Security & Privacy_:

Clique no botão _Open Anyway_ na aba _General_.

Agora vamos corrigir a bizarrice do pappermint. Com o terminal aberto e o tema do pappermint aplicado, pressione `CMD` + `,` para abrir as preferências do terminal.

- Vá em _Profiles_, na aba _Text_ mude a fonte para _Source Code Pro_ e um tamanho que te agrade (eu uso 12).
- Na aba _Windows_ mude o _Window Size_ para 80 x 20.

Terminando, certifique-se que o tema Peppermint está selecionado e clique no botão _Default_ para fazer dele o tema padrão.

### VSCode

```json
{
    "editor.fontSize": 13,
    "editor.fontFamily": " 'Fira Code', Menlo, Monaco, 'Courier New', monospace",
    "editor.wordWrap": "on",
    "editor.lineHeight": 22,
    "editor.codeLens": true,
    "workbench.colorTheme": "Default Dark+",
    "workbench.iconTheme": "vs-seti",
    "explorer.openEditors.visible": 0,
    "terminal.integrated.fontFamily": "Source Code Pro"
}
```

