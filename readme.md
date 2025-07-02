# Bonsai BIM × Claude Desktop – Guia de Instalação (Windows 10/11)

> **Objetivo**  Ligação do Blender + Bonsai BIM a modelos de linguagem (Claude, ChatGPT, etc.) via **Bonsai\_mcp** (versão personalizada do BlenderMCP) usando o **Model Context Protocol**.

---

## Índice

1. [Visão geral](#1-visão-geral)
2. [Pré‑requisitos](#2-pré-requisitos)
3. [Instalar ](#3-instalar-uv-opcional--recomendado)[*uv*](#3-instalar-uv-opcional--recomendado)[ (opcional ↔ recomendado)](#3-instalar-uv-opcional--recomendado)
4. [Clonar o repositório Bonsai\_mcp](#4-clonar-o-repositório-bonsai_mcp)
5. [Criar e ativar ambiente Python](#5-criar-e-ativar-ambiente-python)
6. [Instalar dependências](#6-instalar-dependências)
7. [Testar o servidor MCP](#7-testar-o-servidor-mcp)
8. [Instalar o add‑on no Blender](#8-instalar-o-add‑on-no-blender)
9. [Configurar o Claude Desktop](#9-configurar-o-claude-desktop)
10. [Fluxo de uso](#10-fluxo-de-uso)
11. [Problemas comuns](#11-problemas-comuns)
12. [Créditos](#12-créditos)



## 1. Visão geral

O **Bonsai\_mcp** expõe 11 ferramentas IFC (consultar entidades, propriedades, hierarquia espacial, etc.) para qualquer LLM compatível com MCP. O servidor (Python) conversa por socket com o **addon** no Blender e com o **Claude Desktop** (ou outro cliente MCP).



## 2. Pré‑requisitos

| Item                                  | Versão mínima | Download                                                                                       |
| ------------------------------------- | ------------- | ---------------------------------------------------------------------------------------------- |
| Blender                               | 4.0           | [https://www.blender.org/download/](https://www.blender.org/download/)                         |
| Bonsai BIM (BlenderBIM)               | 0.0.2405+     | [https://blenderbim.org/download.html](https://blenderbim.org/download.html)                   |
| Python                                | 3.12          | [https://www.python.org/downloads/](https://www.python.org/downloads/)                         |
| Claude Desktop                        | 0.9+          | [https://claude.ai/download](https://claude.ai/download)                                       |
| **uv** (opcional – instalador rápido) | 0.3+          | [https://github.com/astral-sh/uv/releases/tag/0.7.18](https://github.com/astral-sh/uv/releases/tag/0.7.18)|



## 3. Instalar *uv* (opcional ↔ recomendado)

```powershell
# PowerShell como ADMINISTRADOR
irm https://astral.sh/uv/install.ps1 | iex
# adicionar ao PATH (caso script use .local\bin)
setx PATH "%USERPROFILE%\.local\bin;%PATH%"
# fechar & abrir Powershell e testar
uv --version
```

Se preferir **não** usar *uv*, pule para o passo 5 e, na configuração do Claude, use `python.exe` em vez de `uv`.



## 4. Clonar o repositório Bonsai\_mcp

```powershell
mkdir C:\BonsaiIA
cd C:\BonsaiIA
git clone https://github.com/JotaDeRodriguez/Bonsai_mcp.git
cd Bonsai_mcp
```



## 5. Criar e ativar ambiente Python

```powershell
python -m venv venv
venv\Scripts\activate
```



## 6. Instalar dependências

O projeto ainda não possui `requirements.txt`/empacotamento; instale direto:

```powershell
pip install --upgrade pip
pip install mcp[cli] httpx
```



## 7. Testar o servidor MCP

```powershell
python tools.py --help
```

Saída esperada:

```
BlenderMCP server starting up
Could not connect to Blender on startup … (isso é normal nesta etapa)
```

Deixe essa janela aberta quando for usar o Blender **ou** execute via *uv* a partir do Claude (próximo passo).



## 8. Instalar o add‑on no Blender

1. Baixe `addon.py` deste repositório.
2. **Blender ▸ Edit ▸ Preferences ▸ Add‑ons ▸ Install…**
3. Selecione `addon.py` → marque a caixa **Interface: Blender MCP – IFC**.
4. Carregue um `.ifc` com o Bonsai BIM ativo.
5. No 3D View, pressione **N** → aba **Blender MCP – IFC** → **Connect to Claude**.



## 9. Configurar o Claude Desktop

Abra **Claude ▸ Settings ▸ Developer ▸ Edit Config**.

### 9.1 Com *uv* (recomendado)

```json
{
  "mcpServers": {
    "Bonsai-mcp": {
      "command": "uv",
      "args": [
        "--directory",
        "C:\\BonsaiIA\\Bonsai_mcp",
        "run",
        "tools.py"
      ]
    }
  }
}
```

### 9.2 Sem *uv* (Python direto)

```json
{
  "mcpServers": {
    "Bonsai-mcp": {
      "command": "C:\\BonsaiIA\\Bonsai_mcp\\venv\\Scripts\\python.exe",
      "args": ["tools.py"]
    }
  }
}
```

Reinicie o Claude Desktop. Vai em **Claude ▸ Settings ▸ Developer ▸ Edit Config** e confirme se não apareceu algum erro. Se tudo estiver correto, aparecerá o ícone 🛠 mostrando as ferramentas IFC.


## 10. Fluxo de uso

1. **Iniciar o servidor** (Claude o fará automaticamente; ou você mesmo `python tools.py`).
2. **No Blender**, clique em **Connect**.
   
   ![image](https://github.com/user-attachments/assets/d6fe8c0d-4413-494f-a4f2-7a16b4279846)

4. No Claude, pergunte coisas como:
   - `Liste todas as paredes deste modelo.`
   - `Mostre a estrutura espacial.`
   - `Exporte paredes para CSV.`
5. O LLM usa as ferramentas IFC expostas pelo Bonsai\_mcp.



## 11. Problemas comuns

| Sintoma                                     | Causa + Solução                                                                     |
| ------------------------------------------- | ----------------------------------------------------------------------------------- |
| `spawn uv ENOENT` no Claude                 | *uv* não está no PATH → adicione `C:\Users\<user>\.local\bin` ou use Python direto. |
| `Failed to connect to Blender…` no terminal | Add‑on não está ativo ou botão **Connect** não foi clicado.                         |
| Ferramentas não aparecem no Claude          | Config JSON incorreto → verifique barras `\\` e reinicie.                           |
| IFC grande fica lento                       | Divida o modelo por níveis ou use filtros (`list_ifc_entities` com `limit`).        |



## 12. Créditos

- **Bonsai\_mcp** – versão personalizada [BlenderMCP]([https://github.com/sidahuja/blendermcp](https://github.com/JotaDeRodriguez/Bonsai_mcp)) por [@JotaDeRodriguez](https://github.com/JotaDeRodriguez)
- [**Como instalar e configurar o Blender MCP com o tutorial Claude AI**](https://www.youtube.com/watch?v=PBSvqfx4gwQ&t=2s) por [@unitedtoptech6288](https://www.youtube.com/@unitedtoptech6288)
- **Sequential Thinking** tool – [Model Context Protocol servers](https://modelcontextprotocol.io/introduction)
- **Bonsai BIM** – IfcOpenShell dentro do Blender

---

> **Licença MIT** – Este projeto é aberto — sinta-se à vontade para fazer sua própria versão, propor melhorias e enviar pull requests (PRs)

