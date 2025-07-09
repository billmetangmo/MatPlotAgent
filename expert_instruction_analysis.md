# Rôle de `expert_instruction` dans le benchmark MatPlotBench

## Vue d'ensemble du projet

**MatPlotAgent** est un framework d'agent LLM conçu pour automatiser les tâches de visualisation de données scientifiques. **MatPlotBench** est un benchmark de 100 cas de test pour évaluer les approches de visualisation de données.

## Qu'est-ce que `expert_instruction` ?

Dans le benchmark MatPlotBench, `expert_instruction` fait partie d'une stratégie comparative qui évalue l'efficacité des LLMs avec différents niveaux de détail dans les instructions :

### Les deux types d'instructions

1. **`simple_instruction` (instruction simple/novice)** : 
   - Instructions courtes et générales
   - Exemple : *"Generate a series of boxplots using matplotlib and numpy libraries..."*

2. **`expert_instruction` (instruction experte)** :
   - Instructions détaillées et techniques étape par étape
   - Exemple : *"1. Import the required libraries: matplotlib.pyplot and numpy. 2. Import the Polygon module from matplotlib.patches. 3. Set the random seed for reproducibility using np.random.seed(19920215)..."*

## Objectif du benchmark

Le benchmark compare comment les LLMs performent avec :
- Des instructions **simples** (novice_instruction)
- Des instructions **expertes** détaillées (expert_instruction)

## Workflow dans MatPlotAgent

```python
def mainworkflow(expert_instruction, simple_instruction, workspace, max_try=3):
    # Query expanding
    query_expansion_agent = QueryExpansionAgent(expert_instruction, simple_instruction, model_type=args.model_type)
    expanded_simple_instruction = query_expansion_agent.run('simple')
    # ... reste du workflow
```

L'agent `QueryExpansionAgent` utilise les deux types d'instructions pour :
1. **Interpréter** les exigences de l'utilisateur  
2. **Transformer** les instructions simples en instructions compatibles avec les LLMs
3. **Générer** du code pour créer des visualisations

## Résultats du benchmark

Le benchmark démontre que **MatPlotAgent améliore significativement les performances** par rapport au décodage direct :

| Modèle | Direct Decoding | MatPlotAgent w/ GPT-4V |
|--------|----------------|------------------------|
| GPT-4 | 48.86 | 61.16 (**+12.30**) |
| GPT-3.5 | 38.03 | 47.51 (**+9.48**) |

## Structure des données

Chaque entrée du benchmark contient :
```json
{
    "simple_instruction": "Instruction courte et générale",
    "expert_instruction": "Instructions détaillées étape par étape avec 1. 2. 3. ...",
    "id": 1
}
```

## Conclusion

`expert_instruction` sert à :
- **Évaluer** l'impact du niveau de détail des instructions sur la performance des LLMs
- **Fournir** un référentiel d'instructions expertes pour l'expansion de requêtes
- **Démontrer** l'efficacité du framework MatPlotAgent dans l'amélioration des capacités de visualisation des LLMs

Le benchmark montre que même avec des instructions expertes détaillées, MatPlotAgent apporte des améliorations substantielles, validant l'approche du framework pour l'automatisation de la visualisation scientifique.