# Publicação no GitHub — passo a passo para iniciante

## 1. Criar os dois repositórios

No GitHub, clique em **New repository** e crie:

- `bookhub-arquitetura-n2`
- `bookhub-web`

Marque os repositórios como **Private** se o professor não exigir público. Não marque “Add a README” se você vai enviar uma pasta que já possui README.

## 2. Instalar e configurar Git

No PowerShell, confirme:

```powershell
git --version
```

Se o comando não existir, instale o Git for Windows em `git-scm.com/download/win` e reabra o VS Code.

Configure o nome e e-mail uma única vez:

```powershell
git config --global user.name "Francieli Ribeiro"
git config --global user.email "SEU_EMAIL_DO_GITHUB"
```

## 3. Publicar este repositório de arquitetura

No PowerShell:

```powershell
cd C:\caminho\paraookhub-arquitetura-n2
git init
git add .
git commit -m "docs: adiciona arquitetura DDD, C4 e inspeção SOLID"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/bookhub-arquitetura-n2.git
git push -u origin main
```

Substitua `SEU_USUARIO` pelo seu usuário do GitHub. Se o GitHub pedir login pelo navegador, conclua a autenticação. Se pedir senha no terminal, use um Personal Access Token, não a senha normal da conta.

## 4. Publicar o projeto Web

Extraia o ZIP do backend e do frontend numa pasta de trabalho, ou use a pasta do seu projeto local. Não publique `.env` com senha. Antes do `git add`, crie/edite `.gitignore` com:

```gitignore
node_modules/
.env
uploads/
dist/
*.log
```

Depois:

```powershell
cd C:\caminho\paraookhub-web
git init
git add .
git commit -m "feat: adiciona BookHub web"
git branch -M main
git remote add origin https://github.com/SEU_USUARIO/bookhub-web.git
git push -u origin main
```

Uma alternativa é manter backend e frontend em dois repositórios, `bookhub-api` e `bookhub-frontend`. Para o trabalho de Arquitetura, o repositório `bookhub-arquitetura-n2` é o obrigatório da Questão 1.

## 5. Colocar os links no documento

Depois do push, abra cada repositório no GitHub e copie a URL da barra do navegador. No PDF/documento, escreva:

- Repositório da arquitetura DDD: `https://github.com/SEU_USUARIO/bookhub-arquitetura-n2`
- Repositório do projeto Web: `https://github.com/SEU_USUARIO/bookhub-web`

## 6. Segurança

Nunca faça commit de:

- `.env`;
- senha do MySQL;
- token JWT;
- arquivos pessoais;
- `node_modules`.

Se um `.env` já foi enviado, remova-o do histórico e troque imediatamente a senha do banco.
