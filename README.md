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

When a deal ends, **the table is left readable for three seconds before the
result card comes up**. A deal ends on a play, and that play is the thing you
most want to look at: what beat you, what you were caught holding, what the
chop landed on. The card covers the middle of the table, so raising it the
instant the deal resolves put the winning play on screen for one frame and then
took it away. A tap anywhere skips the wait, and while it is running the only
thing taking input is the skip layer, so a tap meant for the card's button
cannot land on a card in the hand behind it. `_capture_deal_over` measures the
pause and reports it, since a pause that quietly stopped happening is exactly
the kind of thing nobody notices in a screenshot.

### Reading the two purse bars

Both bars are drawn against the **same denominator** — your starting stake plus
the foe's bankroll, every đồng on the table. A duel is zero-sum and money only
ever moves between the two seats, so two bars against one total read as one pot
being divided, which is what is happening. Clean someone out and their rival's
bar is full and theirs is empty, at the same instant the duel ends. Drawn
against their own starting stakes instead, both would look full at once and say
nothing.

**Each bar also carries the mark it started the duel at.** The current split on
its own tells you who is ahead but not which way the money has been moving: by
the fourth rung you sit down carrying three bankrolls, so a bar two-thirds full
may mean you are winning comfortably or that you have just been taken for a
third of everything. Past the mark the fill brightens — that stretch is money
won off the other seat — and back from it a dim ghost shows the ground given
up. The mark is pinned per *duel*, not per deal: the question these bars answer
is whether this opponent is going well or badly.

Three details that were not optional once there were three tones on one bar:
the segments are rounded only on the ends that are the bar's own ends, or it
read as two pills laid end to end rather than one bar with a division in it;
the mark is pale rather than inked, because it has to be legible over the
near-black empty track as well as over a fill; and the amount is drawn with an
outline, because it sits centred over the bar and the mark lands right behind
the digits exactly when the split is interesting.

Because a duel is zero-sum, **a bad duel costs you stake rather than ending
you**: the run erodes instead of stopping dead, which is a far better shape
than an HP bar that empties. Your purse carries between duels and is never
topped up — what you walk out with is exactly what you took off the table. Win
and you choose one of three relics, then face the next rung. Get cleaned out and
the run is over. Every win also earns five turns of the gacha machine (below)
before the relic choice.

**The purses do not touch.** A run's purse is the roguelike's health bar that
happens to be denominated in money: the mode stakes it, it moves between the two
seats as the duel is played, and it ends when the run does. It is never paid
into the classic wallet, and the classic wallet never pays for it — **a run
costs nothing to start**, so a player who has been unlucky at the classic table
is never locked out of the ladder.

What *does* cross is a **prize for beating an opponent**, paid in classic money
(see [Money](#money-classic-mode)). That direction is deliberate and it is the
only one: with no house allowance, the ladder is how a broke player earns a seat
back at the classic table. The run's own purse is untouched by it.

That was not always true, and the way it broke is instructive. A run was bought
out of the wallet for $150 and paid its surviving purse back in — fine at a
$1,000 starting bank, and broken the moment that came down to $200: the second
buy-in would have emptied the wallet, which triggered the broke allowance
instead, so **every run after the first one was free** and the second one
actually paid $50. `_test_bankroll` pins both directions now.

**Finishing a whole run pays a bonus**: three free turns of the machine, and a
relic chosen *before* the next run's first opponent. Beating the fifth opponent
is the longest thing in the game, and this is what it is worth — spins and a
relic, not a number in the other mode's wallet.
The head start is saved, because somebody who wins a run and then puts the game
down should still have it in the morning; the reward screen serves both moments
with a different heading, since a head start counting opponents down would be
counting a ladder nobody has climbed yet, and `take_item` knows not to advance
a ladder that is still on its first rung.

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
on its centre, so a card drawn at scale `s` covers `position + CARD_SIZE/2 ±
CARD_SIZE * s/2`; treating `position` as the visible top-left drops the row and
hangs it out of the bottom of the strip it is meant to sit in.

**The rows are measured, not fixed** (`_measure()`). How big a card can be here
is decided by the room between the title and the reveal buttons, and that is
not the same on a phone as at the design size — a fixed 100px row left a strip
of dead space above the buttons and thirteen cards too small to read. Three
bounds, smallest wins: four rows have to fit the height available (this is the
one that binds), the fan has to stop clear of the suit-count column on the
right, and `MAX_SCALE` stops it reaching back over the radio mark on the left.

**Both sets of four look like buttons, because both are.** A candidate row used
to be a `flat` Button with a lit strip that only appeared *after* you picked
it — so until you had already clicked one, nothing on the screen said a row
could be clicked at all. The whole state now lives on the button's own
styleboxes, which hands over hover and press feedback for free and makes a
resting row a raised tray the hand is lying on. The four reveals were on the
engine's default theme, which read as a button on the old black scrim and as a
hole cut in the cloth once this screen got felt; they use the table's own
secondary button now, which also brings a real disabled face — a reveal you
have bought and one you cannot afford are both disabled, and the flat default
gave no sign of either.

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

### The gacha machine

Clean an opponent out and you get five turns of a crank before anything else
happens: a sweet-shop capsule machine loaded with fifty-two capsules, one per
card. The five you draw make a poker hand, and the hand pays **into the
classic-mode wallet** — the one thing that outlives a run, so a good draw is
money you keep however the next rung goes.

**The screen says so three times over**, because this is the one place the two
economies meet and it turns up mid-run, when the player is also holding a duel
purse. Those are two different piles of money: the purse is the ladder's health
bar and dies with the run, and this is not. So the subtitle names the *Classic*
wallet rather than "your wallet", the payout line names it again, and a running
**Classic wallet: $195** sits under the sockets from the moment the screen
opens — present before the hand is paid, so the player is already looking at
the number when it moves rather than meeting it and its change at once.

`_test_lottery_subtitle_fits` guards the subtitle against the lanterns hanging
78 units in from each edge. It is a plain unwrapped Label, so a sentence one
clause too long does not stack — it runs out underneath them at both ends,
which is a thing only a capture shows and only if somebody looks. Saying "money
you keep whatever happens on the next rung" in the subtitle is exactly what
tipped it over, which is why that idea lives in the running balance instead.

**You do the drawing, five times.** Nothing happens until *Turn the Crank* is
pressed — which plays `gacha_pull`, the crank's own sound, pitched up a hair per
turn so the fifth is audibly the last. Nothing else plays on that press: a UI
blip stacked in front of the mechanism is two sounds for one button. Then a
capsule falls through the neck, disappears inside the cabinet, and
arrives at the door. The same button becomes *Onward* once the fifth is down.
The draw is the one thing on the ladder handed to you rather than played for,
and taking it yourself five times is the difference between being given a prize
and watching a cutscene about one.

| Hand | Pays | Comes up |
| --- | --- | --- |
| High card | $10 | 50.0% |
| Pair | $25 | 42.3% |
| Two pair | $60 | 4.8% |
| Three of a kind | $120 | 2.1% |
| Straight | $250 | 0.40% |
| Flush | $400 | 0.21% |
| Full house | $700 | 0.14% |
| Four of a kind | $2,000 | 0.025% |
| Straight flush | $5,000 | 0.002% |
| Royal flush | $10,000 | 0.0003% |

Weighted by those frequencies the machine is worth about **$24.45 a duel** —
roughly one classic first place, with a jackpot on top that almost nobody will
ever see. `_test_poker` measures it over 20,000 draws and fails outside a wide
band, which is a guard against somebody adding a zero rather than a pin on the
figure. The bottom rung pays rather than paying nothing on purpose: half of all
draws are high card, and a mini-game that hands you nothing half the time is
one people learn to skip.

**Poker is not Tiến Lên, and the 2 is where the two disagree.** This game ranks
it above the ace; poker puts it at the bottom. `Poker.poker_rank()` is the only
place either ordering knows about the other, and it is what makes 2-3-4-5-6 a
straight and A-2-3-4-5 the wheel — a straight to the 5, so in one suit it is a
straight flush and never the royal that pays five figures. `_test_poker` pins
all three, along with the K-A-2-3-4 wrap a naive high-minus-low check lets
through.

The five cards are settled the moment the screen opens (`Poker.draw_five`) and
everything after is animation over a result that has already happened, so the
payout is testable without a scene tree and a dropped frame cannot change what
you won.

The machine is drawn in code like everything else — ink outlines and flat
pastel fills, `Ink` doing the marks — down to the iced top with drips and the
hundreds-and-thousands scattered over it, which is what makes it a sweet-shop
cabinet rather than a vending machine. The capsules are two-tone shells with
the card printed across the seam: rank above, pip below, and the seam drawn as
two short bars at the sides so the middle stays clear. A rank glyph hangs
upward off its baseline, so the gap beneath it is the only place the pip can
go.

The heap is a toy physics sim, and the two numbers that matter are the ones
that decide how full the globe looks. 52 capsules come to about 70% of its
area and a settled heap packs at roughly 78%, so they stand nearly to the neck
— a machine packed full, with just enough room left for the heap to settle into
the hole when one is taken out of the middle of it.

Three things it needs to not look dead or broken:

- **Verlet integration, not Euler.** Velocity is read back off how far a
  capsule actually moved after the constraints have had their say. A capsule
  held up by the two beneath it then has its velocity cancelled by the thing
  that held it, instead of accumulating downwards until it tunnels through.
- **Four relaxation passes, not one.** At this density every capsule rests on
  two others, which is exactly the case a single pass cannot resolve — one pass
  leaves the heap visibly overlapping itself.
- **A jostle on every crank turn.** Take a capsule out of the middle of a
  packed globe and the hole stays there, because everything around it is
  already at rest. Shaking the cabinet is what closes it.

And two more that only showed up on a still:

- **The wall constraint has to run last on every relaxation pass.** Separating a
  pair is exactly what pushes the outer one through the glass, so a pass ending
  on the separation left capsules outside the globe until the next frame caught
  them — which is the frame you photograph. There is a `GLASS_PAD` on top of
  that, because a capsule stopped at exactly `GLOBE_R - CAPSULE_R` has its edge
  on the centre line of the glass's own 4px stroke and its ink outline half a
  pixel beyond it.
- **The capsules turn.** Fifty-two of them printed the right way up read as a
  pattern rather than as loose objects in a jar; the whole point of a heap is
  that nothing in it agreed on which way was up. Turning is not simulated off
  the contacts — that would be a rigid-body solver for a decoration — it is set
  spinning by the shake a crank turn gives the cabinet and slows down again.
  Drawn about the origin under `draw_set_transform`, since `draw_string` cannot
  be rotated any other way, with the shine left unrotated because it is the lamp
  over the machine rather than a mark on the capsule. A drawn capsule rights
  itself over its trip to the tray: the five have to be read as a poker hand at
  the end, and a hand printed at five angles is not one.

The machine is layered the way it is built: you look *through* the front of the
glass at the capsules, so the rim and the shine are drawn in front of the heap,
and the lid and collar — the two white panels holding the globe in — in front of
that again. Drawing the whole globe first, which is the obvious order, put the
heap over the two panels that are supposed to be holding it. A capsule on its
way down to the neck is still inside the globe, so it is drawn with the heap
rather than with the ones that have come out.

The five sockets are drawn every frame, filled or not, and wider than the
capsules that land in them — a ring exactly the capsule's size is covered the
instant it is used, which is the same as not drawing it at all.

The heap sleeps when nothing is moving, so the idle state — which is what this
screen spends most of its time in — costs nothing per frame. Spin counts
towards being awake, measured as the speed of a capsule's rim so it is in the
same units as everything else.

## Relics

Twenty-two of them, offered three at a time after each win, weighted
60/42/30/16/10 by rarity and never repeating one you already hold.

| Relic | | Rarity | Effect | Unlocked by |
| --- | --- | --- | --- | --- |
| Nước Mía | sugarcane juice | Common | Your chops can beat any single card, not just a 2 | — |
| Bánh Mì | a baguette sandwich | Common | You take $10 from your opponent at the start of every deal | — |
| Cà Phê Sữa Đá | iced coffee over condensed milk | Common | You are dealt 15 cards instead of 13 | — |
| Ghế Nhựa | the little plastic stool | Common | You sit down with $15 more | — |
| Cầu Khỉ | the monkey bridge, one plank over a canal | Common | A 2 may end your straights. Each straight you play pays $2 into your Classic wallet | — |
| Đôi Đũa | a pair of chopsticks | Common | Every pair you play pays $1 into your Classic wallet | — |
| Nước Mắm | the secret ingredient | Uncommon | You take $5 from your opponent every deal, and $10 more for each deal you win | Play 3 hands in Classic mode |
| Bột Ngọt | the other secret ingredient | Uncommon | Every round you win is worth $5 more | Win a hand in Classic mode |
| Gương | a mirror, hung behind your opponent | Uncommon | You can see your opponent's two lowest cards | Buy a reveal while choosing a hand |
| Phở Đặc Biệt | the phở bowl with everything in it | Uncommon | You sit down with $30 more | Clean out an opponent in Roguelike mode |
| Bánh Pía | a flaky layered pastry filled with durian | Rare | Each card left in your hand at the end of a deal costs you $5 less | Win 3 hands in Classic mode |
| Lì Xì | a red envelope of lucky money | Rare | Your straights of four or more cards may skip one rank: 3-4-5-7 counts as a straight | Chop an opponent in Classic mode |
| Áo Dài | the long split-panel dress | Rare | Every spade in your hand counts as a heart — same rank, but the highest suit | Clean out 3 opponents in Roguelike mode |
| Nhang Trầm | incense | Rare | The first time you run out of money, you are staked $40 and play on. Once a duel | Play 15 hands in Classic mode |
| Dao Bầu | a heavy kitchen cleaver | Rare | Each chop you land takes an extra $20 from your opponent | Chop an opponent in Roguelike mode |
| Sinh Tố Bơ | an avocado smoothie over condensed milk | Epic | One of the four hands is turned face up before you choose, free | Buy 10 reveals while choosing a hand |
| Bánh Tét | a sticky-rice log wrapped in banana leaf | Epic | One deal in four, your 3s, 4s, 5s and 6s are dealt as Js, Qs, Ks and As | Win 10 hands in Classic mode |
| Đèn Ông Sao | the paper star lantern children carry at mid-autumn | Epic | You lead every deal, and never have to start with your lowest card | Chop an opponent 10 times in Roguelike mode |
| Trứng Vịt Lộn | the duck egg with something inside | Epic | The lowest card in your hand is dealt as an ace | Chop an opponent 8 times in Classic mode |
| Nón Lá | a palm-leaf hat | Legendary | Every spade and club in your hand counts as a heart — half your hand, in the highest suit | Defeat all opponents in Roguelike mode |
| Bầu | a gourd from the bầu cua game | Legendary | Every deal you win takes an extra $30 from your opponent | Collect 8 different relics |
| Sảnh Rồng | a dragon straight | Legendary | Your straights of four or more cards beat any single card or pair | Win 25 hands in Classic mode |
| Kính Mát | a pair of sunglasses | Legendary | You can see your opponent's two highest cards | Win a hand with a single 3 of spades, in either mode |
| Gà Chọi | a fighting rooster | Legendary | Your triples beat a single 2 | Clean out 12 opponents in Roguelike mode |

### Finding one

A relic is a thing you find, and the list on the menu is a list of things to
go and find rather than a manual. Each one is in one of three states, and the
compendium says exactly as much as the state allows:

| State | Shows |
| --- | --- |
| Undiscovered | the picture and the rarity, and the one line saying what to go and do — not even the name |
| Unlocked | the name, and "find this relic by defeating opponents" — the condition is met, so it can now turn up in an offer |
| Obtained | everything: name, gloss, and what it actually does |

**Uncommon** is the tier of small constant advantages — things already on the
table that you stop noticing — and its conditions are the first few things any
player does, so it opens up almost immediately and the reward screen stops
being four commons on rung two.

Commons carry no condition and are in the pool from the first duel; they
still have to be won once before the compendium will explain them. Everything
above common names a counter in `Progress` and a number, and is not offered
until that counter reaches it. The conditions are spread deliberately across
both modes — the classic table opens as many relics as the ladder does — so a
player who only ever sits down at one of them still finds things.

`scripts/progress.gd` owns both halves: the counters (classic wins, hands,
chops in either mode, duels and runs won, reveals bought, relics collected,
and going out on the 3♠ alone) and the set of ids actually taken off a reward
screen. It is static like `Loc` and `Prefs`, shares `user://settings.cfg` with
the language and the bankroll, and the tests set `sandboxed` so a run of them
can neither read the player's progress nor unlock relics behind their back.
`_test_relic_unlocks` walks one relic through all three states, checks that a
fresh save can be offered nothing but commons, and pins every condition to a
counter something actually books — a typo there would be a relic nothing could
ever unlock.

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
| Áo Dài | ♠ → ♥ | |
| Nón Lá | ♠ → ♥ | ♣ → ♥ |

**Both promote into hearts — the top suit — and that is the whole design.** The
first version mapped off-suits onto a set of read-suits one-for-one in suit
order, which meant every promotion landed on the *adjacent* suit: ♠→♣, ♦→♥.
An adjacent promotion can never win a standoff, because no suit sits between
the two. Measured over 4,000 deals, Áo Dài rescued **0 of 3,077** same-rank
standoffs — a relic that was provably, not merely weakly, doing nothing. Nón
Lá only worked at all because its mappings happened to skip a suit.

Promoting to hearts outright fixes it, and the sim agrees: Áo Dài moved from
**+1.0** (worse than no relic, i.e. noise) to **−3.5**, and Nón Lá from −2.5 to
**−7.0**, which finally puts a legendary above the rares around it.

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
each card overlaps the next. There is no edge stripe or arrow: the card has to
read as a card first. Bánh Tét is the one relic that moves a rank rather than
a suit, and it crosses out the rank it lifted in the same corner, ahead of the
suit mark — a card that had both done to it shows both.

Each relic has its own icon, drawn in code in the same cel-shaded style as
the foe portraits — the game has no image assets and a relic is not going to
be the exception. `scripts/ink.gd` owns the look (ink outlines, flat fills,
hard shadows) and both `FoePortrait` and `ItemIcon` draw through it, so a
relic and a face are made of the same marks. `ItemIcon.ART` maps every relic to the
method that draws it, and the catalogue test fails a relic that has no entry —
so a new item cannot ship as a blank circle.

**One list, not two.** That used to be an array saying which relics had art
beside a `match` saying how to draw each, and the two fell out of step the
moment a relic was added to one and not the other: the test passed, because it
asked the array, and the icon rendered as nothing, because the match had no
case. A dictionary cannot disagree with itself, and the test now also checks
that every method it names exists.

Two of the twenty-two were redrawn after being rendered next to the rest:
Bột Ngọt was a cream sachet spilling cream crystals, one pale shape where the
spill is the half that names the relic, so the packet became foil; and Bầu's
first silhouette was a teardrop rather than the pinched double-lobed calabash
the name means.

Every figure a relic quotes in its description is checked against the figure
it actually delivers (`_test_item_text_matches_effect`). The duel moved from
hit points to money and the numbers were rescaled without the text following,
so six of these cards spent a while promising HP the game no longer had —
nothing else catches that, because a description is never read by the code.

**And every description has to fit the two lines the compendium row gives
it** (`_test_relic_text_fits`), measured with the real font at the real size
against the narrowest 16:9 row. The row clips rather than grows, so a
description one word too long does not push the layout about — it silently
loses its last line, which is the line that says what the relic pays you.

The effect lines are written to be read once and understood: one sentence
where one will do, the money as a figure rather than a proportion, and "your"
on anything one-sided, since a relic bends *your* legal moves and not the
table's. They say what happens rather than how it is implemented — Nhang Trầm
is "the first time you run out of money, you are staked $40 and play on",
not a credit line with an amount.

### Asking what a relic does

The relics you are carrying sit as badges down the left edge of a duel, and
**tapping one opens the card that explains it** — rarity, icon, name, what the
name means, and the effect line, over a dimmed table. Tapping anywhere puts it
away.

A relic taken four opponents ago is one a player has stopped remembering the
details of, and the badge is already the thing they look at when they try. It
was a name and nothing else, which is enough to recognise and not enough to
answer the question.

The card is the same `ItemCard` the reward screen deals out, with
`actionable = false`: no Take button, because it is explaining something you
already own, and the effect line gets that room instead. Two things the dim
layer behind it is doing besides dimming — it swallows the tap that dismisses
the card, so asking a question can never land a move on the hand or on Play,
and it is why a relic card left open is closed when a deal ends rather than
being left sitting over the result card.

`_check_relic_inspect` drives it through the badge's own input rather than by
calling the handler, so a badge that draws but ignores the mouse — which is
what every badge did before this — fails there. It checks for the effect
*text*: a card that comes up naming the wrong relic, or naming the right one
with no explanation under it, is the failure worth catching, because the
sentence is the whole point.

### Reading the other seat's hand

Two relics do it, **two cards each, from opposite ends of a sorted hand**.
Gương catches the lowest two in the mirror — the honest thing for a mirror
behind a seat to see, and the cards a player can least act on. Kính Mát reads
the top two over the rim of the glasses: whether the other seat is still
holding a 2 is most of what there is to know, and it is the end of the hand you
can actually play against.

Same number of cards, and the rarities are a tier apart on purpose. What
separates them is *which* end, and the end Kính Mát reads is the one that
decides rounds.

They stack. A seat holding both reads each end and still cannot see the middle,
which is why the fan is built **slot by slot** rather than as a run of cards
from one end.

Both were larger first. Kính Mát showed the *whole* hand, which was not a relic
— it was the end of the guessing: every round after it became arithmetic, and
the duel it was won in stopped being a game. Gương showed four, which turned
out to be most of the low end of a thirteen-card hand and not much of a
question either.

**Two of the commons pay classic money.** Cầu Khỉ and Đôi Đũa are the only
relics that reach outside the duel: the run's purse is untouched and the money
goes to the wallet the other table draws on, so a run you go on to lose can
still have been worth something. Paid on the *play* rather than on the round —
what they reward is putting the shape down, and a round you lose with a straight
was still a straight.

Cầu Khỉ also lets a 2 finish a straight, which is the one card runs are barred
from everywhere else. Only at the top of the run, because A-2 is where a 2
actually falls in the ladder.

**A relic that compiles is not a relic that works.** Gương shipped doing
nothing: it set `see_foe_cards` correctly, `battle.gd` sliced that many of the
foe's lowest cards correctly and handed them to the fan, and the fan turned
none of them over — `TableFx.build_fan` decided face-up-ness for the fan as a
whole (`reveal.size() == count`), which was true for Kính Mát when it showed
everything and never for a partial reveal. Every test it had passed, because
they all stopped at the `Ruleset`.

So both are now checked doing the thing they claim: `_check_foe_reveal` in the
screenshot harness plays a real duel wearing one or both, and compares the fan
**card by card** against what should be readable — identity, not a count, since
a mirror showing cards the foe is not holding would pass a count and be a worse
bug than showing none. It lives there rather than in `test_rules.gd`
because the fan is built from `CardVisual`, and a `--script` run has no
autoloads for that to compile against.

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

### The personality dials

A "personality" is not a branch in the AI. `AIController` plays one greedy
game — lead the biggest combo that includes your lowest card, answer with the
cheapest thing that wins, save chops for last — and every foe is that game with
some of the following knobs on `FoeDef` turned up. Every combination still
plays strictly legal Tiến Lên.

| Dial | Type | What it does at the table |
| --- | --- | --- |
| `conservative` | flag | Leads the smallest shape it can and will not spend a 2 or a bomb on a round it could afford to lose. Lead high and it folds. |
| `dumps_trash` | flag | Weights shape size over card value, so it sheds many low cards at once and empties fast. |
| `pass_chance` | 0–1 | Sits out rounds it could have won, but only when winning would cost it a Queen or better. |
| `face_hoard` | 0–1 | Reluctance to spend J and up, priced per face card in the play. |
| `shape_care` | 0–1 | Reluctance to orphan a pair or take a card out of a run of three or more, even when that is the cheapest shed. |
| `risk` | 0–1 | Taste for playing loud: leads higher than it needs to and sometimes answers with the biggest thing it holds instead of the cheapest. |
| `blunder_chance` | 0–1 | Throws the calculation out for one turn and takes a random legal move instead. |

Two rules keep the dials from reading as broken rather than human. **Bombs are
excluded from random and showy picks** — spending one on a whim is not a
misplay a person makes. And **caution switches off in the endgame**: at six
cards or fewer, hoarding costs more than it saves (cards still in hand when a
deal ends cost money, and a hoarded 2 costs the most), so even the most patient
foe plays to empty its hand.

Speed is the eighth dial and the one you feel first: `think_min`/`think_max`
give each seat its own pause before playing, so a table of AIs does not tick
like one metronome.

The ladder foes layer the rule-benders on top of these — `hand_size`,
`cheat_swaps`, and the foe relics (`bomb_any_single`, `chop_stake_bonus`,
`income_per_deal`, `deal_win_bonus`). The classic table's regulars use the
dials and nothing else; see [The classic table](#the-classic-table).

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

### Running the harness

It is split into groups, and `SHOTS` picks a subset — working on one screen
should not cost a run of every other one:

| Group | What it covers | Roughly |
| --- | --- | --- |
| `menu` | the main menu, both states of the redeem button | 0.5s |
| `relics` | the compendium's pages, the debug grant, the list in Vietnamese | 6s |
| `stats` | the lifetime record, in both languages, off a staged save | 0.6s |
| `rogue` | the ladder end to end: picker, duel, gacha machine, reward room | 25s |
| `music` | what plays where — printed findings, no captures | 0.6s |
| `classic` | the four-player table, its two selection checks, fast forward | 9s |
| `art` | icon sheet, lens cards, portrait damage stages — no scenes played | 1.3s |

```sh
godot --fixed-fps 60 --path . res://tests/screenshot.tscn                # all of it
SHOTS=classic godot --fixed-fps 60 --path . res://tests/screenshot.tscn  # one group
SHOTS=menu,relics godot --fixed-fps 60 --path . res://tests/screenshot.tscn
```

Every group is safe on its own: progress is staged and the language reset
before any of them run, and each starts whatever run or table state it needs
rather than inheriting it from the group before. Each prints what it took, and
the run prints a total.

Three things make the whole run about 45 seconds rather than three minutes.
**`--fixed-fps`** tells Godot to stop synchronising to real time; the harness
**disables vsync and uncaps `max_fps`**, which is also what stops the run
crawling when its window is not the focused one — a throttled presentation rate
is exactly what an unfocused window gets. And it runs at **2x
`Engine.time_scale`**, because most of a capture run is the game waiting on
purpose: a foe taking its time over a play, a card settling, the beat before a
result card comes up. That compresses all of it at once without a line of
test-only code in the game. It is kept at 2 rather than higher because the
gacha heap clamps its own step to 1/30s, so past about there it gets fewer
steps per second of game time than it wants and the capsules are caught
mid-settle. `SHOTS_SPEED=1` puts it back to real time when a capture looks like
it was photographed too early.

### Safe-area runs

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

- **Whose turn it is, at the classic table.** The three opponents each get a
  border round their name box; you have no box, and a fourth one down there
  would crowd the hand for the sake of a card count you can already see by
  looking at your cards. So your turn is said by *the bottom of the screen
  going warm* — light spilling up off the edge, breathing at half a hertz.
  It was a 10px gold bar with a drop shadow, and the trouble with a bar is that
  it is a **thing**: a hard-edged object lying across the table that the eye
  reads as another piece of furniture and then learns to ignore. What it is
  trying to say is not "here is a bar", it is "you".

  `TurnGlow` draws it as a grid of gradient-shaded quads rather than blurring
  anything — Godot has no cheap blur that does not want a backbuffer, and a
  blur of a hard bar is still a hard bar underneath. `draw_polygon` interpolates
  between per-vertex colours, so a grid of quads with the alpha computed at
  each corner *is* a smooth two-dimensional falloff, with no shader and no
  texture. It falls off as a power curve going up (so the light stays low and
  reads as spill rather than as a painted gradient) and on cosine shoulders at
  each end (a linear taper leaves a visible crease where it meets the flat
  middle). It spans the full drawable width rather than the safe area: light
  going off the edge of the screen has no business stopping short of it, and
  it is neither a tap target nor something to read. It sits behind the hand,
  so the cards are lit rather than washed over, and it restarts on each turn
  instead of resuming, so every turn opens at the same brightness.

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

The main menu offers the two modes, each with its sub-items hung off it as
indented rows joined to it by a bracket: the relic list under the roguelike,
since relics only exist there, and the earned spins and the shop under Classic,
since classic wins earn the one and classic money pays for the other. They read
as part of a mode rather than as more things to do.

Classic has two of them, and **one trunk branches to both** rather than each
carrying its own bracket. Every sub-item drops its line from the mode button's
own bottom edge, so the verticals are the same line and the result looks like a
branch. Hung off the item above it instead, the second bracket started below the
first and the two came out disconnected — which reads as two unrelated things
rather than as one mode's list. The rows themselves are measured off the button
heights (`SUB_TOP`, `SUB_STEP`) rather than picked, so adding a third would land
clear of the second instead of on top of it, and the language picker below the
block is positioned from the last row for the same reason.

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

It is **paged, eight relics at a time**, two columns of four with a chevron on
each side and a page count under it. It scrolled once — the catalogue outgrew a
single screen when the epics landed — and paging is the better trade: eight
rows on a page can be half again as tall as fourteen crammed into a scroll
view, which buys the icon its full 96px back and gives the effect line room for
two comfortable lines. A page also holds still, and a list you are comparing
two relics in should not move under the finger doing the comparing. The
chevrons wrap rather than stopping at the ends; three pages is few enough that
a dead control on the first and last is worse than a list that comes round
again.

The grid measures itself against the live area rather than sitting at fixed
sizes, because the room left between the chevrons is not the same at the design
size as it is on a phone with a camera cutout down one side. `_page` is a field
rather than a local so it survives the full rebuild every debug tap causes.

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
- **Rice_and_Wine** is the victory song, and is held back from the table pool
  along with the menu arrangements. It starts the moment an opponent is cleaned
  out and carries through the gacha machine, the relic choice and the opponent
  picker; the next duel's `play_table()` is what ends it. A track that means
  "you won" must not also be able to come up in the middle of a duel you are
  losing, which is what `Music.reserved_tracks()` and `_test_music_tracks`
  between them enforce.
- **Everything else in the folder** plays at a table. The pool is read off disk
  by `Music.table_tracks()` rather than listed in the script, so dropping a song
  into `assets/music` is the whole job of adding one. Godot rewrites what a
  `res://` directory looks like once the project is exported — the editor lists
  an `.import` sidecar beside each source, an exported build lists the source
  with a `.remap` suffix — so both are trimmed back before a name is kept, and
  every survivor is checked with `ResourceLoader.exists`.

The four entry points differ on purpose:

- `play_menu()` is **idempotent**, because the menu rebuilds itself on a
  language switch and restarting the song every time somebody taps Tiếng Việt
  would be maddening.
- `play_table()` **always switches**, to a looping track and never the one just
  playing. The roguelike uses it: a new opponent is what marks a new song.
- `play_victory()` is **one named track**, and idempotent for the same reason
  `play_menu()` is: the end of a duel can be booked more than once as the last
  play resolves, and restarting the song under the victory card would be
  audible.
- `play_playlist()` **shuffles**, and is what the classic table uses. The track
  does not loop, and its ending is the cue for a different one — a classic
  session is one hand after another with nothing else to hang a change of song
  on. Only the player actually carrying the music picks the next track; the one
  fading out behind it finishes too, and is ignored.

Screens in between — opponent select, the reward room, the relic list, the
gacha machine — call none of them, so a track carries across them. That is what
lets the victory song be a single call at the moment the duel is won and still
be playing three screens later.

Music starts at **50%** on a fresh install (`Sfx.DEFAULT_LEVELS`): it is the bed
under the game, not the game, and a first launch at full blast is the kind of
thing a player turns off rather than turns down. Both levels are on the in-game
menu's sliders.

Adding a table song is dropping the file in `assets/music`; a menu arrangement
is one more name in `MENU_TRACKS`, and anything else that belongs to a moment
rather than to the table has to be named in `reserved_tracks()` or it stays in
the shuffle as well. Looping is set on load rather than per-file
in the importer, so nothing else has to be remembered — and `_test_music_tracks`
scans the folder and fails if a name does not resolve to an importable file,
since the symptom otherwise is silence with no error.

## Debug grants

In a debug build, the Relics screen doubles as a cheat: tap a relic twenty
times and it is granted — seeded into every run started from then on, and
added to the run in progress immediately. Twenty taps again takes it back.

Nothing advertises it. There is no banner, and the screen is identical to the
one a player sees; the only feedback is on the row you are tapping, and it does
not appear until the eighteenth tap, by which point nobody is there by
accident.

Grants live in `Run.granted_ids` and are not saved, so they last the session and
no longer. The whole affordance is behind `OS.is_debug_build()`, so a release
export has no way into it — which also means it will not work in a release
build you export to a phone for testing.

## Adding a relic

Same shape: one entry in `ItemDef.catalogue()` naming a rarity, the `Ruleset`
fields it sets, and — unless it is a common — an unlock condition built with
`_at(<a counter in Progress.STATS>, n)`; plus `item.<id>.name` and
`item.<id>.desc` in `scripts/loc.gd` (name, gloss, and description — the gloss
English-only), and a `_draw_<id>()` case in `scripts/item_icon.gd` laid out
inside its 96x96 box. Nothing else needs to know it exists — the reward screen,
the badge strip, the offer roll, the compendium and `_test_relic_games` all
enumerate the catalogue, and the compendium pages itself, so a new relic simply
lands at the end of the last page (or starts a new one). A condition on a counter nothing books yet
means one more entry in `Progress.STATS`, an `unlock.<stat>` and
`unlock.<stat>.one` string, and a `Progress.bump()` wherever the thing being
counted happens. If the effect needs a hook that isn't on `Ruleset` yet, add the
field there and read it in the one place that owns it (`Combo` for legality
and comparison, `TienLenGame.start()` for the deal, `scripts/battle.gd` for
money), rather than special-casing the item.

### Fast forward

Going out first at the classic table means sitting through the other three
finishing — about fifteen seconds with nothing left for you to decide. So the
moment you are out of a hand the others are still playing, a **fast forward
mark** appears above the Play button, in the corner your thumb is already in:
it takes over that corner at exactly the moment Play stops being useful. It
multiplies every seat's thinking time by `FAST_FORWARD_SCALE`, which takes a
hand from about **60 seconds to about 5**.

Not zero. At zero the three seats resolve the rest of the hand inside one
frame, the cards teleport into their final places, and you cannot see who took
what — which defeats the only reason to still be watching.

It is a **toggle**, not a one-way switch, so a hand that turns interesting
again — somebody chopping for second place — can be watched properly. It goes
away on its own when the hand ends, because the condition that offers it
(`_can_fast_forward`) is false once `game.is_over()`, and that is the same
moment the end-of-hand card comes up.

The mark is drawn (`scripts/fast_forward.gd`) rather than worded. Play and Pass
are this table's two decisions and they are worded buttons; a third worded
button stacked over them read as a third decision, which this is not. Gold at
rest and breathing towards white, because it appears mid-hand with nothing else
on screen changing to announce it — a still mark in a corner you have stopped
looking at simply does not get seen. Engaged, it stops breathing and fills in
solid: the two states have to differ in more than brightness, since the pulse
takes the offered mark all the way to white twice a second anyway.

A translucent wider stroke for a halo was the first attempt at the glow and it
came out a smudge — at this stroke width the miter on a chevron's tip runs a
long way past the point, and the ghost behind the mark read as a printing
error. The pulse is colour plus a little stroke weight instead.

`_check_fast_forward` drives a real hand and reports the four things that
matter: hidden while you are still in it, offered once you are out, the toggle
actually engaging, and gone again by the time the card is up.

The end-of-hand card carries **two** buttons: *Deal Again* in the warm gold
every action button in the game wears, and *Return to Main Menu* under it in the
quiet style, because that is the way out rather than the way on. The in-game
menu has a way out too, but it is behind a button you have to know to open, and
the end of a hand is the moment a player is actually deciding whether to sit
down again.

`ResultCard` grows by exactly `SECONDARY_H + SECONDARY_GAP` when asked for the
second button, and builds its buttons from the bottom edge up — so `Deal Again`
and every line above it stay at the pixel they were at, and `body_bottom()`
comes out unchanged. The duel's card asks for no secondary and is untouched.

### Resetting everything

The one action in the game that cannot be undone lives in the **top-right
corner of the Relics screen**, not on the main menu. That screen is the one
that shows what you have collected, so it is where somebody who wants to start
over goes looking — and it is two taps in rather than under a thumb that landed
on the menu by accident. Top right specifically: diagonally opposite the back
button and as far from the two chevrons as the screen allows. It was bottom
left first, which on a phone held in two hands is exactly where a thumb rests.
It is a **framed button** in the game's own quiet style, reading "Reset Relic
Progress", rather than a hidden gesture: the confirmation is the safety, and a
reset nobody can find is no use to the player who actually wants one. It was
flat text with an ellipsis, which read as a caption — the one control on the
screen that looked like it was labelling something rather than offering it. The
text is warmed towards the colour the game uses for a loss, so it is findable
and never inviting. It does not appear at all while the screen is embedded in
the in-game menu, since wiping the save from inside a run in progress is not a
thing to offer.

The confirmation is a `ResultCard`, the same lacquer panel every other
"something has happened" moment uses, rather than an engine dialog that would
look like it came from a different program. **Keeping your progress is the big
gold button and resetting is the quiet one under it** — the opposite of every
other card in the game, and the right way round for the one press that cannot
be taken back.

This is the card that forced `ResultCard.PAD`. Every line on a card used to be
laid out across the full panel width, which is invisible until something wraps
— and this warning is the only four-line piece of prose in the game, so it ran
right up against the gold border with its last line touching the rule. Lines
now sit inset by 26 either side, the same idea as the 48 the buttons already
had, and this card's rule moved down to clear the wrapped text with air rather
than underlining it.

Taking 52px off every line's width is not a local change, though: it pushed the
duel's deal-over card onto a third line, since its longest sentence names the
opponent and so is as long as the longest foe name. `_test_result_cards_fit`
measures both cards against their rules with the real font, filled with the
worst case a card can actually be handed — the longest opponent name, a full
hand, and a four-figure sum.

**It locks the relics again and does nothing else.** `Progress.wipe()` clears
the relics found and every unlock counter — which is all of them, since gating
a relic is what `STATS` is for — and touches nothing outside that: not the
wallet, not the spins banked, not the run in progress or the head start owed on
the next one. The line it draws is between a *record* and a *holding*: those are
things earned and not yet spent, and a reset that took them would be one nobody
could afford to press. Putting the
bankroll back up would turn this into a way to top up: a broke player
could wipe their relics, which cost them nothing they were still using, and
walk away with a full wallet. Progress is a record of what you have done and
money is the score; throwing away the record should not pay. The confirmation
says so in as many words, since a player weighing it up should not have to find
that out afterwards.

`wipe()` is deliberately a different name from the `reset()` the tests use —
one clears scratch state and the other throws away everything the player has,
and those should not be one call anybody can reach for by accident. `_test_wipe`
fills every counter, collects every relic, banks spins, starts a run, wipes,
and checks it in **both directions** — that the whole catalogue reads exactly
as it does on a fresh install, and that the wallet, the spins, the run in
progress and the head start are all exactly where they were.

Keeping spins through a reset is what forced the ledger to be **stored rather
than derived**. It used to be `classic_wins / WINS_PER_SPIN - spins_used`,
which could never drift out of step with the wins behind it — but
`classic_wins` is also the unlock condition for four relics, so it *has* to go
back to zero on a reset, and a derived balance went to zero with it.
`spins_banked` and `wins_to_spin` are now real numbers, advanced together by
`Progress.book_classic_win()` so a win can never move one without the other. A
save written before that change is migrated once on load from the two numbers
that used to derive it, so nobody loses a spin they earned.

### Spin to Win

**Every three classic-mode wins earns one turn of the gacha machine**, redeemed
from a sub-item hung under the Classic button on the main menu. Until there is
one owing the button is disabled and counts up instead — *Wins Required: 2 of
3* — because a dead button with no reason written on it reads as broken. Once
one is owing it says *Spin to Win!* and **breathes gold**: every other item on
this menu is always there and always the same, so a button that has just become
live has nothing else to distinguish it, and "you have earned a turn, come and
take it" is the one thing the screen is trying to say. The tween is owned by
the button rather than the menu, so a language switch rebuilding the screen
takes it with it instead of leaving one running against a freed stylebox.

Three wins rather than five: a four-player hand is won a quarter of the time,
so five is twenty hands, and that is a very long way between rewards for
something meant to keep you coming back to the mode.

It gives the classic table somewhere for its wins to go. The mode is otherwise
a thing you play for its own sake with nothing accumulating between sessions,
and the machine already pays into the classic wallet, so the loop closes on
itself: wins earn spins, spins pay money, money buys hands.

The balance is stored as **wins earned minus spins taken**, never as a running
balance. `spins_used` is the only thing persisted, and `spins_available()` is
`classic_wins / WINS_PER_SPIN - spins_used` floored at zero — so a save edited by hand, a
counter a patch resets, or a spin interrupted half-way can never leave a
balance that disagrees with the wins behind it. The spin is booked *before* the
machine opens, so backing out of the screen cannot hand over a free turn.
`_test_spins` walks the count up to the first spin, redeems it, is refused a
second, banks three at once, and checks that an over-redeemed save reads as
none owing rather than as a negative the menu would have to special-case.

## The shop

Hung off Classic on the main menu, next to the earned spins, and the only place
the wallet is ever spent. Three columns of cosmetics — the cloth, the mark sunk
into the middle of it, the backs of the cards — and turns of the Lucky Draw
across the bottom.

**Nothing in it changes how a hand plays, and nothing in it can.** Classic is
plain Tiến Lên played for money against three other people; a shop that sold an
edge would make the money it is bought with meaningless. `_test_shop` walks
every entry and fails if one carries a field the rules engine reads — `Ruleset`
is the only way anything in this game bends a rule, so "has no Ruleset field"
*is* the statement "cannot cheat". The relics, which do bend rules, are the
roguelike's, are found rather than bought, and never reach a classic table.

**Every row shows the thing rather than naming it.** The felts are drawn as a
patch of cloth lit at one end the way the table lights it, the marks are drawn
on whatever cloth is currently on the table, and the card backs are drawn as
card backs. A shop selling appearance that showed a list of names and prices
would be asking the player to buy sight unseen. Buying wears it immediately —
paying for a felt and then having to go and put it on is a second step nobody
wants for a thing they just bought.

| Column | Free default | Bought |
| --- | --- | --- |
| Table felt | Market Green | Lacquer Red, Midnight Blue, Rượu Nếp Plum ($250–300) |
| Table mark | Đông Sơn Drum | Lotus, Star Lantern, Old Coin ($300–350) |
| Card backs | House Red | Jade Chiếu $150, Indigo Weave $200, Hoa Mai $300, Vàng Mười Gold $500 |
| Chop effect | Payday | Jackpot, Blast, Lotus Burst, Bowl of Phở ($200–450) |

### Chop effects

A bomb is the biggest swing either mode has, so it gets the loudest flourish in
the game, and the shop sells which flourish it is (`scripts/chop_fx.gd`). One
entry point — `ChopFx.play(table, at, tier, pile, spare)` — reads the worn
effect and fires it, so both tables ask the same question and neither knows
what the answer will be.

| Effect | What happens |
| --- | --- |
| Payday | The original: coins and banknotes out of the middle |
| Jackpot | The same coins and notes thrown far harder, bigger and in three waves, and the ceiling gives way |
| Blast | A firecracker is lobbed onto the table, and everything already lying there is thrown off it |
| Lotus Burst | Pink petals with gold stamens, blown out and fluttering down |
| Bowl of Phở | A bowl slides on from one side, tips over the middle, and carries on off the **other**, throwing its two chopsticks clear and leaving broth and noodles across the cards |

**Blast is thrown, not summoned.** A red firecracker comes in from off-screen
tumbling, lands short of the pile, burns its fuse for a quarter of a second and
then goes off in a white flash and a scatter of red paper and gold foil. The
cards are thrown at *that* moment rather than when the chop resolved, so the
explosion is the cause of them leaving rather than something happening beside
it — which is the whole difference between a firecracker and a decoration.

Because of that half-second gap, the cards to spare are held as **nodes rather
than as a count**. The pile can be added to or swept between the chop and the
bang, and a count would then protect whichever cards happened to be newest at
that point instead of the ones that actually won the round.

**The bomb never blows up its own cards.** `spare` says how many of the pile's
newest cards were the winning play, and Blast leaves exactly those standing in
the wreckage — the play being celebrated has to still be readable when the
flourish is over. Blast animates the *real* card visuals rather than copies, so
what flies off is what was lying there, and each tween is created on the card
rather than on the table, so a card freed underneath it takes its animation with
it instead of leaving one running against a freed node.

`_capture_chop_fx` checks that by identity too, and had to be rewritten to do
it: comparing the pile's size before and after counted the AI's next play as a
card that survived the blast, and reported a working effect as a broken one. It
now names the doomed and spared cards up front and reports how many of each
actually went — `threw 4 of 4, left the bomb's 2 standing`.

The card backs are listed **cheapest first**, so where one sits in the column
says what it is worth without anybody having to compare prices, and the gilt
one is last.

### The shimmer

**Vàng Mười Gold** — the purest gold there is, which is the phrase people reach
for when they mean the real thing rather than gold-coloured — is the only back
that moves. A band of light travels across the table and catches each card as it
passes.

It is a **shader**, not a redraw, and that distinction is the only thing that
makes an animated card back affordable. Done in `draw_*` it would mean every
face-down card re-tessellating its whole pattern every frame, which is exactly
the cost that made Hoa Mai hitch. Instead the card's draw commands are stored
once and the GPU re-renders them each frame with `shaders/shimmer.gdshader` over
the top: **1,050 draw calls, identical to the free default**, and the shimmer
comes free.

Two details it depends on. The sweep is in `SCREEN_UV` rather than per card, so
one band crosses the whole table rather than every card pulsing in step — which
is what light moving over a real table does. And every shimmering card shares
**one** `ShaderMaterial`: a material each would put every card in its own batch,
and they all want the same sweep anyway.

`face_down` is a property with a setter rather than a plain field, because
turning a card over is what decides whether it wears the shimmer, and `TableFx`
flips cards mid-flight as well as at build time. A face-up card is the card, not
the back.

**Nothing sold may cost more to draw than the free default.** A card back is a
figure repeated twenty-one times on a grid, and the whole pattern is redrawn
every time a fan is rebuilt — which is every play, across every face-down card
on the table. Hoa Mai first drew its blossom as five `draw_circle` calls and a
centre: six calls a repeat, 126 a card, and a table of face-down hands measured
**5,145 draw calls a frame against 1,050 for the default**. A circle is
tessellated rather than batched, so the cost is worse than the count suggests,
and it hitched the table on every play.

It is one polygon now — a twenty-point petalled outline worked out once at load
rather than per repeat — and measures 1,036, which is the default's own figure.
`_measure_card_backs` in the capture harness reports every back's draw calls and
flags any that runs more than a quarter over the free one, since this is a cost
that is invisible in a screenshot and only shows up as a feeling.

**A swatch cannot show an animation**, so tapping a chop-effect row plays it
over the shop — the same call the table makes, with the same worn effect. The
swatch itself is a mark rather than a picture: what comes out of the middle and
how much of it.

**Jackpot is not Payday with more in it.** It was, at first — three times the
count — and the two were nearly indistinguishable, because a denser burst of
the same pieces at the same speed fills the gaps rather than covering more
table. What separates them is `force` and `waves` on `MoneyBurst`: pieces
thrown two and a half times as hard and drawn bigger, in three waves a beat
apart, so the money clears the table instead of landing back in the middle.

On top of that the **ceiling gives way**: forty-six coins rain down over a
second and a half, huge at the top of the screen and shrinking as they fall
towards the table. Big-near-the-top is the right way round for this camera — a
coin near the ceiling is near the eye, and the table it is falling towards is
the far end of the room. The fall is eased *in*, so a coin hangs at full size
and then drops away; eased out it shrank almost the moment it appeared and the
entire gradient the effect is built on happened in the few pixels above the top
of the screen. They arrive staggered rather than as a curtain, since a curtain
dropped in one go is a wipe and a rain that keeps coming is a machine that will
not stop paying out.

The rain is made of `MoneyBurst.coin`, now static and drawable into any canvas,
rather than a second coin that would have to be kept in step with the burst's
by hand. All of it together costs 1,550 draw calls a frame against Payday's
1,615 — the extra pieces batch, so the difference is free.

Four details the bowl needed before it read as one:

- **In one side and out the other.** A bowl that arrives and leaves the same
  way looks like it was pulled back; one that carries on across reads as
  somebody walking past the table with it.
- **The chopsticks go over with it.** They stand in the bowl until it tips past
  a quarter, then launch over the low lip on their own ballistic arc, one a
  little ahead of the other — a pair leaving in perfect step reads as one
  object.
- **They are drawn under the bowl until they leave it.** Over it, two sticks
  read as lying across the rim; under it, the body hides everything below the
  rim and only the tops show, which is what being *in* a bowl looks like. Once
  they are in the air they draw in front of everything, which is equally what
  that looks like.
- **The noodles curl.** Each strand is fourteen points following a wave laid
  across its own direction — a curl rather than a zigzag — opening out towards
  the far end, with its own winding, amplitude, phase and handedness. Three
  points made a bent line, and the spill read as a sunburst of matchsticks.
  They are inked as well: a dark stroke slightly wider underneath each, because
  plain cream strokes vanished against a face-up card, which is exactly where
  this effect is meant to land them. And each starts somewhere of its own
  rather than at the middle. The mess also outlives the bowl now — it used to
  be fading while the bowl was still leaving, so most of what the effect was
  for happened behind it. There were green herbs in it once; they read as green
  noodles, which is not a thing.

The bowl is drawn in **its own rotated frame** rather than by leaning its
points. Shearing the body against a tilted rim pushed the arc above the rim line
at some angles, which makes the polygon self-intersecting — and Godot's
triangulator rejects those outright, so the bowl silently did not draw at all.
A rotation cannot turn a simple polygon into a crossed one. This is the third
time that failure has come up in this codebase (the Bột Ngọt sachet and the
spill quad were the others), and it is always silent: the shape is simply
absent, and only the console says why.

The felt and the mark dress the **classic** table only: the duel has its own
cold slate room, and that is part of what makes a duel feel unlike a Sunday
afternoon with the neighbours. A card back and a chop effect are worn in
**both**: one is the same fifty-two cards wherever they are dealt, and the other
is an effect on a *play* rather than a piece of furniture. A player who paid for
either should not have it taken away for climbing the ladder.

**A Lucky Draw costs $50 and is worth $24.28.** That figure is measured, not
estimated — `_test_spin_price` runs 40,000 draws through the real payout table
and fails if the price ever falls under what a turn pays. Sold under its own
expected value it is not a shop, it is a money printer: buy spins, win more than
they cost, buy more spins. The margin is what makes it a gamble on the big hand
instead, and the test also fails if the price climbs so far above the value that
nobody would ever buy one.

Buying goes through `Run.spend`, which refuses rather than overdrawing and —
unlike `charge` — never triggers the broke allowance. Being re-staked for
spending your last dollar in the shop would make everything in it free once you
were poor enough, which is the same shape of bug that made roguelike runs free.

### The wallet cheat

Pressing the **Classic wallet** line on the shop `CHEAT_TAPS` (50) times pays
`CHEAT_PAYS` ($1,000) into it, and the count starts again so it can be used
more than once.

Deliberately unadvertised: no countdown, no hint, nothing on the screen that
suggests it is there. That is the difference between this and the compendium's
relic grant, which counts down out loud once somebody is clearly trying —
that one is a debug affordance, this one is the owner's. Fifty presses on one
line of text is not somewhere anybody arrives by accident.

Also unlike the relic grant, it is **not** gated on `OS.is_debug_build()`,
because it is meant to work in the build its owner actually plays. Wrapping
`_on_wallet_input` in that check takes it out of release. The game is
single-player and offline with nothing bought for real money, so the only
wallet it can move is the wallet of whoever is doing the pressing.

The press target is sized to the text rather than to the label: the label is
laid out across the whole screen so it can centre itself, and a full-width
invisible button would swallow presses aimed at nothing. `_capture_shop` drives
it through that target's own input and checks all three cases — forty-nine
presses do nothing, the fiftieth pays, and the fifty-first does nothing, since a
cheat that fires early or repeatedly is one somebody else finds.

Ownership and what is worn live in `Prefs` beside the other table settings,
because they are a preference and not a record of anything the player did.
Anything unowned, unknown or dropped from a later build falls back to the free
default, so a save can never dress the table in something it has not paid for or
something this build cannot draw.

## Money (classic mode)

You sit down with **$200**, and the balance carries across sessions in
`user://settings.cfg`. Twenty hands' worth of table fees before the winnings
are counted, so the wallet is something a player is aware of from the first
sitting rather than a number that takes an evening to move.

| Event | Amount |
| --- | --- |
| Sitting down for a hand | −$10 |
| Finishing 1st / 2nd / 3rd / 4th | +$25 / +$10 / +$5 / nothing |
| Chopping another player | +$10 |
| Being chopped | −$10 |

**There is no house allowance.** Hit $0 and you are out: the wallet floors at
nothing rather than going into debt, and the classic table costs $10 to sit at,
so at zero there is nowhere to sit. A cleaned-out player used to be re-staked
$100, which made going broke free — and anything free is a thing to do on
purpose.

What there is instead is the other mode. **A run costs nothing to start, and
beating an opponent pays classic money**, so the way back from broke is to go
and climb the ladder rather than to be handed a float:

| Opponent | Pays |
| --- | --- |
| 1st | $10 |
| 2nd | $25 |
| 3rd | $50 |
| 4th | $75 |
| 5th | $100 |

Rung one is exactly one hand back at the table; a whole run pays $260, which is
twenty-six. Deliberately not more — the ladder is the way back from broke, not
a better way to earn than the classic table itself. `_test_bankroll` checks the
shape rather than the figures: every rung pays more than the one below it, the
first pays at least a table fee, and a run driven from a $0 wallet really does
end able to sit down again.

Two screens have to *say* this rather than merely enforcing it, because a
player who has just run out needs to be told where money comes from now. The
main menu shuts Classic with the reason on the button ("Classic Mode — needs
$10"), and the end-of-hand card disables Deal Again and says to go and win more
on the ladder. A button that takes a press and does nothing reads as broken. Only your wallet is tracked — two AI players chopping
each other costs nobody anything. The numbers live in `scripts/run_state.gd`
(`Run`), whose payout maths is pure and covered by `_test_bankroll`.

## Statistics

A lifetime record, on its own **Stats** button under the two modes on the main
menu. Nothing on the screen can be pressed except the way back — it is a page, not a
screen you do anything on — and nothing on it changes what the game does.

**Two panels, one per mode, and each is itself two columns.** The left column
is the mode read as a whole and the right is how it breaks down. Five of the
left column's lines are common to both modes and in the same order, so those
rows can be read across the two panels as well as down one:

| | |
| --- | --- |
| Time played | Chops landed |
| Hands played | Times chopped |
| Win rate | |

The roguelike's left column adds hands won, opponents defeated, duels lost and
runs completed; the classic table's adds the best streak of firsts, what is in
the wallet right now, and every dollar into and out of it.

The right columns are where the two modes stop rhyming. The classic table's is
**one line per chair** — placed 1st, 2nd, 3rd, 4th — built from the same
ordinals the end-of-hand card is written with. There is deliberately no "hands
won" line beside it: that is the first row of the column, said twice. The
roguelike's right column is the ladder rather than the table:

| Line | What it is |
| --- | --- |
| Best win streak | deals taken in a row |
| Furthest rung | the highest rung a run has ever reached, out of five |
| Biggest purse | the fattest purse ever held, caught at its peak mid-duel rather than at the end of one |
| Relics collected | distinct relics ever taken, out of the catalogue |
| Favourite relic | the one taken on the most runs, and how many |
| Nemesis | the opponent that has ended the most runs, and how many |

**Nemesis is the mirror of the favourite relic**, and between them they are the
two lines on this screen anybody will quote at anybody else. Both are tallied
per occurrence rather than per kind, and both break ties by catalogue order so
the answer is stable rather than depending on the order a dictionary comes back
in.

Then, in both panels, the seven shapes as you lay them down (singles, pairs,
triples, quads, straights, pair runs, and bombs — bomb-shaped plays whether or
not there was anything under them to chop), and the longest straight that mode
has ever seen.

**The gacha machine gets a band of its own along the foot**: draws taken, best
winnings, total won, and the best hand it has ever paid. It is reached from
both modes and belongs to neither, so it cannot go in either column without
saying something untrue about where it came from.

Two things on the screen are hands of cards rather than numbers — the longest
straight in each mode, and the machine's best hand — and both are drawn as
real `CardVisual`s. The suit glyphs are not in the font iOS falls back to (see
**Language** below), so a straight written out as "10♠ J♦ Q♥" would ship as a
row of blanks on a phone. Drawing the cards is both the only thing that works
and the better picture.

`scripts/stats.gd` (`Stats`) keeps it, static like `Prefs` and `Progress` and
sharing `user://settings.cfg` with them. **It is deliberately a second copy of
numbers `Progress` already keeps.** `Progress` exists to unlock relics, and the
Relics screen offers to wipe it so the collection can be found again; a record
of everything you have ever played is not a thing that should vanish because
somebody wanted to re-earn a relic. So `classic_hands` lives in both, and only
one of the two is resettable. `_test_stats` leans on exactly that, and on the
classic table's finishing places — booked by the same call as the hand and the
win, so a screen can never claim more firsts than hands taken.

Play time is counted every frame from both tables' `_process` and written to
disk every thirty seconds, with a flush on the way out, so a table put down
mid-deal still counts the minutes it took rather than rounding them away. The
tests and the capture harness set `Stats.sandboxed`, like every other owner of
that file, so neither can write play into the developer's own record.

The layout is checked rather than eyeballed: `_test_screens_fit` pins both this
screen's column arithmetic and the main menu's, because both overflow silently
— the menu's language picker slides off the bottom edge, and this screen's last
row is a hand of cards, which drawn past the panel's foot reads as a layer
painted over it rather than as something that did not fit. 720 is the tightest
the screen ever is: the stretch is `expand`, so a taller aspect ratio gives
width and never height. Adding the button was what forced the menu's whole
block up and in by a few pixels.

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

### No symbols in strings

**Nothing a player reads may be built out of a symbol character.** Godot's
built-in font carries Latin and punctuation; what iOS falls back to for the
rest does not carry symbols at all — so ♠♣♦♥, ☰, ✓ and ⏩ all arrived on a
phone as *blanks*.

That is the worst way for this to fail. The sentence still turns up, still
reads as a sentence, and has quietly lost the part that carried the meaning:
"The first play must include the 3♠" became "The first play must include the
3", which is not even wrong. And it is invisible on a desktop run, because
macOS has a font for every one of them.

So a suit in a sentence is **spelled out** — "a single 7 of hearts", "một lá
7 cơ" — through `Card.label()`, which every message that names a card goes
through. `Card.text()` still returns "7♥" and is for console output and test
failures only. A suit *on a card* is a vector, drawn by
`CardVisual.draw_suit`, which is also what the hand-selection screen counts
with. The menu button's hamburger is a texture from `Ink.hamburger_texture()`
rather than ☰, the fast-forward mark is ASCII `>>` rather than ⏩, and the
bought-reveal button says "Bought" rather than wearing a ✓.

`_test_text_is_renderable` holds the line: it walks every string in
`Loc.STRINGS` in both languages, plus the sentences the game builds at
runtime, and fails on any character in the arrow, dingbat, geometric-shape,
misc-symbol or emoji blocks.

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

The suit lenses were the soft spot in the set until they were reworked to
promote into hearts; see the lens table above for what was wrong and what it
measured.
Collapsing every off-suit onto the single best suit held instead measures at
12.5 and 8, but it produces four cards of one suit where the colour-preserving
mapping produces two and two, so it is not what these relics are meant to do.
If they need to come up, it should be through a second effect rather than by
bending the mapping.

Two caveats the sim prints for itself: the stand-in player
is the greedy AI, which has no model of hidden information (so Kính Mát
measures as exactly zero), deals straight into a duel with no hand to pick (so
Sinh Tố Bơ does too), and rarely chooses to chop (so Nước Mía and Dao Bầu
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
godot --fixed-fps 60 --path . res://tests/screenshot.tscn   # every screen, ~45s
SHOTS=classic godot --fixed-fps 60 --path . res://tests/screenshot.tscn   # ~9s
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

## The boot splash

`tests/make_splash.gd` draws it and writes `user://boot_splash.png`:

```sh
godot --resolution 1280x720 --path . res://tests/make_splash.tscn
```

Three layers, in draw order: the lacquer and the Đông Sơn drum face behind,
three cards over them, the words over everything. Children draw after their
parent, so the text is a sibling added last rather than more drawing inside the
backdrop. The cards are real `CardVisual`s, so the splash cannot drift from the
game's own card faces — and they are the 3♠ that opens every hand, an ace, and
the 2 that beats them both.

The credit line is set in **Snell Roundhand**, a macOS system script face. Only
that line uses it: a script face has no business trying to set "Tiến Lên", so
the title stays on the game's own font. The font is used to *render* the PNG
and is not shipped, which is what keeps this clear of Apple's font licence.

The artwork fades to exactly `boot_splash/bg_color`, so one 16:9 image sits
seamlessly on any phone's aspect. `stretch_mode` is **Keep** (1) — note that
Godot 4.7 replaced the old `fullsize` boolean with this enum, so a project
still setting `fullsize` is setting a key nothing reads.

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
- `scripts/purse_bar.gd` — both duel purses drawn against the one pot on the
  table, since a duel is zero-sum
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
- `shaders/shimmer.gdshader` — the light travelling across the gilt card back.
  A shader so that an animated back costs no more to draw than a still one
- `scripts/table_fx.gd` — opponent card fans (face down, with the lowest few
  readable under Gương and the top two under Kính Mát), the toss into the
  middle, and the sweep out of it
- `scripts/poker.gd` — `Poker`: five-card hands and what they pay, for the
  gacha machine and nothing else
- `scripts/lottery.gd` — the machine itself: the cabinet, the heap of 52
  capsules, and the crank
- `scripts/item_card.gd` — a relic as a card: full size on the reward screen,
  as a HUD badge in a duel, and full size again read-only when a badge is
  tapped. Every single-line label on it shrinks to fit and ellipsises as a
  backstop:
  relic names are Vietnamese and often four syllables (Cà Phê Sữa Đá, Trứng
  Vịt Lộn), a Label does not clip by default, and at a fixed size the long ones
  ran straight out of the badge and across whatever was beside it
- `scripts/compendium.gd` — the menu's relic list, eight to a page: every icon,
  and as much of each name, gloss and effect as the player has earned
- `scripts/progress.gd` — `Progress`: what the player has done across every
  session, which relics that unlocks, and which they have actually won
- `scripts/stats.gd` — `Stats`: the lifetime record, kept per mode. A second
  copy of some of what `Progress` counts, on purpose: this is the half the
  Relics screen's reset does not reach
- `scripts/statistics.gd` — the screen that shows it, two panels side by side
  with the gacha machine's own band along the foot
- `scripts/music.gd` — `Music` autoload: the track pools, the crossfade, and
  the classic table's shuffle
- `scripts/layout.gd` — `Layout`: where the edges of the screen actually are
- `scripts/scene_fx.gd` — `SceneFx` autoload: the card-throw screen transition
- `scripts/festive_fx.gd` — lanterns and hoa mai petals for the red screens
- `scripts/hand_drag.gd` — dragging cards around inside your own hand
- `scripts/prefs.gd` — `Prefs`: table settings remembered between sessions,
  and what the shop has sold — which cosmetics are owned and which is worn
- `scripts/shop_item.gd` — `ShopItem`: the shop's catalogue. Colours, shapes
  and lottery tickets, and a test that fails if one ever carries a rule
- `scripts/shop.gd` — the shop screen: three columns of swatches drawn with the
  same code that draws the real thing, so nothing is bought sight unseen
- `scripts/menu_dragon.gd` — the dragon silhouette that crosses the main menu
- `scripts/money_burst.gd` — the coins-and-banknotes explosion a chop sets off
- `scripts/chop_fx.gd` — `ChopFx`: which explosion that is. One call both
  tables make, dispatching to whichever effect the shop has sold
- `scripts/pass_cross.gd` — the red X stamped over a seat that has folded
- `scripts/turn_glow.gd` — "it is your turn", said by the bottom edge of the
  screen going warm rather than by an object appearing on the table
- `scripts/fire_streak.gd` — who is on a roll and how hot, shared by both
  tables; no scene or audio dependencies
- `scripts/table_flames.gd` — the fire that engulfs a hot seat's cards, drawn
  behind them and purely decorative
- `scripts/game_menu.gd` — the in-game menu opened over a table without
  leaving it, embedding the real `Compendium` rather than a copy
- `scripts/fast_forward.gd` — the pulsing skip mark offered once you are out of
  a classic hand the other three are still playing
- `scripts/menu.gd` / `opponent_select.gd` / `reward_select.gd` /
  `classic_table.gd` / `battle.gd`
- `tests/test_rules.gd` — headless rule, engine, personality, relic, and
  ladder tests
