<!-- [[TOC]] -->

## Motivation and Problems

For the "Love of the Game", the game being teaching myself enough Rust to have trenchant opinions on other languages. Things I'd like it to do:

- **Markdown**  Rendering.
  There's CommonMark and GFM. I have special markup that will require me to extend these. Not sure if I'll need _both_ specs.
- **TSX** for templating.
  Because I argue it's the best templating language ever.
- **SASS _and_ Tailwind** Rendering.
  Use `grass` for the former.
- **Live-Reloading**
  For posts and all assets.
- **Asset Management**
  Thumbnails, optimized WebP generation, folder organization.

## Development Log (of sorts)

Ah the famous learning curve. But you can be very productive with Rust if you know what to focus on. For baby Rustaceans like me who're prototyping, the big things are:

- Not fighting the famed Borrow Checker
- Not using lifetimes as much as possible
- Just waving away any possible errors for now
- Knowing about and using popular libraries

That's it. Concretely,

- `clone` and `own` everything to avoid fighting the borrow checker and dealing with explicit lifetimes.
- `unwrap` or `expect` to avoid handling the failure cases inside `Result` and `Option`.

I'll add a few more:

- Use `String` instead of `&str` (avoids borrowing/owning)
- Use `Vec<T>` to get owned collections (avoids borrowing/owning)
- Use `Arc<T>` to share `T` across threads (avoids lifetimes)

So yeah, you're side-stepping what makes Rust 'great' in the first place. But going back and dealing with these things is for the next iteration of your app. Super-easy to do so: `grep` for these things and work at them.

### Other Prototyping Shortcuts

- Use `dbg!` a lot.
- TODO: Complete this.

### Datetime

Rust's STDLIB is as shit as Python's with datetime manipulation (which is rather disappointing). Here's things it can and cannot do:

| Action                   | Can it? |
| ------------------------ | ------- |
| Current timestamp        | Yes     |
| Measure elapsed time     | Yes     |
| Add seconds to timestamp | Yes     |
| Get year/month/day       | No      |
| Format as 2026-10-09     | No      |
| Parse ISO 8601 dates     | No      |
| Time zones               | No      |
| Calendar arithmetic      | No      |

Like Python you are expected to use richer libraries: `chrono` or `jiff` (I like the latter.)

## References

- [Great explainer on Async Rust](https://www.youtube.com/watch?v=wXtngLBkK4Q)
