---
title: Keep Expensive Modifier Setup Separate from Updates
impact: MEDIUM-HIGH
impactDescription: Stops editors, charts and players from being rebuilt on every change
tags: modifiers, reactivity, autotracking, ember-modifier
---

## Keep Expensive Modifier Setup Separate from Updates

A function-based modifier tracks everything it reads while it runs: its arguments and any tracked state. When any of it changes, the modifier calls its teardown and runs again from the start. That's cheap for an event listener, but a modifier that creates an editor, a chart or a media player rebuilds it on every change, losing focus, cursor position and undo history.

It can rerun even when nothing changed: setting a tracked property invalidates it although the new value equals the old one, so a parent that writes the same value back reruns the modifier.

**Incorrect (the editor is rebuilt on every keystroke):**

```glimmer-js
// app/components/code-editor.gjs
import { modifier } from 'ember-modifier';
import { createEditor } from 'some-editor';

const editor = modifier((element, [value, onChange]) => {
  const instance = createEditor(element, { value });
  instance.on('change', () => onChange(instance.getValue()));
  return () => instance.destroy();
});

export const CodeEditor = <template>
  <div {{editor @value @onChange}}></div>
</template>;
```

Each edit calls `@onChange`, the parent updates `@value`, and the modifier destroys the editor and creates a new one.

**Correct (set up once, then apply changes in place):**

```glimmer-js
// app/components/code-editor.gjs
import Component from '@glimmer/component';
import { modifier } from 'ember-modifier';
import { createEditor } from 'some-editor';

export class CodeEditor extends Component {
  // Read once, when the component is created
  initialValue = this.args.value;
  editor;
  syncing = false;

  // Reads no tracked state while it runs, so it runs once per element
  setupEditor = modifier((element) => {
    this.editor = createEditor(element, { value: this.initialValue });
    this.editor.on('change', () => {
      // Report the user's edits, not values applied by syncValue
      if (!this.syncing) this.args.onChange(this.editor.getValue());
    });
    return () => this.editor.destroy();
  });

  // Reruns whenever @value is set, and only touches the editor when it differs
  syncValue = modifier((element, [value]) => {
    if (this.editor.getValue() !== value) {
      this.syncing = true;
      this.editor.setValue(value);
      this.syncing = false;
    }
  });

  <template>
    <div {{this.setupEditor}} {{this.syncValue @value}}></div>
  </template>
}
```

- **Setup reads nothing that changes.** It reads `this.initialValue`, a plain field, and reads `@onChange` only when the editor fires an event, after the modifier has run.
- **Updates are cheap and compare first.** A rerun with an unchanged value does nothing. Many editors fire their change event from `setValue()` too, so `syncing` keeps a value the parent set from being reported back as an edit.
- **Order:** modifiers on one element install in the order they're written, so `this.editor` exists when `syncValue` first runs.

A class-based modifier can do the same in `modify()`: create the instance on the first call, keep it in a field, and update it on later calls, like the chart modifier in [advanced-modifiers](advanced-modifiers.md).

Reference: [ember-modifier](https://github.com/ember-modifier/ember-modifier), [Ember Guides: Autotracking In-Depth](https://guides.emberjs.com/release/in-depth-topics/autotracking-in-depth/)
