---
layout: post
title: Rust Traits are not interfaces (and a little on Lifetimes)
date: "2023-01-23"
categories:
  - rust
---

I have been learning rust lately. I eventually came to some clarity (hopefully) after reading lots of different sources so I've compiled there here with some commentary.  

> Note: I am still learning so please let me know if I've got something not quite right.  This is my current understanding and your mileage may vary.

## Using Traits for dynamic implementations

My first approach to using traits was to use them exactly like I had with interfaces from other languages.  This led me to using things like `Box` and `dyn` to create [trait objects](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#defining-a-trait-for-common-behavior).  

> We create a trait object by specifying some sort of pointer, such as a & reference or a Box<T> smart pointer, then the dyn keyword, and then specifying the relevant trait. (We’ll talk about the reason trait objects must use a pointer in Chapter 19 in the section “Dynamically Sized Types and the Sized Trait.”) ^[1](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#defining-a-trait-for-common-behavior)

When implementing it I found I was forced to use the Box<dyn Close> and pass the types around since their `Size` was unknown:

```rust

trait Listener {
    fn Accept(&mut self) -> Result<Box<dyn Connection>>, io::Error>;
}

impl Listener for PipeServer {
    fn Accept(&mut self) -> Result<Box<dyn Connection>>, io::Error>;
}

fn start_handler(conn: Box<dyn Connection>){
  ...snip...
}
```

This has it's place in implementations where you don't know the types ahead of time, such as libraries where you don't know the types that will be used.  There isn't anything particularly wrong with this but there can be a slight performance hit due to using [dynamic dispatch instead of static dispatch](https://doc.rust-lang.org/book/ch17-02-trait-objects.html#trait-objects-perform-dynamic-dispatch):

> monomorphization process performed by the compiler when we use trait bounds on generics: the compiler generates non-generic implementations of functions and methods for each concrete type that we use in place of a generic type parameter. The code that results from monomorphization is doing static dispatch, which is when the compiler knows what method you’re calling at compile time. This is opposed to dynamic dispatch, which is when the compiler can’t tell at compile time which method you’re calling. In dynamic dispatch cases, the compiler emits code that at runtime will figure out which method to call. ^[1]https://doc.rust-lang.org/book/ch17-02-trait-objects.html#trait-objects-perform-dynamic-dispatch

### Monomorphization approach with Traits

To get around this and avoid the additional runtime overhead, I added an [Associated type](https://doc.rust-lang.org/book/ch19-03-advanced-traits.html?highlight=Associated%20types#specifying-placeholder-types-in-trait-definitions-with-associated-types) and then specified the type on the implementation which allows the compiler to need the box.  I could also use `impl Trait` in the parameter position instead of needing the `Box<dyn >` type:

```rust
trait Connection {
    fn close(&mut self) -> io::Result<()>;
}

trait Listener {
    type Type: Connection; // specify trait restriction
    fn Accept(&mut self) -> Result<Self::Type, io::Error>;
}

impl Listener for PipeServer {
    type Type = PipeInstance;  // this is concrete type that implements trait Connection
    fn Accept(&mut self) -> Result<Self::Type, io::Error> {
      ...snip...
    }

fn start_handler(con: impl Connection) {
  ...snip...
}
```

Note that this is using the `impl Trait` in the parameter location which is [syntactic sugar for Trait Bound Syntax](https://doc.rust-lang.org/book/ch10-02-traits.html#traits-as-parameters).  And so the `start_handler` could have been writing like:

```rust
fn start_handler<T: Connection>(con: T) {
  ...snip...
}
```

Although they are similar there are two slight differences:

- If you have two (or more) parameters then when using `Trait Bound` Syntax the two parameters must be the same. With `impl Trait` they could be different types
- With `Trait Bound` syntax you can use the [turbofish](https://rust-lang.github.io/impl-trait-initiative/explainer/apit_turbofish.html) syntax (`start_handler::<PipeInstance>`) and with `impl Trait` you cannot use that syntax.

## Lifetimes with Traits

Now that I removed the overhead of using Trait objects I ran into an issue with Lifetimes I didn't have when I wasn't using traits since  `start_handler` actually spins off a thread:

```rust
fn start_handler(con: impl Connection) -> thread::JoinHandle<()>
 {
    let newconnection = con;
    
    let h = thread::spawn(move || {
      ...snip...
    });
    h
  }

  !!! doesn't compile:

  error[E0310]: the parameter type `impl Connection` may not live long enough
  help: consider adding an explicit lifetime bound...
   |
41 | fn start_handler(con: impl Connection + 'static) -> thread::JoinHandle<()>
```

The above using `impl Trait` doesn't compile but interesting if I use the concrete type it does(!?!):

```rust
fn start_handler(con: PipeInstance) -> thread::JoinHandle<()>
 {
    let newconnection = con;
    
    let h = thread::spawn(move || {
      ...snip...
    });
    h
  }
```

So what is going on here and why is `'static` required?  Specifically is using `'static` bad?  As a newbie I really thought it was something to avoid but found a few articles that helped me understand this.  

### T vs &T

I found two existing questions that were similar to my situation ([1](https://users.rust-lang.org/t/lifetime-when-moving-ownership-to-thread/24591), [2](https://users.rust-lang.org/t/why-does-thread-spawn-need-static-lifetime-for-generic-bounds/4541)) in the Rust help that eventually led to more understanding.  It took me some time to parse the answers so let's break it down:

From the [first answer](https://users.rust-lang.org/t/lifetime-when-moving-ownership-to-thread/24591/4) we learn that since the complier doesn't know what `Connection` is, it treats it as type `T`. **Type `T` is encompasses borrowed `&T`**.   This means that we could possibly ["smuggle a borrowed item to the thread"](https://users.rust-lang.org/t/lifetime-when-moving-ownership-to-thread/24591/2).  

The second answer gives us an example of the difference between `T` and borrowed `&T` when running on a thread (try it out on the [rust playground](https://play.rust-lang.org/?version=stable&mode=debug&edition=2021&gist=949148884adb8d20bf5c01a2c5c4dd44))

```rust
fn test<T: Send + Sync + std::fmt::Display>(val: T) {
    thread::spawn(move || println!("{}", val));
}

fn evil() {
    let x = 10;
    return test(&x);
}

!!! doesn't compile:

error[E0597]: `x` does not live long enough
17 |     return test(&x);
   |            -----^^-
   |            |    |
   |            |    borrowed value does not live long enough
   |            argument requires that `x` is borrowed for `'static`
18 | }
   |  - `x` dropped here while still borrowed
```

This makes sense since it would create a dangling pointer to the `x` when `x` goes out of scope on function `evil`.  So the Compiler doesn't allow us to do this.  

So if `T` is encompasses `&T` that means we could pass in a reference that would go out of scope.  So we need the `'static'` to ensure that the variable lives at least as long as the program. This makes sense intuitively once you know the rule.  On top of that we also understand that if the struct that `T` represents contains a borrowed pointer this would also be a problem.

But then comes the question, isn't `'static` something that should be avoided?

## Using 'Static

As a beginner, I thought using static would be wrong but lets look into this a bit more.  This is pretty common mis-conception as pointed out by the [link provided in the second answer](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#2-if-t-static-then-t-must-be-valid-for-the-entire-program):

> They get told that "str literal" is hardcoded into the compiled binary and is loaded into read-only memory at run-time so it's immutable and valid for the entire program and that's what makes it 'static. These concepts are further reinforced by the rules surrounding defining static variables using the static keyword

If we continue reading the post a bit more we learn a type can also be *bounded by* a `'static` lifetime which is different from a the case where a variable is compiled into the binary. (I highly recommend reading the post for other insights beyond this).

> a type with a 'static lifetime is different from a type bounded by a 'static lifetime. The latter can be dynamically allocated at run-time, can be safely and freely mutated, can be dropped, and can live for arbitrary durations.

This means static isn't all that bad; It is ensuring that we don't have any borrowed items when we are using the object, because if we did the thread could point at something that goes out of scope (creating a dangling pointer).

Ok so that means we can add `'static` and it will compile:

```rust
fn start_handler(con: impl Connection + 'static) -> thread::JoinHandle<()>
 {
    let newconnection = con;
    
    let h = thread::spawn(move || {
      ...snip...
    });
    h
  }
```

but why doesn't the example where `PipeInstance` is passed need the static declaration?

This is because it is an Owned Type and doesn't contain Borrowed types! This means that the lifetime is in the current Scope.  Because it is a concrete type, the compiler can find the type and confirm this that it does not contain any borrowed types. Once it is switched to a `impl Trait` it can't be confirmed by the compiler, so I needed to make it explicit. 

As an example, If I add a Borrowed type on the struct of the `PipeInstance`, I would need to annotate the type of Lifetimes. In order to use it in the `start_handler` function I would then need to ensure that the lifetime of the borrowed item was at least `'static`!

This is very nicely described in the [Misconception #6](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#6-boxed-trait-objects-dont-have-lifetimes).

Another point for `'static` lifetimes not being terrible in this scenario is the definition of [`spawn` function](https://doc.rust-lang.org/std/thread/fn.spawn.html#:~:text=The%20%27static%20constraint%20means%20that%20the%20closure%20and,outlive%20the%20lifetime%20they%20have%20been%20created%20in.) itself:

```rust
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
where
    F: FnOnce() -> T,
    F: Send + 'static,
    T: Send + 'static,
```

> The 'static constraint means that the closure and its return value must have a lifetime of the whole program execution. The reason for this is that threads can outlive the lifetime they have been created in.
>
>Indeed if the thread, and by extension its return value, can outlive their caller, we need to make sure that they will be valid afterwards, and since we can’t know when it will return we need to have them valid as long as possible, that is until the end of the program, hence the 'static lifetime.

So there we have it, `'static` lifetimes aren't something that we need to be afraid of.

### Elision Hints

So all this lifetime stuff can be a bit confusing, especially due the fact that many [Lifetimes can be Elided](https://doc.rust-lang.org/nomicon/lifetime-elision.html) so we don't seem them when we are doing basic work.

One thing I've done during this learning period was to turn on [`rust-analyzer.inlayHints.lifetimeElisionHints.enable`](https://rust-analyzer.github.io/manual.html#vs-code-2) in [VSCode Extension](https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer). At least initially these helps me see the various lifetimes that my program. I might turn it off in the future but for now it helps me understand the implicit lifetimes.

One thing to note, which was initially confusing for me, is that a signature like `fn start_handler(con: PipeInstance) -> thread::JoinHandle<()>` won't have a Lifetime!  This was initially confusing because I expected to see `'static` but now see that it is an Owned type (with no borrowed types in the struct) its life time is the current scope and so the Lifetime annotations aren't required to track it.

## Conclusion

Well hope that helps someone (most likely myself when I forget all this in 6 months). This is my current understand as of the writing of this post, so something might change or be incorrect. Look forward to learning and sharpening this understanding, so leave a comment if something seems off.
