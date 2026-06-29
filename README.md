local Players = game:GetService("Players")
local Workspace = game:GetService("Workspace")
local RunService = game:GetService("RunService")
local VirtualInputManager = game:GetService("VirtualInputManager")

local player = Players.LocalPlayer

local AUTO_PARRY = true
local KILL_AURA = false
local AUTO_TELE_BOSS = false

local PARRY_DISTANCE = 26
local AURA_DISTANCE = 22

-- GUI Menu
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "PilgrammedHackMenu"
screenGui.ResetOnSpawn = false
screenGui.Parent = player:WaitForChild("PlayerGui")

local mainFrame = Instance.new("Frame")
mainFrame.Size = UDim2.new(0, 380, 0, 420)
mainFrame.Position = UDim2.new(0.5, -190, 0.3, 0)
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 20)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.Parent = screenGui
Instance.new("UICorner", mainFrame).CornerRadius = UDim.new(0, 16)

local titleBar = Instance.new("Frame")
titleBar.Size = UDim2.new(1, 0, 0, 50)
titleBar.BackgroundColor3 = Color3.fromRGB(0, 120, 255)
titleBar.Parent = mainFrame
Instance.new("UICorner", titleBar).CornerRadius = UDim.new(0, 16)

local title = Instance.new("TextLabel")
title.Size = UDim2.new(1, 0, 1, 0)
title.BackgroundTransparency = 1
title.Text = "PILGRAMMED HACK MENU"
title.TextColor3 = Color3.new(1, 1, 1)
title.TextScaled = true
title.Font = Enum.Font.GothamBold
title.Parent = titleBar

local yOffset = 70

local function addToggle(name, default, callback)
    local toggleFrame = Instance.new("Frame")
    toggleFrame.Size = UDim2.new(0.9, 0, 0, 60)
    toggleFrame.Position = UDim2.new(0.05, 0, 0, yOffset)
    toggleFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 35)
    toggleFrame.Parent = mainFrame
    Instance.new("UICorner", toggleFrame).CornerRadius = UDim.new(0, 12)

    local label = Instance.new("TextLabel")
    label.Size = UDim2.new(0.7, 0, 1, 0)
    label.Position = UDim2.new(0, 15, 0, 0)
    label.BackgroundTransparency = 1
    label.Text = name
    label.TextColor3 = Color3.new(1,1,1)
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.TextScaled = true
    label.Font = Enum.Font.Gotham
    label.Parent = toggleFrame

    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0.25, 0, 0.7, 0)
    btn.Position = UDim2.new(0.72, 0, 0.15, 0)
    btn.BackgroundColor3 = default and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
    btn.Text = default and "ON" or "OFF"
    btn.TextColor3 = Color3.new(1,1,1)
    btn.TextScaled = true
    btn.Font = Enum.Font.GothamBold
    btn.Parent = toggleFrame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 10)

    btn.MouseButton1Click:Connect(function()
        default = not default
        btn.Text = default and "ON" or "OFF"
        btn.BackgroundColor3 = default and Color3.fromRGB(0, 200, 0) or Color3.fromRGB(200, 0, 0)
        callback(default)
    end)

    yOffset = yOffset + 75
end

addToggle("Auto Parry", AUTO_PARRY, function(v) AUTO_PARRY = v end)
addToggle("Kill Aura", KILL_AURA, function(v) KILL_AURA = v end)
addToggle("Auto Tele Boss", AUTO_TELE_BOSS, function(v) AUTO_TELE_BOSS = v end)

-- Core Functions
local rootPart = nil
local function getRoot()
    if rootPart and rootPart.Parent then return rootPart end
    local char = player.Character
    rootPart = char and char:FindFirstChild("HumanoidRootPart")
    return rootPart
end

local function mobileParry()
    VirtualInputManager:SendKeyEvent(true, Enum.KeyCode.F, false, game)
    task.wait(0.04)
    VirtualInputManager:SendKeyEvent(false, Enum.KeyCode.F, false, game)
end

local parryParams = RaycastParams.new()
parryParams.FilterType = Enum.RaycastFilterType.Exclude
parryParams.IgnoreWater = true

local function doParry()
    if not AUTO_PARRY then return end
    local root = getRoot()
    if not root then return end
    parryParams.FilterDescendantsInstances = {player.Character or {}}
    local direction = root.CFrame.LookVector * PARRY_DISTANCE
    local result = Workspace:Raycast(root.Position, direction, parryParams)
    if result and result.Instance then
        local n = result.Instance.Name:lower()
        if n:find("attack") or n:find("hitbox") or n:find("projectile") then
            mobileParry()
            task.wait(math.random(0.16, 0.3))
        end
    end
end

local function killAura()
    if not KILL_AURA then return end
    local root = getRoot()
    if not root then return end
    for _, v in ipairs(Workspace:GetChildren()) do
        local hrp = v:FindFirstChild("HumanoidRootPart")
        local hum = v:FindFirstChild("Humanoid")
        if hrp and hum and hum.Health > 0 and v \~= player.Character then
            if (hrp.Position - root.Position).Magnitude <= AURA_DISTANCE then
                local tool = player.Character and player.Character:FindFirstChildWhichIsA("Tool")
                if tool then tool:Activate() end
                task.wait(0.25)
            end
        end
    end
end

local function teleToBoss()
    if not AUTO_TELE_BOSS then return end
    local root = getRoot()
    if not root then return end
    for _, v in ipairs(Workspace:GetChildren()) do
        local hum = v:FindFirstChild("Humanoid")
        local hrp = v:FindFirstChild("HumanoidRootPart")
        if hum and hrp and hum.MaxHealth >= 800 and hum.Health > 0 then
            root.CFrame = CFrame.new(hrp.Position + Vector3.new(0,8,0))
            task.wait(0.8)
            break
        end
    end
end

RunService.Heartbeat:Connect(doParry)
RunService.Heartbeat:Connect(killAura)
RunService.Heartbeat:Connect(teleToBoss)

print("✅ Pilgrammed Hack Menu - READY TO USE!")
