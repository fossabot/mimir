# Query

A query is a collection of facts about the current game world's state.

Mímir represents these facts in Rust as a `HashMap<FactKey, FactType>`, where the `FactKey` generic type indicates the unique name of the fact, and the `FactType` generic type indicates the type of the fact's value.

```rs
struct Query<FactKey, FactType> {
    facts: HashMap<FactKey, FactType>,
}
```