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
against a published 85.2%. That last case sets the honest limit: at these trial
counts the answer is good to about half a percent, which is why the tutor quotes
whole numbers and says "about". The tutor also says out loud what its own number
leaves out — equity against *any two cards* is not equity against the hand of
somebody who just bet into you.

The bot knows its own cards, the pot, and nothing else, and it comes in three
strengths you pick before the first hand. The interesting part is how the easy
one is made weak: not by handing it bad rules but by giving it a hundred and
fifty trials to read its own hand instead of nine hundred, so it genuinely
misjudges what it is holding. On top of that it calls past the price it is being
offered and never bluffs. Bet into it across thirty hands and it folds not once.
The hard bot pays exactly the right price, bets thinner than is comfortable, and
bluffs about one hand in four; the same mindless betting that breaks even against
the easy bot loses half a stack to it.

Because the opponent differs, the advice differs with it. Against the bot that
calls too much the tutor lowers the bar for a value bet — you can bet worse hands
when worse hands get called — and tells you to stop bluffing, because a bluff
against something that never folds is money handed over. Against the bot that
bluffs one hand in four it says the opposite: its bet is less news than it looks.
That is the half of poker a pot-odds calculation leaves out, and having three
opponents to feel the difference against is the only honest way to teach it.

When you leave, it tells you how you played. Every decision is scored against
the advice that was on the screen when you made it — how many matched, how often
you called past the price and what those calls were worth by the odds at the
time, how often you folded a price worth paying or checked a hand that wanted
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
Anglo-Saxon futhorc and wrote modern English in it. The output is the real
Unicode runic block from U+16A0 up, so the characters are the characters and
will paste into anything that can show them.

Run it with no arguments and it checks itself rather than demonstrating itself.
The dust jacket of The Hobbit carries a full line of Tolkien's own runes, so
that line is in the source, and the program transliterates the same words and
compares. It matches rune for rune — including the doubled ᚳᚳ Tolkien used for
the "ck" in BACK, which is his spelling rather than a letter of the alphabet.
That reading is where the mapping for TH, E, H, O, B, I, T, R, A, N, D, G and
the word separator comes from; the digraph runes for EE, OO, EA and ST, and
calc for K, follow the futhorc's own values, and the source marks which is
which.

The ordering of the chart is the whole program. Each replacement sweeps the
entire text, so a piece containing another piece has to be tried first — "th"
before "t" and "h", or THE comes out three runes long. The same order run
backwards does the decoding, where ᚳᚳ must be read as "ck" before ᚳ is read as
"c". Two arrays whose sequence carries as much meaning as their contents.

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
```

`lux convert rust|swift|go <file>` prints any of them as real source in that
language, and `lux build <file>` compiles a native binary. Every program here
converts on all three and compiles warning-clean.

Written by David M. Anderson, with the assistance of Claude (Anthropic).
MIT licensed.
