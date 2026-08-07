# luxplay

Small lux programs written for the fun of it. Every one runs with `lux run
<file>`, and every one converts clean to Rust, Swift, and Go.

`mandelbrot.lux` draws the Mandelbrot set in text. Four constants near the top
are the window on the number plane; shrink it around a point and run again to
zoom. Nothing to type, nothing to install.

`rps.lux` plays rock, paper, scissors, and it has no dice. For every pair of
moves you have just played it keeps a tally of what you played next, so each
turn it looks up the pair you are sitting on and counters whatever usually
follows. It announces its guess before you answer, which is the part that
stings. Play it as a habit and it takes about six rounds to lock on; play it as
real chance and the best it can do is break even, because a table of counts has
nothing to work with.

`random.lux` builds the thing lux deliberately does not have — a random number
generator — out of multiplication and a remainder, and seeds it from the system
clock by asking `date` for the nanosecond through `run`. It rolls a die sixty
thousand times and prints the histogram, then shuffles a hand of cards. Copy the
three functions at the top into anything that needs to roll something.

`hangman.lux` and `maze.lux` are what the generator is for. Hangman picks its
word from a list; the maze digs itself with a depth-first walk that only ever
breaks into a cell it has never stood in, which is why exactly one path connects
any two points in it.

`poker.lux` is the big one: heads-up Texas Hold'em against a bot, with a tutor
that talks you through every decision. Before each of your turns it deals the
rest of the hand out a thousand times or more, every unknown card falling every
way it can, and counts how often you finish in front. That share is your equity,
and next to the price of the call it answers the only question the game asks. So
the advice is not a chart it looks you up in — it is a simulation run fresh for
the cards you are actually holding, which is why it can tell you that a hand
worth folding at one price is worth calling at another.

The equity engine is checked rather than trusted. A made royal flush returns
exactly 100%, an ace-high straight lying on the board returns a thousand ties
out of a thousand, and pocket aces against one unknown hand come back at 85.0%
against a published 85.2%. That last case sets the limit: at these trial
counts the answer is good to about half a percent, which is why the tutor quotes
whole numbers and says "about". The tutor also says out loud what its own number
leaves out — equity against *any two cards* is not equity against the hand of
somebody who just bet into you.

You do not have to know the game to start. There is a fourth choice on the
opening prompt that walks you through hold'em in one screen, and after that the
program names things as they happen rather than in advance: what a flop is the
first time one lands, what a kicker is the first time one decides a hand, what
the price means the first time there is one to pay. Each note appears once and
never again. Typing `?` at any prompt shows what beats what, which is the
question a beginner has constantly and the one a poker program usually assumes
you already know.

The bot knows its own cards, the pot, and nothing else, and it comes in three
strengths you pick before the first hand. The interesting part is how the easy
one is made weak: not by handing it bad rules but by giving it a hundred and
fifty trials to read its own hand instead of nine hundred, so it genuinely
misjudges what it is holding. On top of that it calls past the price it is being
offered and never bluffs. Bet into it across thirty hands and it folds not once.
The hard bot pays exactly the right price, bets hands it is only just ahead
with, and bluffs about one hand in four; the same mindless betting that breaks even against
the easy bot loses half a stack to it.

Because the opponent differs, the advice differs with it. Against the bot that
calls too much the tutor lowers the bar for a value bet — you can bet worse hands
when worse hands get called — and tells you to stop bluffing, because a bluff
against something that never folds is money handed over. Against the bot that
bluffs one hand in four it says the opposite: its bet is less news than it looks.
That is the half of poker a pot-odds calculation leaves out, and having three
opponents to feel the difference against is the only way to teach it.

When you leave, it tells you how you played. Every decision is scored against
the advice that was on the screen when you made it — how many matched, how often
you called when the price was wrong and what those calls were worth by the odds
at the time, how often you folded a price worth paying or checked a hand that wanted
betting. Chips at the end are mostly the cards you were dealt; this is the part
that was yours. Spots the tutor itself called close are not scored either way,
because marking you against advice it did not really have would be worse than
saying nothing. The recommendation and the score come out of one function for
the same reason: if the sentence you read and the answer you are marked against
could drift apart, the summary would be worthless.

Interpreted, a hand takes about a second. Run `lux build poker.lux` and the same
program does it in eighty milliseconds, which is roughly eleven times faster and
is what a compiled language buys you the moment a program starts simulating
rather than printing.

`todo.lux` is the only app here rather than a program: it keeps a file, so it
survives being closed. `lux run todo.lux` shows the list, `add walk the dog`
puts something on it, `done 2` ticks it off, `drop 2` removes it, and `clear`
throws out everything ticked. The list lives in `todo.txt` next to wherever you
run it, one item per line, readable and editable in anything — a line beginning
`x ` is one you have finished.

Two small things in it are worth reading. Asking whether a line *starts with*
the tick can't be done by looking at the first character, because lux won't give
you one, and `contains` would answer yes to "book x ray appointment". Splitting
the line at every `x ` answers it exactly: only a line that begins with the mark
has an empty first piece. And the filename is passed into the function that
reports a failed save rather than read from the top of the file, because a
function cannot see a file-level `let` — except on Swift, where it can, which is
[lux#76](https://github.com/anderix/lux/issues/76).

`blackjack.lux` is poker's counterpart and deliberately the opposite technique.
Poker simulates because the space of ways a hand can finish is far too large to
walk through; blackjack is small enough to walk through, so this one works the
whole tree out exactly and shows you what standing, hitting and doubling are
each worth in chips before you choose. Which method is right is decided by the
size of the problem rather than by taste.

The tables are built downward. Twenty-one is settled — you stand, and the value
depends only on the dealer. Twenty is settled once twenty-one is. Sixteen is
settled once everything above it is, because every card you can draw takes you
higher. Three passes get there with no recursion and no circles: hard totals of
eleven and up first, since they can never reach a soft total; then soft totals,
which fall back only on hard totals of twelve and up; then the rest.

The numbers were checked rather than trusted. An independent implementation of
the same model agrees with all 940 the tables produce — every dealer
distribution and every stand, hit and double value for every total against every
upcard — to five decimal places, and it reproduces the famous pair at sixteen
against a ten: standing -0.5404, hitting -0.5398. Those six ten-thousandths are
why the program says hitting sixteen is not good, only less bad, and it can show
you the gap rather than assert it.

Probabilities come from what is genuinely unseen rather than from a fresh deck,
so they move as the shoe wears down, and the dealer's hole card stays in the
count until it is turned over — the tutor knows exactly what you know. That
makes insurance real rather than decorative: it needs the hole card to be a ten
more than a third of the time, tens are just under a third of a fresh shoe, and
the program tells you when that has stopped being true. It is the one decision
in blackjack that counting can turn around.

`connect.lux` is Connect Four against minimax with alpha-beta pruning. The bot
has no openings and no rules about good moves; it plays the position out to a
fixed depth, scores what it finds, and walks the answers back up assuming you
will always pick the reply worst for it. That assumption is why it never hopes —
it will not set a trap that only works if you miss it. The board is one flat
array of forty-two, which copies faster than a grid of grids and makes every
direction a single stride, so walking a line of four is addition.

Difficulty is the same search told to stop sooner — but the three depths were
measured, not chosen, because the obvious assumption turns out to be false.
Looking further ahead does not reliably make this bot better. Three hundred
games a pair across three seeds: depth five beats depth four comfortably and
depth two in all three hundred, while depth six loses to both of them and depth
three loses to depth two nearly every game.

That is not a search bug. Pruned and unpruned searches were compared across
2,352 positions and agree exactly. It is the guess at the bottom: the evaluator
counts threats, and an unanswered threat counts the same as an unanswerable one,
so a position judged just after the bot moves looks better than the same
position judged once the reply is on the board. Searches that stop on different
sides of that are not measuring the same thing. It is the odd-even effect, and
the fix is a better guess rather than a deeper search — worth knowing, because
"just look further ahead" is the first thing anybody reaches for.

It is also the one program here where compiling changes what the program feels
like rather than just how fast it finishes: the hard bot takes about a second a
move interpreted and four milliseconds built.

`nim.lux` is the opposite of connect four: a game where searching is a waste of
time, because the answer is a formula. Rows of matches, take as many as you like
from one row, whoever takes the last one wins. The bot has no lookahead and no
evaluator. Write each row's count in binary, stack them up, and ask of each
column whether the number of ones in it is odd; those answers read as a binary
number are the nim-sum, and it is zero exactly when the player about to move has
already lost. lux has no bitwise operators, no `^` and no shift, so the nim-sum
is assembled a column at a time out of division and remainder — which is what it
is anyway, and harder to mistake for a machine instruction once it is written
out.

The opening rows are 1, 3, 5 and 7, whose nim-sum is zero, so the game is
decided before either player has touched it. That is why it asks whether you
want to move first, and why going first and playing perfectly still loses. Type
`show` during a game and it prints the columns.

Both last-match rules are there, and the gap between them is narrower than it
looks: misère, where taking the last match loses, plays identically to the
normal game until at most one row still holds two or more, and only then
inverts. That is measured rather than argued. All 4,095 positions of four rows
up to seven matches were solved by exhaustive game-tree search and compared
against what the program believes, which agreed on every one under both rules,
and the two strategies picked different moves in 192 positions — every one of
them in the endgame.

`guess.lux` is the guessing game backwards. You think of the number and the
program finds it — out of a hundred in seven questions, out of a million in
twenty — and it does not matter which number you picked, because the program
never guesses at your number. It guesses at the middle of what it has not ruled
out, so there is nowhere in the range harder to find than anywhere else. The
lesson is that doubling the range does not double the work, it adds one
question. It also catches you moving your number without needing to suspect
anything: if your answers cannot all be true at once, the range runs out and
there is nothing left to guess. Then it offers you the same game the usual way
round, which is how you find out that knowing the trick and doing it are
different things.

One real bug came out of checking it. The worst-case bound counted doublings up
to the range, and that is one short for every exact power of two — ten doublings
reach 1024, but a range of 1024 takes eleven guesses, because the last halving
leaves two numbers and one of them still has to be asked about. Running every
number in seven different ranges through the actual search found it, and the
bound is now tight: the worst case observed equals the number claimed in all of
them.

`flash.lux` is the second app here, and the useful one. Flashcards in five
boxes: a card you get right moves up a box, a card you miss drops straight back
to the first however high it had climbed, and the box decides how long the card
is left alone — one day, two, four, eight, sixteen. A word you know well comes
up three times a month and a word you keep losing comes up daily, without you
deciding which is which. Getting it wrong is the whole of the sorting mechanism.

The deck is a plain text file of `front = back` lines that you can write in any
editor, mail to somebody, or keep in git; the schedule lives in a second file
beside it, so throwing the schedule away costs you the timing but never the
cards. `lux run flash.lux` asks what is due, `all` goes through everything,
`add der Hund = the dog` adds a card, `stats` shows where the deck stands, and
`-d german.txt` points the whole thing at a different deck. Run it with no deck
and it writes you ten German words rather than printing instructions about a
file format.

Two things in it are lux showing through. There is no `trim`, and `replace`
cannot fake one — it would take the spaces out of the middle of a phrase as
happily as off the ends — so tidying a line means splitting it on the space and
putting back only the pieces that hold something. And the field holding a card's
box number is called `boxNumber`, because `box` is reserved in Rust and a lux
struct with a field of that name produces Rust that will not compile; that is
[lux#77](https://github.com/anderix/lux/issues/77), which was filed the same
afternoon over a field named `move`.

`sudoku.lux` makes a puzzle and then solves it twice, because the search is the
dull part. Backtracking will finish any sudoku in about thirty lines and there
is nothing clever in it; what matters is which empty square you try next. So the
same search runs over the same puzzle with one difference — one solver takes the
first empty square it finds, the other takes the square with the fewest values
still open to it. Over twenty-five generated puzzles the plain solver averaged
3,795 values written and the careful one 95. Forty times fewer, for about ten
lines of difference.

The reason is better than the number. A square with one possible value is not a
decision, so taking those first means the decisions you do face are the smallest
ones available. The cost is shown rather than hidden: finding the most
constrained square means looking at every square, so the better solver does far
more work per step and wins anyway. Cheaper steps against fewer of them is most
of what choosing an algorithm consists of.

Puzzles are made rather than stored — a pattern grid disguised by renaming digits
and swapping rows within a band, then emptied one square at a time, each kept out
only if what remains still has exactly one answer. Twenty-five puzzles were
checked end to end with no failures: every grid legal, every answer consistent
with its clues, every puzzle with exactly one answer, and the two solvers always
agreeing.

`mastermind.lux` is guess.lux with the halving taken away. A code has no order,
nothing to be higher or lower than, and no middle to aim at — but the scoring
rule cuts anyway, because an answer is only consistent with some of the 1296
codes. So the program keeps the list of everything still consistent with what you
have said, and that list is the whole mind of it: no theory about your code, no
idea which colours you like, only what has not been ruled out.

Choosing what to say next is where the decision lives. Every candidate splits the
survivors into groups by the score it would come back with, and after you answer
only the matching group remains, so a guess whose largest group is small is a
good guess even when it is unlikely to be right. The program picks the candidate
whose largest group is smallest — it plays against its own worst case instead of
hoping.

It only ever guesses codes that are still possible, which is easy to trust and
not quite the best play. Played against all 1296 codes that costs exactly one
guess in the worst case: 1,252 solved within five, the other 44 in six, averaging
4.47. Knuth showed in 1977 that allowing guesses you already know are wrong
brings the worst case down to five and that five is the floor. So the price of
only ever saying things that might be true is those 44 codes.

`cipher.lux` shifts letters along the alphabet, enciphers with a keyword, and
breaks the first of those without being given the key. Twenty-five keys means
trying all of them breaks a shift cipher, but the program does not need you to
read twenty-five lines of nonsense to spot the English one: English letters do
not turn up equally often, shifting the text keeps their shape while changing
their names, so lining the tally up against what English looks like finds the
shift by arithmetic. All 25 shifts of a test paragraph were recovered.

Run the same breaker on text enciphered with a keyword and it fails, says so, and
explains why — which is the better half of the lesson. On a real shift cipher the
right answer fits English at 45.1 against 501.5 for the next best; with a keyword
it is 380.7 against 568.5, far too close to mean anything. A keyword changes the
shift from letter to letter, the tally comes out flat, and a flat tally has
nothing in it to line up. That is the three hundred years Vigenère went unbroken,
demonstrated rather than asserted. Both ciphers are checked against the examples
every account of them uses.

The technique underneath is the most lux-shaped thing in this folder. lux will
not give you one character out of a string and refuses an empty separator, so
walking a string is closed off — fine for shifting every letter the same way, and
fatal for a keyword, where the same letter needs a different shift depending on
where it sits. But `split` gives the positions back: splitting on "a" leaves the
runs between the a's, and running lengths along those runs says exactly where
each one was. Do that for every character you care about and you have rebuilt,
out of a function that hides positions, precisely where every character is.
Thirty-odd passes to learn what one subscript would have told you — not
efficient, but worth seeing, because a thing you were not given can often be
rebuilt out of the things you were.

`weekday.lux` is the one that does not search at all. Give it a date and it works
out the day of the week by arithmetic, twice over, by two methods with nothing in
common — Zeller's congruence from 1882, which moves the start of the year to
March so February's changing length lands where it cannot disturb anything, and a
plain count of days from a fixed point. It complains if they disagree. They do
not: every date from 1583 to 2400 was checked, all 298,769, with no disagreement,
no gap in the day count and no break in the weekday sequence.

It also settles something countable. The thirteenth falls on a Friday more often
than on any other day, and the Gregorian calendar closes after exactly four
hundred years — 146,097 days, 20,871 weeks, nothing left over — so this is a
count rather than an argument. All 4,800 thirteenths in one cycle: Friday 688,
Sunday and Wednesday 687, Monday and Tuesday 685, Thursday and Saturday 684. The
same counts come out starting from 1600, 2000 or 2400, which is the cycle proving
itself. It is uneven because 4,800 does not divide by seven, so something has to
be left over, and it turns out to be Friday.

`dashboard.lux` is the one that looks outward. Everything else here makes its
own data; this reads the machine — uptime, load against the core count, memory,
and disk — through the two doors a program has onto the world, `readFile` for
the files the kernel keeps in /proc and `run` for asking another program. Both
hand failure back as a value rather than a crash, so a line it cannot fill in
says so and the rest of the report still prints. The /proc reads are Linux; run
it anywhere else and what you get is the program explaining itself line by line,
which is the failure working rather than a bug.

`rule30.lux` is the smallest program here and the one with the most going on.
A row of cells, each on or off; to work out a cell's next state you look at it
and its two neighbours, and a rule is nothing but an answer for each of the
eight neighbourhoods three cells can make — eight yes-or-nos, a number from 0 to
255. Rule 30 starts from a single cell and produces stripes down its left side
and, down its right, something nobody has found a shortcut for. Change one
constant: 90 draws Sierpinski's triangle, 110 can compute anything a computer
can, 184 models traffic jams.

`life.lux` is Conway's Game of Life, and it does not only draw. Four rules, none
of which mentions a shape: a living cell with two or three living neighbours
lives on, one with fewer or more dies, and a dead cell with exactly three comes
alive. Everything else falls out of that. Nobody put a glider in.

So the program watches. After each generation it keeps the list of live cells,
and if an arrangement it has already seen comes round again it can say what it
was looking at — same cells in the same place is a still life or an oscillator
and the gap is the period, same cells somewhere else is a spaceship and the gap
between the positions is how far it travels. It names a glider without being
told that gliders exist. No signature string is built and nothing is hashed:
the live-cell list is in scan order on a flat array, so shifting a pattern adds
the same amount to every entry, which makes the gaps between the entries the
thing that survives a move.

Every verdict was checked against what these patterns are known to do. The
blinker, toad and beacon come back at period 2, the pulsar at 3, the
pentadecathlon at 15, the glider every 4 generations one square diagonally and
the lightweight spaceship every 4 two squares sideways. The diehard dies out
completely at generation 130. The r-pentomino settles at 1,103 and the acorn at
5,206, both from a handful of cells.

Those last figures are why the grid sizes are measured rather than chosen. A
grid too small does not crop the picture, it changes the answer and says nothing
about having done so: on 40 by 20 the diehard runs into the wall at generation 64
and gets reported as a still life, when what it actually does is vanish. Each
size here is the smallest that was tried where making it bigger stopped changing
the verdict, and when a pattern does reach the edge the program says so and says
that the verdict is about a grid of that size rather than about Life. The
r-pentomino is the honest case: it settles at 1,103 whatever the grid, but the
six gliders it throws off die against the wall, so 110 cells are left where an
endless sheet would have 116.

It is the second program here where building changes what it is rather than how
long it takes, and by more than connect four does. The diehard's 130 generations
take about fourteen seconds under `lux run` and four thousandths of a second
built. The acorn needs 5,206 generations on a grid of 340 by 260, which is
seventeen seconds built and was never attempted the other way.

`morse.lux` goes both ways — `lux run morse.lux hello world`, or `-d` and a
string of dots and dashes to come back. It is built the way lux forces rather
than the way the textbook prints: with no way to take a single character out of
a string, encoding replaces every occurrence of each letter across the whole
text, thirty-six passes over the text instead of one pass over the alphabet.
Decoding needs no characters at all, since Morse arrives pre-cut by its own
spaces and `split` takes it apart along them. Run it with no arguments and it
sends a sentence one way and brings it back the other.

`drill.lux` is times tables, timed. `lux run drill.lux 7` for the sevens, a
second number for how many questions, and at the end it reports how long each
answer took and which facts are worth another look. The clock is `date`, asked
before and after each question, which costs a few thousandths of a second per
reading — irrelevant when what is being measured is a child thinking, and fatal
if it were code.

`runes.lux` writes English in Tolkien's runes and reads it back. Not
translation — transliteration, one mark at a time, which is exactly what the
runes on Thror's map are: ordinary English that Bilbo's readers decode from the
chart on the compass rose. Tolkien did not invent the letters; he took the
Anglo-Saxon futhorc and wrote modern English in it, calling the result "English
runes". The output is the real Unicode runic block from U+16A0 up, so the
characters are the characters and will paste into anything that can show them —
including three that exist because of Tolkien. The futhorc had no K of the shape
he needed, no SH, and no rune for the OO of "soon", so he drew his own, and when
the runic block was first encoded they had nowhere to go. Unicode 7.0 added them
in 2014 as U+16F1, U+16F2 and U+16F3, and this uses two of the three.

The chart follows Per Lindberg's *Tolkien's English Runes* (Mellonath Daeron,
2023), which reads the inscriptions in the book and tabulates what Tolkien
actually used, and naming a source that precisely matters: the popular rune
charts disagree with each other, and the first version of this program was
wrong in six places because it was assembled from them. Tolkien's mode is
orthographic, one rune per letter, with exceptions that are the whole interest
of it. J shares I's rune and V shares U's. Q is not a letter at all — it is
written CW, so QUEEN comes out CWEEN. OO is two sounds and two runes: the [u] of
"soon" has its own, while the [o] of "door" is an ordinary O written with a
single ᚩ, which is why the map says DOR — and English settles which is meant
almost perfectly by whether an R follows. Six pairs get a rune apiece: TH, NG,
EE, EA, ST and OO. Only aesc goes unused, because telling an [æ] from an [a] is
something a program cannot hear.

Run it with no arguments and it checks itself rather than demonstrating itself.
The dust jacket of The Hobbit carries a line of Tolkien's own runes, so that
line is in the source and the program transliterates the same words and
compares — allowing one documented substitution, since the only Unicode
transcription of it was made in 2009, five years before the K it needed existed,
and writes cen twice instead. Everything else has to agree. It also reproduces the door
inscription's `f[ee]t` and `[th]r[ee]` without being shown them.

Reading back is deliberately not the mirror of writing. A J returns as an I, a V
as a U, DOOR as DOR, QUEEN as CWEEN. Each of those is Tolkien's own system doing
what it does, and hiding it would mean inventing runes he never used.

The ordering of the chart is the whole program. Each replacement sweeps the
entire text, so a piece containing another piece has to be tried first — "th"
before "t" and "h", or THE comes out three runes long. The same order run
backwards decodes, and where two letters share a rune the one listed first is
the one that comes home.

Three limits shaped these more than anything else, and each one left a mark
worth reading. lux cannot hand you a single character out of a string, so
hangman's words are written as arrays of letters — which is what a word is
underneath anyway. lux has no lookup table, so the same alphabet is written out
twice, in lower case and in capitals, and the pairing between the two rows is
how a typed letter gets lower-cased. And a lux array grows but never shrinks, so
the maze's trail back is an array plus a marker saying how much of it still
counts, with the next push writing over whatever was abandoned above it.

## Running them

Each program is one file and needs nothing but lux:

```
lux run mandelbrot.lux
lux run poker.lux
lux run todo.lux add walk the dog
lux run flash.lux add der Hund = the dog
```

`lux convert rust|swift|go <file>` prints any of them as real source in that
language, and `lux build <file>` compiles a native binary. Every program here
converts on all three and compiles warning-clean.

Written by David M. Anderson, with the assistance of Claude (Anthropic).
MIT licensed.
