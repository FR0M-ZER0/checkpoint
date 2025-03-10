# 🛠 Guia de Contribuição

Bem-vindo! Este documento define as diretrizes para contribuir com este projeto.

## 📁 Estrutura do Repositório

Este projeto é composto por três repositórios:  

- **Repositório raiz:** Contém configurações globais, documentação e scripts de automação.
- **Frontend:** Código do frontend da aplicação.  
- **Backend:** Código da API e controladores.

## 📌 Regras para Commits

Os commits devem seguir o padrão [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/):

**Estrutura:**  
<tipo>(<escopo>): <descrição>



**Exemplos:**  
feat(auth): adiciona suporte a autenticação JWT fix(api): corrige erro na validação de entrada do usuário chore(ci): adiciona configuração do GitHub Actions

markdown
Copiar
Editar

**Tipos válidos:**  
- `feat`: Nova funcionalidade  
- `fix`: Correção de bug  
- `chore`: Alterações que não afetam o código-fonte (CI/CD, docs, etc.)  
- `refactor`: Refatoração do código  
- `docs`: Atualizações de documentação  
- `test`: Adição ou atualização de testes  

## 🔀 Padrão de Pull Requests (PRs)

Cada PR deve seguir estas diretrizes:  

1. O título deve seguir o mesmo padrão dos commits:  
feat(frontend): adiciona modal de login

markdown
Copiar
Editar
2. Adicione uma descrição clara com o **propósito da mudança** e **como testar**.  
3. Vincule issues relacionadas, se aplicável (`Closes #123`).  
4. Aguarde pelo menos **1 aprovação** antes do merge.  

## ✅ Checklist antes de abrir um PR

- [ ] Código testado e funcional  
- [ ] Commits seguindo o padrão  
- [ ] Branch atualizada com a main pelo menos 1 dia antes do Pull Request.
- [ ] PR aprovado por pelo menos 1 membro da equipe  
