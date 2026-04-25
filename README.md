# Setup Profissional de Python no Windows

> Material de apoio da aula — Pós-graduação em Engenharia de Dados · Xperiun

---

A proposta aqui não é só instalar Python e sair rodando código.

É montar um ambiente de trabalho que você vai conseguir reproduzir, que outro colega vai conseguir clonar e rodar, e que não vai te dar dor de cabeça quando você precisar juntar dois projetos na mesma máquina.

No mercado, não adianta falar "na minha máquina roda". Você precisa ter controle de versão, dependências organizadas, credenciais separadas do código e um projeto que faça sentido para quem vier depois de você.

Esse guia resolve isso passo a passo.

---

## Índice

1. [O que vamos construir](#1-o-que-vamos-construir)
2. [Onde cada coisa é feita](#2-onde-cada-coisa-é-feita)
3. [Instalando o Pyenv](#3-instalando-o-pyenv)
4. [Instalando versões do Python](#4-instalando-versões-do-python)
5. [Usando versões diferentes em projetos diferentes](#5-usando-versões-diferentes-em-projetos-diferentes)
6. [Criando um ambiente virtual com venv](#6-criando-um-ambiente-virtual-com-venv)
7. [Instalando bibliotecas e entendendo o pip freeze](#7-instalando-bibliotecas-e-entendendo-o-pip-freeze)
8. [Instalando o pipx](#8-instalando-o-pipx)
9. [Instalando o uv](#9-instalando-o-uv)
10. [Criando um projeto com uv](#10-criando-um-projeto-com-uv)
11. [Entendendo o pyproject.toml](#11-entendendo-o-pyprojecttoml)
12. [Ativando o ambiente do projeto](#12-ativando-o-ambiente-do-projeto)
13. [Variáveis de ambiente e o arquivo .env](#13-variáveis-de-ambiente-e-o-arquivo-env)
14. [Lendo o .env com Python](#14-lendo-o-env-com-python)
15. [Executando o projeto](#15-executando-o-projeto)
16. [Protegendo o .env com .gitignore](#16-protegendo-o-env-com-gitignore)
17. [Iniciando o versionamento com Git](#17-iniciando-o-versionamento-com-git)
18. [Primeiro commit e envio para o GitHub](#18-primeiro-commit-e-envio-para-o-github)
19. [Simulando o colega que clona o projeto](#19-simulando-o-colega-que-clona-o-projeto)
20. [O colega cria o próprio .env](#20-o-colega-cria-o-próprio-env)
21. [Boa prática: o arquivo .env.example](#21-boa-prática-o-arquivo-envexample)
22. [Erros comuns no Windows](#22-erros-comuns-no-windows)

---

## 1. O que vamos construir

Ao final deste processo, você vai ter:

- controle de versões do Python com `pyenv`
- ambientes isolados por projeto com `venv` e `uv`
- dependências declaradas de forma clara
- credenciais separadas do código com `.env`
- arquivos sensíveis protegidos com `.gitignore`
- projeto versionado e publicado no GitHub

Isso não é exercício de sala de aula. É o setup que usamos em projetos reais.

---

## 2. Onde cada coisa é feita

Antes de começar, guarde esta regra:

| Onde | Para quê |
|---|---|
| PowerShell como Administrador | apenas para instalar o `pyenv` |
| Terminal do VSCode | todo o restante |

O terminal do VSCode é onde você vai trabalhar de agora em diante. Acostume-se com ele desde o início.

---

## 3. Instalando o Pyenv

### O que é o Pyenv?

O `pyenv` é uma ferramenta que permite controlar versões do Python na sua máquina.

Na prática: você pode ter o Python 3.10 em um projeto e o Python 3.12 em outro, na mesma máquina, sem um quebrar o outro.

Isso resolve um problema muito comum: você instala uma versão nova do Python e algo que funcionava antes para de funcionar. Com o `pyenv`, cada projeto fica com sua versão, isolada.

### Instalando

Abra o **PowerShell como Administrador** e execute:

```powershell
cd $env:USERPROFILE
Invoke-WebRequest -UseBasicParsing -Uri "https://raw.githubusercontent.com/pyenv-win/pyenv-win/master/pyenv-win/install-pyenv-win.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
```

O que esse comando faz:

- `cd $env:USERPROFILE` — vai até a pasta do seu usuário, algo como `C:\Users\SeuNome`
- `Invoke-WebRequest ...` — baixa o script de instalação do repositório oficial do pyenv-win
- `&"./install-pyenv-win.ps1"` — executa o script que acabou de ser baixado


<details>
  <summary>Caso surja uma falha na instalação, expanda aqui.</summary>
Em algumas versões do windows existe um bloqueio para execução de scripts externos, retornado uma mensagem como essa: 

```
& : O arquivo install-pyenv-win.ps1 não pode ser carregado porque a execução de scripts foi
desabilitada neste sistema. Para obter mais informações, consulte about_Execution_Policies em
https://go.microsoft.com/fwlink/?LinkID=135170.
No linha:2 caractere:173
+ ... n.ps1" -OutFile "./install-pyenv-win.ps1"; &"./install-pyenv-win.ps1"
+                                                 ~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : ErrodeSegurança: (:) [], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess
```
Para resolver o problema, basta executar antes de tudo o comando abaixo: 

`Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process`

Basicamente, esse comando realiza as seguintes alterações: 

- `RemoteSigned`: Adiciona permissão para que scripts criados localmente sejam executados e exige que scripts baixados da internet sejam assinados por um editor confiável.

- `Scope Process`: Aplica essa mudança apenas à janela atual do PowerShell. Assim que você fechá-la, a segurança volta ao normal, o que é mais seguro para o seu computador.
</details>



Depois da instalação, **feche o PowerShell** e abra o VSCode. A partir daqui, tudo será feito no terminal do VSCode.

> O terminal novo reconhece as alterações de instalação. Trabalhar em um terminal antigo pode fazer o `pyenv` não ser encontrado.

---

## 4. Instalando versões do Python

No terminal do VSCode:

```bash
pyenv --version
pyenv install 3.10.11
pyenv install 3.12.0
pyenv versions
pyenv global 3.12.0
python --version
```

Entendendo cada comando:

- `pyenv --version` — confirma que o pyenv foi instalado. Se mostrar uma versão, está tudo certo.
- `pyenv install 3.10.11` — instala o Python 3.10.11. Não substitui nada: fica disponível ao lado de outras versões.
- `pyenv install 3.12.0` — instala o Python 3.12.0. As duas versões coexistem.
- `pyenv versions` — lista todas as versões instaladas via pyenv.
- `pyenv global 3.12.0` — define a versão padrão da máquina. É o Python usado quando você não está dentro de um projeto com versão específica.
- `python --version` — confirma qual versão está ativa. O esperado é `Python 3.12.0`.

---

## 5. Usando versões diferentes em projetos diferentes

Agora vamos provar, na prática, por que esse controle importa.

```bash
mkdir projeto-a
mkdir projeto-b

cd projeto-a
pyenv local 3.10.11
python --version

cd ..
cd projeto-b
pyenv local 3.12.0
python --version

cd ..
python --version
```

O que está acontecendo:

- `pyenv local 3.10.11` — dentro de `projeto-a`, define que a versão do Python será a 3.10.11. O pyenv cria um arquivo `.python-version` com essa informação.
- `python --version` dentro de `projeto-a` — deve retornar `Python 3.10.11`
- `python --version` dentro de `projeto-b` — deve retornar `Python 3.12.0`
- `python --version` fora das pastas — volta para a versão global

O ponto central: **cada projeto decide qual versão do Python ele usa**, sem depender do que está instalado globalmente na máquina.

---

## 6. Criando um ambiente virtual com venv

Mesmo que dois projetos usem a mesma versão do Python, eles podem precisar de bibliotecas completamente diferentes. É por isso que usamos ambientes virtuais.

### O que é um ambiente virtual?

Um ambiente virtual é um espaço isolado para as dependências de um projeto específico.

- o Python é o motor
- o ambiente virtual é a garagem daquele projeto
- as bibliotecas ficam ali dentro, separadas de tudo

### Criando

```bash
cd projeto-a
python -m venv venv
venv\Scripts\activate
```

- `python -m venv venv` — cria o ambiente virtual. O segundo `venv` é o nome da pasta criada.
- `venv\Scripts\activate` — ativa o ambiente no Windows. Quando ativado, o terminal passa a mostrar `(venv)` no início da linha.

Esse `(venv)` indica que você está dentro do ambiente isolado daquele projeto.

---

## 7. Instalando bibliotecas e entendendo o pip freeze

Com o ambiente ativado:

```bash
pip install pandas
pip freeze
deactivate
```

- `pip install pandas` — instala o pandas dentro do ambiente virtual ativo.
- `pip freeze` — lista todos os pacotes instalados no ambiente.

**Observação importante:** mesmo instalando só o `pandas`, o `pip freeze` vai mostrar vários pacotes. Isso acontece porque o pandas depende de outras bibliotecas. Você pediu uma, o sistema instalou ela e todas as dependências necessárias.

Com o tempo, usar só `pip install` e `pip freeze` pode deixar o ambiente confuso — você acumula dependências sem saber o que foi escolha sua e o que entrou por tabela. É justamente por isso que vamos usar o `uv`.

---

## 8. Instalando o pipx

```bash
pip install pipx
python -m pipx ensurepath
```

**Feche e reabra o terminal do VSCode depois desse passo.**

### O que é o pipx?

O `pipx` serve para instalar ferramentas Python de terminal de forma isolada, sem misturar com os pacotes dos seus projetos.

---

## 9. Instalando o uv

```bash
pip install uv
uv --version
```

### O que é o uv?

O `uv` é uma ferramenta moderna para gerenciar projetos Python. Ele cuida de ambiente virtual, dependências, execução do projeto e sincronização do ambiente — tudo em um lugar só.

Em vez de juntar `venv`, `pip`, `pip freeze` e `requirements.txt` manualmente, o `uv` organiza tudo de forma mais limpa e profissional.

---

## 10. Criando um projeto com uv

```bash
uv init projeto-final
cd projeto-final
uv add pandas
uv add requests
uv add python-dotenv
```

- `uv init projeto-final` — cria o projeto e monta a estrutura inicial
- `uv add pandas` — adiciona o pandas como dependência oficial do projeto

A diferença do `uv add` para o `pip install`: você está **declarando** que o projeto depende daquela biblioteca. Isso fica registrado de forma que qualquer pessoa que clonar consiga reproduzir o mesmo ambiente.

---

## 11. Entendendo o pyproject.toml

```bash
# No Git Bash
cat pyproject.toml

# No PowerShell
Get-Content pyproject.toml
```

### O que é esse arquivo?

O `pyproject.toml` é o arquivo central de projetos Python modernos. Ele guarda nome, versão e dependências do projeto — apenas o que você declarou, sem dependências automáticas poluindo a lista.

Compare com o `pip freeze`: lá aparecem 6 pacotes para 1 instalado. Aqui aparecem só os que você pediu.

---

## 12. Ativando o ambiente do projeto

```bash
.venv\Scripts\Activate.ps1
```

O `uv` cria automaticamente a pasta `.venv` ao iniciar o projeto. Quando você ativa esse ambiente, tudo o que for executado passa a usar o contexto daquele projeto.

---

## 13. Variáveis de ambiente e o arquivo .env

```powershell
New-Item .env
code .env
```

Adicione este conteúdo:

```
DB_HOST=servidor.banco.com
DB_USER=admin
DB_PASSWORD=senha-super-secreta-123
API_KEY=ghp_chave-fake-para-teste
```

### Por que isso existe?

Porque nem tudo deve ficar escrito diretamente no código.

❌ Forma errada:
```python
senha = "minha-senha-real"
```

✅ Forma certa:
```python
senha = os.getenv("DB_PASSWORD")
```

A lógica é simples:
- o código diz **o que precisa** → `DB_HOST`, `DB_USER`, `DB_PASSWORD`
- o `.env` entrega **os valores** → `servidor.banco.com`, `admin`, `senha-real`

Isso deixa o código mais limpo, mais seguro e mais fácil de adaptar para ambientes diferentes.

---

## 14. Lendo o .env com Python

Edite o `main.py`:

```python
from dotenv import load_dotenv
import os

load_dotenv()

db_host = os.getenv("DB_HOST")
db_user = os.getenv("DB_USER")
db_password = os.getenv("DB_PASSWORD")
api_key = os.getenv("API_KEY")

print(f"Conectando em {db_host} com usuário {db_user}")
print("Credenciais carregadas com sucesso!")
```

- `load_dotenv()` — carrega o conteúdo do `.env` e disponibiliza para o Python
- `os.getenv("DB_HOST")` — busca o valor da variável. Se não existir, retorna `None`.

---

## 15. Executando o projeto

```bash
uv run main.py
```

Saída esperada:

```
Conectando em servidor.banco.com com usuário admin
Credenciais carregadas com sucesso!
```

---

## 16. Protegendo o .env com .gitignore

Abra o `.gitignore` gerado pelo projeto e adicione:

```
# Credenciais
.env
```

### O que é o .gitignore?

É um arquivo que diz ao Git: "esses arquivos não devem ser versionados."

O `.env` pode conter senhas, chaves e tokens. Esse tipo de informação não deve ir para o repositório. Essa é uma das primeiras coisas que qualquer profissional faz ao iniciar um projeto.

---

## 17. Iniciando o versionamento com Git

```bash
git init
git status
git add .
git status
```

- `git init` — transforma a pasta em um repositório Git
- `git status` — mostra o estado atual dos arquivos
- `git add .` — adiciona os arquivos para a área de preparação do commit

Como o `.env` está no `.gitignore`, ele fica de fora. Esse é exatamente o comportamento esperado.

---

## 18. Primeiro commit e envio para o GitHub

```bash
git add .
git commit -m "Setup inicial do projeto profissional"
git branch -M main
git remote add origin https://github.com/seu-usuario/projeto-final.git
git pull origin main --allow-unrelated-histories
git push -u origin main
```

- `git commit -m "..."` — cria um registro da versão atual do projeto
- `git branch -M main` — define a branch principal como `main`
- `git remote add origin ...` — conecta o projeto local ao repositório remoto no GitHub
- `git pull origin main --allow-unrelated-histories` — necessário quando o repositório já foi criado com algum arquivo inicial no GitHub
- `git push -u origin main` — envia o projeto para o GitHub

### O que verificar no GitHub depois do envio

✅ Deve estar lá: `main.py`, `pyproject.toml`, `.gitignore`, `uv.lock`

❌ Não deve estar lá: `.env`

Se o `.env` não apareceu, você protegeu corretamente suas credenciais.

---

## 19. Simulando o colega que clona o projeto

```bash
deactivate
git clone https://github.com/seu-usuario/projeto-final.git projeto-colega
cd projeto-colega
uv sync
uv run main.py
```

- `uv sync` — sincroniza o ambiente com as dependências declaradas no `pyproject.toml`
- `uv run main.py` — executa o projeto

O `.env` não veio com o clone. Ao rodar, os valores vão aparecer como `None`. Isso é esperado — e é exatamente o comportamento correto.

O que isso demonstra:
- o código foi compartilhado ✅
- as dependências foram compartilhadas ✅
- as credenciais não foram compartilhadas ✅

---

## 20. O colega cria o próprio .env

```powershell
New-Item .env
code .env
```

Conteúdo:

```
DB_HOST=servidor.banco.com
DB_USER=colega
DB_PASSWORD=senha-do-colega
API_KEY=chave-do-colega
```

```bash
uv run main.py
```

Saída esperada:

```
Conectando em servidor.banco.com com usuário colega
Credenciais carregadas com sucesso!
```

Mesmo projeto. Máquinas diferentes. Credenciais diferentes. Tudo funcionando.

---

## 21. Boa prática: o arquivo .env.example

Crie também um arquivo chamado `.env.example`:

```
DB_HOST=
DB_USER=
DB_PASSWORD=
API_KEY=
```

Esse arquivo **pode** subir para o GitHub — não expõe valores reais.

Ele serve para mostrar quais variáveis o projeto precisa. Quando outro colega clonar o repositório, ele sabe exatamente o que deve criar no próprio `.env`.

---

## 22. Erros comuns no Windows

### "pyenv não é reconhecido como comando"

Feche e reabra o terminal. Se o problema persistir, verifique se as variáveis de ambiente foram configuradas. Às vezes é necessário reiniciar o computador.

### "A execução de scripts está desabilitada neste sistema"

Abra o PowerShell como Administrador e execute:

```powershell
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### "uv não é reconhecido após instalação"

Use `pip install uv` diretamente — foi o método que funcionou no Windows durante os testes.

### "venv\Scripts\activate não funciona no PowerShell"

Use:

```powershell
.venv\Scripts\Activate.ps1
```

### "O repositório já tem conteúdo e o push foi rejeitado"

Execute antes do push:

```bash
git pull origin main --allow-unrelated-histories
```

Depois repita o `git push`.

### "O .env subiu para o GitHub"

Remova o arquivo do histórico:

```bash
git rm --cached .env
```

Adicione ao `.gitignore`, faça um novo commit e suba. Troque todas as credenciais que estavam no arquivo — considere que elas foram expostas.

---

## Resumo

Você saiu de um cenário onde tudo ficava solto e sem controle, e montou um setup com:

- `pyenv` controlando versões do Python por projeto
- `uv` organizando dependências e ambiente de forma moderna
- `.env` separando configuração e credenciais do código
- `.gitignore` protegendo o que não deve subir
- `git` e `GitHub` versionando e compartilhando o projeto

Isso não é só instalar ferramenta. É começar a trabalhar com Python de um jeito mais limpo, mais seguro e mais próximo do que você vai encontrar em projetos reais.

---

> Material desenvolvido por Iago Braz · [Linkedin](https://www.linkedin.com/in/iagobraz/)  · [Pós-graduação em Engenharia de Dados](https://xperiun.com/ed/postech/)
