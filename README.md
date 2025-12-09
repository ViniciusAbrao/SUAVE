````markdown
# ✅ README – Reprodutibilidade do Ambiente SUAVE 2.5.2 + Modificações

Este documento descreve como **exportar**, **fazer backup**, **transportar** e **restaurar** completamente o ambiente SUAVE usado neste projeto, incluindo as **modificações manuais feitas dentro do SUAVE no ambiente virtual** e também a pasta `~/Downloads/SUAVE`.

Seguindo estes passos, qualquer máquina conseguirá rodar o projeto *exatamente igual* ao ambiente original.

---

## 1. 📦 Exportar o Ambiente Conda Completo

Dentro do ambiente:

```bash
conda activate suave_env
conda env export > suave_env_full.yml
````

Isso gera um arquivo YAML contendo:

* Python version
* Pacotes instalados
* Versões exatas
* Dependências congeladas

> **IMPORTANTE:** este YAML **não** contém alterações nos arquivos `.py` do SUAVE — por isso precisamos dos próximos passos.

---

## 2. 📁 Backup do SUAVE Modificado Dentro do Ambiente Virtual

Primeiro descubra onde o SUAVE está instalado:

```bash
python - << 'EOF'
import SUAVE
print(SUAVE.__file__)
EOF
```

Isso retorna algo como:

```
/home/abrao/anaconda3/envs/suave_env/lib/python3.8/site-packages/SUAVE-2.5.2-py3.8.egg/SUAVE/__init__.py
```

O diretório que queremos salvar é:

```
.../site-packages/SUAVE-2.5.2-py3.8.egg
```

### Fazer o backup:

```bash
cd /home/abrao/anaconda3/envs/suave_env/lib/python3.8/site-packages

# copiar a pasta inteira para sua HOME/Downloads
cp -r SUAVE-2.5.2-py3.8.egg ~/Downloads/SUAVE_2.5.2_mod

# compactar
cd ~/Downloads
tar czf SUAVE_2.5.2_mod.tar.gz SUAVE_2.5.2_mod
```

Esse arquivo contém **todas as modificações feitas no código-fonte do SUAVE**.

---

## 3. 📁 Backup da Pasta de Downloads (`~/Downloads/SUAVE`)

Se você tem a pasta original (tutoriais, airfoils, scripts), salve também:

```bash
cd ~/Downloads
tar czf SUAVE_downloads_original.tar.gz SUAVE
```

---

## 4. 🚚 Transportar os Arquivos para Outra Máquina

Leve estes arquivos:

* `suave_env_full.yml`
* `SUAVE_2.5.2_mod.tar.gz`
* (opcional) `SUAVE_downloads_original.tar.gz`

Você pode usar pendrive, GitHub, Google Drive, etc.

---

## 5. 🔧 Restaurar o Ambiente em Outra Máquina

### 5.1 Criar o ambiente Conda

```bash
cd SUAVE/my_files
conda env create -f suave_env_full.yml
conda activate suave_env
```

### 5.2 Instalar SUAVE oficial antes de sobrescrever

```bash
cd SUAVE/trunk
python setup.py install
```

### 5.3 Descobrir o caminho do SUAVE na máquina nova

```bash
python - << 'EOF'
import SUAVE
print(SUAVE.__file__)
EOF
```

Saída exemplo:

```
/home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/SUAVE-2.5.2-py3.8.egg/SUAVE/__init__.py
```

O diretório-alvo será o mesmo `SUAVE-2.5.2-py3.8.egg`.

### 5.4 Sobrescrever com o SUAVE modificado

Supondo que o backup está em:

```bash
~/Downloads/SUAVE/my_files/SUAVE_2.5.2_mod/
  ├── SUAVE/
  └── EGG-INFO/
```

e o `site-packages` é:

```bash
/home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/
```

Apagar o `.egg` antigo. No terminal:

```bash
rm /home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/SUAVE-2.5.2-py3.8.egg
```

> (não tem problema se esse arquivo existir só como arquivo, é isso mesmo)

---

Copiar o pacote `SUAVE` modificado:

```bash
cp -r ~/Downloads/SUAVE/my_files/SUAVE_2.5.2_mod/SUAVE \
      /home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/
```

Isso vai criar:

```bash
/home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/SUAVE/
```

---

Copiar o `EGG-INFO` como metadado do pacote:

```bash
cp -r ~/Downloads/SUAVE/my_files/SUAVE_2.5.2_mod/EGG-INFO \
      /home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/SUAVE-2.5.2-py3.8.egg-info
```

(Esse nome `SUAVE-2.5.2-py3.8.egg-info` é padrão para o metadado; não é obrigatório, mas deixa o ambiente organizado.)

---

### 4️⃣ Conferir se o Python está pegando o SUAVE certo

Ainda com o `suave_env` ativado:

```bash
python - << 'EOF'
import SUAVE, os
print("SUAVE está sendo importado de:")
print(os.path.dirname(SUAVE.__file__))
EOF
```

Você deve ver algo como:

```bash
/home/outro_usuario/anaconda3/envs/suave_env/lib/python3.8/site-packages/SUAVE
```

Se aparecer isso, o SUAVE modificado do seu backup está ativo ✅

---

Se quiser, depois montamos um pequeno `restore_suave.sh` para você rodar na máquina nova e ele já fazer esses `rm` + `cp` automaticamente.


---

## 6. ✔️ Verificar Instalação

```bash
python - << 'EOF'
import SUAVE
print("SUAVE OK – versão:", SUAVE.__version__)
EOF
```

Também teste com seu script:

```bash
cd ~/Downloads/SUAVE/Tutorials-2.5.2
python3 tut_eVTOL.py
```

Se rodar sem erros, a reproducibilidade está garantida.

---

## 7. 🧩 Resumo dos Arquivos

| Arquivo                           | Conteúdo                                    |
| --------------------------------- | ------------------------------------------- |
| `suave_env_full.yml`              | Ambiente Conda completo (pinado)            |
| `SUAVE_2.5.2_mod.tar.gz`          | SUAVE modificado dentro do ambiente virtual |
| `SUAVE_downloads_original.tar.gz` | Pasta SUAVE baixada com tutoriais e scripts |

---

## 8. 📌 Observações Finais

* O Conda **NÃO salva** alterações dentro de pacotes (`.py`).
* Por isso o backup do diretório `SUAVE-2.5.2-py3.8.egg` é obrigatório.
* Para versionamento futuro, considere colocar essa cópia em um repositório Git.

---


