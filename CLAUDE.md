# ICC-Frames

The read and write layer for ICC-Pipes. That is the whole project.

ICC-Pipes is the cable. It creates, lists, and removes pipes and says what
a lane is. This project slices what you want to send into frames, puts
them on the lanes, takes them off the other end, and pieces them back
together in order. Nothing more.

## Where this sits

    what the bytes mean        someone else's, above this
    slice, carry, reassemble   this project
    the lane itself            ICC-Pipes, below this

Pipes does not know what is plugged into it. Frames does not know what
the bytes are. Each layer touches only the one below it.

## What this is

- **write**: given a pipe and bytes, slice the bytes into frames and put
  them on the lanes this side writes.
- **read**: given a pipe, take frames off the lanes this side reads, put
  them back together, and hand back the bytes.

What goes into write comes out of read, the same bytes in the same
order. A JPEG goes in as a JPEG and comes out as a JPEG, top at the top.
Order holds across every lane, not just within one. That is the whole
point of the layer. Pipes only promises order within a lane; Frames
promises it for the payload.

A **frame** is the unit. It is one write to one lane, at most PIPE_BUF
so it lands whole. Its header carries what read needs to put the payload
back in order and nothing else: where this piece goes, and where it
ends. If a field is not needed to slice or reassemble, it does not
exist. The frame is the only thing this project defines.

Both operations take a pipe directory that ICC-Pipes handed out. Both
open lanes with `<>`. That open never blocks, with or without anyone on
the other end, and a frame written through it returns with nobody
reading. The one thing that waits is a read on an empty lane, so read
is bounded. Nothing in this project blocks on open.

## What this is not

Out of scope. Do not build, stub, or "leave room for" any of these:

- **The wire.** Creating, listing, removing, or holding pipes. Choosing
  lane counts, lane pairing, or which side is which. That is Pipes. Do not
  copy its scripts here, wrap them, or reimplement them.
- **The content.** What the payload bytes mean. Message types, schemas,
  encodings, field names, timestamps, or any tag that says what a
  payload is. The frame header is not a place to sneak these in. Bytes
  in, the same bytes out.
- Anything Pipes already lists as out of scope for itself: routing,
  discovery, persistence, replay, liveness, auth, retries, queues, other
  transports, config, plugins, options.
- Deciding what a Claude writes or how it interprets what it reads.

If a request touches any of the above, stop and say it is out of scope.
Before adding anything, ask: is this the cable, is this what goes through
the cable, or is this slicing it up and putting it back together? Only
the last one belongs here.

## Depends on ICC-Pipes

Frames does not work without a pipe. It never creates one. Tests and
examples get a pipe from the Pipes scripts in a sibling checkout and
remove it when done. Do not vendor Pipes into this repo.

Do not duplicate Pipes' documentation. If a fact about FIFOs is needed,
point at Pipes' SKILL.md. If Pipes is missing a fact, that is a change to
Pipes, not a paragraph here.

## Testing

A test harness is allowed **only to prove read and write work**: get a
pipe from Pipes, write a payload bigger than one frame, read it back,
compare bytes, both directions, remove the pipe. The harness must not
grow into a client, protocol, or example app. If a test needs more than
a few lines of setup, read or write is too complicated, not the test.

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
- **One frame.** The header carries what reassembly needs and nothing
  else. It never grows to say what the payload is.

## Layout

```
.claude/skills/icc-frames/SKILL.md     the skill definition Claude Code loads
.claude/skills/icc-frames/scripts/     read, write. One script each.
tests/                                 the minimal harness described above
```

Do not add directories without a reason that fits the scope above.
