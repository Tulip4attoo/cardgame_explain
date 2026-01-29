Ok, ta đã đi qua p1 về các mảnh ghép chính tạo nên source code Balatro. Phần này, mình muốn đi sâu hơn về How và Why - các mảnh ghép đó vận hành cùng nhau thế nào và tại sao nó vẫn hoạt động tốt, đặc biệt lại không phải là 1 lựa chọn tệ trong trường hợp phát triển Balatro.

# How – Những mảnh ghép đó vận hành cùng nhau ra sao?

## Monolith về state: mọi thứ đọc/ghi thông qua G

Toàn bộ game state nằm trong một biến global duy nhất G. Bất kỳ file nào cũng có thể đọc/ghi trực tiếp.

G được tạo một lần duy nhất:

``` lua
-- game.lua
Game = Object:extend()

function Game:init()
    G = self
    self:set_globals()
end

-- globals.lua (dòng cuối)
G = Game()
```

Mọi nơi đều có thể truy cập G trực tiếp:

``` lua
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
```

Không có getter/setter, không có encapsulation:

``` lua
-- Muốn biết người chơi có bao nhiêu tiền?
G.GAME.dollars

-- Muốn lấy joker đầu tiên?
G.jokers.cards[1]

-- Muốn check đang ở state nào?
G.STATE == G.STATES.SHOP

-- Muốn thay đổi? Gán thẳng:
G.GAME.dollars = G.GAME.dollars + 10
```

Điều này đi đôi với ưu điểm là truy cập các giá trị rất đơn giản, không cần truyền dependencies. Điều này khiến cho việc mod game hay tạo joker mới các thứ rất là đơn giản.

## Logic được tổ chức theo kiểu thủ tục tập trung

Thay vì phân tán logic vào nhiều class/module nhỏ, Balatro gom logic theo chức năng vào các "god functions".

Ví dụ điển hình vẫn là Tất cả joker effects nằm trong một function

```lua
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
```

Ở đây, cái function này sẽ check qua tất cả các case của context/game state/trường hợp trong game, xong rồi define nội dung logic hoạt động của Joker.

Về cơ bản thì các chức năng đều sẽ được quy về trong 1 function, và LocalThunk không hề ngần ngại việc tạo ra các function khổng lồ 1k - 2k lines. Tuy vậy, các function này viết rất đơn giản, vì thật sự cái flow và logic của game cũng vô cùng tự nhiên và dễ đoán - chính điều này mới là điều quan tọng giúp game vẫn được implement tốt dù cho dùng các function khổng lồ.

## File chỉ đóng vai trò "ngăn tủ" sắp xếp code

Các file trong Balatro không phải là modules độc lập. Chúng chỉ là cách tổ chức code cho dễ tìm.

Không có exports/imports:

``` lua
-- main.lua
require "game"
require "globals"
require "card"
require "functions/state_events"
require "functions/common_events"
```

Khi require, Lua thực thi file đó và mọi thứ không có local trở thành global. Không có module.exports, không có `import { x } from`.

Mọi file đều có thể gọi mọi thứ:

``` lua
-- Trong card.lua có thể gọi function từ common_events.lua
local effects = eval_card(self, context)

-- Trong state_events.lua có thể gọi method từ card.lua
scoring_hand[i]:get_chip_bonus()

-- Trong button_callbacks.lua có thể đọc state từ globals
if G.STATE == G.STATES.SHOP then

-- Tất cả đều có thể truy cập G
G.GAME.dollars = G.GAME.dollars + 10
```

So sánh với module thực sự:

``` lua
-- Nếu là module thực sự (Balatro KHÔNG làm thế này):
-- card.lua
local Card = {}
function Card:calculate_joker(context) ... end
return Card

-- state_events.lua
local Card = require("card")
local eval = require("common_events").eval_card
```

Balatro không làm vậy. Mọi thứ global, file chỉ là cách nhóm code lại.

## Cách vận hành

```
┌─────────────────────────────────────────────────────────┐
│                    Global G                             │
│  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐        │
│  │ G.GAME  │ │G.jokers │ │ G.hand  │ │G.STATE  │ ...    │
│  └─────────┘ └─────────┘ └─────────┘ └─────────┘        │
└─────────────────────────────────────────────────────────┘
        ↑ đọc/ghi           ↑ đọc/ghi           ↑ đọc/ghi
        │                   │                   │
┌───────┴───────┐   ┌───────┴────────┐   ┌──────┴─────────┐
│   card.lua    │   │state_events.lua│   │button_callbacks│
│               │   │                │   │                │
│calculate_joker│   │evaluate_play   │   │ G.FUNCS.xxx    │
│ (1,770 lines) │   │ (500 lines)    │   │                │
└───────────────┘   └────────────────┘   └────────────────┘
```

- State tập trung: G là single source of truth
- Logic tập trung: God functions xử lý từng mảng lớn
- Files phẳng: Không có hierarchy/module, chỉ là cách phân chia code


# Why – Tại sao kiến trúc này vẫn tạo ra một game tốt?

Trước hết, mình sẽ nói rõ quan điểm: với mình, Balatro là game được implement rất tốt:
+ anim rất đỉnh
+ hoạt động trơn tru dù synergy phức tạp
+ mình không chắc mình từng gặp bug nào trong quá trình chơi chưa
+ kể cả khi nhiều tính toán thì game vẫn chạy khá mượt
+ chưa kể còn tự viết shader

Nên là 1 người luôn coi sản phẩm cuối cùng >> các "best practice", mình thấy có nhiều điểm nên học hỏi từ kiến trúc này.

Balatro được làm bởi một người (LocalThunk) trong khoảng 3-4 năm. Với quy mô này, kiến trúc cần phục vụ một mục tiêu: ship game.

Rất khó để biết thật sự LocalThunk có suy nghĩ và định hướng gì về mặt code. Tuy vậy, ta có thể thấy 1 điều, những chỉnh sửa nếu cần sẽ rất nhanh. Vd như:

```lua
-- Muốn thêm Joker mới?
-- ->Mở card.lua, tìm calculate_joker, thêm block:

if self.ability.name == 'New Joker' then
    return { mult_mod = 10 }
end

-- Test ngay, thấy không cân bằng? Sửa 10 thành 8
-- Xong. Không cần tạo file mới, không cần register, không cần config.
```

So với kiến trúc "thường gặp":

```
Tạo file NewJoker.lua
→ Implement interface JokerEffect
→ Register vào JokerRegistry
→ Thêm vào config/jokers.json
→ Rebuild
→ Test
→ Sửa... lặp lại từ đầu
```

Tất nhiên nói vậy, với game dạng này thì ta follow Components over Inheritence thôi, cũng không làm như trên (hoặc làm cũng ok hehe).

Rất rõ ràng, đây là 1 cách làm hoạt động tốt với dự án 1 người. Khi bạn làm một mình và cần test 150 Jokers với hàng trăm synergies, việc giảm friction cho mỗi thay đổi cực kỳ quan trọng. Chưa kể, cách bạn viết code đương nhiên là align và context switch là không nhiều, do đó, các god function rất lớn cũng không quá là vấn đề.

Mình nghĩ cách làm này không work với dự án nhiều người tham gia, khi đó, việc tách biệt các phần để test và viết thêm/chỉnh sửa cho dễ quan trọng hơn. Khi đó, ta viết dạng module, và tất nhiên sẽ tăng thêm friction cho code.

## Mental model nằm trong đầu một người

LocalThunk không cần viết documentation để giải thích cho ai (dù mình nghĩ vẫn cần viết trong quá trình phát triển), không cần meeting để align về architecture (cái này quan trọng), toàn bộ game nằm trong đầu một người.

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

Khi toàn bộ code nằm trong đầu bạn, indirection khả năng là tốn thời gian của bạn hơn. Mỗi layer abstraction là một chỗ bạn phải nhảy qua khi debug/code/hình dung.

Rant: mie, vẫn không thể hiểu sao có những dev khen bọn langchain khi nó add 3 4 layer of abstraction cho 1 cái string operation???

## Ưu tiên đúng: gameplay > architecture

Balatro không bán được chục triệu bản vì code đẹp hay code xấu. Nó bán chạy vì:
- Gameplay loop addictive
- Joker synergies thú vị và surprising
- Balance tốt (hầu hết Jokers đều viable)
- Ít bugs ảnh hưởng gameplay
- Performance game tốt
- Hình ảnh và âm thanh rất có chất riêng

Những thứ này đến từ việc dễ iterate và dễ debug, không phải từ SOLID principles hay design patterns.

### Trade-off cho nhiều miếng ghép what + how ta vừa nói

| Hy sinh | Được |
| :--- | :--- |
| Code không theo chuẩn | Ship nhanh |
| Khó mở rộng team | Không cần team |
| LLM khó làm việc cùng | 2020-2023 chưa có LLM coding, mà chắc gì cần LLM? |

### Có cái nhìn bao quát nhanh hơn

Cá nhân mình cho rằng, look up table thì trông cool hơn, if-else trông nhà quê nhưng lại nhìn cái là hiểu ngay.

Vd như đoạn code này:

``` lua
    if self.base.value == '2' then self.base.nominal = 2; self.base.id = 2
    elseif self.base.value == '3' then self.base.nominal = 3; self.base.id = 3
    elseif self.base.value == '4' then self.base.nominal = 4; self.base.id = 4
    elseif self.base.value == '5' then self.base.nominal = 5; self.base.id = 5
    elseif self.base.value == '6' then self.base.nominal = 6; self.base.id = 6
    elseif self.base.value == '7' then self.base.nominal = 7; self.base.id = 7
    elseif self.base.value == '8' then self.base.nominal = 8; self.base.id = 8
    elseif self.base.value == '9' then self.base.nominal = 9; self.base.id = 9
    elseif self.base.value == '10' then self.base.nominal = 10; self.base.id = 10
    elseif self.base.value == 'Jack' then self.base.nominal = 10; self.base.face_nominal = 0.1; self.base.id = 11
    elseif self.base.value == 'Queen' then self.base.nominal = 10; self.base.face_nominal = 0.2; self.base.id = 12
    elseif self.base.value == 'King' then self.base.nominal = 10; self.base.face_nominal = 0.3; self.base.id = 13
    elseif self.base.value == 'Ace' then self.base.nominal = 11; self.base.face_nominal = 0.4; self.base.id = 14 end
```

Mình tin rằng kiểu viết này rất tường minh và đọc cái là check được luôn, giảm được cognitive load cho dev, 1 điều vô cùng quan trọng của những dự án phức tạp. Và debug, quá là đơn giản luôn.

# Kết luận

Cá nhân mình cho rằng, kiến trúc của Balatro không phải "best practice", mình không học tập hoàn toàn. Tuy vậy, nó là rất phù hợp cho context cụ thể:

- Một người làm
- Một sản phẩm cụ thể (không phải framework/platform)
- Giảm mental model và cognitive load
- Timeline là "xong khi nào xong", không phải sprint planning

Trong context này, God function + global state + if-else chains là công cụ phù hợp.

Bài học lớn hơn: Architecture phục vụ mục tiêu, không phải ngược lại. Balatro chứng minh rằng code không theo chuẩn vẫn có thể tạo ra sản phẩm xuất sắc, miễn là nó phù hợp với cách làm việc của team (dù team đó chỉ có một người).


# Suy nghĩ thêm - nếu dùng code Balatro với LLM

Mình dùng LLM để code khá nhiều, nên mình nghĩ thêm mục này là quan trọng với bản thân.

Ta so sánh 2 approaches - làm kiểu Balatro với làm kiểu Modular.

## Kiểu Balatro (monolithic if-else)

Với người:
- Mở file, Ctrl+F "Fibonacci" là thấy ngay logic
- Muốn sửa? Sửa tại chỗ là được
- Context trong đầu: "À cái này nằm trong calculate_joker"

Với LLM:
- "Hãy sửa Fibonacci Joker"
- LLM cần đọc calculate_joker (1,770 dòng) để hiểu context
- Hoặc LLM đoán mò → sai
- Context window bị chiếm ~15-20k tokens chỉ cho 1 function

## Kiểu modular (mỗi joker 1 file/function)

``` lua
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
```

Với người:
- Phải mở đúng file jokers/fibonacci.lua
- Cần hiểu system load jokers như nào
- Nhiều file hơn để navigate

Với LLM:
- "Hãy sửa Fibonacci Joker"
- LLM chỉ cần đọc 1 file nhỏ (~20 dòng)
- Context rõ ràng, ít token
- Dễ generate code mới theo pattern

## Trade-off thực tế


| Aspect | Monolithic (Balatro) | Modular |
| :--- | :--- | :--- |
| **Người đọc lần đầu** | Dễ - tất cả ở 1 chỗ | Khó - phải hiểu structure |
| **Người sửa nhanh** | Rất dễ - Ctrl+F | Phải tìm đúng file |
| **LLM đọc** | Bloat context | Nhẹ, focused |
| **LLM generate** | Khó - cần hiểu cả hệ thống | Dễ - theo template |
| **Thêm feature mới** | Copy-paste block | Tạo file mới theo pattern |

Tất nhiên, nếu tạo doc khéo thì vẫn giải quyết được thôi, nhưng mình vẫn cho rằng code kiểu Balatro không tối ưu cho làm việc với LLM. Ngoài ra, khả năng ngay từ ban đầu, LLM sẽ lái bạn ra 1 cái kiến trúc khác, không bao giờ có chuyện LLM dùng kiến trúc như Balatro. Lái LLM theo 1 hướng nó không chịu thì kể ra cũng hên xui đó.

Trong thời đại AI-assisted coding, kiến trúc modular có lợi thế mới:
- Smaller context = LLM hiểu tốt hơn
- Clear patterns = LLM generate chính xác hơn
- Isolated changes = Ít risk LLM phá code khác

Đây là góc nhìn mà 5 năm trước không ai nghĩ tới khi thiết kế architecture.

Tất nhiên, chắc gì đã dùng LLM để code, mà có dùng thì cứ vẽ kiến trúc rồi lái thôi, cũng phải lái xong mới biết được.