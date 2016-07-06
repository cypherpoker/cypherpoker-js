# mental-poker-js

Mental Poker API implemented in JavaScript, based on the
[thesis of Choongmin Lee](http://www.clee.kr/thesis.pdf).

## Introduction

Mental Poker makes it possible to play a fair game of poker over a physical
distance without the need for a trusted third party, using cryptographic
methods to shuffle and then deal from a deck of cards.

According to the specification of Choongmin Lee, a coalition, even if it is of
the maximum size, cannot gain advantage over honest players except that players
in the coalition can share their own hands.

## Getting started

### Establishing a game

Firstly, every player should generate points of an elliptic curve, serving as a
deck of cards, on the client side. Players must share their generated points
with each other in order to finish setting up a new game.

```js
import { Game, Player } from 'mental-poker-js';

// Points and secrets of self are auto-generated
const players = [new Player()]; // More players should be present
const game = new Game(players);

// Broadcast `game.playerSelf.points` and receive the points of other players
// After that, the initial deck generation process should occur
game.generateInitialDeck();
```

### Cascaded shuffling

Each player sequentially shuffles the order of the game's deck points, keeping
the result in secret by encrypting it as a whole using elliptic curve point
multiplication.

```js
// Receive an encrypted deck from an opponent if not acting first in the turn
if (encryptedDeckOfAnOpponent) {
  game.addDeckToSequence(encryptedDeckOfAnOpponent);
}

// Shuffle the deck by self and then pass it to the next opponent
const deck = game.shuffleDeck();
```

### Locking the deck

Each player sequentially locks the game's deck points, keeping the result in
secret by encrypting the points with different keys using elliptic curve point
multiplication.

```js
// Receive an encrypted deck from an opponent
game.addDeckToSequence(encryptedDeckOfAnOpponent);

// Decrypt, lock and then pass the deck to the next opponent in turn
const deck = game.lockDeck();
```

### Drawing a card

In the previous step, locking was done using a different secret key for each
card. In order to unlock a single card of the deck, every player must provide
the secret corresponding to the locked card at the selected index.

In order to open a card, the secret shall be broadcast to every participant.
Otherwise, each player should only reveal the secret for the person drawing the
card.

```js
// Select a random unowned index of the deck
const cardIndex = game.getRandomUnownedCardIndex();

// Obtain the secret of each opponent at the given card index
// After that, the card can be unlocked
const cardId = game.pickCard(cardIndex);

// Draw the card for self without revealing it
game.drawCard(cardIndex);

// Or open it as a community card on the table
game.openCard(cardIndex);
```

## Performance

Execute `npm start` to run a benchmark which measures the performance of each
step mentioned above.
