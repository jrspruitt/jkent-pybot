jkent-pybot
===========

This is a modular, plugin-based IRC bot written in Python.  Plugins can be dynamically loaded and unloaded at runtime.  A design goal is the abillity to develop plugins without being able to crash the bot.

Plugins have a simple, easy to pick up API.  All events, commands, and triggers use a simple decorator convention.

Clone with:  `git clone --recursive https://github.com/jkent/jkent-pybot.git`

### Dependencies

Pybot will run under Python 2 and Python 3.  It has only one external dependency, `six`.  This package can be found under most package managers as `python-six` or `python3-six`. You can also install six using `pip install six`.

### Configuring

You need to copy `config.ini.sample` to `config.ini` and edit it to your liking.

### Running

Pybot is a python package, which means you need to run it as `python pybot` or `python3 pybot`.  Yes, you need to provide the path to the pybot package itself.

### Plugins

#### anyurl

This plugin will fetch and reply with the og:title or title of a HTML document.


#### base

This plugin handles some of the core behaviors of the bot, such as setting the nick, joining channels, and auto-reconnect.  Its required, please don't unload it unless you know what you're doing.


#### choose

A fun yet frustrating plugin that gives random responses. 

    should I|<nick> <question>?
    !choose a or b, c.


#### config

This plugin allows live editing of the config.ini file.  Usage is limited to level 1000.

    !config reload
    !config save
    !config set <section> <option> <value>
    !config unset <section> <option>
    !config list <section>


#### debug

This plugin prints all IRC traffic and module events while loaded.  Usage is limited to level 900.

    !raw <message> - send a raw IRC message


#### eval

This is a dangerous plugin that allows arbitrary execution of python code.  Usage is limited to level 1000.

    !eval <code>


#### gateway

The gateway plugin allows for linking between services such as Slack and Discord.  See [bridge applications](#bridge-appplications) for info on bridges.

    [gateway]
    secret = <random key here>
    routes = irc:#a:discord:#b discord:#b:irc:#a

Routes are in the form of `source_realm:#source_channel:dest_realm:#dest_channel`.  A route creates a 1 way "transport" for messages.  The gateway plugin realm is `irc`.

    !gateway reload - reloads the routes


#### math

The math plugin is a nifty calculator that has support for functions and variables.  Its state is saved in a database as workbooks which can be switched out as needed.

    !math [expr]
    !math var=[expr]
    !math func([var[, ...]])=[expr]
    !math workbook [name]
    !math varlist
    !math funclist
    !math describe <funcname> [description]


#### notes

Leave a message for someone when the bot next sees them talk.

    !tell <nick> <message>


#### perms

Manage bot permissions.  Usage is limited to level 1000.

    !perms list
    !perms allow [-][<plugin>=<n>]
    !perms deny [-][<plugin>=<n>]

Where plugin is the name of a plugin and n is the level to set.  Plugin can be the special constant ANY.


#### plugin

Load, unload, reload plugins at runtime.  Usage is limited to level 1000.

    !plugin load <name>
    !plugin reload [!]<name>
    !plugin unload [!]<name>
    !plugin list


For reload and unload, the "bang" means force.  Use with caution.

## Bridge applications

The bridges directory includes bridge programs for communicating with the gateway plugin.  They have configuration files of their own, with example config.py.sample files.

## For Developers

### Plugins
Here's a simple "Hello world" style plugin:

    import * from plugin
    
    class Plugin(BasePlugin):
        @hook
        def hello_trigger(self, msg, args, argstr):
            msg.reply('Hello %s!' % argstr)

You would call the trigger on IRC via either:

    !hello world

or if directed (conversational) style triggers are enabled:

    pybot, hello world

To which the bot would reply:

    <pybot> Hello world!

### Hooks
There are five types of hooks:

  * event
  * command
  * trigger
  * timestamp
  * url

All except for timestamp hooks can be used via the `@hook` decorator.  `@hook` is a smart decorator that uses the naming convention of your method to determine the name and type of the hook.  Alternatively, it can be called as `@hook(names)` and `@hook(type, names)`.

Timestamp hooks can be created 3 different ways: one-shot timeouts, one-shot timers, and repeating intervals.  They are discussed in more detail with the Bot class.

### Bot class
Anything that you may need to access should be accessable from the bot class.  Plugins get a reference to the *bot instance* they are running on (`self.bot`).

var          |description
:------------|:-----------
`channels`   |A dict with keys being channels, value is a dict with keys 'joined' and 'nicks'
`core`       |The core instance the bot is running under
`hooks`      |An instance of the HookManager class
`nick`       |A string identifying the bot's current nickname
`plugins`    |An instance of the PluginManager class
`allow_rules`|Allow rules for the permission system
`deny_rules` |Deny rules for the permission system

method                              |description
:-----------------------------------|:-----------
`set_interval(fn, seconds[, owner])`|Install timestamp hook, calls `fn` every `seconds`
`set_timeout(fn, seconds[, owner])` |Install timestamp hook, calls `fn` after `seconds`
`set_timer(fn, timestamp[, owner])` |Install timestamp hook, calls `fn` at `timestamp`
`join(channels[, keys])`            |Convenience method for JOIN
`notice(target, text)`              |Convenience method for NOTICE
`part(channels[, message])`         |Convenience method for PART
`privmsg(target, text)`             |Convenience method for PRIVMSG

### Hook class
method             |description
:------------------|:-----------
`bind(fn[, owner])`|Binds a hook in preparation to install

### EventHook class
### CommandHook class
### TriggerHook class
### TimestampHook class
### UrlHook class

### HookManager class *(the hook manager)*
method              |description
:-------------------|:----------
`install(hook)`     |Install a bound `hook`
`uninstall(hook)`   |Uninstall `hook`
`call(hooks, *args)`|Call hooks using as many args as possible
`find(model)`       |Search for hooks by model hook instance
`modify(hook)`      |Context manager for modifying *installed* hooks

