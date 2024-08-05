---
title: "ChatBot Memory"
date: 2024-08-05
author: "Sebastien Tadiello"
description: >
    Setup a memory to your chatbot
categories: ["LLM", "Python", "Chatbot","Artificial Intelligence","Mathematics","NLP","Machine Learning","Memory"]
---

# ChatBot Memory

Nous présentons ici une approche pour la gestion de la mémoire à court terme dans les chatbots, en utilisant une combinaison de techniques de stockage et de résumé automatique pour optimiser le contexte conversationnel. La méthode introduite repose sur une structure de mémoire dynamique qui limite la taille des données tout en préservant les informations essentielles à travers des résumés intelligents. Cette approche permet non seulement d'améliorer la fluidité des interactions mais aussi d'assurer une continuité contextuelle lors de longues sessions de dialogue. En outre, l'utilisation de techniques asynchrones garantit que les opérations de gestion de la mémoire n'interfèrent pas avec la réactivité du chatbot.

# Modélisation Mathématique de la Gestion des Conversations

Dans cette section, nous formalisons mathématiquement la gestion de la mémoire de conversation dans le chatbot. La mémoire est structurée comme une liste de paires représentant les échanges entre l'utilisateur et le bot.

## Structure de la Mémoire de Conversation

La mémoire de conversation peut être définie comme une liste ordonnée de paires $(u_i, d_i)$, où $u_i$ représente l'entrée utilisateur et $d_i$ la réponse du bot pour le $i$-ième échange. Cette liste est notée $\mathcal{C}$ :

$$
\mathcal{C} = [(u_1, d_1), (u_2, d_2), \ldots, (u_n, d_n)]
$$

où $n$ est le nombre total d'échanges dans l'historique actuel.

## Mise à Jour de la Mémoire

Lorsqu'un nouvel échange se produit, une nouvelle paire $(u_{n+1}, d_{n+1})$ est ajoutée à la mémoire. Si la taille de $\mathcal{C}$ dépasse une limite maximale prédéfinie $M_{\text{max}}$, l'échange le plus ancien est retiré :

$$
\mathcal{C} = 
\begin{cases} 
\mathcal{C} \cup \{(u_{n+1}, d_{n+1})\}, & \text{si } |\mathcal{C}| < M_{\text{max}} \\
(\mathcal{C} \setminus \{(u_1, d_1)\}) \cup \{(u_{n+1}, d_{n+1})\}, & \text{si } |\mathcal{C}| = M_{\text{max}}
\end{cases}
$$

## Comptage des Mots

Pour gérer l'espace de mémoire et décider quand la compression est nécessaire, nous calculons le nombre total de mots $W(\mathcal{C})$ dans la mémoire :

$$
W(\mathcal{C}) = \sum_{(u_i, d_i) \in \mathcal{C}} (|u_i| + |d_i|)
$$

où $|u_i|$ et $|d_i|$ sont respectivement le nombre de mots dans $u_i$ et $d_i$.

## Compression de la Mémoire

Lorsque $W(\mathcal{C})$ dépasse un seuil $W_{\text{max}}$, la mémoire est compressée pour maintenir la pertinence du contexte. Cette compression est réalisée par un modèle de résumé $\mathcal{S}$, tel que BART :

$$
\mathcal{C}_{\text{compressed}} = \mathcal{S}(\mathcal{C})
$$

où $\mathcal{C}_{\text{compressed}}$ est la version résumée de la mémoire, réduisant le nombre total de mots tout en préservant l'essence des interactions passées.

## Intégration dans le Modèle de Langage

Le modèle de langage utilise le contexte compressé pour générer des réponses pertinentes. Le prompt $P$ utilisé par le modèle est construit comme suit :

$$
P = f(\mathcal{C}_{\text{compressed}}, \text{contexte})
$$

où $\text{contexte}$ est le contexte supplémentaire récupéré à partir d'un pipeline RAG, et $f$ est une fonction de concaténation qui prépare le texte pour le modèle.

Cette approche assure que le chatbot dispose toujours d'un contexte conversationnel à jour, permettant des interactions plus naturelles et engageantes avec l'utilisateur.

# Implémentation du Code pour la Gestion de la Mémoire d'un Chatbot

Dans cette section, nous allons examiner un exemple de code en Python qui illustre la gestion de la mémoire dans un chatbot. Le code utilise PyTorch et les transformers de Hugging Face pour gérer et compresser l'historique des conversations.

## Préparation de l'Environnement

Nous commençons par vérifier si un GPU est disponible, ce qui permet d'accélérer le traitement si nécessaire.

``` python
import torch
from transformers import pipeline
import logging

if torch.cuda.is_available():
    device: int = 0
else:
    device: int = -1

MAX_MEMORY_SIZE: int = 2000

```

## Définition de la Classe ChatbotMemory

La classe ChatbotMemory gère l'historique des conversations et effectue des opérations de mise à jour et de compression. Donc à chaques fois que update_memory est appelé, le texte en mémoire est compté et traité au besoin.

```python
class ChatbotMemory:
    def __init__(self, conv: list = []):
        self.conversation_history = conv

    def update_memory(self, user_input: str, bot_response: str) -> None:
        self.conversation_history.append(f"'user': {user_input}, 'bot': {bot_response}")

        if memory_counter(self.conversation_history) > 1000:
            self.conversation_history = compressed_memory(self.conversation_history)
            logging.info("Mémoire compressée.")

        if len(self.conversation_history) > MAX_MEMORY_SIZE:
            self.conversation_history.pop(0)
            logging.info("Mémoire réduite.")
        return 0

    def get_memory(self):
        return self.conversation_history
```

## Compression et Comptage de la Mémoire

La fonction _get_compressed_memory utilise le modèle BART pour résumer l'historique des conversations.

```python
def _get_compressed_memory(sentence: str) -> str:
    summarizer = pipeline("summarization", model="facebook/bart-large-cnn", device=device)
    summary = summarizer(sentence, max_length=50, min_length=5, do_sample=False)
    return summary[0]['summary_text']
```

La fonction compressed_memory applique la fonction _get_compressed_memory à chaque segment de l'historique des conversations. Pour cela nous optimisons la procédure en effectuant un traitement par Batch. Cette méthode est dissocié de la fonction _get_compressed_memory de manière à pouvoir introduire de nouvelles méthodes de compression.

```python
def compressed_memory(conv_hist: list) -> list:
    return [_get_compressed_memory(' '.join(conv_hist[i:i+5])) for i in range(0, len(conv_hist), 5)]
```

La fonction memory_counter compte le nombre total de mots dans l'historique. (Note qu'il pourrais être interessant de réaliser cette étape avec des tokens plutot que des mots.)

```python
def memory_counter(conv_hist: list) -> int:
    st = ''.join(conv_hist)
    return len(st.split())
```

## Conclusion

Ce code établit un cadre efficace pour la gestion de la mémoire dans un chatbot, en utilisant des techniques de compression pour maintenir un contexte pertinent et en améliorant la performance globale du système. L'utilisation de modèles de résumé comme BART assure que même lorsque la mémoire est compressée, le contexte essentiel est préservé.

[Code sur GitHub](https://github.com/sebDtSci/ShortTerm-memory)