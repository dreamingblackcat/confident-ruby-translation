>Ruby is designed to make programmers happy.
>– Yukhiro "Matz" Matsumoto
This is a book about Ruby, and about joy.
If you are anything like me, the day you first discovered the expressive power of
Ruby was a very happy day. For me, I think it was probably a code example like this
which made it "click":
```ruby
3.times do
puts "Hello, Ruby world!"
end
```
To this day, that's still the most succinct, straightforward way of saying "do this
three times" I've seen in any programming language. Not to mention that after using several supposedly object-oriented languages, this was the first one I'd seen
where everything, including the number "3", really was an object. Bliss!

###Ruby meets the real world
Programming in Ruby was something very close to a realization of the old dream of
programming in pseudocode. Programming in short, clear, intent-revealing stanzas.
No tedious boilerplate; no cluttered thickets of syntax. The logic I saw in my head,
transferred into program logic with a minimum of interference.
But my programs grew, and as they grew, they started to change. The real world
poked its ugly head in, in the form of failure modes and edge cases. Little by little,
my code began to lose its beauty. Sections became overgrown with complicated
nested if/then/else logic and && conditionals. Objects stopped feeling like
entities accepting messages, and started to feel more like big bags of attributes.
begin/rescue/end blocks started sprouting up willy-nilly, complicating once
obvious logic with necessary failure-handling. My tests, too, became more and more
convoluted.
I wasn't as happy as I once had been.

###Confident code
If you've written applications of substantial size in Ruby, you've probably
experienced this progression from idealistic beginnings to somewhat less satisfying
daily reality. You've noticed a steady decline in how much fun a project is the larger
and older it becomes. You may have even come to accept it as the inevitable
trajectory of any software project.
In the following pages I introduce an approach to writing Ruby code which, when
practiced dilligently, can help reverse this downward spiral. It is not a brand new set of practices. Rather, it is a collection of time-tested techniques and patterns, tied
together by a common theme: self confidence.
This book's focus is on where the rubber meets the road in object-oriented
programming: the individual method. I'll seek to equip you with tools to write
methods that tell compelling stories, without getting lost down special-case rabbit
holes or hung up on tedious type-checking. Our quest to write better methods will
sometimes lead us to make improvements to our overall object design. But we'll
continually return to the principal task at hand: writing clear, uncluttered methods.
So what, exactly do I mean when I say that our goal is to write methods which tell a
story? Well, let me start with an example of a story that isn't told very well.

###A good story, poorly told
Have you ever read one of those "choose your own adventure" books? Every page
would end with a question like this:
>If you fight the angry troll with your bare hands, turn to page 137.
>If you try to reason with the troll, turn to page 29.
>If you don your invisibility cloak, turn to page 6.
You'd pick one option, turn to the indicated page, and the story would continue.
Did you ever try to read one of those books from front to back? It's a surreal
experience. The story jumps forward and back in time. Characters appear out of
nowhere. One page you're crushed by the fist of an angry troll, and on the next
you're just entering the troll's realm for the first time.
What if each individual page was this kind of mish-mash? What if every page read
like this:
> You exit the passageway into a large cavern. Unless you came from page 59,
in which case you fall down the sinkhole into a large cavern. A huge troll, or
possibly a badger (if you already visited Queen Pelican), blocks your path.
Unless you threw a button down the wishing well on page 8, in which case
there nothing blocking your way. The [troll or badger or nothing at all] does
not look happy to see you.
If you came here from chapter 7 (the Pool of Time), go back to the top of
the page and read it again, only imagine you are watching the events
happen to someone else.
If you already received the invisibility cloak from the aged lighthouse-
keeper, and you want to use it now, go to page 67. Otherwise, forget you
read anything about an invisibility cloak.
If you are facing a troll (see above), and you choose to run away, turn to
page 84.
If you are facing a badger (see above), and you choose to run away, turn to
page 93...
Not the most compelling narrative, is it? The story asks you to carry so much mental
baggage for it that just getting through a page is exhausting.

###Code as narrative
What does this have to do with software? Well, code can tell a story as well. It might
not be a tale of high adventure and intrigue. But it's a story nonetheless; one about a problem that needed to be solved, and the path the developer(s) chose to
accomplish that task.
A single method is like a page in that story. And unfortunately, a lot of methods are
just as convoluted, equivocal, and confusing as that made-up page above.
In this book, we'll take a look at many examples of the kind of code that
unnecessarily obscures the storyline of a method. We'll also explore a number of
techniques for minimizing distractions and writing methods that straightforwardly
convey their intent.

###The four parts of a method
I believe that if we take a look at any given line of code in a method, we can nearly
always categorize it as serving one of the following roles:
1. Collecting input
2. Performing work
3. Delivering output
4. Handling failures
(There are two other categories that sometimes appear: "diagnostics", and "cleanup".
But these are less common.)
Let's test this assertion. Here's a method taken from the MetricFu project.

```ruby
def location(item, value)
sub_table = get_sub_table(item, value)
if(sub_table.length==0)
raise MetricFu::AnalysisError, "The #{item.to_s}
'#{value.to_s}' "\
"does not have any rows in the analysis table"
else
first_row = sub_table[0]
case item
when :class
MetricFu::Location.get(first_row.file_path,
first_row.class_name, nil)
when :method
MetricFu::Location.get(first_row.file_path,
first_row.class_name, first_row.method_name)
when :file
MetricFu::Location.get(first_row.file_path, nil, nil)
else
raise ArgumentError, "Item must be :class, :method, or :file"
end
end
end
```
Don't worry too much right now about what this method is supposed to do. Instead,
let's see if we can break the method down according to our four categories.
First, it gathers some input:
```ruby
sub_table = get_sub_table(item, value)
```
Immediately, there is a digression to deal with an error case, when sub_table has
no data.
```ruby
if(sub_table.length==0)
raise MetricFu::AnalysisError, "The #{item.to_s} '#{value.to_s}'
"\
"does not have any rows in the analysis table"
```
Then it returns briefly to input gathering:
```ruby
else
first_row = sub_table[0]
```
Before launching into the "meat" of the method.
```ruby
when :class
MetricFu::Location.get(first_row.file_path, first_row.class_name,
nil)
when :method
MetricFu::Location.get(first_row.file_path, first_row.class_name,
first_row.method_name)
when :file
MetricFu::Location.get(first_row.file_path, nil, nil)
```
The method ends with code dedicated to another failure mode:
```ruby
else
raise ArgumentError, "Item must be :class, :method, or :file"
end
end
```
Let's represent this breakdown visually, using different colors to represent the
different parts of a method.
<img> Here comes figure 1</img>
This method has no lines dedicated to delivering output, so we haven't included
that in the markup. Also, note that we mark up the top-level else...end
delimiters as "handling failure". This is because they wouldn't exist without the
preceding if block, which detects and deals with a failure case.
The point I want to draw your attention to in breaking down the method in this way
is that the different parts of the method are mixed up. Some input is collected; then
some error handling; then some more input collection; then work is done; and so
on.
This is a defining characteristic of "un-confident", or as I think of it, "timid code":
the haphazard mixing of the parts of a method. Just like the confused adventure
story earlier, code like this puts an extra cognitive burden on the reader as they
unconsciously try to keep up with it. And because its responsibilities are
disorganized, this kind of code is often difficult to refactor and rearrange without
breaking it.
In my experience (and opinion), methods that tell a good story lump these four
parts of a method together in distinct "stripes", rather than mixing them will-nilly.
But not only that, they do it in the order I listed above: First, collect input. Then
perform work. Then deliver output. Finally, handle failure, if necessary.
(By the way, we'll revisit this method again in the last section of the book, and
refactor it to tell a better story.)

###How this book is structured
This book is a patterns catalog at heart. The patterns here deal with what Steve
McConnell calls "code construction" in his book Code Complete. They are
"implementation patterns", to use Kent Beck's terminology. That means that unlike
the patterns in books like Design Patterns or Patterns of Enterprise Application
Architecture, most of these patterns are "little". They are not architectural. They
deal primarily with how to structure individual methods. Some of them may seem
more like idioms or stylistic guidelines than heavyweight patterns.
The material I present here is intended to help you write straightforward methods
that follow this four-part narrative flow. I've broken it down into six parts:
• First, a discussion of writing methods in terms of messages and roles.
• Next, a chapter on "Performing Work". While it may seem out of order based
on the "parts of a method" I laid out above, this chapter will equip you with a
way of thinking through the design of your methods which will set the stage
for the patterns to come.
After that comes the real "meat" of the book, the patterns. Each pattern is written in
five parts:
1. A concise statement of the indications for a pattern. Like the indications
label on a bottle of medicine, this section is a quick hint about the situation
the pattern applies to. Sometimes the indications may be a particular
problem or code smell the pattern can be used to address. In other cases the
indications may simply be a statement of the style of code the pattern can
help you achieve.
2. A synopsis that briefly sums up the pattern. This part is intended to be most
helpful when you are trying to remember a particular pattern, but can't
recall its name.
3. A rationale explaining why you might want to use the pattern.
4. A worked example which uses a concrete scenario (or two) to explain the
motivation for the pattern and demonstrates how to implement it.
5. A conclusion, summing up the example and reflecting on the value (and
sometimes, the potential pitfalls and drawbacks) of the pattern.
The patterns are divided into three sections, based on the part of a method they
apply to:
• A section on patterns for collecting input.
• A set of patterns for delivering results such that code calling your methods
can also be confident.
• Finally, a few techniques for dealing with failures without obfuscating your
methods.
• After the patterns, there is another chapter, "Refactoring for Confidence",
which contains some longer examples of applying the patterns from this
book to some Open-Source Ruby projects.
```ruby
3.times { rejoice! }
```
I could tell you that writing code in a confident style will reduce the number of bugs
in the code. I could tell you that it will make the code easier to understand and
maintain. I could tell that it will make the code more flexible in the face of changing
requirements.
I could tell you all of those things, and they would be true. But that's not why I think
you should read on. My biggest goal in this book is to help bring back the joy that
you felt when you first learned to write Ruby. I want to help you write code that
makes you grin. I want you to come away with habits which will enable you to write
large-scale, real-world-ready code with the same satisfying crispness as the first few
Ruby examples you learned.
Sound good? Let's get started.
