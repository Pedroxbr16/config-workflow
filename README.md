  # 🚀 Deploy Automático via GitHub Actions + SSH + Docker

Este repositório está configurado para realizar **deploy automático em um servidor remoto com Docker**, toda vez que um `git push` for feito na branch `main`.

> ✅ Compatível com servidores Linux comuns e com **WSL** (Windows Subsystem for Linux)

---

## 📦 Requisitos

- Repositório hospedado no GitHub
- Docker e Docker Compose instalados no servidor (ou WSL)
- Acesso SSH ao servidor
- Git configurado no servidor e com acesso ao repositório

---

## 🔧 Etapas de Configuração

### ✅ 1. Gere uma chave SSH para o GitHub Actions

No seu terminal local:

```bash
ssh-keygen -t rsa -b 4096 -C "github-deploy"
```

Isso gerará dois arquivos:

- `~/.ssh/id_rsa` → chave **privada**
- `~/.ssh/id_rsa.pub` → chave **pública**

---

### ✅ 2. Instale a chave pública no seu servidor (ou WSL)

No terminal local, execute:

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub usuario@ip-do-servidor
```

> Ou cole manualmente o conteúdo da `id_rsa.pub` no arquivo `~/.ssh/authorized_keys` do servidor.

---

### ✅ 3. Adicione os segredos no GitHub

No repositório:

> Vá em **Settings > Secrets and variables > Actions > New repository secret** e crie os seguintes:

| Nome               | Valor                              |
|--------------------|------------------------------------|
| `SERVER_HOST`      | IP ou domínio do servidor          |
| `SERVER_USER`      | Usuário SSH (ex: `ubuntu`, `root`) |
| `SERVER_SSH_KEY`   | Conteúdo da chave privada `id_rsa` |

---

### ✅ 4. Crie o workflow de deploy

Crie o arquivo `.github/workflows/deploy.yml` com o seguinte conteúdo:

```yaml
name: Deploy automático

on:
  push:
    branches:
      - main  # Altere para sua branch principal, se diferente

jobs:
  deploy:
    name: Deploy via SSH
    runs-on: ubuntu-latest

    steps:
      - name: Deploy no servidor
        uses: appleboy/ssh-action@v1.0.0
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          script: |
            cd /caminho/do/seu/projeto
            git pull
            docker compose down
            docker compose up -d
```

---

## 🧪 Testando

1. Faça um commit e `push` na branch `main`:
   ```bash
   git add .
   git commit -m "Testando deploy automático"
   git push origin main
   ```

2. Acesse o GitHub → aba **Actions** → veja o job rodando.

---

## ⚠️ Observações sobre uso no WSL

- O WSL precisa estar **com Docker funcionando** (via Docker Desktop ou local).
- O **terminal do WSL deve estar ativo** ou configurar startup automático.
- Use caminhos absolutos corretos no `cd` do script (`/home/seu-usuario/projeto`).

---

## 💡 Dica Extra

Quer logs no servidor? Edite o script:

```yaml
script: |
  cd /caminho/projeto
  echo "$(date) - Início do deploy" >> deploy.log
  git pull >> deploy.log
  docker compose down >> deploy.log
  docker compose up -d >> deploy.log
  echo "$(date) - Deploy finalizado" >> deploy.log
```
