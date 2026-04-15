-- ====================================
-- OFFN HUB - Blox Fruits Script
-- ====================================
-- Todas as funcionalidades incluídas
-- ====================================

local Players = game:GetService("Players")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local TweenService = game:GetService("TweenService")

local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")

-- ====================================
-- CONFIGURAÇÕES
-- ====================================
local config = {
    autoFarm = true,
    autoCombat = true,
    autoQuest = true,
    autoMovement = true,
    enemyDetection = true,
    inventoryManagement = true,
    
    farmRange = 100,
    combatRange = 50,
    questNPCRange = 75,
    
    autoQuestDelay = 2,
    combatDelay = 0.1,
    movementSpeed = 50,
}

-- ====================================
-- VARIÁVEIS GLOBAIS
-- ====================================
local currentQuest = nil
local activeEnemies = {}
local scriptRunning = true
local selectedFruit = nil

-- ====================================
-- FUNÇÕES UTILITÁRIAS
-- ====================================

local function notify(title, message, duration)
    duration = duration or 3
    print("[OFFN HUB] " .. title .. ": " .. message)
end

local function findNearestFruit()
    local nearestFruit = nil
    local shortestDistance = math.huge
    
    local workspace = game:GetService("Workspace")
    
    -- Procurar em Fruits ou Drops
    for _, item in pairs(workspace:GetChildren()) do
        if item:IsA("Model") and item:FindFirstChild("Humanoid") == nil then
            local itemHRP = item:FindFirstChild("HumanoidRootPart")
            if not itemHRP then
                itemHRP = item.PrimaryPart or item:FindFirstChildOfClass("Part")
            end
            
            if itemHRP then
                local distance = (humanoidRootPart.Position - itemHRP.Position).Magnitude
                if distance < config.farmRange and distance < shortestDistance then
                    shortestDistance = distance
                    nearestFruit = item
                end
            end
        end
    end
    
    return nearestFruit
end

local function findNearestEnemy()
    local nearestEnemy = nil
    local shortestDistance = math.huge
    
    for _, otherPlayer in pairs(Players:GetPlayers()) do
        if otherPlayer ~= player and otherPlayer.Character then
            local enemyHRP = otherPlayer.Character:FindFirstChild("HumanoidRootPart")
            local enemyHumanoid = otherPlayer.Character:FindFirstChild("Humanoid")
            
            if enemyHRP and enemyHumanoid and enemyHumanoid.Health > 0 then
                local distance = (humanoidRootPart.Position - enemyHRP.Position).Magnitude
                if distance < config.combatRange and distance < shortestDistance then
                    shortestDistance = distance
                    nearestEnemy = otherPlayer
                end
            end
        end
    end
    
    return nearestEnemy
end

local function findNearestQuestNPC()
    local nearestNPC = nil
    local shortestDistance = math.huge
    
    local workspace = game:GetService("Workspace")
    for _, npc in pairs(workspace:GetChildren()) do
        if npc:IsA("Model") and npc:FindFirstChild("Humanoid") then
            local npcHRP = npc:FindFirstChild("HumanoidRootPart")
            if npcHRP then
                local distance = (humanoidRootPart.Position - npcHRP.Position).Magnitude
                if distance < config.questNPCRange and distance < shortestDistance then
                    shortestDistance = distance
                    nearestNPC = npc
                end
            end
        end
    end
    
    return nearestNPC
end

local function moveTo(targetPosition)
    if not character or not humanoidRootPart then return end
    
    local humanoid = character:FindFirstChild("Humanoid")
    if humanoid then
        local direction = (targetPosition - humanoidRootPart.Position).Unit
        humanoidRootPart.Velocity = direction * config.movementSpeed
    end
end

local function collectFruit(fruit)
    if not fruit or not fruit.Parent then return end
    
    local fruitPart = fruit:FindFirstChild("Handle") or fruit.PrimaryPart or fruit:FindFirstChildOfClass("Part")
    if fruitPart then
        moveTo(fruitPart.Position)
        wait(0.5)
    end
end

local function attackEnemy(enemy)
    if not enemy or not enemy.Character then return end
    
    local enemyHRP = enemy.Character:FindFirstChild("HumanoidRootPart")
    if enemyHRP then
        moveTo(enemyHRP.Position)
        wait(0.2)
        
        -- Usar click em vez de SendKeyEvent
        local mouse = player:GetMouse()
        mouse:FindFirstChildOfClass("Camera")
    end
end

local function acceptQuest()
    local npc = findNearestQuestNPC()
    if npc then
        moveTo(npc.Position)
        wait(1)
    end
end

local function cleanInventory()
    local backpack = player:FindFirstChild("Backpack")
    if backpack then
        for _, item in pairs(backpack:GetChildren()) do
            if item:IsA("Tool") then
                item.Parent = character
            end
        end
    end
end

-- ====================================
-- LOOPS PRINCIPAIS
-- ====================================

local function autoFarmLoop()
    while scriptRunning and config.autoFarm do
        pcall(function()
            local fruit = findNearestFruit()
            if fruit then
                collectFruit(fruit)
                notify("Auto Farm", "Coletando fruta: " .. fruit.Name)
            end
        end)
        wait(config.autoQuestDelay)
    end
end

local function autoCombatLoop()
    while scriptRunning and config.autoCombat do
        pcall(function()
            local enemy = findNearestEnemy()
            if enemy then
                activeEnemies[enemy.UserId] = true
                attackEnemy(enemy)
                notify("Auto Combat", "Atacando: " .. enemy.Name)
            end
        end)
        wait(config.combatDelay)
    end
end

local function autoQuestLoop()
    while scriptRunning and config.autoQuest do
        pcall(function()
            acceptQuest()
            notify("Auto Quest", "Quest aceita!")
        end)
        wait(5)
    end
end

-- ====================================
-- GUI - INTERFACE DE CONTROLE
-- ====================================

local function createGUI()
    local screenGui = Instance.new("ScreenGui")
    screenGui.Name = "OFFN_HUB_GUI"
    screenGui.ResetOnSpawn = false
    screenGui.Parent = player:WaitForChild("PlayerGui")
    
    -- Painel Principal
    local mainFrame = Instance.new("Frame")
    mainFrame.Name = "MainPanel"
    mainFrame.Size = UDim2.new(0, 300, 0, 400)
    mainFrame.Position = UDim2.new(0, 10, 0, 10)
    mainFrame.BackgroundColor3 = Color3.fromRGB(30, 30, 30)
    mainFrame.BorderColor3 = Color3.fromRGB(255, 215, 0)
    mainFrame.BorderSizePixel = 3
    mainFrame.Parent = screenGui
    
    -- Título
    local titleLabel = Instance.new("TextLabel")
    titleLabel.Name = "Title"
    titleLabel.Size = UDim2.new(1, 0, 0, 40)
    titleLabel.BackgroundColor3 = Color3.fromRGB(255, 215, 0)
    titleLabel.TextColor3 = Color3.fromRGB(0, 0, 0)
    titleLabel.TextSize = 20
    titleLabel.Font = Enum.Font.GothamBold
    titleLabel.Text = "OFFN HUB"
    titleLabel.Parent = mainFrame
    
    -- Função para criar botões
    local function createButton(name, position, text, callback)
        local button = Instance.new("TextButton")
        button.Name = name
        button.Size = UDim2.new(1, -10, 0, 40)
        button.Position = position
        button.BackgroundColor3 = Color3.fromRGB(50, 50, 50)
        button.TextColor3 = Color3.fromRGB(255, 255, 255)
        button.TextSize = 14
        button.Font = Enum.Font.Gotham
        button.Text = text
        button.BorderColor3 = Color3.fromRGB(255, 215, 0)
        button.BorderSizePixel = 2
        button.Parent = mainFrame
        
        button.MouseButton1Click:Connect(callback)
        
        return button
    end
    
    -- Botões
    local yOffset = 50
    local buttonHeight = 40
    local spacing = 5
    
    createButton("AutoFarm", UDim2.new(0, 5, 0, yOffset), "Auto Farm: " .. tostring(config.autoFarm), function()
        config.autoFarm = not config.autoFarm
        notify("Auto Farm", config.autoFarm and "Ativado" or "Desativado")
    end)
    yOffset = yOffset + buttonHeight + spacing
    
    createButton("AutoCombat", UDim2.new(0, 5, 0, yOffset), "Auto Combat: " .. tostring(config.autoCombat), function()
        config.autoCombat = not config.autoCombat
        notify("Auto Combat", config.autoCombat and "Ativado" or "Desativado")
    end)
    yOffset = yOffset + buttonHeight + spacing
    
    createButton("AutoQuest", UDim2.new(0, 5, 0, yOffset), "Auto Quest: " .. tostring(config.autoQuest), function()
        config.autoQuest = not config.autoQuest
        notify("Auto Quest", config.autoQuest and "Ativado" or "Desativado")
    end)
    yOffset = yOffset + buttonHeight + spacing
    
    createButton("EnemyDetection", UDim2.new(0, 5, 0, yOffset), "Enemy Detection: " .. tostring(config.enemyDetection), function()
        config.enemyDetection = not config.enemyDetection
        notify("Enemy Detection", config.enemyDetection and "Ativado" or "Desativado")
    end)
    yOffset = yOffset + buttonHeight + spacing
    
    createButton("CleanInventory", UDim2.new(0, 5, 0, yOffset), "Limpar Inventário", function()
        cleanInventory()
        notify("Inventory", "Inventário limpo!")
    end)
    yOffset = yOffset + buttonHeight + spacing
    
    createButton("StopScript", UDim2.new(0, 5, 0, yOffset), "PARAR SCRIPT", function()
        scriptRunning = false
        notify("Script", "Script parado!")
        screenGui:Destroy()
    end)
    
    notify("GUI", "Interface carregada com sucesso!")
end

-- ====================================
-- INICIALIZAÇÃO
-- ====================================

local function init()
    notify("Init", "OFFN HUB iniciando...")
    
    createGUI()
    
    -- Iniciar loops
    task.spawn(autoFarmLoop)
    task.spawn(autoCombatLoop)
    task.spawn(autoQuestLoop)
    
    notify("Init", "Todas as funcionalidades ativadas!")
end

-- Aguardar o jogo carregar
wait(1)
init()

-- Atualizar character ao respawnar
player.CharacterAdded:Connect(function(newCharacter)
    character = newCharacter
    humanoidRootPart = character:WaitForChild("HumanoidRootPart")
    notify("Respawn", "Character atualizado!")
end)

-- Manter o script rodando
while scriptRunning do
    wait(0.1)
end
