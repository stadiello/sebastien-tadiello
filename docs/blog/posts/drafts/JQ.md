---
title: "Exemple JQ"
date: 2024-06-21
draft: true
---


# Trouver 10 contacts les plus récurrent

``` bash
zcat ff14b3ba.jsons.gz | jq 'select(.link)' -C | grep from | sort  | uniq -c |sort -r | head -10 > top10

zcat ff14b3ba.jsons.gz \
| jq 'select(.link)' -C \
| grep from \
| sort  \
| uniq -c \
|sort -r \
| head -10 > top10

# mieux 
zcat ff14b3ba.jsons.gz | jq -C '.from' | sort | uniq -c | sort -r

zcat ff14b3ba.jsons.gz | jq -C '.from' | sort | uniq -c | sort -r | head -10 > top10
```

[Link text](https://www.linkedin.com/in/sebastien-t-ababa2128/)

``` mermaid
graph LR
  A[Start] --> B{Error?};
  B -->|Yes| C[Hmm...];
  C --> D[Debug];
  D --> B;
  B ---->|No| E[Yay!];
```

<!-- <span style="background-color: #00ff00">surlignement vert</span> -->
<span style="background-color: #b90077">surlignement vert</span>
<span style="background-color: #06b90a">surlignement vert</span>
<span style="background-color: #056eb6">surlignement vert</span>