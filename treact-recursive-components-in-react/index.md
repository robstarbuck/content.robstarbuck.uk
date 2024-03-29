---
title: Treact - Recursive Components in React 
date: 2022-04-30
---

React is ideally suited to the task of creating [trees](https://en.wikipedia.org/wiki/Tree_(data_structure)), being at its core, a tree of functions that call one another. In this demo we’ll look at creating a tree using a data set containing trees (of the Kingdom Plantae). So the theme is trees.

Here’s the [demo](https://robstarbuck.github.io/treact/) that we are working towards.

<iframe src="https://robstarbuck.github.io/treact/" title="Treact" style="aspect-ratio: 4 / 3"
    data-breakout-url="employboy-demo"
    data-breakout-class="full-page-iframe"
    data-breakout-button-color="#0d95c0"

>
</iframe>

## Flat vs Nested Data

As our subject is trees it’s their taxonomic rank that we’ll be exploring. There are many but we'll be using only a subset:

```
1. Kingdom
2. Order
3. Family
4. Genus
5. Species (or Common Name)
```

So an ash tree would come out as

```
1. Plantae
	2. Lamiales
		3. Oleaceae
			4. Fraxinus
				5. Fraxinus excelsior (Ash)
```

One common way to represent tree data is to nest the containing information. 

```json
[
  {
    "taxon": "Kingdom",
    "value": "Plantae",
    "children": [
      {
        "taxon": "Order",
        "value": "Lamiales",
        "children": [
          {
            "taxon": "Family",
            "value": "Oleaceae",
            "children": [
              {
                "taxon": "Genus",
                "value": "Fraxinus",
                "children": [
                  {
                    "taxon": "Species",
                    "value": "Fraxinus excelsior",
                    "children": []
                  }
                ]
              }
            ]
          }
        ]
      }
    ]
  }
]
```

Whilst this seems a natural fit it has many disadvantages.

- **1.** The shape of the data is hard to work with, finding a leaf node (a tree) requires recursion.
- **2.** Each datum has no idea of its context, eg a species only knows its genus by knowing its parent.
- **3.** The hierarchy is very rigid, if we only want to look at everything beneath families we still need to go through the root *Plantae*.

A more flexible approach is to keep our data flat. In this instance we already know the levels of our tree (the taxonomic ranks we’ll be using), this might not be true for other trees. A family tree for example could span any number of generations. Here however our levels are known to us in advance (those being Kingdom, Order, Family, Genus, Species)

As such the Ash tree can be easily represented:

```json
{
  "Kingdom": "Plantae",
  "Order": "Lamiales",
  "Family": "Oleaceae",
  "Genus": "Fraxinus",
  "Species": "Fraxinus excelsior"
}
```

Certainly much easier to grok from my point of view and I’ll demonstrate further advantages later on. With our data model agreed upon (well I’m happy with it), let’s render it out in React.

## A Basic Implementation

We can get a basic implementation working in 50 or so (readable) lines of Typescript.

```jsx

import "./App.css";
import { FC } from "react";
import allTrees from "./data.json";

// Types
type Tree = typeof allTrees[number];
type TaxonomicRank = keyof Tree;
type TaxonomyProps = { children: Array<Tree>; rank: TaxonomicRank }

// This is only a subset of - domain, kingdom, phylum, class, order, family, genus, species
const taxonomicRanks: Array<TaxonomicRank> = [
  "Kingdom",
  "Order",
  "Family",
  "Genus",
  "Species"
];

const renderTaxon = (taxon) => {

  // We are only passing the taxa of this rank to the next Taxonomy
  // Without this filter, the component will call itself indefinitely

  const childrenOfTaxon = children.filter(
    (t) => t[rank] === taxon[rank]
  );

  const subRank = taxonomicRanks.at(taxonomicRanks.indexOf(rank) + 1);

  return (
    <details open>
      <summary>
        {taxon[rank]}
      </summary>

      {/* Here the element now calls itself */}

      {subRank && <Taxonomy rank={subRank}>{childrenOfTaxon}</Taxonomy>}
    </details>
  );
}

// Our recursive taxonomy which calls itself where child taxonomies exist
const Taxonomy: FC<TaxonomyProps> = (props) => {
  const { children, rank } = props;

  // Find the unique taxa in our child taxonomies
  // EG unique taxa with a key of "Genus"
  const taxaInRank = children.reduce<Array<Tree>>((a, c) => {
    if (a.find((v) => v[rank] === c[rank])) {
      return a;
    }
    return [...a, c];
  }, []);

  return (
    <div>
      {/* Loop through the taxa in the rank */}
      {taxaInRank.map(renderTaxon)}
    </div>
  );
};

function App() {
  return (
    <div className="App">
      {/* Our initial call to our Taxonomy */}
      <Taxonomy rank={taxonomicRanks[0]}>{allTrees}</Taxonomy>
    </div>
  );
}

export default App;
```
Here’s the same code on [github](https://github.com/robstarbuck/treact/blob/basic/src/App.tsx). 

With minimal CSS we already have our tree rendering every level of our taxa up to the tree itself, our leaf node.

![Basic Screenshot](basic-screenshot.png)

## Improvements

Because we’ve kept our schema flat by abstracting the keys (in this case taxonomic ranks) we can modify which taxonomic ranks we view. Currently we are recursing through "Kingdom", "Order", "Family", "Genus", "Species”. If we only want to observe the “Species” though, this can easily be achieved by passing it in to our initial Taxonomy. 

```tsx
function App() {
  const rank = "Species";
  return (
    <div className="App">
      {/* Our initial call to our Taxonomy */}
      <Taxonomy rank={rank}>{allTrees}</Taxonomy>
    </div>
  );
}
```

With this in mind, it’s easy to see how `rank` may be stateful, allowing users to limit the ranks being viewed.

We can also opt to observe different keys of our trees, for instance Species is probably not all that useful to most people who know trees by their common names.

```json
{
    "Name": "Ash",
    "Species": "Fraxinus excelsior"
    // ...
}
```

Again, this is easily toggled by switching between two different taxa.

```tsx
const taxaByName: Array<TaxonomicRank> = [
  "Kingdom",
  "Order",
  "Family",
  "Genus",
  "Name",
];

const taxaBySpecies: Array<TaxonomicRank> = [
  "Kingdom",
  "Order",
  "Family",
  "Genus",
  "Species",
];
```

The complete source code for this project can be found in the [repo](https://github.com/robstarbuck/treact/).

## Summary

Even when everything in your repo is screaming “TREE!” keeping data flat can may prove more flexible. In this instance we knew our groups in advance, where this might not be the case extra keys might indicate relationships between nodes.

```json
[
    // ...
    {
        "id": "LETOTHEJUST",
        "fatherId": "THEOLDDUKE",
        "name": "Leto Attreides"
    },
    {
        "id": "MUADDIB",
        "fatherId": "LETOTHEJUST",
        "name": "Paul Attreides"
    },
    {
        "id": "STALIAOFTHEKNIFE",
        "fatherId": "LETOTHEJUST",
        "name": "Alia Attreides"
    }
]
```

Of course in an ordinary family tree there is more than one parent, I’ll skip over that.

Owing to React's nature Recursive Components are a good use case for the language and implementations that I’ve seen in Vue and Angular aren’t quite as comprehensive.

## Links

- [Source Code](https://github.com/robstarbuck/treact/)