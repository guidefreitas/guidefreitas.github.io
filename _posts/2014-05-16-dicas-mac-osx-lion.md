---
layout: post
title:  "Dicas para Mac OSX Lion"
date:   2014-05-16 22:00:00 -0300
categories: osx
---

Olá ,

Resolvi escrever um post com algumas dicas que achei interessantes e úteis para Mac OS Lion. Para ativar as dicas abaixo basta digitar os comandos no Terminal.

## Arrumando fontes serrilhadas em monitores externos (não da Apple)

Por algum motivo o Mac OS não aplica o anti-alias corretamente aos monitores externos fazendo com que as fontes fiquem serrilhadas.

*Ativar*

    defaults -currentHost write -g AppleFontSmoothing -int 2

*Desativar*

    defaults -currentHost delete -g AppleFontSmoothing

## Ferramenta de Diagnóstico para Rede sem Fio

Esta ferramenta permite realizar diversos testes como performance, captura de pacotes etc:

Abra pelo Finder o caminho:

    /System/Library/CoreServices e clique em Wi-Fi Diagnostics.app

## Mudar o modo de input/output da entrada do jack 3.5mm

O que muita gente não sabe (inclusive eu antes de descobrir essa dica) é que a saída de fone de ouvido dos macbooks de 13" funcionam também como entrada para microfones. Para alterar esta configuração segure a tecla Option, clique no símbolo de auto-falante na menu bar  e altere o modo "Use Audio port for:" para input.

## Manter o Mac acordado sem utilizar programas externos

Abra uma janela de terminal e digite:

    pmset noidle

Enquanto a janela estiver aberta o mac não entrará em modo sleep.

## Stress test##

Abra uma janela de terminal para cada núcleo de processador que vc tenha na máquina, digite o comando abaixo e aperte Return:

    yes > /dev/null


Aguarde alguns instantes e sua máquina ficará a todo vapor.

## Mostrar o caminho completo das pastas no título do Finder
Abra o terminal e digite:

*Ativar*

    defaults write com.apple.finder _FXShowPosixPathInTitle -bool TRUE; killall Finder

*Desativar*

    defaults delete com.apple.finder _FXShowPosixPathInTitle;killall Finder

## Escondendo o Desktop

Permite ocultar todos o ítens do desktop. Abra uma tela de terminal e digite:


*Ativar*

    defaults write com.apple.finder CreateDesktop -bool FALSE;killall Finder

*Desativar*

    defaults delete com.apple.finder CreateDesktop;killall Finder


## Jogue Tetris no terminal

Abra uma tela to terminal e faça:

1) Digite emacs e aperte Return
2) Segure ESC e aperte X
3) Digite tetris e aperte Return

## Desativar animação de janelas

Abra uma tela do terminal e digite:

    defaults write NSGlobalDomain NSAutomaticWindowAnimationsEnabled -bool NO