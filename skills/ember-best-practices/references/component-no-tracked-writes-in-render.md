---
title: Don't Update Tracked State While Rendering
impact: MEDIUM
impactDescription: Avoids backtracking assertions and stale caches
tags: components, tracked, autotracking, rendering
---

## Don't Update Tracked State While Rendering

Getters and helpers run while a template renders. If one of them writes a tracked property that the render has already read, Ember throws in development:

> You attempted to update `sortedCache` on …, but it had already been used previously in the same computation.

The usual cause is a lazy cache kept in a `@tracked` field: the getter reads the field, finds it empty and fills it, all in the same render.

**Incorrect (a getter fills a tracked cache):**

```glimmer-js
// app/components/leaderboard.gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';

export class Leaderboard extends Component {
  @tracked sortedCache;

  get sorted() {
    this.sortedCache ??= [...this.args.players].sort((a, b) => b.score - a.score);
    return this.sortedCache;
  }

  <template>
    {{#each this.sorted as |player|}}
      <p>{{player.name}}: {{player.score}}</p>
    {{/each}}
  </template>
}
```

Besides throwing, the cache never updates when `@players` changes.

**Correct (derive the value with `@cached`):**

```glimmer-js
// app/components/leaderboard.gjs
import Component from '@glimmer/component';
import { cached } from '@glimmer/tracking';

export class Leaderboard extends Component {
  // Memoized, and recomputed when @players changes
  @cached
  get sorted() {
    return [...this.args.players].sort((a, b) => b.score - a.score);
  }

  <template>
    {{#each this.sorted as |player|}}
      <p>{{player.name}}: {{player.score}}</p>
    {{/each}}
  </template>
}
```

- **Derived values** belong in getters, memoized with `@cached` when they're expensive ([component-cached-getters](component-cached-getters.md)).
- **Values computed once per instance** that never need to re-render anything, such as an id, belong in a plain class field, which isn't tracked ([component-minimal-tracking](component-minimal-tracking.md)).
- **Tracked state** changes in response to something that happens after rendering, such as an event handler.

Reference: [Ember API: @cached](https://api.emberjs.com/ember/release/functions/@glimmer%2Ftracking/cached), [Ember Guides: Autotracking In-Depth](https://guides.emberjs.com/release/in-depth-topics/autotracking-in-depth/)
