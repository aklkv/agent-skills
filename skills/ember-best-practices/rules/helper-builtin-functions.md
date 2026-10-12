---
title: Use Helper Libraries Effectively
impact: MEDIUM
impactDescription: Reduces custom helper maintenance and keeps templates concise
tags: templates, helpers, ember-truth-helpers
---

## Use Helper Libraries Effectively

Leverage community helper libraries to write cleaner templates and avoid creating unnecessary custom helpers for common operations.

## Problem

Reinventing common functionality with custom helpers adds maintenance burden and bundle size when well-maintained helper libraries already provide the needed functionality.

**Incorrect:**

```glimmer-js
// app/utils/is-equal.js - Unnecessary custom helper
export function isEqual(a, b) {
  return a === b;
}

// app/components/user-badge.gjs
import { isEqual } from '../utils/is-equal';

class UserBadge extends Component {
  <template>
    {{#if (isEqual @user.role "admin")}}
      <span class="badge">Admin</span>
    {{/if}}
  </template>
}
```

## Solution

**Note:** Ember 7.1 builds these helpers into strict-mode templates as keywords. On older versions, install the addon:

```bash
npm install ember-truth-helpers
```

Use the helpers built into strict-mode templates. Since Ember 7.1, `eq`, `neq`, `not`, `and`, `or`, `lt`, `lte`, `gt`, `gte`, `array`, `hash` and `fn` are keywords that need no import. On older versions, import them from `ember-truth-helpers` and `@ember/helper`:

**Correct:**

```glimmer-js
// app/components/user-badge.gjs
import Component from '@glimmer/component';

class UserBadge extends Component {
  <template>
    {{! eq is a keyword since Ember 7.1 }}
    {{#if (eq @user.role "admin")}}
      <span class="badge">Admin</span>
    {{/if}}
  </template>
}
```

## Comparison Helpers

Keywords since Ember 7.1. On older versions, install `ember-truth-helpers` and import them from it.

```glimmer-js
// app/components/comparison-examples.gjs
import Component from '@glimmer/component';

class ComparisonExamples extends Component {
  <template>
    {{! Equality }}
    {{#if (eq @status "active")}}Active{{/if}}

    {{! Negation }}
    {{#if (not @isDeleted)}}Visible{{/if}}

    {{! Logical AND }}
    {{#if (and @isPremium @hasAccess)}}Premium Content{{/if}}

    {{! Logical OR }}
    {{#if (or @isAdmin @isModerator)}}Moderation Tools{{/if}}

    {{! Comparisons }}
    {{#if (gt @score 100)}}High Score!{{/if}}
    {{#if (lte @attempts 3)}}Try again{{/if}}
  </template>
}
```

## Array and Object Helpers

`array` and `hash` are keywords since Ember 7.1; on older versions, import them from `@ember/helper`. `get` is imported from `@ember/helper`.

```glimmer-js
// app/components/collection-helpers.gjs
import Component from '@glimmer/component';
import { get } from '@ember/helper';

class CollectionHelpers extends Component {
  <template>
    {{! Create array inline }}
    {{#each (array "apple" "banana" "cherry") as |fruit|}}
      <li>{{fruit}}</li>
    {{/each}}

    {{! Create object inline }}
    {{#let (hash name="John" age=30 active=true) as |user|}}
      <p>{{user.name}} is {{user.age}} years old</p>
    {{/let}}

    {{! Dynamic property access }}
    <p>{{get @user @propertyName}}</p>
  </template>
}
```

## String Helpers

```glimmer-js
// app/components/string-helpers.gjs
import Component from '@glimmer/component';
import { concat } from '@ember/helper'; // Built-in to Ember

class StringHelpers extends Component {
  <template>
    {{! Concatenate strings }}
    <p class={{concat "user-" @user.id "-card"}}>
      {{concat @user.firstName " " @user.lastName}}
    </p>

    {{! With dynamic values }}
    <img
      src={{concat "/images/" @category "/" @filename ".jpg"}}
      alt={{concat "Image of " @title}}
    />
  </template>
}
```

## Action Helpers (fn)

`fn` and `on` are keywords since Ember 7.1. On older versions, import `fn` from `@ember/helper` and `on` from `@ember/modifier`.

```glimmer-js
// app/components/action-helpers.gjs
import Component from '@glimmer/component';

class ActionHelpers extends Component {
  updateValue = (field, event) => {
    this.args.onChange(field, event.target.value);
  };

  deleteItem = (id) => {
    this.args.onDelete(id);
  };

  <template>
    {{! Partial application with fn }}
    <input {{on "input" (fn this.updateValue "email")}} />

    {{#each @items as |item|}}
      <li>
        {{item.name}}
        <button {{on "click" (fn this.deleteItem item.id)}}>
          Delete
        </button>
      </li>
    {{/each}}
  </template>
}
```

## Conditional Helpers (if/unless)

`if` and `unless` are keywords in every Ember version, inline or as blocks; there is nothing to import.

```glimmer-js
// app/components/conditional-inline.gjs
import Component from '@glimmer/component';

class ConditionalInline extends Component {
  <template>
    {{! Ternary-like behavior }}
    <span class={{if @isActive "active" "inactive"}}>
      {{@user.name}}
    </span>

    {{! Conditional attribute }}
    <button disabled={{if @isProcessing true}}>
      {{if @isProcessing "Processing..." "Submit"}}
    </button>

    {{! With default value }}
    <p>{{if @description @description "No description provided"}}</p>
  </template>
}
```

## Practical Combinations

**Dynamic Classes:**

```glimmer-js
// app/components/dynamic-classes.gjs
import Component from '@glimmer/component';
import { concat } from '@ember/helper';

class DynamicClasses extends Component {
  <template>
    <div
      class={{concat
        "card "
        (if @isPremium "premium ")
        (if (and @isNew (not @isRead)) "unread ")
        @customClass
      }}
    >
      <h3>{{@title}}</h3>
    </div>
  </template>
}
```

**List Filtering:**

```glimmer-js
// app/components/filtered-list.gjs
import Component from '@glimmer/component';
import { tracked } from '@glimmer/tracking';
import { cached } from '@glimmer/tracking';
import { concat } from '@ember/helper';

class FilteredList extends Component {
  @tracked filter = 'all';

  @cached
  get filteredItems() {
    if (this.filter === 'all') return this.args.items;
    return this.args.items.filter((item) => item.status === this.filter);
  }

  <template>
    <select {{on "change" (fn (mut this.filter) target.value)}}>
      {{#each (array "all" "active" "pending" "completed") as |option|}}
        <option value={{option}} selected={{eq this.filter option}}>
          {{option}}
        </option>
      {{/each}}
    </select>

    {{#each this.filteredItems as |item|}}
      <div class={{concat "item " item.status}}>
        {{item.name}}
      </div>
    {{/each}}
  </template>
}
```

## Complex Example

```glimmer-js
// app/components/user-profile-card.gjs
import Component from '@glimmer/component';
import { concat, get } from '@ember/helper';

class UserProfileCard extends Component {
  updateField = (field, value) => {
    this.args.onUpdate(field, value);
  };

  <template>
    <div
      class={{concat
        "profile-card "
        (if @user.isPremium "premium ")
        (if (and @user.isOnline (not @user.isAway)) "online ")
      }}
    >
      <h2>{{concat @user.firstName " " @user.lastName}}</h2>

      {{#if (or (eq @user.role "admin") (eq @user.role "moderator"))}}
        <span class="badge">
          {{get (hash admin="Administrator" moderator="Moderator") @user.role}}
        </span>
      {{/if}}

      {{#if (and @canEdit (not @user.locked))}}
        <div class="actions">
          {{#each (array "profile" "settings" "privacy") as |section|}}
            <button {{on "click" (fn this.updateField "activeSection" section)}}>
              Edit
              {{section}}
            </button>
          {{/each}}
        </div>
      {{/if}}

      <p class={{if @user.verified "verified" "unverified"}}>
        {{if @user.bio @user.bio "No bio provided"}}
      </p>
    </div>
  </template>
}
```

## Performance Impact

- **Library helpers**: ~0% overhead (compiled into efficient bytecode)
- **Custom helpers**: 5-15% overhead per helper call
- **Inline logic**: Cleaner templates, better tree-shaking

## When to Use

- **Library helpers**: For all common operations (equality, logic, arrays, strings)
- **Custom helpers**: Only for domain-specific logic not covered by library helpers
- **Component logic**: For complex operations that need @cached or multiple dependencies

## Complete Helper Reference

**Keywords in strict-mode templates since Ember 7.1 (no import):**

- `fn` - Partial application / bind arguments
- `hash` - Create object inline
- `array` - Create array inline
- `eq`, `neq` - Equality (===, !==)
- `not`, `and`, `or` - Logic
- `lt`, `lte`, `gt`, `gte` - Numeric comparisons
- `on` - Event listener modifier

**Keywords in every version:**

- `if`, `unless` - Conditional value or block
- `mut` - Create settable binding (use sparingly)

**Imported from `@ember/helper`:**

- `concat` - Concatenate strings
- `get` - Dynamic property access

**On Ember < 7.1:** import `fn`, `hash` and `array` from `@ember/helper`, `on` from `@ember/modifier`, and the comparison and logic helpers from `ember-truth-helpers`.

## References

- [Ember Built-in Helpers](https://guides.emberjs.com/release/templates/built-in-helpers/)
- [Template Helpers API](https://api.emberjs.com/ember/release/modules/@ember%2Fhelper)
- [fn Helper Guide](https://guides.emberjs.com/release/components/helper-functions/)
- [ember-truth-helpers](https://github.com/jmurphyau/ember-truth-helpers)
