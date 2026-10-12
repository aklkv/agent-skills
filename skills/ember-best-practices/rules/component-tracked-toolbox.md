---
title: Use Tracked Collections for Complex State
impact: HIGH
impactDescription: Cleaner state management
tags: components, tracked, state-management, performance
---

## Use Tracked Collections for Complex State

For complex state patterns like maps, sets, and arrays that need fine-grained reactivity, use the tracked collections from `@ember/reactive/collections` instead of marking entire structures as @tracked. They're built into Ember since 6.8; on older versions, use the `tracked-built-ins` addon.

**Incorrect (tracking entire structures):**

```javascript
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';

class TodoList extends Component {
  @tracked items = []; // Entire array replaced on every change

  addItem = (item) => {
    // Creates new array, invalidates all consumers
    this.items = [...this.items, item];
  };

  removeItem = (index) => {
    // Creates new array again
    this.items = this.items.filter((_, i) => i !== index);
  };
}
```

**Correct (using tracked collections):**

```javascript
import Component from '@glimmer/component';
import { trackedArray } from '@ember/reactive/collections';

class TodoList extends Component {
  items = trackedArray([]);

  // Use arrow functions for methods used in templates (no @action needed)
  addItem = (item) => {
    // Efficiently adds to tracked array
    this.items.push(item);
  };

  removeItem = (index) => {
    // Efficiently removes from tracked array
    this.items.splice(index, 1);
  };
}
```

**Also useful for Maps and Sets:**

```javascript
import { trackedMap, trackedSet } from '@ember/reactive/collections';

class Cache extends Component {
  cache = trackedMap(); // Fine-grained reactivity per key
  selected = trackedSet(); // Fine-grained reactivity per item
}
```

Tracked collections provide fine-grained reactivity and better performance than replacing entire structures.

Reference: [RFC #1068: Tracked Collections](https://rfcs.emberjs.com/id/1068-tracked-collections/)
