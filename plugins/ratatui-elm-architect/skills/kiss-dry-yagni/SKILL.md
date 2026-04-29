---
name: kiss-dry-yagni
description: Use when writing, reviewing, or refactoring Rust code and you spot signs of over-engineering — long fns, deep nesting, duplicated logic, premature traits/generics, "just in case" code paths, or features nobody asked for.
---

# KISS, DRY, YAGNI (Rust)

## Overview

Three rules for keeping Rust code simple, maintainable, and easy to evolve.

| Principle | Question | Failure mode |
|-----------|----------|--------------|
| **KISS** — Keep It Simple, Stupid | "Is there a simpler way?" | Cleverness, deep nesting, god functions |
| **DRY** — Don't Repeat Yourself | "Does this knowledge live in exactly one place?" | Copy-paste, drifting rules, shotgun surgery |
| **YAGNI** — You Aren't Gonna Need It | "Is this required *right now*?" | Speculative abstractions, dead config, gold plating |

**Review prompt:** ask all three before approving any change.

## When to Use

Apply when:
- Designing a new module, struct, trait, or fn
- Reviewing a PR that feels "too clever", "too generic", or trait-happy
- Refactoring code that has grown organically
- Tempted to add a config knob, feature flag, or generic parameter "for later"
- An fn exceeds ~20 lines, nests deeply, or is hard to name
- The same logic / rule / formula appears in 2+ places
- Scope is creeping with features nobody asked for

Skip for pure mechanical edits (renames, formatting, `cargo fmt`). Document the trade-off when measured performance demands intentional complexity.

---

## KISS

**Simplicity is a primary design goal.** The simplest code that solves the problem is usually the best.

Limits (target / hard cap):

- Fn length: 10 / 20 lines
- Cyclomatic complexity: 5 / 10
- Nesting depth: 3
- Parameters: 4 (use a struct beyond)
- Module / file: 150 / 200 lines
- Generic parameters per item: 2 (more = ask if it's earning its weight)

Two heuristics catch most violations.

### 1. Decompose instead of nesting

```rust
// BAD — pricing, discounts, and category rules tangled in one function
fn calculate_price(order: &Order) -> Money {
    let mut total = Money::zero();
    for item in &order.items {
        let mut price = item.base_price;
        if item.category == Category::Food {
            if item.is_organic {
                if item.weight_kg > 1.0 {
                    price = price * 0.90;
                } else {
                    price = price * 0.95;
                }
            }
            // ...50 more lines of conditions
        }
        total = total + price;
    }
    total
}

// GOOD — small, named, composable pieces
fn order_total(order: &Order) -> Money {
    order.items.iter().map(item_price).sum()
}

fn item_price(item: &Item) -> Money {
    apply_discounts(item.base_price, item)
}
```

### 2. Guard clauses instead of nested else

```rust
// BAD — pyramid of doom
fn process(user: Option<&User>) -> Result<(), Error> {
    if let Some(u) = user {
        if u.is_active {
            if u.has_permission() {
                Ok(()) // business logic
            } else { Err(Error::NoPermission) }
        } else { Err(Error::Inactive) }
    } else { Err(Error::NotFound) }
}

// GOOD — flat, with guards
fn process(user: Option<&User>) -> Result<(), Error> {
    let user = user.ok_or(Error::NotFound)?;
    if !user.is_active { return Err(Error::Inactive); }
    if !user.has_permission() { return Err(Error::NoPermission); }
    Ok(()) // business logic at zero indentation
}
```

Names should be explicit enough to make comments unnecessary. Prefer composition over inheritance, immutability by default.

---

## DRY

**Every piece of knowledge has exactly one authoritative representation.**

DRY is about *knowledge*, not text. Two functions that look similar but encode different rules are not duplication — extracting them creates false coupling.

### Centralize a rule in a value object

```rust
// BAD — "valid email" lives in three places that will drift
fn handler(input: &str) -> Result<(), Error> {
    if !is_valid_email(input) { return Err(Error::InvalidEmail); }
    // ...
}

#[derive(Validate)]
struct UserEntity {
    #[validate(email)]
    email: String,
}

// + a third copy in the form layer

// GOOD — one type owns the rule; everyone else uses it
pub struct Email(String);

impl Email {
    pub fn parse(value: impl Into<String>) -> Result<Self, EmailError> {
        let v = value.into();
        if !is_valid_email(&v) { return Err(EmailError::Invalid(v)); }
        Ok(Self(v))
    }
}
// Entity, form, controller all consume `Email`.
```

### The Rule of Three

> **Don't abstract until you've seen the pattern three times.**

Seen once → copy. Seen twice → note. Seen three times → abstract. Premature abstraction costs more than duplication.

### When duplication is fine

- Similar shape, different *types* (kept separate for type safety)
- Test code — clarity beats DRY; Arrange/Act/Assert wins
- Per-environment configuration

### When it isn't

- Business rules
- Validation
- Algorithms / formulas
- Domain calculations

---

## YAGNI

**Don't implement a feature until it is actually required.**

No code "just in case." No abstractions for hypothetical futures. No options nobody asked for.

### Build only what is asked

```rust
// BAD — five formats, only CSV was requested
impl ExportService {
    fn export(&self, data: &Data, format: Format) -> Vec<u8> {
        match format {
            Format::Csv  => self.csv(data),   // requested
            Format::Xml  => self.xml(data),   // not requested
            Format::Json => self.json(data),  // not requested
            Format::Pdf  => self.pdf(data),   // not requested
            Format::Xlsx => self.xlsx(data),  // not requested
        }
    }
}

// GOOD — only CSV exists. No trait, no enum, no plugin point.
pub struct CsvExporter;
impl CsvExporter {
    pub fn export(data: &Data) -> Vec<u8> { /* CSV only */ }
}
```

If a real PDF requirement appears, add a separate `PdfExporter` struct alongside it — no shared trait yet. Extract a trait only when a *third* exporter shows up; at that point the shared shape is real, not speculative (Rule of Three).

### YAGNI checklist

Before adding any feature, abstraction, or option:

- [ ] Required **now**, in the current ticket?
- [ ] Failing test or explicit acceptance criterion exists?
- [ ] Inside agreed scope / MVP?
- [ ] Customer / user explicitly asked for it?

Any **no** → don't build it yet.

---

## Anti-patterns and red flags

If you catch yourself thinking any of these, stop.

| Thought | What it actually is |
|---------|---------------------|
| "I'll add a config option in case we ever need it" | YAGNI violation |
| "Let's make this generic now, it'll save time later" | Speculative generality |
| "I'll keep both implementations around for safety" | Dead code |
| "The abstraction is ugly but it's DRY" | False DRY — text, not knowledge |
| "Nobody asked for it but it would be cool" | Gold plating |
| "We might need caching" (no profiling done) | Premature optimization |
| Three+ trait / newtype layers to call one fn | Lasagna code |
| `<T, U, V, W>` on a type with one call site | Premature generics |

Each one is a smell that one of the three principles is being violated.

---

## Pre-commit checklist

**KISS** — fns short, complexity low, nesting ≤ 3, params ≤ 4, no nested `else`, names explicit.

**DRY** — no duplicated *knowledge*; validation and domain rules live in exactly one place (often a newtype / value object).

**YAGNI** — every code path maps to a current requirement; no "just in case" branches; no traits or generics without a third use case.

### Target metrics

| Metric | Target | Hard cap |
|--------|--------|----------|
| Lines per fn | < 10 | < 20 |
| Cyclomatic complexity | < 5 | < 10 |
| Lines per module / file | < 150 | < 200 |
| Knowledge duplication | 0% | < 3% |
| Dependencies per type | < 5 | < 7 |
| Generic params per item | ≤ 2 | ≤ 3 |

---

## References

- *The Pragmatic Programmer* — Andy Hunt & Dave Thomas
- *Clean Code* — Robert C. Martin
- [YAGNI — Martin Fowler](https://martinfowler.com/bliki/Yagni.html)
- [KISS Principle (Wikipedia)](https://en.wikipedia.org/wiki/KISS_principle)
- [DRY Principle (Wikipedia)](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)
