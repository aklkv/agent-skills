---
title: Write Precise Signatures for Components, Helpers and Modifiers
impact: MEDIUM
impactDescription: Type-checked invocations and accurate editor completions
tags: components, helpers, modifiers, typescript, glint, signatures
---

## Write Precise Signatures for Components, Helpers and Modifiers

Glint checks every invocation against the signature of the component, helper or modifier being invoked, and editors read the same signature for completions and hover docs. A signature that is looser than the template, or says something the template doesn't do, switches those checks off for every caller.

**Incorrect (loose types that don't match the template):**

```glimmer-ts
// app/components/user-list.gts
import Component from '@glimmer/component';
import { UserCard } from './user-card';

type Args = {
  users: any[];
  onSelect: Function;
};

interface Signature {
  Args: Args;
  Element: null;
  Blocks: {
    default: [any];
    empty: [];
  };
}

export class UserList extends Component<Signature> {
  <template>
    <ul>
      {{#each @users as |user|}}
        <li>{{yield (component UserCard user=user onSelect=@onSelect)}}</li>
      {{/each}}
    </ul>
  </template>
}
```

`any` and `Function` turn off checking for every caller, `empty` is declared but never yielded, and without `...attributes` callers can't add a `class` or an `aria-*` attribute.

**Correct (an exported interface that matches the template):**

```glimmer-ts
// app/components/user-list.gts
import Component from '@glimmer/component';
import type { WithBoundArgs } from '@glint/template';
import { UserCard, type User } from './user-card';

export interface UserListSignature {
  Args: {
    users: User[];
    /** Called with the user whose card was clicked. */
    onSelect: (user: User) => void;
  };
  Element: HTMLUListElement;
  Blocks: {
    default: [UserCard: WithBoundArgs<typeof UserCard, 'user' | 'onSelect'>, index: number];
  };
}

export class UserList extends Component<UserListSignature> {
  <template>
    <ul ...attributes>
      {{#each @users as |user index|}}
        <li>{{yield (component UserCard user=user onSelect=@onSelect) index}}</li>
      {{/each}}
    </ul>
  </template>
}
```

- **Name and shape:** declare `<ComponentName>Signature` as an exported `interface`, with `Args` written inline. Other code refers to its parts as `UserListSignature['Args']`.
- **`Args`:** document an argument with a JSDoc comment on its field; editors show it on hover.
- **`Element`:** the element `...attributes` is spread on, or a union when branches render different elements. Leave it out when the component has no `...attributes`; don't write `Element: null`.
- **`Blocks`:** only the blocks the template yields. Label positional block params (`UserCard:`, `index:`); editors use the labels as block param names.
- **Yielded components:** type them as `typeof X`, or as `WithBoundArgs<typeof X, 'arg'>` when yielded through `(component X arg=…)`, so callers are still checked on the args they pass.
- **Component arguments:** type a component passed in as an argument as `ComponentLike<Signature>` from `@glint/template`.

**Correct (a wrapper derives its signature from the component it wraps):**

```glimmer-ts
// app/components/primary-button.gts
import type { TOC } from '@ember/component/template-only';
import { Button, type ButtonSignature } from './button';

export interface PrimaryButtonSignature {
  Args: {
    size?: ButtonSignature['Args']['size'];
  };
  Element: ButtonSignature['Element'];
  Blocks: {
    default: [];
  };
}

export const PrimaryButton: TOC<PrimaryButtonSignature> = <template>
  <Button @kind="primary" @size={{@size}} ...attributes>{{yield}}</Button>
</template>;
```

Template-only components take the same signature through `TOC`. Deriving `Element` and individual args from `ButtonSignature` keeps the wrapper in step when `Button` changes. `Blocks` still lists only what this template yields.

**Incorrect (an untyped helper):**

```ts
// app/helpers/format-currency.ts
import { helper } from '@ember/component/helper';

export default helper(([amount]: any[], { currency }: any) =>
  new Intl.NumberFormat('en-US', { style: 'currency', currency: currency ?? 'USD' }).format(amount),
);
```

**Correct (a plain function whose parameter types are the signature):**

```ts
// app/helpers/format-currency.ts
export function formatCurrency(
  amount: number,
  { currency = 'USD' }: { currency?: string } = {},
): string {
  return new Intl.NumberFormat('en-US', { style: 'currency', currency }).format(amount);
}

export default formatCurrency;
```

Write every helper as a plain function unless it needs services. Export it by name for imports, and as the default so resolver lookup (`.hbs` templates) finds it too. Glint reads positional arguments from the parameters and named arguments from the trailing object, so a plain function needs no signature interface. Ember passes that object only when the caller passes a named argument, hence the `= {}` default. A helper that needs services is a class, declared as `Helper<{ Args: { Positional: [...]; Named: {...} }; Return: ... }>`.

**Incorrect (an untyped modifier):**

```ts
// app/modifiers/on-click-outside.ts
import { modifier } from 'ember-modifier';

export default modifier((element: any, [onClickOutside]: any[], named: any) => {
  // ...
});
```

**Correct (types written on the function's parameters):**

```ts
// app/modifiers/on-click-outside.ts
import { modifier } from 'ember-modifier';

export default modifier(
  (
    element: HTMLElement,
    [onClickOutside]: [(event: MouseEvent) => void],
    { capture = false }: { capture?: boolean },
  ) => {
    const handler = (event: MouseEvent) => {
      if (event.target instanceof Node && !element.contains(event.target)) {
        onClickOutside(event);
      }
    };

    document.addEventListener('click', handler, { capture });
    return () => document.removeEventListener('click', handler, { capture });
  },
);
```

The element's type limits which elements the modifier can be applied to. A class-based modifier, for one that needs services, declares `Modifier<{ Element: ...; Args: { Positional: [...]; Named: {...} } }>` and types `modify()`'s parameters from it.

Reference: [TypeScript: Invokables](https://guides.emberjs.com/release/typescript/core-concepts/invokables/), [Glint: Component Signatures](https://typed-ember.gitbook.io/glint/environments/ember/component-signatures)
