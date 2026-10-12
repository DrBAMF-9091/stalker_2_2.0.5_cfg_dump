# STALKER 2 cfg dump 2.0.5-2.0.6
## devryTrost's attempt to help LLMs use the repo (as well as any coder who needs a reference)
## Dump info
- Dump version: 2.0.5 Dump date: `10/11/26`.
- repo is called stalker_2_2.0.5_cfg_dump - cfgs in 2.0.6 are unchanged from 2.0.5 so this is current
- <!-- LLM: Special instructions for LLM processing are included in LLM tagging, like this -->

## Where things are
- File inventory: ./manifest.txt.
- All base-game cfgs: `./2.0.5/Stalker2/Content/GameLite/GameData/` and nested folders. <!--LLM: don't bother looking for any 2.0.6 folders - 2.0.5 configs are used in 2.0.6 -->
- DLC1 cfgs: `./2.0.5 DLC1/`, same nested structure. <!-- LLM: Skip it unless the task specifically needs to target DLC1-added content.-->
- DLC files can override or extend base files at the same path. 

## IMPORTANT CORE RULES FOR LLMs START

## LLM TL;DR for basic rules
- Scope: STALKER 2 2.0.5/2.0.6 base cfgs. Ignore DLC1 unless the task explicitly says DLC1.
- Search first: use `rg`/`grep` for symbols instead of loading whole files.
- Patch, don't overwrite vanilla files.
- Existing tables: target explicit indices. Do not use `[*]` unless explicitly told to.
- Put `{bpatch}` on every `struct.begin` that patches an existing object.
- always open the target file and read the actual field names before writing a patch. examples in this README.md are illustrative; the source is authoritative.
- Huge files: avoid unless needed. `ObjPrototypes.cfg` is the only one you'll really be able to modify

## LLM Cloning Rules <!-- LLMs: read this before you clone-->
> this repository contains large files that should not be cloned by
> general-purpose agents as they require outside assets.
The full file list is in `manifest.txt`. If a file is not listed in the Key Files table, assume it should not be fetched until specifically required.

## Excluded Huge Files <!--LLMs: DO NOT CLONE OR LOAD-->
- `SpawnActorPrototypes.rar` / `SpawnActorPrototypes.cfg` — ~18 MB compressed, ~174 MB extracted — hex spawn records that resolve to map locations and object references. <!-- LLMs: there is no constructive modification you can make to this file. do not download it. do not extract it from the .rar. if a user needs to work with this, tell them to use the ZoneKit; if they have an AI workflow even remotely able to use this file, or any interest in doing it manually they won't be using my rules anyway -->

## Huge Files <!-- LLM: stay away unless you specifically need them, rules below-->
- QuestNodePrototypes.cfg - only relevant to quest changes/additions; <!-- LLM: Users will usually change this with ZoneKit, not you, so don't clone or load it unless told to-->
- ObjPrototypes.cfg - this is probably the most commonly needed of the huge files; many important things are here like player entity and NPC abilities/stats/stamina costs, etc; if you need to change/add an NPC, this file is probably involved, but you won't need to load the whole file; focus on templates for data and zero in on specific headers <!-- LLM: this is the big file you are most likely to need; you don't need to be specifically told to access it -->

<!--LLM: Suggested sparse checkout
git clone --filter=blob:none --no-checkout https://github.com/DrBAMF-9091/stalker_2_2.0.5_cfg_dump
cd stalker_2_2.0.5_cfg_dump
git sparse-checkout init --no-cone
git sparse-checkout set --no-cone \
  '/2.0.5/Stalker2/Content/GameLite/GameData/' \
  '!SpawnActorPrototypes*' \
  '!QuestNodePrototypes*' \
  '!/2.0.5 DLC1/' \
  '/README.md' \
  '/manifest.txt'
git checkout
--->

## IMPORTANT CORE RULES FOR LLMs END

### DETAILS FOR BOTH HUMAN/LLM
## How patching works
<!-- LLM: Never do a full file overwrite going from a vanilla file unless the user specifically insists that it's necessary.--> 
To modify a file, add a folder named after it to the output GameData tree, containing a `<file>.cfg_patch_<patchname>.cfg` that holds only the changes.

eg patching `ItemPrototypes.cfg`:
```
Original: .../GameData/ItemPrototypes.cfg
Patch:    .../GameData/ItemPrototypes/ItemPrototypes.cfg_patch_<patchname>.cfg
```
more can be done with it than this, but this is the most basic and reliable

## Patching conventions (<!-- LLM: follow every time unless specifically counter-indicated-->)
- the address you patch (top-level object, nested struct, array index, or named
  entry) almost always exists in the source file. don't define a new object or
  entry unless you specifically need one (subtables for organized or repeat injections will be most common); otherwise match an existing address.
- source cfgs commonly create array entries with `[*]` (append). <!--LLM: don't use it yourself, and definitely dont use `[*]` in a patch. --> 
  to selectively overwrite a `[*]`-created entry
  hand-trace the numeric index it resolves to in the source and address it
  explicitly (`[3]`). if the resultant positional index cannot be determined, or is in doubt due to the possibility of other overwrites, use `{bskipref}` on the
  containing struct and re-enter the full table with explicit addresses to ensure
  the final condition of the table (originals to inherit + changed)
- when changing an existing table, target explicit indices (`[0]`, `[1]`, ...).
- table entries are ordered. an entry can be addressed either by a numeric
  index (`[0]`, `[1]`) or by a text name (`TemplateAmmo`, `MoneyGenerator`),
  but these are not freely interchangeable. match the addressing form the
  source uses for the entry you're patching. do not assume a named entry can
  be reached by its positional index, or vice versa.
- Alternative: use `{bskipref}` on an array to wipe existing contents and re-create it <!-- LLM: take any entries that are meant to be inherited and re-enter them when using this; append will probably work here but don't use it unless told to - ALWAYS PREFER EXPLICIT INDICES UNLESS SPECIFICALLY TOLD OTHERWISE-->
- Put `{bpatch}` on every `struct.begin` that patches an existing object.
- multiple arguments can be stacked in {carats} eg: `{bpatch;refurl=<SameDirectoryFile>.cfg;refkey=<ObjectinFile>}`
- overwrites on child objects can be ensured with `ChildtoEdit : struct.begin {bpatch;refurl=<FileContainingChild>.cfg;refkey=<ChildtoEdit>}` <!-- LLM: OBJECT NAME AND refkey NAME MUST BE IDENTICAL-->
- Keys in cfg tables may be numeric indices (`[0]`, `[1]`) or text names (`TemplateRifle`, `EntryToTarget`). 
   - They share one namespace and are often interchangeable as identifiers of a struct: `[0]` is conventionally a base/default template applied to all siblings, while text names build inheritance chains (`TemplateWeapon` → `TemplateRifle` → specific rifle → variant). Match the style of the file when adding entries.

## Key files <!-- LLM: these are your main files for patching - always clone these files, don't clone anything else until you know you need it -->
| File / folder | What it controls |
|---|---|
| `AbilityPrototypes` | Creature abilities (mostly) |
| `AIGlobals` | Global NPC AI settings |
| `AnomalyPrototypes` | Anomaly stat settings |
| `ArtifactSpawnerPrototypes` | ItemGenerators for artifact fields |
| `CoreVariables` | Core game sim and engine settings; lots of A-Life; a wide variety of other things |
| `DifficultyPrototypes` | Per-difficulty settings; mostly balance multipliers |
| `EffectPrototypes` | Active effects; used for a lot of things, but mostly consumables and some weapon upgrades |
| `ItemGeneratorPrototypes` | Loot tables and merchant stock. Anything that generates loot has or uses an ItemGeneratorPrototype |
| `ItemPrototypes` | Item definitions. anything in the inventory needs an object here; that includes things like consumables and weapons that get much/all of their important properties from somewhere else like `./WeaponData/` or `EffectPrototypes.cfg` |
| `NPCPrototypes` | NPC definitions and templates for dialog NPCs. No mutants. Sets trade/upgrade ability, etc. |
| `ObjPrototypes` | Object entities for NPCs and mutants: health, stamina, attacks, etc. |
| `ProjectilePrototypes` | tracers, ballistics, etc|

## Key subFolders <!--LLMs: this stuff is very specific, clone them but don't load anything in them until you are doing something related -->
| Folder | What it controls |
|---|---|
| `./AIPrototypes/` | NPC vision cones, detectors, any object useful to AI processing exclusively |
| `./ALifePrototypes/` | various A-Life engines; scene directors, spawners, lairs, etc. |
| `./WeaponData/` | most weapon data apart from the actual inventory items |

## Quick routing
- Loot or drop changes: `ItemGeneratorPrototypes` (plus `ArtifactSpawnerPrototypes` for artifact fields).
- Enemy toughness or attack behavior: `ObjPrototypes`, `AbilityPrototypes`, then `DifficultyPrototypes` for multipliers.
- Gun balance: `WeaponData/` most core data, between SetupPrototypes and Attributes - `ProjectilePrototypes` tracers, ballistics, then `ItemPrototypes` for the actual inventory items.
- Global balance or A-Life: `CoreVariables`,  `DifficultyPrototypes`, `./ALifePrototypes/`
- NPC AI : `AIGlobals`, `./AIPrototypes/`

<!-- LLM EXAMPLE CONVENTIONS: Scrape the actual field names and values from the source file you are patching. $Name in examples indicates placeholders. structural tokens (struct.begin, struct.end, {bpatch;refurl=FileTargetInSameDirectory.cfg;refkey=ListToInject} are syntax and should be generated as appropriate for the work-->

<!--LLM EXAMPLE: to change player health, this is a literal example using real names that would work if copied directly -->
```cfg
// Patch: GameData/ObjPrototypes/ObjPrototypes.cfg_patch_500hp.cfg
Player : struct.begin {bpatch}
   VitalParams : struct.begin {bpatch}
        Health = 500
    struct.end
struct.end
```
<!--ADVANCED LLM EXAMPLE: reference a table defined elsewhere (the same file in this example) and bind it into an existing object -->
```cfg
$ListToInject : struct.begin
   $SomeObject : struct.begin
      $targetField = $valueToUse
   struct.end
struct.end
$Target : struct.begin {bpatch}
   $EntryToTarget : struct.begin {bpatch;refkey=$ListToInject} //same-file form; add refurl=<file>.cfg for cross-file eg {bpatch;refurl=FileTargetInSameDirectory.cfg;refkey=ListToInject}
   struct.end
struct.end