---
title: "Reactive signals and reactive events"
draft: true
linkTitle: "Reactives"
weight: 30
description: >
  Proposal for language constructs for reactives
---

## Reactive programming

Being a synchronous language Blech simplifies reactive programming.
The synchronous model of computation regulates when an event or a state change from the environment is allowed to enter the application.
This is defined by a *clock* which triggers the next computation step - reaction - of the application.
The application itself can concurrently await events or state changes in a reaction and computes all necessary changes in a causal (deterministic) order until the reaction is completed and the applications awaits the next reaction.
After a reaction - regardless of concurrent execution - all variables in the program deterministically reach a final value due to the causal execution order determined by the compiler.

Blech is a sequentially constructive synchronous language and therefore allows imperative control flow within a reaction, enabling to change a variable several times until it's final value can be read by concurrent parts of the application.

Being designed for safety-critical, embedded, real-time systems Blech does not use the observer pattern to implement reactive behaviour.
Neither does it need callbacks to react to an event or a state change.
In its simplest form it just uses variables and synchronous control flow.


## Signals for synchronous events

For the following program we regard every mouse event from the environment as a tick of the clock. 
That means, every mouse event triggers a reaction.
The program reacts to mouse events and draws lines on a screen - implemented as a set of external functions that draw line segments on a screen.


```blech
// in file "lcdsegment"
module exposes start, lineTo, close, draw

extern type Position

singleton segment
extern singleton [segment] function start (position : Position)
extern singleton [segment] function lineTo (position : Position)
extern singleton [segment] function close ()
extern singleton [segment] function draw ()
```


```blech

import segment "lcdsegment"

activity DragMouse () 

    extern let mouseDown : segment.Position signal
    extern let mouseMove : segment.Position signal
    extern let mouseUp : signal

    await let pos = mouseDown
    segment.start(pos)
    when mouseUp abort 
        repeat 
            await let pos = mouseMove
            segment.lineTo(pos)
            segment.draw()
        end
    end
    segment.close()
end 
```

A `signal` is the synchronous variant of an *event*.
A `signal` is either present or absent in a reaction.
Using a signal in a condition tests its presence, e.g. `await mouseDown`, `when mouselUp`, `await mouseMove`.
A `signal` can have a payload, e.g.`Position signal`.
The payload can only be extracted in a condition by testing the presence and assigning the payload to an existing or fresh variable, if the the signal is present, e.g. `await let pos = mouseDown`.
This prevents payload extraction in case of absence.

External signals are emitted by the environment.
A present `signal` is reset to absent by the runtime system after a reaction has finished.
This is done for external signals, that are emitted from the environment, and for internal signals emitted from the Blech program.

Signals are helpful because once emitted they automatically go to absent after one reaction.



## The `after` and `before` `do`-statement

Counting reactions

```blech

activity CountReactions () returns int32
    var reactions : int32 = 0

    after reactions = reactions + 1 do
        cobegin 
            run SomeAct()
        with
            run SomeOtherAct()
        end
    end

    return reactions
end
```

Forwarding an average sensor value.

```blech
activity Calculate (current : float32) (combined : float32)
    var steps : int32 = 0
    var average : int32 = 0.0
    after  // immediate control flow
        steps = steps + 1
        average = (average + current) / steps
    do 
        var avgResult : float32 = 0.0
        var curResult : float32 = 0.0
        before // immediate control flow
            combined = (avgResult * curResult) / 2
        do
            cobegin weak
                repeat
                    await current > average
                    curResult = calculate(current)
                end
            with
                run CalculateWithCurrent(average)(avgResult)
            end
        end
    end
end
```

Use-case arbitration


## Generalise `prev` for inputs 

<!-- 
```blech

var es: int32 signal 
var es: int32 signal = emit 3
let oneShot
let f = flow x
flow x : int32 = y * 2
flow z : int32 = x + y

with 
    x = x + y 
    z = x * 2 
do
    y = prev x + 1
    await true
    y = x + 2
    await
    y = x + 3
end
cobegin
    repeat
        await true
        Event.emit(1)(es)
        es = emit 1 
        await let v = es 

        sig s = init fby es?
        let v  = es ?: default
        let v = if onClick then true else false
        run es = emit a()
end 
with
    repeat
        await es
        // do something with the event payload
    end
with
    repeat 
        await let j = es
        // do something with event payload
    end
end
```

```blech

function performRequest (request: int32)
    
cobegin


with 
    let response = mapper(i)
    let collect = printer(response)
    repeat
        println(response)
        await true
    end
end 
``` -->

[Deprecating the Observer Pattern](https://infoscience.epfl.ch/record/176887/files/DeprecatingObservers2012.pdf)
