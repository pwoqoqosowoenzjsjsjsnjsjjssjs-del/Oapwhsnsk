-- LocalScript: StarterPlayer > StarterPlayerScripts
-- DHZ HUB
-- KEY: DHZONTOP
-- GUI estilo LUVIX vermelho/escuro.
-- O botão ANTI HIT mantém a GUI original, mas usa a lógica funcional do Ultra TP.

local Players = game:GetService("Players")
local ProximityPromptService = game:GetService("ProximityPromptService")
local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local RunService = game:GetService("RunService")

local player = Players.LocalPlayer
if not player then
	return
end

local playerGui = player:WaitForChild("PlayerGui")

local previous = playerGui:FindFirstChild("DHZ_HUB_GUI")
if previous then
	previous:Destroy()
end

local SCRIPT_KEY = "DHZONTOP"
local IMAGE_ID = "rbxassetid://81245576355054"

-- O botão visual continua sendo ANTI HIT,
-- mas a lógica executada por ele é a lógica funcional do Ultra TP.
local UltraTP = false
local RouteRunning = false

local TeleportPoints = {
	Vector3.new(500.62, 70.28, -366.64),
	Vector3.new(508.30, 70.28, -366.03),
	Vector3.new(519.43, 70.28, -366.47),
	Vector3.new(529.22, 70.28, -366.71),
	Vector3.new(546.80, 70.28, -364.40)
}

--==============================================================
-- HELPERS
--==============================================================

local function create(className, properties, parent)
	local object = Instance.new(className)

	for key, value in pairs(properties) do
		object[key] = value
	end

	object.Parent = parent
	return object
end

local function round(object, radius)
	return create(
		"UICorner",
		{
			CornerRadius = radius
		},
		object
	)
end

--==============================================================
-- GUI ROOT
--==============================================================

local gui = create(
	"ScreenGui",
	{
		Name = "DHZ_HUB_GUI",
		ResetOnSpawn = false,
		IgnoreGuiInset = true,
		DisplayOrder = 100,
		ZIndexBehavior = Enum.ZIndexBehavior.Sibling,
	},
	playerGui
)

local connections = {}

local function connect(signal, callback)
	local connection = signal:Connect(callback)
	table.insert(connections, connection)
	return connection
end

local root = create(
	"Frame",
	{
		Name = "Screen",
		Size = UDim2.fromScale(1, 1),
		BackgroundTransparency = 1,
	},
	gui
)

--==============================================================
-- SOUNDS
--==============================================================

local function makeSound(name, id, volume)
	local sound = Instance.new("Sound")
	sound.Name = name
	sound.SoundId = id
	sound.Volume = volume or 0.35
	sound.Parent = gui
	return sound
end

local SoundClick = makeSound(
	"Click",
	"rbxassetid://6895079853",
	0.35
)

local SoundOpen = makeSound(
	"Open",
	"rbxassetid://9119713951",
	0.30
)

local SoundSuccess = makeSound(
	"Success",
	"rbxassetid://6026984224",
	0.35
)

local SoundError = makeSound(
	"Error",
	"rbxassetid://138081500",
	0.30
)

local SoundToggle = makeSound(
	"Toggle",
	"rbxassetid://12221967",
	0.30
)

local function playSound(sound)
	if not sound then
		return
	end

	pcall(function()
		sound:Stop()
		sound.TimePosition = 0
		sound:Play()
	end)
end

--==============================================================
-- RGB BORDER
--==============================================================

local function rgbBorder(object, thickness, transparency)
	local stroke = create(
		"UIStroke",
		{
			Color = Color3.new(1, 1, 1),
			Thickness = thickness,
			Transparency = transparency,
			ApplyStrokeMode = Enum.ApplyStrokeMode.Border,
		},
		object
	)

	return create(
		"UIGradient",
		{
			Color = ColorSequence.new({
				ColorSequenceKeypoint.new(0, Color3.new(0.235, 0, 0)),
				ColorSequenceKeypoint.new(0.35, Color3.new(0.706, 0.078, 0.078)),
				ColorSequenceKeypoint.new(0.60, Color3.new(1, 0.235, 0.235)),
				ColorSequenceKeypoint.new(1, Color3.new(0.471, 0, 0)),
			}),
			Rotation = 135,
		},
		stroke
	)
end

--==============================================================
-- TEXT HELPER
--==============================================================

local function text(
	className,
	name,
	content,
	size,
	position,
	dimensions,
	parent
)
	return create(
		className,
		{
			Name = name,
			Text = content,
			Font = Enum.Font.GothamBold,
			TextSize = size,
			TextColor3 = Color3.fromRGB(248, 248, 248),
			TextStrokeColor3 = Color3.fromRGB(180, 180, 180),
			TextStrokeTransparency = 1,
			TextXAlignment = Enum.TextXAlignment.Center,
			TextYAlignment = Enum.TextYAlignment.Center,
			BackgroundTransparency = 1,
			BorderSizePixel = 0,
			Position = position,
			Size = dimensions,
		},
		parent
	)
end

--==============================================================
-- KEY PANEL
--==============================================================

local keyPanel = create(
	"CanvasGroup",
	{
		Name = "KeyPanel",
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(0.5, 0.5),
		Size = UDim2.fromOffset(280, 200),
		BackgroundColor3 = Color3.new(0.063, 0.016, 0.024),
		BackgroundTransparency = 0.10,
		BorderSizePixel = 0,
		GroupTransparency = 0,
		ZIndex = 50,
	},
	root
)

round(
	keyPanel,
	UDim.new(0, 20)
)

local keyRGB = rgbBorder(
	keyPanel,
	2,
	0.08
)

local keyAccent = create(
	"Frame",
	{
		Name = "KeyAccent",
		Size = UDim2.fromOffset(3, 28),
		Position = UDim2.fromOffset(9, 11),
		BackgroundColor3 = Color3.new(1, 0.176, 0.176),
		BorderSizePixel = 0,
		ZIndex = 52,
	},
	keyPanel
)

round(keyAccent, UDim.new(0, 2))

local keyTitle = text(
	"TextLabel",
	"KeyTitle",
	"DHZ HUB",
	16,
	UDim2.fromOffset(18, 7),
	UDim2.new(1, -30, 0, 22),
	keyPanel
)

keyTitle.Font = Enum.Font.GothamBlack
keyTitle.TextXAlignment = Enum.TextXAlignment.Left
keyTitle.ZIndex = 51

local keyTitleGradient = create(
	"UIGradient",
	{
		Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.new(0.549, 0.039, 0.039)),
			ColorSequenceKeypoint.new(0.5, Color3.new(1, 0.275, 0.275)),
			ColorSequenceKeypoint.new(1, Color3.new(1, 0.706, 0.706))
		}),
	},
	keyTitle
)

local keySubtitle = text(
	"TextLabel",
	"KeySubtitle",
	"ACCESS KEY",
	9,
	UDim2.fromOffset(18, 28),
	UDim2.new(1, -30, 0, 16),
	keyPanel
)

keySubtitle.Font = Enum.Font.GothamMedium
keySubtitle.TextColor3 = Color3.new(1, 0.471, 0.471)
keySubtitle.TextTransparency = 0.25
keySubtitle.TextXAlignment = Enum.TextXAlignment.Left
keySubtitle.ZIndex = 51

local keyBox = create(
	"TextBox",
	{
		Name = "KeyBox",
		AnchorPoint = Vector2.new(0.5, 0),
		Position = UDim2.new(0.5, 0, 0, 62),
		Size = UDim2.fromOffset(252, 44),
		BackgroundColor3 = Color3.new(0.11, 0.024, 0.035),
		BorderSizePixel = 0,
		Text = "",
		PlaceholderText = "DIGITE A KEY",
		PlaceholderColor3 = Color3.fromRGB(170, 120, 125),
		TextColor3 = Color3.fromRGB(255, 238, 238),
		Font = Enum.Font.GothamBold,
		TextSize = 12,
		ClearTextOnFocus = false,
		ZIndex = 51,
	},
	keyPanel
)

round(
	keyBox,
	UDim.new(0, 12)
)

local keyBoxGradient = rgbBorder(
	keyBox,
	1.1,
	0.20
)

local verifyButton = text(
	"TextButton",
	"Verify",
	"VERIFICAR KEY",
	12,
	UDim2.fromOffset(14, 116),
	UDim2.fromOffset(252, 44),
	keyPanel
)

verifyButton.BackgroundColor3 = Color3.new(0.11, 0.024, 0.035)
verifyButton.BackgroundTransparency = 0
verifyButton.AutoButtonColor = false
verifyButton.Active = true
verifyButton.Selectable = true
verifyButton.Modal = false
verifyButton.ZIndex = 51

round(
	verifyButton,
	UDim.new(0, 12)
)

local verifyGradient = rgbBorder(
	verifyButton,
	1.1,
	0.20
)

local keyStatus = text(
	"TextLabel",
	"KeyStatus",
	"",
	10,
	UDim2.fromOffset(0, 168),
	UDim2.new(1, 0, 0, 20),
	keyPanel
)

keyStatus.ZIndex = 51

--==============================================================
-- MAIN PANEL
--==============================================================

local panel = create(
	"CanvasGroup",
	{
		Name = "Panel",
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(
			1110 / 1536,
			311 / 691
		),
		Size = UDim2.fromOffset(
			280,
			205
		),
		BackgroundColor3 = Color3.new(
			0.063,
			0.016,
			0.024
		),
		BackgroundTransparency = 0.15,
		BorderSizePixel = 0,
		GroupTransparency = 0,
		Visible = false,
	},
	root
)

round(
	panel,
	UDim.new(0, 20)
)

local panelScale = create(
	"UIScale",
	{
		Scale = 1
	},
	panel
)

local panelGradient = rgbBorder(
	panel,
	2,
	0.08
)

--==============================================================
-- HEADER
--==============================================================

local header = create(
	"Frame",
	{
		Name = "Header",
		Size = UDim2.new(1, 0, 0, 46),
		BackgroundTransparency = 1,
		BorderSizePixel = 0,
		Active = true,
	},
	panel
)

local headerAccent = create(
	"Frame",
	{
		Name = "HeaderAccent",
		Size = UDim2.fromOffset(3, 26),
		Position = UDim2.fromOffset(8, 10),
		BackgroundColor3 = Color3.new(1, 0.176, 0.176),
		BorderSizePixel = 0,
		ZIndex = 14,
	},
	header
)

round(headerAccent, UDim.new(0, 2))

local hubTitle = text(
	"TextLabel",
	"Title",
	"DHZ HUB",
	15,
	UDim2.fromOffset(16, 8),
	UDim2.new(1, -66, 0, 18),
	header
)

hubTitle.Font = Enum.Font.GothamBlack
hubTitle.TextXAlignment = Enum.TextXAlignment.Left
hubTitle.ZIndex = 14

local hubTitleGradient = create(
	"UIGradient",
	{
		Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.new(0.549, 0.039, 0.039)),
			ColorSequenceKeypoint.new(0.5, Color3.new(1, 0.275, 0.275)),
			ColorSequenceKeypoint.new(1, Color3.new(1, 0.706, 0.706))
		}),
	},
	hubTitle
)

local subtitle = text(
	"TextLabel",
	"Subtitle",
	"STEAL AN EGG V1",
	9,
	UDim2.fromOffset(16, 25),
	UDim2.new(1, -66, 0, 13),
	header
)

subtitle.Font = Enum.Font.GothamMedium
subtitle.TextColor3 = Color3.new(1, 0.471, 0.471)
subtitle.TextTransparency = 0.25
subtitle.TextXAlignment = Enum.TextXAlignment.Left
subtitle.ZIndex = 14

--==============================================================
-- ANTI HIT
--==============================================================

local antiHit = text(
	"TextButton",
	"AntiHit",
	"ANTI HIT: OFF",
	13,
	UDim2.fromOffset(14, 56),
	UDim2.fromOffset(252, 56),
	panel
)

antiHit.BackgroundColor3 = Color3.new(0.11, 0.024, 0.035)
antiHit.BackgroundTransparency = 0.10
antiHit.AutoButtonColor = false
antiHit.TextXAlignment = Enum.TextXAlignment.Left
antiHit.TextColor3 = Color3.new(1, 0.922, 0.922)

local antiPadding = create(
	"UIPadding",
	{
		PaddingLeft = UDim.new(0, 24),
	},
	antiHit
)

local antiDot = create(
	"Frame",
	{
		Name = "Dot",
		Size = UDim2.fromOffset(6, 6),
		Position = UDim2.new(0, 10, 0.5, -3),
		BackgroundColor3 = Color3.new(0.353, 0.078, 0.078),
		BorderSizePixel = 0,
		ZIndex = 17,
	},
	antiHit
)

round(antiDot, UDim.new(0, 3))

local antiArrow = text(
	"TextLabel",
	"Arrow",
	"›",
	18,
	UDim2.new(1, -26, 0, 0),
	UDim2.new(0, 20, 1, 0),
	antiHit
)

antiArrow.Font = Enum.Font.GothamBlack
antiArrow.TextColor3 = Color3.new(1, 0.35, 0.35)
antiArrow.ZIndex = 17

local antiHitGradient = rgbBorder(
	antiHit,
	1.2,
	0.18
)

round(
	antiHit,
	UDim.new(0, 14)
)

--==============================================================
-- PREMIUM DHZ
--==============================================================

local premium = text(
	"TextLabel",
	"Premium",
	"PREMIUM DHZ",
	12,
	UDim2.fromOffset(14, 120),
	UDim2.fromOffset(252, 42),
	panel
)

premium.BackgroundColor3 = Color3.new(0.11, 0.024, 0.035)
premium.BackgroundTransparency = 0.10
premium.TextColor3 = Color3.new(1, 0.82, 0.82)

local premiumGradient = rgbBorder(
	premium,
	1.0,
	0.24
)

round(
	premium,
	UDim.new(0, 10)
)

--==============================================================
-- FLOATING BUBBLE
--==============================================================

local bubble = create(
	"ImageButton",
	{
		Name = "FloatingButton",
		AnchorPoint = Vector2.new(0.5, 0.5),
		Position = UDim2.fromScale(
			704 / 1536,
			95 / 691
		),
		Size = UDim2.fromOffset(
			76,
			76
		),
		BackgroundColor3 = Color3.new(
			0.063,
			0.016,
			0.024
		),
		BorderSizePixel = 0,
		Image = IMAGE_ID,
		ScaleType = Enum.ScaleType.Crop,
		AutoButtonColor = false,
		ZIndex = 10,
		Visible = false,
	},
	root
)

round(
	bubble,
	UDim.new(1, 0)
)

local bubbleScale = create(
	"UIScale",
	{
		Scale = 1
	},
	bubble
)

local bubbleGradient = rgbBorder(
	bubble,
	2,
	0.05
)

--==============================================================
-- SCALE / RESIZE
--==============================================================

local visualScale = 1
local opened = true

local animationSerial = 0

local fadeTween
local scaleTween

local function clampPosition(object)
	local screen = root.AbsoluteSize
	local half = object.AbsoluteSize / 2
	local center = object.AbsolutePosition + half

	local x = math.clamp(
		center.X,
		half.X + 6,
		math.max(
			half.X + 6,
			screen.X - half.X - 6
		)
	)

	local y = math.clamp(
		center.Y,
		half.Y + 6,
		math.max(
			half.Y + 6,
			screen.Y - half.Y - 6
		)
	)

	object.Position = UDim2.fromOffset(
		x,
		y
	)
end

local function resize()
	local screen = root.AbsoluteSize

	if screen.X < 1 or screen.Y < 1 then
		return
	end

	visualScale = math.clamp(
		math.min(
			screen.X / 1536,
			screen.Y / 691
		),
		0.65,
		1
	)

	if scaleTween then
		scaleTween:Cancel()
	end

	panelScale.Scale =
		opened
		and visualScale
		or visualScale * 0.94

	bubbleScale.Scale = visualScale

	clampPosition(panel)
	clampPosition(bubble)
end

--==============================================================
-- PANEL OPEN / CLOSE
--==============================================================

local function togglePanel()
	playSound(SoundOpen)

	opened = not opened

	animationSerial += 1

	local serial = animationSerial

	if fadeTween then
		fadeTween:Cancel()
	end

	if scaleTween then
		scaleTween:Cancel()
	end

	panel.Visible = true

	local info = TweenInfo.new(
		0.17,
		Enum.EasingStyle.Quad,
		Enum.EasingDirection.Out
	)

	fadeTween = TweenService:Create(
		panel,
		info,
		{
			GroupTransparency =
				opened and 0 or 1
		}
	)

	scaleTween = TweenService:Create(
		panelScale,
		info,
		{
			Scale =
				visualScale
				*
				(
					opened
					and 1
					or 0.94
				)
		}
	)

	fadeTween:Play()
	scaleTween:Play()

	task.delay(
		0.18,
		function()
			if
				gui.Parent
				and serial == animationSerial
				and not opened
			then
				panel.Visible = false
			end
		end
	)
end

--==============================================================
-- DRAG FIX
--==============================================================

local function draggable(handle, target, onTap)
	local dragging = false
	local dragMode = nil
	local touchInput = nil

	local startPoint = nil
	local startPosition = nil

	local moved = false

	connect(
		handle.InputBegan,
		function(input)
			if dragging then
				return
			end

			if input.UserInputType == Enum.UserInputType.MouseButton1 then
				dragging = true
				dragMode = "Mouse"
			elseif input.UserInputType == Enum.UserInputType.Touch then
				dragging = true
				dragMode = "Touch"
				touchInput = input
			else
				return
			end

			startPoint = Vector2.new(
				input.Position.X,
				input.Position.Y
			)

			startPosition = target.Position
			moved = false
		end
	)

	connect(
		UserInputService.InputChanged,
		function(input)
			if not dragging then
				return
			end

			local valid = false

			if dragMode == "Mouse" then
				if input.UserInputType == Enum.UserInputType.MouseMovement then
					valid = true
				end
			elseif dragMode == "Touch" then
				if input == touchInput then
					valid = true
				end
			end

			if not valid then
				return
			end

			local current = Vector2.new(
				input.Position.X,
				input.Position.Y
			)

			local delta = current - startPoint

			if delta.Magnitude > 6 then
				moved = true
			end

			if moved then
				target.Position = UDim2.new(
					startPosition.X.Scale,
					startPosition.X.Offset + delta.X,
					startPosition.Y.Scale,
					startPosition.Y.Offset + delta.Y
				)

				clampPosition(target)
			end
		end
	)

	connect(
		UserInputService.InputEnded,
		function(input)
			if not dragging then
				return
			end

			local ended = false

			if
				dragMode == "Mouse"
				and input.UserInputType == Enum.UserInputType.MouseButton1
			then
				ended = true
			elseif
				dragMode == "Touch"
				and input == touchInput
			then
				ended = true
			end

			if not ended then
				return
			end

			dragging = false
			dragMode = nil
			touchInput = nil

			if not moved and onTap then
				onTap()
			end

			moved = false
			startPoint = nil
			startPosition = nil
		end
	)
end

draggable(
	header,
	panel
)

draggable(
	bubble,
	bubble,
	togglePanel
)

connect(
	bubble.Activated,
	function(input)
		if
			input
			and input.UserInputType.Name:match("Gamepad")
		then
			togglePanel()
		end
	end
)

local authenticated = false

--==============================================================
-- TELA RÁPIDA AO PEGAR O OVO
-- Usa o MESMO PromptTriggered que já ativa a lógica existente.
-- Não altera drag, botão, key ou rota.
--==============================================================

local EggLoading = Instance.new("Frame")
EggLoading.Name = "EggLoading"
EggLoading.Size = UDim2.fromScale(1, 1)
EggLoading.Position = UDim2.fromScale(0, 0)
EggLoading.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
EggLoading.BackgroundTransparency = 0
EggLoading.BorderSizePixel = 0
EggLoading.Visible = false
EggLoading.ZIndex = 200
EggLoading.Parent = gui

local EggLoadingCard = Instance.new("Frame")
EggLoadingCard.Position = UDim2.fromScale(0, 0)
EggLoadingCard.Size = UDim2.fromScale(1, 1)
EggLoadingCard.BackgroundColor3 = Color3.new(0.035, 0.008, 0.014)
EggLoadingCard.BackgroundTransparency = 0.02
EggLoadingCard.BorderSizePixel = 0
EggLoadingCard.ZIndex = 201
EggLoadingCard.Parent = EggLoading

local EggLoadingText = Instance.new("TextLabel")
EggLoadingText.BackgroundTransparency = 1
EggLoadingText.AnchorPoint = Vector2.new(0.5, 0.5)
EggLoadingText.Position = UDim2.fromScale(0.5, 0.46)
EggLoadingText.Size = UDim2.new(0.9, 0, 0, 36)
EggLoadingText.Text = "CARREGANDO..."
EggLoadingText.TextColor3 = Color3.fromRGB(248, 248, 248)
EggLoadingText.Font = Enum.Font.LuckiestGuy
EggLoadingText.TextSize = 17
EggLoadingText.ZIndex = 202
EggLoadingText.Parent = EggLoadingCard

local EggLoadingSub = Instance.new("TextLabel")
EggLoadingSub.BackgroundTransparency = 1
EggLoadingSub.AnchorPoint = Vector2.new(0.5, 0.5)
EggLoadingSub.Position = UDim2.fromScale(0.5, 0.515)
EggLoadingSub.Size = UDim2.new(0.9, 0, 0, 20)
EggLoadingSub.Text = "PEGANDO O OVO"
EggLoadingSub.TextColor3 = Color3.fromRGB(190, 110, 120)
EggLoadingSub.Font = Enum.Font.LuckiestGuy
EggLoadingSub.TextSize = 10
EggLoadingSub.ZIndex = 202
EggLoadingSub.Parent = EggLoadingCard

local EggBarBack = Instance.new("Frame")
EggBarBack.AnchorPoint = Vector2.new(0.5, 0.5)
EggBarBack.Position = UDim2.fromScale(0.5, 0.565)
EggBarBack.Size = UDim2.new(0.72, 0, 0, 8)
EggBarBack.BackgroundColor3 = Color3.fromRGB(58, 20, 27)
EggBarBack.BorderSizePixel = 0
EggBarBack.ZIndex = 202
EggBarBack.Parent = EggLoadingCard

round(
	EggBarBack,
	UDim.new(1, 0)
)

local EggBar = Instance.new("Frame")
EggBar.Size = UDim2.new(0, 0, 1, 0)
EggBar.BackgroundColor3 = Color3.fromRGB(230, 55, 70)
EggBar.BorderSizePixel = 0
EggBar.ZIndex = 203
EggBar.Parent = EggBarBack

round(
	EggBar,
	UDim.new(1, 0)
)

local EggBarGradient = create(
	"UIGradient",
	{
		Color = ColorSequence.new({
			ColorSequenceKeypoint.new(0, Color3.new(0.235, 0, 0)),
			ColorSequenceKeypoint.new(0.35, Color3.new(0.706, 0.078, 0.078)),
			ColorSequenceKeypoint.new(0.60, Color3.new(1, 0.235, 0.235)),
			ColorSequenceKeypoint.new(1, Color3.new(0.471, 0, 0))
		}),
	},
	EggBar
)

local LoadingSerial = 0

local function ShowEggLoading()
	LoadingSerial += 1

	local MySerial = LoadingSerial

	EggLoading.Visible = true
	EggLoading.BackgroundTransparency = 1
	EggLoadingCard.BackgroundTransparency = 1
	EggLoadingText.TextTransparency = 1
	EggLoadingSub.TextTransparency = 1
	EggBarBack.BackgroundTransparency = 1
	EggBar.BackgroundTransparency = 0
	EggBar.Size = UDim2.new(0, 0, 1, 0)

	TweenService:Create(
		EggLoading,
		TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
		{
			BackgroundTransparency = 0
		}
	):Play()

	TweenService:Create(
		EggLoadingCard,
		TweenInfo.new(0.12, Enum.EasingStyle.Quad, Enum.EasingDirection.Out),
		{
			BackgroundTransparency = 0.02
		}
	):Play()

	TweenService:Create(
		EggLoadingText,
		TweenInfo.new(0.12),
		{
			TextTransparency = 0
		}
	):Play()

	TweenService:Create(
		EggLoadingSub,
		TweenInfo.new(0.12),
		{
			TextTransparency = 0
		}
	):Play()

	TweenService:Create(
		EggBarBack,
		TweenInfo.new(0.12),
		{
			BackgroundTransparency = 0
		}
	):Play()

	local BarTween = TweenService:Create(
		EggBar,
		TweenInfo.new(2, Enum.EasingStyle.Linear),
		{
			Size = UDim2.new(1, 0, 1, 0)
		}
	)

	BarTween:Play()

	task.delay(2, function()
		if MySerial ~= LoadingSerial then
			return
		end

		local Fade = TweenService:Create(
			EggLoading,
			TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.In),
			{
				BackgroundTransparency = 1
			}
		)

		TweenService:Create(
			EggLoadingCard,
			TweenInfo.new(0.15),
			{
				BackgroundTransparency = 1
			}
		):Play()

		TweenService:Create(
			EggLoadingText,
			TweenInfo.new(0.12),
			{
				TextTransparency = 1
			}
		):Play()

		TweenService:Create(
			EggLoadingSub,
			TweenInfo.new(0.12),
			{
				TextTransparency = 1
			}
		):Play()

		TweenService:Create(
			EggBarBack,
			TweenInfo.new(0.12),
			{
				BackgroundTransparency = 1
			}
		):Play()

		Fade:Play()

		Fade.Completed:Once(function()
			if
				MySerial == LoadingSerial
				and EggLoading.Parent
			then
				EggLoading.Visible = false
			end
		end)
	end)
end

--==============================================================
-- ANTI HIT BUTTON = LÓGICA DO ULTRA TP
--==============================================================

gui:SetAttribute("AntiHit", false)

local function updateAntiHitButton()
	if UltraTP then
		antiHit.Text = "ANTI HIT: ON"
		antiDot.BackgroundColor3 = Color3.new(1, 0.235, 0.235)
		gui:SetAttribute("AntiHit", true)
	else
		antiHit.Text = "ANTI HIT: OFF"
		antiDot.BackgroundColor3 = Color3.new(0.353, 0.078, 0.078)
		gui:SetAttribute("AntiHit", false)
	end
end

connect(
	antiHit.Activated,
	function()
		playSound(SoundToggle)

		UltraTP = not UltraTP
		updateAntiHitButton()
	end
)

--==============================================================
-- ROTA DO ULTRA TP
--==============================================================

local function RunTeleportRoute(Character)
	if not UltraTP then
		return
	end

	if not Character or not Character.Parent then
		return
	end

	local Root = Character:FindFirstChild("HumanoidRootPart")

	if not Root then
		Root = Character:WaitForChild("HumanoidRootPart", 5)
	end

	if not Root then
		return
	end

	for _, Position in ipairs(TeleportPoints) do
		if not UltraTP then
			return
		end

		if not Character.Parent then
			return
		end

		Character:PivotTo(
			CFrame.new(Position)
		)

		RunService.Heartbeat:Wait()
	end

	playSound(SoundSuccess)
end

--==============================================================
-- PROXIMITY PROMPT
--==============================================================

connect(
	ProximityPromptService.PromptTriggered,
	function(Prompt, Player)
		if not authenticated then
			return
		end

		if Player ~= player then
			return
		end

		if not UltraTP then
			return
		end

		if RouteRunning then
			return
		end

		local Character = player.Character

		if not Character then
			return
		end

		-- O próprio evento que já detecta a interação exibe a tela.
		-- A rota continua funcionando normalmente por trás.
		task.spawn(ShowEggLoading)

		RouteRunning = true

		task.spawn(function()
			local Success, Error = pcall(function()
				RunTeleportRoute(Character)
			end)

			RouteRunning = false

			if not Success then
				warn("[DHZ HUB] Erro no Ultra TP:", Error)
				playSound(SoundError)
			end
		end)
	end
)

updateAntiHitButton()

--==============================================================
-- KEY SYSTEM
--==============================================================

local verifyBusy = false

local function unlock()
	if authenticated then
		return
	end

	authenticated = true

	playSound(SoundSuccess)

	keyStatus.Text = "ACCESS GRANTED"

	keyStatus.TextColor3 = Color3.fromRGB(
		90,
		225,
		115
	)

	local keyScale = create(
		"UIScale",
		{
			Scale = 1
		},
		keyPanel
	)

	local out = TweenService:Create(
		keyScale,
		TweenInfo.new(
			0.18,
			Enum.EasingStyle.Quad,
			Enum.EasingDirection.In
		),
		{
			Scale = 0.92
		}
	)

	local fade = TweenService:Create(
		keyPanel,
		TweenInfo.new(0.18),
		{
			GroupTransparency = 1
		}
	)

	out:Play()
	fade:Play()

	fade.Completed:Once(function()
		if not gui.Parent then
			return
		end

		keyPanel.Visible = false

		panel.Visible = true
		bubble.Visible = true

		panel.GroupTransparency = 1

		local panelIn = TweenService:Create(
			panel,
			TweenInfo.new(
				0.20,
				Enum.EasingStyle.Quad,
				Enum.EasingDirection.Out
			),
			{
				GroupTransparency = 0
			}
		)

		panelIn:Play()

		resize()
	end)
end

local function verifyKey()
	if verifyBusy then
		return
	end

	verifyBusy = true

	playSound(SoundClick)

	local entered = tostring(
		keyBox.Text or ""
	)

	entered = entered:gsub(
		"^%s+",
		""
	)

	entered = entered:gsub(
		"%s+$",
		""
	)

	entered = string.upper(
		entered
	)

	local originalColor =
		verifyButton.BackgroundColor3

	TweenService:Create(
		verifyButton,
		TweenInfo.new(0.07),
		{
			BackgroundColor3 =
				Color3.fromRGB(
					72,
					71,
					84
				)
		}
	):Play()

	task.delay(
		0.08,
		function()
			if verifyButton.Parent then
				TweenService:Create(
					verifyButton,
					TweenInfo.new(0.12),
					{
						BackgroundColor3 =
							originalColor
					}
				):Play()
			end
		end
	)

	if entered == SCRIPT_KEY then
		keyStatus.Text =
			"KEY CORRETA"

		keyStatus.TextColor3 =
			Color3.fromRGB(
				95,
				235,
				125
			)

		task.delay(
			0.12,
			function()
				verifyBusy = false
				unlock()
			end
		)
	else
		playSound(SoundError)

		keyStatus.Text =
			"KEY INVALIDA"

		keyStatus.TextColor3 =
			Color3.fromRGB(
				235,
				85,
				85
			)

		local oldPosition =
			keyBox.Position

		local shakeTween =
			TweenService:Create(
				keyBox,
				TweenInfo.new(
					0.05,
					Enum.EasingStyle.Linear,
					Enum.EasingDirection.Out,
					3,
					true
				),
				{
					Position =
						oldPosition
						+ UDim2.fromOffset(
							4,
							0
						)
				}
			)

		shakeTween:Play()

		task.delay(
			0.35,
			function()
				if keyBox.Parent then
					keyBox.Position =
						oldPosition
				end

				verifyBusy = false
			end
		)
	end
end

-- Clique/touch direto para evitar falhas de Activated em mobile/executor.
connect(
	verifyButton.InputBegan,
	function(input)
		if
			input.UserInputType == Enum.UserInputType.MouseButton1
			or input.UserInputType == Enum.UserInputType.Touch
		then
			verifyKey()
		end
	end
)

-- Enter no TextBox também verifica.
connect(
	keyBox.FocusLost,
	function(enterPressed)
		if enterPressed then
			verifyKey()
		end
	end
)

--==============================================================
-- RGB ANIMATION
--==============================================================

local elapsed = 0

connect(
	RunService.RenderStepped,
	function(dt)
		elapsed =
			(
				elapsed
				+ dt * 32
			)
			% 360

		panelGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		bubbleGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		keyRGB.Rotation =
			(
				135 + elapsed
			)
			% 360

		keyTitleGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		hubTitleGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		keyBoxGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		verifyGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		antiHitGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		premiumGradient.Rotation =
			(
				135 + elapsed
			)
			% 360

		EggBarGradient.Rotation =
			(
				135 + elapsed
			)
			% 360
	end
)

--==============================================================
-- RESIZE
--==============================================================

connect(
	root:GetPropertyChangedSignal(
		"AbsoluteSize"
	),
	resize
)

--==============================================================
-- CLEANUP
--==============================================================

connect(
	gui.Destroying,
	function()
		if fadeTween then
			fadeTween:Cancel()
		end

		if scaleTween then
			scaleTween:Cancel()
		end

		for _, connection in ipairs(connections) do
			connection:Disconnect()
		end
	end
)

task.defer(
	resize
)

print("================================")
print("             DHZ HUB")
print("================================")
print(" KEY: DHZONTOP")
print(" VERIFY: FIXED")
print(" PREMIUM DHZ: READY")
print(" ANTI HIT BUTTON -> ULTRA TP LOGIC: READY")
print(" EGG LOADING FULLSCREEN 2s: READY")
print(" SOUND EFFECTS: READY")
print(" DRAG FIX: READY")
print("================================")
