# Node.js  Tooling Group Meeting 2024-04-04

## Present

* Darcy Clarke: @darcyclarke
* Erick Wendell: @erickwendel_
* Jacob Smith: @jakobjingleheimer
* Joyee Cheung: @joyeecheung
* Luke Karrys: @lukekarrys
* Marco Ippolito: @marco-ippolito
* Matteo Collina: @mcollina
* Michael Dawson: @mhdawson
* Paolo Insogna: @ShogunPanda
* Rafael Gonzaga: @RafaelGSS
* Richard Lau: @richardlau
* Robert Nagy: @ronag
* Ruben Bridgewater: @BridgeAR
* Ruy Adorno: @ruyadorno
* Stephen Belanger: @qard
* Tobias Nießen: @tniessen
* Vinícius Lourenço Claro Cardoso: @H4ad
* Yagiz Nizipli: @anonrig
* Wes Todd: @wesleytodd

## Agenda

### Intro

The tooling group is a group that has been put together in 2017 and since then has been trying to bring together different people from user-land to collaborate and propose upstream improvements to the node runtime.

A few examples of the work that came out from this group in the past are file system recursive read, argument parser, etc.

This group met for the last time in the OpenJS Collaborator Summit 2023 in Vancouver.

### http static server

- WIP PR: https://github.com/nodejs/node/pull/45096
- Tobias: single-command http server like the one that python has. We should think about our subcommand strategy
- Matteo: I am one of the people objecting to this PR because I think if we are landing a static file server we should land something that conforms to the http standard and it should be performant. And TBH node.js is pretty slow at serving files today. We should provide something similar to the ? module of express.
- Robert: I had a send file API.
- Matteo: That was what I was saying, we need a lower level API that does all the MIME types etc. right.
- Robert: does libuv implement sendFile Linux API?
- Richard: no
- Tobias: we should mention in the doc that this command not for production, just for testing and for people who don't want to install something from npm to just do this
- Ruy: Just from the web dev experience point of view it's valuable to have a minimal, developer-oriented, not production ready, not-peformant API that's baked into the runtime that we can use
- Qard: send file does exist in libuv as of ??
- Jacob: during web developemt a lot of the times the browser really need you to serve this file from a server (instead of file:://?) and I think this would help a lot of people
- Yagiz: is there a champion to pursue this
- Ruy pointed to Antoine

### node run

- WIP PRs:
  - https://github.com/nodejs/node/pull/52267
  - https://github.com/nodejs/node/pull/52190
- Yagiz starts an intro of the `node run` v.s. `node --run` debate
- PR of implementing `node run`: https://github.com/nodejs/node/pull/52190
- Tried a PR to reserve the `node run` but there are blocks: https://github.com/nodejs/node/pull/52267
- Yagiz: I think --run is bad for UX
- Rafael: I don't think we'll be able to find a good solution until we release 22
- Yagiz: reserve any extension-less positional argument as a subcommand, also compatible with ESM resolution
- Tobias: it would break `run/index.js` or `test/index.js` for the case of a `node test` sub-command
- Richard: we should go through a deprecation cycle
- Matteo: I would be better to ship it as `node --run` because it's backportable, especially 20, there will be shorter adoption cycle. In parallel we can deprecate `node run` on 22.
- Yagiz: I am okay with a deprecation cycle if folks think it would work better
- Ruben: I don't believe that we should just break people. Is there a downside on keeping both?
- Darcy: previously there were concerns about not documenting this very well
- Joyee: reading the Deno doc recently and noticed that we don't have a page for tutorial of command line interface. We should have a TL;DR page for begineers, instead of only having an alphabetically sorted page. Not volunterring just putting the idea out there.
- Wes: We can just document that `node run` doesn't do all the package manager does. And make it explicit that it's intentional. I am now leaning more towards the flag than the subcommand. People do run `node test` which is `$cwd/test.js`. I think that's a pretty cool feature and adding subcommands would break that workflow. I think it's enough to justify a flag.
- Yagiz: the limitation is already documented in the PR. Geoffrey insisted and we added it.
- Ruy: what don't we support right now?
- Yagiz: I want to move it to C++ eventually so am not trying too hard in the current JS-based implementation. Like recursive bin directory support (?)
- Darcy: ?
- Wes: 
- Yagiz: it's a slippery slope to document "what we don't support but npm supports" because whenever npm adds something we'd need to update our docs
- Ruy: Using `--run` also sets an expectation to users that this is a different implementation from `npm run`
- Micheal: we should just be explicit about a specific list of things we support and not going to support everything
- Luke: we (npm) can contribute to the documentation. It's been very difficult to document the entire surface. It would be helpful for the contributors to know if they don't need something they can use the built-in command.
- Ruy: any more concerns
- Wes: I think we need to think through about the consequences of having the whole .bin on your PATH
- Micheal: I think it's good to not have the package manager to run the scripts. Not sure if I am thinking about the same use case that you brought up
- Wes: I think anyone using npm start in producion is wrong. And `node [--]run` should be the right way to do it. We should figure out how to pave the way for people to do it right. I am thinking more about using it for developent time.
- Micheal: do we have a common understanding about what the target of this command is?
- Matteo: I have projects that say spawn 5 commands in one scripts and I think being able to save 1 sec (5\*200ms) would be quite valuable. We really need to document this though. Especially how we increase the V8 space limits. Also our doc by default tells people to do npm start. We should change that.
- Darcy: sounds like there's consensus here that npm should run faster. It would be cool if we can contribute back to the existing tool for it to be more performant.
- Luke: there is an interesting to be discussed about shared primitives. Things like walking through the directory tree to find package.json. It would be great if Node.js can provide performant primitives for this.
- Joyee: someone in the node run PR posted a HackerNews thread where someone analyzed why npm run is slow and AFAICT most of that is tied to how npm run works. Also someone from npm mentioned in the thread that npm needs to be able to install on top of itself so they can't do lazy loading. I think most of the slownees is application logic and not too much that Node.js can help with.
- Wes: It's only fast when the command itself is fast. If the command is already very slow then 200ms speedup is not very significant. And in real world people tend to run very slow commands.
- Wes: about `npm` not being able to bundle itself, maybe one possible thing is to avoid doing a global install of `npm`. I think that may be pretty promising to help them speed it up.
- Matteo: For me it can save me 1 sec so I think it's meaningful.
- Matteo: people should just not install `npm` globally because the permission is difficult. This includes our own installer.
- Ruy: there's one last comment from Tierney in the PR. What it's supposed to be? It should be a subset, different from npm. I think the scope is better defined now.
- Vinicius (from the chat): it's a bad idea to ship another binary to handle the subcommands? like we have npx or corepack?
- Tobias: I think that is just a terrible idea. Just do that as node2.

### Wrap up

- Ruy: Conclusion, hopefully a lot of progress was made to the current `node --run` implementation ([ref](https://github.com/nodejs/node/pull/52190)). This group is a great place to collaborate on issues like this and hopefully we can get back on having regular calls in the future.

## Upcoming Meetings

* **Node.js Project Calendar**: <https://nodejs.org/calendar>

Click `+GoogleCalendar` at the bottom right to add to your own Google calendar.

