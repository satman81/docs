---
sidebar_label: Using Apps with Nym
description: "Tutorials for building Privacy Enhanced Applications (or integrating existing apps with Nym)"
hide_title: false
title: Using Apps with Nym
---

import useBaseUrl from '@docusaurus/useBaseUrl';
import ThemedImage from '@theme/ThemedImage';



Nym is a general purpose system. We aim to provide the strongest possible protections for internet traffic and transactions.

The system is still very young, but it's starting to be able to do useful work. You can start using it today.

Many existing applications are able to use the SOCKS5 proxy protocol. They can use the `nym-socks5-client` to bounce their network traffic through the Nym network, like this:

<!-- ![Socks5 architecture](/img/docs/nym-socks5-architecture.png) -->
<ThemedImage
  alt="Overview diagram of the Nym network"
  sources={{
    light: useBaseUrl('/img/docs/nym-socks5-architecture.png'),
    dark: useBaseUrl('/img/docs/nym-socks5-architecture-dark.png'),
  }}
/>

The Nym network already runs the mixnet, and the `nym-network-requester` and `nym-client` components. In order to use existing applications with Nym, you only need to set up the `nym-socks5-client`.

:::note
If you have not yet set up a local socks5 client, follow the instructions [here](/docs/stable/develop-with-nym/socks5-client) before continuing. 
:::

Note that the nym-network-requester we're running works only for specific applications. We are not running an open proxy, we have an allowed list of applications that can use the mixnet (currently Blockstream Green, Electrum, and KeyBase). We can add other applications upon request, just come talk to us in our dev chat. Or, you can [set up your own](/docs/stable/run-nym-nodes/nodes/requester) `nym-network-requester`, it's not very hard to do if you have access to a server.

The Nym SOCKS5 proxy, though, does something quite interesting and different. Rather than simply copy data between TCP streams and making requests directly from the machine it's running on, it does the following:

* takes a TCP data stream in, e.g. a request from a crypto wallet
* chops up the TCP stream into multiple Sphinx packets, assigning sequence numbers to them, while leaving the TCP connection open for more data
* sends the Sphinx packets through the mixnet to a nym-network-requester. Packets are shuffled and mixed as they transit the mixnet.
* nym-network-requester reassembles the original TCP stream using the sequence numbers, and makes the intended request.
* nym-network-requester then does the whole process in reverse, chopping up the response into Sphinx packets and sending it back through the mixnet to the crypto wallet.
* The crypto wallet receives its data, without even noticing that it wasn't talking to a "normal" SOCKS5 proxy.