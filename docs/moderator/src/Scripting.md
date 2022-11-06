
## But First, Some Context
 
Languages such as `Lua`, `Python`, `JavaScript` (and family) have very straightforward ways to create plugin systems or hot reload code. Many of these languages also have wrappers available that allows them to be easily integrated with `C`-compatible codebases. This opens a door to a very wide range of language integrations, but usually with relatively high runtime cost for context switching and/or data exchange.
 
On the other side of the coin, `C` uses [ABI](https://en.wikipedia.org/wiki/Application_binary_interface) to allow loading of dynamic code. In Unix-land that's usually a `.so` file, or a `.dll` on windows. This allows us to integrate `C`, `C++` and `Rust` code directly. The big advantage in this case is that, differently from the languages mentioned above, this kind of integration has very little extra runtime cost, if any. On the downside, this type of integration is highly technical for both sides of the integration: base program development and plugin development. There are other downsides to this, but I guess complexity, security and stability covers most of it.
 
## Rust in Rust Integration and Security
 
Integrating `Rust` directly in your game through a `cdylib` opens it up to literally run arbitrary `Rust` code (huh, who would've guessed?). This means the plugin code can do all sorts of nasty things, such as (and not limited to):
 - Run `unsafe` code blocks
 - Create references that could degrade performance and parallelism
 - Heap or stack overflow your game
 - Read and write files in the host FS
 - Collect and send arbitrary data to servers (e.g. make HTTP requests)
 - Download and execute malware
 
This is due to the fact that sandboxing this kind of code is practically impossible. We could ship the game with a modified standard library to try and stop people from making HTTP requests, but that's easily circumvented by using LDPRELOAD or any of the other multitude of ways. It is arbitrary code after all!
 
But again, this is an almost zero runtime cost way to integrate code, and also the most powerful one as one could possibly overwrite entire game systems with relative ease.
 
With all of the above said, `Rust` in `Rust` is a great option with the assumption that the code comes from trusted sources. This makes this type of integration ideal for things like DLCs or any official expansions.
 
On the other hand, allowing arbitrary code to easily be executed outside of a sandbox is always a huge problem. Still it's easy to find examples of big games with big mods that use that type of integration. If you ever played the Enderal Skyrim mod you probably remember you had to download and run the `Enderal Launcher.exe` in order to play with the mod, even if this kind of integration was not directly supported in the game. The same can be said about Stardew Valley and the SMAPI mod tool.
 
This probably has liability implications and I am not a lawyer. But from a practical point of view, having ABI support is not inherently bad, but directly distributing untrusted code could be. If players have to go to a third party and download an unofficial tool for these mods, edit game files and etcetera, it'd be unreasonable to blame the game for malicious code execution.
 
## Scripting Language Integration
 
## Lua (and some others)
 
The performance of something such as `Lua` will not be as good as directly integrating a dynamic library, but will be enough for most game modding use cases. `Lua-JIT` helps a lot, but may be unavailable on some platforms (e.g. some mobile or console).
 
Integrating a scripting language such as `Lua` has the big advantage of it being easily sandboxed. Of course, by easily sandboxed I mean at least in the sense of File Systems and HTTP requests. Most of these languages are not memory safe and [`Lua` is not an exception](https://www.lua.org/bugs.html). But it is still a lot better than giving full access to the host's operating system.
 
### Mun
 
That takes me to [Mun](https://mun-lang.org/). Mun has a lot of potential, even if it is not yet production ready. It is statically typed and AOT compiled (in contrast to Lua's JIT compilation). The language is experimental; very active yet very young piece of tech. Still, if being a bit experimental fazed you off you'd probably not be reading this in the first place. The entire Rust gamedev ecosystem is built on experimental tech.
 
Pros:
 - Statically typed
 - Well integrated in Rust
 - Ahead of Time (AOT) compilation
 - Good performance for the use case
 - Very close to Rust (no null, explicit error handling)
 - Easy Hot-Reloading (runtime cost, useful for development and may be disabled for releases)
 - Active Development
 
Cons:
 - C ABI (needs unsafe, `unsafe { my_mun_runtime.update() };`)
 - Does not solve the sandboxing issue
 - Experimental
 - High context switching cost
 
If security is not a major concern, and you expect players to download random DyLib files and a random mod manager like many games do, Mun is a very strong contender. I'll definitely give it a real try soon.
If security IS a concern, if you're going to distribute untrusted code and would need it to be properly sandboxed that takes us to...
 
 
### WebAssembly (WASM)
 
WebAssembly is sandboxed. That's basically what it does, it runs fast and secure. It was designed from the ground up to protect users from malicious plugins/modules, and that makes it the perfect candidate for this type of extension.
You can distribute untrusted code locked within its sandbox, and you can define what that sandbox allows or doesn't.
 
Pros:
 - Sandboxed, this is the only one in this list that actually reduces possible attack surface
 - Language support, modders could use many languages, Python, go, even Rust, as long as it supports WASM as a target
 - Built to extend, plugin/module work is a first-class use case of WebAssembly
 - Good performance for the use case (generally)
 
Cons:
 - Watch the sandbox, badly configured sandboxes can open your host to arbitrary code
 - More limited Host-Mod communication
 - No threading ([yet?](https://github.com/WebAssembly/threads))
 - Some crates won't work on the mod side (threading, file i/o, etcetera)
 - High context switching cost
 
 
### Comparison
 
With this absolutely scientific and statistically relevant blurb of not anecdotal words with 100% objectivelly deterministic metrics, this is the proven truth given you by science!
 
|   |Fast|Safe|Ergonomic|Stable|
|---|---|---|---|---|
|Rust cdylib|X| | |X|
|Lua||X|X|X|
|Mun|X| |X||
|WASM|X|X|X||
|Java*||Log4J||
 
 
With this, the choices are clear:
 
If you want Safe, Easy to use and Stable, Lua is for you (also python, lisp or other sandboxable languages).
 
If you want Fast and Easy to use, but do not care for Safe or Stable, Mun takes it.
 
If you need Fast and Safe, WASM is for you.
 
If you're a masochist and want to fight the ABI on every single platform you're supporting... If you like to over-engineer every single aspect of your code and want to squeeze out that extra 5% FPS with manual `cdylib` loading... If you're a sadistic host that wants all your modding community to learn how to create cross-platform dynamic library code... you have my blessing. You can not use Rust in Rust with the C ABI. Now go. Preach the way.
 
 
<sup>* - just wanted to bash Java</sup>

