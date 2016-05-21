Sandboxed, concurrent plugins API
---------------------------------

I personally have spent among thousands of hours in vim over the past
decade. In this, probably hundreds configuring and wrestling a vim
configuration.


Goal
~~~~

Expose a public python API to editor's buffers, layout, keybindings
and etc.

In my rationale, I will put my personal experience as a heavy vim user
who has actively tried to maintain a consistent vim configuration across
multiple programming languages and 3 OS (OS X, Linux, FreeBSD), a python
programmer.

I will back the design decisions with historical anecdotes observed
in the editor community and python community.

My intention is to develop a serious contender to vim, sublime and
atom.

Anecdotes and observations
~~~~~~~~~~~~~~~~~~~~~~~~~~

Python
------

- Saltstack has a great system of states and modules that would be
  great for reuse by other libraries. The system of modules provide
  a library of system abstractions that extract tons of interesting
  information and actions in a cross-platform way.

  However, saltstack binds the actions of these modules to a builtin
  ``__salt__`` object.  Making it all but impossible to utilize these
  functions outside of salt [1]_ [2]_, meaning, precious time and
  resources must be wasted duplicating these abstraction in other
  projects.

  Use of the ``__salt__`` built-in however, is done to alleviate a more
  complicated issue of issuing asynchronous calls, cyclic dependencies
  and race conditions.

  Eventually, you will deal with a concurrent plugin architecture
  and bang your head on the table dealing with load order issues,
  dependencies, and you may being to travel down this path.

  To save a lot of people's time, lets play out the psychodrama a bit:

  pyvim could avoid a lot of hassle very quickly by using a built-in
  dictionary instead of interacting with raw python objects.

  No need to worry about catching import exceptions due to issues
  loading plugins out of order on startup, or during runtime.
  With asynchronous tasks, we could make available a dunder
  ``__pyvim__['run']('format_buffer')``. Also we have no need
  to care much about offering a stable internal API, plugin
  developers are only passing strings and hashes around.

  Also, that may be invoking a task with unique environment
  characteristics we intentionally want to run in its own
  process for sandboxing. Maybe a future pair-coding plugin
  could level this, because a plugin using ``__pyvim__`` could
  just as well being routing info a remote pyvim client.

  What if you want extensions that only run for python buffers?
  Well having a __pyvim__ object dictionary helps you avoid
  more headaches and fail silently without having everything break.

  I feel there may be the temptation for pyvim to not skirt away
  from complicated issues with dependencies and instead opt for
  a vanilla system of emitting and subscribing events via raw
  strings instead of interacting with real python objects.

- Performance
  Issues about python's performance are often overstated.

VIM
---

- VimLanguage
- Whackamole: Where's that behavior coming from?
- Bug reporting plugins
- Pyrrhic success
- C
- Autotools: Personally, I've grown to like the system, as well as CMake.

Neovim
------

http://dilbert.com/strip/2008-09-03

- Remote plugins

  Huge diversion on many levels.

  Core developers lose because instead of cleaning up and gutting out vim
  internals, they're now maintaining this infallible, pure msgpack rpc
  libuv system. Now we're so invested in this remote approach, the project
  ends up betting the farm over something that may be a failure.

  Users and plugin developers get screwed.  The whole remote plugin thing is
  a huge hand wave. Instead of getting tight integration and making your
  bread and butter (plugin devs) satisfied, let's create this short wave
  radio to bounce radio waves off the ionosphere in hopes things get through.

  If you want to do remote plugins, you're #1 job should be making sure it is
  unnoticeable to both plugin developers and users.

  I could already ``import vim`` in vim by configuring with it! And apt-get
  vim will already work with python. So now I need to install neovim on the
  system for pip 2 and 3 and virtualenvs?

  The problem of making a good plugin API gets pushed around. Now you need
  the python client, the lua client.

  Another red herring, making plugins still is miserable. Genering a decent
  bug report impossible, Is it a shortcoming of vim's plugin architecture?
  Well we'll never see it directly, its like inferring gravitation waves,
  we're so far from the machinery, we're left wondering whether its our
  python environment, our python client api version, the users environment,
  the neovim msgpack api limitation, we never see vim's internal handling,
  which in itself holds its own debt and complexities, all masking what is
  the waddeling lack of decisiveness for plugins.
- Indecisiveness
  We a standard spec for plugins more than we need anything else.

  VimL would be tolerable even.
- Inheriting a huge legacy
- C99 and uncrustified C is still C
- Lua
  This is the biggest fad for game developers and others trying to integrate
  a scripting language into the programs.

  It is my opinion people pick Lua based on 2 false premises:

  - Performance: The performance you're getting wouldn't matter. A lot of that
    overhead is either in more expensive processes in the core program or in
    the way the lua/python/js/viml/whatever script is written.

    Using lua isn't going to make tarpit of a plugin architecture perform
    better.

  - Better than VimL

    VimL is far more active than lua, despite it being picked by popular games
    such as World of Warcraft, Civ5 and window managers like awesome, vim
    still has more people interacting with it.

    The reality is thinking that problem is with VimL as a language and not
    the architecture of vim.

    There is reason to take issue as programmers with having the burden of
    a specialized language that serves no purpose outside of configuring an
    editor.

    lua isn't really any better in this regard. In the unlikely occasion you
    actually are using lua in more than one application, the syntax issue
    isn't the hard part, it's working with how they abstracted the main
    application to the scripting language.

    as an example, Awesome window manager broke configurations all the time
    for quite a few years. they just didn't care about deprecation warnings,
    the warning was for you when your config broke after upgrading it. and
    the convention to script widgets changes everywhere.

    what is idiomatic lua? we have pythonic, pep8, javascript, idiomatic.js.

  - It's respected

    Lua's respect is more memetic than anything.  Developers have a case where
    they may be in a pinch and just tack it on, see the note on integration
    below.

    Developers tack it on to C and C++ applications because its a lot easier
    than integrating python. They buy into the "benchmarks" [9]. Strangers
    come forth with purported stealth projects running these highly world
    changing systems, this sort of stuff is programming snake oil and pure
    hype.

    

  - Easier to integrate: For the core maintainers, yes, but what about your
    extension developers?  A lot of Civfanatics miss the python days of
    Civilization 4, which later switched to Lua in Civilization 5.

    As of 2016 May 20th the neovim plugins available (deoplete, lldb.nvim) are
    all in python.

    lua doesn't have anything to offer as a language over JS and Python. Even
    if it objectively was more powerful and concise, its hard to beat the
    value of a language people already know.

    In United States, there is a holiday joke about a special cake made of
    fruits and nuts traditionally given during Winter holidays. [3]_

    The late Johnny Carson, a comedian in USA, coined the joke "The worst gift
    is fruitcake. There is only one fruitcake in the entire world, and people
    keep sending it to each other."

    That is what lua is. A fruitcake. Those nuts and fruits are chalked full
    of rich nutrients and protein, lots of fiber to help you digest. Lua has
    "such a novel codebase" and "luajit can perform 2000x better than python".
    Well, that's great, people still aren't going out of there way to code lua.
    They're barfing it out and trying to put this "architecturally superior"
    language to the back of their mind, never to write anything useful it.
    Civ5 extensions sucks compared to Civ4. Neovim's viml to lua translator is
    also failing, a good sign that its not as straight forward as thought. [6]_

    eventually you're hoping
    to use luajit, from someone who's actually interacted with it with vim
    plugins, its really fun to use luajit and just flat out segfault with no
    useful debug symbols. [4] Also, its important to understand lua is being taken
    a language to supplant VimL, even going as far as to write a translator [5]_

    The bigger picture is, eventually with lua, you're going to be dealing with
    the same exact pitfalls python faces. You still have to deal with packages
    and paths in lua. [7]_


Atom
----

- Black swan


Values
------

- Python first


Maybes
------

- GUI

.. [1] https://gist.github.com/tony/6d8d975c817d2e4d43dd
.. [2] https://github.com/saltstack/salt/issues/22842
.. [3] http://www.villagevoice.com/restaurants/a-short-history-of-fruitcake-6429393
.. [4] https://github.com/jeaye/color_coded/issues/26 and https://archive.is/aa3xt
.. [5] https://github.com/neovim/neovim/pull/243
.. [6] https://www.reddit.com/r/neovim/comments/3jl1gd/is_the_viml_lua_translator_stalled/
.. [7] http://www.inf.puc-rio.br/~roberto/docs/jucs-c-apis.pdf
.. [9] http://lua-users.org/lists/lua-l/2011-10/msg00825.html
