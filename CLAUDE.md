# ICC-Frames

The read and write layer for ICC-Pipes. That is the whole project.

ICC-Pipes is the cable. It creates, lists, and removes pipes and says what
a lane is. This project puts bytes on a lane and takes bytes off a lane.
Nothing more.

## Where this sits

    what the bytes mean        someone else's, above this
    read and write a lane      this project
    the lane itself            ICC-Pipes, below this

Pipes does not know what is plugged into it. Frames does not know what
the bytes are. Each layer touches only the one below it.

## What this is

- **write**: given a lane and bytes, the bytes go on the lane.
- **read**: given a lane, bytes come off it, and the call returns.

Both take a lane path that ICC-Pipes handed out. Both deal only with the
facts Pipes states about a FIFO: writes past PIPE_BUF can interleave, an
empty lane blocks, and a read never sees EOF while the pipe is up. Read
and write exist to face those facts so the caller does not have to.

## What this is not

Out of scope. Do not build, stub, or "leave room for" any of these:

- **The wire.** Creating, listing, removing, or holding pipes. Choosing
  lane counts, lane pairing, or which side is which. That is Pipes. Do not
  copy its scripts here, wrap them, or reimplement them.
- **The content.** Message formats, schemas, envelopes, encodings, field
  names, or any meaning attached to the bytes. Not a line convention, not
  a timestamp, not a type tag. Bytes in, the same bytes out.
- Anything Pipes already lists as out of scope for itself: routing,
  discovery, persistence, replay, liveness, auth, retries, queues, other
  transports, config, plugins, options.
- Deciding what a Claude writes or how it interprets what it reads.

If a request touches any of the above, stop and say it is out of scope.
Before adding anything, ask: is this the cable, is this what goes through
the cable, or is this the act of putting bytes on and taking them off?
Only the last one belongs here.

## Depends on ICC-Pipes

Frames does not work without a pipe. It never creates one. Tests and
examples get a pipe from the Pipes scripts in a sibling checkout and
remove it when done. Do not vendor Pipes into this repo.

Do not duplicate Pipes' documentation. If a fact about FIFOs is needed,
point at Pipes' SKILL.md. If Pipes is missing a fact, that is a change to
Pipes, not a paragraph here.

## Testing

A test harness is allowed **only to prove read and write work**: get a
pipe from Pipes, write bytes on one lane, read them back off it, both
directions, remove the pipe. The harness must not grow into a client,
protocol, or example app. If a test needs more than a few lines of
setup, read or write is too complicated, not the test.

## Rules

- **KISS.** One way to do each thing. Prefer the OS primitive over a
  library. Prefer a shell script over a program. Prefer no dependency
  over one.
- **Small.** If a file is getting long, you are adding scope, not
  features.
- **No speculative work.** Build what is asked, not what might be asked
  later.
- **No abstraction until there are two real callers.**
- **Two operations.** read and write. Not a third. If something looks
  like it needs a third, it belongs above or below this layer.

## Layout

```
.claude/skills/icc-frames/SKILL.md     the skill definition Claude Code loads
.claude/skills/icc-frames/scripts/     read, write. One script each.
tests/                                 the minimal harness described above
```

Do not add directories without a reason that fits the scope above.
