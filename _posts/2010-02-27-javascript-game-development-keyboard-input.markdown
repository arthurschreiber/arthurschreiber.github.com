---
layout: post
title: Javascript Game Development - Keyboard Input
description: How to process keyboard input in JavaScript based games.
keywords:
  - keydown
  - keyup
  - javascript
  - html
  - html5
  - game
  - engine
  - development
---

Now that we have our basic game loop layed out, we can focus on implementing other aspects
of our javascript based game. What I want to show in this post is how you process keyboard
input.

## Keyboard Events

If you've done any previous web development with javascript, you'll most likely know the
`keydown` and `keyup` events that get fired by the browser when a key is pressed down and
released again.

Let's say we want to create a game where the player controls a small black rectangle and
can move it around in a bigger area (very sophisticated, I know!).

First, we'll need a class that represents our player object:

{% highlight javascript %}
function Player() {
  this.x = 0;
  this.y = 0;
}

Player.prototype.draw = function(context) {
  context.fillRect(this.x, this.y, 32, 32);
};

Player.prototype.moveLeft = function() {
  this.x -= 1;
};

Player.prototype.moveRight = function() {
  this.x += 1;
};

Player.prototype.moveUp = function() {
  this.y -= 1;
};

Player.prototype.moveRight = function() {
  this.y += 1;
};
{% endhighlight %}

Next, we'll need to hook up on instance of the Player class to our existing game code.

{% highlight javascript %}
Game.start = function() {
  ...
  
  Game.player = new Player();
  
  ...
};

Game.draw = function() {
  ...
  
  Game.player.draw(Game.context);

  ...
};
{% endhighlight %}

Last, we need to setup our event handling code.

{% highlight javascript %}
window.addEventListener('keydown', function(event) {
  switch (event.keyCode) {
    case 37: // Left
      Game.player.moveLeft();
    break;

    case 38: // Up
      Game.player.moveUp();
    break;

    case 39: // Right
      Game.player.moveRight();
    break;

    case 40: // Down
      Game.player.moveDown();
    break;
  }
}, false);
{% endhighlight %}

This [first version](/examples/player_input/version_1.html) kinda works, but it is pretty
choppy and only ever works for one key. Why?

The choppiness is due to the fact that the `keydown` event is fired repeatedly with small
pauses between each event, and these pauses are not synced up with our game update code.
So we'll have to figure out a way to get them in sync.

The problem that only ever one key is recognized comes from the fact that the browser
only ever repeats `keydown` events for the last key that was pressed.

For working around these two problems, I'm using this super simple helper object:

{% highlight javascript %}
var Key = {
  _pressed: {},

  LEFT: 37,
  UP: 38,
  RIGHT: 39,
  DOWN: 40,
  
  isDown: function(keyCode) {
    return this._pressed[keyCode];
  },
  
  onKeydown: function(event) {
    this._pressed[event.keyCode] = true;
  },
  
  onKeyup: function(event) {
    delete this._pressed[event.keyCode];
  }
};
{% endhighlight %}

You can hook it up to `keydown` and `keyup` events like this:

{% highlight javascript %}
window.addEventListener('keyup', function(event) { Key.onKeyup(event); }, false);
window.addEventListener('keydown', function(event) { Key.onKeydown(event); }, false);
{% endhighlight %}

This object tracks the key status of individual keys based on the `keydown`/`keyup` events
fired by the browser. Using it, we can easily check in our game update code whether a specific
key is currently pressed down or not. Now we add the following function to our Player class:

{% highlight javascript %}
Player.prototype.update = function() {
  if (Key.isDown(Key.UP)) Game.player.moveUp();
  if (Key.isDown(Key.LEFT)) Game.player.moveLeft();
  if (Key.isDown(Key.DOWN)) Game.player.moveDown();
  if (Key.isDown(Key.RIGHT)) Game.player.moveRight();
};
{% endhighlight %}

Then we only need to hook this also up to the Game.update method:

{% highlight javascript %}
Game.update = function() {
  ...
  Game.player.update();
  ...
};
{% endhighlight %}

Check out this [final version](/examples/player_input/version_2.html).
