# BASH

| Comando | Explicação |
| ------- | ---------- |
| TAB | Completa os comandos |
| whoami | Mostra o usuário atual |
| pwd | Exibe a pasta atual |
| mkdir | Cria uma pasta |
| cd | Troca de pasta |
| ls | Lista o conteúdo dos diretórios |
| ss -nltp | Lista as portas TCP |
| grep -v | Trás todas as linhas que não batem |
| rm -rf | Remove todos os arquivos e diretórios |
| CTRL + z | Manda o processo para background |
| CTRL + l | Limpa a tela |
| ALT + # | Comenta a limpa e passa para baixo |
| CTRL + d | Sai da sessão |

# VIM

O vim é um editor de texto para terminal, uma evolução do antigo **vi**.
Todas as combinações de teclas deste documento devem seguir seu estilo de caixa, pois o vim é *case-sensitive*.

### Modos de Operação

- **Comando** - Modo de comando, utilizado para a tabela abaixo, basta apertar a tecla ESC
- **Inserção** - Modo de edição de texto, apertar **i** em modo de comando
- **Visual** - Modo para selecionar linhas, apertar **v** em modo de comando
- **Bloco Visual** - Modo para selecionar colunas, apertar **CTRL + v** em modo de comando

### Comandos

| Combinação | Explicação |
| ---------- | ---------- |
| :wq, :x, ZZ | Salva e sai |
| :35, :150 | Vai para a linha 35, vai para a linha 150 |
| yy, 4yy | Copia uma linha, copia 4 linhas |
| dd, cc, 4dd, 4cc | Apaga ou recorta uma linha, apaga ou recorta 4 linhas |
| pp, 4pp | Cola uma linha, cola 4 linhas |
| u | Desfazer |
| CTRL + r | Refazer |
