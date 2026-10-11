-- Auto Clicker (меню для телефона)
-- Старт/Стоп, точка клика двигается стрелками (удержание = непрерывно),
-- интервал между кликами, перетаскивание и сворачивание меню, полное выключение.

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local VIM = game:GetService("VirtualInputManager")
local player = Players.LocalPlayer

-- ===== Состояние =====
local alive = true
local running = false
local interval = 0.10          -- секунды между кликами
local steps = { 2, 10, 40 }    -- варианты шага стрелок (пиксели)
local stepIdx = 2
local useTouch = false         -- false = мышь, true = касание
local clicks = 0
local connections = {}

local cam = workspace.CurrentCamera
local vp = cam and cam.ViewportSize or Vector2.new(800, 400)
local point = Vector2.new(vp.X / 2, vp.Y / 2)   -- точка клика в экранных координатах

-- ===== GUI =====
local parent = (gethui and gethui()) or player:WaitForChild("PlayerGui")
local old = parent:FindFirstChild("AutoClickerGui")
if old then old:Destroy() end

local gui = Instance.new("ScreenGui")
gui.Name = "AutoClickerGui"
gui.ResetOnSpawn = false
gui.IgnoreGuiInset = true      -- координаты GUI = реальные координаты экрана
gui.DisplayOrder = 999
gui.Parent = parent

local function mk(class, props, par)
	local o = Instance.new(class)
	for k, v in pairs(props) do o[k] = v end
	o.Parent = par
	return o
end
local function round(o, r) Instance.new("UICorner", o).CornerRadius = UDim.new(0, r or 6) end

-- метка точки клика (не перехватывает нажатия)
local marker = mk("Frame", {
	Size = UDim2.fromOffset(26, 26), BackgroundTransparency = 1, Active = false,
}, gui)
local ring = mk("Frame", { Size = UDim2.new(1, 0, 1, 0), BackgroundTransparency = 1, Active = false }, marker)
round(ring, 13)
local stroke = Instance.new("UIStroke")
stroke.Thickness = 2
stroke.Color = Color3.fromRGB(255, 80, 80)
stroke.Parent = ring
local dot = mk("Frame", {
	Size = UDim2.fromOffset(4, 4), Position = UDim2.new(0.5, -2, 0.5, -2),
	BackgroundColor3 = Color3.fromRGB(255, 80, 80), BorderSizePixel = 0, Active = false,
}, marker)
round(dot, 2)

local function placeMarker()
	local v = workspace.CurrentCamera and workspace.CurrentCamera.ViewportSize or vp
	point = Vector2.new(math.clamp(point.X, 0, v.X), math.clamp(point.Y, 0, v.Y))
	marker.Position = UDim2.fromOffset(point.X - 13, point.Y - 13)
end

-- меню
local main = mk("Frame", {
	Size = UDim2.new(0, 230, 0, 242), Position = UDim2.new(0, 10, 0, 40),
	BackgroundColor3 = Color3.fromRGB(28, 28, 36), BorderSizePixel = 0, ClipsDescendants = true,
}, gui)
round(main, 8)

local bar = mk("Frame", { Size = UDim2.new(1, 0, 0, 24), BackgroundColor3 = Color3.fromRGB(40, 40, 52), BorderSizePixel = 0 }, main)
local title = mk("TextLabel", {
	Size = UDim2.new(1, -34, 1, 0), Position = UDim2.new(0, 8, 0, 0), BackgroundTransparency = 1,
	Text = "Auto Clicker", TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.GothamBold,
	TextSize = 12, TextXAlignment = Enum.TextXAlignment.Left,
}, bar)
local minBtn = mk("TextButton", {
	Size = UDim2.new(0, 28, 0, 24), Position = UDim2.new(1, -28, 0, 0), BackgroundTransparency = 1,
	Text = "–", TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.GothamBold, TextSize = 16,
}, bar)

local body = mk("Frame", { Size = UDim2.new(1, 0, 1, -24), Position = UDim2.new(0, 0, 0, 24), BackgroundTransparency = 1 }, main)

local function btn(text, x, y, w, h, color, size)
	local b = mk("TextButton", {
		Size = UDim2.fromOffset(w, h), Position = UDim2.fromOffset(x, y), BackgroundColor3 = color,
		Text = text, TextColor3 = Color3.new(1, 1, 1), Font = Enum.Font.GothamBold, TextSize = size or 13,
		AutoButtonColor = true,
	}, body)
	round(b, 6)
	return b
end

local GREEN, RED = Color3.fromRGB(50, 150, 90), Color3.fromRGB(190, 70, 70)
local DARK = Color3.fromRGB(60, 60, 78)

-- Старт / Стоп
local startBtn = btn("СТАРТ", 6, 6, 218, 34, GREEN, 15)

-- Стрелки (крестовина)
local upBtn    = btn("▲", 46, 46, 40, 30, DARK, 14)
local leftBtn  = btn("◀", 4 + 2, 80, 40, 30, DARK, 14)
local downBtn  = btn("▼", 46, 80, 40, 30, DARK, 14)
local rightBtn = btn("▶", 86, 80, 40, 30, DARK, 14)
leftBtn.Position = UDim2.fromOffset(6, 80)

-- Справа от стрелок: координаты, шаг, центр
local coordLabel = mk("TextLabel", {
	Size = UDim2.fromOffset(92, 30), Position = UDim2.fromOffset(132, 46), BackgroundColor3 = Color3.fromRGB(18, 18, 24),
	TextColor3 = Color3.fromRGB(140, 255, 160), Font = Enum.Font.Code, TextSize = 11, Text = "",
}, body)
round(coordLabel, 5)
local stepBtn = btn("Шаг: 10", 132, 80, 44, 30, DARK, 10)
local centerBtn = btn("Центр", 180, 80, 44, 30, DARK, 10)

-- Интервал
mk("TextLabel", {
	Size = UDim2.fromOffset(60, 28), Position = UDim2.fromOffset(6, 118), BackgroundTransparency = 1,
	Text = "Интервал, с:", TextColor3 = Color3.fromRGB(190, 190, 205), Font = Enum.Font.Gotham,
	TextSize = 11, TextXAlignment = Enum.TextXAlignment.Left,
}, body)
local minusBtn = btn("−", 78, 118, 34, 28, DARK, 16)
local intervalBox = mk("TextBox", {
	Size = UDim2.fromOffset(62, 28), Position = UDim2.fromOffset(116, 118),
	BackgroundColor3 = Color3.fromRGB(18, 18, 24), TextColor3 = Color3.new(1, 1, 1),
	Text = "0.10", Font = Enum.Font.Code, TextSize = 13, ClearTextOnFocus = false,
}, body)
round(intervalBox, 5)
local plusBtn = btn("+", 182, 118, 34, 28, DARK, 16)

-- Режим и счётчик
local modeBtn = btn("Режим: Мышь", 6, 152, 108, 26, DARK, 11)
local countLabel = mk("TextLabel", {
	Size = UDim2.fromOffset(104, 26), Position = UDim2.fromOffset(120, 152), BackgroundColor3 = Color3.fromRGB(18, 18, 24),
	TextColor3 = Color3.fromRGB(230, 230, 240), Font = Enum.Font.Code, TextSize = 11, Text = "Кликов: 0",
}, body)
round(countLabel, 5)

-- Полное выключение
local offBtn = btn("ВЫКЛЮЧИТЬ СКРИПТ", 6, 184, 218, 28, RED, 12)

-- ===== Перетаскивание меню =====
do
	local dragging, startPos, startInput
	bar.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			dragging, startPos, startInput = true, main.Position, i.Position
			i.Changed:Connect(function()
				if i.UserInputState == Enum.UserInputState.End then dragging = false end
			end)
		end
	end)
	table.insert(connections, UIS.InputChanged:Connect(function(i)
		if dragging and (i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseMovement) then
			local d = i.Position - startInput
			main.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + d.X, startPos.Y.Scale, startPos.Y.Offset + d.Y)
		end
	end))
end

-- ===== Логика =====
local function updateCoords()
	coordLabel.Text = string.format("X:%d  Y:%d", point.X, point.Y)
	placeMarker()
end

local function movePoint(dx, dy)
	point = point + Vector2.new(dx, dy) * steps[stepIdx]
	updateCoords()
end

-- стрелка: тап = 1 шаг, удержание = непрерывное движение
local function bindArrow(b, dx, dy)
	local active
	b.InputBegan:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			local mine = {}
			active = mine
			movePoint(dx, dy)
			task.spawn(function()
				task.wait(0.35)
				while alive and active == mine do
					movePoint(dx, dy)
					task.wait(0.04)
				end
			end)
			i.Changed:Connect(function()
				if i.UserInputState == Enum.UserInputState.End and active == mine then active = nil end
			end)
		end
	end)
	b.InputEnded:Connect(function(i)
		if i.UserInputType == Enum.UserInputType.Touch or i.UserInputType == Enum.UserInputType.MouseButton1 then
			active = nil
		end
	end)
end
bindArrow(upBtn, 0, -1)
bindArrow(downBtn, 0, 1)
bindArrow(leftBtn, -1, 0)
bindArrow(rightBtn, 1, 0)

stepBtn.Activated:Connect(function()
	stepIdx = stepIdx % #steps + 1
	stepBtn.Text = "Шаг: " .. steps[stepIdx]
end)

centerBtn.Activated:Connect(function()
	local v = workspace.CurrentCamera.ViewportSize
	point = Vector2.new(v.X / 2, v.Y / 2)
	updateCoords()
end)

-- интервал
local function setInterval(v)
	interval = math.clamp(math.floor(v * 100 + 0.5) / 100, 0.01, 60)
	intervalBox.Text = string.format("%.2f", interval)
end
minusBtn.Activated:Connect(function() setInterval(interval - 0.05) end)
plusBtn.Activated:Connect(function() setInterval(interval + 0.05) end)
intervalBox.FocusLost:Connect(function()
	local v = tonumber((intervalBox.Text:gsub(",", ".")))
	setInterval(v or interval)
end)

modeBtn.Activated:Connect(function()
	useTouch = not useTouch
	modeBtn.Text = "Режим: " .. (useTouch and "Тач" or "Мышь")
end)

-- сам клик
local function clickAt(x, y)
	if useTouch then
		pcall(function()
			VIM:SendTouchEvent(1, 0, x, y)
			VIM:SendTouchEvent(1, 2, x, y)
		end)
	else
		pcall(function()
			VIM:SendMouseButtonEvent(x, y, 0, true, game, 0)
			VIM:SendMouseButtonEvent(x, y, 0, false, game, 0)
		end)
	end
end

-- метка под самим меню? тогда клик попадёт в меню, пропускаем
local function pointUnderMenu()
	if not body.Visible and main.AbsoluteSize.Y <= 24 then
		local p, s = main.AbsolutePosition, main.AbsoluteSize
		return point.X >= p.X and point.X <= p.X + s.X and point.Y >= p.Y and point.Y <= p.Y + s.Y
	end
	local p, s = main.AbsolutePosition, main.AbsoluteSize
	return point.X >= p.X and point.X <= p.X + s.X and point.Y >= p.Y and point.Y <= p.Y + s.Y
end

local function setRunning(v)
	running = v
	startBtn.Text = v and "СТОП" or "СТАРТ"
	startBtn.BackgroundColor3 = v and RED or GREEN
	local c = v and Color3.fromRGB(80, 255, 120) or Color3.fromRGB(255, 80, 80)
	stroke.Color, dot.BackgroundColor3 = c, c
	if v then
		task.spawn(function()
			while alive and running do
				if pointUnderMenu() then
					title.Text = "Метка под меню!"
				else
					title.Text = "Auto Clicker"
					clickAt(point.X, point.Y)
					clicks += 1
					countLabel.Text = "Кликов: " .. clicks
				end
				task.wait(interval)
			end
		end)
	else
		title.Text = "Auto Clicker"
	end
end

startBtn.Activated:Connect(function() setRunning(not running) end)

-- сворачивание
local minimized = false
minBtn.Activated:Connect(function()
	minimized = not minimized
	body.Visible = not minimized
	main.Size = minimized and UDim2.new(0, 230, 0, 24) or UDim2.new(0, 230, 0, 242)
	minBtn.Text = minimized and "+" or "–"
end)

-- полное выключение
offBtn.Activated:Connect(function()
	alive = false
	running = false
	for _, c in ipairs(connections) do pcall(function() c:Disconnect() end) end
	gui:Destroy()
end)

-- пересчёт метки при смене размера экрана
if workspace.CurrentCamera then
	table.insert(connections, workspace.CurrentCamera:GetPropertyChangedSignal("ViewportSize"):Connect(placeMarker))
end

updateCoords()
