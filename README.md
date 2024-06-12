<!-- Move text down -->
<br>

<!-- Header -->
<h1 align="center">The Wayland Protocol Documentation But Better</h1>

<!-- Subheading -->
<h3 align="center">A living document that tries to better explain the Wayland Protocol</h3>

<!-- Move text down -->
<br>

<!-- Move text down -->
<br>

# Table of Contents

- [Why?](#why)
- [What is Wayland?](#what-is-wayland-)
  - [What is a Wayland Client](#what-is-a-wayland-client-)
- [Welcome to The Wire(d)](#welcome-to-the-wired)
  - [Messages](#layer-0-whispers)
  - [Events and Requests](#layer-1-conduits)

<!-- Move text down -->
<br>

<!-- Move text down -->
<br>

# Why?

The **currently provided**, **official documentation**, **sucks**..a lot. A lot of **important** protocol level **information** is just **left out or just skimmed over**. Really the **WHOLE** website boils down to "We made Wayland! Here is a high level overview. K thx bye". This document will _hopefully_ provide **clearer insight** into the **Wayland specification** and how to **properly implement it**.<sup>[[1][link-source-1]] [[Trust Me][link-source-trust]]</sup>

<!-- Move text down -->
<br>

<!-- Move text down -->
<br>

# What is _Wayland_ ?

To keep it brief **Wayland** is the _new_ (first initial release in 2008<sup>[[2][link-source-2]]</sup>) **replacement** for the now **obsolete X Window System** (**X11**, or simply **X**). **Unlike X11**, who **tries to be everything** all at once, **Wayland's** only **goal** is to **provide a way for [clients](#what-is-a-wayland-client-) to draw their own windows** without having to worry about things like the DRM Kernel Driver or weird monitor configurations.<sup>[[3][link-source-3]]</sup>

---

### What is a _Wayland Client_ ?

A **Wayland Client** (or simply _client_ in this document) is **any application** that **implements** the **Wayland Client-Side Protocols**. These **_clients_** are **windowed applications** that **communicate** with the **Wayland Sever bi-directionally** (Client-Server and Server-Client) over the **[Wire Protocol](#welcome-to-the-wired)**.<sup>[[4][link-source-4]]</sup>

<!-- Move text down -->
<br>

<!-- Move text down -->
<br>

# Welcome to The Wire(d)

**The Wire Protocol** (not to be confused by **The Wired**<sup>[[5][link-source-5]]</sup>) is the **communication protocol** used by both **Wayland Servers and Clients**. Although, **communication** is done **via** a **UNIX Domain Socket** (mostly because of the high volume of data being sent) nothing (or no one) is stopping you from using **alternative means** like **TCP, UDP, Satellite, or [Smoke Signals][link-source-smoke-signals]**.<sup>[[6][link-source-6]]</sup>

### Layer 0: Whispers

**The Wire Protocol** revolves around a **stream of Messages** sent **to and from** the **server/client**. Each **Message** is **built** with the **following primitives**:

| Primitive Type |                                                                                                          Description                                                                                                          |           Overview           |
| :------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :--------------------------: |
|     array      |                          A **blob** of **arbitrary data**. **Prefixed** with a **32-bit integer** specifying its **length in bytes**. It is then **padded** to **32 bits** with **undefined data**.                           |   \<length>DATA\<padding>    |
|      enum      |                                                                          A **single 32-bit value** from an **enumeration** of a **known constant**.                                                                           |              0               |
|       fd       |                                                                                                        Refer to %LINK%                                                                                                        |            %LINK%            |
|     fixed      |                                                                                            A **24.8-bit signed fix point number**.                                                                                            |            0.247             |
|      int       |                                                                                                 A **32-bit signed integer**.                                                                                                  |              -1              |
|     new_id     |                                                                                 A **32-bit object ID**. Gets **allocated** when **received**.                                                                                 |              0               |
|     object     |                                                                                                  A **32-bit** object **ID**.                                                                                                  |              1               |
|     string     | A **UTF-8\*** string **prefixed** with a **32-bit integer** specifying its **length in bytes**. The **main string content** is **ended with a NUL terminator**. It is **then padded** to **32 bits** with **undefined data**. | \<length>DATA_NULL\<padding> |
|      uint      |                                                                                                A **32-bit unsigned integer**.                                                                                                 |              1               |

\* Although the protocol **_does not explicitly state_** what encoding should be used, **most people** end up using **UTF-8**. 😛

Each **Message**, sent or received, is **structured as a sentence** (Message Header + Message Payload). Each **sentence** is **made up of words** (Primitive types). The **Message header** is made up of **two words**. The **first word** is a **32 bit value** that contains the **affected ObjectID**. The **second word** is made up of **two letters** (16-bit values). The **upper 16 bits** is the **size of the message** (including the header). The **lower 16 bits** are the **event/request** Operation Code (also known as an **opcode**). After the **Message header** then comes the **payload/arguments**. It must be noted that **EVERY argument** must be **aligned to 32 bits** (**padding** must be done with **undefined data**) and that **ALL data** must be **encoded** in the **host's byte order**.<sup>[[7][link-source-7]] [[8][link-source-8]] [[9][link-source-9]]</sup>

### Layer 1: Conduits

<!-- Sources -->

[link-source-1]: https://wayland.freedesktop.org/docs/html/
[link-source-2]: https://cgit.freedesktop.org/wayland/wayland/commit/?id=97f1ebe8d5c2e166fabf757182c289fed266a45a
[link-source-3]: https://wayland.freedesktop.org/docs/html/ch01.html#sect-Compositing-manager-display-server
[link-source-4]: https://wayland.freedesktop.org/docs/html/apb.html#id-1.10.2
[link-source-5]: https://sel.fandom.com/wiki/The_Wired
[link-source-6]: https://wayland-book.com/protocol-design/wire-protocol.html#transports
[link-source-7]: https://wayland.freedesktop.org/docs/html/ch04.html#sect-Protocol-Wire-Format
[link-source-8]: https://wayland-book.com/protocol-design/wire-protocol.html#wire-protocol-basics
[link-source-9]: https://wayland-book.com/protocol-design/wire-protocol.html#messages
[link-source-trust]: https://i.kym-cdn.com/photos/images/original/002/051/481/93e.jpg
[link-source-smoke-signals]: https://github.com/MaxineToTheStars/the-wayland-protocol-documentation-but-better/blob/main/resources/smoke.png
