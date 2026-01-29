Ok, mình đang muốn làm 1 game roguelike deckbuilder (+ autobattler), nên muốn tham khảo source code của Balatro để học hỏi. Với mình, Balatro là 1 game hay, có mức độ tương tác, synergy phức tạp giữa các thành phần (joker, enhancement, seal,...), có 1 bộ joker không hề ít (~200?), game chạy nhanh, mượt và hầu như không có bug.

Mình muốn viết 1 series về những thứ mình rút ra từ Balatro. Đây là bài đầu tiên, nói về tổng quan về code của Balatro.

# 1. Cấu trúc File/Folder của Balatro

Balatro là 1 project Lua, viết trên Love2d. Toàn bộ code được viết monolith trên 1 folder chung, với rất ít sub folder. Code có nhiều file không ngắn (3k-4k lines), không nhiều utils.

```
Balatro/
├── main.lua              # Entry point - LÖVE framework
├── conf.lua              # LÖVE configuration
├── game.lua              # ⭐ Core game logic (3,629 lines)
├── card.lua              # ⭐ Card system (4,771 lines)
├── globals.lua           # Global constants & configs
├── cardarea.lua          # Card container management
├── blind.lua             # Boss/blind definitions
├── back.lua              # Deck definitions
├── tag.lua               # Tag system
├── card_character.lua    # Card visual character
├── challenges.lua        # Challenge mode definitions
├── version.jkr           # Version file
│
├── engine/               # 🔧 Game engine layer
├── functions/            # 🎮 Game logic functions
├── localization/         # 🌍 Multi-language support
└── resources/            # 🎨 Assets (textures, sounds, shaders)
```

## Thống kê Lines of Code

| Layer | Files | Lines | % |
| :--- | :---: | :---: | :---: |
| Root (Game Objects) | 11 | ~12,500 | 37% |
| functions/ | 6 | ~16,300 | 48% |
| engine/ | 15 | ~4,900 | 15% |
| **Total Lua** | **32** | **~33,700** | **100%** |

## Chi tiết 1 số folder

### 📁 Root - Game Object Classes
*Các file core định nghĩa các entities trong game:*

| File | Lines | Mô tả |
| :--- | :---: | :--- |
| **card.lua** | 4,771 | **Lớn nhất!** Toàn bộ logic về Card - joker effects/cách tính toán để trigger joker các thứ, scoring, editions |
| **game.lua** | 3,629 | Game state, initialization, tất cả data definitions (jokers, hands, etc.) |
| blind.lua | 751 | Boss blind definitions & effects |
| cardarea.lua | 668 | Container quản lý nhóm cards (hand, deck, joker slots) |
| tag.lua | 595 | Tag system (skip blind rewards) |
| globals.lua | 522 | Constants, colors, UI types |
| main.lua | 388 | LÖVE entry point, game loop |
| back.lua | 288 | Deck definitions (Red Deck, Blue Deck, etc.) |

### 📁 functions/
*Các function xử lý state game/define UI cho game (nhưng không có card)*

| File | Lines | Mô tả |
| :--- | :---: | :--- |
| **UI_definitions.lua** | 6,436 | **Lớn nhất!** Mọi UI layout - menus, HUD, popups |
| button_callbacks.lua | 3,203 | Xử lý user interactions, button clicks |
| common_events.lua | 2,745 | Card evaluation, shop logic, booster packs |
| misc_functions.lua | 2,022 | Utilities, hand evaluation algorithm |
| state_events.lua | 1,642 | Scoring flow, round progression |
| test_functions.lua | 237 | Debug/test utilities |

### 📁 engine/ - Low-level Engine
*Framework tự build, không phụ thuộc vào game logic:*

| File | Lines | Mô tả |
| :--- | :---: | :--- |
| controller.lua | 1,382 | Input handling (mouse, keyboard, gamepad, touch) |
| ui.lua | 1,054 | UI rendering system |
| moveable.lua | 517 | Animation & movement system |
| node.lua | 389 | Scene graph / node tree |
| text.lua | 315 | Text rendering |

### 📁 resources/ - Assets

```text
resources/
├── textures/
│   ├── 1x/          # Standard resolution
│   └── 2x/          # High resolution (retina)
├── sounds/          # Audio files
├── fonts/           # m6x11plus.ttf (pixel font)
├── shaders/         # 19 shader effects
└── gamecontrollerdb.txt  # Controller mappings
```

**Shaders đáng chú ý:**
*   `CRT.fs` - Hiệu ứng màn hình CRT
*   `hologram.fs`, `holo.fs` - Holographic cards
*   `foil.fs`, `polychrome.fs` - Card editions
*   `dissolve.fs` - Card destruction effect
*   `negative.fs` - Negative joker effect

## Điểm đáng chú ý về kiến trúc

Balatro là 1 project monolithic nhưng có tổ chức: về cơ bản là không dùng framework phức tạp, các function tự define, tách biệt rõ ràng:
- Root files = game objects/entities
- engine/ = low-level, reusable
- functions/ = game-specific logic (về mặt game, còn logic của mỗi joker thì nằm ở `card.lua`)

# Lua Global Environment - Nhiều file dùng chung một môi trường

Trong Lua, khi bạn require một file, nội dung của file đó được thực thi và mọi biến không có từ khóa local sẽ trở thành global - có thể truy cập từ bất kỳ file nào khác.

`main.lua` require hơn 25 file:

```lua
require "engine/object"
require "engine/controller"
require "back"
require "game"
require "globals"
require "card"
...
```

Sau khi tất cả được load, mọi file đều có thể truy cập những gì file khác đã định nghĩa. Ví dụ `card.lua` có thể dùng G mặc dù G được tạo trong `globals.lua` mà không cần import, không cần khai báo. 

```lua
function Card:calculate_joker(context)
    local joker = G.jokers.cards[1]  -- G từ file khác, dùng trực tiếp
end
```

Chú ý, thứ tự require rất quan trọng ở đây - file nào cần dùng biến gì thì file định nghĩa biến đó phải được require trước, điều này thể hiện ở việc `global` được require trước `card`.

## Biến G - Central State của game

Toàn bộ state về data game của Balatro nằm trong một biến global duy nhất tên là G.

### G được tạo như thế nào?

``` lua
-- game.lua (line 1-9)
Game = Object:extend()

function Game:init()
    G = self              -- G = chính instance này
    self:set_globals()
end

-- globals.lua (line 522)
G = Game()
```

Khi `Game()` được gọi, constructor gán `G = self`. Từ đó G là reference đến Game instance duy nhất.

## G chứa gì?

Sau `set_globals()`, G chứa:

```lua
-- Constants và configs:
G.STATES = { SELECTING_HAND = 1, SHOP = 5, GAME_OVER = 4, ... }
G.SETTINGS = { language = 'en-us', GAMESPEED = 1, ... }
G.C = { MULT = HEX('FE5F55'), CHIPS = HEX("009dff"), ... }  -- colors
G.handlist = { "Flush Five", "Flush House", ... }

-- Runtime state (khi đang chơi):
G.STATE         -- state hiện tại (đang chọn bài, đang ở shop, ...)
G.GAME          -- data của run hiện tại (chips, ante, round, hands, ...)
G.deck          -- CardArea: bộ bài
G.hand          -- CardArea: bài trên tay
G.play          -- CardArea: bài đang đánh
G.jokers        -- CardArea: jokers đang có
G.CONTROLLER    -- input handler
```

Về cơ bản, nếu làm mode Balatro, ta sẽ chủ yếu thay đổi các giá trị của G.

## Ví dụ trong game

Function xử lý khi người chơi đánh bài:

``` lua
-- functions/state_events.lua (line 571+)
G.FUNCS.evaluate_play = function(e)
    -- Lấy cards từ G.play
    local text, disp_text, poker_hands, scoring_hand = G.FUNCS.get_poker_hand_info(G.play.cards)

    -- Cập nhật G.GAME
    G.GAME.hands[text].played = G.GAME.hands[text].played + 1
    G.GAME.last_hand_played = text

    -- Duyệt qua G.jokers
    for i = 1, #G.jokers.cards do
        local effects = eval_card(G.jokers.cards[i], {
            cardarea = G.jokers,
            full_hand = G.play.cards,
        })
    end

    -- Cập nhật điểm
    G.GAME.chips = G.GAME.chips + score
end
```

Trong function này,  truy cập: `G.FUNCS, G.play.cards, G.GAME.hands, G.GAME.last_hand_played, G.jokers.cards, G.GAME.chips`, tất cả đều phải qua một biến G.

# God Functions trong Balatro

"God function" là những function xử lý quá nhiều logic, thường rất dài và chứa nhiều nhánh if/else. Các function chính của Balatro đều là các functions như vậy.

## 1. Card:calculate_joker() - ~1,770 dòng

Function này xử lý toàn bộ effect của mọi Joker trong game. Cấu trúc của nó là một chuỗi if-else khổng lồ:

```lua
-- card.lua (line 2291-4063)
function Card:calculate_joker(context)
    if self.debuff then return nil end

    if self.ability.set == "Joker" and not self.debuff then
        -- Blueprint: copy joker bên phải
        if self.ability.name == "Blueprint" then
            local other_joker = nil
            for i = 1, #G.jokers.cards do
                if G.jokers.cards[i] == self then other_joker = G.jokers.cards[i+1] end
            end
            if other_joker and other_joker ~= self then
                local other_joker_ret = other_joker:calculate_joker(context)
                -- ...
            end
        end

        -- Brainstorm: copy joker đầu tiên
        if self.ability.name == "Brainstorm" then
            -- ...
        end

        -- Rồi check theo context
        if context.open_booster then
            if self.ability.name == 'Hallucination' then
                -- tạo Tarot card ngẫu nhiên
            end
        elseif context.selling_self then
            if self.ability.name == 'Luchador' then
                -- disable boss blind
            end
            if self.ability.name == 'Diet Cola' then
                -- thêm double tag
            end
            if self.ability.name == 'Invisible Joker' then
                -- duplicate random joker
            end
        elseif context.selling_card then
            if self.ability.name == 'Campfire' then
                -- tăng x_mult
            end
        elseif context.reroll_shop then
            if self.ability.name == 'Flash Card' then
                -- tăng mult
            end
        elseif context.discard then
            if self.ability.name == 'Ramen' then
                -- giảm x_mult, có thể tự hủy
            end
            if self.ability.name == 'Castle' then
                -- tăng chips
            end
            -- ... 20+ jokers khác
        elseif context.joker_main then
            -- MAIN SCORING - đây là phần lớn nhất
            if self.ability.name == 'Joker' then
                return { mult_mod = self.ability.mult }
            end
            if self.ability.name == 'Greedy Joker' then
                -- +mult cho mỗi Diamond
            end
            if self.ability.name == 'Lusty Joker' then
                -- +mult cho mỗi Heart
            end
            -- ... 100+ jokers khác
        end
    end
end
```

Mỗi Joker có logic riêng, và tất cả nằm trong một function duy nhất. Game có ~150 Jokers, mỗi cái có thể trigger ở nhiều context khác nhau (selling, discarding, scoring, etc.).

Về sau ta sẽ có mục riêng nói về các handle/define abilities/trigger joker, nhưng rất rõ ràng đây là 1 function khổng lồ và handle toàn bộ các case cho joker, chứ không hề tách ra làm bất kỳ hàm con nào.

## 2. eval_card() - Dispatcher function

```lua
-- functions/common_events.lua (line 580-656)
function eval_card(card, context)
    local ret = {}

    if context.cardarea == G.play then
        -- Card đang được chơi
        ret.chips = card:get_chip_bonus()
        ret.mult = card:get_chip_mult()
        ret.x_mult = card:get_chip_x_mult(context)
        ret.jokers = card:calculate_joker(context)
        ret.edition = card:get_edition(context)
    end

    if context.cardarea == G.hand then
        -- Card đang cầm trên tay
        ret.h_mult = card:get_chip_h_mult()
        ret.x_mult = card:get_chip_h_x_mult()
        ret.jokers = card:calculate_joker(context)
    end

    if context.cardarea == G.jokers then
        -- Joker slot
        ret.jokers = card:calculate_joker(context)
    end

    return ret
end
```

Function này đóng vai trò dispatcher - gọi đúng method tùy theo card đang ở đâu.

## 3. G.FUNCS.evaluate_play() - Main scoring flow

```lua
-- functions/state_events.lua (line 571-1066, ~500 dòng)
G.FUNCS.evaluate_play = function(e)
    -- 1. Xác định hand type
    local text, disp_text, poker_hands, scoring_hand = G.FUNCS.get_poker_hand_info(G.play.cards)

    -- 2. Joker "before" effects
    for i=1, #G.jokers.cards do
        local effects = eval_card(G.jokers.cards[i], {before = true, ...})
    end

    -- 3. Blind modification
    mult, hand_chips = G.GAME.blind:modify_hand(...)

    -- 4. Score từng card trong scoring hand
    for i=1, #scoring_hand do
        local effects = eval_card(scoring_hand[i], {cardarea = G.play, ...})
        if effects.chips then hand_chips = hand_chips + effects.chips end
        if effects.mult then mult = mult + effects.mult end
        if effects.x_mult then mult = mult * effects.x_mult end
    end

    -- 5. Cards held in hand effects
    for i=1, #G.hand.cards do
        local effects = eval_card(G.hand.cards[i], {cardarea = G.hand, ...})
    end

    -- 6. Main joker effects
    for i=1, #G.jokers.cards do
        local effects = eval_card(G.jokers.cards[i], {joker_main = true, ...})
        if effects.jokers.mult_mod then mult = mult + effects.jokers.mult_mod end
        if effects.jokers.Xmult_mod then mult = mult * effects.jokers.Xmult_mod end
    end

    -- 7. Final calculation
    chip_total = hand_chips * mult
    G.GAME.chips = G.GAME.chips + chip_total
end
```

## Các game/system khác có dùng pattern này không?

Cúng có 1 số game dùng các god function kiểu này, ví dụ như Dwarf Fortress (trước khi rewrite) - Game nổi tiếng với codebase "spaghetti" - một function xử lý combat có thể dài hàng ngàn dòng với mọi edge case.

## Cách các hệ thống lớn hơn tổ chức khác đi

+ Entity-Component-System (ECS):
    + Thay vì: `Card:calculate_joker()` chứa mọi joker logic
    + Dùng:
        + Mỗi Joker là một Component với method riêng
        + System loop qua tất cả Components

+ Data-driven với scripting:
    + Thay vì hardcode trong `calculate_joker`:

```
if self.ability.name == 'Joker' then
    return { mult_mod = self.ability.mult }
end
```

-> Dùng data table:

```
joker_effects = {
    ['Joker'] = function(self, context)
        return { mult_mod = self.ability.mult }
    end,
    ['Greedy Joker'] = function(self, context)
        -- ...
    end,
}
```
Và gọi:
```
return joker_effects[self.ability.name](self, context)
```

+ Strategy Pattern (OOP) (cách này dĩ nhiên là không phù hợp)
```
class JokerEffect:
    def calculate(self, context): pass

class BasicJoker(JokerEffect):
    def calculate(self, context):
        return {'mult_mod': self.mult}

class GreedyJoker(JokerEffect):
    def calculate(self, context):
        # count diamonds...
```

Chúng ta sẽ phỏng đoán về lý do Balatro dùng god function vào phần sau.

# Context - input của phần lớn các hàm trong Balatro

Rất nhiều hàm của Balatro có input là context, vd như:

```lua
function Card:get_chip_x_mult(context)

function Card:calculate_joker(context)

function eval_card(card, context)
```

Context là một Lua table được tạo tại chỗ, chứa thông tin về để cho vào function. Không phải và không liên quan đến G. Nói chung là dạng args thôi.

Ví dụ như:
```lua
-- Khi kết thúc round:
G.jokers.cards[i]:calculate_joker({
    end_of_round = true,
    game_over = game_over
})

-- Khi đang score một hand:
G.jokers.cards[k]:calculate_joker({
    cardarea = G.play,
    full_hand = G.play.cards,
    scoring_hand = scoring_hand,
    scoring_name = text,           -- "Pair", "Flush", etc.
    poker_hands = poker_hands,
    other_card = scoring_hand[i],
    individual = true
})

-- Khi discard:
G.jokers.cards[j]:calculate_joker({
    discard = true,
    other_card = G.hand.highlighted[i],
    full_hand = G.hand.highlighted
})
```

Đây là pattern "context object" - gom nhiều thông tin vào một table thay vì truyền nhiều parameters riêng lẻ:
+ Thay vì: `calculate_joker(is_discard, is_scoring, other_card, full_hand, ...)`
+ Thì ta dùng context table: `calculate_joker({ discard = true, other_card = card, full_hand = hand })`

# If-else xuất hiện dày đặc trong Balatro

Có 1 đặc trưng trong source code của Balatro là hệ thống If-Else xuất hiện rất dày đặc. Có thể chia thành vài loại:

## If-else chain cho mapping/priority

Ví dụ xác định poker hand - check từ cao xuống thấp:

```lua
-- functions/state_events.lua (line 544-555)
if next(poker_hands["Flush Five"]) then text = "Flush Five"; scoring_hand = poker_hands["Flush Five"][1]
elseif next(poker_hands["Flush House"]) then text = "Flush House"; scoring_hand = poker_hands["Flush House"][1]
elseif next(poker_hands["Five of a Kind"]) then text = "Five of a Kind"; scoring_hand = poker_hands["Five of a Kind"][1]
elseif next(poker_hands["Straight Flush"]) then text = "Straight Flush"; scoring_hand = poker_hands["Straight Flush"][1]
elseif next(poker_hands["Four of a Kind"]) then text = "Four of a Kind"; scoring_hand = poker_hands["Four of a Kind"][1]
elseif next(poker_hands["Full House"]) then text = "Full House"; scoring_hand = poker_hands["Full House"][1]
elseif next(poker_hands["Flush"]) then text = "Flush"; scoring_hand = poker_hands["Flush"][1]
elseif next(poker_hands["Straight"]) then text = "Straight"; scoring_hand = poker_hands["Straight"][1]
elseif next(poker_hands["Three of a Kind"]) then text = "Three of a Kind"; scoring_hand = poker_hands["Three of a Kind"][1]
elseif next(poker_hands["Two Pair"]) then text = "Two Pair"; scoring_hand = poker_hands["Two Pair"][1]
elseif next(poker_hands["Pair"]) then text = "Pair"; scoring_hand = poker_hands["Pair"][1]
elseif next(poker_hands["High Card"]) then text = "High Card"; scoring_hand = poker_hands["High Card"][1] end
```

Phần code đọc có vẻ lạ và hơi buồn cười, ít gặp, nhưng làm rất tốt nhiệm vụ được giao. 12 hands, 12 elseif. Thứ tự ở đây rất quan trọng vì cần check hand (được xếp hạng) mạnh nhất trước.

## Nested if-else cho conditions phức tạp

Những function như calculate_joker, mỗi Joker có logic riêng, cần check từng cái một. Hệ If-else ở đây có thể là 3-4-5 vòng if-else lồng nhau.

Đây chắc chắn là 1 practice/pattern không thường gặp, và thậm chí nhiều đồng chí (chả biết tọa ra được cái gì đánh chú ý) cười cợt trên reddit/X, nhưng it works, và thậm chí là works very well!

```lua
-- card.lua (line 3065-3200+)
if context.individual then
    if context.cardarea == G.play then

        if self.ability.name == 'Hiker' then
            -- Thêm permanent chips cho card
            context.other_card.ability.perma_bonus = context.other_card.ability.perma_bonus +
self.ability.extra
        end

        if self.ability.name == 'Lucky Cat' and context.other_card.lucky_trigger then
            -- Tăng x_mult khi Lucky Card trigger
            self.ability.x_mult = self.ability.x_mult + self.ability.extra
        end

        if self.ability.name == 'Photograph' then
            -- x_mult cho first face card
            if context.other_card == first_face then
                return { x_mult = self.ability.extra }
            end
        end

        if self.ability.name == 'Fibonacci' and (
            context.other_card:get_id() == 2 or
            context.other_card:get_id() == 3 or
            context.other_card:get_id() == 5 or
            context.other_card:get_id() == 8 or
            context.other_card:get_id() == 14) then
            -- +mult cho Ace, 2, 3, 5, 8 (Fibonacci numbers)
            return { mult = self.ability.extra }
        end

        if self.ability.name == 'Even Steven' and
            context.other_card:get_id() <= 10 and
            context.other_card:get_id() >= 0 and
            context.other_card:get_id() % 2 == 0 then
            -- +mult cho even cards
        end

        if self.ability.name == 'Odd Todd' and
            context.other_card:get_id() % 2 == 1 then
            -- +chips cho odd cards
        end

        -- ... 50+ jokers nữa
    end
end
```

Dĩ nhiên ta có thể dùng Table lookup thay vì if-else chain:
```lua
-- Thay vì:
if center.name == "Half Joker" then H = H/1.7 end
if center.name == "Square Joker" then H = W end
-- ...

-- Có thể dùng:
local size_modifiers = {
    ["Half Joker"] = function(W, H) return W, H/1.7 end,
    ["Square Joker"] = function(W, H) return W, W end,
    ["Wee Joker"] = function(W, H) return W*0.7, H*0.7 end,
}
if size_modifiers[center.name] then
    W, H = size_modifiers[center.name](W, H)
end
```

Set thay vì nhiều or:
```lua
-- Thay vì:
if id == 2 or id == 3 or id == 5 or id == 8 or id == 14 then

-- Có thể dùng:
local fibonacci = {[2]=true, [3]=true, [5]=true, [8]=true, [14]=true}
if fibonacci[id] then
```

Chúng ta sẽ bàn kỹ hơn về điều này ở phần sau. Mình dự định bài này đi hết 3 phần What - How - Why, nhưng có vẻ chỉ riêng What đã là khá dài rồi.