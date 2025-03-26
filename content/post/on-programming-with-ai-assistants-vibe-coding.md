---
title: "On Programming with AI Assistants"
description: "I skip Vibe Coding. I treat LLMs like smart junior devs: great for research & repetitive tasks, not final code. This helps my focus, reduces context switching, and improves my coding quality of life."
date: 2025-03-26
tags:
 - programming
 - ai
 - llm
---

[Vibe Coding](https://en.wikipedia.org/wiki/Vibe_coding), the new hot buzzword, describes the act of programming _through_ an LLM, rather than _with_ it. It's akin to using AI to _compile_ natural language (like English) into a programming language. This can be empowering; suddenly anyone can create programs that make the computer work for them better. Of course, there are several problems with this, such as LLMs generating literal copies of code from open-source projects, without any license warning, or outright wrong code. I think that taking whatever output an LLM gives and using it directly in a product is not a good idea. However, if it's for a simple tool you're creating to help you do something, it's absolutely valid in my opinion.

Instead, I prefer to treat LLMs as a junior developer. This junior developer is knowledgeable, capable of doing extensive research very quickly, and providing results (mostly quantitative results), but lacks real-world experience. I wouldn't trust it with any real code, but I gladly take the opportunity to ask for assistance. This includes asking about standard library functionality, inquiring about suitable 3rd party libraries to solve a problem, asking for the correct usage of a module, or seeking help understanding an issue. Sometimes I might not even realize that I'm dealing with a classic problem that's been discussed and has several proposed solutions. LLMs are also pretty good at diagnosing issues, obscure compiler errors, and weird behaviors.

I think that the reason you use generative AI is to do things you think it would do a better or faster job than you. That means that either you think you're not good enough to do it, or you need the extra help to get it done. I think it's important to understand the difference. I can see how much harm generative AI can do (and is already doing) with people delegating creative work to it, but I try to focus on the transformative effect on education and learning. Using it as an assistant that helps you get things done and learn from it is the single most important distinction that can make or break the positive impact of AI.

I've been using Copilot Chat for a few months now as an occasional pair-programming buddy. I found that code completion is not really for me; when I'm writing code, I'm in _write_ mode, if I have to stop and read the suggestion it has made for me, it often breaks my chain of thought, not to mention the extra mental effort to figure if that code is copyrighted or has other licensing issues. This isn't to say that I never use generated code, I've had cases where I asked the assistant to change code for me or finish some repetitive code, like asking it to complete the content type function on my [webserver implementation](https://github.com/CrociDB/ush):

```haskell
data ContentType
    = TextPlain
    | ApplicationOctetStream
    | TextHTML
    | TextJavascript
    | TextCSS
    | ImageJPEG
    | ImagePNG
    | AudioMPEG
    | AudioOgg
    | Other String
    | Undefined String
    deriving (Show)

contentTypeToString :: ContentType -> String
contentTypeToString TextPlain = "text/plain"
contentTypeToString (Undefined _) = "text/plain"
```

I just fed this asked it to complete all the possible values of `contentTypeToString` and, in a matter of seconds, I had:

```Haskell
contentTypeToString :: ContentType -> String
contentTypeToString TextPlain = "text/plain"
contentTypeToString ApplicationOctetStream = "application/octet-stream"
contentTypeToString TextHTML = "text/html"
contentTypeToString TextJavascript = "text/javascript"
contentTypeToString TextCSS = "text/css"
contentTypeToString ImageJPEG = "image/jpeg"
contentTypeToString ImagePNG = "image/png"
contentTypeToString AudioMPEG = "audio/mpeg"
contentTypeToString AudioOgg = "audio/ogg"
contentTypeToString (Other str) = str
contentTypeToString (Undefined _) = "text/plain"
```

A key advantage of this approach, particularly when combined with good editor integration, is the minimization of context switching, consider the [Doorway Effect](https://en.wikipedia.org/wiki/Doorway_effect), where changing environments can cause you to forget details relevant to the previous environment. The same applies your screen windows. Constantly switching between my code editor and a browser window to search for solutions disrupts my mental flow. Being able to open a chat with the LLM and ask about what's going on without even taking my hands off the keyboard has been incredibly beneficial. 

It's not about raw productivity; I'm not overly concerned with speed. Instead, it's about _quality of life_. By staying within the editor and interacting with the LLM directly, I minimize mental interference and can think more clearly about the LLM's suggestions. This reduced cognitive load leads to better, more considered decisions about how to implement the solution and significantly less stress overall.

