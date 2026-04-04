# 🏗️ Verificação de Escadas — NBR 9077 + NBR 9050
## Sistema Multi-Agente CrewAI + IfcOpenShell

**Projeto acadêmico — Master Internacional em IA para Arquitetura e Construção**  
**Zigurat Institute of Technology | Módulo 5 — Agentic IA**

---

## Objetivo

Sistema multi-agente que lê um arquivo IFC real via IfcOpenShell e verifica automaticamente a conformidade de escadas com as normas brasileiras **NBR 9077:2001** (Saídas de Emergência) e **NBR 9050:2020** (Acessibilidade), produzindo laudo técnico `.docx` e checklist `.xlsx`.

**Projeto analisado:** Residência Rua das Palmeiras 142 — Recife/PE  
**IFC utilizado:** `palmeiras_142_enriquecido.ifc` (original + escadas adicionadas via IfcOpenShell API)

---

## O que o sistema verifica

| Critério | Norma | Parâmetro |
|---|---|---|
| Altura do espelho (degrau) | NBR 9077 §5.2 | 16 a 18 cm |
| Profundidade do cobertor | NBR 9077 §5.2 | mín. 28 cm |
| Largura da escada | NBR 9077 §5.2.2 | mín. 1,20 m |
| Degraus por lance | NBR 9077 §5.2.3 | 3 a 18 unidades |
| Profundidade do patamar | NBR 9077 §5.2.4 | mín. 1,20 m |
| Largura da porta de acesso | NBR 9050 §4.6.3 / NBR 9077 §5.4 | mín. 0,90 m |
| Altura da porta de acesso | NBR 9050 §4.6.3 | mín. 2,10 m |
| Sentido de abertura da porta | NBR 9077 §5.4 | para fora da caixa de escada |

---

## Arquitetura do sistema

```
IFC (IfcOpenShell)
       │
       ▼
┌──────────────────┐     ┌──────────────────────┐     ┌─────────────────┐
│  Agente 1        │────▶│  Agente 2            │────▶│  Agente 3       │
│  Especialista BIM│     │  Auditor NBR         │     │  Redator Técnico│
│                  │     │                      │     │                 │
│  Analisa dados   │     │  verificar_escadas() │     │  gerar_xlsx()   │
│  IFC e qualidade │     │  verificar_portas()  │     │  gerar_docx()   │
│  dos Psets       │     │  + Parecer técnico   │     │                 │
└──────────────────┘     └──────────────────────┘     └─────────────────┘
                                                              │
                                              ┌───────────────┴───────────────┐
                                              ▼                               ▼
                                    checklist_nbr9077.xlsx         laudo_nbr9077.docx
```

**Ferramentas Python determinísticas** (sem LLM):
- `verificar_escadas` — verifica geometria dos degraus, largura e patamares
- `verificar_portas` — verifica dimensões e sentido de abertura das portas
- `gerar_xlsx` — checklist Excel com abas Resumo, Escadas e Portas
- `gerar_docx` — laudo técnico Word com parecer e recomendações

---

## Estrutura do repositório

```
├── nbr9077_escadas_crewai.ipynb       # Notebook principal (Google Colab)
├── palmeiras_142_enriquecido.ifc      # Arquivo IFC com escadas modeladas
├── README.md                          # Este arquivo
└── outputs/                           # Gerado na execução
    ├── laudo_nbr9077_*.docx
    └── checklist_nbr9077_*.xlsx
```

---

## Dependências

```
ifcopenshell>=0.8
crewai>=1.0
crewai-tools
python-docx
openpyxl
langchain-openai   # (instalado automaticamente pelo crewai)
```

---

## Como executar no Google Colab

**1.** Abra o notebook no Colab:  
`Arquivo → Abrir notebook → GitHub` → cole a URL do repositório

**2.** Faça upload do arquivo IFC:
```python
from google.colab import files
uploaded = files.upload()   # selecione palmeiras_142_enriquecido.ifc
```

**3.** Cole sua API Key na Célula 2:
```python
os.environ['OPENAI_API_KEY'] = 'sk-...'
```

**4.** Execute tudo:  
`Runtime → Run all` — aguarde ~3-5 minutos

**5.** Os arquivos `.docx` e `.xlsx` são baixados automaticamente ao final.

---

## Como executar localmente

```bash
# Clonar o repositório
git clone https://github.com/seu-usuario/nbr9077-escadas-crewai
cd nbr9077-escadas-crewai

# Criar ambiente virtual
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# Instalar dependências
pip install ifcopenshell crewai crewai-tools python-docx openpyxl

# Configurar API Key
export OPENAI_API_KEY='sk-...'

# Executar o notebook (converta para .py ou use jupyter)
jupyter notebook nbr9077_escadas_crewai.ipynb
```

---

## Casos de teste no IFC

O arquivo `palmeiras_142_enriquecido.ifc` contém propositalmente:

| Elemento | Espelho | Cobertor | Largura | Patamar | Status esperado |
|---|---|---|---|---|---|
| ESC-01 (Principal) | 17 cm ✅ | 29 cm ✅ | 1,20 m ✅ | 1,20×1,20 m ✅ | **CONFORME** |
| ESC-02 (Serviço) | 21 cm ❌ | 23 cm ❌ | 1,10 m ❌ | 1,10×0,90 m ❌ | **NÃO CONFORME** |
| Porta ESC-01 | 1,00 m ✅ | 2,10 m ✅ | SWING_LEFT ✅ | — | **CONFORME** |
| Porta ESC-02 | 0,80 m ❌ | 2,10 m ✅ | SWING_RIGHT ❌ | — | **NÃO CONFORME** |

---

## Outputs gerados

**`laudo_nbr9077_*.docx`** — Laudo técnico com:
- Capa com identificação do projeto e normas aplicadas
- Sumário executivo com índice de conformidade
- Tabela de parâmetros normativos
- Análise detalhada por escada (status, medições, não-conformidades)
- Análise detalhada por porta (dimensões, sentido de abertura)
- Parecer técnico do Agente Auditor
- Recomendações de adequação

**`checklist_nbr9077_*.xlsx`** — Checklist Excel com:
- Aba **Resumo**: índice geral + parâmetros normativos + legenda
- Aba **Escadas**: linha por escada com status colorido (verde/vermelho/amarelo)
- Aba **Portas de Acesso**: linha por porta com status e tipo de abertura

---

## Autor

**Sergio Rosenboim**  
Arquiteto | BIM, Data & AI for Construction  
Master Internacional em IA para Arquitetura e Construção — Zigurat Institute of Technology
