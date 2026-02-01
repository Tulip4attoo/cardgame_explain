
Nói về hệ thống card và joker

Tiếp theo bài viết p1 (https://tulip4attoo.github.io/balatro-p1/), mình muốn đào sâu hơn về cách Balatro tạo ra và sử dụng joker trong game của mình, về mặt code.

# Hệ thống card của Balatro



Trong Balatro, Card là 1 trong những cấu trúc dữ liệu chính. Card ngoài là Playing card, Joker, tarot card, còn là base cho voucher, booster.

Về cơ bản, các card được difine ở `game.lua`, trong đó Playing cards được define riêng, còn lại các card đặc biệt thì:

```lua
    self.P_CENTERS = {
        c_base={max = 500, freq = 1, line = 'base', name = "Default Base", pos = {x=1,y=0}, set = "Default", label = 'Base Card', effect = "Base", cost_mult = 1.0, config = {}},

        --Jokers
        j_joker=            {order = 1,  unlocked = true,   start_alerted = true, discovered = true,  blueprint_compat = true, perishable_compat = true, eternal_compat = true, rarity = 1, cost = 2, name = "Joker", pos = {x=0,y=0}, set = "Joker", effect = "Mult", cost_mult = 1.0, config = {mult = 4}},
        j_greedy_joker=     {order = 2,  unlocked = true,   discovered = false, blueprint_compat = true, perishable_compat = true, eternal_compat = true, rarity = 1, cost = 5, name = "Greedy Joker", pos = {x=6,y=1}, set = "Joker", effect = "Suit Mult", cost_mult = 1.0, config = {extra = {s_mult = 3, suit = 'Diamonds'}}},
        j_lusty_joker=      {order = 3,  unlocked = true,   discovered = false, blueprint_compat = true, perishable_compat = true, eternal_compat = true, rarity = 1, cost = 5, name = "Lusty Joker", pos = {x=7,y=1}, set = "Joker", effect = "Suit Mult", cost_mult = 1.0, config = {extra = {s_mult = 3, suit = 'Hearts'}}},
        j_wrathful_joker=   {order = 4,  unlocked = true,   discovered = false, blueprint_compat = true, perishable_compat = true, eternal_compat = true, rarity = 1, cost = 5, name = "Wrathful Joker", pos = {x=8,y=1}, set = "Joker", effect = "Suit Mult", cost_mult = 1.0, config = {extra = {s_mult = 3, suit = 'Spades'}}},
        j_gluttenous_joker= {order = 5,  unlocked = true,   discovered = false, blueprint_compat = true, perishable_compat = true, eternal_compat = true, rarity = 1, cost = 5, name = "Gluttonous Joker", pos = {x=9,y=1}, set = "Joker", effect = "Suit Mult", cost_mult = 1.0, config = {extra = {s_mult = 3, suit = 'Clubs'}}},
```

Mỗi joker, tarot, consumable, voucher,... đều được define ở `P_CENTERS`. Ta có thể dễ dàng đọc và hiểu phần lớn các option của mỗi cards. Chú ý có 1 số thứ không được dùng, vd như `cost_mult`, hoặc mang ý nghĩa về mặt giúp dễ phân loại trong quá trình code (như `effect = "Suit Mult"`).

Với Special Card (Joker/Tarot/Planet), flow tạo ra các card này là:

```lua
  create_card(_type, area, legendary, _rarity, skip_materialize, soulable, forced_key)
```

Flow:
  1. Có thể force card cụ thể (forced_key)
  2. Hoặc random từ pool theo type + rarity
  3. Tạo Card object với center từ P_CENTERS
  4. Apply edition nếu có (holo/foil/polychrome/negative)
  5. Animation materialize

# Cách tạo ra Joker instance

Về cơ bản thì cơ chế tạo các instance Joker là thế này:

Definition → Instance → State

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

Tại sao cần State? Vì có rất nhiều joker sẽ thay đổi trong quá trình chơi (vd như `Ceremonial Dagger` hay hàng loạt các lá bài khác).

Ta không đi quá sâu vào hệ thống cards, hay việc truy xuất type của card như thế nào (`ability.set`), mình thấy việc này không cần thiết lắm.

# CardArea - Quản Lý Nhóm Cards (cardarea.lua)

Cái này thật ra cũng không cần quá chú ý làm gì. Nhưng nếu như làm mod thì sẽ biết khá rõ, vì nó liên quan trực tiếp tới việc xử lý API những thành phần quan trọng

```lua
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
```

# Về logic effect của Joker

```lua
Card:calculate_joker(context) - Logic Effect (~1,770 dòng)
```

Ta đã nhắc khá nhiều về hàm này ở p1 và p2. Đây là god function xử lý toàn bộ Joker effects. Context là gì?

```lua
  -- Các context khác nhau trigger các effect khác nhau:
  {joker_main = true}              -- Main scoring phase
  {discard = true}                 -- Khi discard
  {selling_self = true}            -- Khi bán joker này
  {selling_card = true}            -- Khi bán card khác
  {open_booster = true}            -- Khi mở booster pack
  {individual = true}              -- Check từng card trong scoring hand
  {end_of_round = true}            -- Kết thúc round
```

Cấu trúc function:

```lua
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
```

Return value:

```lua
{
    mult = 10,           -- Cộng mult
    mult_mod = 4,        -- Cộng mult (alias)
    Xmult_mod = 1.5,     -- Nhân mult
    x_mult = 2,          -- Nhân mult (alias)
    chips = 30,          -- Cộng chips
    message = "+4 Mult", -- Hiển thị UI
    colour = G.C.MULT,   -- Màu message
}
```

# Flow Tổng Thể: Từ Definition → Scoring

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

Chú ý là cái logic và flow ở mục 4 đã rất là rõ ràng và tường minh. Điều này vô cùng quan trọng để tránh đi các vấn đề về loop hay lỗi logic khi xử lý score, trigger joker khi tính toán.

# Hệ thống được thiết kế đơn giản nhưng hiệu quả

1. Data-driven: Mọi card được định nghĩa trong P_CENTERS/P_CARDS
2. Definition + Instance: Template bất biến + state có thể thay đổi
3. Context-based evaluation: Joker effects trigger dựa trên game context
4. God function: Tất cả joker logic nằm trong calculate_joker() ~1,770 dòng
5. CardArea containers: Quản lý nhóm card theo chức năng

Chúng ta sẽ chú ý, ở đây, Balatro hoàn toàn không dùng pattern kiểu Components over Inheritance (nói chung là không dùng cả Inheritance và Components) trong việc tạo và kiểm soát Joker.

Cái mà joker "trả về" khi trigger chỉ là:

```lua
return {
    mult_mod = 4,        -- +mult
    Xmult_mod = 1.5,     -- xmult
    chip_mod = 30,        -- +chips
    h_dollars = 3,        -- +tiền
    message = "+4 Mult",  -- hiển thị UI
}
```

Đó chỉ là 1 table (dạng dạng dictionary trong python). Không có lớp lang, các attributes gì phức tạp, không có behavior, không có lifecycle, không có component nào attach vào card.

So sánh 1 cách đơn giản

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

Mọi joker dù phức tạp đến đâu, cuối cùng vẫn quy về **một thời điểm, trả về một table số**, rồi `evaluate_play()` cộng/nhân vào score. Không có component nào attach vào card, không có system nào tick qua component.