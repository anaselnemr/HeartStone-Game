# 🃏 HeartStone — A Java Card-Game Engine

A local 2-player, hot-seat Hearthstone clone built from scratch in Java with a Swing UI. Pick a hero, summon minions, sling spells, and trigger legendary effects across a clean, fully-modeled card-game engine — turn structure, mana economy, fatigue, board state, and all the keyword interactions you'd expect from the genre.

---

## Highlights

- **5 playable hero classes**, each with a unique Hero Power costing 2 mana.
- **13 hand-tuned spells** spanning direct damage, AoE, board control, transformation, and stat manipulation.
- **Minion keywords**: Taunt, Divine Shield, Charge, plus summoning sickness rules for newly-played minions.
- **7 legendary minions** with named, signature triggers — game-defining effects that fire on entry, attack, or destruction.
- **Faithful Hearthstone systems**: 30 HP heroes, 10-mana cap with one-per-turn mana crystal growth, 10-card hand cap, 7-minion board cap, fatigue damage when the deck runs dry.
- **Swing-based UI** with custom card art, animated turn flow, and full mouse-driven play.

---

## Heroes

| Hero | Class | Hero Power |
|------|-------|------------|
| **Jaina Proudmoore** | Mage | **Fireblast** — Deal 1 damage to any character. |
| **Rexxar** | Hunter | **Steady Shot** — Deal 2 damage to the enemy hero. |
| **Uther Lightbringer** | Paladin | **Reinforce** — Summon a 1/1 Silver Hand Recruit. |
| **Anduin Wrynn** | Priest | **Lesser Heal** — Restore 2 health to any character. |
| **Gul'dan** | Warlock | **Life Tap** — Take 2 damage; draw a card. |

---

## Spells

| Spell | Cost | Effect |
|------|------|--------|
| **Polymorph** | 4 | Transform a minion into a 1/1 Sheep. |
| **Flamestrike** | 7 | Deal 4 damage to all enemy minions. |
| **Pyroblast** | 10 | Deal 10 damage to any character. |
| **Kill Command** | 3 | Deal 5 damage to a target. |
| **Multi-Shot** | 4 | Deal 3 damage to two random enemy minions. |
| **Holy Nova** | 5 | Deal 2 damage to all enemies, restore 2 health to all friendly characters. |
| **Divine Spirit** | 2 | Double a minion's current health. |
| **Shadow Word: Death** | 3 | Destroy a minion with 5 or more attack. |
| **Seal of Champions** | 3 | Give a minion +3 attack and Divine Shield. |
| **Level Up!** | 6 | Give all friendly Silver Hand Recruits +2/+2. |
| **Curse of Weakness** | 2 | Give all enemy minions −2 attack until your next turn. |
| **Siphon Soul** | 6 | Destroy a minion. Restore 3 health to your hero. |
| **Twisting Nether** | 8 | Destroy all minions on the board. |

---

## Minion Keywords

- **Taunt** — Enemies must attack this minion before they can target anything else.
- **Divine Shield** — The first source of damage dealt to this minion is ignored; the shield then breaks.
- **Charge** — Skips summoning sickness. Can attack the turn it is played.

---

## Legendary Triggers

| Legendary | Trigger |
|-----------|---------|
| **Kalycgos** | Reduces the cost of Dragon spells in your hand by (4). |
| **Prophet Velen** | Doubles the effect of your spells and Hero Power. |
| **Wilfred Fizzlebang** | Cards drawn by your Hero Power cost (0). |
| **Chromaggus** | Whenever you draw a card, put another copy into your hand. |
| **Icehowl** | Cannot attack heroes — but charges into minions on the turn it's played. |
| **King Krush** | Comes in with Charge, ready to swing on summon. |
| **Tirion Fordring** | Enters play with Divine Shield and Taunt. |

---

## Tech Stack

- **Language**: Java
- **UI**: Swing (custom rendering for cards, board, hand, and hero portraits)
- **Assets**: bundled card art, fonts, and a CSV-driven neutral minion pool

---

## Project Structure

```
HeartStone-Game/
├── src/                   # Game engine + Swing UI source
├── bin/                   # Compiled class output
├── images/                # Card art, hero portraits, board assets
├── fonts/                 # Custom typography
├── neutral_minions.csv    # Data-driven neutral minion roster
├── Game Instructions.TXT  # Quick rules reference
└── README.md
```

---

## Getting Started

```bash
# Clone
git clone https://github.com/anaselnemr/HeartStone-Game.git
cd HeartStone-Game

# Compile
javac -d bin src/**/*.java

# Run
java -cp bin Main
```

> Requires JDK 8 or later. The game launches into the Swing UI; pick a hero per player and start trading blows.

---

## Course Context

Built for the **Object-Oriented Programming / Software Engineering** track of the **B.Sc. in Computer Science & Engineering** at the **German University in Cairo (GUC)**, 2022. Designed as an end-to-end exercise in OOP modeling — class hierarchies for cards, minions, spells, and heroes; polymorphic effect dispatch; event-driven game state; and a clean separation between the rules engine and the rendering layer.

---

## Authors

Anas ElNemr  ·  Ahmed Eltawel
