# 📝 Organisation des Prompts dans MatPlotAgent

## 🗂️ Structure générale

Les prompts sont organisés dans **3 agents principaux**, chacun avec son fichier `prompt.py` :

```
agents/
├── query_expansion_agent/
│   ├── agent.py
│   ├── prompt.py          ← Prompts d'expansion de requêtes
│   └── __init__.py
├── plot_agent/
│   ├── agent.py
│   ├── prompt.py          ← Prompts de génération de code
│   └── __init__.py
├── visual_refine_agent/
│   ├── agent.py
│   ├── prompt.py          ← Prompts de raffinement visuel
│   └── __init__.py
└── utils.py               ← Fonctions utilitaires pour les prompts
```

## 🎯 1. Query Expansion Agent (`agents/query_expansion_agent/prompt.py`)

**Rôle** : Transformer les instructions utilisateur vagues en instructions détaillées

### Prompts disponibles :

```python
SYSTEM_PROMPT = '''According to the user query, expand and solidify the query into a step by step detailed instruction (or comment) on how to write python code to fulfill the user query's requirements. Import the appropriate libraries. Pinpoint the correct library functions to call and set each parameter in every function call accordingly.'''

EXPERT_USER_PROMPT = '''Here is the user query: [User Query]:
"""
{{query}}
"""
You should understand what the query's requirements are, and output step by step, detailed instructions on how to use python code to fulfill these requirements. Include what libraries to import, what library functions to call, how to set the parameters in each function correctly, how to prepare the data, how to manipulate the data so that it becomes appropriate for later functions to call etc,. Make sure the code to be executable and correctly generate the desired output in the user query.'''
```

## 🛠️ 2. Plot Agent (`agents/plot_agent/prompt.py`)

**Rôle** : Générer le code Python matplotlib à partir des instructions

### Prompts disponibles :

#### **Génération initiale :**
```python
INITIAL_SYSTEM_PROMPT = '''You are a cutting-edge super capable code generation LLM. You will be given a natural language query, generate a runnable python code to satisfy all the requirements in the query. You can use any python library you want. When you complete a plot, remember to save it to a png file.'''

INITIAL_USER_PROMPT = '''Here is the query:
"""
{{query}}
"""
If the query requires data manipulation from a csv file, process the data from the csv file and draw the plot in one piece of code.
When you complete a plot, remember to save it to a png file. The file name should be """{{file_name}}""".'''
```

#### **Raffinement basé sur feedback visuel :**
```python
VIS_SYSTEM_PROMPT = '''You are a cutting-edge super capable code generation LLM. You will be given a piece of code and natural language instruction on how to improve it. Base on the given code, generate a runnable python code to satisfy all the requirements in the instruction while retaining the original code's functionality. You can use any python library you want. When you complete a plot, remember to save it to a png file.'''

VIS_USER_PROMPT = '''Here is the code and instruction:
"""
{{query}}
"""
When you complete a plot, remember to save it to a png file. The file name should be """{{file_name}}""".'''
```

#### **Chain-of-Thought :**
```python
ZERO_SHOT_COT_PROMPT = '''
Here is the query:
"""
{{query}}
"""
Let's think step by step when you complete the query.
When you complete a plot, remember to save it to a png file. The file name should be """{{file_name}}"""'''
```

#### **Gestion d'erreurs :**
```python
ERROR_PROMPT = '''There are some errors in the code you gave:
{{error_message}}
please correct the errors.
Then give the complete code and don't omit anything even though you have given it in the above code.'''
```

## 👁️ 3. Visual Refine Agent (`agents/visual_refine_agent/prompt.py`)

**Rôle** : Analyser l'image générée et suggérer des améliorations

### Prompts disponibles :

```python
SYSTEM_PROMPT = '''Given a piece of code, a user query, and an image of the current plot, please determine whether the plot has faithfully followed the user query. Your task is to provide instruction to make sure the plot has strictly completed the requirements of the query. Please output a detailed step by step instruction on how to use python code to enhance the plot.'''

USER_PROMPT = '''Here is the code: [Code]:
"""
{{code}}
"""
Here is the user query: [Query]:
"""
{{query}}
"""
Carefully read and analyze the user query to understand the specific requirements. Examine the provided Python code to understand how the current plot is generated. Check if the code aligns with the user query in terms of data selection, plot type, and any specific customization. Look at the provided image of the plot. Assess the plot type, the data it represents, labels, titles, colors, and any other visual elements. Compare these elements with the requirements specified in the user query. Note any differences between the user query requirements and the current plot. Based on the identified discrepancies, provide step-by-step instructions on how to modify the Python code to meet the user query requirements. Suggest improvements for better visualization practices, such as clarity, readability, and aesthetics, while ensuring the primary focus is on meeting the user's specified requirements.
Remember to save the plot to a png file. The file name should be """{{file_name}}"""'''

ERROR_PROMPT = '''There are some errors in the code you gave:
{{error_message}}
please correct the errors.
Then give the complete code and don't omit anything even though you have given it in the above code.'''
```

## 🔧 4. Utilitaires (`agents/utils.py`)

**Fonction de gestion des templates :**
```python
def fill_in_placeholders(prompt_messages, placeholders: dict):
    """Remplace les placeholders {{variable}} dans les prompts"""
    filled_messages = deepcopy(prompt_messages)
    # ... logique de remplacement
```

## 🎭 Utilisation des placeholders

Les prompts utilisent des **placeholders** au format `{{variable}}` :

- `{{query}}` : La requête utilisateur
- `{{file_name}}` : Le nom du fichier de sortie  
- `{{error_message}}` : Messages d'erreur
- `{{code}}` : Code Python existant

## 🔄 Workflow des prompts

1. **QueryExpansionAgent** : `SYSTEM_PROMPT` + `EXPERT_USER_PROMPT`
2. **PlotAgent** : `INITIAL_SYSTEM_PROMPT` + `INITIAL_USER_PROMPT`
3. **VisualRefineAgent** : `SYSTEM_PROMPT` + `USER_PROMPT` (avec image)
4. **PlotAgent** (raffinement) : `VIS_SYSTEM_PROMPT` + `VIS_USER_PROMPT`

## 📍 Accès aux prompts

```python
# Query Expansion
from agents.query_expansion_agent.prompt import SYSTEM_PROMPT, EXPERT_USER_PROMPT

# Plot Generation  
from agents.plot_agent.prompt import INITIAL_SYSTEM_PROMPT, INITIAL_USER_PROMPT, VIS_SYSTEM_PROMPT, VIS_USER_PROMPT, ZERO_SHOT_COT_PROMPT, ERROR_PROMPT

# Visual Refinement
from agents.visual_refine_agent.prompt import SYSTEM_PROMPT, USER_PROMPT, ERROR_PROMPT
```

## 🎛️ Options de configuration

- `no_sysprompt=True` : Désactive le system prompt pour certains tests
- Support pour différents modèles (GPT-3.5, GPT-4, etc.)
- Gestion des erreurs avec re-prompting automatique