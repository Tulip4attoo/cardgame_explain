Đây là tổng quan chi tiết về cấu trúc codebase của Balatro:

---
1. Cấu trúc File/Folder của Balatro

Tổng quan

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

Thống kê Lines of Code
┌─────────────────────┬───────┬─────────┬──────┐
│        Layer        │ Files │  Lines  │  %   │
├─────────────────────┼───────┼─────────┼──────┤
│ Root (Game Objects) │ 11    │ ~12,500 │ 37%  │
├─────────────────────┼───────┼─────────┼──────┤
│ functions/          │ 6     │ ~16,300 │ 48%  │
├─────────────────────┼───────┼─────────┼──────┤
│ engine/             │ 15    │ ~4,900  │ 15%  │
├─────────────────────┼───────┼─────────┼──────┤
│ Total Lua           │ 32    │ ~33,700 │ 100% │
└─────────────────────┴───────┴─────────┴──────┘
---
Chi tiết từng folder

📁 Root - Game Object Classes

Các file core định nghĩa các "thực thể" trong game:
┌────────────────────┬───────┬───────────────────────────────────────────────────────────────────────────┐
│        File        │ Lines │                                   Mô tả                                   │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ card.lua           │ 4,771 │ Lớn nhất! Toàn bộ logic về Card - joker effects, scoring, editions        │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ game.lua           │ 3,629 │ Game state, initialization, tất cả data definitions (jokers, hands, etc.) │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ blind.lua          │ 751   │ Boss blind definitions & effects                                          │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ cardarea.lua       │ 668   │ Container quản lý nhóm cards (hand, deck, joker slots)                    │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ tag.lua            │ 595   │ Tag system (skip blind rewards)                                           │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ globals.lua        │ 522   │ Constants, colors, UI types                                               │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ main.lua           │ 388   │ LÖVE entry point, game loop                                               │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ back.lua           │ 288   │ Deck definitions (Red Deck, Blue Deck, etc.)                              │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ challenges.lua     │ 738   │ Challenge mode setups                                                     │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ card_character.lua │ 164   │ Card sprite/character handling                                            │
├────────────────────┼───────┼───────────────────────────────────────────────────────────────────────────┤
│ conf.lua           │ 11    │ LÖVE window config                                                        │
└────────────────────┴───────┴───────────────────────────────────────────────────────────────────────────┘
📁 functions/ - Game Logic

Các function xử lý gameplay, được tách riêng khỏi object definitions:
┌──────────────────────┬───────┬──────────────────────────────────────────────┐
│         File         │ Lines │                    Mô tả                     │
├──────────────────────┼───────┼──────────────────────────────────────────────┤
│ UI_definitions.lua   │ 6,436 │ Lớn nhất! Mọi UI layout - menus, HUD, popups │
├──────────────────────┼───────┼──────────────────────────────────────────────┤
│ button_callbacks.lua │ 3,203 │ Xử lý user interactions, button clicks       │
├──────────────────────┼───────┼──────────────────────────────────────────────┤
│ common_events.lua    │ 2,745 │ Card evaluation, shop logic, booster packs   │
├──────────────────────┼───────┼──────────────────────────────────────────────┤
│ misc_functions.lua   │ 2,022 │ Utilities, hand evaluation algorithm         │
├──────────────────────┼───────┼──────────────────────────────────────────────┤
│ state_events.lua     │ 1,642 │ Scoring flow, round progression              │
├──────────────────────┼───────┼──────────────────────────────────────────────┤
│ test_functions.lua   │ 237   │ Debug/test utilities                         │
└──────────────────────┴───────┴──────────────────────────────────────────────┘
📁 engine/ - Low-level Engine

Framework tự build, không phụ thuộc vào game logic:
┌────────────────────┬───────┬──────────────────────────────────────────────────┐
│        File        │ Lines │                      Mô tả                       │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ controller.lua     │ 1,382 │ Input handling (mouse, keyboard, gamepad, touch) │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ ui.lua             │ 1,054 │ UI rendering system                              │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ moveable.lua       │ 517   │ Animation & movement system                      │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ node.lua           │ 389   │ Scene graph / node tree                          │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ text.lua           │ 315   │ Text rendering                                   │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ sprite.lua         │ 216   │ Sprite rendering                                 │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ sound_manager.lua  │ 207   │ Audio system                                     │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ event.lua          │ 195   │ Event queue system                               │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ profile.lua        │ 188   │ Performance profiling                            │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ particles.lua      │ 177   │ Particle effects                                 │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ animatedsprite.lua │ 107   │ Animated sprites                                 │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ save_manager.lua   │ 84    │ Save/load system                                 │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ string_packer.lua  │ 72    │ String serialization                             │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ object.lua         │ 37    │ Base object class                                │
├────────────────────┼───────┼──────────────────────────────────────────────────┤
│ http_manager.lua   │ 23    │ HTTP requests (crash reports)                    │
└────────────────────┴───────┴──────────────────────────────────────────────────┘
📁 localization/ - Ngôn ngữ

15 ngôn ngữ: en-us, ja, ko, zh_CN, zh_TW, de, fr, es_ES, es_419, pt_BR, it, ru, pl, nl, id

📁 resources/ - Assets

resources/
├── textures/
│   ├── 1x/          # Standard resolution
│   └── 2x/          # High resolution (retina)
├── sounds/          # Audio files
├── fonts/           # m6x11plus.ttf (pixel font)
├── shaders/         # 19 shader effects
└── gamecontrollerdb.txt  # Controller mappings

Shaders đáng chú ý:
- CRT.fs - Hiệu ứng màn hình CRT
- hologram.fs, holo.fs - Holographic cards
- foil.fs, polychrome.fs - Card editions
- dissolve.fs - Card destruction effect
- negative.fs - Negative joker effect

---
Entry Point Flow

Từ main.lua:

-- 1. Load tất cả modules theo thứ tự
require "engine/object"
require "engine/controller"
require "back"
require "tag"
-- ... 25+ requires

-- 2. LÖVE callbacks
function love.load()
    G:start_up()        -- Initialize game
    -- Steam integration
end

function love.update(dt)
    G:update(dt)        -- Game loop
end

function love.draw()
    G:draw()            -- Render
end

---
Điểm đáng chú ý về kiến trúc

1. Monolithic nhưng có tổ chức: Không có framework phức tạp, nhưng tách biệt rõ ràng:
- engine/ = low-level, reusable
- functions/ = game-specific logic
- Root files = game objects/entities


Lua Global Environment - Nhiều file dùng chung một môi trường

Trong Lua, khi bạn require một file, nội dung của file đó được thực thi và mọi biến không có từ khóa local sẽ
trở thành global - có thể truy cập từ bất kỳ file nào khác.

main.lua require hơn 25 file:

-- main.lua
require "engine/object"
require "engine/controller"
require "back"
require "game"
require "globals"
require "card"
-- ...

Sau khi tất cả được load, mọi file đều có thể truy cập những gì file khác đã định nghĩa. Ví dụ card.lua có thể
dùng G mặc dù G được tạo trong globals.lua:

-- card.lua
function Card:calculate_joker(context)
    local joker = G.jokers.cards[1]  -- G từ file khác, dùng trực tiếp
end

Không cần import, không cần khai báo. Thứ tự require quan trọng - file nào cần dùng biến gì thì file định
nghĩa biến đó phải được require trước.

---
Biến G - Central State của game

Toàn bộ state của Balatro nằm trong một biến global duy nhất tên là G.

G được tạo như thế nào

-- game.lua (line 1-9)
Game = Object:extend()

function Game:init()
    G = self              -- G = chính instance này
    self:set_globals()
end

-- globals.lua (line 522)
G = Game()

Khi Game() được gọi, constructor gán G = self. Từ đó G là reference đến Game instance duy nhất.

G chứa gì

Sau set_globals(), G chứa:

Constants và configs:
G.STATES = { SELECTING_HAND = 1, SHOP = 5, GAME_OVER = 4, ... }
G.SETTINGS = { language = 'en-us', GAMESPEED = 1, ... }
G.C = { MULT = HEX('FE5F55'), CHIPS = HEX("009dff"), ... }  -- colors
G.handlist = { "Flush Five", "Flush House", ... }

Runtime state (khi đang chơi):
G.STATE         -- state hiện tại (đang chọn bài, đang ở shop, ...)
G.GAME          -- data của run hiện tại (chips, ante, round, hands, ...)
G.deck          -- CardArea: bộ bài
G.hand          -- CardArea: bài trên tay
G.play          -- CardArea: bài đang đánh
G.jokers        -- CardArea: jokers đang có
G.CONTROLLER    -- input handler

Ví dụ thực tế

Function xử lý khi người chơi đánh bài:

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

Trong function này truy cập: G.FUNCS, G.play.cards, G.GAME.hands, G.GAME.last_hand_played, G.jokers.cards,
G.GAME.chips. Tất cả qua một biến G.




God Functions trong Balatro

"God function" là những function xử lý quá nhiều logic, thường rất dài và chứa nhiều nhánh if/else. Balatro có
vài function như vậy.

1. Card:calculate_joker() - ~1,770 dòng

Function này xử lý toàn bộ effect của mọi Joker trong game. Cấu trúc của nó là một chuỗi if-else khổng lồ:

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

Mỗi Joker có logic riêng, và tất cả nằm trong một function duy nhất. Game có ~150 Jokers, mỗi cái có thể
trigger ở nhiều context khác nhau (selling, discarding, scoring, etc.).

2. eval_card() - Dispatcher function

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

Function này đóng vai trò dispatcher - gọi đúng method tùy theo card đang ở đâu.

3. G.FUNCS.evaluate_play() - Main scoring flow

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

---
Các game/system khác có dùng pattern này không?

Có, đặc biệt trong indie games và prototypes. Nhưng ở production code lớn hơn, người ta thường tránh.

Ví dụ tương tự

1. Dwarf Fortress (trước khi rewrite)
Game nổi tiếng với codebase "spaghetti" - một function xử lý combat có thể dài hàng ngàn dòng với mọi edge
case.

2. Early Unity games
Nhiều game Unity thời đầu có pattern "Manager" class với Update() function khổng lồ xử lý mọi thứ.

3. Game Jam projects
Khi thời gian hạn chế, developers thường dump mọi logic vào một file/function cho nhanh.

Cách các hệ thống lớn hơn tổ chức khác đi

Entity-Component-System (ECS):
Thay vì: Card:calculate_joker() chứa mọi joker logic
Dùng:    Mỗi Joker là một Component với method riêng
        System loop qua tất cả Components

Data-driven với scripting:
-- Thay vì hardcode trong calculate_joker:
if self.ability.name == 'Joker' then
    return { mult_mod = self.ability.mult }
end

-- Dùng data table:
joker_effects = {
    ['Joker'] = function(self, context)
        return { mult_mod = self.ability.mult }
    end,
    ['Greedy Joker'] = function(self, context)
        -- ...
    end,
}

-- Và gọi:
return joker_effects[self.ability.name](self, context)

Strategy Pattern (OOP):
class JokerEffect:
    def calculate(self, context): pass

class BasicJoker(JokerEffect):
    def calculate(self, context):
        return {'mult_mod': self.mult}

class GreedyJoker(JokerEffect):
    def calculate(self, context):
        # count diamonds...

Tại sao Balatro vẫn dùng god function?

1. Một developer - LocalThunk làm một mình, không cần coordinate với team
2. Không cần extensibility - Không có mod system chính thức (ban đầu), không cần plugin architecture
3. Dễ debug - Tất cả logic ở một chỗ, dễ trace
4. Performance - Không có overhead từ virtual dispatch, object creation
5. Đủ tốt - Game ship được, players không care code đẹp hay xấu



context là một Lua table được tạo tại chỗ mỗi khi gọi function, chứa thông tin về "đang xảy ra chuyện gì".
Không phải G.

---
Context là gì

Mỗi lần gọi calculate_joker, caller tạo một table mới với các field khác nhau:

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

-- Khi mở booster pack:
G.jokers.cards[i]:calculate_joker({
    open_booster = true,
    card = self
})

-- Khi mua card:
G.jokers.cards[i]:calculate_joker({
    buying_card = true,
    card = self
})

-- Khi bán card:
G.jokers.cards[i]:calculate_joker({
    selling_card = true,
    card = card
})

-- Khi reroll shop:
G.jokers.cards[i]:calculate_joker({
    reroll_shop = true
})

---
Các context flags phổ biến
┌───────────────────────┬─────────────────────────────────────┐
│         Flag          │                Nghĩa                │
├───────────────────────┼─────────────────────────────────────┤
│ context.joker_main    │ Đang trong phase scoring chính      │
├───────────────────────┼─────────────────────────────────────┤
│ context.before        │ Trước khi score hand                │
├───────────────────────┼─────────────────────────────────────┤
│ context.after         │ Sau khi score hand                  │
├───────────────────────┼─────────────────────────────────────┤
│ context.individual    │ Đang xét từng card riêng lẻ         │
├───────────────────────┼─────────────────────────────────────┤
│ context.repetition    │ Đang check lặp lại (Red Seal, etc.) │
├───────────────────────┼─────────────────────────────────────┤
│ context.discard       │ Đang discard cards                  │
├───────────────────────┼─────────────────────────────────────┤
│ context.end_of_round  │ Kết thúc round                      │
├───────────────────────┼─────────────────────────────────────┤
│ context.selling_self  │ Joker đang tự bán chính nó          │
├───────────────────────┼─────────────────────────────────────┤
│ context.selling_card  │ Đang bán một card khác              │
├───────────────────────┼─────────────────────────────────────┤
│ context.open_booster  │ Mở booster pack                     │
├───────────────────────┼─────────────────────────────────────┤
│ context.buying_card   │ Mua card                            │
├───────────────────────┼─────────────────────────────────────┤
│ context.setting_blind │ Chọn blind                          │
└───────────────────────┴─────────────────────────────────────┘
---
Trong calculate_joker kiểm tra context như thế nào

function Card:calculate_joker(context)
    -- Check context flags để biết đang ở phase nào

    if context.open_booster then
        -- Joker effects khi mở booster
        if self.ability.name == 'Hallucination' then
            -- Có chance tạo Tarot card
        end

    elseif context.selling_self then
        -- Joker effects khi bán chính nó
        if self.ability.name == 'Luchador' then
            G.GAME.blind:disable()
        end

    elseif context.discard then
        -- Joker effects khi discard
        if self.ability.name == 'Castle' then
            -- context.other_card là card đang bị discard
            if context.other_card:is_suit(G.GAME.current_round.castle_card.suit) then
                self.ability.extra.chips = self.ability.extra.chips + self.ability.extra.chip_mod
            end
        end

    elseif context.joker_main then
        -- Main scoring effects
        if self.ability.name == 'Joker' then
            return { mult_mod = self.ability.mult }
        end
    end
end

---
Tóm lại

- context là parameter truyền vào, không phải global
- Mỗi caller tạo một table mới với các field phù hợp
- calculate_joker dùng context.xxx để biết đang ở tình huống nào
- Đây là pattern "context object" - gom nhiều thông tin vào một table thay vì truyền nhiều parameters riêng lẻ

-- Thay vì:
calculate_joker(is_discard, is_scoring, other_card, full_hand, ...)

-- Dùng context table:
calculate_joker({ discard = true, other_card = card, full_hand = hand })


If-else dày đặc trong Balatro

Có thể chia thành vài loại:

---
1. If-else chain cho mapping/priority

Ví dụ xác định poker hand - check từ cao xuống thấp:

-- functions/state_events.lua (line 544-555)
if next(poker_hands["Flush Five"]) then text = "Flush Five"; scoring_hand = poker_hands["Flush Five"][1]
elseif next(poker_hands["Flush House"]) then text = "Flush House"; scoring_hand = poker_hands["Flush
House"][1]
elseif next(poker_hands["Five of a Kind"]) then text = "Five of a Kind"; scoring_hand = poker_hands["Five of a
Kind"][1]
elseif next(poker_hands["Straight Flush"]) then text = "Straight Flush"; scoring_hand = poker_hands["Straight
Flush"][1]
elseif next(poker_hands["Four of a Kind"]) then text = "Four of a Kind"; scoring_hand = poker_hands["Four of a
Kind"][1]
elseif next(poker_hands["Full House"]) then text = "Full House"; scoring_hand = poker_hands["Full House"][1]
elseif next(poker_hands["Flush"]) then text = "Flush"; scoring_hand = poker_hands["Flush"][1]
elseif next(poker_hands["Straight"]) then text = "Straight"; scoring_hand = poker_hands["Straight"][1]
elseif next(poker_hands["Three of a Kind"]) then text = "Three of a Kind"; scoring_hand = poker_hands["Three
of a Kind"][1]
elseif next(poker_hands["Two Pair"]) then text = "Two Pair"; scoring_hand = poker_hands["Two Pair"][1]
elseif next(poker_hands["Pair"]) then text = "Pair"; scoring_hand = poker_hands["Pair"][1]
elseif next(poker_hands["High Card"]) then text = "High Card"; scoring_hand = poker_hands["High Card"][1] end

12 hands, 12 elseif. Thứ tự quan trọng vì cần check hand mạnh nhất trước.

---
2. If-else cho special cases của từng card/joker

Trong set_ability(), mỗi Joker có kích thước đặc biệt:

-- card.lua (line 238-258)
if center.name == "Half Joker" then
    H = H/1.7              -- Joker này cao bằng nửa
end

if center.name == "Photograph" then
    H = H/1.2              -- Thấp hơn một chút
end

if center.name == "Square Joker" then
    H = W                  -- Hình vuông
end

if center.name == "Wee Joker" then
    H = H*0.7
    W = W*0.7              -- Nhỏ hơn cả chiều
end

---
3. If-else khổng lồ trong calculate_joker

Mỗi Joker có logic riêng, check từng cái một:

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

---
4. Nested if-else cho conditions phức tạp

-- Check card có phải face card không
if id == 11 or id == 12 or id == 13 or next(find_joker("Pareidolia")) then
    return true
end

-- Check có phải số Fibonacci không
if context.other_card:get_id() == 2 or
    context.other_card:get_id() == 3 or
    context.other_card:get_id() == 5 or
    context.other_card:get_id() == 8 or
    context.other_card:get_id() == 14 then

---
Cách khác để viết (mà Balatro không dùng)

Table lookup thay vì if-else chain:
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

Set thay vì nhiều or:
-- Thay vì:
if id == 2 or id == 3 or id == 5 or id == 8 or id == 14 then

-- Có thể dùng:
local fibonacci = {[2]=true, [3]=true, [5]=true, [8]=true, [14]=true}
if fibonacci[id] then

---
Tại sao Balatro giữ if-else

1. Đọc được ngay - Không cần trace vào table/function khác
2. Dễ thêm case mới - Copy-paste block, sửa tên và logic
3. Không có abstraction overhead - Mỗi Joker có logic khác nhau, khó abstract chung
4. Performance - If-else chain có thể nhanh hơn table lookup trong một số trường hợp (ít items, Lua JIT)

Với 150 Jokers mỗi cái có behavior riêng, việc tạo abstraction chung có thể phức tạp hơn là cứ if-else thẳng.


So sánh 2 approaches

Kiểu Balatro (monolithic if-else)

Với người:
- Mở file, Ctrl+F "Fibonacci", thấy ngay logic
- Muốn sửa? Sửa tại chỗ, xong
- Context trong đầu: "À cái này nằm trong calculate_joker"

Với LLM:
- "Hãy sửa Fibonacci Joker"
- LLM cần đọc calculate_joker (1,770 dòng) để hiểu context
- Hoặc LLM đoán mò → sai
- Context window bị chiếm ~15-20k tokens chỉ cho 1 function

---
Kiểu modular (mỗi joker 1 file/function)

-- jokers/fibonacci.lua
return {
    name = "Fibonacci",
    calculate = function(self, context)
        if context.individual and context.cardarea == G.play then
            local dominated_fibos = {[2]=true, [3]=true, [5]=true, [8]=true, [14]=true}
            if dominated_fibos[context.other_card:get_id()] then
                return { mult = self.ability.extra }
            end
        end
    end
}

Với người:
- Phải mở đúng file jokers/fibonacci.lua
- Cần hiểu system load jokers như nào
- Nhiều file hơn để navigate

Với LLM:
- "Hãy sửa Fibonacci Joker"
- LLM chỉ cần đọc 1 file nhỏ (~20 dòng)
- Context rõ ràng, ít token
- Dễ generate code mới theo pattern

---
Trade-off thực tế
┌───────────────────┬────────────────────────────┬───────────────────────────┐
│      Aspect       │    Monolithic (Balatro)    │          Modular          │
├───────────────────┼────────────────────────────┼───────────────────────────┤
│ Người đọc lần đầu │ Dễ - tất cả ở 1 chỗ        │ Khó - phải hiểu structure │
├───────────────────┼────────────────────────────┼───────────────────────────┤
│ Người sửa nhanh   │ Rất dễ - Ctrl+F            │ Phải tìm đúng file        │
├───────────────────┼────────────────────────────┼───────────────────────────┤
│ LLM đọc           │ Bloat context              │ Nhẹ, focused              │
├───────────────────┼────────────────────────────┼───────────────────────────┤
│ LLM generate      │ Khó - cần hiểu cả hệ thống │ Dễ - theo template        │
├───────────────────┼────────────────────────────┼───────────────────────────┤
│ Thêm feature mới  │ Copy-paste block           │ Tạo file mới theo pattern │
└───────────────────┴────────────────────────────┴───────────────────────────┘
---
Vấn đề thực tế khi dùng LLM với code Balatro

Nếu muốn LLM giúp sửa/thêm Joker:

User: "Thêm Joker mới tên 'Prime Number' - +mult cho 2,3,5,7,11,13"

LLM cần biết:
1. Format của joker effect trong calculate_joker
2. Context.individual là gì
3. Cách return { mult = x }
4. Cách check card id

→ Phải feed ~2000 dòng code để LLM hiểu pattern
→ Hoặc feed ví dụ tương tự (Fibonacci) + giải thích

Với modular code:
User: "Thêm Joker mới theo pattern của fibonacci.lua"

LLM chỉ cần:
1. Đọc fibonacci.lua (20 dòng)
2. Copy, sửa tên và logic

---
Kết luận

Code của Balatro được optimize cho một người làm việc một mình, không có LLM, không có team.

Trong thời đại AI-assisted coding, kiến trúc modular có lợi thế mới:
- Smaller context = LLM hiểu tốt hơn
- Clear patterns = LLM generate chính xác hơn
- Isolated changes = Ít risk LLM phá code khác

Đây là góc nhìn mà 5 năm trước không ai nghĩ tới khi thiết kế architecture.



How – Những mảnh ghép đó vận hành cùng nhau ra sao?

---
1. Monolith về state: mọi thứ đọc/ghi thông qua G

Toàn bộ game state nằm trong một biến global duy nhất G. Bất kỳ file nào cũng có thể đọc/ghi trực tiếp.

G được tạo một lần duy nhất:
-- game.lua
Game = Object:extend()

function Game:init()
    G = self
    self:set_globals()
end

-- globals.lua (dòng cuối)
G = Game()

Mọi nơi đều truy cập G trực tiếp:
-- Trong state_events.lua - xử lý scoring
G.GAME.hands[text].played = G.GAME.hands[text].played + 1
G.GAME.chips = G.GAME.chips + score

-- Trong card.lua - xử lý joker effect
if #G.jokers.cards > 0 then
    local other_joker = G.jokers.cards[1]
end

-- Trong button_callbacks.lua - xử lý UI
G.STATE = G.STATES.SHOP
G.GAME.dollars = G.GAME.dollars - cost

-- Trong common_events.lua - tạo card mới
local card = create_card('Tarot', G.consumeables)
G.consumeables:emplace(card)

Không có getter/setter, không có encapsulation:
-- Muốn biết người chơi có bao nhiêu tiền?
G.GAME.dollars

-- Muốn lấy joker đầu tiên?
G.jokers.cards[1]

-- Muốn check đang ở state nào?
G.STATE == G.STATES.SHOP

-- Muốn thay đổi? Gán thẳng:
G.GAME.dollars = G.GAME.dollars + 10

Ưu điểm: đơn giản, không cần truyền dependencies. Nhược điểm: bất kỳ code nào cũng có thể sửa bất kỳ state
nào.

---
2. Logic được tổ chức theo kiểu thủ tục tập trung

Thay vì phân tán logic vào nhiều class/module nhỏ, Balatro gom logic theo chức năng vào các "god functions".

Ví dụ 1: Tất cả joker effects nằm trong một function

-- card.lua: Card:calculate_joker() ~1,770 dòng
function Card:calculate_joker(context)
    if self.ability.name == "Blueprint" then
        -- 30 dòng logic Blueprint
    end

    if self.ability.name == "Brainstorm" then
        -- 20 dòng logic Brainstorm
    end

    if context.open_booster then
        if self.ability.name == 'Hallucination' then ... end
    elseif context.selling_self then
        if self.ability.name == 'Luchador' then ... end
        if self.ability.name == 'Diet Cola' then ... end
    elseif context.discard then
        if self.ability.name == 'Ramen' then ... end
        if self.ability.name == 'Castle' then ... end
        -- 20+ jokers khác
    elseif context.joker_main then
        if self.ability.name == 'Joker' then ... end
        if self.ability.name == 'Greedy Joker' then ... end
        -- 100+ jokers khác
    end
end

150 jokers, tất cả logic nằm trong 1 function. Không có JokerStrategy interface, không có từng Joker class
riêng.

Ví dụ 2: Scoring flow nằm trong một function

-- state_events.lua: G.FUNCS.evaluate_play() ~500 dòng
G.FUNCS.evaluate_play = function(e)
    -- 1. Xác định hand type
    local text, disp_text, poker_hands, scoring_hand = G.FUNCS.get_poker_hand_info(G.play.cards)

    -- 2. Joker "before" effects
    for i=1, #G.jokers.cards do
        local effects = eval_card(G.jokers.cards[i], {before = true})
    end

    -- 3. Blind modification
    mult, hand_chips = G.GAME.blind:modify_hand(...)

    -- 4. Score từng card
    for i=1, #scoring_hand do
        local effects = eval_card(scoring_hand[i], {cardarea = G.play})
        if effects.chips then hand_chips = hand_chips + effects.chips end
        if effects.mult then mult = mult + effects.mult end
        if effects.x_mult then mult = mult * effects.x_mult end
    end

    -- 5. Main joker effects
    for i=1, #G.jokers.cards do
        local effects = eval_card(G.jokers.cards[i], {joker_main = true})
    end

    -- 6. Final calculation
    G.GAME.chips = G.GAME.chips + hand_chips * mult
end

Toàn bộ flow từ đánh bài → tính điểm nằm trong 1 function. Đọc từ trên xuống là hiểu flow.

Ví dụ 3: Hand ranking bằng if-else chain

-- state_events.lua
if next(poker_hands["Flush Five"]) then text = "Flush Five"
elseif next(poker_hands["Flush House"]) then text = "Flush House"
elseif next(poker_hands["Five of a Kind"]) then text = "Five of a Kind"
elseif next(poker_hands["Straight Flush"]) then text = "Straight Flush"
elseif next(poker_hands["Four of a Kind"]) then text = "Four of a Kind"
elseif next(poker_hands["Full House"]) then text = "Full House"
elseif next(poker_hands["Flush"]) then text = "Flush"
elseif next(poker_hands["Straight"]) then text = "Straight"
elseif next(poker_hands["Three of a Kind"]) then text = "Three of a Kind"
elseif next(poker_hands["Two Pair"]) then text = "Two Pair"
elseif next(poker_hands["Pair"]) then text = "Pair"
elseif next(poker_hands["High Card"]) then text = "High Card" end

Không có priority table, không có HandRanking enum. 12 hands = 12 dòng if-else, thứ tự từ trên xuống =
priority.

---
3. File chỉ đóng vai trò "ngăn tủ" sắp xếp code

Các file trong Balatro không phải là modules độc lập. Chúng chỉ là cách tổ chức code cho dễ tìm.

Không có exports/imports:
-- main.lua
require "game"
require "globals"
require "card"
require "functions/state_events"
require "functions/common_events"

Khi require, Lua thực thi file đó và mọi thứ không có local trở thành global. Không có module.exports, không
có import { x } from.

File chứa gì thì tên nói lên:
┌────────────────────────────────┬──────────────────────────────────────────────┐
│              File              │                   Chứa gì                    │
├────────────────────────────────┼──────────────────────────────────────────────┤
│ card.lua                       │ Class Card và tất cả methods của nó          │
├────────────────────────────────┼──────────────────────────────────────────────┤
│ game.lua                       │ Class Game, data definitions, initialization │
├────────────────────────────────┼──────────────────────────────────────────────┤
│ globals.lua                    │ Function set_globals() và khởi tạo G         │
├────────────────────────────────┼──────────────────────────────────────────────┤
│ functions/state_events.lua     │ Các function xử lý game states               │
├────────────────────────────────┼──────────────────────────────────────────────┤
│ functions/common_events.lua    │ Các function dùng chung (eval_card, etc.)    │
├────────────────────────────────┼──────────────────────────────────────────────┤
│ functions/button_callbacks.lua │ Xử lý khi user click buttons                 │
└────────────────────────────────┴──────────────────────────────────────────────┘
Mọi file đều có thể gọi mọi thứ:

-- Trong card.lua có thể gọi function từ common_events.lua
local effects = eval_card(self, context)

-- Trong state_events.lua có thể gọi method từ card.lua
scoring_hand[i]:get_chip_bonus()

-- Trong button_callbacks.lua có thể đọc state từ globals
if G.STATE == G.STATES.SHOP then

-- Tất cả đều có thể truy cập G
G.GAME.dollars = G.GAME.dollars + 10

So sánh với module thực sự:

-- Nếu là module thực sự (Balatro KHÔNG làm thế này):
-- card.lua
local Card = {}
function Card:calculate_joker(context) ... end
return Card

-- state_events.lua
local Card = require("card")
local eval = require("common_events").eval_card

Balatro không làm vậy. Mọi thứ global, file chỉ là cách nhóm code lại.

---
Tóm lại cách vận hành

┌─────────────────────────────────────────────────────────┐
│                    Global G                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │
│  │ G.GAME  │ │G.jokers │ │ G.hand  │ │G.STATE  │ ...   │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘       │
└─────────────────────────────────────────────────────────┘
        ↑ đọc/ghi          ↑ đọc/ghi         ↑ đọc/ghi
        │                  │                 │
┌───────┴───────┐  ┌───────┴───────┐  ┌──────┴────────┐
│   card.lua    │  │state_events.lua│  │button_callbacks│
│               │  │               │  │               │
│calculate_joker│  │evaluate_play  │  │ G.FUNCS.xxx   │
│ (1,770 lines) │  │ (500 lines)   │  │               │
└───────────────┘  └───────────────┘  └───────────────┘

- State tập trung: G là single source of truth
- Logic tập trung: God functions xử lý từng mảng lớn
- Files phẳng: Không có hierarchy, chỉ là cách nhóm code


# 3. Why – Tại sao kiến trúc này vẫn tạo ra một game tốt?

---
Solo dev, product-led: iteration nhanh, dễ chỉnh sửa

Balatro được làm bởi một người (LocalThunk) trong khoảng 3-4 năm. Với quy mô này, kiến trúc cần phục vụ một
mục tiêu: ship game.

Iteration loop của solo dev:
Ý tưởng → Code → Test → Thấy không hay → Sửa → Test lại → OK → Tiếp

Với code kiểu Balatro, loop này rất ngắn:

-- Muốn thêm Joker mới?
-- Mở card.lua, tìm calculate_joker, thêm block:

if self.ability.name == 'New Joker' then
    return { mult_mod = 10 }
end

-- Test ngay, thấy không cân bằng? Sửa 10 thành 8
-- Xong. Không cần tạo file mới, không cần register, không cần config.

So với kiến trúc "chuẩn":
Tạo file NewJoker.lua
→ Implement interface JokerEffect
→ Register vào JokerRegistry
→ Thêm vào config/jokers.json
→ Rebuild
→ Test
→ Sửa... lặp lại từ đầu

Khi bạn làm một mình và cần test 150 Jokers với hàng trăm synergies, việc giảm friction cho mỗi thay đổi cực
kỳ quan trọng.

---
Mental model nằm trong đầu một người

LocalThunk không cần viết documentation để giải thích cho teammate. Không cần meeting để align về
architecture. Toàn bộ game nằm trong đầu một người.

Với mental model này:
- G là tất cả → không cần nhớ dependency injection
- calculate_joker chứa mọi Joker → không cần nhớ file nào ở đâu
- If-else rõ ràng → đọc là hiểu, không cần trace abstraction

Ví dụ debug:

Người chơi report: "Fibonacci Joker không hoạt động với Ace"

Solo dev với code Balatro:
1. Ctrl+F "Fibonacci" trong card.lua
2. Thấy: id == 2 or id == 3 or id == 5 or id == 8 or id == 14
3. À, Ace là 14, có rồi... check tiếp điều kiện khác
4. Tìm ra bug trong 5 phút

Solo dev với code modular + abstraction:
1. Tìm file fibonacci.lua
2. Thấy nó dùng CardUtils.isFibonacci(card)
3. Mở CardUtils, tìm isFibonacci
4. Thấy nó gọi card:get_id() rồi check
5. get_id() có edge case không? Mở Card class...
6. 20 phút sau mới hiểu flow

Khi toàn bộ code nằm trong đầu bạn, indirection là kẻ thù. Mỗi layer abstraction là một chỗ bạn phải nhảy qua
khi debug.

---
Ưu tiên đúng: gameplay > architecture

Balatro không bán được triệu bản vì code đẹp. Nó bán vì:
- Gameplay loop addictive
- Joker synergies thú vị và surprising
- Balance tốt (hầu hết Jokers đều viable)
- Ít bugs ảnh hưởng gameplay

Những thứ này đến từ việc dễ iterate và dễ debug, không phải từ SOLID principles hay design patterns.

Trade-off LocalThunk chọn:
┌─────────────────────────────┬──────────────────────────────────┐
│           Hy sinh           │               Được               │
├─────────────────────────────┼──────────────────────────────────┤
│ Code không "đẹp" theo chuẩn │ Ship nhanh                       │
├─────────────────────────────┼──────────────────────────────────┤
│ Khó mở rộng team            │ Không cần team                   │
├─────────────────────────────┼──────────────────────────────────┤
│ Khó làm mod system          │ Game hoàn chỉnh không cần mod    │
├─────────────────────────────┼──────────────────────────────────┤
│ LLM khó làm việc cùng       │ 2020-2023 chưa có LLM coding     │
├─────────────────────────────┼──────────────────────────────────┤
│ Technical debt              │ Game xong rồi, debt không matter │
└─────────────────────────────┴──────────────────────────────────┘
Về synergy và balance:

Việc tất cả Jokers nằm trong một function có lợi ích không ngờ: dễ thấy interaction.

-- Trong calculate_joker, scrolling qua thấy:

if self.ability.name == 'Blueprint' then
    -- Copy joker bên phải
    local other_joker_ret = other_joker:calculate_joker(context)
end

-- ... 500 dòng sau ...

if self.ability.name == 'Hologram' then
    -- +x_mult khi thêm playing card
    self.ability.x_mult = self.ability.x_mult + #context.cards * self.ability.extra
end

Khi đọc liên tục như này, dev tự nhiên thấy: "À Blueprint có thể copy Hologram effect không nhỉ?" → Test → Tìm
ra synergy/bug.

Với code modular, mỗi Joker isolated trong file riêng, khó thấy được big picture của interactions.

---
Kết luận

Kiến trúc của Balatro không phải "best practice". Nó là right practice cho context cụ thể:

- Một người làm
- Một sản phẩm cụ thể (không phải framework/platform)
- Mục tiêu là ship game hoàn chỉnh, không phải maintain 10 năm
- Timeline là "xong khi nào xong", không phải sprint planning

Trong context này, God function + global state + if-else chains là công cụ phù hợp.

Bài học lớn hơn: Architecture phục vụ mục tiêu, không phải ngược lại. Balatro chứng minh rằng code "xấu" theo
sách vẫn có thể tạo ra sản phẩm xuất sắc, miễn là nó phù hợp với cách làm việc của team (dù team đó chỉ có một
người).