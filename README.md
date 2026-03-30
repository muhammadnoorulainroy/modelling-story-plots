# Story Plot Ontology

A modular OWL ontology for representing story plots, developed for the Knowledge Representation and Reasoning course (CPS2 M1, EMSE). The main use case is the *Narn i Chîn Húrin* by J.R.R. Tolkien.

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

## Teacher Modules

`fo.ttl`, `meo.ttl`, and `rcc.ttl` are the teacher-provided starting points used by the project. They were kept as the base vocabularies and minimally repaired/enriched where needed so the ontology remains loadable and semantically usable:

- `fo.ttl`: repaired broken child/sibling links and kept the misspelled `fo:isASibingOf` as a deprecated backward-compatibility alias.
- `meo.ttl`: repaired dead OWL predicates (`owl:equivalentClass`, `rdfs:subClassOf`, `owl:inverseOf`) and restored the intended ancestry axioms.
- `rcc.ttl`: keeps the teacher RCC8 vocabulary and adds the expected OWL characteristics, inverses, and property disjointness.

## Loading

All `owl:imports` are local file imports now, so loading [narn-instances.ttl](C:/Projects/Personal/CPS2/Semester%202/KGR/story_plot_ontology/narn-instances.ttl) in Protégé should resolve the rest of the project from the same folder.

If you prefer to load manually, use this order:

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

**Interior vs tangential containment.** The place module does not equate every `loc:partOf` relation with RCC8 `ntpp`. Instead it distinguishes `loc:interiorPartOf` and `loc:tangentialPartOf`, which bridge respectively to `rcc:ntpp` and `rcc:tpp`. This avoids false inferences such as forcing Anfauglith to be both `tpp` and `ntpp` of Beleriand.

**Event vs PlotEvent.** An `evt:Event` is an in-world occurrence. A `plot:PlotEvent` is a narrative element describing such an event. `plot:describesEvent` bridges the two.

**Action typing.** `act:Action ≡ evt:Event ⊓ ∃evt:hasAgent.Thing`, so any event with an agent is automatically classified as an action. `act:UseItemAction` is also defined from `evt:usesItem`, while `act:AttackAction` is kept as a conservative subclass to avoid classifying every patient-bearing action as an attack.

**Dual reality.** `plot:NarrativeWork`, `plot:Author`, `plot:Editor`, and `plot:Publisher` stay in the real world. `plot:Story`, `plot:StorySection`, and `plot:PlotEvent` stay in the fictional narrative layer.

**Derived section order.** `plot:nextSection` is inferred from `plot:endsWith`, `plot:nextPlotEvent`, and the inverse of `plot:startsWith`, so section adjacency follows from plot-event ordering instead of being duplicated manually.

## Key Inferences

The ontology uses property chains across modules:

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

The ontology supports these questions:

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

For the location-conflict question ("can someone be in L1 and L2 at the same time?"), the ontology provides the needed spatial and temporal vocabulary, but the answer is obtained by query patterns combining temporal overlap with RCC8 disconnection; it is not reduced to a single closed-world OWL inconsistency axiom.

## Documentation

[docs/documentation.html](docs/documentation.html) is the main human-readable documentation. The source `.ttl` files are the authoritative version of the ontology.
