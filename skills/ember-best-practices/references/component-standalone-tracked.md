---
title: Keep App State on Classes, Not in Standalone tracked() Values
impact: MEDIUM
impactDescription: Keeps state per instance, or in a service, rather than in module scope
tags: components, tracked, reactivity, testing
---

## Keep App State on Classes, Not in Standalone tracked() Values

Since Ember 7.3, `tracked` from `@glimmer/tracking` can also be called as a function: `tracked(0)` returns a reactive value that's read and written through `.value`. It doesn't replace `@tracked` fields. App state still belongs on a class: a component's `@tracked` fields, so each instance gets its own, or a service for state that's shared ([service-shared-state](service-shared-state.md)). Use the function form only in code with no class to hold the value, such as tests and demos.

**Incorrect (a template-only component keeps its state in module scope):**

```glimmer-js
// app/components/counter.gjs
import { tracked } from '@glimmer/tracking';

const count = tracked(0);
const increment = () => count.value++;

export const Counter = <template>
  <button type="button" {{on "click" increment}}>{{count.value}}</button>
</template>;
```

`count` lives in the module, so every `<Counter />` on the page shares one count.

**Correct (component state in a `@tracked` field):**

```glimmer-js
// app/components/counter.gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';

export class Counter extends Component {
  @tracked count = 0;

  increment = () => this.count++;

  <template>
    <button type="button" {{on "click" this.increment}}>{{this.count}}</button>
  </template>
}
```

**Correct (a tracked value for a test's state):**

```glimmer-js
// tests/integration/components/name-tag-test.gjs
import { module, test } from 'qunit';
import { setupRenderingTest } from 'ember-qunit';
import { render, settled } from '@ember/test-helpers';
import { tracked } from '@glimmer/tracking';
import { NameTag } from 'my-app/components/name-tag';

module('Integration | Component | name-tag', function (hooks) {
  setupRenderingTest(hooks);

  test('it shows the new name', async function (assert) {
    const name = tracked('Ada');

    await render(<template><NameTag @name={{name.value}} data-test-name-tag /></template>);
    name.value = 'Grace';
    await settled();

    assert.dom('[data-test-name-tag]').includesText('Grace');
  });
});
```

Before 7.3, a test needed a class with a `@tracked` field, or a `trackedObject`, to hold a single value.

- **Equality:** writing an equal value to a `tracked()` value doesn't notify anything (it compares with `Object.is`), so mutating an array or object in `.value` and assigning it back does nothing. Assign a new array or object, or use a tracked collection ([component-tracked-toolbox](component-tracked-toolbox.md)). A `@tracked` field still notifies on every write unless it's declared `@tracked({ equals: Object.is })`.
- **Older versions:** before Ember 7.3, `tracked` is only a decorator.

Reference: [Ember 7.3 Released](https://blog.emberjs.com/ember-released-7-3/), [RFC #1071](https://github.com/emberjs/rfcs/pull/1071), [Ember API: tracked](https://api.emberjs.com/ember/release/functions/@glimmer%2Ftracking/tracked)
