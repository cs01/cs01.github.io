# What could be better than SSH? Three tools to consider

SSH is awesome. It lets you securely connect to remote computers and act like you’re on a local computer. It’s so common it’s even a verb:

> “ssh into the server and restart it”

But it has some downsides:

*   you need to reconnect if your computer goes to sleep or you switch internet connections
*   each terminal needs to maintain its own connection
*   it’s slow to respond to high bandwidth commands on slow connections

It turns out something _could_ be better than SSH if it solved these problems, and in fact, some tools do exist that augment SSH. They are, in no particular order, **tmux, mosh, and Eternal Terminal.**

### [Eternal Terminal](https://mistertea.github.io/EternalTerminal/)

![](https://cdn-images-1.medium.com/max/1200/0*xgrcB7K53UBQ_21f.png)

ET is the most ssh-like of the three. You can think of it like a drop-in replacement for ssh that reconnects automatically. Imagine: you connect to a remote machine, travel across the world reconnecting to 5 different WiFi networks, **yet you’ve maintained your initial remote session!**

You can run it like this:

<pre name="a17d" id="a17d" class="graf graf--pre graf-after--p">et hostname</pre>

### [Mosh — the “mobile shell”](https://mosh.org/)

![](https://cdn-images-1.medium.com/max/1200/0*mmzifSOVzghF6gga.jpg)

Mosh is similar to ET in that it persistently keeps you connected to a remote machine, **but it does it in a very different way**. Instead of forwarding every single thing from the remote machine to your machine like SSH and ET, it efficiently forwards you _only the latest snapshot_ of the terminal.

It uses SSH to establish the initial connection, but then switches over to UDP and something called State Synchronization Protocol. This has pros and cons.

Pros:

*   it is highly efficient
*   it works well over crappy (low bandwidth and/or intermittent) connections
*   the connection persists across WiFi networks and interruptions (same as ET)
*   it lets you type at times when SSH and ET would still be waiting for a command to finish or connection to be re-established

Cons:

*   It contains no history! You can’t scroll up because it literally can only show you the latest state of the output on the remote machine. There is a workaround though. Keep reading.
*   It does not work with tmux’s “control mode” (tmux -CC)
*   It does not have port forwarding

You can run it like this:

<pre name="9ce2" id="9ce2" class="graf graf--pre graf-after--p">mosh hostname</pre>

### [tmux — the “terminal multiplexer”](https://github.com/tmux/tmux/wiki)

![](https://cdn-images-1.medium.com/max/1200/1*HQF7Tcfihcnrw5_FcL_rmg.png)a single terminal split vertically by tmux ([source](https://medium.com/@gveloper/using-iterm2s-built-in-integration-with-tmux-d5d0ef55ec30))

Unlike ET and mosh, it does **not** establish or maintain remote connections. So what does it do?

It is an _enhancement_ to ssh, EternalTerminal, or mosh which lets you **re-use a single connection in multiple terminals**.

You can do this in two ways:

1.  Split a single terminal into “virtual” panes:

<pre name="b1ef" id="b1ef" class="graf graf--pre graf-after--li">tmux</pre>

2\. Or integrate with the terminal to open new “virtual” terminal windows. This is called “command mode”:

<pre name="89a4" id="89a4" class="graf graf--pre graf-after--p">tmux -CC</pre>

Command mode integrates with your [terminal](https://iterm2.com/documentation-tmux-integration.html), so when you go to open a new tab or window, it will actually ask you: should I reuse the connection with tmux, or should I open a new terminal like I normally do?

![](https://cdn-images-1.medium.com/max/1600/1*oxjVl7zVglqOh4FtMQMpEg.png)Prompt when running tmux command mode in iTerm2

tmux turns the terminal into “just another program”, so when you use it, you can also

*   search through **all** the output in the terminal in a way similar to vim
*   scroll through the terminal output using **its own history** (kind of like scrolling through text in _le_ss or _vim_), not the terminal’s. When using mosh, this is the way to view history.

**It basically lets you split a single session into multiple sessions without having to establish or maintain multiple connections.** It “multiplexes” a bunch of virtual sessions through a single connection in a way that’s totally transparent to you, the user.

tmux is kinda like vim. It’s powerful, but you have to bite the bullet and learn some annoying, unintuitive keystrokes.

### So which is right for you?

![](https://cdn-images-1.medium.com/max/1600/1*yGLIMxiY43Q9Lyrpji-3pg.png)

Use SSH if you…

*   are fine reconnecting to a machine every time your internet goes out or you want to put your computer to sleep
*   have relatively reliable, high speed connections
*   want scrolling to work like it does normally
*   **want the option** touse tmux to easily establish multiple “physical” terminal windows (`tmux -CC`) over a single connection
*   **want the option** to use tmux to create multiple “virtual” terminal panes over a single connection
*   want the option to do port forwarding
*   do not mind waiting to type more text until the last command you ran has finished

Use Eternal Terminal if you want everything SSH offers but also

*   want seamlessly persistent connections

Use Mosh if you…

*   want seamlessly persistent connections
*   have unreliable and/or low bandwidth internet connection
*   **are okay being required to use tmux to scroll up and look at history**
*   are okay **not** being able to use tmux to easily establish multiple “physical” terminal windows (`tmux -CC`) over a single connection
*   **want the option to** use tmux to create multiple “virtual” terminal panes over a single connection
*   **do not** need to do any port forwarding

Use tmux if you…

*   want to create multiple terminals from a single terminal and a single remote connection
*   want the ability to search through your history
*   are okay learning a bunch of new keystrokes for multiple panes. (note: this does not apply to tmux’s “control mode”).

### My Choice

I settled on EternalTerminal, and sometimes use tmux with it.

### Links

*   [Eternal Terminal Homepage](https://mistertea.github.io/EternalTerminal/)
*   [Mosh Homepage](https://mosh.org/)
*   [tmux Homepage](https://github.com/tmux/tmux/wiki)
*   [Using iTerm2’s built-in integration with tmux](https://medium.com/@gveloper/using-iterm2s-built-in-integration-with-tmux-d5d0ef55ec30)
*   [iTerm2 tmux documentation](https://iterm2.com/documentation-tmux-integration.html)