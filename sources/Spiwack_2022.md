---
category: reading
title: "Linearly Qualified Types: Generic inference for capabilities and uniqueness"
tags:
  - Computer
  - Science
  - Programming
  - Languages
citekey: Spiwack_2022
year: 2022
status: unread
itemType: journalArticle
url: http://arxiv.org/abs/2103.06127
dateread:
---

> [!Cite]
> Spiwack, Arnaud, Csongor Kiss, Jean-Philippe Bernardy, Nicolas Wu, and Richard Eisenberg. 2022. “Linearly Qualified Types: Generic Inference for Capabilities and Uniqueness.” _Proceedings of the ACM on Programming Languages_ 6 (ICFP): 137–64. [https://doi.org/10.1145/3547626](https://doi.org/10.1145/3547626).

>[!Synth]
>**Contribution**: 
>
>**Related**: 
>

>[!Data]
> **FirstAuthor**:: Spiwack, Arnaud  
> **Author**:: Kiss, Csongor  
> **Author**:: Bernardy, Jean-Philippe  
> **Author**:: Wu, Nicolas  
> **Author**:: Eisenberg, Richard  
~    
> **Title**: Linearly Qualified Types: Generic inference for capabilities and uniqueness  
> **Year**: 2022   
> **Citekey**: Spiwack_2022  
> **itemType**: journalArticle  
> **Journal**: *Proceedings of the ACM on Programming Languages*  
> **Volume**: 6  
> **Issue**: ICFP   
> **Pages**: 137-164  
> **DOI**: 10.1145/3547626    

> [!LINK] 
>**URL**: http://arxiv.org/abs/2103.06127  
>**Zotero URL**: [arXiv Fulltext PDF](zotero://select/library/items/QQDM2B7Y)  
>
>  Attachment: [arXiv Fulltext PDF](file:///home/jpyamamoto/Zotero/storage/QQDM2B7Y/Spiwack%20et%20al.%20-%202022%20-%20Linearly%20Qualified%20Types%20Generic%20inference%20for%20ca.pdf).



> [!Abstract]
>
> A linear parameter must be consumed exactly once in the body of its function. When declaring resources such as file handles and manually managed memory as linear arguments, a linear type system can verify that these resources are used safely. However, writing code with explicit linear arguments requires bureaucracy. This paper presents linear constraints, a front-end feature for linear typing that decreases the bureaucracy of working with linear types. Linear constraints are implicit linear arguments that are filled in automatically by the compiler. We present linear constraints as a qualified type system,together with an inference algorithm which extends GHC's existing constraint solver algorithm. Soundness of linear constraints is ensured by the fact that they desugar into Linear Haskell.
>.
> 
# Notes
>.


# Annotations
- Este artículo menciona el concepto de los **Linear Constraints**, en donde la linealidad se define al nivel de las operaciones, no sobre la estructura de datos. Esto puede incluirse como algo a revisar en trabajo futuro.

%% Import Date: 2024-06-17T13:29:35.651-06:00 %%
