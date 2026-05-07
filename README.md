local TeleportService = game:GetService("TeleportService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Stats = game:GetService("Stats")

local player = Players.LocalPlayer
local playerGui = player:WaitForChild("PlayerGui")

-- ==================== SCREEN WATERMARK ====================
local watermarkGui = Instance.new("ScreenGui")
watermarkGui.Name = "KanyeWatermark"
watermarkGui.IgnoreGuiInset = true
pcall(function() watermarkGui.Parent = (gethui and gethui()) or playerGui end)

local watermark = Instance.new("TextLabel")
watermark.Text = "DISCORD.GG/KANYEHUB"
watermark.Font = Enum.Font.LuckiestGuy
watermark.TextSize = 25
watermark.TextColor3 = Color3.fromRGB(255, 255, 255)
watermark.Size = UDim2.new(0, 300, 0, 50)
watermark.Position = UDim2.new(0.5, -150, 0.5, 120) 
watermark.BackgroundTransparency = 1
watermark.Parent = watermarkGui

-- ==================== TOP PANEL (FPS/PING) ====================
local topGui = Instance.new("ScreenGui")
topGui.Name = "KanyeHopper_Top"
pcall(function() topGui.Parent = (gethui and gethui()) or playerGui end)

local topFrame = Instance.new("Frame")
topFrame.Size = UDim2.new(0, 240, 0, 80) 
topFrame.Position = UDim2.new(0.5, -120, 0.1, 0)
topFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
topFrame.BackgroundTransparency = 0.3 --
topFrame.Parent = topGui
Instance.new("UICorner", topFrame).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", topFrame).Color = Color3.fromRGB(255, 255, 255)

local topTitle = Instance.new("TextLabel")
topTitle.Text = "PUB SERVER HOPPER"
topTitle.Font = Enum.Font.LuckiestGuy
topTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
topTitle.TextSize = 18
topTitle.Size = UDim2.new(1, 0, 0, 30)
topTitle.BackgroundTransparency = 1
topTitle.Parent = topFrame

local statsLabel = Instance.new("TextLabel")
statsLabel.Text = "FPS: 0 | PING: 0MS" --
statsLabel.Font = Enum.Font.LuckiestGuy
statsLabel.TextColor3 = Color3.fromRGB(255, 255, 255)
statsLabel.TextSize = 15
statsLabel.Size = UDim2.new(1, 0, 0, 20)
statsLabel.Position = UDim2.new(0, 0, 0.4, 0)
statsLabel.BackgroundTransparency = 1
statsLabel.Parent = topFrame

local topDiscord = Instance.new("TextLabel")
topDiscord.Text = "DISCORD.GG/KANYEHUB"
topDiscord.Font = Enum.Font.LuckiestGuy
topDiscord.TextColor3 = Color3.fromRGB(255, 255, 255)
topDiscord.TextSize = 15
topDiscord.Position = UDim2.new(0, 0, 0.7, 0)
topDiscord.Size = UDim2.new(1, 0, 0, 20)
topDiscord.BackgroundTransparency = 1
topDiscord.Parent = topFrame

-- ==================== FORCED STATS UPDATER ====================
task.spawn(function()
    while task.wait(0.5) do -- Faster refresh to avoid "--"
        local fps = math.floor(1 / RunService.RenderStepped:Wait())
        local ping = math.floor(Stats.Network.ServerTick.Value)
        
        -- Fallback if ServerTick is weird
        if ping <= 0 then
            ping = math.floor(player:GetNetworkPing() * 1000)
        end
        
        statsLabel.Text = "FPS: " .. tostring(fps) .. " | PING: " .. tostring(ping) .. "MS"
    end
end)

-- ==================== MAIN HOPPER GUI ====================
local mainGui = Instance.new("ScreenGui")
mainGui.Name = "KanyeHopper_Main"
pcall(function() mainGui.Parent = (gethui and gethui()) or playerGui end)

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 200, 0, 85)
mainFrame.Position = UDim2.new(0.65, 0, 0.45, 0) --
mainFrame.BackgroundColor3 = Color3.fromRGB(0, 0, 0)
mainFrame.BackgroundTransparency = 0.3
mainFrame.ClipsDescendants = true
mainFrame.Parent = mainGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 12)
Instance.new("UIStroke", mainFrame).Color = Color3.fromRGB(255, 255, 255)

local mainTitle = Instance.new("TextLabel")
mainTitle.Text = "PUBLIC SERVER HOPPER"
mainTitle.Font = Enum.Font.LuckiestGuy
mainTitle.TextColor3 = Color3.fromRGB(255, 255, 255)
mainTitle.TextSize = 13
mainTitle.Size = UDim2.new(0.8, 0, 0, 30)
mainTitle.Position = UDim2.new(0.05, 0, 0, 5)
mainTitle.BackgroundTransparency = 1
mainTitle.TextXAlignment = "Left"
mainTitle.Parent = mainFrame

local backArrow = Instance.new("TextButton")
backArrow.Text = "◀"
backArrow.Font = Enum.Font.LuckiestGuy
backArrow.TextColor3 = Color3.fromRGB(0, 0, 0)
backArrow.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
backArrow.Size = UDim2.new(0, 20, 0, 20)
backArrow.Position = UDim2.new(0.85, 0, 0.1, 0)
backArrow.Parent = mainFrame
Instance.new("UICorner", backArrow).CornerRadius = UDim.new(0, 5)

local hopButton = Instance.new("TextButton")
hopButton.Text = "HOP SERVER"
hopButton.Font = Enum.Font.LuckiestGuy
hopButton.TextSize = 18
hopButton.TextColor3 = Color3.fromRGB(0, 0, 0)
hopButton.BackgroundColor3 = Color3.fromRGB(255, 255, 255)
hopButton.Size = UDim2.new(0.9, 0, 0, 38)
hopButton.Position = UDim2.new(0.05, 0, 0.45, 0)
hopButton.Parent = mainFrame
Instance.new("UICorner", hopButton).CornerRadius = UDim.new(0, 10)

-- Minimize Logic
local isMinimized = false
backArrow.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    if isMinimized then
        mainFrame:TweenSize(UDim2.new(0, 200, 0, 35), "Out", "Quad", 0.3, true)
        hopButton.Visible = false
        backArrow.Text = "▶"
    else
        mainFrame:TweenSize(UDim2.new(0, 200, 0, 85), "Out", "Quad", 0.3, true)
        task.wait(0.2)
        hopButton.Visible = true
        backArrow.Text = "◀"
    end
end)

-- Hop Function
hopButton.MouseButton1Click:Connect(function()
    hopButton.Text = "HOPPING..."
    local success, servers = pcall(function()
        return HttpService:JSONDecode(game:HttpGet("https://games.roblox.com/v1/games/" .. game.PlaceId .. "/servers/Public?sortOrder=Desc&limit=100")).data
    end)
    
    if success then
        local targetServers = {}
        for _, s in ipairs(servers) do
            if s.playing < s.maxPlayers and s.id ~= game.JobId then
                table.insert(targetServers, s.id)
            end
        end
        if #targetServers > 0 then
            TeleportService:TeleportToPlaceInstance(game.PlaceId, targetServers[math.random(1, #targetServers)])
        else
            hopButton.Text = "SERVER FULL RETRY ERROR"
            task.wait(1)
            hopButton.Text = "HOP SERVER"
        end
    end
