# Keyboard Shortcuts Implementation Plan

**Approach:** TDD - Types → Tests → Code

---

## Overview

Add vim-style keyboard shortcuts to Nila Code CLI input:

- **Option + Left/Right Arrow**: Navigate by word (vim `w`/`b` behavior)
- **Option + Delete**: Delete word backward (vim `db` behavior)
- **Ctrl + A**: Jump to line start (vim `0` behavior)

**Strategy:** Use `ink-text-input` for cursor management, layer custom shortcuts on top.

**Word Boundary Rules (Vim Behavior):**

Vim lowercase word definition:
- A word is either:
  - Alphanumeric + underscore sequence: `hello_world`, `test123`
  - OR a sequence of special characters: `---`, `===`, `(())`
- Whitespace always separates words

Examples:
- `hello-world` → 3 words: `hello`, `-`, `world`
- `foo_bar` → 1 word: `foo_bar`
- `test123` → 1 word: `test123`

---

## Phase 1: Type Definitions

**Create:** `src/utils/input-shortcuts.ts`

```typescript
export type DeletionResult = {
  newValue: string;
  newPosition: number;
};
```

**Completion Criteria:**

- [x] Type definitions exported
- [x] Types compile (`bun run typecheck`)

---

## Phase 2: Write Tests

### Utility Tests

**Create:** `tests/utils/input-shortcuts.test.ts`

Test cases:
- `findNextWordBoundary()`: next word, multiple spaces, end of string, special chars as words, underscore in words, grouped special chars, empty string
- `findPreviousWordBoundary()`: previous word, multiple spaces, start of string, special chars as words, underscore in words, grouped special chars
- `deleteWordBackward()`: delete word, partial word, at start, multiple spaces

### Component Tests

**Update:** `tests/components/Input.test.tsx`

Add test sections:
- Option + Right/Left Arrow navigation
- Option + Delete deletion
- Ctrl + A jump to start
- Backward compatibility (existing tests must pass)

**Completion Criteria:**

- [ ] Test files created
- [ ] Tests fail (red phase - no implementation yet)
- [ ] All edge cases covered

---

## Phase 3: Implementation

### Install Dependency

```bash
bun add ink-text-input@^6.0.0
```

### Word Boundary Utilities

**Implement:** `src/utils/input-shortcuts.ts`

```typescript
export function findNextWordBoundary(text: string, position: number): number {
  if (position >= text.length) return text.length;

  let i = position;
  const startChar = text[i];

  if (isWordChar(startChar)) {
    while (i < text.length && isWordChar(text[i])) i++;
  } else if (isSpecialChar(startChar)) {
    while (i < text.length && isSpecialChar(text[i])) i++;
  }

  while (i < text.length && /\s/.test(text[i])) i++;
  return i;
}

export function findPreviousWordBoundary(text: string, position: number): number {
  if (position <= 0) return 0;

  let i = position - 1;
  while (i > 0 && /\s/.test(text[i])) i--;
  if (i <= 0) return 0;

  const charType = isWordChar(text[i]) ? 'word' : isSpecialChar(text[i]) ? 'special' : 'space';

  if (charType === 'word') {
    while (i > 0 && isWordChar(text[i - 1])) i--;
  } else if (charType === 'special') {
    while (i > 0 && isSpecialChar(text[i - 1])) i--;
  }

  return i;
}

export function deleteWordBackward(text: string, position: number): DeletionResult {
  if (position === 0) {
    return { newValue: text, newPosition: 0 };
  }

  const wordStart = findPreviousWordBoundary(text, position);
  return {
    newValue: text.slice(0, wordStart) + text.slice(position),
    newPosition: wordStart
  };
}

function isWordChar(char: string): boolean {
  return /[a-zA-Z0-9_]/.test(char);
}

function isSpecialChar(char: string): boolean {
  return !/[a-zA-Z0-9_\s]/.test(char);
}
```

### Enhanced Input Component

**Replace:** `src/components/Input.tsx`

Key changes:
- Replace custom input with `TextInput` from `ink-text-input`
- Add `useInput` hook to intercept Option/Ctrl shortcuts
- Track cursor position for word navigation
- Maintain backward compatibility

**Critical:** Investigate `ink-text-input` v6.0.0 cursor control API. May need custom cursor tracking.

**Completion Criteria:**

- [ ] Dependency installed
- [ ] Utilities implemented
- [ ] All utility tests pass (green phase)
- [ ] Input component refactored
- [ ] Shortcuts integrated
- [ ] Existing tests still pass
- [ ] Type checking passes

---

## Phase 4: Verification

### Run Tests

```bash
bun test tests/utils/input-shortcuts.test.ts
bun test tests/components/Input.test.tsx
bun test
```

### Manual Testing

Create `src/examples/keyboard-shortcuts-demo.tsx` for manual verification.

Test in actual CLI:
1. Type "hello world test"
2. Option+Left to jump back by words
3. Option+Delete to delete "test"
4. Ctrl+A to jump to start
5. Option+Right to jump forward
6. Submit with Enter

**Completion Criteria:**

- [ ] All tests pass
- [ ] Manual demo works
- [ ] All shortcuts verified in CLI
- [ ] Edge cases tested
- [ ] Visual appearance unchanged

---

## Critical Files

### Create
- `src/utils/input-shortcuts.ts`
- `tests/utils/input-shortcuts.test.ts`
- `src/examples/keyboard-shortcuts-demo.tsx` (optional)

### Modify
- `src/components/Input.tsx`
- `tests/components/Input.test.tsx`
- `package.json`

### Preserve
- All existing tests must pass
- API unchanged (`onSubmit`, `disabled`)
- Visual appearance unchanged

---

## Risks

**ink-text-input cursor control:**
- May not expose controlled cursor position
- Mitigation: Research API first, fallback to custom cursor layer

**ANSI escape sequences:**
- Option+key sequences vary by terminal
- Mitigation: Test on Terminal, iTerm2, VSCode

**Breaking changes:**
- Changing Input could break tests
- Mitigation: Run tests after each change

---

## Final Checklist

- [ ] All new tests pass
- [ ] All existing tests pass
- [ ] Type checking passes
- [ ] Option+Arrow navigates by word
- [ ] Option+Delete deletes word backward
- [ ] Ctrl+A jumps to start
- [ ] Regular typing works
- [ ] Submit works
- [ ] Disabled state works
- [ ] Visual appearance unchanged
