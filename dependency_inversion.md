Dependency Inversion
====================

The main goals of the Dependency Inversion Principle (DIP) are to
create software that is:

- reusable: our code is modular; we can reuse modules
  independently in other sections of the system
- easy to change: a small change can be made without causing a cascade
  of changes
- flexible: making a single change will not break unexpected,
  unrelated parts of the system


Essentially, the Dependency Inversion Principle follows other clean
code concepts in the desire to create code that is designed to
withstand constantly evolving requirements. The two fundamental ideas
involved in adhering to the Dependency Inversion Principle are that:
- "**High-level modules** should not depend on **low-level
  modules**. Both should depend on abstractions."
- "**Abstractions** should not depend on **details**. Details should
  depend on abstractions."

Looking at my original implementation of Tic Tac Toe in Elixir, with
these ideas in mind, it is very clear that it breaks the
Dependency Inversion Principle in the most fundamental way. The basic
format of this program is that the ```Game``` module takes input from a
```Player``` and uses it to update the ```Board```. It then
switches to the other player, gets their input to update the board,
and continues to loop through this process until the game is over
(there is a tie, or one of the players has won).

```
defmodule Game do
  def game_loop(current_board, current_player) do
    Board.display(current_board)
    move = Player.make_a_move(current_player)
    Board.update(current_board, move)
    if game_over?
      #end the game
    else
      game_loop(updated_board, next_player) #start a new game
    end
  end

  def next_player do
    #switches to the other player
  end
end
```

```
defmodule Player do
  def make_a_move(current_player) do
    #logic that gets the move from the current player
  end
end
```
```
defmodule Board do
  def create_new_board do
    #creates a new, empty board
  end

  def update_board(current_board, move) do
    #logic to update the board with the player's move
  end
end
```

```
Game.game_loop
```
If we focus on the interactions between ```Game```, ```Board``` and
```Player```, we can see that ```Game``` is the offender in
fundamentally breaking the Dependency Inversion Principle: it is
*very* dependent on the ```Board``` and ```Player``` modules. We can
think of ```Game``` as the **high-level module**: it is dealing with
the larger, abstract idea of running through the loop of a game.
```Board``` and ```Player``` are the **low-level modules** in this
example. They deal with the specific implementation of how to get a
move from a player, and how to update a board with that move. As we can
see in the code above, our high-level ```Game``` module directly uses
the lower-level ```Player``` and ```Board``` modules, and is
therefore dependent on any change than may be made to the either of
these modules. 

The ```Game``` module has logic in it that continues getting player
moves until the game is over. If this loop were a bit more generic
and didn't specifically mention the ```Player``` and the
```Board```, we could use it for *any* program where we want to loop
through a game. However, currently the ```Game``` module can only be
used in this specific program, with these specific ```Player``` and
```Board``` modules. Which means that```Game``` is **not**
reusable. This system is **not** easy to change either: every time we
want to make a small change to either ```Player``` or
```Board```we would also have to change ```Game``` as well to make
sure that it can properly communicate with the new behavior of
```Player``` and ```Board```. Likewise, this system is **not**
flexible: making a very small change to ```Player``` could
potentially break ```Game```, which could then even pass that bug
along to ```Board``` as well.

So how can we change that? How can we refactor this code to make is
reusable, easy to change and flexible? By utilizing dependency
injection, we can refactor the ```Game``` module to behave so that it
it doesn't depend directly on the ```Player``` and ```Board``` modules.


```
defmodule Game do
  def game_loop(board_module, current_board, current_player) do
    board_object.display(current_board)
      move = Player.make_a_move(current_player)
      board_module.update(current_board, move)
      if game_over?
        #end the game
      else
        start_game_loop(board_module, updated_board,
next_player) #start a new game
      end
    end
  end

  def next_player do
    #switches to the other player
  end
end
```

Instead of directly calling on ```Player``` and ```Board``` *within*
```Game```, we can simply pass a *player* and a *board*  into
```Game``` when we first call ```Game.start_game_loop``` .

```
new_board = Player.create_new_board
Game.start_game_loop(Player, new_board, Board)
```

This seems like a really small change, but it is a powerful one. We can
now pass any objects into ```Game.start_game_loop```, as long as the
objects implement the *player* interface and *board* interface. In
other words, ```Game``` can take in *any* object that is a board,
and *any* object that is a player. We have inverted the
dependencies. Instead of ```Game``` depending on the **details** of
```Board``` and ```Player```, it is now only dependent on the
**abstractions** of the board and player interfaces.
Since```Game``` is now just a generic game module, we could
possibly use it when creating a Chess program (or any other game program
for that matter). This Chess program could use a
```ChessPlayer``` module that implements the player interface, and a
```ChessBoard``` that implements the board interface.

Going forward, instead of having ```Player``` and ```Board``` be
dependent on the details of the ```Game``` module, we should now also
refactor them to be dependent on the **abstractions** of the player
and board (that ```Game``` will expect). By using this pattern of
having modules be dependent on **abstractions**, rather than on
specific implementations, we have complete flexibility to reuse our
code in many different ways. As mentioned above, we can swap out
different Player and Board modules to play different sorts of games.
We should also make sure that our game's ```Setup```, and
```Rules``` are completely independent of the specific game details
as well. This way we can decide, at runtime, how we would like our
```Game``` to behave: maybe only allowing for an unbeatable
computer, or allowing the user to choose which marker they'd like to
use (instead of X or O), etc. By adhering to the Dependency
Inversion Principle we can truly create *modular* modules and
therefore allow for our software to be reusable, easy to change and
flexible.

--------------

Source:

http://www.objectmentor.com/resources/articles/dip.pdf

