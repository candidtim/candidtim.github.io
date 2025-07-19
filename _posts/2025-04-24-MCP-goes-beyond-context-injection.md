---
layout: post
title:  "MCP goes beyond context injection"
date:   2025-04-24
categories: mcp ai
description: >
  Is MCP just a content provider with a fancy name? I argue it's much more - it
  is at the core of AI-Agent architecture.
---

[Model Context Protocol](https://modelcontextprotocol.io/introduction) attracts
[a lot of interest](https://trends.google.com/trends/explore?date=today%203-m&q=mcp&hl=en-GB)
in the software development community. Naturally, an easy way to understand
MCP is to start with MCP server implementations, by using or implementing
them. But, focusing on this alone can make it appear like a context pipe for a
chat. MCP, however, does much more. With its core concepts - [hosts, servers
and clients](https://modelcontextprotocol.io/docs/concepts/architecture)  - MCP
arguably provides a good abstraction for building **AI Agents**.

![Chat vs Agent architecture with MCP](/assets/mcp/chat-vs-agent.png)

Before talking about AI agents, though, let's have a closer look at some
characteristics of the core MCP concepts: †

- Indeed, an MCP host - a program that uses LLM - can use any number of MCP
  servers. This is the obvious part: connecting Notion and Google Drive to
  Claude, or letting Cursor access a Postgres DB and GitHub tickets.
- An MCP host is also not limited to a single LLM - it can use any number of
  LLMs, each for its own purpose.
- MCP servers can expose
  [Resources](https://modelcontextprotocol.io/docs/concepts/resources) and
  [Tools](https://modelcontextprotocol.io/docs/concepts/tools), providing
  information but also rendering things actionable. Ultimately, an MCP host
  controls the use of LLM and how to expose the tools to it.
- And finally, MCP does not dictate what a client program *does*, beyond the
  fact that it can somehow use the MCP servers. Most crucially, a client
  doesn't have to be a chat! Most of the time we *think* of a chat because it
  is a popular way we interact with LLM, but generally speaking a client need
  not expose any textual interface at all. Typically, a client would implement
  a set of domain-specific features, beyond answering questions or generating
  text. Well, as any useful software.

For example, imagine an AI Agent that scans your Todoist lists, identifies
items that need a meeting planned with other people, exchanges with your peers
over Slack or e-mail to agree on a suitable time slot, and books it in your
agendas. ††

![Meeting Planner Agent](/assets/mcp/meeting-planner-agent.png)

What is MCP then? As in "*protocol*", it really sits right in the middle of
everything. Whereas some platforms can expose their already existing services
via MCP server implementations, new software leveraging the LLMs can use **MCP
as a glue between the three**.

In some way Model Context Protocol is a misnomer, its name doesn't do it
justice. Although technically correct, it enables the implementations that go
beyond piping more context to chat interfaces. **MCP is what enables the AI
Agent architecture**.

Personally, I have found that writing an MCP server is the most intuitive way
to learn more about it. Creating new client applications on the other hand is
about inventing new use cases and new services, which is naturally harder, but
arguably more exciting. I am keen to see more of these in the future!

--

Footnotes:

- † I use the word "client" in its most generic sense, as in "client-server
architecture". Technically, in MCP, it is a "host", but since Anthropic also
[confuses the two](https://modelcontextprotocol.io/clients), I allow myself a
more traditional use of the word.

- †† It is worth noting that there exist [important security
concerns](https://elenacross7.medium.com/%EF%B8%8F-the-s-in-mcp-stands-for-security-91407b33ed6b)
due to which it is not yet possible to give such an agent full autonomy.

