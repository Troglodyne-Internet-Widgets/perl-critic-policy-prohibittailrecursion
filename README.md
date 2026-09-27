# NAME

Perl::Critic::Policy::Subroutines::ProhibitTailRecursion - Do not call a sub from itself: Perl gives every call a stack frame.

# VERSION

version 0.001

# Perl::Critic::Policy::Subroutines::ProhibitTailRecursion

Perl does not optimize a tail call.  Every call makes a new stack frame, the
last call of a sub too, so a sub that calls itself uses one frame for each
level of its input, and a deep enough input fills the stack:

```perl
sub walk {
    my ( $node, @seen ) = @_;
    walk( $_, @seen, $node ) for children($node);    # violates
    return;
}

my $walk = sub {
    __SUB__->($_) for children( $_[0] );             # violates
};
```

Walk the input with a loop instead.  Push what is still to be visited onto an
array, and visit it in a c-style `for` loop, which sees what the loop pushes
while it runs.  A `foreach` over the array does not:

```perl
my @todo = ($root);
for ( my $i = 0; $i < scalar(@todo); $i++ ) {
    push( @todo, children( $todo[$i] ) );
}
```

## What it reports

Inside a named sub, a call to that sub by its name: `walk(...)`,
`walk @args`, `&walk(...)`, `&walk`, and `Pkg::walk(...)` in its own
package.  It also reports a call of the same name as a method on the invocant
of the sub, `$self->walk`, `$class->walk`,
`__PACKAGE__->walk` or `$self->Pkg::walk`, which is how a method
recurses.  A call anywhere in the body counts, in a loop, in the block of a
`map` or a `sort`, or in an anonymous sub, because each one adds a frame
when it runs.

Anywhere, a call through `__SUB__`: `__SUB__->(...)` and
`&{ __SUB__ }(...)`.

## What it leaves alone

- `goto &walk` and `goto __SUB__`.  `goto` replaces the frame of the
sub rather than adding one, so it is the one tail call that Perl makes cheap.
- `\&walk`, a reference and not a call.
- A call to the same name on any other object, and `$self->SUPER::walk`,
which can reach a different sub.
- A named sub declared inside another.  It is a sub of its own, and it
is checked on its own.
- Calls from one sub to another that calls the first.  This policy looks at
one sub at a time.

# CONFIGURATION

There is nothing to configure.

## METHODS

What [Perl::Critic::Policy](https://metacpan.org/pod/Perl%3A%3ACritic%3A%3APolicy) asks of a policy, answered here rather than called
from anywhere.

### supported\_parameters

None.

### default\_severity

Medium: the code works until an input is deep enough, and then it fails.

### default\_themes

`performance`.

### applies\_to

A named sub, for a call to it by name, and a word, for `__SUB__`.

### violates

One violation for each call of the sub to itself.

# AUTHORS

Current Maintainers:

- George S. Baugh <george@troglodyne.net>

# COPYRIGHT AND LICENSE

Copyright (c) 2026 Troglodyne LLC

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:
The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.
THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
