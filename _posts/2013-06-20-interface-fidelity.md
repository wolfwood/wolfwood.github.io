---
layout: post
---
# {{ page.title }}

When people speak of digitization, of media, the humanities, the
world, it is usually with an implicit nod to fears regarding the
fidelity of the digital artifact.  The process of producing such
artifacts is [quantization][quant].

[quant]: http://en.wikipedia.org/wiki/Quantization_(signal_processing) "quantization"

![Quantization Error](http://upload.wikimedia.org/wikipedia/commons/2/22/Quanterr.png)

Regardless of encoding method, given enough bits (upping bits per
sample or sample rate) we can arbitrarily bound the error.

Writing a library, Operating System, etc. that offers an interface can
present comparable fidelity issues. There is the source interface
(hardware in the case of an OS, a drawing primitive library like Cairo
for a window toolkit like GTK+) and the 'destination' interface: the
interface that the software presents to yet higher levels of software.

There is no inherent reason to expect that the two levels of interface
present the same level of expressive power, in fact we often write
libraries specifically to constrain expression, often under the banners
of simplicity and/or correctness.

We can think of a higher-level, destination interface as a compression
of a low-level source interface. While matters are complicated but
state that may be hidden in the interface, the intuition is that if
every source symbol has a corresponding destination symbol, the
compression is lossless, and the interfaces are of equal
expressiveness. A necessary but not sufficient shortcut for these kind
of examinations is: if the number of states (symbols) for the
low-level interface as grater than for the high-level one, they cannot
have the same expressive power.

The XOmB kernel attempts to provide lossless interfaces to hardware.
It does so with two important design feature.  The first is
statelessness, this simplifies matters because the only state is that
which resides in the source interface.  The second is direct exposure
of the hardware to the greatest extent possible. the combination of the
two make a comparison of expressiveness trivial, because the
interfaces are the same. This is in stark contrast so POSIX-compliant
systems, where the interfaces are so complex, stateful and dissimilar
to hardware that it is difficult to know where to start in trying to
make similar evaluations.

In short, XOmB's design is not a result of obtuseness or devotion to
an ideal, so much as laziness when it comes to trying to make formal
claims.  The expressive equivalence of XOmB is so obvious I debated
whether this post was worth writing at all. However, I think we need
to think more about how we make claims about interfaces, particularly
in light of Wilkie's [discussion of interface
design](http://blog.davewilkinsonii.com/posts/a-language-for-interfaces)
and our [joint frustration with the way we currently mis-evaluate
software designs](http://blog.davewilkinsonii.com/posts/kaashoeks-law).

<3
