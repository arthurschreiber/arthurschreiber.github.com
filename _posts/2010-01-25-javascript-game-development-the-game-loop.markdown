---
layout: post
title: Javascript Game Development - The Game Loop
categories: [javascript]
---

One of the most important parts of a game engine is the so called "game loop". It
is the central piece of the game's engine and is responsible for trying to balance
running a game's logic, and executing it's drawing operations.

A very basic game loop would look something like this in JavaScript
(this does not work in Browsers!):

{% highlight javascript  %}
var Game = { };

Game.draw = function() { ... draw entities here ... };
Game.update = function() { ... run game logic here ... };

while (!Game.stopped) { // While the game is running
  Game.update();        // Update Entities (e.g. Position)
  Game.draw();          // Draw Entities to the Screen
}
{% endhighlight %}

Writing a game loop for execution in the browser is a tad more tricky than that.
We can't just use a while loop that runs forever, as JavaScript execution blocks
the browser from repainting its window, making the game seem to have "locked-up".

So we'll have to try "emulating" a real loop, while giving back control to the
browser after every drawing operation to not lock-up the interface.

## setInterval to the rescue!

[window.setInterval][mdc:setInterval] is exactly what we're looking for. It provides a way to run code
in a loop while allowing the browser to repaint the window in between the loop runs.

{% highlight javascript %}
Game.fps = 50;

Game.run = function() {
  Game.update();
  Game.draw();
};

// Start the game loop
Game._intervalId = setInterval(Game.run, 1000 / Game.fps);

...

// To stop the game, use the following:
clearInterval(Game._intervalId);
{% endhighlight %}

Check out [the example page][example:setInterval] for a game loop that uses setInterval.

## That's it?

Well, yeah. At least for a basic game. The main problem with this kind of
loop is that the updating and drawing operations are "glued" together (this
means your game's logic will run as fast as your drawing operations). So if your
frame rate drops below 60fps, your game will also seem to be running slower.

For some games, this might never be a problem, for others this behaviour can lead
to jerkiness. Also, if you want to add a multiplayer component, you want to make
sure that your logic runs with the same speed on all your clients.

This can be achieved using a so called "game loop with fixed time steps".
Basically, we try to run the logic a fixed amount of times per second, while
trying to draw as many frames per second as possible.

{% highlight javascript %}
Game.run = (function() {
  var loops = 0, skipTicks = 1000 / Game.fps,
      maxFrameSkip = 10,
      nextGameTick = (new Date).getTime();
  
  return function {
    loops = 0;
    
    while ((new Date).getTime() > nextGameTick && loops < maxFrameSkip) {
      Game.update();
      nextGameTick += skipTicks;
      loops++;
    }
    
    Game.draw();
  };
})();

// Start the game loop
Game._intervalId = setInterval(Game.run, 0);
{% endhighlight %}

Now, this will give us a game loop that runs Game updates at 50fps, while trying to run
the drawing code as fast as possible.

This gives very, very smooth animations, on my machine, I get around 200 drawing
operations per second with Google Chrome, while the number of updating operations
stays constant at 50 per second.

Check out the [example][example:fixed_timestep_setInterval].

## More problems?!

But now we're facing another problem: We're burning a lot of CPU cycles, way more than are
actually needed to give our players smooth animations. Most computer screens have a refresh
rate of either 50 or 60 fps, so having 200 rendering operations per second is total overkill,
and brings my CPU usage up to 48%, according to Chrome's Task Manager.

That's why Mozilla introduced [window.mozRequestAnimationFrame][mdc:mozRequestAnimationFrame] (which also has a WebKit twin
called window.webkitRequestAnimationFrame) a way to easily limit the number of
drawing operations per second to the number of screen refreshes per second.
(Currently, both mozRequestAnimationFrame as well as webkitRequestAnimationFrame do not
truly sync with the screen refresh rates, but simply help in limiting drawing
operations to a sane amount, but real sync will be implemented eventually.)

To be also compatible with browsers that do not implement window.\*RequestAnimationFrame,
we'll have to provide a fallback based on setInterval, but limited to 60 frames per second:

{% highlight javascript %}
(function() {
  var onEachFrame;
  if (window.webkitRequestAnimationFrame) {
    onEachFrame = function(cb) {
      var _cb = function() { cb(); webkitRequestAnimationFrame(_cb); }
      _cb();
    };
  } else if (window.mozRequestAnimationFrame) {
    onEachFrame = function(cb) {
      var _cb = function() { cb(); mozRequestAnimationFrame(_cb); }
      _cb();
    };
  } else {
    onEachFrame = function(cb) {
      setInterval(cb, 1000 / 60);
    }
  }
  
  window.onEachFrame = onEachFrame;
})();

window.onEachFrame(Game.run);
{% endhighlight %}

This gives us smooth game play without totally burning the CPU (according to Chrome's
Task Manager, this gives a CPU load of 10-20% when using webkitRequestAnimationFrame).

See for yourself [here][example:fixed_timestep_optimal].

## Conclusion

Wow, that's been a long journey we had to undertake to get a game loop that's giving
us both smooth animations while making sure that our game runs at a constant pace.

I'll try to post some more articles about game development using Javascript and HTML5
over the next weeks, so make sure to check back from time to time!

[mdc:setInterval]: https://developer.mozilla.org/en/DOM/window.setInterval
[mdc:mozRequestAnimationFrame]: https://developer.mozilla.org/en/DOM/window.mozRequestAnimationFrame
[example:setInterval]: /examples/game_loop/setInterval.html
[example:fixed_timestep_setInterval]: /examples/game_loop/fixed_timestep_setInterval.html
[example:fixed_timestep_optimal]: /examples/game_loop/fixed_timestep_optimal.html