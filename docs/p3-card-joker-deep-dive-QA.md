# Deep Dive: Hệ thống Card/Joker trong Balatro - Q&A

  Phân Tích Hệ Thống Card trong Balatro

  1. Cấu Trúc Dữ Liệu Chính

  Card Class (card.lua:1-77)

  Card = Moveable:extend()  -- Kế thừa từ Moveable (animation/positioning)

  Các thuộc tính quan trọng của một Card object:
  ┌────────────────────┬───────────────────────────────────────────────────┐
  │      Property      │                       Mô tả                       │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.config.center │ Reference đến định nghĩa trong P_CENTERS          │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.config.card   │ Reference đến P_CARDS (chỉ playing card)          │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.ability       │ State table - copy từ center + trạng thái mutable │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.base          │ Cho playing card: suit, value, nominal            │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.edition       │ Holo, Foil, Polychrome, Negative                  │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.seal          │ Red, Blue, Gold, Purple                           │
  ├────────────────────┼───────────────────────────────────────────────────┤
  │ self.playing_card  │ Flag phân biệt playing card vs special card       │
  └────────────────────┴───────────────────────────────────────────────────┘
  ---
  2. Hai Bảng Dữ Liệu Master (game.lua)

  P_CENTERS - Registry của TẤT CẢ card abilities

  P_CENTERS = {
    j_joker = {
      order = 1,
      name = "Joker",
      set = "Joker",           -- Phân loại: Joker/Tarot/Planet/Spectral/...
      rarity = 1,              -- 1=Common, 2=Uncommon, 3=Rare, 4=Legendary
      cost = 2,
      pos = {x=0, y=0},        -- Vị trí sprite
      config = { mult = 4 },   -- Parameters cho ability
      blueprint_compat = true,
      -- ...
    },
    c_fool = { set = "Tarot", consumeable = true, ... },
    c_mercury = { set = "Planet", config = {hand_type = "Pair"}, ... },
    -- ... ~200+ entries
  }

  P_CARDS - 52 lá bài thường

  P_CARDS = {
    H_2 = {name = "2 of Hearts", value = '2', suit = 'Hearts', pos = {x=0, y=0}},
    S_A = {name = "Ace of Spades", value = 'Ace', suit = 'Spades', pos = {x=12, y=3}},
    -- ... 52 cards
  }

  ---
  3. Cách Card Được Tạo Ra

  Playing Card → create_playing_card() (common_events.lua:1927)

  create_playing_card(card_init, area, skip_materialize, silent, colours)

  Flow:
  1. Random chọn từ P_CARDS (front)
  2. Dùng c_base làm center
  3. Tạo Card object với playing_card flag
  4. Thêm vào G.playing_cards array
  5. Emplace vào CardArea (deck/hand)

  Special Card (Joker/Tarot/Planet) → create_card() (common_events.lua:2082)

  create_card(_type, area, legendary, _rarity, skip_materialize, soulable, forced_key)

  Flow:
  1. Có thể force card cụ thể (forced_key)
  2. Hoặc random từ pool theo type + rarity
  3. Tạo Card object với center từ P_CENTERS
  4. Apply edition nếu có (holo/foil/polychrome/negative)
  5. Animation materialize

  ---
  4. Kiến Trúc 3 Lớp: Definition → Instance → State

  ┌─────────────────────────────────────────────────────────────────┐
  │                    P_CENTERS (Immutable)                        │
  │   j_joker = { config = { mult = 4 }, blueprint_compat = true }  │
  └─────────────────────────────────────────────────────────────────┘
                                ↓ reference
  ┌─────────────────────────────────────────────────────────────────┐
  │                    Card Object (Instance)                       │
  │   self.config.center ──→ P_CENTERS.j_joker                      │
  │   self.ability = { mult = 4, bonus = 0, ... }  ← Mutable copy   │
  └─────────────────────────────────────────────────────────────────┘

  Ý nghĩa:
  - P_CENTERS = Template (không đổi)
  - card.ability = State (có thể thay đổi trong game)

  Ví dụ: Joker có mult = 4 ban đầu, nhưng bonus có thể tăng qua Hiker effect.

  ---
  5. Phân Biệt Các Loại Card

  Theo ability.set:
  ┌────────────┬──────────────────────┬───────────────────────────────┐
  │    Set     │        Mô tả         │             Ví dụ             │
  ├────────────┼──────────────────────┼───────────────────────────────┤
  │ "Joker"    │ Joker cards          │ Joker, Blueprint, DNA         │
  ├────────────┼──────────────────────┼───────────────────────────────┤
  │ "Tarot"    │ Tarot consumeable    │ The Fool, Death, The Magician │
  ├────────────┼──────────────────────┼───────────────────────────────┤
  │ "Planet"   │ Planet consumeable   │ Mercury, Venus, Earth         │
  ├────────────┼──────────────────────┼───────────────────────────────┤
  │ "Spectral" │ Spectral consumeable │ Familiar, Grim, Incantation   │
  ├────────────┼──────────────────────┼───────────────────────────────┤
  │ "Voucher"  │ Permanent upgrades   │ Overstock, Clearance Sale     │
  ├────────────┼──────────────────────┼───────────────────────────────┤
  │ "Default"  │ Playing cards        │ 2♠, K♥, A♦                    │
  └────────────┴──────────────────────┴───────────────────────────────┘
  Theo flags:

  card.playing_card          -- != nil → là playing card
  card.ability.consumeable   -- true → là Tarot/Planet/Spectral (dùng 1 lần)

  ---
  6. Card:set_ability() - Khởi Tạo Ability (card.lua:223)

  function Card:set_ability(center, initial, delay_sprites)
      self.config.center = center

      -- Copy properties từ center.config vào self.ability
      self.ability = {
          name = center.name,
          effect = center.effect,
          set = center.set,
          mult = center.config.mult or 0,
          x_mult = center.config.Xmult or 1,
          extra = copy_table(center.config.extra),  -- Deep copy
          -- ...
      }

      -- Special cases: custom geometry
      if center.name == "Half Joker" then H = H/1.7 end
      if center.name == "Square Joker" then H = W end

      -- Card-specific initialization
      if center.name == "To Do List" then
          self.ability.to_do_poker_hand = pseudorandom_element(G.handlist)
      end
      if center.name == "Invisible Joker" then
          self.ability.invis_rounds = 0
      end
  end

  ---
  7. Card:calculate_joker(context) - Logic Effect (~1,770 dòng)

  Đây là god function xử lý toàn bộ Joker effects.

  Context là gì?

  -- Các context khác nhau trigger các effect khác nhau:
  {joker_main = true}              -- Main scoring phase
  {discard = true}                 -- Khi discard
  {selling_self = true}            -- Khi bán joker này
  {selling_card = true}            -- Khi bán card khác
  {open_booster = true}            -- Khi mở booster pack
  {individual = true}              -- Check từng card trong scoring hand
  {end_of_round = true}            -- Kết thúc round

  Cấu trúc function:

  function Card:calculate_joker(context)
      if self.debuff then return nil end

      -- Blueprint/Brainstorm: copy joker khác
      if self.ability.name == "Blueprint" then
          -- Tìm joker bên phải, gọi calculate_joker của nó
      end

      -- Dispatch theo context
      if context.open_booster then
          if self.ability.name == 'Hallucination' then
              -- Tạo Tarot card ngẫu nhiên
          end
      elseif context.selling_self then
          if self.ability.name == 'Luchador' then
              -- Disable boss blind
          end
          if self.ability.name == 'Diet Cola' then
              -- Thêm double tag
          end
      elseif context.discard then
          if self.ability.name == 'Ramen' then ... end
          if self.ability.name == 'Castle' then ... end
          -- ... 20+ jokers
      elseif context.joker_main then
          if self.ability.name == 'Joker' then
              return { mult_mod = self.ability.mult }
          end
          if self.ability.name == 'Greedy Joker' then
              -- Count diamonds, add mult
          end
          -- ... 100+ jokers
      end
  end

  Return value:

  {
      mult = 10,           -- Cộng mult
      mult_mod = 4,        -- Cộng mult (alias)
      Xmult_mod = 1.5,     -- Nhân mult
      x_mult = 2,          -- Nhân mult (alias)
      chips = 30,          -- Cộng chips
      message = "+4 Mult", -- Hiển thị UI
      colour = G.C.MULT,   -- Màu message
  }

  ---
  8. CardArea - Quản Lý Nhóm Cards (cardarea.lua)

  CardArea = Moveable:extend()

  -- Các loại area:
  G.deck          -- Bộ bài
  G.hand          -- Bài trên tay
  G.play          -- Bài đang đánh
  G.discard       -- Bài đã bỏ
  G.jokers        -- Joker slots
  G.consumeables  -- Tarot/Planet/Spectral slots
  G.shop          -- Shop area

  Key methods:

  CardArea:emplace(card, location, stay_flipped)  -- Thêm card
  CardArea:remove_card(card)                       -- Xóa card
  CardArea:shuffle(seed)                           -- Xáo bài
  CardArea:draw_to_hand()                          -- Rút bài

  ---
  9. So Sánh Joker vs Consumeable vs Playing Card
  ┌───────────────────┬────────────┬─────────────────────────────┬───────────────────────────┐
  │      Aspect       │   Joker    │         Consumeable         │       Playing Card        │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ set               │ "Joker"    │ "Tarot"/"Planet"/"Spectral" │ "Default"                 │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ consumeable flag  │ false      │ true                        │ false                     │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ Lifecycle         │ Persistent │ Single-use                  │ Persistent                │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ CardArea          │ G.jokers   │ G.consumeables              │ G.hand/G.deck             │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ P_CARDS ref       │ nil        │ nil                         │ Yes (suit/value)          │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ calculate_joker() │ Full logic │ Minimal                     │ Không dùng                │
  ├───────────────────┼────────────┼─────────────────────────────┼───────────────────────────┤
  │ Có base table?    │ Không      │ Không                       │ Có (suit, value, nominal) │
  └───────────────────┴────────────┴─────────────────────────────┴───────────────────────────┘
  ---
  10. Flow Tổng Thể: Từ Definition → Scoring

  1. Game Start
     └─ load P_CENTERS, P_CARDS vào memory

  2. Create Card
     └─ create_card("Joker") hoặc create_playing_card()
     └─ Card:init() → Card:set_ability() → copy ability từ P_CENTERS

  3. Card vào CardArea
     └─ CardArea:emplace(card) → thêm vào G.jokers / G.hand / etc.

  4. Player chơi bài
     └─ evaluate_play() (state_events.lua)
         ├─ get_poker_hand_info() → xác định hand type
         ├─ Loop: eval_card() cho từng scoring card
         ├─ Loop: eval_card() cho từng joker
         │       └─ Card:calculate_joker(context)
         │           └─ If-else chain → return {mult, chips, ...}
         └─ Tổng hợp score → G.GAME.chips

  ---
  Tóm Tắt

  Hệ thống Card của Balatro có thiết kế đơn giản nhưng hiệu quả:

  1. Data-driven: Mọi card được định nghĩa trong P_CENTERS/P_CARDS
  2. Definition + Instance: Template bất biến + state có thể thay đổi
  3. Context-based evaluation: Joker effects trigger dựa trên game context
  4. God function: Tất cả joker logic nằm trong calculate_joker() ~1,770 dòng
  5. CardArea containers: Quản lý nhóm card theo chức năng

  Bạn muốn tôi đi sâu hơn vào phần nào không? Ví dụ:
  - Chi tiết về một số Joker cụ thể?
  - Flow scoring chi tiết hơn?
  - Cách enhancement/edition hoạt động?

## Q: Cái `pos` trong định nghĩa joker ở P_CENTERS là gì?

Ví dụ:
```lua
j_joker = {order = 1, ..., pos = {x=0,y=0}, ...}
```

### A:

`pos = {x=0, y=0}` là **tọa độ sprite trên spritesheet**.

Balatro không lưu mỗi card một file ảnh riêng. Tất cả Joker sprites nằm trên **một tấm ảnh lớn** (`resources/textures/1x/Jokers.png`), và `pos` cho biết vị trí của sprite trong grid đó.

```
Spritesheet (Jokers.png):
┌─────┬─────┬─────┬─────┬─────┐
│x=0  │x=1  │x=2  │x=3  │x=4  │  y=0
│Joker│Greedy│Lusty│Wrathful│Gluttonous│
├─────┼─────┼─────┼─────┼─────┤
│x=0  │x=1  │x=2  │x=3  │x=4  │  y=1
│Jolly│Zany │Mad  │Crazy│Droll│
├─────┼─────┼─────┼─────┼─────┤
│...  │     │     │     │     │  y=2
```

Khi render, engine cắt ra ô tương ứng:
```lua
sprite_x = pos.x * CARD_WIDTH
sprite_y = pos.y * CARD_HEIGHT
```

Đây là kỹ thuật **texture atlas** — giảm số lần load texture, tăng performance khi render nhiều card cùng lúc.

---

## Q: `cost_mult` trong định nghĩa joker là gì?

### A:

`cost_mult` được định nghĩa cho mọi card trong P_CENTERS nhưng **không được sử dụng ở bất kỳ đâu khác** trong codebase — chỉ xuất hiện trong `game.lua` ở phần định nghĩa, và tất cả đều có giá trị `1.0`.

Nhiều khả năng đây là một **field dự phòng** (reserved/vestigial) — LocalThunk có thể đã plan cho cơ chế nhân giá card (ví dụ: một số joker hiếm có `cost_mult = 1.5` → giá bán/mua x1.5), nhưng cuối cùng không dùng đến hoặc xử lý giá theo cách khác.

---

## Q: Những cái `effect` như `effect = "Card Mult"` trong joker là gì? Có tác dụng gì không?

### A:

Có **hai nhóm** effect khác nhau:

**Nhóm được dùng trong logic** — enhancement effects của **playing card**:
- `"Stone Card"`, `"Lucky Card"`, `"Glass Card"`, `"Steel Card"`, `"Gold Card"`, `"Wild Card"`, `"Mult Card"` — các giá trị này được check bằng `self.ability.effect ==` trong `card.lua` và `common_events.lua`

**Nhóm KHÔNG được dùng trong logic** — effect trên **joker**:
- `"Card Mult"`, `"Random Mult"`, `"Credit"`, `"Type Mult"`, `"Hand Size Mult"`, `"Discard Chips"`, `"No Discard Mult"`, `"Suit Mult"`, ...

Các effect string trên joker không xuất hiện trong bất kỳ điều kiện hay logic nào. Toàn bộ joker logic trong `calculate_joker()` check bằng `self.ability.name`, không phải `self.ability.effect`.

Vậy với joker thì `effect` chỉ là **label mô tả** cho dev đọc — kiểu annotation/comment inline. Giống `cost_mult`, nó nằm đó nhưng code không bao giờ đọc nó.

---

## Q: Try-catch trong code có vẻ rất ít, ngược hẳn với code web?

### A:

Đúng. Toàn bộ codebase chỉ có **3 chỗ** dùng `pcall` (tương đương try-catch của Lua):

1. `main.lua:255` — set window mode (có thể fail tùy platform)
2. `misc_functions.lua:1745` — đọc localization string (có thể thiếu key)
3. `string_packer.lua:60` — decompress data (có thể corrupt)

Cả 3 đều ở **ranh giới với hệ thống bên ngoài** (OS window, file data, localization). Logic game thì hoàn toàn không có error handling.

Điều này hợp lý vì bản chất khác nhau:

- **Code web**: nhận input từ user/network/API — không kiểm soát được, mọi thứ có thể sai bất cứ lúc nào
- **Code game (single-player, offline)**: toàn bộ data do chính game tạo ra và kiểm soát — nếu `G.jokers.cards[1]` tồn tại thì nó chắc chắn là Card object hợp lệ, không cần check

Balatro không có system boundary nào đáng kể ngoài OS/file I/O. Mọi thứ trong game loop là internal state do game tự quản lý, nên không cần defensive coding.

---

## Q: Ví dụ "Ice Cream" joker (+100 Chips, -5 Chips mỗi hand), state chip hiện tại nằm ở đâu? Check ra sao?

### A:

State nằm ngay trong `self.ability.extra`:

**Definition** (game.lua:420):
```lua
j_ice_cream = { config = {extra = {chips = 100, chip_mod = 5}} }
```

Khi tạo card, `set_ability()` copy config vào `self.ability`:
```lua
self.ability.extra.chips = 100     -- state hiện tại, mutable
self.ability.extra.chip_mod = 5    -- hằng số: mỗi hand trừ bao nhiêu
```

Hai chỗ logic trong `calculate_joker()`:

**1. Scoring** (`card.lua:3915`) — context `joker_main`:
```lua
if self.ability.name == 'Ice Cream' then
    return {
        chip_mod = self.ability.extra.chips,  -- trả về chips hiện tại
    }
end
```

**2. Sau mỗi hand** (`card.lua:3571`) — context `after`:
```lua
if self.ability.name == 'Ice Cream' and not context.blueprint then
    if self.ability.extra.chips - self.ability.extra.chip_mod <= 0 then
        -- Chips sắp về 0 → destroy joker
        G.jokers:remove_card(self)
        self:remove()
    else
        -- Trừ chips
        self.ability.extra.chips = self.ability.extra.chips - self.ability.extra.chip_mod
    end
end
```

Kết quả:
```
Hand 1:  chips = 100 → score +100 → trừ 5 → còn 95
Hand 2:  chips = 95  → score +95  → trừ 5 → còn 90
...
Hand 20: chips = 5   → score +5   → trừ 5 → ≤ 0 → destroy
```

State nằm trực tiếp trên card object, tại `self.ability.extra.chips`. Không có store riêng, không có external tracking — card tự giữ state của mình.

---

## Q: Cách hoạt động của joker trong Balatro rất đơn giản, trực tiếp?

### A:

Đúng, flow chỉ có 3 bước:
1. **Define** ở P_CENTERS
2. **Tạo instance** — `Card:init()` gọi `Card:set_ability()`, copy config thành state riêng
3. **Khi có event** — `Card:calculate_joker(context)` check context, check name, chạy if-else, tính toán và trả về kết quả

`set_ability` được gọi ở:

| Chỗ gọi | Khi nào |
|----------|---------|
| `card.lua:43` — `Card:init()` | **Tạo card mới** — mọi card đều qua đây |
| `card.lua:1143` | **Dùng Tarot card** — enhance playing card |
| `card.lua:3448` | **Midas Mask joker** — biến card thành Gold Card |
| `card.lua:3471` | **Caino joker** — reset card về base |
| `common_events.lua:2158` | **Copy card** (vd: DNA joker duplicate) |

Chỗ chính là `Card:init()`. Các chỗ còn lại là thay đổi ability giữa game (enhance, transform). Không có registry pattern, không có event bus, không có middleware.

---

## Q: Cũng không có pattern kiểu Components over Inheritance? Cuối cùng vẫn quy về + mult bao nhiêu, x mult bao nhiêu?

### A:

Đúng. Không có component nào cả. Cái mà joker "trả về" khi trigger chỉ là:

```lua
return {
    mult_mod = 4,        -- +mult
    Xmult_mod = 1.5,     -- xmult
    chip_mod = 30,        -- +chips
    h_dollars = 3,        -- +tiền
    message = "+4 Mult",  -- hiển thị UI
}
```

Đó chỉ là **một table số**. Không có behavior, không có lifecycle, không có component nào attach vào card.

So sánh:

**Component pattern** (kiểu Unity/ECS):
```
Card
  ├── MultModifier component (+4 mult)
  ├── TriggerCondition component (on_discard, on_score...)
  ├── SelfDestruct component (khi hết charges)
  └── Animation component
```

**Balatro thực tế:**
```
Card
  └── ability table (mấy con số)
      └── calculate_joker() check tên → trả về mấy con số
```

Mọi joker dù phức tạp đến đâu, cuối cùng vẫn quy về **một thời điểm, trả về một table số**, rồi `evaluate_play()` cộng/nhân vào score. Không có component nào sống trên card, không có system nào tick qua component.

Cái duy nhất gần giống "state component" là `self.ability.extra` — nhưng nó cũng chỉ là mấy con số nằm im, chờ `calculate_joker()` đọc/ghi khi được gọi.

---

## Q: Cái return table số đó là ở đâu?

### A:

Đó là return value từ `Card:calculate_joker()`. Ví dụ:

```lua
-- card.lua:3915
if self.ability.name == 'Ice Cream' then
    return {
        chip_mod = self.ability.extra.chips,
        message = localize{...},
        colour = G.C.CHIPS
    }
end
```

Nơi **nhận** return này là `eval_card()` → rồi `evaluate_play()` dùng nó:

```lua
-- state_events.lua (scoring loop)
for i = 1, #G.jokers.cards do
    local effects = eval_card(G.jokers.cards[i], {joker_main = true, ...})
    --              ^^^^^^^^^ bên trong gọi card:calculate_joker()
    --              trả về table {mult_mod = 4, ...}

    if effects.jokers.mult_mod then mult = mult + effects.jokers.mult_mod end
    if effects.jokers.Xmult_mod then mult = mult * effects.jokers.Xmult_mod end
    if effects.jokers.chip_mod then hand_chips = hand_chips + effects.jokers.chip_mod end
end
```

Flow: `evaluate_play()` → `eval_card()` → `calculate_joker()` → return table số → cộng/nhân vào score.
