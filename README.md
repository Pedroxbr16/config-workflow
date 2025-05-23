# 🚀 Deploy Local Automático via GitHub Actions + Self-Hosted Runner + Docker (WSL)

Este repositório está configurado para realizar **deploy automático** na sua máquina local (WSL ou Linux) toda vez que um `git push` for feito na branch `main`, usando um **runner self-hosted**.

> ✅ Ideal para quem quer testar CI/CD sem expor portas, sem VPS nem SSH externo.

---

## 📦 Requisitos

- Repositório no GitHub
- Windows Subsystem for Linux (WSL) ou qualquer máquina Linux com:
  - Docker e Docker Compose instalados  
  - Git configurado e com permissão de leitura/escrita  
- Acesso de administrador (para rodar o runner e serviços Docker)

---

## 🔧 Etapas de Configuração

### 1. Registrar seu Runner Self-Hosted no GitHub

1. No GitHub, abra **Settings > Actions > Runners** do seu repositório.
2. Clique em **“Add runner”** e escolha **Linux**.
3. Copie e execute, no seu WSL/terminal Linux, os comandos sugeridos. Exemplo:
   ```bash
   # dentro da pasta que você quer manter o runner
   mkdir actions-runner && cd actions-runner
   # faça download do pacote
   curl -o actions-runner-linux-x64-2.308.1.tar.gz         -L https://github.com/actions/runner/releases/download/v2.308.1/actions-runner-linux-x64-2.308.1.tar.gz
   tar xzf ./actions-runner-linux-x64-2.308.1.tar.gz

   # registre contra o GitHub (o token vem na página)
   ./config.sh --url https://github.com/SEU_USUARIO/SEU_REPOSITORIO                --token SEU_TOKEN_FORNECIDO_PELO_GITHUB
   ```

4. Inicie o runner:
   ```bash
   ./run.sh
   ```
   Você pode deixá-lo rodando em um tmux, screen ou configurar como serviço systemd.

---

### 2. Criar o Workflow de Deploy Local

No seu repositório, crie o arquivo `.github/workflows/deploy-local.yml` com o conteúdo:

```yaml
name: Deploy Local Self-Hosted

on:
  push:
    branches:
      - main

jobs:
  deploy:
    name: Deploy no Self-Hosted Runner
    runs-on: [self-hosted, linux]

    steps:
      - name: Checkout do código
        uses: actions/checkout@v3

      - name: Atualizar e reiniciar containers Docker
        run: |
          cd /caminho/absoluto/para/seu/projeto
          git pull origin main
          docker compose down
          docker compose up -d --build
```

> 🔑 **Importante**: use caminhos **absolutos** no `cd`, pois o runner self-hosted roda fora do contexto da ação.

---

## 🧪 Testando

1. **Garanta** que seu runner esteja online (veja na aba **Runners** do GitHub).
2. Faça um commit e `push` na branch `main`:
   ```bash
   git add .
   git commit -m "Teste deploy local"
   git push origin main
   ```
3. Acesse no GitHub **Actions** → abra o workflow **Deploy Local Self-Hosted** → e acompanhe a execução.

---

## ⚠️ Observações sobre uso no WSL

- Se você usar **Docker Desktop**, verifique se o WSL está conectado ao Docker e que o daemon está ativo.
- Mantenha o runner rodando em segundo plano (service, tmux, screen).
- No **Windows**, certifique-se de que o firewall permite comunicação interna entre processos.

---

## 💡 Dica Extra

Para registrar o runner como **serviço systemd** (Linux puro), crie `/etc/systemd/system/github-runner.service`:
```ini
[Unit]
Description=GitHub Actions Runner
After=docker.service

[Service]
WorkingDirectory=/home/SEU_USUARIO/actions-runner
ExecStart=/home/SEU_USUARIO/actions-runner/run.sh
User=SEU_USUARIO
Restart=always

[Install]
WantedBy=multi-user.target
```

Depois:
```bash
sudo systemctl daemon-reload
sudo systemctl enable github-runner
sudo systemctl start github-runner
```

Pronto! Agora todo push na `main` dispara automaticamente seu deploy local via Docker, sem precisar de VPS ou SSH externo. 🚀  
