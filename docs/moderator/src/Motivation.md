# Motivation
 
This crate was created as a way to explore some more advanced features of the Rust language that I did not have a chance to use.
 
It tries to create a flexible and easy to integrate a mod / expansion / DLC pipeline for Bevy games. This means allowing mods not only to access existing components and resources, but also create new ones, add new systems, etcetera.
 
It aims to allow high moddability, allowing mod authors to touch even the core systems of the game; like Minecraft.
 
The idea is to have two levels of integration:
 - Scripting language
 - Direct Rust integration

More details on the language R&D coming up next.

# How will our mods work?

To be able to design how mods are going to be created, we first need to decide what is important for us.
For that I'll be using five different criteria, most important first: flexibility, ergonomics, performance, security and stability.

1 - **Flexibility** - how flexible the environment is when creating a mod.
- Best case: Modders can create and/or modify anything in the game, add new assets, creativity is the limit!
- Worst case: Modders have access to a couple data files where they can change some values; new content is limited to existing logic/fetures.

2 - **Ergonomics** - how easy to use the modding solution is.
- Best case: Modding environment is trivial to set up, has a very intuitive API, easy syntax, supports hot-reloading and other quality of life features for modders.
- Worst case: Modding requires jumping through complex hoops to set up the initial environment, has a very high entry barrier requiring understanding of complex concepts such as linking, compiling, LLVM.

3 - **Performance** - pretty self explanatory, how performant the mod runtime is.
- Best case: Same performance as the game's own code, no performance difference between the base game and the mods.
- Worst case: Performance is a lot worse than that of the base game, limiting what can be done in a mod and how many mods can run at the same time.

4 - **Security** - how safe a player is when downloading a mod from an untrusted source
- Best case: Mods are run in a completely sandboxed environment where they can't make web requests, file i/o or access the host system in any unsafe way.
- Worst case: Mods have complete access to `std` or are otherwise capable of spamming threads, making web requests or have full access to the host system.

5 - **Stability** - how stable and battle tested the solution is.
- Best case: Very stable, battle tested with a robust ecosystem.
- Worst case: Using a piece of tech that's still in a pre-alpha/preview stage.

Supporting scripting is a basic requirement, as flexibility is our main focus. With that in mind, we can ignore a data-only approach.
Now it's time to decide how we are going to enable scripting/programming in our mods, we have a few options:
 - Supporting a classic scripting language such as Lua or Python.
 - Supporting a binary interface such as the C ABI or WASM.
 - Creating our own scripting language, possibly based on an existing language's syntax... no

In the next section, we'll go a little bit further on the pros and cons of the ~~three~~ two options we have, and choose what we should use for an initial MVP.



