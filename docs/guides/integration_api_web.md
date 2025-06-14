# Guide d'intégration avec l'API Web

## Table des matières

1. [Introduction](#introduction)
2. [Architecture de l'API](#architecture-de-lapi)
3. [Endpoints pour les agents logiques](#endpoints-pour-les-agents-logiques)
   - [Conversion de texte en ensemble de croyances](#conversion-de-texte-en-ensemble-de-croyances)
   - [Exécution de requêtes](#exécution-de-requêtes)
   - [Génération de requêtes](#génération-de-requêtes)
4. [Exemples d'utilisation](#exemples-dutilisation)
   - [Exemple avec curl](#exemple-avec-curl)
   - [Exemple avec Python](#exemple-avec-python)
   - [Exemple avec JavaScript](#exemple-avec-javascript)
5. [Gestion des erreurs](#gestion-des-erreurs)
6. [Bonnes pratiques](#bonnes-pratiques)
7. [Limites et considérations](#limites-et-considérations)
8. [Ressources supplémentaires](#ressources-supplémentaires)

## Introduction

L'API Web fournit un accès aux fonctionnalités des agents logiques via des endpoints REST, permettant l'intégration de ces capacités dans des applications tierces. Ce guide explique comment utiliser l'API pour effectuer des opérations logiques, avec des exemples concrets dans différents langages de programmation.

L'API Web offre les avantages suivants:
- **Accessibilité**: Utilisation des agents logiques sans installation locale
- **Scalabilité**: Traitement côté serveur pour les opérations intensives
- **Interopérabilité**: Intégration facile avec différentes plateformes et langages
- **Maintenance simplifiée**: Mises à jour centralisées sans impact sur les clients

## Architecture de l'API

L'API Web est construite selon les principes REST et utilise le format JSON pour les requêtes et les réponses. Elle est organisée autour des ressources suivantes:

- **Belief Sets**: Ensembles de croyances logiques
- **Queries**: Requêtes logiques à exécuter sur les ensembles de croyances
- **Results**: Résultats des requêtes logiques

L'API est actuellement ouverte et ne met pas en œuvre de mécanismes d'authentification ou d'autorisation spécifiques. La protection de l'API, si nécessaire, est supposée être gérée par des mécanismes externes (par exemple, un pare-feu, un proxy inverse, une gateway API).

## Endpoints pour les agents logiques

### Conversion de texte en ensemble de croyances

**Endpoint**: `POST /api/logic/belief-set`

**Description**: Convertit un texte en langage naturel en un ensemble de croyances logiques formalisé.

**Requête**:
```json
{
  "text": "Si le ciel est nuageux, alors il va pleuvoir. Le ciel est nuageux.",
  "logic_type": "propositional",
  "options": {
    "include_explanation": true,
    "timeout": 10.0
  }
}
```

**Réponse**:
```json
{
  "success": true,
  "belief_set": {
    "id": "bs-12345",
    "logic_type": "propositional",
    "content": "nuageux => pluie\nnuageux",
    "source_text": "Si le ciel est nuageux, alors il va pleuvoir. Le ciel est nuageux."
  },
  "message": "Conversion réussie.",
  "processing_time": 0.45
}
```

### Exécution de requêtes

**Endpoint**: `POST /api/logic/query`

**Description**: Exécute une requête logique sur un ensemble de croyances existant.

**Requête**:
```json
{
  "belief_set_id": "bs-12345",
  "query": "pluie",
  "logic_type": "propositional",
  "options": {
    "include_explanation": true,
    "timeout": 5.0
  }
}
```

**Réponse**:
```json
{
  "success": true,
  "belief_set_id": "bs-12345",
  "logic_type": "propositional",
  "result": {
    "query": "pluie",
    "result": true,
    "formatted_result": "Tweety Result: Query 'pluie' is ACCEPTED (True).",
    "explanation": "La requête 'pluie' est acceptée car, selon l'ensemble de croyances, le ciel est nuageux et s'il est nuageux, alors il pleut. Par application du Modus Ponens, on peut conclure qu'il pleut."
  },
  "message": "Requête exécutée avec succès.",
  "processing_time": 0.12
}
```

### Génération de requêtes

**Endpoint**: `POST /api/logic/generate-queries`

**Description**: Génère des requêtes logiques pertinentes basées sur un texte et un ensemble de croyances.

**Requête**:
```json
{
  "belief_set_id": "bs-12345",
  "text": "Si le ciel est nuageux, alors il va pleuvoir. Le ciel est nuageux.",
  "logic_type": "propositional",
  "options": {
    "max_queries": 5,
    "timeout": 8.0
  }
}
```

**Réponse**:
```json
{
  "success": true,
  "belief_set_id": "bs-12345",
  "logic_type": "propositional",
  "queries": [
    "pluie",
    "nuageux => pluie",
    "nuageux && pluie",
    "!(nuageux && !pluie)",
    "!nuageux || pluie"
  ],
  "message": "Requêtes générées avec succès.",
  "processing_time": 0.35
}
```
## Exemples d'utilisation

### Exemple avec curl

Voici comment utiliser l'API avec curl pour convertir un texte en ensemble de croyances et exécuter une requête:

```bash
# Convertir un texte en ensemble de croyances
BELIEF_SET_ID=$(curl -s -X POST \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Si le ciel est nuageux, alors il va pleuvoir. Le ciel est nuageux.",
    "logic_type": "propositional",
    "options": {
      "include_explanation": true
    }
  }' \
  http://localhost:5000/api/logic/belief-set | jq -r '.belief_set.id')

echo "Ensemble de croyances créé avec ID: $BELIEF_SET_ID"

# Exécuter une requête sur l'ensemble de croyances
curl -s -X POST \
  -H "Content-Type: application/json" \
  -d "{
    \"belief_set_id\": \"$BELIEF_SET_ID\",
    \"query\": \"pluie\",
    \"logic_type\": \"propositional\",
    \"options\": {
      \"include_explanation\": true
    }
  }" \
  http://localhost:5000/api/logic/query | jq
```

### Exemple avec Python

Voici comment utiliser l'API avec Python:

```python
import requests
import json

# Configuration
API_BASE_URL = "http://localhost:5000/api/logic"
HEADERS = {
    "Content-Type": "application/json"
}

# Fonction pour convertir un texte en ensemble de croyances
def create_belief_set(text, logic_type="propositional", options=None):
    if options is None:
        options = {"include_explanation": True}
    
    payload = {
        "text": text,
        "logic_type": logic_type,
        "options": options
    }
    
    response = requests.post(
        f"{API_BASE_URL}/belief-set",
        headers=HEADERS,
        data=json.dumps(payload)
    )
    
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Erreur {response.status_code}: {response.text}")

# Fonction pour exécuter une requête
def execute_query(belief_set_id, query, logic_type="propositional", options=None):
    if options is None:
        options = {"include_explanation": True}
    
    payload = {
        "belief_set_id": belief_set_id,
        "query": query,
        "logic_type": logic_type,
        "options": options
    }
    
    response = requests.post(
        f"{API_BASE_URL}/query",
        headers=HEADERS,
        data=json.dumps(payload)
    )
    
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Erreur {response.status_code}: {response.text}")

# Exemple d'utilisation
def main():
    # Texte à analyser
    text = "Si le ciel est nuageux, alors il va pleuvoir. Le ciel est nuageux."
    
    # Créer un ensemble de croyances
    belief_set_response = create_belief_set(text)
    belief_set_id = belief_set_response["belief_set"]["id"]
    print(f"Ensemble de croyances créé avec ID: {belief_set_id}")
    print(f"Contenu: {belief_set_response['belief_set']['content']}")
    
    # Exécuter une requête
    query = "pluie"
    query_response = execute_query(belief_set_id, query)
    result = query_response["result"]["result"]
    explanation = query_response["result"]["explanation"]
    
    print(f"Résultat de la requête '{query}': {result}")
    print(f"Explication: {explanation}")

if __name__ == "__main__":
    main()
```

### Exemple avec JavaScript

Voici comment utiliser l'API avec JavaScript (Node.js):

```javascript
const axios = require('axios');

// Configuration
const API_BASE_URL = 'http://localhost:5000/api/logic';
const HEADERS = {
  'Content-Type': 'application/json'
};

// Fonction pour convertir un texte en ensemble de croyances
async function createBeliefSet(text, logicType = 'propositional', options = { include_explanation: true }) {
  const payload = {
    text,
    logic_type: logicType,
    options
  };
  
  try {
    const response = await axios.post(
      `${API_BASE_URL}/belief-set`,
      payload,
      { headers: HEADERS }
    );
    
    return response.data;
  } catch (error) {
    console.error('Erreur lors de la création de l\'ensemble de croyances:', error.response?.data || error.message);
    throw error;
  }
}

// Fonction pour exécuter une requête
async function executeQuery(beliefSetId, query, logicType = 'propositional', options = { include_explanation: true }) {
  const payload = {
    belief_set_id: beliefSetId,
    query,
    logic_type: logicType,
    options
  };
  
  try {
    const response = await axios.post(
      `${API_BASE_URL}/query`,
      payload,
      { headers: HEADERS }
    );
    
    return response.data;
  } catch (error) {
    console.error('Erreur lors de l\'exécution de la requête:', error.response?.data || error.message);
    throw error;
  }
}

// Exemple d'utilisation
async function main() {
  try {
    // Texte à analyser
    const text = 'Si le ciel est nuageux, alors il va pleuvoir. Le ciel est nuageux.';
    
    // Créer un ensemble de croyances
    const beliefSetResponse = await createBeliefSet(text);
    const beliefSetId = beliefSetResponse.belief_set.id;
    console.log(`Ensemble de croyances créé avec ID: ${beliefSetId}`);
    console.log(`Contenu: ${beliefSetResponse.belief_set.content}`);
    
    // Exécuter une requête
    const query = 'pluie';
    const queryResponse = await executeQuery(beliefSetId, query);
    const result = queryResponse.result.result;
    const explanation = queryResponse.result.explanation;
    
    console.log(`Résultat de la requête '${query}': ${result}`);
    console.log(`Explication: ${explanation}`);
  } catch (error) {
    console.error('Une erreur est survenue:', error);
  }
}

main();
```

## Gestion des erreurs

L'API utilise les codes de statut HTTP standards pour indiquer le succès ou l'échec d'une requête:

- **200 OK**: La requête a réussi.
- **400 Bad Request**: La requête est mal formée ou contient des paramètres invalides (par exemple, JSON malformé, champs manquants).
- **404 Not Found**: La ressource demandée (par exemple, un endpoint spécifique) n'existe pas.
- **422 Unprocessable Entity**: La requête est syntaxiquement correcte mais contient des erreurs sémantiques (par exemple, un type de logique non supporté, une requête logique invalide pour le type de logique donné).
- **500 Internal Server Error**: Une erreur inattendue s'est produite côté serveur.

En cas d'erreur, l'API renvoie un objet JSON avec les informations suivantes:

```json
{
  "error": "invalid_query",
  "message": "La requête contient une syntaxe invalide",
  "status_code": 422,
  "timestamp": "2025-05-27T17:40:12.345Z",
  "details": {
    "query": "pluie &&",
    "position": 7,
    "expected": "expression"
  }
}
```

## Bonnes pratiques

1. **Gestion du token d'authentification**:
   - Si des mécanismes de protection externes sont en place (pare-feu, API Gateway), suivez leurs recommandations pour la gestion des accès.

2. **Optimisation des performances**:
   - Réutilisez les ensembles de croyances pour plusieurs requêtes
   - Limitez le nombre de requêtes simultanées
   - Utilisez des timeouts appropriés pour les opérations longues

3. **Gestion des erreurs**:
   - Implémentez une gestion robuste des erreurs
   - Utilisez des retries avec backoff exponentiel pour les erreurs temporaires
   - Loggez les erreurs pour le débogage

4. **Validation côté client**:
   - Validez les entrées avant de les envoyer à l'API
   - Vérifiez la syntaxe des requêtes logiques
   - Assurez-vous que les types de logique sont supportés

## Limites et considérations

1. **Complexité des requêtes**:
   - Les requêtes très complexes peuvent entraîner des timeouts
   - La logique du premier ordre et la logique modale sont plus coûteuses en calcul
   - Limitez la taille des ensembles de croyances pour de meilleures performances

2. **Disponibilité et scalabilité**:
   - L'API peut avoir des limites de rate limiting
   - Implémentez des mécanismes de mise en cache pour réduire la charge
   - Prévoyez des stratégies de fallback en cas d'indisponibilité

3. **Sécurité**:
   - Ne transmettez pas de données sensibles dans les requêtes
   - Utilisez HTTPS pour toutes les communications
   - Vérifiez régulièrement les vulnérabilités potentielles

4. **Compatibilité**:
   - L'API peut évoluer, vérifiez régulièrement la documentation
   - Testez votre intégration après chaque mise à jour majeure
   - Utilisez des versions spécifiques de l'API si disponibles

## Ressources supplémentaires

- [Description technique de l'API Web](../composants/api_web.md)
- [Guide d'utilisation des agents logiques](utilisation_agents_logiques.md)
- [Exemples de logique propositionnelle](exemples_logique_propositionnelle.md)
- [Exemples de logique du premier ordre](exemples_logique_premier_ordre.md)
- [Exemples de logique modale](exemples_logique_modale.md)
- [Exemple d'intégration avec l'API Web](../../examples/logic_agents/api_integration_example.py)