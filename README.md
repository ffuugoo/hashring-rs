# HashRing

[![Build Status](https://travis-ci.org/jeromefroe/hashring-rs.svg?branch=master)](https://travis-ci.org/jeromefroe/hashring-rs)
[![Coverage Status](https://coveralls.io/repos/github/jeromefroe/hashring-rs/badge.svg?branch=master)](https://coveralls.io/github/jeromefroe/hashring-rs?branch=master)
[![crates.io](https://img.shields.io/crates/v/hashring.svg)](https://crates.io/crates/hashring/)
[![docs.rs](https://docs.rs/hashring/badge.svg)](https://docs.rs/hashring/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/jeromefroe/hashring-rs/master/LICENSE)

[Documentation](https://docs.rs/hashring/)

A minimal implementation of consistent hashing as described in [Consistent
Hashing and Random Trees: Distributed Caching Protocols for Relieving Hot
Spots on the World Wide Web] (https://www.akamai.com/es/es/multimedia/documents/technical-publication/consistent-hashing-and-random-trees-distributed-caching-protocols-for-relieving-hot-spots-on-the-world-wide-web-technical-publication.pdf).
Clients can use the `HashRing` struct to add consistent hashing to their
applications. `HashRing`'s API consists of three methods: `add`, `remove`,
and `get` for adding a node to the ring, removing a node from the ring, and
getting the node responsible for the provided key.

## Example

Below is a simple example of how an application might use `HashRing` to make
use of consistent hashing. Since `HashRing` exposes only a minimal API clients
can build other abstractions, such as virtual nodes, on top of it. The example
below shows one potential implementation of virtual nodes on top of `HashRing`

```rust
extern crate hashring;

use std::net::{IpAddr, SocketAddr};
use std::str::FromStr;

use hashring::HashRing;

#[derive(Debug, Copy, Clone)]
struct VNode {
    id: usize,
    addr: SocketAddr,
}

impl VNode {
    fn new(ip: &str, port: u16, id: usize) -> Self {
        let addr = SocketAddr::new(IpAddr::from_str(&ip).unwrap(), port);
        VNode {
            id: id,
            addr: addr,
        }
    }
}

impl ToString for VNode {
    fn to_string(&self) -> String {
        format!("{}|{}", self.addr, self.id)
    }
}

impl PartialEq for VNode {
    fn eq(&self, other: &VNode) -> bool {
        self.id == other.id && self.addr == other.addr
    }
}

fn main() {
    let mut ring: HashRing<VNode, &str> = HashRing::new();

    let mut nodes = vec![];
    nodes.push(VNode::new("127.0.0.1", 1024, 1));
    nodes.push(VNode::new("127.0.0.1", 1024, 2));
    nodes.push(VNode::new("127.0.0.2", 1024, 1));
    nodes.push(VNode::new("127.0.0.2", 1024, 2));
    nodes.push(VNode::new("127.0.0.2", 1024, 3));
    nodes.push(VNode::new("127.0.0.3", 1024, 1));

    for node in nodes {
        ring.add(node);
    }

    println!("{:?}", ring.get("foo"));
    println!("{:?}", ring.get("bar"));
    println!("{:?}", ring.get("baz"));
}
```