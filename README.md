# Story Plot Ontology

A modular OWL ontology for representing story plots, developed for the Knowledge Representation and Reasoning course (CPS2 M1, EMSE). The use case is the *Narn i Chin Hurin* (Tale of the Children of Hurin) by J.R.R. Tolkien.

## Files

### Provided as starting points (included with fixes)

| File | Prefix | What it covers |
|------|--------|----------------|
| `fo.ttl` | `fo:` | Family relations (parent, child, sibling, spouse, cousin) |
| `meo.ttl` | `meo:` | Middle-earth races and peoples (Elves, Men, Dwarves, etc.) |

### New modules

| File | Prefix | What it covers |
|------|--------|----------------|
| `rcc.ttl` | `rcc:` | RCC8 spatial relations with OWL axioms |
| `soc.ttl` | `soc:` | Social relations (friendship, enmity, alliance, service, fostering) |
| `place.ttl` | `loc:` | Place categories and spatial properties |
| `time.ttl` | `tm:` | Instants, intervals, Allen's 13 relations |
| `event.ttl` | `evt:` | Event hierarchy, participant roles, cardinality constraints |
| `action.ttl` | `act:` | Actions as events with intentional agents |
| `plot.ttl` | `plot:` | Narrative structure with dual-reality separation |

### Instance data

| File | What it is |
|------|-----------|
| `narn-instances.ttl` | Example data from the Narn |

## How to load in Protege

Load `narn-instances.ttl` and let `owl:imports` pull everything in.

## Main Design Choices

**Place vs Region.** A `loc:Place` is a named story-world location. A `rcc:Region` is its spatial extent. `loc:occupiesRegion` links the two.

**Event vs PlotEvent.** An `evt:Event` is an in-world occurrence. A `plot:PlotEvent` is a narrative element describing such an event. `plot:describesEvent` bridges the two.

**Dual reality.** `plot:NarrativeWork`, `plot:Author`, `plot:Editor`, and `plot:Publisher` stay in the real world. `plot:Story`, `plot:StorySection`, and `plot:PlotEvent` stay in the fictional narrative layer.
