---
{"dg-publish":true,"permalink":"/pl/null-references-the-billion-dollar-mistake/","noteIcon":"","created":"2024-07-19T21:29:12.430+08:00","updated":"2024-07-19T21:30:12.858+08:00"}
---

#PL 
In his 2009 presentation “Null References: The Billion Dollar Mistake,” Tony Hoare, the inventor of null, has this to say:

> I call it my billion-dollar mistake. At that time, I was designing the first comprehensive type system for references in an object-oriented language. My goal was to ensure that all use of references should be absolutely safe, with checking performed automatically by the compiler. But I couldn’t resist the temptation to put in a null reference, simply because it was so easy to implement. This has led to innumerable errors, vulnerabilities, and system crashes, which have probably caused a billion dollars of pain and damage in the last forty years.

### Key Takeaways
- Null references have historically been a bad idea
- Early compilers provided opt-out switches for run-time checks, at the expense of correctness
- Programming language designers should be responsible for the errors in programs written in that language
- Customer requests and markets may not ask for what's good for them; they may need regulation to build the market
- If the billion dollar mistake was the null pointer, the C gets function is a multi-billion dollar mistake that created the opportunity for malware and viruses to thrive

### Reference
- [Presentation](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare/)