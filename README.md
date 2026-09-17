--[[
    SCRIPT PARA MURDER MYSTERY 2 - DELTA EXECUTOR
    NOME: Lucas Lua
    VERSÃO: 2.0
    
    COMO USAR:
    loadstring(game:HttpGet('https://raw.githubusercontent.com/SEU_USUARIO/SEU_REPO/main/lucas_lua_mm2.lua'))()
--]]

-- ==================== INICIALIZAÇÃO ====================
local player = game.Players.LocalPlayer
if not player then return end

local char = player.Character or player.CharacterAdded:Wait()
local hrp = char:WaitForChild("HumanoidRootPart")
local hum = char:WaitForChild("Humanoid")
local cam = workspace.CurrentCamera
local mouse = player:GetMouse()

-- ==================== CONFIGURAÇÕES ====================
local settings = {
    chamsActive = true,
    aimbotActive = false,
    walkSpeed = 16,
    fovSize = 150
}

-- ==================== FUNÇÃO DETECTAR ROLE ====================
local function getRole(plr)
    if plr:FindFirstChild("PlayerRole") then
        return plr.PlayerRole.Value
    end
    
    if plr.Character then
        if plr.Character:FindFirstChild("PlayerRole") then
            return plr.Character.PlayerRole.Value
        end
        
        for _, tool in pairs(plr.Character:GetChildren()) do
            if tool:IsA("Tool") then
                if tool.Name:lower():find("knife") or tool.Name:lower():find("faca") then
                    return "Murderer"
                elseif tool.Name:lower():find("gun") or tool.Name:lower():find("revolver") or tool.Name:lower():find("pistol") then
                    return "Sheriff"
                end
            end
        end
    end
    
    local backpack = plr:FindFirstChild("Backpack")
    if backpack then
        for _, tool in pairs(backpack:GetChildren()) do
            if tool:IsA("Tool") then
                if tool.Name:lower():find("knife") or tool.Name:lower():find("faca") then
                    return "Murderer"
                elseif tool.Name:lower():find("gun") or tool.Name:lower():find("revolver") or tool.Name:lower():find("pistol") then
                    return "Sheriff"
                end
            end
        end
    end
    
    return "Innocent"
end

-- ==================== CHAMS ====================
local function clearChams()
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr.Character then
            for _, part in pairs(plr.Character:GetDescendants()) do
                if part:IsA("Highlight") and part.Name == "LucasChams" then
                    part:Destroy()
                end
            end
        end
    end
end

local function createChams()
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local role = getRole(plr)
            local color = Color3.new(0, 1, 0)
            
            if role == "Murderer" then
                color = Color3.new(1, 0, 0)
            elseif role == "Sheriff" then
                color = Color3.new(0, 0.5, 1)
            end
            
            for _, part in pairs(plr.Character:GetChildren()) do
                if part:IsA("BasePart") then
                    local oldChams = part:FindFirstChild("LucasChams")
                    if oldChams then oldChams:Destroy() end
                    
                    local highlight = Instance.new("Highlight")
                    highlight.Name = "LucasChams"
                    highlight.FillColor = color
                    highlight.FillTransparency = 0.35
                    highlight.OutlineColor = color
                    highlight.OutlineTransparency = 0.4
                    highlight.DepthMode = Enum.HighlightDepthMode.AlwaysOnTop
                    highlight.Adornee = part
                    highlight.Parent = part
                end
            end
        end
    end
end

spawn(function()
    while wait(1) do
        if settings.chamsActive then
            pcall(createChams)
        end
    end
end)

-- ==================== VISIBILIDADE ====================
local function isVisible(part)
    if not part or not part.Parent then return false end
    
    local raycastParams = RaycastParams.new()
    raycastParams.FilterType = Enum.RaycastFilterType.Blacklist
    raycastParams.FilterDescendantsInstances = {char}
    raycastParams.IgnoreWater = true
    
    local origin = cam.CFrame.Position
    local direction = (part.Position - origin).Unit * 500
    local result = workspace:Raycast(origin, direction, raycastParams)
    
    if result then
        local hit = result.Instance
        return hit:IsDescendantOf(part.Parent) or result.Distance >= (part.Position - origin).Magnitude - 2
    end
    return true
end

-- ==================== AIMBOT ====================
local function getClosestPlayer()
    local closest = nil
    local minDist = math.huge
    local center = Vector2.new(cam.ViewportSize.X/2, cam.ViewportSize.Y/2)
    
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character and plr.Character:FindFirstChild("HumanoidRootPart") then
            local torso = plr.Character.HumanoidRootPart
            if plr.Character:FindFirstChild("Humanoid") and plr.Character.Humanoid.Health > 0 then
                local screenPos, onScreen = cam:WorldToViewportPoint(torso.Position)
                
                if onScreen and screenPos.Z > 0 then
                    local dist = (Vector2.new(screenPos.X, screenPos.Y) - center).Magnitude
                    local worldDist = (hrp.Position - torso.Position).Magnitude
                    
                    if dist < settings.fovSize and worldDist < 300 and isVisible(torso) then
                        if dist < minDist then
                            minDist = dist
                            closest = plr
                        end
                    end
                end
            end
        end
    end
    return closest
end

spawn(function()
    while wait(0.05) do
        if settings.aimbotActive then
            local target = getClosestPlayer()
            if target and target.Character and target.Character:FindFirstChild("HumanoidRootPart") then
                local torso = target.Character.HumanoidRootPart
                if isVisible(torso) then
                    cam.CFrame = CFrame.new(cam.CFrame.Position, torso.Position)
                end
            end
        end
    end
end)

-- ==================== FOV CIRCLE ====================
local fovCircle = Instance.new("Frame")
fovCircle.Parent = player.PlayerGui
fovCircle.BackgroundTransparency = 1
fovCircle.Size = UDim2.new(0, settings.fovSize * 2, 0, settings.fovSize * 2)
fovCircle.Position = UDim2.new(0.5, -settings.fovSize, 0.5, -settings.fovSize)
fovCircle.ZIndex = 999
fovCircle.Visible = false

local circle = Instance.new("ImageLabel")
circle.Parent = fovCircle
circle.BackgroundTransparency = 1
circle.Size = UDim2.new(1, 0, 1, 0)
circle.Image = "rbxassetid://3926305904"
circle.ImageColor3 = Color3.fromRGB(255, 0, 0)
circle.ImageTransparency = 0.6

spawn(function()
    while wait(0.1) do
        if settings.aimbotActive then
            fovCircle.Visible = true
            fovCircle.Size = UDim2.new(0, settings.fovSize * 2, 0, settings.fovSize * 2)
            fovCircle.Position = UDim2.new(0.5, -settings.fovSize, 0.5, -settings.fovSize)
        else
            fovCircle.Visible = false
        end
    end
end)

-- ==================== TELEPORTAR PRO VOID ====================
local function teleportAllToVoid()
    local count = 0
    for _, plr in pairs(game.Players:GetPlayers()) do
        if plr ~= player and plr.Character then
            local targetHrp = plr.Character:FindFirstChild("HumanoidRootPart")
            local targetHum = plr.Character:FindFirstChild("Humanoid")
            
            if targetHrp then
                pcall(function()
                    targetHrp.CFrame = CFrame.new(math.random(-100, 100), -500, math.random(-100, 100))
                    if targetHum then targetHum.Health = 0 end
                    count = count + 1
                end)
            end
        end
    end
    return count
end

-- ==================== LOOP VELOCIDADE ====================
spawn(function()
    while wait(0.1) do
        if hum then
            hum.WalkSpeed = settings.walkSpeed
        end
    end
end)

-- ==================== PAINEL ====================
local screenGui = Instance.new("ScreenGui")
screenGui.Parent = player.PlayerGui
screenGui.Name = "LucasLuaGUI"
screenGui.ResetOnSpawn = false

local mainFrame = Instance.new("Frame")
mainFrame.Parent = screenGui
mainFrame.BackgroundColor3 = Color3.fromRGB(15, 15, 25)
mainFrame.BackgroundTransparency = 0.05
mainFrame.BorderSizePixel = 0
mainFrame.Size = UDim2.new(0, 300, 0, 420)
mainFrame.Position = UDim2.new(0, 20, 0.5, -210)
mainFrame.Active = true
mainFrame.Draggable = true
mainFrame.ClipsDescendants = true

local border = Instance.new("UIStroke")
border.Parent = mainFrame
border.Color = Color3.fromRGB(100, 50, 200)
border.Thickness = 2

local titleBar = Instance.new("Frame")
titleBar.Parent = mainFrame
titleBar.BackgroundColor3 = Color3.fromRGB(25, 20, 45)
titleBar.BorderSizePixel = 0
titleBar.Size = UDim2.new(1, 0, 0, 35)

local title = Instance.new("TextLabel")
title.Parent = titleBar
title.BackgroundTransparency = 1
title.Size = UDim2.new(0.7, 0, 1, 0)
title.Position = UDim2.new(0, 10, 0, 0)
title.Text = "🔪 Lucas Lua"
title.TextColor3 = Color3.fromRGB(255, 255, 255)
title.TextSize = 15
title.TextXAlignment = Enum.TextXAlignment.Left
title.Font = Enum.Font.GothamBold

local minBtn = Instance.new("TextButton")
minBtn.Parent = titleBar
minBtn.BackgroundTransparency = 1
minBtn.Size = UDim2.new(0, 30, 1, 0)
minBtn.Position = UDim2.new(0.75, 0, 0, 0)
minBtn.Text = "−"
minBtn.TextColor3 = Color3.fromRGB(255, 255, 255)
minBtn.TextSize = 20
minBtn.Font = Enum.Font.GothamBold

local closeBtn = Instance.new("TextButton")
closeBtn.Parent = titleBar
closeBtn.BackgroundTransparency = 1
closeBtn.Size = UDim2.new(0, 30, 1, 0)
closeBtn.Position = UDim2.new(0.88, 0, 0, 0)
closeBtn.Text = "✕"
closeBtn.TextColor3 = Color3.fromRGB(255, 50, 50)
closeBtn.TextSize = 16
closeBtn.Font = Enum.Font.GothamBold

closeBtn.MouseButton1Click:Connect(function()
    screenGui:Destroy()
end)

local scroll = Instance.new("ScrollingFrame")
scroll.Parent = mainFrame
scroll.BackgroundTransparency = 1
scroll.Size = UDim2.new(1, 0, 1, -35)
scroll.Position = UDim2.new(0, 0, 0, 35)
scroll.CanvasSize = UDim2.new(0, 0, 0, 550)
scroll.ScrollBarThickness = 4
scroll.ScrollBarImageColor3 = Color3.fromRGB(100, 50, 200)

local isMinimized = false
minBtn.MouseButton1Click:Connect(function()
    isMinimized = not isMinimized
    mainFrame.Size = isMinimized and UDim2.new(0, 300, 0, 35) or UDim2.new(0, 300, 0, 420)
    scroll.Visible = not isMinimized
    statusBar.Visible = not isMinimized
end)

function createToggle(text, yPos, default, callback)
    local container = Instance.new("Frame")
    container.Parent = scroll
    container.BackgroundTransparency = 1
    container.Size = UDim2.new(1, -16, 0, 35)
    container.Position = UDim2.new(0, 8, 0, yPos)
    
    local label = Instance.new("TextLabel")
    label.Parent = container
    label.BackgroundTransparency = 1
    label.Size = UDim2.new(0.68, 0, 1, 0)
    label.Text = text
    label.TextColor3 = Color3.fromRGB(220, 220, 240)
    label.TextSize = 12
    label.TextXAlignment = Enum.TextXAlignment.Left
    label.Font = Enum.Font.Gotham
    
    local toggle = Instance.new("TextButton")
    toggle.Parent = container
    toggle.BackgroundColor3 = default and Color3.fromRGB(0, 180, 80) or Color3.fromRGB(50, 50, 70)
    toggle.BorderSizePixel = 0
    toggle.Size = UDim2.new(0, 55, 0, 24)
    toggle.Position = UDim2.new(0.75, 0, 0.5, -12)
    toggle.Text = default and "ON" or "OFF"
    toggle.TextColor3 = Color3.fromRGB(255, 255, 255)
    toggle.TextSize = 11
    toggle.Font = Enum.Font.GothamBold
    
    local state = default
    toggle.MouseButton1Click:Connect(function()
        state = not state
        toggle.BackgroundColor3 = state and Color3.fromRGB(0, 180, 80) or Color3.fromRGB(50, 50, 70)
        toggle.Text = state and "ON" or "OFF"
        pcall(function() callback(state) end)
    end)
end

function createButton(text, yPos, color, callback)
    local btn = Instance.new("TextButton")
    btn.Parent = scroll
    btn.BackgroundColor3 = color
    btn.BorderSizePixel = 0
    btn.Size = UDim2.new(1, -16, 0, 32)
    btn.Position = UDim2.new(0, 8, 0, yPos)
    btn.Text = text
    btn.TextColor3 = Color3.fromRGB(255, 255, 255)
    btn.TextSize = 12
    btn.Font = Enum.Font.GothamBold
    
    btn.MouseButton1Click:Connect(function()
        pcall(callback)
    end)
end

local y = 8

createToggle("🎨 CHAMS", y, true, function(s)
    settings.chamsActive = s
    if not s then clearChams() else createChams() end
end)
y = y + 40

createButton("🔄 ATUALIZAR CHAMS", y, Color3.fromRGB(100, 50, 200), function()
    clearChams()
    wait(0.1)
    createChams()
    print("✅ Chams atualizadas!")
end)
y = y + 40

createToggle("🎯 AIMBOT (Todos)", y, false, function(s)
    settings.aimbotActive = s
end)
y = y + 40

createButton("💀 TELEPORTAR PRO VOID", y, Color3.fromRGB(180, 30, 30), function()
    local count = teleportAllToVoid()
    print("💀 " .. count .. " jogadores teleportados!")
end)
y = y + 50

local speedFrame = Instance.new("Frame")
speedFrame.Parent = scroll
speedFrame.BackgroundTransparency = 1
speedFrame.Size = UDim2.new(1, -16, 0, 55)
speedFrame.Position = UDim2.new(0, 8, 0, y)

local speedLabel = Instance.new("TextLabel")
speedLabel.Parent = speedFrame
speedLabel.BackgroundTransparency = 1
speedLabel.Size = UDim2.new(1, 0, 0, 20)
speedLabel.Text = "⚡ VELOCIDADE: " .. settings.walkSpeed
speedLabel.TextColor3 = Color3.fromRGB(220, 220, 240)
speedLabel.TextSize = 12
speedLabel.TextXAlignment = Enum.TextXAlignment.Left
speedLabel.Font = Enum.Font.Gotham

local speedInput = Instance.new("TextBox")
speedInput.Parent = speedFrame
speedInput.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
speedInput.BorderSizePixel = 0
speedInput.Size = UDim2.new(1, 0, 0, 26)
speedInput.Position = UDim2.new(0, 0, 0, 25)
speedInput.Text = tostring(settings.walkSpeed)
speedInput.TextColor3 = Color3.fromRGB(255, 255, 255)
speedInput.TextSize = 12
speedInput.Font = Enum.Font.Gotham

speedInput.FocusLost:Connect(function()
    local num = tonumber(speedInput.Text)
    if num and num >= 0 and num <= 500 then
        settings.walkSpeed = num
        speedLabel.Text = "⚡ VELOCIDADE: " .. num
    else
        speedInput.Text = tostring(settings.walkSpeed)
    end
end)
y = y + 60

local fovFrame = Instance.new("Frame")
fovFrame.Parent = scroll
fovFrame.BackgroundTransparency = 1
fovFrame.Size = UDim2.new(1, -16, 0, 55)
fovFrame.Position = UDim2.new(0, 8, 0, y)

local fovLabel = Instance.new("TextLabel")
fovLabel.Parent = fovFrame
fovLabel.BackgroundTransparency = 1
fovLabel.Size = UDim2.new(1, 0, 0, 20)
fovLabel.Text = "👁️ FOV: " .. settings.fovSize
fovLabel.TextColor3 = Color3.fromRGB(220, 220, 240)
fovLabel.TextSize = 12
fovLabel.TextXAlignment = Enum.TextXAlignment.Left
fovLabel.Font = Enum.Font.Gotham

local fovInput = Instance.new("TextBox")
fovInput.Parent = fovFrame
fovInput.BackgroundColor3 = Color3.fromRGB(40, 40, 60)
fovInput.BorderSizePixel = 0
fovInput.Size = UDim2.new(1, 0, 0, 26)
fovInput.Position = UDim2.new(0, 0, 0, 25)
fovInput.Text = tostring(settings.fovSize)
fovInput.TextColor3 = Color3.fromRGB(255, 255, 255)
fovInput.TextSize = 12
fovInput.Font = Enum.Font.Gotham

fovInput.FocusLost:Connect(function()
    local num = tonumber(fovInput.Text)
    if num and num >= 50 and num <= 400 then
        settings.fovSize = num
        fovLabel.Text = "👁️ FOV: " .. num
    else
        fovInput.Text = tostring(settings.fovSize)
    end
end)

local statusBar = Instance.new("Frame")
statusBar.Parent = mainFrame
statusBar.BackgroundColor3 = Color3.fromRGB(25, 20, 45)
statusBar.BorderSizePixel = 0
statusBar.Size = UDim2.new(1, 0, 0, 22)
statusBar.Position = UDim2.new(0, 0, 1, -22)

local statusText = Instance.new("TextLabel")
statusText.Parent = statusBar
statusText.BackgroundTransparency = 1
statusText.Size = UDim2.new(1, 0, 1, 0)
statusText.Text = "✅ Lucas Lua | Carregado"
statusText.TextColor3 = Color3.fromRGB(100, 255, 100)
statusText.TextSize = 11
statusText.Font = Enum.Font.Gotham

print("✅ SCRIPT LUCAS LUA CARREGADO!")
print("🔴 Assassino = Vermelho | 🔵 Xerife = Azul | 🟢 Inocente = Verde")

game.StarterGui:SetCore("SendNotification", {
    Title = "Lucas Lua",
    Text = "Script carregado com sucesso!",
    Duration = 4
})
