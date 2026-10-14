Pattern Matching WG21 proposal: [https\://wg21.link/p2688r6](https://wg21.link/p2688r6)  
LLVM Code of Conduct: [https\://llvm.org/docs/CodeOfConduct.html](https://llvm.org/docs/CodeOfConduct.html)  
Date: Oct 14, 2026

Slide Deck for the Presentation: https://github.com/llvm/llvm-wgs/blob/main/clang-language-wg/wg21-pattern-matching/2026-10-14 Pattern Matching Slide Deck.html

Attendees:

* Aaron Ballman (chair)  
* Michael Park (presenter)  
* Corentin Jabot  
* Zhihao Yuan  
* Vlad Serebrennikov  
* Bruno Cardoso Lopes  
* Ryszard Goc  
* Muhammad Bassiouni  
* Ambrose Leeb  
* Shafik Yaghmour  
* Feedable

Minutes:

* Michael started with a presentation. Planning to present this paper in Brazil in November, pending EWG scheduling.  
* Michael: looking for feedback both in terms of implementer concerns and delights, but also in terms of usability. Particularly if it’s helpful for our own code base.  
* Michael: will take questions throughout the session, but hold off on them until the tour is complete.  
* Michael: On r6, not going through the change history though.  
* Michael: a match is a new control flow construct; it sits somewhere between \`switch\` and a chain of \`if/else\` statements. It is an expression (and can be used as a statement through the usual means). If the pattern is matched (perhaps with a filter function), then the handler is invoked. Matches can be composed. There is exhaustiveness checking.  
* Michael: can do value matching, such as on enum values. If you remove Coin::Quarter, you will get an error because the match is not exhaustive.  
* Michael: if the pattern is a function designator, it will call the function with an argument from the match operand.  
* Michael: you can introduce names to bind values for the match; no longer using \`let\` syntax, can now use general C++ declaration syntax. There is also a wildcard pattern \`\_\` that always matches and executes the handler.  
* Michael: there is an \`or\` pattern, so you can say \`case 1 or 2\`. There’s also a decomposition pattern that works similar to structured bindings. You can match on value patterns or declaration patterns with it. e.g. \`case \[1, 1\]\` or \`case \[int r1, int r2\]\`.  
* Michael: { P } projects the value state and recursively applies P. \`{}\` matches the empty state. So if you have a std::optional, you can do \`case { int roll }\` which matches if the optional had a value and then does a declaration pattern. Also works with \`std::variant\`, \`std::expected\`, etc and not just \`std::optional\`.  
* Michael: if there are the same types like \`std::expected\<std::string, std::string\>\`, you can use name selectors like \`case {.value: const auto \&contents }\` and \`case {.error : const auto \&message }\`.  
* Michael: it also supports open class hierarchies for polymorphic types (it uses dynamic\_cast, semantically).  
* Michael: and that’s basically it in terms of patterns you can have. There is a way to match on a single case in the pattern and the result of the match is a bool (does it match the case, true or false). Useful for requires clauses, etc.  
* Michael: there is also a pattern condition. You can do \`if (case { thing })\` (same for \`while\`, \`?:\`, etc. You can chain conditions together using \`&&\` (evaluated left to right).  
* Vlad: in the example matching on types, you perform dynamic\_cast. Do you leave the door open for a design which allows people to explain their own RTTI schemes?  
* Michael: yes, we’ll get there. The short answer is: you wouldn’t be able to opt in for the open class hierarchy syntax specifically. That should be only used by polymorphism from the language, it’s more tied to typeid. The alternative pattern can be used instead for custom RTTI.  
* Ambrose: Regarding enums; users can create enum values that don’t correspond to any declared enumerator? How is this handled, particularly compared to the exhaustiveness checking?  
* Michael: for exhaustiveness checking, there are required states and residual states. For enums, the values of the enumerators are required states. All the values outside of that are residual states. So you can put \`case \_\` to handle the residual states, but it’s not required for it to be considered exhaustive. If you cover all the named enumerators and add \`case \_\`, but if someone adds a new enum value to the list, because of the \`case \_\`, it suppresses the diagnostic you’d want. The solution we have right now is that if a pattern has an if guard, that case is not considered for exhaustiveness. So you can do \`case \_ if (true)\`, that allows you to get the exhaustive checking for the new enumerators.  
* Vlad: I expect we will end up with various diagnostic flags (like we do for switch) where we ask for \`default\` even if they cover all the enumerators. I suspect users will have to make the choice because the language cannot pick one behavior that satisfies all users.  
* Feedable: what to do about actual bit-field enums? e.g. 1 \<\< 0, 1 \<\< 1, 1 \<\< 2, etc and you use \`|\` to combine the enumerator values.  
* Ambrose: the language doesn’t have proper support for them.  
* Feedable: yeah, this is about matching on bitwise ORed enumerator values; can you match on those and how does exhaustiveness checking work?  
* Michael: that is something I’d have to think more about, it’s a good use case that we don’t have a solution for yet.  
* Ambrose: Rust solves this with a particular kind of attribute  
* Vlad: we can also leave this to QoI; it may not be hard to recognize those enums  
* Feedable: why a decomposition pattern and an alternative pattern? You can have zero or many in alternative?  
* Michael: alternative is zero or one; I think you do want to distinguish between a one-element struct vs a variant of different options. Technically you could probably get away with square brackets for both, but if you’re reading the patterns, it would be hard to understand.  
* Feedable: for generic code, you basically don’t want that distinction.  
* Michael: I think you do?  
* Feedable: then in generic code, you’d have to say all the various combinations to match, which is problematic.  
* Michael: an example would be handy, if you can send me one  
* Feedable: what happens if you have an auto parameter and you put that auto value in an if statement? e.g., std::variant\<int, float\> a; if(case {auto x} \= a)  x;  
* Michael: here you would get case instantiations. x can be int or float. That if scope would become a template region.  
* Vlad: that sounds like implicit template regions from the structure bindings case  
* Ambrose: that’s how expansion statements work  
* Michael: with pattern matching, there’s a corresponding scope that becomes a template region; so it’s not as bad as with packs in structured bindings where the template region isn’t corresponding to a single scope.  
* Ambrose: the main difference to a lambda is now you have a thing that’s inside of a function when the function itself is not a dependent context. There’s a lot that goes into it; what happens if you put \_\_builtin\_va\_start inside of it, etc? I did most of the work for that with expansion statements, so it’s a little bit easier these days, but still pretty involved. You will run into weird problems though; we’ve not solved them all with expansion statements  
* Aaron: is \`or\` being introduced as a full keyword now? It's an alternate keyword (digraph) that means \`||\` today, so is the or pattern support \`A or B\` as well as \`A || B\`?   
* Michael: it’s the digraph, you can do \`or\` or \`||\`.  
* Ryszard: what is the exhaustiveness we expect with polymorphic matching? If we had Shape, Circle, Square, do we have to have a fallback?  
* Michael: always requires a wildcard pattern.  
* Michael: continuing the presentation. Moved on to custom alternative types. Introduces std::alternative\_traits to add customization points for the {} syntax behavior. This showed a notional lowering of the pattern matching syntax to alternative\_traits calls.  
* Michael: there is also a named selector protocol to handle the \`.value\` syntax where you can name parts of a class type.  
* Michael: moved on to examples from the Clang and LLVM code base where pattern matching could be used to replace existing functionality in the compiler. Showed an example from JSONFormat::writeArtifact, Implementation::parseLanguage (showing string value dispatch as a replacement for StringSwitch), ARM::parseArchISA (showing string predicate dispatch using starts\_with)  
* Michael: can do AST pattern matching by using DeclNodes.td and update tablegen to generate an alternative\_traits for the Decl hierarchy. Then you can use pattern matching using \`case { FunctionDecl \&FD } \=\> use(\&FD) \` syntax to match on AST nodes and dispatch with exhaustiveness checking.  
* Vlad: on the slide about named selector protocol; when I was reading the paper, the paper has an example for this with std::expected. I was confused by the syntax because the syntax is so similar to designated initializers. I had to check to see which public members std::expected has, and those are different. The names after the dots have no correspondence to what’s in std::expected, which is very confusing.  
* Michael: yeah.  
* Michael: the strongest argument I can make is that the braces tell you it has something to do with alterantive\_traits. But I totally hear the concern.  
* Vlad: dot followed by identifier is so ingrained that it refers to class members, that’s going to be hard to break  
* Ambrose: can we get rid of the dot entirely? Do we need it here?  
* Michael: yes, you can omit the \`:P\` so can do \`{empty}\` which looks like value matching within the braces.  
* Feedable: for std::expected, the problem of \<int, int\>, it’s already solved by std::unexpected. That could be used in an alternative pattern and get rid of the dot.  
* Michael: you can do \`.index\<0\>\` for example..  
* (we then got cut off due to meeting time limit)