local FIREBASE_URL = "..."

local TweenService = game:GetService("TweenService")
local UserInputService = game:GetService("UserInputService")
local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Workspace = game:GetService("Workspace")
local Players = game:GetService("Players")
local Player = Players.LocalPlayer

local CoreGui = (gethui and gethui()) or game:GetService("CoreGui")
local GuiName = "MobyNotifierGui"

for _, v in pairs(CoreGui:GetChildren()) do
    if v.Name == GuiName then v:Destroy() end
end

local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = GuiName
ScreenGui.Parent = CoreGui
ScreenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
ScreenGui.ResetOnSpawn = false

local ClickSound = Instance.new("Sound", ScreenGui)
ClickSound.SoundId = "rbxassetid://75311202481026"
ClickSound.Volume = 0.3
local function playClick() pcall(function() ClickSound:Play() end) end

local isMobile = UserInputService.TouchEnabled and not UserInputService.MouseEnabled
local TARGET_SCALE = isMobile and 0.8 or 1.0
local HIDE_SCALE = TARGET_SCALE - 0.15

local Frame = Instance.new("CanvasGroup", ScreenGui)
Frame.BackgroundColor3 = Color3.fromRGB(5, 5, 5)
Frame.BorderColor3 = Color3.fromRGB(0, 0, 0)
Frame.BorderSizePixel = 0
Frame.Position = UDim2.new(0.5, -303, 0.5, -182)
Frame.Size = UDim2.new(0, 606, 0, 365)
Frame.Active = true
Frame.GroupTransparency = 1

local UICorner = Instance.new("UICorner", Frame)
UICorner.CornerRadius = UDim.new(0, 9)
local UIScale = Instance.new("UIScale", Frame)
UIScale.Scale = HIDE_SCALE

TweenService:Create(Frame, TweenInfo.new(0.4, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {GroupTransparency = 0}):Play()
TweenService:Create(UIScale, TweenInfo.new(0.5, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {Scale = TARGET_SCALE}):Play()

local MobyNotifier = Instance.new("TextLabel", Frame)
MobyNotifier.BackgroundTransparency = 1
MobyNotifier.Position = UDim2.new(0, 15, 0, 10)
MobyNotifier.Size = UDim2.new(0, 150, 0, 30)
MobyNotifier.Font = Enum.Font.GothamBold
MobyNotifier.Text = "Moby Notifier"
MobyNotifier.TextColor3 = Color3.fromRGB(255, 255, 255)
MobyNotifier.TextSize = 20
MobyNotifier.TextXAlignment = Enum.TextXAlignment.Left

local MobyNotifier_2 = Instance.new("TextLabel", Frame)
MobyNotifier_2.BackgroundTransparency = 1
MobyNotifier_2.Position = UDim2.new(0, 15, 0, 35)
MobyNotifier_2.Size = UDim2.new(0, 130, 0, 20)
MobyNotifier_2.Font = Enum.Font.GothamMedium
MobyNotifier_2.Text = ".gg/mobynotifier"
MobyNotifier_2.TextColor3 = Color3.fromRGB(122, 122, 122)
MobyNotifier_2.TextSize = 14
MobyNotifier_2.TextXAlignment = Enum.TextXAlignment.Left

local tweenInfo = TweenInfo.new(0.15, Enum.EasingStyle.Quad, Enum.EasingDirection.Out)
local dragTweenInfo = TweenInfo.new(0.08, Enum.EasingStyle.Quart, Enum.EasingDirection.Out)

local ContentArea = Instance.new("Frame", Frame)
ContentArea.BackgroundTransparency = 1
ContentArea.Position = UDim2.new(0, 160, 0, 75)
ContentArea.Size = UDim2.new(0, 420, 0, 275)

local MainPage = Instance.new("Frame", ContentArea)
MainPage.BackgroundTransparency = 1
MainPage.Size = UDim2.new(1, 0, 1, 0)

local LogScroll = Instance.new("ScrollingFrame", MainPage)
LogScroll.Active = true
LogScroll.BackgroundTransparency = 1
LogScroll.BorderSizePixel = 0
LogScroll.Size = UDim2.new(1, 0, 1, -10)
LogScroll.ScrollBarThickness = 2
LogScroll.ScrollBarImageColor3 = Color3.fromRGB(50, 50, 50)
LogScroll.AutomaticCanvasSize = Enum.AutomaticSize.Y

local LogLayout = Instance.new("UIListLayout", LogScroll)
LogLayout.SortOrder = Enum.SortOrder.LayoutOrder
LogLayout.Padding = UDim.new(0, 8)

local LogEntries = {}
local CurrentFilter = "AJ"
local RenderedIDs = {}

local function ApplyFilter()
    for _, entry in ipairs(LogEntries) do
        local v = entry.NumericValue
        if CurrentFilter == "AJ" then
            entry.UI.Visible = true
        elseif CurrentFilter == "10m+" then
            entry.UI.Visible = (v >= 10000000 and v < 50000000)
        elseif CurrentFilter == "50m+" then
            entry.UI.Visible = (v >= 50000000 and v < 100000000)
        elseif CurrentFilter == "100m+" then
            entry.UI.Visible = (v >= 100000000)
        else
            entry.UI.Visible = false
        end
    end
end

local function JoinServer(placeId, jobId)
    pcall(function() TeleportService:TeleportToPlaceInstance(tonumber(placeId), jobId, Player) end)
end

local function ForceServer(placeId, jobId)
    task.spawn(function()
        for i = 1, 50 do
            pcall(function() TeleportService:TeleportToPlaceInstance(tonumber(placeId), jobId, Player) end)
            task.wait(2.5)
        end
    end)
end

local function CreateLogNotification(dbKey, brainrotName, moneyStr, numVal, jobId, placeId)
    if RenderedIDs[dbKey] then return end
    RenderedIDs[dbKey] = true

    local LogItem = Instance.new("Frame", LogScroll)
    LogItem.BackgroundTransparency = 1
    LogItem.Size = UDim2.new(1, -10, 0, 45)

    local line = Instance.new("Frame", LogItem)
    line.BackgroundColor3 = Color3.fromRGB(20, 20, 20)
    line.BorderSizePixel = 0
    line.Size = UDim2.new(1, 0, 0, 1)
    line.Position = UDim2.new(0, 0, 1, -1)

    local Left = Instance.new("Frame", LogItem)
    Left.BackgroundTransparency = 1
    Left.Size = UDim2.new(0.6, 0, 1, 0)
    Left.Position = UDim2.new(0, 5, 0, 0)

    local LL = Instance.new("UIListLayout", Left)
    LL.FillDirection = Enum.FillDirection.Horizontal
    LL.VerticalAlignment = Enum.VerticalAlignment.Center
    LL.Padding = UDim.new(0, 8)

    local Icon = Instance.new("ImageLabel", Left)
    Icon.Size = UDim2.new(0, 14, 0, 14)
    Icon.BackgroundTransparency = 1
    Icon.Image = "rbxassetid://136959386531965"
    Icon.ImageColor3 = Color3.fromRGB(255, 255, 255)

    local Name = Instance.new("TextLabel", Left)
    Name.BackgroundTransparency = 1
    Name.AutomaticSize = Enum.AutomaticSize.X
    Name.Font = Enum.Font.GothamMedium
    Name.Text = brainrotName
    Name.TextColor3 = Color3.fromRGB(200, 200, 200)
    Name.TextSize = 12

    local Money = Instance.new("TextLabel", Left)
    Money.BackgroundTransparency = 1
    Money.AutomaticSize = Enum.AutomaticSize.X
    Money.Font = Enum.Font.GothamBold
    Money.Text = moneyStr
    Money.TextColor3 = Color3.fromRGB(255, 255, 255)
    Money.TextSize = 13

    local Right = Instance.new("Frame", LogItem)
    Right.BackgroundTransparency = 1
    Right.Size = UDim2.new(0.4, 0, 1, 0)
    Right.Position = UDim2.new(0.6, -5, 0, 0)

    local RL = Instance.new("UIListLayout", Right)
    RL.FillDirection = Enum.FillDirection.Horizontal
    RL.HorizontalAlignment = Enum.HorizontalAlignment.Right
    RL.VerticalAlignment = Enum.VerticalAlignment.Center
    RL.Padding = UDim.new(0, 8)

    local jBtn = Instance.new("TextButton", Right)
    jBtn.BackgroundColor3 = Color3.fromRGB(0, 106, 255)
    jBtn.Size = UDim2.new(0, 50, 0, 26)
    jBtn.Font = Enum.Font.GothamBold
    jBtn.Text = "JOIN"
    jBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    jBtn.TextSize = 11
    Instance.new("UICorner", jBtn).CornerRadius = UDim.new(0, 6)
    jBtn.MouseEnter:Connect(function() TweenService:Create(jBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(30, 130, 255)}):Play() end)
    jBtn.MouseLeave:Connect(function() TweenService:Create(jBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(0, 106, 255)}):Play() end)
    jBtn.MouseButton1Click:Connect(function() playClick() JoinServer(placeId, jobId) end)

    local fBtn = Instance.new("TextButton", Right)
    fBtn.BackgroundColor3 = Color3.fromRGB(0, 106, 255)
    fBtn.Size = UDim2.new(0, 60, 0, 26)
    fBtn.Font = Enum.Font.GothamBold
    fBtn.Text = "FORCE"
    fBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
    fBtn.TextSize = 11
    Instance.new("UICorner", fBtn).CornerRadius = UDim.new(0, 6)
    fBtn.MouseEnter:Connect(function() TweenService:Create(fBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(30, 130, 255)}):Play() end)
    fBtn.MouseLeave:Connect(function() TweenService:Create(fBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(0, 106, 255)}):Play() end)
    fBtn.MouseButton1Click:Connect(function() playClick() ForceServer(placeId, jobId) end)

    table.insert(LogEntries, {NumericValue = numVal, UI = LogItem, PlaceId = placeId, JobId = jobId})
    ApplyFilter()
end

local CommunityPage = Instance.new("Frame", ContentArea)
CommunityPage.BackgroundTransparency = 1
CommunityPage.Size = UDim2.new(1, 0, 1, 0)
CommunityPage.Visible = false

-- TITOLO PROFILE & COMMUNITY
local ProfileTitle = Instance.new("TextLabel", CommunityPage)
ProfileTitle.BackgroundTransparency = 1
ProfileTitle.Position = UDim2.new(0, 0, 0, 0)
ProfileTitle.Size = UDim2.new(1, -20, 0, 30)
ProfileTitle.Font = Enum.Font.GothamBold
ProfileTitle.Text = "Profile & Community"
ProfileTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
ProfileTitle.TextSize = 18
ProfileTitle.TextXAlignment = Enum.TextXAlignment.Left

local ProfileCard = Instance.new("Frame", CommunityPage)
ProfileCard.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
ProfileCard.Position = UDim2.new(0, 0, 0, 35)
ProfileCard.Size = UDim2.new(1, -20, 0, 80)
Instance.new("UICorner", ProfileCard).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", ProfileCard).Color = Color3.fromRGB(25, 25, 25)

local AvatarOuter = Instance.new("Frame", ProfileCard)
AvatarOuter.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
AvatarOuter.Position = UDim2.new(0, 15, 0.5, -24)
AvatarOuter.Size = UDim2.new(0, 48, 0, 48)
Instance.new("UICorner", AvatarOuter).CornerRadius = UDim.new(1, 0)

local AvatarImage = Instance.new("ImageLabel", AvatarOuter)
AvatarImage.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
AvatarImage.Position = UDim2.new(0.5, 0, 0.5, 0)
AvatarImage.AnchorPoint = Vector2.new(0.5, 0.5)
AvatarImage.Size = UDim2.new(1, -2, 1, -2)
AvatarImage.Image = "rbxthumb://type=AvatarHeadShot&id=".. Player.UserId .."&w=150&h=150"
Instance.new("UICorner", AvatarImage).CornerRadius = UDim.new(1, 0)

-- CONTAINER PER NOME + BADGE
local NameContainer = Instance.new("Frame", ProfileCard)
NameContainer.BackgroundTransparency = 1
NameContainer.Position = UDim2.new(0, 75, 0, 15)
NameContainer.Size = UDim2.new(0, 300, 0, 24)

local NameLayout = Instance.new("UIListLayout", NameContainer)
NameLayout.FillDirection = Enum.FillDirection.Horizontal
NameLayout.VerticalAlignment = Enum.VerticalAlignment.Center
NameLayout.Padding = UDim.new(0, 8)

local DisplayNameText = Instance.new("TextLabel", NameContainer)
DisplayNameText.BackgroundTransparency = 1
DisplayNameText.AutomaticSize = Enum.AutomaticSize.X
DisplayNameText.Size = UDim2.new(0, 0, 0, 24)
DisplayNameText.Font = Enum.Font.GothamBlack
DisplayNameText.Text = Player.DisplayName
DisplayNameText.TextColor3 = Color3.fromRGB(255, 255, 255)
DisplayNameText.TextSize = 17
DisplayNameText.TextXAlignment = Enum.TextXAlignment.Left
DisplayNameText.LayoutOrder = 1

-- BADGE BUYER
local BuyerBadge = Instance.new("Frame", NameContainer)
BuyerBadge.BackgroundColor3 = Color3.fromRGB(45, 35, 15)
BuyerBadge.Size = UDim2.new(0, 75, 0, 20)
BuyerBadge.LayoutOrder = 2
Instance.new("UICorner", BuyerBadge).CornerRadius = UDim.new(0, 4)

local BuyerStroke = Instance.new("UIStroke", BuyerBadge)
BuyerStroke.Color = Color3.fromRGB(255, 185, 50)
BuyerStroke.Thickness = 1

local BuyerText = Instance.new("TextLabel", BuyerBadge)
BuyerText.BackgroundTransparency = 1
BuyerText.Size = UDim2.new(1, 0, 1, 0)
BuyerText.Font = Enum.Font.GothamBold
BuyerText.Text = "👑 BUYER"
BuyerText.TextColor3 = Color3.fromRGB(255, 185, 50)
BuyerText.TextSize = 11

-- ANIMAZIONE GLOW PER BADGE
task.spawn(function()
    while BuyerBadge and BuyerBadge.Parent do
        TweenService:Create(BuyerStroke, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Color = Color3.fromRGB(255, 220, 100)}):Play()
        task.wait(1)
        TweenService:Create(BuyerStroke, TweenInfo.new(1, Enum.EasingStyle.Sine, Enum.EasingDirection.InOut), {Color = Color3.fromRGB(255, 150, 30)}):Play()
        task.wait(1)
    end
end)

local UsernameText = Instance.new("TextLabel", ProfileCard)
UsernameText.BackgroundTransparency = 1
UsernameText.Position = UDim2.new(0, 75, 0, 42)
UsernameText.Size = UDim2.new(0, 200, 0, 15)
UsernameText.Font = Enum.Font.GothamMedium
UsernameText.Text = "@" .. Player.Name
UsernameText.TextColor3 = Color3.fromRGB(130, 130, 130)
UsernameText.TextSize = 13
UsernameText.TextXAlignment = Enum.TextXAlignment.Left

local DiscordCard = Instance.new("Frame", CommunityPage)
DiscordCard.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
DiscordCard.Position = UDim2.new(0, 0, 0, 130)
DiscordCard.Size = UDim2.new(1, -20, 0, 60)
Instance.new("UICorner", DiscordCard).CornerRadius = UDim.new(0, 8)
Instance.new("UIStroke", DiscordCard).Color = Color3.fromRGB(25, 25, 25)

local DcIcon = Instance.new("ImageLabel", DiscordCard)
DcIcon.BackgroundTransparency = 1
DcIcon.Position = UDim2.new(0, 20, 0.5, -12)
DcIcon.Size = UDim2.new(0, 24, 0, 24)
DcIcon.Image = "rbxassetid://77743122983414"

local DcText = Instance.new("TextLabel", DiscordCard)
DcText.BackgroundTransparency = 1
DcText.Position = UDim2.new(0, 60, 0, 0)
DcText.Size = UDim2.new(0, 150, 1, 0)
DcText.Font = Enum.Font.GothamBold
DcText.Text = "Join our Discord"
DcText.TextColor3 = Color3.fromRGB(255, 255, 255)
DcText.TextSize = 15
DcText.TextXAlignment = Enum.TextXAlignment.Left

local CopyBtn = Instance.new("TextButton", DiscordCard)
CopyBtn.BackgroundColor3 = Color3.fromRGB(88, 101, 242)
CopyBtn.Position = UDim2.new(1, -85, 0.5, -14)
CopyBtn.Size = UDim2.new(0, 70, 0, 28)
CopyBtn.Font = Enum.Font.GothamBold
CopyBtn.Text = "Copy"
CopyBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
CopyBtn.TextSize = 12
Instance.new("UICorner", CopyBtn).CornerRadius = UDim.new(0, 6)

CopyBtn.MouseEnter:Connect(function() TweenService:Create(CopyBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(100, 115, 255)}):Play() end)
CopyBtn.MouseLeave:Connect(function() TweenService:Create(CopyBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(88, 101, 242)}):Play() end)
CopyBtn.MouseButton1Click:Connect(function()
    playClick()
    pcall(function() setclipboard("https://discord.gg/mobynotifier") end)
    CopyBtn.Text = "Copied!"
    TweenService:Create(CopyBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(46, 204, 113)}):Play()
    task.wait(1.5)
    CopyBtn.Text = "Copy"
    TweenService:Create(CopyBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(88, 101, 242)}):Play()
end)

local function showMainPage()
    CommunityPage.Visible = false
    MainPage.Visible = true
end

local function showProfilePage()
    MainPage.Visible = false
    CommunityPage.Visible = true
end

local TopControls = Instance.new("Frame", Frame)
TopControls.BackgroundColor3 = Color3.fromRGB(12, 12, 12)
TopControls.AnchorPoint = Vector2.new(1, 0)
TopControls.Position = UDim2.new(1, -15, 0, 10)
TopControls.Size = UDim2.new(0, 96, 0, 30)
Instance.new("UICorner", TopControls).CornerRadius = UDim.new(0, 6)
Instance.new("UIStroke", TopControls).Color = Color3.fromRGB(25, 25, 25)

local TopLayout = Instance.new("UIListLayout", TopControls)
TopLayout.FillDirection = Enum.FillDirection.Horizontal
TopLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center
TopLayout.VerticalAlignment = Enum.VerticalAlignment.Center
TopLayout.SortOrder = Enum.SortOrder.LayoutOrder

-- SETTINGS (icona ingranaggio) - PRIMO
local SettingsBtn = Instance.new("TextButton", TopControls)
SettingsBtn.Name = "Settings"
SettingsBtn.BackgroundTransparency = 1
SettingsBtn.Size = UDim2.new(0, 32, 0, 30)
SettingsBtn.Text = ""
SettingsBtn.LayoutOrder = 1

local SettingsIcon = Instance.new("ImageLabel", SettingsBtn)
SettingsIcon.BackgroundTransparency = 1
SettingsIcon.AnchorPoint = Vector2.new(0.5, 0.5)
SettingsIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
SettingsIcon.Size = UDim2.new(0, 16, 0, 16)
SettingsIcon.Image = "rbxassetid://110986349331865"
SettingsIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)

SettingsBtn.MouseEnter:Connect(function() TweenService:Create(SettingsIcon, tweenInfo, {ImageColor3 = Color3.fromRGB(180, 180, 180)}):Play() end)
SettingsBtn.MouseLeave:Connect(function() TweenService:Create(SettingsIcon, tweenInfo, {ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play() end)
SettingsBtn.MouseButton1Click:Connect(function() playClick() showMainPage() end)

-- COMMUNITY (icona persone) - SECONDO
local CommunityBtn = Instance.new("TextButton", TopControls)
CommunityBtn.Name = "Community"
CommunityBtn.BackgroundTransparency = 1
CommunityBtn.Size = UDim2.new(0, 32, 0, 30)
CommunityBtn.Text = ""
CommunityBtn.LayoutOrder = 2

local CommunityIcon = Instance.new("ImageLabel", CommunityBtn)
CommunityIcon.BackgroundTransparency = 1
CommunityIcon.AnchorPoint = Vector2.new(0.5, 0.5)
CommunityIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
CommunityIcon.Size = UDim2.new(0, 16, 0, 16)
CommunityIcon.Image = "rbxassetid://116227018364238"
CommunityIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)

CommunityBtn.MouseEnter:Connect(function() TweenService:Create(CommunityIcon, tweenInfo, {ImageColor3 = Color3.fromRGB(180, 180, 180)}):Play() end)
CommunityBtn.MouseLeave:Connect(function() TweenService:Create(CommunityIcon, tweenInfo, {ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play() end)
CommunityBtn.MouseButton1Click:Connect(function() playClick() showProfilePage() end)

-- CLOSE (icona X) - TERZO
local CloseBtn = Instance.new("TextButton", TopControls)
CloseBtn.Name = "Close"
CloseBtn.BackgroundTransparency = 1
CloseBtn.Size = UDim2.new(0, 32, 0, 30)
CloseBtn.Text = ""
CloseBtn.LayoutOrder = 3

local CloseIcon = Instance.new("ImageLabel", CloseBtn)
CloseIcon.BackgroundTransparency = 1
CloseIcon.AnchorPoint = Vector2.new(0.5, 0.5)
CloseIcon.Position = UDim2.new(0.5, 0, 0.5, 0)
CloseIcon.Size = UDim2.new(0, 16, 0, 16)
CloseIcon.Image = "rbxassetid://119410757402001"
CloseIcon.ImageColor3 = Color3.fromRGB(255, 255, 255)

CloseBtn.MouseEnter:Connect(function() TweenService:Create(CloseIcon, tweenInfo, {ImageColor3 = Color3.fromRGB(180, 180, 180)}):Play() end)
CloseBtn.MouseLeave:Connect(function() TweenService:Create(CloseIcon, tweenInfo, {ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play() end)
CloseBtn.MouseButton1Click:Connect(function()
    playClick()
    local closeTweenScale = TweenService:Create(UIScale, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {Scale = HIDE_SCALE})
    local closeTweenFade = TweenService:Create(Frame, TweenInfo.new(0.3, Enum.EasingStyle.Quint, Enum.EasingDirection.In), {GroupTransparency = 1})
    closeTweenScale:Play()
    closeTweenFade:Play()
    closeTweenScale.Completed:Wait()
    ScreenGui:Destroy()
end)

local sidebarButtons = {}
local function createSidebarButton(text, yPos, filter, default)
    local btn = Instance.new("TextButton", Frame)
    btn.Position = UDim2.new(0, 15, 0, yPos)
    btn.Size = UDim2.new(0, 130, 0, 30)
    btn.BackgroundColor3 = default and Color3.fromRGB(0, 106, 255) or Color3.fromRGB(12, 12, 12)
    btn.Text = text
    btn.TextColor3 = default and Color3.fromRGB(255, 255, 255) or Color3.fromRGB(122, 122, 122)
    btn.Font = Enum.Font.GothamBold
    btn.TextSize = 14
    btn.TextXAlignment = Enum.TextXAlignment.Left
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 8)

    local icon = Instance.new("ImageLabel", btn)
    icon.Position = UDim2.new(0, -24, 0.5, 0)
    icon.AnchorPoint = Vector2.new(0, 0.5)
    icon.Size = UDim2.new(0, 16, 0, 16)
    icon.BackgroundTransparency = 1
    icon.Image = "rbxassetid://126446801335121"
    icon.ImageColor3 = Color3.fromRGB(255, 255, 255)

    local stroke = Instance.new("UIStroke", btn)
    stroke.Color = default and Color3.fromRGB(0, 106, 255) or Color3.fromRGB(25, 25, 25)

    local padding = Instance.new("UIPadding", btn)
    padding.PaddingLeft = UDim.new(0, 30)

    local data = {Button = btn, Stroke = stroke, IsActive = default}
    table.insert(sidebarButtons, data)

    btn.MouseButton1Click:Connect(function()
        playClick()
        showMainPage()
        CurrentFilter = filter
        for _, d in ipairs(sidebarButtons) do
            if d.Button ~= btn then
                d.IsActive = false
                TweenService:Create(d.Button, tweenInfo, {BackgroundColor3 = Color3.fromRGB(12, 12, 12), TextColor3 = Color3.fromRGB(122, 122, 122)}):Play()
                TweenService:Create(d.Stroke, tweenInfo, {Color = Color3.fromRGB(25, 25, 25)}):Play()
            end
        end
        data.IsActive = true
        TweenService:Create(btn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(0, 106, 255), TextColor3 = Color3.fromRGB(255, 255, 255)}):Play()
        TweenService:Create(stroke, tweenInfo, {Color = Color3.fromRGB(0, 106, 255)}):Play()
        ApplyFilter()
    end)
end

createSidebarButton("100m+", 75, "100m+", false)
createSidebarButton("50m+", 111, "50m+", false)
createSidebarButton("10m+", 147, "10m+", false)
createSidebarButton("AJ", 183, "AJ", true)

local joinBtn = Instance.new("TextButton", Frame)
joinBtn.BackgroundColor3 = Color3.fromRGB(0, 106, 255)
joinBtn.Position = UDim2.new(0, 15, 0, 320)
joinBtn.Size = UDim2.new(0, 130, 0, 30)
joinBtn.Font = Enum.Font.GothamBold
joinBtn.Text = "Auto Joiner"
joinBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
joinBtn.TextSize = 14
joinBtn.TextXAlignment = Enum.TextXAlignment.Left
Instance.new("UICorner", joinBtn).CornerRadius = UDim.new(0, 8)
local p_join = Instance.new("UIPadding", joinBtn)
p_join.PaddingLeft = UDim.new(0, 30)

local joinIcon = Instance.new("ImageLabel", joinBtn)
joinIcon.BackgroundTransparency = 1
joinIcon.Position = UDim2.new(0, -24, 0.5, 0)
joinIcon.AnchorPoint = Vector2.new(0, 0.5)
joinIcon.Size = UDim2.new(0, 16, 0, 16)
joinIcon.Image = "rbxassetid://130912924018715"

joinBtn.MouseEnter:Connect(function() TweenService:Create(joinBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(30, 130, 255)}):Play() end)
joinBtn.MouseLeave:Connect(function() TweenService:Create(joinBtn, tweenInfo, {BackgroundColor3 = Color3.fromRGB(0, 106, 255)}):Play() end)
joinBtn.MouseButton1Click:Connect(function()
    playClick()
    local best = nil
    local maxVal = -1
    for _, e in ipairs(LogEntries) do
        if e.NumericValue > maxVal then
            maxVal = e.NumericValue
            best = e
        end
    end
    if best then ForceServer(best.PlaceId, best.JobId) end
end)

-- SISTEMA DI DRAG FLUIDO (PC + MOBILE)
local dragging = false
local dragStart = nil
local startPos = nil
local dragInput = nil

local function updateDrag(input)
    if not dragging or not dragStart or not startPos then return end
    local delta = input.Position - dragStart
    local targetPos = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    TweenService:Create(Frame, dragTweenInfo, {Position = targetPos}):Play()
end

Frame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = Frame.Position
        
        TweenService:Create(UIScale, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {Scale = TARGET_SCALE * 0.98}):Play()
        
        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
                TweenService:Create(UIScale, TweenInfo.new(0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {Scale = TARGET_SCALE}):Play()
            end
        end)
    end
end)

Frame.InputChanged:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseMovement or input.UserInputType == Enum.UserInputType.Touch then
        dragInput = input
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if input == dragInput then
        updateDrag(input)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 or input.UserInputType == Enum.UserInputType.Touch then
        dragging = false
        TweenService:Create(UIScale, TweenInfo.new(0.2, Enum.EasingStyle.Quint, Enum.EasingDirection.Out), {Scale = TARGET_SCALE}):Play()
    end
end)

showMainPage()

task.spawn(function()
    while task.wait(2) do
        if not Workspace:FindFirstChild("Debris") then
            local d = Instance.new("Folder")
            d.Name = "Debris"
            d.Parent = Workspace
        end
    end
end)

task.spawn(function()
    while task.wait(3) do
        pcall(function()
            local req = game:HttpGet(FIREBASE_URL)
            if req and req ~= "null" then
                local data = HttpService:JSONDecode(req)
                for key, info in pairs(data) do
                    if type(info) == "table" and info.jobId and info.jobId ~= "" then
                        CreateLogNotification(key, info.name, info.value, info.numValue, info.jobId, info.placeId)
                    end
                end
            end
        end)
    end
end)
