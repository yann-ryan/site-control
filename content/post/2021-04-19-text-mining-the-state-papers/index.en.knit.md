---
title: Text Mining the State Papers
author: R package build
date: '2021-04-19'
slug: text-mining-the-state-papers
categories: []
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2021-04-24T14:21:08+02:00'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


### Calendars of State Papers and Networking Archives

Most of the [quantitative research](https://networkingarchives.github.io/blog/2021/04/15/my-network-analysis-workflow/) on the [Networking Archives](https://networkingarchives.org/) project has been using the *metadata* from the digitised correspondence of [State Papers Online](https://www.gale.com/primary-sources/state-papers-online). Metadata in this sense means everything except the content of the letters: including author names, recipient names, date, place of sending and so on, in the research of seventeenth-century intelligencing. Gale State Papers Online brings together a number of historical primary sources, not only the manuscript images from the State Papers, but also full text versions of the 'Calendars of State Papers', a set of printed finding aids mostly produced in the nineteenth century. These printed summaries represent another huge store of data available to us which we also use in the analysis of the data.  

The data the project is working with has a complicated history. A typical piece of correspondence now in the State Papers Online data started its life as part of the working papers of a [Secretary of State](https://en.wikipedia.org/wiki/Secretary_of_State_(England)). Secretaries were *supposed* to transfer all the documents relating to the state to what was then called the State Paper Office. This office was established in 1610 and intended as a central place for the Monarch's private papers, [kept and managed by the secretaries](https://www.jstor.org/stable/43737460).  

In practice it was rarely this simple. Secretaries' papers were fraught with complex issues of contested ownership. Today many governments assume that any correspondence by a sitting Head of State or a politician in the course of their working day is public property (with lots of limits for national security, of course). Historically the rules have been much more ambiguous. Is a letter from a diplomat containing personal news, sent to a Secretary of State, personal or private?

Even when it was clear that a document was in some way 'public', politicians and their heirs often resisted the transfer of their libraries to the state, for a variety of reasons. Private documents could expose secrets they would rather stayed buried but also, and more usually, these libraries were valuable assets to be inherited. [Elizabeth Raleigh](https://en.wikipedia.org/wiki/Elizabeth_Raleigh), wife of Walter, lobbied to stop the transfer his personal library (and scientific equipment) to the paper office after his execution in 1618. It was, she argued, her son's only inheritance and an important asset for his education. Sometimes these transfers came long after the death of the secretary in question. In the eighteenth century the Conway Papers were 'found', [by the future Prime Minister Horace Walpole](https://oxford.universitypressscholarship.com/view/10.1093/acprof:oso/9780199679133.001.0001/acprof-9780199679133-chapter-7), and split between the British Library and the newly-established State Paper Office. The Cecil Papers, about 30,000 documents, mostly the working papers of [Lord Burghley](https://en.wikipedia.org/wiki/William_Cecil,_1st_Baron_Burghley) and his son, Robert Cecil, both secretaries under sucessive monarchs, were archived at Hatfield House, where they still remain today, though they have actually made their way to State Papers Online, as we'll see below.

What the Paper Office did manage to get hold of was sorted into various categories and re-arranged over time, had to survive fires, mould, rats, [cooking accidents](https://oxford.universitypressscholarship.com/view/10.1093/acprof:oso/9780199679133.001.0001/acprof-9780199679133-chapter-7), and so forth, and generally sat unloved and mostly unseen on dusty shelves, though historians (such as John Evelyn), did request and get permission to borrow bundles of documents and take them home. It wasn't until the 19th century when efforts were made to make the records more accessible. First, the Public Records Office was established (in 1854 but based on an earlier archival amalgamation in 1838), which moved the records from the Paper Office to a purpose-built archive in Chancery Lane, and made them more accessible to the public.

#### Format of the Calendars

Sort of parallel to this, efforts were made to produce printed summaries of the documents in the archive, in order to make them easier for historians to use. These were large printed volumes, containing descriptions of all of the documents found in the State Papers, organized mostly by reign. They were produced throughout the century, many edited by women, such as [Mary Ann Everett Green](https://doi.org/10.1093/ref:odnb/11395), and her niece Sophia Crawford Lomas. History owes these women a huge debt of favour.


<div class="figure" style="text-align: center">
<img src="Screenshot 2021-04-12 at 10.43.37.png" alt="Title page of one volume of the Calendar of State Papers Domestic for the reigns of Elizabeth and James I" width="415" />
<p class="caption">(\#fig:unnamed-chunk-1)Title page of one volume of the Calendar of State Papers Domestic for the reigns of Elizabeth and James I</p>
</div>


Digitised copies of these calendars are mostly out of copyright and can be found on Google Books and [archive.org](https://archive.org/details/calendarofstatep05grea/mode/2up). Below is a fairly typical entry. It has the date of the document in the left-hand margin, then an identifier to help find the original document, then usually information on the sender and recipient (and sometimes the place of origin). This is followed by a brief description or summary of the contents, and in this case, followed by noting the number of pages of the original manuscript, and the language if not English.








































