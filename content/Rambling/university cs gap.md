

# Why a CS Degree Isn't Enough Anymore

> [!abstract] TL;DR
> Lectures, a good GPA, and waiting for a six-figure offer to drop out of the sky is a bet that the job market still looks like 2015. It doesn't. Making the unit tests turn green is a different skill from building software, and the difference is now the whole ballgame.

If your plan is to go to lectures, get a good GPA, and wait for a six-figure offer to drop out of the sky at graduation, you are going to get a reality check.

Universities are still teaching like it's 2015. If you don't lock in right now, you're spending four years and tens of thousands of dollars to graduate with the exact same skill set as a free chatbot. Here's why, and what to actually do about it.

## The biggest lie: the test-passing bubble

Think about how a normal programming assignment works. The professor hands you a pre-built skeleton repo. The architecture is already designed, the build system is already configured, ninety percent of the boilerplate is already written. Your entire job is to fill in a fifty-line function and submit it at 11:58 p.m. until the unit tests turn green.

And the second the semester ends, you throw that code in the trash and never look at it again.

Stanford economist Erik Brynjolfsson pointed at the exact problem: clean, isolated textbook coding problems are exactly the kind of book learning that large language models have completely memorized. If your homework can be graded by a simple input-output script, an AI can already one-shot it. You're being trained for a task that's already been automated by the thing you're worried about.

The homework was never measuring whether you can engineer. It was measuring whether you can make a script exit zero. Not the same job.

## The chalkboard has never run a program

Even worse, a lot of CS is taught in a vacuum that ignores physical hardware. In data structures and algorithms, a linked list of a million items and a flat array of a million items are both O(n). On a chalkboard, they look identical. Beautiful, even.

Try doing that on a real CPU.

<table style="width: 100%; border-collapse: collapse; color: #cdd6f4; margin-bottom: 20px; font-size: 0.95em;">
  <tr style="background-color: #2e1065; color: #ffffff;">
    <th style="border: 1px solid #7aa2f7; padding: 10px; text-align: left;">Traversal</th>
    <th style="border: 1px solid #7aa2f7; padding: 10px; text-align: left;">On paper</th>
    <th style="border: 1px solid #7aa2f7; padding: 10px; text-align: left;">On a CPU</th>
  </tr>
  <tr style="background-color: #2a2a35;">
    <td style="border: 1px solid #7aa2f7; padding: 10px;"><strong>Flat array</strong></td>
    <td style="border: 1px solid #7aa2f7; padding: 10px;"><code>O(n)</code></td>
    <td style="border: 1px solid #7aa2f7; padding: 10px;">Contiguous RAM. One 64-byte cache line pulls the next several items into L1 for free.</td>
  </tr>
  <tr style="background-color: #1e1e2e;">
    <td style="border: 1px solid #7aa2f7; padding: 10px;"><strong>Linked list</strong></td>
    <td style="border: 1px solid #7aa2f7; padding: 10px;"><code>O(n)</code></td>
    <td style="border: 1px solid #7aa2f7; padding: 10px;">Nodes scattered across the heap. Every hop follows a pointer and stalls for hundreds of clock cycles.</td>
  </tr>
</table>

On paper, same time complexity. In production, one is twenty times slower than the other. The chalkboard is not aware of this, because the chalkboard has never executed a single instruction.

> [!warning]
> The abstraction gets presented as the truth and the hardware gets treated as a detail. Without a mental model of cache lines and memory hierarchy, you don't know why your code is slow. You just know it is, and you're guessing.

## Nobody teaches you how to use a computer

The second problem is that universities teach theoretical computer science but almost never teach you how to actually operate a computer. Even MIT admitted this. Three instructors noticed that students who could prove compiler theorems and build neural networks on a whiteboard had almost zero proficiency with the actual tools of software engineering.

Students holding the arrow key down for thirty seconds to scroll to the bottom of a file. Manually renaming variables across twenty files by hand. Fixing a Git merge conflict by copying their code to the desktop, deleting the entire repository, and cloning it all over again.

MIT had to create a class called [[The Missing Semester]] of Your CS Education to teach students how to use a computer. At MIT. For people who can prove compiler theorems.

Nobody sits you down and teaches you how to pipe data through the shell, attach an interactive debugger, read a stack frame, or read a flame graph to find the bottleneck. There's no class on untangling a Git merge conflict, resolving a cascading deadlock in production, writing a clean Dockerfile, or navigating a million-line legacy codebase with no documentation.

And Git is not a mysterious cloud backup button. It's a content-addressed directed acyclic graph where every commit is an immutable snapshot pointing at a tree of hashed blobs. Once you understand the graph, you can rebase, bisect a regression across five hundred commits in thirty seconds, and cherry-pick without panicking. Until then, you're building software with one hand tied behind your back.

## The clone resume

Once students realize their classes aren't enough, they usually fall into the next trap. Open a hundred new-grad resumes right now and ninety of them have the same three projects: a full-stack to-do app, a React movie dashboard, and a wrapper around the OpenAI API with a Tailwind landing page.

Hiring managers skip right past these. When you build a basic web wrapper, ninety percent of the actual computer science is hidden inside third-party packages. Calling an API and rendering a JSON array doesn't prove you can engineer systems. It proves you can read a quickstart guide.

If you want your resume to stop a senior engineer in their tracks, delete the five tutorial clones and build one infrastructure tool from scratch. A multi-threaded HTTP server on a raw TCP socket in C, Rust, or Go. Parse the byte stream yourself, handle keep-alive, implement gzip from the spec. Or a toy in-memory Redis that parses the wire protocol over a non-blocking event loop and handles concurrent clients and key expiration. Or a Unix shell that parses command tokens, forks children, and wires up file descriptors for input and output redirection.

Do that and you're not competing with people who prompt-engineered a weekend web app anymore. You walk into interviews able to talk about socket buffers, race conditions, and memory allocation like someone who's already been doing the job for years.

## The teaching problem

Your professors are not hired to teach you. They're hired to bring in grants, publish papers, and not embarrass the department. Teaching is the thing they do between the actual job and the committee meetings, and a world-class researcher can be a genuinely terrible teacher with zero consequences to their career. Nobody trained them. There's no pedagogy class in a CS PhD. There's no feedback loop on whether their lecture actually transferred knowledge. The lecture is a performance, and the performance is optimized for the professor's convenience, not your understanding. If the room is confused, the room is wrong.

So think about where you actually learned anything. Not in the lecture hall. At 2 a.m., with a textbook open, a video at 1.75x, and a program that doesn't work, changing one line at a time until it does. That's not a failure of the system. That's the system. The lecture was never where the knowledge entered your brain.

Which means college is mostly regulated self-learning. The regulation is real, and it does real work. A syllabus, a deadline, a proctor, a grade, and a peer group that's also panicking at 2 a.m. It forces you to actually sit down and do the thing instead of intending to. But the learning is something you do alone, in a chair, with the material, and nobody can do it for you. You are paying tens of thousands of dollars for a schedule and a piece of paper, and the actual education is outsourced to you.

So do more of it on purpose. Pick a problem nobody assigned you. Read a paper you don't understand and chase the references until you do. Reproduce a result. Build something nobody grades. Break a program and figure out why. That's independent research, and it doesn't come with a rubric. It comes with a question you can't stop thinking about. The students who figure this out spend four years running their own experiments and shipping real things. The students who don't spend four years waiting for a lecture to teach them something, and then graduation feels like a cliff.

## So what do you actually do

Stop optimizing your schedule for an easy 4.0. Nobody at a tech company is going to ask what you got in your easy elective. Take the brutal upper-level systems courses everyone else avoids: operating systems, compilers, computer networks, database internals. The ones where you lose sleep writing a virtual memory page table manager, an instruction parser, or a B+ tree storage engine. Those build the mental model an AI can't replace.

Then break out of the classroom early. One summer internship, one merged pull request to a real open source systems project, or one piece of software that actual students on your campus use is worth more than fifty isolated homework assignments. Real engineering only starts when requirements change, users send malformed data, someone tries to DDoS your service, and you have to maintain code you wrote six months ago and no longer recognize.

The degree keeps getting framed as the achievement. It isn't. It's the access. The GPA, the tests turning green, the piece of paper, none of it is the muscle.

A CS degree is not a golden ticket anymore. It's a gym membership. Pay tuition, copy-paste your way through the tests, abuse ChatGPT for four years, and you graduate with a piece of paper and zero leverage. But if you use those four years to learn your tools, take the hardest systems classes in the catalog, get internships at top companies, and build real infrastructure from the ground up, there has never been an easier time to lap the entire field.

