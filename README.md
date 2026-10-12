# onda. — seu diff tem uma história maior

> **Antes do merge, enxergue além do diff.**

O **Onda** é uma ferramenta local para responder a uma pergunta que todo programador já fez: **“Se eu mexer aqui, quem mais pode sentir?”** Ele analisa importações locais de projetos Python, JavaScript e TypeScript, caminha pelo grafo de dependências na direção inversa e mostra arquivos e testes potencialmente afetados — com o **caminho exato** que explica cada conexão.

Gera um **HTML interativo, portátil e offline**: desmarque arquivos alterados para simular uma mudança menor, clique em consumidores para ver o rastro causal, confira testes e pontos de atenção. Também exporta JSON para automações. Não executa o código do projeto nem envia arquivos para servidor algum.

## Comece em 30 segundos

Python 3.10+; **nenhuma dependência para usar a CLI**:

```bash
# Na pasta deste repositório, gere o relatório de exemplo:
python -m onda demo --changed commerce/price.ts security/session.ts --html index.html

# Analise as alterações da sua própria árvore Git (staged, unstaged e arquivos novos):
python -m onda /caminho/para/seu-repositorio --html impacto.html

# Ou compare com uma referência Git, como main:
python -m onda /caminho/para/seu-repositorio --base main --html impacto.html --json impacto.json

# Simule uma mudança sem depender de Git:
python -m onda /caminho/para/seu-repositorio --changed src/pricing.ts src/auth.ts --html impacto.html
```

Abra o HTML gerado diretamente no navegador. Ele já contém os estilos, o script e os dados da análise: não carrega bibliotecas, fontes ou scripts externos.

Instalação opcional do comando `onda`:

```bash
python -m pip install -e .
onda /caminho/para/seu-repositorio --html impacto.html
```

## Como funciona

1. Lê arquivos `.py`, `.js`, `.jsx`, `.ts`, `.tsx`, `.mjs`, `.cjs`, `.mts` e `.cts` até 512 KB; ignora diretórios gerados e dependências instaladas.
2. Extrai imports Python com `ast` e imports/reexports/`require()`/`import()` com heurística textual em JS/TS. Resolve **imports relativos locais** JS/TS e módulos locais Python (inclusive pacotes dentro de `src/`).
3. Inverte as arestas: se `checkout.ts` importa `price.ts`, uma mudança em `price.ts` pode atingir `checkout.ts`.
4. Usa busca em largura para mostrar o **menor caminho conhecido** de cada mudança até cada dependente e teste.
5. Sinaliza alcance amplo, ausência de teste conectado, áreas sensíveis, alterações em manifestos, imports não resolvidos e declarações possivelmente removidas no diff Git.
6. Sugere um comando de teste **apenas** quando detecta `pytest`, `vitest` ou `jest` a partir de arquivos visíveis. O comando é texto para revisão; **nunca é executado**.

Exemplo de rastro: `commerce/price.ts → commerce/checkout.ts → api/orders.ts → tests/orders.test.ts`.

## Rode os testes do Onda

```bash
python -m unittest discover -s tests -v
```

O projeto inclui um pequeno repositório de demonstração em `demo/`. Os testes Python da própria ferramenta usam só a biblioteca padrão. Os testes TypeScript do exemplo são ilustrativos; se quiser executá-los, instale as dependências em `demo/` e rode `npm test` lá.

## Escopo e honestidade

**Onda mostra possibilidade de impacto, não certeza de falha nem cobertura real.** O grafo não detecta com garantia imports dinâmicos complexos, aliases de TypeScript/bundlers, geração de código, APIs HTTP, eventos, filas, banco de dados, reflexão, referências em strings, efeitos de configuração ou chamadas em runtime. O parser JS/TS é heurístico e pode produzir falsos positivos em comentários ou falsos negativos em sintaxes não previstas. Imports sem caminho local resolvido são informados como lacunas quando relativos; pacotes externos ficam fora do grafo. Arquivos excluídos ou muito grandes não são analisados. Alterações em arquivos apagados e imports quebrados podem reduzir a visibilidade do grafo.

O relatório contém apenas **nomes de arquivos, cadeias de dependência, sinais e estatísticas** — não inclui o conteúdo dos arquivos lidos. Ainda assim, nomes de arquivos podem ser sensíveis: compartilhe o HTML/JSON apenas com quem você confia. Nenhum dado sai da sua máquina pelo Onda. O comando sugerido deve ser conferido antes de uso.

O Onda não substitui execução de testes, análise de cobertura, revisão de segurança ou revisão humana. Uma lista curta **não é** sinônimo de baixo risco.

## Estrutura

- `onda/analyzer.py` — indexação, resolução de imports, busca em largura e sinais.
- `onda/report.py` — relatório HTML autocontido com simulador no navegador.
- `onda/__main__.py` — CLI.
- `demo/` — repositório TypeScript fictício usado nos exemplos.
- `index.html` — relatório pronto para explorar.
- `tests/` — testes automatizados.

## Ideias para próximas versões

- Parsers específicos de linguagem para reduzir falsos positivos e expandir para Go/Rust.
- Integração opcional com cobertura real e CI, mantendo a análise explicável.
- Mapeamento de contratos de API e migrações entre serviços, com confirmação explícita do usuário.
- Comparação visual entre dois relatórios ao longo do tempo.

Contribuições são bem-vindas, especialmente para **precisão, acessibilidade e documentação honesta dos limites**. Licença MIT.
