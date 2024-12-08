# Practical Session #1: Introduction

1. Find in news sources a general public article reporting the discovery of a software bug. 
Describe the bug. If possible, say whether the bug is local or global and describe the failure that 
manifested its presence. Explain the repercussions of the bug for clients/consumers and the company 
or entity behind the faulty program. Speculate whether, in your opinion, testing the right scenario 
would have helped to discover the fault.

2. Apache Commons projects are known for the quality of their code and development practices. They 
use dedicated issue tracking systems to discuss and follow the evolution of bugs and new features. 
The following link https://issues.apache.org/jira/projects/COLLECTIONS/issues/COLLECTIONS-794?filter=doneissues 
points to the issues considered as solved for the Apache Commons Collections project. Among those 
issues find one that corresponds to a bug that has been solved. Classify the bug as local or global. 
Explain the bug and the solution. Did the contributors of the project add new tests to ensure that 
the bug is detected if it reappears in the future?

3. Netflix is famous, among other things we love, for the popularization of *Chaos Engineering*, a 
fault-tolerance verification technique. The company has implemented protocols to test their entire 
system in production by simulating faults such as a server shutdown. During these experiments they 
evaluate the system's capabilities of delivering content under different conditions. The technique 
was described in [a paper](https://arxiv.org/ftp/arxiv/papers/1702/1702.05843.pdf) published in 2016. 
Read the paper and briefly explain what are the concrete experiments they perform, what are the 
requirements for these experiments, what are the variables they observe and what are the main 
results they obtained. Is Netflix the only company performing these experiments? Speculate how these 
experiments could be carried in other organizations in terms of the kind of experiment that could be 
performed and the system variables to observe during the experiments.

4. [WebAssembly](https://webassembly.org/) has become the fourth official language supported by web 
browsers. The language was born from a joint effort of the major players in the Web. Its creators 
presented their design decisions and the formal specification in 
[a scientific paper](https://people.mpi-sws.org/~rossberg/papers/Haas,%20Rossberg,%20Schuff,%20Titzer,%20Gohman,%20Wagner,%20Zakai,%20Bastien,%20Holman%20-%20Bringing%20the%20Web%20up%20to%20Speed%20with%20WebAssembly.pdf) 
published in 2018. The goal of the language is to be a low level, safe and portable compilation 
target for the Web and other embedding environments. The authors say that it is the first industrial 
strength language designed with formal semantics from the start. This evidences the feasibility of 
constructive approaches in this area. Read the paper and explain what are the main advantages of 
having a formal specification for WebAssembly. In your opinion, does this mean that WebAssembly 
implementations should not be tested? 

5.  Shortly after the appearance of WebAssembly another paper proposed a mechanized specification 
of the language using Isabelle. The paper can be consulted here: 
https://www.cl.cam.ac.uk/~caw77/papers/mechanising-and-verifying-the-webassembly-specification.pdf. 
This mechanized specification complements the first formalization attempt from the paper. 
According to the author of this second paper, what are the main advantages of the mechanized 
specification? Did it help improving the original formal specification of the language? 
What other artifacts were derived from this mechanized specification? How did the author verify the 
specification? Does this new specification removes the need for testing?

## Answers

1. In July 2024, a bug in the antivirus CrowdStrike Falcon caused important crashes on various information
    systems running Windows around the world, leading to outages on sometimes critical systems
    (banks, airports, hospitals...). The bug was triggered by an update of the software, containing an invalid
    configuration file. The reading of such file was causing an out of bound memory read ; as the
    software works as a driver for the Windows kernel, it was leading to a kernel crash, causing the
    infamous "Blue Screen of Death" on affected machines, making them unusables. Althrough the bug had
    been quickly identified, CrowdStrike deployed a patch and Microsoft created a tool to repair broken
    machines, the outage lead to a loss of more than 5 billion dollars, as several important companies or
    services were impacted ; it also damaged the reputation of CrowdStrike. As CrowdStrike's tools are 
    closed source, the nature of the software component which malfunctionned is not exactly known.
    This bug can be classified as a local bug, as it is clearly due to an internal malfunctionning of
    the driver ; however, as it occurs in Windows kernel, its consequences are more important, as the
    execution of every application of the user space obviously relies on the kernel.
    However, we can speculate that appropriate programming techniques may had helped to avoid such bug :
    - use of defensive programming pratices (avoid out-of-bound access of memory by, for example, checking 
    the acceded addresses, or indices), if such protection is trivial.
    - unit / integration tests (if it is easy to isolate / make input data for the bugged component);
    althroug it seems there is no widely used testing solution, a proposal can be found [here](https://github.com/wpdk/wdutf).
    - use of static analysis tools : language analysis tools such as compiler's warning, `clang-tidy`, or driver
    analysis tools, such as [SDV](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/static-driver-verifier)... 
    may detect code patterns which can frequently lead to bugs ; they are however limited to only simple patterns, as
    the extraction of semantical properties from a program using static analysis is a notoriously undecidable problem.
    - use of dynamic analysis tools : there is various means to test the code at runtime : of course debuggers,
    but also memory testing tools, which allows to control program's behaviour over memory, such as [KASAN](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/kasan);
    fuzzing tools (which test a program by running it and testing its inputs, 
    [a reasearch example for Windows drivers can be found here](https://research.checkpoint.com/2020/bugs-on-the-windshield-fuzzing-the-windows-kernel/)). Microsoft also proposes a 
    [driver verifier](https://learn.microsoft.com/en-us/windows-hardware/drivers/devtest/driver-verifier).

    More generally, the use of low-level languages, which allows direct, manual memory handling, can easily lead to such memory
    corruption bugs, can also be discussed. As we operate at low level, the use of runtime protections, such as
    garbage collector is not possible ; they can also cause problematic performance overheads. A language such
    [Rust](https://rust-lang.org) address theses issues, by proposing semantic constraints limiting possibilities
    to erroneously manipulate the memory, which avoids the use of runtime checks, preferring compile-time
    checks. Rust programming for Windows is still emerging; [however there is already an official library
    from Microsoft for Windows driver programming in Rust](https://github.com/microsoft/windows-drivers-rs)

    ***Issue related to CrowdStrike's bug:***
    https://www.messageware.com/what-caused-the-crowdstrike-outage-a-detailed-breakdown/

2. We use the issue COLLECTION-814, which can be found at the address :
    https://issues.apache.org/jira/browse/COLLECTIONS-814. This issue is about a discrepency between the
    `CollectionUtils.removeAll` method and its Javadoc. According to the Javadoc, this method was
    expected to throw a `NullPointerException` if at least one of its arguments was `null`. However,
    the reporter noted that this `NullPointerException` was not threw if the first parameter was an empty
    list and the second parameter was `null`. As this bug was only linked to a missing check in the 
    function, and not related to an external interaction, it can be classified as a local bug. It
    had been fixed by the folowing pull request : https://github.com/apache/commons-collections/pull/340,
    by non-null checks on both parameters, by using the `Objects.requireNonNull` method. The corresponding
    test method (`testRemoveAll` in `ListUtilsTest.java`) was updated accordingly to the changes, by
    adding two tests, one for each parameter, using `assertThrow`.

3. 

4. 

5. 