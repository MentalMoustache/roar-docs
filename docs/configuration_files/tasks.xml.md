# `tasks.xml`

This file controls various things that you can *do* in the game. 

Tasks follow the pattern of:

- [**Requirements**](https://github.com/roarengine/roar-docs/blob/master/docs/concepts/requirements.md) : minimum thresholds to meet
- [**Costs**](https://github.com/roarengine/roar-docs/blob/master/docs/concepts/costs.md) : removed or decremented from the player
- [**Modifiers**](https://github.com/roarengine/roar-docs/blob/master/docs/concepts/modifiers.md) : given to the player

For a task to "complete", a player MUST: 

1. Meet all the requirements, and;
2. Be able to pay all the costs.

If these conditions are met, the `<start_costs>` are applied to the player (which can in fact _increase_ certain stats, not just decrease - eg. your `infamy` goes up as a "cost"), and the `<on_complete>` modifiers are applied (and again, these can cause stats to go down or items to be removed, rather than only causing increases and gains).

Here's a typical task, showing many of (but not all) the features of tasks.

```xml
<task
  type="instant_complete"
  ikey="grow_your_family"
  label="Grow Your Family"
  description="Do something for someone and get rewarded"
  location="new_york"
  tags="associate"
>
  <tag value="associate" />
  <tag value="random_tag" />
  <tag value="third_tag" />

  <start_costs>
    <stat_cost ikey="energy" value="3"/>
  </start_costs>
  <start_requirements>
    <item_requirement ikey="tommy_gun" number_required="1" consumed="false"/>
    <level_requirement level="2"/>
  </start_requirements>
  <on_complete>
    <grant_stat ikey="cash" value="900"/>
    <grant_xp value="3"/>
  </on_complete>
</task>
```

## Task types

Tasks come in two types `instant_complete` and `normal`.

### Instant Complete
An instant complete task is completed as soon as the player accepts the task. Consequently it should not have the following elements: `<on_start>`, `<complete_costs>` or `<complete_requirement>`.

This kind of task is often used to implement "crafting" or for tracking player actions.

### Normal Tasks
A normal task has several phases, an initial phase where the player can accept the task, a phase where the player is completing the requirements for the task, then a phase where the player can complete the task. The various elements below control what happens at each phase transition.

This kind of task is often used to implement "quests" and quest chains.

## `visibility` (requirement)
The visibility element is a [requirement](../concepts/requirements.md) that determines whether the user should see the task when they list all available tasks. It defaults to true if not provided.

## `start_costs` (cost)
The [cost](../concepts/costs.md) for the player to move the task into an "active" state. Defaults to nothing.

## `start_requirements` (requirement)
The additional [requirements](../concepts/requirement.md) that the player must satisfy to start the task.

## `on_start` (modifier)
A [modifier](../concepts/modifiers.md) that happens to the player when they successfully start a task.

## `complete_costs` (cost)
The [cost](../concepts/costs.md) for the player to complete the task.

## `complete_requirements` (requirement)
The additional [requirements](../concepts/requirement.md) that the player must satisfy to complete the task.

## `on_complete` (modifier)
A [modifier](../concepts/modifiers.md) that happens to the player when they successfully complete a task.

## `tag`
For sorting and filtering tasks on the client.


### Types and usage

Combining the requirements, costs and modifiers for tasks allows you to increment and decrement attribute flags that can enable various types of tasks, including.

- Sub-tasks
- Chaining
- Single completion quests
- Daily quests

Let your imagination go wild.
