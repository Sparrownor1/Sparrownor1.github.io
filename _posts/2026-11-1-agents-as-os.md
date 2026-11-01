---
title: Musings on AI Agents and Operating Systems
---

<!--  
- Agents are beginning to look very similar to OSs, both in usage and in shape
  - Agents are more widely being used as a primary means of interacting with computing and the internet 
  - There are lots of patterns we can see repeated 
  - There are probably lessons we can learn from OS design to further improve agents

- shell is now the agent chat box, but semantic
- address space/memory
  - code is now prompt? what is system prompt?
  - context is address space? 'working memory'
  - skills are akin to dynamically loading code
  - working memory looks different to agents, it's in the filesystem as opposed to being backed by the same stuff that code is
  - 'virtual' context?
- agent is a task/process or thread
  - isolated 'address space' 
  - a2a begins to look like IPC
  - if agent is a task, what is a thread?
- tool calls are syscalls
  - the way action is taken outside of the isolation boundary
- what are model weights?
- GPU is the CPU?
  - we see a fetch-(decode)-execute happening at a higher layer of model-tool call
- harness manages all of the above, including creation of new agents, managing context, exposing functionality, it is the kernel!
  - except we are right now in an age of one agent per harness, so concepts like scheduling aren't relevant yet - they happen below the harness
  - is the concept of a multi-agent harness interesting at all?
-->

AI agents (and the LLM stack in general) are beginning to develop similarities to traditional operating systems, both in usage and architecture — they are on track (we hope) to be one of the primary ways people interact with computing and the internet, and looking at the software in and around agents, we can see a lot of familiar patterns emerge, modulo the additional behaviors LLMs bring of understanding natural language and nondeterminism (woohoo!). 
I hope to examine some of these common shapes, and put forth some questions about any further lessons we can learn from the mature architecture of OSes (OSs?) and bring to designing agents.

## Interfaces - The Shell and GUI

~~In the old days~~ Before the GUI, people used to interact with computers via text-based devices, like the [TTY](https://en.wikipedia.org/wiki/Teleprinter) or serial console, which provided shell-like functionality to control a machine. 
We are still in this phase with LLMs, chatbots, and agents, primarily interacting with them through a text stream, where we send and receive *semantic* commands and their output. That being said, the ability to input audio, visual, and video data is a versatile benefit!

> I like to think of steering models as sending semantic 'signals'! Of course, no steering is as reliable as a Ctrl-C in moments of panic or dread.

The GUI moment has yet to arrive for agents, though there have been debates as to whether text is All You Need, or whether richer interfaces still have a place in tomorrow's artificially intelligent computers.

It is easy to reach for the exciting notion of GUI eventually meaning *Generative* UI.
Glimmers of this idea appear as we generate HTML documents to make more digestible what would be hard-to-parse text walls from LLMs, but the scope can grow fantastically larger.
As a middle ground, we may have *interpretable* UI, backed by frameworks of components that can be programmed against and render at runtime, but one can even imagine a world where every pixel on your display is determined by a diffusion process running every 30ms.

## Processes

An individual **agent** seems to look most like a **process**. It is an instance of an individual running prompt 'program' that has isolated context and its own ability to take action externally via tools ('syscalls'). It has the ability to `fork` off new subagent 'children', with the same context or an entirely new prompt (like `exec`). Moreover, we are seeing agent-to-agent (A2A) messaging crop up that resemble traditional inter-process communication (IPC) layers!

Of course, asserting totally that agents are parallels to processes would bring me unease — there are definitely some fuzzy boundaries here. 
For one, what is really isolated? 
An agents entire 'address space' (more on this later) could be considered to be the context *and* working memory that is has, but working memory, often stored externally, is often shared! 
This is one argument to believe agents are like threads, but this could also be chalked up to be 'shared memory'. 
That being said, we don't yet have the behaviors of a sharing prompts (programs) or obvious risks of races among agents (apart from harness bugs), which are almost core ideas of threads. 

> It is more likely that agent threads (heh) are closer to being a parallel to OS threads, but I still think there is exciting room for design and exploration here!

## Syscalls

**Syscalls** are the way action is taken by a process outside of the isolation boundary (sometimes), that usually require some privileged access to shared state to complete. This definition is pretty faithfully met by traditional **tool calling**.

## Kernel

Just as the **kernel** is responsible for carrying out privileged operations on behalf of processes, managing shared state, and process lifecycle, the **harness** owns similar functionality in agentic systems.

One responsbility that the harness doesn't take on is scheduling, which usually delegated to be closer to GPUs. 

## Programs

In this world, we can treat **prompts** as **code**! 
If we proceed with the emerging pattern of simply adding 'semantic powers' to existing operating system elements to get AI-native versions, we see that this falls out naturally.
A program is an exact specification of behavior for a computer, while a prompt is a semantic one. Instead of programming languages, we use spoken ones!

> *Model weights* are also a specification of behavior. They could be considered equivalent to the 'kernel' program!

We can think of the **system prompt** and **skills** as encapsulated prompt bodies, akin to **libraries** and frameworks. Progressive disclosure of skills even looks a lot like **dynamic linking**!

A few open questions and ideas here:
- Will we have parallels to *binaries*? 
As of now, everyone is writing the equivalent of interpreted shell scripts — extensive functionality usually requires lengthy prompts. 
There is no room for optimization in speed or size, and proprietary behavior is distributed very hesitantly and is constantly at risk of being exfiltrated. 
It could be very beneficial to be able to have either compressed, optimized, or obfuscated versions of prompts. 
- Of course, the existence of a binary format would necessitate some parallel definition of an 'instruction set' and 'machine code' that can be understood by LLMs, is ideally more terse, and yet somehow cannot be regurgitated. 
We see shadows of this in encrypted thinking traces. 
Maybe we will even have *compilers*!
- Are thinking traces considered part of the prompt? 
How do they fit in to the program story?
- Are LLMs running prompts perhaps *self-appending* programs? 
We could consider outputs to be either working memory (not part of the program at all) or appendages to the prompt. 
And we could maybe consider the 'program counter' to also be jumping around semantically rather than serially.

## Address Spaces

Now on to **address spaces** and **memory**. 
Interestingly, *there is no clear* parallel for an address space in the agentic systems of today. 
An address space contains all the working memory of a given process: this includes everything from executable code, to read-only data, to zeroed-out data that stores variables. 
Additionally, one is backed by the same 'matter' throughout — individually addressable slots of bits and bytes.
Instead, these kinds of memory in agents are spread across a variety of mediums — the prompt and context, databases, as well as in the filesystem!

For the sake of ideation, perhaps a more convenient parallel to draw might be to **context**. Just as memory is a finite resource that is be used up by programs, context is somewhat finite as well. Here, an immediate question that comes to mind is if we can build **virtualized context**, where the *illusion* of having the entire context is created, while not all of the data (perhaps parts of the KV-cache) is quickly accessible. Perhaps there is some spatial locality here we can take advantage of to 'page' the KV cache and 'fault' parts of it in when necessary.

## Further Musings

- How do model weights fit in here? Are they part of the 'kernel'? Should they be considered code? They are, after all, a part of the behavior specification and static.
- What is the 'CPU'? There is a parallel execution engine that is split across the LLM, GPU, and harness. There is almost a fetch-execute cycle going on, where the LLM 'fetches' the syscall to execute, and the harness executes it.
- Shared caches

---
