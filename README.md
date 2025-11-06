-- ===================================
-- == XIBATAS HUB - BLOX FRUITS ==
-- == CRIADO POR: SLOW.EXE. ==
-- == AVISO: SCIRPT BETA RISCO DE BAN! ==
-- ===================================

-- Serviços do Jogo
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local RunService = game:GetService("RunService")
local UserInputService = game:GetService("UserInputService")
local VirtualUser = game:GetService("VirtualUser")

-- Jogador
local player = Players.LocalPlayer
local character = player.Character or player.CharacterAdded:Wait()
local humanoidRootPart = character:WaitForChild("HumanoidRootPart")
local humanoid = character:WaitForChild("Humanoid")

-- Variáveis de Controle
local isFarming = false
local farmConnection = nil

-- ===================================
-- == CRIAÇÃO DA INTERFACE (GUI) ==
-- ===================================

-- ScreenGui
local screenGui = Instance.new("ScreenGui")
screenGui.Name = "VampireHubGui"
screenGui.Parent = player:WaitForChild("PlayerGui")
screenGui.ResetOnSpawn = false
screenGui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling

-- Frame Principal
local mainFrame = Instance.new("Frame")
mainFrame.Name = "MainFrame"
mainFrame.Parent = screenGui
mainFrame.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
mainFrame.BorderColor3 = Color3.fromRGB(139, 0, 0)
mainFrame.BorderSizePixel = 2
mainFrame.Position = UDim2.new(0.5, -150, 0.5, -100)
mainFrame.Size = UDim2.new(0, 300, 0, 200)
mainFrame.Active = true -- Necessário para arrastar
mainFrame.Draggable = false -- Vamos controlar o arrastar manualmente

-- Barra de Título
local titleBar = Instance.new("Frame")
titleBar.Name = "TitleBar"
titleBar.Parent = mainFrame
titleBar.BackgroundColor3 = Color3.fromRGB(80, 0, 0)
titleBar.BorderSizePixel = 0
titleBar.Size = UDim2.new(1, 0, 0, 30)

local titleLabel = Instance.new("TextLabel")
titleLabel.Name = "TitleLabel"
titleLabel.Parent = titleBar
titleLabel.BackgroundColor3 = Color3.fromRGB(80, 0, 0)
titleLabel.BorderSizePixel = 0
titleLabel.Size = UDim2.new(1, -30, 1, 0)
titleLabel.Position = UDim2.new(0, 5, 0, 0)
titleLabel.Font = Enum.Font.GothamBold
titleLabel.Text = "Vampire Hub"
titleLabel.TextColor3 = Color3.fromRGB(255, 0, 0)
titleLabel.TextScaled = true
titleLabel.TextXAlignment = Enum.TextXAlignment.Left

-- Botão de Fechar
local closeButton = Instance.new("TextButton")
closeButton.Name = "CloseButton"
closeButton.Parent = titleBar
closeButton.BackgroundColor3 = Color3.fromRGB(139, 0, 0)
closeButton.BorderSizePixel = 0
closeButton.Position = UDim2.new(1, -25, 0, 2.5)
closeButton.Size = UDim2.new(0, 22.5, 0, 22.5)
closeButton.Font = Enum.Font.GothamBold
closeButton.Text = "X"
closeButton.TextColor3 = Color3.fromRGB(255, 255, 255)
closeButton.TextScaled = true

-- Botão de Ativar/Desativar
local toggleButton = Instance.new("TextButton")
toggleButton.Name = "ToggleButton"
toggleButton.Parent = mainFrame
toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
toggleButton.BorderColor3 = Color3.fromRGB(139, 0, 0)
toggleButton.BorderSizePixel = 1
toggleButton.Position = UDim2.new(0.5, -75, 0.5, -20)
toggleButton.Size = UDim2.new(0, 150, 0, 40)
toggleButton.Font = Enum.Font.Gotham
toggleButton.Text = "Ativar Farm"
toggleButton.TextColor3 = Color3.fromRGB(255, 0, 0)
toggleButton.TextScaled = true

-- Label de Status
local statusLabel = Instance.new("TextLabel")
statusLabel.Name = "StatusLabel"
statusLabel.Parent = mainFrame
statusLabel.BackgroundColor3 = Color3.fromRGB(10, 10, 10)
statusLabel.BorderSizePixel = 0
statusLabel.Position = UDim2.new(0, 0, 1, -25)
statusLabel.Size = UDim2.new(1, 0, 0, 25)
statusLabel.Font = Enum.Font.Gotham
statusLabel.Text = "Status: Inativo"
statusLabel.TextColor3 = Color3.fromRGB(150, 0, 0)
statusLabel.TextScaled = true

-- ===================================
-- == FUNCIONALIDADE DO PAINEL ==
-- ===================================

-- Função para tornar o Frame arrastável
local dragging = false
local dragStart, startPos

mainFrame.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = true
        dragStart = input.Position
        startPos = mainFrame.Position
    end
end)

UserInputService.InputChanged:Connect(function(input)
    if dragging and input.UserInputType == Enum.UserInputType.MouseMovement then
        local delta = input.Position - dragStart
        mainFrame.Position = UDim2.new(startPos.X.Scale, startPos.X.Offset + delta.X, startPos.Y.Scale, startPos.Y.Offset + delta.Y)
    end
end)

UserInputService.InputEnded:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1 then
        dragging = false
    end
end)

-- Função do botão de fechar
closeButton.MouseButton1Click:Connect(function()
    screenGui:Destroy()
    if farmConnection then
        farmConnection:Disconnect()
        isFarming = false
    end
end)

-- ===================================
-- == LÓGICA DO FARM (BLOX FRUITS) ==
-- ===================================

-- Função para pegar a missão
local function getQuest()
    -- Encontra o NPC que dá missões (geralmente o mais próximo)
    local questGiver
    for _, npc in pairs(workspace.NPCs:GetChildren()) do
        if npc:FindFirstChild("HumanoidRootPart") and (npc.Name:find("Quest") or npc.Name:find("QuestGiver")) then
            if not questGiver or (humanoidRootPart.Position - npc.HumanoidRootPart.Position).Magnitude < (humanoidRootPart.Position - questGiver.HumanoidRootPart.Position).Magnitude then
                questGiver = npc
            end
        end
    end

    if questGiver and questGiver:FindFirstChild("HumanoidRootPart") then
        -- Teleporta o jogador para o NPC para garantir a interação
        local oldCFrame = humanoidRootPart.CFrame
        humanoidRootPart.CFrame = questGiver.HumanoidRootPart.CFrame * CFrame.new(0, 0, 3)
        wait(0.5)
        
        -- Tenta ativar o ProximityPrompt
        local prompt = questGiver:FindFirstChildOfClass("ProximityPrompt")
        if prompt then
            prompt.HoldDuration = 0
            fireproximityprompt(prompt)
        end
        
        wait(0.5)
        humanoidRootPart.CFrame = oldCFrame -- Retorna o jogador
        return true
    end
    return false
end

-- Função para encontrar o alvo mais próximo
local function getTarget()
    local questData = player:FindFirstChild("PlayerStats") and player.PlayerStats:FindFirstChild("CurrentQuest")
    if not questData or not questData.Value then
        return nil
    end
    
    local targetName = questData.Value
    local closestEnemy = nil
    local maxDistance = math.huge

    for _, enemy in pairs(workspace.Enemies:GetChildren()) do
        if enemy.Name == targetName and enemy:FindFirstChild("Humanoid") and enemy.Humanoid.Health > 0 and enemy:FindFirstChild("HumanoidRootPart") then
            local distance = (humanoidRootPart.Position - enemy.HumanoidRootPart.Position).Magnitude
            if distance < maxDistance then
                maxDistance = distance
                closestEnemy = enemy
            end
        end
    end

    return closestEnemy
end

-- Função principal de farm
local function startFarm()
    while isFarming do
        -- Verifica se precisa de uma nova missão
        local hasQuest = player:FindFirstChild("PlayerStats") and player.PlayerStats:FindFirstChild("CurrentQuest") and player.PlayerStats.CurrentQuest.Value ~= nil
        local questComplete = player:FindFirstChild("PlayerStats") and player.PlayerStats:FindFirstChild("QuestProgress") and player.PlayerStats.QuestProgress.Value >= 10

        if not hasQuest or questComplete then
            statusLabel.Text = "Status: Pegando Missão..."
            getQuest()
            wait(1)
        else
            local target = getTarget()
            if target and target.Humanoid.Health > 0 then
                statusLabel.Text = "Status: Farmando " .. target.Name
                
                -- Equipa a melhor arma (ex: Espada)
                for _, tool in pairs(player.Backpack:GetChildren()) do
                    if tool:IsA("Tool") and (tool.ToolTip:find("Sword") or tool.ToolTip:find("Espada")) then
                        humanoid:EquipTool(tool)
                        break
                    end
                end

                -- Lógica de combate: Voar, Puxar e Atacar
                if target:FindFirstChild("HumanoidRootPart") then
                    -- 1. Voar para cima do alvo
                    local targetPos = target.HumanoidRootPart.Position
                    humanoidRootPart.CFrame = CFrame.new(targetPos.X, targetPos.Y + 25, targetPos.Z)
                    
                    -- 2. Puxar o alvo para baixo do jogador
                    target.HumanoidRootPart.CFrame = CFrame.new(humanoidRootPart.Position.X, humanoidRootPart.Position.Y - 5, humanoidRootPart.Position.Z)
                    
                    -- 3. Atacar
                    VirtualUser:ClickButton1(Vector2.new(0,0))
                    VirtualUser:CaptureController()
                end
            else
                statusLabel.Text = "Status: Procurando Inimigos..."
                wait(1) -- Espera os inimigos respawarem
            end
        end
        wait() -- Pequena pausa para não travar
    end
end

-- ===================================
-- == CONEXÃO DOS BOTÕES ==
-- ===================================

toggleButton.MouseButton1Click:Connect(function()
    isFarming = not isFarming
    
    if isFarming then
        statusLabel.Text = "Status: Iniciando..."
        toggleButton.Text = "Desativar Farm"
        toggleButton.BackgroundColor3 = Color3.fromRGB(100, 0, 0)
        
        -- Inicia o loop de farm em uma thread separada
        farmConnection = RunService.Heartbeat:Connect(function()
            if isFarming then
                pcall(startFarm) -- pcall para evitar erros que quebram o script
            end
        end)
    else
        statusLabel.Text = "Status: Inativo"
        toggleButton.Text = "Ativar Farm"
        toggleButton.BackgroundColor3 = Color3.fromRGB(40, 40, 40)
        
        -- Para o loop de farm
        if farmConnection then
            farmConnection:Disconnect()
            farmConnection = nil
        end
    end
end)
