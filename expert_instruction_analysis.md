# Rôle de `expert_instruction` dans le benchmark MatPlotBench

## Vue d'ensemble du projet

**MatPlotAgent** est un framework d'agent LLM conçu pour automatiser les tâches de visualisation de données scientifiques. **MatPlotBench** est un benchmark de 100 cas de test pour évaluer les approches de visualisation de données.

## ⚠️ Usage Benchmark vs Usage Réel - IMPORTANT

### Dans le Benchmark (ce qu'on voit dans le code) :
```python
# Usage BENCHMARK - pour tester les performances
mainworkflow(expert_instruction, simple_instruction, workspace=directory_path)
```

### En Usage Réel (pour un utilisateur normal) :
```python
# Usage RÉEL - l'utilisateur ne fournit que sa demande
user_request = "Create a scatter plot with correlation data"
query_expansion_agent = QueryExpansionAgent("", user_request, model_type='gpt-4')
expanded_instruction = query_expansion_agent.run('simple')
```

**Non, un utilisateur normal n'a PAS besoin de fournir expert_instruction !**

### Pourquoi cette confusion ?

Le code actuel est principalement **orienté recherche/benchmark**, donc tous les scripts utilisent les deux instructions pour comparer les performances. Mais le système `QueryExpansionAgent` utilise en fait un **prompt générique** qui améliore automatiquement n'importe quelle instruction utilisateur :

```python
# Le prompt système de QueryExpansionAgent
SYSTEM_PROMPT = '''According to the user query, expand and solidify the query into a step by step detailed instruction on how to write python code to fulfill the user query's requirements.'''
```

Il n'a pas besoin d'instructions expertes pré-écrites !

## Qu'est-ce que `expert_instruction` et `simple_instruction` ?

Dans le benchmark MatPlotBench, ces deux types d'instructions font partie d'une stratégie comparative qui évalue l'efficacité des LLMs avec différents niveaux de détail :

### Les deux types d'instructions

1. **`simple_instruction` (instruction simple/novice)** : 
   - Instructions courtes et générales, proches du langage naturel
   - Style d'instruction qu'un utilisateur novice donnerait
   - Exemple : *"Generate a series of boxplots using matplotlib and numpy libraries. The boxplots should include a basic plot, a notched plot, a plot with changed outlier point symbols..."*
   - **Utilisée comme baseline** pour évaluer les performances des LLMs

2. **`expert_instruction` (instruction experte)** :
   - Instructions détaillées et techniques étape par étape
   - Style d'instruction qu'un expert en programmation donnerait
   - Exemple : *"1. Import the required libraries: matplotlib.pyplot and numpy. 2. Import the Polygon module from matplotlib.patches. 3. Set the random seed for reproducibility using np.random.seed(19920215)..."*
   - **Utilisée comme référence** pour l'expansion de requêtes

## Objectif du benchmark

Le benchmark compare comment les LLMs performent avec :
- Des instructions **simples** (novice_instruction)
- Des instructions **expertes** détaillées (expert_instruction)

## Rôle spécifique de `simple_instruction`

### Dans le workflow MatPlotAgent :

`simple_instruction` est **l'instruction principale** utilisée dans le pipeline :

```python
def mainworkflow(expert_instruction, simple_instruction, workspace, max_try=3):
    # Query expanding - utilise les deux types
    query_expansion_agent = QueryExpansionAgent(expert_instruction, simple_instruction, model_type=args.model_type)
    expanded_simple_instruction = query_expansion_agent.run('simple')  # Traite la simple_instruction
    
    # Plot Agent - utilise l'instruction simple expandée
    action_agent = PlotAgent(config, expanded_simple_instruction)
    novice_log, novice_code = action_agent.run_initial(args.model_type, 'novice.png')
```

### Comment `simple_instruction` est utilisée :

1. **Input principal** : `simple_instruction` représente ce qu'un utilisateur novice demanderait
2. **Expansion de requête** : L'agent `QueryExpansionAgent` utilise `expert_instruction` comme référence pour **enrichir** et **clarifier** la `simple_instruction`
3. **Génération de code** : La `simple_instruction` expandée est utilisée par `PlotAgent` pour générer le code matplotlib
4. **Évaluation** : Les résultats sont comparés aux méthodes de décodage direct

### Avantage de cette approche :

- **Réaliste** : Simule les instructions qu'un utilisateur réel donnerait
- **Extensible** : Permet d'améliorer les instructions vagues grâce au knowledge des `expert_instruction`
- **Mesurable** : Fournit une baseline claire pour évaluer les améliorations

## Résultats du benchmark

Le benchmark démontre que **MatPlotAgent améliore significativement les performances** par rapport au décodage direct :

| Modèle | Direct Decoding | MatPlotAgent w/ GPT-4V |
|--------|----------------|------------------------|
| GPT-4 | 48.86 | 61.16 (**+12.30**) |
| GPT-3.5 | 38.03 | 47.51 (**+9.48**) |

## Comparaison concrète avec un exemple

**Pour le même objectif (créer des boxplots)** :

### `simple_instruction` :
```
"Generate a series of boxplots using matplotlib and numpy libraries. The boxplots should include a basic plot, a notched plot, a plot with changed outlier point symbols, a plot without outlier points, a horizontal boxplot, and a plot with changed whisker length."
```

### `expert_instruction` :
```
"1. Import the required libraries: matplotlib.pyplot and numpy.
2. Import the Polygon module from matplotlib.patches.
3. Set the random seed for reproducibility using np.random.seed(19920215).
4. Generate new fake data for the plot by creating arrays: spread, center, flier_high, flier_low, and data.
   - Generate the spread array by multiplying np.random.rand(50) by 100.
   - Generate the center array by multiplying np.ones(25) by 60.
   [... 13 étapes détaillées au total]"
```

## Structure des données

Chaque entrée du benchmark contient :
```json
{
    "simple_instruction": "Instruction courte et générale (utilisée dans le pipeline)",
    "expert_instruction": "Instructions détaillées étape par étape (référence pour l'expansion)",
    "id": 1
}
```

## Conclusion

### Rôles complémentaires :

**`simple_instruction`** sert à :
- **Représenter** les demandes réelles d'utilisateurs novices
- **Servir de baseline** pour mesurer les améliorations du framework
- **Être l'input principal** traité par le pipeline MatPlotAgent
- **Tester** la capacité du système à interpréter des instructions vagues

**`expert_instruction`** sert à :
- **Fournir** un référentiel d'instructions expertes pour l'expansion de requêtes
- **Guider** l'amélioration des instructions simples
- **Évaluer** l'impact du niveau de détail sur les performances
- **Démontrer** que même avec des instructions expertes, MatPlotAgent apporte de la valeur

### Impact du système :
Le benchmark prouve que MatPlotAgent transforme efficacement les `simple_instruction` (style utilisateur novice) en code de qualité, avec des améliorations substantielles :
- **+12.30 points** pour GPT-4
- **+9.48 points** pour GPT-3.5

Cela valide l'approche du framework pour combler le gap entre les instructions utilisateur naturelles et la génération de code de visualisation de qualité professionnelle.

## 🎯 Pour l'utilisateur final - Usage Pratique

### Ce que vous devez retenir :

1. **Usage normal** : Vous donnez juste votre demande en langage naturel
   ```python
   "Create a histogram of my sales data with error bars"
   ```

2. **Le système s'occupe du reste** : MatPlotAgent améliore automatiquement votre demande grâce à :
   - Son agent d'expansion de requêtes (QueryExpansionAgent)  
   - Son agent de génération de code (PlotAgent)
   - Son agent de raffinement visuel (VisualRefineAgent)

3. **Pas besoin d'instructions techniques** : Le framework transforme votre langage naturel en instructions techniques automatiquement

4. **Dans le benchmark** : Les deux types d'instructions servent à mesurer les performances, pas à définir l'usage normal

### Résumé :
**MatPlotAgent = Interface simple pour l'utilisateur + Intelligence technique en arrière-plan**