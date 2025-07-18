local Players = game:GetService("Players")
local LocalPlayer = Players.LocalPlayer
local Camera = workspace.CurrentCamera
local RunService = game:GetService("RunService")
local UIS = game:GetService("UserInputService")

-- Configuração da interface
local ScreenGui = Instance.new("ScreenGui", LocalPlayer.PlayerGui)
ScreenGui.Name = "AimbotESPGui"

local MainFrame = Instance.new("Frame", ScreenGui)
MainFrame.Size = UDim2.new(0, 120, 0, 80)
MainFrame.Position = UDim2.new(0, 10, 0, 10)
MainFrame.BackgroundTransparency = 0.6
MainFrame.BackgroundColor3 = Color3.fromRGB(30,30,30)
MainFrame.Active = true
MainFrame.Draggable = true

local AimbotToggle = Instance.new("TextButton", MainFrame)
AimbotToggle.Size = UDim2.new(0, 100, 0, 25)
AimbotToggle.Position = UDim2.new(0, 10, 0, 10)
AimbotToggle.Text = "Aimbot: OFF"
AimbotToggle.BackgroundColor3 = Color3.fromRGB(80,80,80)

local ESPToggle = Instance.new("TextButton", MainFrame)
ESPToggle.Size = UDim2.new(0, 100, 0, 25)
ESPToggle.Position = UDim2.new(0, 10, 0, 45)
ESPToggle.Text = "ESP: OFF"
ESPToggle.BackgroundColor3 = Color3.fromRGB(80,80,80)

local aimbotEnabled = false
local espEnabled = false

AimbotToggle.MouseButton1Click:Connect(function()
    aimbotEnabled = not aimbotEnabled
    AimbotToggle.Text = aimbotEnabled and "Aimbot: ON" or "Aimbot: OFF"
end)
ESPToggle.MouseButton1Click:Connect(function()
    espEnabled = not espEnabled
    ESPToggle.Text = espEnabled and "ESP: ON" or "ESP: OFF"
end)

-- Funções auxiliares
local function IsEnemy(player)
    -- Troque "Team" pela propriedade de time usada no seu jogo
    return player.Team ~= LocalPlayer.Team
end

local function GetClosestEnemy()
    local closestEnemy = nil
    local minDist = math.huge
    for _,player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and IsEnemy(player) and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local dist = (Vector2.new(pos.X, pos.Y) - UIS:GetMouseLocation()).magnitude
                -- Área de ataque: dist < 400 pixels do centro da tela
                if dist < 400 and dist < minDist then
                    minDist = dist
                    closestEnemy = player
                end
            end
        end
    end
    return closestEnemy
end

-- ESP
local ESPBoxes = {}

local function UpdateESP()
    for _,v in pairs(ESPBoxes) do v:Destroy() end
    ESPBoxes = {}

    if not espEnabled then return end

    for _,player in ipairs(Players:GetPlayers()) do
        if player ~= LocalPlayer and IsEnemy(player) and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            local pos, onScreen = Camera:WorldToViewportPoint(head.Position)
            if onScreen then
                local box = Instance.new("Frame", ScreenGui)
                box.Size = UDim2.new(0, 40, 0, 40)
                box.Position = UDim2.new(0, pos.X-20, 0, pos.Y-20)
                box.BackgroundTransparency = 0.7
                box.BackgroundColor3 = Color3.fromRGB(255,0,0)
                box.BorderSizePixel = 0
                table.insert(ESPBoxes, box)
            end
        end
    end
end

-- AIMBOT
local lastShot = 0
local shotInterval = 0.3 -- 1 tiro a cada 0.3s (ajuste para evitar anti-cheat)

local function AimAndShoot()
    if not aimbotEnabled then return end
    local enemy = GetClosestEnemy()
    if enemy and enemy.Character and enemy.Character:FindFirstChild("Head") then
        local head = enemy.Character.Head
        Camera.CFrame = CFrame.new(Camera.CFrame.Position, head.Position)
        -- Simula tiro
        if tick() - lastShot > shotInterval then
            lastShot = tick()
            -- Substitua pela função de disparo do seu jogo!
            -- Por exemplo: LocalPlayer.Character:FindFirstChild("Weapon"):Fire()
            -- Simulação de clique do mouse para disparar:
            UIS.InputBegan:Fire({
                UserInputType = Enum.UserInputType.MouseButton1,
                Position = UIS:GetMouseLocation()
            })
        end
    end
end

-- Loop principal
RunService.RenderStepped:Connect(function()
    UpdateESP()
    AimAndShoot()
end)
