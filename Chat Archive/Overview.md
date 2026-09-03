# Chat Archive Overview

This folder preserves project design conversations that provide historical context for decisions documented elsewhere in the GNet knowledge base.

## [[Cosx vs Twiter Pair GNet Chat]]

Discussion of early GNet bootstrap, discovery, addressing, and terminal networking. It develops the idea that link-local broadcast should be minimal: a host uses a generic `DISCOVER` protocol to find foundational infrastructure such as a router, receives directed replies and network configuration, then uses routed directory services for higher-level discovery. The discussion also separates physical hub attachment numbers from logical routed addresses, keeps hubs deliberately dumb, considers ROM-booted GNet terminals and terminal-session services, and explores smart-card-based user authentication with directory/authentication services rather than embedding identity in the link layer.
