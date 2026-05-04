# HeartStone — A Java Card-Game Engine

A faithful Java re-implementation of Blizzard's *Hearthstone*, built from scratch as a
university project. Two players sit at one keyboard, each pick one of five hero classes
(Mage, Hunter, Paladin, Priest, Warlock), and battle through a turn-based card duel —
playing minions, casting spells, and triggering hero powers until one hero's HP hits zero.
The engine is a pure object-oriented Java MVC: a `Game` core (model + rules), a
`Controller` that wires Swing input to the model, and a hand-painted Swing UI with
animated card art, sound effects, and class-specific backgrounds.

---

## Gameplay

- **Local hot-seat PvP only.** Two human players share one machine — there is no AI
  opponent and no networked multiplayer. The game flow is `StartView` -> `MainView` ->
  `HeroView` (each player picks a hero) -> `GameView` (the board) -> `EndView` / `PlayAgain`.
- **Standard Hearthstone-style turn loop** (`engine/Game.java`):
  1. A coin flip picks who goes first; the starter draws 3 cards, the opponent draws 4.
  2. Each turn starts with `+1 total mana` (capped at 10), all minions wake up
     (`sleeping = false`) and refresh their attack flag, the active hero draws one card,
     and the hero power becomes available again.
  3. The active hero may play any combination of minions, cast spells, attack with
     awake minions, and use their hero power once per turn — provided every action
     passes the eight-rule `ActionValidator`.
  4. End the turn -> the other player gets the same loop.
- **Win condition**: bring your opponent's hero to 0 HP. Death triggers the
  `HeroListener.onHeroDeath()` callback, which fires `GameListener.onGameOver()` and
  swaps the UI to `EndView`.
- **Fatigue**: drawing from an empty deck takes escalating self-damage (1, 2, 3, ...) on
  every subsequent draw — the same fatigue rule as the real game.
- **Mulligan-free start**: the game does not implement the opening mulligan; you play
  the cards you're dealt.

## Card Mechanics — what is actually implemented

The `model` package has 14 neutral minions defined in `neutral_minions.csv`, plus 13
class spells (split across the five hero classes), plus class-specific Legendary minions
and the Paladin's summoned Silver Hand Recruits. Every mechanic below is grep-able to a
real method in the source.

### Minion keywords (`model/cards/minions/Minion.java`)

| Keyword | Implementation |
|---|---|
| **Taunt** | `Game.validateAttack(...)` raises `TauntBypassException` if any opposing minion has `isTaunt() == true` and the chosen target is not that taunt. |
| **Divine Shield** | `Minion.attack(...)` consumes the shield instead of taking damage — including the symmetric "both attacker and defender are divine" handshake (each strips the other's shield in one swing). Spells (`Flamestrike`, `HolyNova`, `KillCommand`, `Pyroblast`, `MultiShot`) also strip Divine Shield instead of dealing their damage. |
| **Charge** | The constructor sets `sleeping = !charge`. Charge minions can attack the turn they are played; non-Charge minions must wait one turn. |
| **Sleeping / Summoning sickness** | Cleared at end-of-turn for the next active hero in `Game.endTurn()`. `validateAttack` blocks attacks from sleeping minions with `CannotAttackException`. |

> **Not implemented**: Battlecry, Deathrattle, Stealth, Windfury, card-draw triggers,
> spell damage, freeze, secrets, weapons, overload. The game is intentionally scoped to
> the four keywords above plus a handful of bespoke per-card effects (below).

### Spells — abstract dispatch by tag interface

`Hero.castSpell(...)` is overloaded against five tag interfaces in
`model/cards/spells/`:

- `FieldSpell` — operates on the caster's whole field (`performAction(ArrayList<Minion>)`).
- `MinionTargetSpell` — operates on one chosen minion.
- `HeroTargetSpell` — operates on one chosen hero.
- `LeechingSpell` — destroys a minion and returns the heal amount.
- `AOESpell` — operates on both fields at once (typically the opposing field).

A single spell can implement multiple tags (e.g. `KillCommand` is both
`MinionTargetSpell` and `HeroTargetSpell`; `Pyroblast` is too), and the right overload
fires automatically based on which target the player clicks.

| Spell | Class | Cost | Effect (verbatim from the source) |
|---|---|---|---|
| **Polymorph** | Mage | 4 | Renames the target to "Sheep" and sets attack=1, HP=1, taunt/divine off, sleeping. |
| **Flamestrike** | Mage | 7 | AOE 4 damage to every enemy minion (Divine Shields strip first). |
| **Pyroblast** | Mage | 10 | 10 damage to a chosen minion **or** hero. |
| **Kill Command** | Hunter | 3 | 5 damage to a chosen minion **or** 3 damage to a chosen hero. |
| **Multi-Shot** | Hunter | 4 | 3 damage to two random distinct enemy minions. |
| **Holy Nova** | Priest | 5 | 2 damage to all enemy minions, +2 HP to all friendly minions. |
| **Divine Spirit** | Priest | 3 | Doubles a chosen minion's max HP and current HP. |
| **Shadow Word: Death** | Priest | 3 | Destroys a minion with attack >= 5 (else `InvalidTargetException`). |
| **Seal of Champions** | Paladin | 3 | +3 attack and grants Divine Shield to a chosen minion. |
| **Level Up!** | Paladin | 6 | +1/+1 to every Silver Hand Recruit you control. |
| **Curse of Weakness** | Warlock | 2 | -2 attack to every enemy minion (clamped at 0). |
| **Siphon Soul** | Warlock | 6 | Destroys a minion; heals your hero for 3. |
| **Twisting Nether** | Warlock | 8 | Destroys every minion on both fields. |

### Hero powers (2 mana, once per turn)

Each hero overrides `useHeroPower(...)` with class-specific behavior:

- **Mage (Jaina Proudmoore)** — *Fireblast*: 1 damage to a chosen minion **or** hero. Strips Divine Shield instead of dealing damage if present.
- **Hunter (Rexxar)** — *Steady Shot*: 2 damage straight to the opposing hero.
- **Paladin (Uther Lightbringer)** — *Reinforce*: summon a 1/1 "Silver Hand Recruit" to your field (raises `FullFieldException` if your field is full).
- **Priest (Anduin Wrynn)** — *Lesser Heal*: +2 HP to a chosen minion or hero. With Prophet Velen on the field, the heal becomes +8.
- **Warlock (Gul'dan)** — *Life Tap*: deal 2 damage to yourself and draw a card. With Wilfred Fizzlebang on the field, drawn minions cost 0 mana.

### Class-specific Legendary effects

Several legendary minions trigger by name-check (`Hero.fieldContains(String)`),
threaded through the relevant action paths:

- **Kalycgos** (Mage) — your single-target / AOE spells cost 4 less while it lives (Mage's `castSpell(...)` overrides).
- **Prophet Velen** (Priest) — quadruples the Priest's hero-power output.
- **Wilfred Fizzlebang** (Warlock) — minions you draw via Life Tap cost 0 mana.
- **Chromaggus** (neutral) — every drawn card is duplicated to your hand (if there is room).
- **Icehowl** (neutral, `model/cards/minions/Icehowl.java`) — overrides `attack(Hero)` to throw `InvalidTargetException`: this minion can never attack heroes directly.
- **King Krush** (Hunter) and **Tirion Fordring** (Paladin Taunt + Divine Shield) — built directly inside the hero's `buildDeck()` rather than the CSV.

### Card pool

- 14 neutral minions defined as CSV rows in `neutral_minions.csv` (name, mana, rarity, attack, HP, taunt, divine, charge): Goldshire Footman, Stonetusk Boar, Bloodfen Raptor, Frostwolf Grunt, Wolfrider, Chillwind Yeti, Boulderfist Ogre, Core Hound, Argent Commander, Sunwalker, Chromaggus, The Lich King, Icehowl, Colossus of the Moon.
- Each hero deck is built in their `buildDeck()` override: 13-15 random neutral minions (no more than 2 copies of any non-Legendary, max 1 of any Legendary), 2 copies each of their class spells, and 1 copy of their class Legendary minion.
- Decks are `Collections.shuffle()`'d after building.

## Tech Stack

- **Java SE 1.8** — declared in `.classpath` (`JavaSE-1.8`).
- **UI: Java Swing** (`javax.swing.*`) with `JFrame`, `CardLayout`, raw absolute
  positioning. No FXML, no JavaFX, no MVVM — every panel is hand-laid-out with
  `setBounds(x, y, w, h)`.
- **Audio: `javax.sound.sampled`** — looping intro / main music plus per-minion
  attack sound effects (`*A.wav`) loaded from `images/`.
- **Assets**: `.PNG` card portraits, `.gif` minion idle animations, `.wav` voice
  lines (one per minion), all bundled under `images/`. Two custom TrueType fonts
  (`background.ttf`, `BelweBdBTBold.ttf`) under `fonts/`.
- **Tests**: JUnit 5 wired into the classpath (`org.eclipse.jdt.junit.JUNIT_CONTAINER/5`)
  — no test source files ship with this snapshot.
- **Build**: Eclipse project (`.classpath` + `.project` + `.settings/`). No Maven /
  Gradle pom — the canonical run path is "open in Eclipse and hit Run on `StartView`".

## Project Structure

```
src/
  engine/                      core game loop + action gating
    Game.java                  turn flow, coin flip, win condition, validates 8 action rules
    ActionValidator.java       interface implemented by Game (validateTurn / validateAttack /
                               validateManaCost / validatePlayingMinion / validateUsingHeroPower)
    GameListener.java          onGameOver() callback fired when a hero dies
    Controller.java            Swing controller, ~6.8k LOC, wires every JButton click to Game
    Controller2.java           vestigial commented-out subclass (kept in tree, not used)

  model/
    cards/
      Card.java                abstract: name, manaCost, rarity, Cloneable
      Rarity.java              enum: BASIC, COMMON, RARE, EPIC, LEGENDARY
      minions/
        Minion.java            attack/HP, taunt/divine/charge/sleeping/attacked, attack(Minion|Hero)
        Icehowl.java           overrides attack(Hero) -> InvalidTargetException
        MinionListener.java    onMinionDeath(Minion) -> Hero removes from field
      spells/
        Spell.java             abstract Card
        AOESpell, FieldSpell, HeroTargetSpell, MinionTargetSpell, LeechingSpell  tag interfaces
        13 concrete spells     Polymorph, Flamestrike, Pyroblast, KillCommand, MultiShot,
                               HolyNova, DivineSpirit, ShadowWordDeath, SealOfChampions,
                               LevelUp, CurseOfWeakness, SiphonSoul, TwistingNether
    heroes/
      Hero.java                abstract: 30 HP, mana, deck (30), hand (max 10), field (max 7),
                               fatigue, draws + plays + casts + attacks, listenToMinions()
      HeroListener.java        onHeroDeath, damageOpponent, endTurn
      Mage / Hunter / Paladin / Priest / Warlock  concrete heroes; each overrides buildDeck()
                               and useHeroPower(...) (and Mage overrides castSpell to bake in
                               the Kalycgos discount)

  exceptions/                  checked exceptions for every illegal action
    HearthstoneException.java  abstract base
    NotYourTurnException, NotEnoughManaException, FullHandException (carries the burned card),
    FullFieldException, HeroPowerAlreadyUsedException, InvalidTargetException,
    NotSummonedException, CannotAttackException, TauntBypassException

  view/                        Swing UI (every screen is its own JFrame)
    StartView.java             entry point: intro animation + music, hero pick screen
    MainView.java              main menu (Start / Game Board / Credits / End)
    HeroView.java              both players pick a class (Mage/Hunter/Paladin/Warlock/Priest)
    PreHeroView.java           transition GIF between menu and hero pick
    GameView.java              the play board: ~2.9k LOC of hand-positioned JButtons /
                               JLabels for each card slot, mana crystal, shield, attack /
                               health overlay
    GameView2.java             supplementary board panel
    EndView.java               winner screen
    PlayAgain.java             rematch flow (re-instantiates both heroes, restarts the Game)
    CreditsView.java           team photos + names
    RulesView.java             in-game rules screen
    font.java                  font loader helper

images/                        ~400 .PNG / .gif / .wav assets (one per minion + UI chrome)
fonts/                         background.ttf / BelweBdBTBold.ttf / Gamefont.{otf,ttf}
neutral_minions.csv            14 rows defining the neutral minion pool
Game Instructions.TXT          original how-to-play text
.classpath / .project          Eclipse project files
```

## How to Build & Run

This is an Eclipse project, not a Maven / Gradle build. The fastest path is Eclipse;
plain `javac` works too.

### Option 1 — Eclipse (the originally intended path)

1. `git clone https://github.com/anaselnemr/HeartStone-Game.git`
2. Open Eclipse -> `File` -> `Import` -> `Existing Projects into Workspace` -> pick the
   cloned folder.
3. Make sure the project compiles against **Java 8 (JavaSE-1.8)** — the classpath
   pins this; on newer JDKs (11+) you may need to flip the JRE library to a 1.8-
   compatible runtime, otherwise some Swing internals will warn.
4. Right-click `src/view/StartView.java` -> `Run As` -> `Java Application`.

### Option 2 — command line

```bash
git clone https://github.com/anaselnemr/HeartStone-Game.git
cd HeartStone-Game

# Compile every .java in src/ to bin/ (run from the repo root so the working dir
# matches the relative paths the views use for images/, fonts/, neutral_minions.csv)
mkdir -p bin
javac -source 1.8 -target 1.8 -d bin $(find src -name "*.java")

# Run — the working directory MUST be the repo root, otherwise the views will not
# find images/, fonts/, or neutral_minions.csv (they are loaded with relative paths)
java -cp bin view.StartView
```

> **Heads-up**: the views use OS-style separators in some path strings
> (`"images//Startviewaudio.wav"`). On Linux / macOS these double-slashes are
> harmless (`//` collapses to `/`). On Windows they Just Work because the backslash
> is also tolerated. If a sound file fails to load, double-check you launched from
> the repo root.

### How to play

From the original `Game Instructions.TXT`:

- Click `Start Game` from the main menu — both players pick a hero.
- Click any card in your hand to play a minion or cast a non-targeted spell.
- For a targeted spell: click the spell, then click the target.
- To attack: click your minion (it turns red = selected), then click the enemy minion or hero.
- Your hero power sits on the top-right of your portrait — click it to cast.
- Visual cues on the field: shield icon = Taunt, yellow shield above = Divine Shield,
  Z icon top-right = Sleeping (summoning sickness).

## Coursework Context

Built jointly by **Anas ElNemr** and **Ahmed Eltawel** during their B.Sc. in Computer
Science & Engineering at the **German University in Cairo (GUC)**, circa 2022. The
project sits squarely in the OOP / Software Engineering coursework zone:

- The class hierarchy demonstrates **inheritance** (`Hero` -> `Mage` / `Hunter` / ...;
  `Card` -> `Spell` / `Minion`; `HearthstoneException` -> 9 typed children) and
  **polymorphism** (every `castSpell` overload dispatches by tag interface; the same
  `attack(...)` works on minion or hero targets).
- The engine cleanly separates **rules** (`Game` implements `ActionValidator` with 8
  validation methods) from **state** (`Hero` owns deck / hand / field) from **view**
  (`Swing`), with **listeners** (`MinionListener` / `HeroListener` / `GameListener`)
  bridging the layers — a textbook MVC-with-callbacks setup.
- Every illegal action raises a **typed checked exception**, all descending from
  `HearthstoneException` — explicitly demonstrating Java's exception hierarchy.
- `Card implements Cloneable` and the deck-build path uses `clone()` to ensure a
  legendary appears at most once per deck.

## Authors

- **Anas ElNemr** — [@anaselnemr](https://github.com/anaselnemr)
- **Ahmed Eltawel** — [@ahmedeltawel](https://github.com/ahmedeltawel) (commit author of `Final commit`)

The in-game `CreditsView` also pictures **Amera** and **Slim** as part of the
extended credits screen — full names not enumerated in the source.

## Acknowledgements

This repository is a friendly mirror of
[github.com/ahmedeltawel/HeartStone-Game](https://github.com/ahmedeltawel/HeartStone-Game)
and shares the same source. Both authors share full credit for the engine, the
Swing UI, and every line of the card-mechanic code. Card art, sound effects, fonts,
and the *Hearthstone* concept are (c) Blizzard Entertainment — used here strictly
for the educational coursework that produced this project, with no commercial intent.
