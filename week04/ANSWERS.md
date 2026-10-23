# Week 4: Explain it

## 1. Array index as `:key` when the list is filtered

Vue uses the key to decide which existing component instance belongs to which item. With the index as key, position 0 is always "the same card", so after filtering, the card at position 0 is reused for a different event. Its local state (for example an open details panel) stays at that position, so the panel appears open on the wrong event. With `:key="e.id"`, the state follows the event, not the position.

## 2. Why `visible` is a computed and not a function in the template

A computed is cached: it recalculates only when something it reads (`search`, `category` or `events`) changes. A function called in the template runs again on every re-render of the component, and the template uses `visible` several times (the count, the `v-if` and the `v-for`), so it would run several times per render. When you type in the search box, `search` changes, so the computed recalculates once per keystroke, but any other re-render reuses the cached result.

## 3. Why `EventCard` emits `book` instead of changing `event.booked`

First, props are read-only: data flows down and events flow up, so the owner of the data (`App`) is the only place that changes it, and there is one source of truth. Second, the parent can apply rules the card knows nothing about (check that a seat is left, add the id to `myBookings`, save to `localStorage`). It also keeps the card reusable and easy to test, because the test only needs to check that `book` was emitted with the right id.

## 4. `const { booked } = reactive(event); booked++`

Destructuring copies the current value of `booked` (a number) into a plain local variable. That variable is no longer connected to the reactive proxy, so `booked++` changes only the local copy, and Vue never sees a change on the reactive object. To keep reactivity, change the property directly (`event.booked++`) or use `toRefs(event)` so `booked` stays a ref.

## 5. Where `myBookings` lives

It lives in `App.vue`, because `App` owns the events and is the common parent of everything that depends on it: the header count, `onBook`, and saving to `localStorage`. `EventCard` and `EventList` are children; cards are created and destroyed as the list is filtered, and siblings cannot share state with each other. Keeping the state in the lowest common owner means it is not lost when a card disappears and every part of the page reads the same data.