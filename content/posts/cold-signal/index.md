---
title: "Cold Signal: A Text Adventure CTF Postmortem"
categories: ["Post", "Blog"]
tags:
  ["CTF", "Security", "Python", "Docker", "Game Design", "AI", "Claude Code"]
date: 2025-02-01
draft: false
images:
  - "post_photo.jpeg"
---

This is year two of building custom CTF challenges for [NoCo Hackers](https://nocohackers.com/), our local security meetup in Fort Collins. Last year I built [Horsetooth Liquidators](/posts/horsetooth-ctf/) -- a vulnerable e-commerce web app with eight challenges. This year I wanted to try something a bit different.

**Cold Signal** is a text adventure CTF. Players SSH into an abandoned Arctic research station, explore rooms, interact with the station's AI, and find flags by exploiting vulnerabilities embedded in the game. No browser, no web forms. Just a terminal and a mystery. You can [try it in your browser here](https://georgetipton.com/cold-signal).

<a href="https://georgetipton.com/cold-signal/"> 
<video autoplay loop muted playsinline style="width:100%; border-radius:8px;">
  <source src="images/broadcast-783.webm" type="video/webm">
</video>
</a>

![Cold Signal](images/cold-signal.png)

The idea was to make a CTF that was a little "outside the box". Ultimately, it worked in some ways and didn't in others, and I think the lessons are worth sharing.

<hr>

# Making CTFs Fun (and Where That Gets Complicated)

I grew up on video games, with things like GameSharks and Action Replays, allowing you to modify some memory values, and suddenly you could walk through walls. Those along with things like hardware modifications, automation bots, and ["lag switching"](https://freight.cargo.site/m/J1911089167207615754105035203739/TCP-s-Evolution_-From-Secure-Networks-to-Gaming-Exploits-in-Halo-2--1.pdf) were probably my first real exposure to the idea that systems have rules, and if you understand how the system works underneath, you can break those rules (or use the tools made by people that understood these things, at least). I think a lot of security people got their start through the same exposure. I've contemplated giving talks on the influence and overlap of security in gaming culture (and vice versa), and it definitely played a role in my decision to make this years CTF as much about a game as it is about the CTF challenges.

I also had also experienced some more inspirational and innovative CTFs in 2025 that not only achieved being a traditional CTF under the hood, but were fronted with well written, rich atmosphere and narrative, making the CTF more interesting to play and explore. [Johnny Reina's](https://github.com/jreina) CTF from LASCON 2025 was a good example of this with [IRC bots](https://github.com/jreina/lascon-badge-game-2025-irc-bots) that added "life" to the CTF with distinct personalities across channels, a [fake corporate website](https://github.com/jreina/lascon-badge-game-2025-the-corp) to investigate, and a world that was interesting to explore.

I've always been impressed by text adventures since Zork and Colossal Cave Adventure -- the idea that you can build real atmosphere with nothing but words. So I built [Cold Signal](https://georgetipton.com/cold-signal/): an abandoned research station, a crew that disappeared under mysterious circumstances, and an AI that's helpful but clearly hiding something.

{{< shortcode-gallery match="images/inspiration/*" sortOrder="asc" rowHeight="250" margins="5" thumbnailResizeOptions="600x600 q90 Lanczos" thumbnailHoverEffect="enlarge" showExif=false previewType="blur" lastRow="center" embedPreview=true loadJQuery=true >}}

The challenges were more so simulations of real vulnerability classes such as session fixation, path traversal, configuration manipulation, that were embedded in the game world. I also included a few extra flags like hiding a [spectrogram](https://github.com/Geo-Bit/cold-signal/blob/main/broadcast-783.webm) of the landing page's background audio for a steganography challenge outside the game itself.

| Challenge                  | Type                                                                                |
| -------------------------- | ----------------------------------------------------------------------------------- |
| **The Archive Chamber**    | Session fixation -- load a preserved session to inherit elevated access             |
| **VESPERA's Hidden Files** | Path traversal -- bypass access control via `../` before path normalization         |
| **The Listening Array**    | Config manipulation -- disable safety filters to reveal unfiltered signal data      |
| **Broadcast 783**          | Steganography -- decode a spectrogram hidden in the landing page's background audio |

<hr>

# The reception was lukewarm

Our annual CTFs are still small, maybe 10-20 participants and the people playing the most are experienced CTF players who are there to score points and win. For those players, the narrative was more decoration. The reality is CTF challenges should seek to _teach_ people something. Cold Signal was built to have people _experience_ something. Those aren't the same thing, and I didn't fully reckon with that distinction going in.

Where things broke down:

- **Narrative as friction** -- Competitive players didn't want to read room descriptions or parse dialogue for hints. They just wanted to find a vulnerability, exploit it, and move on.
- **Unfamiliar mechanics** -- Save states and access levels are natural game concepts, but they were confusing in a CTF context. Players didn't intuitively understand the systems they needed to exploit.
- **Red herrings** -- I added an interactive access card as atmospheric flavor. Players saw a tangible item that matched their problem (need elevated access) and spent 20+ minutes trying to use it instead of discovering the actual solution. Classic game design tool. Terrible CTF design tool.
- **Wrong audience** -- My hope was to appeal to newer players who'd find a game more approachable than a challenge board. But I didn't market it that way, and the beginners I was designing for didn't show up.

{{< shortcode-gallery match="images/red-herring/*" sortOrder="asc" rowHeight="250" margins="5" thumbnailResizeOptions="600x600 q90 Lanczos" thumbnailHoverEffect="none" showExif=false previewType="blur" lastRow="center" embedPreview=true loadJQuery=true >}}

_Red herrings work in adventure games. Not so much in CTFs._

The best CTFs I've played strike a balance where there's both atmosphere and narrative, but the underlying challenges still involve real technologies. Players learn real tools, read real documentation, interact with real systems. The story makes it interesting and the technology makes it educational. Too much narrative and you lose the core audience that plays CTFs to sharpen real skills. Too little and you're back to security homework. My game simulated everything inside a single Python app, and that tipped the scale too far toward game and not far enough toward CTF.

<hr>

# Building It with AI Agents

The other big experiment this year was the development process itself. Last year I used Cursor with Claude 3.5 Sonnet and ChatGPT. It worked, but I was constantly experiencing [context rot](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents), having to re-explain context every time I switched between writing narrative, coding the game engine, or setting up deployment. The AI would lose track of earlier decisions, and I'd waste time getting it back up to speed.

This year I used [Claude Code](https://docs.anthropic.com/en/docs/claude-code) with five specialized [subagents](https://code.claude.com/docs/en/sub-agents), each defined as a markdown file in [`.claude/agents/`](https://github.com/Geo-Bit/2025CTF-ColdSignal/tree/main/.claude/agents) with its own system prompt, tool access, and domain boundaries:

<div style="display:flex; justify-content:center;">
<pre style="background:#1a1a2e; color:#a0a0b8; padding:1.2em 1.6em; border-radius:8px; font-size:0.85em; line-height:1.6; overflow-x:auto;">
╔═══════════════════════════════════════════════════════╗
║  <span style="color:#e0e0ff; font-weight:bold;">CLAUDE CODE</span>                                          ║
╠═══════════════════════════════════════════════════════╣
║                                                       ║
║  <span style="color:#7fdbca;">&gt;</span> agents --status                                    ║
║                                                       ║
║  <span style="color:#addb67;">[●]</span> <span style="color:#c792ea;">ctf-narrative-agent</span> <span style="color:#4a4a6a;">·····</span> story &amp; atmosphere     ║
║  <span style="color:#addb67;">[●]</span> <span style="color:#c792ea;">python-game-dev</span> <span style="color:#4a4a6a;">·········</span> engine &amp; architecture  ║
║  <span style="color:#addb67;">[●]</span> <span style="color:#c792ea;">challenge-designer</span> <span style="color:#4a4a6a;">······</span> vulns &amp; difficulty     ║
║  <span style="color:#addb67;">[●]</span> <span style="color:#c792ea;">tech-stack-architect</span> <span style="color:#4a4a6a;">····</span> docker &amp; security      ║
║  <span style="color:#addb67;">[●]</span> <span style="color:#c792ea;">project-coordinator</span> <span style="color:#4a4a6a;">·····</span> tasks &amp; coordination   ║
║                                                       ║
║  <span style="color:#7fdbca;">&gt;</span> <span style="color:#7fdbca; animation:blink 1s step-end infinite;">_</span>                                                  ║
╚═══════════════════════════════════════════════════════╝
</pre>
</div>

Each agent file is just YAML frontmatter and a system prompt -- a name, description, tool allowlist, and the role definition. I've found using an LLM to help generate agent system prompts on high-level requirements or role-description can really speed things up as well. When Claude Code invokes a subagent, it spins up a fresh context scoped to that domain, so switching from narrative to infrastructure doesn't carry over stale context from the previous task. I also used a project-level [CLAUDE.md](https://docs.anthropic.com/en/docs/claude-code/memory) to give all agents shared context about the game's structure, the flag format, and the file layout -- things every agent needed regardless of domain.

I only have rough dev metrics from last year, but using Claude Code cut development time from roughly 65 hours to about 42. But the bigger win was simply the consistency of everything from dialogue to challenges. The code stayed clean because one agent enforced the architecture. Security boundaries between intentional game vulnerabilities and real flaws stayed clear because the tech-stack-architect's prompt included an explicit security boundary checklist separating educational exploits from actual risk.

The agents weren't fully autopilot, though. I found myself still haven't to actively act as a human in the loop to provide clarification, additional context, or even flat out reject features and enhancements. Fortunately that is pretty well baked into the Claude Code UI and experience.

If I were starting over, I'd probably begin with 2-3 agents and add more only when the scope demanded it -- the setup overhead for five was real. But for any project that crosses multiple domains, I'd use this approach again without hesitation.

<hr>

# Where This Goes

Cold Signal was an experiment with mixed results. The AI development workflow was awesome, but the narrative-heavy CTF format needs still needs work.

But I still think there's something here. CTFs have come a long way since [DEF CON first ran one in 1996](<https://en.wikipedia.org/wiki/Capture*the_flag*(cybersecurity)>). We've gone from attack-defense wargames to jeopardy-style platforms to [education-first formats](https://dl.acm.org/doi/10.1145/3626252.3630912) designed specifically for learners. There's definitely a convergence happening between security education and game design, and I think we're still early in figuring out what it looks like.

AI-assisted development makes the experimentation cheaper, too. Building a text adventure CTF, deploying it via Docker and SSH, and iterating on it over the course of a few weeks would have been totally impractical with my personal life a couple of years ago. The barrier to trying creative approaches to security education is lower than it's ever been. More people should take advantage of that.

I don't want to stop trying to be innovative or creative in this space, though. I think there's a huge opportunity for creatives to get involved in building CTF experiences. It would be great to see more traditional game developers and designers participating in this area, and I feel like it's a space for non-traditional security people to help build cool things.

If you're interested in this space, or want to chat more about my experiencing developing CTFs, please don't hesistate to [reach out](https://www.linkedin.com/in/george-tipton/), I'm always happy to chat!

# Resources

- [Play Cold Signal in your browser](https://georgetipton.com/cold-signal)
- [Cold Signal Web GitHub Repository](https://github.com/Geo-Bit/cold-signal)
- [Cold Signal GitHub Repository](https://github.com/Geo-Bit/2025CTF-ColdSignal)
- [Horsetooth Liquidators (2024 CTF)](/posts/horsetooth-ctf/)
