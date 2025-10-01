> Este documento descreve uma abordagem prática e organizada para equipes de desenvolvedores humanos que usam assistentes de código integrados ao VSCode (Copilot, ChatGPT, Cody, Tabnine etc.).
> O objetivo é manter colaboração estável, rastreável e segura, mesmo quando cada dev aceita sugestões diferentes das IAs.

---

## Sumário

1. [Premissa](#premissa)
2. [Principais desafios](#principais-desafios)
3. [Abordagem estável e robusta (práticas recomendadas)](#abordagem-estável-e-robusta-práticas-recomendadas)

   1. [Fonte única da verdade — Git](#1-fonte-única-da-verdade---git)
   2. [Padronização forte](#2-padronização-forte)
   3. [Fluxo de branches simples](#3-fluxo-de-branches-simples)
   4. [CI/CD como guardião](#4-cicd-como-guardião)
   5. [Revisão de código com olhar humano](#5-revisão-de-código-com-olhar-humano)
   6. [Prompts e configurações compartilhados](#7-prompts-e-configurações-compartilhados)
   7. [Práticas de equipe](#8-práticas-de-equipe)

4. [Modelos práticos: PR template, checklist e exemplos de commit](#modelos-práticos-pr-template-checklist-e-exemplos-de-commit)
5. [Comparativo rápido: GitHub Flow / Git Flow / Trunk-Based](#comparativo-rápido-github-flow--git-flow--trunk-based)
6. [Workflow prático — dia a dia (exemplo passo a passo)](#workflow-prático--dia-a-dia-exemplo-passo-a-passo)
7. [Anexos úteis (snippets de arquivos)](#anexos-úteis-snippets-de-arquivos)

---

# Premissa

- **Quem**: desenvolvedores humanos em equipe.
- **O que**: uso de assistentes de código no VSCode para acelerar escrita, refactor e testes.
- **Não**: agentes autônomos fazendo commits sem revisão humana.
- **Meta**: manter a qualidade, rastreabilidade e previsibilidade do repositório.

---

# Principais desafios

1. **Divergência de estilo**

   - Problema: sugestões diferentes por dev → código diferente em padrões e nomes.
   - Risco: inconsitência que aumenta custo de manutenção.

2. **Código não testado / incorreto**

   - Problema: IA sugere algo que compila, mas falha em casos reais ou viola invariantes.
   - Risco: bugs introduzidos por confiança cega na sugestão.

3. **Perda de rastreabilidade**

   - Problema: commits grandes "IA gerou tudo" sem explicação.
   - Risco: revisão difícil, rollback arriscado, história pouco útil.

4. **Dependência de contexto local**

   - Problema: IA usa contexto do workspace local do dev (versões, branches, arquivos abertos) e propõe soluções incompatíveis com o estado do repositório remoto.
   - Risco: integração inesperada e conflitos.

---

# Abordagem estável e robusta (práticas recomendadas)

## 1. Fonte única da verdade — **Git**

- **Regras de ouro**

  - `main` (ou `trunk`) é sempre a referência para produção.
  - **Branch protection**: exigir PRs, checks verdes e revisão antes do merge.
  - Habilitar **status checks obrigatórios** (CI, linter, testes).
  - Considerar **signed commits** ou regras de verificação se for necessário rastrear autoria.

- **Por quê?**
  Mantém disciplina: sugestões da IA só entram no histórico após passar pelas mesmas proteções que código humano.

---

## 2. Padronização forte

- **Editor + workspace**

  - Fornecer um `.editorconfig` e configurações em `.vscode/` (`settings.json`, `extensions.json`) no repositório.
  - Incluir snippets e templates compartilhados (helpers, testes).

- **Linters & formatadores**

  - Prettier/ESLint, `gofmt`/`golangci-lint`, `black`/`ruff`, etc., com configuração centralizada.
  - Executar localmente (pre-commit hooks) e no CI.

- **Hooks locais**

  - Usar `pre-commit` (ou Husky) para bloquear commits que não passam format/lint/test.

- **Por quê?**
  A IA pode gerar código diferente — mas padronização força a homogeneidade no momento do commit/merge.

---

## 3. Fluxo de branches simples

- **Escolha recomendada** para times que usam IA no VSCode:

  - **GitHub Flow** (bom equilíbrio) ou **Trunk-Based** se CI for maduro.

- **Boas práticas**

  - Branches curtas e focadas (`feature/`, `fix/`, `chore/`).
  - Nome padrão: `feature/<ticket>-<breve-descrição>`.
  - PR pequeno — facilita revisão humana e reduz conflitos.

- **Por quê?**
  Branches curtas reduzem a janela em que duas pessoas divergem e limitam impacto de sugestões diferentes da IA.

---

## 4. CI/CD como guardião

- **Checks mínimos** (obrigatórios antes do merge):

  - Build e testes unitários.
  - Lint e formatter.
  - Análise estática (CodeQL, Sonar, linters).
  - Testes de integração quando aplicável.

- **Checks adicionais (opcionais)**:

  - Scans de segurança e dependências (Dependabot/Snyk).
  - Coverage mínimo (ex.: não permitir queda > X%).
  - Deploy em ambiente de preview (review app) para PRs maiores.

- **Por quê?**
  Automatiza a validação, reduz confiança cega nas sugestões da IA.

---

## 5. Revisão de código com olhar humano

- **Tamanho do PR**

  - Ideal: pequenos (ex.: < 300 linhas mudadas). Caso maior, divida em PRs menores.

- **Tipo de revisão**

  - Validar intenção, edge cases e testes — não só estilo (o CI cobre estilo).
  - Marcar PRs com `ai-assisted` quando a IA foi usada intensivamente — sinaliza para o revisor.

- **Política**

  - Sempre que IA gerar lógica crítica (auth, pagamentos, parsing), exigir pelo menos 2 revisores humanos.
  - Usar **draft PRs** enquanto o autor valida localmente.

- **Por quê?**
  Garantir que a lógica faz sentido no contexto do sistema, não apenas que o código "pareça correto".

---

## 6. Prompts e configurações compartilhados

- **Arquivo no repo**: `AI-GUIDELINES.md` ou `.ai/prompts.md`.
- **Conteúdo sugerido**:

  - Prompts padrão para gerar código (incluindo constraints de estilo, test-first, idioma dos identificadores).
  - Blocos “do / don’t” (o que aceitar sem revisão e o que sempre revisar).
  - Exemplos de prompts para gerar testes unitários junto com implementação.

- **Configurações do VSCode**:

  - `.vscode/extensions.json` (recomendar plugins).
  - `.vscode/settings.json` compartilhado (formatOnSave, editor.tabSize).

- **Por quê?**
  Reduz variação entre devs ao instruir a IA com o mesmo contexto e regras.

---

## 7. Práticas de equipe

- **Pair programming com IA**

  - Um dev escreve, outro valida sugestões da IA em tempo real — ótima para conhecimento compartilhado.

- **Code review coletivo em áreas críticas**

  - Revisões em grupo para alterações críticas (segurança, infra).

- **Retros periódicas sobre uso da IA**

  - Ajustar prompts, decidir o que funciona, eliminar padrões ruins gerados pela IA.

- **Treinamento**

  - Sessões práticas (30–60 min) sobre “como usar o assistente de código eficientemente”.

- **Por quê?**
  Cultura e prática são tão importantes quanto ferramentas.

---

# Modelos práticos: PR template, checklist e exemplos de commit

## Exemplo de **PR template** (`.github/pull_request_template.md`)

```md
## Descrição

<!-- Descreva sucintamente o que foi feito e o porquê -->

## Tipo de mudança

- [ ] Bugfix
- [ ] Feature
- [ ] Refactor
- [ ] Documentação

## Issue relacionada

Closes #<numero>

## Como testar

1. Passos para reproduzir/validar
2. Comandos/testes a rodar

## Checklist antes do merge

- [ ] Código está formatado (pre-commit)
- [ ] Lint OK
- [ ] Testes unitários passando
- [ ] Testes de integração (se aplicável)
- [ ] PR marcado como `ai-assisted` (se a IA foi usada para gerar código importante)

## Notas
```

## Exemplo de **PR checklist** (curto)

- Tests: `npm test` / `go test ./...`
- Linter: `npm run lint` / `golangci-lint run`
- Coverage: não reduziu > 5%
- Build: `npm run build` / `go build ./...`

## Exemplo de **Conventional Commit**

- `feat(auth): adicionar refresh token`
- `fix(storage): corrigir race condition em cache`
- `chore(deps): atualizar axios para 1.5.0`

---

# Comparativo rápido: GitHub Flow / Git Flow / Trunk-Based

## GitHub Flow

- **Quando usar**: times que fazem deploy frequente e preferem simplicidade.
- **Processo**:

  1. Criar branch curta desde `main`.
  2. Fazer commits pequenos.
  3. Abrir PR para `main`.
  4. CI + revisão → merge → deploy.

- **Prós**: simples, rápido.
- **Contras**: arriscado sem CI/testes.

## Git Flow

- **Quando usar**: releases planejadas (prod com ciclos longos).
- **Processo**:

  - `develop` como linha principal de desenvolvimento.
  - `feature/*` a partir de `develop`.
  - `release/*` para preparar versão.
  - `hotfix/*` para correção urgente em `main`.

- **Prós**: organizado, bom para versionamento.
- **Contras**: overhead, muitos merges.

## Trunk-Based Development (TBD)

- **Quando usar**: times que já têm CI/CD muito maduro e disciplina forte.
- **Processo**:

  - Commits frequentes no `trunk` (ou branch muito curta).
  - Features incompletas atrás de **feature flags**.

- **Prós**: integrações rápidas, menos conflitos.
- **Contras**: exige testes automatizados sólidos e feature flags.

---

# Workflow prático — dia a dia (exemplo passo a passo)

> Exemplo para um desenvolvedor que usa VSCode + Copilot/ChatGPT.

1. **Issue / ticket**

   - Abrir/pegar uma issue com descrição clara (aceptance criteria).

2. **Criar branch**

   ```bash
   git checkout -b feature/1234-login
   ```

3. **Escrever testes primeiro (opcional, recomendado)**

   - Criar testes unitários/smoke antes de implementar a lógica.

4. **Usar a IA localmente**

   - Pedir ao assistente: _“Implemente a função X, escreva testes unitários correspondentes, siga as convenções do projeto (nomes em pt-br, gofmt, incluir docstring).”_
   - Aceitar sugestões parcialmente — refatorar e adaptar.

5. **Validar localmente**

   - Rodar lint, formatter, testes.
   - Corrigir problemas manuais. Não confiar só na sugestão.

6. **Commits pequenos e claros**

   - `git add . && git commit -m "feat(auth): adicionar endpoint de login (testes inclusos)"`
   - Evitar um único commit gigante.

7. **Push & Draft PR**

   ```bash
   git push -u origin feature/1234-login
   ```

   - Abrir PR em modo _draft_ se ainda está validando.
   - Preencher template, indicar se IA foi usada.

8. **Revisão**

   - Solicitar pelo menos 1 revisor; 2 para áreas críticas.
   - Responder comentários e iterar.

9. **CI green**

   - Esperar checks obrigatórios passarem.
   - Se falhar, consertar localmente e commitar.

10. **Merge**

    - Squash/merge conforme política do time.
    - Caso use Trunk-Based: permitir merges frequentes e usar feature flag.

11. **Retrospectiva**

    - Se a IA ajudou ou atrapalhou, anotar no `AI-GUIDELINES.md` e discutir na retro.

---

# Anexos úteis (snippets para copiar)

## `.github/workflows/ci.yml` — checklist mínimo (exemplo)

```yaml
name: CI
on: [push, pull_request]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup
        run: # instalar runtime, deps
      - name: Run linters
        run: npm run lint
      - name: Run tests
        run: npm test
```

## `AI-GUIDELINES.md` — exemplo de prompt padrão

```md
# Prompt padrão para geração de código (usar com Copilot/ChatGPT)

Contexto:

- Projeto em Go
- Nomes de identificadores em pt-br
- Usar gofmt e seguir goimports
- Escrever testes unitários com table-driven tests

Pedido:
"Implemente a função <NOME> que faz <OBJETIVO>. Gere também testes unitários cobrindo casos feliz e casos de erro. Retorne apenas o código Go sem explicações. Não altere arquivos fora da pasta <pasta>."
```

---

# Resumo final

- Trate a IA como **copiloto**, não como autor final.
- Mantenha **Git** como fonte única da verdade com proteções de branch e CI forte.
- Padronize editor, linters e prompts — centralize em arquivos do repositório.
- Enfatize **revisão humana**, commits pequenos.
- Escolha um fluxo de branches que combine com maturidade do time: GitHub Flow é bom padrão; Trunk-Based se CI/feature flags estiverem maduros; Git Flow só para ciclos longos.

---

Se quiser, eu já preparo para você (pronto para colar no repositório):

- `.github/pull_request_template.md` completo;
- `AI-GUIDELINES.md` com 6–8 prompts prontos;
- Exemplo de `.vscode/settings.json` e `extensions.json`;
- Um **workflow visual** (passo a passo) para o fluxo que você escolher (GitHub Flow / TBD / Git Flow).
