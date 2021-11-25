# Custom directive

`Async-graphql` can easily customize directives, which can extend the behavior of GraphQL.

To create a custom directive, you need to implement the `CustomDirective` trait, and then use the `Directive` macro to 
generate a factory function that receives the parameters of the directive and returns an instance of the directive.

Currently `Async-graphql` only supports directive located at `FIELD`.

```rust
struct ConcatDirective {
    value: String,
}

#[cfg_attr(feature = "single-threaded-runtime", async_trait::async_trait(?Send))]
#[cfg_attr(not(feature = "single-threaded-runtime"), async_trait::async_trait)]
impl CustomDirective for ConcatDirective {
    async fn resolve_field(&self, _ctx: &Context<'_>, resolve: ResolveFut<'_>) -> ServerResult<Option<Value>> {
        resolve.await.map(|value| {
            value.map(|value| match value {
                Value::String(str) => Value::String(str + &self.value),
                _ => value,
            })
        })
    }
}

#[Directive(location = "field")]
fn concat(value: String) -> impl CustomDirective {
    ConcatDirective { value }
}
```

Register the directive when building the schema:

```rust
let schema = Schema::build(Query, EmptyMutation, EmptySubscription)
    .directive(concat)
    .finish();
```
