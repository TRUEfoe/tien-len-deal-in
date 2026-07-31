# Tiến Lên: Deal In

A roguelike take on Tiến Lên (Vietnamese Thirteen) — in the way Balatro is a
roguelike take on poker. Built in Godot 4.7 for iPhone and Android, landscape.

## Modes

**Classic** — a full four-player hand of Southern-rules Tiến Lên against
three AI opponents, played for money (see below).

### Who opens a deal

The **winner of a deal leads the next one, and may open with anything**. Only
the first deal of a session is opened by the holder of the lowest card, who must
include it — with a full four-player deal that is always the 3♠. Both modes work
this way, and `_test_winner_opens` pins down the corners: a four-player table, a
seat that does not exist falling back rather than seating nobody, and cheating
before the first play not stealing a lead that was already earned.

**Roguelike** — a five-opponent run, **played for money**. Pick an opponent
from the ones offered on the current rung, then duel it: you sit down with a
stake, it sits down with a bankroll, and every payment is a **transfer** —
what one seat loses the other takes. Clean them out and their money is yours.

That is the whole model, and it replaced hit points because HP was a borrowed
abstraction. The game is about money everywhere else — the classic table has a
bankroll, an entry fee, chop stakes and payouts; the relics are street objects;
the setting is a neighbourhood gambling table. Nobody at a Tiến Lên table has
hit points. They have a wallet. One economy now runs both modes.

Money moves from three places, so the purse is moving the whole time rather
than only between deals:

- **Losing a round** pays the round's winner, doubled when the combo that took
  it was four cards or more — the same "buried under a stack" threshold that
  lights a player on fire.
- **Getting chopped** pays the chopper on the spot (3 x the chop tier).
- **Losing a deal** pays per card left in hand: 2 per number card, 3 per
  J/Q/K/A, 5 per 2 — all in `BattleRules.SCALE` units, so they read as money.

Round stakes exist because heads-up deals are usually close. Measured over 4000
deals, **48% end with the loser holding one to three cards**. Charging only for
the cards left in hand meant the most common and most tense kind of deal barely
registered.

Because a duel is zero-sum, **a bad duel costs you stake rather than ending
you**: the run erodes instead of stopping dead, which is a far better shape
than an HP bar that empties. Your purse carries between duels and is never
topped up — what you walk out with is exactly what you took off the table. Win
and you choose one of three relics, then face the next rung; win the whole run
and the purse is paid into the wallet the classic table draws on. Get cleaned
out and the run is over, buy-in gone.

### Choosing your hand

A heads-up deal is the luckiest moment in the game: you are handed thirteen
cards and everything after is downstream of them. So every deal now opens on
**four hands dealt face down**, and you take the one you want; the opponent
takes one of the three you left, at random, seeing no more than you do.

The hands are **dealt in**, one card to each in turn the way a person deals —
fast enough to read as one motion rather than fifty-two separate ones.

Picking is two steps: **select, then confirm**. A hand you click lights up, its
row lifts, and its radio mark fills; a large gold **Take this hand** commits.
Committing on a single click gave no feedback and no way back on the biggest
decision in the duel, and it stopped you narrowing to a favourite and *then*
buying a reveal to settle it.

Four things can be bought before you choose, out of the very purse you are
about to duel with:

| Reveal | Price | Shows |
| --- | --- | --- |
| Show lowest card | 5% of purse | flips each hand's lowest card face up |
| Show random card | 20% of purse | flips one more card in each hand, never the lowest |
| Show suits | 25% of purse | a suit table beside every hand |
| Show face cards | 35% of purse | how many J/Q/K/A each hand holds |

Every price is a **fraction of the purse**, never a flat fee — a fixed price
becomes pocket change once you are carrying three bankrolls, and the whole
point is that a reveal has to hurt on rung five as much as it does on rung one.

They are charged against what is *left*, not what you started with, so each
purchase makes the next cheaper in absolute terms. That is what stops buying
all four from being simply impossible: taken in order they come to about two
thirds of the purse rather than the 85% the rates suggest — on a $150 purse,
$10 then $30 then $30 then $30, leaving $50 to duel with.

Which card the random reveal turns over is settled when the hands are dealt,
not when the reveal is bought — otherwise re-rolling it would be a matter of
buying it, and it is meant to be one draw of luck. It never picks the lowest
card: that one is already for sale next door, and paying twice for the same
card is a refund, not a gamble.

Everything in a row — the cards, the radio mark, the suit pips and the face-card
count — is laid out from the row's **middle**, not its top. `CardVisual` pivots
on its centre, so a card drawn at 42% covers `position + CARD_SIZE/2 ±
CARD_SIZE * 0.42/2`; treating `position` as the visible top-left drops the row
39px and hangs it out of the bottom of the strip it is meant to sit in.

Once you confirm, **the two chosen hands are played out to their seats** —
yours down the screen, the opponent's up to theirs, the two nobody took swept
away. That flight is the only moment the opponent's choice is visible.

**What you spend here you cannot win with.** That is the whole tension, and it
is the right kind of decision at a gambling table: information is worth money,
and you decide how much. Prices are a fraction of the purse, so they scale with
the run rather than becoming free by rung five.

Every candidate is thirteen cards even for a seat owed more by a relic — four
comparable thirteens is what makes the choice a choice, and Cà Phê Sữa Đá's
extra cards are drawn after you have picked (`TienLenGame.preset_hands`, which
also strips the chosen cards out of the deck before dealing anyone else in;
`_test_preset_hands` pins that down, since a duplicated card is the kind of bug
that only shows up as an impossible quad three deals later).

The rules engine supports 2-4 players, so more foes can join a duel later
(max 4 seats including you). With fewer than four players the "3♠ opens" rule
generalizes to "the lowest dealt card opens."

## Relics

Fourteen of them, offered three at a time after each win, weighted 60/30/10
by rarity and never repeating one you already hold.

| Relic | | Rarity | Effect |
| --- | --- | --- | --- |
| Nước Mía | sugarcane juice | Common | Your bombs blow up any single card, not only a 2 |
| Bánh Mì | the baguette sandwich | Common | Take $10 off your opponent at the start of every deal |
| Cà Phê Sữa Đá | iced coffee over condensed milk | Common | You are dealt 15 cards instead of 13 |
| Rượu Thuốc | wine steeped with medicinal roots | Common | You sit down with $30 more, paid in on pickup |
| Chiếu Cói | a woven sedge mat | Rare | Every card left in your hand costs you $5 less |
| Lì Xì | a red envelope of lucky money | Rare | Your straights of four or more may skip one rank: 3-4-5-7 is legal |
| Áo Dài | the long split-panel dress | Rare | Your spades are read as clubs and your diamonds as hearts, at the same rank |
| Nhang Trầm | agarwood incense | Rare | Once each duel, being cleaned out stakes you $40 instead |
| Dao Bầu | the heavy kitchen cleaver | Rare | Your chops take an extra $20 off whoever they land on |
| Kính Râm | dark glasses | Rare | You can see your opponent's hand |
| Gà Chọi | a fighting cock | Rare | Your triples beat a single 2 |
| Nón Lá | the conical palm-leaf hat | Legendary | Your spades are read as diamonds and your clubs as hearts, at the same rank |
| Ông Địa | the earth god | Legendary | Every deal you win takes an extra $30 off your opponent |
| Sảnh Rồng | a dragon straight | Legendary | A straight of four or more beats any single or pair |

Relic names stay Vietnamese in every language — they are the things
themselves, not translations of them — so the English build prints the middle
column above under the name as a gloss. The Vietnamese build leaves it blank
and the effect text moves up into its place: telling a Vietnamese player that
nước mía is sugarcane juice would read as an insult rather than a
translation. `_check_item_gloss` pins down both halves.

The two suit relics are lenses, not restrictions. The deal is untouched — all
52 cards, dealt normally — and what changes is how your side of the table
*reads* them. Each off-suit is re-read as a suit you do keep, of the same
colour, **at the same rank**:

| | reads as | |
| --- | --- | --- |
| Nón Lá (♦♥) | ♠ → ♦ | ♣ → ♥ |
| Áo Dài (♣♥) | ♠ → ♣ | ♦ → ♥ |

Rank never moves, and that is the point. Restricting the *deal* to two suits
was the first version, and it capped every rank at two cards — a lensed hand
could never hold a triple or a quad. Promoting each off-suit by a rank was the
second, and it split four Kings into two Kings and two Aces, which defeats the
same goal by a different route. Re-reading the suit alone leaves four Kings as
four Kings: under Nón Lá they arrive as K♦ K♦ K♥ K♥, still a four of a kind.

Two cards can therefore come through identical (a native K♦ and a K♠ read as
K♦). Rank is what pairs, triples and quads are made of, so that works, and a
card cannot beat its own twin.

The lenses do not blend. Nón Lá covers the same ground in better suits, so a
player holding both plays under the hat and the dress is ignored —
`suit_lens_rank` decides, and merge order does not matter.

A re-read card is simply drawn **as the card it now is** — rank corner and big
pip are the new suit — with one mark to say what happened: the suit it was
dealt in, small and faded, pencilled out in the bottom-left corner. That
corner because it is the part still showing when a full hand is fanned out and
each card overlaps the next. Nothing marks the rank, since a lens never moves
it, and there is no edge stripe or arrow: the card has to read as a card first.

Each relic has its own icon, drawn in code in the same cel-shaded style as
the foe portraits — the game has no image assets and a relic is not going to
be the exception. `scripts/ink.gd` owns the look (ink outlines, flat fills,
hard shadows) and both `FoePortrait` and `ItemIcon` draw through it, so a
relic and a face are made of the same marks. `ItemIcon.draws()` reports
whether a relic has art, and the catalogue test fails if one does not, so a
new item cannot ship as a blank circle.

Every figure a relic quotes in its description is checked against the figure
it actually delivers (`_test_item_text_matches_effect`). The duel moved from
hit points to money and the numbers were rescaled without the text following,
so six of these cards spent a while promising HP the game no longer had —
nothing else catches that, because a description is never read by the code.

A relic is not code. Each one is a handful of numbers on a `Ruleset`
(`scripts/ruleset.gd`), and `ItemDef.compile()` folds a whole collection into
the one object the rules engine and the battle scene read. `Combo.identify()`
and `Combo.beats()` each take a ruleset and default to the strict game, so
what a relic widens is *your* legal moves — the foe answering you is still
judged under whatever rules it holds, which is usually none.

## Opponents

Eleven foes across five rungs. The first rung offers three to choose from
since you are picking blind; every rung after that offers two.

| Rung | Foe (EN / VI) | Bankroll | Traits |
| --- | --- | --- | --- |
| 1 | Madam Eight · Bà Tám, The Market Auntie | $150 | Passes often, plays conservatively, hoards her 2s |
| 1 | Uncle Four · Ông Tư, Three Beers In | $140 | Misplays often, passes often, plays slowly |
| 1 | Little Ti · Thằng Tí, The Kid Next Door | $130 | Reckless, plays fast, misplays often |
| 2 | Mister Three · Cậu Ba, The Back-Alley Shark | $100 | 16 cards, dumps trash fast, cheats |
| 2 | Miss Four · Chị Tư, The Coffee-Stall Sister | $180 | Never breaks a run, sits on the face cards, slow |
| 3 | Uncle Seven · Chú Bảy, The Xe Ôm Rider | $210 | Dumps trash, never hesitates, leads high |
| 3 | Madam Nine · Bà Chín, The Numbers Runner | $270 | Bombs any single card, leads high, gambles |
| 4 | Master Six · Thầy Sáu, The Fortune Teller | $230 | Palms three cards a deal, conservative, slow |
| 4 | Brother Five · Anh Năm, The Cyclo Man | $130 | 15 cards, recovers between deals, outlasts you |
| 5 | The Masked One · Kẻ Bịt Mặt | $180 | 16 cards, bombs any single, chops hit harder |
| 5 | The Boss Lady · Bà Trùm, The House | $280 | Recovers, winning a deal costs you extra, passes often |

Names are localized like everything else. The Vietnamese names are a kinship
term plus a birth-order number — Bà Tám is literally "Madam Eight", Chị Tư is
"Miss Four" — so the English side keeps the numeral rather than mistaking Tám
for a given name. Thằng Tí really is a nickname ("little one") and the two
boss names are epithets, so those three translate as themselves.

Traits are real behaviour, not labels. Bà Tám declines about 10% of the
rounds she could win and folds rather than spend a 2 — so leading high cards
makes her give up tempo. Cậu Ba swaps his two worst cards for the best in
the undealt stock before each deal (the speech bubble is your tell). The two
tier-5 bosses hold relics of their own, expressed as the same `Ruleset` the
player's items compile to: anything an item can do, a foe can do back.

## Screen sizes

The project stretches `canvas_items` with an `expand` aspect, so **the drawable
area is 1280x720 only on a 16:9 display**. An iPhone 17 Pro is about 2.17:1,
which comes out near 1566x720; a 4:3 tablet is taller instead. Laying out
against the design size alone dumps every spare pixel on one side — which is
exactly the margin that showed up on a phone.

And **not all of the drawable area is usable.** In landscape the camera cutout
eats one side — which side depends on which way the phone is turned — and the
home-indicator strip along the bottom swallows a tap and pulls up the iOS
navigation gesture instead of playing a card.

`scripts/layout.gd` owns both problems. `Layout.size()` is the whole drawable
area; `Layout.safe()` is that less the cutout, the home indicator, and small
floors (`MIN_SIDE`, `MIN_BOTTOM`) that apply even where the device reports
nothing — iOS often reports no bottom inset in landscape, yet a tap in the last
few pixels still triggers the gesture. Insets are read from
`DisplayServer.get_display_safe_area()` and converted into game units, and only
on a handheld: a desktop query describes the whole monitor rather than the
window, so comparing the two would invent insets that are not there.

Anything full-bleed — a background, the transition curtain — is sized to
`size()`, so the cutout area is filled with the scene's own colour rather than
black. Anything a finger has to hit or an eye has to read is laid out inside
`safe()`: the hand rests clear of the home indicator, and the two thumb buttons
sit in the corners of the safe rect rather than the screen.

Screens ask for these at build time rather than trusting a constant, and follow
three habits:

- **Full-bleed things are sized to the live area** — backgrounds, the
  transition's card grid, the petal layer, the menu's dim shade.
- **Centred things are measured from `size().x`**, not from 1280.
- **Edge-anchored things mirror onto the real edge.** The classic table seats
  West and East by subtracting from the live width, and the opponent-select
  road spreads its five stops across it, so a wider screen seats a wider table
  rather than leaving a gap.

`Layout.center_offset()` is for the remaining case: a cluster of absolutely
placed parts that only needs to stay together. The battle table's foe portrait,
name block and trait chips move as one by that offset, while the hand and the
two action buttons use the full width and reach the true corners.

A desktop window reports no insets, so the harness can stand in for a phone:
`Layout.debug_insets` is set from `SAFE_DEMO`, which is the only way to see the
safe-area layout before shipping it. Check a change on all four, since each
catches something the others do not — the wide runs exercise the width maths,
and the two demo runs catch anything pinned to one side only:

```sh
godot --resolution 1566x720 --path . res://tests/screenshot.tscn              # phone aspect
godot --resolution 1280x720 --path . res://tests/screenshot.tscn              # 16:9
SAFE_DEMO=1 godot --resolution 1566x720 --path . res://tests/screenshot.tscn  # cutout left
SAFE_DEMO=2 godot --resolution 1566x720 --path . res://tests/screenshot.tscn  # cutout right
```

## Presentation and polish

- **Screen transitions.** Every screen change goes through the `SceneFx`
  autoload: a hand of face-down cards is thrown across the screen from the
  dealer's corner, the scene swaps while it is covered, and the cards are
  swept off the other side. A backdrop fades in under the cards so no sliver
  of the old scene survives, however the rotated cards fall.
- **The road.** Opponent select is an overworld map: five stops from the
  market to the gambling den, beaten stops stamped with a red seal, the
  current one ringed in pulsing gold, the rest still question marks. Each
  stop carries a drawn mark — awning, coffee cup, signpost, temple gate,
  fanned cards — matching who tends to wait there.
- **Tết dressing.** Red is the lucky colour, so the main menu and the reward
  screen wear lacquer red with gold text, swaying paper lanterns, and hoa mai
  petals drifting down (`FestiveFx`).
- **Table juice.** Hands slide up from the table edge on each new deal, a
  tapped card eases up instead of snapping and stays picked out while the other
  seats play — the hand is torn down on every refresh, so selection is carried
  across by `Card` instance id rather than by slot, damage and healing float off the
  purses as red −$N / green +$N numbers, chops jolt the whole table, and the
  status line turns gold on your turn.

## Table presentation

Both modes share their table look through `TableBackdrop`, `TableFx` and
`TableFlames`, so the classic table and a roguelike duel never drift apart:

- **The table itself.** Both modes opened on a flat fill once — one green
  rectangle, one navy one. A flat fill has no light in it and no material, so
  the cards sat *on* a colour rather than on a table, and every HUD element
  around the edges competed with the middle for the eye. `TableBackdrop` hangs
  a lamp over the pile instead: a warm pool of light exactly where the cards
  land, falling away to near-black in the corners, over cloth with a fine weave
  and the speckle of its fibres in it. Then a gold line ruled round the middle
  the way a card table rules one — doubled, because lacquerwork doubles a line —
  and the face of a **Đông Sơn bronze drum** sunk into the felt under the pile:
  a fourteen-rayed sun inside concentric bands, at watermark contrast. Radial,
  so it suits the middle of a table, and about as Vietnamese as a mark gets.

  The classic table is green felt in a lit room (`TableBackdrop.felt`); the duel
  is the same table after dark in a colder one (`.slate`). A duel is not a
  Sunday afternoon with the neighbours and should not be lit like one.

  The lighting is the project's only shader (`shaders/felt.gdshader`, canvas
  item, fine on the mobile renderer) because it is a *gradient*: drawing one
  with `draw_*` means stacking a hundred translucent circles every frame and
  the banding shows anyway. Everything on top of it is the same code-drawn
  marks as the rest of the game. Nothing on the layer animates, so it draws
  once and costs nothing per frame afterwards.

  One trap worth knowing: the layer is **sized**, not anchored. Anchors on a
  Control added in code do not resolve until a sort that may never come, and
  every other layer here gets away with it because none of them reads its own
  size. A `ColorRect` *is* its size — left to anchors, the cloth comes out a
  zero-by-zero rectangle and the whole backdrop silently disappears.

- **Action buttons** sit in the two bottom corners under the thumbs of a
  phone held in landscape — Pass on the left, Play on the right, the primary
  one warm and solid and the other receding into the felt. Hands are laid out
  in the band between them.
- **Rearranging your hand.** Cards are dealt sorted, but you can drag one
  along the row to group what you mean to play (`HandDrag`, both modes). The
  card follows your finger and rides over its neighbours; whatever it
  displaces slides out of the way as it passes, and the row restacks so the
  overlap stays left-over-right. Selection lands on *release*, since press is
  also where a drag starts, and a card only becomes a drag once the finger has
  travelled past `CardVisual.DRAG_SLOP`.

  The arrangement lives in the engine's own hand array, which is what makes it
  survive the rebuilds the other seats trigger between your turns — and a
  fresh deal builds a new array, so a new hand comes out sorted for free.
  Nothing in the rules reads a hand in order after the deal: `TienLenGame`
  takes `hands[i][0]` as the lowest card only while working out who opens,
  which is settled before anyone can touch a card, and `play()` erases by
  identity. `_check_hand_drag_reorders` pins down both halves — that a drag
  moves the card and that it does not change what is in the hand.
- **Fire.** A player on a roll has their cards catch light. It is earned by
  the play, not by running low: a four-card-or-longer straight, a triple, or
  three consecutive pairs light you up; burying someone under a stack of four
  or more to take their round burns hotter; chopping burns hottest. A hotter
  play stokes an existing fire, a lesser one never damps it down.

  **Only folding takes it away.** Being answered over the top does not: a run
  is a thing you hold until you run out of cards to hold it with, so you keep
  the fire for as long as you can keep answering the table and it goes out the
  moment you cannot. Two players trading big plays are therefore both alight.
  Catching light plays a cue; a fresh deal clears everyone.
- **Chops pay out.** A bomb is the biggest swing either mode has — it moves
  money at the classic table and burst damage in the duel — so it blows the
  pile apart in `MoneyBurst`: a shockwave ring, brass coins tumbling
  edge-on-to-face, and banknotes that hang and flutter down on their own
  drag while the coins drop straight. Pieces scale with the chop tier, and
  the effect frees itself once the last note is off the screen. It is added
  under the end-of-deal overlay, so a chop that finishes a battle does not
  rain money over the result.

  A seat's cards burn as one object rather than card by card: the fire wraps
  the whole span of them, standing tallest over the middle and sliding down
  past both edges, out of a white-hot bed that deepens to red at the tips.
  Each flame is a column with a drifting centreline and an uneven width, so
  the fire churns instead of pulsing. It draws behind the cards, so what you
  see is the flames licking past their edges — and in the roguelike it sits
  between the foe's portrait and its fan, in front of the avatar but behind
  the cards it is holding. With nothing lit the layer draws nothing and stops
  animating entirely.

## The classic table

Your hand along the bottom, three of the neighbourhood's regulars around the
table, and the current round in the middle.

**Nineteen regulars, three per hand**, drawn at random and named on their panels:

| | | |
| --- | --- | --- |
| Brother Tan | Auntie Truc | Auntie Vi |
| Uncle Peter | Chị Linda | Old Man Khang |
| Lil Steven | Young Henry | Kid Ivan |
| Chị Phượng | Mr. Charly | Bác Trọng |
| Big Shot Kenny | Bác Nguyệt | Chị Phương |
| Auntie Susan | Auntie Zoua | Cousin Hai |
| Master Wendey | | |

They fall into rough families — the steady, the cautious, the reckless, the
shovers, the showmen, and the three who are mostly not paying attention — and
inside a family they differ in what they protect. Auntie Truc will not spend
a face card; Bác Trọng will not break a run; Kid Ivan will do either without
noticing. Cousin Hai is the cockiest seat and backs it — he answers everything
and spends his face cards early, and the blunder rate is what the swagger costs
him. Master Wendey is the opposite and the sharpest seat at the table: soft
spoken, almost never misplays, and keeps her shapes together. Speed is part of it: the kids answer in under a second, Bác Nguyệt
takes three, and the spread is most of what tells them apart while you wait. The
distracted three are distracted differently: Old Man Khang dithers, Auntie Susan
folds rather than decide, and Auntie Zoua takes her own hand apart without
noticing she has done it.

Names carry the kinship term in English for some and in Vietnamese for others,
which is how a table like this actually sounds. **Chị** is kept rather than
translated to "Sister" wherever it reads naturally, so Chị Linda and Chị Phượng
are Chị in both builds. Chị Phượng and Chị Phương are still two different people
— Phượng and Phương are different names — and the diacritic is what keeps them
apart on both sides, exactly as it does in Vietnamese. `_test_table_personas`
fails if any two regulars would show the same name in either language, since
two identical panels at one table would be unreadable.

**The hand is dealt in.** Every card flies out of the middle of the table to
the seat it belongs to, one at a time round the table rather than thirteen to
one player and then thirteen to the next — which is what a deal actually looks
like, and the reason it reads as dealing rather than as four hands fading in.
Cards arrive face down; yours turn over as they land, the way you would pick
them up. The AI holds off until the last one has landed.

It works off the visuals `_refresh()` has already built and placed, so it never
has to know how a hand or a fan is laid out: it takes each card's resting spot,
moves it back to the deck, and lets it fly home.

**Standings** sit in the top-right corner: firsts won this visit, one row per
player, sorted by wins and yours picked out in gold. The count goes last
because the column is right-aligned, so the numbers line up against the edge.

Wins are booked against the **persona, not the seat** — three different
regulars turn up every hand, so a tally by chair would credit the win to the
furniture. The list shows everyone with a win to their name plus whoever is at
the table right now, so this hand's three appear on nought instead of only
showing up once they have beaten you; it is capped at eight rows so a long
session cannot run down into the east seat's panel. It is scene state on
purpose: dealing again keeps it, and walking out to the main menu tears the
table down and resets it.

**The end of a hand** is its own card (`ResultCard`), in the same lacquer red
and gold as the menu: where you came in one word as big as the panel will take,
what it paid under it, the four standings with yours picked out in a gold
strip, and the balance. Then **Deal Again**, sized across the panel and wearing
the same warm gold as the Play button — the thing to press at the end of a hand
should look like the thing you have been pressing all through it.

The card is **shared with the duel**, because it was not once: the classic
table got this treatment and the roguelike was left on a default-themed dialog
whose button was hard to pick out from the text above it. One card means that
cannot happen twice. Only the writing differs — the duel puts the result in the
headline (gold when it went your way, the warning colour when it did not), the
detail underneath, and where **both purses stand** on the line below the rule,
since that is the arithmetic a player is actually doing between deals. Its rule
sits lower, because a duel's detail line is a sentence where the classic card's
is "+$25".

What it deliberately does *not* say: a sentence repeating the placing the big
number already gave you, and a breakdown of the entry fee. The line about
being staked again only appears on the hand that empties your pockets. The
placing is written with `ordinal.*`, whose Vietnamese is the placing itself
(Nhất/Nhì/Ba/Tư) capitalised, because here it stands alone as a headline
rather than mid-sentence.

**Whose turn, and who is out.** A gold border round a name box says that seat
is thinking. You have no name box — a fourth panel down there would crowd the
hand for the sake of a card count you can read off your own cards — so your
turn lights the bottom edge of the screen instead: a thick rounded gold bar
under the hand that breathes, bar and glow together, while the table waits on
you. The hand is laid out to leave that strip clear, which matters on a phone
where the safe area is short enough that the cards would otherwise sit on it.

A seat that folds is struck out with a red X over its name box, dimmed under
it so struck-out-and-faded read as one state. Yours goes over the **Play
button** rather than over anything with your name on it: the button is what
you are reaching for, and a struck-out button says "not this round" far more
plainly than a greyed-out one. The engine clears `passed` when a round ends,
so every X lifts by itself at exactly the moment those seats are back in.

**Only the chair that came last turns over.** After every hand the three
regulars who finished above the loser keep their seats, and the one who came
last is replaced by somebody new — never by themselves, and never by anyone
already at the table. If *you* came last, nobody moves: the seat that lost its
chair is yours, and you are not going anywhere.

That is the difference between a table and a lobby. What you learn about a
regular in one hand is worth something in the next, and knocking one of them
out of the game means something. `_test_seat_rotation` pins the rule down, and
runs the draw sixty times — "the newcomer happens to be somebody already
sitting there" is a one-in-fifteen bug that a single draw will not find.

**They talk.** Four moments each, in their own voice, in both languages
(`seatbark.<id>.<moment>`):

| Moment | When |
| --- | --- |
| `dealt` | one of the three, on looking at a fresh hand |
| `win_deal` | going out first |
| `buried` | having a round taken off them by four cards or more |
| `lose_deal` | finishing last, said before the results card covers the table |

The lines are the character: Lil Steven opens with "Thank you, whoever dealt
this!" and wins with "I'M RICH!"; Auntie Truc wins with "And I still have two
aces — that's how you do it"; Old Man Khang wins with "I won? Oh, I won!
Somebody tell me what I did." One bubble serves all three seats, with its tail
pointing back at whichever panel spoke — including **upward** for the north
seat, which sits above its own bubble rather than beside it and otherwise got
an arrow aimed at empty table, and a cooldown between lines — three
regulars all talking at once is a crowd, not a table. `_test_seat_barks` fails
if any of the seventy-six keys is missing or empty in either language, since a
missing bark is silent rather than loud and nothing else would catch it.

Deliberately **style only**: the rule-bending flags (`bomb_any_single` and the
rest) are read off `game.rules`, which classic never sets, so a persona carrying
one would be claiming an edge it never gets — and a money game should be plain
Tiến Lên. `_test_table_personas` enforces that, along with unique ids, unique
names in both languages, and a think window inside 0.25-3.8s.

## Adding a foe

Foes are data: add a static factory to `scripts/foe.gd` with a name, bankroll,
palette, silhouette, and trait flags, then list it in `roster()`. Its name,
title, traits, and trash talk are Loc keys, so the text itself goes in
`scripts/loc.gd`. `FoePortrait` renders any `FoeDef` including its four
damage stages, `AIController` reads the trait flags, and the opponent-select
screen picks up the new entry automatically. Read the name through
`display_name()`, never the raw `name` field — that one is a fallback for the
classic-table personas, which are play styles rather than characters. Give it a `tier` from 1 to 5 to
place it on a rung of the ladder; tier 0 keeps it out of the roguelike.
`_standard_barks()` wires up the whole bark table from the foe's id, so the
only hand-written entries are the ones with a CHEAT line.

## The in-game menu

The main menu offers the two modes, with the relic list hung off the roguelike
button as an indented sub-item joined to it by an elbow — relics only exist in
that mode, so it reads as part of it rather than as a third thing to do.

Behind it, over the lacquer red, **a dragon's shadow crosses the panel now and
then** (`MenuDragon`): con rồng as the banners draw it, serpentine and
wingless, four short clawed legs, a maned head with antlers, beard and long
whiskers, ending in a flame of a tail. The body is a curve that every part is
hung off — the crest rides its top edge, the legs its belly, the head its
front — and the wave travels head to tail, so it swims rather than slides. It
rests between crossings on purpose: something moving across the background at
all times stops being noticed within about a minute.

The ☰ Menu button on either table opens a menu over the top of it rather than
leaving. It carries volume for sound effects and music, when the middle of the
table gets swept, the language switch, the relic list, and the way out. Nothing
is torn down while it is open, so the deal in progress is exactly where it was
when it closes.

**Clear played cards** — *Always*, *Each Round* or *Never*, both modes,
remembered in `settings.cfg` alongside the language and the volumes (`Prefs`).
Always leaves only the play you have to beat on the table; Each Round lets a
round pile up and sweeps it when the next one opens; Never stacks the whole
deal in the middle and clears it when the next hand is dealt.

**Each Round is the default** — the middle then holds the round being fought
over and nothing else, which is the one thing a player has to read on their
turn. The enum is appended to rather than reordered, since the choice is
written to disk as its number and has to keep meaning what it meant.

The sweep happens on the way *in* — just before the next play lands — rather
than the moment a round ends. Two reasons: a round you have just lost stays up
through the pause afterwards, where it can still be read; and a play that ends
the round by taking it does not get wiped off the table a frame after landing,
which is what a sweep keyed on `is_leading()` after the move would do.

**Swept cards are gathered, not deleted.** `TableFx.sweep_to` reparents them
into a discard layer and tweens them into a neat stack off to the left of the
middle — quick, card by card, shrinking as they go, with a little scatter and
tilt so the stack does not read as one card. They sit there until the hand
ends. Cards vanishing out of the middle is the one thing on this table that
could not happen at a real one.

Reparenting rather than waiting is what lets the play that triggered the sweep
land the same frame: the middle counts as empty the instant `sweep_to`
returns, while the cards that were in it are still travelling. A toss that is
still in the air when the sweep catches it is called off first — the tweens
carrying a card are kept on the card itself for exactly this — and its flip is
finished by hand, since killing the tween that was going to turn it over is
what would otherwise leave a card face down in the discard all deal.

Dismissing works two ways, and both close one layer at a time: Escape (or
Android back), and clicking off the thing itself — the dim around the panel
closes the menu, and the space around the relic rows closes the list back to
the menu. The panel, its buttons and the relic rows all stop the mouse, so only
a click that hit none of them counts as clicking off. The standalone relic
screen in the main menu is deliberately left inert here: that is a whole scene
rather than an overlay, and a stray tap should not navigate out of it.

The relic list is the real `Compendium` scene embedded — `embedded = true` makes
its back button emit `closed` instead of changing scene — so the encyclopedia in
a duel can never drift from the one in the main menu.

Switching language mid-game is the awkward part: a table builds its fixed labels
once at startup, and `_refresh()` only rewrites the ones that change every turn.
Each table therefore hands the menu a `relabel` callable that re-words its own
static text, which is why `battle.gd` keeps references to its title, trait chips
and stage line, and tracks `overlay_button_key` alongside the overlay button's
wording.

## Sound

`Sfx` creates a **Sfx** and a **Music** bus in code at startup rather than
shipping a bus layout resource, and every cue plays on Sfx. Levels are 0 to 1,
remembered in `user://settings.cfg` next to the language and the bankroll, and
applied as `linear_to_db` — with silence mapped to a bus *mute*, since
`linear_to_db(0)` is negative infinity and poisons the bus it reaches. That
conversion is static (`clamp_level`, `level_muted`, `level_db`) so
`_test_audio_levels` can check it without an audio device or the autoload.

**Music** is two pools of tracks in `assets/music`, crossfaded over about a
second by the `Music` autoload:

- The **Great Wall of China** arrangements — full, drums only, melody only — are
  the main menu's theme. One is picked per visit, so the menu sounds a little
  different each time you come back to it.
- **Everything else in the folder** plays at a table. The pool is read off disk
  by `Music.table_tracks()` rather than listed in the script, so dropping a song
  into `assets/music` is the whole job of adding one. Godot rewrites what a
  `res://` directory looks like once the project is exported — the editor lists
  an `.import` sidecar beside each source, an exported build lists the source
  with a `.remap` suffix — so both are trimmed back before a name is kept, and
  every survivor is checked with `ResourceLoader.exists`.

The three entry points differ on purpose:

- `play_menu()` is **idempotent**, because the menu rebuilds itself on a
  language switch and restarting the song every time somebody taps Tiếng Việt
  would be maddening.
- `play_table()` **always switches**, to a looping track and never the one just
  playing. The roguelike uses it: a new opponent is what marks a new song.
- `play_playlist()` **shuffles**, and is what the classic table uses. The track
  does not loop, and its ending is the cue for a different one — a classic
  session is one hand after another with nothing else to hang a change of song
  on. Only the player actually carrying the music picks the next track; the one
  fading out behind it finishes too, and is ignored.

Screens in between — opponent select, the reward room, the relic list — call
none of them, so a track carries across them.

Music starts at **50%** on a fresh install (`Sfx.DEFAULT_LEVELS`): it is the bed
under the game, not the game, and a first launch at full blast is the kind of
thing a player turns off rather than turns down. Both levels are on the in-game
menu's sliders.

Adding a table song is dropping the file in `assets/music`; a menu arrangement
is one more name in `MENU_TRACKS`. Looping is set on load rather than per-file
in the importer, so nothing else has to be remembered — and `_test_music_tracks`
scans the folder and fails if a name does not resolve to an importable file,
since the symptom otherwise is silence with no error.

## Debug grants

In a debug build, the Relics screen doubles as a cheat: tap a relic ten times
and it is granted — seeded into every run started from then on, and added to
the run in progress immediately. Ten taps again takes it back.

Nothing advertises it. There is no banner, and the screen is identical to the
one a player sees; the only feedback is on the row you are tapping, and it does
not appear until the fourth tap, by which point nobody is there by accident.

Grants live in `Run.granted_ids` and are not saved, so they last the session and
no longer. The whole affordance is behind `OS.is_debug_build()`, so a release
export has no way into it — which also means it will not work in a release
build you export to a phone for testing.

## Adding a relic

Same shape: one entry in `ItemDef.catalogue()` naming a rarity and the
`Ruleset` fields it sets, plus `item.<id>.name` and `item.<id>.desc` in
`scripts/loc.gd` (name, gloss, and description — the gloss English-only), and
a `_draw_<id>()` case in `scripts/item_icon.gd` laid out
inside its 96x96 box. Nothing else needs to know it exists — the reward screen,
the badge strip, the offer roll, and `_test_relic_games` all enumerate the
catalogue. If the effect needs a hook that isn't on `Ruleset` yet, add the
field there and read it in the one place that owns it (`Combo` for legality
and comparison, `TienLenGame.start()` for the deal, `scripts/battle.gd` for
money), rather than special-casing the item.

## Money (classic mode)

You sit down with **$1,000**, and the balance carries across sessions in
`user://settings.cfg`.

| Event | Amount |
| --- | --- |
| Sitting down for a hand | −$10 |
| Finishing 1st / 2nd / 3rd / 4th | +$25 / +$10 / +$5 / nothing |
| Chopping another player | +$10 |
| Being chopped | −$10 |

Hit $0 and the house stakes you a fresh $1,000 rather than leaving you
unable to sit down. Only your wallet is tracked — two AI players chopping
each other costs nobody anything. The numbers live in `scripts/run_state.gd`
(`Run`), whose payout maths is pure and covered by `_test_bankroll`.

## Language

English and Tiếng Việt, switched on the main menu and remembered in
`user://settings.cfg`. A fresh install starts in Vietnamese if the device
locale is Vietnamese, English otherwise.

Every player-visible string lives in `scripts/loc.gd` — UI, combo names
("a pair of 7s" / "đôi 7"), rejected-move messages, foe titles and traits,
every foe's barks, relic names and descriptions. `Loc` is a static class rather than an autoload so the
rules engine and the headless tests can reach it. `_check_foe_names` pins down that every
foe is named in both languages, that the two names differ, and that no foe's
title merely repeats its name back. Adding a language means a
code in `LANGUAGES`, a name in `LANGUAGE_NAMES`, and one more entry per
string; anything missing falls back to English.

## Balance

`tests/sim_battles.gd` plays hundreds of full money duels headlessly and
reports each foe's win rate, battle length, and how its traits actually show
up in play. It caught Bà Tám winning 0% of battles when "conservative" was
first implemented as "always lead the smallest combo", and it caught Anh Năm
healing his way to an 88% win rate two rungs before the boss.

It also settled the shape of round damage. Paying per card in the winning
combo was the obvious version and the wrong one: run with identical
personalities at identical bankrolls, one dealt 13 cards and one dealt 16, it moved
the big hand from a 43% win rate to 61%, because a bigger hand simply wins
more rounds. A flat 1, doubled at four cards, leaves that experiment near
even. The same run showed that halving the cards-left-in-hand damage — the
tempting way to make room for round damage — is exactly the wrong lever,
since that penalty *is* the price of holding a lot of cards.

Both seats run through their real `Ruleset`, so the sim also prices every
relic: it replays one mid-ladder duel per item and reports the drop in the
foe's win rate. Roughly, against Chú Bảy, a common is worth 5-15 points and a
legendary 10-20. Round damage forced three of them to be re-cut, and the
measurements are why:

- **Nhang Trầm** was worth 5 points. Being staked a single đồng stopped meaning
  anything once the next round you lost finished the job, so a revive now
  leaves you on 8.
- **Sảnh Rồng** was worth 2.5. A six-card straight is 2.9% of round wins; the
  threshold came down to four.
- **Cà Phê Sữa Đá** measured at *minus* 5 points — a hand of 12 lost more
  often than a hand of 13. Under round damage a smaller hand is simply short
  of answers, and the saving on cards-left-in-hand does not cover it. Tuning
  the number could not fix a backwards premise, so the item was turned around:
  it now deals you 15, which is the same double-edged bargain the big-handed
  foes make.

The suit lenses are the soft spot in the set: Áo Dài measures at 5 points and
Nón Lá at 5.5, which is light for a rare and very light for a legendary.
Collapsing every off-suit onto the single best suit held instead measures at
12.5 and 8, but it produces four cards of one suit where the colour-preserving
mapping produces two and two, so it is not what these relics are meant to do.
If they need to come up, it should be through a second effect rather than by
bending the mapping.

Two caveats the sim prints for itself: the stand-in player
is the greedy AI, which has no model of hidden information (so Kính Râm
measures as exactly zero) and rarely chooses to chop (so Nước Mía and Dao Bầu
read far lower than they play).

Foe win rates at their listed bankrolls climb roughly 20-26% on rung 1, ~40-75% on rung
2, ~55% on rung 3, ~62% on rung 4, and ~75% for the two bosses — which is
before the four relics you arrive with. A battle runs 3-4 deals, down from 6
before round damage, with four times as many moments where the bar moves.

## Running

Open the project in Godot 4.7, or:

```sh
godot --path . # run the game
godot --headless --path . --script res://tests/test_rules.gd   # rule tests
godot --headless --path . --script res://tests/sim_battles.gd  # balance sim
godot --path . res://tests/screenshot.tscn  # UI screenshots + icon sheet
godot --headless --path . --script res://tests/make_icons.gd   # app icon PNGs
```

## Shipping to the App Store

`tests/make_icons.gd` and `tests/store_shots.gd` produce the two sets of
artwork Apple asks for; everything else is entered by hand in Xcode or App
Store Connect.

```sh
godot --headless --path . --script res://tests/make_icons.gd        # icon PNGs
godot --resolution 2868x1320 --path . res://tests/store_shots.tscn  # 6.9" iPhone
godot --resolution 2778x1284 --path . res://tests/store_shots.tscn  # 6.5" iPhone
godot --resolution 2732x2048 --path . res://tests/store_shots.tscn  # 13" iPad
```

Screenshots land in `user://store/<width>x<height>/`, five per size, numbered in
the order they should appear in the listing: the four-hand choice first, since
that is the thing this game has that other card games do not, then a duel, the
four-player table, the relic offer, and the menu. Every shot is landscape,
because the game is. `store_shots.gd` is a tool, not a test — it asserts
nothing and fails nothing; it exists to sell the game, where `screenshot.gd`
exists to catch layout bugs.

Note that it reads the window's real pixel size from `DisplayServer` rather
than `get_viewport().get_visible_rect()`: under `canvas_items` stretch the
latter returns design units, so a 2868x1320 render would file itself under
1564x720 and the next size along would overwrite it.

## The app icon

`assets/icon/icon_master.svg` is the artwork, with size variants beside it, and
`assets/icon/png/` holds it rasterised to every slot Xcode's asset catalogue
asks for — 1024 down to 20. `tests/make_icons.gd` regenerates the PNGs from the
master through Godot's own ThorVG, so no separate SVG converter has to be
installed; it drops the alpha channel on the way out, since the App Store
rejects an icon that has one and the art is full-bleed opaque anyway.

The icon deliberately **carries no name**. It is the game's own marks: lacquer
red under the lamp, the Đông Sơn drum sun off the table felt, three fanned
cards in ♠♥♦ so both card colours show, and the doubled gold hairline the
lacquerwork uses everywhere else. The gold border is drawn *inside* the
artwork, so it survives iOS's corner mask whether or not the icon is shown
squared off.

## Layout

- `scripts/card.gd` — card + Tiến Lên ordering (3 low … 2 high; ♠♣♦♥)
- `scripts/combo.gd` — combo identification and beat/chop rules, optionally
  bent by a `Ruleset`
- `scripts/ruleset.gd` — one seat's version of the rules: legality,
  comparison, the deal, and money, all in one mergeable object
- `scripts/item.gd` — `ItemDef`: the relic catalogue and the offer roll
- `scripts/ink.gd` — `Ink`: the cel-shaded marks shared by every drawn thing
- `scripts/item_icon.gd` — one code-drawn icon per relic
- `scripts/tien_len_game.gd` — match state machine (2–4 players, uneven
  hand sizes, the undealt stock, chop metadata for damage)
- `scripts/ai_controller.gd` — greedy AI, tuned by foe personality traits
- `scripts/battle_rules.gd` — the duel's money (per round, per chop, per card
  left in hand), shared with the sim
- `scripts/foe.gd` — foe roster: looks, playstyle, and trash talk
- `scripts/foe_portrait.gd` — cel-shaded portraits with four damage stages
- `scripts/speech_bubble.gd` — cel-shaded bark bubble
- `scripts/audio.gd` — `Sfx` autoload mapping game events to the sound library
- `scripts/run_state.gd` — `Run` autoload: the five-rung ladder, the carried purse,
  the relics in hand, and the classic bankroll
- `scripts/loc.gd` — `Loc`: every player-visible string, English + Vietnamese
- `scripts/card_visual.gd` — code-drawn card faces and backs (no image assets)
- `scripts/hand_choice.gd` — the four face-down hands and the paid reveals
- `scripts/result_card.gd` — `ResultCard`: the end-of-hand and end-of-deal card
- `scripts/table_backdrop.gd` — the lit felt, the ruled line, and the drum
- `shaders/felt.gdshader` — the lamp over the table and the cloth under it
- `scripts/table_fx.gd` — opponent card fans (face down, or face up under
  Kính Râm), the toss into the middle, and the sweep out of it
- `scripts/item_card.gd` — a relic as a card, full size or as a HUD badge
- `scripts/compendium.gd` — the menu's relic list: every icon, gloss and effect
- `scripts/music.gd` — `Music` autoload: the track pools, the crossfade, and
  the classic table's shuffle
- `scripts/layout.gd` — `Layout`: where the edges of the screen actually are
- `scripts/scene_fx.gd` — `SceneFx` autoload: the card-throw screen transition
- `scripts/festive_fx.gd` — lanterns and hoa mai petals for the red screens
- `scripts/hand_drag.gd` — dragging cards around inside your own hand
- `scripts/prefs.gd` — `Prefs`: table settings remembered between sessions
- `scripts/menu_dragon.gd` — the dragon silhouette that crosses the main menu
- `scripts/money_burst.gd` — the coins-and-banknotes explosion a chop sets off
- `scripts/pass_cross.gd` — the red X stamped over a seat that has folded
- `scripts/menu.gd` / `opponent_select.gd` / `reward_select.gd` /
  `classic_table.gd` / `battle.gd`
- `tests/test_rules.gd` — headless rule, engine, personality, relic, and
  ladder tests
