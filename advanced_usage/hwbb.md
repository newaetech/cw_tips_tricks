# Hardware Bit-Banging 

**Supported Capture Hardware:**\
❌ CW-Nano\
❌ CW-Lite\
❌ CW-Pro\
✅ CW-Husky

**Required ChipWhisperer software installation:**\
❌ not released yet \
✅ **requires develop branch**
<!-- TODO: update on next (post SCA101) release -->


## What Is Hardware Bit-Banging and What Can It Do For You?

We have several demo notebooks to demonstrate the usage and capabilities of
{py:class}`scope.bitbanger <chipwhisperer.capture.scopes.cwhardware.ChipWhispererHuskyBitBanger.BitBanger>`,
Husky's hardware bit-banging module. These notebooks show
`scope.bitbanger` being used to "talk" 
[SWD](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/demos/husky/10%20-%20Husky%20Hardware%20SWD%20Bitbanging.ipynb), 
[1-wire](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/demos/husky/11%20-%20Husky%20Hardware%201-Wire%20Bitbanging.ipynb),
[and I2C](https://github.com/newaetech/chipwhisperer-jupyter/blob/main/demos/husky/12%20-%20Husky%20Hardware%20I2C%20Bitbanging.ipynb),
with different targets, and capturing power traces synchronized to the SWD,
1-wire, and I2C communications.

If you haven't seen those notebooks yet and are asking yourself "what *is*
hardware bit-banging and what could it do for me?", let us answer with a
picture: hardware bitbanging lets you drive a serial interface with
precise timing (that's the hardware-driven part); this allows you to collect
very interesting power traces such as this (click to expand):

```{figure} ../Images/i2c_trace3.png
:width: 100%
:align: center
I2C Power Traces.
```

The top two lines are the hardware-bit-banged I2C lines (clock and data),
the overlapping blue and green analog waveforms are power traces collected
from a locked vs unlocked target, and the bottom blue analog waveform is the
amplified difference of those two sets of power traces.

If you have bit-banging needs, go to the notebooks linked above to learn
more! The notebooks focus on what `scope.bitbanger` can do. After you've
gone over them, come back to this note, where we focus a bit more on what
`scope.bitbanger` *can't* do, to help you better understand what it's best
suited for.

## What It Can't Do
The most important thing to understand is that while `scope.bitbanger` is
incredibly flexible and powerful, it won't generally work well if there is
variability in the timing of the target's response (e.g. when the target
controls the timing of its response).

`scope.bitbanger` is meant for protocols where you can say something to the
target, then get a response with *perfectly predictable timing*. It can also
be useful in cases where you don't care about the target's response.


That's because `scope.bitbanger` gets programmed to listen to the target on
very specific clock cycles, and it can't be told to wait. If the target gets
to choose *when* it responds, then `scope.bitbanger` is not the right tool
for the job. So for example, UART wouldn't generally work, because while
`scope.bitbanger` can be used to send out a UART sequence to the target, we
generally don't know when the target will choose to respond.
`scope.bitbanger` does not have a protocol state machine that can wait on
the target. If you need to trigger on a UART response that will occur at
some unknown time, then {py:class}`scope.UARTTrigger
<chipwhisperer.capture.trace.TraceWhisperer.UARTTrigger>` is the right tool
for you -- *not* `scope.bitbanger`.

Another example is I2C: while `scope.bitbanger` can talk with I2C
targets, it won't work with I2C targets that use clock stretching.
(Clock stretching is a flow control mechanism where the target holds the
clock line low, to give itself more time to respond. Husky isn't alone in
not supporting it: many devices don't.)

## It's *Very* Low-Level
If it's not already apparent to you, it's very important to understand that
`scope.bitbanger` is *very* low level - you need to understand the protocol
you're talking to all the way down to the "bits on the wire" level in order
to use it. Our demo notebooks hide this complexity, thanks to the
{py:class}`SWD <chipwhisperer.capture.scopes.cwhardware.ChipWhispererHuskyBitBanger.SWDHelper>`,
{py:class}`1-wire <chipwhisperer.capture.scopes.cwhardware.ChipWhispererHuskyBitBanger.OneWireHelper>`, and
{py:class}`I2C <chipwhisperer.capture.scopes.cwhardware.ChipWhispererHuskyBitBanger.I2CHelper>`
helper classes that we provide. If you can use these
as-is, great; if not, you will need to do some homework and understand how
to talk with your protocol down at the bit level.

We've gone through a lot of effort to make `scope.bitbanger` as flexible as
possible. You should be able to implement many protocols beyond the SWD,
1-wire and I2C examples that we demonstrate in our notebooks.


## What It *Can* Do
If you haven't already, have a look at all the demo notebooks linked above;
they show lots of interesting example, and are written so that you can
follow along even if you don't have the target hardware required to actually
execute the notebooks.

One useful feature not highlighted in those notebooks is its triggering
flexibility: `scope.bitbanger` can be programmed to emit any number of
triggers, at any point during the bit-bang sequence, all in one go.

One possible use of this is to combine multiple `scope.bitbanger` triggers with 
{py:meth}`scope.adc.segments <chipwhisperer.capture.scopes._OpenADCInterface.TriggerSettings.segments>`
to very efficiently capture all the power trace samples that you may
be interested in. This can be very useful when there is a lot of data to
bit-bang (for example: sending the RP2350 debug key with
{py:meth}`scope.bitbanger.SWDHelper.set_dbgkey_rp2350()
<chipwhisperer.capture.scopes.cwhardware.ChipWhispererHuskyBitBanger.SWDHelper.set_dbgkey_rp2350>`).

This multiple triggering capability can also be applied to glitching: if you
would like to glitch at multiple times during a bit-banged conversation,
it's very easy to do.

## Where It May Not Be Necessary
If you only need a power trace at the end of the communication, then
software-driven bit-banging via either 
{py:class}`scope.io <chipwhisperer.capture.scopes.cwhardware.ChipWhispererExtra.GPIOSettings>` (for pins on the [20-pin connector](../../Capture/20-pin-connector.md))
or 
{py:class}`scope.userio <chipwhisperer.capture.scopes.cwhardware.ChipWhispererHuskyMisc.USERIOSettings>` (for USERIO pins)
may be sufficient. It will be considerably slower, and you will lose
`scope.bitbanger`'s triggering flexibility, but in many cases the edge
trigger module can be suitable. Much (but not all!) of what is shown in the
`scope.bitbanger` demo notebooks could also be done with this simpler
approach. But do keep in mind that the edge trigger can only fire once per
arming.


If you do anything cool with `scope.bitbanger`, [let us know about
it!](https://forum.newae.com).
