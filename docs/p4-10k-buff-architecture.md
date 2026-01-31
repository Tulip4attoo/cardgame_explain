# 10 Kings: Buff/Modifier Architecture

Tài liệu này tổng kết quyết định kiến trúc cho hệ thống buff/modifier của 10 Kings, dựa trên phân tích so sánh với Balatro.

## Nguyên tắc chính

**Balatro-style hook system cho prep phase, ECS cho combat. Không ép mọi thứ vào components.**

Ranh giới rõ ràng:
- Trước combat → data-driven, phase-hooked (Balatro-style)
- Trong combat → ECS (miniplex)
- Convert deltas → components chỉ tại boundary giữa hai phase

## Tại sao không full ECS cho modifiers?

- Hooks xảy ra ở vài phase cố định, không phải per-frame
- State sống trên card/modifier instance (giống Ice Cream giữ `chips` trên `self.ability.extra`), mutate tại hook time
- Component graph thêm boilerplate mà không có lợi ích mới

## Tại sao không pure Balatro-style cho mọi thứ?

- Facility buff có spatial logic (pattern 1-1, row, column, diagonal, L-shape)
- Module có thể thay đổi pattern → cần composition, không hardcode if-else được
- Nhiều buff lifetime khác nhau (this-turn, this-ante, permanent, conditional)
- Output không quy về 1 table số đơn giản như Balatro

## Phân chia theo hệ thống

| Hệ thống | Approach | Lý do |
|-----------|----------|-------|
| **Run Modifiers** | Balatro-style (hook + data) | Global rule, output là số/flag, giống Joker |
| **Facility Buffs** | Data-driven pipeline | Spatial pattern + module composition |
| **Equipment/Module** | Data-driven attachment blueprints | Add/override components khi slotted, composable |
| **Discard Effects** | Balatro-style | Temporary trigger → output số |
| **Combat** | ECS (miniplex) | Real-time simulation, nhiều entity + behavior |

## Core Types

```typescript
type Context = {
  deck: DeckState;
  baseGrid: Grid;
  battleGrid: Grid;
  rng: Rng;
};

type BuffDelta = {
  mult?: number;
  xMult?: number;
  chips?: number;
  gold?: number;
  addComponent?: ComponentSpec[];
};

interface Modifier {
  key: string;
  state: Record<string, any>;
  onSetup?(ctx: Context): BuffDelta | void;
  onResolveBuffs?(ctx: Context): BuffDelta | void;
  onAfterCombat?(ctx: Context): BuffDelta | void;
}

function applyPhase(
  mods: Modifier[],
  hook: keyof Modifier,
  ctx: Context
): BuffDelta[] {
  return mods.flatMap(m => m[hook]?.(ctx) ?? []);
}
```

## Flow trong game loop

```
1. Draw / Placement
      ↓
2. Run onSetup + onResolveBuffs cho tất cả RunModifiers + Attachments
      → accumulate BuffDelta[]
      ↓
3. Convert deltas → ECS components trên units/facilities
      (AttackMod, HpMod, AuraEmitter, etc.)
      ↓
4. Combat chạy qua ECS (miniplex)
      ↓
5. Run onAfterCombat hooks
      → payouts, state decay, unlocks
```

## Facility Buff: Data-driven pattern

Spatial pattern là data, không phải if-else:

```typescript
type BuffPattern = (origin: GridPos) => GridPos[];

const PATTERNS = {
  single: (o) => [o],
  row:    (o) => allCellsInRow(o.row),
  column: (o) => allCellsInColumn(o.col),
  cross:  (o) => [...allCellsInRow(o.row), ...allCellsInColumn(o.col)],
  lShape: (o) => /* ... */,
};

// Module transform pattern
const modules = {
  "Range Expander": {
    transformPattern: (cells) => expandByOne(cells),
  },
  "Axis Flip": {
    transformPattern: (cells, origin) => flipRowToColumn(cells, origin),
  },
};
```

## Run Modifiers: Balatro-style

Mỗi modifier là data + hook function. State sống trên modifier instance.

```typescript
// modifiers/melee-supremacy.ts
export const meleeSupremacy: Modifier = {
  key: "melee_supremacy",
  state: {},
  onResolveBuffs(ctx) {
    // giống calculate_joker - check điều kiện, trả về số
    const meleeUnits = ctx.battleGrid.units.filter(u => u.tag === "melee");
    return meleeUnits.map(u => ({
      target: u,
      attackDamage: +5,
    }));
  },
};
```

## So với Balatro

| Aspect | Balatro | 10 Kings |
|--------|---------|----------|
| Modifier logic | 1 god function 1,770 dòng | Mỗi modifier 1 file riêng |
| Output | `{mult, xmult, chips}` | `BuffDelta` (richer, có `addComponent`) |
| Spatial | Không có | Data-driven pattern + module composition |
| State | `self.ability.extra` | `modifier.state` |
| Dispatch | if-else theo name | Hook system (`onSetup`, `onResolveBuffs`, ...) |
| Combat | Không có (chỉ scoring) | ECS riêng, nhận deltas từ prep phase |

Giữ sự đơn giản của Balatro (hook dispatcher + data), nhưng tách file cho readability và tách spatial logic ra pipeline riêng. Components chỉ xuất hiện khi cần: tại combat boundary hoặc khi attachment được slotted.
