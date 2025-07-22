---
title: Monopoly Deal and Reinforcement Learning - Building an environment.
date: 2025-07-21T02:13:04Z
image:
author: James Malcolm
description: Let's explore an applied area of reinforcement learning on Monopoly Deal by starting to build a gym environment. Part one of a series.
tags:
  - data
  - reinforcement learning
  - ai
---

Join me on a series of posts about training a reinforcement learning from scratch to play Monopoly Deal.

This post isn't designed as a polished, run-it-once-and-it-works tutorial - there are plenty of those for reinforcement learning. Instead, it's a training journal documenting real issues and solutions as they arise.

If you're wanting introduction to reinforcement learning, StableBaselines is a good starting point, and a package we'll be using later. I really like Joseph Suarez, author and contributor to [PufferLib](https://github.com/PufferAI/PufferLib) reinforcement learning [quick start](https://x.com/jsuarez5341/status/1854855861295849793) here as well.

{{< collapse summary="Reinforcement Learning Primer">}}

Reinforcement Learning (RL) is an area of machine learning where an **agent** learns to make decisions by interacting with an **environment**. The goal is for the agent to maximize its cumulative **reward** over time by learning the best sequence of **actions** to take in various situations.

Think of training a dog: a treat (reward) for a correctly performed trick (action) reinforces that behavior. RL powers a lot from game-playing AI, like AlphaGo, to robotic control and automated trading strategies. For a deep dive, Sutton and Barto's ["Reinforcement Learning: An Introduction"](http://incompleteideas.net/book/the-book-2nd.html) is the classic text.

To experiment with RL, you need a robust way to define these environments and agents. This is where **Gymnasium** comes in. Developed by OpenAI (as a continuation of "Gym"), Gymnasium is the de-facto standard toolkit for creating and comparing RL algorithms. It provides a universal interface to a vast collection of simulated environments, from classic control problems like balancing a pole on a cart to physics simulations. You can explore its features and environments on the official [Gymnasium website](https://gymnasium.farama.org/).

While Gymnasium provides the playground, **Stable-Baselines3** provides the players. It's a set of high-quality, reliable implementations of state-of-the-art RL algorithms built on PyTorch. Instead of coding complex algorithms like Proximal Policy Optimization (PPO) or Soft Actor-Critic (SAC) from scratch, we can import them with a single line of code. This powerful library makes it incredibly easy to train, evaluate, and deploy high-performance agents in Gymnasium environments. You can get started quickly by checking out the [Stable-Baselines3 documentation](https://stable-baselines3.readthedocs.io/en/master/).

{{</ collapse >}}

# Let’s build a base environment

Monopoly Deal is a card-based strategy game based on the board game monopoly. There is existing research out there on using reinforcement learning for Monopoly the board game, but limited to none out there for monopoly deal.

Deal differs to Monopoly the board game in a few ways which makes it tricker to model. The first is that there is no random-dice roll, each player starts their turn by picking up two additional cards and can play up to three moves. Up to three moves is the key here, as there may be a valid strategy in not playing a move or two out of the three moves allowed.

To get an interesting environment, we need action cards such as Sly Deal or Forced Deal, as well as rent cards. Otherwise, it can become just luck of the draw to build sets. We’ll begin solely focused on a two-player game.

**Note:** We start at a two-player game here for simplicity. In later posts we’ll explore expanding it. Adding additional players is a fair bit more complex as not only are you thinking about if to play a Sly Deal, but you’re now having to think about *who* to play the Sly Deal against.

Ok, so let’s take a look at our outline for a base environment here:

```python
import gymnasium as gym


class MonopolyDealEnv(gym.Env):

    def __init__(self):
        pass

    def step(self, action):
        pass

    def reset(self):
        pass

    def _create_deck(self):
        pass

    def _play_action_card(self):
        pass

    def _charge_rent(self):
        pass

    def _sly_deal(self):
        pass

    def _deal_breaker(self):
        pass
```

We can see from the code above, we have a really bare skeleton of a game. A player can take a step, play action cards, play deal breakers etc.

Let’s begin by adding in the constraints of the game, namely:

* Max hand size = 7
* Win condition = 3
* Number of players = 2
* Actions per turn = 3

In code, we can define these parameters in the `__init__` function.

```python
class MonopolyDealEnv(gym.Env):
	def __init__(self):
		self.MAX_HAND_SIZE = 7
		self.WIN_CONDTION = 3
		self.NUM_PLAYERS = 2
		self.ACTIONS_PER_TURN = 3
	...
```

### Show me the cards

We can define the deck, by creating a list of cards available and multiplying them by their quantity contained in a standard deck, available [here]( [https://monopolydealrules.com/index.php?page=cards#top](https://monopolydealrules.com/index.php?page=cards#top)).

```python
class CardType(Enum):
    MONEY_1 = 0
    MONEY_2 = 1
    MONEY_3 = 2
    MONEY_4 = 3
    MONEY_5 = 4
    PROP_BROWN = 5
    PROP_LIGHT_BLUE = 6
    PROP_PINK = 7
    PROP_ORANGE = 8
    PROP_RED = 9
    PROP_YELLOW = 10
    PROP_GREEN = 11
    PROP_BLUE = 12
    RENT_WILD = 13
    SLY_DEAL = 14
    FORCED_DEAL = 15
    DEAL_BREAKER = 16
    JUST_SAY_NO = 17

class MonopolyDealEnv(gym.Env):
	...

	def _create_deck(self) -> List[CardType]:
		"""For each card type, multiple how many cards there in a standard deck.
		Shuffle deck.
		"""
	    deck = []
	    deck.extend([CardType.MONEY_1] * 6)
	    deck.extend([CardType.MONEY_2] * 5)
	    deck.extend([CardType.MONEY_3] * 3)
	    deck.extend([CardType.MONEY_4] * 3)
	    deck.extend([CardType.MONEY_5] * 2)
	    deck.extend([CardType.PROP_BROWN] * 2)
	    deck.extend([CardType.PROP_LIGHT_BLUE] * 3)
	    deck.extend([CardType.PROP_PINK] * 3)
	    deck.extend([CardType.PROP_ORANGE] * 3)
	    deck.extend([CardType.PROP_RED] * 3)
	    deck.extend([CardType.PROP_YELLOW] * 3)
	    deck.extend([CardType.PROP_GREEN] * 3)
	    deck.extend([CardType.PROP_BLUE] * 2)
	    deck.extend([CardType.RENT_WILD] * 3)
	    deck.extend([CardType.SLY_DEAL] * 3)
	    deck.extend([CardType.FORCED_DEAL] * 2)
	    deck.extend([CardType.DEAL_BREAKER] * 2)
	    deck.extend([CardType.JUST_SAY_NO] * 3)
	    random.shuffle(deck)
	    return deck
	...
```

Reinforcement learning can either be episodic, or continous. Games are naturally episodic, in the sense you die/win/fail the game is reset and you a respawned. Self-driving cars are also episodic in that if they crash, the episode is reset.

To reset the game, we can draw on real-life and what will happen to start a new game of monopoly deal. These are:

* Shuffle the cards
* Clear the discard pile (table if physical)
* Dish out new cards to players

In the reinforcement learning world - this is the same! Although we need to clear some other variables we store about the previous games. We implement this reset in the `reset()` function of the gym. Here, we reset everything back to a clean state.

```python
class MonopolyDealEnv(gym.Env):
	...

	def reset(self) -> List[CardType]:
		"""For each card type, multiple how many cards there in a standard deck.
		Shuffle deck.
		"""
	    self.deck = self._create_deck()  # Get new set of cards
	    self.discard_pile = []
		self.players = [] for i in range(self.NUM_PLAYERS):
			player = {
				'hand': [],
				'properties': {color: 0 for color in self.PROPERTY_SETS.keys()},
				'money_pile': 0,
				'complete_sets': [],
				'just_say_no_cards': 0
			}
        # Deal initial hand
		for _ in range(5):
			if self.deck:
				player['hand'].append(self.deck.pop())

			self.players.append(player)

		self.current_player = 0
		self.turn_actions_taken = 0
		self.max_actions_per_turn = 3
		self.game_over = False
		self.winner = None
		return self._get_observation()
	...
```

## What can a player do?

At the heart of reinforcement learning is deciding what an action an agent can do. In our case, let's look at the options:

* Agent can pass. It's always a valid, legal move
* Play card as a property
* Play an action card
* Play as money

More formally, we can define this as:
```
Action Space: A = A_type × A_card × A_target × A_property 

Where: 
- A_type ∈ {0,1,2,3} = {pass, money, property, action_card}
- A_card ∈ {0,1,...,6} = hand positions
- A_target ∈ {0,1} = opponent selection
- A_property ∈ {0,1,...,7} = property color targets 

Total action space: |A| = 4 × 7 × 2 × 8 = 448 actions
```

In code, we can define this as:

```python
from gymnasium import spaces


class MonopolyDealEnv(gym.Env):
	def __init__(self):
		...
		self.action_space = spaces.MultiDiscrete([4, 7, 2, 8])
	...
```

Cool. Now that we have the action space, how should we reward actions? Recall, that reinforcement learning is all about making actions to maximise a reward.

What actions give rewards? Well, we assign this. For example, winning the game gives the largest rewards! Playing a property correct should give a smaller but positive reward, whereas playing a rent card incorrectly should give a negative reward.

More formally, Our reward structure follows potential-based shaping theory:

F(s,a,s') = γΦ(s') - Φ(s)

Where Φ(s) is a potential function measuring "progress toward goal." This transformation preserves optimal policies while providing dense signals.

For Monopoly Deal:
Φ(s) = Σᵢ wᵢ · fᵢ(s)

Where:
- f₁(s) = complete_sets(s) × 200     # Terminal objective
- f₂(s) = property_progress(s) × 25  # Intermediate progress
- f₃(s) = exploration_bonus(s) × 20  # Encourage diversity

We can mock this up in code by fleshing out our template above:

```python
class MonopolyDealEnv(gym.Env):

	def _play_as_money(self, card_type: CardType) -> float:
	"""Play card as money."""
		money_value = self._get_money_value(card_type)
		self.players[self.current_player]['money_pile'] += money_value

		# Penalize playing properties as money (they should build sets!)
		if card_type.value >= 5: # This is a property card
			return -2 # Negative reward for "wasting" a property
		else:
			return money_value * 0.5

	def _play_as_property(self, card_type: CardType) -> float:
		"""Play card as property."""
		color = self._get_property_color(card_type)
		if not color:
			return -1 # Invalid property
		player = self.players[self.current_player]
		old_count = player['properties'][color]
		player['properties'][color] += 1
		new_count = player['properties'][color]
		# Check if this completes a set
		set_size = self.PROPERTY_SETS[color]['size']

		if new_count >= set_size and color not in player['complete_sets']:
			player['complete_sets'].append(color)
			return 200 # Large reward for completing a set
		# Curiosity bonus for trying new colors
		if color not in self.colors_attempted:
			self.colors_attempted.add(color) bonus = 20 # Exploration bonus
		else:
			bonus = 0

		# much higher progressive rewards for building towards sets
		if new_count == 1:
			return 10 + bonus # First property of a color
		elif new_count == 2:
			return 25 + bonus # Second property (closer to completion)
		else:
			return 15 + bonus # Additional properties
```

At this stage, reward modelling feels like a bit of an art. We are imparting our knowledge of the game by rewarding actions that we think are positive and that we think are negative.

# Let's simulate a turn

Ok, let's think about what happens every time someone plays a turn.

Each turn in Monopoly Deal follows this sequence:

1. **Draw Phase**: Pick up 2 cards (if deck available)
2. **Action Phase**: Choose up to 3 actions from:
   - Play properties to build sets
   - Play money cards for liquidity
   - Use action cards (Rent, Sly Deal, etc.)
   - Pass (always valid)
3. **Discard Phase**: Reduce hand to 7 cards maximum
4. **Win Check**: Game ends if any player has 3 complete property sets

This structure maps to our RL formulation as:
- **State**: Current game configuration (hands, properties, money)
- **Action**: One of our 448 possible moves
- **Reward**: Immediate feedback based on strategic value
- **Transition**: New game state after action execution

We can sketch some pseudocode for this:

```python
def step(self, action):
    # 1. Validate
    if not self._is_valid_action(action):
        return self._get_observation(), -1, False, False, {}

    # 2. Execute
    reward = self._execute_action(action)

    # 3. Update state & 4. Check win
    self._check_win_condition()

    # 5. Return results
    return self._get_observation(), reward, self.game_over, False, {}
```

Now, let's flesh it out with our actual implementation. Recall, that action is a Tuple of four components: action_type, card_idx, target_player, and target_color.

```python
class MonopolyDealEnv(gym.Env):
    ...

    def step(self, action: Tuple[int, int, int, int]) -> Tuple[np.ndarray, float, bool, bool, Dict]:
        """Execute one step in the environment."""
        if self.game_over:
            return self._get_observation(), 0, True, False, {}

        self.total_steps += 1

        action_type, card_idx, target_player, target_color = action
        reward = 0
        info = {}

        if not self._is_valid_action(action):
            return self._get_observation(), -1, False, False, {'invalid_action': True}

        current_hand = self.players[self.current_player]['hand']

        if action_type == 3:  # Pass
            reward = self._end_turn()
        else:
            card_type = current_hand[card_idx]
            current_hand.pop(card_idx)
            self.turn_actions_taken += 1

            if action_type == 0:  # Play as money
                reward = self._play_as_money(card_type)
            elif action_type == 1:  # Play as property
                reward = self._play_as_property(card_type)
            elif action_type == 2:  # Play action card
                reward = self._play_action_card(card_type, target_player, target_color)

            # Draw back up to hand limit if needed
            while len(current_hand) < 5 and self.deck:
                current_hand.append(self.deck.pop())

            # End game if deck is empty
            if not self.deck:
                self.game_over = True
                # Winner is player with most complete sets
                max_sets = max(len(player['complete_sets']) for player in self.players)
                agent_sets = len(self.players[0]['complete_sets'])

                if agent_sets == max_sets:
                    # Agent ties or wins
                    if agent_sets > 0:  # Agent actually has sets
                        self.winner = 0  # Agent wins on tiebreaker
                        return self._get_observation(), 50, True, False, {'winner': 'deck_empty_win'}
                    else:
                        self.winner = None  # True tie, no one wins
                        return self._get_observation(), 10, True, False, {'winner': 'deck_empty_tie'}
                else:
                    # Opponent wins
                    self.winner = 1  # Opponent wins
                    return self._get_observation(), -30, True, False, {'winner': 'deck_empty_loss'}

            # End turn if max actions reached
            if self.turn_actions_taken >= self.max_actions_per_turn:
                self._end_turn()

        # Check win condition
        self._check_win_condition()

        # Add win/loss rewards if game ended
        if self.game_over and hasattr(self, 'winner'):
            if self.winner == 0:
                reward += 100  # Agent wins
            elif self.winner == 1:
                reward -= 50   # Agent loses

        return self._get_observation(), reward, self.game_over, False, info

```

This has a lot more detail to it than the pseudocode, but the principles remain the same.

## Training Issue #1: Stalling

{{< collapse summary="Full code at this point">}}

For brevity of not explaining each piece of code, here is the full script at this stage. There are two:

environment.py:

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np
import random
from typing import List, Dict, Tuple, Optional
from enum import Enum


class CardType(Enum):
    MONEY_1 = 0
    MONEY_2 = 1
    MONEY_3 = 2
    MONEY_4 = 3
    MONEY_5 = 4
    PROP_BROWN = 5
    PROP_LIGHT_BLUE = 6
    PROP_PINK = 7
    PROP_ORANGE = 8
    PROP_RED = 9
    PROP_YELLOW = 10
    PROP_GREEN = 11
    PROP_BLUE = 12
    RENT_WILD = 13
    SLY_DEAL = 14
    FORCED_DEAL = 15
    DEAL_BREAKER = 16
    JUST_SAY_NO = 17


class MonopolyDealEnv(gym.Env):

    def __init__(self):
        super().__init__()
        self.MAX_HAND_SIZE = 7
        self.WIN_CONDITION = 3
        self.NUM_PLAYERS = 2

        # Property set definitions (properties_needed, base_rent, full_set_rent)
        self.PROPERTY_SETS = {
            'BROWN': {'size': 2, 'rent': [1, 2], 'cards': [CardType.PROP_BROWN]},
            'LIGHT_BLUE': {'size': 3, 'rent': [1, 2, 3], 'cards': [CardType.PROP_LIGHT_BLUE]},
            'PINK': {'size': 3, 'rent': [1, 2, 4], 'cards': [CardType.PROP_PINK]},
            'ORANGE': {'size': 3, 'rent': [1, 3, 5], 'cards': [CardType.PROP_ORANGE]},
            'RED': {'size': 3, 'rent': [2, 3, 6], 'cards': [CardType.PROP_RED]},
            'YELLOW': {'size': 3, 'rent': [2, 4, 6], 'cards': [CardType.PROP_YELLOW]},
            'GREEN': {'size': 3, 'rent': [2, 4, 7], 'cards': [CardType.PROP_GREEN]},
            'BLUE': {'size': 2, 'rent': [3, 8], 'cards': [CardType.PROP_BLUE]},
        }

        # Money values
        self.MONEY_VALUES = {
            CardType.MONEY_1: 1,
            CardType.MONEY_2: 2,
            CardType.MONEY_3: 3,
            CardType.MONEY_4: 4,
            CardType.MONEY_5: 5
        }

        self.action_space = spaces.MultiDiscrete([4, 7, 2, 8])

        # Observation space
        obs_size = (
                len(CardType) +  # Hand composition (count of each card type)
                8 * self.NUM_PLAYERS +  # Property counts per color per player
                8 * self.NUM_PLAYERS +  # Complete set status per color per player
                self.NUM_PLAYERS +  # Money for each player
                self.NUM_PLAYERS +  # Complete sets count for each player
                4  # Game state (current_player, turn_actions_taken, cards_in_deck, discard_pile_size)
        )

        self.observation_space = spaces.Box(
            low=0, high=20, shape=(obs_size,), dtype=np.int32
        )

        self.reset()

    def _create_deck(self) -> List[CardType]:
        deck = []

        # Money cards (varying distribution)
        deck.extend([CardType.MONEY_1] * 6)
        deck.extend([CardType.MONEY_2] * 5)
        deck.extend([CardType.MONEY_3] * 3)
        deck.extend([CardType.MONEY_4] * 3)
        deck.extend([CardType.MONEY_5] * 2)

        # Property cards (2-4 of each type based on rarity)
        deck.extend([CardType.PROP_BROWN] * 2)
        deck.extend([CardType.PROP_LIGHT_BLUE] * 3)
        deck.extend([CardType.PROP_PINK] * 3)
        deck.extend([CardType.PROP_ORANGE] * 3)
        deck.extend([CardType.PROP_RED] * 3)
        deck.extend([CardType.PROP_YELLOW] * 3)
        deck.extend([CardType.PROP_GREEN] * 3)
        deck.extend([CardType.PROP_BLUE] * 2)

        # Action cards (limited and powerful)
        deck.extend([CardType.RENT_WILD] * 3)
        deck.extend([CardType.SLY_DEAL] * 3)
        deck.extend([CardType.FORCED_DEAL] * 2)
        deck.extend([CardType.DEAL_BREAKER] * 2)
        deck.extend([CardType.JUST_SAY_NO] * 3)

        random.shuffle(deck)
        return deck

    def _get_property_color(self, card_type: CardType) -> Optional[str]:
        """Get the property color for a card type."""
        for color, info in self.PROPERTY_SETS.items():
            if card_type in info['cards']:
                return color
        return None

    def _can_play_as_money(self, card_type: CardType) -> bool:
        """Check if card can be played as money."""
        return card_type in self.MONEY_VALUES or card_type.value >= 5  # Properties can be money

    def _get_money_value(self, card_type: CardType) -> int:
        """Get money value of a card."""
        if card_type in self.MONEY_VALUES:
            return self.MONEY_VALUES[card_type]
        elif card_type.value >= 5:  # Property cards
            return 1  # Properties are worth 1M when used as money
        return 0

    def reset(self, seed=None, options=None) -> tuple:
        """Reset the environment to initial state."""
        # Handle seed for reproducibility
        if seed is not None:
            random.seed(seed)
            np.random.seed(seed)

        self.deck = self._create_deck()
        self.discard_pile = []

        # Initialize players
        self.players = []
        for i in range(self.NUM_PLAYERS):
            player = {
                'hand': [],
                'properties': {color: 0 for color in self.PROPERTY_SETS.keys()},
                'money_pile': 0,
                'complete_sets': [],
                'just_say_no_cards': 0
            }
            # Deal initial hand
            for _ in range(5):
                if self.deck:
                    player['hand'].append(self.deck.pop())
            self.players.append(player)

        self.current_player = 0
        self.turn_actions_taken = 0
        self.max_actions_per_turn = 3
        self.game_over = False
        self.winner = None

        return self._get_observation(), {}

    def _get_observation(self) -> np.ndarray:
        """Get current state observation."""
        obs = []

        # Current player's hand composition
        hand_counts = [0] * len(CardType)
        current_hand = self.players[self.current_player]['hand']
        for card in current_hand:
            hand_counts[card.value] += 1
        obs.extend(hand_counts)

        # Property counts for each player
        for player in self.players:
            for color in self.PROPERTY_SETS.keys():
                obs.append(player['properties'][color])

        # Complete set status for each player (0/1 for each color)
        for player in self.players:
            for color in self.PROPERTY_SETS.keys():
                set_size = self.PROPERTY_SETS[color]['size']
                is_complete = 1 if player['properties'][color] >= set_size else 0
                obs.append(is_complete)

        # Money for each player
        for player in self.players:
            obs.append(player['money_pile'])

        # Complete sets count for each player
        for player in self.players:
            obs.append(len(player['complete_sets']))

        # Game state
        obs.extend([
            self.current_player,
            self.turn_actions_taken,
            len(self.deck),
            len(self.discard_pile)
        ])

        return np.array(obs, dtype=np.int32)

    def _is_valid_action(self, action: Tuple[int, int, int, int]) -> bool:
        """Check if an action is valid."""
        action_type, card_idx, target_player, target_color = action
        current_hand = self.players[self.current_player]['hand']

        # Pass action is always valid
        if action_type == 3:
            return True

        # Check if card index is valid
        if card_idx >= len(current_hand):
            return False

        card_type = current_hand[card_idx]

        if action_type == 0:  # Play as money
            return self._can_play_as_money(card_type)

        elif action_type == 1:  # Play as property
            return card_type.value >= 5  # Only property cards

        elif action_type == 2:  # Play action card
            if card_type in [CardType.RENT_WILD, CardType.SLY_DEAL, CardType.FORCED_DEAL, CardType.DEAL_BREAKER]:
                return target_player != self.current_player
            return card_type in [CardType.JUST_SAY_NO]

        return False

    def step(self, action: Tuple[int, int, int, int]) -> Tuple[np.ndarray, float, bool, bool, Dict]:
        """Execute one step in the environment."""
        if self.game_over:
            return self._get_observation(), 0, True, False, {}

        action_type, card_idx, target_player, target_color = action
        reward = 0
        info = {}

        if not self._is_valid_action(action):
            return self._get_observation(), -5, False, False, {'invalid_action': True}

        current_hand = self.players[self.current_player]['hand']

        if action_type == 3:  # Pass
            reward = self._end_turn()
        else:
            # Play the card
            card_type = current_hand[card_idx]
            current_hand.pop(card_idx)
            self.turn_actions_taken += 1

            if action_type == 0:  # Play as money
                reward = self._play_as_money(card_type)
            elif action_type == 1:  # Play as property
                reward = self._play_as_property(card_type)
            elif action_type == 2:  # Play action card
                reward = self._play_action_card(card_type, target_player, target_color)

            # Draw back up to hand limit if needed
            while len(current_hand) < 5 and self.deck:
                current_hand.append(self.deck.pop())

            # End turn if max actions reached
            if self.turn_actions_taken >= self.max_actions_per_turn:
                self._end_turn()

        # Check win condition
        self._check_win_condition()

        # New Gym API: return (obs, reward, terminated, truncated, info)
        return self._get_observation(), reward, self.game_over, False, info

    def _play_as_money(self, card_type: CardType) -> float:
        """Play card as money."""
        money_value = self._get_money_value(card_type)
        self.players[self.current_player]['money_pile'] += money_value
        return money_value * 0.5  # Small reward for gaining money

    def _play_as_property(self, card_type: CardType) -> float:
        """Play card as property."""
        color = self._get_property_color(card_type)
        if not color:
            return -1  # Invalid property

        player = self.players[self.current_player]
        player['properties'][color] += 1

        # Check if this completes a set
        set_size = self.PROPERTY_SETS[color]['size']
        if player['properties'][color] >= set_size and color not in player['complete_sets']:
            player['complete_sets'].append(color)
            return 25  # Big reward for completing a set

        return 3  # Small reward for property progress

    def _play_action_card(self, card_type: CardType, target_player: int, target_color: int) -> float:
        """Play an action card."""
        current_player_data = self.players[self.current_player]
        target_player_data = self.players[target_player]

        if card_type == CardType.RENT_WILD:
            return self._charge_rent(target_player, target_color)

        elif card_type == CardType.SLY_DEAL:
            return self._sly_deal(target_player, target_color)

        elif card_type == CardType.DEAL_BREAKER:
            return self._deal_breaker(target_player, target_color)

        return 0

    def _charge_rent(self, target_player: int, color_idx: int) -> float:
        """Charge rent for a property color."""
        colors = list(self.PROPERTY_SETS.keys())
        if color_idx >= len(colors):
            return -1

        color = colors[color_idx]
        current_player_data = self.players[self.current_player]
        target_player_data = self.players[target_player]

        # Calculate rent based on properties owned
        properties_owned = current_player_data['properties'][color]
        if properties_owned == 0:
            return -1  # Can't charge rent without properties

        set_info = self.PROPERTY_SETS[color]
        rent_amount = set_info['rent'][min(properties_owned - 1, len(set_info['rent']) - 1)]

        # Collect rent (take from money first, then properties as money)
        collected = min(rent_amount, target_player_data['money_pile'])
        target_player_data['money_pile'] -= collected
        current_player_data['money_pile'] += collected

        return collected * 2  # Good reward for successful rent

    def _sly_deal(self, target_player: int, color_idx: int) -> float:
        """Steal one property from target."""
        colors = list(self.PROPERTY_SETS.keys())
        if color_idx >= len(colors):
            return -1

        color = colors[color_idx]
        target_player_data = self.players[target_player]
        current_player_data = self.players[self.current_player]

        if target_player_data['properties'][color] > 0:
            target_player_data['properties'][color] -= 1
            current_player_data['properties'][color] += 1

            # Remove from complete sets if necessary
            if color in target_player_data['complete_sets']:
                set_size = self.PROPERTY_SETS[color]['size']
                if target_player_data['properties'][color] < set_size:
                    target_player_data['complete_sets'].remove(color)

            return 15  # Good reward for stealing

        return -2  # Penalty for invalid steal

    def _deal_breaker(self, target_player: int, color_idx: int) -> float:
        """Steal complete property set from target."""
        colors = list(self.PROPERTY_SETS.keys())
        if color_idx >= len(colors):
            return -1

        color = colors[color_idx]
        target_player_data = self.players[target_player]
        current_player_data = self.players[self.current_player]

        # Can only steal complete sets
        if color in target_player_data['complete_sets']:
            properties_to_steal = target_player_data['properties'][color]
            target_player_data['properties'][color] = 0
            current_player_data['properties'][color] += properties_to_steal

            target_player_data['complete_sets'].remove(color)
            if color not in current_player_data['complete_sets']:
                current_player_data['complete_sets'].append(color)

            return 30  # Huge reward for deal breaker

        return -5  # Big penalty for invalid deal breaker

    def _end_turn(self) -> float:
        """End current player's turn."""
        self.current_player = (self.current_player + 1) % self.NUM_PLAYERS
        self.turn_actions_taken = 0
        return 0

    def _check_win_condition(self):
        """Check if any player has won."""
        for i, player in enumerate(self.players):
            if len(player['complete_sets']) >= self.WIN_CONDITION:
                self.game_over = True
                self.winner = i
                return
```

train.py:

For this initial implementation, we use a PPO model. PPO is a simple but strong algorithm that [OpenAI used to solve DoTA](https://cdn.openai.com/dota-2.pdf). The focus of this post isn't on the model, that'll come with future posts, the focus is on getting an environment that works as we'd hope.

```python
import gymnasium as gym
from gymnasium import spaces
import numpy as np
import random
from stable_baselines3 import PPO, DQN, A2C
from stable_baselines3.common.env_util import make_vec_env
from stable_baselines3.common.callbacks import (
    EvalCallback, CheckpointCallback, CallbackList, BaseCallback
)
from stable_baselines3.common.monitor import Monitor
from stable_baselines3.common.vec_env import DummyVecEnv, SubprocVecEnv
from stable_baselines3.common.utils import set_random_seed
import matplotlib.pyplot as plt
import pandas as pd
import os
from typing import Dict, Any
import warnings

warnings.filterwarnings("ignore")

# Import our custom environment
from environments.initial import MonopolyDealEnv


class FlattenedMonopolyDealEnv(gym.Wrapper):
    """
    Wrapper to flatten the multi-discrete action space for algorithms that need it.
    This converts the multi-discrete action space to a single discrete action space.
    """

    def __init__(self, env):
        super().__init__(env)

        # Store original action space for conversion
        self.original_action_space = env.action_space

        # Calculate total number of possible actions
        self.action_mapping = []
        total_actions = 1
        for size in env.action_space.nvec:
            total_actions *= size

        # Create flattened action space
        self.action_space = spaces.Discrete(total_actions)

        # Build action mapping
        self._build_action_mapping()

    def _build_action_mapping(self):
        """Build mapping from flat actions to multi-discrete actions."""
        nvec = self.original_action_space.nvec
        self.action_mapping = []

        for flat_action in range(self.action_space.n):
            multi_action = []
            temp = flat_action

            for i in reversed(range(len(nvec))):
                multi_action.insert(0, temp % nvec[i])
                temp //= nvec[i]

            self.action_mapping.append(tuple(multi_action))

    def step(self, action):
        # Convert flat action to multi-discrete
        if isinstance(action, np.ndarray):
            action = action.item()

        multi_action = self.action_mapping[action]
        result = self.env.step(multi_action)

        # Ensure we always return 5 values for new Gym API
        if len(result) == 5:
            # New API: (obs, reward, terminated, truncated, info)
            return result
        elif len(result) == 4:
            # Old API: (obs, reward, done, info) - convert to new API
            obs, reward, done, info = result
            return obs, reward, done, False, info  # terminated=done, truncated=False
        else:
            raise ValueError(f"Unexpected number of values from step: {len(result)}")

    def reset(self, **kwargs):
        # Handle both old and new reset API
        result = self.env.reset(**kwargs)
        if isinstance(result, tuple):
            return result  # Return (obs, info) tuple
        else:
            return result, {}  # Convert old API to new API format

    def seed(self, seed=None):
        """Set the seed for the environment."""
        if hasattr(self.env, 'seed'):
            return self.env.seed(seed)
        else:
            # For environments that don't have seed method, set random seeds
            random.seed(seed)
            np.random.seed(seed)
            return [seed]


class TrainingMetricsCallback(BaseCallback):
    """
    Custom callback to track training metrics specific to Monopoly Deal.
    """

    def __init__(self, verbose=0):
        super().__init__(verbose)
        self.episode_rewards = []
        self.episode_lengths = []
        self.win_rates = []
        self.complete_sets_history = []
        self.recent_wins = []
        self.episode_count = 0

    def _on_step(self) -> bool:
        # Check if episode is done
        if self.locals['dones'][0]:
            self.episode_count += 1

            # Get episode info
            info = self.locals['infos'][0]
            episode_reward = self.locals['rewards'][0]

            # Track basic metrics
            self.episode_rewards.append(episode_reward)

            # Track wins (assuming our agent is player 0)
            env = self.training_env.envs[0].env  # Get the actual environment
            if hasattr(env, 'winner') and env.winner == 0:
                self.recent_wins.append(1)
            else:
                self.recent_wins.append(0)

            # Keep only last 100 wins for win rate calculation
            if len(self.recent_wins) > 100:
                self.recent_wins.pop(0)

            current_win_rate = np.mean(self.recent_wins) if self.recent_wins else 0
            self.win_rates.append(current_win_rate)

            # Log every 100 episodes
            if self.episode_count % 100 == 0:
                avg_reward = np.mean(self.episode_rewards[-100:]) if len(self.episode_rewards) >= 100 else np.mean(
                    self.episode_rewards)
                print(f"Episode {self.episode_count}: Avg Reward: {avg_reward:.2f}, Win Rate: {current_win_rate:.2%}")

        return True


def create_monopoly_env(rank=0):
    """Create a single Monopoly Deal environment.
    """
    def _init():
        env = MonopolyDealEnv()
        env = FlattenedMonopolyDealEnv(env)
        # Set seed before wrapping with Monitor
        env.seed(1000 + rank)
        env = Monitor(env)
        return env
    return _init


def train_with_ppo(total_timesteps=500000, n_envs=4):
    print("Training with PPO...")

    # Create vectorized environment (using DummyVecEnv to avoid multiprocessing issues)
    env = DummyVecEnv([create_monopoly_env(i) for i in range(n_envs)])

    # Create evaluation environment
    eval_env = DummyVecEnv([create_monopoly_env()])

    # Create callbacks
    metrics_callback = TrainingMetricsCallback()

    eval_callback = EvalCallback(
        eval_env,
        best_model_save_path='./models/ppo_best/',
        log_path='./logs/ppo_eval/',
        eval_freq=10000,
        deterministic=True,
        render=False,
        n_eval_episodes=20
    )

    checkpoint_callback = CheckpointCallback(
        save_freq=50000,
        save_path='./models/ppo_checkpoints/',
        name_prefix='monopoly_deal_ppo'
    )

    callback_list = CallbackList([metrics_callback, eval_callback, checkpoint_callback])

    # Create PPO model with custom hyperparameters
    model = PPO(
        "MlpPolicy",
        env,
        verbose=2,  # More verbose logging
        tensorboard_log="./logs/ppo_tensorboard/",
        learning_rate=3e-4,
        n_steps=1024,  # Smaller steps = more frequent logging
        batch_size=64,  # Batch size for training
        n_epochs=10,  # Epochs per update
        gamma=0.99,  # Discount factor
        gae_lambda=0.95,  # GAE lambda
        clip_range=0.2,  # PPO clipping range
        ent_coef=0.01,  # Entropy coefficient for exploration
        vf_coef=0.5,  # Value function coefficient
        max_grad_norm=0.5,  # Gradient clipping
        policy_kwargs=dict(
            net_arch=[256, 256, 128]  # Network architecture
        )
    )

    # Train the model
    model.learn(
        total_timesteps=total_timesteps,
        callback=callback_list,
        tb_log_name="ppo_monopoly_deal"
    )

    # Save final model
    model.save("./models/monopoly_deal_ppo_final")

    return model, metrics_callback


def main():
    # Training configuration
    timesteps = 50_000  # Start with smaller number for testing
    ppo_model, ppo_metrics = train_with_ppo(total_timesteps=timesteps)

if __name__ == "__main__":
    main()
```



{{</ collapse >}}

At this stage, we've defined our environment and we have an training script above. For now, take the training script with a grain of salt - the focus this post is on the development of a reinforcement learning environment.

When running this training script, you should notice it stalls after a short while. For me, today, it was after 80 updates, roughly 37,000 timesteps. Let's take a look at the callback logs for this:

```
----------------------------------------
| time/                   |            |
|    fps                  | 3413       |
|    iterations           | 7          |
|    time_elapsed         | 8          |
|    total_timesteps      | 28672      |
| train/                  |            |
|    approx_kl            | 0.06316833 |
|    clip_fraction        | 0.531      |
|    clip_range           | 0.2        |
|    entropy_loss         | -5.63      |
|    explained_variance   | 5.96e-08   |
|    learning_rate        | 0.0003     |
|    loss                 | 24         |
|    n_updates            | 60         |
|    policy_gradient_loss | -0.0381    |
|    value_loss           | 90.9       |
----------------------------------------
-----------------------------------------
| time/                   |             |
|    fps                  | 3369        |
|    iterations           | 8           |
|    time_elapsed         | 9           |
|    total_timesteps      | 32768       |
| train/                  |             |
|    approx_kl            | 0.059141707 |
|    clip_fraction        | 0.526       |
|    clip_range           | 0.2         |
|    entropy_loss         | -5.48       |
|    explained_variance   | -1.19e-07   |
|    learning_rate        | 0.0003      |
|    loss                 | 29.6        |
|    n_updates            | 70          |
|    policy_gradient_loss | -0.0486     |
|    value_loss           | 51.3        |
-----------------------------------------
----------------------------------------
| time/                   |            |
|    fps                  | 3338       |
|    iterations           | 9          |
|    time_elapsed         | 11         |
|    total_timesteps      | 36864      |
| train/                  |            |
|    approx_kl            | 0.06325258 |
|    clip_fraction        | 0.544      |
|    clip_range           | 0.2        |
|    entropy_loss         | -5.34      |
|    explained_variance   | 1.37e-06   |
|    learning_rate        | 0.0003     |
|    loss                 | 16.8       |
|    n_updates            | 80         |
|    policy_gradient_loss | -0.0566    |
|    value_loss           | 50.4       |
----------------------------------------
```

The reason for this, is that although there are 448 actions to choose from, at any one time only 10-30% of them are valid moves. For example, you can't play a House if you don't have a set, you can't play a rent card as a property - we need a way to check if the moves are valid, and mask the moves as valid.

The high proportion of invalid moves makes the training slow and leads to sample inefficiency. We can improve this, by using a technique called action masking.

The earliest introduction of action masking I could find was in 1998, where Even-Dar discovered that eliminating provably sub-optimal actions can speed up training. But where it more directly applies to our situation is in the development of [AlphaGo, where they implement action masking for legal moves](https://www.researchgate.net/publication/292074166_Mastering_the_game_of_Go_with_deep_neural_networks_and_tree_search).

Before we do action masking, we need to define a valid move. We can do this by filling out the adding to our environment.

```python
class MonopolyDealEnv(gym.Gym):

	...
	def _is_valid_action(self, action: int) -> bool:
	    player = self.players[self.current_player]
	    opponent = self.players[(self.current_player + 1) % 2]

	    if action == 0:  # Pass
	        return True
	    elif 1 <= action <= 7:  # Play as money
	        card_idx = action - 1
	        return card_idx < len(player['hand']) and self._can_play_as_money(player['hand'][card_idx])
	    elif 8 <= action <= 14:  # Play as property
	        card_idx = action - 8
	        return card_idx < len(player['hand']) and self._get_property_color(player['hand'][card_idx]) is not None
	    elif 15 <= action <= 22:  # Rent
	        color_idx = action - 15
	        color = self.COLOR_ORDER[color_idx]
	        return CardType.RENT_WILD in player['hand'] and player['properties'][color] > 0
	    elif 23 <= action <= 30:  # Sly Deal
	        color_idx = action - 23
	        color = self.COLOR_ORDER[color_idx]
	        return CardType.SLY_DEAL in player['hand'] and opponent['properties'][color] > 0 and color not in opponent['complete_sets']
	    elif 31 <= action <= 38:  # Deal Breaker
	        color_idx = action - 31
	        color = self.COLOR_ORDER[color_idx]
	        return CardType.DEAL_BREAKER in player['hand'] and color in opponent['complete_sets']
	    return False

	def _can_play_as_money(self, card_type: CardType) -> bool:
	    return card_type in self.MONEY_VALUES or card_type.value >= 5
	...
```

Now that we define what is, and what isn't a valid move we can begin to mask invalid actions.

```python
import gymnasium as gym


class ActionMaskedMonopolyDealEnv(gym.Wrapper):
    """
    Wrapper that provides action masking to help the agent learn faster.    This tells the agent which actions are valid at each step.    """
    def __init__(self, env):
        super().__init__(env)

    def _get_action_mask(self):

        """Get a boolean mask of valid actions."""
        mask = np.zeros(self.action_space.n, dtype=bool)

        # Test each possible action
        for i in range(self.action_space.n):

            if self.env._is_valid_action(i):
                mask[i] = True
        return mask

	def step(self, action):
	    obs, reward, terminated, truncated, info = self.env.step(action)
	    # Add updated action mask to info
	    if not terminated and not truncated:
	        info['action_mask'] = self._get_action_mask()
	    return obs, reward, terminated, truncated, info

	def seed(self, seed=None):
	    """Set the seed for the environment."""
	    return self.env.seed(seed)

	def reset(self, **kwargs):
	    obs = self.env.reset(**kwargs)
	    # Add action mask to info
	    if isinstance(obs, tuple):
	        obs, info = obs
	        info['action_mask'] = self._get_action_mask()
	        return obs, info
	    else:
	        return obs, {'action_mask': self._get_action_mask()}
```

Let's break this down a bit. First, we'll note that we use a GymWrapper. As per [Gymnasium's documentation](https://gymnasium.farama.org/introduction/create_custom_env/#using-wrappers) a wrapper is helpful for when you want to alter the environment without changing the underlying environment.

Our main improvement is that of the `_get_action_mask()` function. This function creates a numpy array of 0s for the size action_space. For each action within the action space, we check to see if it's a valid action. if it is, the 0 is replaced with a 1.

Let's train again, this time we hope to see the logging for Invalid moves decrease with timesteps, and the loss decrease nicely. Give it a go, and check the results. What you'll see is not what we want to see...
