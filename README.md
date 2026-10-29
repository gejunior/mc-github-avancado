# mc-github-avancado

# Git Avançado: Merge, Conflitos e Workflow

---

## 1. Revisão Rápida e Por Que Não Trabalhar Apenas na `main`?

No curso básico, aprendemos o fluxo linear: `git add` -> `git commit` -> `git push` diretamente na branch `main`.

Em projetos reais e trabalhos em equipe, commitar direto na `main` causa graves problemas:

- Código incompleto ou com erro vai para a versão principal e impede outros colegas de trabalharem.
- Se duas pessoas mexem no mesmo arquivo ao mesmo tempo, o histórico vira uma bagunça.
- Não há revisão de código antes de uma funcionalidade entrar no ar.

A solução para isso é o domínio de **Ramificações (Branches)**, **Estratégias de Integração** e **Workflows**.

---

## 2. Ramificações (Branches) na Prática

Uma **branch** nada mais é do que um ponteiro móvel leve para um commit. Quando você cria uma nova branch, você cria uma “linha do tempo paralela” onde pode testar ideias, criar funcionalidades ou corrigir bugs com total segurança, sem afetar a `main`.

- **`HEAD`:** É o ponteiro especial do Git que indica “onde você está trabalhando agora” (qual branch ou commit está ativo no seu diretório).

### Comandos essenciais de gerenciamento de branches

```bash
# Listar todas as branches locais (a atual terá um asterisco *)
git branch

# Listar todas as branches (locais e remotas)
git branch -a

# Criar uma nova branch e já mudar para ela (comando moderno recomendado)
git switch -c feature/sistema-login

# Alternar entre branches existentes
git switch main
git switch feature/sistema-login

# Deletar uma branch local que já foi integrada (merged)
git branch -d feature/sistema-login

# Deletar uma branch local à força (mesmo sem ter sido integrada)
git branch -D experimento-falho

# Deletar uma branch que está lá no GitHub (remota)
git push origin --delete feature/sistema-login
```

---

## 3. Fusão e Integração de Código: As Estratégias

Quando terminamos o trabalho em uma branch separada, precisamos integrar essas mudanças de volta à branch principal (`main`) ou atualizar nossa branch com novidades da equipe. Existem diferentes formas e comportamentos para isso:

### 3.1. `git merge`: Fast-Forward vs. Three-Way Merge

Para unir mudanças na `main`, primeiro mudamos para quem vai **receber** o código (`main`) e depois chamamos o `merge`:

```bash
git switch main
git merge feature/sistema-login
```

O Git pode agir de duas maneiras diferentes dependendo do histórico:

1. **Fast-Forward (Avanço Rápido):**
    - **Quando acontece:** Quando ninguém fez nenhum commit novo na `main` desde que você criou a sua branch `feature/sistema-login`.
    - **O que o Git faz:** Ele não precisa criar um commit novo de fusão; ele simplesmente “desliza” a etiqueta da `main` para frente até alcançar sua branch. O histórico continua uma linha reta.
    - **Dica Estratégica:** Se você *quiser* forçar o registro de que aquela branch existiu no histórico, use `git merge --no-ff feature/sistema-login`.
2. **Three-Way Merge (Merge Commit / Fusão de 3 Pontas):**
    - **Quando acontece:** Quando a `main` recebeu commits novos enquanto você trabalhava em paralelo na sua branch (as histórias divergiram).
    - **O que o Git faz:** O Git compara 3 snapshots (o ancestral comum entre as duas branches + a ponta da `main` + a ponta da sua `feature`) e cria automaticamente um novo **Commit de Merge** unindo os dois caminhos.

### 3.2. `git merge` vs. `git rebase` (Diferenças Estratégicas)

Uma das dúvidas mais comuns em entrevistas e no mercado é a diferença entre **Merge** e **Rebase**:

- **`git merge`:** Preserva o histórico exatamente como ele aconteceu cronologicamente (com bifurcações e commits de merge). É **não-destrutivo** (não altera commits existentes).
- **`git rebase`:** Pega todos os commits da sua branch e os “reescreve” (reaplica um por um) no topo da ponta mais recente da `main`. O resultado é um **histórico perfeitamente linear e limpo**, sem bifurcações ou commits de merge extras.

```bash
# Estando na sua branch de feature, atualize a base dela com a main:
git switch feature/relatorio
git rebase main
```

> [!CAUTION]
**A Regra de Ouro do Rebase:NUNCA faça `git rebase` em branches públicas/compartilhadas** (como a `main` ou uma branch onde vários colegas estão trabalhando juntos). Como o rebase reescreve os commits (muda os hashes), ele quebra o histórico de quem já baixou aquela branch. Use rebase apenas para limpar e atualizar **a sua própria branch local** antes de integrá-la!
> 

### 3.3. `git fetch` vs. `git pull`

Quando trabalhamos em equipe, o repositório no GitHub muda o tempo todo.

- **`git fetch`:** Vai até o GitHub, baixa todas as novidades e novas branches da equipe para o seu computador, mas **não altera seus arquivos locais**. É a forma segura de inspecionar o que mudou (`git log origin/main`) antes de misturar com seu código.
- **`git pull`:** É a junção de dois comandos seguidos: `git fetch` + `git merge` (ou `git pull --rebase` para fazer `fetch` + `rebase`). Ele baixa e já tenta integrar imediatamente na sua branch atual.

---

## 4. Conflitos de Merge: Identificação, Análise e Resolução Segura

### Por que um conflito acontece?

O Git é muito inteligente: se você editou a linha 10 de um arquivo e seu colega editou a linha 80 do mesmo arquivo, o Git junta os dois sozinho sem perguntar nada.
Porém, se **duas branches alteraram exatamente a mesma linha do mesmo arquivo** de maneiras diferentes (ou se um deletou um arquivo que o outro estava editando), o Git para o processo e diz: *“Eu não sei qual dos dois códigos é o correto. Decida você, humano!”*

### Como identificar e analisar um conflito

Ao rodar `git merge outra-branch` (ou `git pull`) e ocorrer um conflito, o terminal avisará `CONFLICT (content): Merge conflict in arquivo.js`.

1. **Mantenha a calma!** Nada foi perdido.
2. Rode `git status`: ele mostrará claramente quais arquivos estão na seção `Unmerged paths (both modified)`.
3. Abra o arquivo conflitado no editor de código (como o VS Code). Você verá as **marcações de conflito** inseridas pelo Git:

```
<<<<<<< HEAD
const taxaDesconto = 0.15; // Código que estava na SUA branch atual (ex: main)
=======
const taxaDesconto = 0.20; // Código que está vindo da OUTRA branch (ex: feature/desconto)
>>>>>>> feature/desconto
```

### Entendendo os marcadores:

- Tudo entre `<<<<<<< HEAD` e `=======` é o que **já estava na sua branch atual**.
- Tudo entre `=======` e `>>>>>>> nome-da-branch` é o que **veio da branch que você está tentando juntar**.

### Passo a passo para resolver de forma segura:

1. **Analise e converse com a equipe:** Qual versão deve ficar? A sua (`0.15`), a do colega (`0.20`), ou uma combinação das duas?
2. **Limpe o arquivo:** Apague as linhas de marcação (`<<<<<<< HEAD`, `=======`, `>>>>>>> ...`) e deixe no arquivo **apenas o código final desejado**:
    
    ```jsx
    const taxaDesconto = 0.20;
    ```
    
3. **Avise o Git que o conflito foi resolvido** adicionando o arquivo ao Stage:
    
    ```bash
    git add arquivo.js
    ```
    
4. **Finalize a fusão** criando o commit de merge:
    
    ```bash
    git commit -m "Resolve conflito na taxa de desconto mantendo valor atualizado"
    ```
    

> [!TIP]
**Botão de Emergência (Abortar):**
Se no meio da resolução do conflito você perceber que fez besteira e quiser cancelar a tentativa de merge para voltar exatamente ao estado anterior, basta rodar:
> 
> 
> ```bash
> git merge --abort
> ```
> 

---

## 5. Ferramentas de Produtividade e Segurança em Equipe

### Guardando trabalho inacabado na gaveta (`git stash`)

Imagine que você está no meio de uma alteração que ainda não funciona (não quer fazer commit ainda), mas precisa mudar urgente para a `main` para ajudar um colega.

```bash
# Guarda todas as alterações atuais numa "gaveta temporária" e limpa o diretório
git stash

# Muda de branch, resolve o que precisa, volta para sua branch e recupera da gaveta:
git stash pop
```

### Desfazendo commits em branches compartilhadas (`git revert` vs `git reset`)

Se um commit com erro **já foi enviado para o GitHub (`git push`)** e outros colegas já baixaram, você **não deve** apagar esse commit da história com `git reset`, pois causará conflitos graves na equipe.

A maneira segura na indústria é usar o **`git revert`**:

```bash
# Cria um NOVO commit que aplica exatamente o inverso do commit problemático
git revert <hash-do-commit>
```

Assim, o erro é desfeito e o histórico continua íntegro para todos!

---

## 6. Fluxos de Trabalho na Indústria (Git Workflows)

Ter comandos avançados não basta; uma equipe precisa combinar **regras de organização** (Workflow). Os dois modelos mais famosos do mercado são:

### 6.1. GitHub Flow (Ágil, Moderno e Contínuo)

É o fluxo mais recomendado atualmente para a maioria das equipes web, projetos acadêmicos e startups.

- Existe apenas uma branch principal longa: a **`main`** (tudo que está na `main` deve estar sempre funcionando e pronto para produção).
- Para qualquer tarefa nova, cria-se uma branch descritiva a partir da `main`: ex. `feature/carrinho-compras` ou `fix/erro-calculo-frete`.
- Faz-se os commits na branch e envia para o GitHub (`git push -u origin feature/carrinho-compras`).
- Abre-se um **Pull Request (PR)** no site do GitHub.
- Outro membro da equipe faz o **Code Review** (revisa o código, comenta, testa).
- Sendo aprovado, faz-se o Merge do PR para a `main` e deleta-se a branch da tarefa.

### 6.2. Git Flow (Clássico, Estruturado por Versões)

Muito utilizado em softwares que possuem lançamentos planejados por versão (ex: `v1.0`, `v1.1`, aplicativos mobile ou sistemas bancários tradicionais).
Utiliza múltiplas branches com papéis rígidos:

- **`main`:** Contém apenas o código oficial de produção (versões lançadas).
- **`develop`:** Branch de integração onde os desenvolvedores juntam as novidades do dia a dia.
- **`feature/*`:** Criadas a partir da `develop` para novas funcionalidades e voltadas para a `develop`.
- **`release/*`:** Criada quando a `develop` está pronta para virar uma nova versão (apenas ajustes finos e testes antes de ir para a `main`).
- **`hotfix/*`:** Criada diretamente a partir da `main` para corrigir um bug crítico em produção com urgência (depois integrada tanto na `main` quanto na `develop`).

### Padrão de Nomenclatura de Branches e Commits (Conventional Commits)

No mercado, costuma-se padronizar nomes de branches e mensagens de commit:

- **Branches:** `feature/nome-da-funcionalidade`, `fix/nome-do-bug`, `docs/atualiza-readme`, `refactor/melhora-login`.
- **Commits Semânticos:**
    - `feat: adiciona botao de exportar PDF`
    - `fix: corrige divisao por zero no relatorio`
    - `docs: adiciona instrucoes de instalacao no README`
    - `refactor: simplifica logica de validacao de CPF`

---

## 7. Gestão de Projetos no GitHub: Projects, Issues e Fechamento Automático

Em equipes profissionais e projetos acadêmicos (como TCCs, Projetos Integradores ou Fábricas de Software), não saímos programando sem antes planejar e distribuir as tarefas. O próprio GitHub oferece ferramentas nativas para isso, dispensando aplicativos externos como Trello ou Jira.

### 7.1. GitHub Issues (Rastreamento de Tarefas e Bugs)

Uma **Issue** (Pendência/Tarefa) funciona como um “cartão de tarefa” ou “chamado” dentro do repositório. Ela serve para:

- Documentar uma nova funcionalidade que precisa ser criada (*Feature Request*).
- Relatar um erro encontrado no sistema (*Bug Report*).
- Discutir melhorias de arquitetura ou documentação.

Cada Issue criada recebe um **número único sequencial** no repositório (ex: `#1`, `#2`, `#15`) e pode conter:

- **Assignees (Responsáveis):** Quem da equipe vai programar aquela tarefa.
- **Labels (Etiquetas):** Categorias visuais como `bug`, `enhancement` (melhoria), `documentation`, `good first issue`.
- **Milestones (Marcos):** Agrupamento de issues para uma entrega específica (ex: `Entrega Parcial 1`, `Versão 1.0`).

### 7.2. GitHub Projects (Quadros Kanban e Automação)

O **GitHub Projects** é uma ferramenta de gerenciamento visual integrada diretamente às suas Issues e Pull Requests. Com ele, você organiza o trabalho no estilo **Kanban** (colunas como *Todo*, *In Progress*, *In Review* e *Done*) ou em formato de tabela/roadmap.

**Como criar e vincular um Project:**

1. No seu perfil do GitHub (ou dentro da aba **Projects** do repositório), clique em **New project**.
2. Escolha o template **Board** (Quadro Kanban) ou **Team backlog**.
3. Dentro do repositório, ao criar uma **Issue**, você seleciona o seu Projeto no menu lateral direito (**Projects**).
4. **Automação inteligente:** Quando uma Issue é criada, ela entra automaticamente na coluna **Todo** (A fazer). Quando alguém abre um Pull Request ou fecha a Issue, o cartão se move **sozinho** para a coluna **Done** (Concluído)!

### 7.3. Fechando Issues Automaticamente via Commits e Pull Requests

Um dos recursos mais produtivos do GitHub é vincular seu código diretamente às tarefas do quadro. Se você usar **palavras-chave especiais** seguidas do número da Issue (`#numero`), o GitHub **fecha a Issue automaticamente** e move o cartão do GitHub Projects para *Done* assim que o código chega na branch `main`!

#### Palavras-chave aceitas pelo GitHub (em inglês, não importa maiúscula/minúscula):

- `close`, `closes`, `closed`
- `fix`, `fixes`, `fixed`
- `resolve`, `resolves`, `resolved`

#### Exemplo 1: Fechando uma Issue diretamente pela mensagem do Commit

Imagine que você pegou a **Issue `#4`** (*“Corrigir cálculo de desconto no carrinho”*). Ao terminar o código, você faz o commit referenciando `#4`:

```bash
git commit -m "fix: corrige calculo de desconto no carrinho (fixes #4)"
```

*(Ou em um commit de múltiplas linhas no terminal:)*

```bash
git commit -m "feat: adiciona tela de recuperacao de senha" -m "Closes #12"
```

Quando esse commit for enviado (`git push`) ou integrado (`merge`) na branch principal (`main`), o GitHub:

1. Cria um link automático no histórico da Issue `#4` apontando exatamente para o seu commit (rastreabilidade total!).
2. Muda o status da Issue `#4` para **Closed** (Fechada).
3. Move o cartão `#4` lá no **GitHub Projects** automaticamente para a coluna **Done**!

#### Exemplo 2: Fechando múltiplas Issues de uma vez

```bash
git commit -m "feat: finaliza modulo de autenticacao e validacao. Closes #5, fixes #8"
```

#### Exemplo 3: Apenas referenciar a Issue sem fechá-la ainda

Se você ainda está na metade da tarefa e quer apenas que seu commit apareça registrado na linha do tempo da Issue `#4` (sem fechá-la), basta citar o número **sem** a palavra-chave antes:

```bash
git commit -m "wip: cria estrutura inicial do banco para a tarefa #4"
```

> [!TIP]
**Onde mais funciona?**
Além da mensagem de commit, você pode colocar `Closes #4` na **descrição do seu Pull Request (PR)** no site do GitHub. Assim que o seu colega aprovar o PR e clicar em *Merge pull request*, a Issue `#4` será fechada automaticamente!
> 

---

## 8. Laboratório Prático: Simulando Conflitos, Issues e Workflow Completo 🛠️

Neste exercício guiado, você vai simular um conflito real de equipe e praticar o fluxo de branches:

### Parte 1: Criando uma fusão limpa

1. Crie uma pasta `lab-git-avancado`, rode `git init` e crie um arquivo `sistema.txt` contendo:
    
    ```
    Sistema Acadêmico IF
    Versão: 1.0
    Módulo: Inicial
    ```
    
2. Faça o commit na `main`: `git add .` e `git commit -m "chore: estrutura inicial do sistema"`.
3. Crie e mude para uma branch chamada `feature/modulo-notas`:
    
    ```bash
    git switch -c feature/modulo-notas
    ```
    
4. Crie um novo arquivo `notas.txt`, adicione ao Git e faça um commit `feat: adiciona modulo de notas`.
5. Volte para a `main` (`git switch main`) e integre a branch (`git merge feature/modulo-notas`). Observe que ocorreu um **Fast-Forward**!

### Parte 2: Provocando e Resolvendo um Conflito Real

1. Estando na `main`, crie uma nova branch `feature/atualiza-versao`:
    
    ```bash
    git switch -c feature/atualiza-versao
    ```
    
2. Nessa branch, edite a **segunda linha** de `sistema.txt` para `Versão: 2.0-beta` e faça o commit:
    
    ```bash
    git commit -am "feat: atualiza versao para 2.0-beta"
    ```
    
3. Agora volte para a `main` (`git switch main`) e edite a **mesma segunda linha** de `sistema.txt` para `Versão: 1.1-estavel`. Faça o commit na `main`:
    
    ```bash
    git commit -am "fix: atualiza versao para 1.1-estavel"
    ```
    
4. Veja o gráfico de divergência no terminal:
    
    ```bash
    git log --oneline --graph --decorate --all
    ```
    
5. Agora tente fundir a branch na `main`:
    
    ```bash
    git merge feature/atualiza-versao
    ```
    
6. **O conflito aconteceu!** Abra o arquivo `sistema.txt`, analise as marcações `<<<<<<<`, `=======` e `>>>>>>>`, resolva deixando `Versão: 2.0-oficial`, remova os marcadores, faça `git add sistema.txt` e conclua com `git commit -m "merge: resolve conflito de versao definindo 2.0-oficial"`.
7. Rode novamente `git log --oneline --graph --decorate --all` e veja o **Three-Way Merge** formado no gráfico!

### Parte 3: Fluxo Completo com GitHub Projects, Issues e Pull Request

1. Crie um repositório no GitHub e envie sua `main`.
2. Na aba **Projects** do repositório, crie um novo quadro Kanban (**Board**) e vincule-o ao repositório.
3. Vá na aba **Issues** e crie duas tarefas:
    - **Issue `#1`:** *“Documentar diferença entre Merge e Rebase no README”* (adicione ao seu Project na coluna *Todo*).
    - **Issue `#2`:** *“Adicionar participantes da equipe no sistema.txt”* (adicione ao seu Project na coluna *Todo*).
4. No terminal, crie uma branch `docs/explica-merge-rebase`:
    
    ```bash
    git switch -c docs/explica-merge-rebase
    ```
    
5. Crie um arquivo `README.md` explicando a diferença entre `merge` e `rebase`, adicione ao Stage e faça o commit já programado para fechar a **Issue `#1`**:
    
    ```bash
    git add README.md
    git commit -m "docs: explica diferenca entre merge e rebase. Closes #1"
    ```
    
6. Envie a branch para o GitHub:
    
    ```bash
    git push -u origin docs/explica-merge-rebase
    ```
    
7. Acesse o repositório no navegador, abra um **Pull Request** da `docs/explica-merge-rebase` para a `main`, revise suas alterações e faça o Merge pela interface do GitHub.
8. **Verifique a mágica:** Volte na aba **Issues** e no seu quadro do **GitHub Projects** — veja que a Issue `#1` foi fechada sozinha com o link do seu commit e o cartão foi movido automaticamente para a coluna **Done**!
9. Agora repita o processo para resolver e fechar a **Issue `#2`** usando `fixes #2`!
