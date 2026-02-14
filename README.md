-- 채팅 스팸 봇 + 토글 UI + 속도 조절 슬라이더 + UI 고정 버튼

-- "고정" 버튼 위치를 조금 아래로 이동 (원래 10 → 50으로 변경)



if game.CoreGui:FindFirstChild("ChatSpamUI") then

    game.CoreGui.ChatSpamUI:Destroy()

end



local TextChatService = game:GetService("TextChatService")

local Players = game:GetService("Players")

local RunService = game:GetService("RunService")

local UserInputService = game:GetService("UserInputService")



local player = Players.LocalPlayer

local textChannel = TextChatService.TextChannels:WaitForChild("RBXGeneral", 5)



if not textChannel then

    warn("RBXGeneral 채널을 찾을 수 없습니다.")

    return

end



local isSpamming = false

local spamConnection = nil

local messageIndex = 1



local baseInterval = 3.5

local speedMultiplier = 1 -- 0.1 \~ 5 배율



local messages = {

    "게임도 못하는데 왜캐 자존심은쌔 ㅋㅋ",

    "느그 보호자가",

    "니 먹여살릴려고",

    "CU에서 알바해서 돈 벌어오고",

    "엄마가 짜장면 살줄게\~ 이랬잖아 ㅋㅋ",

    "애🤓🤓:엄마 최고!!",

    "애🤓🤓:저도 커서 엄마 처럼 CU에서 알바할게요!!",

    "애 보호자:오구오구 우리 아들\~ 우리가 너무 자랑 스럽다\~",

    "엄마가 빨리 짜장면 사줄게~~ 오늘 우리 아들 생일 이잖아\~",

    "(짜장면이 1만 1천원임) 애 보호자:엄마가 다음에 더 돈벌어서 사줄게..미안해 ㅜㅜ",

    "그리고 애 집 반지하임 ㅋㅋㅋㅋㅋㅋ",

    "애집 비오면 물탱크 마냥 집이 수족관이됨 ㅋㅋㅋ",

    "애 좋은 글카 살돈이 없어서",

    "글카가 1000번대임 ㅋㅋㅋㅋ",

    "야 글카 1000번대로 게임이 돌아가냐?",

    "돌아가는것도 개신기 하노 ㅋㅋㅋ",

    "뭐라고? 돈이 없어서 pc살돈이 없다고?",

    "폰으로 게임이 돌아가냐?",

    "애 신고하면",

    "브레인롯 이벤트 못해서 울것노 ㅋㅋㅋ",

    "애:😡😡😡😡 신고 하지말라고!!!!!",

    "애🤓🤓:나 브훔 이벤트 해야한다고!!",

    "애🤓🤓:그리고 까불지 마라",

    "애🤓🤓:나 무려 고든학교 6학년 1진남이다!!!",

    "애🤓🤓:우리 아빠 경찰이다!!!",

    "놀리지 말라고! 경찰서 가고싶어??!?",

    "우리 아빠 진짜로 경찰서 서장이다!!!!",

    "애 친구가 없어서",

    "학교 뒤 자기 그림자랑 얘기함 ㅋㅋ",

    "애🤓🤓:나 오늘 친구들 한테 맞았다!",

    "난 맞는게 재밌더라 ㅎㅎ",

    "나 친구들 한테 맛있는 것도 사줬어 ㅎㅎ",

    "사실..뺐긴거야 ㅠㅠ 으에 😭😭😭",

    "나 사실 친구가 없어ㅓㅓㅠㅠㅠㅠㅠㅠ",

    "이거 빈말이 아니라 애가 진짜로",

    "학교에서 저런거 내가 다봄 ㅋㅋㅋㅋ",

    "으 냄새나 오지마",

    "꺄아악 오지마요 ㅠㅠㅠㅠ",

    "ㅅ..살ㄹ려주세요 ㅠㅠ 냄새나요 ㅠㅠㅠ",

    "너 머리 안싰지?",

    "냄새가 화면을 뚫고 나오노 ㅋㅋ",

    "겜도 못하고 키배도 못하고 ㅋㅋ",

    "오지마 ㅋㅋㅋ",

    "왜 지고 울분하는거야 ㅋㅋ",

    "말을 안하니깐",

    "내가 만만했냐?",

    "니가 세상에서 가장",

    "쎄다고 생각하는",

    "중2병은 아니겠지?",

    "애는 날 지적할게 없어서",

    "트집을 잡고",

    "반박할게 하나라서",

    "그거 딱 하나만 말하노 ㅋㅋ",

    "키배도 재미가 있어야 할만하지",

    "애는 어캐든 이길려고 지 할말만 하는게 참 ㅋㅋ",

    "너 보호자가 너 버렸잖아",

    "겜 실력도 딸리니깐 뇌도 같이 딸리냐?",

    "뭐가 잘났다고 그러세요 ㅋㅋ",

    "차단이라도 하게?",

    "에겐이세요?",

    "진짜 없으세요?",

    "그냥 말해본건데 진짠가여?",

    "그냥 해본말인데 왜캐 찔린애 마냥 그러세욬ㅋ",

    "그냥 빈말이였는데..진짜였다니 죄송합니다ㅋㅋㅋ",

    "애는 너무 쉽다 말도 못하고",

    "챗 속도도 지 뇌 발달하는 속도마냥",

    "개느리네 ㅋㅋ",

    "그냥",

    "접어주세요 ㅋㅋ",

    "말할 가치가 없노 ㅉ",

    "접어라 ㅋㅋㅋㅋ"

}



-- UI 생성

local gui = Instance.new("ScreenGui")

gui.Name = "ChatSpamUI"

gui.ResetOnSpawn = false

gui.Parent = game.CoreGui



local frame = Instance.new("Frame", gui)

frame.Size = UDim2.fromOffset(280, 260)

frame.Position = UDim2.fromScale(0.5, 0.1)

frame.AnchorPoint = Vector2.new(0.5, 0)

frame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)

frame.BorderSizePixel = 0

frame.Active = true

frame.Draggable = true

Instance.new("UICorner", frame).CornerRadius = UDim.new(0, 12)



local title = Instance.new("TextLabel", frame)

title.Size = UDim2.new(1, 0, 0, 40)

title.Text = "저능아 죽이는 자동챗"

title.Font = Enum.Font.GothamBold

title.TextSize = 22

title.TextColor3 = Color3.fromRGB(255, 180, 60)

title.BackgroundTransparency = 1



local statusLabel = Instance.new("TextLabel", frame)

statusLabel.Size = UDim2.new(1, -20, 0, 30)

statusLabel.Position = UDim2.fromOffset(10, 50)

statusLabel.Text = "상태: OFF"

statusLabel.Font = Enum.Font.GothamSemibold

statusLabel.TextSize = 18

statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)

statusLabel.BackgroundTransparency = 1



local toggleBtn = Instance.new("TextButton", frame)

toggleBtn.Size = UDim2.fromOffset(200, 50)

toggleBtn.Position = UDim2.fromOffset(40, 90)

toggleBtn.Text = "시작"

toggleBtn.Font = Enum.Font.GothamBold

toggleBtn.TextSize = 20

toggleBtn.TextColor3 = Color3.new(1,1,1)

toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 170, 80)

toggleBtn.BorderSizePixel = 0

Instance.new("UICorner", toggleBtn).CornerRadius = UDim.new(0, 10)



-- 고정 버튼 위치 아래로 이동 (Y축 50으로 변경)

local lockBtn = Instance.new("TextButton", frame)

lockBtn.Size = UDim2.fromOffset(60, 30)

lockBtn.Position = UDim2.fromOffset(200, 50) -- 여기서 10 → 50으로 변경

lockBtn.Text = "고정"

lockBtn.Font = Enum.Font.GothamBold

lockBtn.TextSize = 16

lockBtn.TextColor3 = Color3.new(1,1,1)

lockBtn.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

lockBtn.BorderSizePixel = 0

Instance.new("UICorner", lockBtn).CornerRadius = UDim.new(0, 8)



local isLocked = false

lockBtn.MouseButton1Click:Connect(function()

    isLocked = not isLocked

    frame.Draggable = not isLocked

    lockBtn.Text = isLocked and "고정됨" or "고정"

    lockBtn.BackgroundColor3 = isLocked and Color3.fromRGB(200, 80, 80) or Color3.fromRGB(100, 100, 100)

end)



-- 속도 조절 슬라이더

local sliderFrame = Instance.new("Frame", frame)

sliderFrame.Size = UDim2.fromOffset(220, 40)

sliderFrame.Position = UDim2.fromOffset(30, 150)

sliderFrame.BackgroundColor3 = Color3.fromRGB(50, 50, 50)

sliderFrame.BorderSizePixel = 0

Instance.new("UICorner", sliderFrame).CornerRadius = UDim.new(0, 8)



local sliderLabel = Instance.new("TextLabel", sliderFrame)

sliderLabel.Size = UDim2.new(1, 0, 0.5, 0)

sliderLabel.Text = "속도 배율: 1배 (3.5초)"

sliderLabel.Font = Enum.Font.Gotham

sliderLabel.TextSize = 16

sliderLabel.TextColor3 = Color3.new(1,1,1)

sliderLabel.BackgroundTransparency = 1



local sliderBar = Instance.new("Frame", sliderFrame)

sliderBar.Size = UDim2.new(1, -20, 0, 10)

sliderBar.Position = UDim2.fromOffset(10, 20)

sliderBar.BackgroundColor3 = Color3.fromRGB(100, 100, 100)

sliderBar.BorderSizePixel = 0

Instance.new("UICorner", sliderBar).CornerRadius = UDim.new(0, 5)



local sliderKnob = Instance.new("Frame", sliderBar)

sliderKnob.Size = UDim2.fromOffset(20, 20)

sliderKnob.Position = UDim2.new(0.2, -10, -0.5, 0)

sliderKnob.BackgroundColor3 = Color3.fromRGB(0, 170, 255)

sliderKnob.BorderSizePixel = 0

Instance.new("UICorner", sliderKnob).CornerRadius = UDim.new(0, 10)



-- 슬라이더 드래그 기능

local dragging = false

sliderKnob.InputBegan:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then

        dragging = true

    end

end)



UserInputService.InputEnded:Connect(function(input)

    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then

        dragging = false

    end

end)



RunService.RenderStepped:Connect(function()

    if dragging then

        local mousePos = UserInputService:GetMouseLocation()

        local relativeX = math.clamp((mousePos.X - sliderBar.AbsolutePosition.X) / sliderBar.AbsoluteSize.X, 0, 1)

        

        sliderKnob.Position = UDim2.new(relativeX, -10, -0.5, 0)

        

        speedMultiplier = 0.1 + (relativeX * 4.9)

        sliderLabel.Text = "속도 배율: " .. math.floor(speedMultiplier * 10)/10 .. "배 (" .. math.floor(baseInterval / speedMultiplier * 10)/10 .. "초)"

    end

end)



-- 토글 기능

toggleBtn.MouseButton1Click:Connect(function()

    isSpamming = not isSpamming

    

    if isSpamming then

        toggleBtn.Text = "중지"

        toggleBtn.BackgroundColor3 = Color3.fromRGB(200, 60, 60)

        statusLabel.Text = "상태: 실행 중"

        statusLabel.TextColor3 = Color3.fromRGB(0, 255, 100)

        

        spamConnection = RunService.Heartbeat:Connect(function()

            if isSpamming then

                local currentTime = tick()

                local effectiveInterval = baseInterval / speedMultiplier

                if not lastSpamTime or (currentTime - lastSpamTime) >= effectiveInterval then

                    local message = messages[messageIndex]

                    

                    pcall(function()

                        textChannel:SendAsync(message)

                    end)

                    

                    messageIndex = messageIndex + 1

                    if messageIndex > #messages then

                        messageIndex = 1

                    end

                    

                    lastSpamTime = currentTime

                end

            end

        end)

    else

        toggleBtn.Text = "시작"

        toggleBtn.BackgroundColor3 = Color3.fromRGB(50, 170, 80)

        statusLabel.Text = "상태: OFF"

        statusLabel.TextColor3 = Color3.fromRGB(200, 200, 200)

        

        if spamConnection then

            spamConnection:Disconnect()

            spamConnection = nil

        end

        lastSpamTime = nil

    end

end)



-- 전역 변수

local lastSpamTime = nil



-- GUI 닫을 때 정리

gui.Destroying:Connect(function()

    if spamConnection then

        spamConnection:Disconnect()

    end

end)
