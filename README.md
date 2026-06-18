-- Arsenal Hitbox Expander + ESP - Created by ferhacks
-- Huge hitboxes + See enemies through walls

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local Workspace = game:GetService("Workspace")
local StarterGui = game:GetService("StarterGui")
local Camera = Workspace.CurrentCamera
local LocalPlayer = Players.LocalPlayer

-- Settings
local HitboxEnabled = false
local ESPEnabled = false
local TeamCheck = false
local HitboxSize = 15
local HitboxTransparency = 0.7

-- UI
local ScreenGui = Instance.new("ScreenGui")
ScreenGui.Name = "FerhacksCombo"
ScreenGui.ResetOnSpawn = false
ScreenGui.Parent = LocalPlayer:WaitForChild("PlayerGui")

local Frame = Instance.new("Frame")
Frame.Size = UDim2.new(0, 260, 0, 380)
Frame.Position = UDim2.new(0, 10, 0.3, 0)
Frame.BackgroundColor3 = Color3.fromRGB(25, 25, 25)
Frame.BorderSizePixel = 0
Frame.Active = true
Frame.Parent = ScreenGui

Instance.new("UICorner", Frame).CornerRadius = UDim.new(0, 10)

-- Title
local Title = Instance.new("TextLabel")
Title.Size = UDim2.new(1, 0, 0, 40)
Title.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
Title.Text = "FERHACKS COMBO"
Title.TextColor3 = Color3.fromRGB(255, 100, 100)
Title.TextSize = 18
Title.Font = Enum.Font.GothamBold
Title.Parent = Frame
Instance.new("UICorner", Title).CornerRadius = UDim.new(0, 10)

-- Creator
local Creator = Instance.new("TextLabel")
Creator.Size = UDim2.new(1, 0, 0, 15)
Creator.Position = UDim2.new(0, 0, 0, 37)
Creator.BackgroundTransparency = 1
Creator.Text = "Created by ferhacks"
Creator.TextColor3 = Color3.fromRGB(150, 150, 150)
Creator.TextSize = 10
Creator.Parent = Frame

-- Status
local Status = Instance.new("TextLabel")
Status.Size = UDim2.new(1, 0, 0, 25)
Status.Position = UDim2.new(0, 0, 1, -30)
Status.BackgroundTransparency = 1
Status.Text = "Ready"
Status.TextColor3 = Color3.fromRGB(200, 200, 200)
Status.TextSize = 12
Status.Parent = Frame

-- Button function
local function CreateButton(text, yPos, callback)
    local BtnFrame = Instance.new("Frame")
    BtnFrame.Size = UDim2.new(0.9, 0, 0, 45)
    BtnFrame.Position = UDim2.new(0.05, 0, 0, yPos)
    BtnFrame.BackgroundColor3 = Color3.fromRGB(35, 35, 35)
    BtnFrame.Parent = Frame
    Instance.new("UICorner", BtnFrame).CornerRadius = UDim.new(0, 8)
    
    local Label = Instance.new("TextLabel")
    Label.Size = UDim2.new(0.6, 0, 1, 0)
    Label.Position = UDim2.new(0, 10, 0, 0)
    Label.BackgroundTransparency = 1
    Label.Text = text
    Label.TextColor3 = Color3.new(1, 1, 1)
    Label.TextSize = 13
    Label.Font = Enum.Font.Gotham
    Label.TextXAlignment = Enum.TextXAlignment.Left
    Label.Parent = BtnFrame
    
    local Btn = Instance.new("TextButton")
    Btn.Size = UDim2.new(0, 70, 0, 32)
    Btn.Position = UDim2.new(1, -80, 0.5, -16)
    Btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    Btn.Text = "OFF"
    Btn.TextColor3 = Color3.new(1, 1, 1)
    Btn.TextSize = 13
    Btn.Font = Enum.Font.GothamBold
    Btn.Parent = BtnFrame
    Instance.new("UICorner", Btn).CornerRadius = UDim.new(0, 6)
    
    Btn.MouseButton1Click:Connect(function()
        callback(Btn)
    end)
    
    return Btn
end

-- Hitbox Button
local HitboxBtn = CreateButton("HITBOX", 60, function(btn)
    HitboxEnabled = not HitboxEnabled
    if HitboxEnabled then
        btn.Text = "ON"
        btn.BackgroundColor3 = Color3.fromRGB(255, 50, 50)
        Status.Text = "Hitbox: EXPANDED"
        Status.TextColor3 = Color3.fromRGB(255, 100, 100)
        ExpandAllHitboxes()
    else
        btn.Text = "OFF"
        btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        Status.Text = "Hitbox: NORMAL"
        Status.TextColor3 = Color3.fromRGB(200, 200, 200)
        RestoreAllHitboxes()
    end
end)

-- ESP Button
local ESPBtn = CreateButton("ESP PLAYERS", 115, function(btn)
    ESPEnabled = not ESPEnabled
    if ESPEnabled then
        btn.Text = "ON"
        btn.BackgroundColor3 = Color3.fromRGB(0, 200, 100)
        Status.Text = "ESP: ON"
        Status.TextColor3 = Color3.fromRGB(0, 255, 100)
        UpdateESPState()
    else
        btn.Text = "OFF"
        btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
        Status.Text = "ESP: OFF"
        Status.TextColor3 = Color3.fromRGB(200, 200, 200)
        HideAllESP()
    end
end)

-- Team Check Button
CreateButton("TEAM CHECK", 170, function(btn)
    TeamCheck = not TeamCheck
    if TeamCheck then
        btn.Text = "ON"
        btn.BackgroundColor3 = Color3.fromRGB(0, 150, 200)
    else
        btn.Text = "OFF"
        btn.BackgroundColor3 = Color3.fromRGB(80, 80, 80)
    end
    -- Refresh
    if HitboxEnabled then ExpandAllHitboxes() end
    if ESPEnabled then UpdateESPState() end
end)

-- Size Label
local SizeLabel = Instance.new("TextLabel")
SizeLabel.Size = UDim2.new(1, 0, 0, 20)
SizeLabel.Position = UDim2.new(0, 0, 0, 225)
SizeLabel.BackgroundTransparency = 1
SizeLabel.Text = "Hitbox Size: 15"
SizeLabel.TextColor3 = Color3.fromRGB(200, 200, 200)
SizeLabel.TextSize = 12
SizeLabel.Parent = Frame

-- Size Buttons
local function CreateSizeBtn(text, pos, size)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(0, 60, 0, 30)
    btn.Position = UDim2.new(pos, 0, 0, 250)
    btn.BackgroundColor3 = Color3.fromRGB(60, 60, 60)
    btn.Text = text
    btn.TextColor3 = Color3.new(1, 1, 1)
    btn.TextSize = 11
    btn.Parent = Frame
    Instance.new("UICorner", btn).CornerRadius = UDim.new(0, 6)
    
    btn.MouseButton1Click:Connect(function()
        HitboxSize = size
        SizeLabel.Text = "Hitbox Size: " .. size
        if HitboxEnabled then ExpandAllHitboxes() end
    end)
end

CreateSizeBtn("Small", 0.05, 10)
CreateSizeBtn("Medium", 0.37, 15)
CreateSizeBtn("BIG", 0.7, 25)

-- Info
local Info = Instance.new("TextLabel")
Info.Size = UDim2.new(0.9, 0, 0, 40)
Info.Position = UDim2.new(0.05, 0, 0, 290)
Info.BackgroundTransparency = 1
Info.Text = "ESP: See enemies through walls\nHitbox: Shoot anywhere near them"
Info.TextColor3 = Color3.fromRGB(180, 180, 180)
Info.TextSize = 10
Info.Font = Enum.Font.Gotham
Info.TextWrapped = true
Info.Parent = Frame

-- ============================================
-- HITBOX SYSTEM
-- ============================================
local ExpandedCharacters = {}

local function ExpandHitbox(character)
    if not character then return end
    
    local player = Players:GetPlayerFromCharacter(character)
    if player and TeamCheck and player.Team == LocalPlayer.Team then return end
    if character == LocalPlayer.Character then return end
    if ExpandedCharacters[character] then return end
    
    local humanoid = character:FindFirstChildOfClass("Humanoid")
    if not humanoid or humanoid.Health <= 0 then return end
    
    ExpandedCharacters[character] = {}
    
    -- Head
    local head = character:FindFirstChild("Head")
    if head then
        ExpandedCharacters[character].Head = {Size = head.Size, Massless = head.Massless}
        head.Size = Vector3.new(HitboxSize, HitboxSize, HitboxSize)
        head.Massless = true
    end
    
    -- Torso
    local torso = character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso") or character:FindFirstChild("HumanoidRootPart")
    if torso then
        ExpandedCharacters[character].Torso = {Size = torso.Size, Massless = torso.Massless}
        torso.Size = Vector3.new(HitboxSize * 0.8, HitboxSize * 1.2, HitboxSize * 0.8)
        torso.Massless = true
    end
end

local function RestoreHitbox(character)
    if not ExpandedCharacters[character] then return end
    
    local data = ExpandedCharacters[character]
    
    if data.Head then
        local head = character:FindFirstChild("Head")
        if head then
            head.Size = data.Head.Size
            head.Massless = data.Head.Massless
        end
    end
    
    if data.Torso then
        local torso = character:FindFirstChild("Torso") or character:FindFirstChild("UpperTorso") or character:FindFirstChild("HumanoidRootPart")
        if torso then
            torso.Size = data.Torso.Size
            torso.Massless = data.Torso.Massless
        end
    end
    
    ExpandedCharacters[character] = nil
end

function ExpandAllHitboxes()
    for _, player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and player.Character then
            ExpandHitbox(player.Character)
        end
    end
end

function RestoreAllHitboxes()
    for character, _ in pairs(ExpandedCharacters) do
        if character and character.Parent then
            RestoreHitbox(character)
        end
    end
    ExpandedCharacters = {}
end

-- ============================================
-- ESP SYSTEM
-- ============================================
local ESPObjects = {}

local function CreateESP(player)
    if ESPObjects[player] then return end
    
    local esp = {}
    
    -- Highlight (red glow through walls)
    local highlight = Instance.new("Highlight")
    highlight.FillColor = Color3.fromRGB(255, 0, 0)
    highlight.OutlineColor = Color3.new(1, 1, 1)
    highlight.FillTransparency = 0.6
    highlight.OutlineTransparency = 0
    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
    highlight.Enabled = false
    highlight.Parent = game.CoreGui
    esp.Highlight = highlight
    
    -- Billboard (name, health, distance)
    local billboard = Instance.new("BillboardGui")
    billboard.Size = UDim2.new(0, 120, 0, 60)
    billboard.StudsOffset = Vector3.new(0, 3, 0)
    billboard.AlwaysOnTop = true
    billboard.Enabled = false
    
    -- Name
    local nameLabel = Instance.new("TextLabel")
    nameLabel.Size = UDim2.new(1, 0, 0.33, 0)
    nameLabel.BackgroundTransparency = 1
    nameLabel.TextColor3 = Color3.new(1, 0, 0)
    nameLabel.TextStrokeTransparency = 0
    nameLabel.Font = Enum.Font.GothamBold
    nameLabel.TextSize = 14
    nameLabel.Parent = billboard
    
    -- Health
    local hpLabel = Instance.new("TextLabel")
    hpLabel.Size = UDim2.new(1, 0, 0.33, 0)
    hpLabel.Position = UDim2.new(0, 0, 0.33, 0)
    hpLabel.BackgroundTransparency = 1
    hpLabel.TextColor3 = Color3.new(0, 1, 0)
    hpLabel.TextStrokeTransparency = 0
    hpLabel.Font = Enum.Font.Gotham
    hpLabel.TextSize = 12
    hpLabel.Parent = billboard
    
    -- Distance
    local distLabel = Instance.new("TextLabel")
    distLabel.Size = UDim2.new(1, 0, 0.34, 0)
    distLabel.Position = UDim2.new(0, 0, 0.66, 0)
    distLabel.BackgroundTransparency = 1
    distLabel.TextColor3 = Color3.new(1, 1, 0)
    distLabel.TextStrokeTransparency = 0
    distLabel.Font = Enum.Font.Gotham
    distLabel.TextSize = 11
    distLabel.Parent = billboard
    
    billboard.Parent = game.CoreGui
    
    esp.Billboard = billboard
    esp.NameLabel = nameLabel
    esp.HpLabel = hpLabel
    esp.DistLabel = distLabel
    
    ESPObjects[player] = esp
end

function UpdateESPState()
    for player, esp in pairs(ESPObjects) do
        if player.Character then
            local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
            local head = player.Character:FindFirstChild("Head")
            local root = player.Character:FindFirstChild("HumanoidRootPart")
            
            if humanoid and humanoid.Health > 0 then
                -- Team check
                if TeamCheck and player.Team == LocalPlayer.Team then
                    esp.Highlight.Enabled = false
                    esp.Billboard.Enabled = false
                    continue
                end
                
                esp.Highlight.Adornee = player.Character
                esp.Highlight.Enabled = true
                
                if head then
                    esp.Billboard.Adornee = head
                    esp.Billboard.Enabled = true
                    
                    esp.NameLabel.Text = player.Name
                    esp.HpLabel.Text = math.floor(humanoid.Health) .. "/" .. math.floor(humanoid.MaxHealth) .. " HP"
                    
                    if root then
                        local dist = (Camera.CFrame.Position - root.Position).Magnitude
                        esp.DistLabel.Text = math.floor(dist) .. " studs"
                    end
                end
            else
                esp.Highlight.Enabled = false
                esp.Billboard.Enabled = false
            end
        else
            esp.Highlight.Enabled = false
            esp.Billboard.Enabled = false
        end
    end
end

function HideAllESP()
    for _, esp in pairs(ESPObjects) do
        esp.Highlight.Enabled = false
        esp.Billboard.Enabled = false
    end
end

-- Initialize ESP
for _, p in ipairs(Players:GetPlayers()) do
    if p ~= LocalPlayer then CreateESP(p) end
end

Players.PlayerAdded:Connect(function(p)
    CreateESP(p)
    p.CharacterAdded:Connect(function(char)
        wait(0.5)
        if HitboxEnabled then ExpandHitbox(char) end
    end)
end)

-- ============================================
-- MAIN LOOP
-- ============================================
RunService.Heartbeat:Connect(function()
    -- Update Hitbox
    if HitboxEnabled then
        for _, player in ipairs(Players:GetPlayers()) do
            if player ~= LocalPlayer and player.Character then
                if not ExpandedCharacters[player.Character] then
                    local humanoid = player.Character:FindFirstChildOfClass("Humanoid")
                    if humanoid and humanoid.Health > 0 then
                        ExpandHitbox(player.Character)
                    end
                end
            end
        end
        
        -- Cleanup dead
        for char, _ in pairs(ExpandedCharacters) do
            if char and char.Parent then
                local humanoid = char:FindFirstChildOfClass("Humanoid")
                if not humanoid or humanoid.Health <= 0 then
                    RestoreHitbox(char)
                end
            else
                ExpandedCharacters[char] = nil
            end
        end
    end
    
    -- Update ESP
    if ESPEnabled then
        UpdateESPState()
    end
end)

-- Cleanup
LocalPlayer.CharacterRemoving:Connect(function()
    RestoreAllHitboxes()
    HideAllESP()
end)

-- Notification
pcall(function()
    StarterGui:SetCore("SendNotification", {
        Title = "FERHACKS Combo",
        Text = "Hitbox: Shoot anywhere near enemies\nESP: See through walls",
        Duration = 5
    })
end)

print("=== FERHACKS COMBO LOADED ===")
print("Hitbox: Toggle to expand enemy hitboxes")
print("ESP: Toggle to see enemies through walls")
