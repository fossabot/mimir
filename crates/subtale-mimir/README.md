> Mímir is a contextual query engine for video games with dynamic events (e.g. dialog, animations) driven by their current world's state.

## Documentation

Our official documentation at **[mimir.subtale.com][docs]** offers significantly more detail than this README, including:

* [a tutorial for newcomers][tutorial]
* [a full changelog][changelog]
* [example use cases][use-cases]

## Inspiration

Mímir is heavily inspired (both in concept and general architecture) by [Elan Ruskin's amazing session from GDC 2012 on AI-driven Dynamic Dialog][gdc].

Fundamentally speaking, Mímir is simply a Rust implementation of Elan's proposed system for dynamic dialog. However, Mímir does offer some differences and/or extensions that cater specifically to games developed internally at Subtale (documented below).

## High-level overview

Your game's world is defined as a collection of facts: the player killed x amount of enemies, an NPC has opened y amount of doors, the player is currently near z NPC, etc.

In Mímir, facts are collected together into a map ([`Query`][query]), where the key is the unique identifier of the fact, and the value is the fact's value (represented as a `f64`).

Also, your game will (most likey!) have predefined rules that define behaviour that should occur when one or more facts are true. We represent rules as a map ([`Rule<T>`][rule]), where the key is the unique identifier of the fact, and the value is a predicate ([`Evaluator`][evaluator]) that is evaluated against the fact's value.

Finally, rules can be stored together in collections known as rulesets ([`Ruleset<T>`][ruleset]). Rulesets allow a query to be evaluated against many rules at once: Mímir will always look to match a query against the rule in the ruleset with the most requirements (i.e. more specific). *(If multiple rules are matched with the same specificity, one is chosen at random.)*

## Example

```rs
use subtale_mimir::{evaluator::{Evaluator, FloatEvaluator}, rule::{Query, Rule, Ruleset}};

// create a rule requiring that five enemies have been killed
let mut rule = Rule::new("You killed 5 enemies!");
rule.insert("enemies_killed", FloatEvaluator::EqualTo(5.));

// create a more specific rule that also requires at least 2 doors to have been opened
let mut more_specific_rule = Rule::new("You killed 5 enemies and opened 2 doors!");
more_specific_rule.insert("enemies_killed", FloatEvaluator::EqualTo(5.));
more_specific_rule.insert("doors_opened", FloatEvaluator::gte(2.));

// bundle the rules into a ruleset
let ruleset = Ruleset::new(vec![rule, more_specific_rule]);

// run a query against the ruleset
let mut query = Query::new();
query.insert("enemies_killed", 2.5 + 1.5 + 1.);

assert_eq!(
ruleset.evaluate(&query).unwrap().outcome,
"You killed 5 enemies!"
);

// run a more specific query against the ruleset
let mut more_specific_query = Query::new();
more_specific_query.insert("enemies_killed", 2.5 + 1.5 + 1.);
more_specific_query.insert("doors_opened", 10.);

assert_eq!(
ruleset.evaluate(&more_specific_query).unwrap().outcome,
"You killed 5 enemies and opened 2 doors!"
);
```

In the above example, we define a ruleset with two rules. Both rules require that 5 enemies have been killed, but one rule is more specific (also requiring that more than 2 doors have been opened).

The first query evaluates to the simpler rule, because the query does not satisfy the doors opened requirement. However, the second query evaluates to the more complex rule because the query *does* satistfy the doors opened requirement (note that even though the simpler rule is still satisfied, Mímir does not evaluate it as true because it's less specific/contains fewer requirements).

[docs]: https://mimir.subtale.com
[tutorial]: https://mimir.subtale.com/tutorial
[changelog]: https://mimir.subtale.com/changelog
[use-cases]: https://mimir.subtale.com/use-cases/tips
[query]: https://mimir.subtale.com/concepts/query
[rule]: https://mimir.subtale.com/concepts/rule
[evaluator]: https://mimir.subtale.com/concepts/evaluator
[ruleset]: https://mimir.subtale.com/concepts/ruleset
[gdc]: https://www.youtube.com/watch?v=tAbBID3N64A
