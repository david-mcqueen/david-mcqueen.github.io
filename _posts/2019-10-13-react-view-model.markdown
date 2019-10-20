---
layout: post
title: "Re-rendering Nested Objects in React"
date: 2019-10-13 21:13:00 +0000
categories: blog posts
---

# The Summary.

1. Stuff doesn't work.
2. Stuff now works.


Thanks for stopping by, cya.

---

#### ...

# The Summary Part II - The Expanded Summary.

React isn't re-rendering Component _prop_ changes for values which are nested. I implemented a solution where the component is updated directly view a ViewModel and the updated data is rendered to the screen.

# The Background.

I'm working on a project in which I have significant, if not all, of the logic within a single object. The project is nothing major, just a blackjack game where the aforementioned single object _is the game_ and handles everything to do with it. The _game_ object contains all of everything pertaining to the game; the players, the deck, etc. It is also the parent for all of the logic, such as dealing a card to the current player, determining the players score, and calculating which player(s) have won against the dealer.

An abridged structure of this all powerful game object can be defined as such;

```javascript
    - GameBoard
    | - Players: Player[]
    | - Deck: Deck

    - Player
    | - Hand: Hand

    - Hand
    | - Cards: Card[]

    - Deck
    | - Cards: Card[]

    - Card
```

- The _GameBoard_ has a collection of _Players_, and a _Deck_ of cards (the un-abridged version has more, but for the purpose of this example it just has the two). It is responsible for marshalling the entire game. This is the parent.

- The _Deck_ is a collection of cards which handles the shuffling and dealing of cards.

- The _Player_ has a _Hand_.

- The _Hand_ is just a collection of _Card_.

It is all self contained, and it works. Except it doesn't when it comes to the UI.

Have a look at [this sandbox](https://codesandbox.io/s/react-component-propsrender-yjify?fontsize=14) for a working<sup>*</sup> example of the code to see what I am trying to achieve.

<sup>\* It doesn't work, thats the problem.</sup>

Click on the _Deal_ button and you'll see what I mean.

![Components](/assets/images/react-rerender/not-working.png)

A new card has been dealt to all players (again, just for the purpose of this example everybody gets a card). The Component props have updated, and there are now 6 cards in the _Hand_ as shown by the React dev tools on the right. However only the initial 2 render (e.g. the 8 of Clubs, and 9 of Spades for player 1).

# The Problem?

The change I'm looking for is within the _Cards_ array as the _Player_ gets dealt a new card. The top UI component starts rendering from the _GameBoard_ & _Player_ components down;
```javascript
    GameBoard
        |- Player
            |- Hand
                |-Cards[]   <--- The change is within here
```

Lets have a quick look at the structure of the UI components;

```javascript
    - GameBoardComponent
    | - GamePlayerList

    - GamePlayerList
    | - GamePlayer
    | - GamePlayer
    ...
    | - GamePlayer

    - GamePlayer
```

- The _GameBoardComponent_ passes the players down to the _GamePlayerList_ via its props.

- The _GamePlayerList_ loops each of the players and passes them into the _GamePlayer_ component.

- The _GamePlayer_ is the bottom component and loops all of the players cards, rendering each of them.

The problem I have lies within how React determines if a re-render is needed. I did initially start writing a detailed insight as to what is happening in the React source code, but it started to get fairly in depth and distracting away from the main point of this post - The solution implemented.

As such, for now just take my word for it that the problem exists (as demonstrated in the above sandbox) and that a workable solution has been implemented. I'll write a post in the future looking into the source code to see what is actually happening.

Onwards!


# The Pre-Solution Considerations.

* I want to keep the _GameBoard_ object as the main control for this project. It's structure is data driven as opposed to being UI driven, I just want the UI to render what the _GameBoard_ contains.

* Due to this point, the state can be fairly dum or even non-existent. _The GameBoard_ is everything so there is no real need for state, at least to the extent of it containing each player, each players cards etc.

* I consider using Redux, but that was getting a bit complex and involed moving a lot of the logic into Redux to keep the sate immutable and only modified from Redux itself, which was not what I wanted to achieve. I don't want to flatten my game logic so that it fits into State.

* Overriding ```shouldComponentUpdate()``` could allow me to implement my own deep comparrison logic on the _GameBoard_ object, but this is actively advised against in the [React docs](https://reactjs.org/docs/react-component.html#shouldcomponentupdate);

> We do not recommend doing deep equality checks or using JSON.stringify() in shouldComponentUpdate(). It is very inefficient and will harm performance.



# The Solution.

Don't use React. Use a different JavaScript library.

OK thats maybe not the solution, but it was funny though. Right?

Moving on...

In short, the solution calls for the UI Component getting a View Model (VM) directly from the Game control object (in this case, the _Player_ object) in lieu of the data being passed in as _props_ from their parent.

The _GamePlayer_ UI Component still takes the _Player_ in its props, but then instead of rendering the cards from this object directly I utilise _useState_ and pass the state setter to the _Player_ object to get the data the component needs to render. The _Player_ becomes responsible for updating the state (ViewModel) for the component which is rendering it's data. As there are multiple _Players_, each one is responsible for itself. We don't need to track in state which player has been updated.

This is basically shortcutting the _props_ which are being passed all the way down from the top level component to the child component, and forms a direct bond between the Model and the View Model. It doesn't result in any more renders, as it is using _state_ so will only render once there is a change.

The main downside of this solution is that the _Player_ object needs to manually update the VM state of its component when the data changes. However for my situation this is not a major drawback, there is only 1 or 2 places which can update the data so I only need to update the VM from a couple of locations.

Lets take a look at some code;

_GamePlayer_

```javascript
const [state, updatePlayerState] = useState <IPlayerState>(player.getViewModel);
player.setViewModelCallback(updatePlayerState);
```

_Player_

```javascript
public setViewModelCallback( callback: Dispatch<SetStateAction<IPlayerState>> ) {
    this.viewModelCallback = callback;
}

getViewModel = (): IPlayerState => ({
    cards: this.getCards()
});

public acceptCard(card: Card) {
    this.Hand.acceptCard(card);
    this.notify();
}

private notify() {
    if (this.viewModelCallback) {
      const playerState = this.getViewModel();
      this.viewModelCallback(playerState);
    }
}
```

- The _GamePlayer_ component gets a state setter from the _useState_ hook, and passes this as a callback into the _Player_ object.

- The _Player_ object stores the VM callback. Any time the _Player_ accepts a card, via the _acceptCard_ function, `this.notify()` is called which gets all of the data required and passes it into the callback. The callback itself is the _updatePlayerState_ stateSetter from the _GamePlayer_ component so we are infact setting the state on the UI cimponent.

- The UI Component is then aware it's state has changed so performs a render.

An updated version of the code, [in this sandbox](https://codesandbox.io/s/react-component-vm-render-ph8du?fontsize=14), shows a working implementation<sup>*</sup> with the code snippets above. The _Player_ is dealt a new card, setting in motion the update for the ViewModel and ultimetly the re-rendering of the data.

<sup>* This one does actually work</sup>

# The Grand Finale.

This method allows for the components data to be controlled directly via the Model controlling the data, whilst still allowing React to control the small amount of state needed, and the re-rendering of it's own UI lifecycle. 

I think its important to point out that I'm not against using State, Redux, or any other of the plethora of available tools. I'm sure this Blackjack game could be made in a more idiomatic way utilising more of State or Redux, but it's not how I want to achieve it in this situation. If a better implementation becomes apparent in the future, then as is always the case with software, I'm at liberty to change it. For now, it works and I'm happy with it. With all that being said, I best crack on with finishing the game and stop blogging...



--- 


As always, hit that subscribe button and click the little bell to join the _Notification Squad_... oh wait. Sorry. This isn't youtube. Ermm, come back another time?