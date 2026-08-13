+++
date = '2026-08-26T19:27:49+01:00'
draft = false
title = 'How I'm learning Rust with UI programming'
+++

Rust has quickly become a major player in the low-level programming sector, adapting to use cases once thought impossible. The trend across programming languages is clear: they should limit the programmer's ability to make mistakes and enforce proper programming principles. This is especially visible in business, companies prefer C# and Java over C and C++ because they're strongly typed and forgiving of human error. Yet this gap in the low-level sector went unfilled until Rust.

C++ requires significant self-discipline and rigorous practices to remain maintainable. Businesses favour C# and Java because most programmers are mediocre, not exceptional. But that's okay, successful businesses aren't built by great developers, but by making the most of ordinary ones.

# Kindle's Extracts

Kindle stores its highlights and notes in a simple `My Clippings.txt` file on the device. This plain text format makes it perfect for learning Rust—a straightforward string parsing problem that teaches fundamental language concepts without unnecessary complexity.

# UI in Rust

Rust's UI libraries are still relatively immature, but this is actually an advantage. They don't carry the baggage of GTK's or Qt's aged architecture, nor React's complexity. They can be engineered from the ground up to be maintainable.

## Iced-rs

Iced-rs is built on Elm's architecture, a functional language with Haskell-like principles. It uses a **declarative** approach to UI programming, which I've found to be the most logical paradigm. While the syntax may seem unfamiliar at first, Iced-rs code becomes the easiest to read out of all UI libraries once you understand the core concepts.

Iced-rs is built on **four core principles**:

1. **The Model** — An immutable data structure holding your application's state.
2. **Messages** — Events (user clicks, file selections, timers) encoded as enum variants.
3. **The Update Function** — Pure logic that takes the current model and a message, returning a new model and optional async tasks.
4. **The View Function** — A pure function that renders the UI from the current model. Declarative, not imperative.

Here's what a minimal Iced-rs app looks like:

```rust
#[derive(Default)]
struct Model {
    count: i32,
}

#[derive(Clone)]
enum Message {
    Increment,
    Decrement,
}

fn update(model: &mut Model, message: Message) -> Task<Message> {
    match message {
        Message::Increment => {
            model.count += 1;
            Task::none()
        }
        Message::Decrement => {
            model.count -= 1;
            Task::none()
        }
    }
}

fn view(model: &Model) -> Element<Message> {
    column![
        button("Increment").on_press(Message::Increment),
        text(model.count.to_string()),
        button("Decrement").on_press(Message::Decrement),
    ].into()
}

fn main() -> iced::Result {
    iced::application(Model::default, update, view)
        .title("Counter")
        .run()
}
```

This declarative pattern scales beautifully. The key insight: **you never mutate the UI directly**. Instead, you describe what the UI should look like based on your current state.

# Building Our App

These principles are sufficient to build a working application. We'll also integrate the `rfd` crate, which provides native file picker dialogs. You can find the full source code on [GitHub](https://github.com/K0Stek122/kindle-extractor-rs).

## The View Layer

Here's a key excerpt from our `view()` function that renders the main UI:

```rust
fn view(model: &Frozen) -> Element<'_, Message> {
    let content: Element<'_, Message> = if !model.kindle_data.is_empty() {
        column![
            row![ // TOP MENU
                button("Delete Entry")
                    .style(button::danger)
                    .on_press(Message::DeleteCurrentEntry),
                button("Export All")
                    .style(styles::button_style)
                    .on_press(Message::ExportAll),
                // ... more buttons
            ]
            .spacing(5)
            .height(Length::FillPortion(5))
            .align_y(Alignment::Center),
            row![ // MAIN CONTENT
                container(scrollable(
                    Column::from_vec(
                        model.kindle_data
                            .iter()
                            .map(|book_data|
                                button(book_data.name.as_str())
                                    .width(Length::Fill)
                                    .on_press(Message::BookSelected(book_data.id))
                                    .into()  // <-- Converts Button to Element
                            )
                            .collect()
                    )
                ))
                // ... right pane with quotes
            ]
        ].into()
    } else {
        center(column![
            button("Please pick 'My Clippings.txt'")
                .on_press(Message::OpenClippings),
        ]).into()
    };

    toast::view(content, &model.toasts)
}
```

### Understanding the Widgets

- **`column![]` and `row![]` macros** — These are layout containers. `column!` stacks widgets vertically; `row!` stacks them horizontally. They're macros because they accept variable numbers of children with a convenient syntax. Normal widgets do not allow that in Iced-rs.
    
- **`button()`** - Creates a clickable button. The `.on_press(Message::X)` method attaches an event handler that emits a message when clicked.
    
- **`scrollable()`** - Wraps content to make it scrollable when it overflows.
    
- **`container()`** - A generic layout wrapper for padding, styling, and sizing.
    
- **`.into()`** - This is Rust's type coercion operator. Iced's `Element<Message>` is a type-erased widget container. Since different widgets (buttons, text, columns) have different types, we call `.into()` to convert them into the common `Element` type. Without it, the compiler wouldn't know how to mix different widget types in a collection.

### State Management & Message Flow

Notice the cycle:

1. **User clicks a button** → emits `Message::BookSelected(id)`
2. **`update()` receives the message** → updates `model.current_displayed_book`
3. **`view()` re-renders** → reads the new `model.current_displayed_book` and displays its quotes
4. **UI reflects the change** → the right pane now shows the selected book's quotes

This unidirectional data flow makes reasoning about state changes trivial. There's no hidden state or side effects—everything flows through messages.

# The Parser

The UI is just the shell, the real logic lives in the parser. This is where we transform the raw `My Clippings.txt` file into structured data.

```rust
pub fn parse_clippings(file_path: &str, starting_id: usize) -> io::Result<Vec<BookData>> {
    let extracts = retrieve_clippings_from_file(file_path)?;

    let mut books: Vec<BookData> = Vec::new();
    for extract in extracts {
        let (name, author) = get_book_name_and_author(&extract);
        let (page, date, quote) = get_extract_quote_metadata(&extract);
        let date_added = date.format("%Y-%m-%d %H:%M:%S").to_string();

        let book = match books.iter_mut().find(|b| b.name == name) {
            Some(b) => b,
            None => {
                books.push(BookData {
                    id: starting_id + books.len(),
                    name,
                    author,
                    quotes: Vec::new(),
                });
                books.last_mut().unwrap()
            }
        };

        let id = book.quotes.len() as i32;
        book.quotes.push(Quote {
            id,
            quote,
            page_number: page,
            date_added,
        });
    }

    Ok(books)
}
```

The parser splits `My Clippings.txt` by the ========== delimiter and processes each extract. For each one, it extracts the book name and author (using regex to handle both parenthetical and dash-separated formats), then pulls the page number, date, and quote text using carefully crafted patterns. Crucially, it groups quotes by book, if a book already exists, new quotes are appended to it; otherwise, a fresh `BookData` struct is created. This gives us a clean, hierarchical structure ready for display or export.

# The Exporter

Once we've parsed the data, we need a way to export it in useful formats.

```rust
pub fn export_to_md(book_data: &BookData, path: &Path) -> io::Result<()> {
    let mut content = format!("# {}\n\n", book_data.name);
    if !book_data.author.is_empty() {
        content.push_str(&format!("*by {}*\n\n", book_data.author));
    }

    let blocks: Vec<String> = book_data
        .quotes
        .iter()
        .map(|quote| {
            format!(
                "---\n> {}\n- Quote ID: {}\n- Page: {}\n- Date Added: {}",
                quote.quote, quote.id, quote.page_number, quote.date_added
            )
        })
        .collect();
    content.push_str(&blocks.join("\n\n"));

    let file_path = path.join(format!("{}.md", sanitize_filename(&book_data.name)));
    std::fs::write(file_path, content)
}

pub fn save_quotes(books: &[BookData], path: &Path) -> io::Result<()> {
    match bincode::serialize(books) {
        Ok(bytes) => fs::write(path, bytes),
        Err(err) => Err(io::Error::new(io::ErrorKind::Other, err)),
    }
}
```

The exporter handles two formats: markdown (human-readable) and binary (efficient). The markdown exporter builds a formatted document with the book title, author, and quotes separated by dividers perfect for reading or sharing. For persistence, `save_quotes()` serializes the entire `BookData` vector using `bincode`, a compact binary format. This lets users save their parsed data and reload it later without re-parsing the Clippings file. The `sanitize_filename()` helper prevents filesystem errors by replacing invalid characters.

# Conclusion

Once you know a programming language you do not need to go through any tutorials of other programming langauges. All you need is practice, practice, practice. You already *have* the problem-solving skills necessary, what you lack is knowledge. You gain that knowledge not by watching Rust programming tutorials but by doing your projects.

# Further Reading

- [GitHub Repository](https://github.com/K0Stek122/kindle-extractor-rs)
- [Iced-rs Documentation](https://book.iced.rs/)
- [Elm Programming Language](https://elm-lang.org/)
