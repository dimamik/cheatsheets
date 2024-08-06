## Erlang Code replacements

### Links

[Code loading (erlang docs)](https://www.erlang.org/doc/system/code_loading)

```
Erlang supports change of code in a running system. Code replacement is done on the module level.

The code of a module can exist in two variants in a system: current and old. When a module is loaded into the system for the first time, the code becomes 'current'. If then a new instance of the module is loaded, the code of the previous instance becomes 'old' and the new instance becomes 'current'.

Both old and current code is valid, and can be evaluated concurrently. Fully qualified function calls always refer to current code. Old code can still be evaluated because of processes lingering in the old code.

If a third instance of the module is loaded, the code server removes (purges) the old code and any processes lingering in it is terminated. Then the third instance becomes 'current' and the previously current code becomes 'old'.

To change from old code to current code, a process must make a fully qualified function call.
```

### Example

```erl
%% File: old.erl
-module(m).
-export([loop/0]).

loop() ->
    receive
        code_switch ->
            %% m:loop/0 is a fully qualified function call
            %% vs loop/0 is not, so the states will differ
            m:loop();
        Msg ->
            io:format("Received old message: ~p~n", [Msg]),
            loop()
    end.
```

```sh
erlc old.erl
erl
1> l(m).
{module,m}
2> Pid ! hello_world.
Received old message: hello_world
hello_world
```

```erl
%% File: new.erl
-module(m).
-export([loop/0]).

loop() ->
    receive
        code_switch ->
            m:loop();
        Msg ->
            io:format("Received new message: ~p~n", [Msg]),
            loop()
    end.
```

```sh
erlc new.erl
erl
1> l(m).
{module,m}
2> Pid ! hello_world.
Received old message: hello_world
hello_world
3> Pid ! code_switch.
4> Pid ! hello_world.
Received new message: hello_world
hello_world
```
