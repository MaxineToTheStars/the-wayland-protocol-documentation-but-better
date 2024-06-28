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
  - [Interfaces and Operation Codes](#layer-2-hymns)
- [Connection Time](#jacking-in)

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

**The Wire Protocol** revolves around a **stream of _Messages_** sent **to and from** the **server/client**. Each **_Message_** is **built** with the **following primitives**:

| Primitive Type |                                                                                                          Description                                                                                                          |           Overview           |
| :------------: | :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: | :--------------------------: |
|     array      |                          A **blob** of **arbitrary data**. **Prefixed** with a **32-bit integer** specifying its **length in bytes**. It is then **padded** to **32 bits** with **undefined data**.                           |   \<length>DATA\<padding>    |
|      enum      |                                                                          A **single 32-bit value** from an **enumeration** of a **known constant**.                                                                           |              0               |
|       fd       |                                         A **0-bit** value on the **primary transport** that **refers** to a **file descriptor** on the **receiving end** using an **ancillary data**                                          |    Will be elaborated on     |
|     fixed      |                                                                                            A **24.8-bit signed fix point number**.                                                                                            |            0.247             |
|      int       |                                                                                                 A **32-bit signed integer**.                                                                                                  |              -1              |
|     new_id     |                                                                                 A **32-bit object ID**. Gets **allocated** when **received**.                                                                                 |              0               |
|     object     |                                                                                                  A **32-bit** object **ID**.                                                                                                  |              1               |
|     string     | A **UTF-8\*** string **prefixed** with a **32-bit integer** specifying its **length in bytes**. The **main string content** is **ended with a NUL terminator**. It is **then padded** to **32 bits** with **undefined data**. | \<length>DATA_NULL\<padding> |
|      uint      |                                                                                                A **32-bit unsigned integer**.                                                                                                 |              1               |

\* Although the protocol **_does not explicitly state_** what encoding should be used, **most people** end up using **UTF-8**. 😛

Each **Message**, sent or received, is **structured as a sentence** (Message Header + Message Payload). Each **sentence** is **made up of words** (Primitive types). The **Message Header** is made up of **two words**. The **first word** is a **32 bit value** that contains the **affected ObjectID**. The **second word** is made up of **two letters** (16-bit values). The **upper 16 bits** is the **size of the Message** (including the header). The **lower 16 bits** are the **event/request** Operation Code (also known as an **opcode**). After the **Message Header** then comes the **Message Arguments/Outputs**. It must be noted that **EVERY ARGUMENT/OUTPUT** must be **aligned to 32 bits** (**padding** must be done with **undefined data**) and that **ALL DATA** must be **encoded** in the **host's byte order**.<sup>[[7][link-source-7]] [[8][link-source-8]] [[9][link-source-9]]</sup>

### Layer 1: Conduits

Each **Message sent over The Wire** is **categorized** as either a **request or an event**. A **request** is a **Client-Server Message** (eg a client requesting for the current window to be shown). An **event** is a **Server-Client Message** (eg a server telling a client all available interfaces/protocols). Each **Message** contains a **signature** similar to the **example shown**.<sup>[[10][link-source-10]]</sup>

```sh
32 Bit Value                  : ObjectID
16 Bit Value + 16 Bit Value   : Message Length + Request Operation Code
Padded 32 Bit Value(s)        : Payload
```

### Layer 2: Hymns

The **heart of Wayland** can be **found here**:

```sh
cat /usr/share/wayland/wayland.xml
```

Each **Wayland Protocol** is **defined** in a **`.xml` file**. Each **file contains** a **series of interfaces**, each **interface contains** a **series of events and/or requests**, and each **event and/or request** has a **list of arguments and/or outputs**. In order for **any type of communication to happen** between **server and client**, some **ground rules** have to be established. Specifically that **`ObjectID: 1` is STRICTLY RESERVED for `wl_display`**. With **`wl_display`** being **bound to `1`** it allows for **clients to request `wl_display::get_registry`** at connection time. However, **how does the server know** which **request/event** we want? **Operation Codes** (or simply _opcode(s)_ in this document) allow us to **select which request/event** we are **tying our Message to**. Each **interface** has a **set of opcodes** to choose from. **Starting** from **0** all the way **to n**<sup>**th**</sup>, n being **the final request/event**.<sup>[[10][link-source-10]] [[11][link-source-11]]</sup> Take the following interface below:

```sh
<interface name="wl_registry" version="1">
  <request name="bind">
    <arg name="name" type="uint" />
    <arg name="id" type="new_id" />
  </request>

  <event name="global">
    <arg name="name" type="uint" />
    <arg name="interface" type="string" />
    <arg name="version" type="uint" />
  </event>
</interface>
```

The **interface `wl_registry`** has **two opcodes**. **Opcode 0** is a **request** that **calls `wl_registry::bind`** and **opcode 1** is an **event** that **calls `wl_registry::global`**.<sup>[[Trust Me][link-source-trust]]</sup>

<!-- Move text down -->
<br>

<!-- Move text down -->
<br>

# Jacking In

To **locate a UNIX socket** most implementations **follow what libwayland does**<sup>[[6][link-source-6]]</sup>:

1. If **WAYLAND_SOCKET is set**, **interpret** it as a **file descriptor**.
2. If **WAYLAND_DISPLAY is set**, **concat** with **XDG_RUNTIME_DIR** to **form the path**.
3. **Assume** WAYLAND_DISPLAY is **set to `wayland-0`** and **concat** with **XDG_RUNTIME_DIR** to **form the path**.
4. Give up (sudo rm -rf /\* --no-preserve-root)

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
[link-source-10]: https://wayland-book.com/protocol-design/interfaces-reqs-events.html#interfaces-requests-and-events
[link-source-11]: https://wayland-book.com/registry.html#globals--the-registry
[link-source-trust]: https://github.com/MaxineToTheStars/the-wayland-protocol-documentation-but-better/blob/main/resources/trust.png
[link-source-smoke-signals]: https://github.com/MaxineToTheStars/the-wayland-protocol-documentation-but-better/blob/main/resources/smoke.png
