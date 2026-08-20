# Automatic Invoice Organizer

Aplicação desktop em Python que automatiza a organização de notas fiscais (XML/PDF) para um escritório de contabilidade: lê o XML da nota, extrai os dados automaticamente, localiza a pasta correta da empresa e renomeia e move os arquivos — sem intervenção manual.

## O problema

No dia a dia de um escritório contábil, cada nota fiscal emitida exigia o mesmo fluxo manual repetido dezenas de vezes por dia: baixar o arquivo, abrir para anotar o número, localizar a pasta da empresa na rede, criar a pasta do mês (quando não existia), renomear o arquivo e movê-lo. Um processo mecânico que ninguém tinha parado para questionar.

Esse projeto nasceu para eliminar esse fluxo manual.

## O que a aplicação faz

1. Lê o XML da nota fiscal mais recente na pasta de downloads
2. Extrai automaticamente número, prestador e tomador do serviço
3. Localiza a pasta da empresa correspondente (com busca incremental via SQLite)
4. Cria as pastas de ano e mês automaticamente, caso ainda não existam
5. Renomeia e move o XML e o PDF correspondentes para o destino correto, com nome padronizado

**Resultado estimado:** entre 50 minutos e 3 horas economizadas por dia em volume alto de emissão, eliminando renomeação e movimentação manual de arquivos.

## Arquitetura

O projeto segue uma separação em camadas inspirada em Clean Architecture:

```
domain/          # Regras de negócio puras — nomenclatura de arquivos, identificação de pastas fiscais
application/     # Casos de uso — orquestra domain + infrastructure
infrastructure/  # Detalhes técnicos — sistema de arquivos, banco de dados (SQLite), leitura de XML
presentation/    # Interface gráfica (Tkinter)
```

Essa separação permite trocar qualquer camada externa (ex.: trocar Tkinter por uma interface web, ou SQLite por outro banco) sem tocar nas regras de negócio em `domain/`.

## Stack

- **Python** (stdlib: `sqlite3`, `xml.etree.ElementTree`, `pathlib`, `tkinter`)
- **SQLite** para busca incremental de empresas cadastradas
- Parsing de XML com resolução dinâmica de namespace (compatível com diferentes leiautes de NFS-e emitidos por prefeituras distintas)

## Como rodar

```bash
git clone https://github.com/gabrieldemoraisfreitas/automatic-invoice-organizer.git
cd automatic-invoice-organizer
python main.py
```

Antes de rodar, ajuste os caminhos em `infrastructure/config.py` para o seu ambiente (veja Limitações abaixo) e crie um `empresas.txt` com uma empresa por linha, seguindo o modelo em `empresas.example.txt`.

## Limitações conhecidas

- Os caminhos de diretório (`config.py`) estão fixos para o ambiente local original — a versão de produção precisaria externalizar isso em variáveis de ambiente ou arquivo de configuração
- Assume que os dois arquivos mais recentes na pasta de downloads são o par XML/PDF da nota — funciona bem no fluxo real de uso, mas é uma premissa que pode falhar em outros contextos
- Cada prefeitura tem seu próprio leiaute de XML; o parser atual lida com o namespace dinamicamente, mas nem todos os formatos de NFS-e do Brasil foram testados

## Próximos passos

Evolução natural do projeto: um sistema que capture o faturamento da empresa automaticamente direto do portal de cada prefeitura, eliminando também o passo de download manual.
