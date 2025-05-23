# Lista de Comandos do Pacman (Gerenciador de Pacotes Arch Linux)

Abaixo estão os comandos mais usados do Pacman, que é o gerenciador de pacotes para Arch Linux e derivados.

---

## Comandos Básicos

- `pacman -Syu`  
  Atualiza o banco de dados de pacotes e atualiza o sistema (sincroniza pacotes).

- `pacman -Sy`  
  Atualiza apenas o banco de dados dos pacotes.

- `pacman -S <pacote>`  
  Instala um ou mais pacotes.

- `pacman -Ss <termo>`  
  Busca por um pacote no repositório.

- `pacman -Qi <pacote>`  
  Mostra informações detalhadas sobre um pacote instalado.

- `pacman -Qs <termo>`  
  Procura por pacotes instalados que correspondam ao termo.

- `pacman -R <pacote>`  
  Remove um pacote instalado.

- `pacman -Rs <pacote>`  
  Remove o pacote e suas dependências que não são usadas por outros pacotes.

- `pacman -Rc <pacote>`  
  Remove o pacote e as dependências, incluindo dependências que podem ser usadas por outros.

- `pacman -U <arquivo.pkg.tar.zst>`  
  Instala um pacote a partir de um arquivo local.

- `pacman -Sw <pacote>`  
  Baixa o pacote sem instalar.

---

## Limpeza e Cache

- `pacman -Sc`  
  Remove pacotes antigos do cache que não estão instalados.

- `pacman -Scc`  
  Remove todos os pacotes do cache (uso cuidadoso).

- `pacman -Qdt`  
  Lista dependências órfãs, que podem ser removidas.

- `pacman -Rns $(pacman -Qdtq)`  
  Remove as dependências órfãs.

---

## Outros Comandos Úteis

- `pacman -Q`  
  Lista todos os pacotes instalados.

- `pacman -Qm`  
  Lista pacotes instalados manualmente (fora do repositório oficial).

- `pacman -Ql <pacote>`  
  Lista os arquivos instalados por um pacote.

- `pacman -V`  
  Exibe a versão do pacman.

- `pacman --help`  
  Exibe a ajuda do pacman.

---

## Exemplos comuns

- Atualizar sistema completo:  
  ```bash
  sudo pacman -Syu
