---
category: reading
title: Linear Types can Change the World!
tags: 
citekey: Wadler_1990
year: 1990
status: unread
itemType: conferencePaper
url: https://www.semanticscholar.org/paper/Linear-Types-can-Change-the-World!-Wadler/24c850390fba27fc6f3241cb34ce7bc6f3765627
dateread:
---

> [!Cite]
> Wadler, P. 1990. “Linear Types Can Change the World!” In . [https://www.semanticscholar.org/paper/Linear-Types-can-Change-the-World!-Wadler/24c850390fba27fc6f3241cb34ce7bc6f3765627](https://www.semanticscholar.org/paper/Linear-Types-can-Change-the-World!-Wadler/24c850390fba27fc6f3241cb34ce7bc6f3765627).

>[!Synth]
>**Contribution**: 
>
>**Related**: 
>

>[!Data]
> **FirstAuthor**:: Wadler, P.  
~    
> **Title**: Linear Types can Change the World!  
> **Year**: 1990   
> **Citekey**: Wadler_1990  
> **itemType**: conferencePaper    

> [!LINK] 
>**URL**: https://www.semanticscholar.org/paper/Linear-Types-can-Change-the-World!-Wadler/24c850390fba27fc6f3241cb34ce7bc6f3765627  
>**Zotero URL**: [Wadler - 1990 - Linear Types can Change the World!.pdf](zotero://select/library/items/GVYUILD7)  
>
>  Attachment: [Wadler - 1990 - Linear Types can Change the World!.pdf](file:///home/jpyamamoto/Zotero/storage/GVYUILD7/Wadler%20-%201990%20-%20Linear%20Types%20can%20Change%20the%20World!.pdf).



> [!Abstract]
>
> The linear logic of J.-Y. Girard suggests a new type system for functional languages, one which supports operations that \change the world". Values belonging to a linear type must be used exactly once: like the world, they cannot be duplicated or destroyed. Such values require no reference counting or garbage collection , and safely admit destructive array update. Linear types extend Schmidt's notion of single threading; provide an alternative to Hudak and Bloss' update analysis; and ooer a practical complement to Lafont and Holmstrr om's elegant linear languages. An old canard against functional languages is that they cannot change the world: they do not \naturally" cope with changes of state, such as altering a location in memory, changing a pixel on a display, or sensing when a key is pressed. As a prototypical example of this, consider the world as an array. An array (of type Arr) is a mapping from indices (of type Ix) to values (of type Val). For instance, the world might be a mapping of variable names to values, or le names to contents. At any time, we can do one of two things to the world: nd the value associated with an index, or update an index to be associated with a new value. Of course it is possible to model this functionally; we just use the two operations A program that interacts with the world might have the form main : Args ! Arr ! Arr; where the rst parameter is the list of arguments that make up the command line, the second parameter is the old world, and the result is the new world. An example of a program is main les a = update \stdout" (concat lookup i a j i les]) a:
>.
> 
# Notes
>.


# Annotations%% begin annotations %%


%% end annotations %%

%% Import Date: 2024-04-12T09:39:10.055-06:00 %%
