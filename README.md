# Story Plot Ontology

A modular OWL ontology for representing story plots, developed for the Knowledge Representation and Reasoning course (CPS2 M1, EMSE). The main use case is the *Narn i Chin Hurin* by J.R.R. Tolkien.

## Scope

The project is split into reusable modules for:

- characters and peoples (`meo.ttl`)
- family relations (`fo.ttl`)
- social relations (`soc.ttl`)
- places and spatial containment (`place.ttl`)
- RCC8 spatial relations (`rcc.ttl`)
- time and Allen interval relations (`time.ttl`)
- events (`event.ttl`)
- actions (`action.ttl`)
- plot structure and dual reality (`plot.ttl`)
- illustrative instances (`narn-instances.ttl`)

The model separates the fictional layer (characters, places, events, plot events) from the real-world layer (book, author, editor, publisher).

## Provided Modules

`fo.ttl`, `meo.ttl`, and `rcc.ttl` were given as starting points. They were kept as the base vocabularies and minimally repaired where needed:

- `fo.ttl`: fixed broken `subPropertyOf` links on `isASonOf`/`isADaughterOf`, renamed the misspelled `isASibingOf` to `isASiblingOf`, added `Male`/`Female` disjointness.
- `meo.ttl`: fixed `owl:inverseOf` typos, added missing `Atan rdfs:subClassOf Eruhin, Mortal`, restored Peredhel restrictions.
- `rcc.ttl`: added symmetry, transitivity, inverses, irreflexivity, and partial pairwise disjointness to the 8 RCC8 relations.

## Loading

All `owl:imports` use local file references, so loading `narn-instances.ttl` in Protege resolves the rest of the project from the same folder.

Manual load order:

1. `fo.ttl`
2. `meo.ttl`
3. `rcc.ttl`
4. `place.ttl`
5. `time.ttl`
6. `soc.ttl`
7. `event.ttl`
8. `action.ttl`
9. `plot.ttl`
10. `narn-instances.ttl`

## Main Design Choices

**Place vs Region.** A `loc:Place` is a named story-world location. A `rcc:Region` is its spatial extent. `loc:occupiesRegion` links the two.

**Interior vs tangential containment.** `loc:interiorPartOf` maps to `rcc:ntpp` and `loc:tangentialPartOf` maps to `rcc:tpp` via separate property chains. This prevents edge-of-region places from getting the wrong RCC8 relation.

**Event vs PlotEvent.** An `evt:Event` is an in-world occurrence. A `plot:PlotEvent` is a narrative element. `plot:describesEvent` bridges the two.

**Action typing.** `act:Action = Event AND hasAgent some Thing`, so any event with an agent is automatically classified as an action. `act:UseItemAction` is also defined by equivalence. `act:AttackAction` is kept as a subclass to avoid classifying curses as attacks.

**Dual reality.** `plot:NarrativeWork`, `plot:Author`, `plot:Editor`, and `plot:Publisher` stay in the real world. `plot:Story`, `plot:StorySection`, and `plot:PlotEvent` stay in the fictional layer.

**Marriage with differentiated roles.** `hasHusband`, `hasWife`, `hasOfficiant` with `rdfs:range fo:Male`/`fo:Female` and `propertyDisjointWith` between husband and wife.

**Cross-module property chains.** Birth events derive `isAChildOf`, death events derive `killed`, marriage events derive `isMarriedTo`, residence events derive `livedIn`.

**Derived section order.** `plot:nextSection` is inferred from `endsWith`, `nextPlotEvent`, and `startsWith` inverse.

**OWL 2 DL compatibility.** Reflexive properties have no domain/range (to avoid universal classification). Pairwise disjointness is limited to simple properties. These tradeoffs are documented in the source files.

## Key Inferences

| Asserted pattern | Inferred fact |
|---|---|
| Birth has newborn + father | `fo:isAChildOf` |
| Birth has newborn + mother | `fo:isAChildOf` |
| Marriage has husband + wife | `fo:isMarriedTo` |
| Death has killer + deceased | `evt:killed` |
| Residence has resident + location | `soc:livedIn` |
| `soc:serves` then `soc:enemyOf` | `soc:enemyOf` |
| `loc:interiorPartOf` + occupied regions | `rcc:ntpp` |
| `loc:tangentialPartOf` + occupied regions | `rcc:tpp` |
| `loc:adjacentTo` + occupied regions | `rcc:ec` |
| interval ordering via instants | Allen interval relations |
| event temporal extents + Allen relations | event-level temporal relations |

## Competency Questions

1. Who is parent of X?
2. Who is married to X?
3. Where did character X live or travel?
4. Who was involved in event E?
5. Did event E1 happen before event E2?
6. Where did event E occur?
7. What event ends story S?
8. What item was used in event E or action A?
9. Who killed whom?
10. Who authored, edited, and published the narrative work?

For the location-conflict question ("can someone be in L1 and L2 at the same time?"), the ontology provides the spatial and temporal vocabulary, but the answer comes from query patterns combining temporal overlap with RCC8 disconnection.

## Documentation

- `docs/documentation.html` has design decisions, usage examples with Turtle snippets, competency questions, axiom summary, and AI tool usage.
- `widoco-docs/index-en.html` is auto-generated from the ontology using WIDOCO. It lists every class, property, and individual with their `rdfs:label` and `rdfs:comment`. Includes a WebVOWL visualization at `widoco-docs/webvowl/index.html`.
- The source `.ttl` files are the authoritative version of the ontology. Every class and property has `rdfs:label` and `rdfs:comment`.
