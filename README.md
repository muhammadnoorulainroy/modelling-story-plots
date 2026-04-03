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
- compatibility bridges for the legacy example vocabulary (`alignment.ttl`)
- canonical Narn base graph (`narn.ttl`)
- canonical Narn plot graph (`narn-plot.ttl`)
- illustrative overlay instances (`narn-instances.ttl`)

The model separates the fictional layer (characters, places, events, plot events) from the real-world layer (book, author, editor, publisher).

## Provided Modules

`fo.ttl`, `meo.ttl`, and `rcc.ttl` were given as starting points. They were kept as the base vocabularies and minimally repaired where needed:

- `fo.ttl`: fixed broken `subPropertyOf` links on `isASonOf`/`isADaughterOf`, renamed the misspelled `isASibingOf` to `isASiblingOf`, added `Male`/`Female` disjointness.
- `meo.ttl`: fixed `owl:inverseOf` typos, added missing `Atan rdfs:subClassOf Eruhin, Mortal`, restored Peredhel restrictions.
- `rcc.ttl`: added symmetry, transitivity, inverses, irreflexivity, and partial pairwise disjointness to the 8 RCC8 relations.

## Loading

All `owl:imports` use local file references, so loading `narn-instances.ttl` in Protege resolves the rest of the project from the same folder. The instance layer has three parts: a canonical story-world base graph (`narn.ttl`), the supplied canonical plot graph (`narn-plot.ttl`), and an additive overlay (`narn-instances.ttl`) that enriches those same `https://purl.org/narn/` individuals with the final event, time, action, and plot structures.

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
10. `alignment.ttl`
11. `narn.ttl`
12. `narn-plot.ttl`
13. `narn-instances.ttl`

## Main Design Choices

**Place vs Region.** A `loc:Place` is a named story-world location. A `rcc:Region` is its spatial extent. `loc:occupiesRegion` links the two.

**Interior vs tangential containment.** `loc:interiorPartOf` maps to `rcc:ntpp` and `loc:tangentialPartOf` maps to `rcc:tpp` via separate property chains. This prevents edge-of-region places from getting the wrong RCC8 relation.

**Event vs PlotEvent.** An `evt:Event` is an in-world occurrence. A `plot:PlotEvent` is a narrative element. `plot:describesEvent` bridges the two.

**Action typing.** `act:Action = Event AND hasAgent some Thing`, so any event with an agent is automatically classified as an action. `act:UseItemAction` is also defined by equivalence. `act:AttackAction` is kept as a subclass to avoid classifying curses as attacks.

**Dual reality.** `plot:NarrativeWork`, `plot:Author`, `plot:Editor`, and `plot:Publisher` stay in the real world. `plot:Story`, `plot:StorySection`, and `plot:PlotEvent` stay in the fictional layer.

**Appellations instead of duplicate identities.** Túrin's and Niënor's alternate names are modeled as `soc:Appellation` individuals plus temporally scoped `soc:AppellationUse` situations. This preserves identity while still allowing queries about which name was used where and when.

**Temporalized relationship states.** Timeless binary social relations are kept where appropriate, but some narrative relations such as romantic attachment are also represented as explicit states (`soc:LoveState`) linked to an interval. This allows time-sensitive questions without reifying every social relation in the ontology.

**Temporalized service.** Service is now represented both as a binary relation (`soc:serves`) and as an explicit `soc:ServiceState` for time-sensitive questions such as who served a lord during a given phase of the story.

**Presence states for scene-level questions.** `evt:Presence` makes co-presence and place-at-time questions explicit. `evt:Residence` and `evt:Captivity` are modeled as presence-bearing states, and a property chain derives `soc:presentIn`.

**Identity knowledge and recognition.** Beyond `soc:AppellationUse`, the ontology now includes `soc:IdentityKnowledgeState` and `evt:Recognition` so it can answer who knew or recognized a character under which appellation, where, and during which interval.

**Marriage with differentiated roles.** `hasHusband`, `hasWife`, `hasOfficiant` with `rdfs:range fo:Male`/`fo:Female` and `propertyDisjointWith` between husband and wife.

**Cross-module property chains.** Birth events derive `isAChildOf`, death events derive `killed`, marriage events derive `isMarriedTo`, residence events derive `livedIn`.

**Faction and battle participation.** Battles can have collective combatants (`evt:Faction`) with explicit members. This supports questions about who fought on which side and which battle a character fought in.

**Reign-to-place bridge.** `evt:Reign` now uses a dedicated `evt:hasRuler` role, and a property chain derives `loc:rulesPlace`. This supports governance questions without asserting ruler/place links by hand.

**Derived section order.** `plot:nextSection` is inferred from `endsWith`, `nextPlotEvent`, and `startsWith` inverse.

**Transitive section and plot causality.** `plot:precedesSection` and `plot:causallyPrecedesPlotEvent` lift immediate adjacency and immediate causation into reusable transitive query properties.

**Narrative mention bridge.** `plot:mentionsParticipant`, `plot:mentionsPlace`, `plot:mentionsItem`, and `plot:mentionsAppellation` are inferred from section membership, plot-event descriptions, and event-level roles. This supports section-level literary questions without duplicating data in the plot layer.

**Causal plot bridge.** `plot:leadsTo` is derived when one plot event describes an action whose result is described by another plot event.

**Story/work-level bridge.** Story-level and book-level properties such as `plot:storyContainsEvent`, `plot:storyMentionsParticipant`, `plot:storyMentionsAppellation`, and `plot:narratesEvent` let queries move from real-world works to fictional events, characters, places, items, and names.

**OWL 2 DL compatibility.** Reflexive properties have no domain/range (to avoid universal classification). Pairwise disjointness is limited to simple properties. These tradeoffs are documented in the source files.

## Key Inferences

| Asserted pattern | Inferred fact |
|---|---|
| Birth has newborn + father | `fo:isAChildOf` |
| Birth has newborn + mother | `fo:isAChildOf` |
| Marriage has husband + wife | `fo:isMarriedTo` |
| Death has killer + deceased | `evt:killed` |
| Residence has resident + location | `soc:livedIn` |
| Presence has present entity + location | `soc:presentIn` |
| Journey has traveler + destination | `soc:travelledTo` |
| Journey has traveler + origin | `soc:travelledFrom` |
| AppellationUse names character + uses appellation | `soc:knownAs` |
| IdentityKnowledgeState has knower + known character | `soc:knowsIdentityOf` |
| Recognition has recognizer + recognized character | `soc:knowsIdentityOf` |
| LoveState has lover + beloved | `soc:loved` |
| Reign has ruler + location | `loc:rulesPlace` |
| Battle has combatant faction + faction member | `evt:involvesFactionMember` |
| faction membership + battle combatant | `evt:foughtInBattle` |
| `soc:serves` then `soc:enemyOf` | `soc:enemyOf` |
| `loc:interiorPartOf` + occupied regions | `rcc:ntpp` |
| `loc:tangentialPartOf` + occupied regions | `rcc:tpp` |
| `loc:adjacentTo` + occupied regions | `rcc:ec` |
| interval ordering via instants | Allen interval relations |
| event temporal extents + Allen relations | event-level temporal relations |
| section + plot event + described event role | `plot:mentionsParticipant` / `plot:mentionsPlace` / `plot:mentionsItem` / `plot:mentionsAppellation` |
| plot event + action result + inverse description | `plot:leadsTo` |
| story + section-level event bridge | `plot:storyContainsEvent` |
| story + section-level mention bridge | `plot:storyMentionsParticipant` / `plot:storyMentionsPlace` / `plot:storyMentionsItem` / `plot:storyMentionsAppellation` |
| narrative work + story bridge | `plot:narratesEvent` / `plot:narratesParticipant` / `plot:narratesPlace` / `plot:narratesItem` / `plot:narratesAppellation` |

## Competency Questions

These are questions the ontology can answer, either directly from assertions or through reasoner inferences.

**Family and social (fo, soc):**

- Who is parent of X? Who are X's children?
- Who is married to X?
- Is X a descendant of Y? (transitive ancestry across generations)
- Who is a sibling of X?
- Who fostered character X?
- Who are the friends/allies/enemies of X?
- Who serves whom? Who rules over whom?
- By what names was character X known, and during which interval?
- Who served whom during a specific interval?
- Who knew character X under which name, where, and during which interval?

**Places and space (place, rcc):**

- Is place P inside place Q? (via `partOf`, transitive)
- What is the capital of realm R?
- Who ruled place P? What place did ruler X govern?
- Are two places spatially disconnected? Adjacent? (via RCC8 on their regions)
- Can someone be in L1 and L2 at the same time? (if regions are `rcc:dc`, no)

**Time (time):**

- Did event E1 happen before event E2? (via Allen `beforeInterval` chain)
- Did event E1 happen during event E2? (via Allen `duringInterval`)
- What events overlap in time?
- Who was present at place P during interval T?

**Events and actions (event, action):**

- Who was involved in event E? In what role?
- Where did event E occur?
- What item was used in event E?
- Who killed whom? (inferred from Death events)
- Where did character X live? (inferred from Residence events)
- Where did character X travel to or from? (inferred from Journey events)
- Which side did character X fight for? Which battle did X fight in?
- Who was present at place P, and who shared that place during interval T?
- Who recognized or identified character X during event E?
- What are the consequences of action A?
- What events are sub-events of a war?

**Plot structure (plot):**

- What event starts/ends story S?
- What section does plot event PE belong to?
- What is the next section after section S? (inferred via chain)
- What sections come after section S? (transitively)
- Which characters, places, or items are mentioned in section S? (inferred via plot/event bridge)
- Which characters, places, items, or appellations are mentioned in the story as a whole?
- Which events, characters, places, items, or appellations are narrated by narrative work W?
- Which plot event causally leads to another plot event?
- Which plot events are downstream consequences of plot event P?
- Who authored, edited, and published the narrative work?

**Cross-module inferences:**

- From a Birth event, who is a child of whom?
- From a Marriage event, who is married to whom?
- If A serves B and B is enemy of C, is A enemy of C?

## CQ Coverage Matrix

This matrix distinguishes questions the current ontology answers well from questions that still require SPARQL joins or additional modelling. It is intentionally explicit about the limits imposed by OWL 2 DL and the Open World Assumption.

| Question family | Status | Current answer path / limitation |
|---|---|---|
| Parent, child, sibling, spouse, ancestor | Strong | Answerable directly or by DL inference via `fo:` axioms and event-to-family chains |
| Residence and travel history | Strong | `soc:livedIn`, `soc:travelledTo`, and `soc:travelledFrom` are inferred from Residence and Journey events |
| Event participation and role questions | Strong | Supported by `evt:hasParticipant` and role-specific subproperties such as `hasKiller`, `hasCaptive`, `hasTraveler` |
| Temporal order and containment of events | Strong | Derived through Allen interval relations and event-level chains such as `evt:beforeEvent` and `evt:duringEvent` |
| Spatial containment and RCC8 relations | Strong | Answerable via place mereology plus region-level RCC8 bridges |
| Plot order, plot containment, story start/end | Strong | Supported by `plot:nextPlotEvent`, `plot:nextSection`, `plot:containsEvent`, `plot:startsWithEvent`, and `plot:endsWithEvent` |
| Section-level mentions of characters, places, items, and appellations | Strong | Inferred with `plot:mentionsParticipant`, `plot:mentionsPlace`, `plot:mentionsItem`, and `plot:mentionsAppellation` |
| Alternate names and aliases | Strong | `soc:knownAs` is inferred from `soc:AppellationUse`; interval and place come from `soc:validDuring` and `soc:usedInPlace` |
| Romantic attachment during an interval | Good | Supported for modeled cases through `soc:LoveState`; not all social relations are temporalized |
| Service during a specific interval | Good | Supported through explicit `soc:ServiceState` individuals with `soc:hasServant`, `soc:hasLord`, and `soc:serviceHoldsDuring` |
| Action consequences and plot causality | Good | `act:resultsIn` supports event/action consequences; `plot:leadsTo` supports one-step narrative causality |
| Downstream plot consequences | Good | `plot:causallyPrecedesPlotEvent` supports transitive causal-upstream questions |
| Impossible co-location at the same time | Query-only | Requires SPARQL over temporal overlap plus RCC8 `dc`; not an OWL inconsistency under OWA |
| Which side someone fought on | Good | Battles now combine `evt:Faction`, `evt:hasMember`, and derived `evt:foughtInBattle` / `evt:involvesFactionMember` |
| Who was present at place P at time T | Good | Supported through explicit `evt:Presence` states plus presence-bearing `evt:Residence` and `evt:Captivity`, with `soc:presentIn` derived by property chain |
| Which alias was active during event E | Medium | Possible via interval joins, but not exposed by a dedicated event-appellation bridge |
| Which service, alliance, or enmity held during interval T | Medium | Service is temporalized through `soc:ServiceState`, but alliance and enmity are still atemporal binaries |
| Who ruled place P, or what place did ruler X govern | Good | Derived through `evt:Reign`, `evt:hasRuler`, and `loc:rulesPlace` |
| Which events/participants/places/items/appellations belong to the whole story or book | Good | Supported by `plot:storyContainsEvent`, `plot:storyMentions*`, and `plot:narrates*`, now including appellations |
| Who knew or recognized a character's identity, and under what name | Good | Supported through `soc:IdentityKnowledgeState`, `soc:knowsIdentityOf`, and `evt:Recognition`; explicit forgetting or loss of knowledge is still not modeled |

## Documentation

- [`docs/documentation.html`](docs/documentation.html) has design decisions, usage examples with Turtle snippets, competency questions, axiom summary, and AI tool usage.
- [`widoco-docs/index-en.html`](widoco-docs/index-en.html) is auto-generated from the ontology using WIDOCO. It lists every class, property, and individual with their `rdfs:label` and `rdfs:comment`. Includes a [WebVOWL visualization](widoco-docs/webvowl/index.html).
- The source `.ttl` files are the authoritative version of the ontology. Every class and property has `rdfs:label` and `rdfs:comment`.
