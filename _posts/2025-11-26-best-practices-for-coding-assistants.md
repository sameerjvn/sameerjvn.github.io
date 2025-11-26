I've been trying out coding assistants more aggressively in order to see what value we can extract from them. So far I've tried Codium (before it got acquired), Github Copilot, Cursor, Claude Code and Amazon Q. Here are some lessons I've learned from using them:

## 1. Enable Tab Completions

I'm a huge fan of tab-completions by assistants like Codium and Copilot. In most cases, the assistants are smart enough to know where you're going and fill up the gap correctly. Especially useful for writing test cases for your code. If the assistant sees a pattern like `test__database_client__returns_404_on_page_not_found`, it's quite obvious what the next test should look like. This is where a broader rule applies which is being consistent within your code base. Follow coding conventions and make them explicit in the `README.md` or a `RULES.md` (More on this in the next section) so other developers and coding assistants know what clean code looks like.

There are definitely times where the assistant fills up a function or a class definition that isn't really what you want. But I've found that even in those cases, there will be snippets of logic that are salvageable. So I would consider tab-completions an overall win.

Unfortunately this is not a feature in all coding assistants. My favourite one so far is Claude Code but it doesn't have tab completions. You win some, you lose some.
    
## 2. Write a RULES.md for your repo

Different assistants do this differently, but almost all assistants read some file and pulls context when you start a conversation. Claude Code does it with a `CLAUDE.md`, Amazon Q reads `.amazonq/RULES.md`. Add as many instructions as you can here so it knows exactly how to code. 

I usually add naming conventions, code style guides, test instructions, as well as design suggestions like "Write functions with dependency injection so we can test using mocks" or "Use dataclasses for internal classes but Pydantic `BaseModel` for external-facing classes that need serialization". (At this point, consider using an LLM to clean up your prompts so they make sense to others).

One simple yet valuable lesson is to simply tell it the language and version you're using so you hint it in the direction you want it to go. Something like 

> Write C++ code for files in src/ and python code for files in test/

will prevent it from doing a `grep src/*.py` and save you time and tokens.

You could extend this section to another guideline that's broader than agentic assistants. Most humans read your repo and know how to complete your code based on patterns. However, code repos are owned by and made in collaboration with multiple developers who have different styles. This is not really a problem, but it's important to know what is. It's a slippery slope from mixing camel-case and snake-case, to having code that's unreadable and everyone's on vacation and the person who developed the code left the team 2 years ago.

It's easier to write clean code when everyone knows what it looks like, so add a `styleguide` section in your `README.md` for the humans as well.
    
## 3. Ask your assistant to document WHY before HOW
Ideally at this point you might have something like

>- Use docstrings for classes and public methods
>- Keep comments minimal - write self-documenting code
>- Document complex logic inline when necessary

in your `RULES.md`. While this is good general advice, I would argue that this is not enough. Honestly, in most cases, it's easy to understand how code works without comments. It's ideally evident from function names and type hints to identify what it's used for as well.

The most difficult part comes from _why_ it's designed that way. It might seem arbitrary to see a concurrency limit of 6 for a restAPI client until you increase it to 10 and it raises alerts in a different part of your organization because you've overloaded the CPUs on someone else's server that isn't built for high-availability. These situations are unfortunately common but avoidable. Make business logic and constraints explicit in comments or while creating docs by adding a line like:

> **CRITICAL: Focus on the "WHY" - the business purpose, problem being solved, and rationale behind design decisions. The "how" (implementation details) is secondary and should be included as a byproduct of explaining the why.**"

In your `RULES.md`
    
## 4. Trust relevant shell commands
This is a fantastic feature in Claude Code and a sort of dealbreaker in Amazon Q. 

To explain this a bit, the first time an agentic coding assistant suggests a shell command like `grep -r "APIClient"`, it asks for your approval to execute it. Since executing shell commands allows the assistant to do anything, coding assistants require your explicit approval. 

Amazon Q has a very safe approach where every command requires user approval. Claude Code is more permissive. You can allow certain "types" of commands to be whitelisted in a session so you don't have to approve a `grep` command repeatedly. I think this is extremely user friendly and intuitive. This is what I expect a smart coding assistant to be able to do: understand that certain commands are safe in certain contexts and allow the user to decide that. I haven't used Amazon Q enough to configure whitelisted commands but it feels like this should be something they provide right out of the box.

## 5. Index your repo locally
Almost every coding assistants does classic RAG: embed the codebase once, then search the embedding vectors for relevant context during generation. Codium, Cursor, Amazon Q all do this. However, Claude Code doesn’t. Instead, Claude Code `grep`s for relevant files and pieces of code.

To be honest, I'm still not sure about which approach is better. `grep`ing for relevant context is more intuitive and what I would do, and sometimes indexing is done with a poor embedding model, which means the codebase is stored "fuzzily", losing some meaning of the code. This pushes me towards choosing the `grep` approach.

However, one major issue with Claude Code is that it sometimes goes into tool-calling limbo where it spends more than a minute just searching for stuff and calling tools. Sit and count to 60 in front of a codebase while Claude Code searches for something. Instead search for it yourself. Which would you choose? This might be personal preference, but I think poor quality generation is better than long waiting times, which pushes me to recommend indexing the codebase. Again, different coding assistants do this differently, so check how to configure your coding assistant to index your codebase.

# Conclusion

Coding assistants are valuable tools, but not replacements for good software development practices. If you make your code well maintained and easily understandable for humans, coding assistants can help automate your processes and get you a long way. Good luck!
