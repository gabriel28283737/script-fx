local DrRayLibrary = loadstring(game:HttpGet("https://raw.githubusercontent.com/AZYsGithub/DrRay-UI-Library/main/DrRay.lua"))()
local window = DrRayLibrary:Load("General Xiter", "Default")

-- Criando abas
local aimbotTab = window.newTab("Aimbot", "ImageIdHere")
local espTab = window.newTab("ESP", "ImageIdHere")
local creditsTab = window.newTab("Créditos", "ImageIdHere")
local miscTab = window.newTab("Misc", "ImageIdHere")
local weaponsTab = window.newTab("Armas", "ImageIdHere") -- Nova aba para armas

-- Adicionar botão de créditos
creditsTab.newButton("Feito por General", "Sem funcionalidade", function() end)

-- Variáveis de controle
local aimbotEnabled = false
local teamCheck = false
local wallCheck = false
local espEnabled = {Red = false, Blue = false, Black = false}
local hitboxEnabled = false

-- Função para verificar times
local function isTeammate(player)
    return player.Team == game.Players.LocalPlayer.Team
end

-- Função de Aimbot
local function aimbot()
    local playerService = game:GetService("Players")
    local localPlayer = playerService.LocalPlayer
    local camera = workspace.CurrentCamera

    game:GetService("RunService").RenderStepped:Connect(function()
        if not aimbotEnabled then return end

        local closestPlayer = nil
        local shortestDistance = math.huge

        for _, player in pairs(playerService:GetPlayers()) do
            if player ~= localPlayer and player.Character and player.Character:FindFirstChild("Humanoid") and player.Character:FindFirstChild("HumanoidRootPart") then
                local rootPart = player.Character.HumanoidRootPart
                local screenPosition, onScreen = camera:WorldToScreenPoint(rootPart.Position)
                
                if onScreen then
                    local distance = (Vector2.new(screenPosition.X, screenPosition.Y) - Vector2.new(camera.ViewportSize.X / 2, camera.ViewportSize.Y / 2)).Magnitude
                    if distance < shortestDistance then
                        closestPlayer = player
                        shortestDistance = distance
                    end
                end
            end
        end

        if closestPlayer and closestPlayer.Character and closestPlayer.Character:FindFirstChild("HumanoidRootPart") then
            camera.CFrame = CFrame.new(camera.CFrame.Position, closestPlayer.Character.HumanoidRootPart.Position)
        end
    end)
end

-- Função para criar ESP usando Highlight
local function createESP(color)
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character then
            if not player.Character:FindFirstChild("ESP") then
                local highlight = Instance.new("Highlight", player.Character)
                highlight.FillColor = color
                highlight.OutlineTransparency = 1
                highlight.Name = "ESP"
            end
        end
    end
end

-- Função para remover ESP
local function removeESP()
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player.Character then
            local esp = player.Character:FindFirstChild("ESP")
            if esp then
                esp:Destroy()
            end
        end
    end
end

-- Atualizar ESP de acordo com a cor ativada
local function updateESP()
    removeESP()
    if espEnabled.Red then createESP(Color3.fromRGB(255, 0, 0)) end
    if espEnabled.Blue then createESP(Color3.fromRGB(0, 0, 255)) end
    if espEnabled.Black then createESP(Color3.fromRGB(0, 0, 0)) end
end

game:GetService("RunService").RenderStepped:Connect(updateESP)

-- Função para ajustar hitboxes automaticamente
local function adjustHitboxes()
    for _, player in ipairs(game:GetService("Players"):GetPlayers()) do
        if player ~= game.Players.LocalPlayer and player.Character and player.Character:FindFirstChild("Head") then
            local head = player.Character.Head
            if not isTeammate(player) then
                head.Size = hitboxEnabled and Vector3.new(5, 5, 5) or Vector3.new(2, 1, 1)
                head.CanCollide = not hitboxEnabled
            end
        end
    end
end

game:GetService("RunService").RenderStepped:Connect(adjustHitboxes)

-- Configurações do Aimbot
aimbotTab.newToggle("Aimbot", "Ativa o Aimbot", false, function(state)
    aimbotEnabled = state
end)

aimbotTab.newToggle("Team Check", "Não mira no seu time", false, function(state)
    teamCheck = state
end)

aimbotTab.newToggle("Wall Check", "Evita mirar em jogadores atrás de paredes", false, function(state)
    wallCheck = state
end)

-- Configurações de ESP
espTab.newToggle("ESP Vermelho", "Ativa ESP Vermelho", false, function(state)
    espEnabled.Red = state
end)

espTab.newToggle("ESP Azul", "Ativa ESP Azul", false, function(state)
    espEnabled.Blue = state
end)

espTab.newToggle("ESP Preto", "Ativa ESP Preto", false, function(state)
    espEnabled.Black = state
end)

-- Configuração de Hitbox
miscTab.newToggle("Hitbox Cabeça", "Aumenta a hitbox da cabeça", false, function(state)
    hitboxEnabled = state
    adjustHitboxes()
end)

-- =======================
--        NOVA ABA ARMAS
-- =======================

local selectedWeapon = nil

-- Função para buscar armas no jogo
local function getWeapons()
    local weapons = {}
    
    for _, item in ipairs(workspace:GetChildren()) do
        if item:IsA("Tool") or item:IsA("Model") then
            table.insert(weapons, item.Name)
        end
    end
    
    return weapons
end

-- Criar a lista de armas na aba
local function updateWeaponList()
    weaponsTab.clear() -- Limpa a aba antes de recriar a lista
    
    local weapons = getWeapons()
    
    for _, weaponName in ipairs(weapons) do
        weaponsTab.newButton(weaponName, "Selecionar arma", function()
            selectedWeapon = weaponName
        end)
    end
    
    -- Botão de puxar arma selecionada
    weaponsTab.newButton("Puxar Arma", "Equipe a arma selecionada", function()
        if selectedWeapon then
            local player = game.Players.LocalPlayer
            local backpack = player:FindFirstChild("Backpack")

            if backpack then
                for _, item in ipairs(workspace:GetChildren()) do
                    if item:IsA("Tool") and item.Name == selectedWeapon then
                        item.Parent = backpack
                        player.Character.Humanoid:EquipTool(item)
                        break
                    end
                end
            end
        end
    end)
end

workspace.ChildAdded:Connect(function()
    wait(0.5)
    updateWeaponList()
end)

updateWeaponList()

-- Inicializar Aimbot
aimbot()
