-- СЕРВИСЫ
local Players = game:GetService("Players")
local VirtualInputManager = game:GetService("VirtualInputManager")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- СОСТОЯНИЕ
local isClicking = false
local clickX = 500
local clickY = 500
local clickInterval = 0.1

-- СОЗДАНИЕ UI
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "MobileClickerUI"
screenGui.ResetOnSpawn = false
screenGui.Parent = playerGui

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 260, 0, 320)
mainFrame.Position = UDim2.new(0.5, -130, 0.5, -160)
mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 40)
mainFrame.BorderSizePixel = 0
mainFrame.Active = true
mainFrame.Parent = screenGui

local corner = Instance.new("UICorner", mainFrame)
corner.CornerRadius = UDim.new(0, 12)

-- ЗАГОЛОВОК И КНОПКА СВОРАЧИВАНИЯ
local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, -60, 0, 40)
title.Position = UDim2.new(0, 0, 0, 0)
title.BackgroundTransparency = 1
title.Text = "Кликер"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = mainFrame

local minimizeBtn = Instance.new("TextButton")
minimizeBtn.Size = UDim2.new(0, 35, 0, 35)
minimizeBtn.Position = UDim2.new(1, -40, 0, 5)
minimizeBtn.BackgroundColor3 = Color3.fromRGB(80, 80, 100)
minimizeBtn.Text = "—"
minimizeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minimizeBtn.TextScaled = true
minimizeBtn.Font = Enum.Font.GothamBold
minimizeBtn.Parent = mainFrame

local minimizeCorner = Instance.new("UICorner", minimizeBtn)
minimizeCorner.CornerRadius = UDim.new(0, 8)

-- КНОПКА ПОЛНОГО ЗАКРЫТИЯ
local closeBtn = Instance.new("TextButton")
closeBtn.Size = UDim2.new(0, 35, 0, 35)
closeBtn.Position = UDim2.new(1, -80, 0, 5)
closeBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
closeBtn.Text = "X"
closeBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
closeBtn.TextScaled = true
closeBtn.Font = Enum.Font.GothamBold
closeBtn.Parent = mainFrame

local closeCorner = Instance.new("UICorner", closeBtn)
closeCorner.CornerRadius = UDim.new(0, 8)

-- КОНТЕЙНЕР ДЛЯ ЭЛЕМЕНТОВ (скрывается при сворачивании)
local content = Instance.new("Frame")
content.Size = UDim2.new(1, 0, 1, -50)
content.Position = UDim2.new(0, 0, 0, 50)
content.BackgroundTransparency = 1
content.Parent = mainFrame

-- СТРЕЛКИ ДЛЯ ВЫБОРА ТОЧКИ КЛИКА
local arrowSize = UDim2.new(0, 40, 0, 40)

local upArrow = Instance.new("TextButton")
upArrow.Size = arrowSize
upArrow.Position = UDim2.new(0.5, -20, 0, 0)
upArrow.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
upArrow.Text = "▲"
upArrow.TextColor3 = Color3.fromRGB(255, 255, 255)
upArrow.TextScaled = true
upArrow.Parent = content
local upCorner = Instance.new("UICorner", upArrow)
upCorner.CornerRadius = UDim.new(0, 8)

local downArrow = Instance.new("TextButton")
downArrow.Size = arrowSize
downArrow.Position = UDim2.new(0.5, -20, 0, 80)
downArrow.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
downArrow.Text = "▼"
downArrow.TextColor3 = Color3.fromRGB(255, 255, 255)
downArrow.TextScaled = true
downArrow.Parent = content
local downCorner = Instance.new("UICorner", downArrow)
downCorner.CornerRadius = UDim.new(0, 8)

local leftArrow = Instance.new("TextButton")
leftArrow.Size = arrowSize
leftArrow.Position = UDim2.new(0, 10, 0, 40)
leftArrow.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
leftArrow.Text = "◀"
leftArrow.TextColor3 = Color3.fromRGB(255, 255, 255)
leftArrow.TextScaled = true
leftArrow.Parent = content
local leftCorner = Instance.new("UICorner", leftArrow)
leftCorner.CornerRadius = UDim.new(0, 8)

local rightArrow = Instance.new("TextButton")
rightArrow.Size = arrowSize
rightArrow.Position = UDim2.new(1, -50, 0, 40)
rightArrow.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
rightArrow.Text = "▶"
rightArrow.TextColor3 = Color3.fromRGB(255, 255, 255)
rightArrow.TextScaled = true
rightArrow.Parent = content
local rightCorner = Instance.new("UICorner", rightArrow)
rightCorner.CornerRadius = UDim.new(0, 8)

-- ОТОБРАЖЕНИЕ КООРДИНАТ
local coordsLabel = Instance.new("TextLabel")
coordsLabel.Size = UDim2.new(1, 0, 0, 30)
coordsLabel.Position = UDim2.new(0, 0, 0, 45)
coordsLabel.BackgroundTransparency = 1
coordsLabel.Text = "X: 500  Y: 500"
coordsLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
coordsLabel.TextScaled = true
coordsLabel.Font = Enum.Font.Gotham
coordsLabel.Parent = content

-- СЛАЙДЕР ДЛЯ ИНТЕРВАЛА
local intervalLabel = Instance.new("TextLabel")
intervalLabel.Size = UDim2.new(1, 0, 0, 25)
intervalLabel.Position = UDim2.new(0, 0, 0, 130)
intervalLabel.BackgroundTransparency = 1
intervalLabel.Text = "Интервал: 0.1 сек"
intervalLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
intervalLabel.TextScaled = true
intervalLabel.Font = Enum.Font.Gotham
intervalLabel.Parent = content

local intervalBack = Instance.new("Frame")
intervalBack.Size = UDim2.new(0.8, 0, 0, 30)
intervalBack.Position = UDim2.new(0.1, 0, 0, 160)
intervalBack.BackgroundColor3 = Color3.fromRGB(60, 60, 80)
intervalBack.Parent = content
local intervalCorner = Instance.new("UICorner", intervalBack)
intervalCorner.CornerRadius = UDim.new(0, 6)

local intervalFill = Instance.new("Frame")
intervalFill.Size = UDim2.new(0.1, 0, 1, 0)
intervalFill.BackgroundColor3 = Color3.fromRGB(80, 150, 255)
intervalFill.Parent = intervalBack
local fillCorner = Instance.new("UICorner", intervalFill)
fillCorner.CornerRadius = UDim.new(0, 6)

-- КНОПКИ СТАРТ/СТОП
local startBtn = Instance.new("TextButton")
startBtn.Size = UDim2.new(0.45, 0, 0, 45)
startBtn.Position = UDim2.new(0.03, 0, 1, -55)
startBtn.BackgroundColor3 = Color3.fromRGB(50, 180, 50)
startBtn.Text = "СТАРТ"
startBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
startBtn.TextScaled = true
startBtn.Font = Enum.Font.GothamBold
startBtn.Parent = content
local startCorner = Instance.new("UICorner", startBtn)
startCorner.CornerRadius = UDim.new(0, 8)

local stopBtn = Instance.new("TextButton")
stopBtn.Size = UDim2.new(0.45, 0, 0, 45)
stopBtn.Position = UDim2.new(0.52, 0, 1, -55)
stopBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
stopBtn.Text = "СТОП"
stopBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
stopBtn.TextScaled = true
stopBtn.Font = Enum.Font.GothamBold
stopBtn.Parent = content
local stopCorner = Instance.new("UICorner", stopBtn)
stopCorner.CornerRadius = UDim.new(0, 8)

-- ФУНКЦИЯ КЛИКА
local function performClick()
    if not isClicking then return end
    VirtualInputManager:SendMouseButtonEvent(clickX, clickY, 0, true, game, 0)
    task.wait(0.01)
    VirtualInputManager:SendMouseButtonEvent(clickX, clickY, 0, false, game, 0)
end

-- ЦИКЛ КЛИКОВ
local clickConnection
local function startClicking()
    isClicking = true
    startBtn.BackgroundColor3 = Color3.fromRGB(30, 100, 30)
    stopBtn.BackgroundColor3 = Color3.fromRGB(200, 50, 50)
    
    clickConnection = RunService.Heartbeat:Connect(function()
        if isClicking then
            task.spawn(performClick)
            task.wait(clickInterval)
        end
    end)
end

local function stopClicking()
    isClicking = false
    startBtn.BackgroundColor3 = Color3.fromRGB(50, 180, 50)
    stopBtn.BackgroundColor3 = Color3.fromRGB(150, 30, 30)
    
    if clickConnection then
        clickConnection:Disconnect()
        clickConnection = nil
    end
end

-- ОБРАБОТЧИКИ КНОПОК
startBtn.MouseButton1Click:Connect(function()
    if not isClicking then
        startClicking()
    end
end)

stopBtn.MouseButton1Click:Connect(function()
    stopClicking()
end)

-- СТРЕЛКИ
upArrow.MouseButton1Click:Connect(function()
    clickY = math.max(0, clickY - 10)
    coordsLabel.Text = "X: " .. clickX .. "  Y: " .. clickY
end)

downArrow.MouseButton1Click:Connect(function()
    clickY = clickY + 10
    coordsLabel.Text = "X: " .. clickX .. "  Y: " .. clickY
end)

leftArrow.MouseButton1Click:Connect(function()
    clickX = math.max(0, clickX - 10)
    coordsLabel.Text = "X: " .. clickX .. "  Y: " .. clickY
end)

rightArrow.MouseButton1Click:Connect(function()
    clickX = clickX + 10
    coordsLabel.Text = "X: " .. clickX .. "  Y: " .. clickY
end)

-- СЛАЙДЕР (перетаскивание)
local draggingSlider = false

local function updateSlider(input)
    local relX = input.Position.X - intervalBack.AbsolutePosition.X
    local percent = math.clamp(relX / intervalBack.AbsoluteSize.X, 0, 1)
    intervalFill.Size = UDim2.new(percent, 0, 1, 0)
    clickInterval = 0.01 + (percent * 0.99)
    intervalLabel.Text = string.format("Интервал: %.2f сек", clickInterval)
end

intervalBack.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = true
        updateSlider(input)
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingSlider and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        updateSlider(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingSlider = false
    end
end)

-- ПЕРЕТАСКИВАНИЕ МЕНЮ
local draggingUI = false
local dragStart
local startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingUI = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if draggingUI and (input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch) then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(
            startPos.X.Scale, startPos.X.Offset + delta.X,
            startPos.Y.Scale, startPos.Y.Offset + delta.Y
        )
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        draggingUI = false
    end
end)

-- СВОРАЧИВАНИЕ
local minimized = false
local originalSize = mainFrame.Size

minimizeBtn.MouseButton1Click:Connect(function()
    minimized = not minimized
    if minimized then
        content.Visible = false
        mainFrame.Size = UDim2.new(0, 260, 0, 45)
        minimizeBtn.Text = "+"
    else
        content.Visible = true
        mainFrame.Size = originalSize
        minimizeBtn.Text = "—"
    end
end)

-- ПОЛНОЕ ВЫКЛЮЧЕНИЕ
closeBtn.MouseButton1Click:Connect(function()
    stopClicking()
    screenGui:Destroy()
end)

-- АВТООСТАНОВКА ПРИ ВЫКЛЮЧЕНИИ
RunService.Heartbeat:Connect(function()
    if not screenGui.Parent then
        stopClicking()
    end
end)
